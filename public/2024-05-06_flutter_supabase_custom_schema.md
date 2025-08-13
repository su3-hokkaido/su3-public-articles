---
title: Flutter x Supabase でカスタムスキーマを作成したときの手順と `permission denied for schema` のエラー解消
tags:
  - Dart
  - PostgreSQL
  - Flutter
  - pgadmin4
  - Supabase
private: false
updated_at: '2024-05-06T00:20:42+09:00'
id: bcdada0c403e3bc2b3e1
organization_url_name: null
slide: false
ignorePublish: false
---
# これなに
カスタムスキーマを作成した際に `permission denied for schema` というエラーが出たので、その解消方法をメモ。
ついでに Flutter で Supabase のカスタムスキーマに切り替えた時のおおよその手順もメモ。


# 実際に遭遇したエラー
`my_schema_test` というスキーマを作成して、スキーマ切り替えテストで debugPrint していたらこんな感じのエラーが出てきた。
のちにわかるものですが、権限設定が不足していたようです。
```zsh
PostgrestException(message: permission denied for schema my_schema_test, code: 42501, details: Unauthorized, hint: null)
```


# 参考ガイド
以下の公式ガイドに記載されている通り、権限付与をしてくださいとのことで、これが全てでした。
https://supabase.com/docs/guides/api/using-custom-schemas?queryGroups=language&language=curl
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/68b58620-612f-bc8e-2f6a-8d28a2effb68.png)


# カスタムスキーマ利用の全体の手順
### 1: スキーマ作成
GUI からスキーマを作成、かんたん。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/21d011f0-34b7-b1ad-6870-3eac15a9f2ff.png)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/8d696609-2cab-dc42-eaa1-71893d85f560.png)


### 2: 権限設定
前述の通りスキーマを作成したら権限付与
```sql
GRANT USAGE ON SCHEMA my_schema_test TO anon, authenticated, service_role;
GRANT ALL ON ALL TABLES IN SCHEMA my_schema_test TO anon, authenticated, service_role;
GRANT ALL ON ALL ROUTINES IN SCHEMA my_schema_test TO anon, authenticated, service_role;
GRANT ALL ON ALL SEQUENCES IN SCHEMA my_schema_test TO anon, authenticated, service_role;
ALTER DEFAULT PRIVILEGES FOR ROLE postgres IN SCHEMA my_schema_test GRANT ALL ON TABLES TO anon, authenticated, service_role;
ALTER DEFAULT PRIVILEGES FOR ROLE postgres IN SCHEMA my_schema_test GRANT ALL ON ROUTINES TO anon, authenticated, service_role;
ALTER DEFAULT PRIVILEGES FOR ROLE postgres IN SCHEMA my_schema_test GRANT ALL ON SEQUENCES TO anon, authenticated, service_role;
```

### 3: コードでスキーマ指定
以下の通りスキーマを指定する。
```dart
  await Supabase.initialize(
    url: dotenv.env['SUPABASE_URL']!,
    anonKey: dotenv.env['SUPABASE_ANON_KEY']!,
    postgrestOptions: PostgrestClientOptions(schema: 'my_schema_test'), // ここを追記
  );
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/6253b406-6c5f-bc97-de4e-8522f2a39bd3.png)


# おわりに
日本語の情報は無かったので FlutterFlow とか Stackflow とか Github issue とか漁ってました。
だけど最終的に公式のガイド見つけたのでそれで解決しました、よかったよかった。
