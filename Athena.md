# AWS Athena
---
## Create database

```
create database DATABASENAME
```

## Create table

```
CREATE EXTERNAL TABLE IF NOT EXISTS DATABASENAME.cloudfront_logs (
  `date` DATE,
  time STRING,
  location STRING,
  bytes BIGINT,
  requestip STRING,
  method STRING,
  host STRING,
  uri STRING,
  status INT,
  referrer STRING,
  useragent STRING,
  querystring STRING,
  cookie STRING,
  resulttype STRING,
  requestid STRING,
  hostheader STRING,
  requestprotocol STRING,
  requestbytes BIGINT,
  timetaken FLOAT,
  xforwardedfor STRING,
  sslprotocol STRING,
  sslcipher STRING,
  responseresulttype STRING,
  httpversion STRING,
  filestatus STRING,
  encryptedfields INT
)
PARTITIONED BY(year string, month string, day string, hour string)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
LOCATION 's3://YOUR_BUCKET/structured/2020/06/11/';
```
## Adding data to a partition

```
ALTER TABLE cloudfront_logs 
ADD PARTITION (year='2020',month='06',day='11',hour='07') 
LOCATION 's3://YOUR_BUCKET/structured/2020/06/11/07'
```

## Sample Queries

```
SELECT *
FROM cloudfront_logs
WHERE year='2020'
        AND month= '06'
        AND day='11'
        AND status = 200
```
