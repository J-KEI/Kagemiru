# レポートデータスキーマ

クライアントプラグインがホストへPOSTする標準JSONフォーマット。
アダプタが増えても、この形式に変換して送ることでホスト側の実装は変えない。

```json
{
  "reported_at": "2026-09-06T09:20:00+09:00",
  "adapter": "cloudsecure",
  "login": {
    "fail_count_24h": 14,
    "recent": [
      { "user": "admin", "ip": "203.0.113.9", "result": "fail", "time": "09:12" }
    ]
  },
  "updates": {
    "core": { "from": "6.9", "to": "7.0" },
    "plugins": [
      { "name": "Contact Form 7", "from": "5.8", "to": "5.9.1" }
    ]
  },
  "settings": {
    "login_url_change": true,
    "two_factor": true,
    "simple_waf": false,
    "rest_api_disabled": true
  },
  "server_errors_1h": 0
}
```

## フィールド補足

| フィールド | 型 | 備考 |
|---|---|---|
| `adapter` | string | データを生成したアダプタ名。`cloudsecure` / `siteguard` 等 |
| `login.recent` | array | 直近のログイン試行。件数は送信間隔に応じて絞る |
| `updates.core` | object\|null | 更新不要な場合は `null` |
| `settings.*` | boolean | プラグインの主要機能ON/OFF。アダプタごとにキーは異なってよいが、
  可能な限り共通キー名(上記4種)に正規化する |

## リクエストヘッダー

| ヘッダー | 内容 |
|---|---|
| `X-Site-Id` | ホストが発行したサイト識別子 |
| `X-Signature` | `hash_hmac('sha256', <raw body>, <連携キー>)` |

## 死活監視(Pull)側の記録形式

ホストのDBに保存する形式であり、クライアントからの送信物ではない。

```json
{ "checked_at": "2026-09-06T09:25:00+09:00", "http_status": 200, "response_ms": 340 }
```
