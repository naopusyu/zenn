---
title: "Laravel11 フレームワークのconfig読み込みを無効化する手段"
emoji: "🙆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [
    "laravel",
    "php"
]
published: true
published_at: "2024-12-06 06:00"
---

この記事は [カオナビ Advent Calendar 2024](https://qiita.com/advent-calendar/2024/kaonavi) シリーズ2 6日目の記事です。

## はじめに

2024年の4月にこんな記事を書いた。

https://zenn.dev/naopusyu/articles/d7cb654c3f262a

configから不要な項目を消したけど、
Laravel11からフレームワーク側でもっているconfigがベースになるので消した項目が復活するって状況になりました。

それから半年、挙動が変わっていることに気づいたので、改めてどのように動くのがまとめておきます。

## 環境

- PHP 8.4.1
- laravel/laravel 11.3.3
- laravel/framework 11.33.2

## 何があった

こんなプルリクエストが出ていた。

https://github.com/laravel/framework/pull/51579

この内容を読むと、`\Illuminate\Foundation\Application`の`dontMergeFrameworkConfiguration`メソッドを実行すればフレームワーク側のconfigを参照しなくなる（Laravel10の頃の挙動になる）って感じですね。

## 確認してみる

プルリクエストを見ると`bootstrap/app.php`に次のように書くことで設定できるようです。

```diff php:bootstrap/app.php
<?php

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

- return Application::configure(basePath: dirname(__DIR__))
+ $app = Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        //
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();

+ $app->dontMergeFrameworkConfiguration();

+ return $app;

```

この状態で前回と同じように`config/database.php`にある`pgsql`を消して確認します。

```diff php:config/database.php
-        'pgsql' => [
-            'driver' => 'pgsql',
-            'url' => env('DB_URL'),
-            'host' => env('DB_HOST', '127.0.0.1'),
-            'port' => env('DB_PORT', '5432'),
-            'database' => env('DB_DATABASE', 'laravel'),
-            'username' => env('DB_USERNAME', 'root'),
-            'password' => env('DB_PASSWORD', ''),
-            'charset' => env('DB_CHARSET', 'utf8'),
-            'prefix' => '',
-            'prefix_indexes' => true,
-            'search_path' => 'public',
-            'sslmode' => 'prefer',
-        ],
+        // 'pgsql' => [
+        //     'driver' => 'pgsql',
+        //     'url' => env('DB_URL'),
+        //     'host' => env('DB_HOST', '127.0.0.1'),
+        //     'port' => env('DB_PORT', '5432'),
+        //     'database' => env('DB_DATABASE', 'laravel'),
+        //     'username' => env('DB_USERNAME', 'root'),
+        //     'password' => env('DB_PASSWORD', ''),
+        //     'charset' => env('DB_CHARSET', 'utf8'),
+        //     'prefix' => '',
+        //     'prefix_indexes' => true,
+        //     'search_path' => 'public',
+        //     'sslmode' => 'prefer',
+        // ],
```

- dontMergeFrameworkConfiguration実行前

```
% php artisan tinker
Psy Shell v0.12.4 (PHP 8.4.1 — cli) by Justin Hileman
> 
> config('database.connections.pgsql');
= [
    "driver" => "pgsql",
    "url" => null,
    "host" => "127.0.0.1",
    "port" => "5432",
    "database" => "laravel",
    "username" => "root",
    "password" => "",
    "charset" => "utf8",
    "prefix" => "",
    "prefix_indexes" => true,
    "search_path" => "public",
    "sslmode" => "prefer",
  ]
```

- dontMergeFrameworkConfiguration実行後

```
% php artisan tinker
Psy Shell v0.12.4 (PHP 8.4.1 — cli) by Justin Hileman
> 
> config('database.connections.pgsql');
= null
```

nullになりましたね🎉
つまりフレームワーク側のconfigを読み込んでいないようですね🎉

## まとめ

Laravel11に上げるといきなり項目が復活したって人もいたかもしれないです。
半年以上経過すると、そういう状況も変わっているかもしれないっていうのが今回よくわかりました。

たまには過去に調べた内容を再度見直しをしてもよいかもしれないです。

ちなみにこの変更は11.11.0で入ったようです。（知らなかった。。。）

https://github.com/laravel/framework/blob/v11.33.2/CHANGELOG.md#v11110---2024-06-18
