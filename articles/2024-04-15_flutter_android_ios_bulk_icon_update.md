---
title: "Flutter で Android や iOS のアプリアイコンを一括で変更したい！" # 記事のタイトル
emoji: "📱" # Choose an appropriate emoji for each article
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Android", "Google", "iOS", "Dart", "Flutter"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
# これなに
クロスプラットフォームなんだから Manifest だったり Assets.xcassets だったりと個別に設定やらないでアプリアイコンを一発で設定くらいできるでしょう。
そんな思いから何かないかなあと思って調べてみました。


# 設定手順
### 1. `flutter_launcher_icons` package を pubspec.yaml に追加
以下をターミナルで実行して追加する
```zsh
$ flutter pub add flutter_launcher_icons
```

### 2. 任意の場所にアイコンファイルを追加
基本的にはプロジェクト配下に `assets` フォルダを作成して、その中で設定しておくのが良さそう
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/342bda88-72b5-5a7a-27e8-3adf3bd45cce.png)

### 3. `pubspec.yaml` でアイコンファイルのパスを通す
どこに書いても大丈夫だけど、僕は `flutter:` セクションの直前くらいに書いておいた
```yaml
flutter_launcher_icons:
  android: true
  ios: true
  remove_alpha_ios: true
  image_path: "assets/icons/application_icon.png"
```


### 4. アイコン適用コマンドをターミナルで実行
これを実行したのち、ビルドしてみるとアイコンが適用できていました。
```zsh
$ flutter pub run flutter_launcher_icons
```


# おわりに
上記の手順については全てこの公式ページに書いてありました、素晴らしい！
https://pub.dev/packages/flutter_launcher_icons
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/8c19ead0-e1c5-f025-5bda-796704fdb444.png)
