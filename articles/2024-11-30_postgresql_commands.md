---
title: "PostgreSQL でよく使うコマンド" # 記事のタイトル
emoji: "🐘" # Choose an appropriate emoji for each article
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["SQL", "PostgreSQL", "postgres", "Database", "psql"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
# これなに

 - 普段 mysql ばかり使っていて、PostgreSQL を使ってみようと思ったら結構コンソール上のコマンドが違っていた
 - そのため、今後 PostgreSQL を使うことがあればと思って SQL 文以外でよく使いそうなコマンドをメモ
 - 故に、この記事は個人的なメモなので、誰かに向けたものではないのでご了承ください


# 前提条件

- PostgreSQL version: 15.1

```sql
practice_db=# SELECT version();
                                                        version                                                        
-----------------------------------------------------------------------------------------------------------------------
 PostgreSQL 15.1 on aarch64-unknown-linux-musl, compiled by gcc (Alpine 12.2.1_git20220924-r4) 12.2.1 20220924, 64-bit
(1 row)

practice_db=# 
```

# 基本コマンド

コマンド | 実行内容
--- | ---
psql -U <ユーザー名> -d <DB名> | PostgreSQL ログイン
\c <DB名> | データベースを切り替える
\dt | テーブルの一覧を表示する
\l | データベースの一覧を表示する
\d <テーブル名> | テーブルの詳細情報を表示する
\du | ユーザーの一覧を表示する
SELECT version() | PostgreSQL バージョン確認
\q | psqlを終了する


# その他の便利なコマンド

コマンド | 実行内容
--- | ---
\d+ <テーブル名> | テーブルの詳細情報をさらに詳しく表示する
\i <ファイル名> | ファイルからSQLコマンドを実行する
\timing | クエリの実行時間を表示する


# 実行例

個人的にコマンドを見ただけではどんなものが出てくるのか想像がつかないものだけ以下に実行例を掲載します

## `\d+ <テーブル名>` 実行例

mysql でいう describe 的なもの

```sql
practice_db=# \d+ test_users
                                                                   Table "public.test_users"
   Column   |            Type             | Collation | Nullable |                Default                 | Storage  | Compression | Stats target | Description 
------------+-----------------------------+-----------+----------+----------------------------------------+----------+-------------+--------------+-------------
 id         | integer                     |           | not null | nextval('test_users_id_seq'::regclass) | plain    |             |              | 
 username   | character varying(50)       |           | not null |                                        | extended |             |              | 
 email      | character varying(200)      |           | not null |                                        | extended |             |              | 
 created_at | timestamp without time zone |           |          | CURRENT_TIMESTAMP                      | plain    |             |              | 
 updated_at | timestamp without time zone |           |          | CURRENT_TIMESTAMP                      | plain    |             |              | 
 deleted_at | timestamp without time zone |           |          |                                        | plain    |             |              | 
Indexes:
    "test_users_pkey" PRIMARY KEY, btree (id)
    "test_users_email_key" UNIQUE CONSTRAINT, btree (email)
Access method: heap

practice_db=# 
```


## `\timing` 実行例

- `\timing` を実行すると実行時間を表示するか否かを切り替えられる
- timing on, off それぞれの実行例を以下の通り示す

```sql
-- timing ON ケース
practice_db=# \timing
Timing is on.
practice_db=# select * from test_users;
 id | username | email | created_at | updated_at | deleted_at 
----+----------+-------+------------+------------+------------
(0 rows)

Time: 0.914 ms
practice_db=# 
```

```sql
-- timing OFF ケース
practice_db=# \timing
Timing is off.
practice_db=# select * from test_users;
 id | username | email | created_at | updated_at | deleted_at 
----+----------+-------+------------+------------+------------
(0 rows)

practice_db=# 
```


# あとがき

- GUI ばっかり触っていると GUI 無い環境になったときに困る
- なので、こういったものは覚えた方が良いと思ってメモしておいた（絶対忘れそうだから）
