# Secure the Cluster (WIP)
The Kibana instance we have created is open to anyone and running over an unencrypted HTTP connection. To secure the connection, certificates must be created and signed using the built-in security utilities in Elasticsearch.

## Create the Public Key Infrastructure
Must be logged in as root to run the below commands.

1. Create the certificate authority on the **master-1** node using Elasticsearch's certificate utility
```
/usr/share/elasticsearch/bin/elasticsearch-certutil ca --out /etc/elasticsearch/ca --pass elastic_ca
```

2. Use the certificate authority to generate the node certificate for the master-1 node.

   The value for `--dns` should be the DNS of the master-1 node and the value for `--ip` should be the private IP of the master-1 node
```
/usr/share/elasticsearch/bin/elasticsearch-certutil cert --ca /etc/elasticsearch/ca --ca-pass elastic_ca --name master-1 --dns ip-10-255-255-254 --ip 10.255.255.254 --out /etc/elasticsearch/master-1 --pass elastic_master_1
```

3. Use the certificate authority to generate the node certificate for the data-1 node (run this command on **master-1** node)

   The value for `--dns` should be the DNS of the data-1 node and the value for `--ip` should be the private IP of the data-1 node
```
/usr/share/elasticsearch/bin/elasticsearch-certutil cert --ca /etc/elasticsearch/ca --ca-pass elastic_ca --name data-1 --dns ip-10-255-255-244 --ip 10.255.255.244 --out /etc/elasticsearch/data-1 --pass elastic_data_1
```

4. Use the certificate authority to generate the node certificate for the data-2 node (run this command on **master-1** node)

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

7. `exit` from root and `cd /tmp` to change directory to the temp folder again

8. Open a terminal on your **local** machine and secure copy the .pem file (using that same .pem file as the private key) to the master-1 EC2 instance using the public IP of master-1
```
scp -i /local/path/to/your/keypair_file.pem keypair_file.pem ubuntu@publicipaddress:/tmp
```

9. On the **master-1** node, set permissions on the .pem file so only the owner (ubuntu) can read the file
```
chmod 400 keypair_file.pem
```

10. Secure copy the data-1 certificate to the data-1 node using the private IP of the data-1 node
```
scp -i keypair_file.pem data-1 10.255.255.244:/tmp
```

11. Secure copy the data-2 certificate to the data-2 node using the private IP of the data-2 node
```
scp -i keypair_file.pem data-2 10.255.255.247:/tmp
```

12. Remove the certificates and .pem file from the master-1 node as they are no longer needed
```
rm data-1
rm data-2
rm keypair_file.pem
```

13. On the **data-1** node (as root user), `cd /tmp` to change directory to the temp folder

14. Change ownership of the certificate to user `root` and group `elasticsearch`
```
chown root:elasticsearch data-1
```
15. Move the certificate to `/etc/elasticsearch`
```
mv data-1 /etc/elasticsearch/
```

16. On the **data-2** node (as root user), `cd /tmp` to change directory to the temp folder

17. Change ownership of the certificate to user `root` and group `elasticsearch`
```
chown root:elasticsearch data-2
```
18. Move the certificate to `etc/elasticsearch`
```
mv data-2 /etc/elasticsearch/
```

## Encrypt the Transport Network

1. On the **master-1** node in the `/etc/elasticsearch` directory (as root user), set the permissions on the `master-1` certificate to allow the elasticsearch group to be able to read it
```
chmod 640 master-1
```

2. Use Elasticsearch's keystore utility to add a keystore password
```
/usr/share/elasticsearch/bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
```
   - Enter a value for the password when prompted (`elastic_master_1` for this example)

3. Use Elasticsearch's keystore utility to add a truststore password
```
/usr/share/elasticsearch/bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
```
   - Enter a value for the password when prompted (value should be the same as for the keystore password, as it uses the same certificate; `elastic_master_1` for this example)

4. Open `/etc/elasticsearch/elasticsearch.yml` in the editor of your choice and add the below security configurations (optionally create a Security section to put these under):
```
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: full
xpack.security.transport.ssl.keystore.path: master-1
xpack.security.transport.ssl.truststore.path: master-1
```

### The next steps repeat the same process for data-1 and data-2 nodes, replacing the node-specific values where necessary.

5. On the **data-1** node in the `/etc/elasticsearch` directory (as root user), set the permissions on the `data-1` certificate to allow the elasticsearch group to be able to read it
```
chmod 640 data-1
```

6. Use Elasticsearch's keystore utility to add a keystore password
```
/usr/share/elasticsearch/bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
```
   - Enter a value for the password when prompted (`elastic_data_1`)

7. Use Elasticsearch's keystore utility to add a truststore password
```
/usr/share/elasticsearch/bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
```
   - Enter a value for the password when prompted (`elastic_data_1`)

8. Open `/etc/elasticsearch/elasticsearch.yml` in the editor of your choice and add the below security configurations (optionally create a Security section to put these under):
```
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: full
xpack.security.transport.ssl.keystore.path: data-1
xpack.security.transport.ssl.truststore.path: data-1
```

9. On the **data-2** node in the `/etc/elasticsearch` directory (as root user), set the permissions on the `data-2` certificate to allow the elasticsearch group to be able to read it
```
chmod 640 data-2
```

10. Use Elasticsearch's keystore utility to add a keystore password
```
/usr/share/elasticsearch/bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
```
   - Enter a value for the password when prompted (`elastic_data_2`)

11. Use Elasticsearch's keystore utility to add a truststore password
```
/usr/share/elasticsearch/bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
```
   - Enter a value for the password when prompted (`elastic_data_2`)

12. Open `/etc/elasticsearch/elasticsearch.yml` in an editor and add the below security configurations (optionally create a Security section to put these under):
```
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: full
xpack.security.transport.ssl.keystore.path: data-2
xpack.security.transport.ssl.truststore.path: data-2
```

## Set Built-in User Passwords

1. Use Elasticsearch's password setup utility to set the passwords for the built-in users
```
/usr/share/elasticsearch/bin/elasticsearch-setup-passwords interactive
```
   - elastic: elastic_566
   - apm_system: apt_system_566
   - kibana: kibana_566
   - logstash_system: logstash_system_566
   - beats_system: beats_system_566
   - remote_monitoring_user: remote_monitoring_user_566

2. Open `/etc/kibana/kibana.yml` in an editor and configure Kibana to use the Kibana username and password set above
```
elasticsearch.username: "kibana"
elasticsearch.password: "kibana_566"
```

3. Start Kibana
```
systemctl start kibana
```

4. Enter the public IP of the master-1 node into a web browser using HTTP port 8080, example:
```
http://12.34.56.78:8080
```
   You should now see this screen:
   
   <img src="https://user-images.githubusercontent.com/104564793/173765667-9cd29c46-b353-4787-ad57-7b0a6d72a8a0.png" width=800>

5. Login using username `kibana` and password `kibana_566`

6. Under Dev Tools, confirm you can interact with the cluster
<img src="https://user-images.githubusercontent.com/104564793/173766338-2ff0719b-9a74-4812-a278-dcb1e7a89e46.png" width=800>
