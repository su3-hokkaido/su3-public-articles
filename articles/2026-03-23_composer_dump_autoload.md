---
title: "composer dump-autoload を忘れて HTTP 500 にハマった話"
emoji: "🔰"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["PHP","Laravel","Composer","Docker","初心者"]
published: true
---
# これなに

`composer.json` のオートロード設定を変更したあと `composer dump-autoload` を実行し忘れると、PHPが起動時に Fatal error を起こして HTTP 500 になることがあります。本記事はその問題の仕組みと対処法について個人的なメモとして記載します。


# 発生したこと

ローカル環境で動作確認しようと思い、 Docker で作成した Laravel アプリケーションにアクセスしたところ、ブラウザに HTTP 500 が表示されました。
Laravel のログには何も出力されておらず、コンテナ内で `php artisan` を実行すると以下のエラーが出力されていました。

```bash
PHP Fatal error: Uncaught Error: Failed opening required
  '/var/www/html/tests/Helper/TestConstants.php'
  in /var/www/html/vendor/composer/autoload_real.php:41
```


# 原因

`composer.json` に登録されたファイルパスが変更されていたにもかかわらず、`vendor/composer/` 配下のオートローダーファイルが古いパスのまま残っていたことが原因だったようです。

```bash
composer.json:          tests/Helpers/TestConstants.php  (変更後・複数形)
autoload_static.php:    tests/Helper/TestConstants.php   (変更前・単数形)
```

つまり、**ソースコードとオートローダーのマッピングにズレが生じていた**ことが原因のようでした。


# そもそも composer dump-autoload とは

##＃ PHP のオートロードの仕組み

PHP では、クラスを使うたびに `require` でファイルを読み込む必要があります。Composer はこれを自動化するために、「クラス名 → ファイルパス」のマッピングを `vendor/composer/` 配下に生成します。

主なファイル:

| ファイル | 役割 |
|---|---|
| `autoload_psr4.php` | PSR-4 の名前空間 → ディレクトリのマッピング |
| `autoload_classmap.php` | クラス名 → ファイルパスの直接マッピング |
| `autoload_static.php` | 上記を静的配列にまとめた高速版 |
| `autoload_files.php` | `composer.json` の `files` で指定されたファイル一覧 |

### composer dump-autoload の役割

`composer.json` の `autoload` / `autoload-dev` セクションを読み直し、上記のマッピングファイルを再生成するコマンドです。パッケージのインストール・更新は行いません。

```bash
# 基本
composer dump-autoload

# 最適化（クラスマップを事前に全解決。本番向け）
composer dump-autoload --optimize
```

### composer install / update との違い

| コマンド | パッケージ操作 | オートローダー再生成 |
|---|---|---|
| `composer install` | する | する |
| `composer update` | する | する |
| `composer dump-autoload` | しない | する |

`composer install` や `composer update` を実行すればオートローダーも再生成されますが、パッケージの変更が不要なときは `dump-autoload` だけで十分なようです。


# 見落としやすい理由

### 1. エラーメッセージが Composer を示唆しない

エラーの発生箇所は `vendor/composer/autoload_real.php` でしたが、メッセージには「ファイルが見つからない」としか出てきません。ファイルのリネームやディレクトリ移動を疑いがちで、オートローダーの再生成という発想にたどり着きにくかったですね。

### 2. Laravel のログに残らない

Composer のオートローダーは Laravel の bootstrap より前に実行されます。そのため、Laravel のエラーハンドリングやログが動く前に PHP が Fatal error で停止し、`storage/logs/` には何も出力されないことも原因のひとつでした。

### 3. Docker 環境での仕組みの見落とし

ホスト側で `composer dump-autoload` を実行しても、コンテナ内の `vendor/` がボリュームマウントで分離されている場合、コンテナ内のオートローダーは更新されません。逆もまた然りです。**どちらの環境のオートローダーが古いのかを意識する必要があります。**


# 本事象が再現しやすいケース

- `composer.json` の `autoload.files` に登録したファイルをリネーム・移動した
- `autoload.psr-4` の名前空間やディレクトリ名を変更した
- ブランチを切り替えたら `composer.json` のオートロード設定が異なっていた
- Docker イメージをリビルドしたが `composer dump-autoload` を含めていなかった
- CI/CD でキャッシュされた `vendor/` を使い回している


# 対処法と予防策

### 対処法: オートローダーの再生成

```bash
# ローカル
composer dump-autoload

# Docker コンテナ内
docker exec <container> composer dump-autoload -d /path/to/project
```

### 予防策: composer.json 変更時のチェックリスト

1. `autoload` / `autoload-dev` セクションを変更したら `composer dump-autoload` を実行する
2. Docker を使っている場合は、**実行環境側のコンテナ内で**実行する
3. Dockerfile の `RUN` に `composer dump-autoload --optimize` を含める
4. CI/CD で `vendor/` をキャッシュしている場合は、`composer.json` の変更をキャッシュキーに含める


# まとめ

いろいろと書きましたが、まとめると以下のポイントに意識していけば良いのだと今回学習しました。
みなさんも自分と同じ問題にブチ当たらないようにお気をつけくださいませ。。。

- `composer dump-autoload` は、`composer.json` のオートロード設定からマッピングファイルを再生成するコマンド
- `composer.json` のパスを変更しても `vendor/composer/` のマッピングは自動更新されません
- Laravel のログに残らないため、原因特定が遅れやすい
- Docker 環境ではホストとコンテナどちらのオートローダーが古いかを意識することが必要
- `autoload` セクションを変更したら、必ず `composer dump-autoload` を実行
