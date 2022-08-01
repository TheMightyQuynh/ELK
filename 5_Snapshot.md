# S3 Snapshot Repository

## Create S3 Repository
1. Created an S3 bucket named `elk-update-3-step-test` with setting `Block all public access` checked (i.e. do not allow public access)
2. Configured the below bucket policy
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::524741049628:user/elk-3step-test"
            },
            "Action": [
                "s3:ListBucket",
                "s3:GetBucketLocation",
                "s3:ListBucketMultipartUploads",
                "s3:ListBucketVersions"
            ],
            "Resource": "arn:aws:s3:::elk-update-3-step-test"
        },
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::524741049628:user/elk-3step-test"
            },
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
                "s3:DeleteObject",
                "s3:AbortMultipartUpload",
                "s3:ListMultipartUploadParts"
            ],
            "Resource": "arn:aws:s3:::elk-update-3-step-test/*"
        }
    ]
}
```
3. Added IAM user `elk-3step-test` (requires using `AWSAdministratorAccess` AWS account)
4. Created the below custom policy `ElasticsearchSnapshotRepoTestAccess` and attached it to the user
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
5. Copied access key ID and secret access key of the newly created user

## Install S3 Plugin
1. SSHed into all three nodes and sudo su
2. Installed S3 plugin on all three nodes
```
cd /usr/share/elasticsearch
bin/elasticsearch-plugin install --batch repository-s3
```
3. Used Elasticsearch's keystore utility to add the S3 access key ID on all three nodes
```
/usr/share/elasticsearch/bin/elasticsearch-keystore add s3.client.default.access_key
```
   - Entered the access key ID from step 5 above
4. Used Elasticsearch's keystore utility to add the S3 secret access key on all three nodes
```
/usr/share/elasticsearch/bin/elasticsearch-keystore add s3.client.default.secret_key
```
   - Entered the secret key from step 5 above
5. Restarted Elasticsearch on all three nodes
```
systemctl restart elasticsearch
```

## Register the Repository
Ran the below command in Kibana (signed in as `elastic` user)
```
PUT _snapshot/s3_test_repo
{
 "type": "s3",
 "settings": {
   "bucket": "elk-update-3-step-test",
   "region": "eu-central-1"
 }
}
```
Alternately, the below command may be run from CLI.
```
curl -u elastic:elastic_566 -X PUT "https://localhost:9200/_snapshot/s3_test_repo?pretty" -H 'Content-Type: application/json' -d'
{
 "type": "s3",
 "settings": {
   "bucket": "elk-update-3-step-test",
   "region": "eu-central-1"
 }
}
' --insecure
```

Success:
```
{
  "acknowledged" : true
}
```

## Taking a Snapshot
Run the below command in Kibana (signed in as `elastic` user)
```
PUT _snapshot/s3_test_repo/UNIQUE_NAME_OF_SNAPSHOT
{
  "indices": "INDEX1, INDEX2",
  "ignore_unavailable": true,
  "include_global_state": false
}
```
Alternately, the below command may be run from CLI.
```
curl -u elastic:elastic_566 -X PUT "https://localhost:9200/_snapshot/s3_test_repo/snapshot_1?wait_for_completion=true&pretty" --insecure
```
Success:
```
{
  "snapshot" : {
    "snapshot" : "snapshot_1",
    "uuid" : "GtQ7PQZGS-yvAdh_Ad6hbg",
    "version_id" : 6082399,
    "version" : "6.8.23",
    "indices" : [
      "random_data_test",
      ".kibana_task_manager",
      ".kibana_1",
      ".security-6",
      "random_data_test2",
      "logs",
      "shakespeare",
      "bank"
    ],
    "include_global_state" : true,
    "state" : "SUCCESS",
    "start_time" : "2022-07-06T07:22:16.109Z",
    "start_time_in_millis" : 1657092136109,
    "end_time" : "2022-07-06T07:22:20.156Z",
    "end_time_in_millis" : 1657092140156,
    "duration_in_millis" : 4047,
    "failures" : [ ],
    "shards" : {
      "total" : 8,
      "failed" : 0,
      "successful" : 8
    }
  }
}
```

## Restore from Snapshot
See all snapshot repositories:
```
GET _snapshot/_all
```

See all snapshots in a repository:
```
GET _snapshot/s3_test_repo/_all
```

Restore a snapshot:
```
POST /_snapshot/s3_test_repo/name_of_snapshot/_restore?wait_for_completion=true
{
  "indices": "*"
}
```
Success:
![image](https://user-images.githubusercontent.com/104564793/182156180-a08bc67c-4b14-4bed-ab44-8b0397c857d8.png)

See [Restore snapshot API documentation](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/restore-snapshot-api.html) for full list of possible parameters and usage.

## Delete Snapshot Repository
The DELETE command unregisters the repository by removing the reference to the location of the snapshot. The snapshots themselves are not touched.
```
DELETE /_snapshot/s3_test_repo
```

If new snapshots are added to a repository after the repository has already been registered on the target cluster (the cluster running 7.17 in this case), the repository must be unregistered and re-registered for the latest updates to appear.
