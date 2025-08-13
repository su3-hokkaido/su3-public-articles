---
title: CloudFormation で生成した Stack が詰まってしまった場合の解決方法
tags:
  - AWS
  - CloudFormation
  - aws-cli
  - yml
  - stack
private: false
updated_at: '2024-11-20T23:37:41+09:00'
id: dbbdf7849e4c39aaefb3
organization_url_name: null
slide: false
ignorePublish: false
---
# これなに

- CloudFormation を利用して CLI から Stack を作成して実行したい
- しかし、その時にステータスが処理中系の状態になっており CLI でのコマンド実行が失敗してしまい、処理を止めたい時にどうにかしたいと思ったのでそれの解決方法を調べてみた


# 記事作成の背景

- 以下の通り、該当の Stack 名称がスタックしてしまってエラーが発生したことで本記事作成に至った

```zsh
$ aws cloudformation deploy --template-file 4_cloudformation_db_stack.yml --stack-name su3hokkaido-app --parameter-overrides $(cat dev.cfg) --capabilities CAPABILITY_NAMED_IAM --no-execute-changeset

An error occurred (ValidationError) when calling the CreateChangeSet operation: Stack:arn:aws:cloudformation:ap-northeast-1:314146317837:stack/su3hokkaido-app/2d6a1ab0-a72d-11ef-9c48-06a6ece35625 is in ROLLBACK_COMPLETE state and can not be updated.
$ 
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/6f8b6a1b-ec72-ffd3-d826-4aa6de8f2f7b.png)


# 解決方法

### 1: Stack を削除する

特になければ一旦はこちらの方法で解決可能。
削除したら CloudFormation の画面で該当の Stack が削除されたことを確認してみましょう。

**実行コマンド**

```zsh
$ aws cloudformation delete-stack --stack-name ${STACK_NAME}
```

**実行例**

```zsh
$ aws cloudformation delete-stack --stack-name su3hokkaido-app
```

### 2: Stack を強制的に削除する

前述の方法がダメならこちらで強制的に削除しましょう

**実行コマンド**

```zsh
# スタックの削除保護を無効にする
$ aws cloudformation update-termination-protection --stack-name ${STACK_NAME} --no-enable-termination-protection

# スタックを削除する
$ aws cloudformation delete-stack --stack-name ${STACK_NAME}

# スタックの削除が完了するまで待つ
$ aws cloudformation wait stack-delete-complete --stack-name ${STACK_NAME}
```

**実行例**

```zsh
$ aws cloudformation update-termination-protection --stack-name su3hokkaido-app --no-enable-termination-protection
$ aws cloudformation delete-stack --stack-name su3hokkaido-app
$ aws cloudformation wait stack-delete-complete --stack-name su3hokkaido-app
```

# あとがき

GUI も便利ですが CLI から実行した方が理解を深めることができるので、ガイドを参考にしながら学習を進めたい…

https://docs.aws.amazon.com/cli/latest/reference/cloudformation/delete-stack.html

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/590b08b5-f9c5-ca60-644d-af4170db6951.png)

