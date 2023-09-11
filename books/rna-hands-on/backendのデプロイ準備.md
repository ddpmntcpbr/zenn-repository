---
title: "はじめに「未経験転職は難しい。だからこそ」"
---

## この章でやること

backendアプリのデプロイ準備として、コードの追加・修正を行います。

## 本番環境でのbackend構成

開発環境（＋テスト環境）と本番環境では、いくつかの設定を適切に切り替える必要があります。一覧としてまとめすが、各環境でbackendの構成は以下のように切り替わります。

|項目|開発環境|本番環境|
|---|---|----|
|サーバー構成|Rails puma サーバーのみ|nginx サーバー + Rails pumaサーバー（socket通信）|
|Dockerfile|rails/Dockerfile|rails/Dockerfile.prod & nginx/Dockerfile.prod|
|DBマイグレーション|railsコンソール内から`$ rails db:migrate`コマンドを実行する|エントリーポイント用のshファイルにマイグレーションをコマンドを記載し、毎デプロイ時に実行する|
|DB接続設定|アクセス情報が rails/config.database.yml 内に直接記述する|credentials 内に暗号化されたアクセス情報を rails/config.database.yml で呼び出す|
|環境変数|rails/config/development.yml を参照|ails/config/production.yml を参照|

新しい概念がたくさん出てきていると思いますので、ひとつずつ説明します。

### サーバー構成

開発環境においては、railsコンテナ内で`$ rails s`コマンドを実行することで、 http://localhost:3000 に対してアクセスをできるようになっていました。

このとき、Railsに標準で搭載されている**puma**というサーバーが起動しています。pumaがブラウザからのリクエストを受け取り、適切にレスポンスを返すようにしてやり取りが成立しています。

しかし、pumaはあくまで開発用の簡易的なサーバーでしかなく、これ単体のみでの本番運用には適していません。例えば本番環境では短時間に多くのリクエストが集中するようなケースが考えられますが、pumaは多くにリクエストを適切に処理することを想定していないため、簡単に落ちてしまいます。

そこで、Railsを本番環境で運用する際は、本番運用により適した**webサーバー**と組み合わせることが一般的です。（もちろんRailsに限らず、他の言語、フレームワークでも同様です）。

クライアントからのリクエストを直接pumaサーバーで処理するのではなく、**まずwebサーバーで受け取ってからpumaに流す**、という構成にします。これによって上記例の負荷分散に加え、セキュリティー、パフォーマンス等さまざまな面でのメリットを享受できます。

webサーバーとして有名なのはいくつかありますが、今回はその中でももっともデファクトスタンダートに位置している**nginx**（エンジンエックス、と読みます）を利用します。

まとめると、以下のようなコードの追加・修正を行います。

- nginx 用のDockerfileを新規作成
- nginx の設定ファイルを新規作成
- Rails の puma 設定の修正（本番環境においてはnginxと通信を行うようにする）

### Dockerfile

前提として、今回アプリをデプロイしようと考えている AWS ECS(Elastic Container Service) というのは、dockerコンテナ化したアプリをそのまま動かすことができるサービスとなっています。今回の開発環境においてもdockerコンテナ化したRailsアプリにアクセスをしながら開発を行っている通り、**dockerを用いた開発の場合に開発環境と本番環境を似たような条件で動作させられる**点がメリットになっています。

しかし、完全に同じ条件コンテナを利用するわけではなく、各環境ごとに少しコンテナの設定を変えるのが一般的です。そのため、本番環境コンテナ用のDockerfileは、既存のDockerfileとは別に用意する必要があります。

上記のnginxの話も含めると、以下のふたつのDockerfileを新規作成します。

- nginx/Dockerfile.prod を新規作成
- rails/Dockerfile.prod を新規作成

これらdockerfileによってビルドされたコンテナを AWS ECS にデプロイして本番環境として運用する、というイメージになります。

### DBマイグレーション

Railsにおいては、DBの変更があった際には、マイグレーションコマンドを実行することによってDBのマイグレーションを行う必要があります。

