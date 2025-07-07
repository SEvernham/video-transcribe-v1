# video-transcribe-v1

# Video Subtitles AWS Translate - CloudFormation Deployment

A comprehensive AWS CloudFormation solution that automates the process of generating, reviewing, and translating subtitles for video content. This solution uses AWS Transcribe for initial transcription, Amazon A2I for human review, and AWS Translate for Spanish translation.

## 🎯 Purpose

This CloudFormation template creates a complete video subtitle generation pipeline designed for organizations that need:

- **Automated Transcription**: Generate accurate subtitles from video content
- **Human Review**: Enable human reviewers to correct and improve transcription quality
- **Multilingual Support**: Automatically translate subtitles to Spanish
- **Workflow Automation**: Orchestrate the entire process with minimal manual intervention
- **Quality Control**: Ensure high-quality subtitles through human review

## 🏗️ Architecture Overview

The solution creates an integrated subtitle generation pipeline:

```
Video Upload → AWS Transcribe → Human Review (A2I) → AWS Translate → Final Output
```

### Components Created

1. **S3 Buckets** - Input and output storage for media files and transcriptions
2. **Lambda Functions** - Process transcriptions, create human review tasks, and translate content
3. **Step Functions Workflow** - Orchestrates the entire subtitle generation process
4. **A2I Flow Definition** - Configures human review tasks for transcription verification
5. **SNS Topic & Subscription** - Sends notifications about workflow progress and errors
6. **IAM Roles & Policies** - Secure permissions for all components
7. **S3 Event Notifications** - Triggers the workflow when new videos are uploaded

## 📊 What Gets Processed

### Media Types Supported
- **Video Formats**: MP4, MOV, WebM
- **Audio Formats**: M4A

### Processing Steps
1. **Transcription**: AWS Transcribe generates initial subtitles
2. **Confidence Evaluation**: Low-confidence transcriptions are flagged for review
3. **Human Review**: Workers verify and correct transcriptions
4. **Translation**: Corrected subtitles are translated to Spanish
5. **Output Generation**: Final SRT files (English and Spanish) are saved

## 📁 What Gets Created

### AWS Resources

| Resource Type | Purpose | Configuration |
|---------------|---------|---------------|
| **S3 Buckets** | Store input videos and output subtitles | Input and output buckets |
| **Lambda Functions** | Process transcriptions and translations | Python 3.11 runtime |
| **Step Functions** | Orchestrate the workflow | State machine with error handling |
| **A2I Flow Definition** | Configure human review tasks | Custom HTML UI |
| **SNS Topic** | Send workflow notifications | Email notifications |
| **IAM Roles** | Secure service permissions | Least privilege access |
| **S3 Event Notifications** | Trigger workflow on upload | Configured for supported media types |

## ⚠️ Prerequisites (Manual Setup Required)

Before deploying this CloudFormation template, you **MUST** complete these manual setup steps:

### 1. Create an SNS Topic for Notifications

1. **Navigate to SNS Console**:
   - Go to the AWS Management Console
   - Search for "SNS" or navigate to the SNS service

2. **Create a New Topic**:
   - Click "Topics" on left pane.
   - Click "Create topic"
   - Select "Standard" type
   - Enter a name (e.g., "VideoTranscriptionNotifications")
   - Enter a display name (e.g., "VideoTranscribe")
   - Click "Create topic"

3. **Create a Subscription**:
   - With your new topic selected, click "Create subscription"
   - Select "Email" from the Protocol dropdown
   - Enter your email address in the Endpoint field
   - Click "Create subscription"
   - Check your email and confirm the subscription

4. **Copy the SNS Topic ARN**:
   - From the topic details page, copy the ARN (it will look like: `arn:aws:sns:region:account-id:VideoTranscriptionNotifications`)
   - Save this ARN for use when launching the CloudFormation template

### 2. Create an Amazon A2I Private Workforce

1. **Navigate to Amazon SageMaker Console**:
   - Go to the AWS Management Console
   - Search for "Amazon SageMaker AI" or navigate to the SageMaker service

2. **Create a Private Workforce**:
   - In the left navigation pane, click on "Ground Truth"
   - In the left navigation pane, click on "Labeling workforces"
   - Click on "Private" tab
   - Click "Create private team"
   - Enter a team name (e.g., "TranscriptionReviewers")
   - Choose "Add workers by email"
   - Enter email addresses for your workers
   - Enter an organization name (e.g., "Transciption Team")
   - Enter a contact email
   - In the SNS topic drop down, select "Create new topic"
   - Enter a name for the private workforce notification topic (e.g., "Subtitle_Review_Task")
   - Click "Create"
   - Click "Create private team"

