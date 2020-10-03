# INSTALLATION STEPS

### To install locally on a Debian System:
1. Download and install the public signing key
```bash
$ wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```
2. Install the apt-transport-https package 
```bash
$ sudo apt-get install apt-transport-https
```
3. Save the repository definition
```bash
$ echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list
```
4. Install the Elasticsearch Debian package with 
```bash
$ sudo apt-get update && sudo apt-get install elasticsearch
```
5. Install the Kibana Debian package with
```bash
$ sudo apt-get update && sudo apt-get install kibana
```
6. Install the Logstash Debian package with
```bash
$ sudo apt-get update && sudo apt-get install logstash
```
7. It is not required to change any of the configuration files for Elasticsearch or Kibana in order to host it locally on the system.

8. Start the Elasticsearch and Kibana services and navigate to the browser to view the Kibana console. The following commands can be used to start the service:
```bash
$ sudo systemctl start elasticsearch
$ sudo systemctl start kibana
```
9. Navigate to [local Elasticsearch instance](localhost:9200) on a browser to check if Elasticsearch is working or not. If this is visible, then the service is running properly.

![ES in browser](/screenshots/ELK/ES.jpg)

10. Navigate to [Kibana on browser](localhost:5601) on a browser to access the Console and start with analysis and visualizations

![Kibana in browser](/screenshots/ELK/Kibana.jpg)
