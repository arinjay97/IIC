# ANALYSIS AND VISUALIZATION OF LOGS IN CLOUDWATCH

An EC2 instance is required with Yii2 basic application being served by Apache for this.

### CREATE LOG GROUP AND LOG STREAM IN CLOUDWATCH
A log group and log stream have to be created in Cloudwatch. This is where the application logs will be stored.

1. Download CloudWatch agent package. The package varies for every different instance platform and the link can be found at [Cloudwatch Agent Package Download](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/download-cloudwatch-agent-commandline.html) AWS documentation.

2. AsAmazon Linux 2 AMI is being used, run the following command
```bash
sudo wget https://s3.amazonaws.com/amazoncloudwatch-agent/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm
```

3. Install the package
```bash
sudo rpm -U ./amazon-cloudwatch-agent.rpm
```

4. Run the CloudWatch Agent configuration Wizard by using the following command 
```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
```