開発環境においては rails コンソール内から`$ rails db:migrate`コマンドを実行すればよいのですが、今回アプリをデプロイする AWS ECS (Fargate) では、本番運用されている Rails に対して気軽に rails コンソールを起動することが難しい仕様になっています。

そこで、**本番環境用のエントリーポイントファイルにマイグレーションコマンドの実行する記述する**、という方法を取ります。エントリーポイントファイルとは、ビルドされたdockerコンテナを起動したタイミングで実行されるファイルのことです。

開発環境では`rails/entrypoint.sh`がこれに該当しており、railsコンテナを起動するたびにここに記載のshコマンドが実行されています（ゴミとして残ってしまったサーバーファイルの削除を行なっています）。

```sh:rails/entrypoint.sh
#!/bin/bash
set -e

rm -f /myapp/tmp/pids/server.pid

exec "$@"
```

これの本番環境用のファイルを別に用意し、その中でマイグレーションコマンドを記述します。すでに本番運用されているRailsアプリに変更差分を加えようとした際には、コンテナの再ビルド&起動が自然と行われますので、そのタイミングで自動的にマイグレーションが走ってくれるようになります。

まとめると、以下のような対応を行います。

- rails/entrypoint.prod.sh を新規作成する
- rails/Dockerfie.prod 内で rails/entrypoint.prod.sh をエントリーポイントファイルとして設定する

### DB接続設定

DBへの接続設定は`rails/config/database.yml`に記述されますが、開発環境（+テスト環境）では、データベース名、パスワードといった、アクセスに必要となる情報が平文で記述されています。

```yml:rails/config/database.yml
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

  test:
    <<: *default
    host: db
    database: myapp_test
    password: password
```

これらはローカルに構築したmysqlコンテナに接続するために使用されているものであるので、第三者に流出したとしても特にリスクはありません。

しかし、本番環境においては、DBにアクセスするための情報は絶対に第三者に漏らしてはなりません。コード内に平文として記載することは御法度になるので、適切に暗号化して管理する必要があります。

DBアクセスに限らず、第三者に漏らしてはならない秘匿情報を扱い仕組みとして、Railsには**credentials**という機能がありますので、これを活用します。

- credentials に本番環境のDBアクセス情報を記述する
- rails/config/database.yml の本番環境用の設定として credentials の値を参照するようにする

:::message
暗号化すべき秘匿情報をコード中に平文（暗号化されていない普通のテキスト）で記載したコミットを github リポジトリに push してはいけません。

たとえリポジトリがプライペートに管理されていたとしても、 「push された時点で世界中に知れ渡ったもの」として捉え、適切な事後対応（パスワードの変更等）を行うべきというのがエンジニア業界での常識となっています。
:::

### 環境変数

フロントエンドのドメインは、環境変数として管理していました。開発環境では`rails/config/settings/development.yml`に記述していましたので、本番環境用として`rails/config/settings/production.yml`を作成します。

- rails/config/settings/production.yml を新規作成する。

## 実装

ここから実際にコード追加、修正を行なっていきます！

### サーバー構成

- nginx 用のDockerfileを新規作成
- nginx の設定ファイルを新規作成
- Rails の puma 設定の修正（本番環境においてはnginxと通信を行うようにする）

まずアプリトップに`nginx`ディレクトリを新規作成し、配下に`Dockerfile.prod`を以下のように実装してください。

```dockerfile:nginx/Dockerfile.prod
FROM nginx:latest

# ヘルスチェック用
RUN apt-get update && apt-get install -y curl vim sudo lsof

# インクルード用のディレクトリ内を削除
RUN rm -f /etc/nginx/conf.d/*

# Nginxの設定ファイルをコンテナにコピー
ADD nginx.conf /etc/nginx/myapp.conf

# ビルド完了後にNginxを起動
CMD /usr/sbin/nginx -g 'daemon off;' -c /etc/nginx/myapp.conf

EXPOSE 80
```

↓

次に、同じく`nginx`ディレクトリ配下に、nginxの設定ファイルとなる`nginx.conf`を以下のように実装してください。

