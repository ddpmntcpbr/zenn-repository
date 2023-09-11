## この章でやること

Rails と Next.js の間の情報の疎通を行えるようにします。

## 概要

- Next.js に Rails にリクエストを送信するライブラリをインストールする
- Rails 側で Next.js からのリクエストを受信できる設定を行う
- Next.js から Rails のヘルスチェック用アクションを実行するリクエストを送信し、意図通りのレスポンスが受け取れることを確認する。

## 手順

まずは、Next.js で Rails に対してリクエストを送信するライブラリをインストールします。

Next.jsから外部APIにリクエストを送信する手段はいくつかあるのですが、ここでは`axios`と`SWR`を利用します。

:::message
`SWR`とは？

Next.jsの開発元である Vercel 社が提供しているデータフェッチライブラリです。

他のデータフェッチライブラリと比べてどのように便利であるのか、をいきなり説明するのは難しいのですが、公式が提供しているだけあって Next.js にとってとにかく融通が聞いてくれて楽、ということだけいったん分かってもらえればOKです。

当教材では、Railsからデータを取得する（GETメソッドのリクエスト）ときは`SWR`を利用し、Railsに対して何らかのデータ変更を命じる（POST, PATHC, DELETEなどのメソッドのリクエスト）ときは`axios`というライブラリを使用します。

`axios`に関しては、必要な場面でまた詳しく解説します。
:::

`next`コンテナに入り、`SWR`をインストールします。

```sh:nextコンテナ
npm install axios swr
```

`package.json`, `package-lock.json`が更新されていることを確認ください。

↓

axiosを用いてAPIからのデータ取得を行う関数(fetcher)を定義します。この関数は、今後さまざまな場面で汎用的に使用したいので、特定のページ内ではなく、共通利用することを前提にした位置に配置します。

`next/src/utils`ディレクトリを作成し、その配下に`index.ts`ファイルを作成してください。

```ts:next/src/utils/index.ts
import axios from 'axios'

export const fetcher = (url: string) =>
  axios
    .get(url)
    .then((res) => res.data)
    .catch((err) => {
      console.log(err)
      throw err
    })
```

引数で受け取った`url`に対して、GETメソッドのリクエストを送信します。その後、正常にレスポンスが帰ってきたら（`res`）、`res.data`を返り値として返し、何らかの異常が発生したら、エラー内容(`err`)をログ出力させた上で、エラーを返します。

↓

Next.jsのトップページにあたる `src/pages/index.tsx`を、以下のように書き換えてください。

```tsx:front/src/pages/index.tsx
import type { NextPage } from 'next'
import useSWR from 'swr'
import { fetcher } from '@/utils'

const Index: NextPage = () => {
  const url = 'http://localhost:3000/api/v1/health_check'
  const { data, error } = useSWR(url, fetcher)

  if (error) return <div>An error has occurred.</div>
  if (!data) return <div>Loading...</div>

  return (
    <>
      <div>Rails疎通確認</div>
      <div>レスポンスメッセージ: {data.message}</div>
    </>
  )
}

export default Index
```

`swr`ライブラリから取得した`useSWR`という関数を利用し、RailsAPIに対してリクエストを送っています。

`useSWR`の挙動としては、まず第一引数の`url`を、第二引数の`fetcher`に渡して`fetcher`を動作させます。その後、レスポンスが正常にその結果を`data`に、エラーが発生したらその内容を`error`に代入します。

`useSWR`を使用する場合は、以下のようなガード節を設けるのが一般的です。

```tsx:
  if (error) return <div>An error has occurred.</div>
  if (!data) return <div>Loading...</div>
```

もし`error`に値が含まれている場合は、専用のエラー表示を行います。この時点でリターンをしているため以下の処理は実行されず、この時点でリターンしたHTML要素が画面上に描画されます（当然、ここで返すHTMLは任意でカスタマイズできます）。

もし`error`が空かつ、さらに`data`も空である場合は、まだデータ取得が完了していない状態（正常とも異常とも判断できない）であるので、「ローディング中・・・」であることを示すようなHTML要素をリターンして画面上に描画させます（こちらも、返すHTMLは任意でカスタマイズできます）。

そして、`error`が空かつ、`data`に値が含まれている場合は、レスポンスを正常に受け取った証拠であるので、肝心の`<>...</>`部分をリターンする、という流れになります。

`useSWR`とガード節を組み合わせることで、画面読み込み直後のローディング表示と、レスポンス取得完了後のメイン画面表示の切り替えをシンプルに実装できます。

↓

今回はリクエストURLとしてRailsAPIのヘルスチェックアクション(`/api/v1/health_check`)を設定しているので、Railsとの疎通が正常に行えれば、`{data.message}`の部分に「Success Health Check!」が入ることになります。

しかし、この状態で http://localhost:8000/ にアクセスしてみると、「An error has occurred.」が表示されているはずです。

エラーになっている理由は、RailsAPI側でのCORS設定がまだ行われていないためです。

