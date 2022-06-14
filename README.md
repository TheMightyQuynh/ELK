# Setting up Elasticsearch 6.8.23 on AWS

This example is based off A Cloud Guru's <a href="https://learn.acloud.guru/course/1e3ff00e-95bf-451b-be04-44d4bce6bfba/dashboard">Elastic Search Deep Dive</a> course. In the course he is using CentOS 7 servers on the learning platform's Cloud Playground, here I am using Ubuntu 22.04 on AWS which has a few key differences.

## Prerequisites
1. 3 EC2 instances
   - Application and OS Images: Ubuntu Server 22.04 LTS (HVM), SSD Volume Type (Free tier eligible)
   - Instance type: t2.micro (Free tier eligible - 1vCPU, 1GiB memory) for master-1 node, at least t2.small (1vCPU, 2GiB memory) for data-1 and data-2 nodes
   - Key pair (login) - only needs to be done once, not for each instance:
     - Key pair type: RSA
     - Private key file format: .ppk (for use with PuTTy)
   - Network settings:
      - Auto-assign public IP: Enable
      - Security Group:
         - Inbound rules: Custom TCP - Port 9300 with the same Security Group as Source; SSH - Port 22
         - Outbound rules: All traffic
2. Connect to each instance using an SSH client, e.g. PuTTy for this example
   - Open PuTTy and enter the public IPv4 address of the EC2 instance in Host Name
   - Under Category, go to Connection > SSH > Auth and under Authentication parameters, click Browse and load the .ppk file saved in step 1
   - Click Open to connect
   - Login as `ubuntu`

Once you are able to connect to all three instances, you can install Elasticsearch on them.

## Install Elasticsearch
Elasticsearch requires OpenJDK 11 to run. Note: These steps differ from the course!

1. Install Java
```
sudo apt install openjdk-11-jdk
```
4. Import the GPG key for Elasticsearch packages into APT
```
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```
5. Add the Elastic source list to the `source.list.d` directory where APT can search for new sources
```
echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list
```
6. Update the package list
```
sudo apt update
```
7. Install Elasticsearch
```
sudo apt install elasticsearch 
```
8. Create a symlink to enable Elasticsearch to automatically start on system boot
```
sudo systemctl enable elasticsearch
```

The above steps should be done for all three EC2 instances (nodes). The nodes are now ready to be configured.

## Configure Elasticsearch
Note: Some of the configuration settings are different from those shown in the course (Elasticsearch 6.x vs 7.x). 

1. For each EC2 instance hosting a node, edit the configuration file in the editor of your choice.
```
sudo nano /etc/elasticsearch/elasticsearch.yml 
```
2. For the master-1 node, uncomment and update following configurations:
```
cluster.name: playground
node.name: master-1
network.host: [_local_, _site_]
discovery.zen.ping.unicast.hosts: ["`private IP master-1`"]
discovery.zen.minimum_master_nodes: 1
```
Note that in place of `discovery.seed_hosts` and `cluster.initial_master_nodes` as mentioned in the course (Elasticsearch 7.x and above), the configuration for Elasticsearch 6.x uses `discovery.zen.ping.unicast.hosts` and `discovery.zen.minimum_master_nodes`, respectively.

   - Add the following configurations at the bottom:
```
node.master: true
node.data: false
node.ingest: true
node.ml: false
```
   - Save and exit.

3. For the data-1 node, uncomment and update the following configurations:
```
cluster.name: playground
node.name: data-1
node.attr.temp: hot
network.host: [_local_, _site_]
discovery.zen.ping.unicast.hosts: ["`private IP master-1`"]
discovery.zen.minimum_master_nodes: 1
```
   - Add the following configurations at the bottom:
```
node.master: false
node.data: true
node.ingest: false
node.ml: false
```
   - Save and exit.

4. For the data-2 node, uncomment and update the following configurations:
```
cluster.name: playground
node.name: data-2
network.host: [_local_, _site_]
discovery.zen.ping.unicast.hosts: ["`private IP master-1`"]
discovery.zen.minimum_master_nodes: 1
```
   - Add the following configurations at the bottom
```
node.master: false
node.data: true
node.ingest: false
node.ml: false
```
   - Save and exit.

7. Configure the JVM heap on the master-1 node to free up more system memory for Kibana
```
sudo nano /etc/elasticsearch/jvm.options
```
8. Update the heap min and max size to 256m (default is 1GiB, e.g. `-Xms1g` `-Xmx1g`) if you chose instance type t2.micro (1GiB memory) for master-1, or 768m if you chose t2.small (2GiB memory) for master-1
```
-Xms256m
-Xmx256m
```
9. Start Elasticsearch on all three instances (you may need to run the command as sudo)
```
systemctl start elasticsearch
```

## Handy Commands
Start Elasticsearch:
```
systemctl start elasticsearch
```

Stop Elasticsearch:
```
systemctl stop elasticsearch
```

Restart Elasticsearch:
```
systemctl restart elasticsearch
```

Check status:
```
systemctl status elasticsearch
```

Check status:
```
curl localhost:9200
```

Check cluster configuration:
```
curl localhost:9200/_cat/nodes?v
```

Check cluster health:
```
curl localhost:9200/_cat/health?v
```
