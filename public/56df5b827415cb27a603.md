---
title: macOS Sequoia 15.1 からパスが通らなくて Permission denied になったときの解消手順
tags:
  - SSH
  - Terminal
  - macOS
  - chmod
  - Sequoia
private: false
updated_at: '2024-12-10T12:09:47+09:00'
id: 56df5b827415cb27a603
organization_url_name: null
slide: false
ignorePublish: false
---
# これなに

macOS Sequoia 15.1 にバージョンアップした時に ssh が通らなくなったのでその解消を行った時の手順を記録しました


# 遭遇した事象

いつも通り踏み台に ssh しようと思ったら以下のようなメッセージが出た

```zsh
% ssh test-bastion
Warning: Permanently added '[nlb0.bastion.test.bastion.com]:1030' (ED25519) to the list of known hosts.
Load key "/Users/su3-hokkaido/.ssh/keypair": Permission denied
su3-hokkaido@nlb0.bastion.test.bastion.com: Permission denied (publickey).
% 
```


# 直接原因

対象のファイルにアクセスするための権限がなかった


# 解消方法

Read & write 権限を付与することで解消

```zsh
chmod 600 ~/.ssh/keypair
```

所有者が現在のユーザーではないケースもあるので、これもじっこうしておくとべたー

```
sudo chown $(whoami) ~/.ssh/known_hosts
```


# 補足

- `chmod 600`:
    - 所有者に read & write の権限を付与
    - グループや他のユーザーには一切の権限を付与しない
- `chmod 644`:
    - 所有者に read & write の権限を付与
    - グループや他のユーザーには read 権限を付与する


# あとがき

- OS アップデートしたら発生した事象ですが、根本原因は調べていないのでよくわかりません（ごめんなさい）
- たぶん同様の事象に出会っている人がいるっぽいので、[StackOverflow](https://stackoverflow.com/questions/79213830/how-do-i-get-the-issue-permission-denied-fixed-on-macos-sequoia-15-1
) とか [Reddit](https://www.reddit.com/r/LibreWolf/comments/1gei8ys/cannot_open_librewolf_after_the_latest_macos_151/?rdt=36097) にも質問が挙がっていた模様

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/d4ad3454-4364-9a16-36f8-e9d50401d518.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/6ec57675-681e-6106-5ee5-6fd50cf51067.png)

