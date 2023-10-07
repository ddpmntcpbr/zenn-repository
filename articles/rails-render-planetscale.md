---
title: "Railsアプリを無料で公開しよう！Render + PlanetScale デプロイ手順"
emoji: "🐑"
type: "tech"
topics: ["rails"]
published: false
---

## Railsアプリの用意

```:ディレクトリ構造
.
├── Dockerfile
├── Gemfile
├── Gemfile.lock
├── docker-compose.yml
└── entrypoint.sh
```

#### docker-compose.yml

```yml:./docker-compose.yml
version: '3'
services:
  db:
    image: mysql:8.0.32
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: myapp_development
      MYSQL_USER: user
      MYSQL_PASSWORD: password
    ports:
      - "3307:3306"
    volumes:
      - mysql_data:/var/lib/mysql
  web:
    build: .
    command: bash -c "bundle exec rails s -b '0.0.0.0'"
    volumes:
      - .:/myapp
    ports:
      - 3000:3000
    depends_on:
      - db
    tty: true
    stdin_open: true
volumes:
  mysql_data:
```

#### Dockerfile

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

```sh:rails/entrypoint.sh
#!/bin/bash
set -e

rm -f /myapp/tmp/pids/server.pid

exec "$@"
```

#### Gemfile & Gemfile.lock

```ruby:rails/Gemfile
source "https://rubygems.org"
gem "rails", "~> 7.0.4"
```

```ruby:rails/Gemfile.lock
(空のファイルを作成)
```

```sh:ターミナル
docker compose run --rm web rails new . --database=mysql
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

webコンテナの Rails から dbコンテナの mysql にアクセスするため、`config/database.yml`を以下のように書き換えてください。

```yml:config/database.yml
default: &default
  adapter: mysql2
  encoding: utf8mb4
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: root
  port: 3306

development:
  <<: *default
  host: db
  database: myapp_development
  password: password
```

```sh:ターミナル
docker compose build
```

```sh:ターミナル
docker compose up -d
```

http://localhost:3000 で Rails にアクセスできることを確認ください。

![](https://storage.googleapis.com/zenn-user-upload/09c62e8cd95f-20231003.png)

動作確認のための簡単な機能として、**Post**モデルに対する CRUD を実装します。

webコンテナ内に入り、①DB作成、②scaffoldによる機能一括作成、③マイグレーション実行、を順に行ってください。

```sh:ターミナル
docker compose exec web /bin/bash
```

```sh:webコンテナ
rails db:create
```

```sh:webコンテナ
rails g scaffold Post body:text
```

```sh:webコンテナ
rails db:migrate
```

↓

テストデータを作成します。以下のように`db/seeds.rb`を作成し、`$ rails db:seed`を実行してください。

```rb:db/seeds.rb
Post.create(body: "test1")
Post.create(body: "test2")
Post.create(body: "test3")
```

```sh:webコンテナ
rails db:seed
```

↓

http://localhost:3000/posts で posts 一覧画面にアクセスし、各種CRUD操作（作成、読込、更新、削除）を画面上から行えることを確認してください。以上で、アプリ実装は完了です。

![](https://storage.googleapis.com/zenn-user-upload/76643c6993cd-20231005.png)

↓

アプリ実装が完了したら、お使いの GitHub アカウントでリポジトリを作成し、ソースコードのプッシュまで行ってください。リポジトリの公開範囲は、パブリック／プライベートのどちらでも構いません。

![](https://storage.googleapis.com/zenn-user-upload/7c3dfe3195b0-20231005.png)

## PlanetScale


[PlanetScale](https://planetscale.com/)のトップページにアクセスしてください。

![](https://storage.googleapis.com/zenn-user-upload/b9f8a64eaab7-20231004.png)

↓

「Get Started」にアクセスすると Sign Up 画面に入れますので、メールアドレス認証 or GitHub アカウント認証のいずれかの手段でアカウントを作成してください。筆者は GitHub アカウント認証にしています。

![](https://storage.googleapis.com/zenn-user-upload/5f6d7ec7678a-20231004.png)

↓

アカウント作成が完了するとダッシュボードにアクセスできるようになりますので、「Create a new database」からデータベースの新規作成を開始します。

![](https://storage.googleapis.com/zenn-user-upload/487ea25d9765-20231004.png)

↓

以下の設定項目を入力してください。

|項目|値|
|---|---|
|Database name|任意。ここでは `myapp_production` としています|
|Region|ap-northeast-1(Tokyo)|
|Plan type|Hobby|
|Cluster size|Hobbyプランでは設定変更不可のためスキップ|
|Autoscaling storage|Hobbyプランでは設定変更不可のためスキップ|

![](https://storage.googleapis.com/zenn-user-upload/baa0ff9197dc-20231004.png)

![](https://storage.googleapis.com/zenn-user-upload/ea32b1c4b6b0-20231004.png)

![](https://storage.googleapis.com/zenn-user-upload/07ebc0ee1310-20231004.png)

![](https://storage.googleapis.com/zenn-user-upload/08a796cbc0d3-20231004.png)

↓

画面下にスクロールすると、「Please add a creadit or debit card to this organization」と記載されてますので、「Add new card」から手元のクレジットカードorデビットカード情報を登録してください。

![](https://storage.googleapis.com/zenn-user-upload/5798e1fb55f8-20231004.png)

↓

カード情報の入力が完了したら、「Create database」からデータベースを作成してください。

![](https://storage.googleapis.com/zenn-user-upload/ac3777e976cc-20231004.png)

↓

データベースが作成されたら、そのままデータベースに対するアクセスキーを生成する画面に遷移します。「Select your language or framework」で「Rails」を選択してください。

![](https://storage.googleapis.com/zenn-user-upload/47da16eed942-20231004.png)

↓

画面下にスクロールし、「Create a password」から「Password name」を任意で入力（デフォルト入力値でもOK）し、「Create password」をクリックしてください。

なお、「Password name」はあくまでパスワード情報を一意に識別するための文字列であって、パスワードそのものではありません。したがって、自分が分かりやすいもので問題ありません。

![](https://storage.googleapis.com/zenn-user-upload/eb050fd4c05d-20231004.png)

↓

パスワード生成が完了すると、`Username`と`Password`が表示されます。**これらは秘匿情報となりますので、絶対に第三者へ流出させないでください**。

![](https://storage.googleapis.com/zenn-user-upload/7cb4cf6292b9-20231004.png)

↓

以下スクロールすると、Railsアプリからデータベースのアクセスするための設定チュートリアルが記載されていますので、必要な部分に絞ってしたがって進めていきます。

↓

### Installation

**Installation**では、gemとして`mysql2`と`planetscale_rails`（development, test 環境のみ）の導入が提案されています。

しかし、当ページの手順でRailsアプリを作成している場合、`mysql2`はすでに導入済みであること、 今回は本番環境でのみ PlanetScale データベースを利用することから`planetscale_rails`は導入不要であることから、ここの操作はスキップします。

![](https://storage.googleapis.com/zenn-user-upload/8ec3092aabe7-20231004.png)

↓

### Update production credentials

**Update production credentials**では、データベースのアクセス情報の管理方法について記載されています。

![](https://storage.googleapis.com/zenn-user-upload/1c4a5e09e4b5-20231005.png)

Rails の credentials を利用して、データベースへのアクセス情報を保存します。webコンテナ内から vim で credentials ファイルを開いてください。

```sh:webコンテナ
EDITOR="vi" rails credentials:edit
```

画面上の`config/credentials.yml.enc`の内容をそのまま credentials ファイルに貼り付けてください。

```yml:config/credentials.yml.enc
.
.
planetscale:
  username: xxxxxxxxxxxxxxxxxx
  host: aws.connect.psdb.cloud
  database: myapp_production
  password: xxxxxxxxxxxxxxxxxx
