---
title: "はじめに「未経験転職は難しい。だからこそ」"
---

## この章でやること

railsアプリの初期設定として、必要になるいくつかのgemを導入します。

## 手順

### pryコンソールを導入

デバックを簡単に行えるpryコンソールを導入します。`Gemfile`に以下のgemを追加してください（本番環境では使用しないため、開発環境、テスト環境でのみインストールしています。）

```diff ruby:rails/Gemfile
.
.
+ group :development, :test do
+   # pry コンソールを使えるようにする。
+   gem "pry-byebug"
+   gem "pry-doc"
+   gem "pry-rails"
+ end
```

`$ bundle install`で gem ファイルをインストールします。

```sh:railsコンテナ
bundle install
```

`$ rails c`で立ち上がるコンソールが`pry`になっていることを確認します。

```sh:railsコンテナ
# rails c
Loading development environment (Rails 7.0.4)
[1] pry(main)>
```

アクションの中で`binding.pry`コードを挿入することにより、挿入箇所でコード実行を一時停止してデバッグができるようになります、

試しに、先ほど作成したヘルスチェック用アクションに`binding.pry`を挿入し、再度`http://localhost:3000/api/v1/health_check`にアクセスしてみます。

```diff ruby:rails/app/controllers/api/v1/health_check_controller.rb
class Api::V1::HealthCheckController < ApplicationController
  def index
+   binding.pry
    render json: { message: "Success Health Check!" }, status: :ok
  end
end
```

処理が待機中のまま停止し、`$ rails s -b '0.0.0.0'`でサーバーを立ち上げているターミナル画面を確認すると、デバッグモードに入れていることが確認できます。

```sh:railsコンテナ
From: /myapp/app/controllers/api/v1/health_check_controller.rb:4 Api::V1::HealthCheckController#index:

    2: def index
    3:   binding.pry
 => 4:   render json: { message: "Success Health Check!" }, status: :ok
    5: end

[1] pry(#<Api::V1::HealthCheckController>)>
```

開発中に、コードを部分的に止めて確認を指定場合に活用できます。

### rspecを導入

Railsのテストフレームワークとして最も一般的である`rspec`を導入します。

:::message
Railsのデフォルトのテストフレームワークには「minitest」という別のものが採用されていますが、日本で Rails を採用している企業のほとんどがテストフレームワークには rspec を採用しています。
:::

Gemfileにgemを追加し、インストールします。

```diff ruby:rails/Gemfile
.
.
group :development, :test do
  .
  .
+  # テストフレームワーク rspec を導入する
+  gem 'rspec-rails'
end
```

```sh:railsコンテナ
bundle install
```

↓

インストールが完了したら、下記コマンドを実行して、 rspec の初期設定ファイルを作成します。

```sh:railsコンテナ
rails g rspec:install
```

以下の設定ファイルが作られます。

|ファイル名|役割|
|---|---|
|spec/spec_helper.rb|rspec全体の設定ファイル|
|spec/rails_helper.rb|railsに関するrspecの設定ファイル|
|.rspec|rspecの実行環境に関する設定を行うファイル|

初期状態では`spec_helper.rb`のみが読み込まれている状態です。

`.rspec`を以下のように変更します。

```diff
- --require spec_helper
+ --require rails_helper
+ --format documentation
```

`.rspec`の初期設定では`spec_helper.rb`のみを読み込んでいたところを、`rails_helper.rb`を読み込むように変更しています。ただし、`rails_helper.rb`の中で`spec_helper.rb`を読み込む処理が記述されているので、実質的に両方とも読み込んでいることになっています。

また、`--format documentation`を記述することで、テスト実行時のログを閲覧できるようにしています。

## health_check_controller#index に対するテスト

rspecを導入したので、試しにヘルスチェックコントローラーに対するテストを作成します。

rspecのテストは、`/spec`ディレクトリ配下のファイルに記述していきます。

:::message
`spec`というのは、`rspec`の世界における「テスト」という意味に解釈してください。

また、`request`というのは、いわゆるアクションの動作を確認するためのテストのことで「リクエストスペック」と言います。

他に頻繁に作成するのが、モデルの性質をテストする「モデルスペック」で、これは`spec/models`配下にファイルを作成していきます。
:::

