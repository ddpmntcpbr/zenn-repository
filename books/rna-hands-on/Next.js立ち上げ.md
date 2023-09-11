---
title: "はじめに「未経験転職は難しい。だからこそ」"
---

## この章でやること

今回のアプリのフロントエンドを担う`Next.js`の立ち上げを行います。

## Next.jsとは？

[Next.js](https://nextjs.org/)とは、Javascriptライブラリの`React.js`を基盤としたWebフレームワークです。

「Next.jsどころかReact.jsすら分かりません」という方でも、コードはすべて記載しておりますし、必要な知識の分は都度解説していきますので、安心して先に進んでください。

## 手順

`Next.js`が起動するための新しいコンテナを`docker-compose.yml`に定義します。

まず一度、起動しているdockerコンテナを停止させます。


```sh:/
docker compose stop
```

↓

アプリルート直下に`next`ディレクトリを作成し、その配下に`Dockerfile`を配置してください。

```sh:/
mkdir next
```

```dockerfile:next/Dockerfile
FROM node:19.4.0
WORKDIR /app
```

ディレクトリ構造はこのようになります。

```
.
├── docker-compose.yml
├── rails
|   ├── ..
|   .
|   .
└── next
    └── Dockerfile
```

↓

`docker-compose.yml` に `next` コンテナに関する設定を追加します。

```diff yml:docker-compose.yml
version: '3'
services:
  db:
    .
    .
  rails:
    .
    .
+ next:
+   build:
+     context: ./next/
+     dockerfile: Dockerfile
+   tty: true
+   stdin_open: true
+   volumes:
+     - ./next:/app
+   ports:
+     - "8000:3000"
volumes:
  mysql_data:
```

↓

`next`コンテナを追加した状態で、コンテナのビルド&起動を行います（ビルドの際、nextコンテナに必要な node の docker image が新規でダウンロードされます）。

```sh:/
docker compose build
```

```sh:/
docker compose up -d
```

`db`、`rails` に加えて`next`コンテナも起動します（`$ docker ps`で確認できます）。

`next`コンテナに入り、node が正しくインストールされていることを確認します。

```sh:/
docker compose exec next /bin/bash
```

```sh:nextコンテナ
node -v # nodeのバージョンを確認
```

```
v19.4.0
```

`next/Dockerfile`で指定したバージョンの node が正しくインストールされていることが分かります。

この中で、`Next.js`をインストールを実行します。

```sh:nextコンテナ
npx create-next-app@latest
```

#### コマンドの補足

- `npx create-next-app`: Next.jsをインストールするためのコマンド
- `@latest`: バージョン`13.1.1`の Next.js をインストールする。ここを`@latest`にすることで最新バージョンをインストールできますが、今回は教材での開発アプリと挙動と同じにするためにバージョンを固定します。

↓

どのような設定で`Next.js`を構築するか質問されるので、以下のように答えてください。

|質問|回答|説明|
|---|---|---|
|Need to install the following packages: create-next-app@x.x.x|y|x.x.x のNext.jsをインストールしますか？|
|What is your project named?|my-app|アプリ名|
|Would you like to use ESLint with this project?|Yes|ESLintの使用（ESLintは別章で解説します）|
|Would you like to use Tailwind CSS with this project?|No|Tailwind CSSの使用（CSSフレームワークの一種。今回は使用しません）|
|Would you like to use `src/` directory with this project?|Yes|HTML構造に関するディレクトリを `/src` 配下に配置する|
|Use App Router (recommended)?|No|App Routerの使用（今回は使用しません）|
|Would you like to customize the default import alias?|Yes|ライブラリimportのエイリアス設定|
|What import alias would you like configured?|@/*|エイリアスを`@/`に設定|

↓

`Next.js`のディレクトリ構成が生成されました。

```
next
├── Dockerfile
└── my-app
    ├── README.md
    ├── next-env.d.ts
    ├── next.config.js
    ├── node_modules
    ├── package-lock.json
    ├── package.json
    ├── public
    ├── src
    └── tsconfig.json
```

使いやすくするために、my-app ディレクトリ配下のファイルをを全て1階層上に上げ、my-app ディレクトリは削除してください。

```
next
├── Dockerfile
├── README.md
├── next-env.d.ts
├── next.config.js
├── node_modules
├── package-lock.json
├── package.json
├── public
├── src
│   ├── pages
│   └── styles
└── tsconfig.json
```

これで`Next.js`の準備ができましたので、nextサーバーを起動してみましょう。`$ npm run dev`でサーバーを起動した後、`http://localhost:8000` からアクセスができます。

```sh:nextコンテナ
npm run dev
```

![](https://storage.googleapis.com/zenn-user-upload/56a47b282a88-20230617.png)

:::message
Next.jsの開発サーバーはデフォルトでは3000番ポートを使用します(localhost:3000)。ただし今回は、docker-compose.ymlで、ホストの8000番ポートとnextコンテナの3000番ポートを接続しているため、ホスト側からは8000番ポート(localhost:8000)でアクセスができます。
:::

nextサーバーを停止したいときは、nextコンテナ内で `Ctrl+C` を押下してください。