```conf:nginx/nginx.conf
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections 1024;
}

http {
  upstream myapp {
    # ソケット通信したいのでpuma.sockを指定
    server unix:///myapp/tmp/sockets/puma.sock;
  }

  server {
    listen 80;
    # ドメインもしくはIPを指定
    server_name localhost;

    access_log /var/log/nginx/access.log;
    error_log  /var/log/nginx/error.log;

    # ドキュメントルートの指定
    root /myapp/public;

    proxy_connect_timeout 600;
    proxy_read_timeout    600;
    proxy_send_timeout    600;

    client_max_body_size 100m;
    error_page 404             /404.html;
    error_page 505 502 503 504 /500.html;
    keepalive_timeout 600;
    location /healthcheck {
      root   /usr/share/nginx/html;
      empty_gif;
      break;
    }

    location / {
      try_files $uri @myapp;
    }



    # リバースプロキシ関連の設定
    location @myapp {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header Host $http_host;
      proxy_pass http://myapp;
    }
  }
}
```

↓

最後に、デフォルトで作成されている puma の設定ファイル`rails/config/puma.rb`の中身をいったん削除し、以下のように書き換えてください。

```ruby:rails/config/puma.rb
threads_count = ENV.fetch("RAILS_MAX_THREADS") { 5 }.to_i
threads threads_count, threads_count
environment ENV.fetch("RAILS_ENV") { "development" }
plugin :tmp_restart

app_root = File.expand_path("..", __dir__)
bind "unix://#{app_root}/tmp/sockets/puma.sock"
```

### Dockerfile

- nginx/Dockerfile.prod を新規作成（済）
- rails/Dockerfile.prod を新規作成

`rails/Dockerfile.prod`を以下のように実装してください。

```dockerfile:rails/Dockerfile.prod
FROM ruby:3.1.2

ENV LANG C.UTF-8
ENV TZ Asia/Tokyo
ENV RAILS_ENV=production

RUN mkdir /myapp
WORKDIR /myapp
COPY Gemfile /myapp/Gemfile
COPY Gemfile.lock /myapp/Gemfile.lock

# Deal with Bundler bugs
RUN gem update --system
RUN bundle update --bundler

RUN bundle install

COPY . /myapp
RUN mkdir -p tmp/sockets
RUN mkdir -p tmp/pids

VOLUME /myapp/public
VOLUME /myapp/tmp

# Add a script to be executed every time the container starts.
COPY entrypoint.prod.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.prod.sh
ENTRYPOINT ["entrypoint.prod.sh"]
EXPOSE 3000
```

### DBマイグレーション

- rails/entrypoint.prod.sh を新規作成する
- rails/Dockerfie.prod 内で rails/entrypoint.prod.sh をエントリーポイントファイルとして設定する（済）

`rails/entrypoint.prod.sh`を以下のよう実装してください。

```sh:rails/entrypoint.prod.sh
#!/bin/bash
set -e

echo "Start entrypoint.prod.sh"

echo "rm -f /myapp/tmp/pids/server.pid"
rm -f /myapp/tmp/pids/server.pid

echo "bundle exec rails db:create RAILS_ENV=production"
bundle exec rails db:create RAILS_ENV=production

echo "bundle exec rails db:migrate RAILS_ENV=production"
bundle exec rails db:migrate RAILS_ENV=production

echo "bundle exec rails db:seed RAILS_ENV=production"
bundle exec rails db:seed RAILS_ENV=production

echo "exec pumactl start"
bundle exec pumactl start
```

冒頭2行`#!/bin/bash`と`set -e`は、shファイルとしてコマンド実行をしてもらうためにおまじないです。3行目からが、コンテナ起動時に実行されるshコマンドになります。

`echo`は単純に文字列をログ出力するだけのコマンドです。動作自体には不要ですが、あとで開発者が分かりやすくなるようにいれています。

初回デプロイにおいては、そもそもマイグレーションを行うDBがないところからのスタートになりますので、

- db:create
- db:migrate
- db:seed

を順に実行させます。このうち、 db:create と db:seed は2回目以降のデプロイでは実行不要のため、のちほど削除する予定です。

最後の`bundle exec pumactl start`が puma サーバーの起動コマンドになります。

### DB接続設定

