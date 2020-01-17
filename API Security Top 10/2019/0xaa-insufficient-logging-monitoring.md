API10:2019 Insufficient Logging & Monitoring (不十分なロギングとモニタリング)
============================================

| 脅威エージェント/攻撃経路 | セキュリティ上の弱点 | 影響 |
| - | - | - |
| API 依存 : 攻撃難易度 **2** | 蔓延度 **3** : 検出難易度 **1** | 技術的影響 **2** : ビジネス依存 |
| 攻撃者はロギングやモニタリングの欠陥を活用し、警告を出すことなくシステムを悪用する。 | ロギングやモニタリングを行っていない場合、もしくはロギングやモニタリングが不十分な場合、怪しい行動をトラッキングし、タイムリーに反応することはほとんど不可能である。 | 現在進行中の悪意ある行動の可視性がないのであれば、攻撃者には完全にシステムを侵害するための十分な時間がある |

## API が脆弱かどうかの確認

以下の場合、API は脆弱である。

* 少しもログが生成されていない、ロギングレベルが適切に設定されていない、もしくはログメッセージに十分な詳細が含まれていない。
* ログの完全性が保証されていない (たとえば、[Log Injection][1])
* ログが継続的にモニタリングされていない
* API 基盤が継続的にモニタリングされていない。

## 攻撃シナリオ例

### シナリオ #1

管理 API のアクセスキーが、公開リポジトリに漏えいした。そのリポジトリオーナーは、漏えいした可能性について E-mail で知らされたが、そのインシデントの対応に 48 時間以上かかった。そして、アクセスキーの公開によって機微データへのアクセスを許した可能性がある。不十分なロギングが原因で、この企業は悪意ある者が何のデータにアクセスしたかを評価することができない。

### シナリオ #2

ビデオ共有プラットフォームは、"大規模な" クレデンシャルスタッフィング攻撃に襲われた。ログインの失敗がログに記録されているにもかかわらず、攻撃の期間中、アラートはトリガーされなかった。ユーザの苦情に対する反応として、API ログは解析され、攻撃者は見つかった。この企業は、ユーザに対してパスワードをリセットする要求を公表し、規制当局にインシデントの報告をしなければならなかった

## 対策方法

* 認証施行の失敗、アクセス拒否、検証エラーがすべてログに記録されている。
* ログは、ログ管理ソリューションによって使われるのに適した形式で記録されており、悪意ある者を特定するために十分な詳細が含まれている。
* ログは、機微データとして処理され、停止や移行時にそれらの完全性は保証されている。
* モニタリングシステムを構成して、インフラ、ネットワーク、API 機能を継続的にモニタリングする。
* セキュリティ情報やイベント管理 (SIEM) システムを使用して、API スタックやホストの全コンポーネントからログを収集して管理する
* カスタムダッシュボードとアラートを構成し、怪しいアクティビティを検出して、より迅速に対応できるようにする。

## References

### OWASP

* [OWASP Logging Cheat Sheet][2]
* [OWASP Proactive Controls: Implement Logging and Intrusion Detection][3]
* [OWASP Application Security Verification Standard: V7: Error Handling and
  Logging Verification Requirements][4]

### External

* [CWE-223: Omission of Security-relevant Information][5]
* [CWE-778: Insufficient Logging][6]

[1]: https://www.owasp.org/index.php/Log_Injection
[2]: https://www.owasp.org/index.php/Logging_Cheat_Sheet
[3]: https://www.owasp.org/index.php/OWASP_Proactive_Controls
[4]: https://github.com/OWASP/ASVS/blob/master/4.0/en/0x15-V7-Error-Logging.md
[5]: https://cwe.mitre.org/data/definitions/223.html
[6]: https://cwe.mitre.org/data/definitions/778.html