↓

`spec/requests/api/v1`ディレクトリ構造を作成し、その配下に`health_check_spec.rb`を新規作成してください。

```ruby:rails/spec/requests/api/v1/health_check_spec.rb
require "rails_helper"

RSpec.describe "Api::V1::HealthCheck", type: :request do
  describe "GET api/v1/health_check" do
    subject { get(api_v1_health_check_path) }

    it "正常にレスポンスが返る" do
      subject
      res = JSON.parse(response.body)
      expect(res["message"]).to eq "Success Health Check!"
      expect(response).to have_http_status(:success)
    end
  end
end
```

:::message
テストの読み方を簡単に解説します。

テストを行う対象を定義するのが`subject`です。当テストでは、/api/v1/health_check へ GETリクエストを送信する操作を`subject`として定義しています。

その後、`it`節で具体的なテストを実行しています。`subject`で定義した操作を実行した後、返ってきたレスポンスのボディーが以下を満たしていることを検証しています。

- `{ "message": "Success Health Check!" }`が含まれていること
- レスポンスのHTTPコードが200（サクセス）であることを評価しています。
:::

↓

試しにテストを実行しましょう。その前に、教材の手順に通りに進めているとまだDBを作成していないはずなので、この段階でDBを作成します（テスト環境のDBが未作成だとrspecが正常に起動しない）。

以下コマンドでDBを作成してください。


```sh:railsコンテナ
rails db:create
```

```
Created database 'myapp_development'
Created database 'myapp_test'
```

↓

DBが作成できたら、テストを動かすことができます。`rails`コンテナ内で`$ rspec`を実行することで、全てのテストを実行することができます。また、`spec`ディレクトリからの相対パスをコマンドオプションとして指定することで、特定のファイルだけをテストを実行することもできます。

```sh:railsコンテナ
rspec spec/requests/api/v1/health_checks_spec.rb
```

```
Api::V1::HealthChecks
  GET api/v1/health_check
    正常にレスポンスが返る

Finished in 0.34633 seconds (files took 5.32 seconds to load)
1 example, 0 failures
```

テストがグリーンになっていたら成功です。

#### 補足

Railsチュートリアルのみしか経験がない人にとっては、rspecによるテストは馴染みがなく、とっつきづらく感じるかもしれません。実際、rspec特有の書き方を覚える必要があるので、自力で書けるようになるにはある程度の慣れが必要になります。

当教材ではrspec初見の方でも学習を進められるよう、全てのテストコードを教材内に記載しています。コピペでも構わないので、まずは習うより慣れる、を優先いただければと思います。

rspecについて本格的に学習をしたい場合は、下記の教材が最も有名でオススメできるものになります。

