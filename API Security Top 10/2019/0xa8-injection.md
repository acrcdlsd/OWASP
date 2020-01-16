API8:2019 Injection (インジェクション)
===================

| 脅威エージェント/攻撃経路 | セキュリティ上の弱点 | 影響 |
| - | - | - |
| API 依存 : 攻撃難易度 **3** | 蔓延度 **2** : 検出難易度 **3** | 技術的影響 **3** : ビジネス依存 |
| 攻撃者は、利用可能なあらゆるインジェクションベクタ (たとえば、直接入力、パラメータ、統合サービスなど) を介して悪意あるデータを API に供給し、インタプリタに送信されることを期待する。 | インジェクションの欠陥は、非常に一般的であり、SQL、LDAP、NoSQL のクエリや OS コマンド、XML パーサ、ORM でよく見つかる。これらの欠陥は、ソースコードレビュー時に容易に検出できる。攻撃者はスキャナやファザーを使用する可能性がある。 | インジェクションは、情報漏えいやデータ消失につながる可能性がある。また、DoS や完全なホストの乗っ取りにつながる可能性もある。 |

## API が脆弱かどうかの確認

以下の場合、API はインジェクションの欠陥に対して脆弱である。

* クライアントから提供されるデータが、API によって検証、フィルタリング、サニタイジングされていない。
* Client-supplied data is directly used or concatenated to SQL/NoSQL/LDAP queries, OS commands, XML parsers, and Object Relational Mapping (ORM)/Object Document Mapper (ODM).
クライアントから提供されるデータが、SQL/NoSQL/LDAP クエリや OS コマンド、XML パーサ、オブジェクト関係マッピング (ORM) に直接使用・連結されている。
* 外部システム (たとえば、統合システム) から入ってくるデータが、API によって検証、フィルタリング、サニタイジングされていない。

## 攻撃シナリオ例

### シナリオ #1

ペアレンタル・コントロールデバイスのファームウェアは、appId がマルチパートパラメータとして送信されることを想定しているエンドポイント `/api/CONFIG/restore` を提供する。デコンパイラを用いることで、攻撃者は、appId がサニタイジングなしでシステムコールに直接渡されることを発見する。

```c
snprintf(cmd, 128, "%srestore_backup.sh /tmp/postfile.bin %s %d",
         "/mnt/shares/usr/bin/scripts/", appid, 66);
system(cmd);
```

以下のコマンドで、攻撃者は同様の脆弱なファームウェアを持つ任意のデバイスをシャットダウンすることができる。

```
$ curl -k "https://${deviceIP}:4567/api/CONFIG/restore" -F 'appid=$(/etc/pod/power_down.sh)'
```

### シナリオ #2

予約があるオペレーションのための基本的な CRUD 機能を持つアプリケーションが存在する。攻撃者は、予約削除のリクエストの `bookingId` クエリストリングパラメータを用いて、NoSQL インジェクションが可能であることを何とか特定した。これは、リクエストが `DELETE /api/bookings?bookingId=678` のようになっているだろう。API サーバは以下の関数を用いて、削除リクエストを処理する。

```javascript
router.delete('/bookings', async function (req, res, next) {
  try {
      const deletedBooking = await Bookings.findOneAndRemove({'_id' : req.query.bookingId});
      res.status(200);
  } catch (err) {
     res.status(400).json({error: 'Unexpected error occured while processing a request'});
  }
});
```

攻撃者は、リクエストを傍受し、`bookingId` クエリストリングパラメータを以下に示すように変更する。このケースでは、攻撃者は他ユーザの予約を削除することができる。

```
DELETE /api/bookings?bookingId[$ne]=678
```

## 対策方法

インジェクションを防ぐためには、コマンドとクエリからデータを分ける必要がある。

* 単一の、信頼できる、活発に維持されているライブラリを用いてデータ検証を行う。
* クライアントから提供されrう全てのデータや統合システムから来る他のデータの検証、フィルタリング、サニタイジングを行う。
* 特殊文字は、対象のインタプリタの特有のシンタックスを用いてエスケープされるべきである。
* パラメータ化されたインタフェイスを提供する安全な API を選択する。
* 返ってくるレコードの数を常に制限し、インジェクションが起こった際の大量の漏えいを防止する。
* 十分なフィルタを用いて入力データを検証し、各入力パラメータに有効な値の場合のみ許可する。
* 全ての文字列パラメータに対してデータタイプと厳密なパターンを定義する。

## References

### OWASP

* [OWASP Injection Flaws][1]
* [SQL Injection][2]
* [NoSQL Injection Fun with Objects and Arrays][3]
* [Command Injection][4]

### External

* [CWE-77: Command Injection][5]
* [CWE-89: SQL Injection][6]

[1]: https://www.owasp.org/index.php/Injection_Flaws
[2]: https://www.owasp.org/index.php/SQL_Injection
[3]: https://www.owasp.org/images/e/ed/GOD16-NOSQL.pdf
[4]: https://www.owasp.org/index.php/Command_Injection
[5]: https://cwe.mitre.org/data/definitions/77.html
[6]: https://cwe.mitre.org/data/definitions/89.html
