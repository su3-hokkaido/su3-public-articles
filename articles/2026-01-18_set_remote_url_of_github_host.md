---
title: "GitHub アカウントを複数使っている時に各リポジトリの作業アカウントを設定する手順メモ"
emoji: "🔰"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['Git','GitHub','初心者','個人メモ','フリーランス']
published: true
---
# これなに

- 自分は個人開発用と仕事用などで複数のGitHubアカウントを運用しています
- その際に各リポジトリで作業しているアカウントを設定したく、リモートURLのホスト名をSSH configで設定したホスト名に変更する手順をまとめました

# この記事で想定している対象読者

- 個人開発用と仕事用でGitHubアカウントを分けて利用されている方
- 業務委託で会社ごとにアカウントを複数運用されている方

# 事前準備
### ローカルマシンに Git をインストールする

公式ガイドをご覧ください。

https://git-scm.com/install/

### GitHub アカウントを作成する

同様に公式ガイドをご覧ください。

https://docs.github.com/ja/get-started/start-your-journey/creating-an-account-on-github

### SSH key を作成して GitHub に登録する

同様に公式のドキュメントをご覧ください。
SSHキーの作成自体も記載がありますが、自分の個人メモ記事も掲載しておきます。

https://docs.github.com/ja/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent

https://qiita.com/su3-hokkaido/items/05e9fb6b766bfefcc4f3

### SSH config で各アカウントのホスト名を設定する

作成した SSH キーの情報を元に `~/.ssh/config` を以下のように編集します。
アカウントごとに任意のホスト名を設定してください。

```zsh
# github-work, github-su3 は任意の名称でお願いします
Host github-work
    HostName github.com
    IdentityFile ~/.ssh/sshkey-github-su-work-profile
    User git
    IdentitiesOnly yes
    AddKeysToAgent yes
    UseKeychain yes

Host github-su3
    HostName github.com
    IdentityFile ~/.ssh/sshkey-github-su3-hokkaido
    User git
    IdentitiesOnly yes
    AddKeysToAgent yes
    UseKeychain yes
```

# リモートURLのホスト名をSSH configで設定したホスト名に変更する手順

ターミナルから以下のコマンドを対象のリポジトリのルートディレクトリで実行します。
これだけです、すごく簡単。

```bash
# git remote set-url origin git@[指定したいGitHubアカウントのホスト名]:[リポジトリのgitファイルパス]
git remote set-url origin git@github-su3:su3-hokkaido/sample_laravel.git
```

# あとがき

結構アカウントが違うことでgit操作ができなかったりする問題によく出会い、その度にコマンドを探しているのが面倒になったので個人メモとしてこの記事を作成しました。
そのため運用するに当たって必要最低限の情報を記載しただけなので、もしさらに細かい内容を知りたいという方はぜひ色々調べてみてください。
記事内容に誤りや他にも有益な情報があればぜひコメントで教えてください🙇‍♂️
