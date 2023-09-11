---
title: "はじめに「未経験転職は難しい。だからこそ」"
---

## この章でやること

backendタスク用のALBを作成します。

## ALBとは？

ALB(Application Load Balancer)は、AWSにデプロイしたアプリケーションにおける**ロードバランサー**の役割を果たすものです。

ロードバランサーというのは、クライアントからのリクエストをアプリケーション本体よりも先に受け取り、適切にアプリケーション側へ受け渡すことによって、**アプリケーションの負荷を軽減する**役割をもった要素です。

例えば、同じアプリケーションを2つ並列して稼働させておき、それらを束ねる位置にロードバランサーを1つ設置します。クライアント（ブラウザ）からのリクエストがあった場合、まずロードバランサーでそのリクエストを受け、2つのアプリのうちより負荷が軽そうな方にリクエストを横流しするようにします。このようなシステムにすることで、2つの並列稼働しているアプリが均等に働いてくれることになる（適切に負荷が分散されている）ので、より安定したシステムの運用が行えます。

ただし、**当教材でALBを作成するのは負荷分散が目的ではありません**。もちろんALBは負荷分散を行うことが趣旨の機能ですが、他にもいくつかの機能を提供しています。

当教材では、そのうちのひとつである、**ECSにデプロイしたアプリケーションに、独自ドメインでアクセスできるようにする**、という機能を目当てに利用したいと思います。

当教材ではタスクは（frontend, backendそれぞれで）1つずつしか起動しないので、実質的に負荷分散は行われません。そのため、**独自ドメインでアプリにアクセスできるようにALBを作成する**という認識で進めていただく方が、以降内容を理解しやすいかと思います。

## 手順

### ALB用のセキュリティグループの作成

ALB本体を作成する前に、ALB用のセキュリティグループを作成します。

「VPC > セキュリティグループ」にアクセスし、「セキュリティグループを作成」を押下します。

![](https://storage.googleapis.com/zenn-user-upload/451fa4534d35-20230527.png)

↓

以下のような項目を入力してください。

#### 基本的な詳細

|項目|値|
|---|---|
|セキュリティグループ名|zenn-clone-alb-backend-security-group|
|説明|任意|
|VPC|zenn-clone-vpc|

![](https://storage.googleapis.com/zenn-user-upload/510ab2e0b17e-20230527.png)

#### インバウンドルール

外部から入ってくるトラフィックのうち、アクセス許可する範囲を定義します。ここでは、任意のHTTP、HTTPS通信を全て許可します。

|タイプ|ソース|
|---|---|
|HTTP|Anywhere-IPv4(0.0.0.0/0)|
|HTTP|Anywhere-IPv6(::/0)|
|HTTPS|Anywhere-IPv4(0.0.0.0/0)|
|HTTPS|Anywhere-IPv6(::/0)|

![](https://storage.googleapis.com/zenn-user-upload/89464c3b021c-20230527.png)

#### アウトバウンドルール

ECS(backend)へトラフィックを送信できるように設定します。

|タイプ|送信先|
|---|---|
|HTTP|カスタム > zenn-clone-ecs-backend-security-group|

![](https://storage.googleapis.com/zenn-user-upload/6244483ff782-20230527.png)

↓

「セキュリティグループを作成」を押下します。

![](https://storage.googleapis.com/zenn-user-upload/1f89b91f2252-20230527.png)

### ALB(backend)用のターゲットグループの作成

ALB(backend)用の**ターゲットグループ**を作成します。ターゲットグループというのは、ALBが受け取ったリクエストをどのようにしてアプリケーション側に渡すのかを定義したものです。

検索窓から「EC2 > ターゲットグループ」に入り、「ターゲットグループの作成」を押下します。

![](https://storage.googleapis.com/zenn-user-upload/6d3964d9e0c4-20230818.png)

以下項目を入力して「次へ」進みます。

|項目|値|
|---|---|
|基本的な設定|IPアドレス|
|ターゲットグループ名|zenn-clone-alb-backend-tg|
|プロトコル|HTTP|
|ポート|80|
|IPアドレスタイプ|IPv4|
|VPC|zenn-clone-vpc|
|プロトコルバージョン|HTTP1|
|ヘルスチェックプロトコル|HTTP|
|ヘルスチェックパス|/api/v1/health_check|

![](https://storage.googleapis.com/zenn-user-upload/1cffee409bc3-20230818.png)

![](https://storage.googleapis.com/zenn-user-upload/db3e3c5bcc36-20230818.png)

![](https://storage.googleapis.com/zenn-user-upload/be2d00dfee2e-20230818.png)

↓

ターゲットの登録は特に変更せず、「ターゲットグループの作成」を押下します。

![](https://storage.googleapis.com/zenn-user-upload/25c050aae8b0-20230818.png)

![](https://storage.googleapis.com/zenn-user-upload/d1b15f5a4f74-20230818.png)

↓

ターゲットグループが正常に作成されればOKです！

![](https://storage.googleapis.com/zenn-user-upload/15f59c5d8c19-20230818.png)

### ALBの作成

検索窓から「EC2 > ロードバランサー」に入り、「ロードバランサーの作成」を押下します。

![](https://storage.googleapis.com/zenn-user-upload/e0871cefaed0-20230818.png)

↓

ロードバランサーの種類として「Application Load Balancer」を選択します。

![](https://storage.googleapis.com/zenn-user-upload/bab9e3e6d4d2-20230818.png)

↓

#### 基本的な設定

|項目|値|
|---|---|
|名前|zenn-clone-alb-backend|
|スキーム|インターネット向け|
|IP アドレスタイプ|ipv4|

![](https://storage.googleapis.com/zenn-user-upload/40a84d68f029-20230818.png)

#### ネットワークマッピング

|項目|値|
|---|---|
|VPC|zenn-clone-alb-backend|
|マッピング|ap-northeast-1a, ap-northeast-1c それぞれでパブリックサブネットを選択 |

![](https://storage.googleapis.com/zenn-user-upload/d494b2df634d-20230818.png)

#### セキュリティグループ

|項目|値|
|---|---|
|セキュリティグループ|zenn-clone-alb-backend-security-group|

#### リスナーとルーティング

「リスナーの追加」ボタンで、リスナーを二つ登録します。

|プロトコル|ポート|デフォルトアクション|
|---|---|---|
|HTTP|80|zenn-clone-alb-backend-tg|
|HTTPS|443|zenn-clone-alb-backend-tg|

![](https://storage.googleapis.com/zenn-user-upload/d5e6928ee651-20230818.png)

#### リスナーとルーティング > セキュアリスナーの設定

|項目|値|
|---|---|
|セキュリティポリシー|推奨のポリシーを選択|
|デフォルトの SSL/TLS 証明書|ACMから > 取得したドメインを選択|

![](https://storage.googleapis.com/zenn-user-upload/0dd4afc3c31d-20230818.png)

↓

「ロードバランサーの作成」ボタンを押下し、正常に作成されればOKです！

![](https://storage.googleapis.com/zenn-user-upload/ee18c5fe1409-20230818.png)
