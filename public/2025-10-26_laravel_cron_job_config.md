---
title: Laravel 12 x Ubuntu 24 で非同期ジョブを実装してみた
tags:
  - PHP
  - MySQL
  - Ubuntu
  - cron
  - Laravel
private: false
updated_at: '2025-10-26T19:00:46+09:00'
id: 2b2652905f58e6a7a8d3
organization_url_name: null
slide: false
ignorePublish: false
---
# これなに

今回は Laravel 12 x Ubuntu 24 で非同期ジョブを実行するための手順をまとめました
初めて実装したので色々と過不足や誤りがある可能性もありますので、もしあればコメントなどで教えてください

# 前提条件

- Ubuntu 24.04
- MySQL 8.0.43
- Laravel 12.34.0

なお、環境情報の確認結果は以下の通りです

```zsh
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 24.04 LTS
Release:	24.04
Codename:	noble
$
```

```zsh
$ php artisan --version
Laravel Framework 12.34.0
$
```

```zsh
$ mysql --version
mysql  Ver 8.0.43-0ubuntu0.24.04.1 for Linux on x86_64 ((Ubuntu)
$
```

# cron とは

スケジュールされたタスクを指定日時に自動で実行するためのサービス。
cronを使ってジョブをキューに投入したり、スケジュールされたタスクを実行することができます。


# 実装内容およびコマンド紹介

## 1: cron ジョブ実装

まずは cron ジョブを実行するためのコードを作成します。
今回編集するファイルは以下の通りです。

- `resources/views/emails/test-notification.blade.php`
- `app/Mail/TestEmailNotification.php`
- `app/Services/TestEmailService.php`
- `app/Jobs/SendTestEmailJob.php`
- `app/Console/Commands/SendTestEmailCommand.php`
- `bootstrap/app.php`


### メール送信機能の実装

`resources/views/emails/test-notification.blade.php` というファイルを作成して、メール本文を定義します。

