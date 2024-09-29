---
title: "Gitマージでdry-runをやってみる"
emoji: "📌"
type: "tech"
topics:
  - "git"
published: true
published_at: "2022-09-13 22:54"
---

調べればいくらでも出てきそうな内容だけど自分で試したのでやり方を書いておくことにする。

```bash
$ git merge --no-commit --no-ff ブランチ名
```

問題なければcommitする。  
元に戻すときは下記を実行する。

```sh
$ git merge --abort
```

個人的にいきなりマージするより、1回確認できた方が良いので役立っている。  
