# Repository Directory

Complete guide to all repositories in the AWS Services Dashboard ecosystem.

## Repository Overview

| Repository | Primary Language | Role | Lines of Code | Status |
|------------|-----------------|------|---------------|--------|
| aws-services-site | JavaScript/React | Frontend Dashboard | ~5,000 | ✅ Active |
| aws-infrastructure-fetcher | JavaScript/Node.js | Data Collection | ~2,000 | ✅ Active |
| nodejs-aws-reporter | JavaScript/Node.js | Report Generation | ~1,500 | ✅ Active |
| synepho-s3cf-site | HCL/Terraform | Infrastructure | ~1,000 | ✅ Active |

---

## 1. aws-services-site

**Frontend Dashboard - React Application**

### Repository Information
- **GitHub**: https://github.com/jxman/aws-services-site
- **Production URL**: https://aws-services.synepho.com
- **Primary Language**: JavaScript (React)
- **Build Tool**: Vite
- **Package Manager**: npm

### Purpose
User-facing web application that displays AWS infrastructure data in an intuitive dashboard with visualizations and downloadable reports.

### Key Technologies
- React 18.3+
- Vite 6.0+ (build tool)
- React Router 7+ (navigation)
- TanStack Query (data fetching)
- Recharts (data visualization)
- Tailwind CSS (styling)
- date-fns (date formatting)

### Project Structure
```
aws-services-site/
├── public/               # Static assets
├── src/
│   ├── components/      # Reusable UI components
│   ├── views/          # Page components
│   ├── hooks/          # Custom React hooks
│   ├── App.jsx         # Main application component
│   └── main.jsx        # Application entry point
├── index.html          # HTML template
├── package.json        # Dependencies
├── vite.config.js      # Build configuration
└── tailwind.config.js  # Tailwind CSS configuration
```

### Dependencies Integration
- **Consumes**: JSON data from S3 (via CloudFront)
- **Triggers**: Reporter Lambda for Excel generation
- **Hosted on**: Infrastructure from synepho-s3cf-site

### Development Setup
```bash
git clone https://github.com/jxman/aws-services-site.git
cd aws-services-site
npm install
npm run dev
```

### Deployment
Automated via GitHub Actions - pushes to main branch trigger build and deploy to S3.

### Key Features
- Real-time data visualization
- Regional resource distribution
- Account-level insights
- Excel report downloads
- Responsive mobile design

---

## 2. aws-infrastructure-fetcher

**Data Collection Lambda - Node.js**

### Repository Information
- **GitHub**: https://github.com/jxman/aws-infrastructure-fetcher
- **Execution Environment**: AWS Lambda (Node.js 18+)
- **Primary Language**: JavaScript (Node.js)
- **Trigger**: EventBridge Schedule
- **Package Manager**: npm

### Purpose
Serverless function that collects AWS infrastructure data across multiple accounts and regions, formats it as JSON, and publishes to S3 for frontend consumption.

### Key Technologies
- Node.js 18+
- AWS SDK v3
- AWS Lambda runtime
- EventBridge (scheduling)
- IAM role assumption (multi-account)

### Project Structure
```
aws-infrastructure-fetcher/
├── src/
│   ├── collectors/     # Service-specific data collectors
│   ├── utils/         # Helper functions
│   ├── config/        # Configuration management
│   └── index.js       # Lambda handler
├── package.json       # Dependencies
├── template.yaml      # SAM template (if used)
└── README.md          # Documentation
```

### Dependencies Integration
- **Reads from**: AWS services (EC2, RDS, S3, Lambda, etc.)
- **Writes to**: S3 data bucket (JSON output)
- **Triggers**: CloudFront invalidation (optional)
- **Deployed by**: Infrastructure from synepho-s3cf-site

### Execution Flow
1. Triggered by EventBridge schedule (e.g., every 15 minutes)
2. Assumes IAM roles for each monitored AWS account
3. Queries AWS services across all regions
4. Aggregates and formats data
5. Uploads JSON to S3 bucket
6. Logs execution metrics to CloudWatch

