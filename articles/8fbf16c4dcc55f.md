---
title: "LaravelのViewに渡した値をMiddlewareで書き換える摩訶不思議なコード"
emoji: "🎃"
type: "tech"
topics:
  - "laravel"
  - "php"
published: true
published_at: "2023-06-09 15:01"
---

## はじめに

こんなことを知ったところで、何も役には立たないかもしれないですが、
今回、LaravelのViewに渡した値をMiddlewareで書き換えるっていうことをやってみます。

## 環境

- PHP 8.2.6
- Laravel 10.13.5

## 準備

適当にViewに値を渡す処理とbladeテンプレートを用意しておきます。

```php:routes/web.php
  use Illuminate\Support\Facades\Route;

  Route::get('/', function () {
      return view('user', ['user' => ['name' => '太郎']]);
  });
```

```php:resources/views/user.blade.php
名前は {{ $user['name'] }} です。
```


一応、`php artisan serve`でサーバーを立ち上げて確認です。

![](https://storage.googleapis.com/zenn-user-upload/f8119cefa54f-20230609.png)

## 実装

### Middlewareを作成

まず、適当なMiddlewareをコマンドを使って作成します。

```bash
$ php artisan make:middleware ReplaceViewData

   INFO  Middleware [app/Http/Middleware/ReplaceViewData.php] created successfully. 
```

### ルートにMiddlewareを登録

出来上がったら、サンプルコードの`routes/web.php`の内容を次のように変更します。

```diff php:routes/web.php
  use App\Http\Middleware\ReplaceViewData;
  use Illuminate\Support\Facades\Route;

  Route::get('/', function () {
      return view('user', ['user' => ['name' => '太郎']]);
+ })->middleware(ReplaceViewData::class);
- });
```

### Middlewareの実装

では、`ReplaceViewData`Middlewareの実装です。
何をやっているかは後で書くとして、まずは書いたコードをドン！！

```php:app/Http/Middleware/ReplaceViewData.php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class ReplaceViewData
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        $response = $next($request);

        // 1. \Illuminate\Http\Responseを想定
        if ($original instanceof \Illuminate\Http\Response) {
                $original = $response->original;
        
                // 2. originalプロパティの型はmixedなので、\Illuminate\View\Viewのインスタンスか確認
                if ($original instanceof \Illuminate\View\View) {
                    // 3. Viewに渡した値を取得し、書き換える
                    $data = $original->getData();
                    $user = $data['user'];
                    $user['name'] = '花子';
                    $original->with('user', $user);

                    // 4. レスポンスクラスに渡している値を書き換える
                    $response->setContent($original);
                }
        }
        
        return $response;
    }
}
```

### Middlewareで何をやっているのか

順番に何をやっているのか書いていきます。

#### 1. \Illuminate\Http\Responseを想定

レスポンスは次の3種類のクラスが返ってくると思われます（他にもあるかもですが。。。）

- \Illuminate\Http\Response
- \Illuminate\Http\JsonResponse
- \Illuminate\Http\RedirectResponse

そこで今回は`\Illuminate\Http\Response`だけを考えます。
`\Illuminate\Http\JsonResponse`、`\Illuminate\Http\RedirectResponse`は考慮しない形です。

:::message
今回のサンプルだと、`\Illuminate\Http\Response`が返ってきそうな形ですが、
実際に使うってなった場合を考慮した形で実装しています。
:::

#### 2. originalプロパティの型はmixedなので、\Illuminate\View\Viewのインスタンスか確認

レスポンスで返す値は`\Illuminate\Http\ResponseTrait`（`\Illuminate\Http\Response`が読み込んでいるTrait）の[`$original`プロパティ](https://github.com/laravel/framework/blob/v10.13.5/src/Illuminate/Http/ResponseTrait.php#L11-L16)に入っています。
レスポンスなので、string,intなど色々な型が想定できますね。

なので、今回はViewに渡す値を書き換えたいので、`\Illuminate\View\View`のインスタンスかどうか調べる必要があったりします。

:::message
今回のサンプルだと、`viewヘルパ`を使っているので判定は不要でしたが、
実際に使うってなった場合を考慮した形で実装しています。
:::

#### 3. Viewに渡した値を取得し、書き換える

Viewに渡す値は`\Illuminate\View\View`の[`getData`メソッド](https://github.com/laravel/framework/blob/v10.13.5/src/Illuminate/View/View.php#L313-L316)で取得することができます。
取得した値を加工は適当に`name`の値を変えておきます。
書き換えた値を再度Viewに渡すには[`with`メソッド](https://github.com/laravel/framework/blob/v10.13.5/src/Illuminate/View/View.php#L237-L246)を使います。

:::message
今回はViewに渡している値の形はわかっている状態で書いています。
実際は配列のキーなどのチェックが必要になるかと思います。
:::

#### 4. レスポンスクラスに渡している値を書き換える

Viewに渡している値を書き換えるだけではレスポンスの値には反映しないので、再度`\Illuminate\Http\Response`に値を設定してあげる必要があります。

`$original`プロパティは`public`なので、直接値を設定することも可能ですが、
最終的に返すのは、`Symfony\Component\HttpFoundation\Response`の[`$content`プロパティ](https://github.com/symfony/http-foundation/blob/6.3/Response.php#L403-L413)になるので、`$original`プロパティに値を設定しても意味がないです。

そこで`\Illuminate\Http\Response`のインスタンス生成時にも呼ばれている[`setContent`メソッド](https://github.com/laravel/framework/blob/v10.13.5/src/Illuminate/Http/Response.php#LL48C3-L48C3)を使うことで`$content`プロパティにも値を設定しています。

## 確認

では、再度`php artisan serve`でサーバーを立ち上げて確認したいと思います。

![](https://storage.googleapis.com/zenn-user-upload/a178640901ff-20230609.png)

無事に太郎が花子に変わりましたね

## まとめ

実際にViewに渡した値を書き換える機会はほぼ無いかもしれないですが、
Middlewareクラスを使えば簡単に書き換えることは可能でした。

こんなこともできるってことがわかったことが今回の収穫ですね。

:::message alert
実際にこのようなコードを採用する場合は、組織内でしっかり共有しておかないと値が変わった！などの混乱を招くだけなので、ご利用は計画的にお願いします。
:::

:::message
ちなみに、`\Illuminate\Http\JsonResponse`の場合は、`$response->getData()`で取得し、
`$response->setData()`で書き込みができるかと思います。
:::