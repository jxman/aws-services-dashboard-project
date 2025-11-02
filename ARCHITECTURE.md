# AWS Services Dashboard - Architecture

## System Overview

The AWS Services Dashboard is a serverless, event-driven system for monitoring and reporting on AWS infrastructure. The system automatically discovers and catalogs 38+ AWS regions and 394+ AWS services, providing real-time visibility into service availability across all AWS regions through an interactive web dashboard.

## High-Level Architecture

```
┌────────────────────────────────────────────────────────────────────────┐
│                         User Access Layer                              │
└────────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌────────────────────────────────────────────────────────────────────────┐
│  CloudFront Distribution                                               │
│  - SSL/TLS termination (ACM)                                          │
│  - Edge caching                                                        │
│  - Custom error responses                                              │
└───────────────────────┬────────────────────────────────────────────────┘
                        │
        ┌───────────────┴───────────────┐
        │                               │
        ▼                               ▼
┌─────────────────┐           ┌─────────────────┐
│   S3 Bucket     │           │   S3 Bucket     │
│  (Static Site)  │           │  (JSON Data)    │
│                 │           │                 │
│  - React App    │           │  - Account data │
│  - HTML/CSS/JS  │           │  - Region data  │
│  - Assets       │           │  - Service data │
└─────────────────┘           └────────┬────────┘
                                       ▲
                                       │
                        ┌──────────────┴──────────────┐
                        │                             │
                        ▼                             │
┌─────────────────────────────────────┐              │
│  AWS Infrastructure Fetcher Lambda  │              │
│                                     │              │
│  - Scheduled execution (EventBridge)│              │
│  - Multi-account data collection    │              │
│  - JSON formatting & upload         │──────────────┘
│  - CloudWatch logging               │
└────────────┬────────────────────────┘
             │
             ▼
┌────────────────────────────────────────────────────────┐
│  AWS Services (Data Sources)                           │
│                                                        │
│  - EC2 (instances, volumes, snapshots)                │
│  - RDS (databases)                                     │
│  - S3 (buckets)                                        │
│  - Lambda (functions)                                  │
│  - CloudWatch (metrics)                                │
│  - Cost Explorer (billing data)                        │
└────────────────────────────────────────────────────────┘


┌─────────────────────────────────────┐
│  Reporter Lambda (Excel Generation) │
│                                     │
│  - Triggered by user request        │
│  - Reads from S3 data bucket        │
│  - Generates Excel workbook         │
│  - Returns download to user         │
└─────────────────────────────────────┘
```

## Component Details

### 1. Frontend (aws-services-site)

**Technology**: React 18.3 + Vite 5.3 + Tailwind CSS 3.4

**Current Version**: 2.2.0

**Key Features**:
- Single-page application (SPA) with 7 views
- Client-side routing (React Router 7.9)
- Data visualization (Recharts 3.3)
- Responsive design with mobile navigation
- Real-time data fetching (TanStack Query 5.90)
- Light/Dark mode toggle with localStorage persistence
- Error boundaries for graceful error handling
- Excel and CSV export capabilities

**Views Implemented**:
- **Dashboard**: Key metrics, statistics, What's New preview
- **Regions**: Browse 38+ AWS regions with coverage analysis
- **Services**: Catalog of 394+ AWS services with regional availability
- **Coverage**: Service × Region availability matrix heatmap
- **What's New**: Infrastructure changes and AWS announcements timeline
- **Reports**: Excel/CSV report generation center
- **About**: Architecture, technology stack, contact information

**Data Flow**:
1. User navigates to https://aws-services.synepho.com
2. CloudFront serves cached static assets (5-minute TTL)
3. React app loads and initializes
4. TanStack Query fetches JSON data from S3 (via CloudFront)
5. Components render visualizations with theme support
6. User interactions trigger CSV/Excel downloads

**Project Structure**:
- `src/views/`: 7 page components
- `src/components/`: 20+ reusable UI components organized by function
- `src/hooks/`: 3 custom hooks (useAWSData, useInfrastructureChanges, useAWSAnnouncements)
- `src/contexts/`: ThemeContext for light/dark mode
- `src/utils/`: Calculation, formatting, and helper functions
- `src/config/`: Configuration including data URLs

**Performance**:
- Build size: 944 KB total (~125 KB gzipped)
- Code splitting: Vendor chunks for React, Router, and Query
- Bundle optimization: Tree-shaking and minification
- 0 ESLint errors/warnings

### 2. Data Fetcher (aws-infrastructure-fetcher)

**Technology**: Node.js 20.x running on AWS Lambda

**Infrastructure**: AWS SAM (Serverless Application Model)

