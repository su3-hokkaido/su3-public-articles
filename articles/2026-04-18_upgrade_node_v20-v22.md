---
title: "Node v20 => v22 アップグレード後に Mixed Content をやらかしたので ClaudeCode と一緒に対応した記録"
emoji: "🚨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Laravel","vite","AWS","ALB","初心者"]
published: true
---
## これなに

Node v20 => v22 にアップグレードした際に画面が真っ白になってしまったので、その際に発生した Mixed Content とかいうエラーに対処した際の記録です。

## 発生したこと

ブラウザで DevTools の Console を開くとこんなエラーが。

```bash
Mixed Content: The page at 'https://example.co/admin/login' was loaded over HTTPS,
but requested an insecure script 'http://example.co/build/assets/admin-XXX.js'.
This request has been blocked; the content must be served over HTTPS.
```

`https` で開いているページが、`http` のスクリプトをロードしようとしてブラウザに全部ブロックされていたようです。

## 問題発生前後での条件

今回の問題にさしあたって、状況を整理すると

1. 直前のデプロイで環境変数も nginx 設定も触っていない
2. Node アップグレードしただけ
3. アップグレード前は以前は同じ構成で問題なく動いていた

## 環境

- **アプリケーション**: Laravel 13.4.0
- **PHP**: 8.3
- **インフラ**: AWS（EC2 + ALB）
  - ALB が TLS 終端
  - EC2 上の nginx は 80 番のみ Listen（443 を持たない）
- **フロントエンド**: Vite + Vue + Inertia 系
- **問題発生時のフロントビルド環境**:
  - Node v22 にアップグレード済み
  - `vite: ^8.0.8`
  - `laravel-vite-plugin: ^3.0.1`
  - `@vitejs/plugin-vue: ^6.0.5`

## 調査の記録

### Step 1: まず `APP_URL` を確認

何かの拍子で `APP_URL` が書き変わってしまった可能性があるので確認

```bash
$ grep APP_URL /var/www/example/current/.env
APP_URL=https://example.co
```

→ ✅ `https` なので正しい。よって、これは原因ではない。

### Step 2: ALB 配下を疑う（TrustProxies 不在?）

ALB が TLS 終端する構成では、EC2 内の nginx には HTTP でリクエストが届きます。
Laravel が「自分は HTTP で動いている」と誤認すると、`asset()` などが `http://` を生成してしまう、というのは Laravel + ALB 構成で発生しやすい問題なので確認。

```bash
$ sudo ss -tlnp | grep -E ':80|:443'
LISTEN 0  511  0.0.0.0:80  0.0.0.0:*  users:(("nginx",...))
```

→ nginx は **80 番のみ Listen**。443 を持っていない = ALB が TLS 終端していることがほぼ確定。

nginx access.log を見ると：

```bash
10.1.0.169 - - [18/Apr/2026:13:40:37 +0900] "GET / HTTP/1.1" 404 ...
                                            "ELB-HealthChecker/2.0"
```

VPC 内 IP からの ALB ヘルスチェックが定期的に来ており、**ALB 配下構成で確定**。

### Step 3: TrustProxies の設定をトライ

Laravel 11 以降は `bootstrap/app.php` で TrustProxies を設定することになっているようなので、それをトライ：

```php
use Illuminate\Http\Request;

return Application::configure(basePath: dirname(__DIR__))
    ->withMiddleware(function (Middleware $middleware) {
        $middleware->trustProxies(
            at: '*',
            headers: Request::HEADER_X_FORWARDED_FOR
                | Request::HEADER_X_FORWARDED_HOST
                | Request::HEADER_X_FORWARDED_PORT
                | Request::HEADER_X_FORWARDED_PROTO
                | Request::HEADER_X_FORWARDED_AWS_ELB,
        );
    })
    // ...
```

これを追加して `php artisan optimize` を実行。`tinker` で確認：

```php
>>> request()->isSecure()
= true
```

ところが、ブラウザの **InPrivate モード**（キャッシュ無し）で開いても **Mixed Content が再現**。そこでサーバが返す HTML を直接確認してみる：

```bash
$ curl -sk https://example.co/admin/login | grep -oE '<(script|link)[^>]*' | head -5
<link rel="modulepreload" as="script" href="http://example.co/build/assets/admin-XXX.js" />
...
```

→ HTML 内の URL がまだ `http://`。**TrustProxies が効いていない**。

## 応急処置：`URL::forceScheme('https')` でいったん復旧

切り分けに時間をかけている間もサービスが止まっているので、まず確実に復旧させることを優先するため `app/Providers/AppServiceProvider.php` の `boot()` に forceScheme を追加：

```php
public function boot(): void
{
    if ($this->app->environment('production')) {
        \Illuminate\Support\Facades\URL::forceScheme('https');
    }

    // ... 既存の処理
}
```

