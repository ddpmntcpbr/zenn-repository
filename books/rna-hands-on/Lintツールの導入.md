## この章でやること

`Next.js`に対して、Lintツールである`ESLint`と`Prettier`を導入します。

## ESLint / Prettier とは？

いずれも、Javascriptの静的コード解析ツール（Lintツール）です。

以前の章でRubyの静的コード解析ツールとして`rubocop`を導入しましたが、それのJavascript版だと思ってください。

ESLint, Prettier という二つのものを導入するのは、それぞれで規定・修正できるルールが異なるためです。`Next.js`に限らず、モダンなJavascriptフレームワークを導入している開発現場では、両者を併用することが多いと思われます。

## 手順

ESlintとPrettier、およびそれらの関連ライブラリをインストールします。下記コマンドを`next`コンテナで実行してください。

```sh:nextコンテナ
npm install --save-dev prettier eslint eslint-config-next typescript-eslint @typescript-eslint/eslint-plugin @typescript-eslint/parser eslint-config-prettier eslint-plugin-prettier eslint-plugin-react eslint-plugin-react-hooks eslint-plugin-import
```

:::message
ESlintとPretiierは開発環境でのみ使用するライブラリのため、`$ npm install --save-dev` としています
:::

`next/package.json`を確認すると、`devDependencies`の中で上記ライブラリのバージョンが明記され、`next/package-lock.json`でライブラリの依存関係が定義されていることが確認できます。

↓

コードルールを設定します。すでに作成されている`next/.eslintrc.json`に以下を記載してください（こちらは、https://github.com/gihyo-book/ts-nextbook-app/blob/main/.eslintrc.json を参考にしています）

```json:next/.eslintrc.json
{
  "extends": [
    "next",
    "next/core-web-vitals",
    "eslint:recommended",
    "plugin:prettier/recommended",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:import/recommended",
    "plugin:import/typescript"
  ],
  "rules": {
    "react/react-in-jsx-scope": "off",
    "import/order": [2, { "alphabetize": { "order": "asc" } }],
    "import/no-unresolved": "off",
    "prettier/prettier": [
      "error",
      {
        "trailingComma": "all",
        "endOfLine": "lf",
        "semi": false,
        "singleQuote": true,
        "printWidth": 80,
        "tabWidth": 2
      }
    ]
  }
}

```

↓

`next/package.json`の実行コマンド（`$ npm run xxx`）のうち、Lintツールの実行に関するコマンドを追加・修正します。

```diff next/package.json
{
  .
  .
  .
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
-   "lint": "next lint --dir src"
+   "lint": "next lint --dir src",
+   "format": "next lint --fix --dir src"
  },
  .
  .
  .
}
```

- `$ npm run lint`とすることで、`/src`ディレクトリ配下のファイルについて、コード解析を行います。今後、HTML構造に関するソースコードはすべて`/src`ディレクトリ配下で管理されるため、解析対象ディレクトリを`/src`のみとすることを明示しています。
- `$ npm run formt`とすることで、`/src`ディレクトリ配下のファイルについて、コード解析の結果ルールに違反している箇所があった場合、自動的に修正してくれるようになります。

↓

まずは`$ npm run lint`を実行してみます。

```sh:nextコンテナ
npm run lint
```

```
> my-app@0.1.0 format
> next lint --dir src
.
.
./src/pages/api/hello.ts
10:29  Error: Insert `,`  prettier/prettier

./src/pages/index.tsx
3:1  Error: `next/font/google` import should occur before import of `next/head`  import/order

info  - Need to disable some ESLint rules? Learn more here: https://nextjs.org/docs/basic-features/eslint#disabling-rules
```

2箇所のルール違反があることが分かります。

#### 補足

`$ npm run lint`を実行したとき、以下のような警告メッセージが表示される場合があります。

```
=============

WARNING: You are currently running a version of TypeScript which is not officially supported by @typescript-eslint/typescript-estree.

You may find that it works just fine, or you may not.

SUPPORTED TYPESCRIPT VERSIONS: >=3.3.1 <5.1.0

YOUR TYPESCRIPT VERSION: 5.1.3

Please only submit bug reports when using the officially supported version.

=============
```

`Next.js`の初期立ち上げ時にインストールした`typescript`のバージョンが高すぎることで、一部関連ライブラリの対応が追いついていないことの警告です。

こちらが出た場合は、`typescript`のバージョンを警告メッセージにしたがって少し下げてください（開発には影響ありません）。`package.json`を変更し、`$ npm install`でライブラリの依存関係を再定義します。

```diff json:next/package.json
{
  .
  .
  "dependencies": {
    "@types/node": "20.3.1",
    "@types/react": "18.2.12",
    "@types/react-dom": "18.2.5",
    "next": "13.4.6",
    "react": "18.2.0",
    "react-dom": "18.2.0",
-   "typescript": "5.1.3",
+   "typescript": "5.0"
  },
  .
  .
}

```

```sh:nextコンテナ
npm install
```

これで再度`$ npm run lint`を実行しても、警告メッセージが表示されなくなるはずです。

↓

次に`$ npm run format`を実行してみます。

```sh:nextコンテナ
npm run format
```

```
> my-app@0.1.0 format
> next lint --fix --dir src

✔ No ESLint warnings or errors
```

先ほどルール違反となっていた箇所が自動修正された上で、オールグリーンが表示されました。

## VSCodeの拡張機能追加

エディタとしてVSCodeを利用している場合はLintツールの利用がさらに便利になります。

拡張機能を利用することでファイル保存時にLintツールが働き、自動的に修正を加えてくれるようになります（実質、`$ npm run format`コマンドを実行する必要がなくなります）。

ここから導入手順を紹介しますが、みなさんのエディタの現時点での設定状況によって、対応が異なる可能性がある点はご了承ください（VSCodeでのLintツール適用については、解説記事が多く見つかるはずです）

まず拡張機能として、以下を導入してください。

- ESLint
- Prettier ESLint

![](https://storage.googleapis.com/zenn-user-upload/d8e73ae29e43-20230618.png)

![](https://storage.googleapis.com/zenn-user-upload/f9f0d0455535-20230618.png)

↓

VSCodeの`settings.json`を開き、以下を追加してください。これにより、ファイル保存時にESLintによる修正が実行されます。

- 参考: [【VSCode】settings.json の開き方](https://daeudaeu.com/vscode-settings-json/)

```diff json:settings.json
{
+ "editor.codeActionsOnSave": {
+   "source.fixAll.eslint": true
+ },
}
```

↓

一度 VSCode を再起動し、適当なファイルで検証してみます。

`next/src/pages/index.tsx`を開いてください。今回のLintルールでは、`import`するライブラリの順序は、参照元（`from`以降）のアルファベット順で並べるルールが適用されています。

![](https://storage.googleapis.com/zenn-user-upload/ea60176059b9-20230618.png)

↓

適当に順番を入れ替えると、警告シンタックスが表示されます。

![](https://storage.googleapis.com/zenn-user-upload/eb40c96040d5-20230618.png)

↓

この状態でファイル保存を行うと自動でLintツールによる修正が行われ、ライブラリの実装順序が元に戻されます（ルール通りに修正されます）

![](https://storage.googleapis.com/zenn-user-upload/127dfbdb86a2-20230618.png)
