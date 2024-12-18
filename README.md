# EventBridge + Lambda AWS SAM Application

This project demonstrates how to create a serverless application using AWS SAM. It includes an AWS Lambda function triggered by an EventBridge rule. The Lambda function logs incoming events to AWS CloudWatch Logs.

---

## Prerequisites

Before you begin, ensure you have the following tools installed:

1. **AWS SAM CLI**: [Install AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html)
2. **AWS CLI**: [Install AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) and configure it with your AWS credentials:

   ```bash
   aws configure
   ```

3. **Python**: Ensure Python 3.13 or a compatible runtime is installed. Add it to your PATH if not already.
   
4. **Docker (Optional)**: For container-based builds, install [Docker Desktop](https://www.docker.com/products/docker-desktop).

---

## Application Architecture

- **Event Source**: Amazon EventBridge listens for events matching a specific pattern.
- **Lambda Function**: A Lambda function is triggered by EventBridge and logs the event details to CloudWatch Logs.
- **CloudWatch Logs**: Captures and stores the logs from the Lambda function.

---

## Project Structure

```
.
├── hello_world
│   ├── app.py             # Lambda function code
│   └── __init__.py        # (Optional) Module initialization
├── template.yaml          # AWS SAM template
└── README.md              # This file
```

---

## Setup Instructions

### 1. Clone the Repository

```bash
git clone <repository-url>
cd eventbridge-lambda
```

### 2. Build the Application

```bash
sam build
```

### 3. Deploy the Application

Deploy the application to your AWS account:

```bash
sam deploy --guided
```

During deployment, you will be prompted for:

- **Stack Name**: Enter a name for your stack (e.g., `eventbridge-lambda-app`).
- **AWS Region**: Choose your preferred AWS region.
- **Other Configurations**: Accept the defaults or customize as needed.

The deployment will create an EventBridge rule, Lambda function, and necessary permissions.

### 4. Test the Application

#### Manually Trigger an Event

Send a test event to EventBridge using the AWS CLI:

```bash
aws events put-events --entries '[{
  "Source": "custom.myapp",
  "DetailType": "MyApp Event",
  "Detail": "{\"status\": \"trigger\"}"
}]'
```

#### Verify Logs

1. Go to the **CloudWatch Logs** console in the AWS Management Console.
2. Find the log group for your Lambda function (e.g., `/aws/lambda/HelloWorldFunction`).
3. View the logs to see the received event details.

---

## Cleanup

To avoid incurring charges, delete the stack:

```bash
sam delete
```

This will remove all resources created by the application.

---

## Lambda Function Code

The Lambda function logs the incoming event details:

```python
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    logger.info("Event received from EventBridge!")
    logger.info(f"Event details: {event}")
    return {
        "statusCode": 200,
        "body": "Hello, EventBridge!"
    }
```

---

## SAM Template

The `template.yaml` defines the Lambda function and EventBridge rule:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: SAM Template for EventBridge Hello World

Resources:
  HelloWorldFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: app.lambda_handler
      Runtime: python3.13
      CodeUri: hello_world/
      Policies:
        - CloudWatchLogsFullAccess
      Events:
        EventBridgeRule:
          Type: CloudWatchEvent
          Properties:
            Pattern:
              source:
                - "custom.myapp"
              detail-type:
                - "MyApp Event"
              detail:
                status:
                  - "trigger"

  EventBridgeRule:
    Type: AWS::Events::Rule
    Properties:
      Name: MyEventBridgeRule
      EventPattern:
        source:
          - "custom.myapp"
        detail-type:
          - "MyApp Event"
        detail:
          status:
            - "trigger"
      Targets:
        - Arn: !GetAtt HelloWorldFunction.Arn
          Id: "HelloWorldFunctionTarget"

  LambdaInvokePermission:
    Type: AWS::Lambda::Permission
    Properties:
      Action: lambda:InvokeFunction
      FunctionName: !GetAtt HelloWorldFunction.Arn
      Principal: events.amazonaws.com
      SourceArn: !GetAtt EventBridgeRule.Arn

Outputs:
  HelloWorldFunction:
    Description: Lambda Function ARN
    Value: !GetAtt HelloWorldFunction.Arn
```

---

## License

N/A

