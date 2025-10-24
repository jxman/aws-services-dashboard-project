# CLAUDE.md Addition Template

Add this section to each component repository's CLAUDE.md file to establish awareness of the full project ecosystem.

---

## Copy and paste the section below into your component repository's CLAUDE.md:

---

# Related Repositories

This repository is part of the **AWS Services Dashboard** project ecosystem.

## Project Hub
**Central Documentation**: https://github.com/jxman/aws-services-dashboard-project

For complete project documentation, architecture diagrams, development workflows, and deployment procedures, refer to the hub repository above.

## Related Components

- **Frontend**: [aws-services-site](https://github.com/jxman/aws-services-site) - React dashboard for visualizing AWS infrastructure data
- **Data Fetcher**: [aws-infrastructure-fetcher](https://github.com/jxman/aws-infrastructure-fetcher) - Lambda function that collects AWS data
- **Reporter**: [nodejs-aws-reporter](https://github.com/jxman/nodejs-aws-reporter) - Lambda function that generates Excel reports
- **Infrastructure**: [synepho-s3cf-site](https://github.com/jxman/synepho-s3cf-site) - Terraform infrastructure for hosting and deployment
- **Documentation Hub**: [aws-services-dashboard-project](https://github.com/jxman/aws-services-dashboard-project) - Central documentation and coordination

## This Repository's Role

**[CUSTOMIZE THIS SECTION FOR EACH REPOSITORY]**

### For aws-services-site (Frontend):
This is the **frontend dashboard** that displays AWS infrastructure data to users through an interactive web interface.

**Integration Points**:
- **Consumes**: JSON data from S3 bucket (published by Data Fetcher)
- **Triggers**: Reporter Lambda for Excel report generation
- **Hosted on**: Infrastructure deployed by synepho-s3cf-site

**Data Flow**:
1. React app fetches JSON data from S3 via CloudFront
2. Displays visualizations using Recharts
3. User clicks "Download Report" → triggers Reporter Lambda
4. Reporter returns Excel file for download

---

### For aws-infrastructure-fetcher (Data Fetcher):
This is the **data collection Lambda** that gathers AWS infrastructure data and publishes it for the frontend.

**Integration Points**:
- **Reads from**: AWS services (EC2, RDS, S3, Lambda, CloudWatch, etc.)
- **Writes to**: S3 data bucket (JSON format)
- **Triggers**: CloudFront invalidation (optional)
- **Deployed by**: Terraform in synepho-s3cf-site

**Data Flow**:
1. EventBridge triggers Lambda on schedule
2. Lambda queries AWS services across all regions
3. Aggregates and formats data as JSON
4. Uploads to S3 bucket
5. Frontend consumes this data

---

### For nodejs-aws-reporter (Reporter):
This is the **Excel report generator** that creates downloadable reports from collected AWS data.

**Integration Points**:
- **Reads from**: S3 data bucket (JSON files created by Data Fetcher)
- **Triggered by**: Frontend dashboard (user action)
- **Returns to**: User via download
- **Deployed by**: Terraform in synepho-s3cf-site

**Data Flow**:
1. User clicks "Download Report" in frontend
2. Frontend triggers Reporter Lambda
3. Lambda reads latest data from S3
4. Generates Excel workbook with multiple sheets
5. Returns Excel file to user

---

### For synepho-s3cf-site (Infrastructure):
This is the **infrastructure as code** repository that deploys and manages all AWS resources for the project.

**Integration Points**:
- **Deploys**: All infrastructure for the entire ecosystem
- **Manages**: CloudFront, S3, Lambda, Route53, ACM, IAM, OIDC
- **Provides**: Resource configuration and outputs
- **CI/CD**: GitHub Actions with OIDC authentication

**Managed Resources**:
- Static site hosting (S3 + CloudFront)
- Lambda functions (Data Fetcher + Reporter)
- DNS and SSL certificates
- IAM roles and policies
- GitHub Actions OIDC provider
- Monitoring and logging

---

## Key Architecture Points

### Data Flow Overview
```
AWS Services ─▶ Data Fetcher Lambda ─▶ S3 Bucket ─▶ CloudFront ─▶ React Frontend
                                                                        │
                                                                        ▼
                                         Reporter Lambda ◀──────── User Request
```

### Deployment Flow
```
Developer ─▶ Git Push ─▶ GitHub Actions ─▶ OIDC Auth ─▶ AWS Deployment
```

## Cross-Repository Workflows

### Adding a New Feature

**If the feature requires changes across multiple repositories:**

1. **Update documentation** in the hub repository first
2. **Implement infrastructure** changes in synepho-s3cf-site
3. **Deploy infrastructure** via GitHub Actions
4. **Implement backend** changes in fetcher/reporter
5. **Deploy Lambda** functions via GitHub Actions
6. **Implement frontend** changes in aws-services-site
7. **Deploy frontend** via GitHub Actions
8. **Update documentation** in hub repository with final details

### Testing Integration Points

Before deploying changes that affect integration:

1. **Test locally** where possible
2. **Verify data contracts** (JSON structure, API responses)
3. **Check environment variables** and configuration
4. **Deploy to dev environment** first (if available)
5. **Monitor CloudWatch logs** during deployment
6. **Verify end-to-end flow** in production

## Important Reminders

### Deployment Policy
**CRITICAL**: All production deployments MUST use GitHub Actions workflows with OIDC authentication. Local deployments are deprecated.

```bash
# Deploy via GitHub Actions (REQUIRED)
gh workflow run "Deploy to Production" --ref main
gh run watch
```

### Data Format Compatibility

When changing data structures:

1. **Frontend and Data Fetcher must align** on JSON schema
2. **Reporter must align** with data structure for Excel generation
3. **Version data format** if making breaking changes
4. **Test thoroughly** before deploying

### Infrastructure Dependencies

When modifying infrastructure:

1. **Plan before applying** - use `terraform plan`
2. **Deploy via GitHub Actions** - never apply locally in production
3. **Monitor deployment** - watch GitHub Actions logs
4. **Verify resources** - check AWS console after deployment

## Documentation References

For detailed information, refer to the hub repository:

- **Architecture**: [ARCHITECTURE.md](https://github.com/jxman/aws-services-dashboard-project/blob/main/ARCHITECTURE.md)
- **Development Guide**: [DEVELOPMENT.md](https://github.com/jxman/aws-services-dashboard-project/blob/main/DEVELOPMENT.md)
- **Deployment Guide**: [DEPLOYMENT.md](https://github.com/jxman/aws-services-dashboard-project/blob/main/DEPLOYMENT.md)
- **Repository Directory**: [REPOSITORIES.md](https://github.com/jxman/aws-services-dashboard-project/blob/main/REPOSITORIES.md)

---

**End of template - customize the "This Repository's Role" section for each repository**
