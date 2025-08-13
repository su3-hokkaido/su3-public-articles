---
title: Flutter で Android と iOS のスワイプして戻る画面遷移機能を無効化する
tags:
  - Android
  - Google
  - iOS
  - Dart
  - Flutter
private: false
updated_at: '2024-08-03T22:08:19+09:00'
id: 48f5d8bf8b5a487f691f
organization_url_name: null
slide: false
ignorePublish: false
---
# なにこれ
この戻る機能を無くしたいので、それを実装した時のメモを書く
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/82a1a37c-2c3f-7b0d-1883-297374b3cf65.png)

# 解決方法
`PopScope` を利用して `canPop: false` を実装するだけで大丈夫でした。

### Before
```dart
class _HomeViewState extends State<HomeView> {
  int selectedIndex;

  _HomeViewState({Key? key, required this.selectedIndex});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      // コードが続く
```

### After
```dart
class _HomeViewState extends State<HomeView> {
  int selectedIndex;

  _HomeViewState({Key? key, required this.selectedIndex});

  @override
  Widget build(BuildContext context) {
    return PopScope( // ここを追加
      canPop: false, // false で無効化
      child: Scaffold( // Scaffold は child に入れる
        // コードが続く
```

# おわりに
この実装方法については公式サイトで紹介されていました。
デフォルトでは canPop は true ですよということでした。
https://api.flutter.dev/flutter/widgets/PopScope-class.html
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/94af2d76-850f-b53b-05a7-74822f93588e.png)


# 追記
### 2024/08/03

Android の戻るボタンによってアプリをバックグラウンドへ移動させて、ホームに戻る処理を実装したくなったので以下をまとめました。

https://qiita.com/su3-hokkaido/items/d9454744832450cb6629
