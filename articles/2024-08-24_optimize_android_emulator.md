---
title: "【週末お掃除】Android emulator のパフォーマンス改善" # 記事のタイトル
emoji: "🧹" # Choose an appropriate emoji for each article
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Android", "AndroidStudio", "emulator", "AndroidEmulator"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
# これなに
- Android emulator パフォーマンスが出ない問題について対応したメモです
- 多分いろんな先人達がやり尽くしていると思われるので、参考程度まで
- とりあえずこれやったら体感だいぶ速くなった


# 解決方法
### Device Manager から対象の AVD の Show on Disk を開く
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/9f2aed00-31c7-72ad-415a-374d219e0bbc.png)

### config.ini を編集する
- GPU mode の変更, RAM と heap はデフォルト値の4倍に修正
- GPU: auto => host
- RAM: 2048 => 8192
- heap: 256 => 1024

```ini
hw.gpu.mode=host
hw.ramSize=8192
vm.heapSize=1024
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/5054bca2-fea1-78e5-d7fa-95aac7ced692.png)


# 今回の話で困ったこと
なぜか上記の設定値は GUI で編集できなかったので、仕方なくファイルを直接変更して対応してみた。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/97ebba43-280b-f82b-7161-9b6e45a69243.png)

