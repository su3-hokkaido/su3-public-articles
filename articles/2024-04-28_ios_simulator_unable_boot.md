---
title: "iOS Simulator で Unable to boot the Simulator とエラーが出た時の対処方法" # 記事のタイトル
emoji: "📱" # Choose an appropriate emoji for each article
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["iPhone", "Xcode", "iOS", "Simulator", "Flutter"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
# これなに
iOS シミュレーターを起動しようと思ったらこのメッセージが出てきてなにも起動できなかったので、その時の試行内容と解決方法をメモ。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/88e84412-8114-9e59-f1f2-56036ab4099d.png)


# 解決方法
キャッシュ以下すべてを削除して復活しました。
```zsh
rm -rf ~/Library/Developer/CoreSimulator/Caches/*
```


# エラーログ
`cat ~/Library/Logs/CoreSimulator/CoreSimulator.log` して調べてみると以下のようなエラーが吐かれていた。

`Failed to start launchd_sim: could not bind to session, launchd_sim may have crashed or quit responding` から察するに、なんかクラッシュしとるらしいとのこと。知らんけど。
```zsh
Apr 28 09:25:17 su3-hokkaido CoreSimulatorService[1434] <Error>: Unable to boot simulator session com.apple.CoreSimulator.SimDevice.DA5009B0-DB8A-4FE0-BB46-EC63E908D72C (isTimeout = YES): Error Domain=com.apple.SimLaunchHostService.RequestError Code=4 "Failed to start launchd_sim: could not bind to session, launchd_sim may have crashed or quit responding" UserInfo={NSLocalizedDescription=Failed to start launchd_sim: could not bind to session, launchd_sim may have crashed or quit responding}
Apr 28 09:25:17 su3-hokkaido CoreSimulatorService[1434] <Error>: Error Domain=NSPOSIXErrorDomain Code=60 "Operation timed out" UserInfo={NSUnderlyingError=0x600001cdf540 {Error Domain=com.apple.SimLaunchHostService.RequestError Code=4 "Failed to start launchd_sim: could not bind to session, launchd_sim may have crashed or quit responding" UserInfo={NSLocalizedDescription=Failed to start launchd_sim: could not bind to session, launchd_sim may have crashed or quit responding}}, NSLocalizedDescription=Unable to boot the Simulator., Session=com.apple.CoreSimulator.SimDevice.DA5009B0-DB8A-4FE0-BB46-EC63E908D72C, NSLocalizedFailureReason=launchd failed to respond.}
Apr 28 09:25:17 su3-hokkaido CoreSimulatorService[1434] <Error>: Error Domain=NSPOSIXErrorDomain Code=60 "Operation timed out" UserInfo={NSUnderlyingError=0x600001cdf540 {Error Domain=com.apple.SimLaunchHostService.RequestError Code=4 "Failed to start launchd_sim: could not bind to session, launchd_sim may have crashed or quit responding" UserInfo={NSLocalizedDescription=Failed to start launchd_sim: could not bind to session, launchd_sim may have crashed or quit responding}}, NSLocalizedDescription=Unable to boot the Simulator., Session=com.apple.CoreSimulator.SimDevice.DA5009B0-DB8A-4FE0-BB46-EC63E908D72C, NSLocalizedFailureReason=launchd failed to respond.}
Apr 28 09:25:17 su3-hokkaido com.apple.iphonesimulator[2406] <Error>: Error Domain=NSPOSIXErrorDomain Code=60 "Operation timed out" UserInfo={NSUnderlyingError=0x6000036f54a0 {Error Domain=com.apple.SimLaunchHostService.RequestError Code=4 "Failed to start launchd_sim: could not bind to session, launchd_sim may have crashed or quit responding" UserInfo={NSLocalizedDescription=Failed to start launchd_sim: could not bind to session, launchd_sim may have crashed or quit responding}}, NSLocalizedDescription=Unable to boot the Simulator., Session=com.apple.CoreSimulator.SimDevice.DA5009B0-DB8A-4FE0-BB46-EC63E908D72C, NSLocalizedFailureReason=launchd failed to respond.}
Apr 28 09:25:17 su3-hokkaido CoreSimulatorService[1434] <Error>: Error Domain=com.apple.CoreSimulator.SimError Code=405 "Unable to lookup in current state: Shutdown" UserInfo={NSLocalizedDescription=Unable to lookup in current state: Shutdown}
Apr 28 09:25:17 su3-hokkaido CoreSimulatorService[1434] <Error>: Error looking up host support port: Error Domain=com.apple.CoreSimulator.SimError Code=405 "Unable to lookup in current state: Shutdown" UserInfo={NSLocalizedDescription=Unable to lookup in current state: Shutdown}
Apr 28 09:25:17 su3-hokkaido CoreSimulatorService[1434] <Error>: Error Domain=com.apple.CoreSimulator.SimError Code=405 "Unable to lookup in current state: Shutdown" UserInfo={NSLocalizedDescription=Unable to lookup in current state: Shutdown}
Apr 28 09:25:17 su3-hokkaido CoreSimulatorService[1434] <Error>: Error Domain=com.apple.CoreSimulator.SimError Code=405 "Unable to lookup in current state: Shutdown" UserInfo={NSLocalizedDescription=Unable to lookup in current state: Shutdown}
Apr 28 09:25:17 su3-hokkaido CoreSimulatorService[1434] <Error>: Error looking up host support port: Error Domain=com.apple.CoreSimulator.SimError Code=405 "Unable to lookup in current state: Shutdown" UserInfo={NSLocalizedDescription=Unable to lookup in current state: Shutdown}
Apr 28 09:25:17 su3-hokkaido CoreSimulatorService[1434] <Error>: Error Domain=com.apple.CoreSimulator.SimError Code=405 "Unable to lookup in current state: Shutdown" UserInfo={NSLocalizedDescription=Unable to lookup in current state: Shutdown}
```

# 試したけどダメだったもの
### とりあえずキャッシュ削除してみた
`rm -rf ~/Library/Caches/com.apple.dt.Xcode/` でキャッシュ削除してみたら枠だけ出てきて同じくエラーに。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/0ca4e2f0-e763-8b4c-de8f-3f69b43e8bd7.png)

### 新しいエミュレーター作成してみた
同じエラーになった、やはり Simulator 自体に問題がありそう。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/5f481403-e0d8-a510-8e1a-a1c7adf0ba29.png)

### DerivedData 削除
中間ファイルを削除してみたけれども、これもダメだった。
```zsh
rm -rf ~/Library/Developer/Xcode/DerivedData/
```

# 最後に
まあエミュレーターなんてぶっ壊して OK でしょうの精神でやってましたが、なにが原因だったんだろうか。気になる。。。
