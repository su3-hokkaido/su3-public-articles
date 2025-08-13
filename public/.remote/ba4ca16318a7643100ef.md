---
title: Flutter で Supabase 接続をしてみた手順をまとめてみた
tags:
  - Google
  - Dart
  - PostgreSQL
  - Flutter
  - Supabase
private: false
updated_at: '2024-04-20T14:08:40+09:00'
id: ba4ca16318a7643100ef
organization_url_name: null
slide: false
ignorePublish: false
---
# なにこれ
Flutter で Supabase の接続までをまとめたもの。
公式ページや Stackflow とか Zenn とかであがっている記事を見たりしたけど完全補完できなかったので個人的なメモとしてまとめました。
（そのため、興味があったらこの記事も「ふーん、そうなんだ」くらいで読んでみてください）

あと最後に自分で設定していた時にひっかかったところがあったので、そのメモも書いておきました。

# Supabase 初期セットアップ
まずは supabase で初期セットアップをしましょう！
https://supabase.com/
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/bf6363ed-3feb-aa5e-1210-c8e5afd61813.png)

ユーザー認証部分は割愛しますが、ダッシュボードまで辿り着ければOK
https://supabase.com/dashboard/projects
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/2f88cbd7-9f53-bf9d-8058-c801ecf34310.png)

# Supabase で プロジェクト作成
先ほどのダッシュボード画面から `Start your project` でプロジェクト作成フローを開いて進めていきましょう。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/6d33a631-7960-1fb2-3c93-4aeaa4fda5f4.png)

プロジェクト作成ボタンをクリックしたら完成。あっという間。
プロジェクトトップの Project API keys > anon と Project Configration > URL は後ほど使用します。（このプロジェクト画面にくれば確認できるのでメモしなくてもOK）
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/2bf685a3-c942-39d3-932f-d2d8e0b02f65.png)

# テーブル作成
### SQL editor で作成
こちらの URL から SQL で作成可能です。
URL にプロジェクト指定していないので、まずは直下の画面が出てきますが、そのままプロジェクトを選択して SQL Editor を開いてください。
https://supabase.com/dashboard/project/_/sql/new
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/90734f65-3686-d954-755d-afbd5ae0b83d.png)

参考までに以下の SQL を実行してみました。
```sql
 -- Create the table
 CREATE TABLE countries (
 id SERIAL PRIMARY KEY,
 name VARCHAR(255) NOT NULL
 );
 -- Insert some sample data into the table
 INSERT INTO countries (name) VALUES ('United States');
 INSERT INTO countries (name) VALUES ('Canada');
 INSERT INTO countries (name) VALUES ('Mexico');
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/bebbe328-7b2a-1541-8080-2b8390850945.png)

おお〜カンタン
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/70910811-067e-3d97-9395-8338a8054e5a.png)

### GUI で作成
同様にプロジェクトを指定していないURLを書きました。
画面からだとメニューから Table Editor や Database を開けば同様の画面に到達できます。
https://supabase.com/dashboard/project/_/database/tables
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/402882a4-78c4-61e2-606a-4df82a9fefe3.png)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/9dc4eb9d-8e37-6d74-9138-eff91412724e.png)

`+ New table` をクリックして GUI Editor を開きましょう。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/6e6f5cff-09f4-09cc-a37c-551b881beb66.png)

テーブル名、説明文などを入力していきます。
カラムも直感的に設定できますね。
`Enable Row Level Security (RLS)` はセキュリティ面から ON にしておきましょう。
できたら Save で保存しましょう。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/404bb004-380b-e013-9e9c-fba2b7238b9c.png)

テーブルできた！
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/85560a3a-184d-b134-9fec-b7e755ae147e.png)

出来上がったテーブルは Table Editor から直接みることができます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/96d7d06e-42a0-5581-d269-dd0c221872f8.png)

# Flutter 側の Supabase 接続設定
ここから Flutter 側での接続設定。

## package インストール
`supabase_flutter` を入れましょう。
`flutter_dotenv` は必須ではないですが、環境変数に入れたいのでついでに追加。

```zsh
flutter pub add supabase_flutter
flutter pub add flutter_dotenv
```

## pubspec.yaml で .env のパスを通す
```yaml
flutter:
  assets:
    - .env
```

## プロジェクト直下に .env ファイルを作成する
```zsh
SUPABASE_ANON_KEY="先ほどの supabase で確認した anon の値をここにセット、ダブルクオーテーションはいらないですよ"
SUPABASE_URL="先ほどの supabase で確認した URL の値をここにセット"
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/ee188285-6a69-8794-be82-088f21e0f789.png)

## .gitignore に .env を追加
Git に上がらないように。。。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/0238d557-82b7-0e58-3c8c-72f604f317d1.png)

