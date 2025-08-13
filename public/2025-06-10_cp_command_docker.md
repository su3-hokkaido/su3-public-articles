---
title: ローカルから docker コンテナにファイルをコピーするコマンド
tags:
  - Docker
  - DockerHub
  - コンテナ
  - 便利
  - オーケストレーション
private: false
updated_at: '2025-06-10T17:58:48+09:00'
id: 00a7175c0a272b8c2f32
organization_url_name: null
slide: false
ignorePublish: false
---
# これなに

- ローカル環境から docker コンテナ内にファイルをコピーするまでに必要なコマンドをまとめました
- 単なる基礎的な個人的なメモなのですが、興味がある人は読んでいただいても大丈夫です

# 基本コマンド

### コンテナIDまたは名前を確認する

```zsh
$ docker ps
CONTAINER ID   IMAGE         ...   NAMES
container-id   your-image    ...   your-container
```

### `docker cp` でファイルコピー

ダブルクオーテーションは便宜的に書いているだけなので、実際のコマンドには必要ありません

```zsh
$ docker cp "ファイルパス" "image-id":"コンテナ内のコピー先ディレクトリパス"
```

# 複数コンテナに一気にコピーさせたい時のコマンド

### コンテナ ID がすでに明確になっている時

```zsh
for cid in <container_id1> <container_id2> <container_id3>; do
  docker cp "ファイルパス" "image-id":"コンテナ内のコピー先ディレクトリパス"
  docker cp "ファイルパス" "image-id":"コンテナ内のコピー先ディレクトリパス"
done
```

### キーワードで引っ掛けてコピーする時

```zsh
for pod in $(kubectl get pods -n <namespace> -o jsonpath='{.items[*].metadata.name}' | tr ' ' '\n' | grep your-keyword); do
  kubectl cp "ファイルパス" "image-id":"コンテナ内のコピー先ディレクトリパス"
  kubectl cp "ファイルパス" "image-id":"コンテナ内のコピー先ディレクトリパス"
done
```

# さいごに

ひさしぶりに Qiita に記事投稿しました…

最近コーディングよりアーキテクト的な動きとかマネジメント寄りの動きばっかりしてたので社外向けにさっと書けるネタが無かったのですが、アプリ実装設計で色々やってた時に「あれ、これなんだっけ」ってなったのでメモついでに記事として書きました

今後またコンスタントに記事投稿できたらなあ、と思いながらまた次回は数ヶ月後とかになりそうな雰囲気（ぐぬぬ）
