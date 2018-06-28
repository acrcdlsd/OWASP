# V4: 認証とセッション管理要件

## 制御目標

大抵の場合、リモートサービスにユーザがログインすることは、モバイルアプリのアーキテクチャ全体の不可欠な部分である。たとえ、ほとんどのロジックが末端(エンドポイント)で発生するとしても、MASVSはユーザアカウントとセッションが管理される方法に関するいくつかの基本的な要件を定義する。

## セキュリティ要件

| # | 説明 | L1 | L2 |
| --- | --- | --- | --- |
| **4.1** | アプリがユーザにリモートサービスへのアクセスを提供する場合に、ユーザ名/パスワード認証などのなにかしらの認証がリモートエンドポイントで行われている | ✓ | ✓ |
| **4.2** | ステートフルなセッション管理が使用されている場合に、ユーザ識別子を送信することなくクライアントリクエストを認証するために、リモートエンドポイントはランダムに生成されたセッション識別子を使用する  | ✓ | ✓ |
| **4.3** | ステートレスなトークンベースの認証が使用されている場合に、サーバはセキュアなアルゴリズムを使用して署名されたトークンを提供する | ✓ | ✓ |
| **4.4** | ユーザのログアウト時に、リモートサービスが既存のセッションを破棄する | ✓ | ✓ |
| **4.5** | パスワードポリシーが存在して、リモートエンドポイントで実施されている | ✓ | ✓ |
| **4.6** | 度を超えた回数の認証情報の提示に対して保護するメカニズムをリモートエンドポイントが実装する | ✓ | ✓ |
| **4.7** | 生体認証がもしあれば、それはイベントバウンド(つまり、「true」や「false」を返す意味のAPIの使用)ではない。代わりに、キーチェインやキーストアをアンロックすることに基づいている |   | ✓ |
| **4.8** | 事前に定義された期間アクティブでない場合、もしくはアクセストークンのセッションが切れた後に、セッションがリモートエンドで破棄される |   | ✓ |
| **4.9** | リモートエンドポイントで認証における第二の要素が存在し、二要素認証(2FA)要件が一貫して試行される  |   | ✓ |
| **4.10** | 機微なトランザクションには、ステップアップ認証が必要である |   | ✓ |
| **4.11** | アプリがアカウントのすべてのログイン動作をユーザに通知する。ユーザは、アカウントへのアクセスや特定デバイスをブロックしたりするのに使用するデバイスのリストを表示できます |  | ✓ |

## 参考文献

OWASP モバイルセキュリティテストガイドは、本セクションに記載されている要件を検証するための詳細な指示を提供する。

- For Android - https://github.com/OWASP/owasp-mstg/blob/master/Document/0x05f-Testing-Authentication.md
- For iOS - https://github.com/OWASP/owasp-mstg/blob/master/Document/0x06f-Testing-Authentication-and-Session-Management.md

詳細は以下参照：

- OWASP Mobile Top 10: [M4 - Insecure Authentication(安全でない認証)](https://www.owasp.org/index.php/Mobile_Top_10_2016-M4-Insecure_Authentication), [M6 - Insecure Authorization(安全でない認可制御)](https://www.owasp.org/index.php/Mobile_Top_10_2016-M6-Insecure_Authorization)
- CWE:  https://cwe.mitre.org/data/definitions/287.html