# Setting up Kibana 6.8.23 on AWS
This example is based off A Cloud Guru's <a href="https://learn.acloud.guru/course/1e3ff00e-95bf-451b-be04-44d4bce6bfba/dashboard">Elastic Search Deep Dive</a> course. In the course he is using CentOS 7 servers on the learning platform's Cloud Playground, here I am using Ubuntu 22.04 on AWS.

## Prerequisites
This example assumes you have already set up a three-node cluster on AWS EC2 instances as per the Elasticsearch installation guide and thus have already downloaded and imported the Elasticsearch GPG key.

## Install and configure Kibana
1. SSH into the EC2 instance hosting the master-1 node
2. Add the Kibana source list to the `source.list.d` directory where APT can search for new sources
```
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-6.x.list
```
3. Update the package list
```
sudo apt update
```
4. Install Kibana
```
sudo apt install kibana
```
5. Configure kibana.yml to bind with the appropriate IP address and port
```
sudo nano /etc/kibana/kibana.yml
```
6. Uncomment and update the following configurations:
```
server.port: 8080
server.host: "`private IP master-1`"
```
7. Start Kibana
```
systemctl start kibana
```
8. (Optional) Confirm Kibana is running on the correct IP and port
```
less /var/log/syslog
```
Successful log output:
>Jun 14 07:54:42 ip-10-255-255-254 kibana[1018]: {"type":"log","@timestamp":"2022-06-14T07:54:42Z","tags":["listening","info"],"pid":1018,"message":"Server running at http://10.255.255.254:8080"}
>
>Jun 14 07:54:42 ip-10-255-255-254 kibana[1018]: {"type":"log","@timestamp":"2022-06-14T07:54:42Z","tags":["status","plugin:spaces@6.8.23","info"],"pid":1018,"state":"green","message":"Status changed from yellow to green - Ready","prevState":"yellow","prevMsg":"Waiting for Elasticsearch"}


9. Enter the public IP address of the master-1 node into a web browser using HTTP port 8080, example:
```
http://12.34.56.78:8080
```
You should now see this screen:
![image](https://user-images.githubusercontent.com/104564793/173527664-e34c63c0-808e-41d5-acbe-fbe13048ef70.png)
