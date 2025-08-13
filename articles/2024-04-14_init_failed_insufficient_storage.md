---
title: "Android エミュで Failure [INSTALL_FAILED_INSUFFICIENT_STORAGE] エラー" # 記事のタイトル
emoji: "📱" # Choose an appropriate emoji for each article
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Android", "Kotlin", "AndroidStudio", "emulator", "Flutter"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
# これなに
Android エミュレーターでビルドしたら `Failure [INSTALL_FAILED_INSUFFICIENT_STORAGE: Failed to override installation location]` なるエラーが出た。


# エラーログ
ターミナルで出てきたエラーログは以下の通り。
```zsh 
% flutter run --device-id emulator-5554
Launching lib/main.dart on sdk gphone64 arm64 in debug mode...
Running Gradle task 'assembleDebug'...                            105.4s
✓  Built build/app/outputs/flutter-apk/app-debug.apk.
Installing build/app/outputs/flutter-apk/app-debug.apk...           9.6s
Error: ADB exited with exit code 1
Performing Streamed Install

adb: failed to install /Users/user/Github-su3/myapp/build/app/outputs/flutter-apk/app-debug.apk: Failure [INSTALL_FAILED_INSUFFICIENT_STORAGE: Failed to override installation location]
Error launching application on sdk gphone64 arm64.
```


# 解決方法
エミュレーターを修了した上で、Android Studio の Device Manager から Wipe Data を実行すると解決した。（スクショは起動中になってますが、気にせず）
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/4a020068-2b1a-418e-f010-1d29e41a45cc.png)


# 原因
ストレージスペースがなくなったことにある。
元々エミュレーターのストレージ量は極少なので、必要があれば数字をいじれば解決する。
ただ、多様なユーザーがいると思うので低スペックデバイスがどう振る舞うかを確認できるのであんまりイジらない方が良いと考えています。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/08dff36c-49ef-09df-c735-e529f2f0c813.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/4f274df5-8494-6fdf-027a-2d453b510647.png)

