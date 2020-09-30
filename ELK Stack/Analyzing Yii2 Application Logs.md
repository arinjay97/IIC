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

