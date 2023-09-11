## この章でやること

ローカルでRailsアプリを新規作成し、githubリポジトリへのpushまでを行います。

## 手順

### アプリの名前を決める

早速開発を開始しましょう！まずは開発アプリの名前を任意で決めてください。

当教材では、アプリ名は`rails-next-zenn-clone`としています。

### ローカルでRailsアプリを新規作成

Railsアプリを新規作成するために、まずはRailsが動作するためのdocker環境を用意します。

ローカルの任意の場所で`rails-next-zenn-clone`ディレクトリを作成し、その配下で以下の構成を作成してください（以降、特に説明がない場合は、`.`マークは、アプリトップである`rails-next-zenn-clone`ディレクトリ直下を意味します。）

```:ディレクトリ構造
.
├── docker-compose.yml
└── rails
    ├── Dockerfile
    ├── Gemfile
    ├── Gemfile.lock
    └── entrypoint.sh
```

各ファイルの中身を以下のようにしてください。

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
  rails:
    build:
      context: ./rails/
      dockerfile: Dockerfile
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -b '0.0.0.0'"
    volumes:
      - ./rails:/myapp
    ports:
      - 3000:3000
    depends_on:
      - db
    tty: true
    stdin_open: true
volumes:
  mysql_data:
```

↓

新規Railsアプリを作成します。`rails-next-zenn-clone`直下で以下を実行してください。

```sh:.
docker compose run --rm rails rails new . --force --api --database=mysql --skip-action-cable --skip-sprockets --skip-turbolinks --skip-webpack-install --skip-test --skip-bundle
```

railsコンテナの起動に必要なdockerイメージがダウンロードされた後、railsコンテナ内で`$ rails new`が実行されます。

|オプション名|効果|
|---|---|
|--force||
|--api|APIモードのRailsアプリとして新規作成する|
|--database=mysql|DBとしてMySQLを利用する|
|--skip-action-cable|Action Cable関連の設定ファイルの作成をスキップする|
|--skip-sprockets|Sprockets の設定をスキップする|
|--skip-turbolinks|Turbolinks の設定をスキップする|
|--skip-webpack-install|webpackのインストールをスキップする|
|-skip-test|minitest の設定をスキップする|
|--skip-bundle|新規作成時の`bundle install`実行をスキップする|

↓

railsディレクトリ配下に、Railsアプリが新規作成されたことを確認してください。

```:ディレクトリ構造
rails-next-zenn-clone
├── docker-compose.yml
└── rails
    ├── .git
    ├── .gitattributes
    ├── .gitignore
    ├── .ruby-version
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
    ├── entrypoint.sh
    ├── lib
    ├── log
    ├── public
    ├── storage
    ├── tmp
    └── vendor
```

↓

この後、`rails-next-zenn-clone`ディレクトリの単位でGit管理していくため、自動作成される`rails/.git`は削除してください。

```sh:.
rm -rf rails/.git
```

「`rails/.git`がそもそもないよ！」という人は、ファイル閲覧アプリにおいて、隠しファイルが非表示の設定になっていると思われます。以下を参考に、隠しファイルを見えるようにしておいてください。

- [Macで隠しファイルを表示させるショートカット](https://qiita.com/iwason/items/3cc3f3406903fb137f36)

### Railsサーバーの立ち上げ

新規作成したRailsアプリのサーバーを起動させ、`http://localhost:3000`でアクセスできるようにします。

まずは、`Gemfile`を修正します。アプリ新規作成時にデフォルト設定で色々書き込まれていますが、現時点での最低限のものだけを残し、以降は必要な場面で都度gemを追加していきます。

```ruby:rails/Gemfile
source "https://rubygems.org"
git_source(:github) { |repo| "https://github.com/#{repo}.git" }

ruby "3.1.2"

# railsの起動時間を短縮する（標準gem）
gem "bootsnap", require: false

# MySQLに接続する
gem "mysql2", "~> 0.5"

# pumaサーバーを使えるようにする（標準gem）
gem "puma", "~> 5.0"

# rails本体（標準gem）
gem "rails", "~> 7.0.4"

# タイムゾーン情報を提供する（標準gem）
gem "tzinfo-data", platforms: %i[mingw mswin x64_mingw jruby]
```

↓

railsコンテナ内で`$ bundle install`を行い、gemの依存関係を`Gemfile.lock`に記述させます。

```sh:.
docker compose run --rm rails bundle install
```

↓

`config/database.yml`を修正し、railsがmysqlサーバーに接続できるようにします。

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

↓

Railsファイルの準備が完了しましたので、docker-compose.ymlで定義している`db`コンテナ、`rails`コンテナを両方立ち上げていきます。

まず、`$ docker compose build`でコンテナをビルドします。

```sh:.
docker compose build
```

↓

その後、`$ docker compose up`でコンテナを起動します。

```sh:.
docker compose up -d
```

↓

各コンテナの起動と同時に`rails`サーバーが起動してくれているはずです。`http://localhost:3000/`にアクセスできることを確認してください。

