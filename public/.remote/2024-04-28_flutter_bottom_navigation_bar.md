---
title: Flutter の BottomNavigationBar で4つのアイテムを設定するとテキストが表示されない問題の解決方法
tags:
  - Android
  - iOS
  - Dart
  - Flutter
  - NavigationBar
private: false
updated_at: '2024-04-28T12:54:17+09:00'
id: 83264b3443a053ce56fb
organization_url_name: null
slide: false
ignorePublish: false
---
# なにこれ
Flutter で BottomNavigationBar を使ってメニューを作成しようと思ったらテキストが表示されなくて困ったので、その時の解消方法をメモ


# 事象
3つまでだとこのように良い感じに表示されるが
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/6a92eb77-dde2-29ad-04f8-fae74b6e81b6.png)

4つ以上になるとテキストが見えなくなった！
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/4d66dc22-557e-87a3-bbd9-922a527f0d45.png)


# 解決方法
`type: BottomNavigationBarType.fixed` で固定にしないといけないらしい。
```
bottomNavigationBar: BottomNavigationBar(
  type: BottomNavigationBarType.fixed, // このプロパティを追加する
  items: // ...,
)
```


# 原因
4つ以上にすると `BottomNavigationBarType.shifting` に自動的に切り替わってしまい、テキストが表示されないようになってしまうとのこと。
公式ページ：https://api.flutter.dev/flutter/material/BottomNavigationBar-class.html
```
The bottom navigation bar's type changes how its items are displayed. If not specified, then it's automatically set to BottomNavigationBarType.fixed when there are less than four items, and BottomNavigationBarType.shifting otherwise.
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/69d6ce11-bf6c-30ef-d334-1c3af571ee77.png)


# そして…
無事に直った！よかったよかった
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/2ad13873-bab2-fa40-d37e-84e25b055cdf.png)


