# Repository Directory

Complete guide to all repositories in the AWS Services Dashboard ecosystem.

## Repository Overview

| Repository | Primary Language | Role | Version/Commits | Status |
|------------|-----------------|------|-----------------|--------|
| aws-services-site | JavaScript/React | Frontend Dashboard | v2.2.0 (20+ commits) | ✅ Active |
| aws-infrastructure-fetcher | JavaScript/Node.js | Data Collection | 13 commits | ✅ Active |
| nodejs-aws-reporter | JavaScript/Node.js | Report Generation | 17 commits | ✅ Active |
| synepho-s3cf-site | HCL/Terraform | Infrastructure | Active (multi-env) | ✅ Active |

---

## 1. aws-services-site

**Frontend Dashboard - React Application**

### Repository Information
- **GitHub**: https://github.com/jxman/aws-services-site
- **Production URL**: https://aws-services.synepho.com
- **Current Version**: 2.2.0
- **Primary Language**: JavaScript (React)
- **Build Tool**: Vite 5.3
- **Package Manager**: npm
- **Lines of Code**: ~4,527 (JavaScript/JSX)

### Purpose
User-facing web application that displays AWS infrastructure data in an intuitive dashboard with visualizations, downloadable reports, and comprehensive service/region coverage analysis.

### Key Technologies
- **React** 18.3.1 - Core UI framework
- **Vite** 5.3.1 - Build tool and dev server
- **React Router** 7.9.4 - Client-side navigation
- **TanStack Query** 5.90.5 - Data fetching and caching
- **Recharts** 3.3.0 - Data visualization
- **Tailwind CSS** 3.4.18 - Utility-first styling
- **@headlessui/react** 2.2.9 - Accessible UI components
- **date-fns** 4.1.0 - Date formatting
- **dompurify** 3.3.0 - XSS sanitization

### Project Structure
```
aws-services-site/
├── src/
│   ├── components/         # 20+ reusable UI components
│   │   ├── common/        # Loading, Error, Modal, StatCard, ScrollToTop
│   │   ├── layout/        # Header, Footer, Container, Layout
│   │   ├── dashboard/     # WhatsNewPreview, RecentChangesPreview
│   │   ├── whats-new/     # 8 What's New feature components
│   │   └── ErrorBoundary.jsx
│   ├── views/             # 7 page components
│   │   ├── Dashboard.jsx  # Home with metrics
│   │   ├── Regions.jsx    # AWS regions browser
│   │   ├── Services.jsx   # Services catalog
│   │   ├── Coverage.jsx   # Service × Region matrix
│   │   ├── WhatsNew.jsx   # Infrastructure changes
│   │   ├── Reports.jsx    # Report center
│   │   └── About.jsx      # About page
│   ├── hooks/             # 3 custom hooks
│   │   ├── useAWSData.js
│   │   ├── useInfrastructureChanges.js
│   │   └── useAWSAnnouncements.js
│   ├── contexts/          # ThemeContext (light/dark mode)
│   ├── utils/             # Helpers (calculations, formatters, constants)
│   ├── config/            # aws-config.js (data URLs, schedule)
│   ├── styles/            # Global CSS and Tailwind
│   ├── App.jsx            # Root component with routing
│   └── main.jsx           # Application entry point
├── docs/                  # 10 documentation files
├── public/                # Static assets (favicon, images)
├── dist/                  # Production build (944 KB)
├── deploy.sh              # Automated deployment script
├── index.html             # HTML with SEO meta tags
├── package.json           # Dependencies
├── vite.config.js         # Vite with proxy configuration
├── tailwind.config.js     # Custom theme (dark mode support)
└── .eslintrc.cjs          # ESLint configuration
```

### Dependencies Integration
- **Consumes**: JSON data from S3 (via CloudFront)
  - `/data/complete-data.json` - Main data source
  - `/data/services.json` - Services catalog
- **Downloads**: Excel reports from S3
  - `/reports/aws-service-report-latest.xlsx`
- **Hosted on**: Infrastructure from synepho-s3cf-site
  - **Distribution**: EBTYLWOK3WVOK (CloudFront)
  - **Bucket**: www.aws-services.synepho.com (S3)

