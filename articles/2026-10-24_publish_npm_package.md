---
title: "Changesets と npm Trusted Publishing を利用して npm パッケージ公開フローを作ったメモ"
emoji: "🗂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["npm", "pnpm", "githubactions", "changesets", "TypeScript"]
published: true
---

## これなに

npm パッケージを公開したことなかったので、自分用に公開に必要な手順メモを残します。

（もともとはチームや組織で使えるようにしておきたいなあと思って作ったもので、特に誰かに使いまわしてもらう前提では作っていないのですが、作った当人が忘れてしまうのでメモを残しています）

## 対象読者

こんな感じの人たちが読んでもらえたら嬉しいです。

- npm パッケージってなに？と思っている
- npm パッケージを利用するだけで公開したことはない
- npm パッケージ作って汎用的に使いまわしたい

## npm パッケージとは

そもそも npm とは Node.js のパッケージを管理するシステム。アプリケーションを作成する際、便利なパッケージをそのプロジェクトにインストールして使用します。
パッケージは他のプログラムから利用することを目的としたクラスや関数などのプログラムのこと、俗世間でいうモジュールやらライブラリやらといった類のものです。
npm パッケージはつまりその2つを掛け合わせたもので、npm から提供されるパッケージなんだなあという感じで掴んでもらえれば良いかと思います。

