# Store SES responses in a SQS Queue then push them to DynamoDB periodically

The requirement of the task was to send e-mail from the **Simple Email Service (SES)** and store its notifications in a **Simple Queue Service (SQS)** queue in a, then pass it to a **DynamoDB** table periodically using a **Lambda** function.

The pipeline being created is 
```
SES -> SNS -> SQS -> AWS Lambda -> DynamoDB 
```

Follow the guide [Store SES Notifications in DynamoDB table](/Amazon%20Web%20Services/Pipeline%20to%20store%20SES%20responses%20to%20DynamoDB%20Table.md) to connect SES with a SNS Topic

## Create IAM Role for Lambda Function and add permissions to read from SQS queue and write to DynamoDB Table.

1. 1. Login to the [DynamoDB console](https://console.aws.amazon.com/dynamodb/home) and create a new table.

2. Name the table **SESNotifications** with a primary partition key named **SESMessageId** and a primary sort key named **SnsPublishTime**. To query the table, we can set up secondary indexes as shown below.

| Index Name                    | Partition Key                     | Sort Key                |
|-------------------------------|-----------------------------------|-------------------------|
| SESMessageType-Index          | SESMessageType (String)           | SnsPublishTime (String) |
| SESMessageComplaintType-Index | SESComplaintFeedbackType (String) | SnsPublishTime (String) |

3. Login to the [IAM Console](https://console.aws.amazon.com/iam/home?) and in the navigation pane choose Roles, and then choose **Create Role**.

4. Choose the AWS service trusted entity type, choose Lambda, and then choose Next: Permissions.

![Create Role](/screenshots/Amazon%20Web%20Services/Create%20role.png)

5. Choose **AWSLambdaBasicExecutionRole, AWSLambdaSQSQueueExecutionRole and AmazonSQSFullAccess** managed policies, and then choose Next.

6. Provide a name for the role of your choice, and then complete the creation of the role.

7. Return to the IAM Roles view, and then choose the role you created.

8. Choose Add an Inline Policy.

9. In the Visual editor, choose the Service DynamoDB and the action PutItem under the Write category. Under resources, choose Add ARN and then provide the DynamoDB table ARN created earlier.

![Inline Policy](/screenshots/Amazon%20Web%20Services/Inline%20Policy.png)

10. Add another inline Policy.

11. In the Visual editor, choose the Service SQS and the action **GetQueueAttributes** and **ReceiveMessage** under the Read category. Under the Write category, choose **DeleteMessage**. Under resources, choose Add ARN and then provide the SQS queue ARN created earlier.


## Create a Queue in SQS to store the SES notifications

1. Login to the [SQS Console](https://ap-south-1.console.aws.amazon.com/sqs/v2/home) and choose **Create Queue**

2. Enter a queue name of your choice and choose **Create Queue**

![Creating Queue](/screenshots/Amazon%20Web%20Services/Create%20Queue.png)

3. Go back to the console homepage, select the queue just created and choose **Subscribe to Amazon SNS Topic**. In the drop down menu, choose the topic created previously.


## Create AWS Lambda functionto process SES notifications

1. Login to [Lambda Console](https://ap-south-1.console.aws.amazon.com/lambda/home) and choose **Create Function**.

2. Enter a name for the function and select **Python3.8** under runtime. 

3. Choose the role created earlier under **Permissions**.

![Create Function](/screenshots/Amazon%20Web%20Services/Create%20Function%20Python.png)

4. Set up a new function using the following code. This code checks for the three types of SNS notifications as described at [Amazon SNS Notification Examples for Amazon SES](https://docs.aws.amazon.com/ses/latest/DeveloperGuide/notification-examples.html) stored in the SQS queue and puts the SES notification as an entry in a DynamoDB table.

```python
import boto3
import json
import sys

sqs = boto3.client('sqs')
ddb = boto3.resource('dynamodb')

table = ddb.Table('sqs')
queue_url = '<ENTER QUEUE URL HERE>'

def lambda_handler(event, context):
    while True:
        messages = sqs.receive_message(QueueUrl=queue_url)
        if 'Messages' in messages:
            for message in messages['Messages']:
                body = json.loads(message['Body'])
                receiptHandle = message['ReceiptHandle']
                
                SESMessageType = json.loads(body['Message'])
                SnsPublishTime = body['Timestamp']
                SnsTopicArn = body['TopicArn']
                SESMessageId = SESMessageType['mail']['messageId']
                SESDestinationAddress = json.dumps(SESMessageType['mail']['destination'])
            
                if SESMessageType['notificationType'] == 'Delivery':
                    SESsmtpResponse = SESMessageType['delivery']['smtpResponse']
                    SESreportingMTA = SESMessageType['delivery']['reportingMTA']
                    item = {
                        'SESMessageId' : SESMessageId, 
			            'SnsPublishTime': SnsPublishTime, 
			            'SESsmtpResponse': SESsmtpResponse,
			            'SESreportingMTA': SESreportingMTA,
			            'SESDestinationAddress': SESDestinationAddress, 
			            'SESMessageType': SESMessageType['notificationType'],
                    }
                    table.put_item(TableName = 'sqs', Item = item)
                    print(f"{SESMessageId} put in table")
                    sqs.delete_message(QueueUrl=queue_url, ReceiptHandle=receiptHandle)
                    print(f"Deleted {SESMessageId} from queue")
                    
                if SESMessageType['notificationType'] == 'Bounce':
                    SESbounceSummary = json.dumps(SESMessageType['bounce']['bouncedRecipients'])
                    SESreportingMTA = SESMessageType['bounce']['reportingMTA']
                    item = {
                        'SESMessageId' : SESMessageId, 
			            'SnsPublishTime': SnsPublishTime,
			            'SESreportingMTA': SESreportingMTA,
			            'SESbounceSummary': SESbounceSummary, 
			            'SESMessageType': SESMessageType['notificationType'],
                   }
                    table.put_item(TableName = 'sqs', Item = item)
                    print(f"{SESMessageId} put in table")
                    sqs.delete_message(QueueUrl=queue_url, ReceiptHandle=receiptHandle)
                    print(f"Deleted {SESMessageId} from queue")
                
                if SESMessageType['notificationType'] == 'Complaint':
                    SESComplaintFeedbackType = SESMessageType['complaint']['complaintFeedbackType']
                    SESFeedbackId = SESMessageType['complaint']['feedbackId']
                    item = {
                        'SESMessageId' : SESMessageId, 
			            'SnsPublishTime': SnsPublishTime, 
			            'SESComplaintFeedbackType': SESComplaintFeedbackType,
			            'SESFeedbackId': SESFeedbackId,
			            'SESDestinationAddress': SESDestinationAddress, 
			            'SESMessageType': SESMessageType['notificationType'],
                   }
                    table.put_item(TableName = 'sqs', Item = item)
                    print(f"{SESMessageId} put in table")
                    sqs.delete_message(QueueUrl=queue_url, ReceiptHandle=receiptHandle)
                    print(f"Deleted {SESMessageId} from queue")
    
        else:
            print('No messages in queue')
            break
```


## Configuring the Lambda function to consume every notification in the queue after an interval of every 5 minutes

1. Login to the [CloudWatch console](https://ap-south-1.console.aws.amazon.com/cloudwatch/home) and choose `Events -> Rules` and then **Create Rule**

2. Configure the rule as follows
    - Under **Event Source** choose **Schedule** and in the **Fixed rate of** enter the value as 5 minutes.
    - Under **Targets** choose **Lambda Function** and select the corresponding Lambda function created earlier

![Create Rule](/screenshots/Amazon%20Web%20Services/Create%20Event%20Rule.png)

3. Choose **Configure Details**, enter a name for the rule when prompted and choose Create Rule.

## Testing and viewing data in DynamoDB Table

1. Login to the [SES console](https://ap-south-1.console.aws.amazon.com/ses/home) and select `Identity Management Email Adresses`

2. Select a verified email and choose **Send a Test Email**. For more information about sending test emails, the [AWS Documentation](https://docs.aws.amazon.com/ses/latest/DeveloperGuide/send-email-simulator.html) can be referred

3. Login to the [SQS Console](https://ap-south-1.console.aws.amazon.com/sqs/v2/home) and select the queue previously created.

4. Choose **Send and receive messages** and in the next page select **Poll for messages**. This will show the notifications from SES that are currently stored in the queue. After 5 minutes, they will be processed and stored in the DynamoDB table and then be deleted from the queue.

![Poll for Messages](/screenshots/Amazon%20Web%20Services/Poll%20for%20Messages.png)

3. Login to the [DynamoDB console](https://console.aws.amazon.com/dynamodb/home) and choose **Tables**.

4. Select the table created earlier and choose **Items**. All the emails that have been sent and processed by the Lambda function can be seen in the table now as seen below

![DynamoDB Table](/screenshots/Amazon%20Web%20Services/DynamoDB%20Table.png)
