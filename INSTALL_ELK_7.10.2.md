# Install Elasticsearch 7.10.2
See [installation guide](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/targz.html)

Download Linux archive (this link is for the version which includes only Apache 2.0 licensed code)
```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-oss-7.10.2-linux-x86_64.tar.gz
```
Download hash
```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-oss-7.10.2-linux-x86_64.tar.gz
```
Compare SHA of archive to published checksum
```shasum -a 512 -c elasticsearch-7.10.2-linux-x86_64.tar.gz.sha512 
```
Output should be `elasticsearch-{version}-linux-x86_64.tar.gz: OK`

Unpack archive
```
tar -xzf elasticsearch-7.10.2-linux-x86_64.tar.gz
```
Change directory to Elasticsearch folder
```
cd elasticsearch-7.10.2/
```
Edit configuration files in the `config` directory
#### master-1 node
Settings for `elasticsearch.yml`:
```
cluster.name: playground
node.name: master-1
network.host: [_local_, _site_]
discovery.seed_hosts: ["`private IP master-1`"]
cluster.initial_master_nodes: ["master-1"]
node.master: true
node.data: false
node.ingest: true
node.ml: false
```
Settings for `jvm.options`:
```
-Xms768m
-Xmx768m
```
#### data-*x* nodes
Settings for `elasticsearch.yml`:
TODO: Add the settings

Start Elasticsearch (within `/elasticsearch-7.10.2` directory)
```
bin/elasticsearch
```