（詳しいことは [npm のサイト](https://www.npmjs.com/)に飛んで行って読めば良いと思います、僕も全部はわかっていないので！）

## 今回のゴール

今回は以下の3つを達成することを目的として、npm パッケージ公開までの手順をメモしています。

- **npm のトークンをどこにも保存しない**（GitHub Secrets にも置かない）
- **バージョン番号と CHANGELOG を手で書かない**
- **テンプレートとしてコピーされたリポジトリやフォークから、誤って公開されない**

## 構成

| 役割 | 使ったもの |
|---|---|
| パッケージマネージャ | pnpm |
| バージョン管理・CHANGELOG | Changesets |
| 公開前チェック | publint, Are the Types Wrong? (attw), `pnpm pack` |
| CI / リリース | GitHub Actions（`changesets/action`） |
| npm への認証 | npm Trusted Publishing（OIDC） |

## リリースの流れ

プレ確認のために2ブランチ運用にしています。

```
feature ブランチ（変更 + changeset ファイル）
  │ PR
  ▼
develop
  │ リリース PR
  ▼
main ──push──▶ release.yml
                 ├─ 未リリースの changeset あり → 「Version Packages」PR を作成・更新
                 └─ その PR がマージされた     → タグ vX.Y.Z / GitHub Release / npm 公開
                                                 → main -> develop の戻し PR を作成
```

`develop` / `main` の 2 ブランチ運用でなければ、戻し PR の部分は不要です。

## 1. package.json を公開できる形にする

まずは「何を公開するか」を定義します。

```json
{
  "name": "your-package",
  "version": "0.0.0",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "git+https://github.com/owner/repo.git"
  },
  "type": "module",
  "sideEffects": ["**/*.css"],
  "files": ["dist"],
  "main": "./dist/index.cjs",
  "module": "./dist/index.mjs",
  "types": "./dist/index.d.cts",
  "exports": {
    ".": {
      "import": { "types": "./dist/index.d.mts", "default": "./dist/index.mjs" },
      "require": { "types": "./dist/index.d.cts", "default": "./dist/index.cjs" }
    },
    "./styles.css": "./dist/styles.css",
    "./package.json": "./package.json"
  }
}
```

ポイントは以下の通り。

- **`repository`**: Trusted Publishing で公開すると provenance（どのリポジトリのどのワークフローでビルドされたかの証明）が付きます。その照合に使われるので、実際のリポジトリの URL を正しく書いておきます。
- **`files`**: 公開に含めるものを allowlist で指定します。ビルド成果物の中に公開したくないもの（自分の場合は Storybook のビルド結果）があれば `"!dist/storybook"` のように除外します。
- **`exports`**: ESM と CJS の両方を出すなら、`import` / `require` それぞれに `types` を **先頭に** 書きます。CSS などを配る場合もここに書かないと、利用側から import できません。
- **`engines` は書かない**: 書くと公開されたパッケージにも残り、利用側のインストールに影響します。開発環境の Node.js のバージョンを固定したいだけなら、`.nvmrc` や `devEngines` を使います。

## 2. 公開前にパッケージの中身をチェックする

`package.json` の書き間違いは CI の中でチェックできるようにしています。

```json
{
  "scripts": {
    "build": "tsdown",
    "check:package": "publint --strict && attw --pack . --profile node16"
  }
}
```

- **publint**: `exports` や `files` の設定ミス、存在しないファイルの参照などを検出します。
- **attw**（`@arethetypeswrong/cli`）: ESM / CJS それぞれの解決方法で型が正しく見えるかを確認します。CSS のように型のないエントリポイントは `--exclude-entrypoints` で除外します。
- **`pnpm pack`**: 実際に公開される tarball を作ります。中身を一度目で見ておくと安心です。

ライブラリ固有のルール（自分の場合は「React をバンドルしていないこと」「すべての JS ファイルの先頭が `'use client'` であること」など）があれば、ビルド後の `dist` を検査する小さなスクリプトを足しておくと便利です。

## 3. Changesets を導入する

```sh
pnpm add -D @changesets/cli
pnpm changeset init
```

`.changeset/config.json` はこんな感じにしました。

```json
{
  "$schema": "https://unpkg.com/@changesets/config@4.0.1/schema.json",
  "baseBranch": "develop",
  "access": "public",
  "commit": false,
  "changelog": "@changesets/cli/changelog"
}
```

- **`baseBranch`**: feature ブランチを切る元のブランチです。2 ブランチ運用なら `develop` にします。
- **`access`**: スコープ付きの名前（`@acme/ui`）を公開パッケージにするなら `public` が必要です。

あとは、公開パッケージに影響する変更をした PR で `pnpm changeset` を実行し、major / minor / patch と変更内容を書いた `.changeset/*.md` を一緒にコミットするだけです。
バージョン番号と `CHANGELOG.md` は、後述のワークフローが changeset ファイルから自動で作ってくれます。

公開用のスクリプトも用意しておきます。公開の直前にもビルドとチェックをやり直すようにしています。

```json
{
  "scripts": {
    "release": "pnpm run build && pnpm run check:package && changeset publish"
  }
}
```

## 4. リリース用のワークフローを書く

`main` への push で動く `.github/workflows/release.yml` です（実際には action をコミット SHA で固定していますが、読みやすさのためにタグで書いています）。

```yaml
name: Release

on:
  push:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: false

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write # バージョン更新のコミット、タグ、GitHub Release
      pull-requests: write # Version Packages PR
      id-token: write # npm Trusted Publishing（OIDC）
    steps:
      - uses: actions/checkout@v7
        with:
          persist-credentials: false

      - uses: pnpm/action-setup@v6
      - uses: actions/setup-node@v7
        with:
          node-version-file: .nvmrc
          package-manager-cache: false # 公開に使うビルドにはキャッシュを使わない
      - run: pnpm install --frozen-lockfile

      - uses: changesets/action@v2
        with:
          # 公開を許可したリポジトリ以外では、公開スクリプトを空にする
          publish-script: ${{ github.repository == vars.NPM_PUBLISH_REPOSITORY && 'pnpm run release' || '' }}
          commit-message: 'chore: version packages'
          pr-title: 'chore: version packages'
```

ポイントは以下の通り。

- **`id-token: write`**: これがあるとジョブが OIDC トークンを取得でき、npm へのログインがトークンなしで済みます。`NPM_TOKEN` のようなシークレットは登録しません。
- **キャッシュを使わない**: 依存関係のキャッシュが汚染されていた場合に、それが公開物に混ざるのを避けるためです。npm のドキュメントの例でも、リリース用のビルドではキャッシュを無効にしています。
- **`NPM_PUBLISH_REPOSITORY` で公開するリポジトリを限定**: リポジトリ変数に `owner/repo` を入れておき、一致するときだけ公開スクリプトを渡しています。テンプレートからコピーされたリポジトリやフォークでは、Version Packages PR は作られても公開は行われません。

`changesets/action` は、未リリースの changeset があれば Version Packages PR を作り、なければ（＝その PR がマージされたあとなら）公開を実行します。公開に成功すると、`vX.Y.Z` タグと GitHub Release も作ってくれます。

なお、Trusted Publishing には npm CLI 11.5.1 以上と Node.js 22.14.0 以上が必要です。pnpm の `pnpm publish` も OIDC に対応していて、公開リポジトリから公開パッケージを出す場合は provenance も自動で付きます。

## 5. 最初のバージョンだけは手動で公開する

Trusted Publisher の設定は npmjs.com のパッケージの設定画面で行うので、**パッケージがまだ npm に存在しないと設定できません**。
つまり、最初の 1 回だけは手元から手動で公開する必要があります。

```sh
git switch main && git pull
git status                    # 未コミットの変更がないこと
npm login                     # pnpm は npm のログイン情報を使う
npm whoami

pnpm install --frozen-lockfile
pnpm run build
pnpm run check:package
pnpm pack                     # 任意: 公開される中身を確認
pnpm publish                  # スコープ付きの名前なら --access public
```

- npm アカウントの二要素認証を有効にしておきます。公開時にワンタイムパスワードを聞かれます。
- `pnpm publish` は、ブランチが `main` であることと作業ツリーがクリーンであることを確認してから公開します。
- 公開できたかは `npm view your-package version` で確認できます。

### 手動公開ではタグと GitHub Release が作られない

`changesets/action` がタグと Release を作るのは、自分が公開したバージョンだけです。手動で公開したバージョンにも付けたい場合は、同じ `main` のチェックアウトで手で作ります。

```sh
version="$(node -p 'require("./package.json").version')"
git tag "v$version"
git push origin "v$version"
gh release create "v$version" --title "v$version" \
  --notes-file <(awk -v v="## $version" '$0==v{f=1;next} /^## /{f=0} f' CHANGELOG.md)
```

`CHANGELOG.md` から該当バージョンの節だけを抜き出して、Release の本文にしています。

## 6. npmjs.com で Trusted Publisher を設定する

パッケージが npm に存在するようになったら、npmjs.com のパッケージの Settings にある Trusted Publisher から GitHub Actions を選んで設定します。

| 項目 | 値 |
|---|---|
| Organization or user | GitHub のユーザー名か Organization 名 |
| Repository | リポジトリ名 |
| Workflow filename | `release.yml`（パスではなくファイル名だけ） |
| Environment name | GitHub Environments を使っていなければ空欄 |
| Allowed actions | `npm publish` を許可する |

設定が終わったら、以下も済ませておきます。

- **Publishing access** を「Require two-factor authentication and disallow tokens」にする。これで、トークンを使った公開ができなくなります（Trusted Publishing は OIDC なので影響を受けません）。
- GitHub のリポジトリ変数 `NPM_PUBLISH_REPOSITORY` に `owner/repo` を設定する。

## 7. GitHub 側の設定

- **Settings > Actions > General で、GitHub Actions による PR の作成を許可する**: これがないと Version Packages PR が作られません。
- **既定の `GITHUB_TOKEN` で作った PR では CI が動かない**: Version Packages PR にも CI を必須にしているなら、GitHub App のトークンや fine-grained トークン（contents と pull-requests の write 権限）をシークレットに登録して、`changesets/action` の `github-token` に渡します。

```yaml
      - uses: changesets/action@v2
        with:
          github-token: ${{ secrets.RELEASE_TOKEN || github.token }}
```

`RELEASE_TOKEN` が未登録なら `GITHUB_TOKEN` にフォールバックするようにしています。

## 8. 2 ブランチ運用の場合: main を develop に戻す

`develop` → `main` の運用だと、Version Packages PR のバージョン更新と `CHANGELOG.md` は `main` にしか入りません。放っておくと `develop` と `main` がずれていくので、公開に成功したら `main` → `develop` の PR を自動で作るステップを入れています。

```yaml
      - uses: changesets/action@v2
        id: changesets
        # ...

      - name: Open the back-merge pull request to develop
        if: steps.changesets.outputs.published == 'true'
        env:
          GH_TOKEN: ${{ secrets.RELEASE_TOKEN || github.token }}
          GH_REPO: ${{ github.repository }}
          PUBLISHED: ${{ steps.changesets.outputs.published-packages }}
        run: |
          version="$(jq -r '.[0].version' <<<"$PUBLISHED")"
          open="$(gh pr list --base develop --head main --state open --json number --jq 'length')"
          if [[ "$open" != "0" ]]; then
            exit 0
          fi
          gh pr create --base develop --head main \
            --title "chore: merge main back into develop after v$version" \
            --body "Brings the version bump and CHANGELOG of v$version back to develop."
```

PR は squash ではなく merge commit でマージします。squash すると `main` のコミットが `develop` の履歴につながらず、2 つのブランチがずれたままになるためです。

## 注意点のまとめ

- 最初のバージョンだけは手動公開が必要（Trusted Publisher はパッケージが存在しないと設定できない）。
- 手動で公開したバージョンには、タグと GitHub Release が自動では付かない。
- Trusted Publisher の Allowed actions で `npm publish` を許可しないと、`changeset publish` が失敗する。
- npm の設定より先に Version Packages PR をマージすると、`package.json` と `CHANGELOG.md` のバージョンだけが進み、npm には何も公開されない。`changeset publish` は npm にまだないバージョンを公開するので、設定を終えたあとに公開すれば追いつけます。
- 一度公開したバージョン番号は、unpublish しても再利用できない。公開前のチェックを CI に入れておく。

## 実際に作ったもの

Storybook をマルチプロダクトで使うことがあると思うので、まとめられるようにしたいなあと思った時の構想に使いたいと考えて作ってみました。

[https://www.npmjs.com/package/su3-web-materials](https://www.npmjs.com/package/su3-web-materials)

## さいごに

はじめて npm パッケージ公開やりましたが、AI がなかったらこんなに早くできなかったと思う。。。
AI はあると色々大変な部分もあるのですが、個人的にこういった便利なもの（≒ゴミ）作りたいなと思った時には非常に便利だと改めて感じました。
