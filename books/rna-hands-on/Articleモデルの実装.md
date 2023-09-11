## この章でやること

記事を表すテーブルである`Article`モデルを実装します。

## モデル設計

`Article`モデルにどのようなカラムを持たせるべきかを、サービス仕様の観点から整理してみます。

まず、今回実装するzennライクサービスでは、ひとつのユーザーが複数の記事を作成・投稿できる機能を想定しているので、**`users`と`articles`は1対Nのリレーションを持つ**ことが必要と分かります。

次に、記事には**タイトル**と**本文**が存在するので、それに該当するカラムを用意したいと思います。

そして、記事には「下書き」と「公開済」の二つの状態が存在するので、これを管理するための**ステータス**カラムを用意したいと思います。

ここまでは一般的に思い至る設計ですが、最後の**ステータス**にもうひと工夫をして、よりzennライクに近づけてみたいと思います。

## zennの記事保存の仕組み

zennで記事を書いたことがある人はご存知かと思うのですが、zennでは「記事の執筆」と「記事の保存/更新」が**ページ遷移を挟まずにシームレス**に実現されています。

![](https://storage.googleapis.com/zenn-user-upload/747167d9500d-20230713.gif)

この**ページ遷移を挟まずにシームレス**というのは、Railsがあまり得意ではありません。フロントも含めてRailsのみでアプリを作成する場合は、データの保存が行われたときはページ遷移を発生させることが一般的です。

また、「新規作成画面」と「編集画面」でのルーティングが異なっていることが一般的であり、Railsの一般仕様に則る形ではzennのシームレスな記事保存の挙動を実現できません。

そこで今回は、これを実現するためのトリックを設けます。記事ステータスのひとつとして**未保存**という属性を追加し、記事保存のロジックを以下のように設計します。

- ユーザーが、フロント「記事の新規作成画面」にアクセスしたとき、そのユーザーに紐づく**未保存**の記事レコードの有無を参照する。
- **未保存**の記事がない場合は、**未保存**の記事を新規作成し、そのレコードのidをフロントに返す。**未保存**の記事がある場合は、すでにあるそのレコードのidをフロントに返す。
- フロント「記事の新規作成画面」の「保存」ボタンには、そのidに対する「更新」アクションの発火点を設置する。

つまりは、ユーザーが画面上の「保存」ボタンを押すよりも前に記事レコードのidを確定させてしまうことで、「記事の新規作成画面」を事実上「記事の編集画面」と同一にしてしまう、というアイディアになります。

これにより、「新規作成画面」と「編集画面」の区別をフロント側でする必要がなくなり、自然とシームレスな記事保存を実装できるようになります。

ここのテキストベースの説明だけだとちょっと理解しにくいかもしれません。最終的にフロントまで含めて実装を終えた段階では「そういうことか」理解できるかと思いますので、ここではいったん**ステータスとして、未保存、という属性も持つようにする**ということだけを理解してもらえればOKです。

## モデルの実装

以下コマンドを実行して、Article モデルを追加します。

```sh:railsコンテナ
rails g model Article
```

マイグレーションファイルを以下のように書き換えて、マイグレーションを実行してください。

```rb:rails/db/migrate/xxxxxxxxxxxxxx_create_articles.rb
class CreateArticles < ActiveRecord::Migration[7.0]
  def change
    create_table :articles do |t|
      t.string :title, comment: "タイトル"
      t.text :content, comment: "本文"
      t.integer :status, comment: "ステータス（10:未保存, 20:下書き, 30:公開済み）"
      t.references :user, null: false, foreign_key: true

      t.timestamps
    end
  end
end
```

```sh:railsコンテナ
rails db:migrate
```

schema.rb に artices に関する記述を追加されるかと思います。

↓

Article モデルに各種設定を追加します。まずは、`users`と`articles`を1対Nのリレーションで結びます。

```diff rb:rails/app/models/article.rb
  class Article < ApplicationRecord
+   belongs_to :user
  end
```

```diff rb:rails/app/models/user.rb
  class User < ApplicationRecord
    .
    .
+   has_many :articles, dependent: :destroy
  end
```

:::message
`dependent: :destroy`は、親レコードである users 側が削除された場合に、子レコードである articles も一緒に削除する、というオプションです。

- [Railsガイド > Active Record の関連付け > 1 関連付けを使う理由](https://railsguides.jp/association_basics.html#has-many%E9%96%A2%E9%80%A3%E4%BB%98%E3%81%91)
:::

次に、status カラムに関する enum を定義します。

```diff rb:rails/app/models/article.rb
  class Article < ApplicationRecord
    belongs_to :user
+   enum :status, { unsaved: 10, draft: 20, published: 30 }, _prefix: true
  end
```

enum についてよく分からない方は以下が参考になります。

- [【Rails】Enumってどんな子？使えるの？](https://qiita.com/ozackiee/items/17b91e26fad58e147f2e)
- [Rails5 から enum 使う時は_prefix（接頭辞）_suffix（接尾辞）を使おう](https://qiita.com/emacs_hhkb/items/fce19f443e5770ad2e13)

最後に、いくつかのバリデーションを追加します。

```diff rb:rails/app/models/article.rb
  class Article < ApplicationRecord
    belongs_to :user
    enum :status, { unsaved: 10, draft: 20, published: 30 }, _prefix: true
+   validates :title, :content, presence: true, if: :published?
+   validate :verify_only_one_unsaved_status_is_allowed
+
+   private
+
+     def verify_only_one_unsaved_status_is_allowed
+       if unsaved? && user.articles.unsaved.present?
+         raise StandardError, "未保存の記事は複数保有できません"
+       end
+     end
  end
```

少し補足します。まず、`validates :title, :content, presence: true, if: :published?`では、記事が公開済みである（status == "publided"）である場合のみ、title と content が空であってはならない、というルールです（タイトル、本文が空のまま記事を公開することはできない）

次の、`validate :verify_only_one_unsaved_status_is_allowed`では、ひとりのユーザーが、複数の未保存ステータス記事を保有することを防ぐルールです。通常の操作上これは発生しないことが想定されているので、エラーメッセージの返却ではなく、例外を発生させるようにしています（これが発生している時点でシステムとしては異常なことが起こってしまっているとみなす）

## エラーメッセージの日本語化

以上の実装を踏まえて articles に対するモデルスペックを実装していきたいのですが、それの前にバリデーションエラーが発生したときのエラーメッセージを日本語にする設定を行います。

- 参考: [Railsの日本語化とエラーメッセージのカスタマイズ](https://zenn.dev/machamp/articles/rails-validation-message)

gem `rails-i18n`をインストールしてください。

```diff :rails/Gemfile
  # rails本体（標準gem）
  gem "rails", "~> 7.0.4"

+ # メッセージを日本語化
+ gem "rails-i18n"
+
  # タイムゾーン情報を提供する（標準gem）
  gem "tzinfo-data", platforms: %i[mingw mswin x64_mingw jruby]
```

```sh:railsコンテナ
bundle install
```

`rails/config/application.rb`に以下を追記し、メッセージを日本語化するためのファイルを読み込んでもらうようにします。

```diff rb:rails/config/application.rb
      .
      .
      config.i18n.default_locale = :ja

+     config.i18n.load_path += Dir[Rails.root.join("config/locales/**/*.yml").to_s]
    end
  end
```

そして、メッセージの日本語翻訳に該当するファイル`rails/config/locales/ja.yml`を新規作成します。

```yml:rails/config/locales/ja.yml
ja:
  activerecord:
    attributes:
      article:
        title: タイトル
        content: 本文
```

これを追加したことによって、`article`レコードにおいてエラーメッセージが生成されるとき、`title`カラムは`タイトル`、`content`カラムは`本文`、という日本語に置き換えてメッセージが威生成されるようになりました（具体的にどうなるかは、以下のテストを参照ください）。

## テストの実装

以上の実装を踏まえ、articles に対するモデルスペックを実装していきます。まずは factories を用意します。

```rb:rails/spec/factories/articles.rb
FactoryBot.define do
  factory :article do
    user
    title { Faker::Lorem.sentence }
    content { Faker::Lorem.paragraph }
    status { :published }
  end
end
```

:::message
factory_bot_rails の仕様として「article の親テーブルが user である」ことがモデルファイル側で定義されている場合は、単に`user`という記述を付け加えるだけで、自動的に user レコードも一緒に生成してくれるようになります。

このときに生成される user レコードは、 factories/users.rb の情報を参照して生成されます。
:::

facotires の定義を踏まえ、以下のように article_spec を実装してください。

```rb:rails/spec/models/article_spec.rb
require "rails_helper"

RSpec.describe Article, type: :model do
  context "factoryのデフォルト設定に従った時" do
    subject { create(:article) }

    it "正常にレコードを新規作成できる" do
      expect { subject }.to change { Article.count }.by(1)
    end
  end

  describe "Validations" do
    subject { article.valid? }

    let(:article) { build(:article, title:, content:, status:, user:) }
    let(:title) { Faker::Lorem.sentence }
    let(:content) { Faker::Lorem.paragraph }
    let(:status) { :published }
    let(:user) { create(:user) }

    context "全ての値が正常な時" do
      it "検証が通る" do
        expect(subject).to be_truthy
      end
    end

    context "ステータスが公開済みかつ、タイトルが空の時" do
      let(:title) { "" }

      it "エラーメッセージが返る" do
        expect(subject).to be_falsy
        expect(article.errors.full_messages).to eq ["タイトルを入力してください"]
      end
    end

    context "ステータスが公開済みかつ、本文が空の時" do
      let(:content) { "" }

      it "エラーメッセージが返る" do
        expect(subject).to be_falsy
        expect(article.errors.full_messages).to eq ["本文を入力してください"]
      end
    end

    context "ステータスが未保存かつ、すでに同一ユーザーが未保存ステータスの記事を所有していた時" do
      let(:status) { :unsaved }
      before { create(:article, status: :unsaved, user:) }

      it "例外が発生する" do
        expect { subject }.to raise_error(StandardError)
      end
    end
  end
end
```

facotires にしたがってレコードが正常に生成されること、定義したバリデーションが機能していること、を検証しています。

テストを実行してみると、全て通ることが確認できます。

```sh:railsコンテナ
rspec spec/models/article_spec.rb
```

```
.....

Finished in 2.49 seconds (files took 19.91 seconds to load)
5 examples, 0 failures
```
