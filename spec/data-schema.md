# レポートデータスキーマ(プロトコル仕様)

クライアント実装がホストへPOSTする標準JSONフォーマット。本プロトコルに
準拠する送信であれば、送信元の実装(アダプタの有無等)は問わない。

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
| `adapter` | string | データを生成したアダプタ(実装)名。`cloudsecure` / `siteguard` 等。送信元実装を識別する任意の文字列 |
| `login.recent` | array | 直近のログイン試行。件数は送信間隔に応じて絞る |
| `updates.core` | object\|null | 更新不要な場合は `null` |
| `settings.*` | boolean | セキュリティ機能のON/OFF。送信元によってキーは異なってよいが、可能な限り共通キー名(上記4種)に正規化する |

## リクエストヘッダー

| ヘッダー | 内容 |
|---|---|
| `X-Site-Id` | ホストが発行したサイト識別子 |
| `X-Signature` | `hash_hmac('sha256', <raw body>, <連携キー>)` |

死活監視(Pull)側の記録フォーマットはプロトコル対象外(クライアントは
関与しない、ホスト単体の内部仕様)。[kagemiru-host/spec](https://github.com/J-KEI/kagemiru-host/tree/main/spec)
を参照。
