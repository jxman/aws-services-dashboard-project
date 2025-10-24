# Development Guide

Complete guide for developing across the AWS Services Dashboard ecosystem.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Initial Setup](#initial-setup)
- [Development Workflows](#development-workflows)
- [Testing](#testing)
- [Code Quality](#code-quality)
- [Troubleshooting](#troubleshooting)

---

## Prerequisites

### Required Tools

#### All Repositories
- **Git**: Version control (2.0+)
- **GitHub CLI**: `gh` command (optional but recommended)
- **VS Code**: Recommended IDE with extensions

#### Frontend Development (aws-services-site)
- **Node.js**: 18+ (LTS recommended)
- **npm**: 9+ (comes with Node.js)
- **Browser**: Chrome/Firefox/Safari (latest)

#### Lambda Development (fetcher & reporter)
- **Node.js**: 18+ (LTS recommended)
- **npm**: 9+ (comes with Node.js)
- **AWS CLI**: 2.0+ (for local testing)
- **AWS SAM CLI**: 1.0+ (optional, for local Lambda testing)

#### Infrastructure Development (synepho-s3cf-site)
- **Terraform**: 1.0+
- **AWS CLI**: 2.0+ (configured with credentials)
- **GitHub Actions**: For deployments

### AWS Credentials Setup

#### For Local Development
```bash
# Configure AWS CLI with your credentials
aws configure

# Verify access
aws sts get-caller-identity
```

#### For GitHub Actions (Production)
Use OIDC authentication (no credentials needed locally).
See [DEPLOYMENT.md](./DEPLOYMENT.md) for details.

---

## Initial Setup

### 1. Clone All Repositories

```bash
# Create project workspace
mkdir -p ~/projects/aws-services-dashboard
cd ~/projects/aws-services-dashboard

# Clone all repositories
git clone https://github.com/jxman/aws-services-site.git
git clone https://github.com/jxman/aws-infrastructure-fetcher.git
git clone https://github.com/jxman/nodejs-aws-reporter.git
git clone https://github.com/jxman/synepho-s3cf-site.git
git clone https://github.com/jxman/aws-services-dashboard-project.git
```

### 2. Frontend Setup (aws-services-site)

```bash
cd aws-services-site

# Install dependencies
npm install

# Start development server
npm run dev

# Application runs on http://localhost:5173 (Vite default)
```

**Development URLs**:
- Local dev: http://localhost:5173
- Production: https://aws-services.synepho.com

### 3. Data Fetcher Setup (aws-infrastructure-fetcher)

```bash
cd aws-infrastructure-fetcher

# Install dependencies
npm install

# Set up environment variables
cp .env.example .env
# Edit .env with your configuration

# Test locally (if SAM is installed)
sam local invoke -e events/test-event.json

# Or test with Node.js directly
node src/index.js
```

**Environment Variables**:
```bash
DISTRIBUTION_BUCKET=your-data-bucket
DISTRIBUTION_PREFIX=data/
CLOUDFRONT_DISTRIBUTION_ID=E1234567890ABC
AWS_ACCOUNTS='[{"id":"123456789","name":"Production"}]'
```

### 4. Reporter Setup (nodejs-aws-reporter)

```bash
cd nodejs-aws-reporter

# Install dependencies
npm install

# Set up environment variables
cp .env.example .env
# Edit .env with your configuration

# Test locally
npm test
```

### 5. Infrastructure Setup (synepho-s3cf-site)

```bash
cd synepho-s3cf-site

# Initialize Terraform
terraform init

# Validate configuration
terraform validate

# Format code
terraform fmt -recursive

# Plan changes (read-only)
terraform plan

# IMPORTANT: Never apply locally in production
# Use GitHub Actions for deployments
```

---

## Development Workflows

### Frontend Development Workflow

#### Adding a New View/Page

1. **Create view component**:
```bash
# Create new view file
touch src/views/NewView.jsx
```

```jsx
// src/views/NewView.jsx
export default function NewView() {
  return (
    <div className="container mx-auto p-4">
      <h1 className="text-2xl font-bold">New View</h1>
      {/* Your content */}
    </div>
  );
}
```

2. **Add route**:
```jsx
// src/App.jsx
import NewView from './views/NewView';

// In your Routes
<Route path="/new-view" element={<NewView />} />
```

3. **Add navigation** (if needed):
```jsx
// src/components/Navigation.jsx
<Link to="/new-view">New View</Link>
```

#### Adding Data Fetching

```jsx
import { useQuery } from '@tanstack/react-query';

function MyComponent() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['myData'],
    queryFn: async () => {
      const response = await fetch('https://your-cloudfront-url/data.json');
      return response.json();
    },
    staleTime: 5 * 60 * 1000, // 5 minutes
  });

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return <div>{/* Render data */}</div>;
}
```

#### Styling with Tailwind

```jsx
// Use Tailwind utility classes
<div className="bg-white shadow-md rounded-lg p-6">
  <h2 className="text-xl font-semibold text-gray-800">Title</h2>
  <p className="text-gray-600 mt-2">Content</p>
</div>
```

### Lambda Development Workflow

#### Data Fetcher - Adding a New Service Collector

1. **Create collector module**:
```javascript
// src/collectors/newService.js
export async function collectNewServiceData(client, region) {
  try {
    const command = new DescribeNewServiceCommand({});
    const response = await client.send(command);

    return {
      serviceName: 'NewService',
      region: region,
      resources: response.Items || [],
      count: response.Items?.length || 0
    };
  } catch (error) {
    console.error(`Error collecting NewService data in ${region}:`, error);
    return { serviceName: 'NewService', region, error: error.message };
  }
}
```

2. **Add to main collector**:
```javascript
// src/index.js
import { collectNewServiceData } from './collectors/newService.js';

// In your main handler
const newServiceData = await collectNewServiceData(client, region);
allData.newService = newServiceData;
```

3. **Test locally**:
```bash
# Run with test event
node src/index.js
```

#### Reporter - Adding a New Sheet

1. **Create sheet generator**:
```javascript
// src/generators/newSheet.js
export function generateNewSheet(workbook, data) {
  const sheet = workbook.addWorksheet('New Sheet');

  // Define columns
  sheet.columns = [
    { header: 'Column 1', key: 'col1', width: 20 },
    { header: 'Column 2', key: 'col2', width: 30 },
  ];

  // Add data
  data.forEach(item => {
    sheet.addRow(item);
  });

  // Style header
  sheet.getRow(1).font = { bold: true };
  sheet.getRow(1).fill = {
    type: 'pattern',
    pattern: 'solid',
    fgColor: { argb: 'FF4472C4' }
  };

  return sheet;
}
```

2. **Add to main reporter**:
```javascript
// src/index.js
import { generateNewSheet } from './generators/newSheet.js';

// In your handler
generateNewSheet(workbook, data.newData);
```

### Infrastructure Development Workflow

#### Adding a New Resource

1. **Create module** (recommended):
```bash
mkdir -p modules/new-resource
touch modules/new-resource/{main.tf,variables.tf,outputs.tf,versions.tf}
```

2. **Define resource**:
```hcl
# modules/new-resource/main.tf
resource "aws_new_resource" "example" {
  name = var.resource_name

  tags = var.tags
}
```

3. **Define variables**:
```hcl
# modules/new-resource/variables.tf
variable "resource_name" {
  type        = string
  description = "Name of the resource"
}

variable "tags" {
  type        = map(string)
  description = "Tags to apply to the resource"
  default     = {}
}
```

4. **Define outputs**:
```hcl
# modules/new-resource/outputs.tf
output "resource_id" {
  value       = aws_new_resource.example.id
  description = "ID of the created resource"
}
```

5. **Use module in main configuration**:
```hcl
# main.tf
module "new_resource" {
  source = "./modules/new-resource"

  resource_name = "my-resource"
  tags = {
    Environment = "production"
    ManagedBy   = "terraform"
  }
}
```

6. **Test changes**:
```bash
# Validate syntax
terraform validate

# Format code
terraform fmt -recursive

# Plan changes (dry run)
terraform plan

# Deploy via GitHub Actions (production)
gh workflow run "Terraform Deployment" --ref main
```

---

## Testing

### Frontend Testing

#### Unit Tests (Vitest - if configured)

```javascript
// src/components/__tests__/MyComponent.test.jsx
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import MyComponent from '../MyComponent';

describe('MyComponent', () => {
  it('renders correctly', () => {
    render(<MyComponent />);
    expect(screen.getByText('Expected Text')).toBeInTheDocument();
  });
});
```

Run tests:
```bash
npm test
```

#### Manual Testing Checklist
- [ ] Component renders without errors
- [ ] Data fetching works correctly
- [ ] Loading states display properly
- [ ] Error states display properly
- [ ] Responsive design works on mobile
- [ ] Navigation works correctly
- [ ] Charts/visualizations render correctly

### Lambda Testing

#### Unit Tests (Jest)

```javascript
// __tests__/collector.test.js
const { collectEC2Data } = require('../src/collectors/ec2');

describe('EC2 Collector', () => {
  it('should collect EC2 instance data', async () => {
    const mockClient = {
      send: jest.fn().mockResolvedValue({
        Reservations: [
          { Instances: [{ InstanceId: 'i-1234567890' }] }
        ]
      })
    };

    const result = await collectEC2Data(mockClient, 'us-east-1');

    expect(result.count).toBe(1);
    expect(result.resources).toHaveLength(1);
  });
});
```

Run tests:
```bash
npm test
```

#### Integration Testing with SAM

```bash
# Test with local event
sam local invoke DataFetcherFunction -e events/test-event.json

# Test with local API
sam local start-api
curl http://localhost:3000/fetch
```

### Infrastructure Testing

#### Validation Tests
```bash
# Syntax validation
terraform validate

# Security scanning (optional)
tfsec .

# Cost estimation (optional)
infracost breakdown --path .
```

#### Plan Testing
```bash
# Generate plan
terraform plan -out=plan.tfplan

# Review plan
terraform show plan.tfplan

# Clean up
rm plan.tfplan
```

---

## Code Quality

### Linting

#### Frontend (ESLint)
```bash
# Run linter
npm run lint

# Fix auto-fixable issues
npm run lint -- --fix
```

#### Terraform (tflint - if installed)
```bash
# Initialize tflint
tflint --init

# Run linter
tflint
```

### Code Formatting

#### JavaScript/React (Prettier - if configured)
```bash
# Format all files
npm run format

# Check formatting
npm run format:check
```

#### Terraform
```bash
# Format all .tf files
terraform fmt -recursive

# Check formatting
terraform fmt -check -recursive
```

### Pre-commit Hooks (Optional)

Install husky for automated checks:
```bash
npm install --save-dev husky lint-staged

# Add to package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.{js,jsx}": ["eslint --fix", "prettier --write"],
    "*.tf": ["terraform fmt"]
  }
}
```

---

## Troubleshooting

### Common Frontend Issues

#### Build Failures
```bash
# Clear node_modules and reinstall
rm -rf node_modules package-lock.json
npm install

# Clear Vite cache
rm -rf node_modules/.vite
npm run dev
```

#### CORS Issues
- Ensure CloudFront distribution allows proper origins
- Check S3 bucket CORS configuration
- Verify API Gateway CORS settings (if applicable)

#### React Query Issues
```javascript
// Check network tab in browser DevTools
// Verify queryKey uniqueness
// Check staleTime and cacheTime settings
```

### Common Lambda Issues

#### Timeout Errors
- Increase Lambda timeout (max 15 minutes)
- Optimize code for performance
- Use parallel processing where possible
- Check CloudWatch logs for bottlenecks

#### Memory Errors
- Increase Lambda memory allocation
- Monitor memory usage in CloudWatch
- Optimize data structures
- Use streaming for large datasets

#### IAM Permission Errors
```bash
# Check Lambda execution role
aws iam get-role --role-name YourLambdaExecutionRole

# Test IAM permissions
aws sts assume-role --role-arn arn:aws:iam::ACCOUNT:role/ROLE --role-session-name test
```

### Common Terraform Issues

#### State Lock Errors
```bash
# Check DynamoDB for lock
aws dynamodb get-item \
  --table-name terraform-state-lock \
  --key '{"LockID":{"S":"your-state-file-lock"}}'

# Force unlock (use carefully!)
terraform force-unlock LOCK_ID
```

#### Provider Errors
```bash
# Re-initialize providers
rm -rf .terraform
terraform init

# Upgrade providers
terraform init -upgrade
```

#### Plan Failures
```bash
# Refresh state
terraform refresh

# Target specific resource
terraform plan -target=module.specific_module

# Enable debug logging
TF_LOG=DEBUG terraform plan
```

### Getting Help

1. **Check documentation** in this repository
2. **Review CloudWatch logs** for Lambda functions
3. **Check GitHub Actions logs** for deployment issues
4. **Search existing issues** in the relevant repository
5. **Create new issue** with:
   - Clear description
   - Steps to reproduce
   - Expected vs actual behavior
   - Relevant logs/screenshots

---

## Best Practices

### General Development
- ✅ Write clear, descriptive commit messages
- ✅ Keep commits small and focused
- ✅ Test locally before pushing
- ✅ Update documentation with code changes
- ✅ Use feature branches for new work
- ✅ Request code reviews for significant changes

### Frontend Development
- ✅ Use functional components and hooks
- ✅ Implement proper error boundaries
- ✅ Optimize images and assets
- ✅ Use React.lazy for code splitting
- ✅ Implement proper loading states
- ✅ Handle errors gracefully

### Lambda Development
- ✅ Keep functions small and focused
- ✅ Use environment variables for configuration
- ✅ Implement proper error handling
- ✅ Log meaningful information
- ✅ Optimize cold start performance
- ✅ Use AWS SDK v3 for better tree-shaking

### Infrastructure Development
- ✅ Use modules for reusable components
- ✅ Implement proper tagging strategy
- ✅ Use remote state with locking
- ✅ Never commit sensitive data
- ✅ Use workspaces or separate state per environment
- ✅ Deploy via CI/CD (GitHub Actions)

---

## Resources

### Documentation
- [React Documentation](https://react.dev)
- [Vite Documentation](https://vitejs.dev)
- [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)

### Tools
- [AWS CLI Reference](https://docs.aws.amazon.com/cli/latest/reference/)
- [GitHub CLI](https://cli.github.com/)
- [VS Code](https://code.visualstudio.com/)

### Project-Specific
- [Architecture Documentation](./ARCHITECTURE.md)
- [Repository Directory](./REPOSITORIES.md)
- [Deployment Guide](./DEPLOYMENT.md)