### Development Setup
```bash
git clone https://github.com/jxman/aws-services-site.git
cd aws-services-site
npm install
npm run dev  # Runs on http://localhost:3000
```

**Development Features**:
- Vite proxy: `/data` and `/reports` proxied to production
- Hot module replacement (HMR)
- Fast refresh for React components

### Deployment
Manual deployment via `deploy.sh` script:
- Builds production bundle (`npm run build`)
- Creates 404.html for SPA routing
- Backs up current index.html
- Uploads to S3 (excludes `/data/` and `/reports/`)
- Sets no-cache headers for HTML
- Invalidates CloudFront cache
- **TODO**: Migrate to GitHub Actions CI/CD

### Key Features

**7 Views**:
- ✅ **Dashboard**: Metrics, statistics, What's New preview (last 3 changes)
- ✅ **Regions**: Grid/table views, coverage analysis, search/filter, CSV export
- ✅ **Services**: Catalog browsing, regional availability, search/sort, CSV export
- ✅ **Coverage**: Heatmap matrix, advanced filtering, real-time search
- ✅ **What's New**: Infrastructure changes timeline, AWS announcements, statistics
- ✅ **Reports**: Excel/CSV downloads, metadata summaries
- ✅ **About**: Architecture info, tech stack, contact details

**UI/UX**:
- ✅ Light/Dark mode toggle (sun/moon icons)
- ✅ Mobile-responsive hamburger menu
- ✅ Theme persistence (localStorage)
- ✅ Error boundaries for graceful degradation
- ✅ Loading and empty states
- ✅ Keyboard accessible navigation

**Performance**:
- Build size: 944 KB (~125 KB gzipped)
- Code splitting: Vendor chunks (React, Router, Query)
- 0 ESLint errors/warnings
- Fast page loads (<2s)

---

## 2. aws-infrastructure-fetcher

**Data Collection Lambda - Node.js**

### Repository Information
- **GitHub**: https://github.com/jxman/aws-infrastructure-fetcher
- **Execution Environment**: AWS Lambda (Node.js 20.x)
- **Primary Language**: JavaScript (Node.js ≥20.0.0)
- **Infrastructure**: AWS SAM (Serverless Application Model)
- **Trigger**: EventBridge `cron(0 2 * * ? *)` - Daily at 2:00 AM UTC
- **Package Manager**: npm
- **Commits**: 13 total
- **Last Update**: October 13, 2025 (commit eb0b120)

### Purpose
Serverless function that automatically fetches and maintains comprehensive AWS infrastructure data from AWS SSM Parameter Store. Discovers 38 AWS regions and 394+ services, maintaining availability zone mappings with 24-hour intelligent caching.

### Key Technologies
- **Node.js** 20.x - Lambda runtime
- **AWS SDK** v3.645.0 - AWS service interactions
- **SAM** - Infrastructure deployment
- **Data Source**: AWS SSM Parameter Store (primary)
- **RSS Feed Parser**: AWS What's New announcements

### Project Structure
```
aws-infrastructure-fetcher/
├── src/               # Lambda function source code
│   ├── index.js      # Main Lambda handler
│   └── [modules]     # Data fetching logic
├── scripts/           # Deployment and utility scripts
├── docs/              # Documentation
│   └── DATA_CONTRACT.md  # Output data schema
├── template.yaml      # SAM CloudFormation template
├── package.json       # NPM dependencies
├── CHANGELOG.md       # Version history
└── README.md          # Comprehensive documentation
```

### Dependencies Integration
- **Reads from**:
  - AWS SSM Parameter Store (regions and services catalog)
  - AWS What's New RSS Feed (announcements)
- **Writes to**: S3 distribution bucket
  - `regions.json` - Region metadata with availability zones
  - `services.json` - Service catalog (394+ entries)
  - `complete-data.json` - Single source of truth
  - `cache/services-by-region.json` - Service availability mapping
  - `history/YYYY-MM-DD/` - Daily snapshots (30-day retention)
- **Triggers**: S3 ObjectCreated event → nodejs-aws-reporter Lambda
- **Deployed by**: AWS SAM (template.yaml)

