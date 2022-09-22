# Install Elasticsearch 7.10.2
Download Linux archive
```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.2-linux-x86_64.tar.gz
```
Download hash
```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.2-linux-x86_64.tar.gz.sha512
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
Configuration files are under `config` directory (elasticsearch.yml, jvm.options, etc)

