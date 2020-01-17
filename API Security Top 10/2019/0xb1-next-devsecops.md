DevSecOps のための次
=========================

現代のアプリケーションアーキテクチャの重要性が原因で、セキュアな API を構築することは極めて重要なことである。セキュリティは軽視できず、開発ライフサイクル全体の一部であるべきだ。スキャンとペネトレーションテストは、もはや年 1 回では十分ではない。

DevSecOps は、開発努力につながり、ソフトウェア開発ライフサイクル全体で継続的なセキュリティテストを促進する。これらの目標は、開発スピードを影響を与えることなく、セキュリティの自動化で開発ライフサイクルを強化することである。

疑念がある場合は、[DevSecOps Manifesto][1] の情報を頻繁に手に入れてレビューすること。

| | |
|-|-|
| **脅威モデルの理解** | テストの優先度は脅威モデルに由来する。1 つも持っていない場合は、情報として、[OWASP Application Security Verification Standard (ASVS)][2] と [OWASP Testing Guide][3] の使用を検討すると良い。開発チームを参加させることで、彼らのセキュリティ意識がより高くなる可能性がある。 |
| **ソフトウェア開発ライフサイクル (SDLC) の理解** | 開発チームに参加し、ソフトウェア開発ライフサイクルについてより理解を深める。継続したセキュリティテストにおける貢献は、人、プロセス、ツールと互換性があるべきである。不要な衝突や妨害がないように、全ての人がプロセスに同意する必要がある。 |
| **テスト戦略** | 作業は開発スピードに影響を与えないため、最良の (単純、高速、正確な) 技術を抜け目なく選択し、セキュリティ要件を検証するべきである。[OWASP Security Knowledge Framework][4] と [OWASP Application Security Verification Standard][5] は、機能的かつ非機能的なセキュリティ要件に関する素晴らしいソースである。[projects][6] と [tools][7] には、[DevSecOps community][8] によって提供されたものと同様の他の素晴らしいソースが存在する。 |
| **範囲と制度の達成** | あなたは、開発チームと運用チームの間の架け橋である。カバレッジを達成するためには、機能のみに焦点を当てるだけではなく、オーケストレーションにも焦点を当てるべきである。時間と労力を最大限に利用するために、初めから開発チームと運用チーム両方の近くで働くようにした方が良い。必須のセキュリティが継続的に検証されている状態を目指すべきである。 |
| **調査結果の正確な報告** | より少ない衝突、もしくは全く衝突を発生させることなく価値を提供する。開発チームが使用しているツール (PDF ファイルではない) を用いて調査結果をタイムリーに送信する。開発チームに加わり、調査結果について対処する。彼らを教育するチャンスを利用して、弱点と実際の攻撃シナリオを含めてそれがどのように悪用される可能性があるかを明確に説明する |

[1]: https://www.devsecops.org/
[2]: https://www.owasp.org/index.php/Category:OWASP_Application_Security_Verification_Standard_Project
[3]: https://www.owasp.org/index.php/OWASP_Testing_Project
[4]: https://www.owasp.org/index.php/OWASP_Security_Knowledge_Framework
[5]: https://www.owasp.org/index.php/Category:OWASP_Application_Security_Verification_Standard_Project
[6]: http://devsecops.github.io/
[7]: https://github.com/devsecops/awesome-devsecops
[8]: http://devsecops.org
