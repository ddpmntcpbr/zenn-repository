## この章でやること

ホストOSのターミナルから、AWS CLIを用いてAWSにアクセスできる環境を構築します。

## AWS CLI

AWS CLI をインストールしていない方は、AWS CLI のインストールを行なってください。

https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/getting-started-install.html

以下コマンドを入力して、バージョンが返ってくれば正常にインストールできています（バージョン自体は下記のものと一致している必要はありません）。

```sh:ターミナル
aws --version
```

```
aws-cli/2.11.9 Python/3.11.2 Darwin/22.2.0 exe/x86_64 prompt/off
```

## IAM

IAMとは、AWSにアクセスするユーザーに対する権限を管理する機能です。

チーム開発では、複数のエンジニアが自社サービスを運用しているAWSクラウドにアクセスをすることになります。このとき、エンジニアごとに許可する権限を変更したい、ということはよく発生します（例えば、インフラエンジニアは広い権限を持つが、その他エンジニアは必要最低限の権限のみを持つようにする等）。

IAMで、ユーザーひとりひとりの権限を個別に設定することができます。

### ユーザーとグループの作成

特定の権限を付与したグループを作成する
↓
ユーザーを作成し、グループに登録する

ユーザー自体に権限を付与することも可能ですが、グループ側に権限を持たせることの方が一般的です。一度グループを定義してしまえば、仮にエンジニアが新規でジョインした場合も、そのグループに招待するだけで他エンジニアと同じ権限を持つことが可能になるからです。

### IAMユーザー作成

![](https://storage.googleapis.com/zenn-user-upload/f95bdf54d5a1-20230404.png)

![](https://storage.googleapis.com/zenn-user-upload/e2f4fa4af1f2-20230404.png)

![](https://storage.googleapis.com/zenn-user-upload/11c3ab50948c-20230404.png)

![](https://storage.googleapis.com/zenn-user-upload/1c620fbbf2e3-20230404.png)

![](https://storage.googleapis.com/zenn-user-upload/78dc9c80ef0f-20230404.png)

### AWS CLIを設定

AWS CLI に先ほどのIAMユーザーを設定します。

```sh:ターミナル
aws configure
```

```
AWS Access Key ID: xxxxxxxxxxxxxxxx
AWS Secret Access Key: xxxxxxxxxxxxxxxx
Default region name: ap-northeast-1
Default output format: json
```

これにより、IAMユーザーに付与された範囲の権限でAWS CLIが使用できるようになりました。
