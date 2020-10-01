# Store SES responses in a SQS Queue then push them to DynamoDB periodically

The requirement of the task was to send e-mail from the **Simple Email Service (SES)** and store its notifications in a **Simple Queue Service (SQS)** queue in a, then pass it to a **DynamoDB** table periodically using a **Lambda** function.

The pipeline being created is 
```
SES -> SNS -> SQS -> AWS Lambda -> DynamoDB
```

Follow the guide [Store SES Notifications in DynamoDB table](/Amazon%20Web%20Services/Pipeline%20to%20store%20SES%20responses%20to%20DynamoDB%20Table.md) to set up the following:
1. Connect SES with a SNS Topic
2. Create IAM Role for Lambda Function and add permissions to write to DynamoDB Table.

## Create a Queue in SQS to store the SES notifications

1. Login to the [SQS Console](https://ap-south-1.console.aws.amazon.com/sqs/v2/home) and choose **Create Queue**

2. Enter a queue name of your choice and choose **Create Queue**

![Creating Queue](/screenshots/Amazon%20Web%20Services/Create%20Queue.png)