**Execution Model**:
- Scheduled via EventBridge: `cron(0 2 * * ? *)` - daily at 2:00 AM UTC
- Typical execution: 13-30 seconds (cached) to 1 minute 49 seconds (fresh fetch)
- Memory: Configurable via SAM template
- Runtime: Node.js 20 Lambda environment

**Data Collection Process**:
1. Lambda triggered by EventBridge daily schedule
2. Connects to AWS SSM Parameter Store (primary data source)
3. Discovers all commercial regions, China regions, and GovCloud
4. Fetches 38+ AWS regions and 394+ services
5. Applies 24-hour intelligent caching to minimize API calls
6. Aggregates and formats data as JSON
7. Uploads multiple JSON files to S3 distribution bucket
8. Maintains 30-day historical snapshots

**Data Source**:
- **Primary**: AWS SSM Parameter Store (regions and services catalog)
- **Secondary**: AWS What's New RSS Feed (announcements)
- **Scope**: All commercial, China, and GovCloud regions

**Output Files Generated**:
- `regions.json` - Region metadata with availability zones
- `services.json` - Service catalog (394+ entries)
- `complete-data.json` - Single source of truth combining all data
- `cache/services-by-region.json` - Service availability mapping
- `history/YYYY-MM-DD/` - Daily snapshots (30-day retention)

