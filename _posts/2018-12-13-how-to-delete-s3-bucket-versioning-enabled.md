---
title: バージョニングを有効にした S3 バケットの削除方法
date: 2018-12-13 15:26:00 +0900
tags:
    - AWS
    - S3
    - awscli
---

## tl; dr

バージョニングを有効にした S3 バケットを削除する際、事前に全てのバージョンのオブジェクトを削除する必要がある。

## 確認

バージョニングを有効にしたバケットを作成し、その中にファイルを作成した後、バケットを空にしてからバケットの削除を試す。

```
$ aws s3api create-bucket --bucket BUCKET-NAME
$ aws s3api put-bucket-versioning --bucket BUCKET-NAME --versioning-configuration Status=Enabled
$ echo "hello, world" > KEY-NAME
$ aws s3 cp KEY-NAME s3://BUCKET-NAME
upload: ./KEY-NAME to s3://BUCKET-NAME/KEY-NAME
$ aws s3 rm s3://BUCKET-NAME --recursive
delete: s3://BUCKET-NAME/KEY-NAME
$ aws s3 rb s3://BUCKET-NAME --force
remove_bucket failed: s3://BUCKET-NAME An error occurred (BucketNotEmpty) when calling the DeleteBucket operation: The bucket you tried to delete is not empty. You must delete all versions in the bucket.
```

当たり前だが、面倒くさい…。

試しにバージョニングを無効にしてたら削除できないかしら…？

```
$ aws s3api get-bucket-versioning --bucket BUCKET-NAME
{
    "Status": "Enabled",
    "MFADelete": "Disabled"
}
$ aws s3api put-bucket-versioning --bucket BUCKET-NAME --versioning-configuration Status=Suspended

An error occurred (InvalidBucketState) when calling the PutBucketVersioning operation: A WORM configuration is present on this bucket, so the versioning state cannot be changed.
```

はい、ダメでした。

仕方がないのでバージョニングされたオブジェクトを全て削除する。

```
$ aws s3api list-object-versions --bucket BUCKET-NAME
{
    "Versions": [
        {
            "ETag": "\"ETAG-ID\"",
            "Size": 4,
            "StorageClass": "STANDARD",
            "Key": "KEY-NAME",
            "VersionId": "VERSION-ID-OLD",
            "IsLatest": false,
            "LastModified": "2018-12-13T06:00:19.000Z",
            "Owner": {
                "DisplayName": "OWNER-NAME",
                "ID": "OWNER-ID"
            }
        }
    ],
    "DeleteMarkers": [
        {
            "Owner": {
                "DisplayName": "OWNER-NAME",
                "ID": "OWNER-ID"
            },
            "Key": "KEY-NAME",
            "VersionId": "VERSION-ID-LATEST",
            "IsLatest": true,
            "LastModified": "2018-12-13T06:00:24.000Z"
        }
    ]
}
$ aws s3api delete-object --bucket BUCKET-NAME --key "KEY-NAME" --version-id "VERSION-ID-OLD"
{
    "VersionId": "VERSION-ID-OLD"
}
$ aws s3api delete-object --bucket BUCKET-NAME --key "KEY-NAME" --version-id "VERSION-ID-LATEST"
{
    "DeleteMarker": true,
    "VersionId": "VERSION-ID-LATEST"
}
$ aws s3api list-object-versions --bucket BUCKET-NAME
$ aws s3 rb s3://BUCKET-NAME
remove_bucket: BUCKET-NAME
```

べ、別に忘れてウッカリしてたわけじゃないんだからね！(白状
