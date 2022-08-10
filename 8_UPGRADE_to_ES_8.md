# Upgrade Elasticsearch/Kibana 7.17 to 8.x

Make sure there is a current snapshot taken before beginning the upgrade.

## Upgrade Assistant
Use Upgrade Assistant to identify and resolve any issues and reindex indices created prior to Elasticsearch 7.

![1_Elasticsearch deprecation issues](https://user-images.githubusercontent.com/104564793/182584599-185f0303-708f-40d0-9947-17f8b56ef97e.png)

Various settings in `elasticsearch.yml` are deprecrated and must be updated - Upgrade Assistant provides guidance on what should be done
![6_node master is deprecated msg](https://user-images.githubusercontent.com/104564793/182584818-165a6e27-3856-423f-b9f9-e93f835bcd9f.png)
![10_master-1 node role master ingest](https://user-images.githubusercontent.com/104564793/182586169-b935ffc5-e2c2-4821-ba52-358ca12eeab8.png)

Indices created prior to version 7 must be reindexed - accept Upgrade Assistant's changes to complete reindexing for each index
![2_Reindex dialog](https://user-images.githubusercontent.com/104564793/182584696-e3b3c62b-a436-47fa-a09b-cfd721a24727.png)
![3_Reindex dialog2](https://user-images.githubusercontent.com/104564793/182585075-d48b47f5-309e-4cb8-b854-ccb195b57eb0.png)
![4_Reindex of one index complete](https://user-images.githubusercontent.com/104564793/182586038-701be68f-095c-40ae-acc4-7eafbce4d6fa.png)

## Upgrade Nodes
Disable shard allocation via the Kibana console (Dev Tools).
```
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.enable": "primaries"
  }
}
```
Success:

![image](https://user-images.githubusercontent.com/104564793/183371381-203a3b7f-3ec7-4b30-86de-4e74c6e6bb6b.png)

(Optional) Temporarily stop non-essential indexing and perform a flush. Indexing can be continued during upgrading, but stopping it and performing a flush allows shard recovery to go faster.
```
POST /_flush
```

(Optional) Temporarily stop tasks associated with machine learning jobs and data feeds. Jobs can be continued during upgrading, but it puts increased load on the cluster.
```
POST _ml/set_upgrade_mode?enabled=true
```


Starting with non-master eligible nodes and going from frozen, cold, warm, and finally hot data tiers, upgrade the nodes one by one as follows:

Copy or make note of the settings in `/etc/elasticsearch/elasticsearch.yml` and `/etc/elasticsearch/jvm.options` (optionally) as they will be erased after installing the new version.

Shut down a single node (switch to root account or use with sudo command).
```
systemctl stop elasticsearch
```

Import the Elasticsearch PGP key.
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg

```

Install Elasticsearch 8.x from the latest repository.
```
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
```
```
sudo apt-get update && sudo apt-get install elasticsearch
```

Enter `Y` or `I` to select the option to install the updated versions of `elasticsearch.yml` and `jvm.options`.

Edit `/etc/elasticsearch/elasticsearch.yml` and `jvm.options` (optional) with the correct settings for the node.

Upgrade any plugins using the `elasticsearch-plugin` script.

To list all plugins:
```
/usr/share/elasticsearch/bin/elasticsearch-plugin list
```
In this case, only the `repository-s3` plugin was installed, so remove and install to upgrade it.
```
/usr/share/elasticsearch/bin/elasticsearch-plugin remove repository-s3
```
```
/usr/share/elasticsearch/bin/elasticsearch-plugin install repository-s3
```

Reload the source configuration file for elasticsearch.service.
```
systemctl daemon-reload
```

Start the node.
```
systemctl start elasticsearch
```

If upgrading a data node, re-enable shard allocation via the Kibana console.
```
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.enable": null
  }
}
```

Wait for the node to recover before upgrading the next node; check the cluster health and ensure the status is **green**.
```
GET _cat/health?v=true
```

Repeat these steps for each node to be upgraded.

To check which nodes have been upgraded:
```
GET /_cat/nodes?h=ip,name,version&v=true
```

(Optional) Restart machine learning jobs.
```
POST _ml/set_upgrade_mode?enabled=false
```

## Upgrade Kibana
Once Elasticsearch has been upgraded, Kibana may be upgraded next. Check Upgrade Assistant to ensure this is the case.
![image](https://user-images.githubusercontent.com/104564793/183850359-aaea9a79-7591-4e53-a6d0-493d45dc0f4e.png)

On the **master-1** node (the node hosting Kibana), shut down Kibana (switch to root account or use with sudo command).
```
systemctl stop kibana
```

The latest repository definition should already be saved from the Elasticsearch upgrade earlier but here is the command again just in case.
```
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
```

Install Kibana 8.x from the latest repository.
```
sudo apt-get update && sudo apt-get install kibana
```

Enter `Y` or `I` to select the option to install the updated versions of `kibana.yml`.

Edit `/etc/kibana/kibana.yml` with the correct settings.
```
server.port: 8080
server.host: "`private IP master-1`"
elasticsearch.hosts: ["https://localhost:9200"]
elasticsearch.username: "kibana_system"
elasticsearch.password: "kibana_566"
elasticsearch.ssl.verificationMode: none
```

Edit `/etc/systemd/system/kibana.service` and remove `--logging.dest="/var/log/kibana/kibana.log"` from the line beginning with `ExecStart`
Before:
```
ExecStart=/usr/share/kibana/bin/kibana --logging.dest="/var/log/kibana/kibana.log" --pid.file="/run/kibana/kibana.pid" --deprecation.skip_deprecated_settings[0]="logging.dest"
```
After:
```
ExecStart=/usr/share/kibana/bin/kibana --pid.file="/run/kibana/kibana.pid" --deprecation.skip_deprecated_settings[0]="logging.dest"
```

Reload the source configuration file for kibana.service.
```
systemctl daemon-reload
```

Start Kibana.
```
systemctl start kibana
```
