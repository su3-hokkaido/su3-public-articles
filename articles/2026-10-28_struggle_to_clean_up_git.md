---
title: "【週末お掃除】git に溜まった不要な操作履歴やデータを削除してストレージ容量を節約しようと試行錯誤した記録"
emoji: "🧹"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["git","github","初心者","macOS","掃除"]
published: true
---
## これなに

昨今 AI によって git 管理の在り方自体見直され始めていますが、まだまだみなさん使っているであろう git について。
`.git` には過去の履歴や不要データがたくさん蓄積していき、気付けばかなり肥大化していてストレージを圧迫していることがあります。

自分のマシンは以下のような感じでかなり肥大化していてしんどいのでお掃除しようと思ったので、その作業記録を個人のメモとしてまとめました。
個人用のメモなのでいろいろ不足していたりすることもありますが、必要な人は参考にしていただけたらと思います。

## 対象読者

`.git` によってストレージが食われすぎていてしんどい自分のようなストレージにお金かけられない人たち

## 実行環境

macOS 27 Golden Gate 27.0.1

## ⚠️ 注意

- 後述の手順を実行すると、完全に「元に戻せない（Undoできない）」状態になります。
- 本来は間違えてブランチを消しても reflog から復元できますが、後述するコマンド群を実行した後はリポジトリが極限まで軽量化される代わりに、過去のミスを救済するデータもすべて消去されます。大容量ファイルの削除作業の「最後の仕上げ」としてのみ実行してください。
- 作業を実行する前に、現在のリポジトリのバックアップをとるなどして安全策をとるようにしてください。

## 先にオチだけ

結論だけ先に書くと、最終的に新しいリポジトリ爆誕させてそこにファイルだけ引っ越しました🫠（なんだそりゃ）
とはいうものの、実際何をやったのかをメモしましたので笑いながら読んでください。

## 初期調査

なんかよくわからないけど GitHub のリポジトリを入れているところがかなり肥大化している模様でした。
タイトルですでに `git` が問題だということはすでにわかってはいますが、そこに行き着くまでの調査記録を念のため書きます。

![Storage analysis](/images/2026-10-28_0_large_files.png)

### ディレクトリのサイズ確認

Applications, Developer などが肥大化するのはまあわかるのですが、 Documents がかなり食っていたのは気になったので調べてみると、 GitHub のディレクトリがかなり食っていることが判明。

![Found large files](/images/2026-10-28_1_large_files.png)

du コマンドで調べてみたところこんな感じでした。

```zsh
% du -sh ~/GitHub/*
1.4G	/Users/su3-hokkaido/GitHub/su3-app01
 20M	/Users/su3-hokkaido/GitHub/su3-app02
5.1G	/Users/su3-hokkaido/GitHub/su3-app03
1.4G	/Users/su3-hokkaido/GitHub/su3-app04
4.0K	/Users/su3-hokkaido/GitHub/su3-app05
4.0K	/Users/su3-hokkaido/GitHub/su3-app06
4.0K	/Users/su3-hokkaido/GitHub/su3-app07
1.1G	/Users/su3-hokkaido/GitHub/su3-app08
4.0K	/Users/su3-hokkaido/GitHub/su3-app09
4.0K	/Users/su3-hokkaido/GitHub/su3-app10
7.1G	/Users/su3-hokkaido/GitHub/su3-app11
207M	/Users/su3-hokkaido/GitHub/su3-app12
% 
```

ここからさらに細かい原因を調べるために、特に肥大化が激しい app11 のディレクトリを調べてみることに。

### `.git` 肥大化が判明

さらに分解して調べてみると一つ一つは全然重くなさそう。

