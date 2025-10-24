# Deployment Guide

Complete deployment procedures for the AWS Services Dashboard ecosystem.

## Table of Contents
- [Deployment Overview](#deployment-overview)
- [GitHub Actions CI/CD](#github-actions-cicd)
- [Infrastructure Deployment](#infrastructure-deployment)
- [Frontend Deployment](#frontend-deployment)
- [Lambda Deployment](#lambda-deployment)
- [Rollback Procedures](#rollback-procedures)
- [Monitoring](#monitoring)

---

## Deployment Overview

### Deployment Philosophy

**CRITICAL: All production deployments MUST use GitHub Actions workflows.**

- ✅ **GitHub Actions**: Required for all production deployments
- ❌ **Local deployments**: Deprecated and archived
- ✅ **OIDC authentication**: Secure, no long-lived credentials
- ✅ **Audit trail**: Complete deployment history in GitHub

### Deployment Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    GitHub Repository                         │
│                                                              │
│  Developer pushes to main ───▶ GitHub Actions Triggered     │
└──────────────────────┬───────────────────────────────────────┘
                       │
                       │ OIDC Authentication
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                     AWS Account                              │
│                                                              │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐   │
│  │   Terraform  │   │   Lambda     │   │   S3/CF      │   │
│  │   Modules    │───│  Functions   │───│   Hosting    │   │
│  └──────────────┘   └──────────────┘   └──────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### Deployment Flow

1. **Code Change**: Developer commits to feature branch
2. **Pull Request**: Creates PR, triggers preview/validation
3. **Code Review**: Team reviews changes
4. **Merge to Main**: Triggers production deployment workflow
5. **GitHub Actions**: Authenticates via OIDC, deploys infrastructure/code
6. **Verification**: Automated tests and health checks
7. **Notification**: Deployment status reported

---

## GitHub Actions CI/CD

### OIDC Authentication Setup

#### Why OIDC?
- ✅ No long-lived AWS access keys in GitHub secrets
- ✅ Temporary credentials with automatic rotation
- ✅ Least privilege access per workflow
- ✅ Complete audit trail in CloudTrail
- ✅ Repository isolation (can't be used by other repos)

#### OIDC Resources (Already Deployed)

**IAM Role**: `GithubActionsOIDC-[ProjectName]-Role`
**IAM Policy**: `GithubActions-[ProjectName]-Policy`
**OIDC Provider**: `token.actions.githubusercontent.com`

Each repository has its own dedicated IAM role with repository-specific trust policy.

#### Verifying OIDC Setup

```bash
# Check if OIDC provider exists
aws iam list-open-id-connect-providers

# Check role trust policy
aws iam get-role --role-name GithubActionsOIDC-YourProject-Role

# Verify role permissions
aws iam get-role-policy --role-name GithubActionsOIDC-YourProject-Role --policy-name inline-policy
```

### GitHub Actions Workflows

#### Infrastructure Deployment Workflow

Location: `.github/workflows/terraform-deploy.yml`

```yaml
name: Terraform Deployment

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:

permissions:
  id-token: write
  contents: read

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::${{ secrets.AWS_ACCOUNT_ID }}:role/GithubActionsOIDC-ProjectName-Role
          aws-region: us-east-1
          role-session-name: GithubActionsOIDCSession

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.5.0

      - name: Terraform Init
        run: terraform init

      - name: Terraform Format Check
        run: terraform fmt -check -recursive

      - name: Terraform Validate
        run: terraform validate

      - name: Terraform Plan
        run: terraform plan -out=tfplan

      - name: Terraform Apply
        if: github.ref == 'refs/heads/main' && github.event_name == 'push'
        run: terraform apply -auto-approve tfplan

      - name: Output Important Values
        if: github.ref == 'refs/heads/main' && github.event_name == 'push'
        run: |
          echo "CloudFront Distribution: $(terraform output -raw cloudfront_distribution_id)"
          echo "Dashboard URL: $(terraform output -raw dashboard_url)"
```

#### Frontend Deployment Workflow

Location: `.github/workflows/deploy-frontend.yml`

```yaml
name: Deploy Frontend

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  id-token: write
  contents: read

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build application
        run: npm run build

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::${{ secrets.AWS_ACCOUNT_ID }}:role/GithubActionsOIDC-FrontendDeploy-Role
          aws-region: us-east-1

      - name: Deploy to S3
        run: |
          aws s3 sync dist/ s3://${{ secrets.S3_BUCKET_NAME }}/ --delete

      - name: Invalidate CloudFront
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} \
            --paths "/*"
```

#### Lambda Deployment Workflow

Location: `.github/workflows/deploy-lambda.yml`

```yaml
name: Deploy Lambda Functions

on:
  push:
    branches: [main]
    paths:
      - 'src/**'
      - 'package.json'
  workflow_dispatch:

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci --production

      - name: Create deployment package
        run: |
          zip -r function.zip src/ node_modules/ package.json

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::${{ secrets.AWS_ACCOUNT_ID }}:role/GithubActionsOIDC-LambdaDeploy-Role
          aws-region: us-east-1

      - name: Deploy to Lambda
        run: |
          aws lambda update-function-code \
            --function-name ${{ secrets.LAMBDA_FUNCTION_NAME }} \
            --zip-file fileb://function.zip

      - name: Wait for update to complete
        run: |
          aws lambda wait function-updated \
            --function-name ${{ secrets.LAMBDA_FUNCTION_NAME }}

      - name: Publish new version
        run: |
          aws lambda publish-version \
            --function-name ${{ secrets.LAMBDA_FUNCTION_NAME }}
```

### Required GitHub Secrets

Each repository needs specific secrets configured:

#### Infrastructure Repository
```
AWS_ACCOUNT_ID: Your AWS account ID
TERRAFORM_STATE_BUCKET: S3 bucket for Terraform state
TERRAFORM_LOCK_TABLE: DynamoDB table for state locking
```

#### Frontend Repository
```
AWS_ACCOUNT_ID: Your AWS account ID
S3_BUCKET_NAME: Target S3 bucket for deployment
CLOUDFRONT_DISTRIBUTION_ID: CloudFront distribution ID
```

#### Lambda Repositories
```
AWS_ACCOUNT_ID: Your AWS account ID
LAMBDA_FUNCTION_NAME: Target Lambda function name
```

### Triggering Deployments

#### Via Git Push
```bash
# Commit your changes
git add .
git commit -m "feat: add new feature"

# Push to main (triggers deployment)
git push origin main
```

#### Via GitHub CLI (Manual Trigger)
```bash
# Trigger workflow manually
gh workflow run "Terraform Deployment" --ref main

# Monitor workflow
gh run list --limit 5

# View specific run
gh run view [RUN_ID]

# Watch run in real-time
gh run watch [RUN_ID]

# Open in browser
gh run view [RUN_ID] --web
```

#### Via GitHub UI
1. Navigate to repository
2. Click "Actions" tab
3. Select workflow
4. Click "Run workflow"
5. Select branch (usually `main`)
6. Click "Run workflow" button

---

## Infrastructure Deployment

### Prerequisites

- ✅ Terraform backend configured (S3 + DynamoDB)
- ✅ OIDC provider created in AWS
- ✅ IAM roles created for GitHub Actions
- ✅ GitHub repository secrets configured

### Deployment Steps

#### 1. Initial Infrastructure Setup

```bash
# Clone infrastructure repository
git clone https://github.com/jxman/synepho-s3cf-site.git
cd synepho-s3cf-site

# Initialize Terraform (local validation only)
terraform init

# Validate configuration
terraform validate

# Format code
terraform fmt -recursive

# Review plan locally (optional)
terraform plan

# Deploy via GitHub Actions (REQUIRED for production)
git add .
git commit -m "chore: infrastructure updates"
git push origin main

# Or trigger manually
gh workflow run "Terraform Deployment" --ref main
```

#### 2. Monitor Deployment

```bash
# Watch workflow
gh run watch

# Check outputs after deployment
gh run view --log
```

#### 3. Verify Deployment

```bash
# Check CloudFront distribution
aws cloudfront get-distribution --id $(terraform output -raw cloudfront_distribution_id)

# Check S3 buckets
aws s3 ls

# Check Lambda functions
aws lambda list-functions
```

### Infrastructure Updates

#### Adding New Resources

1. **Create/modify Terraform files**
2. **Validate locally**:
   ```bash
   terraform validate
   terraform fmt -recursive
   ```
3. **Commit and push**:
   ```bash
   git add .
   git commit -m "feat: add new resource"
   git push origin main
   ```
4. **Monitor GitHub Actions**
5. **Verify deployment**

#### Modifying Existing Resources

1. **Update Terraform configuration**
2. **Review plan locally** (optional):
   ```bash
   terraform plan
   ```
3. **Commit and push** (triggers GitHub Actions)
4. **Monitor deployment**
5. **Verify changes**

---

## Frontend Deployment

### Build Process

#### Local Build (Testing)
```bash
cd aws-services-site

# Install dependencies
npm ci

# Build for production
npm run build

# Test build locally
npm run preview
```

#### Production Build (Automated via GitHub Actions)
- Triggered on push to `main` branch
- Runs `npm ci` for clean install
- Runs `npm run build` to create production bundle
- Uploads to S3
- Invalidates CloudFront cache

### Deployment Steps

1. **Make changes to code**
2. **Test locally**:
   ```bash
   npm run dev
   # Verify changes at http://localhost:5173
   ```
3. **Commit and push**:
   ```bash
   git add .
   git commit -m "feat: new feature"
   git push origin main
   ```
4. **Monitor deployment**:
   ```bash
   gh run watch
   ```
5. **Verify at production URL**:
   ```bash
   open https://aws-services.synepho.com
   ```

### Cache Invalidation

After deployment, CloudFront cache is automatically invalidated:

```bash
# Manual invalidation (if needed)
aws cloudfront create-invalidation \
  --distribution-id E1234567890ABC \
  --paths "/*"

# Check invalidation status
aws cloudfront get-invalidation \
  --distribution-id E1234567890ABC \
  --id INVALIDATION_ID
```

---

## Lambda Deployment

### Data Fetcher Deployment

```bash
cd aws-infrastructure-fetcher

# Install production dependencies
npm ci --production

# Create deployment package
zip -r function.zip src/ node_modules/ package.json

# Deploy via GitHub Actions (push to main)
git add .
git commit -m "feat: update data collection"
git push origin main

# Or deploy manually (if needed)
aws lambda update-function-code \
  --function-name aws-infrastructure-fetcher \
  --zip-file fileb://function.zip
```

### Reporter Deployment

```bash
cd nodejs-aws-reporter

# Install production dependencies
npm ci --production

# Create deployment package
zip -r function.zip src/ node_modules/ package.json

# Deploy via GitHub Actions
git push origin main
```

### Environment Variables

Update Lambda environment variables:

```bash
# Update environment variables
aws lambda update-function-configuration \
  --function-name your-function-name \
  --environment "Variables={
    DISTRIBUTION_BUCKET=new-bucket-name,
    DISTRIBUTION_PREFIX=data/,
    CLOUDFRONT_DISTRIBUTION_ID=E1234567890ABC
  }"

# Or via Terraform (recommended)
# Update terraform variables and deploy
```

---

## Rollback Procedures

### Infrastructure Rollback

```bash
# Option 1: Revert Terraform state to previous version
terraform state pull > current-state.json
aws s3 cp s3://terraform-state-bucket/terraform.tfstate.backup ./previous-state.json
terraform state push previous-state.json

# Option 2: Revert Git commit and redeploy
git revert HEAD
git push origin main
# GitHub Actions will deploy previous state

# Option 3: Restore from specific commit
git reset --hard <previous-commit-hash>
git push --force origin main
```

### Frontend Rollback

```bash
# Option 1: Redeploy previous version from Git
git revert HEAD
git push origin main
# GitHub Actions will build and deploy previous version

# Option 2: Restore S3 bucket from versioning
aws s3api list-object-versions \
  --bucket your-bucket-name \
  --prefix index.html

# Restore specific version
aws s3api copy-object \
  --copy-source your-bucket-name/index.html?versionId=VERSION_ID \
  --bucket your-bucket-name \
  --key index.html

# Invalidate CloudFront
aws cloudfront create-invalidation \
  --distribution-id E1234567890ABC \
  --paths "/*"
```

### Lambda Rollback

```bash
# List previous versions
aws lambda list-versions-by-function \
  --function-name your-function-name

# Update alias to previous version
aws lambda update-alias \
  --function-name your-function-name \
  --name production \
  --function-version <previous-version-number>

# Or revert code and redeploy
git revert HEAD
git push origin main
```

---

## Monitoring

### Deployment Health Checks

#### After Infrastructure Deployment
```bash
# Check Terraform outputs
terraform output

# Verify CloudFront distribution
aws cloudfront get-distribution \
  --id $(terraform output -raw cloudfront_distribution_id) \
  --query 'Distribution.Status'

# Check S3 buckets
aws s3 ls

# Verify Lambda functions
aws lambda list-functions \
  --query 'Functions[].FunctionName'
```

#### After Frontend Deployment
```bash
# Check website availability
curl -I https://aws-services.synepho.com

# Verify CloudFront cache status
aws cloudfront get-distribution \
  --id E1234567890ABC \
  --query 'Distribution.{Status:Status,LastModified:LastModifiedTime}'

# Check S3 bucket contents
aws s3 ls s3://your-bucket-name/ --recursive
```

#### After Lambda Deployment
```bash
# Check function configuration
aws lambda get-function-configuration \
  --function-name your-function-name

# Test function
aws lambda invoke \
  --function-name your-function-name \
  --payload '{}' \
  response.json

# Check recent executions
aws logs tail /aws/lambda/your-function-name --follow
```

### CloudWatch Monitoring

```bash
# View Lambda logs
aws logs tail /aws/lambda/your-function-name --follow --since 1h

# Check Lambda metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Invocations \
  --dimensions Name=FunctionName,Value=your-function-name \
  --start-time 2025-01-24T00:00:00Z \
  --end-time 2025-01-24T23:59:59Z \
  --period 3600 \
  --statistics Sum

# Check CloudFront metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/CloudFront \
  --metric-name Requests \
  --dimensions Name=DistributionId,Value=E1234567890ABC \
  --start-time 2025-01-24T00:00:00Z \
  --end-time 2025-01-24T23:59:59Z \
  --period 3600 \
  --statistics Sum
```

### GitHub Actions Monitoring

```bash
# List recent runs
gh run list --limit 10

# View specific run
gh run view <run-id>

# Download logs
gh run view <run-id> --log

# Watch run in real-time
gh run watch <run-id>
```

---

## Troubleshooting Deployments

### Common Issues

#### OIDC Authentication Failures
```
Error: Failed to assume role with OIDC
```

**Solution**:
1. Verify OIDC provider exists
2. Check IAM role trust policy
3. Verify repository name in trust policy
4. Ensure workflow has `id-token: write` permission

#### Terraform State Lock
```
Error: Error acquiring the state lock
```

**Solution**:
```bash
# Check for existing lock
aws dynamodb get-item \
  --table-name terraform-state-lock \
  --key '{"LockID":{"S":"your-state-file"}}'

# Force unlock (use carefully!)
terraform force-unlock <lock-id>
```

#### CloudFront Invalidation Timeout
```
Error: Invalidation is taking too long
```

**Solution**:
- Invalidations can take 5-15 minutes
- Wait for completion: `aws cloudfront wait invalidation-completed`
- Don't create too many invalidations (limit: 3000/month free)

#### Lambda Deployment Failures
```
Error: ResourceConflictException: The operation cannot be performed at this time
```

**Solution**:
```bash
# Wait for previous update to complete
aws lambda wait function-updated \
  --function-name your-function-name

# Then retry deployment
```

---

## Best Practices

### Deployment Best Practices
- ✅ Always use GitHub Actions for production deployments
- ✅ Test changes locally before pushing
- ✅ Use pull requests for code review
- ✅ Tag releases with semantic versioning
- ✅ Monitor deployments until completion
- ✅ Verify functionality after deployment
- ✅ Have rollback plan ready

### Security Best Practices
- ✅ Use OIDC authentication (no long-lived credentials)
- ✅ Implement least privilege IAM policies
- ✅ Enable CloudTrail for audit logging
- ✅ Rotate secrets regularly
- ✅ Use encrypted S3 buckets
- ✅ Enable MFA for sensitive operations

### Operational Best Practices
- ✅ Document all manual steps
- ✅ Automate repetitive tasks
- ✅ Monitor CloudWatch metrics and logs
- ✅ Set up CloudWatch alarms
- ✅ Maintain runbooks for common issues
- ✅ Regular backup verification

---

## Emergency Procedures

### Complete System Outage

1. **Check CloudFront status**
2. **Verify S3 bucket availability**
3. **Check Lambda function health**
4. **Review CloudWatch alarms**
5. **Check AWS Service Health Dashboard**
6. **Implement rollback if needed**
7. **Communicate status to stakeholders**

### Data Loss

1. **Stop all write operations**
2. **Identify scope of data loss**
3. **Restore from S3 versioning**
4. **Re-run data collection Lambda**
5. **Verify data integrity**
6. **Resume normal operations**

### Security Incident

1. **Identify compromised resources**
2. **Rotate all credentials immediately**
3. **Review CloudTrail logs**
4. **Disable compromised access**
5. **Patch vulnerabilities**
6. **Conduct post-incident review**

---

## Resources

- [AWS CloudFormation Best Practices](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/best-practices.html)
- [Terraform Best Practices](https://www.terraform.io/docs/cloud/guides/recommended-practices/index.html)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
