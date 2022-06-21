# Create Snapshot
This doesn't work but here are the steps anyway so I don't forget.

1. Created an S3 bucket named `elk-update-3-step-test`
2. From AWSAdministratorAccess account, added IAM user `elk-3step-test`
3. Created the below custom policy and attached it to the user
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "s3:*"
            ],
            "Effect": "Allow",
            "Resource": [
                "arn:aws:s3:::elk-3-step-test",
                "arn:aws:s3:::elk-3-step-test/*"
            ]
        }
    ]
}
```
4. Copied access key ID and secret access key
5. SSHed into all three nodes and sudo su
6. Installed S3 plugin on all three nodes and restarted Elasticsearch
```
cd /usr/share/elasticsearch
bin/elasticsearch-plugin install --batch repository-s3
systemctl restart elasticsearch
```
7. Add the below configuration to `/etc/elasticsearch/jvm.options` on master-1 node
```
-Des.allow_insecure_settings=true
```
8. Run the below command on master-1 node
```
curl -u elastic:elastic_566 -X PUT "https://localhost:9200/_snapshot/s3_test_repo?pretty" -H 'Content-Type: application/json' -d'
{
 "type": "s3",
 "settings": {
   "bucket": "elk-update-3-step-test",
   "region": "eu-central-1",
   "access_key": "ACCESS-KEY-ID",
   "secret_key": "SECRET-ACCESS-KEY"
 }
}
' --insecure
```

And get an error LOL FML
```
{
  "error" : {
    "root_cause" : [
      {
        "type" : "repository_verification_exception",
        "reason" : "[s3_test_repo] path  is not accessible on master node"
      }
    ],
    "type" : "repository_verification_exception",
    "reason" : "[s3_test_repo] path  is not accessible on master node",
    "caused_by" : {
      "type" : "i_o_exception",
      "reason" : "Unable to upload object [tests-v1f70bzeQK6eKaRj3RYx4w/master.dat] using a single upload",
      "caused_by" : {
        "type" : "amazon_s3_exception",
        "reason" : "Access Denied (Service: Amazon S3; Status Code: 403; Error Code: AccessDenied; Request ID: Y4SCJM79F5S3DWYX; S3 Extended Request ID: r8LxFxwQsoWYKklUnMgqqOHEvpgYgM6ng8VO2j96XX4fXuFJOSSKeWWRG7AVDi054VkUQ5PPtXw=)"
      }
    }
  },
  "status" : 500
}
```
