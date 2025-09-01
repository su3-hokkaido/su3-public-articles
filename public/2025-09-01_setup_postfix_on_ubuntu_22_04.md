---
title: Ubuntu 24.04 で postfix を導入する
tags:
  - PHP
  - Ubuntu
  - postfix
  - 初心者
  - Laravel
private: false
updated_at: '2025-09-01T23:06:48+09:00'
id: 868d4ff90a139c73b07e
organization_url_name: null
slide: false
ignorePublish: false
---
# これなに

- ちょっと今お手伝いしているところで postfix 導入をすることになった
- 実際ちゃんとやったことないので、作業ログとして記録
- そのため記事内容が間違っていてもご容赦ください

# 前提条件

- Ubuntu 24.04

```zsh
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 24.04 LTS
Release:	24.04
Codename:	noble
$ 
```

# セットアップ手順

### postfix インストール

apt アップデートを行ってからインストールします

```zsh
sudo apt update
sudo apt install postfix
```

### main.cf 設定編集

以下のパラメーターを設定します。

- myhostname：サーバーのホスト名
- inet_interfaces：`loopback-only` を設定して受信を無効化する
- relayhost：メールを送信する中継サーバー設定
  - 値なし: 送信先のメールサーバーにダイレクトに送信
  - 値あり: 設定したSMTPサーバーを中継して送信

サンプルは以下の通りです。

```zsh
myhostname = mail.example.com
myorigin = /etc/mailname
inet_interfaces = loopback-only
relayhost =
```

### postfix 再起動

設定内容を反映するために再起動します

```zsh
sudo systemctl restart postfix
```

### SPFレコードをDNSに追加

以下をTXTレコードとしてメインのDNSレコードに追加

```
v=spf1 a mx ip4:[IP address of server] ~all
```

### 設定内容の確認

ここまでの設定内容を確認します。
内容が正常に表示されればOK

```zsh
dig example.com TXT
```

# 付録

## 終わったらメールが送れるか確認しましょう

```zsh
curl -X POST https://examle.com/api/user/auth/signup \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "name": "Test User", 
    "email": "su3.hokkaido@example.com"
  }'
```

## いろいろ確認するのに使ったコマンド集

Laravel で構築しているのですが、その際にいろいろ使った確認コマンドをメモ

### メール設定を確認

```bash
php artisan tinker
>>> config('mail')
>>> config('app.url')
```

### ログの確認

ログ監視

```bash
sudo journalctl --since "1 minute ago" | grep -i postfix
```

Laravelのログ確認

```bash  
tail -f /var/www/hdmp/current/storage/logs/laravel.log
```

### キューの確認

キュー設定確認

```bash
php artisan tinker
>>> config('queue.default')
```

キューワーカーが動いているか確認

```bash
sudo systemctl status laravel-queue-worker
```

もしくは

```bash
ps aux | grep queue
```

# さいごに

完全初心者なメモなので、ところどころ足りてない部分があるかもしれません。
そして多分似たような記事がたくさんあると思うので、類似記事も参考にしていただけると良いかと思います。
