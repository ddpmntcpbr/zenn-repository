---
title: "はじめに「未経験転職は難しい。だからこそ」"
---

## この章でやること

frontend用ECSで使用するタスク定義を作成します。

## 手順

検索窓から「Elastic Container Container > タスク定義」に入り、「新しいタスク定義の作成」を押下します。

![](https://storage.googleapis.com/zenn-user-upload/166c6325b53a-20230819.png)

↓

### 基本設定

|項目|値|
|---|---|
|タスク定義名|zenn-clone-task-definition-frontend|
|起動タイプ|AWS Fargate|
|オペレーティングシステム/アーキテクチャ|Linux/x86_64|
|CPU|.25vCPU|
|メモリ|.5GB|
|タスクロール|ecsTaskExecutionRole|
|タスク実行ロール|ecsTaskExecutionRole|


![](https://storage.googleapis.com/zenn-user-upload/cddb133e5d97-20230819.png)

![](https://storage.googleapis.com/zenn-user-upload/523989cb473e-20230819.png)

### コンテナ - 1

next用のコンテナ1つだけを用意しmす。

|項目|値|
|---|---|
|名前|next|
|イメージ|RDS > zenn-clone-next の latest の URI|
|必須コンテナ|はい|
|ポートマッピング > コンテナポート|８０|
|CPU|0|

![](https://storage.googleapis.com/zenn-user-upload/248c9c8427e5-20230822.png)

![](https://storage.googleapis.com/zenn-user-upload/fe201dbbe746-20230822.png)

#### 環境変数

特に設定は不要です。

本番環境で使用する環境変数は、`.env.production`をビルドしたタイミングでコンテナ内で有効になっています。

![](https://storage.googleapis.com/zenn-user-upload/472aaecc1c46-20230819.png)

#### ログ記録

ログ収集の使用にチェックを入れてください。関連の設定は、デフォルトで以下のような設定になっているはずで、特に変更する必要はありません。

|項目|値|
|---|---|
|awslogs-group|/ecs/zenn-clone-task-definition-frontend|
|awslogs-region|ap-northeast-1|
|awslogs-stream-prefix|ecs|
|awslogs-create-group|true|

![](https://storage.googleapis.com/zenn-user-upload/104cd6ad02ab-20230819.png)

#### ヘルスチェック

今回は設定なしでOKです（このあと設定するALB側のヘルスチェック機能の方を活用します）。

![](https://storage.googleapis.com/zenn-user-upload/92e61a13e910-20230822.png)

↓

以上設定が完了したら、画面最下部の「作成」ボタンをクリックします。正常にタスクが作成されればOKです！

![](https://storage.googleapis.com/zenn-user-upload/e53b276ff0ab-20230819.png)
