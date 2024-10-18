---
title: "Laravel8に上げるときにモデルファクトリのツライ修正をRectorを使って乗り越える"
emoji: "👹"
type: "tech"
topics:
  - "laravel"
  - "php"
  - "rector"
published: true
published_at: "2021-12-04 07:55"
---

https://qiita.com/advent-calendar/2021/kaonavi
この記事は、[株式会社カオナビ Advent Calendar 2021](https://qiita.com/advent-calendar/2021/kaonavi)の4日目の記事です。

## はじめに

2022年1月^[[実際は2022年2月にリリースされました。](https://laravel-news.com/laravel-9-released)]に次のLTSバージョン^[[残念ながらLTSバージョンではなかったです。](https://laravel.com/docs/9.x/releases#support-policy)]になる[Laravel9のリリース](https://blog.laravel.com/laravel-9-release-date)が予定されています。

現在のLTSバージョンのLaravel6を使っている人たちはLaravel9にあげようと考える時期になるかと思いますが、そのためにはLaravel7と8の変更点に対応する必要があります。

その中にはLaravel8でモデルファクトリがクラスベースに書き直され、Laravel7までの使っていたモデルファクトリが完全に使えなくなるっていう考えるだけでツライ修正が待っています。

救済処置なのか、[Laravel8のアップグレードガイド](https://readouble.com/laravel/8.x/ja/upgrade.html#model-factories)には、`laravel/legacy-factories`パッケージを使えばLaravel7までの記述をLaravel8でも使えるようになると書いてあるので、このパッケージを使えば修正は不要になります。

ただし、このパッケージを使ったとしても、問題を先送りしただけでいつかはツライ修正を行う時がくるはずです。

そこで今回は**Rector**を使ってツライ修正を乗り越えるためにやることを書いていこうと思います。

## Rectorとは
https://github.com/rectorphp/rector

:::message
PHPコードのアップグレードや自動でリファクタリングを行うツール
:::

今まで手作業でやっていたこと機械的な修正を自動でやってくれる素晴らしいツールです。

## どんなことができるのか
すでに500件近くの[ルール](https://github.com/rectorphp/rector/blob/main/docs/rector_rules_overview.md)があります。

たとえば、PHP7.4で増えたプロパティの型指定を自動で追加を行なってくれるルールもあったりします。

また、主要なフレームワークなどのパッケージも用意されていて、今回はその中のLaravelのパッケージを使いたいと思います。
https://github.com/driftingly/rector-laravel

## 実行環境

今回は下記の環境を使います。

- PHP 8.0.13
- Laravel 8.74.0

今回Laravel8を使いますが、モデルファクトリやテストコードはLaravel7までの書き方をした下記を用意しました。
※モデルファクトリは初期状態のものです。

:::details database/factories/UserFactory.php
```php:database/factories/UserFactory.php
<?php

/** @var \Illuminate\Database\Eloquent\Factory $factory */

use App\Models\User;
use Faker\Generator as Faker;
use Illuminate\Support\Str;

$factory->define(User::class, function (Faker $faker) {
    return [
        'name' => $faker->name,
        'email' => $faker->unique()->safeEmail,
        'email_verified_at' => now(),
        'password' => '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', // password
        'remember_token' => Str::random(10),
    ];
});
```
:::

:::details tests/UserTest.php
```php:tests/UserTest.php
<?php

namespace Tests;

use App\Models\User;
use Tests\TestCase;

class UserTest extends TestCase
{
    public function test()
    {
        $user = factory(User::class)->create(['name' => 'test']);
        $this->assertSame('test', $user->name);
    }
}
```
:::

## インストール

Rectorは常に使うようなパッケージではないと思うので、作業用ディレクトリを作成し、その中にインストールする形で今回は行きたいと思います。

:::message
この記事と同じバージョンのRectorを使う場合は`0.12.5`を指定して、インストールを行なってください。
:::

```
mkdir --parents tools/rector
composer req --working-dir=tools/rector rector/rector:"0.12.5"
```

インストール後、Rectorのバージョンはこちらです。
```
tools/rector/vendor/bin/rector --version
Rector ebd553fe45184cefec5844e075a0af35ab3a9375
```

尚、LaravelのパッケージはRectorに含まれているようです。
https://github.com/rectorphp/rector/tree/0.12.5/vendor/rector/rector-laravel

:::message alert
Rectorのバージョンが[`0.14.7`](https://github.com/rectorphp/rector/releases/tag/0.14.7)からLaravelのパッケージは含まれなくなりました。
なので、Rectorが`0.14.7`以降のバージョンでこの記事の内容試す場合は、別途Laravelのパッケージをインストールしていただく必要があります。
```
composer req --working-dir=tools/rector driftingly/rector-laravel
```
:::

## 設定ファイルの作成

まずは設定ファイルの作成から行います。
下記コマンドを実行し、`rector.php`ファイルが生成してください。

```
tools/rector/vendor/bin/rector init
```

`rector.php`ファイルが作成されたら、次のように書き換えます。

```php:rector.php
<?php

declare(strict_types=1);

use Rector\Core\Configuration\Option;
use Rector\Core\ValueObject\PhpVersion;
use Rector\Laravel\Set\LaravelSetList;
use Symfony\Component\DependencyInjection\Loader\Configurator\ContainerConfigurator;

return static function (ContainerConfigurator $containerConfigurator): void {
    $parameters = $containerConfigurator->parameters();
    // 対象ディレクトリの指定
    $parameters->set(Option::PATHS, [
        __DIR__ . '/database/factories',
        __DIR__ . '/tests',
    ]);

    // PHPバージョンを指定
    $parameters->set(Option::PHP_VERSION_FEATURES, PhpVersion::PHP_80);

    // ルールの指定
    $containerConfigurator->import(LaravelSetList::LARAVEL_LEGACY_FACTORIES_TO_CLASSES);
};
```

これで、モデルファクトリの書き換えルールと対象のディレクトリの設定ができました。

## ルールの適用

次に下記コマンドを実行してルールを適用します。
`--dry-run`オプションを付けると即時反映ではなく、事前にどのように適用されるのか確認ができます。
事前確認が不要な場合はオプションを付けずに実行してください。

```shell
# 事前確認
tools/rector/vendor/bin/rector process --dry-run
# 適用
tools/rector/vendor/bin/rector process
```

結果はこのような感じになります。
↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓

```diff php
1) database/factories/UserFactory.php:5

    ---------- begin diff ----------
@@ @@
 use Faker\Generator as Faker;
 use Illuminate\Support\Str;

-$factory->define(User::class, function (Faker $faker) {
-    return [
-        'name' => $faker->name,
-        'email' => $faker->unique()->safeEmail,
-        'email_verified_at' => now(),
-        'password' => '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', // password
-        'remember_token' => Str::random(10),
-    ];
-});
+class UserFactory extends \Illuminate\Database\Eloquent\Factories\Factory
+{
+    protected $model = User::class;
+    public function definition()
+    {
+        return [
+            'name' => $this->faker->name,
+            'email' => $this->faker->unique()->safeEmail,
+            'email_verified_at' => now(),
+            'password' => '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', // password
+            'remember_token' => Str::random(10),
+        ];
+    }
+}
    ----------- end diff -----------

Applied rules:
 * FactoryDefinitionRector


2) tests/UserTest.php:8

    ---------- begin diff ----------
@@ @@
 {
     public function test()
     {
-        $user = factory(User::class)->create(['name' => 'test']);
+        $user = User::factory()->create(['name' => 'test']);
         $this->assertSame('test', $user->name);
     }
 }
    ----------- end diff -----------

Applied rules:
 * FactoryFuncCallToStaticCallRector
```

Rectorすごいですねー！！！
さらりとファクトリとテストクラスが書き換わりましたねー、これを手作業でやることを考えると圧倒的に時間の短縮です。

## 仕上げの手作業

やりたいことがすべて自動で修正してくれるわけではないので、[Laravel8のアップグレードガイド](https://readouble.com/laravel/8.x/ja/upgrade.html#seeder-factory-namespaces)に書いてある、クラスベースに書き直されたことでファクトリにネームスペースが付くようになったので、ここは手作業でモデルファクトリと`composer.json`の修正を行います。

```diff php:database/factories/UserFactory.php

-/** @var \Illuminate\Database\Eloquent\Factory $factory */
+namespace Database\Factories;

use App\Models\User;
-use Faker\Generator as Faker;
+use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Str;

-class UserFactory extends \Illuminate\Database\Eloquent\Factories\Factory
+class UserFactory extends Factory
{
```

```diff json:composer.json
    "autoload": {
        "psr-4": {
            "App\\": "app/"
+           "Database\\Factories\\": "database/factories/",
        },
        "classmap": [
            "database/seeds",
-           "database/factories"
        ]
    },
```

`composer.json`の`autoload`を変更しているので、下記コマンドを実行して、クラスマップを更新した方がよさそうです。

```shell
composer dump-autoload
```

また、クラスベースのファクトリを使うためには既存のモデルクラスに`Illuminate\Database\Eloquent\Factories\HasFactory`トレイトを追加する必要があるのでこちらも修正します。

```diff php:app/Models/User.php

use Illuminate\Contracts\Auth\MustVerifyEmail;
+use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
-   use Notifiable;
+   use HasFactory, Notifiable;
```

## 動作確認

最後にユニットテストを実行して動くことを信じましょう。

```shell
vendor/bin/phpunit tests/UserTest.php
PHPUnit 9.5.10 by Sebastian Bergmann and contributors.

.                                                                   1 / 1 (100%)

Time: 00:01.154, Memory: 22.00 MB

OK (1 test, 1 assertion)
```

無事に動きましたー！！！

## まとめ

今回は1件のモデルファクトリでしたが、簡単に修正が行えました。
仮にモデルファクトリが100件ほどあっても、機械的な修正から解放されるので、かなりの恩恵を受けるかと思います。

今回は試してないですが、一部の手作業の修正もRectorの他のルールで対応可能なら、すべて自動で修正可能かもしれないです。（これは別でやってみたい気持ちです。）

これからLaravel8までバージョンに上げようと考えているひとがいれば、Rectorを使ってモデルファクトリのツライ修正を乗り越えていきましょう！！