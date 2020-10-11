---
author: "advincze"
tags: ["aws","athena","manifest","presto"]
categories: ["Development"]
date: "2020-10-11"
title: "AWS Athena manifest"
slug: "athena-manifest"
---

Athena allows you to process data using sql that is stored in s3. Usually you would specify an s3 path (bucket + prefix) below which you store the objects that belong to the table. Sometimes the location of your data does not adhere to this convention.  The objects might be in different locations, or you would want to include some objects to a table or partition. For these cases an s3 symlink inventory can be used.

Let's say you have multiple objects in a single bucket and you only want to process one of them:

#### create manifest

You need an object called `symlink.txt` in s3 containing a list of the objects that belong to the table. The object should contain one s3 location per line like this:

`s3://mymanifestbucket/prefix123/symlink.txt`:

```
s3://mybucket/foo/bar.parquet
s3://otherbucket/baz/qux.parquet
```

#### create athena table:

```sql
CREATE EXTERNAL TABLE foo(
  id string,
  foo float
)
ROW FORMAT SERDE 
    'org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe' 
STORED AS INPUTFORMAT 
    'org.apache.hadoop.hive.ql.io.SymlinkTextInputFormat'
LOCATION 's3://mymanifestbucket/prefix123/symlink.txt'
```