3. **Copy the Workteam ARN**:
   - After the team is created, locate the team name in the Private teams section of the page
   - Copy the Workteam ARN (it will look like: `arn:aws:sagemaker:region:account-id:workteam/private-crowd/TranscriptionReviewers`)
   - Save this ARN for use when launching the CloudFormation template

4. **Note the Labeling Portal URL**:
   - On the same page, note the "Labeling portal sign-in URL"
   - Save this URL for use when launching the CloudFormation template

### 3. Create a Human Task UI

1. **Navigate to Amazon A2I Console**:
   - Select "Augmented AI" from the left pane of the Amazon SageMaker AI console 

2. **Create a Human Task UI**:
   - In the left navigation pane, click on "Worker tasks templates"
   - Click "Create template"
   - Enter a name (e.g., "video-transcription-review-ui")
   - For the template source, select "Upload template file"
   - Copy and paste the contents of the `HumanTask2.html` file into the template editor (be sure to remove the previous contents)
   - Click "Create"

3. **Copy the Human Task UI ARN**:
   - After the UI is created, click on its name
   - Copy the Human Task UI ARN (it will look like: `arn:aws:sagemaker:region:account-id:human-task-ui/VideoTranscriptionReviewUI`)
   - Save this ARN for use when launching the CloudFormation template

## 🚀 Deployment Instructions

### Step 1: Access CloudFormation Console

1. Sign in to the AWS Management Console
2. Navigate to **CloudFormation** service
3. Ensure you're in your desired AWS region

### Step 2: Create Stack

1. Click **Create stack** → **With new resources (standard)**
2. In the **Specify template** section:
   - Select **Upload a template file**
   - Click **Choose file** and select `VideoSubtitles-AWS-Translate-Single.yml`
   - Click **Next**

### Step 3: Configure Parameters

| Parameter | Description | Default | Notes |
|-----------|-------------|---------|-------|
| **Stack name** | Enter descriptive name for the stack | *Required* | (e.g., `video-subtitles-workflow`) |
| **WorkteamArn** | ARN of the A2I workteam | *Required* | From prerequisite step 2.3 |
| **HumanTaskUiArn** | ARN of the A2I human task UI | *Required* | From prerequisite step 3.3 |
| **LabelingPortalURL** | URL for the A2I Labeling portal | *Required* | From prerequisite step 2.4 |
| **EnvironmentName** | Environment name for S3 buckets | *Required* | Lowercase letters, numbers, hyphens only |
| **NotificationTopicArn** | ARN of the SNS Topic | *Required* | From prerequisite step 1.4 |
| **ConfidenceThreshold** | Confidence threshold for human review | 0.95 | Value between 0.0 and 1.0 |

### Step 4: Configure Stack Options

1. **Tags**: Add organizational tags if needed
2. **Permissions**: Leave as default unless using custom execution role
3. **Stack failure options**: Leave as default
4. **Additional Settings**: Leave as default
5. **⚠️ Capabilities**: Check the box acknowledging CloudFormation will create IAM resources with custom names
6. Click **Next**

### Step 5: Review and Deploy

1. Review all configuration details carefully
2. Click **Submit**
3. Wait for **CREATE_COMPLETE** status (typically 5-10 minutes)

## 🧪 Testing the Solution

### Test Workflow

1. **Upload a Test Video**:
   - Navigate to the input S3 bucket (named `{EnvironmentName}-input-bucket`)
   - Upload a video file (.mp4, .mov, .m4a, or .webm)

2. **Monitor the Workflow**:
   - Go to the Step Functions console
   - Find the state machine named `{StackName}-TranscriptionWorkflow` or similar
   - Monitor the execution progress

3. **Complete Human Review Task**:
   - When the workflow reaches the human review step, you'll receive an email notification
   - Click the link to the labeling portal or navigate directly to the URL provided during setup
   - Log in and complete the review task
   - Submit the reviewed subtitles

4. **Verify Output**:
   - Check the output S3 bucket (named `{EnvironmentName}-output-bucket`)
   - Verify that both English and Spanish SRT files are generated
   - Also check the input bucket for the final SRT files copied alongside the original video

### Expected Results

- **Transcription**: Initial SRT file generated from the video
- **Human Review**: Corrected SRT file after human review
- **Translation**: Spanish SRT file generated from the corrected English version
- **Final Output**: Both English and Spanish SRT files copied to the input bucket alongside the original video

## 📧 Notifications

The workflow sends email notifications via SNS at various stages:

- **Transcription Started**: When a new video is uploaded and transcription begins
- **Transcription Completed**: When AWS Transcribe finishes processing
- **Human Review Task Created**: When a task is sent for human review
- **Human Review Completed**: When the review task is finished
- **Translation Completed**: When Spanish translation is complete
- **Workflow Error**: If any step in the process fails

