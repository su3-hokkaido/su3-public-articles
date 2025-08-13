---
title: ssh キー生成コマンド（個人メモ）
tags:
  - SSH
  - ssh-keygen
private: false
updated_at: '2024-11-04T12:32:53+09:00'
id: 05e9fb6b766bfefcc4f3
organization_url_name: null
slide: false
ignorePublish: false
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