**Output Format** (complete-data.json):
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
    "regions": [/* region objects with AZ data */]
  },
  "services": {
    "services": [/* service objects */]
  },
  "servicesByRegion": {
    "byRegion": {
      "region-code": {
        "services": ["service1", "service2"]
      }
    }
  }
}
```

**Performance Optimization**:
- 24-hour intelligent caching reduces redundant API calls
- Parallel region queries for faster execution
- Minimal cold start time with Node.js 20 runtime

### 3. Reporter (nodejs-aws-reporter)

**Technology**: Node.js 20.x running on AWS Lambda

**Infrastructure**: AWS SAM (Serverless Application Model)

**Dependencies**:
- AWS SDK v3.670.0
- ExcelJS for spreadsheet generation

**Execution Model**:
- **Automated Trigger**: S3 ObjectCreated events on `complete-data.json` uploads
- **Schedule**: Automatically runs daily at 2 AM UTC (after data fetcher completes)
- **Manual Trigger**: AWS CLI invocation or Lambda console
- **Output**: Reports stored in S3 with 7-day archive retention
- **File Size**: Approximately 65 KB per report

**Report Generation Process**:
1. S3 event notification triggers Lambda when data fetcher uploads new data
2. Lambda reads latest `complete-data.json` from S3
3. Creates Excel workbook with 4 formatted sheets
4. Applies color-coding, formatting, and data validation
5. Uploads report to S3 as `aws-service-report-latest.xlsx`
6. Sends SNS notification (optional) with download link
7. Archives reports with 7-day retention

**Report Structure** (4 Sheets):

**1. Summary Sheet**:
- Metadata with EST/EDT timestamps
- Schema versions and data source attribution
- Statistics: Total regions (38), total services (395)
- Generation timestamp and report version

**2. Regions Sheet**:
- All 38 AWS regions with service counts
- Launch dates for each region
- Regional service availability statistics
- Sortable and filterable columns

**3. Services Sheet**:
- 395 AWS services catalog
- Regional coverage percentages with color-coding:
  - **Green**: 100% coverage
  - **Orange**: 50-74% coverage
  - **Red**: Limited availability
  - **Gray italic "N/A"**: Missing data
- Service categories and descriptions

**4. Service Coverage Matrix**:
- 15,010-cell grid (38 regions × 395 services)
- Visual service availability by region
- Frozen headers for easy navigation
- Auto-filter and sortable columns
- Color-coded availability indicators

**Features**:
- Automatic daily generation (triggered by data updates)
- Professional Excel formatting with conditional styling
- Frozen panes and auto-filters for usability
- Historical snapshots for trend analysis
- SNS email notifications on completion

### 4. Infrastructure (synepho-s3cf-site)

**Technology**: Terraform ≥1.7.0 (Infrastructure as Code)

**Provider**: AWS (latest version)

**Deployment Environments**: 4 distinct environments
- **Production**: synepho.com (us-east-1 primary)
- **AWS Services**: aws-services.synepho.com (React dashboard)
- **Development**: dev.synepho.com
- **Staging**: Dedicated staging environment

**Project Structure**:
- `modules/` - 5 reusable Terraform modules
- `environments/` - Separate configs for prod, dev, staging, aws-services
- `scripts/` - bootstrap-oidc.sh, create-prerequisites.sh
- `.github/workflows/` - terraform.yml for CI/CD automation

**Managed Resources**:

#### Hosting Infrastructure:
- **S3 Buckets**:
  - Static website buckets (one per environment)
  - Versioning enabled for rollback capability
  - Encryption at rest (SSE-S3)
  - Access logging enabled
- **CloudFront Distributions**:
  - Global CDN with Origin Access Control (OAC)
  - SSL/TLS certificates via ACM with automatic DNS validation
  - Custom error responses for SPA routing (404 → index.html)
  - Security headers via response policies
  - SEO optimizations: X-Robots-Tag headers
  - Dedicated cache behaviors for robots.txt and sitemap.xml
  - 5-minute cache TTL for HTML, longer for static assets
- **Route53**:
  - DNS management for all domains
  - Failover routing for high availability
  - Health checks for primary/secondary origins

#### High Availability:
- **Cross-region replication**: us-east-1 (primary) to us-west-2 (failover)
- **CloudFront origin failover**: Automatic failover configuration
- **Multi-region DNS**: Route53 failover routing policies

#### Lambda Infrastructure (via SAM integration):
- IAM execution roles with least privilege
- Lambda function deployments (Data Fetcher, Reporter)
- EventBridge schedules (cron expressions)
- CloudWatch log groups with retention policies
- S3 event notifications for automated triggering

#### Monitoring Infrastructure:
- **CloudWatch Dashboards**: Regional traffic patterns and metrics
- **CloudWatch Alarms**: Error rates, cache hit ratios, latency thresholds
- **Access Logs**: S3 and CloudFront access logging
- **Metrics**: Request counts, error rates, performance data

#### CI/CD Infrastructure:
- **GitHub OIDC Provider**: OpenID Connect for credential-less authentication
- **Project-Specific IAM Roles**: One role per repository with repository restrictions
- **Least Privilege Policies**: Scoped permissions per workflow
- **Automated Deployments**: GitHub Actions on main branch pushes
- **PR Validation**: `terraform plan` on pull requests (no apply)

**Terraform State Management**:
- **Backend**: S3 bucket (synepho-terraform-state)
- **State Locking**: DynamoDB table prevents concurrent modifications
- **Encryption**: Versioning enabled for state rollback
- **Isolation**: Separate state files per environment (prod/, dev/, aws-services/, staging/)
- **Workspaces**: Environment-specific configurations

## Security Architecture

### Authentication & Authorization

**Frontend Access**:
- Public access via HTTPS only
- No authentication required (monitoring dashboard)
- Data is read-only from user perspective

**Lambda Execution**:
- Dedicated IAM roles per function
- Least privilege permissions
- Cross-account access via AssumeRole

**GitHub Actions**:
- OIDC authentication (no long-lived credentials)
- Repository-specific IAM roles
- Scoped permissions per workflow

### Data Protection

**In Transit**:
- HTTPS/TLS 1.2+ enforced (CloudFront)
- AWS API calls over HTTPS
- S3 transfer encryption

**At Rest**:
- S3 bucket encryption (SSE-S3)
- CloudWatch logs encryption
- Terraform state encryption

**Access Control**:
- S3 buckets: Private with CloudFront OAC only
- Lambda functions: VPC isolation (optional)
- Least privilege IAM policies

## Monitoring & Observability

### CloudWatch Integration

**Metrics**:
- Lambda invocation counts and durations
- Lambda error rates and throttles
- CloudFront request counts and error rates
- S3 bucket sizes and request metrics

**Logs**:
- Lambda execution logs (structured logging)
- CloudFront access logs
- S3 access logs

**Alarms**:
- Lambda errors > 5%
- CloudFront error rate > 1%
- Data collection failures

### Dashboards

**CloudWatch Dashboard**:
- Real-time Lambda performance
- CloudFront traffic patterns
- Error rate monitoring
- Regional distribution

**Application Dashboard** (React):
- User-facing metrics
- Data freshness indicators
- Service health status

## Scalability Considerations

### Current Limits

**Lambda**:
- Concurrent executions: 1000 (default)
- Function timeout: 15 minutes
- Memory: 512MB-3GB (configurable)

**S3**:
- Virtually unlimited storage
- 5,500 GET requests/second per prefix
- 3,500 PUT requests/second per prefix

**CloudFront**:
- Unlimited requests
- Edge location caching
- Origin request reduction

### Scaling Strategies

**Data Collection**:
- Parallel region queries
- Batched API calls
- Incremental updates (delta collection)

**Frontend**:
- Edge caching (CloudFront)
- Lazy loading components
- Virtualized lists for large datasets

**Cost Optimization**:
- Lambda provisioned concurrency (if needed)
- S3 Intelligent-Tiering
- CloudFront cache optimization

## Disaster Recovery

### Backup Strategy

**Infrastructure**:
- Terraform state backed up in S3
- Version-controlled configuration (Git)
- Multi-region state replication (optional)

**Data**:
- S3 versioning enabled
- Cross-region replication (optional)
- Point-in-time recovery capability

### Recovery Procedures

**Infrastructure Loss**:
1. Retrieve Terraform state from S3
2. Re-apply Terraform configuration
3. Restore DNS records if needed

**Data Loss**:
1. Restore from S3 versioning
2. Re-run data collection Lambda
3. Verify data integrity

**Application Loss**:
1. Redeploy from Git repository
2. Rebuild React application
3. Upload to S3 and invalidate CloudFront

## Performance Optimization

### Frontend Optimization

- Code splitting (React.lazy)
- Asset minification and compression
- Image optimization
- Service worker caching (future)

### Backend Optimization

- Lambda cold start reduction
- Connection pooling for AWS SDK
- Parallel API calls
- Result caching in memory

### CDN Optimization

- CloudFront edge caching
- Gzip/Brotli compression
- Cache-Control headers
- Origin shield (if needed)

## Current Status and Metrics

### System Statistics (as of January 2025)

- **Regions Monitored**: 38+ (commercial, China, GovCloud)
- **Services Cataloged**: 394+
- **Service-Region Mappings**: 15,000+
- **Data Freshness**: Updated daily at 2:00 AM UTC
- **Historical Data**: 30-day retention
- **Report Size**: ~65 KB Excel workbooks
- **Frontend Build**: 944 KB total (~125 KB gzipped)
- **Page Load Time**: <2 seconds (CloudFront CDN)
- **Cache Hit Ratio**: >80% (CloudFront)

### Recent Enhancements (October 2025)

**Frontend (v2.2.0)**:
- ✅ Light/Dark mode toggle with localStorage persistence
- ✅ Mobile-responsive hamburger navigation
- ✅ What's New page with infrastructure changes and AWS announcements
- ✅ ErrorBoundary for graceful error handling
- ✅ ESLint configuration (0 errors/warnings)
- ✅ SEO improvements (Phase 1 complete)
- ✅ Dynamic region names (zero maintenance for new regions)

**Data Fetcher**:
- ✅ Node.js 20 runtime upgrade
- ✅ 24-hour intelligent caching (reduces API calls)
- ✅ 30-day historical snapshots
- ✅ Execution time: 13-30 seconds (cached), <2 minutes (fresh)

**Reporter**:
- ✅ Automated S3-triggered generation
- ✅ 4-sheet Excel workbooks with professional formatting
- ✅ 7-day archive retention
- ✅ Color-coded coverage indicators
- ✅ SNS email notifications

**Infrastructure**:
- ✅ Multi-environment support (prod, dev, staging, aws-services)
- ✅ GitHub OIDC authentication (no long-lived credentials)
- ✅ Cross-region failover (us-east-1 to us-west-2)
- ✅ CloudWatch monitoring dashboards
- ✅ Automated GitHub Actions deployments

## Future Enhancements

### Planned Features

**Frontend (High Priority)**:
1. **Automated Testing**: Vitest + React Testing Library
2. **CI/CD Pipeline**: GitHub Actions for build and deploy
3. **Dynamic Meta Tags**: react-helmet-async for SEO
4. **TypeScript Migration**: Gradual migration for type safety

**Frontend (Medium Priority)**:
1. **Data Validation**: Zod for runtime type checking
2. **Performance Monitoring**: Real User Monitoring (RUM)
3. **Accessibility Audit**: WCAG AAA compliance
4. **Progressive Web App**: Offline support and installability

**Backend**:
1. **Real-time Updates**: WebSocket integration for live data
2. **Custom Alerts**: User-defined thresholds and SNS notifications
3. **Advanced Analytics**: Trend analysis and predictions
4. **API Gateway**: RESTful API for programmatic access

### Infrastructure Improvements

1. **Blue-green Deployments**: Zero-downtime frontend updates
2. **Automated E2E Testing**: Playwright/Cypress in CI/CD pipeline
3. **Cost Optimization**: S3 Intelligent-Tiering, Reserved Lambda concurrency
4. **Compliance**: AWS Config rules, security scanning with tfsec
5. **Multi-Cloud Support**: Azure and GCP service catalogs (future)

## Technology Decisions

### Why Serverless?

- No server management overhead
- Pay-per-use pricing model
- Automatic scaling
- High availability built-in
- Reduced operational complexity

### Why React?

- Component-based architecture
- Rich ecosystem and tooling
- Excellent performance
- Strong community support
- Easy to maintain and extend

### Why Terraform?

- Infrastructure as Code
- Multi-cloud support
- State management
- Module reusability
- Strong AWS provider

### Why CloudFront + S3?

- Cost-effective static hosting
- Global CDN with low latency
- HTTPS/SSL built-in
- High availability (99.99% SLA)
- Easy integration with AWS services

## References

- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [Serverless Best Practices](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html)
- [React Best Practices](https://react.dev/learn)
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
