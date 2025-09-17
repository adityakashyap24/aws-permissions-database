# GitHub-based AWS Service Access with Harness IDP

A simple, scalable solution for managing AWS service access using GitHub as your database and Harness IDP for dynamic workflows.

## 🎯 What It Does

1. **Select User Group** → Dynamic picker loads from GitHub
2. **Choose AWS Services** → Automatically shows only AWS services you can access (EC2, RDS, S3, etc.)
3. **Get AWS Access** → Instant AWS credentials and console access for approved services
4. **Audit Trail** → Everything logged in GitHub automatically

## 📁 Simple Structure

```
├── .harness/
│   └── workflows/
│       └── aws-service-access-workflow.yaml  # Main AWS workflow
├── permissions-data/
│   ├── user-groups.json          # User groups and AWS service permissions
│   └── aws-services-registry.json # AWS services catalog with actions/costs
└── access-logs/                  # Auto-generated AWS access logs
```

## 🚀 Quick Setup

### 1. GitHub Setup
```bash
# Create repository named 'aws-permissions-database'
# Add your data files
git add permissions-data/
git commit -m "Add AWS permissions data"
git push
```

### 2. Harness IDP Setup
```bash
# Copy workflow to your repo
cp .harness/ /path/to/your/repo/
# Configure GitHub connector in Harness
# Add GITHUB_TOKEN to Harness secrets
```

### 3. Use It
1. Go to Harness IDP → Workflows
2. Select "AWS Service Access Request" 
3. Pick your user group → AWS services auto-populate
4. Select services + environment → Get instant AWS access

## 📊 Data Format

### User Groups (`user-groups.json`)
```json
{
  "userGroups": {
    "ug1": {
      "name": "Cloud Infrastructure Admins",
      "description": "Full AWS infrastructure management access",
      "awsServices": ["ec2", "rds", "s3", "vpc", "iam", "lambda"],
      "environment": ["dev", "staging", "prod"]
    },
    "ug2": {
      "name": "Development Team",
      "description": "Development environment access",
      "awsServices": ["ec2", "s3", "lambda", "cloudwatch"],
      "environment": ["dev", "staging"]
    }
  }
}
```

### AWS Services (`aws-services-registry.json`)
```json
{
  "awsServices": {
    "ec2": {
      "name": "Amazon EC2",
      "description": "Virtual servers in the cloud",
      "category": "compute",
      "actions": ["ec2:RunInstances", "ec2:TerminateInstances"],
      "consoleUrl": "https://console.aws.amazon.com/ec2/",
      "estimatedCost": "variable",
      "riskLevel": "medium"
    },
    "rds": {
      "name": "Amazon RDS", 
      "description": "Managed database service",
      "category": "database",
      "actions": ["rds:CreateDBInstance", "rds:DeleteDBInstance"],
      "consoleUrl": "https://console.aws.amazon.com/rds/",
      "estimatedCost": "medium",
      "riskLevel": "high"
    }
  }
}
```

## 💫 How It Works

### Dynamic AWS Service Matching
The workflow automatically:
1. Fetches your user group AWS service permissions from GitHub
2. Filters AWS services based on what services you have access to
3. Only shows AWS services you can actually use
4. Validates environment access (dev/staging/prod)
5. No hardcoded mappings - scales to 1000s of user groups

### Example Flow
```
User: "ug1" (Cloud Infrastructure Admins)
AWS Services: ["ec2", "rds", "s3", "vpc", "iam", "lambda"]
Environment: ["dev", "staging", "prod"]

Available AWS Services:
✅ EC2 (in user permissions) → ✅ Access granted  
✅ RDS (in user permissions) → ✅ Access granted
✅ S3 (in user permissions) → ✅ Access granted
❌ Redshift (not in user permissions) → ❌ Access denied

Environment Access:
✅ Production → ✅ Access granted (user has prod access)
```

## 🔍 GitHub Query Patterns

Direct API access for AWS service permissions:
```bash
# Query user groups
curl "https://api.github.com/repos/your-org/aws-permissions-database/contents/permissions-data/user-groups.json?ug=ug1"

# Get specific user group AWS services
curl -s "https://raw.githubusercontent.com/your-org/aws-permissions-database/main/permissions-data/user-groups.json" | jq '.userGroups.ug1.awsServices'

# List all user groups  
curl -s "https://raw.githubusercontent.com/your-org/aws-permissions-database/main/permissions-data/user-groups.json" | jq -r '.userGroups | keys[]'

# Get AWS service details
curl -s "https://raw.githubusercontent.com/your-org/aws-permissions-database/main/permissions-data/aws-services-registry.json" | jq '.awsServices.ec2'
```

