## この章でやること

backend ECS用のタスク定義を行います。

## タスク定義とは?

ECSにおけるコンテナ実行の設定内容を定義するものです。

ECRはコンテナを置いておく場所、ECSはコンテナを持ってきて起動する場所です。この、「コンテナを持ってきて起動する」の具体的な方法を記述するのがタスク定義です。

1~複数個のコンテナの起動方法を設定するものであるので、開発環境における**docker-compose.yml**に相当するもの、考えると分かりやすいと思います。

## IAMロールの作成

タスク定義の事前準備として、ECSタスクを実行するIAMロールの作成を行います。

検索窓からIAMにアクセスし、サイドバー「アクセス管理 > ロール」にアクセスしてください。

![](https://storage.googleapis.com/zenn-user-upload/9c1eaebd5208-20230817.png)

「ロールを作成」ボタンをクリックし、以下の通り入力して「次へ」ボタンをクリックしてください。

|項目|値|
|---|---|
|信頼されたエンティティタイプ|AWSのサービス|
|ユースケース|（プルダウンフォームから）Elastic Container Service|
|択一選択肢|Elastic Container Service Task|

![](https://storage.googleapis.com/zenn-user-upload/0dfa9133601e-20230817.png)

↓

検索フォームを用いて、以下ふたつの許可ポリシーを検索してチェックボックスにチェックを入れ、「次へ」ボタンを押してください。

- **AmazonECSTaskExecutionRolePolicy**
  - ECSタスクを実行する権限
- **CloudWatchAgentServerPolicy**
  - CloudWatchでサーバーログを閲覧する権限

![](https://storage.googleapis.com/zenn-user-upload/d0e96e0976c4-20230817.png)

![](https://storage.googleapis.com/zenn-user-upload/10e188cf3226-20230817.png)

↓

「名前、確認、および作成」に進みますので、以下を入力して「ロールを作成」をクリックしてください。

|項目|値|
|---|---|
|ロール名|ecsTaskExecutionRole|
|説明|任意でOK|

![](https://storage.googleapis.com/zenn-user-upload/9a748cfe1eee-20230817.png)

![](https://storage.googleapis.com/zenn-user-upload/074ffeb2529c-20230817.png)

↓

ロール`ecsTaskExecutionRole`が作成されたらOKです！

![](https://storage.googleapis.com/zenn-user-upload/c4099c4023fd-20230817.png)

## タスク定義

IAMロールの作成が完了したら、タスク定義を始めていきます。

検索窓から「Elastic Container Service」にアクセスし、サイドバーから「タスク定義」にアクセスしてください。

![](https://storage.googleapis.com/zenn-user-upload/d7f60ee9a4a9-20230817.png)

↓

「新しいタスク定義の作成 > 新しいタスク定義の作成」を押下すると、タスク定義が開始します。

### 基本設定

|項目|値|
|---|---|
|タスク定義名|zenn-clone-task-definition-backend|
|起動タイプ|AWS Fargate|
|オペレーティングシステム/アーキテクチャ|Linux/x86_64|
|CPU|.25vCPU|
|メモリ|.5GB|
|タスクロール|ecsTaskExecutionRole|
|タスク実行ロール|ecsTaskExecutionRole|

![](https://storage.googleapis.com/zenn-user-upload/34677487ba89-20230817.png)

![](https://storage.googleapis.com/zenn-user-upload/66a44ecded5c-20230817.png)


### コンテナ - 1

rails用のコンテナの定義を行います。

#### 基本設定

|項目|値|
|---|---|
|名前|rails|
|イメージ|※補足1|
|必須コンテナ|はい|
|ポートマッピング > コンテナポート|3000|
|CPU|0|

![](https://storage.googleapis.com/zenn-user-upload/9a4feae4c328-20230817.png)

![](https://storage.googleapis.com/zenn-user-upload/f9ffb66d9caf-20230817.png)

※補足1: イメージは「Amazon ECR > リポジトリ」の`zenn-clone-rails`のURIをコピペして貼り付けてください。

![](https://storage.googleapis.com/zenn-user-upload/e99bbb046cc4-20230518.png)

#### 環境変数

|キー|バリュー|
|---|---|
|RAILS_LOG_TO_STDOUT|true|
|RAILS_MASTER_KEY|`rails/config/master.key`に記載の文字列を入力|

![](https://storage.googleapis.com/zenn-user-upload/616f28d82326-20230817.png)

#### ログ記録

ログ収集の使用にチェックを入れてください。関連の設定は、デフォルトで以下のような設定になっているはずで、特に変更する必要はありません。

|項目|値|
|---|---|
|awslogs-group|/ecs/zenn-clone-task-definition-backend|
|awslogs-region|ap-northeast-1|
|awslogs-stream-prefix|ecs|
|awslogs-create-group|true|

![](https://storage.googleapis.com/zenn-user-upload/9d0ec405c833-20230817.png)

#### ヘルスチェック

|項目|値|
|---|---|
|コマンド|CMD-SHELL, curl --unix-socket /myapp/tmp/sockets/puma.sock localhost/api/v1/health_check || exit 1|

![](https://storage.googleapis.com/zenn-user-upload/7b4de7a5b29e-20230817.png)

### コンテナ - 2

「+ 他のコンテナの追加」をクリックして、nginxコンテナの設定を行なっていきます。

![](https://storage.googleapis.com/zenn-user-upload/f144a73a2838-20230817.png)

#### 基本設定

|項目|値|
|---|---|
|名前|nginx|
|イメージ|RDS > zenn-clone-nginx の URI をコピー|
|必須コンテナ|はい|
|ポートマッピング > コンテナポート|80|
|CPU|0|

![](https://storage.googleapis.com/zenn-user-upload/cba484dac19e-20230817.png)

![](https://storage.googleapis.com/zenn-user-upload/d2eda1b80e38-20230817.png)

#### 環境変数

nginxコンテナでは設定不要です。

#### ログ記録

railsコンテナと同様に、ログ収集の使用にチェックを入れてください。関連の設定は、デフォルトで以下のような設定になっているはずで、特に変更する必要はありません。

|項目|値|
|---|---|
|awslogs-group|/ecs/zenn-clone-task-definition-backend|
|awslogs-region|ap-northeast-1|
|awslogs-stream-prefix|ecs|
|awslogs-create-group|true|

![](https://storage.googleapis.com/zenn-user-upload/9d0ec405c833-20230817.png)

#### ヘルスチェック

|項目|値|
|---|---|
|コマンド|CMD-SHELL, curl -f http://localhost/api/v1/health_check || exit 1|

![](https://storage.googleapis.com/zenn-user-upload/7df6e826114a-20230817.png)

#### スタートアップの依存関係の順序

railsコンテナの起動が完了してから、nginxコンテナを起動させるように設定します。s

|項目|値|
|---|---|
|コンテナ名|rails|
|条件|Healthy|

![](https://storage.googleapis.com/zenn-user-upload/77b37f4fe334-20230817.png)

### ストレージ

ボリュームソースの設定をします。

|項目|値|
|---|---|
|コンテナ|nginx|
|ソースコンテナ|rails|

![](https://storage.googleapis.com/zenn-user-upload/e154b771e969-20230817.png)


↓

以上設定が完了したら、画面最下部の「作成」ボタンをクリックします。正常にタスクが作成されればOKです！

![](https://storage.googleapis.com/zenn-user-upload/5708972d559b-20230817.png)
