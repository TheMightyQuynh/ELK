# Secure the Cluster (WIP)
The Kibana instance we have created is open to anyone and running over an unencrypted HTTP connection. To secure the connection, certificates must be created and signed using the built-in security utilities in Elasticsearch.

Must be logged in as root to run the below commands.

1. Create the certificate authority on the master-1 node
```
/usr/share/elasticsearch/bin/elasticsearch-certutil ca --out /etc/elasticsearch/ca --pass elastic_ca
```

2. Use the certificate authority to generate the node certificate for the master-1 node.

   The value for `--dns` should be the DNS of the master-1 node and the value for `--ip` should be the private IP of the master-1 node
```
/usr/share/elasticsearch/bin/elasticsearch-certutil cert --ca /etc/elasticsearch/ca --ca-pass elastic_ca --name master-1 --dns ip-10-255-255-254 --ip 10.255.255.254 --out /etc/elasticsearch/master-1 --pass elastic_master_1
```

3. Use the certificate authority to generate the node certificate for the data-1 node (run this command on master-1)

   The value for `--dns` should be the DNS of the data-1 node and the value for `--ip` should be the private IP of the data-1 node
```
/usr/share/elasticsearch/bin/elasticsearch-certutil cert --ca /etc/elasticsearch/ca --ca-pass elastic_ca --name data-1 --dns ip-10-255-255-244 --ip 10.255.255.244 --out /etc/elasticsearch/data-1 --pass elastic_data_1
```

4. Use the certificate authority to generate the node certificate for the data-2 node (run this command on master-1)

   The value for `--dns` should be the DNS of the data-2 node and the value for the `--ip` should be the private IP of the data-2 node
```
/usr/share/elasticsearch/bin/elasticsearch-certutil cert --ca /etc/elasticsearch/ca --ca-pass elastic_ca --name data-2 --dns ip-10-255-255-247 --ip 10.255.255.247 --out /etc/elasticsearch/data-2 --pass elastic_data_2
```

5. Move the data-1 and data-2 nodes to a temp directory
```
mv /etc/elasticsearch/data-1 /tmp/data-1
mv /etc/elasticsearch/data-2 /tmp/data-2
```

6. Change directory to the temp folder and change ownership of the certificates to `ubuntu`
```
cd /tmp/
chown ubuntu:ubuntu data-1
chown ubuntu:ubuntu data-2
```

7. `exit` from root and change directory to the temp folder again
8. Secure copy the data-1 certificate to the data-1 node using the private IP of data-1
```
scp data-1 10.255.255.244:/tmp
