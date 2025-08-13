---
title: "Android アプリで Firebase ネットワークタイムアウトエラー" # 記事のタイトル
emoji: "🔥" # Choose an appropriate emoji for each article
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Android", "Kotlin", "Dart", "Firebase", "Flutter"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
# これなに
Flutter でアプリ実装したけど、Android でネットワークタイムアウトエラーになってしまう事象に遭遇したので調べた
```zsh
E/RecaptchaCallWrapper(25265): Initial task failed for action RecaptchaAction(action=signUpPassword)with exception - A network error (such as timeout, interrupted connection or unreachable host) has occurred.
```

# エラー内容全文
なんかよくわからないけどネットワークがタイムアウトしたらしい。
```zsh
I/FirebaseAuth(25265): Creating user with hoge@hoge.com with empty reCAPTCHA token
W/System  (25265): Ignoring header X-Firebase-Locale because its value was null.
D/EGL_emulation(25265): app_time_stats: avg=25.98ms min=9.95ms max=389.89ms count=39
E/RecaptchaCallWrapper(25265): Initial task failed for action RecaptchaAction(action=signUpPassword)with exception - A network error (such as timeout, interrupted connection or unreachable host) has occurred.
I/flutter (25265): *** registerUserInFirebase > Result: [firebase_auth/network-request-failed] A network error (such as timeout, interrupted connection or unreachable host) has occurred.
```

# 解決方法
`android/app/src/main/AndroidManifest.xml` で以下のようにインターネット接続を許可する。
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- ↓ Allow to exchange via the internet -->
    <uses-permission android:name="android.permission.INTERNET" />
    <!-- ↑ Allow to exchange via the internet -->
```

# 別の理由もある
Android エミュレーターとかデバイスのネットワーク接続状態も確認しましょう。
実はそっちが繋がってなかったら通信できなくて当たり前なので。
