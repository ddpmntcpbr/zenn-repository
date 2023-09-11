---
title: "はじめに「未経験転職は難しい。だからこそ」"
---

## この章でやること

Route53を利用して、アプリに使用する独自ドメインを取得します。

当教材では、`rails-next-zenn-clone-app.com`という文字列でドメインを取得した前提で進めておりますが、読者の皆さんは各々でオリジナルの文字列で独自ドメインを取得いただくこととなります。

## Route53とは?

AWSが提供するDNSサービスです。DNS（Domain Network Service）とは、インターネット上のドメインとIPアドレスの対応づけを行う役割を果たすものです。

お名前.comなどの別サービスで独自ドメインを取得することも可能ですが、AWS上でアプリケーションを運用する場合は、Route53でドメインを取得する方が諸々の設定が楽になります。

## 注意点

- 当教材では、`rails-next-zenn-clone-app.com`という文字列でドメインを取得した前提で進めておりますが、読者の皆さんは各々でオリジナルの文字列で独自ドメインを取得いただくこととなります。
- **独自ドメインの保有には料金が発生します**。ドメインによって金額は異なりますが、当教材で取得したドメインでは**年間13ドル**でした。相場感としてご参考ください。
- 教材執筆時期の関係上、一部スクリーンショットの画面が、AWSの旧コンソールレイアウトになっております。新コンソールレイアウトの場合とは画面の見た目が少し異なると思いますが、手順の流れは変わっておりません。

## 手順

検索窓からRoute53にアクセスし、**ドメインの登録**の入力欄に取得したいドメイン名を入力して、「チェック」します。

![](https://storage.googleapis.com/zenn-user-upload/30d85c6e695b-20230506.png)

↓

好きなトップレベルドメイン（.comや.netのこと）を選択して、「続行」します。

![](https://storage.googleapis.com/zenn-user-upload/c4d9b06be231-20230506.png)

↓

ドメインを購入するために個人情報を入力して、「続行」します。

![](https://storage.googleapis.com/zenn-user-upload/f082285bb822-20230506.png)

↓

自動更新の設定を任意で選択し、規約にチェックを入れて「注文を完了」します。

![](https://storage.googleapis.com/zenn-user-upload/3cbb5d0b52f1-20230506.png)

↓

注文リクエスト完了画面に遷移します。この時点ではまだ購入は確定していません。

![](https://storage.googleapis.com/zenn-user-upload/4f4861533273-20230506.png)

↓

初めてRoute53でドメインを取得する場合、メールアドレス宛に認証メールが送信されます。

![](https://storage.googleapis.com/zenn-user-upload/acd054bc6658-20230506.png)

↓

メール中のURLをクリックして認証を完了させてください。

![](https://storage.googleapis.com/zenn-user-upload/b9c6e12117a7-20230506.png)

↓

AWS側のシステムに特に問題がなければ、認証完了数分以内でドメインの取得が完了するはずです。Route53サイドバー「ドメイン > 登録済みドメイン」から取得したドメインを確認できればOKです！

![](https://storage.googleapis.com/zenn-user-upload/4bee59554b47-20230816.png)
