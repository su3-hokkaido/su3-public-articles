---
title: >-
  Android アプリで Firebase ネットワークタイムアウトエラー：A network error (such as timeout,
  interrupted connection or unreachable host) has occurred. 
tags:
  - Android
  - Kotlin
  - Dart
  - Firebase
  - Flutter
private: false
updated_at: '2024-04-20T16:12:14+09:00'
id: bf084e749027682dd368
organization_url_name: null
slide: false
ignorePublish: false
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
