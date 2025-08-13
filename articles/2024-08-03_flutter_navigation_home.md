---
title: "Flutter で Android のナビゲーションホームボタンと同じようにホーム画面に戻る処理を実装する" # 記事のタイトル
emoji: "🏠" # Choose an appropriate emoji for each article
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Android", "Google", "iOS", "Dart", "Flutter"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
# なにこれ

- 戻るボタンを無効化したけど、戻るボタンで色々処理をさせたい時があった
- その時、前画面に戻るだけでなく、最上位の画面なら pop せずにアプリをバックグラウンドに行かせて、ホーム画面に行く処理を実装したいと思った
- 以前書いた戻るボタン無効化の記事は以下

https://qiita.com/su3-hokkaido/items/48f5d8bf8b5a487f691f


# 解決方法
### `move_to_background` パッケージをインストールする

パッケージ公式ガイドはこちら

https://pub.dev/packages/move_to_background

インストールコマンドはこちら
```dart
flutter pub add move_to_background
```

yaml に直接書く場合はこちら（バージョンは後述の公式ページにあるバージョンを参照してください）
```pubspec.yaml
dependencies:
  move_to_background: ^1.0.2
```

### onPopInvoked を実装

戻るボタンがタップされた時に onPopInvoked で判定して、`MoveToBackground.moveTaskToBack()` を実行させるとアプリをバックグラウンドに移動させてデバイスのホーム画面に戻ることができます。

```dart
import 'package:move_to_background/move_to_background.dart';
...
  PopScope(
    canPop: false,
    onPopInvoked: (bool didPop) async {
      if (!didPop) {
        MoveToBackground.moveTaskToBack();
      }
    },
```


# パッケージのリポジトリ

https://github.com/Sayegh7/move_to_background


# おわりに

2024年8月にこの記事書いているのですが、公式ページ見ると3年以上メンテナンスされていない様子…
いつか自作するしかないのかしら。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/75c66205-c54e-c44b-0807-e8fe3b369870.png)

