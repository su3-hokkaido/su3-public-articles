---
title: "Laravel で Microsoft Teams に簡易的なメッセージを通知する手順"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["PHP", "Laravel", "Teams", "Microsoft", "初心者"]
published: true
---
# これなに

- 例のお手伝いしているところで非同期ジョブの実行結果をチャットツールに通知を投げたくなった
- しかし、例のお手伝いしているところが Slack ではなく Teams を使用している
- なので、今回初めて Teams に通知するにはどのようにするのかをまとめてみました

# この記事で出てくる用語説明

- [Teams](https://www.microsoft.com/en-us/microsoft-teams/group-chat-software): Microsoft 社が提供しているチャットツール
- [Slack](https://slack.com/): 世界で最も有名なチャットツールの1つ、2021年に Salesforce 社が Slack を買収したことは有名
- [Laravel](https://laravel.com/): PHP の有名な開発フレームワークの1つ、データベース操作を効率化する Eloquent ORM や テンプレートエンジンの Blade、コマンドラインツール のArtisan などの様々な便利な機能が揃っている
- [Ubuntu](https://ubuntu.com/): 無料で使える Linux 系のオペレーティングシステム、日本語サポートも充実していてとても便利
- Webhook: 特定のイベントが発生した際に、自動的に別のアプリケーションへHTTPリクエストで通知を送信する仕組みのこと

# 公式リファレンス

Microsoft から以下の公式リファレンスが公開されているので、詳細は以下のページから確認ください。

https://learn.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook?tabs=newteams%2Cdotnet

# 前提条件

- Ubuntu 24.04
- Laravel 12.34.0

環境情報の確認結果は以下の通りです

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

# 実装手順
### Teams 側で Incoming Webhook アプリを追加する

通知を受信したいチャンネルの右側の3点リーダーから `Manage channel` をクリックしてください。

![Access Manage channel](/images/2025-11-10_laravel_teams_01.png)

Connectors の Edit ボタンをクリックします。

![Edit connectors](/images/2025-11-10_laravel_teams_02.png)

Incoming Webhook を追加します。

![Add Incoming Webhook](/images/2025-11-10_laravel_teams_03.png)

ポチポチ設定を進めていくと、通知アプリの名称、アイコンを設定する画面が出てくるので、こちらで任意の名称、アイコンを設定しましょう。
最後に Webhook URL が発行されるので、それをクリップボードにコピーなりで控えてください。

![Copy issued Webhook URL](/images/2025-11-10_laravel_teams_04.png)

## Laravel 側で通知用の機能を実装する

Webhook URL をセットして、POSTメソッドを実行することで Team に通知することができます。
以下のサンプルコードは通知結果を boolean 型（true または false）で返却するメソッドを以下のように実装しています。

```php
public function sentTeamsNotification(): bool {
  try {
    $webhookUrl = 'https://hogehoge'; // ここに発行した Webhook URL をセットする
    $message = 'Test message'; // 任意のメッセージ内容を設定する
    $response = Http::post($webhookUrl, [
        'text' => $message,
    ]);

    if ($response->successful()) {
        Log::info('Teams notification sent successfully');

        return true;
    } else {
        Log::error('Failed to send Teams notification');
        return false;
    }
  } catch (\Exception $e) {
    Log::error('Exception while sending Teams notification', [
        'error' => $e->getMessage(),
        'trace' => $e->getTraceAsString(),
    ]);
    return false;
  }
}
```

## 通知メッセージテスト

実行すると以下のようにメッセージが表示されました！

![Result of notification](/images/2025-11-10_laravel_teams_05.png)


# 最後に

本記事では Laravel で Teams にメッセージを通知する簡単な実装方法について記載しました。

他にもエンコーディングの指定やカードコンポーネントの指定などがありますが、今回の目的はまず簡易的なメッセージ通知だけだったので記載しておりません。
詳しくは上述にも掲載しました [Microsoft 社提供の公式ガイド](https://learn.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook?tabs=newteams%2Cdotnet)をご覧ください。
