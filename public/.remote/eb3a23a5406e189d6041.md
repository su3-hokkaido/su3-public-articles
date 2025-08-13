---
title: >-
  UNPROTECTED PRIVATE KEY FILE!
  というメッセージが出て、SSHプライベートキーにアクセスできない時にファイルパーミッションを許可した時のメモ
tags:
  - AWS
  - SSH
  - EC2
  - chmod
  - gce
private: false
updated_at: '2024-11-13T22:28:18+09:00'
id: eb3a23a5406e189d6041
organization_url_name: null
slide: false
ignorePublish: false
---
# これなに

EC2インスタンスにアクセスしようとした時に、`UNPROTECTED PRIVATE KEY FILE!` と出て、どうしようかな…と思った時のお話。
超初心者すぎるのですが、戒めのためにメモ。


# なにが起きた？

キーペアも作成してインスタンスもできた。
さあ ssh してみようと思ったら ssh に失敗した。
（ホスト名とか Public IP アドレスとかは config に書いてあるのですがここでは割愛）

```zsh
su3@macbook ~ % ssh Test-EC2
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for '/Users/su3/.ssh/test-ec2.pem' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "/Users/su3/.ssh/test-ec2.pem": bad permissions
ec2-user@xxx.xxx.xxx.xxx: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
su3@macbook ~ % 
```

なるほど、ファイルにアクセスできないんですね。


# 解決方法

ファイル所有者に読み取りと書き込みの権限を与えればOKですね

```zsh
chmod 600 ~/.ssh/test-ec2.pem
```

そうして無事に ssh 成功

```zsh
su3@macbook ~ % ssh Test-EC2
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'
Last login: Wed Nov 13 13:10:54 2024 from 999.999.999.999
[ec2-user@ip-999-999-999-999 ~]$ 
```

# 備考

ついでに他にも使いそうなコマンドをザッとメモ

### 所有者に読み取り権限を付与する

```zsh
chmod u+r ファイルパス
```

### 所有者に実行権限を付与する

```zsh
chmod u+x ファイルパス
```

### 該当のフォルダ配下に対して書き込み権限を付与する

```zsh
chmod -R 755 ディレクトリパス
```

### 該当のフォルダ配下に対して書き込み権限を削除する

```zsh
chmod -R a-w ディレクトリパス
```

### 該当のフォルダ配下に対して実行権限を削除する

```zsh
chmod -R a-x ディレクトリパス
```






