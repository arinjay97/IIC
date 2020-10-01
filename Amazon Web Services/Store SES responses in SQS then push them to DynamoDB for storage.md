# Store SES responses in a SQS Queue then push them to DynamoDB periodically

The requirement of the task was to send e-mail from the **Simple Email Service (SES)** and store its notifications in a **Simple Queue Service (SQS)** queue in a, then pass it to a **DynamoDB** table periodically using a **Lambda** function.

The pipeline being created is 
```
SES -> SNS -> SQS -> AWS Lambda -> DynamoDB
```

Follow the guide [Store SES Notifications in DynamoDB table](/Amazon%20Web%20Services/Pipeline%20to%20store%20SES%20responses%20to%20DynamoDB%20Table.md)