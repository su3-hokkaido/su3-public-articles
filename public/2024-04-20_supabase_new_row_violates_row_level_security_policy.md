---
title: Supabase をコールしたら new row violates row-level security policy となった
tags:
  - Google
  - Dart
  - PostgreSQL
  - Flutter
  - Supabase
private: false
updated_at: '2024-04-20T14:05:29+09:00'
id: ccd989050ecb3ba303fd
organization_url_name: null
slide: false
ignorePublish: false
---
# なにこれ
Flutter から Supabase のテーブルにアクセスしようと思ったら以下のエラーにぶつかったので、どのように解決したかをメモしました。
```zsh
PostgrestException(message: new row violates row-level security policy for table "user", code: 42501, details: Unauthorized, hint: null)
```


# 原因
最初に Row level security policy を Enabled にしていたのに、その設定をしていなかったことが原因でした。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/f2d6515c-03b0-e393-e657-d30674ba18a3.png)


# 解決方法
### ポリシー作成画面を開く
Authentication > Policies > Creata a new policy でポリシーを作成します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/2ea19087-d82e-4575-e3c3-0bd2da9a2bc5.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/7e50916d-75b0-115f-78da-84167f7fb643.png)


### ポリシー内容を設定
全部使いたいので、Policy command は ALL にします。
細かい設定が必要であればここで select だけにするとかそういうことも可能。
using は true にしましょう。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/f3e75fb2-889c-51d3-7019-dfa3afa3119b.png)

出来上がったらこんな感じ。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/f1e3a466-30fa-3764-8913-eb02af8dbc9b.png)


# 無事治った！
自分の画面は載せられないですが、デバッグしてみてちゃんと返ってきた。よかったよかった。
