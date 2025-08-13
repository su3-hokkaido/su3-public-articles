---
title: "MySQL でテーブルサイズを調査する手順" # 記事のタイトル
emoji: "📊" # Choose an appropriate emoji for each article
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["MySQL", "SQL", "DB", "Database", "MySQLWorkbench"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
# これなに

RDS インスタンスのランニングコストを節約したいと思って、どのテーブルサイズが大きいのだろうと調べた時のメモです。
わりとみなさんご存知の内容だと思いますし、説明も十分では無いと思うので参考程度にご覧いただければと思います。


# 調査手順

### MySQL に接続

釈迦に説法だと思いますが、一応 MySQL の接続コマンドをメモしておきます。
`-h`, `--port` オプションは任意なので、ご自身の環境に合わせてカスタムしてください。

```zsh
> mysql -u ユーザー名 -h ホスト名 -- port ポート番号 -p
```

### テーブルサイズ調査クエリ

`information_schema` データベースを使用してテーブルのサイズ情報を取得できます。
クエリは以下の通りです。

```sql
SELECT 
    table_schema AS `Database`, 
    table_name AS `Table`, 
    table_rows AS `Rows`,
    ROUND((data_length + index_length) / 1024 / 1024, 2) AS `Size (MB)`
FROM 
    information_schema.TABLES
WHERE 
    table_schema = 'データベース名称'
ORDER BY 
    (data_length + index_length) DESC;
```


取得したクエリ結果は以下のようなものです。
参考程度にご覧ください。

```sql
mysql> SELECT
    ->     table_schema AS `Database`,
    ->     table_name AS `Table`,
    ->     table_rows AS `Rows`,
    ->     ROUND((data_length + index_length) / 1024 / 1024, 2) AS `Size (MB)`
    -> FROM
    ->     information_schema.TABLES
    -> WHERE
    ->     table_schema = 'su3_database'
    -> ORDER BY
    ->     (data_length + index_length) DESC;
+--------------+--------------------+---------+-----------+
| Database     | Table              | Rows    | Size (MB) |
+--------------+--------------------+---------+-----------+
| su3_database | sent_email_history |   41965 |   1167.23 |
| su3_database | user_settings      |   52319 |   1110.16 |
| su3_database | user_accounts      | 3189785 |    410.77 |
+--------------+--------------------+---------+-----------+
3 rows in set (0.02 sec)

mysql>
```


# さいごに

結構忘れがちなので個人的にメモしておきました。
繰り返しになりますが、わりとみなさんご存知の内容だと思いますし、説明も十分では無いと思うので参考程度にご覧いただければと思います。
