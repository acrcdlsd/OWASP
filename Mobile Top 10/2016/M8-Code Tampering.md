# M8：コード改ざん

<table>
 <tr>
  <th>脅威エージェント</th>
  <th>攻撃経路</th>
  <th colspan="2">セキュリティ上の弱点</th>
  <th>技術的な影響</th>
  <th>ビジネスへの影響</th>
 </tr>
 <tr>
  <td>アプリケーション依存 </td>
  <td>攻撃難易度：容易</td>
  <td>普及度：中</td>
  <td>検出難易度：普通</td>
  <td>深刻な影響</td>
  <td>アプリケーション / ビジネス依存</td>
 </tr>
 <tr>
  <td>通常、攻撃者はサードパーティのアプリストアにホストされているアプリを悪意のある形にすることで、コード改ざんを悪用します。また、攻撃者は、フィッシング攻撃を通してユーザをだましてアプリをインストールさせるかもしれません。</td>
  <td>通常、攻撃者は本カテゴリを悪用するために以下のことを行います。
   <ul>
    <li>アプリケーションパッケージのコアバイナリに対して、直接バイナリ変更を加える。</li>
    <li>アプリケーションパッケージ内のリソースに対して、直接バイナリ変更を加える。</li>
    <li>傍受して悪意のある外部コードを実行するためにシステムAPIをリダイレクトさせたり、置き換えたりする。</li>
   </ul>
  <td colspan="2">改ざんされたアプリケーションは、意外にもあなたが思っている以上に存在します。アプリストア内に許可されていないバージョンのモバイルアプリが存在しないか検知して取り除くことを中心に構築されたセキュリティ業界が存在します。コード改ざん検知の問題を解決するために取るアプローチによって、組織は許可されていないバージョンのアプリの検知が制限されたり高い成功率となったりします。<br> 本カテゴリは、バイナリパッチ、ローカルリソースの改ざん、メソッドフッキング、メソッドスウィズリング、動的メモリ改ざんを含みます。 <br> アプリケーションがモバイルデバイスに一旦配信されると、コードとデータといったリソースは端末に常駐することになります。攻撃者は、そのコードを直接改ざんしたり、メモリ内容を動的に改ざんしたり、アプリケーションが使用するシステムAPIを改ざんまたは置き換えたり、アプリケーションのデータとリソースを改ざんしたりすることが可能です。これにより、攻撃者は個人的利益もしくは金銭的利益のために、ソフトウェアが本来意図している使用方法を覆すための直接的な方法を得ることができます。</td>
  <td>コード改ざんによる影響は、改ざんされたもの自身の性質によりますが、広範囲に及ぶ可能性があります。典型的な種類の影響は以下です。
   <ul>
    <li> 許可されていない新機能</li>
    <li> なりすまし犯罪</li>
    <li> 詐欺</li>
   </ul>
  <td>コード改ざんによるビジネスへの影響は以下の結果につながります。
   <ul>
    <li> 著作権侵害による利益損失</li>
    <li> 風評被害</li>
   </ul>
  </td>
 </tr>
</table>



## 脆弱性有無の確認
技術的に、全てのモバイルコードはコード改ざんに対して脆弱です。モバイルコードはコードを作成した組織のコントロール外の環境内で実行されるからです。同時に、コードが実行される環境を改ざんする方法がたくさん存在しているため、攻撃者は自由自在にコードを改ざんすることが可能です。
モバイルコードは本質的に脆弱ですが、認可されていないコード改ざんを検出して防止する価値があるかどうか確認することが重要です。特定のビジネス分野のアプリ(例えばゲーム)は、他のもの(例えばホスピタリティ)よりもコート改ざんの影響に対して脆弱です。そのため、このリスクに対処するかどうかを決定する前にビジネス影響を考慮することが非常に重要です。


