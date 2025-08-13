---
title: Microsoft Teams で Github の通知を受け取るためのセットアップ手順のメモ
tags:
  - 初心者
  - GitHub
  - Teams
  - Microsoft365
private: false
updated_at: '2025-08-12T20:43:26+09:00'
id: 9d1d5f578d856a94818c
organization_url_name: null
slide: false
ignorePublish: false
---
# これなに

- ちょっと別の会社でお手伝いをしていたのですが、そこの会社で使っているのが Slack ではなく Teams だった
- Teams で Github の通知設定をやったことなかったので、何をしたのか備忘録としてメモ

# セットアップ手順

## Github 側の設定

マーケットプレイスから Microsoft Teams for Github をリポジトリ、組織などにインストールします。

https://github.com/marketplace/microsoft-teams-for-github

![Setup Teams on Github (1)](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/abc2e558-996a-4762-a474-044782e64c3b.png)
![Setup Teams on Github (2)](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/731d3b43-2e66-4e93-a22e-a1d075527690.png)

## Teams 側の設定

Apps セクションから Github アプリをインストールします。

https://teams.microsoft.com/l/app/ca9e26b7-dce5-44a0-b2b7-a70a3d65ce25?source=app-details-dialog

![Setup Github on Teams](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/f58e702f-d3f2-4bf5-b07e-a7cd8ffe8936.png)

## Teams チャンネルでの通知設定の追加、解除、確認

### 通知設定の追加

基本的には対象のリポジトリをサブスクライブすれば通知を受け取れます。

```bash
# Sample: @github subscribe su3-hokkaido/test-repo
@github subscribe org/repository
```

issue だけ、プルリクだけ、というような場合は以下のように対象を指定することで対象を絞り込めます。
パラメーターはコメントに書いた通りで、コミットやコメントなんかも指定できます。

```bash
# Parameters: issues pulls commits comments releases deployments 
@github subscribe org/repository issues pulls
```

### 通知設定解除

unsubscribe することで通知設定を解除できます。

```bash
@github unsubscribe org/repository
```

同様に指定のものだけ解除することも可能。

```bash
# Parameters: issues pulls commits comments releases deployments 
@github unsubscribe org/repository issues pulls
```

### 通知設定確認

list で確認可能です

```bash
@github subscribe list

# Sample output is as follows:
# Subscribed to the following repository
# su3-hokkaido/.github
```

`features` をつけることで、何の通知を受け取っているのかも確認可能。

```bash
@github subscribe list features

# Sample output is as follows:
# Subscribed to the following repository
# su3-hokkaido/.github
# issues, pulls
```

## 通知内容

以下のような通知を受け取ることができるようになりました。よかったよかった。

![Teams channel notifications](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/010043fd-eba4-427f-9fba-0fcd503786f6.png)

# 参考資料

以下のリポジトリに設定内容が書いてあるのでそちらを参考にしました。

https://github.com/integrations/microsoft-teams#subscribe-notifications
