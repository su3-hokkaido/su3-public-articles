---
title: "Docker ビルドが 401 で認証エラーになったのでログインし直したら治った話" # 記事のタイトル
emoji: "🐳" # Choose an appropriate emoji for each article
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Docker", "docker-compose", "初心者"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
# これなに

- docker-compose build を実行した時にビルドができなくなった
- 大した内容では無いですが、その時何が原因で、何をしたかの記録をします
- そのため深くは考察しておらず、「こういった内容があるんだ」程度の内容として気軽にご覧いただければと思います

# エラー内容

```zsh
failed to solve: php:8.4-fpm: failed to resolve source metadata for docker.io/library/php:8.4-fpm: unexpected status from HEAD request to https://registry-1.docker.io/v2/library/php/manifests/8.4-fpm: 401 Unauthorized
```

# 原因

エラーメッセージにある通りで、Docker Hub の認証エラーになっているようでした

# 対応方法

基本的には Docker Hub に入り直して解決しました

```zsh
docker logout # 念のためログアウト
docker login  # ログインし直し
```

# その他実行したこと

一応以下も実行しましたが、これらのアクション自体は解決に至りませんでした

- Docker Desktop の再起動
- Docker Desktop の再インストール

# 参考：エラー内容全文

Dockerfile に記載していた `FROM php:8.4-fpm` でエラーになるなんておかしいなあと思ったら単なる認証エラーでした…

```zsh
% docker-compose build
[+] Building 2.4s (11/11) FINISHED                                                                                      
 => [internal] load local bake definitions                                                                         0.0s
 => => reading from stdin 362B                                                                                     0.0s
 => [internal] load build definition from Dockerfile                                                               0.0s
 => => transferring dockerfile: 1.52kB                                                                             0.0s
 => CANCELED [internal] load metadata for docker.io/library/composer:latest                                        2.1s
 => ERROR [internal] load metadata for docker.io/library/php:8.4-fpm                                               2.1s
 => [auth] library/composer:pull token for registry-1.docker.io                                                    0.0s
 => [auth] library/php:pull token for registry-1.docker.io                                                         0.0s
 => [auth] library/php:pull token for registry-1.docker.io                                                         0.0s
 => [auth] library/php:pull token for registry-1.docker.io                                                         0.0s
 => [auth] library/composer:pull token for registry-1.docker.io                                                    0.0s
 => [auth] library/php:pull token for registry-1.docker.io                                                         0.0s
 => [auth] library/composer:pull token for registry-1.docker.io                                                    0.0s
------
 > [internal] load metadata for docker.io/library/php:8.4-fpm:
------
Dockerfile:1

--------------------

   1 | >>> FROM php:8.4-fpm

   2 |     

   3 |     # Set working directory

--------------------

failed to solve: php:8.4-fpm: failed to resolve source metadata for docker.io/library/php:8.4-fpm: unexpected status from HEAD request to https://registry-1.docker.io/v2/library/php/manifests/8.4-fpm: 401 Unauthorized

make: *** [build] Error 1
% 
```