```

### Update database.yml

**Update database.yml**では、Railsからデータベースにアクセスするための設定を変更する内容が記載されています。

![](https://storage.googleapis.com/zenn-user-upload/c3c6ca2d60af-20231005.png)

`development`に関しては今回 PlanetScale データベースを使用しないためスキップでOKです。`production`に関する設定のみ、画面に表示されているヒントを参考に、`config/database.yml`に設定を加えます。

```diff yml:config/database.yml
  default: &default
    adapter: mysql2
    encoding: utf8mb4
    pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
    username: root
    port: 3306

  development:
    <<: *default
    host: db
    database: myapp_development
    password:
+
+ production:
+   <<: *default
+   username: <%= Rails.application.credentials.planetscale&.fetch(:username) %>
+   password: <%= Rails.application.credentials.planetscale&.fetch(:password) %>
+   database: <%= Rails.application.credentials.planetscale&.fetch(:database) %>
+   host: <%= Rails.application.credentials.planetscale&.fetch(:host) %>
+   ssl_mode: verify_identity
+   sslca: "/etc/ssl/certs/ca-certificates.crt"
```

`Rails.application.credentials.planetscale&.fetch(:xxx)`で、先ほど credentials ファイルに保存した`planetscale.xxx`キーの値を参照しています。

また、`sslca`に関する設定を新規で追加しています。これは Render.com におけるSSL証明書のパスを指し示しており、当設定を加えることで Render.com から PlanetScale データベースへのSSH接続を可能にしています。

### Update production schema

**Update production schema**では、PlanetScaleデータベースのマイグレーションを実行する方法が記載されています。

![](https://storage.googleapis.com/zenn-user-upload/35f8a852dc87-20231005.png)

今回は、renderにRailsアプリをデプロイするたびにマイグレーションを自動実行するように設定を行いますので、ここはスキップでOKです。

## Render

https://render.com/docs/deploy-rails

### Raisの修正

Render上でサービスを作成する前に、Railsにいくつかの修正を加えます。

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

こちらの shファイルをRender上で実行できるようにするため、パーミッションを変更します。

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

### Renderサービスの作成

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

接続を進めていくと、Renderからのアクセスを許可するリポジトリを選択する画面に移ります。「All repositories」で全てのリポジトリのアクセスを許可、またh「Only sekect repositories」で今回のデプロイ対象のリポジトリのみのアクセスを許可してください。

（ここのスクリーンショットを撮影し忘れてしまったので、代わりに GitHub 上の Settings における該当箇所の画面を参考イメージとして添付します）

![](https://storage.googleapis.com/zenn-user-upload/5ca1d7ca3c8f-20231005.png)

↓

リポジトリへのアクセス許可が完了すると、Render.comの画面上にリポジトリ名が表示されるようになりますので、対象リポジトリを「Connect」してください。

![](https://storage.googleapis.com/zenn-user-upload/be2f1fea21b7-20231005.png)

↓

以下のように設定を行ってください。

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
|RAILS_MASTER_KEY|Railsアプリの`config/master.key`に記載のランダム文字列をコピペ|

![](https://storage.googleapis.com/zenn-user-upload/33f6be5c3dd8-20231006.png)

↓

以上の設定が完了したら、画面最下部の「Create Web Service」ボタンをクリックして、Webサービスを作成してください。

自動でデプロイログを表示する画面に遷移します。ここまでの実装が正しければ、`Build Command`、`Start Command`が順に実行され、pumaサーバーが起動が確認できるはずです。