- [Everyday Rails - RSpecによるRailsテスト入門](https://leanpub.com/everydayrailsrspec-jp/)

### rubocopの導入

rubyの静的コード解析ツールの`rubocop`を導入します。

:::message
rubocopを導入することにより、定義したコーディングルールに従って、コードのフォーマット、命名規則、コードの複雑さ、冗長性、タイポ等が自動でチェックされるようになります。

特に複数人でアプリを開発する場合には、誰か書いても同じようなコードに修正されていくため、より読みやすく、よりメンテナンスしやすいコードで実装ができるようになっていきます。
:::

Gemfileにgemを追加し、インストールします。

```diff ruby:rails/Gemfile
.
.
group :development, :test do
  .
  .
+   # rubocop を使えるようにする。
+   gem "rubocop-faker"
+   gem "rubocop-rails"
+   gem "rubocop-rspec"
end
```

次に、rubocopのルールを定義するファイルを作成します。

全てのルールを`Enabled`にすることもできますが、その場合かなり厳しめのルールになってしまいますので、ある程度はルールを緩和してあげたほうがよいかと思います。以下のリポジトリのルールがちょうど扱いやすく設定されているので、今回はこれを基準にカスタマイズしたものを使用していきます。

https://github.com/onk/onkcop/tree/master/config

まず、config配下にrubocopディレクトリを作成し、以下ファイルを作成してください。

- config/rubocop/rubocop.yml
- config/rubocop/rails.yml
- config/rubocop/rspec.yml

各ファイルに以下の内容を記述してください。

```yml:rails/rubocop.yml
# 自動生成されるものはチェック対象から除外する
AllCops:
  Exclude:
    - "node_modules/**/*" # rubocop config/default.yml
    - "vendor/**/*" # rubocop config/default.yml
    - "db/schema.rb"
    - "db/migrate/*"
    - "bin/**"
    - "spec/rails_helper.rb"
    - "spec/spec_helper.rb"

#################### Layout ################################
Layout/ClassStructure:
  Enabled: true

Layout/DotPosition:
  EnforcedStyle: trailing

Layout/EmptyLinesAroundAttributeAccessor:
  Enabled: true

Layout/ExtraSpacing:
  Exclude:
    - "db/migrate/*.rb"

Layout/FirstArrayElementIndentation:
  EnforcedStyle: consistent

Layout/FirstHashElementIndentation:
  EnforcedStyle: consistent

Layout/IndentationConsistency:
  EnforcedStyle: indented_internal_methods

Layout/MultilineMethodCallIndentation:
  EnforcedStyle: indented_relative_to_receiver

Layout/SpaceInsideBlockBraces:
  SpaceBeforeBlockParameters: false

Layout/SpaceAroundMethodCallOperator:
  Enabled: true

#################### Lint ##################################

Lint/AmbiguousBlockAssociation:
  Exclude:
    - "spec/**/*_spec.rb"

Lint/DeprecatedOpenSSLConstant:
  Enabled: false

Lint/DuplicateElsifCondition:
  Enabled: true

Lint/EmptyWhen:
  Enabled: false

Lint/InheritException:
  EnforcedStyle: standard_error

Lint/MixedRegexpCaptureTypes:
  Enabled: true

Lint/UnderscorePrefixedVariableName:
  Enabled: false

Lint/UnusedMethodArgument:
  Enabled: false

Lint/Void:
  CheckForMethodsWithNoSideEffects: true

Lint/RaiseException:
  Enabled: true

Lint/StructNewOverride:
  Enabled: true

Layout/LineLength:
  Max: 160
  Exclude:
    - "db/migrate/*.rb"

#################### Metrics ###############################
Metrics/AbcSize:
  Max: 30

Metrics/BlockLength:
  Exclude:
    - "Rakefile"
    - "**/*.rake"
    - "spec/**/*.rb"
    - "Gemfile"
    - "Guardfile"
    - "config/environments/*.rb"
    - "config/routes.rb"
    - "config/routes/**/*.rb"
    - "*.gemspec"
    - "db/seeds.rb"

Metrics/CyclomaticComplexity:
  Max: 10

Metrics/MethodLength:
  Max: 20
  Exclude:
    - "db/migrate/*.rb"

Metrics/PerceivedComplexity:
  Max: 8

#################### Naming ################################

Naming/PredicateName:
  ForbiddenPrefixes:
    - "is_"
    - "have_"
  NamePrefix:
    - "is_"
    - "have_"

Naming/MethodParameterName:
  Enabled: false

#################### Security ##############################
Security/YAMLLoad:
  Enabled: false

#################### Style #################################
Style/AccessorGrouping:
  Enabled: true

Style/Alias:
  EnforcedStyle: prefer_alias_method

Style/AndOr:
  EnforcedStyle: conditionals

Style/ArrayCoercion:
  Enabled: false

Style/AsciiComments:
  Enabled: false

Style/BisectedAttrAccessor:
  Enabled: false

Style/BlockDelimiters:
  AutoCorrect: false
  Exclude:
    - "spec/**/*_spec.rb"

Style/CaseLikeIf:
  Enabled: false

Style/ClassAndModuleChildren:
  Enabled: false

Style/CollectionMethods:
  PreferredMethods:
    detect: "detect"
    find: "detect"
    inject: "inject"
    reduce: "inject"

Style/Documentation:
  Enabled: false

Style/DoubleNegation:
  Enabled: false

Style/EmptyCaseCondition:
  Enabled: false

Style/EmptyElse:
  EnforcedStyle: empty

Style/EmptyMethod:
  EnforcedStyle: expanded

Style/FormatString:
  EnforcedStyle: percent

Style/FrozenStringLiteralComment:
  Enabled: false

Style/GuardClause:
  MinBodyLength: 5

Style/HashAsLastArrayItem:
  Enabled: false

Style/HashLikeCase:
  Enabled: false

Style/HashSyntax:
  Exclude:
    - "**/*.rake"
    - "Rakefile"

Style/IfInsideElse:
  Enabled: false

Style/IfUnlessModifier:
  Enabled: false

Style/Lambda:
  EnforcedStyle: literal

Style/MethodCalledOnDoEndBlock:
  Enabled: true

Style/MixinUsage:
  Exclude:
    - "bin/setup"
    - "bin/update"
    - "spec/dummy/bin/setup"
    - "spec/dummy/bin/update"

Style/NumericLiterals:
  MinDigits: 7
  Strict: true

Style/NumericPredicate:
  Enabled: false

Style/OrAssignment:
  Enabled: false

Style/PerlBackrefs:
  AutoCorrect: false

Style/PreferredHashMethods:
  EnforcedStyle: verbose

Style/RedundantAssignment:
  Enabled: true

Style/RedundantFetchBlock:
  Enabled: false

Style/RedundantFileExtensionInRequire:
  Enabled: false

Style/RedundantReturn:
  AllowMultipleReturnValues: true

Style/RedundantSelf:
  Enabled: false

Style/RescueStandardError:
  EnforcedStyle: implicit

Style/RedundantRegexpCharacterClass:
  Enabled: false

Style/RedundantRegexpEscape:
  Enabled: false

Style/SafeNavigation:
  Enabled: false

Style/Semicolon:
  Exclude:
    - "spec/**/*_spec.rb"

Style/SlicingWithRange:
  Enabled: true

Style/StringLiterals:
  EnforcedStyle: double_quotes

Style/StringLiteralsInInterpolation:
  EnforcedStyle: double_quotes

Style/StringMethods:
  Enabled: true

Style/SymbolArray:
  Enabled: false

Style/TernaryParentheses:
  EnforcedStyle: require_parentheses_when_complex

Style/TrailingCommaInArguments:
  EnforcedStyleForMultiline: comma

Style/TrailingCommaInArrayLiteral:
  EnforcedStyleForMultiline: comma

Style/TrailingCommaInHashLiteral:
  EnforcedStyleForMultiline: comma

Style/WordArray:
  Enabled: false

Style/ZeroLengthPredicate:
  Enabled: false

Style/ExponentialNotation:
  Enabled: true

Style/HashEachMethods:
  Enabled: true

Style/HashTransformKeys:
  Enabled: true

Style/HashTransformValues:
  Enabled: true
```

```ruby:web/.rubocop.yml
require:
  - rubocop-rails

inherit_from:
  - config/rubocop/rubocop.yml
  - config/rubocop/rails.yml
  - config/rubocop/rspec.yml

AllCops:
  TargetRubyVersion: 3.1.2
```

```yml:rails/rails.yml
Rails:
  Enabled: true

Rails/ActiveRecordCallbacksOrder:
  Enabled: true

Rails/BulkChangeTable:
  Enabled: false

Rails/Delegate:
  Enabled: false

Rails/Exit:
  Enabled: false

Rails/FindById:
  Enabled: false

Rails/FilePath:
  Enabled: false

Rails/Inquiry:
  Enabled: false

Rails/MailerName:
  Enabled: true

Rails/MatchRoute:
  Enabled: false

Rails/NegateInclude:
  Enabled: false

Rails/Pluck:
  Enabled: false

Rails/PluckInWhere:
  Enabled: false

Rails/Present:
  Enabled: false

Rails/RenderInline:
  Enabled: false

Rails/RenderPlainText:
  Enabled: false

Rails/SafeNavigation:
  ConvertTry: true

Rails/SaveBang:
  Enabled: true

Rails/ShortI18n:
  Enabled: false

Rails/UnknownEnv:
  Environments:
    - development # rubocop default.yml
    - test # rubocop default.yml
    - production # rubocop default.yml

Rails/WhereExists:
  Enabled: false
```

```yml:rails/config/rspec.yml
require: "rubocop-rspec"

RSpec/AnyInstance:
  Enabled: false

RSpec/ContextWording:
  Enabled: false

RSpec/DescribedClass:
  EnforcedStyle: explicit

RSpec/EmptyLineAfterFinalLet:
  Enabled: false

RSpec/ExampleLength:
  Max: 12

RSpec/ExpectChange:
  EnforcedStyle: block

RSpec/ImplicitExpect:
  Enabled: false

RSpec/InstanceVariable:
  Enabled: false

RSpec/MultipleExpectations:
  Enabled: false

RSpec/LetSetup:
  Enabled: false

RSpec/NamedSubject:
  Enabled: false

RSpec/NestedGroups:
  Max: 4

RSpec/ReturnFromStub:
  Enabled: false

RSpec/MultipleMemoizedHelpers:
  Enabled: true
  Max: 8
  AllowSubject: false
```

↓

次に、これらルール定義ファイルを読み込むためのファイルを作成します。

- `rails/.rubocop.yml`

```yml:rails/.rubocop.yml
require:
  - rubocop-rails

inherit_from:
  - config/rubocop/rubocop.yml
  - config/rubocop/rails.yml
  - config/rubocop/rspec.yml

AllCops:
  TargetRubyVersion: 3.1.2
```

↓

これで準備ができましたので、rubocop による静的コード解析を実行してみます。`rails`コンテナで`$ rubocop`を実行することで、（`config/rubocop/rubocop.yml`で除外設定していない）すべての rails ファイルについて、コード解析を行います。

```sh:railsコンテナ
rubocop
```

すると、様々な箇所で定義したルールに違反していることが分かります。

```
.
.
.
Inspecting 21 files
C..........CCC.......

Offenses:

Gemfile:2:22: C: [Correctable] Layout/SpaceInsideBlockBraces: Space between { and | detected.
git_source(:github) { |repo| "https://github.com/#{repo}.git" }
                     ^
Gemfile:28:7: C: [Correctable] Style/StringLiterals: Prefer double-quoted strings unless you need single quotes to avoid extra backslashes for escaping.
  gem 'rspec-rails'
      ^^^^^^^^^^^^^
config/environments/development.rb:25:7: C: [Correctable] Style/TrailingCommaInHashLiteral: Put a comma after the last item of a multiline hash.
      "Cache-Control" => "public, max-age=#{2.days.to_i}"
      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
config/environments/development.rb:56:1: C: [Correctable] Layout/EmptyLines: Extra blank line detected.
config/environments/production.rb:16:37: C: [Correctable] Layout/ExtraSpacing: Unnecessary spacing detected.
  config.consider_all_requests_local       = false
                                    ^^^^^^
config/environments/production.rb:16:44: C: [Correctable] Layout/SpaceAroundOperators: Operator = should be surrounded by a single space.
  config.consider_all_requests_local       = false
                                           ^
config/environments/production.rb:44:22: C: [Correctable] Layout/SpaceInsideArrayLiteralBrackets: Do not use space inside array brackets.
  config.log_tags = [ :request_id ]
                     ^
config/environments/production.rb:44:34: C: Layout/SpaceInsideArrayLiteralBrackets: Do not use space inside array brackets.
  config.log_tags = [ :request_id ]
                                 ^
config/environments/production.rb:74:50: C: [Correctable] Style/GlobalStdStream: Use $stdout instead of STDOUT.
    logger           = ActiveSupport::Logger.new(STDOUT)
                                                 ^^^^^^
config/environments/test.rb:22:5: C: [Correctable] Style/TrailingCommaInHashLiteral: Put a comma after the last item of a multiline hash.
    "Cache-Control" => "public, max-age=#{1.hour.to_i}"
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

21 files inspected, 10 offenses detected, 9 offenses autocorrectable
```

↓

これらをひとつひとつ手作業で修正するのはしんどいですね。 rubocop には、単なるコード解析ではなく、コードルールに沿った自動修正の機能もあります。

自動修正を行うためには、`-A`オプションをつけます。

```sh:railsコンテナ
rubocop -A
```

```
.
.
21 files inspected, 9 offenses detected, 9 offenses corrected
```

git 差分を見てみると、コードの変更が行われていることが確認できるかと思います。また、再度`$ rubocop -A`を実行してもこれ以上はコード修正が行われないことも確認できるはずです。
