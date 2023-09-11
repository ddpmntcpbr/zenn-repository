## この章でやること

すでに本番環境にリリースされたアプリケーションに対して変更を加えた場合に、再デプロイする方法を説明します

## 流れ

1. ソースコードに変更を追加
2. イメージを再作成し、ECRに再プッシュ
3. タスクを停止させ、その後自動でタスクが再起動するのを待つ

実際にデプロイ済みのbackendアプリに、いくつかの変更を加えていきます。

### 1. ソースコードに変更を追加

ソースコードの変更例として、以下2点を変更してみます。

- (i) entrypoint.prod.sh
- (ii) ヘルスチェックのレスポンスメッセージ

#### (i) entrypoint.prod.sh

`entrypoint.prod.sh`で記載したコマンドのうち、初回デプロイ時以外では実行したくない`rails db:create`コマンドと`rails db:seed`コマンドをコメントアウトしてみます。


```diff sh:rails/entrypoint.prod.sh
#!/bin/bash
set -e

echo "Start entrypoint.prod.sh"

echo "rm -f /myapp/tmp/pids/server.pid"
rm -f /myapp/tmp/pids/server.pid

- echo "bundle exec rails db:create RAILS_ENV=production"
- bundle exec rails db:create RAILS_ENV=production
+ # echo "bundle exec rails db:create RAILS_ENV=production"
+ # bundle exec rails db:create RAILS_ENV=production

echo "bundle exec rails db:migrate RAILS_ENV=production"
bundle exec rails db:migrate RAILS_ENV=production

- echo "bundle exec rails db:seed RAILS_ENV=production"
- bundle exec rails db:seed RAILS_ENV=production
+ # echo "bundle exec rails db:seed RAILS_ENV=production"
+ # bundle exec rails db:seed RAILS_ENV=production

echo "exec pumactl start"
bundle exec pumactl start
```

#### (ii) ヘルスチェックのレスポンスメッセージ

コード変更が反映されたことを分かりやすく確認できるよう、ヘルスチェックのレスポンスメッセージを任意に変更してみます。

今回は、エクスクラメーションマーク`!`を二つに増やしてみました。

```diff ruby:rails/app/controllers/api/v1/health_check_controller.rb
class Api::V1::HealthCheckController < Api::V1::BaseController
  def index
-   render json: { message: "Success Health Check!" }, status: :ok
+   render json: { message: "Success Health Check!!" }, status: :ok
  end
end
```

テストの変更も忘れずに行います。

```diff rails/spec/requests/api/v1/health_checks_spec.rb
require "rails_helper"

RSpec.describe "Api::V1::HealthChecks", type: :request do
  describe "GET api/v1/health_check" do
    subject { get(api_v1_health_check_path) }

    it "正常にレスポンスが返る" do
      subject
      res = JSON.parse(response.body)
-     expect(res["message"]).to eq "Success Health Check!"
+     expect(res["message"]).to eq "Success Health Check!!"
      expect(response).to have_http_status(:success)
    end
  end
end
```

### 2. イメージを再作成し、ECRに再プッシュ

最初にRDSにプッシュしたときと同じ手順で、`rails`イメージを再プッシュします。「Amazon RDS > zenn-clone-rails > プッシュコマンドの表示」にしたがってコマンドを実行していきます。

![](https://storage.googleapis.com/zenn-user-upload/cde62a7637a1-20230526.png)

#### 1. 認証トークンを取得し、レジストリに対して Docker クライアントを認証します。

表示されているコマンドをコピペして実行します。

#### 2. 以下のコマンドを使用して、Docker イメージを構築します。一から Docker ファイルを構築する方法については、「こちらをクリック 」の手順を参照してください。既にイメージが構築されている場合は、このステップをスキップします。

Dockerfileのファイル名および相対パスの位置に合わせてコマンドを修正し、アプリのトップの位置で実行します。

**このとき、キャッシュが残っているとコード変更がイメージに反映されていない可能性があるので --no-cache オプションをつけて build することをオススメします**

```sh:ホストOS(./)
docker build -t zenn-clone-rails -f ./rails/Dockerfile.prod ./rails --no-cache
```

#### 3. 構築が完了したら、このリポジトリにイメージをプッシュできるように、イメージにタグを付けます。

表示されているコマンドをコピペして実行します。

#### 4. 以下のコマンドを実行して、新しく作成した AWS リポジトリにこのイメージをプッシュします

表示されているコマンドをコピペして実行します。

↓

「Amazon RDS > zenn-clone-rails」の`latest`イメージについて、「プッシュされた日時」が、先ほどプッシュを行った日時になっていれば、正常に最新イメージがプッシュできています（過去のイメージは、`-`の名前に変更されてリポジトリ内に残っています）

![](https://storage.googleapis.com/zenn-user-upload/27adb2c8a1e8-20230818.png)

## 3. タスクを停止させ、その後自動でタスクが再起動するのを待つ

現在稼働しているタスクを一度停止します。

zenn-clone-clusterの詳細画面から「タスク」タブを開き、現在稼働中のタスクを停止させてください。

![](https://storage.googleapis.com/zenn-user-upload/e88994e0af53-20230818.png)

↓

タスク停止後、サービスによって新しいタスクが再度作成され、自動的に起動し始めます。

![](https://storage.googleapis.com/zenn-user-upload/991611551099-20230818.png)

↓

新しいタスクの詳細画面に入り、「コンテナ」を確認してください。しばらく時間が経過した後、コード差分に問題がなければ rails コンテナ、 nginx コンテナともにHEALTHYになるはずです。

![](https://storage.googleapis.com/zenn-user-upload/5a3afd85a17c-20230818.png)

↓

ログを確認してみると、`entrypoint.prod.sh`でコメントアウトした「DB作成」と「seedデータ登録」が実行されていないことが確認できます。

![](https://storage.googleapis.com/zenn-user-upload/1946e089156b-20230818.png)

↓

ブラウザからアクセスしてみます。先ほどとは別のタスクとして起動されているので、パブリックIPは変更になっています。

![](https://storage.googleapis.com/zenn-user-upload/594380ca5366-20230818.png)

↓

`http://{Public IP}/api/v1/health_check` にアクセスすると、レスポンスメッセージの変更が反映されていることが確認できます。

![](https://storage.googleapis.com/zenn-user-upload/0076be0af1b7-20230818.png)

:::message
Q. イメージを作り直したにも関わらず、参照イメージを決定しているはずタスク定義を作り直す必要がなかったのはなぜか？

A. 今回使用しているタスク定義では、「リポジトリの最新のイメージを参照する」という設定になっているから。

タスク定義の際、コンテナの設定で参照するイメージのURIをコピーして貼り付けました。URIを見てみると、`xxx/yyy:latest`のようになっています。これは、yyyというリポジトリの最新(latest)のイメージを意味するURIです。

イメージを再プッシュすると、それまでlatestだったイメージは、イメージ名が`-`に変更されるだけでなく、URIも別のものに変更されます。そして新しくプッシュされたイメージのURIとして`xxx/yyy:latest`が使用されることになります。

したがって、URIとしては同じ`xxx/yyy:latest`であったとしても、それが指し示しているイメージは常に最新のものに置き換わっていくので、タスク定義側の設定を変更する必要がなかった、というわけです。
:::
