# ANALYZING SAMPLE APACHE SERVER LOGS 

The pipeline we will use here to analyze Apache server logs is 
```
Logstash --> Elasticsearch --> Kibana
```

The logs will be processed by Logstash through the use of grok filters and stored in an Elasticsearch index. From there, it can be accessed from Kibana to visualize and analyze the data through dashboards.

A sample Dashboard will look as follows:

![Apache Dashboard](https://github.com/arinjay97/IIC-Internship/blob/master/screenshots/Apache%20Logs%20Dashboard.png)

After setting up the ELK stack, the next step is to parse the logs and send them to Elasticsearch by using Logstash. We will create a Logstash configuration.

Using grok filters in a Logstash configuration file, Apache Logs can be filtered and documented in Elasticsearch in a way that makes it easy to visualize the data in Kibana for further analysis.

The Logstash configuration file consists of 3 sections:
1. Input - Where the log data will be taken from
2. Filter - This is where the parsing of the log to get information from them is specified
3. Output - Where the log data is to be stored after parsing

The configuration file that can be used for parsing Apache server logs and sending them to Elasticsearch is as follows:

```config
input {
  file {
    path => "<APACHE LOGS PATH>"
  }
}

filter {
  grok {
    match => { "message" => "%{COMBINEDAPACHELOG}" }
  }
  date {
    match => [ "timestamp" , "dd/MMM/yyyy:HH:mm:ss Z" ]
  }
  geoip {
      source => "clientip"
  }
}

output {
  elasticsearch {
		hosts => [<ELASTICSEARCH URL>]
		index => "<INDEX NAME>"
	}
}
```

Once this file is created, save it with a `.conf` extension. To run logstash with the above configuration run the following command in terminal
```bash
/usr/share/logstash/bin/logstash -f <PATH TO CONFIGURATION FILE>
```

The `-f` allows us to specify which configuration file to use to run Logstash.