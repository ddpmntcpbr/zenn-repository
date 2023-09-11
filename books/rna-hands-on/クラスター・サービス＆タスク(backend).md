## クラスターとは？

後述のサービスを起動するためのグループ。

## サービスとは？

## 手順

### クラスターの作成

「Amazon RDS > クラスター」にアクセスし、「クラスターの作成」を押下します。

![](https://storage.googleapis.com/zenn-user-upload/37943d62820f-20230518.png)

↓

クラスターテンプレートとして「ネットワーキングのみ」を選択します。

![](https://storage.googleapis.com/zenn-user-upload/5835f54c5b31-20230518.png)

↓

クラスター名を入力して、「作成」を押下します。

|項目|値|
|---|---|
|クラスター名|zenn-clone-cluster|

![](https://storage.googleapis.com/zenn-user-upload/2dd5ebae895c-20230518.png)

↓

正常にクラスターが作成されればOKです。

![](https://storage.googleapis.com/zenn-user-upload/9473d9f1241b-20230518.png)

### サービス(backend)の作成

先ほど作成したクラスターの詳細画面で「サービス」タブを開き、「作成」を押下します。

![](https://storage.googleapis.com/zenn-user-upload/2aff349c6c7c-20230518.png)

↓

以下項目を入力して、「次のステップ」を押下します。

|項目|値|
|---|---|
|起動タイプ|FARGATE|
|タスク定義 > ファミリー|zenn-clone-task-backend|
|タスク定義 > リビジョン|latest|
|クラスター|zenn-clone-cluster|
|サービス名|zenn-clone-backend-service|
|タスク数|1|

![](https://storage.googleapis.com/zenn-user-upload/75442b766a8b-20230518.png)

![](https://storage.googleapis.com/zenn-user-upload/97c351695f2c-20230518.png)

↓

ネットワーク構成を設定していきます。以下項目を入力して、「次のステップ」を押下します。

|項目|値|
|---|---|
|クラスター VPC|zenn-clone-vpc（10.0.0.0/16）|
|サブネット|zenn-clone-public-subnet1, zenn-clone-public-subnet2|
|セキュリティーグループ|編集 > 既存のセキュリティグループ > zenn-clone-ecs-backend-security-groupを選択|

![](https://storage.googleapis.com/zenn-user-upload/3f77f035298f-20230518.png)

![](https://storage.googleapis.com/zenn-user-upload/7aaab9d6f061-20230520.png)

ロードバランシングはいったん「なし」にします。

![](https://storage.googleapis.com/zenn-user-upload/f2ffbde89010-20230518.png)

↓

Auto Scaling はデフォルトでOKです。

![](https://storage.googleapis.com/zenn-user-upload/9388ef2a3fc2-20230518.png)

↓

確認画面最下部「サービスの作成」を押下し、サービスを作成します。

↓

「クラスター  zenn-clone-cluster」にアクセスすると、先ほどのサービスが起動していることが確認できます。

![](https://storage.googleapis.com/zenn-user-upload/faada468f171-20230518.png)

↓

サービスの詳細に入ると、サービスで設定したタスクが起動中であることを確認できます。

![](https://storage.googleapis.com/zenn-user-upload/3b8709a1fb44-20230518.png)

タスクの詳細の入ると、タスクで定義している`rails`コンテナ、`nginx`コンテナがそれぞれ起動中であることが分かります。タスクのステータスは、

- PROVISIONING
- PENDING
- RUNNING

の順で移行し、設定に問題がなければ自動的にRUNNINGまで移行します。また、ヘルスチェックステータスもHEALTHYになります。

![](https://storage.googleapis.com/zenn-user-upload/689720560995-20230521.png)

↓


#### ※補足:タスクが正常に起動しない場合

ここに至るまでに何らかの設定を誤ってしまっている可能性が高いです。原因が探るヒントを得るために、以下の2点を確認ください。

1. 各コンテナのログを確認する

各コンテナの起動開始〜ヘルスチェック完了までの間でエラーが発生した場合は、コンテナのログに何らかのエラーメッセージが表示されている可能性があります。

タスク詳細画面の「ログ」から、各コンテナのログを確認できます。

![](https://storage.googleapis.com/zenn-user-upload/611fe684525b-20230521.png)

例えば、`rails`コンテナのログからDB接続に失敗していることが特定できれば、DB周りの設定をひとつずつ確認していきましょう。具体的には、

- `rails`コンテナの環境変数で設定した`DATABASR_URL`が誤っている。
- `rails/database.yml`の`production`の設定は誤っている。
- DBに設定したセキュリティグループが誤っている。

などが原因の候補として考えられそうです。

2. サービスからエラーメッセージを確認

各コンテナのログが一切表示されていない場合は、そもそもコンテナの起動開始のタイミングでエラーになっていると考えられます。

この場合は、サービスの詳細画面の「タスク」タブ > タスクのステータスとして`Stopped`を選択することで、停止したコンテナの状況を確認できます。

（今回はエラーになっていないので、何も表示されていません）
![](https://storage.googleapis.com/zenn-user-upload/291765a1a1b9-20230526.png)

「前回のステータス」において、`STOPPED(error message)`のような形で、何らかのエラーメッセージが表示されている場合があります。このエラーメッセージを起点を調査することで、エラー原因を特定できるかもしれません。

#### ※補足:タスクが正常に起動しない場合ここまで

↓

コンテナが起動するまでの間で裏側で行われていたことを確認してみましょう。タスク詳細画面の「ログ」から、各コンテナのログを確認できます。

![](https://storage.googleapis.com/zenn-user-upload/d9545c0626cc-20230521.png)

↓

`rails`コンテナのログを下の方（時系列の早い方）から見てみると、`entrypoint.prod.sh`で記載したコマンドが実行されていることが確認できるはずです（`echo`コマンドでログ上に文字列を出力しています）。

![](https://storage.googleapis.com/zenn-user-upload/693a90463d6b-20230521.png)

↓

`entrypoint.prod.sh`が終了した後（pumaサーバーが起動した後）、タスク定義で設定したヘルスチェックコマンドが実行され、コンテナが正常に立ち上がっていることをチェックしている様子が確認できます（ヘルスチェックコマンドは何回か繰り返し実行されます）。

![](https://storage.googleapis.com/zenn-user-upload/731c5f81d0e6-20230521.png)

↓

`rails`コンテナのヘルスチェックが正常に完了した時点で、ヘルスチェックステータスがHEALTYに切り替わり、これに基づいて`nginx`コンテナが起動し始めます（タスク定義で、`nginx`コンテナの起動条件として、`rails`コンテナがHEALTYであること、を設定していたため）

`nginx`コンテナのログを見てみると、タスク定義で設定したヘルスチェックコマンドが実行されている様子が確認できます。こちらもヘルスチェックが完了した時点で、ヘルスチェックステータスがHEALTYに切り替わります。

![](https://storage.googleapis.com/zenn-user-upload/46b45a45fa92-20230521.png)

### 実際にアクセスしてみる

`rails`コンテナ、`nginx`コンテナが両方HEALTYとなっている時点で、このアプリケーションはデプロイが完了していることになります。

実際にブラウザからアクセスをして、インターネット上で公開されていることを確認してみましょう。タスクの詳細画面から`パブリック IP`を取得してください。

![](https://storage.googleapis.com/zenn-user-upload/4eefdbed30fd-20230526.png)

`nginx`コンテナのヘルスチェックコマンドと同じ、 http://{Public IP}/api/v1/health_check にブラウザからアクセスしてみると`{"message":"Success Health Check!"}`が返ってくることを確認できます。

![](https://storage.googleapis.com/zenn-user-upload/107d8962e5b5-20230526.png)
