# S3 Snapshot Repository

## Create and Register S3 Repository
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
6. SSHed into all three nodes and sudo su
7. Installed S3 plugin on all three nodes
```
cd /usr/share/elasticsearch
bin/elasticsearch-plugin install --batch repository-s3
```
8. Used Elasticsearch's keystore utility to add the S3 access key ID on all three nodes
```
/usr/share/elasticsearch/bin/elasticsearch-keystore add s3.client.default.access_key
```
   - Entered the access key ID from step 4
9. Used Elasticsearch's keystore utility to add the S3 secret access key on all three nodes
```
/usr/share/elasticsearch/bin/elasticsearch-keystore add s3.client.default.secret_key
```
   - Entered the secret key from step 4
10. Restarted Elasticsearch on all three nodes
```
systemctl restart elasticsearch
```
11. Ran the below command in Kibana (signed in as `elastic` user)
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
Alternately, the below command may be run from CLI (untested).
```
curl -u elastic:elastic_566 -X PUT "https://localhost:9200/_snapshot/s3_test_repo?pretty" -H 'Content-Type: application/json' -d'
{
 "type": "s3",
 "settings": {
   "bucket": "elk-update-3-step-test",
   "region": "eu-central-1"
 }
}
'
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
