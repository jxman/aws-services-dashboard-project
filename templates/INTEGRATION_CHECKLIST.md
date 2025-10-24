# Integration Checklist Template

Use this checklist when making changes that span multiple repositories to ensure proper integration and coordination.

---

## Cross-Repository Change Checklist

### Planning Phase

- [ ] **Identify affected repositories**
  - [ ] Frontend (aws-services-site)
  - [ ] Data Fetcher (aws-infrastructure-fetcher)
  - [ ] Reporter (nodejs-aws-reporter)
  - [ ] Infrastructure (synepho-s3cf-site)
  - [ ] Documentation (aws-services-dashboard-project)

- [ ] **Document the change**
  - [ ] Update hub repository ARCHITECTURE.md if architecture changes
  - [ ] Update hub repository DEVELOPMENT.md if workflow changes
  - [ ] Update hub repository DEPLOYMENT.md if deployment changes
  - [ ] Update hub repository REPOSITORIES.md if integration points change

- [ ] **Define integration points**
  - [ ] Data format changes (JSON schema)
  - [ ] API contract changes
  - [ ] Environment variable changes
  - [ ] IAM permission changes
  - [ ] Infrastructure resource changes

### Implementation Phase

#### Infrastructure Changes (if applicable)

- [ ] **Update Terraform configuration**
  - [ ] Modify or create Terraform modules
  - [ ] Update variables and outputs
  - [ ] Test with `terraform plan`
  - [ ] Document new resources

- [ ] **Deploy infrastructure**
  - [ ] Commit changes to synepho-s3cf-site
  - [ ] Push to trigger GitHub Actions
  - [ ] Monitor deployment
  - [ ] Verify resources created

#### Data Fetcher Changes (if applicable)

- [ ] **Update data collection logic**
  - [ ] Modify collectors or add new ones
  - [ ] Update data format (JSON schema)
  - [ ] Update environment variables
  - [ ] Test locally if possible

- [ ] **Deploy Lambda function**
  - [ ] Commit changes to aws-infrastructure-fetcher
  - [ ] Push to trigger GitHub Actions
  - [ ] Monitor deployment
  - [ ] Test execution
  - [ ] Verify S3 output

#### Reporter Changes (if applicable)

- [ ] **Update report generation logic**
  - [ ] Modify sheet generators
  - [ ] Update data parsing
  - [ ] Test Excel output locally
  - [ ] Update environment variables

- [ ] **Deploy Lambda function**
  - [ ] Commit changes to nodejs-aws-reporter
  - [ ] Push to trigger GitHub Actions
  - [ ] Monitor deployment
  - [ ] Test report generation

#### Frontend Changes (if applicable)

- [ ] **Update React components**
  - [ ] Modify data fetching logic
  - [ ] Update components to handle new data
  - [ ] Update visualizations if needed
  - [ ] Test locally

- [ ] **Deploy frontend**
  - [ ] Commit changes to aws-services-site
  - [ ] Push to trigger GitHub Actions
  - [ ] Monitor deployment
  - [ ] Verify CloudFront invalidation
  - [ ] Test in production

### Testing Phase

- [ ] **Unit tests**
  - [ ] Run tests in each affected repository
  - [ ] Add new tests for new functionality
  - [ ] Ensure all tests pass

- [ ] **Integration tests**
  - [ ] Test data flow from Data Fetcher to S3
  - [ ] Test data flow from S3 to Frontend
  - [ ] Test report generation from S3 data
  - [ ] Test end-to-end user workflow

- [ ] **CloudWatch monitoring**
  - [ ] Check Lambda execution logs
  - [ ] Verify no errors in logs
  - [ ] Check CloudWatch metrics
  - [ ] Verify alarms not triggered

### Documentation Phase

- [ ] **Update component README files**
  - [ ] Document new features
  - [ ] Update setup instructions
  - [ ] Update environment variables list
  - [ ] Update deployment instructions

- [ ] **Update hub repository documentation**
  - [ ] Update ARCHITECTURE.md with changes
  - [ ] Update REPOSITORIES.md integration points
  - [ ] Update DEVELOPMENT.md if workflow changed
  - [ ] Update DEPLOYMENT.md if deployment changed

- [ ] **Update CLAUDE.md files**
  - [ ] Update component repository CLAUDE.md
  - [ ] Update hub repository CLAUDE.md
  - [ ] Document new patterns or conventions

### Deployment Phase

- [ ] **Deploy in correct order**
  1. [ ] Infrastructure first (if changes needed)
  2. [ ] Backend services (Data Fetcher, Reporter)
  3. [ ] Frontend last (to consume new backend)

- [ ] **Monitor each deployment**
  - [ ] Watch GitHub Actions logs
  - [ ] Check AWS CloudWatch logs
  - [ ] Verify resources created/updated
  - [ ] Test functionality after each step

- [ ] **Verify production**
  - [ ] Test production website
  - [ ] Verify data is up-to-date
  - [ ] Test report generation
  - [ ] Check all visualizations

### Post-Deployment Phase

- [ ] **Verify monitoring**
  - [ ] Check CloudWatch dashboards
  - [ ] Verify alarms configured
  - [ ] Check log aggregation
  - [ ] Monitor error rates

- [ ] **Update documentation**
  - [ ] Mark deployment as complete in docs
  - [ ] Document any issues encountered
  - [ ] Update runbooks if needed
  - [ ] Create post-mortem if issues occurred

- [ ] **Clean up**
  - [ ] Remove temporary files
  - [ ] Archive old documentation
  - [ ] Update changelog
  - [ ] Tag release if appropriate

---

## Data Format Change Checklist

Use this when changing the JSON data structure between components.

### Planning

- [ ] **Document current format**
  - [ ] Save example of current JSON structure
  - [ ] Document what will change
  - [ ] Consider backward compatibility