- credentials に本番環境のDBアクセス情報を記述する
- rails/config/database.yml の本番環境用の設定として credentials の値を参照するようにする

まず、AWSの画面から本番環境のDBアクセスに必要な情報を確認します。アクセス情報は、以下のような文字列となります。

```
mysql2://{{ マスターユーザー名 }}:{{ マスターパスワード }}@{{ エンドポイント }}
```

マスターユーザー名、マスターパスワードは、RDSでDBを作成した際に設定した値になります。教材通りに進めていれば、マスターユーザー名は`admin`、マスターパスワードは`任意で設定した値`、になっているはずです。

エンドポイントは、AWSの「RDS > データベース > zenn-clone-db」から確認することができます。

![](https://storage.googleapis.com/zenn-user-upload/2766473e7b72-20230817.png)

仮にマスターパスワードが`password`、エンドポイントが`zenn-clone-db.xxxxxxxxxxxxxxxx.ap-northeast-1.rds.amazonaws.com`の場合は、アクセス情報は以下のようになります。

```
mysql2://admin:password@zenn-clone-db.xxxxxxxxxxxxxxxx.ap-northeast-1.rds.amazonaws.com
```

アクセス情報が確認できたら、railsコンテナで以下を実行し、vimエディターで credentials ファイルを開きます。

```sh:railsコンテナ
EDITOR="vi" bin/rails credentials:edit
```

vim の使い方に慣れていない方もいらっしゃるかと思いますので簡単に補足をします。

vimでファイルを開いた直後は「待機モード」となっていて、ファイルの編集を行えません。`i`キーを入力し、「INSERTモード」にすることで、編集ができるようになります。

ファイルの最終行に、以下を入力してください。

```yml:
.
.
production:
  database_url: mysql2://admin:password@zenn-clone-db.xxxxxxxxxxxxxxxx.ap-northeast-1.rds.amazonaws.com
```

値となる文字列は、先のアクセス情報の文字列を各々で入力してください。

`esc`キーを入力することで「INSERTモード」を抜け、「待機モード」に戻ります。「待機モード」の状態で、`:wq`を入力してエンターキーを押下すると、ファイルの上書き保存が完了します。

credentials の保存が完了すると、`rails/config/credentials.yml.enc`が生成されます。中にはランダムな文字列が並んでおり、これが credentials の内容を暗号化した文章となっています。

暗号の原則として。暗号文を復号するには**復号鍵**となる文字列が別で必要となりますが、この役割を果たしているのが、`rails/config/master.key`です。

master.key はRailsにデフォルトで用意されているファイルで、中にはランダムな文字列が記述されています。これが creadentials の暗号を復号する復号鍵となっています。

当ファイルが第三者に流出すると creadentials の暗号を復号することができてしまうので、**絶対に git commit の中に含めてはいけません**。当然、デフォルトで git 管理外のファイルとして設定されています。

creadentials にDBアクセス情報が正しく登録されているか、確認をしてみます。railsコンソールを起動し、以下コマンドを実行ください。

```sh:railsコンテナ
rails c
```

```sh:railsコンテナ > railsコンソール
Rails.application.credentials.production.database_url
```

```
=> "mysql2://admin:password@zenn-clone-db.xxxxxxxxxxxxxxxx.ap-northeast-1.rds.amazonaws.com"
```

`Rails.application.credentials.xxx.yyy`のような形を取ることで、、credentials のうち該当する部分を取り出すことができます。

↓

creadentials の中身を参照するように、本番環境DBの設定を行います。`rails/config/database.yml`に以下を追記ください。

```diff yml:rails/config/database.yml
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

  test:
    <<: *default
    host: db
    database: myapp_test
    password: password
+
+ production:
+   <<: *default
+   database: myapp_prouction
+   url: <%= Rails.application.credentials.production.database_url %>
```

### 環境変数

`rails/config/settings/production.yml`を以下のように実装すればOKです。

```yml:rails/config/settings/production.yml
front_domain: https://rails-next-zenn-clone-app.com
```

`rails/config/initializers/cors.rb`で、`Settings.front_domain`に対してオリジン許可を設定しています。本番環境のときには、上記文字列が設定されるようになります。
