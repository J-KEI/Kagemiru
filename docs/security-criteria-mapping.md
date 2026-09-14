# レポートデータ項目とCIS Controls v8 (IG1) の対応表

関連Issue: [Kagemiru#3](https://github.com/J-KEI/Kagemiru/issues/3) —
レポートデータの収集項目・粒度を、セキュリティ対応状況の可視化目的に照らして
再評価する

## 目的

[spec/data-schema.md](../spec/data-schema.md) の現行スキーマ
(`login` / `updates` / `settings` / `server_errors_1h`) は、CloudSecure WP
Securityアダプタの実装を叩き台に決まった項目である。本ドキュメントでは、
信頼ある組織が提唱する基準と突き合わせることで、CloudSecure固有の実装から
どの程度汎用化できているか、また項目の過不足を客観的に検証する。

**位置づけの注意**: 本対応表はあくまで「物差し」であり、CIS Controlsに
沿っていること自体を採否の決定基準とはしない。最終判断は Issue #3 の既存の
検討観点(可視化目的への貢献度)を優先する。

## 参照基準

- **CIS Controls v8** / Implementation Group 1 (IG1)
- IG1は「あらゆる組織が実施すべき基本的なサイバーハイジーン」と位置づけられる
  56個のセーフガードで構成され、IT・セキュリティ専門知識が限られた中小規模
  組織を主な対象としている。WordPressサイト運営者という本プロジェクトの
  想定利用者層と親和性が高いため、IG1を採用する。
- 出典: [CIS Critical Security Controls Implementation Group 1](https://www.cisecurity.org/controls/implementation-groups/ig1)
- 本表のセーフガードID・タイトルは二次情報源([Bastion社によるCIS Controls v8まとめ](https://bastion.tech/learn/cis-controls/cis-controls-v8-list/))を参照して作成した。個々の文言は公式ドキュメント
  (CIS本体、要ライセンス同意の上でのダウンロード)で最終確認していないため、
  spec昇格の判断時には公式資料での裏取りを推奨する。

## 対応表: 現行スキーマ → IG1セーフガード

| 現行スキーマ項目 | 対応するIG1セーフガード | 対応度 | 備考 |
|---|---|---|---|
| `login.fail_count_24h` / `login.recent` | 8.2 Collect Audit Logs | ◎ | ログイン試行ログの収集そのもの |
| `updates.core` | 7.3 Perform Automated Operating System Patch Management | ○ | WordPress coreをOS/基盤相当として解釈 |
| `updates.plugins` | 7.4 Perform Automated Application Patch Management | ◎ | アプリケーション(プラグイン)のパッチ管理に直接対応 |
| `settings.two_factor` | 6.3 Require MFA for Externally-Exposed Applications / 6.5 Require MFA for Administrative Access | ◎ | MFA要件に直接対応 |
| `settings.simple_waf` | (直接対応なし) | △ | Control 13(Network Monitoring and Defense)はIG2以降。強いて言えば4.4(サーバーへのファイアウォール実装)が近いが、WAFはアプリケーション層でありネットワークFWとは性質が異なる |
| `settings.login_url_change` | (直接対応なし) | △ | 「隠蔽によるセキュリティ」であり、CIS Controlsが重視する標準的な統制には該当しない。強いて挙げれば4.7(デフォルトアカウント管理)の考え方に近いが別物 |
| `settings.rest_api_disabled` | (直接対応なし) | △ | Control 4(Secure Configuration)全体の「不要な機能を無効化する」という趣旨には沿うが、IG1の個別セーフガードとしての明確な対応はない |
| `server_errors_1h` | (対象外) | ー | セキュリティ統制ではなく可用性・運用指標。CIS Controlsのスコープ外 |

## IG1にあるが現行スキーマにない項目(ギャップ候補)

Issue #3 本文で例示されている項目とも一致する部分が多い。

| IG1セーフガード | 内容 | Issue #3内の例示との対応 |
|---|---|---|
| 5.1 Establish and Maintain an Inventory of Accounts | アカウント一覧の把握 | (新規) |
| 5.3 Disable Dormant Accounts | 休眠アカウントの無効化 | (新規) |
| 5.4 Restrict Administrator Privileges to Dedicated Administrator Accounts | 管理者権限の分離 | (新規) |
| 4.7 Manage Default Accounts on Enterprise Assets and Software | デフォルトアカウント(`admin`ユーザー名等)の管理 | (新規、`login_url_change`とは別軸) |
| 7.1/7.2 Vulnerability Management Process | 脆弱性管理プロセス(スキャン結果等) | 「脆弱性スキャン結果」 |
| 10.1 Deploy and Maintain Anti-Malware Software | マルウェア対策ソフトの導入・検知結果 | 「マルウェア検知」 |

## 現行スキーマにあるがIG1に直接対応しない項目

`settings.login_url_change` / `settings.simple_waf` / `server_errors_1h` は、
CIS Controlsの標準的な統制というよりWordPress特有の対策や運用指標であり、
外部基準に照らすと独自色が強い。これは直ちに「不要」を意味するものではなく、
「可視化目的への貢献度」という既存の判断基準で別途評価する必要がある、という
確認に留める。

## 所感

- 現行スキーマは IG1 の中でも「アクセス制御(Control 6)」
  「継続的な脆弱性管理(Control 7)」「監査ログ管理(Control 8)」の3領域を
  部分的にカバーしている。
- 一方、IG1が重視する「アカウント管理(Control 5)」
  「マルウェア対策(Control 10)」領域は現行スキーマではほぼ手薄。
- `simple_waf` の実行結果(ブロック件数等)まで踏み込むと、それは
  Control 13(Network Monitoring and Defense)相当でありIG1の範囲を超える
  ため、収集する場合はIG2相当の踏み込みであると認識した上で判断する必要が
  ある。

## 次のアクション案(Issue #3側で検討)

- ギャップ候補のうち、CloudSecure/Wordfence等の既存アダプタから実際に取得
  可能なものを洗い出す(host/client双方の実装可否は別途確認)。
- 優先度の高い候補: アカウント一覧・休眠管理者アカウントの検知、
  マルウェアスキャン結果。
- `settings.login_url_change` 等の独自項目は残すか整理するか、Issue #3で
  改めて議論する。