```zsh
% du -sh ~/GitHub/su3-app11/*
1.2M	/Users/su3-hokkaido/GitHub/su3-app11/hogehoge01
804M	/Users/su3-hokkaido/GitHub/su3-app11/hogehoge02
4.0K	/Users/su3-hokkaido/GitHub/su3-app11/hogehoge03
 10M	/Users/su3-hokkaido/GitHub/su3-app11/hogehoge04
 76K	/Users/su3-hokkaido/GitHub/su3-app11/hogehoge05
276K	/Users/su3-hokkaido/GitHub/su3-app11/hogehoge06
3.4M	/Users/su3-hokkaido/GitHub/su3-app11/hogehoge07
 28K	/Users/su3-hokkaido/GitHub/su3-app11/hogehoge08
 18M	/Users/su3-hokkaido/GitHub/su3-app11/hogehoge09
148K	/Users/su3-hokkaido/GitHub/su3-app11/hogehoge10
216K	/Users/su3-hokkaido/GitHub/su3-app11/hogehoge11
 60M	/Users/su3-hokkaido/GitHub/su3-app11/hogehoge12
 12K	/Users/su3-hokkaido/GitHub/su3-app11/hogehoge13
1.1M	/Users/su3-hokkaido/GitHub/su3-app11/hogehoge14
% 
```

もしかして？と思って `.git` を調べたらビンゴ。6GBも食っていました。

```zsh
% du -sh .git/objects
6.3G    .git/objects
%
```

## お掃除作業手順（基本編）

ここからは一番大きく、かつ別に履歴は必要としていないリポジトリ（約7GB食っていたリポジトリ）に対して作業していこうと思います。

### セットアップ

`git-filter-repo` をインストールします。（インストール手順はこちら： https://github.com/newren/git-filter-repo/blob/main/INSTALL.md ）
自分は homebrew 経由でインストールしました。

```zsh
% brew install git-filter-repo
==> Auto-updating Homebrew...
Adjust how often this is run with `$HOMEBREW_AUTO_UPDATE_SECS` or disable with
`$HOMEBREW_NO_AUTO_UPDATE=1`. Hide these hints with `$HOMEBREW_NO_ENV_HINTS=1` (see `man brew`).
==> Downloading https://ghcr.io/v2/homebrew/core/portable-ruby/blobs/sha256:43ff285a82f48c84a06d0676a3800815a19d290d32e44182ddd7a6e95478e18e
######################################################################### 100.0%
==> Pouring portable-ruby-4.0.9.arm64_big_sur.bottle.tar.gz
==> Downloading bottle manifests
✔︎ Bottle Manifest git-filter-repo (2.47.0)          Downloaded    7.3KB/  7.3KB
==> Fetching downloads for: git-filter-repo
✔︎ Bottle git-filter-repo (2.47.0)                   Downloaded  101.9KB/101.9KB
==> Pouring git-filter-repo--2.47.0.all.bottle.1.tar.gz
🍺  /opt/homebrew/Cellar/git-filter-repo/2.47.0: 9 files, 361.5KB
% 
```

### 分析レポート作成

対象リポジトリのディレクトリで `git filter-repo --analyze` を実行して、お掃除前にどんなものが入っているか確認します。
実際の実行ログは以下のような形です。

```zsh
% git filter-repo --analyze
Processed 4009 blob sizes
Processed 382 commits
Writing reports to .git/filter-repo/analysis...done.
% 
```

