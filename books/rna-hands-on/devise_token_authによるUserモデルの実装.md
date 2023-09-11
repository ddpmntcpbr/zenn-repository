## この章でやること

gem `devise_token_auth` を利用して `User` モデルを実装します。

## devise_token_authとは？

Rails API において、認証機能を持ったモデルを簡単に実装できる gem です。

https://github.com/lynndylanhurley/devise_token_auth

Rails単体でのアプリケーション開発で devise を利用したことがある方にとっては、それのAPIモード版、と思っていただければOKです。

認証機能を一から自力で実装するのは大変なので、多くの企業では Rails の認証機能を何らかの認証系 gem で実装していると思われますが、 devise 系との中で最もポピュラーな認証系 gem になります。

今回は、 devise_token_auth を用いて User モデルを実装してきます。

### 参考
- [【Rails API 入門】devise-token-auth](https://qiita.com/tomokazu0112/items/5fdd6a51a84c520c45b5)
- https://blog.furu07yu.com/entry/rails7-devise-token-auth#ActionDispatchRequestSessionDisabledSessionError%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6

## devise_token_auth の導入

devise 関連の gem をインストールします。

```diff :rails/Gemfile
.
.
# 環境毎の設定管理を行う
gem "config"

+ # ユーザー認証を提供する
+ gem "devise"

+ # devise を日本語化する
+ gem "devise-i18n"
+
+ # ユーザーのトークン認証を提供する
+ gem "devise_token_auth"
+
# MySQLに接続する
gem "mysql2", "~> 0.5"
.
.
```

```sh:railsコンテナ
bundle install
```

↓

まず、 devise 用の設定のファイルを以下コマンドで作成します。

```sh:railsコンテナ
rails g devise:install
```

その後、`User`モデルを devise_token_auth 適用モデルとして新規作成します。

```sh:railsコンテナ
rails g devise_token_auth:install User auth
```

migrationファイルを含めたいくつかのファイルにで新規作成、変更が発生したと思われます。まずは、マイグレーションを実行し、 User モデルを作成します。

```sh:railsコンテナ
rails db:migrate
```

rails/db/schema.rb に users テーブルが作成されたことを確認してください。

↓

先ほど自動的に新規作成、変更されたファイルに対して、一部修正を加えていきます。

1. rails/config/routes.rb: 自動追加された User に対するルーティングを適切な定義（api/v1配下）に直します。
2. rails/app/models/user.rb: :cofirmable を有効にします。 :confimrmableを有効にすると、ユーザー新規登録の際に何らかの認証操作が必須となります。今回はメール認証を行います。

```diff rb:rails/config/routes.rb
  Rails.application.routes.draw do
-   mount_devise_token_auth_for "User", at: "auth"
    namespace :api do
      namespace :v1 do
        get "health_check", to: "health_check#index"
+       mount_devise_token_auth_for "User", at: "auth"
      end
    end
  end
```

```diff rb: rails/app/models/user.rb
  # frozen_string_literal: true

  class User < ApplicationRecord
    # Include default devise modules. Others available are:
-   # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
+   # :lockable, :timeoutable, :trackable and :omniauthable
    devise :database_authenticatable, :registerable,
-          :recoverable, :rememberable, :validatable
+          :recoverable, :rememberable, :validatable, :confirmable
    include DeviseTokenAuth::Concerns::User
  end
```

## セッションエラーの回避

ログアウトやユーザー削除で利用するDELETEリクエストを動作させるための追加設定を行います。

Rails7以降では、APIモードの際にSessionへのアクセスがあると、`ActionDispatch::Request::Session::DisabledSessionError`が発生します。

APIモードではSessionアクセスを想定していないためですが、deviseの中でSessionを触ってしまう部分あるためにこのエラーが発生してしまいます。したがって、deviseからSessionへのアクセスをうまく回避するような設定を行う必要があります。

- 参考: [ActionDispatch::Request::Session::DisabledSessionErrorについて](https://blog.furu07yu.com/entry/rails7-devise-token-auth#ActionDispatchRequestSessionDisabledSessionError%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6)

Session回避用の concerns ファイルを以下のように作成してください。

```rb:rails/app/controllers/concerns/devise_hack_fake_session.rb
module DeviseHackFakeSession
  extend ActiveSupport::Concern

  class FakeSession < Hash
    def enabled?
      false
    end

    def destroy
    end
  end

  included do
    before_action :set_fake_session

    private

      def set_fake_session
        if Rails.configuration.respond_to?(:api_only) && Rails.configuration.api_only
          request.env["rack.session"] ||= ::DeviseHackFakeSession::FakeSession.new
        end
      end
  end
end
```

その後、当ファイルを application_controller.rb で読み込んでください。

```diff rb:rails/app/controllers/application_controller.rb
  class ApplicationController < ActionController::API
    include DeviseTokenAuth::Concerns::SetUserByToken
+   include DeviseHackFakeSession
  end
```

これで devise 関連の DELETE メソッドも利用できるようになりました。

## deviseの日本語化

devise関連ファイルを日本語化します。今回のアプリでは主に認証メール（アカウントを仮登録したユーザーに対して送信する、認証URLを含んだメール）の日本語化を目的としています。

先ほどすでにそれを行うための gem `devise_i18n`をインストールしているので、このまま以下コマンドを実行してください。

```sh:rails
rails g devise:i18n:locale ja
```

`rails/config/locales/devise.views.ja.yml`が自動作成されればOKです。ちなみに認証メールに関係するのは、`ja.devise.mailer.confirmation_instructions`の部分になります。

![](https://storage.googleapis.com/zenn-user-upload/006e3efebce6-20230703.png)

その次に、Rails アプリ全体の言語を日本語に設定します。`rails/config/application.rb`に以下を追記してください。

```diff rb:rails/config/application.rb
 .
 .
 module Myapp
   class Application < Rails::Application
     .
     .
     config.api_only = true
+    config.i18n.default_locale = :ja
   end
 end
```

これを設定することで、例えば devise の認証メールを生成するときに、上記で生成した`devise.views.ja.yml`を参照してくれるようになります。

## 認証メールの送信設定

### 送信元の設定を追加

ユーザーに対して認証メールを送信するためには、認証メールの送信元のドメインを Rails の設定ファイル内に定義してあげる必要があります。

本番環境でのことは別途設定しますので、ここでは開発環境においての設定のみを行います。 rails/config/environments/development.rb に以下のように追記してください。

```diff rb:rails/config/environments/development.rb
 require "active_support/core_ext/integer/time"

 Rails.application.configure do
   .
   .
   # 認証メール送信に関する設定
+  config.action_mailer.default_options = { from: "no-replay@example.com" }
+  config.action_mailer.default_url_options = { host: "localhost:3000" }
 end
```

### gem `letter_opener_web` を追加

開発環境で送信されたメールを確認する gem である `letter_opener_web` を追加します。

- 参考: [【Rails】letter_opener_webを用いて開発環境で送信したメールを確認する方法](https://techtechmedia.com/letter_opener_web/)

```diff :rails/Gemfile
  .
  .
  group :development, :test do
+   # 開発環境でメール送信をテストする
+   gem "letter_opener_web"
+
    # pry コンソールを使えるようにする。
    gem "pry-byebug"
    gem "pry-doc"
    gem "pry-rails"
    .
    .
  end
```

```sh:railsコンテナ
bundle install
```

↓

letter_opener_web 用のルーティングを定義します。

```diff rb:rails/config/routes.rb
Rails.application.routes.draw do
+ mount LetterOpenerWeb::Engine, at: "/letter_opener" if Rails.env.development?
  .
  .
end
```

↓

letter_opener_web 用の設定を config/environments/development.rb に追加します。

```diff ruby:rails/config/environments/development.rb
require "active_support/core_ext/integer/time"

Rails.application.configure do
  .
  .
+ config.action_mailer.delivery_method = :letter_opener_web
end
```

一度 rails サーバーを再起動したあと、 http://localhost:3000/letter_opener にアクセスしてください。

一度、railsサーバーを再起動してください。

```sh:railsコンテナ
Ctrl+C
```

```sh:railsコンテナ
rails s -b "0.0.0.0"
```

以下のような画面が表示されればOKです！

![](https://storage.googleapis.com/zenn-user-upload/fe9b3ad44657-20230702.png)

## 動作確認準備

ここからは、devise_token_auth によってどのような機能が実装されたのかを確認していきましょう。

### 事前準備1 Talent API Tester の導入

APIの動作確認としては、Googke Chrome の拡張機能 [Talent API Tester] (https://chrome.google.com/webstore/detail/talend-api-tester-free-ed/aejoelaoggembcahagimdiliamlcdmfm?hl=ja)を使用します。Talent API Tester を用いることで、任意のURIに対してリクエストを送信することができます。今回は、これを用いて `localhost:3000` に対してリクエストを送信し、Userに関するAPIが意図通り動作するかを確認していきます。

拡張機能として追加したあと、画面右上（URL入力欄の右）の拡張機能アイコンから使用できます。

![](https://storage.googleapis.com/zenn-user-upload/f8c7ae659952-20230702.png)

METHOD, URL, その他パラメーターを設定してリクエストを送信できます。例えば、ヘルスチェック用アクションにリクエストを送信すると、以下のように`200 OK`が返ってくることが確認できます。

- METHOD: GET
- URL: http://localhost:3000/api/v1/health_check

![](https://storage.googleapis.com/zenn-user-upload/020e90917671-20230702.png)

また、入力した設定は「Save as」から保存することもできます。適当なプロジェクトを新規作成し、その配下に保存していきましょう。

![](https://storage.googleapis.com/zenn-user-upload/01f6252b1af7-20230702.png)

### 事前準備2 Sequel Ace 導入

Sequel Ace は開発環境のDB状況を可視化してくれるツールです。

レコード作成状況は。`$ rails c`かも確認することもできますが、Sequel Ace を入れておくと大変便利なのでおすすめです。

アプリは AppStore からインストールできます。

https://apps.apple.com/us/app/sequel-ace/id1518036000

※Homebrewからインストールしたい場合は以下を参考ください。

https://qiita.com/ucan-lab/items/b1304eee2157dbef7774

↓

アプリを導入できたら、`database.yml`と`docker-compose.yml`に記述された値に応じて、設定を入力していきます。

|項目|値|
|---|---|
|Name|myapp_development|
|Host|127.0.0.1|
|Username|root|
|Password|password|
|Database|myapp_development|
|Port|3307|

「Add to favorite」をしておくと、設定を保存できます。

![](https://storage.googleapis.com/zenn-user-upload/0ed5a9ff4d83-20230703.png)

↓

「Connect」を押下し、画面上部の「データベースを選択」から`myapp_development`を選択ください。以下のように、テーブルとして`users`が表示されていればOKです。

![](https://storage.googleapis.com/zenn-user-upload/59e59aa9c4ee-20230703.png)

:::message
ar_internal_metadata と schema_migration は Rails の DB にデフォルトで存在するテーブルなので、ここではあまり気にしなくてOKです。
:::

## 動作確認

動作確認を行なっています（ちなみに、deviseによってどのようなルーティングが設定されているのかは、`$ rails routes`から確認できます）

### User新規作成

まずは、 User の新規作成APIを動作させてみます。以下を入力して、「Send」ボタンを押下します。

|項目|値|
|---|---|
|METHOD|POST|
|URL|http://localhost:3000/api/v1/auth|
|HEADERS|Content-Type: application/json|
|BODY|以下参照|

```json:BODY
{
  "email": "test1@example.com",
  "password":"password",
  "confirm_success_url": "http://localhost:8000" # 認証完了後のリダイレクト先URLs
}
```

![](https://storage.googleapis.com/zenn-user-upload/d179fcc678f2-20230708.png)

↓

200 OK がレスポンスとして返ってくることを確認してください。

![](https://storage.googleapis.com/zenn-user-upload/17016cb2d95f-20230708.png)

Sequel Ace で確認すると、新規登録された users レコードが作られていることを確認できます。

![](https://storage.googleapis.com/zenn-user-upload/373e3d162c40-20230708.png)

ただし、この時点ではまだ未認証の状態で、アカウントとして有効化するためには認証作業が必要になります。deviseにおいては、未認証であることは、`user.confirmed_at`が空であることと同義と定義されています。

![](https://storage.googleapis.com/zenn-user-upload/5604d3f74f24-20230708.png)

↓

http://localhost:3000/letter_opener を開きメールを確認してみると、`test1@example.com`宛に認証用のメールが送信されています。

![](https://storage.googleapis.com/zenn-user-upload/6efbd541a555-20230708.png)

↓

「アカウントを有効化する」を押下するとユーザー認証が行われ、`confirm_success_url`で設定した http://localhost:8000 に自動的にリダイレクトされます。

![](https://storage.googleapis.com/zenn-user-upload/4990b712018c-20230708.png)

:::message
リダイレクト先のURLをよく見てみると、`?account_confirmation_success=true`というクエリーパラメーターが自動的に付与されています。

これを活用することで、例えばリダイレクト後の画面で「ユーザー認証に成功しました」のようなメッセージを表示させることができます。
:::

Sequel Ace で確認すると、`user.confirmed_at`に認証された日時が保存されていることが確認でき、認証済みであることが分かります。

![](https://storage.googleapis.com/zenn-user-upload/66ab02a3218d-20230708.png)

### ログイン

先ほど新規作成&認証が完了した user に対して、ログインAPIを動作させてみます。

|項目|値|
|---|---|
|METHOD|POST|
|URL|http://localhost:3000/api/v1/auth/sign_in|
|HEADERS|Content-Type: application/json|
|BODY|以下参照|

```json:BODY
{
  "email": "test1@example.com",
  "password":"password"
}
```

![](https://storage.googleapis.com/zenn-user-upload/68c3da09329a-20230708.png)

↓

200 OK がレスポンスとして返ってくることを確認してください。

![](https://storage.googleapis.com/zenn-user-upload/c770355ac8c7-20230708.png)

さらに、レスポンスのHEADERSの中に、`access-token`, `client`, `uid`が含まれていることを確認し、**中身をどこかにメモしておいてください**。この3つが、ログインユーザーを識別するための認証情報になります。

![](https://storage.googleapis.com/zenn-user-upload/901c64e1d90d-20230708.png)

devise_token_auth においては、これら3つの認証情報を、何らかの手段でブラウザ側で保存します。そして、これら認証情報をリクエストのヘッダー情報として含めることで、Railsは「test1`example.comさんからのリクエストだな」と判断し、ログインユーザーに適切なレスポンスを返します。

次章では、この認証情報を用いたAPIリクエストの実装を行います。
