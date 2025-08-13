---
title: "Your project requires a newer version of the Kotlin Gradle plugin." # 記事のタイトル
emoji: "🔧" # Choose an appropriate emoji for each article
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Android", "Kotlin", "gradle", "Flutter", "FlutterFire"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
# これなに
Flutter で Android をビルドした時に、 `Your project requires a newer version of the Kotlin Gradle plugin.` と言われて、最新の Kotlin Gradle plugin が必要だと怒られた時の対処内容のまとめ。
```bash
┌─ Flutter Fix ──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ [!] Your project requires a newer version of the Kotlin Gradle plugin.                                                                                 │
│ Find the latest version on https://kotlinlang.org/docs/releases.html#release-details, then update ~/android/build.gradle:                              │
│ ext.kotlin_version = '<latest-version>'                                                                                                                │
└────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

# 遭遇した時の環境条件
いつからのタイミングかわからないけど、以下の条件の時この事象に遭遇。
- Chip: M2
- macOS: Sonoma 14.4.1
- Flutter: v3.19.4
- Dart: v3.3.2 
```bash
% flutter --version
Flutter 3.19.4 • channel stable • https://github.com/flutter/flutter.git
Framework • revision 68bfaea224 (2 weeks ago) • 2024-03-20 15:36:31 -0700
Engine • revision a5c24f538d
Tools • Dart 3.3.2 • DevTools 2.31.1
```


# いつ発生したか
`firebase_auth` パッケージを使って Firebase でのログイン認証機能を実装してビルドした時


# 原因
要するに `Module was compiled with an incompatible version of Kotlin. The binary version of its metadata is 1.9.0, expected version is 1.7.1.` がポイントの模様。
```bash
% flutter run --device-id emulator-5554
Launching lib/main.dart on sdk gphone64 arm64 in debug mode...
Building with Flutter multidex support enabled.

FAILURE: Build failed with an exception.

* Where:
Build file '~/android/app/build.gradle' line: 43

* What went wrong:
A problem occurred evaluating project ':app'.
> Could not find method kotlinOptions() for arguments [build_xxx$_run_closure2$_closure6@xxx] on extension 'android' of type com.android.build.gradle.internal.dsl.BaseAppModuleExtension.

* Try:
> Run with --stacktrace option to get the stack trace.
> Run with --info or --debug option to get more log output.
> Run with --scan to get full insights.

* Get more help at https://help.gradle.org

BUILD FAILED in 1s
Running Gradle task 'assembleDebug'...                           1,523ms
Error: Gradle task assembleDebug failed with exit code 1
% flutter run --device-id emulator-5554
Launching lib/main.dart on sdk gphone64 arm64 in debug mode...
Building with Flutter multidex support enabled.
e: ~/.gradle/caches/transforms-3/xxx/transformed/jetified-play-services-measurement-api-21.6.1-api.jar!/META-INF/java.com.google.android.gmscore.integ.client.measurement_api_measurement_api.kotlin_module: Module was compiled with an incompatible version of Kotlin. The binary version of its metadata is 1.9.0, expected version is 1.7.1.

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':app:compileDebugKotlin'.
> A failure occurred while executing org.jetbrains.kotlin.compilerRunner.GradleCompilerRunnerWithWorkers$GradleKotlinCompilerWorkAction
   > Compilation error. See log for more details

* Try:
> Run with --stacktrace option to get the stack trace.
> Run with --info or --debug option to get more log output.
> Run with --scan to get full insights.

* Get more help at https://help.gradle.org

BUILD FAILED in 2s
Running Gradle task 'assembleDebug'...                           2,608ms

┌─ Flutter Fix ──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ [!] Your project requires a newer version of the Kotlin Gradle plugin.                                                                                 │
│ Find the latest version on https://kotlinlang.org/docs/releases.html#release-details, then update ~/android/build.gradle:                              │
│ ext.kotlin_version = '<latest-version>'                                                                                                                │
└────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
Error: Gradle task assembleDebug failed with exit code 1
```

# 修正方法
エラーの指示通り、Kotlin のバージョンを 1.7.10 から 1.9.0 に変更。（エラーでは 1.7.1 って言ってたけどヘーキヘーキ）

## Before | 修正前
```gradle
plugins {
    id "dev.flutter.flutter-plugin-loader" version "1.0.0"
    id "com.android.application" version "7.3.0" apply false
    id "org.jetbrains.kotlin.android" version "1.7.10" apply false

    // ↓ Firebase 認証を使うために追加したもの
    id 'com.google.gms.google-services' version '4.4.1' apply false
}
```

## After | 修正後
```gradle
plugins {
    id "dev.flutter.flutter-plugin-loader" version "1.0.0"
    id "com.android.application" version "7.3.0" apply false
    id "org.jetbrains.kotlin.android" version "1.9.0" apply false

    // ↓ Firebase 認証を使うために追加したもの
    id 'com.google.gms.google-services' version '4.4.1' apply false
}
```

## エラー解消した！
よかったよかった
```bash
% flutter run --device-id emulator-5554
Launching lib/main.dart on sdk gphone64 arm64 in debug mode...
Building with Flutter multidex support enabled.
Note: Some input files use or override a deprecated API.
Note: Recompile with -Xlint:deprecation for details.
Running Gradle task 'assembleDebug'...                             50.7s
✓  Built build/app/outputs/flutter-apk/app-debug.apk.
Installing build/app/outputs/flutter-apk/app-debug.apk...           5.7s
Syncing files to device sdk gphone64 arm64...                    2,007ms

Flutter run key commands.
r Hot reload. 🔥🔥🔥
R Hot restart.
h List all available interactive commands.
d Detach (terminate "flutter run" but leave application running).
c Clear the screen
q Quit (terminate the application on the device).

A Dart VM Service on sdk gphone64 arm64 is available at: http://xxx
The Flutter DevTools debugger and profiler on sdk gphone64 arm64 is available at: http://xxx?uri=http://xxx
```
