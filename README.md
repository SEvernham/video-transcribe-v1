# video-transcribe-v1

# Cross-Account Access Monitor - CloudFormation Deployment

A comprehensive AWS CloudFormation solution that monitors and alerts on cross-account access to your AWS resources. This solution detects when external AWS accounts access resources in your account and sends real-time email notifications via Amazon SNS.

## 🎯 Purpose

This CloudFormation template creates a complete cross-account access monitoring infrastructure designed for organizations that need:

- **Security Monitoring**: Detect unauthorized cross-account access attempts
- **Compliance Requirements**: Maintain audit trails of external account access
- **Incident Response**: Get immediate alerts when external accounts access your resources
- **Organizational Oversight**: Monitor cross-account access patterns across your AWS environment

## 🏗️ Architecture Overview

The solution creates an integrated monitoring pipeline:

```
AWS API Calls → CloudTrail → EventBridge → Lambda → SNS Email Alerts
```

### Components Created

1. **CloudTrail Trail** - Captures management events across all AWS services
2. **S3 CloudTrail Bucket** - Stores CloudTrail logs with automatic lifecycle management
3. **EventBridge Rule** - Filters cross-account access events with customizable service patterns
4. **Lambda Function** - Processes events, validates cross-account access, and filters organization members
5. **SNS Topic & Subscription** - Sends email notifications for confirmed external access
6. **IAM Roles & Policies** - Secure permissions including AWS Organizations integration
7. **Custom Resource** - Dynamically builds EventBridge patterns based on monitored services

## 📊 What Gets Monitored

### Cross-Account Access Detection
- **IAM Operations**: Role assumptions, policy changes, user management
- **STS Operations**: Security token requests, federation activities
- **S3 Management**: Bucket policies, ACL changes, bucket operations
- **DynamoDB Management**: Table policies, backup operations, table management
- **EC2 Management**: Instance operations, security group changes, VPC modifications
- **RDS Management**: Database operations, parameter changes, snapshot sharing
- **Lambda Management**: Function policies, layer operations, function management

### Information Collected
- **Who**: External account ID, user identity, principal ARN, session details
- **What**: AWS service accessed, specific operation performed, resources affected
- **When**: Precise timestamp of access attempt
- **Where**: AWS region, source IP address, user agent
- **Context**: Request parameters, response elements, error details

## 📁 What Gets Created

### AWS Resources

| Resource Type | Purpose | Configuration |
|---------------|---------|---------------|
| **CloudTrail Trail** | Captures management events | Multi-region, log file validation enabled |
| **S3 Bucket (CloudTrail)** | Stores raw CloudTrail logs | 90-day lifecycle, versioning enabled |
| **EventBridge Rule** | Filters cross-account events | Dynamic pattern based on monitored services |
| **Lambda Function** | Processes and validates events | Python 3.11, Organizations API integration |
| **SNS Topic** | Email notification delivery | Cross-account access alerts |
| **Custom Resource** | Dynamic configuration | Builds EventBridge patterns from parameters |
| **IAM Roles** | Secure service permissions | Least privilege access |

### Monitoring Scope

- **Event Type**: Management events only (for regional compatibility)
- **Service Coverage**: Configurable via `MonitoredServices` parameter
- **Account Filtering**: Automatic exclusion of organization member accounts
- **Alert Delivery**: Real-time email notifications via SNS

## 🚀 Deployment Instructions

### Prerequisites

- AWS account with CloudFormation deployment permissions
- Valid email address for receiving notifications
- (Optional) AWS Organization ID for filtering member account access
- Permissions to create IAM roles, CloudTrail, Lambda functions, EventBridge rules, and S3 buckets

### Step 1: Access CloudFormation Console

1. Sign in to the AWS Management Console
2. Navigate to **CloudFormation** service
3. Ensure you're in your primary AWS region (us-east-1 recommended)

### Step 2: Create Stack

1. Click **Create stack** → **With new resources (standard)**
2. In the **Specify template** section:
   - Select **Upload a template file**
   - Click **Choose file** and select `cross-account-access-monitor-v2.5.yaml`
   - Click **Next**

