### To install on Amazon Linux 2:
##### This method uses 3 different EC2 instances each running a different service

1. Login to EC2 console and launch 3 instances with the following configurations (at minimum)
  a. For Elasticsearch: t2.micro
  b. For Kibana: t2.micro
  c. For Logstash: t3a.small (Requires more RAM than Elasticsearch and Kibana)

2. Login to the Elasticsearch instance via SSH 
  - Using terminal for Linux
  ```
   ssh -i /path/my-key-pair.pem my-instance-user-name@my-instance-public-dns-name
  ```
  - Using PUTTY for Windows
  
3. Download and install the public signing key:
```bash
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```

4. Create a file called `elasticsearch.repo` in the `/etc/yum.repos.d/` directory and add the following content to it:
```repo
[elasticsearch]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=0
autorefresh=1
type=rpm-md
```

5. The repository is ready to be used. Install Elasticsearch with the following command:
```bash
sudo yum install --enablerepo=elasticsearch elasticsearch
```

6. Configure Elasticsearch by specifying the private IPv4 DNS of the EC2 instance as the network host in `/etc/elasticsearch/elasticsearch.yml`
```yml
# ---------------------------------- Network -----------------------------------
#
# Set the bind address to a specific IP (IPv4 or IPv6):
#
network.host: <Private IPv4 DNS>
#
# Set a custom port for HTTP:
#
http.port: 9200
#
```

7. Start the Elasticsearch service with the following command.
```bash
sudo systemctl start elasticsearch.service
```
To verify if the service is running properly, navigate to `<Public IPv4 address>:9200`

8. Login to the Kibana instance via SSH
 - Using terminal for Linux
  ```
   ssh -i /path/my-key-pair.pem my-instance-user-name@my-instance-public-dns-name
  ```
  - Using PUTTY for Windows
  
9. Download and install the public signing key:
```bash
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```

10. Create a file called `kibana.repo` in the `/etc/yum.repos.d/` directory and add the following content to it:
```repo
[kibana-7.x]
name=Kibana repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

11. The repository is ready to be used. Install Kibana with the following command:
```bash
sudo yum install kibana
```

12. Configure Kibana by specifying the public IPv4 DNS of the Elasticsearch EC2 instance as the network host in `/etc/kibana/kibana.yml`
```yaml
# The URLs of the Elasticsearch instances to use for all your queries.
elasticsearch.hosts: ["http://<Public IPv4 Address of Elasticsearch EC2 Instance>:9200"]
```

13. Navigate to `<Public IPv4 of Kibana EC2 Instance>:5601` on a browser to see the Kibana UI.

14. Login to the Logstash instance via SSH
 - Using terminal for Linux
  ```
   ssh -i /path/my-key-pair.pem my-instance-user-name@my-instance-public-dns-name
  ```
  - Using PUTTY for Windows

15. Download and install the public signing key:
```bash
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```

16. Create a file called `logstash.repo` in the `/etc/yum.repos.d/` directory and add the following content to it:
```repo
[logstash-7.x]
name=Elastic repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

17. The repository is ready to be used. Logstash requires Java to function. Install java and logstash with the following commands:
```bash
sudo yum install java
sudo yum install logstash
```

18. Configure Logstash by specifying the `http.host` as the Public IPv4 address of the Logstash EC2 Instance
```yaml
# ------------ HTTP API Settings -------------
# Define settings related to the HTTP API here.
#
# The HTTP API is enabled by default. It can be disabled, but features that rely
# on it will not work as intended.
http.enabled: true
#
# By default, the HTTP API is bound to only the host's local loopback interface,
# ensuring that it is not accessible to the rest of the network. Because the API
# includes neither authentication nor authorization and has not been hardened or
# tested for use as a publicly-reachable API, binding to publicly accessible IPs
# should be avoided where possible.
#
http.host: <Public IPv4 of Logstash EC2 Instance>
#
# The HTTP API web server will listen on an available port from the given range.
# Values can be specified as a single port (e.g., `9600`), or an inclusive range
# of ports (e.g., `9600-9700`).
#
http.port: 9600-9700
#
```

#### The ELK Stack has been fully deployed. Logs can be sent from Logstash to Elasticsearch and then be visualized and analyzed in Kibana