### Execution Flow
1. **Triggered by EventBridge**: Daily at 2:00 AM UTC
2. **Check cache**: 24-hour intelligent caching reduces API calls
3. **Fetch from SSM**: Queries Parameter Store for regions and services
4. **Discover regions**: All commercial, China, and GovCloud regions (38+)
5. **Catalog services**: 394+ AWS services with regional availability
6. **Generate mappings**: Service-by-region availability matrix
7. **Upload to S3**: Multiple JSON files for different use cases
8. **Create snapshot**: Historical data in `history/` directory
9. **Log metrics**: CloudWatch logs for monitoring
10. **Trigger reporter**: S3 event notification invokes Excel generator

### Execution Performance
- **Cached execution**: 13-30 seconds (when cache is valid)
- **Fresh fetch**: 1 minute 49 seconds (full data collection)
- **Cache duration**: 24 hours
- **Data coverage**: 38+ regions, 394+ services, 15,000+ mappings

### Environment Variables
- **S3 Bucket Names**: Source and distribution bucket parameters
- **Distribution Bucket**: CloudFront-backed public distribution (optional)
- **SNS Topic**: Notification topic ARN (configurable)
- **EventBridge Schedule**: Customizable cron expression (default: daily 2 AM UTC)

### Development Setup
```bash
git clone https://github.com/jxman/aws-infrastructure-fetcher.git
cd aws-infrastructure-fetcher
npm install

# Build SAM application
sam build

# Deploy with guided setup
sam deploy --guided

# Test locally with SAM
sam local invoke -e events/test-event.json

# Or test with Node.js directly
node src/index.js
```

### Deployment
**Automated Deployment**:
1. Run `npm install` for dependencies
2. Execute `sam build` to prepare deployment package
3. Use `sam deploy --guided` for first-time setup
4. Subsequent deployments: `sam deploy`

**Manual Testing**:
```bash
# Invoke Lambda function via AWS CLI
aws lambda invoke \
  --function-name aws-infrastructure-fetcher \
  --payload '{}' \
  response.json
```

### Data Output Schema

See `docs/DATA_CONTRACT.md` in repository for complete schema.

**complete-data.json Structure**:
```json
{
  "metadata": {
    "lastUpdated": "2025-01-24T02:00:00Z",
    "totalRegions": 38,
    "totalServices": 394,
    "dataSource": "AWS Systems Manager Parameter Store",
    "schemaVersion": "2.0"
  },
  "regions": {
    "regions": [
      {
        "code": "us-east-1",
        "name": "US East (N. Virginia)",
        "availabilityZones": ["us-east-1a", "us-east-1b", ...]
      }
    ]
  },
  "services": {
    "services": [
      {
        "name": "Amazon EC2",
        "code": "ec2",
        "category": "Compute"
      }
    ]
  },
  "servicesByRegion": {
    "byRegion": {
      "us-east-1": {
        "services": ["ec2", "s3", "lambda", ...]
      }
    }
  }
}
```

---

## 3. nodejs-aws-reporter

**Excel Report Generator - Node.js Lambda**

### Repository Information
- **GitHub**: https://github.com/jxman/nodejs-aws-reporter
- **Execution Environment**: AWS Lambda (Node.js 20.x)
- **Primary Language**: JavaScript (Node.js)
- **Infrastructure**: AWS SAM (Serverless Application Model)
- **Trigger**: S3 ObjectCreated events on `complete-data.json` uploads
- **Schedule**: Automatically daily at 2 AM UTC (after data fetcher completes)
- **Package Manager**: npm
- **Commits**: 17 total
- **Created**: October 16, 2025
- **License**: ISC

### Purpose
Automated AWS Lambda function that generates professionally formatted Excel reports from AWS infrastructure data. Consumes data from the aws-infrastructure-fetcher project and creates comprehensive spreadsheets with region, service, and availability information.

### Key Technologies
- **Node.js** 20.x - Lambda runtime
- **AWS SDK** v3.670.0 - AWS service interactions
- **ExcelJS** - Excel spreadsheet generation and formatting
- **SAM** - Infrastructure deployment

