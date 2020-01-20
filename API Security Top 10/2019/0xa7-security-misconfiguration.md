API7:2019 Security Misconfiguration - セキュリティ設定ミス
===================================

| 脅威エージェント/攻撃経路 | セキュリティ上の弱点 | 影響 |
| - | - | - |
| API 依存 : 攻撃難易度 **3** | 蔓延度 **3** : 検出難易度 **3** | 技術的影響 **2** : ビジネス依存 |
| 攻撃者は、しばしば、パッチが適用されていない欠陥、一般的なエンドポイント、保護されていないファイルとディレクトリを見つけ、不正アクセスやシステムの情報を勝ち取ろうと試みる。 | セキュリティ設定ミスは、ネットワークレベルからアプリケーションレベルまで、API スタックのどのレベルでも起こる可能性がある。自動化ツールは、不要なサービスやレガシーオプションなどの設定ミスの検出や悪用に利用することができる。 | セキュリティ設定のミスは、機微なユーザデータを公開するだけではなく、完全なサーバ侵害につながる恐れのあるシステム詳細を公開する可能性もある。 |

## API が脆弱かどうかの確認

以下の場合、API は脆弱である可能性がある。

* 適切なセキュリティハードニングが、アプリケーションスタックのあらゆる部分でかけている、もしくはクラウドサービスで不適切に設定されたパーミッションがある場合
* 最新のセキュリティパッチが適用されていない、もしくはシステムが古い。
* 不要な機能が有効化されている (たとえば、HTTP メソッド)。
* トランスポート層セキュリティ (TLS) が欠けている。
* セキュリティディレクティブは、クライアントに送信されない (たとえば、[Security Headers][1])。
* クロスオリジンリソース共有 (CORS) ポリシーが欠けている、もしくは不適切に設定されている。
* エラーメッセージにスタックトレースが含まれている、もしくは他の機微情報が公開されている。

## 攻撃シナリオ例

### シナリオ #1

攻撃者は、`.bash_history` ファイルがサーバのルートディレクトリの下にあることに気付く。そしてそのファイルは、DevOps チームが API にアクセスするために使用するコマンドが含まれている。

```
$ curl -X GET 'https://api.server/endpoint/' -H 'authorization: Basic Zm9vOmJhcg=='
```

攻撃者は、DevOps チームによってのみ使用され、ドキュメント化されていない API で新たなエンドポイントを見つけるかもしれない。

### シナリオ #2

特定のサービスをターゲットとするために、攻撃者は一般的な検索エンジンを使用して、インターネットから直接アクセス可能なコンピュータを探し出す。攻撃者は、一般的なデーベース管理システム (DBMS) を実行しており、デフォルトポートをリッスンしているホストを見つけ出した。そのホストはデフォルト設定を使用しており、そのデフォルト設定では認証が無効になっていたため、攻撃者は PII、個人の嗜好、認証データを含む数百万のレコードへのアクセス権を勝ち取った。

### シナリオ #3

モバイルアプリケーションのトラフィックを調査することで、攻撃者は全ての HTTP トラフィックがセキュアなプロトコル (たとえば、TLS) で実装されているわけではないことを突き止める。攻撃者は、特に、プロファイル画像のダウロードに関して、このことが事実であることに気付く。API トラフィックがセキュアプロトコルで実装されている事実にもかかわらず、ユーザインタラクションはバイナリであるため、攻撃者は API レスポンスサイズのパターンを見つける。そしてそれを用いて、レンダリングされたコンテンツ (たとえば、プロフィール画像) に対してユーザの嗜好をトラッキングする。

## 対策方法

API ライフサイクルは以下を含むべきである。

* 適切にロックダウンされた環境の迅速かつ簡単なデプロイにつながる反復可能なハードニングプロセス
* API スタック全体の設定をレビュー、アップデートするためのタスク。レビューには、構成ファイル、API コンポーネント、クラウドサービス (たとえば、S3 バケットのパーミッション) が含まれているべきである。
* 全ての API インタラクションが静的アセット (たとえば、画像) にアクセスするためのセキュアな通信チャネル
* 全ての環境の構成と設定の有効性に継続的にアクセスするための自動化されたプロセス


上記に加えて：
* 例外トレースや他の有益な情報が攻撃者に返信されることを防ぐために、適用できるのであれば、エラーレスポンスを含む全ての API レスポンスペイロードスキーマを定義して実行する。
* API が特定の HTTP メソッドによってのみアクセス可能であることを確認する。他の全ての HTTP メソッド (たとえば、`HEAD`) は無効化されるべきである。
* ブラウザベースのクライアント (たとえば、WebApp フロントエンド) からのアクセスを想定している API は、適切なクロスオリジンリソース共有 (CORS) ポリシーを実装するべきである。


## References

### OWASP

* [OWASP Secure Headers Project][1]
* [OWASP Testing Guide: Configuration Management][2]
* [OWASP Testing Guide: Testing for Error Codes][3]
* [OWASP Testing Guide: Test Cross Origin Resource Sharing][9]

### External

* [CWE-2: Environmental Security Flaws][4]
* [CWE-16: Configuration][5]
* [CWE-388: Error Handling][6]
* [Guide to General Server Security][7], NIST
* [Let's Encrypt: a free, automated, and open Certificate Authority][8]

[1]: https://www.owasp.org/index.php/OWASP_Secure_Headers_Project
[2]: https://www.owasp.org/index.php/Testing_for_configuration_management
[3]: https://www.owasp.org/index.php/Testing_for_Error_Code_(OTG-ERR-001)
[4]: https://cwe.mitre.org/data/definitions/2.html
[5]: https://cwe.mitre.org/data/definitions/16.html
[6]: https://cwe.mitre.org/data/definitions/388.html
[7]: https://csrc.nist.gov/publications/detail/sp/800-123/final
[8]: https://letsencrypt.org/
[9]: https://www.owasp.org/index.php/Test_Cross_Origin_Resource_Sharing_(OTG-CLIENT-007)
