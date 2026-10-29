---
title: "Composerのdump-autoloadは--no-devの指定をどう判断しているのか"
emoji: "🌿"
type: "tech"
topics: 
  - "php"
  - "composer"
published: true
publication_name: "kaonavi"
---

## はじめに

Composerで本番環境向けに依存パッケージをインストールするとき、`--no-dev`オプションを指定することがあります。

```bash
composer install --no-dev
```

そのあとに、次のコマンドを実行したとします。

```bash
composer dump-autoload
```

`dump-autoload`コマンドには`--no-dev`を指定していませんが、開発用のオートロード設定(`autoload-dev`)は除外されたままでした。

なぜ、このような動作なるのか気になったので、Composerの公式ドキュメントとソースコードを確認してみました。

## 環境

- PHP 8.5.12
- Composer 2.10.3

## 公式ドキュメントを確認する

まず、Composerの公式ドキュメントを確認します。

https://getcomposer.org/doc/03-cli.md#dump-autoload-dumpautoload

`dump-autoload`コマンドの`--dev`及び`--no-dev`オプションには次の説明があります。

> Composer will by default infer this automatically according to the last `install` or `update` `--no-dev` state.

どちらも、最後の`install`または`update`で指定された`--no-dev`の状態から自動的に判断される、と書かれています。

つまり、`dump-autoload`コマンドは`--dev`及び`--no-dev`オプションが指定されていない場合、直前の状態で判断しているようです。

直前の状態で判断とは？この辺りはソースコードから読み解いていく方がよさそうです。

## ソースコードを確認する

Composerのソースコードを確認します。

`dump-autoload`コマンドの`DumpAutoloadCommand`クラスの処理を順番に追いかけていくと`AutoloadGenerator`クラスの`dump()`メソッドに次の処理があります。

https://github.com/composer/composer/blob/2.10.3/src/Composer/Autoload/AutoloadGenerator.php#L184-L196

この処理を見ていきます。

### 1. devModeが未指定か確認する

```php
if (null === $this->devMode) {
```

最初に、`$this->devMode`が`null`かどうかを確認しています。

`$this->devMode`が`null`の場合は自動判定に進みます。

`null`以外の場合、`--dev`及び`--no-dev`が明示的に指定されているため、自動判定の処理には入りません。

この`$this->devMode`に値を設定するのは、`DumpAutoloadCommand`クラスで行なっています。

https://github.com/composer/composer/blob/2.10.3/src/Composer/Command/DumpAutoloadCommand.php#L102-L110

### 2. installed.jsonを読み込む

```php
// we assume no-dev mode if no vendor dir is present or it is too old to contain dev information
$this->devMode = false;

$installedJson = new JsonFile($config->get('vendor-dir').'/composer/installed.json');
if ($installedJson->exists()) {
    $installedJson = $installedJson->read();

    if (isset($installedJson['dev'])) {
        $this->devMode = $installedJson['dev'];
    }
}
```

次に、`vendor/composer/installed.json`を読み込んでいます。

ファイルの存在確認を行なってから、`installed.json`の `dev`の値を取得しています。

この値が存在する場合、`$this->devMode`に値を設定しています。

つまり、`--dev`及び`--no-dev`オプションを指定していない場合は、`installed.json`の`dev`の値を使って判断していることになります。

## installed.jsonのdevにはどんな値が入る？

`install`コマンドを実行すると`InstallCommand`クラスが動くのでみてみると以下で `--no-dev`オプションを取得し、設定していることがわかります。

https://github.com/composer/composer/blob/2.10.3/src/Composer/Command/InstallCommand.php#L136

この`setDevMode`メソッドを確認してみると引数の型は`bool`型になっていました。

https://github.com/composer/composer/blob/2.10.3/src/Composer/Installer.php#L1381-L1389

つまり、`--no-dev`オプションがあれば`false`、なければ`true`を設定しています。
※`!$input->getOption('no-dev')`と!がついているので意味が逆転します。

このあとの処理を順番に見ていきます。

https://github.com/composer/composer/blob/2.10.3/src/Composer/Command/InstallCommand.php#L152

https://github.com/composer/composer/blob/2.10.3/src/Composer/Installer.php#L327

https://github.com/composer/composer/blob/2.10.3/src/Composer/Installer.php#L876

最終的な書き込みは以下の場所で行われていました。

https://github.com/composer/composer/blob/2.10.3/src/Composer/Installer/InstallationManager.php#L238

ここまでの間で設定した値は書き換わっていないので、`installed.json`の`dev`にはbool値が入っていることがわかります。

この値を`dump-autoload`コマンドが読み取ることで、開発用のオートロード設定を含めるかどうかが決まるようです。

## まとめ

`dump-autoload`コマンドは`installed.json`の`dev`の値を見て、開発用のオートロード設定を含めるかどうか判断していることがわかりました。

ソースコードを追いかけてみて、またひとつComposerのことが詳しくなれました。
