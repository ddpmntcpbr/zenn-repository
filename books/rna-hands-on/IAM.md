---
title: "はじめに「未経験転職は難しい。だからこそ」"
---

## IAMとは

AWSにアクセスするユーザーに対する権限を管理する機能です。

チーム開発では、複数のエンジニアが自社サービスを運用しているAWSクラウドにアクセスをすることになります。このとき、エンジニアごとに許可する権限を変更したい、ということはよく発生します（例えば、インフラエンジニアは広い権限を持つが、その他エンジニアは必要最低限の権限のみを持つようにする等）。

IAMで、ユーザーひとりひとりの権限を個別に設定することができます。

## ユーザーとグループ

特定の権限を付与したグループを作成する
↓
ユーザーを作成し、グループに登録する

ユーザー自体に権限を付与することも可能ですが、グループ側に権限を持たせることの方が一般的です。一度グループを定義してしまえば、仮にエンジニアが新規でジョインした場合も、そのグループに招待するだけで他エンジニアと同じ権限を持つことが可能になるからです。

## 操作
¥![](https://storage.googleapis.com/zenn-user-upload/f95bdf54d5a1-20230404.png)

![](https://storage.googleapis.com/zenn-user-upload/e2f4fa4af1f2-20230404.png)

![](https://storage.googleapis.com/zenn-user-upload/11c3ab50948c-20230404.png)

![](https://storage.googleapis.com/zenn-user-upload/1c620fbbf2e3-20230404.png)

![](https://storage.googleapis.com/zenn-user-upload/78dc9c80ef0f-20230404.png)

```sh:ターミナル
$ aws configure
AWS Access Key ID: xxxxxxxxxxxxxxxx
AWS Secret Access Key: xxxxxxxxxxxxxxxx
Default region name: ap-northeast-1
Default output format: json
```
