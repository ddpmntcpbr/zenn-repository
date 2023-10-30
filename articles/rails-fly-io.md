---
title: "【初心者向け】Rails アプリを完全無料で公開しよう！ Fly.io デプロイ手順"
emoji: "🐑"
type: "tech"
topics: ["rails", "postgresql", "初心者", "初心者向け"]
published: false
---

## この記事は？

当記事を執筆している2023年10月現在において、ローカルで開発した Ruby on Rails アプリを**完全無料**でデプロイできるサービスである`Fly.io`の利用手順を整理したものです。

## 動機

以前までは、Railsアプリの無料デプロイ先としては [Heroku](https://jp.heroku.com/) が主流でした。

しかし、2022年11月より Heroku の無料プランが廃止となり、最安のEcoプランでも**月5ドル**の費用が発生するようになってしまいました（2023年10月時点）。

https://jp.heroku.com/pricing

企業等で本格的に運用するアプリであれば運用コストは飲み込めるかもしれませんが、特に個人開発のアプリの場合、どうしてもコストを無料に抑えたいケースもあると思います。

- **転職用のポートフォリオ**として公開するので、高いパフォーマンスは必要ない。
- 将来的には有料運用は想定しているが、最初の**市場検証フェーズ**では無料から始めたい。
- **単なる趣味アプリ**でしかないので、お金がかかるなら公開はできない。

そこで当記事では、 Heroku の無料プランに代わる手段として、**Fly.io**による無料デプロイ方法を解説していきます。

## Fly.io とは？

Webアプリケーションの公開を含めた様々なサービスを提供する PaaS です。当記事では、**Herokuの代替サービス**くらいの粒度で認識しておいてもらえればと思います。

https://fly.io/

[Fly.ioの料金ページ](https://fly.io/docs/about/pricing/#free-allowances)を確認すると、Freeプランでも以下の機能を利用できると記載されてます。

## 宣伝

zenn上に、**Rails × Next.js × AWS アプリの開発チュートリアル本**をリリースしています。

https://zenn.dev/ddpmntcpbr/books/rna-hands-on

もし、あなたが「**転職用ポートフォリオとしての Rails アプリを無料デプロイする方法を知りたい**」という動機で当記事に辿り着いた場合、こちらの書籍でワンランク上のポートフォリオ開発に挑戦してみることをぜひ検討してもらえたらと思います。**Next.js、AWSに関する予備知識なしでも取り組める内容になっています**。

また、以降の当記事で紹介する方法は、 API モードの Rails アプリおいても同じようにデプロイ可能です。さらにフロントエンドに Next.js を採用している場合、 [Vercel](https://vercel.com/) の無料プランを活用すれば、**Rails(Fly.io) × Next.js(Vercel) の構成も完全無料でデプロイ可能です**。

## おことわり

- 当記事の情報は、2023年10月時点のものとなっております。将来のプラン改定によっては同じ手法でデプロイを行った場合にも料金が発生してしまう可能性もあるため、**必ず最新の公式料金ページをご確認の上で作業ください**。
- PlanetScale でデータベースを作成する際、**クレジットカード**または**デビットカード**の登録が必要になります。

## 手順

### 流れ

1. Rails アプリの実装
2. PlanetScale でデータベースを作成
3. Render.com に Rails アプリをデプロイ

1で、デプロイするための簡易なRailsアプリを実装します。**docker/docker-compose** を用いて開発環境を構築する前提で話を進めていきますが、ローカル直下での開発でも問題はありません。

|項目|バージョン|
|---|---|
|Ruby|3.1.2|
|rails|7.0.4|

開発アプリは以下のリポジトリからも確認できます。

https://github.com/ddpmntcpbr/rails-render-planetscale-app

また、すでに何らかの Rails アプリを用意している場合は、2から読み進めるようにしてください。

### 1. Railsアプリの実装

任意の新規ディレクトリを作成し、直下に以下のファイルを新規作成してください。

```:ディレクトリ構造
.
├── Dockerfile
├── Gemfile
├── Gemfile.lock
├── docker-compose.yml
└── entrypoint.sh
```

```dockerfile:rails/Dockerfile
FROM ruby:3.1.2
RUN apt-get update -qq && apt-get install -y vim

RUN mkdir /myapp
WORKDIR /myapp
COPY Gemfile /myapp/Gemfile
COPY Gemfile.lock /myapp/Gemfile.lock

RUN gem update --system
RUN bundle update --bundler

RUN bundle install
COPY . /myapp

COPY entrypoint.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]
```

```yml:./docker-compose.yml
version: '3'
services:
  db:
    image: postgres
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_volume:/var/lib/postgresql/data
  web:
    build: .
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
    volumes:
      - .:/myapp
    ports:
      - "3000:3000"
    tty: true
    stdin_open: true
    depends_on:
      - db
volumes:
  postgres_volume:
```

```ruby:rails/Gemfile
source "https://rubygems.org"
gem "rails", "~> 7.0.4"
```

```ruby:rails/Gemfile.lock
(空のファイルを作成)
```

```sh:rails/entrypoint.sh
#!/bin/bash
set -e

rm -f /myapp/tmp/pids/server.pid

exec "$@"
```

`docker-compose.yml`で記載の通り、development環境では、ローカル上に構築した mysql を DB として使用します。

↓

以下コマンドで、Railsアプリを新規作成してください。

```sh:ターミナル
docker compose run --rm web rails new . --force --database=postgresql
```

```sh:ディレクトリ構造
.
├── Dockerfile
├── Gemfile
├── Gemfile.lock
├── README.md
├── Rakefile
├── app
├── bin
├── config
├── config.ru
├── db
├── docker-compose.yml
├── entrypoint.sh
├── lib
├── log
├── public
├── storage
├── test
├── tmp
└── vendor
```

↓

webコンテナの Rails から dbコンテナの mysql にアクセスするため、`config/database.yml`を以下のように書き換えてください。

```yml:config/database.yml
default: &default
  adapter: postgresql
  encoding: unicode
  host: db
  username: postgres
  password: password
  pool: 5

development:
  <<: *default
  database: myapp_development
```

↓

設定が完了したら docker を起動します。

```sh:ターミナル
docker compose build
```

```sh:ターミナル
docker compose up -d
```

↓

webコンテナ内に入り、DBを作成してください。

```sh:ターミナル
docker compose exec web /bin/bash
```

```sh:webコンテナ
web rails db:create
```

http://localhost:3000 で Rails にアクセスできることを確認ください。

![](https://storage.googleapis.com/zenn-user-upload/09c62e8cd95f-20231003.png)

↓

動作確認のための簡単な機能として、**Post**モデルに対する CRUD を実装します。

webコンテナ内で scaffold による機能一括作成を行ってください。

```sh:webコンテナ
rails g scaffold Post body:text
```

```sh:webコンテナ
rails db:migrate
```

↓

http://localhost:3000/posts で posts 一覧画面にアクセスし、各種CRUD操作（作成、読込、更新、削除）を画面上から行えることを確認してください。

以上で、アプリ実装は完了になります。

![](https://storage.googleapis.com/zenn-user-upload/76643c6993cd-20231005.png)

↓

アプリ実装が完了したら、お使いの GitHub アカウントでリポジトリを作成し、ソースコードのプッシュまで行ってください。リポジトリの公開範囲は、パブリック／プライベートのどちらでも構いません。

![](https://storage.googleapis.com/zenn-user-upload/234738eff553-20231011.png)

### Fly.ioにデプロイ

Fly.ioのトップページにアクセスします。

https://fly.io/

![](https://storage.googleapis.com/zenn-user-upload/1a1e2b80bfd7-20231011.png)

↓

「Sign Up」からサインアップを行なってください。

![](https://storage.googleapis.com/zenn-user-upload/0e5d93bab76e-20231011.png)

↓

サインアップが完了するとダッシュボードにアクセスできるようになりますので、「**Add a payment method**」からクレジッドカード情報を入力してください。

![](https://storage.googleapis.com/zenn-user-upload/dd8bff73856b-20231011.png)

↓

flyctlをインストールします。

```sh:ターミナル
brew install flyctl
```

↓

ターミナルから Fly.io へログインします。下記コマンド実行後、ブラウザが開きますので、ログイン操作を行ってください。

```sh:ターミナル
fly auth login
```

![](https://storage.googleapis.com/zenn-user-upload/e897c58a7748-20231011.png)

↓

```sh:ターミナル
fly launch --dockerfile Dockerfile
```

アプリ設定をインタラクティブに行うモードに入りますので、以下を参考に設定を行なってください。

```sh:
# アプリ名を任意で入力。
? Choose an app name (leave blank to generate one):
> rails-fly-io-app

# デプロイ先のリージョンを選択。
? Choose a region for deployment:
> Tokyo, Japan (nrt)

# DB として Postgresql を使用するかを選択。
? Would you like to set up a Postgresql database now? (y/N)
> y

# Postgresql の利用プランを選択。無料利用したい場合は Development を選択。
? Select configuration:
> Development - Single node, 1x shared CPU, 256MB RAM, 1GB disk

# リソースの使用を停止し、ノードをシャットダウンするかを選択。無料プランでは No。
? Scale single node pg to zero after one hour? (y/N)
> n

# Redis サーバーを利用するかを選択。今回は No。
? Would you like to set up an Upstash Redis database now? (y/N)
> n

# .dockerignore を作成するかを選択。今回は No。
? Create .dockerignore from 1 .gitignore files? (y/N)
> n

# 今すぐデプロイを行うかを選択。今回は Yes。
? Would you like to deploy now? (y/N)
> y
```








### 3. Render.com に Rails アプリをデプロイ

Render.com に Rails アプリをデプロイします。以下の公式チュートリアルを参考にしながら、今回のケースに沿うように少し改変しながら進めていきます。

https://render.com/docs/deploy-rails

#### Raisの修正

Render.com 上でサービスを作成する前に、Railsにいくつかの修正を加えます。

`config/puma.rb`を開き、以下2箇所のコメントアウトを解除してください。

```diff rb:config/puma.rb
  # Puma can serve each request in a thread from an internal thread pool.
  # The `threads` method setting takes two numbers: a minimum and maximum.
  # Any libraries that use thread pools should be configured to match
  # the maximum value specified for Puma. Default is set to 5 threads for minimum
  # and maximum; this matches the default thread size of Active Record.
  #
  max_threads_count = ENV.fetch("RAILS_MAX_THREADS") { 5 }
  min_threads_count = ENV.fetch("RAILS_MIN_THREADS") { max_threads_count }
  threads min_threads_count, max_threads_count

  # Specifies the `worker_timeout` threshold that Puma will use to wait before
  # terminating a worker in development environments.
  #
  worker_timeout 3600 if ENV.fetch("RAILS_ENV", "development") == "development"

  # Specifies the `port` that Puma will listen on to receive requests; default is 3000.
  #
  port ENV.fetch("PORT") { 3000 }

  # Specifies the `environment` that Puma will run in.
  #
  environment ENV.fetch("RAILS_ENV") { "development" }

  # Specifies the `pidfile` that Puma will use.
  pidfile ENV.fetch("PIDFILE") { "tmp/pids/server.pid" }

  # Specifies the number of `workers` to boot in clustered mode.
  # Workers are forked web server processes. If using threads and workers together
  # the concurrency of the application would be max `threads` * `workers`.
  # Workers do not work on JRuby or Windows (both of which do not support
  # processes).
  #
- # workers ENV.fetch("WEB_CONCURRENCY") { 2 }
+ workers ENV.fetch("WEB_CONCURRENCY") { 2 }

  # Use the `preload_app!` method when specifying a `workers` number.
  # This directive tells Puma to first boot the application and load code
  # before forking the application. This takes advantage of Copy On Write
  # process behavior so workers use less memory.
  #
- # preload_app!
+ preload_app!

  # Allow puma to be restarted by `bin/rails restart` command.
  plugin :tmp_restart
```

↓

`config/environments/production.rb`を以下のように修正してください。

```diff rb:config/environments/production.rb
  require "active_support/core_ext/integer/time"

  Rails.application.configure do
    .
    .
    # Disable serving static files from the `/public` folder by default since
    # Apache or NGINX already handles this.
-   config.public_file_server.enabled = ENV['RAILS_SERVE_STATIC_FILES'].present?
+   config.public_file_server.enabled = ENV['RAILS_SERVE_STATIC_FILES'].present? || ENV['RENDER'].present?
    .
    .
  end
```

↓

デプロイ時のビルド処理を実行するコマンドを記述ファイルとして、`bin/render-build.sh`を以下の通り新規作成してください。

```sh:bin/render-build.sh
#!/usr/bin/env bash
# exit on error
set -o errexit

bundle install
bundle exec rake assets:precompile
bundle exec rake assets:clean
bundle exec rake db:migrate
```

Railsアプリの再デプロイが行われる度に、ここに記述されたコマンドが実行されるようになります（データベースのマーグレーションはここで行っています）。

こちらの shファイルを Render.com 上で実行できるようにするため、パーミッションを変更します。

```sh:webコンテナ
chmod a+x bin/render-build.sh
```

↓

以上でRailsの修正は完了となりますので、忘れずにリポジトリにプッシュしてください。

```sh:ターミナル
git add -A
git commit -m "Renderへのデプロイ準備"
git push origin HEAD
```

#### Render.com サービスの作成

[render.com](https://render.com/)を用いて、Railsアプリのデプロイを行なっていきます。[サイトトップページ](https://render.com)から「GET STARTED」にアクセスしてください。

![](https://storage.googleapis.com/zenn-user-upload/524f184bb43f-20231005.png)

↓

Sign Up ページにアクセスしますので、任意の認証手段でアカウントを作成します。私はGitHubアカウント認証にしました。

![](https://storage.googleapis.com/zenn-user-upload/a5bd6b7e1bd3-20231005.png)

↓

アカウント登録が完了するとダッシュボードにアクセスできるようになりますので、「New +」 > 「Web Service」で、Webサービスの新規作成を開始します。

![](https://storage.googleapis.com/zenn-user-upload/3c3bae97578f-20231005.png)

![](https://storage.googleapis.com/zenn-user-upload/68b0ea2a2685-20231005.png)

↓

デプロイ方法を問われますので、「Build and deploy from a Git repository」を選んで、「Next」を押してください。

![](https://storage.googleapis.com/zenn-user-upload/72672dffded8-20231005.png)

↓

デプロイを行うリポジトリの選択画面に移りますので、「Connect GitHub」から GitHub 接続を開始してください。

![](https://storage.googleapis.com/zenn-user-upload/cbf5a87ff3a6-20231005.png)

↓

接続を進めていくと、 Render.com からのアクセスを許可するリポジトリを選択する画面に移ります。「All repositories」で全てのリポジトリのアクセスを許可、または「Only select repositories」で今回のデプロイ対象のリポジトリのみのアクセスを許可してください。

（ここのスクリーンショットを撮影し忘れてしまったので、代わりに GitHub 上の Settings における該当箇所の画面を参考イメージとして添付します）

![](https://storage.googleapis.com/zenn-user-upload/5ca1d7ca3c8f-20231005.png)

↓

リポジトリへのアクセス許可が完了すると、 Render.com の画面上にリポジトリ名が表示されるようになりますので、対象リポジトリを「Connect」してください。

![](https://storage.googleapis.com/zenn-user-upload/be2f1fea21b7-20231005.png)

↓

構築するWebサービスに関する設定画面に移りますので、以下のように設定を行ってください。

|項目|値|
|---|---|
|Name|任意。ここでは`rails-render-planetscale-app`としています|
|Region|Singapore(Southeast Asia)|
|Branch|main|
|Root Directory|空のままでOK|
|Runtime|Ruby|
|Build Command|./bin/render-build.sh|
|Start Command|bundle exec puma -C config/puma.rb|
|Instance Type|Free (**$0 / month**)|

![](https://storage.googleapis.com/zenn-user-upload/b514b456a842-20231007.png)

![](https://storage.googleapis.com/zenn-user-upload/a83cb8f9e36b-20231006.png)

![](https://storage.googleapis.com/zenn-user-upload/aa725f8d77ae-20231006.png)

↓

画面下部の「Advanced」ボタンをクリックすると詳細設定タブを開きますので、「Add Environment Variable」から以下の環境変数を定義してください。

|key|value|
|---|---|
|RAILS_MASTER_KEY|Railsアプリの`config/master.key`に記載の文字列をコピペ|

![](https://storage.googleapis.com/zenn-user-upload/33f6be5c3dd8-20231006.png)

↓

以上の設定が完了したら、画面最下部の「Create Web Service」ボタンをクリックして、Webサービスを作成してください。

自動で Logs を表示する画面に遷移します。ここまでの実装が正しければ、`Build Command`、`Start Command`で設定したコマンドが順に実行され、最終的に puma サーバーの起動が確認できるはずです。

![](https://storage.googleapis.com/zenn-user-upload/753cc5129af7-20231007.png)

↓

画面上部に、デフォルトのアプリURLである`https://<<アプリ名>>.onrender.com`が記載されていますので、`/posts`にアクセスしてください。

![](https://storage.googleapis.com/zenn-user-upload/eed7828745b4-20231007.png)

↓

`posts`レコードの一覧画面にアクセスができれば、デプロイが正常に完了しています！また、一通りのCRUD操作も問題なく行えるはずです。

![](https://storage.googleapis.com/zenn-user-upload/54242d8b7c46-20231007.png)

## その他Tips

おまけとして、その他の役立つ Tips を記載しておきます。

### アプリの再デプロイがしたい

Render.com ではデフォルトの設定で、リポジトリの`main`ブランチに変更差分が生じた際に自動的に再デプロイをしてくれます。したがって、何か特別なデプロイ作業は必要ありません。

デプロイの関する設定は、「Settings > Build & Deploy」から確認できます。

![](https://storage.googleapis.com/zenn-user-upload/ba20d84a4ce4-20231008.png)

![](https://storage.googleapis.com/zenn-user-upload/f13a54225b89-20231008.png)

### URLを独自ドメインにしたい

「Settings > Custom Domains」から、独自ドメインを設定することができます。[お名前.com](https://www.onamae.com/)等で取得した独自ドメインをこちらから適用してください。

![](https://storage.googleapis.com/zenn-user-upload/798166602c23-20231008.png)

### 本番環境で Rails コンソールを起動したい

Render.com の「Shell」からシェルを起動し、`$ rails s`を実行することで Rails コンソールを起動できる、、、と思うのですが、残念ながら月額7ドルの Starter プランからしか「Shell」を利用できません。この辺りは完全無料では厳しそうです。。。

![](https://storage.googleapis.com/zenn-user-upload/93a1da492c73-20231008.png)

### seed.rb のテストデータを本番環境DBに流し込みたい

Railsコンソールの起動は難しくても、テストデータの流し込みだけであれば可能です。

`db/seeds.rb`を任意に作成した後、`bin/render-build.sh`にテストデータを流し込むコマンドを追加してリポジトリに push します。

```diff sh:bin/render-build.sh
  #!/usr/bin/env bash
  # exit on error
  set -o errexit

  bundle install
  bundle exec rake assets:precompile
  bundle exec rake assets:clean
  bundle exec rake db:migrate
+ bundle exec rake db:seed
```

自動で再デプロイが行われる過程で当コマンドが実行され、本番環境DBにテストデータが流し込まれます。

また、このままでは再デプロイの度にテストデータ生成が繰り返し実行されてしまうので、一度テストデータ生成が完了したら、当該コマンドは削除 or コメントアウトしておきましょう。

```diff sh:bin/render-build.sh
  #!/usr/bin/env bash
  # exit on error
  set -o errexit

  bundle install
  bundle exec rake assets:precompile
  bundle exec rake assets:clean
  bundle exec rake db:migrate
- bundle exec rake db:seed
+ # bundle exec rake db:seed
```

### GUIクライアントアプリから本番環境DBにアクセスしたい

DBアクセスに必要な`username`, `password`, `host`, `database`に基づいて接続できるような設定を行いましょう。

特に MySQL GUI アプリとして [Sequel Ace](https://apps.apple.com/jp/app/sequel-ace/id1518036000?mt=12) を使用する場合は、以下の公式リファレンスにしたがって設定を行えばOKです。

https://planetscale.com/docs/tutorials/connect-mysql-gui

![](https://storage.googleapis.com/zenn-user-upload/6f45ad36ec6f-20231008.png)

## さいごに

ここまで読んでいただきありがとうございました！

もしよろしければ、記事の**いいね**や **Twitter(X)** でのリアクション&アカウントフォロー([@ddpmntcpbr](https://twitter.com/ddpmntcpbr))をお願いします🙏

また、記事内容に不備がございましたら記事コメントまたは Twitter DM でご連絡いただけますと幸いです。