Blade に関しては本記事では割愛しますが、PHPコードとHTMLを混在させて実装できるツールで、Laravel 界隈では頻繁に使われるものです。（詳しい内容については[Laravel Blade の開発者リファレンス](https://laravel.com/docs/12.x/blade)を確認してください）

```php
This is a test email!
```

次に `php artisan make:mail TestEmailNotification` を実行して `app/Mail/TestEmailNotification.php` を作成し、以下のようにメール送信機能を実装します。

```php
<?php

namespace App\Mail;

use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Address;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Mail\Mailables\Envelope;
use Illuminate\Queue\SerializesModels;

class TestEmailNotification extends Mailable
{
    use Queueable, SerializesModels;

    /**
     * Create a new message instance.
     */
    public function __construct()
    {
        //
    }

    /**
     * Get the message envelope.
     */
    public function envelope(): Envelope
    {
        return new Envelope(
            to: [new Address('su3.hokkaido+main@sample.co.jp')],
            cc: [new Address('su3.hokkaido+sub@sample.co.jp')],
            subject: 'Test email notification',
        );
    }

    /**
     * Get the message content definition.
     */
    public function content(): Content
    {
        return new Content(
            text: 'emails.test-notification',
        );
    }

    /**
     * Get the attachments for the message.
     *
     * @return array<int, \Illuminate\Mail\Mailables\Attachment>
     */
    public function attachments(): array
    {
        return [];
    }
}
```

続けて、 `app/Services/TestEmailService.php` に上記のメール送信メソッドを実行するためのサービスを作成します。

```php
<?php

namespace App\Services;

use App\Mail\TestEmailNotification;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Facades\Mail;

class TestEmailService
{
    /**
     * Send test email notification.
     *
     * @return array
     */
    public function sendTestEmail(): array
    {
        Log::info('TestEmailService: Starting test email sending');

        try {
            Mail::send(new TestEmailNotification());

            Log::info('TestEmailService: Test email sent successfully');

            return [
                'success' => true,
                'message' => 'Test email sent successfully',
            ];
        } catch (\Exception $e) {
            Log::error('TestEmailService: Failed to send test email', [
                'error' => $e->getMessage(),
                'trace' => $e->getTraceAsString(),
            ]);

            return [
                'success' => false,
                'message' => 'Failed to send test email',
                'error' => $e->getMessage(),
            ];
        }
    }
}
```

### ジョブの実装

`php artisan make:job SendTestEmailJob` を実行して、作成された `app/Jobs/SendTestEmailJob.php` ファイルを以下のように編集します。

```php
<?php

namespace App\Jobs;

use App\Services\TestEmailService;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;
use Illuminate\Support\Facades\Log;

class SendTestEmailJob implements ShouldQueue
{
    use Queueable;

    /**
     * The number of times the job may be attempted.
     *
     * @var int
     */
    public $tries = 3;

    /**
     * The number of seconds the job can run before timing out.
     *
     * @var int
     */
    public $timeout = 300; // 5 minutes

    /**
     * Create a new job instance.
     */
    public function __construct()
    {
        //
    }

    /**
     * Execute the job.
     */
    public function handle(TestEmailService $emailService): void
    {
        Log::info('SendTestEmailJob started');

        $result = $emailService->sendTestEmail();

        if ($result['success']) {
            Log::info('SendTestEmailJob completed successfully', $result);
        } else {
            Log::error('SendTestEmailJob completed with errors', $result);
            throw new \Exception($result['error'] ?? 'Failed to send test email');
        }
    }

    /**
     * Handle a job failure.
     */
    public function failed(\Throwable $exception): void
    {
        Log::error('SendTestEmailJob failed permanently', [
            'error' => $exception->getMessage(),
            'trace' => $exception->getTraceAsString(),
        ]);
    }
}
```

`php artisan make:command SendTestEmailCommand` を実行して、ジョブを実行するための Command ファイル `app/Console/Commands/SendTestEmailCommand.php` を以下の通り実装します。

```php
<?php

namespace App\Console\Commands;

use App\Jobs\SendTestEmailJob;
use Illuminate\Console\Command;
use Illuminate\Support\Facades\Log;

class SendTestEmailCommand extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send-test';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Send test email notification';

    /**
     * Execute the console command.
     */
    public function handle()
    {
        $this->info('Dispatching test email job...');
        Log::info('Test email job dispatched via command');

        SendTestEmailJob::dispatch();

        $this->info('Test email job has been dispatched to the queue.');

        return Command::SUCCESS;
    }
}
```

### スケジュール設定

最後にスケジュール設定を行うために `bootstrap/app.php` に以下のコードを追記します。

簡単な設定内容としては毎日、日本時間の朝8時に実行されるようにしています。
実行ログは `logs/dailyjob-dailynotification.log` に保存するようにしています。

```php
use Illuminate\Console\Scheduling\Schedule;

...

return Application::configure(basePath: dirname(__DIR__))

...

// Add the following codes within `Application::configure(basePath: dirname(__DIR__))`
    ->withSchedule(function (Schedule $schedule): void {
        // Test email schedule - daily at 8:00 AM JST
        // Sends test email notification
        $schedule->command('email:send-test')
            ->dailyAt('08:00')
            ->timezone('Asia/Tokyo')
            ->appendOutputTo(storage_path('logs/test-email.log'))
            ->onFailure(function () {
                \Illuminate\Support\Facades\Log::error('Test email failed to send');
            })
            ->onSuccess(function () {
                \Illuminate\Support\Facades\Log::info('Test email sent successfully');
            });    })
```

## 2: cron セットアップ
### cron インストール

ubuntu に入って以下のコマンドで cron をインストールします。
※ 以降、各コマンドで sudo で操作していますが、カレントユーザーにインストール等の操作権限があれば sudo は付加する必要はありません。

```zsh
sudo apt-get install cron
```

次に、以下のコマンドでインストールした cron を有効化します

```zsh
sudo systemctl enable cron
```

続けて、以下のコマンドで cron を起動します

```zsh
sudo systemctl start cron
```

### cron のステータス確認

任意ですが、以下のコマンドでインストール先を確認できます。
インストールできていない場合は前のステップを実行してください。

```zsh
which cron
```

```zsh
###############
# Output example is as follows
###############
/usr/sbin/cron
```

以下のコマンドで cron の起動ステータス等を確認します。

```zsh
sudo systemctl status cron
```

```zsh
###############
# Output example is as follows
###############
● cron.service - Regular background program processing daemon
    Loaded: loaded (/usr/lib/systemd/system/cron.service; enabled; preset: enabled)
    Active: active (running) since Sun 2025-10-26 14:33:27 JST; 24min ago
      Docs: man:cron(8)
  Main PID: 1140249 (cron)
     Tasks: 1 (limit: 2314)
    Memory: 352.0K (peak: 31.4M)
       CPU: 6.073s
    CGroup: /system.slice/cron.service
            └─1140249 /usr/sbin/cron -f -
```

### Cron マスタスケジューラー設定

crontab を編集するため、以下のコマンドで crontab の編集を行います。

```zsh
crontab -e
```

crontab を開いたら、以下の1行を任意の箇所に追記してください。
これは該当のプロジェクトを毎分チェックして、スケジュールされている cron ジョブがあれば実行するような設定です。
追記したら、 `wq!` で保存して編集モードを終了してください。

```zsh
* * * * * cd [project root path] && php artisan schedule:run >> /dev/null 2>&1
```

crontab の終了が終わったら、以下のコマンドを実行して設定情報を確認しましょう

```zsh
crontab -l
```

```zsh
###############
# Output example is as follows
###############
# Edit this file to introduce tasks to be run by cron.
# 
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
# 
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').
# 
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
# 
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
# 
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
# 
# For more information see the manual pages of crontab(5) and cron(8)
# 
# m h  dom mon dow   command
* * * * * cd /var/www/sample_laravel_app && php artisan schedule:run >> /dev/null 2>&1
```

## 3: Cron 実行状態等の確認
### ログの確認

以下のコマンドで CRON でフィルターしたリアルタイムの syslog を出力します。

```zsh
sudo tail -f /var/log/syslog | grep CRON
```

### ジョブ一覧確認

プロジェクトディレクトリに移動して、以下のコマンドでスケジュールされているジョブを確認できます。

```zsh
php artisan schedule:list
```

```zsh
###############
# Output example is as follows
###############
su3-hokkaido@sample_laravel_app_test:/var/www/sample_laravel_app$ php artisan schedule:list

  0 8 * * *    php artisan email:send-test ................................ Next Due: 12 hours later

su3-hokkaido@sample_laravel_app_test:/var/www/sample_laravel_app$ 
```

## 4: 補足
### ジョブの即時実行

以下のコマンドで実行できます。

```zsh
php artisan [Command]
```

前述のサンプルジョブを実行する場合は以下のようなコマンドを実行します。

```zsh
php artisan email:send-test
```

### キューテーブルの作成

非同期である cron ジョブを実行するにはキューイングするためのテーブルが必要です。
`jobs` テーブルが存在しない場合は以下のコマンドで `jobs` テーブルを作成してください。
これによりキューイングされたジョブを管理することができるようになります。

```zsh
php artisan queue:table
php artisan migrate
```

### ワーカーの起動

以下のコマンドを実行すると、キューに入っているジョブを実行できるようになります。
ワーカーはキューからエンキューしてジョブを実行するためのものです。

```zsh
php artisan queue:work
```

### 失敗したジョブの確認

以下のコマンドを実行すると `failed_jobs` テーブルに格納されている失敗したジョブを確認できます。

```zsh
php artisan queue:failed
```

なお、 `failed_jobs` テーブルが存在しない場合は以下のコマンドでテーブルを作成してください。

```zsh
php artisan queue:failed-table
php artisan migrate
```

### 失敗したジョブの再実行

先ほどの失敗したジョブの確認後、必要があれば以下のコマンドで失敗したジョブを再実行できます。
`${id}` には失敗したジョブのIDを指定してください。

```zsh
php artisan queue:retry ${id}
```

### 失敗したジョブの削除

失敗したジョブを削除したい場合は以下のコマンドを実行してください。
前セクションの説明同様、`${id}` には失敗したジョブのIDを指定してください。

```zsh
php artisan queue:forget ${id}
```

なお、すべての失敗したジョブを削除する場合は以下のコマンドで削除できます。

```zsh
php artisan queue:flush
```

### cron サービスの起動、停止、再起動、ステータス確認

以下のコマンドは上から順に起動、停止、再起動、ステータス確認です。
前述の実装セクションで出てきているコマンドがいくつかありますが、改めてこちらのセクションにもコマンドをまとめてました。

```zsh
sudo systemctl start cron
```

```zsh
sudo systemctl stop cron
```

```zsh
sudo systemctl restart cron
```

```zsh
sudo systemctl status cron
```

# まとめ

今回は Laravel 12 x Ubuntu 24 での cron の実装方法について紹介しました。
キーワードとしては以下の5つ、これらを活用して快適な非同期ジョブライフを！

- job (ジョブ): 非同期実行するサービス
- queue (キュー): 実行するジョブのキューイングリスト、`jobs` テーブルで管理
- worker (ワーカー): キューからジョブを取り出して実行するためのサービス
- failed_jobs: 失敗したジョブのリスト、`failed_jobs` テーブルで管理
- cron: スケジュールタスクを自動実行するためのサービス
