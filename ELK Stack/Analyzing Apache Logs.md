# ANALYZING SAMPLE APACHE SERVER LOGS 

Using Grok filters in a Logstash configuration file, Apache Logs can be filtered and documented in Elasticsearch in a way that makes it easy to visualize the data in Kibana for further analysis.

The Logstash configuration file consists of 3 sections:
1. Input - Where the log data will be taken from
2. Filter - This is where the parsing of the log to get information from them is specified
3. Output - Where the log data is to be stored after parsing

k
