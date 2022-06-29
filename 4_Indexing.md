# Define Indices
Open Kibana frontend by entering the public IP of master-1 node into a browser and login as `elastic` user.

Under Dev Tools, run the following commands:

```
PUT bank
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1,
  }
}
```
```
PUT shakespeare
{
  "mappings": {
    "type_name": {
      "properties": {
        "speaker": {
          "type": "keyword"
        },
        "play_name": {
          "type": "keyword"
        },
        "line_id": {
          "type": "integer"
        },
        "speech_number": {
          "type": "integer"
        }
      }
    }
  },
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1
  }
}
```
```
PUT logs
{
  "mappings": {
    "type_name": {
      "properties": {
        "geo": {
          "properties": {
            "coordinates": {
              "type": "geo_point"
            }
          }
        }
      }
    }
  },
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1
  }
}
```

Note: `"type_name"` required for 6.x but is deprecated as of 7.x. Creating the indices for `shakespeare` and `logs` will display this warning in the console output:

![image](https://user-images.githubusercontent.com/104564793/174551170-4712ffdd-4396-47f2-a3d1-f4ef832cdec0.png)

# Bulk Index Data
SSH into the master-1 node and elevate to root user.

*! Below sample data doesn't work with 6.x, need to find appropriate data. !*

Run the following commands to download the sample data.
```
curl -O https://raw.githubusercontent.com/linuxacademy/content-elasticsearch-deep-dive/master/sample_data/accounts.json
curl -O https://raw.githubusercontent.com/linuxacademy/content-elasticsearch-deep-dive/master/sample_data/shakespeare.json
curl -O https://raw.githubusercontent.com/linuxacademy/content-elasticsearch-deep-dive/master/sample_data/logs.json
```

Bulk index the files into the appropriate indices.
```
curl -u elastic -k -H 'Content-type: application/x-ndjson' -X POST https://localhost:9200/bank/_bulk --data-binary @accounts.json
curl -u elastic -k -H 'Content-type: application/x-ndjson' -X POST https://localhost:9200/shakespeare/_bulk --data-binary @shakespeare.json
curl -u elastic -k -H 'Content-type: application/x-ndjson' -X POST https://localhost:9200/logs/_bulk --data-binary @logs.json
```
