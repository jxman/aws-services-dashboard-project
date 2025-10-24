# AWS Services Dashboard - Architecture

## System Overview

The AWS Services Dashboard is a serverless, event-driven system for monitoring and reporting on AWS infrastructure across multiple accounts and regions.

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

**Technology**: React 18 + Vite + Tailwind CSS

**Key Features**:
- Single-page application (SPA)
- Client-side routing (React Router)
- Data visualization (Recharts)
- Responsive design
- Real-time data fetching (TanStack Query)

**Data Flow**:
1. User navigates to https://aws-services.synepho.com
2. CloudFront serves cached static assets
3. React app loads and initializes
4. TanStack Query fetches JSON data from S3 (via CloudFront)
5. Components render visualizations

**Key Files**:
- `src/App.jsx`: Main application component
- `src/views/`: Page components (Dashboard, Regions, Accounts, etc.)
- `src/components/`: Reusable UI components
- `src/hooks/`: Custom React hooks for data fetching

### 2. Data Fetcher (aws-infrastructure-fetcher)

**Technology**: Node.js 18+ running on AWS Lambda

**Execution Model**:
- Scheduled via EventBridge (configurable interval)
- Timeout: 15 minutes (configurable)
- Memory: 512MB (configurable)

**Data Collection Process**:
1. Lambda triggered by EventBridge schedule
2. Authenticates to AWS accounts (IAM roles)
3. Queries AWS services across all regions
4. Aggregates and formats data as JSON
5. Uploads to S3 distribution bucket
6. Triggers CloudFront invalidation (optional)

**Collected Data**:
- Account information (ID, name, environment)
- Regional resource distribution
- Service usage metrics
- Cost and billing data
- Resource tags and metadata

**Output Format**:
```json
{
  "lastUpdated": "2025-01-24T10:30:00Z",
  "accounts": [...],
  "regions": [...],
  "services": {...},
  "costs": {...}
}
```

### 3. Reporter (nodejs-aws-reporter)

**Technology**: Node.js 18+ running on AWS Lambda

**Execution Model**:
- Triggered by user action (API Gateway or direct invoke)
- Generates Excel workbooks using ExcelJS
- Returns downloadable file

**Report Generation Process**:
1. User clicks "Download Report" in dashboard
2. Lambda reads latest data from S3
3. Creates Excel workbook with multiple sheets
4. Formats data with styling and charts
5. Returns Excel file for download

**Report Sheets**:
- Summary: High-level overview
- Accounts: Detailed account information
- Regions: Regional distribution
- Services: Service-by-service breakdown
- Costs: Cost analysis and trends

### 4. Infrastructure (synepho-s3cf-site)

**Technology**: Terraform (Infrastructure as Code)

**Managed Resources**:

#### Hosting Infrastructure:
- **S3 Buckets**:
  - Static website bucket (React app)
  - Data distribution bucket (JSON files)
  - Logging bucket (access logs)
- **CloudFront Distribution**:
  - SSL/TLS certificates (ACM)
  - Custom domain (Route53)
  - Origin Access Control (OAC)
  - Custom error responses (SPA routing)
- **Route53**:
  - DNS records for aws-services.synepho.com
  - DNS records for synepho.com

#### Lambda Infrastructure:
- IAM roles and policies
- Lambda function deployments
- EventBridge schedules
- CloudWatch log groups

#### CI/CD Infrastructure:
- GitHub OIDC provider
- GitHub Actions IAM roles
- Deployment policies

**Terraform State Management**:
- Remote state in S3
- State locking via DynamoDB
- Encrypted at rest

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

## Future Enhancements

### Planned Features

1. **Real-time Updates**: WebSocket integration for live data
2. **User Authentication**: Cognito integration for secure access
3. **Custom Alerts**: User-defined thresholds and notifications
4. **Advanced Analytics**: ML-powered insights and predictions
5. **Multi-Cloud Support**: Azure and GCP integration

### Infrastructure Improvements

1. **Multi-region Deployment**: Active-active setup
2. **Blue-green Deployments**: Zero-downtime updates
3. **Automated Testing**: Integration and E2E tests in CI/CD
4. **Cost Optimization**: Reserved capacity and savings plans

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