![](https://storage.googleapis.com/zenn-user-upload/7868ab6f8238-20230629.png)

## CORS設定

RailsAPI側でCORS設定を行い、Next.jsからのリクエストを許可します。

:::message
CORS（Cross-Origin Resource Sharing）はウェブページがそのオリジン（スキーマ、ホスト、ポート番号）と異なるオリジンのリソースに対するアクセスを許可するためのメカニズムです。

あらゆるオリジンからのアクセスを許可してしまうと、悪意を持ったユーザーから攻撃され放題になってしまい大変危険です。したがって、アクセスを許可するオリジンには何らかの制約を設けることが一般的です。

Railsでは、デフォルトでは自分自身（自分と同じのオリジン）以外からのアクセスは全て拒否をする設定となっていますので、今回は Next.js からのリクエストを特別に許可してあげる設定を行います。
:::

CORS設定を制御できる gem である `rack-cors`を追加します。

https://github.com/cyu/rack-cors

Gemileに追記し、`$ bundle install`を行なってください。

```diff :rails/Gemfile
.
.
# pumaサーバーを使えるようにする（標準gem）
gem "puma", "~> 5.0"

+ # cors設定を管理する
+ gem 'rack-cors'

# rails本体（標準gem）
gem "rails", "~> 7.0.4"
.
.
```

```sh:railsコンテナ
bundle install
```

`Gemfile.lock`が更新されることを確認ください。

↓

CORS設定を行う設定ファイルである`rails/config/initializers/cors.rb`がデフォルトで存在していると思いますので、これを以下のように書き換えてください。

```rb:rails/config/initializers/cors.rb
# Be sure to restart your server when you modify this file.

# Avoid CORS issues when API is called from the frontend app.
# Handle Cross-Origin Resource Sharing (CORS) in order to accept cross-origin AJAX requests.

# Read more: https://github.com/cyu/rack-cors

Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins "http://localhost:8000"

    resource "*",
      headers: :any,
      methods: [:get, :post, :put, :patch, :delete, :options, :head]
  end
end
```

Next.js のオリジンにあたる`http://localhost:8000`を、CORSの許可リストとして追加しました。

一度、railsサーバーを再起動させてください。

```sh:railsコンテナ
Ctrl + C
```

```sh:railsコンテナ
rails s -b '0.0.0.0'
```

railsサーバーが再起動したら、再び http://localhost:8000 に戻り、画面をリロードしてください。「Success Health Check!」が表示されていればOKです！

![](https://storage.googleapis.com/zenn-user-upload/b575d8dc7729-20230629.png)

これで Next.js と Rails の疎通が確認できました！

## dotenv-rails による環境変数の管理

さて、Next.js と Rails の疎通はここまでで問題ないのですが、将来を見据えてひとつ設定を付け加えます。

先ほど、CORSを許可するオリジンとして`http://localhost:8000`を記載しましたが、Next.js のオリジンがこれであるのはあくまで開発環境のみです。

この後、AWSにデプロイした場合は、これとは別の文字列が Next.js のオリジンとなります。したがって、CORSが許可するオリジンは、アプリケーションがおれている環境ごとに切り替える必要があります。

このような、環境ごとに異なる値を設定できる変数を、一般に**環境変数**と呼びます。Railsにおいて環境変数を制御する方法はいくつかありますが、ここでは Rails の環境変数を管理する最もポピュラーな gem である `config`　を利用します。

https://github.com/rubyconfig/config

Gemfileに `dotenv-rails` を追加し、`$ bundle install`します。

```diff :rails/Gemfile
.
.
# railsの起動時間を短縮する（標準gem）
gem "bootsnap", require: false

+ # 環境毎の設定管理を行う
+ gem "config"
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

`rails/config`配下に`settings`ディレクトリを作成し、さらにその配下に`development.yml`と`test.yml`を作成してください。

```yml:rails/config/settings/development.yml
front_domain: http://localhost:8000
```

```yml:rails/config/settings/test.yml
front_domain: http://localhost:8000
```

フロントエンドのドメインとして、`http://localhost:8000`を保存しました。この値は、`Settings.front_domain`という形で取り出すことができます。

また、アプリケーションが動作している環境に応じて、

- 開発環境: development.yml
- テスト環境: test.yml
- 本番環境: production.yml

の設定ファイルを切り替えて参照してくれます。この機能を利用することで、環境ごとにことなる変数を別々に定義することができます。

今は開発環境、テスト環境で同じ記述のみを行なっていますが、将来的には本番環境用の設定ファイル`production.yml`においては、異なる記述を行うことになります（本番環境では、ここは実際にインターネット上に公開されるアプリのドメインになる）

↓

`cors.rb`を以下のように修正してください。

```diff rb:rails/config/initializers/cors.rb
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do

-   origins http://localhost:8000
+   origins Settings.front_domain

    resource "*",
      headers: :any,
      methods: [:get, :post, :put, :patch, :delete, :options, :head]
  end
end
```


定数の`http://localhost:8000`を、環境変数`Settings.front_domain`に置き換えることで、環境の応じたオリジン許可が行われるようになります。

一度 Rails サーバーを再起動し、 http://localhost:8000/ から再度アクセスして、疎通が問題なく行えていることを確認してください。

```sh:railsコンテナ
Ctrl + C
```

```sh:railsコンテナ
rails s -b '0.0.0.0'
```

![](https://storage.googleapis.com/zenn-user-upload/5e70e9ca1464-20230702.png)
