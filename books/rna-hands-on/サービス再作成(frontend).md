---
title: "はじめに「未経験転職は難しい。だからこそ」"
---

## この章やること

先ほど作成したALB(frontend)をECS(frontend)に付与します。

## 手順

「ECS > クラスター > zenn-clone-cluster」にアクセスし、稼働している`zenn-clone-frontend-service`サービスをいったん削除します。

![](https://storage.googleapis.com/zenn-user-upload/c1d720e5a74e-20230821.png)

↓

削除が完了したら、再度「作成」を行います。

基本的な設定は先ほど削除したサービスと同様ですが、ALBに関する設定のみ追加があります。念のため、先ほどと同じ設定情報も再掲します。

#### デプロイ設定

|項目|値|
|---|---|
|アプリケーションタイプ|サービス|
|タスク定義 > ファミリー|zenn-clone-task-definition-frontend|
|リビジョン|最新|
|サービス名|zenn-clone-frontend-service|
|サービスタイプ|レプリカ|
|必要なタスク|1|

![](https://storage.googleapis.com/zenn-user-upload/391601fcd105-20230819.png)

![](https://storage.googleapis.com/zenn-user-upload/bafa3cbe57a5-20230819.png)

#### ネットワーキング

|項目|値|
|---|---|
|VPC|zenn-clone-vpc|
|サブネット|パブリックサブネット2つを選択|
|セキュリティグループ|既存のセキュリティグループを使用 > zenn-clone-ecs-frontend-security-group|
|パブリックID|オン|

![](https://storage.googleapis.com/zenn-user-upload/3db90481d97e-20230819.png)

![](https://storage.googleapis.com/zenn-user-upload/50be107c936e-20230819.png)

#### ロードバランシング

前章で作成したALB(frontend)を適用します。

|項目|値|
|---|---|
|ロードバランサーの種類|Application Load Balancer|
|Application Load Balancer|既存のロードバランサーを使用|
|ロードバランサー|zenn-clone-alb-frontend|
|ロードバランス用のコンテナの選択|next 80:80|
|リスナー > ポート|80|
|リスナー > プロトコル|HTTP|
|ターゲットグループ|既存のターゲットグループを使用|
|ターゲットグループ名|zenn-clone-alb-frontend-tg|

![](https://storage.googleapis.com/zenn-user-upload/dc45f54fc75d-20230822.png)

![](https://storage.googleapis.com/zenn-user-upload/7c42e44a30d4-20230822.png)
↓

入力が完了したら、「作成」ボタンを押下してサービスを作成してください。

その後、作成されたサービスの詳細画面から起動タスクの詳細画面に入り、nextコンテナが正常に起動することを確認してください。

![](https://storage.googleapis.com/zenn-user-upload/008d2abdf57a-20230822.png)

↓

ALBを経由してアプリケーションにアクセスできることを確認します。ALB(frontend)の詳細画面に入りに、**DNS名**を確認してください。

![](https://storage.googleapis.com/zenn-user-upload/a4188add1d8c-20230822.png)

ブラウザから`http://{{ ALB DNS名 }}/api/health_check`にアクセスをし、ヘルスチェックメッセージが返ってくればOKです！

![](https://storage.googleapis.com/zenn-user-upload/dfb690576d5a-20230822.png)

### ALB(frontend)への独自ドメイン付与

最後に、ALB(frontend)への独自ドメインの付与を行います。

検索窓から「Route53」に入り、「レコードを作成」ボタンを押下してください。

![](https://storage.googleapis.com/zenn-user-upload/815585fbab36-20230818.png)

↓

もし、「レコードをクイックに作成」の画面が表示されたら、右上の「ウィザードに切り替える」をクリックして画面を切り替えてください。

![](https://storage.googleapis.com/zenn-user-upload/b4eb784e112d-20230818.png)

↓

ルーティングポリシーを選択する画面が表示されたら、「シンプルルーティング」を選択して、「次へ」を押してください。

![](https://storage.googleapis.com/zenn-user-upload/19f0a6937707-20230818.png)

↓

「シンプルなレコードを定義」ボタンを押すとモーダルが表示されます。以下のように入力して、「シンプルなレコードを定義」を押します。

|項目|値|
|---|---|
|レコード名|(空欄)|
|レコードタイプ|A|
|値/トラフィックのルーティング先|Application Load Balancer と Classic Load Balancer へのエイリアス|
|リージョン|アジアパシフィック（東京）|
|ALB|作成したECS(frontend)用のALBを選択|

![](https://storage.googleapis.com/zenn-user-upload/b190838dbba0-20230822.png)

↓

「レコードを作成」をクリックし、レコードが正常に作成されればOKです。

↓

動作確認をします！ブラウザから、`https://{{ 独自ドメイン }}`にアクセスしてください。トップページにアクセスでき、seedデータとして登録したarticlesレコードが並んでいることが確認できます！

![](https://storage.googleapis.com/zenn-user-upload/aac663eea68d-20230822.png)

その他の機能も確認してみてください。現時点で、**サインアップを除いた全ての機能**が有効になっているはずです。