### Environment Variables
- `DISTRIBUTION_BUCKET`: S3 bucket for JSON output
- `DISTRIBUTION_PREFIX`: S3 key prefix
- `CLOUDFRONT_DISTRIBUTION_ID`: For cache invalidation
- `AWS_ACCOUNTS`: JSON array of accounts to monitor

### Development Setup
```bash
git clone https://github.com/jxman/aws-infrastructure-fetcher.git
cd aws-infrastructure-fetcher
npm install
# Local testing with SAM
sam local invoke -e events/scheduled-event.json
```

### Deployment
Deployed via AWS SAM or Terraform as part of infrastructure deployment.

---

## 3. nodejs-aws-reporter

**Excel Report Generator - Node.js Lambda**

### Repository Information
- **GitHub**: https://github.com/jxman/nodejs-aws-reporter
- **Execution Environment**: AWS Lambda (Node.js 18+)
- **Primary Language**: JavaScript (Node.js)
- **Trigger**: User action / API Gateway
- **Package Manager**: npm

### Purpose
Generates formatted Excel workbooks from collected AWS data, allowing users to download comprehensive reports with multiple sheets and visualizations.

### Key Technologies
- Node.js 18+
- ExcelJS (Excel generation)
- AWS SDK v3
- AWS Lambda runtime

### Project Structure
```
nodejs-aws-reporter/
├── src/
│   ├── generators/    # Report sheet generators
│   ├── formatters/    # Data formatting utilities
│   ├── styles/        # Excel styling definitions
│   └── index.js       # Lambda handler
├── package.json       # Dependencies
├── template.yaml      # SAM template (if used)
└── README.md          # Documentation
```

### Dependencies Integration
- **Reads from**: S3 data bucket (JSON files)
- **Returns to**: Frontend (Excel file download)
- **Deployed by**: Infrastructure from synepho-s3cf-site

### Report Structure
The generated Excel workbook includes:
- **Summary Sheet**: High-level overview and key metrics
- **Accounts Sheet**: Detailed account information
- **Regions Sheet**: Regional resource distribution
- **Services Sheet**: Service-by-service breakdown
- **Costs Sheet**: Cost analysis and trends

### Development Setup
```bash
git clone https://github.com/jxman/nodejs-aws-reporter.git
cd nodejs-aws-reporter
npm install
# Local testing with SAM
sam local invoke -e events/report-request.json
```

### Deployment
Deployed via AWS SAM or Terraform as part of infrastructure deployment.

---

## 4. synepho-s3cf-site

**Infrastructure as Code - Terraform**

### Repository Information
- **GitHub**: https://github.com/jxman/synepho-s3cf-site
- **Primary Language**: HCL (Terraform)
- **Terraform Version**: 1.0+
- **AWS Provider**: 5.0+

### Purpose
Complete infrastructure definition and deployment automation for the AWS Services Dashboard, including hosting, Lambda functions, networking, and CI/CD.

### Key Technologies
- Terraform 1.0+
- AWS Provider
- GitHub Actions (CI/CD)
- OIDC authentication

### Project Structure
```
synepho-s3cf-site/
├── environments/
│   ├── prod/          # Production environment
│   └── dev/           # Development environment (optional)
├── modules/
│   ├── s3-cloudfront/ # Static hosting module
│   ├── lambda/        # Lambda function module
│   ├── github-oidc/   # OIDC provider module
│   └── monitoring/    # CloudWatch monitoring module
├── main.tf            # Root module
├── variables.tf       # Input variables
├── outputs.tf         # Output values
├── versions.tf        # Provider versions
└── terraform.tfvars   # Variable values (gitignored)
```

### Managed Resources

#### Hosting Infrastructure
- **S3 Buckets**:
  - Static website bucket
  - Data distribution bucket
  - Logging bucket