- [ ] **Design new format**
  - [ ] Create example of new JSON structure
  - [ ] Document migration path
  - [ ] Consider versioning strategy

### Implementation

- [ ] **Update Data Fetcher**
  - [ ] Modify data collection to output new format
  - [ ] Test JSON output
  - [ ] Document new schema

- [ ] **Update Frontend**
  - [ ] Update data parsing logic
  - [ ] Handle missing fields gracefully
  - [ ] Update TypeScript types (if used)
  - [ ] Test with new data format

- [ ] **Update Reporter**
  - [ ] Update Excel generation to use new format
  - [ ] Test report output
  - [ ] Verify all sheets render correctly

### Testing

- [ ] **Test data compatibility**
  - [ ] Frontend can parse Data Fetcher output
  - [ ] Reporter can parse Data Fetcher output
  - [ ] No errors in CloudWatch logs
  - [ ] All visualizations render correctly

### Deployment

- [ ] **Deploy in correct order**
  1. [ ] Deploy Data Fetcher (outputs new format)
  2. [ ] Verify JSON in S3 is correct
  3. [ ] Deploy Reporter (reads new format)
  4. [ ] Test report generation
  5. [ ] Deploy Frontend (reads new format)
  6. [ ] Test production website

---

## Infrastructure Change Checklist

Use this when adding or modifying AWS resources.

### Planning

- [ ] **Identify new resources**
  - [ ] List AWS resources to create/modify
  - [ ] Document resource dependencies
  - [ ] Estimate costs
  - [ ] Plan IAM permissions

- [ ] **Design Terraform modules**
  - [ ] Create or modify modules
  - [ ] Define variables
  - [ ] Define outputs
  - [ ] Document usage

### Implementation

- [ ] **Write Terraform code**
  - [ ] Create resource definitions
  - [ ] Configure variables
  - [ ] Define outputs
  - [ ] Add tags

- [ ] **Test locally**
  - [ ] Run `terraform validate`
  - [ ] Run `terraform fmt -recursive`
  - [ ] Run `terraform plan`
  - [ ] Review plan output

### Deployment

- [ ] **Deploy via GitHub Actions**
  - [ ] Commit Terraform changes
  - [ ] Push to trigger workflow
  - [ ] Monitor GitHub Actions
  - [ ] Review Terraform apply output

- [ ] **Verify resources**
  - [ ] Check AWS Console
  - [ ] Verify resource configuration
  - [ ] Test resource connectivity
  - [ ] Verify tags applied

### Post-Deployment

- [ ] **Update documentation**
  - [ ] Document new resources
  - [ ] Update architecture diagrams
  - [ ] Update REPOSITORIES.md
  - [ ] Update DEPLOYMENT.md

- [ ] **Update IAM permissions**
  - [ ] Grant necessary permissions to services
  - [ ] Update GitHub Actions roles if needed
  - [ ] Test permissions

---

## Lambda Function Update Checklist

Use this when modifying Lambda functions.

### Planning

- [ ] **Identify changes**
  - [ ] Code changes
  - [ ] Dependency changes
  - [ ] Environment variable changes
  - [ ] IAM permission changes
  - [ ] Memory/timeout changes

### Implementation

- [ ] **Update code**
  - [ ] Modify Lambda handler
  - [ ] Update dependencies in package.json
  - [ ] Run `npm install`
  - [ ] Test locally if possible

- [ ] **Update configuration**
  - [ ] Update environment variables
  - [ ] Update timeout/memory in Terraform
  - [ ] Update IAM permissions in Terraform

### Testing

- [ ] **Local testing**
  - [ ] Run unit tests
  - [ ] Test with sample events
  - [ ] Verify error handling

- [ ] **Integration testing**
  - [ ] Test with AWS services
  - [ ] Verify S3 writes (for Data Fetcher)
  - [ ] Verify Excel generation (for Reporter)

### Deployment

- [ ] **Deploy configuration first** (if needed)
  - [ ] Deploy Terraform changes
  - [ ] Verify environment variables
  - [ ] Verify IAM permissions

- [ ] **Deploy code**
  - [ ] Commit code changes
  - [ ] Push to trigger GitHub Actions
  - [ ] Monitor deployment
  - [ ] Wait for update to complete

### Verification

- [ ] **Test execution**
  - [ ] Invoke function manually
  - [ ] Check CloudWatch logs
  - [ ] Verify outputs (S3 for fetcher, Excel for reporter)
  - [ ] Monitor error rates

---

## Frontend Update Checklist

Use this when modifying the React frontend.

### Planning

- [ ] **Identify changes**
  - [ ] New components
  - [ ] Modified components
  - [ ] New dependencies
  - [ ] Data fetching changes

### Implementation

- [ ] **Update code**
  - [ ] Create/modify components
  - [ ] Update routing
  - [ ] Update data fetching
  - [ ] Update styling

- [ ] **Test locally**
  - [ ] Run development server
  - [ ] Test all pages
  - [ ] Test responsive design
  - [ ] Test data loading

### Build & Deploy

- [ ] **Build for production**
  - [ ] Run `npm run build`
  - [ ] Verify no build errors
  - [ ] Test production build locally

- [ ] **Deploy**
  - [ ] Commit changes
  - [ ] Push to trigger GitHub Actions
  - [ ] Monitor deployment
  - [ ] Verify S3 upload
  - [ ] Verify CloudFront invalidation

### Verification

- [ ] **Test production**
  - [ ] Visit production URL
  - [ ] Test all pages
  - [ ] Test data loading
  - [ ] Test report download
  - [ ] Test on mobile

---

**Use these checklists to ensure thorough and coordinated changes across the AWS Services Dashboard ecosystem.**
