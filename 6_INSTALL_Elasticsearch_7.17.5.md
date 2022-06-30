# Setting up Elasticsearch 7.17.5 on AWS
These steps are the same as for installing Elasticsearch 6, except that the package version being added to APT is **7.x**.

## Prerequisites
1. 3 EC2 instances (new instances, not the ones already running Elasticsearch 6)
   - Application and OS Images: Ubuntu Server 22.04 LTS (HVM), SSD Volume Type (Free tier eligible)
   - Instance type: At least t2.small (1vCPU, 2GiB memory)
   - Key pair (login) - only needs to be done once, not for each instance:
     - Key pair type: RSA
     - Private key file format: .ppk (for use with PuTTy)
   - Network settings:
      - Auto-assign public IP: Enable
      - Security Group:
         - Inbound rules:
            - Custom TCP - Port 9300 with the same Security Group as Source
            - Custom TCP - Port 8080, Source 0.0.0.0/0
            - SSH - Port 22, Source 0.0.0.0/0
         - Outbound rules: All traffic
2. Connect to each instance using an SSH client, e.g. PuTTy for this example
   - Open PuTTy and enter the public IPv4 address of the EC2 instance in Host Name
   - Under Category, go to Connection > SSH > Auth and under Authentication parameters, click Browse and load the .ppk file saved in step 1
   - Click Open to connect
   - Login as `ubuntu`

Once you are able to connect to all three instances, you can install Elasticsearch on them.

## Install Elasticsearch

1. Import the GPG key for Elasticsearch packages into APT
```
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```
2. Add the Elastic source list to the `source.list.d` directory where APT can search for new sources
```
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
```
3. Update the package list
```
sudo apt update
```
4. Install Elasticsearch
```
sudo apt install elasticsearch 
```
5. Create a symlink to enable Elasticsearch to automatically start on system boot
```
sudo systemctl enable elasticsearch
```

The above steps should be done for all three EC2 instances (nodes). The nodes are now ready to be configured.

## Configure Elasticsearch
1. For each EC2 instance hosting a node, edit the configuration file in the editor of your choice.
```
sudo nano /etc/elasticsearch/elasticsearch.yml 
```
2. For the master-1 node, uncomment and update the following configurations:
```
cluster.name: playground
node.name: master-1
network.host: [_local_, _site_]
discovery.seed_hosts: ["`private IP master-1`"]
cluster.initial_master_nodes: ["master-1"]
```
   - Add the following configurations:
```
node.master: true
node.data: false
node.ingest: true
node.ml: false
```
   - Save and exit the editor.

3. For the data-1 node, uncomment and update the following configurations:
```
cluster.name: playground
node.name: data-1
node.attr.temp: hot
network.host: [_local_, _site_]
discovery.seed_hosts: ["`private IP master-1`"]
cluster.initial_master_nodes: ["master-1"]
```
   - Add the following configurations:
```
node.master: false
node.data: true
node.ingest: false
node.ml: false
```
   - Save and exit the editor.

4. For the data-2 node, uncomment and update the following configurations:
```
cluster.name: playground
node.name: data-2
node.attr.temp: warm
network.host: [_local_, _site_]
discovery.seed_hosts: ["`private IP master-1`"]
cluster.initial_master_nodes: ["master-1"]
```
   - Add the following configurations:
```
node.master: false
node.data: true
node.ingest: false
node.ml: false
```
   - Save and exit the editor.

5. Configure the JVM heap on the master-1 node to free up more system memory for Kibana
```
sudo nano /etc/elasticsearch/jvm.options
```
6. Update the initial and max heap size to 768m (default is 1GiB, e.g. `-Xms1g` `-Xmx1g`) if you chose instance type t2.small (2GiB memory) for master-1, else see the section on RAM in the first answer of <a href="https://stackoverflow.com/a/58656748">this Stackoverflow thread</a>
```
-Xms768m
-Xmx768m
```
   - Save and exit the editor.

7. Start Elasticsearch on all three instances (you may need to run the command as root)
```
systemctl start elasticsearch
```
