# USING INSIGHTS TO QUERY LOGS AND CREATE A DASHBOARD

Insights Query can be used to parse log messages and gain information from them. 

For example, to filter only “error” level log messages, the following query can be used to get the results as shown
```sql
filter @message like 'error'
```

![Insights Error](https://github.com/arinjay97/IIC-Internship/blob/master/screenshots/Insights%20-%20error.jpg)
**We can similarly filter out different log levels by replacing error with info or warning.**

In order to parse the log message and gain custom fields we can use glob expressions along with the query syntax
parse @message

For example to get IP from the message 

2020-08-18 10:39:09 [92.118.160.1][-][-][warning][yii\debug\Module::checkAccess] Access to debugger is denied due to IP address restriction. The requesting IP address is 92.118.160.1

We can use the query
```sql
 filter @message like 'warning'
| parse @message "[*]" as ip
```

As the first value in square brackets is the ip, it will be parsed and another field be made by the name of ip containing the value as seen below

![Insights IP](https://github.com/arinjay97/IIC-Internship/blob/master/screenshots/Insights%20-%20ip.jpg)

To get the count of a field and sort it and get the top 10 values with highest coun, use **stats count(\*)** query
For example to get the top 10 User Agents from info logs use the query as follows 

```sql
fields @message
|filter @message like /info/
|filter @message like 'HTTP_USER_AGENT'
|parse @message "'HTTP_USER_AGENT' => '*'" as user
|stats count(*) as Count by user
|sort Count desc
|limit 10
```

The query results in this

![Top 10 User Agents](https://github.com/arinjay97/IIC-Internship/blob/master/screenshots/Insights%20-%20Top%2010%20user%20agents.jpg)

To Visualize warning logs as a function of time we can use the **stats count by bin**. This will plot the timestamps where warning logs were ingested to CloudWatch on a time interval specified.

```sql
fields @message
   | filter @message like /warning/
   | stats count(*) as Count by bin(12h)
```

 The above query will result in the following graph:
 
 ![Insights Graph](https://github.com/arinjay97/IIC-Internship/blob/master/screenshots/Insights%20-%20Graph.jpg)
 
 
### DASHBOARDS
 
Dashboards are customizable home pages where information can be viewed by just a glance, we can add any metric, custom query or visualization in this and view the results on any particular time field. Example of a dashboard created for Yii2 Basic Application can be seen below

![Insights Dashboard](https://github.com/arinjay97/IIC-Internship/blob/master/screenshots/Insights%20Dashboard.png)