## 🛡️ Security & Audit

- **GitHub-native**: All AWS permissions stored in GitHub with version control
- **Automatic logging**: Every AWS access request logged to `access-logs/`
- **AWS IAM integration**: Generates AWS role ARNs and session tokens
- **Environment-based**: Separate access controls for dev/staging/prod
- **Risk assessment**: Each AWS service has risk level (low/medium/high/critical)
- **Cost tracking**: Estimated costs for each AWS service

## ⚙️ Configuration

### Required Harness Secrets
- `GITHUB_TOKEN`: GitHub Personal Access Token with repo permissions

### Required GitHub Repository
- `aws-permissions-database` (or change name in workflow)

### Environment Variables
```bash
# In Harness workflow parameters
GITHUB_ORG=your-org
GITHUB_REPO=aws-permissions-database
```

## 🔧 Customization

### Add New User Group
```json
// In user-groups.json
"new-team": {
  "name": "New Team",
  "description": "Team description", 
  "awsServices": ["ec2", "s3"],
  "environment": ["dev"]
}
```

### Add New AWS Service
```json
// In aws-services-registry.json  
"new-service": {
  "name": "AWS New Service",
  "description": "Service description",
  "category": "compute",
  "actions": ["newservice:CreateResource"],
  "consoleUrl": "https://console.aws.amazon.com/newservice/",
  "estimatedCost": "low",
  "riskLevel": "medium"
}
```

**That's it!** The workflow automatically handles the rest.

## 📈 AWS-Specific Features

### Service Categories
- **Compute**: EC2, Lambda, ECS, EKS
- **Database**: RDS, DynamoDB
- **Storage**: S3, Backup
- **Networking**: VPC, Route53, CloudFront, ELB
- **Security**: IAM, KMS, GuardDuty, Security Hub
- **Analytics**: Redshift, Glue, Athena, SageMaker
- **Management**: CloudFormation, CloudWatch, Config
- **Developer Tools**: CodeBuild, CodeDeploy, CodePipeline

### Environment Controls
- **Development**: Lower risk services, cost-optimized
- **Staging**: Production-like with some restrictions
- **Production**: Full service access with audit requirements

### Risk Management
- **Low Risk**: CloudWatch, Cost Explorer, Lambda
- **Medium Risk**: EC2, S3, ELB
- **High Risk**: RDS, VPC, CloudFormation  
- **Critical Risk**: IAM, KMS (requires additional approval)

### Cost Estimation
- **Free**: IAM, CloudFormation, CodeDeploy
- **Low**: Lambda, CloudWatch, Route53
- **Medium**: RDS, CloudFront, QuickSight
- **High**: Redshift, SageMaker, Direct Connect
- **Variable**: EC2, ECS (depends on usage)

## 🔗 AWS Access Patterns

### From Harness IDP
```
Workflow → Select UG → Pick AWS Services → Select Environment → Get AWS Credentials
```

### From AWS CLI
```bash
# Use generated AWS role
aws sts assume-role \
  --role-arn "arn:aws:iam::123456789012:role/HarnessIDP-CloudInfrastructureAdmins" \
  --role-session-name "HarnessIDP-ug1-1234567890" \
  --external-id "harness-idp-ug1"
```

### From AWS Console
```
Use provided console URLs with session token for direct service access
```

## 🎯 Real-World Examples

### Infrastructure Team (ug1)
- **Services**: EC2, RDS, S3, VPC, IAM, Lambda, CloudFormation, Route53
- **Environments**: dev, staging, prod  
- **Use Case**: Full infrastructure management

### Development Team (ug2)
- **Services**: EC2, S3, Lambda, CloudWatch, DynamoDB
- **Environments**: dev, staging
- **Use Case**: Application development and testing

### Database Team (ug3)
- **Services**: RDS, DynamoDB, S3, Backup, CloudWatch, KMS
- **Environments**: dev, staging, prod
- **Use Case**: Database administration and backup management

### Security Team (ug4)
- **Services**: IAM, KMS, GuardDuty, Security Hub, CloudTrail, Config
- **Environments**: dev, staging, prod
- **Use Case**: Security monitoring and compliance

No external dependencies. No complex setup. Just GitHub + Harness IDP = Scalable AWS service access management.