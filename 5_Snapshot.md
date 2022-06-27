# Create Snapshot

1. Created an S3 bucket named `elk-update-3-step-test`
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
3. From AWSAdministratorAccess account, added IAM user `elk-3step-test`
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
5. Copied access key ID and secret access key
6. Created role `elk-3-step-test-role` with policy `ElasticsearchSnapshotRepoTestAccess` created above attached
7. In EC2, selected instance for master-1 node, then Actions > Security > Modify IAM Role and added the `elk-3-step-test-role` role to the instance
8. SSHed into all three nodes and sudo su
9. Installed S3 plugin on all three nodes
```
cd /usr/share/elasticsearch
bin/elasticsearch-plugin install --batch repository-s3
```
8. Added the below configuration to `/etc/elasticsearch/jvm.options` on all three nodes
```
-Des.allow_insecure_settings=true
```
9. Used Elasticsearch's keystore utility to add the S3 access key ID on **master-1** node
```
/usr/share/elasticsearch/bin/elasticsearch-keystore add s3.client.default.access_key
```
   - Entered the access key ID from step 4
10. Used Elasticsearch's keystore utility to add the S3 secret access key on **master-1** node
```
/usr/share/elasticsearch/bin/elasticsearch-keystore add s3.client.default.secret_key
```
   - Entered the secret key from step 4
11. Restarted Elasticsearch on all three nodes
```
systemctl restart elasticsearch
```
12. Ran the below command in Kibana (signed in as `elastic` user)
Note: I wasn't able to get register the repository using the keystore values, so the `access_key` and `secret_key` values were entered into the command. This is bad practice and has been deprecated.
```
PUT _snapshot/s3_test_repo
{
 "type": "s3",
 "settings": {
   "bucket": "elk-update-3-step-test",
   "region": "eu-central-1",
   "access_key": "ACCESS-KEY-ID",
   "secret_key": "SECRET-ACCESS-KEY"
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
   "region": "eu-central-1",
   "access_key": "ACCESS-KEY-ID",
   "secret_key": "SECRET-ACCESS-KEY"
 }
}
' --insecure
```

Success:
```
#! Deprecation: [access_key] setting was deprecated in Elasticsearch and will be removed in a future release! See the breaking changes documentation for the next major version.
#! Deprecation: [secret_key] setting was deprecated in Elasticsearch and will be removed in a future release! See the breaking changes documentation for the next major version.
#! Deprecation: Using s3 access/secret key from repository settings. Instead store these in named clients and the elasticsearch keystore for secure settings.
{
  "acknowledged" : true
}
```
