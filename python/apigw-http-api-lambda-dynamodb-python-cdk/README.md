
# AWS API Gateway HTTP API to AWS Lambda in VPC to DynamoDB CDK Python Sample!


## Overview

Creates an [AWS Lambda](https://aws.amazon.com/lambda/) function writing to [Amazon DynamoDB](https://aws.amazon.com/dynamodb/) and invoked by [Amazon API Gateway](https://aws.amazon.com/api-gateway/) REST API. 

![architecture](docs/architecture.png)

## Setup

The `cdk.json` file tells the CDK Toolkit how to execute your app.

This project is set up like a standard Python project.  The initialization
process also creates a virtualenv within this project, stored under the `.venv`
directory.  To create the virtualenv it assumes that there is a `python3`
(or `python` for Windows) executable in your path with access to the `venv`
package. If for any reason the automatic creation of the virtualenv fails,
you can create the virtualenv manually.

To manually create a virtualenv on MacOS and Linux:

```
$ python3 -m venv .venv
```

After the init process completes and the virtualenv is created, you can use the following
step to activate your virtualenv.

```
$ source .venv/bin/activate
```

If you are a Windows platform, you would activate the virtualenv like this:

```
% .venv\Scripts\activate.bat
```

Once the virtualenv is activated, you can install the required dependencies.

```
$ pip install -r requirements.txt
```

At this point you can now synthesize the CloudFormation template for this code.

```
$ cdk synth
```

To add additional dependencies, for example other CDK libraries, just add
them to your `setup.py` file and rerun the `pip install -r requirements.txt`
command.

## Deploy
At this point you can deploy the stack. 

Using the default profile

```
$ cdk deploy
```

With specific profile

```
$ cdk deploy --profile test
```

## Security Logging Requirements

### CloudTrail Configuration
This stack requires AWS CloudTrail to be enabled for comprehensive audit logging of API activity. CloudTrail provides visibility into user activity by recording actions taken on your account.

**Organization-wide setup (recommended):**
- Configure an AWS Organizations trail to capture API activity across all accounts
- Ensure the trail logs management events and data events for DynamoDB

**Single account setup:**
```bash
aws cloudtrail create-trail --name security-audit-trail --s3-bucket-name <your-cloudtrail-bucket>
aws cloudtrail start-logging --name security-audit-trail
```

Refer to the [AWS CloudTrail documentation](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-create-and-update-a-trail.html) for detailed setup instructions.

### Logging Features Implemented
This stack implements the following logging capabilities in compliance with AWS Well-Architected Framework SEC04-BP01:

- **VPC Flow Logs**: Captures network traffic information for security analysis
- **API Gateway Access Logs**: Records all API requests with caller identity and request details
- **Lambda Function Logs**: Structured JSON logging with request context and error handling
- **CloudWatch Log Encryption**: All logs encrypted using AWS KMS with automatic key rotation
- **Log Retention**: 1-year retention policy for all log groups
- **DynamoDB Point-in-Time Recovery**: Enables audit trail and data protection

### Accessing Logs
After deployment, you can query logs using CloudWatch Logs Insights:

```bash
# Query Lambda logs
aws logs start-query --log-group-name /aws/lambda/apigw_handler \
  --start-time $(date -u -d '1 hour ago' +%s) \
  --end-time $(date -u +%s) \
  --query-string 'fields @timestamp, event, request_id | sort @timestamp desc'

# Query API Gateway access logs
aws logs start-query --log-group-name <api-log-group-name> \
  --start-time $(date -u -d '1 hour ago' +%s) \
  --end-time $(date -u +%s) \
  --query-string 'fields @timestamp, ip, status, requestTime | sort @timestamp desc'
```

## Distributed Tracing and Monitoring

### AWS X-Ray Tracing
This stack implements end-to-end distributed tracing in compliance with AWS Well-Architected Framework REL06-BP07:

- **API Gateway X-Ray Tracing**: Captures request entry points and API Gateway latency
- **Lambda X-Ray Tracing**: Traces function execution, cold starts, and initialization time
- **DynamoDB Call Tracing**: Automatically instruments boto3 DynamoDB calls using X-Ray SDK
- **Service Map**: Visualizes complete request flow from API Gateway → Lambda → DynamoDB

### Accessing X-Ray Traces
After deployment, view traces in the AWS X-Ray console:

1. **Service Map**: Navigate to AWS X-Ray console → Service map to visualize component interactions
2. **Traces**: View individual request traces showing latency breakdown across services
3. **Analytics**: Use trace analytics to identify performance bottlenecks and error patterns

```bash
# Get trace summaries for the last hour
aws xray get-trace-summaries \
  --start-time $(date -u -d '1 hour ago' +%s) \
  --end-time $(date -u +%s)
```

### CloudWatch Alarms
The stack configures the following CloudWatch alarms for proactive monitoring:

- **Lambda Error Alarm**: Triggers when Lambda function errors occur (threshold: 1 error)
- **API Gateway 5xx Alarm**: Alerts on server errors (threshold: 5 errors in 2 evaluation periods)
- **API Gateway 4xx Alarm**: Monitors client errors (threshold: 10 errors in 2 evaluation periods)

Configure SNS topics to receive alarm notifications:

```bash
# Create SNS topic for alarm notifications
aws sns create-topic --name api-monitoring-alerts

# Subscribe to the topic
aws sns subscribe --topic-arn <topic-arn> --protocol email --notification-endpoint your-email@example.com
```

Then update the CDK stack to add SNS actions to the alarms.

## After Deploy
Navigate to AWS API Gateway console and test the API with below sample data 
```json
{
    "year":"2023", 
    "title":"kkkg",
    "id":"12"
}
```

You should get below response 

```json
{"message": "Successfully inserted data!"}
```

After making requests, check the X-Ray service map to see the complete trace of your request through API Gateway, Lambda, and DynamoDB.

## Cleanup 
Run below script to delete AWS resources created by this sample stack.
```
cdk destroy
```

## Useful commands

 * `cdk ls`          list all stacks in the app
 * `cdk synth`       emits the synthesized CloudFormation template
 * `cdk deploy`      deploy this stack to your default AWS account/region
 * `cdk diff`        compare deployed stack with current state
 * `cdk docs`        open CDK documentation

Enjoy!
