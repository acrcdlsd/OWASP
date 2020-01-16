API4:2019 Lack of Resources & Rate Limiting (リソース不足と帯域制限)
===========================================

| 脅威エージェント/攻撃経路 | セキュリティ上の弱点 | 影響 |
| - | - | - |
| API 依存 : 攻撃難易度 **2** | 蔓延度 **3** : 検出難易度 **3** | 技術的影響 **2** : ビジネス依存 |
| エクスプロイトを行うには、単純な API リクエストが必要である。認証は必要ではない。多数の同時リクエストは、単一のローカルコンピュータから、もしくはクラウドコンピューティングリソースを用いることで実行される。 | 通常、帯域制限が実装されていない API や制限が適切に設定されていない API を見つけ出す。 | エクスプロイトは、DoS の発生や、API が応答しなくなったり、利用できなくなったりすることにつながる。 |


## API が脆弱かどうかの確認

API リクエストは、ネットワーク、CPU、メモリ、ストレージなどのリソースを消費する。リクエストを満たすために要求されるリソースの量は、ユーザの入力やエンドポイントのビジネスロジックに非常に依存している。また、複数の API クライアントからのリクエストが、リソースを奪い合うことを考慮しなければならない。以下の制限の少なくとも1つが設定されていない、もしくは不適切に設定されている (たとえば、低すぎる/高すぎる) 場合、API は脆弱である。

* 実行タイムアウト
* 割り当て可能なメモリの最大値
* ファイル記述子の数
* プロセスの数
* リクエストのペイロードサイズ (たとえば、アップロード)
* クライアント/リソースごとのリクエストの数
* 単一のリクエストレスポンスで返すページごとのレコードの数


## 攻撃シナリオ例

### シナリオ #1

攻撃者は、POST リクエストを `/api/v1/images` に発行することで巨大な画像をアップロードする。アップロードが完了すると、API は異なるサイズで複数のサムネイルを生成する。アップロードされた画像のサイズが原因で、サムネイルを生成している間に利用可能なメモリが使い果たされ、API は応答しなくなる。

### シナリオ #2

1 ページ当たり `200` ユーザを上限として UI 上にユーザのリストを含んでいるアプリケーションが存在する。そのユーザリストは、`/api/users?page=1&size=100` というクエリを用いてサーバから取り出される。攻撃者は、`size` パラメータを `200 000` に変更することで、データベースのパフォーマンスの問題を引き起こす。その間、API は応答しなくなり、このクライアントもしくは他のクライアントからのさらなるリクエストを処理できなくなる (すなわち DoS である)。

同様のシナリオが、整数オーバーフローやバッファオーバーフローのエラーを引き起こすのに使用されるかもしれない。


## 対策方法

* Docker を使うことで、[メモリ][1]、[CPU][2]、[再起動の回数][3]、[ファイル記述子、プロセス][4] を簡単に制限することができる。
* 定義された時間枠でクライアントが API を呼び出すことができる頻度の制限を実装する。
* 制限数と制限がリセットされる時間を提供して、制限を超えたときにクライアントに通知する。
* クエリストリングとリクエストボディのパラメータに関する適切なサーバ側の検証を追加する。特に、レスポンスに返ってくるレコードの数の制御を行うようなもの。
* 文字列の最大長、配列の要素の最大数など、受信する全てのパラメータやペイロードのデータの最大サイズを定義して実行する。



## References

### OWASP

* [Blocking Brute Force Attacks][5]
* [Docker Cheat Sheet - Limit resources (memory, CPU, file descriptors,
  processes, restarts)][6]
* [REST Assessment Cheat Sheet][7]

### External

* [CWE-307: Improper Restriction of Excessive Authentication Attempts][8]
* [CWE-770: Allocation of Resources Without Limits or Throttling][9]
* “_Rate Limiting (Throttling)_” - [Security Strategies for Microservices-based
  Application Systems][10], NIST

[1]: https://docs.docker.com/config/containers/resource_constraints/#memory
[2]: https://docs.docker.com/config/containers/resource_constraints/#cpu
[3]: https://docs.docker.com/engine/reference/commandline/run/#restart-policies---restart
[4]: https://docs.docker.com/engine/reference/commandline/run/#set-ulimits-in-container---ulimit
[5]: https://www.owasp.org/index.php/Blocking_Brute_Force_Attacks
[6]: https://github.com/OWASP/CheatSheetSeries/blob/3a8134d792528a775142471b1cb14433b4fda3fb/cheatsheets/Docker_Security_Cheat_Sheet.md#rule-7---limit-resources-memory-cpu-file-descriptors-processes-restarts
[7]: https://github.com/OWASP/CheatSheetSeries/blob/3a8134d792528a775142471b1cb14433b4fda3fb/cheatsheets/REST_Assessment_Cheat_Sheet.md
[8]: https://cwe.mitre.org/data/definitions/307.html
[9]: https://cwe.mitre.org/data/definitions/770.html
[10]: https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-204-draft.pdf
