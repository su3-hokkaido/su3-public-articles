---
title: "Flutter でデバイスOSの判定をする方法" # 記事のタイトル
emoji: "📱" # Choose an appropriate emoji for each article
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Android", "iOS", "Dart", "MobileApp", "Flutter"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
# これなに

OSごとに処理を分けたいときにどう実装するのかをメモしました


# 解決方法

`dart.io` をインポートして、OS の判定を行います
以下の例ですと、iOS, iPadOS, macOS のいずれかの時に Text を描画し、そうでない OS の場合は空の Container を描画する（つまり何も描画しない）ように処理しています

```dart
import 'dart:io' show Platform;
...

(Platform.isIOS || Platform.isMacOS) ? 
  Text(
    'iOS, iPadOS, macOS の時のみこのテキストを表示',
    textAlign: TextAlign.center,
    style: const TextStyle(color: Colors.black87, fontSize: 24.0),
  ) : Container(),
...
```


# 定義されているメソッド

以下のようにOSの名前でメソッドが書かれているのでわかりやすい
もはや説明不要ですね

- isAndroid
- isFuchsia
- isIOS
- isLinux
- isMacOS
- isWindows

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/2239bad2-aff6-b09e-65f6-8f74e72a2c9b.png)


# あとがき

iOS のみで Apple ID 認証を実装したくてこの判定処理を使いました
他に良い方法があったらぜひ教えてください！