## 🔍 Monitoring and Troubleshooting

### CloudWatch Logs

Each Lambda function has its own log group for monitoring:

- `/aws/lambda/{StackName}-start-transcribe-job`
- `/aws/lambda/{StackName}-check-transcription-status`
- `/aws/lambda/{StackName}-evaluate-confidence-score`
- `/aws/lambda/{StackName}-create-human-loop`
- `/aws/lambda/{StackName}-check-a2i-task-status`
- `/aws/lambda/{StackName}-write-final-output`
- `/aws/lambda/{StackName}-translate-to-spanish`
- `/aws/lambda/{StackName}-copy-reviewed-srt`

### Step Functions Execution

The Step Functions state machine provides a visual representation of the workflow execution, making it easy to identify where issues occur.

### Common Issues and Solutions

#### No Workflow Triggered on Upload
1. **File Format**: Ensure you're uploading a supported format (.mp4, .mov, .m4a, .webm)
2. **S3 Event**: Check S3 event notifications are properly configured
3. **Lambda Permissions**: Verify the trigger Lambda has proper permissions

#### Human Review Task Not Created
1. **A2I Configuration**: Verify the WorkteamArn and HumanTaskUiArn are correct
2. **Confidence Score**: Check if the confidence score is above the threshold
3. **Lambda Logs**: Check the create-human-loop Lambda logs for errors

#### Translation Issues
1. **SRT Format**: Ensure the SRT file format is correct after human review
2. **Lambda Permissions**: Verify the translate Lambda has permissions to use AWS Translate
3. **Region Support**: Confirm AWS Translate supports your language pair in your region

## 💰 Cost Estimation

### Monthly Cost Breakdown (Moderate Usage)

| Service | Component | Estimated Cost |
|---------|-----------|----------------|
| **AWS Transcribe** | 10 hours of video | $15.00 |
| **Amazon A2I** | 50 human review tasks | $25.00 |
| **AWS Translate** | 100,000 characters | $2.00 |
| **Lambda** | 500 invocations + compute time | $0.50 |
| **Step Functions** | 100 workflow executions | $0.50 |
| **S3** | Storage + requests | $1.00 |
| **SNS** | 500 email notifications | $0.05 |
| **Total** | | **~$44.05/month** |

### Cost by Usage Level

| Usage Level | Hours of Video/Month | Monthly Cost |
|-------------|----------------------|--------------|
| **Light** | 5 | $20 - $30 |
| **Moderate** | 10 | $40 - $50 |
| **Heavy** | 25 | $90 - $110 |
| **Enterprise** | 100+ | $350+ |

### Cost Optimization Tips

1. **Confidence Threshold**: Adjust the confidence threshold to balance human review needs
2. **Video Preprocessing**: Improve audio quality before uploading to reduce transcription errors
3. **Batch Processing**: Process videos in batches rather than individually
4. **Reserved Instances**: For high-volume processing, consider reserved instances for Lambda

## 🧹 Cleanup

To remove all resources and stop charges:

### Option 1: CloudFormation Console
1. Go to CloudFormation console
2. Select your stack
3. Click **Delete**
4. Confirm deletion

### Option 2: AWS CLI
```bash
aws cloudformation delete-stack --stack-name your-stack-name
```

**Note**: The S3 buckets may need manual emptying before deletion if they contain files.

## 🔒 Security Considerations

### Access Control
- **IAM Roles**: Solution uses least-privilege IAM roles
- **S3 Buckets**: Private buckets with appropriate access policies
- **Human Review**: Only authorized workers can access review tasks

### Data Protection
- **In Transit**: All data transfers use TLS encryption
- **At Rest**: S3 buckets can be configured with encryption
- **Human Review**: Only necessary content is shared with reviewers

## 📈 Workflow Monitoring

Monitor these key metrics for solution health:

### CloudWatch Metrics
- **Lambda Functions**: Invocations, errors, duration
- **Step Functions**: Execution success/failure rates
- **S3 Buckets**: PutObject and GetObject operations
- **SNS Topic**: Message delivery success/failure

---

## License

**PRODUCTION DEPLOYMENT NOTICE**

This solution is intended as a reference architecture and is provided for educational and operational purposes.

**Required before production deployment:**
- Security review and penetration testing
- Compliance validation for your industry
- Performance testing under expected load
- Integration testing with existing systems
- Disaster recovery and backup procedures

**Disclaimer**: No warranty is provided, express or implied. Production use requires thorough evaluation and testing for your specific environment and security requirements.