### Project Structure
```
nodejs-aws-reporter/
├── src/               # Lambda function source code
│   ├── index.js      # Main Lambda handler
│   ├── generators/   # Report sheet generators
│   ├── formatters/   # Data formatting utilities
│   └── styles/       # Excel styling definitions
├── scripts/           # Utility scripts
├── docs/              # Documentation
│   ├── CLAUDE.md
│   ├── DESIGN.md
│   └── DEPLOYMENT.md
├── .github/workflows/ # CI/CD automation
├── template.yaml      # SAM CloudFormation template
├── package.json       # NPM dependencies
└── README.md          # Comprehensive documentation
```

### Dependencies Integration
- **Reads from**: S3 distribution bucket
  - `complete-data.json` - Source data file
- **Writes to**: S3 distribution bucket
  - `aws-service-report-latest.xlsx` - Latest report
  - Archive reports with 7-day retention
- **Triggered by**: S3 ObjectCreated event notification
  - Automatically runs when data fetcher uploads new data
- **Notifies via**: SNS topic (optional email notifications)
- **Deployed by**: AWS SAM (template.yaml)

### Execution Flow
1. **S3 Event Trigger**: ObjectCreated on `complete-data.json` upload
2. **Download data**: Fetch latest `complete-data.json` from S3
3. **Create workbook**: Initialize Excel workbook with ExcelJS
4. **Generate sheets**: Create 4 formatted worksheets
5. **Apply styling**: Color-coding, conditional formatting, frozen panes
6. **Upload report**: Save to S3 as `aws-service-report-latest.xlsx`
7. **Archive**: Create dated copy in archive directory (7-day retention)
8. **Send notification**: Optional SNS email with download link
9. **Log completion**: CloudWatch logs with execution metrics

### Report Structure (4 Sheets)

**1. Summary Sheet**:
- Metadata with EST/EDT timestamps
- Schema versions and data source attribution
- Statistics: Total regions (38), total services (395)
- Generation timestamp and report version
- Data freshness indicator

**2. Regions Sheet**:
- All 38 AWS regions with details
- Service counts per region
- Launch dates for each region
- Regional service availability statistics
- Sortable and filterable columns
- Auto-width columns for readability

**3. Services Sheet**:
- 395 AWS services catalog
- Service names, codes, and categories
- Regional coverage percentages with color-coding:
  - **Green**: 100% coverage (available in all regions)
  - **Orange**: 50-74% coverage (partial availability)
  - **Red**: <50% coverage (limited availability)
  - **Gray italic "N/A"**: Missing or unavailable data
- Sortable columns for analysis

**4. Service Coverage Matrix**:
- **15,010-cell grid** (38 regions × 395 services)
- Visual service availability matrix
- Service names in rows, region codes in columns
- Checkmarks (✓) or blanks for availability
- Frozen headers for easy navigation
- Auto-filter enabled for all columns
- Color-coded for quick scanning
- Professional formatting

### Report Features
- **File Size**: Approximately 65 KB per report
- **Automated Generation**: Triggered by data updates (daily at 2 AM UTC)
- **Professional Formatting**: Conditional styling, color-coding, borders
- **Usability**: Frozen panes, auto-filters, sortable columns
- **Historical Tracking**: Archived reports for trend analysis
- **Notifications**: SNS email alerts on completion (optional)
- **Error Handling**: Graceful degradation with "N/A" for missing data

### Environment Variables
- **S3 Bucket Name**: Source and destination bucket
- **SNS Topic ARN**: Notification topic (optional)
- **Report Retention**: Archive duration (default: 7 days)

### Development Setup
```bash
git clone https://github.com/jxman/nodejs-aws-reporter.git
cd nodejs-aws-reporter
npm install

# Build SAM application
sam build

# Test locally with SAM
sam local invoke -e events/report-request.json

# Deploy with guided setup
sam deploy --guided
```

### Deployment
**Automated Deployment**:
1. Run `npm install --production` for dependencies
2. Execute `sam build` to prepare deployment package
3. Use `sam deploy --guided` for first-time setup
4. Subsequent deployments: `sam deploy`