![](https://storage.googleapis.com/zenn-user-upload/6ca6f16faed8-20230612.png)

もし、サーバーが立ち上がっていない場合、サーバーの起動にもう少し時間が必要か、あるいは何らかの設定を間違えてしまっているか、が考えられます。

そのときは、`rails`コンテナのログを確認してみましょう。`$ docker ps`で`rails`コンテナの`CONTAINER ID`を確認し、`$ docker logs`コマンドを実行します。

```sh:.
docker logs <<CONTAINER ID>>
```

正常にサーバーが起動できていれば、以下のようなログが出力されているはずです。起動に失敗している場合はここに表示されるエラーメッセージを参考に、トラブルシューティングを行ってください。

```sh:.
=> Booting Puma
=> Rails 7.0.5 application starting in development
=> Run `bin/rails server --help` for more startup options
Puma starting in single mode...
* Puma version: 5.6.5 (ruby 3.1.2-p20) ("Birdie's Version")
*  Min threads: 5
*  Max threads: 5
*  Environment: development
*          PID: 1
* Listening on http://0.0.0.0:3000
```

### コンテナ起動とサーバー起動を分離

現在の`docker-compose.yml`の設定では、「`rails`コンテナの起動」と「`rails`サーバーの起動」のタイミングが完全に一致する仕様になっています。コンテナ起動を行うだけでサーバーまで自動で起動してくれるのは便利な反面、融通が効かない側面もあります。

（`rails`サーバーの再起動が必要になった場合にコンテナごと再起動させなくてはならない、サーバーログを見たいときにわざわざ`$ docker logs`を打ち込まないといけない、等）

ここでは、docker-compose.ymlを微修正し「`rails`コンテナの起動」と「`rails`サーバーの起動」のタイミングを分離し、独立して制御できるようにします。

まずは、dockerコンテナをいったん停止させます。

```sh:.
docker compose down
```

docker-compose.ymlにおいて、`rails`コンテナ起動時に実行するコマンドを、以下のように修正します。


```diff yml:./docker-compose.yml
.
.
  rails:
    build:
      context: ./rails/
      dockerfile: Dockerfile
-   command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -b '0.0.0.0'"
+   command: bash -c "rm -f tmp/pids/server.pid && tail -f log/development.log"
.
.
```

:::message
dockerの仕様として、コンテナ起動後に何らかのコマンドが常時実行させないと、自動的にコンテナが終了してしまいます。それを防ぐために、ここでは適当にログを監視するコマンドを常時実行させています（常時実行できれば何でもOKです）
:::

コンテナを再起動します。この時点ではコンテナのみが起動しており、サーバーは起動していません。

```sh:.
docker compose up -d
```

サーバーを起動します。以下コマンドで、railsコンテナ内に入ります。

```sh:.
docker compose exec rails /bin/bash
```

`root@xxxxxxxxx:/myapp#`に入り、コマンド入力を受け付けるようになるはずなので、ここでサーバーを立ち上げます。（`-b`オプションは必須です）

```sh:railsコンテナ
rails s -b '0.0.0.0'
```

↓

サーバーが起動したら、`http:localhost:3000`で同じようにアクセスができるはずです。

サーバーのみを再起動したい場合は、railsコンテナ内で`Ctrl+C`で停止し、その後再度サーバー起動コマンドを入力すればOKです。

### github リポジトリに push

お使いの github アカウントにて、新規リポジトリを作成してください。

![](https://storage.googleapis.com/zenn-user-upload/7877e2e01bb7-20230611.png)

|項目|値|
|---|---|
|Repository name|任意（当教材では`rails-next-zenn-clone`としています）|
|Public or Private|任意|
|Add a README file|チェックなし（デフォルト）|
|Add .gitignore|None（デフォルト）|
|Choose a license|None（デフォルト）|

↓

以下のコマンドを、アプリルート直下で順番に実行していきます。

```sh:.
echo "# rails-next-zenn-clone" >> README.md # README.md
git init # アプリルート直下に .git を作成
git add -A # 全てのファイルをステージング
git commit -m "first commit" # 全てのファイルをコミット
git branch -M main # ブランチ名を main に変更
git remote add origin https://github.com/ddpmntcpbr/rails-next-zenn-clone.git # git push の先を新規作成したリポジトリに向ける
git push -u origin main # 新規作成したリポジトリに push
```

↓

github の該当リポジトリページで、ローカルで作成したアプリが push されていることを確認できればOKです！

### ※おまけ: sourcetree を用いた git 操作

（こちらの設定は任意です）

初学者の多くは、コマンドライン上からの git 操作に慣れていないかと思います。

コマンドライン上からの git 操作に慣れていない人には、[sourcetree](https://www.sourcetreeapp.com/)というアプリがオススメです。

git 操作を直感的に行うことができ、gitの仕組みもだんだんと理解できるようになっていきます。

インストール手順や操作方法は、分かりやすい参考文献が世の中にたくさんありますので、そちらに譲りますが、gitに苦手意識がある人は導入を検討してみてください。

- [Gitなんて怖くない！超初心者向け、SourceTreeの使い方はじめの一歩！](https://prog-8.com/blogs/how_to_use_sourcetree)
- [Youtube『gitの使い方 基礎知識 - SourceTreeを利用したGitの基本操作』](https://www.youtube.com/watch?v=LyXS3BWJ244&list=PLvEni36L5VuXc3dpowx66hTDuj6qFg5Mm)