`.git/filter-repo/analysis` 配下に生成された各ファイルの詳細は省略しますが、ざっとのイメージは以下の通り。
詳しくは大元のリポジトリ [git-filter-repo](https://github.com/newren/git-filter-repo) を読んだ方が確実かと思います。

- `path-all-sizes.txt` : 全部のファイル
- `path-deleted-sizes.txt` : 過去に削除されたファイル
- `directories-all-sizes.txt` : 全部のディレクトリ
- `directories-deleted-sizes.txt` : 過去に削除されたディレクトリ

### 不要オブジェクトを掃除する

実行コマンドは以下の2つ。

**1: 過去の操作履歴（リブログ）を今すぐ期限切れにする**

```zsh
git reflog expire --expire=now --all
```
- Gitは、コミットの切り替え（checkout）や打ち消し（reset）、ブランチの削除など、あなたがリポジトリ内で行った「すべての操作履歴」を裏で記録しています。これを reflog と呼びます。
- `--expire=now`: 本来なら一定期間（デフォルトで30〜90日間）保管される操作履歴を、「今すぐすべて期限切れ」にします。
- `--all`: すべてのブランチや参照の操作履歴を対象にします。

**2: 不要になったデータ実体を完全に削除し、残ったデータを強力に圧縮する**

```zsh
git gc --prune=now --aggressive
```

- gc（ガベージコレクション）とは Git における「ゴミ集め・掃除」コマンドです。どのコミットやブランチ、操作履歴からもリンクされなくなった孤立したデータを検知して削除します。
- `--prune=now`: 通常、直近2週間に作られたゴミデータは誤操作防止のために残されますが、それを無視して「今あるゴミを今すぐすべて完全削除」します。
- `--aggressive`: 通常の最適化よりも、より時間をかけて強力にファイルを圧縮します。共通するコードやファイル差分を綺麗にまとめて、 `.git` フォルダの容量を限界まで削ります。

実際の実行ログは以下の通り。
時間の計測はしていないのですが、たぶん実行時間で言うと60分くらいかかったかも。

**実行結果**

```zsh
% git reflog expire --expire=now --all
% git gc --prune=now --aggressive
Enumerating objects: 5420, done.
Counting objects: 100% (5420/5420), done.
Delta compression using up to 10 threads
Compressing objects: 100% (5273/5273), done.
Writing objects: 100% (5420/5420), done.
Total 5420 (delta 2924), reused 1282 (delta 0), pack-reused 0 (from 0)
% 
```

## お掃除作業手順（発展編）

実行したけれども、100MB だけしか減っていなかった。。。

```zsh
% du -sh .git/objects
6.2G	.git/objects
%
```

### 追加調査

`git fsck --unreachable --dangling` を実行して本当に孤立したゴミ（Dangling object）があるかを改めて確認したところ以下のようになった。
`.DS_Store` の話しか出てこなくて、消せそうなものが全く無し。

```zsh
# コマンド: git fsck --unreachable --dangling
% git fsck --unreachable --dangling
error: refs/.DS_Store: badRefName: invalid refname format
error: refs/.DS_Store: badRefContent: 
Checking ref database: 100% (1/1), done.
Checking object directories: 100% (256/256), done.
Checking objects: 100% (5420/5420), done.
Verifying commits in commit graph: 100% (382/382), done.
% 
```

そこでさらに容量を食っている上位10ファイル（コミット）を特定してみます。するといろいろ素材で使っていた動画や画像のコミットが！（これだ！）

```zsh
# コマンド: git rev-list --objects --all | grep "$(git verify-pack -v .git/objects/pack/*.pack | sort -k 3 -n | tail -10 | awk '{print $1}')"
% git rev-list --objects --all | grep "$(git verify-pack -v .git/objects/pack/*.pack | sort -k 3 -n | tail -10 | awk '{print $1}')"

b3d8ee51c7b74ae photos/canon/canon_eos200/IMG_5239.JPG
b41cfacbd8297d5 photos/canon/canon_eos200/MVI_5303.MP4
e875256d0382801 photos/canon/canon_eos200/IMG_5322.JPG
e704d9a01e0a9e2 photos/canon/canon_eos200/IMG_5323.JPG
320f19f210690ae photos/canon/canon_eos200/IMG_5324.JPG
1bb6700c6973c3a photos/canon/canon_eos200/IMG_5325.JPG
d94566c2ad22e3f photos/canon/canon_eos200/IMG_5326.JPG
fc3ab40e7884c53 photos/canon/canon_eos200/IMG_5327.JPG
38e68342661e8ce photos/canon/canon_eos200/MVI_5443.MP4
3ba184a31a09506 photos/canon/canon_eos200/MVI_5526.MP4
% 
```

### そんなわけで画像、動画を削除しにいく

`--path` で指定したフォルダを、歴史上から完全に除外（ `--invert-paths` ）してすべてのコミット履歴を自動で書き換えます。
冒頭でインストールした `filter-repo` を使って削除しに行きます。

```zsh
git filter-repo --path photos/canon/canon_eos200/ --invert-paths
```

実行したけど破壊的な上書きだからフレッシュクローンしろと怒られました。

```zsh
% git filter-repo --path photos/canon/canon_eos200/ --invert-paths
Aborting: Refusing to destructively overwrite repo history since
this does not look like a fresh clone.
  (expected at most one entry in the reflog for HEAD)
Please operate on a fresh clone instead.  If you want to proceed
anyway, use --force.
%
```

しかし、ローカルでもう必要なものはバックアップとっているので、気にせず `--force` で強制実行。

```zsh
% git filter-repo --path photos/canon/canon_eos200/ --invert-paths --force
NOTICE: Removing 'origin' remote; see 'Why is my origin removed?'
        in the manual if you want to push back there.
        (was git@github:su3-hokkaido/su3-app11.git)
Parsed 191 commits
New history written in 0.21 seconds; now repacking/cleaning...
Repacking your repo and cleaning out old unneeded objects
HEAD is now at abc1234 hogehoge
Enumerating objects: 4728, done.
Counting objects: 100% (4728/4728), done.
Delta compression using up to 10 threads
Compressing objects: 100% (1885/1885), done.
Writing objects: 100% (4728/4728), done.
Total 4728 (delta 2707), reused 4722 (delta 2705), pack-reused 0 (from 0)
Completely finished after 7.05 seconds.
% 
```

これだけでもう 4GB くらい消えました。

```zsh
# Before
% du -sh .git/objects
6.3G	.git/objects
%

# After
% du -sh .git/objects
2.4G	.git/objects
%
```

最後にもう一度基本編で紹介したコマンドを実行して孤立データを削除しに行きます。

```zsh
% git reflog expire --expire=now --all
% git gc --prune=now --aggressive
Enumerating objects: 4728, done.
Counting objects: 100% (4728/4728), done.
Delta compression using up to 10 threads
Compressing objects: 100% (4590/4590), done.
Writing objects: 100% (4728/4728), done.
Total 4728 (delta 2700), reused 2008 (delta 0), pack-reused 0 (from 0)
% 
```

が、あまり変わらず。

```zsh
% du -sh .git/objects
2.4G	.git/objects
%
```

git に再度 push しようにもデータ量が多すぎて、サーバーが耐えきれずに途中で 500 エラーになってしまいました。

```zsh
% git remote add origin git@github.com:su3-hokkaido/su3-app11.git # filter-repo は origin を削除するので、リモートを再登録する必要あり
% git push -f origin main
Enumerating objects: 4728, done.
Counting objects: 100% (4728/4728), done.
Delta compression using up to 10 threads
Compressing objects: 100% (1890/1890), done.
error: RPC failed; HTTP 500 curl 22 The requested URL returned error: 500
send-pack: unexpected disconnect while reading sideband packet
Writing objects: 100% (4728/4728), 2.43 GiB | 21.06 MiB/s, done.
Total 4728 (delta 2700), reused 4728 (delta 2700), pack-reused 0 (from 0)
fatal: the remote end hung up unexpectedly
Everything up-to-date
% 
```

ここから過去のコミットごとに push し直していくという作業をすれば解決する目処はついたものの、そんな気力はなく諦めました。
（スクリプト作って頑張ってもいいけど、労力的に無駄になりそう）

## 最終奥義

要件的には過去の reflog は全く必要なかったので「もういいや！」となって新しいリポジトリを爆誕させて必要なファイルだけそちらにお引越し。
最初からこっちでよかったね！ﾊﾊﾊﾊ!!

↓ お掃除前

![Before](/images/2026-10-28_2_before.png)

↓ お掃除後

![After](/images/2026-10-28_3_after.png)
