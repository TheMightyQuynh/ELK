# ELK Upgrade
This guided example is based off A Cloud Guru's <a href="https://learn.acloud.guru/course/1e3ff00e-95bf-451b-be04-44d4bce6bfba/dashboard">Elastic Search Deep Dive</a> course. In the course he is using CentOS 7 servers on the learning platform's Cloud Playground to install Elasticsearch/Kibana 7.x, here I am using Ubuntu 22.04 on AWS to install the latest stable version of Elasticsearch/Kibana 6/7.

Go through the guide in this order:
1. [Setting up Elasticsearch 6.8.23 on AWS](https://github.com/TheMightyQuynh/ELK/blob/main/1_INSTALL_Elasticsearch_6.8.23.md)
2. [Setting up Kibana 6.8.23 on AWS](https://github.com/TheMightyQuynh/ELK/blob/main/2_INSTALL_Kibana_6.8.23.md)
3. [Secure the Cluster](https://github.com/TheMightyQuynh/ELK/blob/main/3_Secure_the_Cluster.md)
4. [Define Indices](https://github.com/TheMightyQuynh/ELK/blob/main/4_Indexing.md)
5. [Create a Snapshot Repo on AWS S3](https://github.com/TheMightyQuynh/ELK/blob/main/5_Snapshot.md)
6. [Setting up Elasticsearch 7.17.5 on AWS](https://github.com/TheMightyQuynh/ELK/blob/main/6_INSTALL_Elasticsearch_7.17.5.md)
7. [Setting up Kibana 7.17.5 on AWS](https://github.com/TheMightyQuynh/ELK/blob/main/7_INSTALL_Kibana_7.17.5.md)
8. [Secure the Cluster](https://github.com/TheMightyQuynh/ELK/blob/main/3_Secure_the_Cluster.md) for the cluster running 7.17.5 (same document from step 3)
9. [Install S3 Plugin](https://github.com/TheMightyQuynh/ELK/blob/main/5_Snapshot.md#register-the-repository) and [Register the Repository](https://github.com/TheMightyQuynh/ELK/blob/main/5_Snapshot.md#register-the-repository) for the cluster running 7.17.5
10. [Restore from Snapshot](https://github.com/TheMightyQuynh/ELK/blob/main/5_Snapshot.md#restore-from-snapshot)
11. [Upgrade 7.x to 8.x](https://github.com/TheMightyQuynh/ELK/blob/main/8_UPGRADE_to_ES_8.md)