### Step 3: Configure Parameters

| Parameter | Description | Default | Notes |
|-----------|-------------|---------|-------|
| **NotificationEmail** | Email address for alerts | *Required* | Must be valid email format |
| **OrganizationId** | AWS Organization ID | `""` | Optional - filters out member accounts |
| **MonitoredServices** | Services to monitor | `dynamodb,s3,ec2,rds,lambda,iam,sts` | Comma-separated list |

#### Service Options for MonitoredServices:
- **iam**: IAM role assumptions, policy changes
- **sts**: Security Token Service operations
- **dynamodb**: DynamoDB management operations
- **s3**: S3 bucket management operations
- **ec2**: EC2 instance and VPC operations
- **rds**: RDS database operations
- **lambda**: Lambda function management

### Step 4: Configure Stack Options

1. **Stack name**: Enter descriptive name (e.g., `cross-account-access-monitor`)
2. **Tags**: Add organizational tags (recommended: Environment, Owner, Purpose)
3. **Permissions**: Leave as default unless using custom execution role
4. **Advanced options**: Leave as default
5. Click **Next**

### Step 5: Review and Deploy

1. Review all configuration details carefully
2. **⚠️ Critical**: Check the box acknowledging CloudFormation will create IAM resources with custom names
3. Click **Submit**
4. Wait for **CREATE_COMPLETE** status (typically 5-10 minutes)

### Step 6: Confirm Email Subscription

1. Check your email for SNS subscription confirmation
2. Click the confirmation link to activate notifications
3. **Important**: Alerts will not be sent until subscription is confirmed

### Step 7: Verify Deployment

1. Check the **Outputs** tab for resource information
2. Verify EventBridge rule shows your monitored services
3. Test with cross-account operations (see Testing section)

## 🧪 Testing the Solution

### Test Files Provided

The solution includes two test utilities:

#### 1. Python Test Script (`test-cross-account-event.py`)

**Purpose**: Comprehensive testing with multiple event types

**Usage**:
```bash
# 1. Update account IDs in the script
# Edit lines with 987654321098 (external) and 123456789012 (monitored)

# 2. View test events
python test-cross-account-event.py

# 3. Test Lambda function directly
# Uncomment test_lambda_function_direct() and run
python test-cross-account-event.py

# 4. Test full EventBridge pipeline
# Uncomment send_test_event_to_eventbridge() and run
python test-cross-account-event.py
```

**Features**:
- Creates realistic CloudTrail event structures
- Tests both DynamoDB and IAM cross-account scenarios
- Supports direct Lambda testing and full pipeline testing
- Includes detailed event structure for manual inspection

#### 2. Bash Test Script (`send-test-event.sh`)

**Purpose**: Quick testing with multiple service scenarios

**Usage**:
```bash
# 1. Update configuration variables
CURRENT_ACCOUNT="123456789012"  # Your monitored account
EXTERNAL_ACCOUNT="987654321098"  # Test external account
REGION="us-east-1"              # Your deployment region

# 2. Make executable and run
chmod +x send-test-event.sh
./send-test-event.sh
```

**Features**:
- Tests DynamoDB, IAM, and S3 cross-account access
- Sends events directly to EventBridge
- Provides comprehensive monitoring guidance
- Includes troubleshooting instructions

### Testing Workflow

1. **Deploy the solution** using CloudFormation
2. **Confirm email subscription** from SNS
3. **Update test files** with your actual account IDs
4. **Run tests** using either Python or Bash script
5. **Monitor results**:
   - Check email for notifications (1-2 minutes)
   - View Lambda logs: `/aws/lambda/CrossAccountAccessProcessor`
   - Verify EventBridge rule execution

### Expected Test Results

**Successful Test Indicators**:
- ✅ Email notifications received for external account access
- ✅ Lambda function logs show event processing
- ✅ No alerts for same-account operations
- ✅ Organization member accounts filtered out (if configured)

## 📧 Alert Format

Each alert email contains comprehensive cross-account access information:

```
Subject: Cross-Account Access Alert - Account 123456789012

CROSS-ACCOUNT ACCESS DETECTED

Alert Time: 2025-07-02 15:30:45 UTC

ACCESS DETAILS:
- Source Account: 987654321098
- Target Account: 123456789012
- Event: AssumeRole
- Service: iam.amazonaws.com
- Event Time: 2025-07-02T15:30:30Z
- Region: us-east-1

USER IDENTITY:
- Type: AssumedRole
- Principal ID: AIDACKCEVSQ6C2EXAMPLE:CrossAccountSession
- ARN: arn:aws:sts::987654321098:assumed-role/CrossAccountRole/CrossAccountSession
- User Name: CrossAccountRole

NETWORK DETAILS:
- Source IP: 203.0.113.15
- User Agent: aws-sdk-python/1.34.0

MONITORING CONFIGURATION:
- Monitored Services: dynamodb,s3,ec2,rds,lambda,iam,sts
- Event Sources: ["iam.amazonaws.com", "sts.amazonaws.com", ...]
- Event Type: Management Events Only

RECOMMENDATION:
Please review this access to ensure it is authorized...
```

## 🔍 Monitoring and Troubleshooting

### CloudWatch Logs

**Lambda Function Logs**: `/aws/lambda/CrossAccountAccessProcessor`

**Common Log Messages**:
- `Cross-account access detected from {account}` - Alert triggered
- `Same account access - skipping` - Internal access ignored
- `Organization member access - ignored` - Member account filtered
- `No source account ID found - skipping` - Malformed event

### EventBridge Rule Monitoring

**Rule Name**: `CrossAccountAccessDetection`

**Verification Commands**:
```bash
# Check rule configuration
aws events describe-rule --name CrossAccountAccessDetection

# View rule metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/Events \
  --metric-name MatchedEvents \
  --dimensions Name=RuleName,Value=CrossAccountAccessDetection \
  --start-time $(date -d '1 hour ago' -u +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 300 \
  --statistics Sum
```

### Common Issues and Solutions

#### No Alerts Received
1. **Email Subscription**: Confirm SNS subscription via email link
2. **Account Configuration**: Verify external account IDs are different from monitored account
3. **Organization Filtering**: Check if accounts are in same AWS Organization
4. **Service Monitoring**: Ensure tested services are in `MonitoredServices` parameter

#### False Positives
1. **Organization Setup**: Add Organization ID to filter member accounts
2. **Service Filtering**: Reduce monitored services to focus on critical ones
3. **Custom Logic**: Modify Lambda function for additional filtering

#### High Alert Volume
1. **Service Scope**: Reduce `MonitoredServices` to essential services only
2. **Organization Filtering**: Ensure Organization ID is configured
3. **Time-based Filtering**: Add custom logic for business hours only

## 💰 Cost Estimation

### Monthly Cost Breakdown (Moderate Usage)

| Service | Component | Estimated Cost |
|---------|-----------|----------------|
| **CloudTrail** | Management events (first copy free) | $0.00 |
| **Lambda** | 1,000 invocations + compute time | $0.20 |
| **EventBridge** | 50,000 events processed | $0.05 |
| **SNS** | 100 email notifications | $0.01 |
| **S3 (CloudTrail)** | 5GB storage + requests | $0.50 |
| **CloudWatch Logs** | Lambda execution logs | $0.25 |
| **Total** | | **~$1.00/month** |

### Cost by Usage Level

| Usage Level | Cross-Account Events/Month | Monthly Cost |
|-------------|---------------------------|--------------|
| **Light** | 100 | $0.50 - $1.00 |
| **Moderate** | 1,000 | $1.00 - $3.00 |
| **Heavy** | 10,000 | $3.00 - $8.00 |
| **Enterprise** | 100,000 | $15.00 - $25.00 |

### Cost Optimization Tips

1. **Management Events Only**: Current configuration uses free management events
2. **Service Filtering**: Monitor only critical services to reduce processing
3. **Organization Filtering**: Reduces false positives and processing costs
4. **Lifecycle Management**: 90-day S3 lifecycle keeps storage costs low

## 🛠️ Advanced Configuration

### Custom Service Monitoring

