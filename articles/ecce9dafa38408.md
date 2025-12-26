---
title: "Laravel database/seeders以外にSeederクラスを配置して実行する"
emoji: "🌿"
type: "tech"
topics:
  - "laravel"
  - "php"
published: true
published_at: "2024-04-17 23:22"
---

## はじめに

Laravelで初期データを作成する際にSeederクラスを使うケースがあると思います。
その際に初めから用意されている`database/seeders`配下の`DatabaseSeeder`クラスを使ったり、
同じ階層にSeederクラスを追加して対応しているかと思います。

でも、実は別のところに追加しても実行できたりします。

今回はそんなちょっとした小ネタを書いておきたいと思います。

## 環境

- PHP 8.3.6
- Laravel 11.4.0

## やってみる

まず`DatabaseSeeder`クラスを指定して実行してみると、次のようになるかと

```shell
% php artisan db:seed --class=\\Database\\Seeders\\DatabaseSeeder

   INFO  Seeding database.  

```

では、別のところにクラスを作ってみましょう。

場所はどこでもいいですが、`autoload`の設定が入っているところにする必要があるので、
今回は`app`配下にクラスを1つ作ります。

内容は楽をしたいので、[`DatabaseSeeder`クラス](https://github.com/laravel/laravel/blob/v11.0.6/database/seeders/DatabaseSeeder.php)をコピーして、ネームスペースだけ書き換えます
※登録するデータも一部書き換えています。

```php:app/DatabaseSeeder.php
<?php

namespace App;

use App\Models\User;
use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    /**
     * Seed the application's database.
     */
    public function run(): void
    {
        User::factory(10)->create();
    }
}
```

追加したapp/DatabaseSeeder.phpを実行してみます。

```shell
% php artisan db:seed --class=\\App\\DatabaseSeeder

   INFO  Seeding database.  

```

できましたね🥳

## なぜできるのか

`db:seed`コマンドの処理を見るとオプションで渡されたclass名を98行目でインスタンスを生成しているため実行できるわけですね。

https://github.com/laravel/framework/blob/v11.4.0/src/Illuminate/Database/Console/Seeds/SeedCommand.php#L80-L101

注意点としては、
- classオプションは`\\`を含めてクラスを指定しないと先頭に`Database\\Seeders\`がつく
- Seederクラスは必ず`\Illuminate\Database\Seeder`を継承して、`run`メソッドを実装しておく

## まとめ

`database/seeders`配下で作っておけば気にする必要もないですが、
Laravelの[パッケージ開発](https://laravel.com/docs/11.x/packages)を行っている場合、
パッケージ配下に`seeders`ディレクトリを作ったりするかもしれないのでそういう時に使えるかもしれないですね。
