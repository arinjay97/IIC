# MONITORING METRICS FOR EC2 INSTANCE USING CLOUDWATCH

Collectd is a popular open-source solution with plug-ins that can gather system statistics for a wide variety of applications. By combining the system metrics that the CloudWatch agent can already collect with the additional metrics from collectd, you can better monitor, analyze, and troubleshoot your systems and applications.

To monitor metrics using collectD, it needs to be installed to the instance first through the command
```bash
sudo yum install -y collectd
```

## CONFIGURING COLLECTD AND CLOUDWATCH TO MONITOR METRICS

To monitor the metrics collected by collectD using CloudWatch, the configuration file for CloudWatch needs to be updated as well. This can be achieved in 2 ways:

### METHOD A
1. Follow steps 1-4 mentioned for installing and running the CloudWatch Agent [earlier](/Amazon%20Web%20Services/Store%20Yii2%20application%20logs%20in%20Cloudwatch%20for%20analysis%20and%20visualization.md).

2. When prompted to collect metrics with collectD by the configuration agent wizard, select YES

3. When asked if any host metrics should be monitored, select YES.

4. When asked if EC2 dimensions are to be added with each metric, select YES when prompted. This will allow easier identification of the metric in the CloudWatch console.

5. Configure resolution for metrics as required by the application. There are 3 levels of configuration available:
    - Basic
    - Standard
    - Advanced
More details can be found on what exactly the different levels monitor at [collectD configuration levels](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/create-cloudwatch-agent-configuration-file-wizard.html). For all metrics that collectD can monitor, visit [Metrics Collected by CloudWatch agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/metrics-collected-by-CloudWatch-agent.html) over on AWS Documentation.

6. When completed, the config.json file will be displayed with the following “metrics” section which defines the metrics to be monitored along with their time configuration.
```json
 "metrics": {
                "append_dimensions": {
                        "AutoScalingGroupName": "${aws:AutoScalingGroupName}",
                        "ImageId": "${aws:ImageId}",
                        "InstanceId": "${aws:InstanceId}",
                        "InstanceType": "${aws:InstanceType}"
                },
                "metrics_collected": {
                        "collectd": {
                                "metrics_aggregation_interval": 60
                        },
                        "cpu": {
                                "measurement": [
                                        "cpu_usage_idle",
                                        "cpu_usage_iowait",
                                        "cpu_usage_user",
                                        "cpu_usage_system"
                                ],
                                "metrics_collection_interval": 60,
                                "resources": [
                                        "*"
                                ],
                                "totalcpu": false
                        },
                        "disk": {
                                "measurement": [
					"used_percent",
                                        "inodes_free"
                                ],
                                "metrics_collection_interval": 60,
                                "resources": [
                                        "*"
                                ]
                        },
                        "diskio": {
                                "measurement": [
                                        "io_time"
                                ],
                                "metrics_collection_interval": 60,
                                "resources": [
                                        "*"
                                ]
                        },
                        "mem": {
                                "measurement": [
                                        "mem_used_percent"
                                ],
                                "metrics_collection_interval": 60
                        },
						 "statsd": {
                                "metrics_aggregation_interval": 60,
                                "metrics_collection_interval": 30,
                                "service_address": ":8125"
                        },
                        "swap": {
                                "measurement": [
                                        "swap_used_percent"
                                ],
                                "metrics_collection_interval": 60
                        }
                }
```

7. Restart the Apache server with the command 
```bash
sudo service httpd restart
```

8. Load the Cloudwatch Agent with the new configuration file by running the command
```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json -s
```


### METHOD B

1. Follow steps 1-7 mentioned for installing and running the CloudWatch Agent [earlier](/Amazon%20Web%20Services/Store%20Yii2%20application%20logs%20in%20Cloudwatch%20for%20analysis%20and%20visualization.md).

2. Manually update the config.json file by adding the metrics section and all the required metrics to be monitored as shown in above example config file.

3. Restart the Apache server with the command 
```bash
sudo service httpd restart
```

4. Load the Cloudwatch Agent with the new configuration file by running the command
```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json -s
```


## MONITORING THE METRICS ON CLOUDWATCH

1. Login to the [Cloudwatch AWS console](https://ap-south-1.console.aws.amazon.com/cloudwatch/home?region=ap-south-1)

2. Select the Metrics option on the left hand menu to open the page as seen below to view all metrics

![Metrics](/screenshots/Amazon%20Web%20Services/Metrics.png)

3. Select CWAgent in the Custom Namespaces. Here the metrics divided into what they are monitoring. There is a group for disk metrics, a group for CPU metrics, for disk IO metrics and a group for memory and swap metrics. There may be more or less depending on the configuration done.

![CWAgent Metrics](/screenshots/Amazon%20Web%20Services/CWAgent%20Metrics.png)

4. Select the group for the metric to be monitored and select the particular metric to be monitored. It will be displayed on the graph for specified time period as seen below:

![Metrics Graph](/screenshots/Amazon%20Web%20Services/Metrics%20Graph.png)

Here the percentage of memory used by the instance is being shown as a function of time for a custom time period as a line graph.

The kind of graph that can be selected are Line, Stacked Area or Number. The line and stacked area graphs show the value of the metric over a time period, whereas the number shows the current value of the metric.


5. The graphs can be added to a Dashboard very easily to view easily at a glance as seen in this dashboard

![Insights Dashboard](/screenshots/Amazon%20Web%20Services/Insights%20Dashboard%20Metrics.png)
