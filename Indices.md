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
