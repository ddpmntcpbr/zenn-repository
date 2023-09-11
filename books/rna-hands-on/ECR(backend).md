## この章でやること

Dockerfile.prod に基づいてビルドした railsコンテナとnginxコンテナを、AWS ECR にプッシュします。

## ECRとは？

ECR(Elastic Container Registry)は、Dockerコンテナを保存しておけるリポジトリです。

ローカルでDockerコンテナをビルドし、ECRに対してプッシュします。これを、ECSでデプロイして運用する、という流れとなっていきます。

## 手順

検索窓から「Elastic Container Registry > Repositries」に入り、「リポジトリを作成」を押下します。

![](https://storage.googleapis.com/zenn-user-upload/0a2f4d0b52ce-20230516.png)

↓

backend用に、1.rails, 2.nginxの二つを作成します。

### rails

以下項目を入力して、「リポジトリを作成」を押下します。

|項目|値|
|---|---|
|可視性設定|プライベート|
|リポジトリ名|zenn-clone-rails|

![](https://storage.googleapis.com/zenn-user-upload/b4b01741b643-20230516.png)

↓

リポジトリが作成できてたら、リポジトリの詳細画面に入り、「プッシュコマンドの表示」を押下します。

![](https://storage.googleapis.com/zenn-user-upload/61da7595cf9d-20230516.png)

![](https://storage.googleapis.com/zenn-user-upload/c41d1d14ad30-20230516.png)

↓

こちらのコマンドを手元のターミナルで実行していきますが、一部今回の設定に合わせて微修正が必要です。

#### 1. 認証トークンを取得し、レジストリに対して Docker クライアントを認証します。

表示されているコマンドをコピペして実行します。

ログインが正常に完了すると、ターミナル上で`Login Succeeded`と表示されます。

#### 2. 以下のコマンドを使用して、Docker イメージを構築します。一から Docker ファイルを構築する方法については、「こちらをクリック 」の手順を参照してください。既にイメージが構築されている場合は、このステップをスキップします。

Dockerfileのファイル名および相対パスの位置に合わせてコマンドを修正し、アプリのトップの位置で実行します。

```sh:ホストOS(./)
docker build -t zenn-clone-rails -f ./rails/Dockerfile.prod ./rails
```

狙いのイメージが作成されていることを確認します。

```sh:ホストOS(./)
$ docker images

REPOSITORY         TAG       IMAGE ID       CREATED         SIZE
zenn-clone-rails   latest    d509fae6a32d   3 minutes ago   1.35GB
```

#### 3. 構築が完了したら、このリポジトリにイメージをプッシュできるように、イメージにタグを付けます。

表示されているコマンドをコピペして実行します。

イメージに対して狙いのタグが付与されていることを確認します。

```sh:ホストOS(./)
$ docker images

REPOSITORY                                                           TAG       IMAGE ID       CREATED         SIZE
297087675121.dkr.ecr.ap-northeast-1.amazonaws.com/zenn-clone-rails   latest    d509fae6a32d   4 minutes ago   1.35GB
.
.
```

#### 4. 以下のコマンドを実行して、新しく作成した AWS リポジトリにこのイメージをプッシュします

表示されているコマンドをコピペして実行します。

ECRにrailsのイメージがプッシュされていればOKです。

![](https://storage.googleapis.com/zenn-user-upload/667ec303316c-20230516.png)

### nginx

基本的な手順はrailsと同様です。

再度リポジトリの作成画面に入り、以下項目を入力して、「リポジトリを作成」を押下します。

|項目|値|
|---|---|
|可視性設定|プライベート|
|リポジトリ名|zenn-clone-nginx|

![](https://storage.googleapis.com/zenn-user-upload/3d116fa95960-20230516.png)

↓

リポジトリが作成できてたら、リポジトリの詳細画面に入り、「プッシュコマンドの表示」を押下します。

![](https://storage.googleapis.com/zenn-user-upload/2e0fc2d13d7a-20230516.png)

![](https://storage.googleapis.com/zenn-user-upload/288978abc3ee-20230516.png)

#### 1. 認証トークンを取得し、レジストリに対して Docker クライアントを認証します。

表示されているコマンドをコピペして実行します。

（railsイメージをプッシュしたターミナルを閉じていなければログイン状態が保持されているので、実はスキップしても大丈夫です）。

#### 2. 以下のコマンドを使用して、Docker イメージを構築します。一から Docker ファイルを構築する方法については、「こちらをクリック 」の手順を参照してください。既にイメージが構築されている場合は、このステップをスキップします。

Dockerfileのファイル名および相対パスの位置に合わせてコマンドを修正し、アプリのトップの位置で実行します。

```sh:ホストOS(./)
docker build -t zenn-clone-nginx -f ./nginx/Dockerfile.prod ./nginx
```

狙いのイメージが作成されていることを確認します。

```sh:ホストOS(./)
$ docker images

REPOSITORY         TAG       IMAGE ID       CREATED         SIZE
zenn-clone-nginx   latest    7bfc0ce878e2   About a minute ago   199MB
.
.
```

#### 3. 構築が完了したら、このリポジトリにイメージをプッシュできるように、イメージにタグを付けます。

表示されているコマンドをコピペして実行します。

イメージに対して狙いのタグが付与されていることを確認します。

```sh:ホストOS(./)
$ docker images

REPOSITORY                                                           TAG       IMAGE ID       CREATED         SIZE
297087675121.dkr.ecr.ap-northeast-1.amazonaws.com/zenn-clone-nginx   latest    7bfc0ce878e2   3 minutes ago    199MB
.
.
```

#### 4. 以下のコマンドを実行して、新しく作成した AWS リポジトリにこのイメージをプッシュします

表示されているコマンドをコピペして実行します。

ECRにnginxのイメージがプッシュされていればOKです。

![](https://storage.googleapis.com/zenn-user-upload/9ec8cbcb887d-20230516.png)
