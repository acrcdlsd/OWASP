# V6: プラットフォームの相互作用要件

## 制御目的

本グループ内の制御は、アプリがセキュアな方法でプラットフォームのAPIと標準コンポーネントを使用することを保証する。加えて、本制御はアプリケーション間通信(IPC)を含んでいる。

## セキュリティ検証要件

| # | 説明 | L1 | L2 |
| --- | --- | --- | --- |
| **6.1** | アプリが最低限の必要なパーミッションの集合のみを要求する | ✓ | ✓ |
| **6.2** | 外部ソースやユーザからのすべてのインプットが検証され、必要ならサニタイジングされる。これは、UI、インテントなどのIPCメカニズム、カスタムURL、ネットワークソースを通して受信したデータを含んでいる | ✓ | ✓ |
| **6.3** | これらのメカニズムが適切に保護されていない限り、アプリはカスタムURLスキームを通して機密機能をエクスポートしない | ✓ | ✓ |
| **6.4** | これらのメカニズムが適切に保護されていない限り、アプリがIPC機能を通して機密機能をエクスポートしない | ✓ | ✓ |
| **6.5** | 明示的に必要とされていない限り、WebViewでJavaScriptが無効化されている | ✓ | ✓ |
| **6.6** | WebViewは、必要なプロトコルハンドラの最低限の集合のみを許可するために構成されている(理想的には、HTTPSのみがサポートされている)。file、tel、app-idなどの潜在的に危険なハンドラは無効化される | ✓ | ✓ |
| **6.7** | アプリのネイティブメソッドがWebViewに公開されているなら、WebViewがアプリパッケージの中に含まれているJavaScriptのみをレンダリングすることを検証しなさい | ✓ | ✓ |
| **6.8** | もしあれば、オブジェクトのデシリアライゼーションが安全なシリアライゼーションAPIを使用して実装されている | ✓ | ✓ |

## 参考文献

OWASP モバイルセキュリティテストガイドは、本セクションに記載されている要件を検証するための詳細な指示を提供する。

- Android - https://github.com/OWASP/owasp-mstg/blob/master/Document/0x05h-Testing-Platform-Interaction.md
- iOS - https://github.com/OWASP/owasp-mstg/blob/master/Document/0x06h-Testing-Platform-Interaction.md

詳細は以下参照：

- OWASP Mobile Top 10: M1 - Improper Platform Usage(不適切なプラットフォームの使用)
- CWE: https://cwe.mitre.org/data/definitions/20.html
- CWE: https://cwe.mitre.org/data/definitions/749.html