```bash
$ php artisan optimize:clear
$ php artisan optimize
$ sudo systemctl reload php8.3-fpm  # OPcache クリア
```

→ ブラウザでスーパーリロード → ✅ 画面表示、CRUD 動作確認 OK。

## 根本原因の調査

応急処置で本番が動いた後、根本原因の調査を続行。
重要なポイントとしては、 **なぜ以前は同じ構成（TrustProxies 未設定）で動いていたのか？** という点なので、ここに着目して調査を開始。

### サーバから返る HTML を再確認

```bash
$ curl -sk https://example.co/admin/login | grep -oE '<(script|link)[^>]*' | head -10
<link rel="modulepreload" as="script" href="https://example.co/build/assets/admin-XXX.js" />
<link rel="preload" as="style" href="https://example.co/build/assets/i18n-XXX.css" />
...
```

forceScheme の効果で **scheme は `https` に修正されている**ものの、よく見ると URL がすべて **絶対 URL（`https://example.co/build/...`）** になっているようでした。そこで、それを切り口に調査を続行。

### Vite manifest の中身

```bash
$ head -20 public/build/manifest.json
{
  "_AnnouncementShow-XXX.js": {
    "file": "assets/AnnouncementShow-XXX.js",
    "name": "AnnouncementShow",
    ...
```

manifest 内のパスは **相対パス**（`assets/AnnouncementShow-XXX.js`）。つまり「絶対 URL 化」をしているのは **Laravel の `@vite()` ディレクティブ側**ということ。

### 公式ドキュメントを確認

Laravel 13 の Vite 公式ドキュメントを確認すると、以下の記述を確認：

