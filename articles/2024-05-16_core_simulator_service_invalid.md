---
title: "Loaded CoreSimulatorService is no longer valid for this process" # 記事のタイトル
emoji: "📱" # Choose an appropriate emoji for each article
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Xcode", "iOS", "Simulator", "Swift", "Flutter"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
# これなに
CoreSimulatorService がも使えないよと言われた時の対応方法

# エラー内容
Simulator 起動しようと思ったらこんなエラーが
```zsh
Loaded CoreSimulatorService is no longer valid for this process.
Simulator services will no longer be available.
Error=Error Domain=NSPOSIXErrorDomain Code=61 "Connection refused" 
UserInfo={NSLocalizedDescription=CoreSimulator.framework was changed while the process was running.
This is not a supported configuration and can occur if Xcode.app was updated while the process was running.
Service version (947.17) does not match expected service version (947.13).}
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/9ec845b1-75bd-3901-870e-6d3eb2f4f07d.png)

# 解決方法
とりあえずキャッシュ吹っ飛ばしたら解決しました。
[以前書いた記事](https://qiita.com/su3-hokkaido/items/dd4e44a4680140e6d49b)と同様に以下のコマンドを実行して解決。
```zsh
rm -rf ~/Library/Developer/CoreSimulator/Caches/*
```
