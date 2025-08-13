---
title: >-
  Xcode で Distribute App を実行しようとしたときに Communication with Apple Failed または No
  profiles for 'com.abc' were found となった時のエラー解消メモ
tags:
  - Xcode
  - iOS
  - Swift
  - Flutter
  - AppStoreConnect
private: false
updated_at: '2024-11-30T18:06:59+09:00'
id: 6be4958e44ffb4b4fdb1
organization_url_name: null
slide: false
ignorePublish: false
---
# これなに

- Xcode から Distribute App を実行しようと思ったら以下のエラーが出た
  - `Communication with Apple Failed`
  - `No profiles for 'com.abc' were found`
- そのエラーをどのようにして解決したのかをこの記事でメモ
- そのため、固有のケースによってはこれが当てはまらないケースもあることをご了承ください

![Qiita - Fix xcode provisioning error.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/a70d7029-358e-367a-fcf9-ae3f50d99d31.png)


# 原因

Profiles が無かった

# 解消手順

## Step 1: Profiles を作成

[こちら](https://developer.apple.com/account/resources/profiles/list)から Profiles を作成してください
リンクに入ると以下のスクリーンショットのような画面になるかと思いますので、＋ボタンから手続きを行なってください

https://developer.apple.com/account/resources/profiles/list

![Screenshot at Nov 30 15-30-42.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/4eb0cfde-6cb8-6e64-1bd8-be4aca48c5f4.png)


## Step 2: Profiles をダウンロードして、指定の箇所に配置

Profiles の作成が完了したら、Profiles をダウンロードして `~/Library/MobileDevice/Provisioning Profiles`のディレクトリに配置してください

#### ↓ダウンロードはこちらから（リストページからのダウンロード可能なので、お好きなように）

![Screenshot at Nov 30 15-33-22.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/77868059-0fda-9623-8019-baec4e7d2640.png)

#### ↓実際の配置箇所の確認は以下の通り行いました

```zsh
% pwd
/Users/su3-hokkaido/Library/MobileDevice/Provisioning Profiles
$ ls
Su3_Provisioning_Profile_20241130.mobileprovision
$
```

## Step 3: Try again で Signing 再実行

Try again ボタンをクリックしてあげると無事に成功しました

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/d575625e-38ec-1f45-945e-b44c286156d0.png)


# 補足

おそらく同様の事象に発生した場合は Profiles を再作成するなどによってエラー解消しそうな気がするので、同じような事象に出会った方は試してみる価値があるかも

また、中間ファイルやキャッシュがエラーになっているかもしれないので、DerivedData や Caches を一度削除してから再実行してみるのも良いかもしれません

中間ファイルやキャッシュの削除手順は以下の記事の補足セクションの `試したけどダメだったもの` で掲載しています

https://qiita.com/su3-hokkaido/items/dd4e44a4680140e6d49b#%E8%A9%A6%E3%81%97%E3%81%9F%E3%81%91%E3%81%A9%E3%83%80%E3%83%A1%E3%81%A0%E3%81%A3%E3%81%9F%E3%82%82%E3%81%AE


# あとがき

以下の StackOverflow の質問で同様の手順を踏んでいる人がいたのですが、その人はこの手順では解決できなかった模様
上記内容で解決できないようであれば Copilot とか StackOverflow で解決方法を探しても良いかもです

https://stackoverflow.com/questions/69362808/xcode-couldnt-find-any-ios-app-store-provisioning-profiles-matching-bundle-id

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/da28755c-e1b3-04de-e935-db67f0ce7a45.png)


