# Lambda CloudFront Log Restructure
---
A simple Lambda script to restructure CloudFront logs in S3 to a directory structure more useful for AWS Athena or EMR.

CloudFront does not allow log paths to be customized, aside from Bucket and prefix. This causes a problem for services such as AWS Athena and Elastic Map Reduce, which benefit from partitioned data paths. This script can copy or move the files created by CloudWatch instantly and automatically.

This script will adjust CloudFront logs structure as so:
```
s3://YOUR-BUCKET/raw/ET5OCLG4OFMBJ.2020-06-11-07.d652f9cc.gz
To this:
s3://YOUR-BUCKET/structured/2020/06/11/07/ET5OCLG4OFMBJ.2020-06-11-07.d652f9cc.gz
```

- The function is triggered by S3 as soon as CloudFront writes a log file.
- The function can be set to move or only copy the log files.
- If S3 triggers Lambda with multiple records, the function will handle each correctly.
- The function can handle multiple CloudFront distributions.

## Install / setup:
1. Create a new blank Lambda function
 - Name: CloudFront_Log_Restructure
 - Runtime: Python 2.7
 + Choose or create an execution role
 - Create a new role from AWS policy templates
 - Role name: Lambda_CloudFront_Log_Restructure

2. Set up an S3 trigger
 - Bucket: Where you write your CloudFront logs
 - Event type: All object create events
 - Prefix: raw/
 - Enable trigger: [true]
 + if have error [overlapping for the same event type] then open S3 console -> Properties -> Events to delete old setup

3. Back on "Configure function" page: Basic settings

 - Description: Restructures CloudFront logs in S3 to a directory structure more useful for AWS Athena or EMR
 - Runtime: Python 2.7
 - Handler: index.lambda_handler
 - Memory (MB): 128
 - Timeout: 5 seconds
 
4. Back on "Configure function" page: Function code
 - Change name file py to index.py
 - Replace content file

5. Go into IAM and edit the newly created Role "Lambda_CloudFront_Log_Restructure"

 - On the permissions tab, add an inline policy named "S3-Access", with the following content:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:DeleteObject"
            ],
            "Resource": [
                "arn:aws:s3:::YOUR-BUCKET/raw/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::YOUR-BUCKET/structured/*"
            ]
        }
    ]
}
```

6. Edit your CloudFront distribution(s) to write their logs to the S3 bucket with the prefix "raw/"
