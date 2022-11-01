## Get info
To view cluster settings (enter into browser):
```
http://<Elasticsearch URL>:9200/_cluster/settings?pretty&include_defaults
```

To view node stats (enter into browser):
```
http://<Elasticsearch URL>:9200/_nodes/stats?metric=adaptive_selection,breaker,discovery,fs,http,indices,jvm,os,process,thread_pool,transport&filter_path=nodes.*.adaptive_selection*,nodes.*.breaker*,nodes.*.fs*,nodes.*.os*,nodes.*.jvm*,nodes.*.process*,nodes.*.thread_pool*,nodes.*.discovery.cluster_state_queue,nodes.*.discovery.published_cluster_states,nodes.process.*.*,nodes.*.indices*,nodes.*.http.current_open,nodes.*.http.total_opened,_nodes,cluster_name,nodes.*.attributes,nodes.*.timestamp,nodes.*.transport*,nodes.*.transport_address,nodes.*.transport_address,nodes.*.host,nodes.*.ip,,nodes.*.roles,nodes.*.name&pretty
```

## Test results
🔴 Results of testing restoration of snapshot from ES6 into cluster running ES8.

Command:
```
POST /_snapshot/s3_test_repo/index_data_snapshot/_restore?wait_for_completion=true
{
  "indices": "dev-monitoring_perftest-3stepit-k8s-2022.07.20, dev-logs_uat-3stepit-k8s_kubernetes-2022.07.20"
}
```
Result:
```
{
  "error": {
    "root_cause": [
      {
        "type": "security_exception",
        "reason": "current license is non-compliant for [archive]",
        "license.expired.feature": "archive"
      }
    ],
    "type": "security_exception",
    "reason": "current license is non-compliant for [archive]",
    "license.expired.feature": "archive"
  },
  "status": 403
}
```


🔴 Snapshot taken from ES7 was upgraded to 8 when cluster was upgraded to ES8.

