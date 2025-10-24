# GitHub Repository Topics Template

Add these topics to each repository for discoverability and organization.

## How to Add Topics

1. Go to the repository on GitHub
2. Click the gear icon next to "About" section
3. Add topics in the "Topics" field
4. Click "Save changes"

Or via GitHub CLI:
```bash
gh repo edit --add-topic "topic-name"
```

---

## Common Topics (Add to ALL Repositories)

```
aws-services-dashboard
synepho-project
multi-repo-project
aws
serverless
monitoring
```

---

## Repository-Specific Topics

### aws-services-site (Frontend)
```
aws-services-dashboard
synepho-project
multi-repo-project
aws
react
vite
tailwindcss
dashboard
data-visualization
recharts
frontend
spa
cloudfront
s3-website
```

**Repository Description**:
```
React dashboard for AWS Services Dashboard - visualizes AWS infrastructure data across multiple accounts and regions
```

---

### aws-infrastructure-fetcher (Data Fetcher)
```
aws-services-dashboard
synepho-project
multi-repo-project
aws
lambda
nodejs
serverless
data-collection
aws-sdk
eventbridge
cloudwatch
infrastructure-monitoring
```

**Repository Description**:
```
Lambda function for AWS Services Dashboard - collects infrastructure data from AWS services and publishes to S3
```

---

### nodejs-aws-reporter (Reporter)
```
aws-services-dashboard
synepho-project
multi-repo-project
aws
lambda
nodejs
serverless
excel
exceljs
report-generation
data-export
```

**Repository Description**:
```
Lambda function for AWS Services Dashboard - generates Excel reports from collected AWS infrastructure data
```

---

### synepho-s3cf-site (Infrastructure)
```
aws-services-dashboard
synepho-project
multi-repo-project
aws
terraform
infrastructure-as-code
iac
cloudfront
s3
route53
acm
github-actions
oidc
cicd
```

**Repository Description**:
```
Terraform infrastructure for AWS Services Dashboard - manages hosting, Lambda functions, DNS, SSL, and CI/CD
```

---

### aws-services-dashboard-project (Documentation Hub)
```
aws-services-dashboard
synepho-project
multi-repo-project
documentation
architecture
monorepo-coordination
project-hub
development-guide
```

**Repository Description**:
```
Central documentation hub for AWS Services Dashboard - architecture, development workflows, deployment procedures
```

---

## GitHub CLI Commands

### Add All Common Topics to a Repository

```bash
# Navigate to repository directory first
cd /path/to/repository

# Add common topics
gh repo edit --add-topic "aws-services-dashboard"
gh repo edit --add-topic "synepho-project"
gh repo edit --add-topic "multi-repo-project"
gh repo edit --add-topic "aws"
gh repo edit --add-topic "serverless"
gh repo edit --add-topic "monitoring"
```

### Add Repository-Specific Topics

```bash
# For aws-services-site
gh repo edit --add-topic "react"
gh repo edit --add-topic "vite"
gh repo edit --add-topic "tailwindcss"
gh repo edit --add-topic "dashboard"
gh repo edit --add-topic "data-visualization"
gh repo edit --add-topic "recharts"
gh repo edit --add-topic "frontend"
gh repo edit --add-topic "spa"
gh repo edit --add-topic "cloudfront"
gh repo edit --add-topic "s3-website"

# For aws-infrastructure-fetcher
gh repo edit --add-topic "lambda"
gh repo edit --add-topic "nodejs"
gh repo edit --add-topic "data-collection"
gh repo edit --add-topic "aws-sdk"
gh repo edit --add-topic "eventbridge"
gh repo edit --add-topic "cloudwatch"
gh repo edit --add-topic "infrastructure-monitoring"

# For nodejs-aws-reporter
gh repo edit --add-topic "lambda"
gh repo edit --add-topic "nodejs"
gh repo edit --add-topic "excel"
gh repo edit --add-topic "exceljs"
gh repo edit --add-topic "report-generation"
gh repo edit --add-topic "data-export"

# For synepho-s3cf-site
gh repo edit --add-topic "terraform"
gh repo edit --add-topic "infrastructure-as-code"
gh repo edit --add-topic "iac"
gh repo edit --add-topic "cloudfront"
gh repo edit --add-topic "s3"
gh repo edit --add-topic "route53"
gh repo edit --add-topic "acm"
gh repo edit --add-topic "github-actions"
gh repo edit --add-topic "oidc"
gh repo edit --add-topic "cicd"

# For aws-services-dashboard-project
gh repo edit --add-topic "documentation"
gh repo edit --add-topic "architecture"
gh repo edit --add-topic "monorepo-coordination"
gh repo edit --add-topic "project-hub"
gh repo edit --add-topic "development-guide"
```

### Set Repository Description

```bash
# For aws-services-site
gh repo edit --description "React dashboard for AWS Services Dashboard - visualizes AWS infrastructure data across multiple accounts and regions"

# For aws-infrastructure-fetcher
gh repo edit --description "Lambda function for AWS Services Dashboard - collects infrastructure data from AWS services and publishes to S3"

# For nodejs-aws-reporter
gh repo edit --description "Lambda function for AWS Services Dashboard - generates Excel reports from collected AWS infrastructure data"

# For synepho-s3cf-site
gh repo edit --description "Terraform infrastructure for AWS Services Dashboard - manages hosting, Lambda functions, DNS, SSL, and CI/CD"

# For aws-services-dashboard-project
gh repo edit --description "Central documentation hub for AWS Services Dashboard - architecture, development workflows, deployment procedures"
```

---

## Benefits of Consistent Topics

- **Discoverability**: Easy to find all related repositories
- **Organization**: Group repositories by technology or purpose
- **Search**: Filter repositories by topic
- **Context**: Immediately understand repository purpose
- **Team awareness**: New team members can find all components

---

## Verification

After adding topics, verify by:

1. **GitHub Search**: Search for `topic:aws-services-dashboard` on GitHub
2. **Repository View**: Check that topics appear on repository page
3. **Consistency**: All related repos should show similar topics

---

**End of template**
