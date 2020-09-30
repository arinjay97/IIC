# STORE SES RESPONSES IN DYNAMODB TABLE

The requirement of the task was to send e-mail from the Simple Email Service (SES) and store its responses in a DynamoDB table.

To achieve this, a pipeline involving various Amazon Web Services were used such as **Simple Email Service, Simple Notification Service, AWS Lambda, DynamoDB**.

The pipeline being created is 
```
SES -> SNS -> AWS Lambda -> DynamoDB
```

### Setting up SES with a SNS topic

1. Login to the [SES console](https://ap-south-1.console.aws.amazon.com/ses/home) and select `Identity Management Email Adresses`

2. Select **Verify a New Email Address** and enter an email of choice.

![SES Console]()

3. Login to the [SNS Console](https://ap-south-1.console.aws.amazon.com/sns/v3/home?region=ap-south-1#/topics) and select `Topic -> Create a new Topic`

4. Give the topic a name such as *SESNotifications* and create it.

5. Head back to the SES console and select the verified email address and select **Notifications**

6. In the **SNS Topic Configuration** select the SNS topic created earlier for all 3 types of responses and then `Save Config`
    -Bounces
    -Complaints
    -Deliveries

![SNS Config]()

SES is now connected to the SNS topic. Whenever an email is sent from the verified email address, the response notification is sent to the SNS topic.
