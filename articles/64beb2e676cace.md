---
title: "Composerコマンドを使ってパッケージの脆弱性をチェックする"
emoji: "🛡️"
type: "tech"
topics:
  - "php"
  - "composer"
published: true
published_at: "2022-12-03 00:05"
---

https://qiita.com/advent-calendar/2022/php
この記事は[PHP Advent Calendar 2022](https://qiita.com/advent-calendar/2022/php) 3日目の記事です。

## はじめに

Composer2.4からパッケージの脆弱性をチェックする`audit`コマンドが追加されました。
https://blog.packagist.com/composer-2-4/

これまではComposerとは別のツールを利用する必要があった脆弱性のチェックをComposerだけで完結できるので、今回`audit`コマンドを試してみたいと思います。

ちなみに別のツールは次のものがあったりします。
※ここに書いたツール以外にもあると思います。
- [sensiolabs/security-checker](https://github.com/sensiolabs/security-checker)
※こちらはアーカイブされております。
- [fabpot/local-php-security-checker](https://github.com/fabpot/local-php-security-checker)  
※こちらはアーカイブされております。

## `audit`コマンド

`audit`コマンドの[ドキュメント](https://getcomposer.org/doc/03-cli.md#audit)を見ると3つほどオプションがあるようです。

- `--no-dev`
require-devに指定しているパッケージを含めない。
- `--format`
出力形式を"table" (default), "plain", "json", "summary"から選択できる。
- `--locked`
composer.lockの内容でチェックする。
指定しない場合はvendor配下にインストールしているパッケージでチェックする。

## 環境
- PHP 8.1.12
- Composer 2.4.4

今回は[guzzlehttp/guzzle](https://github.com/guzzle/guzzle)を使って、脆弱性のチェックをしていと思います。

## 実行

では、さっそく、`audit`コマンドを実行してみます。

```
$ composer show | grep guzzlehttp/guzzle
guzzlehttp/guzzle             7.4.2   Guzzle is a PHP HTTP client library

$ composer audit
Found 5 security vulnerability advisories affecting 1 package:
+-------------------+----------------------------------------------------------------------------------+
| Package           | guzzlehttp/guzzle                                                                |
| CVE               | CVE-2022-31091                                                                   |
| Title             | Change in port should be considered a change in origin                           |
| URL               | https://github.com/guzzle/guzzle/security/advisories/GHSA-q559-8m2m-g699         |
| Affected versions | >=7,<7.4.5|>=4,<6.5.8                                                            |
| Reported at       | 2022-06-20T22:24:00+00:00                                                        |
+-------------------+----------------------------------------------------------------------------------+
+-------------------+----------------------------------------------------------------------------------+
| Package           | guzzlehttp/guzzle                                                                |
| CVE               | CVE-2022-31090                                                                   |
| Title             | CURLOPT_HTTPAUTH option not cleared on change of origin                          |
| URL               | https://github.com/guzzle/guzzle/security/advisories/GHSA-25mq-v84q-4j7r         |
| Affected versions | >=7,<7.4.5|>=4,<6.5.8                                                            |
| Reported at       | 2022-06-20T22:24:00+00:00                                                        |
+-------------------+----------------------------------------------------------------------------------+
+-------------------+----------------------------------------------------------------------------------+
| Package           | guzzlehttp/guzzle                                                                |
| CVE               | CVE-2022-31043                                                                   |
| Title             | Fix failure to strip Authorization header on HTTP downgrade                      |
| URL               | https://github.com/guzzle/guzzle/security/advisories/GHSA-w248-ffj2-4v5q         |
| Affected versions | >=7,<7.4.4|>=4,<6.5.7                                                            |
| Reported at       | 2022-06-09T23:36:00+00:00                                                        |
+-------------------+----------------------------------------------------------------------------------+
+-------------------+----------------------------------------------------------------------------------+
| Package           | guzzlehttp/guzzle                                                                |
| CVE               | CVE-2022-31042                                                                   |
| Title             | Failure to strip the Cookie header on change in host or HTTP downgrade           |
| URL               | https://github.com/guzzle/guzzle/security/advisories/GHSA-f2wf-25xc-69c9         |
| Affected versions | >=7,<7.4.4|>=4,<6.5.7                                                            |
| Reported at       | 2022-06-09T23:36:00+00:00                                                        |
+-------------------+----------------------------------------------------------------------------------+
+-------------------+----------------------------------------------------------------------------------+
| Package           | guzzlehttp/guzzle                                                                |
| CVE               | CVE-2022-29248                                                                   |
| Title             | Cross-domain cookie leakage                                                      |
| URL               | https://github.com/guzzle/guzzle/security/advisories/GHSA-cwmx-hcrq-mhc3         |
| Affected versions | >=7,<7.4.3|>=4,<6.5.6                                                            |
| Reported at       | 2022-05-25T13:21:00+00:00                                                        |
+-------------------+----------------------------------------------------------------------------------+
```

いくつか発見できましたね。

出力した内容を確認すると、7.4.5以上であれば解消してそうなので、
7.4.5以上にバージョンを上げた後に改めて、`audit`コマンドを実行してみます。
※この記事を書いている時は7.5.0が最新バージョンでした。

```
$ composer show | grep guzzlehttp/guzzle
guzzlehttp/guzzle             7.5.0   Guzzle is a PHP HTTP client library

$ composer audit
No security vulnerability advisories found
```

はい、バージョンを上げると脆弱性に対しての修正が入っているので、何も発見できませんでした。

## まとめ

今回はComposerコマンドで脆弱性のチェックを試してみました。
別のツールを使っていた時は下記が気になっていましたが、Composerだけを使うことで不要なるのは個人的に大きなメリットかと思います。
- ツールを動かすためのさらにツールを入れないといけない
- ツールのメンテが終了すると、別ツールに移行が必要になる

また、`locked`オプションを使うことで`composer.lock`を使って脆弱性のチェックを行うことができるので、CIなどで定期的にチェックするなら、このオプションを使う方が毎回`composer install`を行わなくてもいいかもしれないですね。

最後に、この記事が誰かの参考になればと思います。