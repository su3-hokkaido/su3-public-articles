---
title: "ssh キー生成コマンド（個人メモ）" # 記事のタイトル
emoji: "🔑" # Choose an appropriate emoji for each article
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["SSH", "初心者"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
# What

単なる個人メモ。

たまに使うけど、覚えているわけではなくていつも公式ガイドを探しに行っているのも面倒になったので、ここにメモしておく。
passphrase はあった方が良いが、テキトーに遊ぶ分にはなくてもOKという気持ち。

```zsh
ssh-keygen -t ed25519 -f ~/.ssh/[なんかテキトーな任意の名前]
```


# Appendix

ごく稀に公開鍵、秘密鍵の知識がない人もいるので注意したい（実際に出会ったことがあるので怖い）