## 防止方法
モバイルアプリは、コンパイル時の完全性からコードが追加、変更されていることを実行中に検出できなければなりません。アプリは実行時、コードの完全性違反に適切に反応できなければなりません。このタイプのリスクを改善するための戦略は、[OWASPのリバースエンジニアリングとコード改ざん防止プロジェクト](https://www.owasp.org/index.php/OWASP_Reverse_Engineering_and_Code_Modification_Prevention_Project)に、技術的な詳細が記述されています。

### Androidのroot検知
通常、改ざんされたアプリはJailbreakもしくはroot化された環境で実行されます。したがって、実行時にこれらのタイプの侵害された環境を検出し、それに応じて反応(サーバへの通知、もしくはシャットダウン)することが妥当です。root化されたAndroidデバイスを検出するために、以下のような一般的な方法が存在します。

テストキーのチェック
 - build.propに、開発者ビルドもしくは非公式のROMを示すro.build.tags=test-keyの行が含まれていなか確認する。
OTA証明書のチェック
 - /etc/security/otacerts.zipファイルが存在するか確認する

既知のroot化されたAPKのチェック
 - com.noshufou.android.su
 - com.thirdparty.superuser
 - eu.chainfire.supersu
 - com.koushikdutta.superuser

SUバイナリのチェック
 - /system/bin/su
 - /system/xbin/su
 - /sbin/su
 - /system/su
 - /system/bin/.ext/.su

SUコマンドを直接実行してみる
 - suコマンドを実行し、カレントユーザのIDをチェックする。もし「0」が返って来たのなら、suコマンドは成功している。
 
### iOSのJailbreak検知


## 攻撃シナリオの例
アプリストアで入手可能な偽のアプリケーションが多数存在しています。マルウェアペイロードが含まれている場合もあります。改ざんされたアプリのほとんどが、オリジナルのコアバイナリやリソースが改ざんされています。攻撃者は、これらを新たなアプリケーションとしてリパックし、サードパーティのストアにリリースします。

**シナリオ# 1：**<br>
ゲームは、本メソッドを使って攻撃する非常に人気のあるターゲットです。攻撃者はゲームの全フリーミアム機能に対して課金することに興味がない人々を引きつけます。コード内で、攻撃者はアプリケーション内購入が成功したかどうかを検出する条件分岐を回避します。この回避によって、被害者は課金しなければ得られないゲームアーティファクトや新しい能力を獲得することができます。攻撃者はまたユーザのIDを盗むスパイウェアを注入しています。

**シナリオ# 2：**<br>
銀行アプリも攻撃ターゲットとして人気があります。これらのアプリは通常、攻撃者にとって有益な機密情報を扱っています。攻撃者は、サードパーティのサイトへユーザ名/パスワードと共にユーザの個人識別情報(PII)を送信する偽のバージョンのアプリを作成することができます。これは、Zeusマルウェアのデスクトップ版を思い出させます。これは通常、銀行を標的とした詐欺につながります。


## 参考資料
### OWASP
 - [OWASP Reverse Engineering and Code Modification Prevention Project](https://www.owasp.org/index.php/OWASP_Reverse_Engineering_and_Code_Modification_Prevention_Project)

### その他
 1. [Arxan Research: State of Security in the App Economy, Volume 2](https://www.arxan.com/assets/1/7/State_of_Security_in_the_App_Economy_Report_Vol._2.pdf), November 2013:
  "Adversaries have hacked 78 percent of the top 100 paid Android and iOS apps in 2013."

 2. [HP Research: HP Research Reveals Nine out of 10 Mobile Applications Vulnerable to Attack](http://www8.hp.com/us/en/hp-news/press-release.html?id=1528865#.UuwZFPZvDi5), 18 November 2013:
 "86 percent of applications tested lacked binary hardening, leaving applications vulnerable to information disclosure, buffer overflows and poor performance. To ensure security throughout the life cycle of the application, it is essential to build in the best security practices from conception."

 3. [North Carolina State University: Dissecting Android Malware: Characterization and Evolution](http://www.csc.ncsu.edu/faculty/jiang/pubs/OAKLAND12.pdf), 7 September 2011:
  "Our results show that 86.0% of them (Android Malware) repackage legitimate apps to include malicious payloads; 36.7% contain platform-level exploits to escalate privilege; 93.0% exhibit the bot-like capability."

 4. [Tech Hive: Apple Pulls Ripoff Apps from its Walled GardenFeb 4th](http://www.techhive.com/article/249310/apple_pulls_ripoff_apps_from_its_walled_garden.html), 2012:
 "While Apple is known for screening apps before they are allowed to sprout up in its walled garden, clearly fake apps do get in. Once they do, getting them out depends on developers who raise a fuss."

 5. [Tech Crunch: Developer Spams Google Play With RipOffs of Well-Known Apps… Again](http://techcrunch.com/2014/01/02/developer-spams-google-play-with-ripoffs-of-well-known-apps-again/), January 2 2014:
 "It's not uncommon to search the Google Play app store and find a number of knock-off or "fake" apps aiming to trick unsuspecting searchers into downloading them over the real thing."

 6. [Extreme Tech: Chinese App Store Offers Pirated iOS Apps Without the Need To Jailbreak](http://www.extremetech.com/mobile/153849-chinese-app-store-offers-pirated-ios-apps-without-the-need-to-jailbreak), April 19 2013:
 "The site offers apps for free that would otherwise cost money, including big-name titles."

 7. [OWASP: Architectural Principles That Prevent Code Modification or Reverse Engineering](https://www.owasp.org/index.php/Architectural_Principles_That_Prevent_Code_Modification_or_Reverse_Engineering), January 11th 2014.

 8. Gartner report: Avoiding Mobile App Development Security Pitfalls, 24 May 2013:
 "For critical applications, such as transactional ones and sensitive enterprise applications, hardening should be used."

 9. Gartner report: Emerging Technology Analysis: Mobile Application Shielding, March 26th, 2013:
 "As more regulated and sensitive data applications move to mobile platforms the need to increase data protection increases. Mobile application shielding presents the opportunity to security providers to offer higher data protection standards to mobile platforms that exceed mobile OS security."

 10. Gartner report: Proliferating Mobile Transaction Attack Vectors and What to Do About Them, March 1st, 2013:
 "Use mobile application security testing services and self-defending application utilities to help prevent hacking attempts."

 11. Gartner report: Select a Secure Mobile Wallet for Proximity, March 1st, 2013:
 "Application hardening can fortify sensitive business code against hacking attempts, such as reverse engineering"

 12. Forrester paper: Choose The Right Mobile Development Solutions For Your Organization, May 6th 2013:
 "5 Key Protections: Data Protection, App Compliance, App-Level Threat Defense, Security Policy Enforcement, App Integrity"

 13. [John Wiley and Sons, Inc: iOS Hacker's Handbook](http://www.amazon.com/iOS-Hackers-Handbook-Charlie-Miller/dp/1118204123), Published May 2012, [ISBN 1118204123](https://www.owasp.org/index.php/Special:BookSources/1118204123).

 14. [McGraw Hill Education: Mobile Hacking Exposed](http://mobilehackingexposed.com/), Published July 2013, [ISBN 0071817018](https://www.owasp.org/index.php/Special:BookSources/0071817018).

 15. [Publisher Unannounced: Android Hacker's Handbook](http://www.amazon.com/Android-Hackers-Handbook-Joshua-Drake/dp/111860864X), To Be Published April 2014.

 16. Software Development Times: More than 5,000 apps in the Google Play Store are copied APKs, or 'thief-ware', November 20 2013:
 http://sdt.bz/66393#ixzz2sHa7dFMp
 "In most cases, the 2,140 copycat developers that were found reassembled the apps almost identically, adding new advertising SDKs to siphon profits away from the original developers.

 17. [InfoSecurity Magazine: Two Thirds of Personal Banking Apps Found Full of Vulnerabilities](http://www.infosecurity-magazine.com/view/36376/two-thirds-of-personal-banking-apps-found-full-of-vulnerabilities/), January 3 2014:
  "But one of his more worrying findings came from disassembling the apps themselves ... what he found was hardcoded development credentials within the code. An attacker could gain access to the development infrastructure of the bank and infest the application with malware causing a massive infection for all of the application’s users."

 18. [InfoSecurity Magazine: Mobile Malware Infects Millions; LTE Spurs Growth](http://www.infosecurity-magazine.com/view/36686/mobile-malware-infects-millions-lte-spurs-growth/), January 29 2014:
 "Number of mobile malware samples is growing at a rapid clip, increasing by 20-fold in 2013... It is trivial for an attacker to hijack a legitimate Android application, inject malware into it and redistribute it for consumption. There are now binder kits available that will allow an attacker to automatically inject malware into an existing application"
