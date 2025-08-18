---
title: "ヤマレコのアプリが落ちたからバグレポートを読んでみた" # 記事のタイトル
emoji: "🐞" # Choose an appropriate emoji for each article
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Java", "Android", "バグ", "RuntimeError", "ヤマレコ"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
# これなに

- ヤマレコのアプリが開けなくなった
- 普段直接デバイスとかエミュレーターのデバッグをしているだけだったので、ちょっと初めてバグレポートを見てみた
- 今回そのバグレポートを読んだ時に気づいたメモとかバグレポート確認手順などをまとめました

# バグレポート生成方法

### Developer options を開く

自分のは Additional Settings というセクションに入っていましたが、これはメーカーごとに置き場所が若干違う可能性あります。

![開発者オプション](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/e9f07a2d-f02c-4181-973c-2165f33fe5fc.jpeg)

### Bug report 作成を実行する

フルレポートは量が多くて選別が大変そうな気がしたので、interactive にしました

![バグレポート作成](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/dc19ee7c-a2ec-487e-ac09-387db732c68b.jpeg)

### 作成処理が終わったら確認する

実行するとこんな感じですね

![作成処理中](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/b3b32e76-d2da-4ce9-b97e-f33c33735258.jpeg)

# バグレポートの中身

ざっくりこんな感じみたいです。
`lshal-debug` のディレクトリを掘っていくと、落ちたところ別にファイルが分かれて置いてあるっぽいですね。

なお、今回は一番上にある `bugreport〜.txt` のファイルを読んでみました。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/d166d458-c230-436f-b7b1-c43552640375.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/89532a9a-aa91-4e60-b448-140a4e2ba8a6.png)

# 今回の本題：ヤマレコが落ちた原因

全部のログは載せられないので、気になったログだけを以下にピックアップ。
これを自分の拙いスキルで読み解くと以下のような状態らしい。

- 非同期処理で落ちた模様
- `application or mock missing database source` と書いてある通り、なんかデータベースソースが無いらしい
- `com.yamareco.memo.App.getDatabaseSource(App.java:56)` のコードがちょっとおかしいかも？（これ以上はコード持っていないからわからん）

```zsh
06-25 19:56:44.749  1000  1811  1854 I ActivityManager: Start proc 22631:com.google.android.webview:sandboxed_process0:org.chromium.content.app.SandboxedProcessService0:0/u0i963 for  {com.yamareco.memo/org.chromium.content.app.SandboxedProcessService0:0}
06-25 19:56:44.783  1000  1811  4797 D CompatibilityChangeReporter: Compat change id reported: 319212206; UID 10470; state: DISABLED
06-25 19:56:44.814 10470 22453 22658 E AndroidRuntime: FATAL EXCEPTION: AsyncTask #1
06-25 19:56:44.814 10470 22453 22658 E AndroidRuntime: Process: com.yamareco.memo, PID: 22453
06-25 19:56:44.814 10470 22453 22658 E AndroidRuntime: java.lang.RuntimeException: An error occurred while executing doInBackground()
06-25 19:56:44.814 10470 22453 22658 E AndroidRuntime: 	at android.os.AsyncTask$4.done(AsyncTask.java:415)
06-25 19:56:44.814 10470 22453 22658 E AndroidRuntime: 	at java.util.concurrent.FutureTask.finishCompletion(FutureTask.java:381)
06-25 19:56:44.814 10470 22453 22658 E AndroidRuntime: 	at java.util.concurrent.FutureTask.setException(FutureTask.java:250)
06-25 19:56:44.814 10470 22453 22658 E AndroidRuntime: 	at java.util.concurrent.FutureTask.run(FutureTask.java:269)
06-25 19:56:44.814 10470 22453 22658 E AndroidRuntime: 	at android.os.AsyncTask$SerialExecutor$1.run(AsyncTask.java:305)
06-25 19:56:44.814 10470 22453 22658 E AndroidRuntime: 	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
06-25 19:56:44.814 10470 22453 22658 E AndroidRuntime: 	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:644)
06-25 19:56:44.814 10470 22453 22658 E AndroidRuntime: 	at java.lang.Thread.run(Thread.java:1012)
06-25 19:56:44.814 10470 22453 22658 E AndroidRuntime: Caused by: java.lang.RuntimeException: application or mock missing database source
06-25 19:56:44.814 10470 22453 22658 E AndroidRuntime: 	at com.yamareco.memo.App.getDatabaseSource(App.java:56)
06-25 19:56:44.814 10470 22453 22658 E AndroidRuntime: 	at com.yamareco.memo.model.helpers.DatabaseManager.<init>(DatabaseManager.java:33)
06-25 19:56:44.814 10470 22453 22658 E AndroidRuntime: 	at com.yamareco.memo.io.DataManager.init(DataManager.java:139)
06-25 19:56:44.814 10470 22453 22658 E AndroidRuntime: 	at com.yamareco.memo.util.InitialHelper$InitAsyncTask.doInBackground(InitialHelper.java:103)
06-25 19:56:44.814 10470 22453 22658 E AndroidRuntime: 	at com.yamareco.memo.util.InitialHelper$InitAsyncTask.doInBackground(InitialHelper.java:81)
06-25 19:56:44.814 10470 22453 22658 E AndroidRuntime: 	at android.os.AsyncTask$3.call(AsyncTask.java:394)
06-25 19:56:44.814 10470 22453 22658 E AndroidRuntime: 	at java.util.concurrent.FutureTask.run(FutureTask.java:264)
06-25 19:56:44.814 10470 22453 22658 E AndroidRuntime: 	... 4 more
06-25 19:56:44.821  1000  1811  4686 W OplusEapManager handleErrorInfo sendEvent: : com.yamareco.memo Crashed
06-25 19:56:44.824  1000  1811 22662 I DropBoxManagerService: add tag=data_app_crash isTagEnabled=true flags=0x2
06-25 19:56:44.826 10470 22453 22453 I Quality : ActivityThread: activityStart delay 373 com.yamareco.memo 22453
06-25 19:56:44.831  1000  1811 22661 W OplusSelfProtectManager: pkg[com.oplus.crashbox] not in policy list
06-25 19:56:44.832  1000  1811  4686 W ActivityTaskManager:   Force finishing activity com.yamareco.memo/.ui.StartViewActivity
06-25 19:56:44.833  1000  1811 22662 D DropBoxManagerService: file :: /data/system/dropbox/data_app_crash@1750849004825.txt
06-25 19:56:44.834  1000  1811  4686 D ActivityTaskManager: onWindowFocusChanged Task{99ce6d2 #6 type=home I=com.android.launcher/.Launcher U=0 rootTaskId=1 visible=true visibleRequested=false mode=fullscreen translucent=true sz=1} would transfer to compact
```

# さいごに

わりとヤマレコ使っているので不具合修正して欲しいなあ（懇願）
