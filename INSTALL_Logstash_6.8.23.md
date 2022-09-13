# Setting up Logstash 6.8.23 on AWS
## Prerequisites
3-node cluster running Elasticsearch and Kibana 6.8.23 on AWS. See [Setting up Elasticsearch 6.8.23 on AWS](https://github.com/TheMightyQuynh/ELK/blob/main/1_INSTALL_Elasticsearch_6.8.23.md).

## Install Logstash
1. SSH into the EC2 instance hosting the master-1 node
2. Download and install the Public Signing Key if not already there
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```
2. Add the 6.x source list to the `source.list.d` directory where APT can search for new sources
```
echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list
```
3. Update the package list and install Logstash
```
sudo apt-get update && sudo apt-get install logstash
```
4. (Optional?) Update the initial and max heap size to 768m (default is 1GiB, e.g. -Xms1g -Xmx1g) in the `/etc/logastash/jvm.options` file
```
-Xms768m
-Xmx768m
```


## Test Logstash installation
Run the most basic Logstash pipeline to test
```
cd /usr/share/logstash
bin/logstash -e 'input { stdin { } } output { stdout {} }'
```
