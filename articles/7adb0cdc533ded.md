---
title: "Laravelのルーティングでオプショナルパラメータと条件指定を組み合わせた時の挙動"
emoji: "🌿"
type: "tech"
topics: [
    "laravel",
    "php"
]
published: true
---

## はじめに

LaravelのルーティングにはURLの一部をパラメータとして受け取ることができます。
パラメータを省略可能にしたい場合やパラメータに数値のみを許可したいなどの条件を付けた場合に、`id?`のようなオプショナルパラメータや`whereNumber('id')`のような条件を指定できます。

今回はこの2つを組み合わせた時にどのように動くのか少し検証したいと思います。

## 環境

- PHP 8.4.8
- Laravel 12.19.3

## オプショナルパラメータ

まずはオプショナルパラメータについてです。

https://laravel.com/docs/12.x/routing#parameters-optional-parameters

ルートパラメータの後に`?`を付けることで、そのパラメータを省略可能にすることができます。

この場合ルートパラメータにはデフォルト値を指定が必要になります。

```php
Route::get('/users/{id?}', function ($id = null) {
    // $idがない場合はnullが渡される
});
```

## ルートパラメータの条件指定

つぎにルートパラメータの条件指定を少し見ておきます。

https://laravel.com/docs/12.x/routing#parameters-regular-expression-constraints

`where`メソッドを使うことで正規表現で条件を指定できます。

```php
Route::get('/users/{id}', function ($id) {
    // ...
})->where('id', '[0-9]+');
```

また、いくつかメソッドが用意されているので正規表現を使わないで書くこともできます。

```php
Route::get('/users/{id}', function ($id) {
    // ...
})->whereNumber('id');
```

その他、いろいろメソッドが用意されているので、こちらのファイルを見ておくといいかもしれないです。

https://github.com/laravel/framework/blob/v12.19.3/src/Illuminate/Routing/CreatesRegularExpressionRouteConstraints.php


## オプショナルパラメータと条件指定の組み合わせ

では、オプショナルパラメータと条件指定を組み合わせた場合、どのような動作になるのか見ていきます。

サンプルコードはこちらです。

```php
Route::get('/users/{id?}', function ($id = null) {
    if (is_null($id)) {
        return 'id = null';
    }
    return 'id = ' . $id;
})->whereNumber('id');
```

この場合、以下のような動作になることを想定しています。

1. `/users` → 正常に動作（ステータスが200）し、`$id`は`null`となるのでレスポンスは`id = null`
2. `/users/123` → 正常に動作（ステータスが200）し、`$id`は`123`となるのでレスポンスは`id = 123`
3. `/users/abc` → ステータスが404エラーになる（数値以外はマッチしない）

実際にこのように動作するかテストコードで確認してみます。

```php:tests/Feature/ExampleTest.php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test(): void
    {
        // id が指定されていない場合は null
        $this->get('/users/')
            ->assertOk()
            ->assertContent('id = null');

        // id が数字の場合はその値を返す
        $this->get('/users/123')
            ->assertOk()
            ->assertContent('id = 123');

        // id が数字以外の場合は 404
        $this->get('/users/abc')
            ->assertNotFound();
    }
}
```

```bash
% ./vendor/bin/phpunit tests/Feature/ExampleTest.php
PHPUnit 11.5.24 by Sebastian Bergmann and contributors.

Runtime:       PHP 8.4.8
Configuration: /laravel-12/phpunit.xml

.                                                                   1 / 1 (100%)

Time: 00:00.133, Memory: 26.00 MB

OK (1 test, 5 assertions)
```

成功したってことは想定通りの動きをしていますね。

## まとめ

オプショナルパラメータを使った場合、

1. パラメータが未指定が優先される
2. パラメータの指定があった場合に条件をチェックする

のような動きになるみたいですね。

実際にこのチェックを行なっているコードはこのへんかなーってところまでみていますが、今回は深追いしないでおきます。

https://github.com/laravel/framework/blob/v12.19.3/src/Illuminate/Routing/Route.php#L340-L355

