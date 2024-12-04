---
title: "Laravel11 keyコマンド/packageコマンド/clear-compiledコマンド"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [
    "laravel",
    "php"
]
published: true
published_at: "2024-12-13 06:00"
---

この記事は [Laravel11にあるartisanコマンドを全部調べる Advent Calendar 2024](https://adventar.org/calendars/10674) 13日目の記事です。

今回はkeyコマンド/packageコマンド/clear-compiledコマンドについて調べました。

## 環境

- PHP 8.4.1
- laravel/laravel 11.3.3
- laravel/framework 11.33.2

## key:generateコマンド

アプリケーションキーを生成する。

```
php artisan key:generate
```

実行すると環境ファイル（.env）のAPP_KEYの値を生成します。

通常はLaravelのインストール時（composer create-projectコマンド実行後）に発生するpost-create-project-cmdイベントで実行されます。

https://github.com/laravel/laravel/blob/v11.3.3/composer.json#L45-L48

本番環境（.envのAPP_ENVがproduction）では実行すると実行確認のメッセージが表示します。

![](/images/aab0de9f5ed7c9/1.png)

Yesを選択すると環境ファイル（.env）のAPP_KEYの値を生成します。

| オプション | 説明 |
| --- | --- |
| `--show` | 値の変更を行わないでキーを表示 |
| `--force` | 本番環境で強制的に実行 |

- `--show`を付けると環境ファイル（.env）のAPP_KEYの値を設定せずに値を表示します

```
advent-calendar-2024 % php artisan key:generate --show 
base64:Hx6JbfTCAmh6Sf0cWX3f5zQ0byYllclHxLjrEi70Jbk=
```

- 本番環境で`--force`を付けると実行確認を行わないで強制的に実行する

## package:discoverコマンド

パッケージの情報を再読み込みする。

```
php artisan package:discover
```

実行するとvendor/composer/install.jsonからLaravelの拡張パッケージの情報を読み込み、パッケージのキャッシュファイル（bootstrap/cache/packages.php）を再構築します。

![](/images/aab0de9f5ed7c9/2.png)

通常、package:discoverコマンドはComposerパッケージの追加、更新、削除後のcomposer dump-autoloadコマンド実行後に発生するpost-autoload-dumpイベントで自動で実行されるようになっています。

https://github.com/laravel/laravel/blob/v11.3.3/composer.json#L35-L38

## clear-compiledコマンド

コンパイル済みのファイルを削除する。

```
php artisan clear-compiled
```

実行するとコンパイル済みのファイル（bootstrap/cache/services.php、bootstrap/cache/packages.php）を削除します。
