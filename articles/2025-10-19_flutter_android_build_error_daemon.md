---
title: "FlutterでAndroidビルドが '/opt/homebrew/share/flutter/bin/flutter' でエラーになった"
emoji: "📱"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Dart", "Android", "モバイル", "初心者"]
published: true
---
# これなに

- Flutter で v3.32.5 から v3.35.6 へアップグレード後にビルドエラーになった
- エラーログに具体的になにをするべきか書いてなかったので、エラー解決記録として本記事を書きました

# 発生したエラー

パッとみた感じでは「なんか Caskroom とやらが怪しい？」と思っていました

```dart
flutter_sample % flutter run --device-id emulator-5554 --debug
Launching lib/main.dart on sdk gphone64 arm64 in debug mode...

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':app:compileFlutterBuildDebug'.
> A problem occurred starting process 'command '/opt/homebrew/Caskroom/flutter/3.32.5/flutter/bin/flutter''

* Try:
> Run with --stacktrace option to get the stack trace.
> Run with --info or --debug option to get more log output.
> Run with --scan to get full insights.
> Get more help at https://help.gradle.org.

BUILD FAILED in 1s
Running Gradle task 'assembleDebug'...                           1,465ms
Error: Gradle task assembleDebug failed with exit code 1
flutter_sample %
```


# 原因判明までにやったこと

先ほどのエラーから「なんか flutter のバージョンをアップグレードしたから何かおかしくなったのかな？」と思って以下のことを実施しましたが、エラーは変わらず。

- `flutter clean` ==> `flutter pub get` ==> `flutter run` でクリーンアップしてからビルド
- `brew uninstall --cask flutter` ==> `brew install --cask flutter` で再インストール
- Copilot で訊いてみたけど「多分 flutter clean とか再インストールしたら治るんじゃない？」的な回答で解決には至らず


# 原因

- こんな時はインターネッツで調べてみようと思って、インターネッツの海を彷徨っていたら[こちらの回答](https://stackoverflow.com/questions/60386994/gradle-build-failing-after-update/64054311#64054311)に辿り着き、要するにDaemon が複数起動している可能性がありそうということになりました。
- 調べた結果、複数 Daemon が起動しているとキャッシュやリソースの競合が発生してしまうため、上記のようなエラーが発生してしまうようです。

# 解決方法

以下のコマンドで daemon を停止したのち、再ビルドすると解決しました

```zsh
cd android
./gradlew --stop
```

実際の daemon 停止ログは以下の通り。

```zsh
android % ./gradlew --stop
Stopping Daemon(s)
2 Daemons stopped
android %
```

停止後に確認したら以下の通り daemon は1つだけ起動している状態になっていました。

```zsh
sample_flutter % ./android/gradlew --status
   PID STATUS   INFO
 80259 IDLE     8.12
 95738 STOPPED  (stop command received)
 96814 STOPPED  (stop command received)

Only Daemons for the current Gradle version are displayed. For more on this, please refer to https://docs.gradle.org/8.12/userguide/gradle_daemon.html#sec:status in the Gradle documentation.
sample_flutter % 
```
