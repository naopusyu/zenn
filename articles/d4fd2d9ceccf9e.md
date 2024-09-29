---
title: "composer.lockのcontent-hashがコンフリクトした時の解消方法"
emoji: "🕌"
type: "tech"
topics:
  - "php"
  - "composer"
published: true
published_at: "2022-06-29 22:31"
---

## はじめに
複数人で開発を行っていると、composer.lockの`content-hash`がコンフリクトするケースがあるかと思います。

こんなやつです。

```
<<<<<<< HEAD
    "content-hash": "cf1dff96707179ceae053c1d5685547f",
=======
    "content-hash": "aa9b56fac6bdcba2a932ace817f0345b",
>>>>>>> master
```

毎回解消方法を忘れるので、手順を残しておこうと思います。

## 手順

### その1

`content-hash`のどちらかを残す。
つまり、コンフリクトをまず解消します。

### その2

次のコマンドを実行する。

```
composer update --lock
```

これで、新たな`content-hash`が生成されます。
これで解消です。
