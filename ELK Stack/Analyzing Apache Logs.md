# ANALYZING SAMPLE APACHE SERVER LOGS 

The pipeline we will use here to analyze Apache server logs is 
```
Logstash --> Elasticsearch --> Kibana
```

The logs will be processed by Logstash through the use of grok filters and stored in an Elasticsearch index. From there, it can be accessed from Kibana to visualize and analyze the data through dashboards.

A sample Dashboard will look as follows:

![Apache Dashboard](/screenshots/Apache%20Logs%20Dashboard.png)

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

The `-f` allows us to specify which configuration file to use to run Logstash. An index will be created in Elasticsearch with the specified name

Leaving the service running will add new log entries to the Elasticsearch index in real time. 

View the index in Kibana by navigating to `Stack Management > Index Management` section. To enable visualizing and querying using Kibana, an Index Pattern has to be created for the corresponding Elasticsearch index. 

The index created by Logstash can be seen as here

![Index Management](/screenshots/Index%20Created%20by%20Logstash.png)

To be able to view in Kibana and create visualizations, we need to create an Index Pattern in Kibana for the Elasticsearch index. This can be done in the `Kibana -> Index Patterns` section. Once an index pattern is created, the fields inside of it can be seen as below:

![Index Pattern](/screenshots/Index%20created%20for%20Kibana.png)

Now the logs are ready to be viewed by us for analysis as well as being used to create visualizations, and after that a dashboard in Kibana. In the `Kibana -> Discover` section in the main menu, on selecting the apache index pattern just created, we will be able to view at the parsed and stored logs.

![Discover](/screenshots/Discover%20Apache.png) 
