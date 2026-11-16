---
title: "HaskellをインストールしてHello Worldを出してみた"
emoji: "🌿"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - "haskell"
  - "helloworld"
published: true
---

## はじめに

Haskellのインストールから**Hello World**を出すところまでをやってみます。

## 環境

- Mac

## インストール

Haskellの実行環境を整えるために、Homebrewを使ってGHC（Glasgow Haskell Compiler）をインストールする。

```bash
$ brew install ghc
```

インストールが終わったら、バージョンを確認する。

```bash
$ ghc --version
The Glorious Glasgow Haskell Compilation System, version 9.14.1
```

## Hello Worldを出してみる

HelloWorld.hsを作成し、次の内容を書きます。

```hs:HelloWorld.hs
main = putStrLn "Hello, World!"
```

保存後、**runghc HelloWorld.hs**を実行する。

:::message
runghcを使えばコンパイルせずに実行ができる。※ GHCと一緒にインストールされている。
:::

```bash
$ runghc HelloWorld.hs
Hello, World!
```

コンパイルし、実行する場合は、**ghc**を使う。

```bash
$ ghc HelloWorld.hs
$ ./HelloWorld
Hello, World!
```

無事にHello Worldを出すことができました。
