---
title: "【※2024年4月より有料化】Rails アプリを公開しよう！ Render.com + PlanetScale デプロイ手順"
emoji: "🐑"
type: "tech"
topics: ["rails", "render", "mysql", "初心者", "初心者向け"]
published: true
---

## （※2024年3月7日追記）重要事項

当記事は、Render.com + PlanetScale の構成で Ruby on Rails アプリを無料デプロイする方法を整理したものでしたが、[2024年4月8日をもって PlanetScale の無料プラン(Hobby)が廃止されることが決定しました](https://planetscale.com/blog/planetscale-forever)。

したがって、**当記事の手順では Rails アプリ無料公開はできなくなります**。

代替案について検討しましたが、**2024年3月7日時点で Rails アプリを期間制限なしで無料公開する方法は見つかっていません**。

PlanetScale については[最安値でも$39/month、特にアジアリージョンでは$47/monthの費用が発生する](https://planetscale.com/pricing)ため、安価に Rails アプリを公開したい場合は他のSaaSの利用をオススメします。

以下、安価なプランが存在するSaaSを表にまとめておきます。

|サービス名|コスト(プラン)|備考|
|---|---|---|
|[Heroku](https://jp.heroku.com/pricing)|**$5/month** (Eco) |「無料SSL」や「スリープ状態への移行なし」にしたい場合は 上限 **$7/month** の Basic プランを選択する|
|[Render.com](https://render.com/pricing)|**$7/month** (Individual + Starter)|アカウントプランとして Individual (無料)に加入し、 PostgreSQL のプランとして Starter に加入する。ただし、**PostgreSQL はサービス開始から90日までは無料の Free プランを利用できる**。|
|[Fly.io](https://fly.io/plans)|**$5/month** (Hobby)|-|

個人的な意見としては、

- 就活用ポートフォリオアプリとして一時的に公開するだけなら、 Render.com の Individual + Free プランで無料で済ませる（90日以内に非公開にする前提）。
- 多少お金がかかってでも安定して長期的に運用したいなら、 Heroku の Starter プランで「無料SSL」「スリープ状態への移行なし」にしておく。

といった使い分けになるのかなと思います（Fly.io の Hobby プランも安価でありますが、他2つに比べて日本語ドキュメントが少ないので、初学者にはややハードルが高いと思います）。

最後に当記事の扱いについて、 Rails アプリの無料公開はできなくなりますが、Render.com + PlanetScale によるデプロイを行う際の参考情報にはなるかと思いますので、記事自体は残してくことにします。

## この記事は？

当記事を執筆している2023年10月現在において、ローカルで開発した Ruby on Rails アプリを**完全無料**でデプロイする方法について整理したものです。

## 動機

以前までは、Railsアプリの無料デプロイ先としては [Heroku](https://jp.heroku.com/) が主流でした。

しかし、2022年11月より Heroku の無料プランが廃止となり、最安のEcoプランでも**月5ドル**の費用が発生するようになってしまいました（2023年10月時点）。

https://jp.heroku.com/pricing

企業等で本格的に運用するアプリであれば運用コストは飲み込めるかもしれませんが、特に個人開発のアプリの場合、どうしてもコストを無料に抑えたいケースもあると思います。

- **転職用のポートフォリオ**として公開するので、高いパフォーマンスは必要ない。
- 将来的には有料運用は想定しているが、最初の**市場検証フェーズ**では無料から始めたい。
- **単なる趣味アプリ**でしかないので、お金がかかるなら公開はできない。

そこで当記事では、 Heroku の無料プランに代わる手段として、**Render.com** + **PlanetScale** による無料デプロイ方法を解説していきます。

## Render.com とは？

Webアプリケーションの公開を含めた様々なサービスを提供する PaaS です。当記事では、**Herokuの代替サービス**くらいの粒度で認識しておいてもらえればと思います。

https://render.com/

[Render.comの料金ページ](https://render.com/pricing)を確認すると、**Plan**としては、個人向け（Individual）プランが無料で提供されています。

![](https://storage.googleapis.com/zenn-user-upload/959d3bf625df-20231007.png)

しかし、**Render.com が提供するデータベースは完全無料では利用できません**。**COMPUTE**内のデータベース（`PostgreSQL`）について確認すると、利用開始から90日のみ無料で、それ以降は**月額7ドル**の料金が発生してしまいます。

![](https://storage.googleapis.com/zenn-user-upload/12e0c81b4552-20231007.png)

今回は、Railsアプリのデプロイのみを Render.com に対して行い、データベースとしては **PlanetScale** という別サービスを活用することで、**期間制約なしの無料デプロイ**を実現してみたいと思います。

## PlanetScale とは？

**サーバーレスな MySQL を提供するサービス**です。

https://planetscale.com/

ここでいうサーバーレスは「**サーバー管理が不要で楽チン**」くらいのニュアンスの理解でいったんは問題ないと思います。従来の MySQL との機能的な差異もありますが、当記事においてはあまり意識する必要はありません。

さて、肝心の[PlanetScaleの料金ページ](https://planetscale.com/pricing)を確認すると、Hobbyプランが無料で提供されています。**これは Render.com の PostgreSQL とは異なり期間の制約がなく、完全無料で利用が可能です**。

![](https://storage.googleapis.com/zenn-user-upload/10e76aa19330-20231007.png)

ここまでの話をまとめると、

- **Rails アプリは、 Render.com の Individual プランを利用**
- **DB（MySQL） は、 PlanetScale の Hobby プランを利用**

という構成で Rails アプリを完全無料でデプロイしていきたいと思います。

次からは、具体的な手順に移っていきます。

## 宣伝

zenn上に、**Rails × Next.js × AWS アプリの開発チュートリアル本**をリリースしています。

https://zenn.dev/ddpmntcpbr/books/rna-hands-on

もし、あなたが「**転職用ポートフォリオとしての Rails アプリを無料デプロイする方法を知りたい**」という動機で当記事に辿り着いた場合、こちらの書籍でワンランク上のポートフォリオ開発に挑戦してみることをぜひ検討してもらえたらと思います。**Next.js、AWSに関する予備知識なしでも取り組める内容になっています**。

また、以降の当記事で紹介する方法は、 API モードの Rails アプリおいても同じようにデプロイ可能です。さらにフロントエンドに Next.js を採用している場合、 [Vercel](https://vercel.com/) の無料プランを活用すれば、**Rails(Render.com + PlanetScale) × Next.js(Vercel) の構成も完全無料でデプロイ可能です**。

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
|MySQL(development環境)|8.0.32|

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
    image: mysql:8.0.32
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: myapp_development
      MYSQL_USER: root
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

↓

webコンテナの Rails から dbコンテナの mysql にアクセスするため、`config/database.yml`を以下のように書き換えてください。

```yml:config/database.yml
default: &default
  adapter: mysql2
  encoding: utf8mb4
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  port: 3306

development:
  <<: *default
  host: db
  username: root
  password: password
  database: myapp_development
```

↓

設定が完了したら、dockerを起動します。

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
rails db:create
```

http://localhost:3000 で Rails にアクセスできることを確認ください。

![](https://storage.googleapis.com/zenn-user-upload/09c62e8cd95f-20231003.png)

↓

動作確認のための簡単な機能として、**Post**モデルに対する CRUD を実装します。

webコンテナ内で scaffoldによる機能一括作成を行ってください。

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

![](https://storage.googleapis.com/zenn-user-upload/7c3dfe3195b0-20231005.png)

### 2. PlanetScale でデータベースを作成

production環境用の mysql DB として PlanetScale 上に DB を作成します。

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

以下スクロールすると、Railsアプリからデータベースのアクセスするための設定チュートリアルが記載されていますので、こちらをヒントにしながら設定を進めていきます。

↓

#### Installation

**Installation**では、gemとして`mysql2`と`planetscale_rails`（development, test 環境のみ）の導入が提案されています。

しかし、当ページの手順でRailsアプリを作成している場合、`mysql2`はすでに導入済みであること、 今回は本番環境でのみ PlanetScale データベースを利用することから`planetscale_rails`は導入不要であることから、ここの操作はスキップします。

![](https://storage.googleapis.com/zenn-user-upload/8ec3092aabe7-20231004.png)

↓

#### Update production credentials

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

#### Update database.yml

**Update database.yml**では、Railsからデータベースにアクセスするための設定を変更する内容が記載されています。

![](https://storage.googleapis.com/zenn-user-upload/c3c6ca2d60af-20231005.png)

`development`に関しては今回 PlanetScale データベースを使用しないためスキップでOKです。`production`に関する設定のみ、画面に表示されているヒントを参考に、`config/database.yml`に設定を加えます。

```diff yml:config/database.yml
  default: &default
    adapter: mysql2
    encoding: utf8mb4
    pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
    port: 3306

  development:
    <<: *default
    host: db
    username: root
    password: password
    database: myapp_development
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

#### Update production schema

**Update production schema**では、PlanetScaleデータベースのマイグレーションを実行する方法が記載されています。

![](https://storage.googleapis.com/zenn-user-upload/35f8a852dc87-20231005.png)

今回は、renderにRailsアプリをデプロイするたびにマイグレーションを自動実行するように設定を行いますので、ここはスキップでOKです。

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
