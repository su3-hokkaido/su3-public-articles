# Su3's article repo

## Overview

This is a repo which I manage my articles written in the markdown format.

## Blogs or article URLs

- Qiita: https://qiita.com/su3-hokkaido
- Zenn: https://zenn.dev/su3_hokkaido
- note: https://note.com/su3_hokkaido

## How to use Zenn CLI

### Official Guide

https://zenn.dev/zenn/articles/zenn-cli-guide

### Header format

L1-L8 should align with the following format.

```md
---
title: "VSCode でのタブキーでの半角スペースの数を変更する" # 記事のタイトル
emoji: "🎸"
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["editor", "setting", "Keyboard", "VSCode"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
```

### Create new article file

```bash
npx zenn new:article
```

### Preview

```bash
# Default: localhost:8000
npx zenn preview
```

```bash
# Specify a port number
npx zenn preview --port 3000
```

## How to use Qiita CLI

### Official Guide

https://github.com/increments/qiita-cli

### Header format

```md
---
title: # Title
tags: # Up to 5 tags can be written such as follows
  - Android
  - Mobile
  - iOS
  - Dart
  - Flutter
private: false
updated_at: # Auto generate
id: # Auto generate
organization_url_name: null
slide: false
ignorePublish: false
---
```

### Login

Run the following command & input your access token if CLI asks you to input it.

```bash
npx qiita login
```

### Preview articles

By running the following command, it downloads articles from Qiita.

```bash
npx qiita preview
```

And you can see articles on your local via [localhost:8888](localhost:8888).

### Create an initial markdown file

Here is a sample command.

```bash
# npx qiita new [a file name]
npx qiita new new_article
```

### Create or update articles on Qiita

Here is a sample command.

```bash
# npx qiita publish [a file name]
npx qiita publish new_article
```

### Bulk-create or bulk-update articles on Qiita 

```bash
npx qiita publish --all
```

### Delete articles

Qiita CLI doesn't provide a command to delete uploaded articles.
Please delete your article via Qiita GUI.

### Pull the latest articles from Qiita

Normal command

```bash
npx qiita pull
```

Force to pull

```bash
npx qiita pull --force
```

### Confirm Qiita CLI version

```bash
npx qiita version
```

### Confirm the credential information

Confirm the following file

```bash
~/.config/qiita-cli/credentials.json
```
