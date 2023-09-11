## やること

先ほど作成したALBをECS(backend)に付与します。

## 手順

「ECS > クラスター > zenn-clone-cluster」にアクセスし、稼働している`zenn-clone-backend-service`サービスを削除します。

![](https://storage.googleapis.com/zenn-user-upload/22c4132142cf-20230527.png)

↓

削除が完了したら、再度「作成」を行います。

基本的な設定は先ほど削除したサービスと同様ですが、ALBに関する設定のみ追加があります。念のため、先ほどと同じ設定情報も再掲します。

#### ステップ1：サービスの設定

ここは変更なしです。

|項目|値|
|---|---|
|起動タイプ|FARGATE|
|タスク定義 > ファミリー|zenn-clone-task-backend|
|タスク定義 > リビジョン|latest|
|クラスター|zenn-clone-cluster|
|サービス名|zenn-clone-backend-service|
|タスク数|1|

#### ステップ2：サービスの設定

「VPC とセキュリティグループ」は変更なしです。

|項目|値|
|---|---|
|クラスター VPC|zenn-clone-vpc（10.0.0.0/16）|
|サブネット|zenn-clone-public-subnet1, zenn-clone-public-subnet2|
|セキュリティーグループ|編集 > 既存のセキュリティグループ > zenn-clone-ecs-backend-security-groupを選択|

↓

「ロードバランシング」から、新しい設定が加わります。

|項目|値|
|---|---|
|ロードバランサーの種類|Application Load Balancer|
|サービス用の IAM ロールの選択 > ロードバランサー名|zenn-clone-alb-backend|

![](https://storage.googleapis.com/zenn-user-upload/94b273679516-20230527.png)

↓

「ロードバランス用のコンテナ」で、「コンテナ名: ポート > nginx:80:80」を選択して「ロードバランサーを追加」を押下し、その後設定を記述していきます。

|項目|値|
|---|---|
|プロダクションリスナーポート|HTTPS|
|ターゲットグループ名|新規作成 > zenn-clone-backend-service-tg|
|ターゲットグループのプロトコル|HTTP|
|パスパターン|/|
|評価順|1|
|ヘルスチェックパス|/api/v1/health_check|

![](https://storage.googleapis.com/zenn-user-upload/1f8feed6a232-20230527.png)

![](https://storage.googleapis.com/zenn-user-upload/475d592436b0-20230527.png)

#### ステップ3：Auto Scaling (オプション)

デフォルトでOK。

![](https://storage.googleapis.com/zenn-user-upload/53ea8e821c2b-20230527.png)

#### ステップ4：確認

「サービスの作成」を押下し、サービスを作成します。

![](https://storage.googleapis.com/zenn-user-upload/963d235716bb-20230527.png)

↓

設定に問題なければ、自動的にタスクが起動し、中の`rails`、`nginx`コンテナがHEALTHYまで遷移します。

![](https://storage.googleapis.com/zenn-user-upload/e7d5e4fe6a37-20230527.png)

↓

いったんこれまでと同様に、パブリックIP直指定でブラウザから`nginx`コンテナにアクセスできることを確認しておきます。

![](https://storage.googleapis.com/zenn-user-upload/1a9af2b0b71b-20230527.png)

![](https://storage.googleapis.com/zenn-user-upload/56630bd39044-20230527.png)

## ALBのターゲットグループの修正

続いて、ALBからECSへの疎通を行います。

サービス作成時にALB接続は行いましたが、今の状態ではまだ、ALBからECSへの通信は成立できていません。理由は、ALBのリスナールールにおけるHTTP/HTTPSトラフィックの転送先が`zenn-clone-alb-backend-tg`（ALB作成時に適当に作ったターゲットグループ）のままになっているためです。

これをサービス作成時に作成した`zenn-clone-backend-service-tg`に変更してあげることで、
ALBとECSの疎通が成立します。

「EC2 > ロードバランサー」から、`zenn-clone-alb-backend`をクリックし、画面下部「リスナー」タブを開きます。

![](https://storage.googleapis.com/zenn-user-upload/a6c78a47c4f2-20230527.png)

↓

「HTTP : 80」にチェックを入れ、「編集」を押下します。

![](https://storage.googleapis.com/zenn-user-upload/5c703318cfc8-20230527.png)

↓

デフォルトアクションとして設定されている「1.転送先」の左側の鉛筆マークをクリックして編集モードにし、ターゲットグループを`zenn-clone-backend-service-tg`に変更して「更新」を押下します。

![](https://storage.googleapis.com/zenn-user-upload/af30915b7f06-20230527.png)

↓

画面左上の`<`から前画面に戻り、「HTTPS : 443」にチェックを入れ、「編集」を押下します。

![](https://storage.googleapis.com/zenn-user-upload/ec5704c095a6-20230527.png)

↓

デフォルトアクションとして設定されている「1.転送先」の左側の鉛筆マークをクリックして編集モードにし、ターゲットグループを`zenn-clone-backend-service-tg`に変更して「更新」を押下します。

![](https://storage.googleapis.com/zenn-user-upload/b33b4701e4b7-20230527.png)

↓


これでターゲットグループの設定が完了しました。ALBとESCの疎通を確認してみます。画面左上の`<`から前画面に戻り、「説明」タブからDNS名を取得します。

![](https://storage.googleapis.com/zenn-user-upload/19c40967745e-20230527.png)

↓

ブラウザから`http://{DNS Name}/api/v1/health_check`にアクセスし、ヘルスチェックメッセージが返ってきたらOKです！

![](https://storage.googleapis.com/zenn-user-upload/5d15a97486af-20230527.png)

### 独自ドメイン(SSL)を設定

Route53で取得した独自ドメインをALBに設定することで、独自ドメインからアプリケーションにアクセスできるようにします。

「Route53 > ホストゾーン」から、先ほど取得したドメイン（`rails-next-zenn-clone-app.com`）の詳細画面に入り、「レコードを作成」を押下します。

![](https://storage.googleapis.com/zenn-user-upload/c89d9695adb2-20230528.png)

↓

もし、「レコードをクイック作成」画面が表示された場合は、「ウィザードに切り替える」をクリックします。

![](https://storage.googleapis.com/zenn-user-upload/4bee67b30e3a-20230528.png)

↓

ルーティングポリシーとして「シンプルルーティング」を選択して「次へ」を押下します。

![](https://storage.googleapis.com/zenn-user-upload/a0df75bd464c-20230528.png)

↓

「レコードを定義」を押下し、モーダルに以下を設定して「シンプルなレコードを定義」を押下します（サブドメイン`backend.rails-next-zenn-clone-app.com`を定義しています）。

|項目|値|
|---|---|
|レコード名|backend|
|レコードタイプ|A - IPv4アドレスと~|
|値/トラフィックのルーティング|Application Load Balancer と Classic Load Balancer へのエイリアス|
||アジアパシフィック(東京) ap-northeast-1|
||backend用ALB (dualstack.zenn-clone-alb-backend-xxx) を選択|

![](https://storage.googleapis.com/zenn-user-upload/06ce976a937c-20230528.png)

↓

「レコードを作成」を押下し、正常にレコードが作成されたことを確認します。これで、ALBへの独自ドメインの設定が完了しました！

![](https://storage.googleapis.com/zenn-user-upload/4c6e9b0580cb-20230528.png)

↓

ブラウザから、`https://backend.rails-next-zenn-clone-app.com/api/v1/health_check`にアクセスすると、ヘルスチェックが返ってくることが確認できます。SSLを用いたアクセスが成立できるようになりました！

![](https://storage.googleapis.com/zenn-user-upload/1b40592dba46-20230528.png)
