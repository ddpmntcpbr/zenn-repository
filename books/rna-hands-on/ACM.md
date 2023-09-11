## この章でやること

ACM(AWS Certificate Manager)によって、Route53で取得した独自ドメインのSSL化を行います。

## ACMとは？

ACM(AWS Certificate Manager)は、取得した独自ドメインにSSL証明書をインストールすることができるサービスです。

ドメインに対して SSL（Secure Sockets Layer）証明書を付与することによって、HTTPSプロトコルによる通信をできるようになり、よりセキュアな環境でアプリを公開できるようになります。

## 手順

AWSコンソールヘッダー左部の検索窓から「AWS Certificate Manager」にアクセスします。

![](https://storage.googleapis.com/zenn-user-upload/729389fe5e8a-20230513.png)

↓

証明書をリクエスト > パブリック証明書をリクエスト、を選択して「次へ」と進みます。

![](https://storage.googleapis.com/zenn-user-upload/46eb5bbb1514-20230513.png)

![](https://storage.googleapis.com/zenn-user-upload/cb85ebf32cad-20230513.png)

↓

Route53で取得したドメイン名を、以下のように入力します。

|完全修飾ドメイン名|
|---|
|ドメイン名|
|*.ドメイン名|

`*`はワイルドカードです。こちらの設定により、サブドメイン（ www.ドメイン名 など ）全てを含めてSSL申請ができます。

![](https://storage.googleapis.com/zenn-user-upload/045e261a7a57-20230513.png)

↓

検証方法として「DNS」を選択し、「リクエスト」を押下します。

![](https://storage.googleapis.com/zenn-user-upload/8f38ded3ab73-20230513.png)

リクエストが完了しました。続いて、DNSの検証を行います。

サイドバー「証明書を一覧」から、該当の証明書リクエストの詳細画面に入ります。

![](https://storage.googleapis.com/zenn-user-upload/6f34c8d5ed10-20230513.png)

![](https://storage.googleapis.com/zenn-user-upload/e9be2a0eee71-20230513.png)

↓

「Route53でレコードを作成」を押下します。

![](https://storage.googleapis.com/zenn-user-upload/aa2078f07555-20230513.png)

↓

デフォルトで作成するレコードの設定が表示されるので、そのまま「レコードを作成」を押下します。

![](https://storage.googleapis.com/zenn-user-upload/f8146ddd6adb-20230513.png)

レコードの作成が完了しました。数分待つと、証明書リクエストのステータスが「発行済み」に更新されます。

![](https://storage.googleapis.com/zenn-user-upload/5fd1b14560cc-20230513.png)

これで、Route53で取得したドメインのSSL化が行えました！
