---
title: "はじめに「未経験転職は難しい。だからこそ」"
---

## この章でやること

- テスト実装に必要な gem である `factory_bot_rails` と `faker` を導入します。
- 上記を用いて、いくつかのテストを実際に実装してみます。

## factory_bot_rails を導入

factory_bot_rails はひとことで言うと**テストデータの定義を簡単にしてくれるgem**です。

https://github.com/thoughtbot/factory_bot_rails

実際に実装しながら学習してきましょう。まずは gem をインストールします。

```diff :rails/Gemfile
  .
  .
  group :development, :test do
    # テスト用データを作成する
+   gem "factory_bot_rails"
    .
    .
  end
```

```sh:railsコンテナ
bundle install
```

:::message
本番環境では使用しないgemであるので、`group :development, :test`配下で定義しています。
:::

インストールが完了したら、[公式ドキュメント](https://github.com/thoughtbot/factory_bot/blob/master/GETTING_STARTED.md#rspec)の記述にしたがい、 factory_bot_rails を Rspec で利用するための設定を行います。

`rails/spec/rails_helper.rb`に、以下の記述を追加してください。

```diff rb:rails/spec/rails_helper.rb
  .
  .
  RSpec.configure do |config|
+   # FactoryBotの宣言を省略
+   config.include FactoryBot::Syntax::Methods
    .
    .
```

準備が整いましたので、実装に入っていきます。今回は、前章で実装した`User`モデルに対するテスト（モデルスペック）を実装してみたいと思います。

まず、factory_bot_rails を活用して`User`モデルのレコードをテストデータとして用意するための準備を行います。`rails/spec/factories`ディレクトリを作成し、配下に`users.rb`を作成してください。

```rb:rails/spec/factories/users.rb
FactoryBot.define do
  factory :user do
    name { "田中太郎" }
    sequence(:email) {|n| "#{n}_" + "test@example.com" }
    password { "password" }
    confirmed_at { Time.current }
  end
end
```

イメージとしては、rspec内で users レコードを作成した場合、`factories/users.rb`の定義を参照してカラムの値を設定してくれるようになります。

`email`だけ書き方を異なっているのは、`User`モデルにおける`email`カラムのユニーク制約（複数のレコードで同じ値を持つことができない）を考慮しているためです。同じテスト内で users レコードを複数作成した場合、`n`の作成したレコード数に応じた数字が入るようになります。つまりは、

- 1_test@example.com
- 2_test@example.com
- 3_test@example.com
- ...

といった文字列を email に利用してくれるので、ユニーク制約を回避することできます。

これを用いて、モデルスペックを実装してみます。`rails/spec/models`ディレクトリを新規作成し、配下に`user_spec.rb`を作成してください。

```rb:rails/spec/models/user_spec.rb
require "rails_helper"

RSpec.describe User, type: :model do
  context "factoryのデフォルト設定に従った場合" do
    let(:user) { create(:user) }

    it "認証済みの user レコードを正常に新規作成できる" do
      expect(user).to be_valid
      expect(user).to be_confirmed
    end
  end
end
```

`create(:user)`の記述が、users レコードを作成している箇所になります。このときに`factories/users.rb`の情報が参照されており、`name`、`email`、`password`が入った状態の users レコードを生成してくれています。

その後、実際のテスト実行を記述する`it`節の中で、`expect(user).to be_valid`としており、「user は valid であること（バリデーションエラーがないこと）」を検証しています。

加えて`expect(user).to be_confirmed`で、 user が認証済み（`confirmed_at`カラムに値が含まれていること）も検証しています。

この時点でテストを実行してみると、テストが通ることが確認できるはずです。

```sh:railsコンテナ
 rspec spec/models/user_spec.rb
```

```
.

Finished in 4 minutes 8.8 seconds (files took 10.82 seconds to load)
1 example, 0 failures
```

:::message
it節内に`binding.pry`を挿入した状態でテストを実行することで、テスト実行途中でコードを止めて状況を確認することができます。

これを活用して user レコードの中身を見てみると、name, email が factories で定義した通りになっていることが確認できるはずです。
:::

## faker を導入

factory_bot_rails でテストデータを手軽に用意する方法を学びましたが、毎回 user.name
が「田中太郎」固定になってしまうのは、テストとしては本来よろしくはありません。他の文字列を使用した場合にも安定してシステムが動作するのかを検証できた方が、テストとしては優秀といえます。

ここでは、テストデータにランダム性を持たせるために`faker`というgemを導入します。

https://github.com/faker-ruby/faker

faker は一言でいうと、**それらしい文字列、数列、その他オブジェクトをランダムに生成してくれるgem**です。実際のRailsの開発現場では、factory_bot_rails と faker を組み合わせてテストデータを用意するケースが多いです。

こちらも導入してみましょう。gemをインストールします。

```diff :rails/Gemfile
  .
  .
  group :development, :test do
    # テスト用データを作成する
    gem "factory_bot_rails"
+   gem "faker"
    .
    .
  end
```

```sh:railsコンテナ
bundle install
```

特に準備とかはいらず、この時点で利用できるようになっています。いったん、テストに利用する前にコンソールを使って動作を確認してみましょう。

```sh:railsコンテナ
rails c
```

`Faker::XXX.xxx`のような形で記述することで、さまざまな文字列・数列をランダム生成してくれます。例えば、`Faker::Name.name`とすると、人名っぽい文字列を生成してくれます。

```sh:railsコンテナ > pryコンソール
[1] pry(main)> Faker::Name.name
=> "前田 颯"

[2] pry(main)> Faker::Name.name
=> "酒井 蓮"

[3] pry(main)> Faker::Name.name
=> "木下 仁"
```

コマンドを実行するたびに、出力結果が異なっていることが分かります。

他にもさまざまなコマンドが用意されておりますので、[詳しくは公式リファレンスを参照ください。](https://www.rubydoc.info/gems/faker/)

↓

faker を用いて、`factories/users.rb`を書き換えてみましょう。

```diff rb:rails/spec/factories/users.rb
  FactoryBot.define do
    factory :user do
-     name { "田中太郎" }
-     sequence(:email) {|n| "#{n}_" + "test@example.com" }
-     password { "password" }
+     name { Faker::Name.name }
+     sequence(:email) {|n| "#{n}_" + Faker::Internet.email }
+     password { Faker::Internet.password(min_length: 10) }
      confirmed_at { Time.current }
    end
  end
```

name は先ほどの`Faker::Name.name`で、名前っぽい文字列をランダムで生成して当てはめるようにしています。

email は`Faker::Internet.email`という、メールアドレスのような形式の文字列を生成するコマンドを使用しています。これと`sequence`を組み合わせて、メールアドレスの一意性を担保しています。

```
[1] pry(main)> Faker::Internet.email
=> "olevia_cummerata@keebler.example"

[2] pry(main)> Faker::Internet.email
=> "lavina@koelpin.example"

[3] pry(main)> Faker::Internet.email
=> "lacy@bahringer.test"
```

password　は`Faker::Internet.password`という、パスワードに使用されていそうな文字列をランダムに生成しています。 min_length で、生成する最小文字数を指定しています。

```
[1] pry(main)> Faker::Internet.password(min_length: 10)
=> "64i4ixzKRBaIp3GU"

[2] pry(main)> Faker::Internet.password(min_length: 10)
=> "rF1KeREl4ff3HS"

[3] pry(main)> Faker::Internet.password(min_length: 10)
=> "bWWlPALHspSRJv5"
```

この状態でテストを実行しても、問題なく通ることが確認できるかと思います。

```sh:railsコンテナ
rspec spec/models/user_spec.rb
```

```
.

Finished in 4 minutes 8.8 seconds (files took 10.82 seconds to load)
1 example, 0 failures
```

## ログインユーザー取得アクションのリクエストスペック実装

前章で実装した、ログインユーザー取得アクションのリクエストスペックを実装していきます。`$ rails g ...`で自動生成されている`rails/spec/requests/api/v1/current/users_spec.rb`を以下のように書き換えてください。

```rb:rails/spec/requests/api/v1/current/users_spec.rb
require "rails_helper"

RSpec.describe "Api::V1::Current::Users", type: :request do
  describe "GET api/v1/current/user" do
    subject { get(api_v1_current_user_path, headers:) }

    let(:current_user) { create(:user) }
    let(:headers) { current_user.create_new_auth_token }

    context "ヘッダー情報が正常に送られた時" do
      it "正常にレコードを取得できる" do
        subject
        res = JSON.parse(response.body)
        expect(res.keys).to eq ["id", "name", "email"]
        expect(response).to have_http_status(:ok)
      end
    end

    context "ヘッダー情報が空のままリクエストが送信された時" do
      let(:headers) { nil }

      it "unauthorized エラーが返る" do
        subject
        res = JSON.parse(response.body)
        expect(res["errors"]).to eq ["ログインもしくはアカウント登録してください。"]
        expect(response).to have_http_status(:unauthorized)
      end
    end
  end
end
```

テストの流れを説明します。

まず、`subject { get(api_v1_current_user_path, headers:) }`で、`api_v1_current_user_path`へのGETリクエストの送信を定義しています。

このとき、`headers:`で、後に定義している`headers`変数をヘッダー情報として含めています。

:::message
`subject { get(api_v1_current_user_path, headers:) }`
と
`subject { get(api_v1_current_user_path, headers: headers) }`
は同じ動作をします。
:::

`let(:current_user) { create(:user) }`で、factoriesファイルにしたがっての user レコードの作成が定義されています。

その後の`let(:headers) { current_user.create_new_auth_token }`では、`current_user`のログインに必要なトークン情報を生成し、`headers`への格納を定義しています。

ここでひとつ、Rspec の大原則をお伝えすると、subject や let での定義は**遅延評価**される仕様となっています。通常のプログラミングコードは上から順番に実行されますが、subject や let は、それが呼び出されるタイミングで初めて評価される（遅延評価）、という仕様になっています。

つまりは、実際の実行順序は以下のようになっています。

- 上から順番にコードが読み込まれていくが、subjectやletの中身は実行されないまま進む
- it節「正常にレコードを取得できる」の実行が開始する
- 1行目で subject が呼び出されているので、ここで初めて subject をみにいく
- subject を定義するためには headers を必要なので、 let(:headers) をみにいく
- let(:headers) を定義するためには current_user が必要なので let(:current_user) をみにいく
- let(:current_user) の中身 `create(:user)` が実行され、user レコードが current_user に格納される
- let(:headers) に戻り、headers にトークン情報を格納される
- subject に戻り、headersをヘッダー情報として含んで、 api_v1_current_user_path へのGETリクエストが送信される（subjectの処理が完了する）
- it節「正常にレコードを取得できる」に戻り、 subject で実行されたリクエストに対するレスポンスが response に格納される
- `res = JSON.parse(response.body)`で、response のボディー情報をjson形式に変換する
- `expect(res.keys).to eq ["id", "name", "email"]`で、response のボディー情報に基づく json データのキーが、`["id", "name", "email"]`であることを評価する
- `expect(response).to have_http_status(:ok)`で、response のステータスコードが 200 OK であることを評価する
- 次のit節「unauthorized エラーが返る」の実行が開始する
- ...

なんとなく、rspecの処理が進んでいくイメージが持てるようになってきたでしょうか？

このコードでテストを実行すると、テストが通ることが確認できるはずです。

```sh:railsコンテナ
rspec spec/requests/api/v1/current/users_spec.rb
```

```
..

Finished in 1.59 seconds (files took 11.71 seconds to load)
2 examples, 0 failures
```
