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

![SES Console](/screenshots/Amazon%20Web%20Services/SES%20console.png)

3. Login to the [SNS Console](https://ap-south-1.console.aws.amazon.com/sns/v3/home?region=ap-south-1#/topics) and select `Topic -> Create a new Topic`

4. Give the topic a name such as *SESNotifications* and create it.

5. Head back to the SES console and select the verified email address and select **Notifications**

6. In the **SNS Topic Configuration** select the SNS topic created earlier for all 3 types of responses and then `Save Config`
    -Bounces
    -Complaints
    -Deliveries

![SNS Config](/screenshots/Amazon%20Web%20Services/SNS%20config.png)

SES is now connected to the SNS topic. Whenever an email is sent from the verified email address, the response notification is sent to the SNS topic.


## Create IAM role for Lambda function and add permissions to write to DynamoDB Table

1. Login to the [DynamoDB console](https://console.aws.amazon.com/dynamodb/home) and create a new table.

2. Name the table **SESNotifications** with a primary partition key named **SESMessageId** and a primary sort key named **SnsPublishTime**. To query the table, we can set up secondary indexes as shown below.

| Index Name                    | Partition Key                     | Sort Key                |
|-------------------------------|-----------------------------------|-------------------------|
| SESMessageType-Index          | SESMessageType (String)           | SnsPublishTime (String) |
| SESMessageComplaintType-Index | SESComplaintFeedbackType (String) | SnsPublishTime (String) |

3. Login to the [IAM Console](https://console.aws.amazon.com/iam/home?) and in the navigation pane choose Roles, and then choose **Create Role**.

4.Choose the AWS service trusted entity type, choose Lambda, and then choose Next: Permissions.

![Create Role](/screenshots/Amazon%20Web%20Services/Create%20role.png)

5.Choose the **AWSLambdaBasicExecutionRole** managed policy, and then choose Next.

6. Provide a name for the role of your choice, and then complete the creation of the role.

7. Return to the IAM Roles view, and then choose the role you created.

8. Choose Add an Inline Policy.

9. In the Visual editor, choose the Service DynamoDB and the action PutItem under the Write category. Under resources, choose Add ARN and then provide the DynamoDB table ARN created earlier.

![Inline Policy](/screenshots/Amazon%20Web%20Services/Inline%20Policy.png)

10. Choose Review, provide a policy name, and then choose Create policy.


## Create AWS Lambda functionto process SES notifications

1. Login to [Lambda Console](https://ap-south-1.console.aws.amazon.com/lambda/home) and choose **Create Function**.

2. Enter a name for the function and select **Node.js** under runtime. 

3. Choose the role created earlier under **Permissions**.

![Create Function](/screenshots/Amazon%20Web%20Services/Create%20Function.png)

4. Set up a new function using the following code. This code checks for the three types of SNS notifications as described at [Amazon SNS Notification Examples for Amazon SES](https://docs.aws.amazon.com/ses/latest/DeveloperGuide/notification-examples.html), and puts the SES notification into an entry in a DynamoDB table.

```node
console.log('Loading event');
var aws = require('aws-sdk');
var ddb = new aws.DynamoDB({params: {TableName: '<TABLE NAME HERE>'}});
exports.handler = function(event, context)
{
  console.log('Received event:', JSON.stringify(event, null, 2));
  var SnsPublishTime = event.Records[0].Sns.Timestamp;
  var SnsTopicArn = event.Records[0].Sns.TopicArn;
  var SESMessage = event.Records[0].Sns.Message;
  SESMessage = JSON.parse(SESMessage);
  var SESMessageType = SESMessage.notificationType;
  var SESMessageId = SESMessage.mail.messageId;
  var SESDestinationAddress = SESMessage.mail.destination.toString();
  var LambdaReceiveTime = new Date().toString();
  if (SESMessageType == 'Bounce')
  {
  var SESreportingMTA = SESMessage.bounce.reportingMTA;
  var SESbounceSummary = JSON.stringify(SESMessage.bounce.bouncedRecipients);
  var itemParams = {Item: {SESMessageId: {S: SESMessageId}, SnsPublishTime: {S: SnsPublishTime},
  SESreportingMTA: {S: SESreportingMTA}, SESDestinationAddress: {S: SESDestinationAddress}, SESbounceSummary: {S: SESbounceSummary},
  SESMessageType: {S: SESMessageType}}};
ddb.putItem(itemParams, function(err, data)
{
  if(err) { context.fail(err)}
  else {
           console.log(data);
           context.succeed();
      }
  });
}
else if (SESMessageType == 'Delivery')
{
  var SESsmtpResponse1 = SESMessage.delivery.smtpResponse;
  var SESreportingMTA1 = SESMessage.delivery.reportingMTA;
  var itemParamsdel = {Item: {SESMessageId: {S: SESMessageId}, SnsPublishTime: {S: SnsPublishTime}, SESsmtpResponse: {S: SESsmtpResponse1},
  SESreportingMTA: {S: SESreportingMTA1},
  SESDestinationAddress: {S: SESDestinationAddress }, SESMessageType: {S: SESMessageType}}};
  ddb.putItem(itemParamsdel, function(err, data)
{
  if(err) { context.fail(err)}
  else {
          console.log(data);
          context.succeed();
      }
  });
}
else if (SESMessageType == 'Complaint')
{
var SESComplaintFeedbackType = SESMessage.complaint.complaintFeedbackType;
var SESFeedbackId = SESMessage.complaint.feedbackId;
var itemParamscomp = {Item: {SESMessageId: {S: SESMessageId}, SnsPublishTime: {S: SnsPublishTime}, SESComplaintFeedbackType: {S: SESComplaintFeedbackType},
SESFeedbackId: {S: SESFeedbackId},
SESDestinationAddress: {S: SESDestinationAddress }, SESMessageType: {S: SESMessageType}}};
ddb.putItem(itemParamscomp, function(err, data)
{
  if(err) { context.fail(err)}
  else {
          console.log(data);
          context.succeed();
      }
  });
}
};
```

5. Subscribe the function to the SNS topic created earlier by logging into the Lambda console for the function, add the SNS trigger from the triggers list on the Designer. A configuration panel appears below the designer.

6. Choose the topic from the drop-down list in the configuration panel.

![SNS Topic Lambda Subscribe](/screenshots/Amazon%20Web%20Services/Lambda%20SNS%20Connection.png)


## Testing and viewing data in DynamoDB Table

1. Login to the [SES console](https://ap-south-1.console.aws.amazon.com/ses/home) and select `Identity Management Email Adresses`

2. Select a verified email and choose **Send a Test Email**. For more information about sending test emails, the [AWS Documentation](https://docs.aws.amazon.com/ses/latest/DeveloperGuide/send-email-simulator.html) can be referred

3. Once an email is sent, login to the [DynamoDB console](https://console.aws.amazon.com/dynamodb/home) and choose **Tables**.

4. Select the table created earlier and choose **Items**. All the emails that have been sent and processed by the Lambda function can be seen in the table now as seen below

![DynamoDB Table](/screenshots/Amazon%20Web%20Services/DynamoDB%20Table.png)
