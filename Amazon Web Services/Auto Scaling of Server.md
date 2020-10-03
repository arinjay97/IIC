# AUTO SCALING OF A SERVER

The requirements of the task are
    -Serve basic Yii2 application on Apache Web Server using an EC2 instance 
    -Configure Auto Scaling for the server whenever the load reaches 50%

**Follow [this guide](/Amazon%20Web%20Services/Launch%20an%20EC2%20instance%20and%20host%20Yii2%20Basic%20Application.md) to setup the EC2 instance and install the Yii2 basic application along with all required parameters**.

## AUTO SCALING CONFIGURATION

To create and configure the Auto Scaling group in a way such that the resources are increased when load is greater than 50%, the following steps can be followed:

1. Visit the [EC2 Console](https://ap-south-1.console.aws.amazon.com/ec2) and select **Instances** on the left hand menu.

2. Select the instance configured previously in the EC2 Console under the instances section.

3. Right click on it and select Instance Settings > Attach to Auto Scaling group.

4. Enter a name for the AutoScaling group and select Create a new Auto Scaling group then select Attach.

5. Now, to configure it select the **Auto Scaling Groups** tab in the EC2 management console. Select the name of the group you just created.

![Auto Scaling](/screenshots/Amazon%20Web%20Services/Auto%20Scaling.png)

6. In the Automatic Scaling tab select Add Policy

7. The easiest way to maintain scaling for 50% load would be the following settings, as no CloudWatch metric Alarms have to be used.

![Target Tracking](/screenshots/Amazon%20Web%20Services/Target%20Tracking.png)

8. But with other policy types there are more customizations that can be made regarding how the scaling should be done for example seen below:

![Step Scaling](/screenshots/Amazon%20Web%20Services/Step%20Scaling.png)