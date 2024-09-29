---
title: "リリース前の Laravel 10をインストールしてみるぞ！"
emoji: "🍞"
type: "tech"
topics:
  - "php"
  - "laravle"
published: true
published_at: "2022-12-13 00:30"
---

https://qiita.com/advent-calendar/2022/laravel
この記事は[Laravel Advent Calendar 2022](https://qiita.com/advent-calendar/2022/laravel) 12日目の記事です。

## はじめに

~~2023年2月にLaravel 10がリリースする予定です。~~
~~https://laravel.com/docs/9.x/releases#support-policy~~

~~今回はリリース前のLaravel 10をインストールしてみるぞ！~~

2023/02/14にLaravel 10にリリースされました。
https://laravel-news.com/laravel-10

すでにLaravel 10はインストールは可能になっているので
以降の手順は参考程度の情報になります。

## 環境

- PHP 8.1.12

※Dockerで環境を作っています。

Laravel 10.xからは最低でもPHP 8.1以上が必要になります。
https://github.com/laravel/laravel/pull/5854

:::message
以降はPHPコンテナ上で操作を行います。
:::

## インストール

インストールは下記のどちらかのコマンドを実行して、しばらくすればインストールが完了します。

:::message
事前にComposerのインストールが必要になります。
:::

### Laravel installerを使う場合

```
composer global require laravel/installer

laravel new laravel-10.x-dev --dev
```

### composerを使う場合

```
composer create-project --prefer-dist laravel/laravel laravel-10.x-dev dev-master
```

## 確認

インストールが終わったら、とりあえずバージョンを確認です。
`10.x-dev`が表示すれば、Laravel 10のインストールが完了です。

```
$ cd laravel-10.x-dev
$ php artisan -V
Laravel Framework 10.x-dev
```

## 動かしてみる

さっと確認したいので、`artisan serve`コマンドを実行しましょう！

今回はDocker上のPHPコンテナを使うので、下記を参考にポートの設定は事前に行う必要があります。
https://stackoverflow.com/questions/57639166/get-php-container-on-port-8000-after-php-artisan-serve


※一部抜粋
```yml:docker-compose.yml
~~~~~~
    ports:
      - "8000:8000"
~~~~~~
```

コマンド実行時のオプションで`--host=0.0.0.0`を指定が必要です。

```
$ php artisan serve --host=0.0.0.0

   INFO  Server running on [http://0.0.0.0:8000].  

  Press Ctrl+C to stop the server
```

ブラウザから`http://127.0.0.1:8000/`にアクセスすれば表示するはずです。

![](https://storage.googleapis.com/zenn-user-upload/4479b8faac39-20221213.png)

右下にも`Laravel v10.x-dev`と出ていますね！！

## まとめ

とりあえず、Laravle 10をインストールしてみました。
これだけでは特にLaravel 9から何が変わったのかわからないっていう話なんですが、
これからリリースまでに色々な変更が入ると思うので、定期的に更新をしてみていくもの面白いかもしれないです。
また、Laravelバージョンアップのために事前に更新して確認するのにも役にたつかもしれないですね！！

## おまけ

個人的に注目しているPRはこちらです。
https://github.com/laravel/framework/pull/44545
https://github.com/laravel/laravel/pull/6010

これまではPHPDocコメントに型が書かれていただけで、実際に指定が入ってなかったのですが、
ついにメソッドの引数、戻り値に型指定が入るようですね！！

これは楽しみにPRがマージされるのを待ちたいと思います！