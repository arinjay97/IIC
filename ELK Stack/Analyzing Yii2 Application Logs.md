# Analyzing Yii2 Application Logs

First we need to setup the yii2 application so that it can generate logs we can store in Elasticsearch and analyze and visualize with Kibana.

The setup is not that hard and just follows a few steps. Run the following commands in order to install it.

```bash
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
cd /var/www/html
composer create-project --prefer-dist yiisoft/yii2-app-basic basic
```

Once it is installed, visit `localhost/basic/web` to generate logs for the application. The logs generated are stored at `./basic/runtime/logs/app.log`.

For more information about Yii2 Logging, visit [Yii2 Documentation](https://www.yiiframework.com/doc/guide/2.0/en/runtime-logging)

As there are 3 different log levels inside the file, we will have to create a custom grok filter to parse and extract required fields.
The supported grok patterns can be seen at [Grok patterns](https://github.com/arinjay97/IIC-Internship/blob/master/ELK%20Stack/Grok%20Patterns)

The pipeline being used here is the same as the one for Apache logs,

`Logstash -> Elasticsearch -> Kibana`

First we need to create the Logstash configuration file. There are 
1. Warning Logs
2. Error Logs
3. Information Logs

Each of them have a different format with different fields, but with grok filters we can extract every field by defining different filters for different log level. The information logs are multiline so we need to define the pattern that starts every log message. 

The following configuration file was used by me to parse and store Yii2 application logs.

```
input {
  file {
    path => "<PATH TO LOG FILE>"
    start_position => "beginning"
    sincedb_path => "/dev/null" 
    codec => multiline {
      pattern => "^%{TIMESTAMP_ISO8601}"
      negate => true
      what => "previous"
    }
  }
}

filter {
  grok {
    match => [ "message", "%{TIMESTAMP_ISO8601:timestamp} \[%{IP:client_ip}\]\[%{USER:client_id}\]\[%{USER:session_id}\]\[%{LOGLEVEL:loglevel}\]%{GREEDYDATA:filtered_msg}" ]
  }
  if [loglevel] == 'warning'{
    grok{
      match => ["filtered_msg", "\[%{GREEDYDATA:category}\] %{GREEDYDATA:warning_message}"]
    }
  }
  if [loglevel] == "error"{
    grok{
      match => ["filtered_msg", "\[%{GREEDYDATA:category}:%{NUMBER:error_code}\] %{NOTSPACE:exception}: %{GREEDYDATA:error_message}" ]
    }
  }
  if [loglevel] == 'info'{
    grok{
      match => ["filtered_msg", "\[%{GREEDYDATA:category}\] %{GREEDYDATA:info}"]
    }
    kv{
      source => "info"
      field_split => "$"
    }
    kv{
      source => "_GET "
      field_split_pattern => "=/>"
    }
    kv{
      source => "_POST "
      field_split_pattern => "=/>"
    }
    kv{
      source => "_FILES "
      field_split_pattern => "=/>"
    }
    kv{
      source => "_COOKIE "
      field_split_pattern => "=/>"
    }
    kv{
      source => "_SESSION "
      field_split_pattern => "=/>"
    }
    kv{
      source => "_SERVER "
      field_split_pattern => "=/>"
    }
  }
  mutate{
    remove_field => ["message", "filtered_msg", "info", "_GET ", "_POST ", "_FILES ", "_COOKIE ", "_SESSION ", "_SERVER "]
  }
}

output { 
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "yii2"
  }
}
```

Once this file is created, save it with a .conf extension. To run logstash with the above configuration run the following command in terminal

`/usr/share/logstash/bin/logstash -f <PATH TO CONFIGURATION FILE>`

After Logstash adds all log entries to Elasticsearch, the index is created and visible in Kibana. To view the logs and create visualizations we need an index pattern in Kibana. Once that is created the logs can be viewed in the `Kibana -> Discover` section. Using the available fields section, we can filter the logs based on their value.

For example, using the loglevel field, we can filter logs to see only those which are either Warning or either Info. The document will be visible as follows

![Info Logs Discover](https://github.com/arinjay97/IIC-Internship/blob/master/screenshots/Info%20Yii2.png)

![Warning Logs Discover](https://github.com/arinjay97/IIC-Internship/blob/master/screenshots/Warning%20Yii2.png)

Every field from the info and warning logs have been parsed and stored seperately. Kibana Query Language (KQL) can be used to query the documents stored in the index pattern.

For example, if the query `error_code.keyword : "400"` is run, it returns all the documents that have the error_code fields value as 400. 
As the error_code field only exists in loglevel error, only the error log entries with code 400 are returned as the result to the query as seen below. Each document can be expanded to see the entire log error message.

![Error KQL](https://github.com/arinjay97/IIC-Internship/blob/master/screenshots/Error%20KQL.png)

To create visualizations, head to the `Kibana -> Visualize` section

To create a visualization, select the type of visualization to create. To create a Pie Chart, select the Pie Chart option. The easiest way to get a visualization is to follow the steps below:
1. Select **Split Slices** in the Buckets section.
2. Select **Terms** in Aggregation. This will allow us to select a field to visualize later.
3. Select **loglevel.keyword** in Field and press on Update.

This will create the following visualization

![Loglevel Visualization](https://github.com/arinjay97/IIC-Internship/blob/master/screenshots/Pie Chart Visualize.png)
