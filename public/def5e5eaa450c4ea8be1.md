---
title: AWS CloudFormation を CLI から生成だけ行なって自動実行させない方法
tags:
  - AWS
  - CloudFormation
  - vpc
  - yml
  - Changesets
private: false
updated_at: '2024-11-20T14:10:44+09:00'
id: def5e5eaa450c4ea8be1
organization_url_name: null
slide: false
ignorePublish: false
---
# これなに

- 単なる初学者のメモです
- CLI から実行しても CloudFormation が自動実行されないなあと思ったけど、そういうオプション指定してたのでそりゃそうだという学習メモ


# CloudFormation を実行せずに生成だけを CLI で行う手順

- `--no-execute-changeset` オプションを付加するだけ
- サンプルコマンドは以下の通り

```zsh
$ aws cloudformation deploy --template-file ${ymlファイルパス} --stack-name su3hokkaido-app --capabilities CAPABILITY_NAMED_IAM --no-execute-changeset
```

# 記事作成の背景

- VPC, subnet, ルートテーブル, ECR の作成を CloudFormation を使って CLI 経由で実行しようとした
- しかし、全く実行される気配がなくて何が原因かわからなかった
- 参考にして実行したコマンドにオプションを指定しているのにそれがわかってなかった

### 生成された Stack

ずっとレビュー中になってる

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/d4287bba-9405-220b-9b40-d3609aba23e2.png)

### 実際のコマンドライン

ちょびっとテキトーな値に書き換えてますが、大体こんな感じです。
実行した yml ファイルの中身は割愛します。

```zsh
$ aws cloudformation deploy --template-file 2_cloudformation_vpc.yml --stack-name su3hokkaido-app --parameter-overrides $(cat dev.cfg) --capabilities CAPABILITY_NAMED_IAM --no-execute-changeset

Waiting for changeset to be created..
Changeset created successfully. Run the following command to review changes:
aws cloudformation describe-change-set --change-set-name arn:aws:cloudformation:ap-northeast-1:999999999999:changeSet/awscli-cloudformation-package-deploy-1732077013/aa1e1a24-2365-4320-a767-e6b0092bc252
$ 
```

### 実際の解決

#### 1: `--no-execute-changeset` を外して実行する

これが一番簡単な方法

#### 2: GUI から実行

- CloudFormation > Stacks から生成した Stack を開いて Change Sets のタブを開く
- タブの中に表示されている生成済みの Change Set を選択して Execute change set で実行する

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/cc6792ea-b7d0-688b-d9a3-4b2a4786561c.png)

#### 3: CLI から実行

生成された Stack 名称を確認

```zsh
# command: aws cloudformation list-change-sets --stack-name ${Stack名称}
$ aws cloudformation list-change-sets --stack-name su3hokkaido-app
{
    "Summaries": [
        {
            "StackId": "arn:aws:cloudformation:ap-northeast-1:999999999999:stack/su3hokkaido-app/22990a90-a6f8-11ef-885b-065399c40a55",
            "StackName": "su3hokkaido-app",
            "ChangeSetId": "arn:aws:cloudformation:ap-northeast-1:999999999999:changeSet/awscli-cloudformation-package-deploy-1732077013/aa1e1a24-2365-4320-a767-e6b0092bc252",
            "ChangeSetName": "awscli-cloudformation-package-deploy-1732077013",
            "ExecutionStatus": "AVAILABLE",
            "Status": "CREATE_COMPLETE",
            "CreationTime": "2024-11-20T04:30:13.543000+00:00",
            "Description": "Created by AWS CLI at 2024-11-20T04:30:13.305774 UTC",
            "IncludeNestedStacks": false
        }
    ]
}
```

以下のコマンドで実行する

```zsh
# command: aws cloudformation execute-change-set --stack-name ${Stack名称} --change-set-name ${ChangeSet名称}
$ aws cloudformation execute-change-set --stack-name su3hokkaido-app --change-set-name awscli-cloudformation-package-deploy-1732077013
```


# 最後に

自戒のため記事作成しました。
ちゃんとひとつひとつのメソッド、コマンド、オプションなどを理解して作業しないと取り返しのつかない問題に直面する可能性もあるので慎重にやっていきましょう！

あー恥ずかしい！
