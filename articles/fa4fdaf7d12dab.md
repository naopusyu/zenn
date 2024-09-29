---
title: "Laravel FormRequestクラスのauthorizeメソッドで何もしないなら実はtrueを返す必要がなくなっていたお話"
emoji: "👺"
type: "tech"
topics:
  - "laravel"
  - "php"
published: true
published_at: "2023-04-17 08:00"
---

## はじめに

`FormRequest`を継承しているクラス見ている時に`authorize`メソッドが未定義だった時、
何もしないでもtrueを返す実装をしないといけないぞって思いながら、
GitHubでコードを調べてみると実は**trueすら返す必要がなかった**ってお話を書いていきます。

## 環境

- PHP 8.2.3
- Laravel 10.7.1

## `authorize`メソッドは何もするためのメソッドなのか

[マニュアル](https://laravel.com/docs/10.x/validation#authorizing-form-requests)をみると、

> 認証されたユーザーが特定のリソースを更新する権限を実際に持っているかどうかを判断できます。
> ※ 英語を直訳しています。

権限があればtrue、なければfalseを返すって感じの実装が必要なメソッドになります。

ただ、権限のチェックはミドルウェアなど別で実施していれば、`FormRequest`でわざわざ実施する必要もなかったりするので、trueを返しておくっていうのが、色々な記事に書かれていたりします。
（もちろん、`authorize`メソッドに権限チェックの処理を書いている記事もありますよ）

## このような実装を行なってきた

次のような、何もしないでtrueを返す実装を行なってきました。

```php

use Illuminate\Foundation\Http\FormRequest;

class PostRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            //
        ];
    }

```

## なぜ、trueを返す必要もないのか

実際に、`authorize`メソッドを使っている箇所（`passesAuthorization`メソッド）を見てみると...
https://github.com/laravel/framework/blob/v10.7.1/src/Illuminate/Foundation/Http/FormRequest.php#L180-L189

- `authorize`メソッドが定義があれば、`authorize`メソッドを実行し、結果を返す
- `authorize`メソッドが定義がなければ、trueを返す

このような実装になっているので、`authorize`メソッドで何もしないのであれば、そもそも定義しなくても良いって話になります。

## いつからこのような実装になった

次のPull Requestで変わったようです。
https://github.com/laravel/framework/pull/25417

マージされた日付をみると...`Sep 3, 2018`
つまり、2018年9月3日、すでに5年ほど前（Laravelのバージョンは5.7）から、trueを返す実装に変わっていたようです。

知らなかった...

## まとめ

何気ないきっかけで、FormRequestクラスの`authorize`メソッドの実装を調べてみると、
Laravel5.6以下では必要だった実装もLaravel5.7以上（調査に使ったLaravel10.7.1）では`何もしないでtrueを返すだけの実装`は**不要**っていうことがわかりましたね！っていうお話でした。