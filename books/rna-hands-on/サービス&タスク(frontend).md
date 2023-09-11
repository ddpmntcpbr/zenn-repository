## この章やること

frontend用タスクを起動するサービスを作成します。

## 手順

### サービス(frontend)の作成

「ECS > クラスター > zenn-clone-cluster」画面から「サービス」タブを開き、「作成」を押下します。

![](https://storage.googleapis.com/zenn-user-upload/603f8cb18421-20230819.png)

↓

必要事項を入力していきます。

#### 環境

|項目|値|
|---|---|
|コンピューティングオプション|起動タイプ|
|起動タイプ|FARGATE|
|プラットフォームバージョン|LATEST|

![](https://storage.googleapis.com/zenn-user-upload/fb6261bf0b6c-20230819.png)

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

ここで、ECS(frontend)用に作成したセキュリティグループを使用します。

![](https://storage.googleapis.com/zenn-user-upload/3db90481d97e-20230819.png)

![](https://storage.googleapis.com/zenn-user-upload/50be107c936e-20230819.png)


#### ロードバランシング

後ほどfrontendにおいてもALBを設置しますが、いったんは無しでOKです。

![](https://storage.googleapis.com/zenn-user-upload/f975c0f26c5e-20230818.png)

↓

入力が完了したら、「作成」ボタンをクリックしてください。サービスが作成されたら、サービスの詳細画面からタスクの詳細画面に入り、画面下部のコンテナを確認してください。

ステータスがRUNNINGになったら、ブラウザからパブリックIPにアクセスしてみてください（`http://{{ パブリックIP }}`）。

![](https://storage.googleapis.com/zenn-user-upload/cbfb3b5b5b29-20230822.png)

エラー画面が表示されるはずです。エラーになっているのは、Rails側との通信が確立できていないためです。

![](https://storage.googleapis.com/zenn-user-upload/4773ef0f0005-20230822.png)

手動でヘルスチェックAPIにアクセスしてみましょう。`http://{{ パブリックIP }}/api/health_check`に正常にアクセスできればOKです！

![](https://storage.googleapis.com/zenn-user-upload/ea93aef2ba82-20230822.png)
