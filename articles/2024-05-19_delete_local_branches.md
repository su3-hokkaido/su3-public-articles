---
title: "【週末おそうじ】ローカルブランチをサクッと削除したい" # 記事のタイトル
emoji: "🧽" # Choose an appropriate emoji for each article
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Git", "GitHub", "branch", "ブランチ", "GithubCLI"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
# なにこれ
ローカルブランチがたくさん溜まってきたので削除したいなあと思って調べたコマンドをまとめた個人的なメモ。


# 特定のブランチを残して他の不要なブランチを削除する
### 残したいブランチが1つの場合
`develop` ブランチを残したい場合のコマンドです。
```zsh
git branch | grep -v "develop" | xargs git branch -d
```

### 残したいブランチが複数ある場合
`master` と `develop` ブランチを残したい場合のコマンドです。
```zsh
git branch | grep -v -e "master" -e "develop" | xargs git branch -d
```


# 全部のローカルブランチを削除する
おそらくこのケースはあまりないように思いますが、とりあえずコマンドを書き残します。
```zsh
git branch | xargs git branch -d
```


# マージされていないブランチを強制的に削除する
`-d` オプションを `-D` に変更すると強制削除になる模様。
以下にサンプルコマンドとして `develop` ブランチを残して他のローカルブランチをすべて削除したケースを書きます。
```zsh
git branch | grep -v "develop" | xargs git branch -D
```


# 最後に
使っていないブランチはどれが何のブランチなのかわからなくなってくるので、定期的にお掃除したいですね。
仕事では全然やってないけど、個人的な趣味でやっている開発は週末くらいしかやらないので忘れやすいし、たまに整理したい。