> Prior to version 3 of the Laravel Vite plugin, static assets had to be imported in your application's entry point using import.meta.glob. **The assets option was introduced due to changes in Vite 8.**
> 
> — [Laravel 13.x Vite ドキュメント](https://laravel.com/docs/13.x/vite)

そして `@vite()` の挙動について、GitHub Discussions より：

> the @vite directive automatically prepends the site origin in the asset urls
>
> — [laravel/framework Discussion #43117](https://github.com/laravel/framework/discussions/43117)

つまり：

- **Vite 8 はメジャーアップデートで破壊的変更**を含む
- それに合わせて **`laravel-vite-plugin` も v3 にメジャーアップ**
- v3 の `@vite()` は **必ずサイトのオリジン（scheme + host）を asset URL に付与**する

この組み合わせ更新が、Node アップグレードに伴う `npm install` で問題が発生したようでした。

### 今回の問題発生の経緯サマリ

```
[1] package.json の caret range (^...) で Vite 系が幅広く許容されていた
    ↓
[2] Node v20 → v22 アップグレード時に npm install を再実行
    ↓
[3] Vite が v8 にメジャー更新 + laravel-vite-plugin が v3 へ
    ↓
[4] @vite() の URL 生成が「絶対 URL 必須」に挙動変化
    （以前は相対パス or scheme 継承形式だった可能性が高い）
    ↓
[5] ALB は X-Forwarded-Proto: https を送るが、TrustProxies 未設定なので
    Laravel は request scheme = http と認識
    ↓
[6] @vite() が "http://example.co/build/..." を出力
    ↓
[7] ブラウザは https ページから http リソースをロードしようとして Mixed Content
    ↓
[8] 全てブロックされて画面真っ白
```

### 「以前なぜ動いていたか」の推測

過去の `laravel-vite-plugin` (v1/v2) 系では、scheme を含めない以下のような形式で出力していた可能性が高いと考えられます：

- プロトコル相対 URL: `//example.co/build/...`
- パス相対 URL: `/build/assets/...`

これらはブラウザが現在のページの scheme を継承するため、Mixed Content にならない。**v3 で「常に絶対 URL（scheme 込み）」に統一したことが回帰の原因**だろうなと考えています。

---

## 恒久対応の選択肢

3つのオプションがあり、最終的にCで手打ちとしました。

### オプション A: `URL::forceScheme('https')`

```php
// AppServiceProvider::boot()
if ($this->app->environment('production')) {
    URL::forceScheme('https');
}
```

| ✅ メリット | ❌ デメリット |
|---|---|
| 確実に効く | コードに環境依存ロジックを書くことになる |
| 全 URL 生成（asset, route, etc）に効く | `request()->ip()` などプロキシ依存メソッドは直らない |
| 1行で済む | `request()->isSecure()` は依然として false |

### オプション B: `.env` の `ASSET_URL`

```bash
# .env
ASSET_URL=https://example.co
```

公式ドキュメントより：

> If your Vite compiled assets are deployed to a domain separate from your application, such as via a CDN, you must specify the ASSET_URL environment variable within your application's .env file
>
> — [Laravel 12.x Vite ドキュメント](https://laravel.com/docs/12.x/vite)

GitHub Discussions では同じ Mixed Content 問題への公式回答として案内されています。

| ✅ メリット | ❌ デメリット |
|---|---|
| 設定が `.env` で完結（コードを汚さない） | 効果範囲は asset URL のみ |
| 環境別に切り替えやすい | `.env` 管理を忘れると効かない |
| CDN 移行時にもそのまま使える | プロキシ依存メソッドは直らない |

### オプション C: TrustProxies を正しく設定

```php
// bootstrap/app.php
->withMiddleware(function (Middleware $middleware) {
    $middleware->trustProxies(
        at: '*',
        headers: Request::HEADER_X_FORWARDED_FOR
            | Request::HEADER_X_FORWARDED_HOST
            | Request::HEADER_X_FORWARDED_PORT
            | Request::HEADER_X_FORWARDED_PROTO
            | Request::HEADER_X_FORWARDED_AWS_ELB,
    );
})
```

| ✅ メリット | ❌ デメリット |
|---|---|
| **最も「正しい」設計** | OPcache クリア漏れなど落とし穴がある |
| `request()->isSecure()` も true になる | EC2 SG で 80 番を ALB SG からのみ許可する前提が必要 |
| `request()->ip()` が真のクライアント IP を返す | spoofing 対策の SG 設計を理解する必要 |

### 推奨：オプション C を本対応 + A or B を defense in depth として残す

実務上は **オプション C を恒久対応とし、A or B を保険として残す**ことを推奨します。
理由としては以下の通り：

1. **C だけだと将来 ALB 設定変更で `X-Forwarded-Proto` が落ちた場合に再発する**
2. **A or B だけだと `request()->ip()` などのプロキシ依存メソッドが壊れたまま**
3. asset URL の不正出力は本番で目立つ（画面が真っ白になる）が、IP の不正取得は静かに壊れる（rate limit、監査ログ、IP 制限が機能しない）

```php
// production は AWS ALB 配下で TLS 終端されるため URL 生成を https に強制する。
// 主対応は bootstrap/app.php の TrustProxies。本処理は ALB 設定変更等で
// X-Forwarded-Proto が落ちた場合の defense in depth として残す。
if ($this->app->environment('production')) {
    URL::forceScheme('https');
}
```

---

## 切り分けで使った検証コマンド集

同じトラブルに当たった人が再利用できるよう、本件で使った検証コマンドをまとめておきます。

### サーバが返す HTML 内の asset URL を確認

```bash
curl -sk https://your-domain/path | grep -oE '<(script|link)[^>]*' | head -10
```

scheme が `http://` なら問題、`https://` なら OK。

### ALB が前段にいるかの確認

```bash
sudo ss -tlnp | grep -E ':80|:443'
sudo tail -n 50 /var/log/nginx/access.log | grep ELB
```

nginx が 443 を持たず、ELB-HealthChecker からのアクセスがあれば ALB 配下確定。

### Laravel が現在の HTTP リクエストをどう認識しているか確認

`routes/web.php` に一時的に追加：

```php
Route::get('/_debug/scheme', function (\Illuminate\Http\Request $request) {
    return response()->json([
        'isSecure'        => $request->isSecure(),
        'scheme'          => $request->getScheme(),
        'x_fwd_proto'     => $request->header('X-Forwarded-Proto'),
        'x_fwd_for'       => $request->header('X-Forwarded-For'),
        'remote_addr'     => $request->server('REMOTE_ADDR'),
        'trusted_proxies' => $request->getTrustedProxies(),
    ]);
});
```

```bash
php artisan route:clear && php artisan route:cache
sudo systemctl reload php8.3-fpm
```

ブラウザで `/_debug/scheme` にアクセスして JSON を確認。各値の意味：

| 結果 | 意味 |
|---|---|
| `trusted_proxies: []` | TrustProxies 設定が反映されていない（OPcache or 構文ミス） |
| `x_fwd_proto: null` | nginx → php-fpm で X-Forwarded-Proto が落ちている |
| `isSecure: false` で他は OK | TrustProxies の `at` の値などが不適切 |
| 全部期待値 | TrustProxies 正常動作 |

## 今回の教訓

### 1. `tinker` の `request()->isSecure()` をそのまま信用しないこと

CLI で実行される `tinker` は実 HTTP リクエストを持たないため、`isSecure()` の結果は実環境の挙動を反映しません。**実 HTTP の検証は必ずブラウザかデバッグルートで行う**こと。

### 2. `php artisan optimize:clear` は OPcache をクリアしないこと

`bootstrap/app.php` を編集しても、PHP-FPM プロセスの OPcache が古いコードを掴んだままになることがあります。設定変更後は：

```bash
sudo systemctl reload php8.3-fpm
```

を実行した方がベター。

### 3. Node アップグレードは周辺の影響を必ず確認すること

「Node のバージョンを上げただけ」と思っていても、`npm install` を再実行すれば `package.json` の caret range で許容された範囲内で **依存パッケージのメジャーアップが連鎖**します。特にフロントエンドツールチェーン（Vite、各種 plugin）は破壊的変更が比較的多いため、**Node アップグレードは実質的にフロントビルド全体のアップグレード**だと考えるべき。

対策案：
- `package-lock.json` を CI で固定し、明示的な意図がある時のみ更新する
- `npm-check-updates` などで事前に何が上がるかを確認する
- メジャーアップが含まれる時は staging で十分検証する

### 4. インフラ構成と Laravel 設定の整合性は最初に確認する

ALB / CloudFront / Cloudflare などのリバースプロキシ配下で Laravel を動かす場合、**TrustProxies の設定はインフラ移行時に必ず確認すべき項目**。
具体的には以下を一度棚卸しすると安心かなと思います：

- `request()->isSecure()` が正しく動くか
- `request()->ip()` が真のクライアント IP を返すか
- `request()->getHost()` がサブドメイン判定に使えるか（マルチテナント SaaS の場合特に重要）
- セッション Cookie の Secure 属性が付くか

### 5. 応急処置と恒久対応を明確に分ける

`URL::forceScheme('https')` のような応急処置は **コメントで意図を明示** し、恒久対応のチケット番号や日付をコードに残すこと。（「Hotfix」とだけ書かれた数行が、3ヶ月後には誰も触れない聖域になっている、というのはあるある）

## 余談１：今回ついでに見つかった別件

切り分けの過程で、ALB / nginx 周りに別の課題も見つかりました。本記事の本題とは別ですが、同じ構成のチームへの参考として。

### nginx access.log のクライアント IP が ALB の内部 IP

```
10.1.0.169 - - [...] "GET /admin/login HTTP/1.1" 200
10.1.1.98  - - [...] "GET /admin/login HTTP/1.1" 200
```

VPC 内の ALB の IP しか記録されておらず、**真のクライアント IP がログに残らない**状態。fail2ban、rate limit、監査などが機能しません。

対策は nginx 側に `ngx_http_realip_module` の設定を入れること：

```nginx
# /etc/nginx/conf.d/realip.conf
set_real_ip_from 10.1.0.0/16;   # VPC CIDR に合わせる
real_ip_header X-Forwarded-For;
real_ip_recursive on;
```

### ALB ヘルスチェックが `GET /` で 404 を返している

```
10.1.0.169 - - [...] "GET / HTTP/1.1" 404 ... "ELB-HealthChecker/2.0"
```

ヘルスチェックが 404 を返し続けているのに ALB がターゲットを healthy 判定しているのは、ALB のヘルスチェック設定で「Success codes」に `200-499` のような幅広い範囲が指定されているからと推測されます。

対策はターゲットグループのヘルスチェック設定で：
- パスを `/admin/login` のような 200 が返るパスに変更する
- もしくは専用の `/healthz` ルートを Laravel 側に追加して 200 を返す
- Success codes を `200` のみに絞る

## 余談２：めちゃくちゃ ClaudeCode に頼った

実のところ結構わからないことが多いので、Claude さんにめちゃくちゃ頼りました…
たぶん以前なら数時間かかっていた調査が数十分で終わったので、AI怖い…と思いました😰

## まとめ

| 項目 | 内容 |
|---|---|
| **症状** | ALB 配下の Laravel で本番デプロイ後、画面真っ白（Mixed Content） |
| **直接原因** | `@vite()` が asset URL を `http://` で出力 |
| **根本原因** | Node v22 アップグレード時に Vite v8 + laravel-vite-plugin v3 へメジャーアップ。`@vite()` の URL 出力が絶対 URL 化したことで、TrustProxies 未設定が露見 |
| **応急処置** | `URL::forceScheme('https')` を `AppServiceProvider::boot()` に追加 |
| **恒久対応** | TrustProxies 設定 + `forceScheme` を保険として残す |
| **教訓** | Node アップグレードは「フロントビルド全体のアップグレード」。インフラ構成と Laravel 設定の整合性は定期的に確認すべき |

同じ構成で運用しているチームの参考になれば幸いです💡

## 参考リンク

- [Asset Bundling (Vite) | Laravel 13.x](https://laravel.com/docs/13.x/vite)
- [How to change the assets URL origin added by `@vite` directive? · laravel/framework Discussion #43117](https://github.com/laravel/framework/discussions/43117)
- [Vite 公式](https://vitejs.dev/)
- [laravel-vite-plugin GitHub](https://github.com/laravel/vite-plugin)
- [Configuring Trusted Proxies | Laravel](https://laravel.com/docs/13.x/requests#configuring-trusted-proxies)
