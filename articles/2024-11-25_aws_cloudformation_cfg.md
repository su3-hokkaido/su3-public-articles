---
title: "AWS CloudFormation で利用する cfg ファイルに関する学習メモ" # 記事のタイトル
emoji: "☁️" # Choose an appropriate emoji for each article
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["AWS", "CloudFormation", "Cluster", "yml", "ECS"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
# これなに

- AWS CloudFormation の実行を色々試したときに気になった以下の点：
    - cfg ファイルとは？
    - どうやって使う？
- これらの内容を個人的な学習メモとして記録した記事です

# cfg ファイルとは？

- 端的に言うと設定ファイル
- CloudFormation では主に任意のパラメーターの値を決め打ちで設定しておきたい時に利用

# どうやって使う？

- cfg ファイルには変数名と値をイコールで結んで設定します
- CloudFormation を実行するための yml ファイル内で cfg ファイルの中身を呼び出すには `!Ref` という記載をする
- 実行時には cat で cfg ファイルの中身を渡して実行する

### cfg ファイル内容の例

例えば `dev.cfg` という名前で作成

```cfg
EnvironmentName=su3hokkaido
ClusterName=su3hokkaido-cluster
DatabasePassword=rds-database-secret-password
Keypairname=keypair01
ApplicationDesiredCount=1
DesiredCount=1
DBInstanceClass=db.t4g.micro
DBName=su3hokkaido_db
DBMasterUsername=su3hokkaido_user
```

### yml ファイル内での呼び出し方

`!Ref` という接頭語を付けた上で変数名を呼び出す

```yml
...
  Cluster:
    Type: AWS::ECS::Cluster
    Properties:
      ClusterName: !Ref ClusterName

  ApplicationTaskDefinition:
    Type: AWS::ECS::TaskDefinition
    Properties:
      Family: todobackend
      Volumes:
        - Name: public
          Host:
            SourcePath: /data/public
      ContainerDefinitions:
        - Name: todobackend
          Image: !Sub ${AWS::AccountId}.dkr.ecr.${AWS::Region}.amazonaws.com/su3hokkaido/todobackend:${ApplicationImageTag}
          MemoryReservation: 395
          Cpu: 245
          MountPoints:
            - SourceVolume: public
              ContainerPath: /public
          Environment:
            - Name: DJANGO_SETTINGS_MODULE
              Value: todobackend.settings_release
            - Name: MYSQL_HOST
              Value: !Sub ${ApplicationDatabase.Endpoint.Address}
            - Name: MYSQL_USER
              Value: todobackend
            - Name: MYSQL_PASSWORD
              Value: !Ref DatabasePassword # ここで dev.cfg から変数を呼び出している
            - Name: MYSQL_DATABASE
              Value: todobackend
            - Name: SECRET_KEY
              Value: some-random-secret-should-be-here
...
```

### 実行時に cat で呼び出す

`--parameter-overrides $(cat dev.cfg)` で cfg ファイルに定義した変数を呼び出している

```zsh
aws cloudformation deploy --template-file cloudformation_task_definition.yml --stack-name su3hokkaido-app --parameter-overrides $(cat dev.cfg) --capabilities CAPABILITY_NAMED_IAM --no-execute-changeset
```


# あとがき

CloudFormation は便利だけどいろんな情報があって大変…
ちゃんと自分のサンプルファイルを作っておこう
