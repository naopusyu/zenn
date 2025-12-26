---
title: "PHPからS3へのファイルアップロードをLocalStackを使ってやってみる"
emoji: "🌿"
type: "tech"
topics:
  - "php"
  - "localstack"
published: true
published_at: "2022-07-27 05:49"
---

[LocalStack](https://localstack.cloud/)が[バージョン1.0をリリース](https://localstack.cloud/blog/2022-07-13-announcing-localstack-v1-general-availability/)しました。
そこで今回ローカル環境からS3にファイルをアップするコードを書いてみた。


## 環境
- PHP 7.4.10
- LocalStack 1.0.1

手元にあったMacに入っていたPHPが古かった

## 準備

### docker-compose.ymlの内容

```yml
version: '3'
services:
  localstack:
    image: localstack/localstack:1.0.1
    ports:
      - 4566:4566
    environment:
      - SERVICES=s3
      - AWS_ACCESS_KEY_ID=dummy
      - AWS_SECRET_ACCESS_KEY=dummy
      - AWS_DEFAULT_REGION=ap-northeast-1
```

- `SERVICES`で使うサービスを指定する（今回はS3のみ指定）
- `AWS_ACCESS_KEY_ID`と`AWS_SECRET_ACCESS_KEY`の値はなんでも良いです
- `AWS_DEFAULT_REGION`は東京リージョン
- 永続化の設定とかあるけど今回はなし

### Docker起動

```sh
$ docker compose up -d --build
```

以降は起動している前提で書いていきます。

### バケット作成

LocalStackのコンテナに`AWS CLI`も一緒に入っていたのでそこからコマンドでバケットを作成する。

```sh
$ docker compose exec localstack aws --endpoint-url=http://localhost:4566 s3 ls s3://

$ docker compose exec localstack aws --endpoint-url=http://localhost:4566 s3 mb s3://local-test
make_bucket: local-test

$ docker compose exec localstack aws --endpoint-url=http://localhost:4566 s3 ls s3://
2020-10-11 21:47:34 local-test
```

## s3にファイルアップロードするPHPのコード

### パッケージのインストール

`AWS SDK for PHP`を使いたいので`Composer`を使ってインストール  

```sh
$ composer require aws/aws-sdk-php
```

### PHPのコード

```php
<?php

require_once 'vendor/autoload.php';

use Aws\S3\S3Client;

$config = [
    'credentials' => [
        'key' => 'dummy',
        'secret' => 'dummy',
    ],
    'region' => 'ap-northeast-1',
    'version' => 'latest',
    'endpoint' => 'http://localhost:4566',
    'use_path_style_endpoint' => true,
];

$s3 = new S3Client($config);

// アップロード
$result = $s3->putObject([
    'Bucket' => 'local-test',  // バケット名
    'Key' => 'hello.txt',      // s3のアップロード先
    'Body' => 'hello world!!', // ファイルの内容
]);

echo 'ファイルのURL => ' . $result['ObjectURL'] . PHP_EOL;

// 内容の取得
$result = $s3->getObject([
    'Bucket' => 'local-test',  // バケット名
    'Key' => 'hello.txt',      // s3のアップロード先
]);

echo 'ファイルの内容 => ' . $result['Body'] . PHP_EOL;
```

大事なところは`endpoint`と`use_path_style_endpoint`です。
LocalStackを使うと実際のS3のURLとは異なるなので`endpoint`にバケット作成で使った`http://localhost:4566`を設定する必要がある。

ただし、指定することで`endpoint`が`http://{バケット名}.localhost:4566`となり実行すると名前解決できずエラーが発生します。  
そこで`use_path_style_endpoint`に`true`を設定することで`endpoint`がパス形式（`http://localhost:4566/{バケット名}`）になり、実行可能になる。

## 動かしてみる

```sh
$ php s3.php
ファイルのURL => http://localhost:4566/local-test/hello.txt
ファイルの内容 => hello world!!
```

ファイルURLの表示とアップした内容の取得が表示していれば成功です。

## 確認

`AWS CLI`を使ってファイルがアップできているか見てみる。

```sh
$ docker compose exec localstack aws --endpoint-url=http://localhost:4566 s3 ls s3://local-test
2020-10-11 23:49:33         13 hello.txt
```

ファイル名が表示していればOKです。

## まとめ

LocalStackを使えば簡単にS3へのアップロードができました。  
（ただし、エンドポイントあたりがややこしいです。）
他のAWSのサービスも利用可能なので別の機会に触ってみたいと思います。
