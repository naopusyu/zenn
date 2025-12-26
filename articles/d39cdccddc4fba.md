---
title: "PHPDocの@inheritDocと{@inheritDoc}の違いについて"
emoji: "🌿"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [
    "php"
]
published: true
---

## はじめに

最近PHPDocを眺める機会が増えて、その中で`@inheritDoc`と`{@inheritDoc}`が登場し、同じでは？って思いながら調べた結果を残しておきます。

## `@inheritDoc`と`{@inheritDoc}`の違い

ドラフトではあるけど、**PSR-19 PHPDoc Tags**に違いが書かれていました。

https://www.php-fig.org/psr/


### @inheritDocとは

https://github.com/php-fig/fig-standards/blob/master/proposed/phpdoc-tags.md#41-making-inheritance-explicit-using-the-inheritdoc-tag

以下の例のようにPHPDocの内容に変更がない場合は`@inheritDoc`を使って親クラスの内容を継承することができる。

```php
class A {
    /**
     * 指定したidを使ってデータベースから値を取得する
     * @param int $id
     * @return array
     */
    public function get(int $id): array {
        // 処理
    }
}

class AA extends A {
    /**
     * @inheritDoc
     */
    public function get(int $id): array {
        // 処理
    }
}
```

## {@inheritDoc}とは

https://github.com/php-fig/fig-standards/blob/master/proposed/phpdoc-tags.md#42-using-the-inheritdoc-inline-tag-to-augment-a-description

親クラスの内容を継承し、**情報を追加する**場合に使う。

```php
class A {
    /**
     * 指定したidを使ってデータベースから値を取得する
     * @param int $id
     * @return array
     */
    public function get(int $id): array {
        // 処理
    }
}

class AA extends A {
    /**
     * {@inheritDoc}
     * キャッシュがあればキャッシュから取得する
     */
    public function get(int $id): array {
        // 処理
    }
}
```

## まとめ

違いましたね。  
今まで使い分けを考えてなかったですが、雑に考えると次のような感じでしょうか？

- `@inheritDoc`はドキュメントを継承する時に使う
- `{@inheritDoc}`はドキュメントを継承し、さらに情報を追加する時に使う

個人的には細く使い分けるより、どちらかに統一して書く方が良いと思いました。
