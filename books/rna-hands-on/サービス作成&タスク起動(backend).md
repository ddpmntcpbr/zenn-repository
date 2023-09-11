## この章でやること

前章でタスク定義を行なったbackend用タスクを実際に起動させることによってnginx+Railsをデプロイし、インターネットからアクセスできることを確認します。

## タスク起動までの流れ

- 開発アプリ用の**クラスター**を作成
- クラスター配下に、backendタスク起動用の**サービス**を作成

**サービス**というのは、タスク定義に基づいたタスクを起動させる主体です。当章ではbackend用タスク用のサービスをひとつ作成しますが、後の章ではfrontend用のものも作成します。

**クラスター**は、複数のサービスを取りまとめる主体です。これは当教材でただ一つだけ作成します。

## 注意点

ECSタスクの運用は、特に大きな課金費用が発生するサービスとなりますので、ご注意ください。今回の開発アプリでは、**24時間の運用で数百円程度の費用が発生する**可能性があります。

作成したサービスを終了させてしまえば、ECSタスク運用に関する費用発生は止まりますので、以降章も含めての学習が最後まで完了できたら、早めにサービスを削除することをオススメします。

## 手順

### クラスターの作成

「Elastic Container Service > クラスター」にアクセスし、「クラスターの作成」を押下します。

![](https://storage.googleapis.com/zenn-user-upload/d4a6c5a126cb-20230818.png)

↓

以下を入力し、「作成」を押下します。

|項目|値|
|---|---|
|クラスター名|zenn-clone-cluster|
|VPC|zenn-clone-vpc|
|サブネット|zenn-clone-vpc内の全てのサブネットを選択（プライベート2つ、パブリック2つ）|

![](https://storage.googleapis.com/zenn-user-upload/cb832e193584-20230818.png)

![](https://storage.googleapis.com/zenn-user-upload/7cf7cddd33be-20230818.png)

↓

正常にクラスターが作成されればOKです。

![](https://storage.googleapis.com/zenn-user-upload/ab8108b809b0-20230818.png)

### サービス(backend)の作成

`zenn-clone-cluster`をクリックしてクラスターの詳細画面に入り、画面下部「サービス」タブから「作成」に進んでください。

![](https://storage.googleapis.com/zenn-user-upload/dbb721eac498-20230818.png)

↓

サービスの設定項目を入力していきます。

#### 環境

|項目|値|
|---|---|
|コンピューティングオプション|起動タイプ|
|起動タイプ|FARGATE|
|プラットフォームバージョン|LATEST|

![](https://storage.googleapis.com/zenn-user-upload/94beb60754c1-20230818.png)

#### デプロイ設定

|項目|値|
|---|---|
|アプリケーションタイプ|サービス|
|タスク定義 > ファミリー|zenn-clone-task-definition-backend|
|リビジョン|最新|
|サービス名|zenn-clone-backend-service|
|サービスタイプ|レプリカ|
|必要なタスク|1|

![](https://storage.googleapis.com/zenn-user-upload/d5885ca1747b-20230818.png)

![](https://storage.googleapis.com/zenn-user-upload/0eb6701484ec-20230818.png)


#### ネットワーキング

|項目|値|
|---|---|
|VPC|zenn-clone-vpc|
|サブネット|パブリックサブネット2つを選択|
|セキュリティグループ|既存のセキュリティグループを使用 > zenn-clone-ecs-backend-security-group|
|パブリックID|オン|

ここでついに、ECS(backend)用に作成したセキュリティグループを使用します。

![](https://storage.googleapis.com/zenn-user-upload/58ee4c48e40e-20230818.png)

![](https://storage.googleapis.com/zenn-user-upload/7b9fee5ab1bd-20230818.png)

#### ロードバランシング

後ほど触れますが、ここではスルーでOKです。

![](https://storage.googleapis.com/zenn-user-upload/f975c0f26c5e-20230818.png)

↓

入力が完了したら「作成」してください。作成したサービス`zenn-clone-backend-service`をクリックして詳細画面に入り、タブ「タスク」を開くと、サービスが起動しているタスクの状況が確認できます。

![](https://storage.googleapis.com/zenn-user-upload/da0590ef9fcc-20230818.png)

タスク名をクリックするとタスクの詳細画面にアクセスできます。タスク詳細画面下部の「コンテナ」欄から、railsコンテナ、nginxコンテナそれぞれの状況が確認できます。

一定の時間が経過した後、両コンテナとも、「ステータス: Running」、「ヘルスステータス: 正常」になっていれば、タスクが正常に起動されています！

![](https://storage.googleapis.com/zenn-user-upload/85a64b012734-20230818.png)

## 動作確認

タスクが起動しているということは、すなわちbackendアプリをデプロイしたということであるので、インターネットからアクセスができる状態になっていますので、確認してみましょう。

タスク詳細画面の「設定 > パブリックIP」が、当タスクのパブリックIPになります。

![](https://storage.googleapis.com/zenn-user-upload/d09db4b932b9-20230818.png)

ブラウザから、`http://{{ パブリックIP }}/api/v1/health_check`にアクセスをしてみてください。ヘルスチェックメッセージが正常に返ってくればOKです　！

![](https://storage.googleapis.com/zenn-user-upload/f3bc49830962-20230818.png)

また、`http://{{ パブリックIP }}/api/v1/articles`にアクセスすると、seedデータとして登録した article レコード（ページネーションの先頭10件）が返ってくることも確認できます。

![](https://storage.googleapis.com/zenn-user-upload/c8d89a618c27-20230818.png)

## ログ確認

サービスによってタスクの起動を始めてから安定するまでの間、どのような処理が行われていたのかをログを見ながら確認してみましょう。

タスク詳細画面のタブから「ログ」を開くことで、タスクを起動してから現在に至るまでのログを確認できます。

![](https://storage.googleapis.com/zenn-user-upload/f561585b96db-20230818.png)

ログを遡っていくと、`rails/entrypoint.prod.sh`で記述したコマンドが実行されている箇所を発見できます（DB作成→マイグレーション→seedデータ登録→railsサーバー起動）。

![](https://storage.googleapis.com/zenn-user-upload/9f554c78a632-20230818.png)

![](https://storage.googleapis.com/zenn-user-upload/679703fd6ec3-20230818.png)

その後、`/api/v1/health_check`に対してヘルスチェックリクエストが送信され、200レスポンスを返すやりとりが確認できます。このやりとりの成立をもって、railsコンテナは「ヘルスステータス: 正常」と判断されます。

![](https://storage.googleapis.com/zenn-user-upload/a150cd3efd57-20230818.png)

railsのヘルスチェック完了以後、nginxコンテナが起動し始めます（コンテナ起動タイミング自体はログからは確認できません）。起動すると、今後はnginxコンテナに対するヘルスチェックが行われ、これをログから確認することができます。

問題がなければ、nginxコンテナも「ヘルスステータス: 正常」と判断されます。

![](https://storage.googleapis.com/zenn-user-upload/a18daeaca2d5-20230818.png)
