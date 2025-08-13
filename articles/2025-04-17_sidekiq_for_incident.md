---
title: "障害対応に使える Sidekiq ジョブの調べ方" # 記事のタイトル
emoji: "🔍" # Choose an appropriate emoji for each article
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Redis", "sidekiq", "ElastiCache", "障害調査", "障害対応"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
# これなに

その昔、仕事で sidekiq を使っていたのですが、障害で死んだ sidekiq のジョブをポチポチ調べるのがしんどかったので、調べた中でいろいろわかったことをメモ。

基本的には個人的な雑記なので、読みやすさや正確性云々は考えてません。悪しからず。
あとフレッシュで正確な情報は [sidekiq の wiki ページ](https://github.com/sidekiq/sidekiq/wiki)を見るのが良い。

※ [2023年10月に note に書いた記事](https://note.com/su3_hokkaido/n/nd0734d1b3ff2)を移行しただけなのですが、基本的な仕様は変わっていないはず。もし細かいところが変わっている場合はご容赦ください。

# 便利な一覧表

- `Sidekiq::DeadSet.new` : 失敗
- `Sidekiq::Workers.new` : 実行中
- `Sidekiq::Queue.new` : 待機中
- `Sidekiq::ScheduledSet.new` : 実行予約（あんま使い所ないので割愛）
- `Sidekiq::RetrySet.new` : リトライ（同上）

# 失敗したジョブの情報取得する

### 失敗したジョブをすべて取得する

`class: CronJob::AbcJob, job_id: fd02749d-29d7-4a1c-938d-7bbbc11ba353, enqueued_at: 2023-10-22T03:05:04Z` 的な感じで書くことでリストできる

```ruby
dead_jobs = Sidekiq::DeadSet.new
dead_jobs.map {|job| "class: #{job["args"][0]["job_class"]}, job_id: #{job["args"][0]["job_id"]}, enqueued_at: #{job["args"][0]["enqueued_at"]}"}
```

### 特定の失敗ジョブの情報を取得する

時間を指定して、dead_abc_jobs.first/last/length とかも使える

```ruby
dead_jobs = Sidekiq::DeadSet.new
dead_abc_jobs = dead_jobs.scan("AbcJob").select{|job| job.args[0]["enqueued_at"].to_time > Date.new(2020,1,12)}
```

# 実行中のジョブの情報を取得する

### 実行中のジョブをすべて取得する

とりあえず全部出す

```ruby
running_jobs = Sidekiq::Workers.new
running_jobs.each {|job| job}
```

# 待機中のジョブの情報を取得する

### 待機中のジョブを全て取得する

とりあえず全部出す

```ruby
queuing_jobs = Sidekiq::Queue.new
queuing_jobs.each
```