**S3 Event Configuration**:
```bash
# Configure S3 event notification to trigger Lambda
aws s3api put-bucket-notification-configuration \
  --bucket your-bucket-name \
  --notification-configuration file://notification-config.json
```

**Manual Testing**:
```bash
# Invoke Lambda function via AWS CLI
aws lambda invoke \
  --function-name aws-service-report-generator \
  --payload '{}' \
  response.json

# Check output
cat response.json
```

### GitHub Actions Integration
- **CI/CD**: Automated deployment via GitHub Actions
- **Authentication**: OIDC (no long-lived credentials)
- **Workflow**: `.github/workflows/terraform.yml`
- **Triggers**: Push to main branch, manual workflow dispatch

---

## 4. synepho-s3cf-site

**Infrastructure as Code - Terraform**

### Repository Information
- **GitHub**: https://github.com/jxman/synepho-s3cf-site
- **Primary Language**: HCL (Terraform)
- **Terraform Version**: ≥1.7.0
- **AWS Provider**: Latest version
- **Status**: Active (multi-environment)

### Purpose
Infrastructure as Code project for Synepho.com deploying resilient static website hosting on AWS using Terraform. Manages multiple production environments with multi-region failover, CDN integration, and automated CI/CD deployment.

### Key Technologies
- **Terraform** ≥1.7.0 - Infrastructure as Code
- **AWS Provider** - Latest version
- **GitHub Actions** - CI/CD automation
- **OIDC** - OpenID Connect authentication (credential-less deployments)
- **Primary Services**: S3, CloudFront, Route53, ACM, CloudWatch, IAM

### Deployment Environments

The project supports **four distinct environments**:
- **Production**: synepho.com (us-east-1 primary)
- **AWS Services**: aws-services.synepho.com (React dashboard)
- **Development**: dev.synepho.com
- **Staging**: Dedicated staging environment

### Project Structure
```
synepho-s3cf-site/
├── modules/           # 5 reusable Terraform modules
│   ├── s3-cloudfront/ # Static hosting module
│   ├── lambda/        # Lambda function module
│   ├── github-oidc/   # OIDC provider module
│   ├── monitoring/    # CloudWatch monitoring module
│   └── [other modules]
├── environments/      # Environment-specific configurations
│   ├── prod/          # Production (synepho.com)
│   ├── aws-services/  # AWS Services Dashboard
│   ├── dev/           # Development
│   └── staging/       # Staging environment
├── scripts/           # Automation scripts
│   ├── bootstrap-oidc.sh        # One-time OIDC setup
│   └── create-prerequisites.sh  # Bootstrap script
├── .github/workflows/
│   └── terraform.yml  # CI/CD workflow
├── main.tf            # Root module
├── variables.tf       # Input variables
├── outputs.tf         # Output values
├── versions.tf        # Provider versions
└── terraform.tfvars   # Variable values (gitignored)
```

### Managed Resources

#### Hosting Infrastructure
- **S3 Buckets**:
  - Static website buckets (one per environment)
  - Versioning enabled for rollback capability
  - Encryption at rest (SSE-S3)
  - Access logging enabled
  - Private buckets (no public access)
- **CloudFront Distributions**:
  - Global CDN with Origin Access Control (OAC)
  - SSL/TLS certificates via ACM with automatic DNS validation
  - Custom error responses for SPA routing (404 → index.html)
  - Security headers via response policies
  - SEO optimizations: X-Robots-Tag headers
  - Dedicated cache behaviors for robots.txt and sitemap.xml
  - 5-minute cache TTL for HTML, longer for static assets
- **Route53**:
  - DNS management for all domains (synepho.com, aws-services.synepho.com)
  - Failover routing for high availability
  - Health checks for primary/secondary origins

#### High Availability Features
- **Cross-region replication**: us-east-1 (primary) to us-west-2 (failover)
- **CloudFront origin failover**: Automatic failover configuration
- **Multi-region DNS**: Route53 failover routing policies
- **Health checks**: Active monitoring of origin availability