## アプリ起動時に初期化処理を追加
`main()` を以下のように書き換えました。
await は基本的に読み込み処理を待つために必要なので、ちゃんと入れましょう。
（なぜそれを書いたかというと、僕は最初に理由もなく「要らんでしょ！」って言って無視してたので）
```dart
Future<void> main() async {
  // Ensure flutter binding has been initialized
  WidgetsFlutterBinding.ensureInitialized();

  // Load .env file
  await dotenv.load(fileName: '.env');

  // Initialize Supabase
  await Supabase.initialize(
    url: dotenv.env['SUPABASE_URL']!,
    anonKey: dotenv.env['SUPABASE_ANON_KEY']!,
  );

  // Run the app
  runApp(MainApp());
}
```

## 接続設定完了！
設定はできたのであとは supabase とやりとりするだけ！

# Flutter で Supabase のテーブルに CRUD 処理を行う
## インスタンス初期化処理
まずは最初に初期化をします。
```dart
final supabase = Supabase.instance.client;
```

## select を実行してみよう
これは自分で作ったテーブルなので、
from にはテーブル名、select には取得したいカラム名を指定します。
```dart
final res = await supabase.from('user_info').select('id, username, email');
```

ちなみにアスタリスクも使えます。
```dart
final res = await supabase.from('user_info').select('*');
```

debug で取得してみました、良い感じ。
```dart
debugPrint('*** res: $res');
```
```zsh
I/flutter (18662): *** res: [{id: ecfd1a1e-3644-49cc-82fb-68e0d55f240d, username: hoge2, email: hoge2@gmail.com}]
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/9e3a02be-389d-fe8f-5580-a856bcbc95b6.png)

## insert 処理
こんな感じで行けます。
ちなみに res には登録結果を入れています。
```dart
try {
  final res = await supabase.from('user_info').insert({
    'username': username,
    'email': email,
    'created_at': DateTime.now().toIso8601String(),
    'updated_at': DateTime.now().toIso8601String(),
    'deleted_at': null,
  });
} catch (e) {
  // エラー処理を書く
}
```

## update 処理
この辺から説明割愛。
とりあえず `match` は where ですね。
```dart
try {
  final res = await supabase.from('user_info').update({
    'username': _usernameController.text,
    'email': _emailController.text,
    'updated_at': DateTime.now().toIso8601String(),
  }).match({'id': widget.user_info.id});
} catch (e) {
  ...
}
```

## upsert 処理
レコードがあったら update, なかったら insert するもの。
rails の `find_or_create_by` っぽい。（初心者らしい感想）
```dart
try {
  final res = await supabase.from('user_info').upsert({
    'id': widget.user_info?.id, // Primary key
    'username': _usernameController.text,
    'email': _emailController.text,
  });
} catch (e) {
  ...
}
```

## delete 処理
id 一致させて削除、カンタン。
```dart
try {
  await supabase.from('user_info').delete().match({'id': user_info.id});
} catch (e) {
  ...
}
```

# where や NOT NULL のようなものの tips
カンタンなもの以外にも色々 filter したくなるので、その辺をメモ

## Filter
### eq
where A = B 的なもの
```dart
final data = await supabase
  .from('countries')
  .select('name, country_id')
  .eq('name', 'Japan');
```

### neq
where A != B 的なもの
```dart
final data = await supabase
  .from('countries')
  .select('name, country_id')
  .neq('name', 'Japan');
```

### match
where A = B 的なもの、複数条件指定可能。
```dart
final data = await supabase
  .from('countries')
  .select('name, country_id')
  .match('name', 'Japan', country_id: 81);
```

### textSearch
where A = '' 的なもの
`&` で複数条件設定できる模様
```dart
final data = await supabase
  .from('countries')
  .select('name, country_id')
  .textSearch('name', "'Japan' & 'Korea'");
```

### gt
where A > B
```dart
final data = await supabase
  .from('countries')
  .select('name, country_id')
  .gt('country_id', 10);
```

### gte
where A >= B
```dart
final data = await supabase
  .from('countries')
  .select('name, country_id')
  .gte('country_id', 10);
```

### lt
where A < B
```dart
final data = await supabase
  .from('countries')
  .select('name, country_id')
  .lt('country_id', 10);
```

### lte
where A <= B
```dart
final data = await supabase
  .from('countries')
  .select('name, country_id')
  .lte('country_id', 10);
```

## Modifier
### order
order by
```dart
final data = await supabase
  .from('countries')
  .select('name, country_id')
  .order('country_id', ascending: true);
```

### limit
```dart
final data = await supabase
  .from('countries')
  .select('name, country_id')
  .limit(10);
```

# Appendix
Supabase から `new row violates row-level security policy` が返ってきて「なんで？」と思ったのですが、それはポリシー設定が原因でした。
上記の設定ではこのエラー対応については特に記載していませんので、詳しくはこちらの記事で。
https://qiita.com/su3-hokkaido/items/ccd989050ecb3ba303fd
