API Security Risks - API セキュリティリスク
==================

[OWASP Risk Rating Methodology][1] は、リスク解析を行うために使用された。

以下の表は、リスクスコアに関する専門用語の要約である。

| 脅威エージェント | 攻撃難易度 | 弱点の蔓延度 | 弱点の検出難易度 | 技術的影響 | ビジネス的影響 |
| :-: | :-: | :-: | :-: | :-: | :-: |
| API 依存 | 容易: **3** | 広範囲 **3** | 容易 **3** | 深刻 **3** | ビジネス依存 |
| API 依存 | 普通: **2** | 一般的 **2** | 普通 **2** | 中程度 **2** | ビジネス依存 |
| API 依存 | 困難: **1** | 希少 **1** | 困難 **1** | 少ない **1** | ビジネス依存 |

**注意**: このアプローチでは、脅威エージェントの可能性を考慮していない。特定のアプリケーションに関する様々な技術的詳細についてもまた考慮していない。これらの要素のいずれかは、攻撃者が特定の脆弱性を見つけてエクスプロイトする全体的な可能性に著しく影響を与える可能性がある。本格付けは、ビジネスに対する実際の影響を考慮していない。組織は、文化、業界、規制環境を考慮して、アプリケーションや API からどの程度のセキュリティリスクを受け入れるかを決定する必要があるだろう。OWASP API Security Top 10 の目的は、このようなリスク解析を行うことではない。

## References

### OWASP

* [OWASP Risk Rating Methodology][1]
* [Article on Threat/Risk Modeling][2]

### External

* [ISO 31000: Risk Management Std][3]
* [ISO 27001: ISMS][4]
* [NIST Cyber Framework (US)][5]
* [ASD Strategic Mitigations (AU)][6]
* [NIST CVSS 3.0][7]
* [Microsoft Threat Modeling Tool][8]

[1]: https://www.owasp.org/index.php/OWASP_Risk_Rating_Methodology
[2]: https://www.owasp.org/index.php/Threat_Risk_Modeling
[3]: https://www.iso.org/iso-31000-risk-management.html
[4]: https://www.iso.org/isoiec-27001-information-security.html
[5]: https://www.nist.gov/cyberframework
[6]: https://www.asd.gov.au/infosec/mitigationstrategies.htm
[7]: https://nvd.nist.gov/vuln-metrics/cvss/v3-calculator
[8]: https://www.microsoft.com/en-us/download/details.aspx?id=49168
