---
title: "はじめに「未経験転職は難しい。だからこそ」"
---

## この章でやること

ここからは、frontend(Next.js)をECSにデプロイしていきます。大まかな流れはバックエンドのときと同様ですが、こちらについてもひとつずつ説明していきます。

まずは、frontend用のECSに使用するセキュリティーグループの作成から始めていきます。

## 手順

VPCダッシュボードから「セキュリティグループ」にアクセスし、「セキュリティグループを作成」を押下します。

![](https://storage.googleapis.com/zenn-user-upload/debe6ee4c31c-20230528.png)

#### 基本的な詳細

|項目|値|
|---|---|
|セキュリティグループ名|zenn-clone-ecs-frontend-security-group|
|説明|任意|
|VPC|zenn-clone-vpc|

![](https://storage.googleapis.com/zenn-user-upload/11ee362c7ff0-20230528.png)

#### インバウンドルール

全てのHTTP/HTTPS通信を許可します。

（HTTPだけでよいかも）

|タイプ|ソース|
|---|---|
|HTTP|Anywhere-IPv4|
|HTTP|Anywhere-IPv6|
|HTTPS|Anywhere-IPv4|
|HTTPS|Anywhere-IPv6|

![](https://storage.googleapis.com/zenn-user-upload/9974be04d27e-20230818.png)

#### アウトバウンドルール

デフォルトのままでOKです（念のため以下の画像のようになっていることを確認してください）。

![](https://storage.googleapis.com/zenn-user-upload/86bb14ec182d-20230528.png)

↓

「セキュリティグループを作成」を押下し、セキュリティグループが正常に作成されたらOKです！

![](https://storage.googleapis.com/zenn-user-upload/e3ad9de74bc2-20230528.png)
