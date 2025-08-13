---
title: 【週末おそうじ】ローカルブランチをサクッと削除したい
tags:
  - Git
  - GitHub
  - branch
  - ブランチ
  - GithubCLI
private: false
updated_at: '2024-05-19T22:29:19+09:00'
id: 2f9aa9c2c82ccbc62818
organization_url_name: null
slide: false
ignorePublish: false
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