- **CloudFront**:
  - Distribution configuration
  - SSL certificates (ACM)
  - Origin Access Control
- **Route53**:
  - DNS records for aws-services.synepho.com
  - DNS records for synepho.com

#### Lambda Infrastructure
- Lambda function deployments
- IAM roles and policies
- EventBridge schedules
- CloudWatch log groups

#### CI/CD Infrastructure
- GitHub OIDC provider
- GitHub Actions IAM roles
- Deployment policies (least privilege)

#### Monitoring
- CloudWatch dashboards
- CloudWatch alarms
- Log aggregation

### Dependencies Integration
- **Deploys**: All infrastructure for other repositories
- **Manages**: DNS, SSL, hosting, Lambda functions
- **Provides**: Resource outputs for other components

### State Management
- **Backend**: S3 bucket with encryption
- **Locking**: DynamoDB table
- **Workspaces**: Separate environments (dev, prod)

### Development Workflow
```bash
git clone https://github.com/jxman/synepho-s3cf-site.git
cd synepho-s3cf-site

# Initialize Terraform
terraform init

# Plan changes
terraform plan

# Apply changes (via GitHub Actions only in production)
# Local development only for testing/validation
```

### Deployment Policy
**CRITICAL**: All production deployments MUST use GitHub Actions workflows. Local deployments are deprecated and archived.

```bash
# Production deployment (required method)
gh workflow run "Terraform Deployment" --ref main
gh run list --limit 5
gh run view [RUN_ID]
```

---

## Cross-Repository Dependencies

### Data Flow
```
┌──────────────────────┐
│  aws-infrastructure- │
│      fetcher         │ ─────▶ Writes JSON to S3
└──────────────────────┘
           │
           │ (deployed by)
           ▼
┌──────────────────────┐
│  synepho-s3cf-site   │ ─────▶ Manages all infrastructure
│    (Terraform)       │
└──────────────────────┘
           │
           │ (hosts)
           ▼
┌──────────────────────┐
│  aws-services-site   │ ─────▶ Reads JSON from S3
│      (React)         │        Displays to users
└──────────────────────┘
           │
           │ (triggers)
           ▼
┌──────────────────────┐
│  nodejs-aws-reporter │ ─────▶ Generates Excel from JSON
│   (Report Lambda)    │
└──────────────────────┘
```

### Version Compatibility

| Component | Node.js | AWS SDK | Terraform | React |
|-----------|---------|---------|-----------|-------|
| Frontend | N/A | N/A | N/A | 18.3+ |
| Data Fetcher | 18+ | v3 | N/A | N/A |
| Reporter | 18+ | v3 | N/A | N/A |
| Infrastructure | N/A | N/A | 1.0+ | N/A |

---

## Development Guidelines

### Branching Strategy
- **main**: Production-ready code
- **develop**: Integration branch (optional)
- **feature/***: Feature branches
- **hotfix/***: Emergency fixes

### Commit Messages
Follow conventional commits:
- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation changes
- `refactor:` Code refactoring
- `test:` Test additions/changes
- `chore:` Maintenance tasks

### Pull Request Process
1. Create feature branch from main
2. Implement changes with tests
3. Update documentation
4. Submit PR with description
5. CI/CD checks must pass
6. Code review required
7. Merge to main

### Testing Requirements
- **Frontend**: Unit tests (Vitest), E2E tests (Playwright - optional)
- **Lambda**: Unit tests (Jest), Integration tests
- **Infrastructure**: Terraform validation and plan checks

---

## Getting Help

### Documentation
- Project hub: This repository
- Architecture: [ARCHITECTURE.md](./ARCHITECTURE.md)
- Development: [DEVELOPMENT.md](./DEVELOPMENT.md)
- Deployment: [DEPLOYMENT.md](./DEPLOYMENT.md)

### Issues and Questions
- Create issues in the relevant repository
- Tag with appropriate labels
- Provide context and reproduction steps

### Contributing
This is a private project. Contact the maintainer for access and contribution guidelines.