#### Lambda Infrastructure (via SAM integration)
- IAM execution roles with least privilege
- Lambda function deployments (Data Fetcher, Reporter)
- EventBridge schedules (cron expressions)
- CloudWatch log groups with retention policies
- S3 event notifications for automated triggering

#### Monitoring Infrastructure
- **CloudWatch Dashboards**: Regional traffic patterns and metrics
- **CloudWatch Alarms**: Error rates, cache hit ratios, latency thresholds
- **Access Logs**: S3 and CloudFront access logging
- **Metrics**: Request counts, error rates, performance data

#### CI/CD Infrastructure (GitHub OIDC)
- **GitHub OIDC Provider**: OpenID Connect for credential-less authentication
- **Project-Specific IAM Roles**: One role per repository with repository restrictions
- **Least Privilege Policies**: Scoped permissions per workflow
- **Automated Deployments**: GitHub Actions on main branch pushes
- **PR Validation**: `terraform plan` on pull requests (no apply)
- **Manual Workflows**: Workflow dispatch for any environment

**OIDC Setup**:
- One-time bootstrap via `scripts/bootstrap-oidc.sh`
- Project-specific IAM roles with least-privilege access
- Eliminates long-lived credential storage in GitHub secrets
- Complete audit trail in CloudTrail

#### Security Features
- **TLS Enforcement**: HTTPS only, TLS 1.2+ minimum
- **Security Headers**: Via CloudFront response policies
- **Access Logging**: S3 and CloudFront access logs
- **Private S3 Buckets**: CloudFront OAC only (no public access)
- **Encryption**: SSE-S3 for all S3 buckets

### Dependencies Integration
- **Deploys**: All infrastructure for other repositories
- **Manages**: DNS, SSL, hosting, Lambda functions, monitoring
- **Provides**: Resource outputs for other components
  - CloudFront distribution IDs
  - S3 bucket names
  - IAM role ARNs
  - Dashboard URLs

### State Management
- **Backend**: S3 bucket (synepho-terraform-state)
- **State Locking**: DynamoDB table prevents concurrent modifications
- **Encryption**: Versioning enabled for state rollback
- **Isolation**: Separate state files per environment
  - `prod/` - Production state
  - `aws-services/` - AWS Services Dashboard state
  - `dev/` - Development state
  - `staging/` - Staging state
- **Workspaces**: Environment-specific configurations

### Development Workflow
```bash
git clone https://github.com/jxman/synepho-s3cf-site.git
cd synepho-s3cf-site

# Initialize Terraform
terraform init

# Validate configuration
terraform validate

# Format code
terraform fmt -recursive

# Plan changes (read-only, safe for local testing)
terraform plan

# IMPORTANT: Never apply locally in production
# Use GitHub Actions for all deployments
```

### Deployment Policy

**CRITICAL**: All production deployments MUST use GitHub Actions workflows.

**GitHub Actions Deployment** (Required):
```bash
# Trigger workflow manually
gh workflow run "Terraform Deployment" --ref main

# Monitor deployment progress
gh run list --limit 5
gh run view [RUN_ID]

# Watch in real-time
gh run watch [RUN_ID]

# Open in browser
gh run view [RUN_ID] --web
```

**Automated Deployment**:
- Push to `main` branch triggers deployment workflow
- Pull requests trigger `terraform plan` (validation only)
- Manual workflow dispatch available for any environment

### Notable Features

**SEO Optimizations**:
- X-Robots-Tag headers for search engine control
- Dedicated cache behaviors for robots.txt and sitemap.xml
- SPA routing support (404 → index.html)
- Custom error pages

**Performance**:
- CloudFront edge caching globally
- Multi-region failover (us-east-1 to us-west-2)
- Origin shield for reduced origin load (optional)

**Security**:
- Private S3 buckets with CloudFront OAC
- TLS enforcement (HTTPS only)
- Security headers via response policies
- Access logging for audit trails

**Cost Optimization**:
- S3 Intelligent-Tiering (optional)
- CloudFront request consolidation
- Efficient caching strategies

### Environment-Specific Configurations

Each environment has its own:
- Domain name and DNS records
- CloudFront distribution
- S3 bucket
- Terraform state file
- IAM roles and policies
- Monitoring dashboards and alarms

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
