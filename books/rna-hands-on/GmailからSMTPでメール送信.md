---
title: "はじめに「未経験転職は難しい。だからこそ」"
---

## この章やること

Gmailを活用したメールサーバーを構築して、本番環境でサインアップ機能を動作できるようにします。

前章までで本番環境へのアプリデプロイは完了できましたが、本番環境においてメール配信を行うインフラが整っていないために、サインアップが行えない状態にありました。

今回は、Gmailのアプリ開発機能を活用して、SMTPでメールを送信する構築をつくります。

:::message
AWS内にも、`AWS SES`というメール配信サービスが存在していますが、利用のための審査があること、有料であることから、採用を見送りました。

Gmailを活用すれば、審査なしかつ無料でメール配信を行うことができます。
:::

#### 参考記事

[Rails6でGmailからSMTPでメール送信する方法](https://labo.kon-ruri.co.jp/rails-send-mail-via-gmail-smtp/#index_id0)

## 流れ

- Googleアカウントのアプリパスワードを発行
- 本番環境でGmailからSMTPでメール送信するようRailsを修正
- Railsイメージの再プッシュ&タスク再起動

## 手順

### Googleアカウントのアプリパスワードを発行

お使いのGoogleアカウントにて、アプリパスワードを発行します。Googleのアカウントページから「セキュリティ」にアクセスしてください。

![](https://storage.googleapis.com/zenn-user-upload/f8b5ede8d8ef-20230603.png)

「Google にログインする方法 > 2 段階認証プロセス」から、任意の手段で2段階認証プロセスを有効にします（すでに有効にしている場合はスキップ）。

![](https://storage.googleapis.com/zenn-user-upload/976b7eab1518-20230603.png)

↓

![](https://storage.googleapis.com/zenn-user-upload/136a1b9fecdc-20230603.png)

↓

「2 段階認証プロセス」ページの画面最下部にある「アプリ パスワード」に入ります。

![](https://storage.googleapis.com/zenn-user-upload/2d4210bd4500-20230603.png)

↓

以下の内容を入力してください。

|項目|値|
|---|---|
|アプリを選択|その他（名前を入力）|
|アプリ名|任意（rails-next-zenn-clone-app）|

![](https://storage.googleapis.com/zenn-user-upload/2814f6f22173-20230603.png)

![](https://storage.googleapis.com/zenn-user-upload/444e311d114c-20230603.png)

↓

英数字16字のパスワードが生成されるので、画面を閉じる前に忘れずにメモをしておきます。

![](https://storage.googleapis.com/zenn-user-upload/8f09b9d60682-20230603.png)

### 本番環境でGmailからSMTPでメール送信するようRailsを修正

creadentialsに、「Googleアカウントのメールアドレス」と「アプリパスワード」を保存します。

```sh:railsコンテナ
EDITOR="vi" bin/rails credentials:edit
```

`production.gmail.username`にメールアドレスを、`production.gmail.password`に先ほど発行したアプリパスワードを入力してください。

```
  production:
    .
    .
    gmail:
      user_name: xxxxxxxxx@gmail.com
      password: xxxxxxxxxxxxxxxxxxxx
```

↓

`$ rails c`で各データにアクセスできることを確認してください。

```sh:railsコンテナ
rails c
```

```sh:railsコンテナ > railsコンソール
Rails.application.credentials.production.gmail.user_name
```

```
=> xxxxxxxxx@gmail.com
```

```sh:railsコンテナ > railsコンソール
Rails.application.credentials.production.gmail.password
```

```
=> xxxxxxxxxxxxxxxxxxxx
```


### Railsイメージの再プッシュ&タスク再起動

RailsイメージをECRに再プッシュし、その後backendタスクを再起動して、コード差分を反映させてください。

手順は「タスク(backend)の再作成」の章をご確認ください。

## 動作確認

本番環境のサインアップページにアクセスし、自信が受信できるメールアドレスを用いてサインアップを行なってください。

![](https://storage.googleapis.com/zenn-user-upload/49631d7d5b0a-20230823.png)

↓

「送信する」ボタンをクリックすると、メールアドレス宛に認証メールが送信されます。「アカウントを有効化する」をクリックしてください。

![](https://storage.googleapis.com/zenn-user-upload/08c2a8a3f496-20230823.png)

↓

アカウントの有効化が完了し、サインインページにリダイレクトされますので、登録したメールアドレス、パスワードでサインインできることを確認してください。

![](https://storage.googleapis.com/zenn-user-upload/7b0177df8f26-20230823.png)
