# Kagemiru (設計ドキュメント)

複数のWordPressサイトのセキュリティ状態(CloudSecure WP Security等)と死活状態を
一元的に可視化するダッシュボードシステムの、全体設計・アーキテクチャ決定事項を
管理するリポジトリ。実装コードはここには含まない。

## 関連リポジトリ

- [kagemiru-host](https://github.com/J-KEI/kagemiru-host) — ダッシュボード側(ホスト)に
  インストールするプラグイン。各サイトからのレポートを受信するREST APIと、
  死活監視のPullジョブを持つ。
- [kagemiru-client](https://github.com/J-KEI/kagemiru-client) — 監視対象の各WordPress
  サイトにインストールするプラグイン。導入済みのセキュリティプラグインを検出し、
  アダプタ経由で標準フォーマットに変換してホストへPush送信する。

## ドキュメント運用

- `spec/` — **確定した**プロトコル仕様(SSOT)。host/client双方が従うべき
  通信方向・認証方式・レポートデータスキーマ等を管理する
- `docs/` — **検討中**の設計課題。Issueでの検討過程で生まれた資料を置き、
  結論が出たらPRで`spec/`へ反映(昇格)する

プラグイン単体の実装詳細(DBスキーマ・管理画面仕様・アダプタ実装等)は
各リポジトリ([kagemiru-host](https://github.com/J-KEI/kagemiru-host)
/ [kagemiru-client](https://github.com/J-KEI/kagemiru-client))の
`spec/` `docs/` を参照。

- `spec/architecture.md` — 通信方向・認証方式・相互接続性の位置づけ
- `spec/data-schema.md` — クライアント→ホスト間のレポートデータスキーマ

## アーキテクチャ概要

- セキュリティ設定・ログイン履歴等は **クライアント→ホストへのPush型**
  (`wp_remote_post`)。CloudSecure WP Security等がREST APIを無効化する場合が
  あるため、ホスト側の受信REST APIに影響されない構成。
- 死活監視のみ **ホスト→サイトへのPull型**。サイトが完全にダウンしている場合
  Push側は動作しないため、外形監視は別ロジックとして分離する。
- kagemiru-host / kagemiru-client は独立したプロダクトであり、本プロトコルに
  準拠していれば互いに他実装と組み合わせることも妨げない。「Kagemiru」として
  相互接続性を保証するのはこの2製品の組み合わせのみ。

詳細は `spec/architecture.md` と `spec/data-schema.md` を参照。