Modify the `MonitoredServices` parameter to focus on specific services:

```bash
# Security-focused monitoring
MonitoredServices: "iam,sts"

# Compute-focused monitoring  
MonitoredServices: "ec2,lambda,rds"

# Data-focused monitoring
MonitoredServices: "s3,dynamodb"

# Comprehensive monitoring (default)
MonitoredServices: "dynamodb,s3,ec2,rds,lambda,iam,sts"
```

### Organization Integration

For AWS Organizations, configure the `OrganizationId` parameter:

```bash
# Find your Organization ID
aws organizations describe-organization --query 'Organization.Id'

# Use in deployment
OrganizationId: "o-1234567890"
```

### Lambda Function Customization

The Lambda function can be extended for:
- Custom filtering logic based on source IP ranges
- Integration with external security systems
- Custom alert formatting and routing
- Additional AWS service monitoring

### EventBridge Integration

The solution can be extended to:
- Route events to multiple targets (Lambda, SQS, SNS)
- Add custom event transformation
- Integrate with AWS Security Hub
- Forward events to external SIEM systems

## 🧹 Cleanup

To remove all resources and stop charges:

### Option 1: CloudFormation Console
1. Go to CloudFormation console
2. Select your stack (`cross-account-access-monitor`)
3. Click **Delete**
4. Confirm deletion

### Option 2: AWS CLI
```bash
aws cloudformation delete-stack --stack-name cross-account-access-monitor
```

**Note**: The S3 CloudTrail bucket may need manual emptying if it contains logs.

## 🔒 Security Considerations

### Access Control
- **IAM Roles**: Solution uses least-privilege IAM roles
- **Resource Isolation**: Each component has minimal required permissions
- **Organization Integration**: Automatic filtering of trusted member accounts

### Data Protection
- **Encryption**: CloudTrail logs support S3 encryption
- **Access Logging**: All component access is logged
- **Network Security**: Source IP addresses captured for analysis

### Compliance
- **Audit Trail**: Complete audit trail of cross-account access
- **Data Retention**: Configurable retention periods for compliance
- **Alert Records**: SNS maintains delivery records for audit purposes

## 📈 Monitoring Solution Health

Monitor these key metrics for solution health:

### CloudWatch Metrics
- **Lambda Function**: Invocations, errors, duration
- **EventBridge Rule**: Matched events, failed invocations
- **SNS Topic**: Messages published, delivery failures
- **CloudTrail**: Log delivery status, errors

### Health Check Commands
```bash
# Check Lambda function health
aws lambda get-function --function-name CrossAccountAccessProcessor

# Verify EventBridge rule status
aws events describe-rule --name CrossAccountAccessDetection

# Check SNS topic configuration
aws sns get-topic-attributes --topic-arn arn:aws:sns:region:account:CrossAccountAccessAlerts

# Monitor CloudTrail status
aws cloudtrail get-trail-status --name CrossAccountAccessTrail
```

## 🔧 Version Information

**Current Version**: 2.5  
**Template File**: `cross-account-access-monitor-v2.5.yaml`  
**Test Files**: `test-cross-account-event.py`, `send-test-event.sh`  

### Version History
- **v2.5**: Fixed CloudFormation syntax errors, functional MonitoredServices parameter
- **v2.4**: Made MonitoredServices parameter functional with dynamic EventBridge patterns
- **v2.3**: Simplified CloudTrail configuration for regional compatibility
- **v2.2**: Fixed S3 bucket policy for CloudTrail
- **v2.1**: Added required IsLogging property
- **v2.0**: Fixed EventBridge patterns and Lambda logic
- **v1.0**: Initial release

---

## License

**PRODUCTION DEPLOYMENT NOTICE**

This solution is intended as a reference architecture and is provided for educational and operational purposes.

**Required before production deployment:**
- Security review and penetration testing
- Compliance validation for your industry
- Performance testing under expected load
- Integration testing with existing security tools
- Disaster recovery and backup procedures

**Disclaimer**: No warranty is provided, express or implied. Production use requires thorough evaluation and testing for your specific environment and security requirements.
