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
