---
title: iOS アプリ公開申請審査をしたらログインサービスのデザイン規約にひっかかってリリースまで2ヶ月かかった話
tags:
  - Xcode
  - iOS
  - Apple
  - Swift
  - AppStoreConnect
private: false
updated_at: '2025-02-06T16:47:07+09:00'
id: 4ee662358d9184bba84e
organization_url_name: null
slide: false
ignorePublish: false
---
# なにこれ

- アプリ公開申請したらログインサービスのデザイン規約を満たせていないためリジェクトされた
- それに対する修正をいろいろやってたら結果的に2ヶ月くらいかかってしまった
- その時に調べた原因と対応方法をまとめた

# リジェクト理由

簡単に振り返ると、「ユーザーがメールアドレスを提供せずにログインできること」が基本的なデザインルールということがリジェクトされた最大のポイントでした。

なお、Apple Store Connect 上で来ていたレビューコメントは以下の通り

```text
The app uses a third-party login service, but does not appear to offer an equivalent login option with the following features:

- The login option limits data collection to the user’s name and email address.

- The login option allows users to keep their email address private as part of setting up their account.

- The login option does not collect interactions with the app for advertising purposes without consent. 
```

要するに以下の要件を満たす必要があるとのこと

1. ユーザーの氏名とメールアドレスのみを収集する
2. ユーザーがメールアドレスを非公開にすることができる
3. ユーザーの同意なしに、アプリ内でのユーザーの行動を広告目的で収集しない

![Screenshot at Dec 08 14-57-29.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/e6235d11-3ea8-dfab-081d-f0dcf1146284.png)
![Screenshot at Dec 08 14-58-59.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/4c44ef1c-da81-8566-9b4c-148f8ccef2d7.png)
![Screenshot at Dec 08 14-59-15.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/6c464cab-c08b-4465-cfe1-1686e6c79eb4.png)


# それぞれの指摘に対する解決方法

## 1. ユーザーの氏名とメールアドレスのみを収集する

こちらは単純に利用規約またはプライバシーポリシーの提示が必要と思われます
実際に自分もリジェクトされた時に「サインアップするときに利用規約提示しているよ」と伝えたらOKになりました

![Screenshot at Dec 08 15-12-50.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/249e0b10-3f68-116b-dfff-b16d4b34a71f.png)

![Simulator Screenshot (2) Privacy Policy - 2024-12-08 at 13.21.13.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/650d9a55-d1df-5f0b-0c57-c029c60621f1.png)


## 2. ユーザーがメールアドレスを非公開にすることができる

修正に時間がかかってしまった最大のポイント。
最適解としては「Apple ID認証を追加すること」で解決できます。

以下は10回以上レビュー審査を往復した結果、Apple側から「Apple ID認証なら要件満たしてるからいけますよ」と言われたコメントのスクショ。

![Screenshot at Feb 06 16-29-33.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/dbdea489-9cb8-1a82-fef7-09a16b0bdef8.png)

ちなみにこの部分、一度 Google 認証でトライしてみたのですが、どうやらスコープにメールアドレスが入っているのがダメらしい。
Apple側から直接的な指摘はなかったものの、Google 認証だとユーザーはメールアドレス自体は直接提供してはいないものの、レスポンスにメールアドレスのレスポンスが存在しているためNGな感じがする。

```dart
const List<String> scopes = <String>[
  'email',
  'https://www.googleapis.com/auth/contacts.readonly',
];

GoogleSignIn _googleSignIn = GoogleSignIn(
  // Optional clientId
  // clientId: 'your-client_id.apps.googleusercontent.com',
  scopes: scopes,
);
```

## 3. ユーザーの同意なしに、アプリ内でのユーザーの行動を広告目的で収集しない

こちらも前述のように同様、利用規約またはプライバシーポリシーにその内容を記載すればOK


# アプリレビューガイド

今回のログインデザインに関する記載は 4.8 に記載されています。
詳しくは以下の URL から確認してみてください。

https://developer.apple.com/app-store/review/guidelines/

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/067f80f2-6f0d-a6e6-094e-f2f495af50bc.png)

# 今回利用したパッケージ

Google認証とApple ID認証を追加実装する際に以下のパッケージを利用しました。
実装の仕方は Readme に書いてあるのでそちらを参考にしてください。

https://pub.dev/packages/google_sign_in

https://pub.dev/packages/sign_in_with_apple


# 最後に

アプリレビューをかけたのが結局合計で11回くらいになりました。

「ログインデザインに合わない」
「掲載されているポリシー読んだ限りでは実装内容はポリシーと合ってるが？」

みたいなオウム返しのようなやりとりが何度も続き少々ストレスでしたが、ようやく公開にこぎつけられて良かった。
同じような境遇に遭遇している人がいてたまたまこの記事に辿り着いた人がいて、さらにその人にとって解決の糸口になれば嬉しい。
