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

5. Configure the Agent as required. As just the application logs are being sent, select NO for collecting metrics and stats from CollectD. When prompted to add additional log files to monitor, select YES.

6. Specify the path to the log file needed to be monitored along with the log group and log stream name for the same. In this case, the path to the log file is `/var/www/html/basic/runtime/logs/app.log`

7. Select NO when added log file and exit the wizard. The following config.json file is created at /opt/aws/amazon-cloudwatch-agent/bin/
```
{
        "agent": {
                "run_as_user": "cwagent",
                "region": "ap-south-1",
                "debug": true
        },
        "logs": {
                "logs_collected": {
                        "files": {
                                "collect_list": [
                                        {
                                                "file_path": "/var/www/html/basic/runtime/logs/app.log",
                                                "log_group_name": "yii2/multiline",
                                                "log_stream_name": "{instance_id}",
                                                "multi_line_start_pattern": "[0-9]{4}-(0[1-9]|1[0-2])-(0[1-9]|[1-2][0-9]|3[0-1]) (2[0-3]|[01][0-9]):[0-5][0-9]"
                                        }
                                ]
                        }
                }
        }
}
```
The multiline start pattern 

8. Start the agent and upload the files to cloudwatch with the following command
```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json -s
```

9. This will start the agent and create the log stream and log group to subsequently store the Yii2 application logs. This can be viewed at the [Cloudwatch Management Console](https://ap-south-1.console.aws.amazon.com/cloudwatch/home)

The Log Group can be viewed by selecting it in the side menu
![Log Groups](https://github.com/arinjay97/IIC-Internship/blob/master/screenshots/Log%20Groups.jpg)

The Log Stream can be selected
![Log Streams](https://github.com/arinjay97/IIC-Internship/blob/master/screenshots/Log%20Streams.jpg)

Inside the Log Stream, all the log messages can be viewed in the Log Events section
![Log Events](https://github.com/arinjay97/IIC-Internship/blob/master/screenshots/Log%20Events.jpg)
