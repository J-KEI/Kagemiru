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

## ドキュメント

- `docs/architecture.md` — アーキテクチャ決定事項(通信方向・認証方式・アダプタ
  パターン等)
- `docs/data-schema.md` — クライアント→ホスト間のレポートデータスキーマ

## アーキテクチャ概要

- セキュリティ設定・ログイン履歴等は **クライアント→ホストへのPush型**
  (`wp_remote_post`)。CloudSecure WP Security等がREST APIを無効化する場合が
  あるため、ホスト側の受信REST APIに影響されない構成。
- 死活監視のみ **ホスト→サイトへのPull型**。サイトが完全にダウンしている場合
  Push側は動作しないため、外形監視は別ロジックとして分離する。
- クライアント側は対応セキュリティプラグインごとに「アダプタ」を実装し、
  対応プラグインを増やす際はアダプタを追加するだけで済む設計。

詳細は `docs/architecture.md` と `docs/data-schema.md` を参照。
