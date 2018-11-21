I needed to copy files from an S3 bucket on one amazon account to another.

## Setup a user on the original AWS account

This will be used to run the actual copy/sync operation. This user has to have access to the both buckets that are being transferred between.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket",
                "s3:GetObject",
                "s3:PutObject",
                "s3:PutObjectAcl",
                "s3:GetBucketLocation"
            ],
            "Resource": [
                "arn:aws:s3:::my-destination-bucket",
                "arn:aws:s3:::my-destination-bucket/*",
                "arn:aws:s3:::my-original-bucket",
                "arn:aws:s3:::my-original-bucket/*"
            ]
        }
    ]
}
```

This will allow the user to be able to kick off a copy operation with both buckets. It's important to note that this user should be able to push ACLs so that the files are actually owned by the new bucket.

## Setup a bucket policy on destination AWS account

Next we need to actually allow the above user to access the bucket on the destination account. We can do this via a S3 bucket policy.

Simply add this policy to the bucket we want to be accessed. Note the bucket name, as well as the ARN of the above user should be included.

```json

{
  "Id": "WebbernetBackup",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1",
      "Action": "s3:*",
      "Effect": "Allow",
      "Resource": [
        "arn:aws:s3:::<DESTINATION BUCKET NAME>",
        "arn:aws:s3:::<DESTINATION BUCKET NAME>/*"
      ],
      "Principal": {
        "AWS": [
          "arn:aws:iam::<ORIGINAL_ACCOUNT_ID>:user/<ORIGINAL_ACCOUNT_USER_NAME>"
        ]
      }
    }
  ]
}
```

## Running the Sync Operation

```shell
$ aws s3 --profile new_user_profile sync s3://my-original-bucket s3://my-destination-bucket --acl 'bucket-owner-full-control'
```

It's important to remember the ACL permissions, otherwise the newly copied files will still be owned by the original account even though they recide on the destination account.

