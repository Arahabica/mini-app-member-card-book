---
title: "お片付け"
---

## 6.1 Stackの削除
Serverless Frameworkを使うと、リソースの削除も簡単です。
下記のコマンド1発でCloudFormationのStackが一括で削除されます。

```
$ sls remove
```

## 6.2 Cloud9の環境を削除
Cloud9の管理画面から今回作成した環境を削除してください。

## 6.3 AWS Systems Manager Parameter Storeのパラメータの削除
AWS Systems Manager Parameter StoreだけStackの外にあるので、AWS Systems Manager Parameter Storeの画面でパラメータを削除してください。

## 6.4 デプロイ用Bucketの削除

なぜかデプロイ用Bucketだけ、残ってしまうので下記の名前のBucketをS3の画面から探して削除してください。
`{{YOUR_NAME}}-line-member-card-dev-deployment`