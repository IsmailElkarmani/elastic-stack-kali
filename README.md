
# Installation and Troubleshooting of Elastic Stack on Kali Linux  

This project documents the installation and configuration of Elastic Stack (Elasticsearch, Logstash, Kibana, Beats) on Kali Linux. It also includes a troubleshooting guide for common errors.  

---

## Table of Contents  
1. [Introduction](#introduction)  
2. [Prerequisites](#prerequisites)  
3. [Installation](#installation)  
   - [Adding the Elastic Repository](#adding-the-elastic-repository)  
   - [Installing Elasticsearch](#installing-elasticsearch)  
   - [Installing Kibana](#installing-kibana)  
   - [Installing Logstash](#installing-logstash)  
   - [Installing Filebeat](#installing-filebeat)  
4. [Verification](#verification)  
5. [Troubleshooting](#troubleshooting)  
6. [Automation](#automation)  
7. [License](#license)  
8. [Author](#author)  

---

## Introduction

Elastic Stack (formerly ELK Stack) is an open-source toolset used for real-time data collection, processing, and visualization. It consists of:  
- **Elasticsearch**: A search and analytics engine  
- **Logstash**: A data processing and ingestion pipeline  
- **Kibana**: A visualization interface for data  
- **Beats**: Lightweight data shippers  

---

## Prerequisites  

Before starting, ensure that you have:  
- **Kali Linux** installed and up to date  
- **Root or sudo access**  
- **Java 11** installed:  
  ```bash
  sudo apt install -y openjdk-11-jdk
  java -version  # Verify installation
  ```
## Installation
#### Adding the Elastic Repository
```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt update
```
#### Installing Elasticsearch
```bash
sudo apt install -y elasticsearch
```
Configuration in /etc/elasticsearch/elasticsearch.yml:
```bash
network.host: localhost
xpack.security.enabled: true
xpack.security.http.ssl.enabled: false
```
Start and enable the service:
```bash
sudo systemctl enable --now elasticsearch
```
Verification:
```bash
curl -X GET "http://localhost:9200"
```
If you get a 401 (missing authentication credentials) error, use this command:
```bash
curl -u elastic:"your_password" -X GET "http://localhost:9200"
```
To retrieve the default Elasticsearch password:
```bash
sudo cat /etc/elasticsearch/elasticsearch.keystore
```
If the password is unknown, reset it:
```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
```
#### Installing Kibana
```bash
sudo apt install -y kibana
```
Configuration in /etc/kibana/kibana.yml:
```bash
server.host: "localhost"
elasticsearch.hosts: ["http://localhost:9200"]
```
Start and enable the service:
```bash
sudo systemctl enable --now kibana
```
Access Kibana via:
http://localhost:5601

If Kibana is not working, check logs:
```bash
sudo journalctl -u kibana --no-pager | tail -50
```
#### Installing Logstash
```bash
sudo apt install -y logstash
```
Example configuration /etc/logstash/conf.d/logstash.conf:
```bash
input {
  beats {
    port => 5044
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "logstash-%{+YYYY.MM.dd}"
  }
  stdout { codec => rubydebug }
}
```
Start and enable the service:
```bash
sudo systemctl enable --now logstash
```
Verify:
```bash
sudo systemctl status logstash
```
#### Installing Filebeat
```bash
sudo apt install -y filebeat
```
Enable and start the service:
```bash
sudo systemctl enable --now filebeat
```
## Verification
```bash
#Elasticsearch:
curl -X GET "http://localhost:9200"
```
```bash
#Kibana:
Access via http://localhost:5601
```
```bash
#Logstash:
Check status:
sudo systemctl status logstash
```
## Troubleshooting 

| Problem | Solution |
|---------|----------|
| curl: (52) Empty reply from server | Check if Elasticsearch is running: `sudo systemctl start elasticsearch` |
| missing authentication credentials (401) | Use: `curl -u elastic:your_password -X GET "http://localhost:9200"` |
| received plaintext HTTP traffic on an HTTPS channel | Disable HTTPS in `elasticsearch.yml` if necessary |
| unable to authenticate user [elastic] | Reset password: `sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic` |
| Kibana not working | Check logs: `sudo journalctl -u kibana --no-pager` |


## Automation

Start all services

Create a script start_services.sh:
```bash
#!/bin/bash
sudo systemctl start elasticsearch kibana logstash filebeat
echo "All Elastic Stack services have been started!"
```
Run it:
```bash
bash start_services.sh
```
## License

This project is licensed under the MIT License.

## Author
[GitHub](https://github.com/IsmailElkarmani)


---
## ⭐ If you find this project helpful, don’t forget to give it a star! ⭐
