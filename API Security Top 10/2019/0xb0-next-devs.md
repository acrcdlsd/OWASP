What's Next For Developers - 開発者向けの次のステップ
==========================

ソフトウェアを作成して保守するタスク、または原稿のソフトウェアを改修するタスクは、困難な可能性がある。このことは API に関しても変わりはない。

我々は、教育と認知がセキュアなソフトウェアを書くためのキーファクタであると信じている。目標を達成するために必要な他の全てのことは、**反復可能なセキュリティプロセスと標準的なセキュリティ制御を制定して使用すること**に依存している。

OWASP には、プロジェクトが始まった瞬間から、セキュリティに対処するための無料でオープンなリソースが大量に存在している。利用可能なプロジェクトの包括的なリストについては、[OWASP Projects page][1] をぜひ訪れてください。

| | |
|-|-|
| **教育** | 専門職や関心に準じて [OWASP Education Project materials][2] を読み始めることができる。ハンズオン学習に関しては、[ロードマップ][3] に **crAPI** - **C**ompletely **R**idiculous **API** を追加した。その一方で、[OWASP DevSlop Pixi Module][4] を用いて WebAppSec を練習することができる。これは、ユーザに現代の Web アプリケーションと API のセキュリティ問題をテストする方法を教え、将来、よりセキュアな API を書く方法を学ばせるための脆弱な WebApp や API サービスインテントである。また、[OWASP AppSec Conference][5] のトレーニングセッションに参加したり、[ローカル支部に参加][6]することもできる。 |
| **セキュリティ要件** | セキュリティはもともと全てのプロジェクトの一部であるべきである。要件を引き出す時、そのプロジェクトにとって "secure" の意味が何であるかを定義することが重要である。セキュリティ要件を設定するためのガイドとして、OWASP は、[OWASP Application Security Verification Standard (ASVS)][7] を用いることを推奨する。アウトソーシングするのであれば、[OWASP Secure Software Contract Annex][8] を考慮することだ。それは、現地の法律や規制に従って適応する必要がある。 |
| **セキュリティアーキテクチャ** | セキュリティはプロジェクトの全てのステージにおいて、依然として懸念事項なままである。[OWASP Prevention Cheat Sheets][9] は、アーキテクチャフェーズでセキュリティを設計する方法に関するガイダンスのための良いスターティングポイントである。他にも多くのものがあるが、[REST Security Cheat Sheet][10] と [REST Assessment Cheat Sheet][11] を見つけることができるだろう。 |
| **標準的なセキュリティ制御** | 標準的なセキュリティ制御を選択することは、独自のロジックを記述することでセキュリティの弱点を取り込んでしまうリスクを低減する。現代のフレームワークのほとんどは、効果的な組み込み制御を搭載しているが、[OWASP Proactive Controls][12] は、プロジェクトに含めるべきセキュリティ制御の有用な概要を提供する。またOWASP は、検証制御などの有用ないくつかのライブラリとツールを提供する。 |
| **セキュアソフトウェア開発ライフサイクル** | API を構築するときに、[OWASP Software Assurance Maturity Model (SAMM)][13] を使用してプロセスを改良することができる。いくつかの他の OWASP プロジェクトは、様々な API 開発フェーズで役立つだろう (たとえば、[OWASP Code Review Project][14])。 |

[1]: https://www.owasp.org/index.php/Category:OWASP_Project
[2]: https://www.owasp.org/index.php/OWASP_Education_Material_Categorized
[3]: https://www.owasp.org/index.php/OWASP_API_Security_Project#tab=Road_Map
[4]: https://devslop.co/Home/Pixi
[5]: https://www.owasp.org/index.php/Category:OWASP_AppSec_Conference
[6]: https://www.owasp.org/index.php/OWASP_Chapter
[7]: https://www.owasp.org/index.php/Category:OWASP_Application_Security_Verification_Standard_Project
[8]: https://www.owasp.org/index.php/OWASP_Secure_Software_Contract_Annex
[9]: https://www.owasp.org/index.php/OWASP_Cheat_Sheet_Series
[10]: https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/REST_Security_Cheat_Sheet.md
[11]: https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/REST_Assessment_Cheat_Sheet.md
[12]: https://www.owasp.org/index.php/OWASP_Proactive_Controls#tab=OWASP_Proactive_Controls_2018
[13]: https://www.owasp.org/index.php/OWASP_SAMM_Project
[14]: https://www.owasp.org/index.php/Category:OWASP_Code_Review_Project
