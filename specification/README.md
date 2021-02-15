<!--
# OpenTelemetry Specification
-->

# OpenTelemetry 仕様

[![Checks](https://github.com/open-telemetry/opentelemetry-specification/workflows/Checks/badge.svg?branch=main)](https://github.com/open-telemetry/opentelemetry-specification/actions?query=workflow%3A%22Checks%22+branch%3Amain)
![GitHub tag (latest SemVer)](https://img.shields.io/github/tag/open-telemetry/specification.svg)

![OpenTelemetry Logo](https://opentelemetry.io/img/logos/opentelemetry-horizontal-color.png)

<!--
_Curious about what OpenTelemetry is? Check out our [website](https://opentelemetry.io) for an explanation!_
-->

_OpenTelemetryとはなにか疑問ですか? [website](https://opentelemetry.io) を見てください!_

<!--
The OpenTelemetry specification describes the cross-language requirements and expectations for all OpenTelemetry implementations. Substantive changes to the specification must be proposed using the [OpenTelemetry Enhancement Proposal](https://github.com/open-telemetry/oteps) process. Small changes, such as clarifications, wording changes, spelling/grammar corrections, etc. can be made directly via pull requests.
-->

OpenTelemetry仕様では、すべてのOpenTelemetry実装に対する言語非依存の要件と期待する事項を記述しています。仕様の大幅な変更は、[OpenTelemetry Enhancement Proposal](https://github.com/open-telemetry/oteps)のプロセスを使って提案しなければなりません。明瞭化、文言の変更、スペルや文法の修正などの小さな変更は、プルリクエストで直接行うことができます。

<!--
Questions that needs additional attention can be brought to the regular
specifications meeting. EU and US timezone friendly meeting is held every
Tuesday at 8 AM pacific time. Meeting notes are held in the [google
doc](https://docs.google.com/document/d/1-bCYkN-DWJq4jw1ybaDZYYmx-WAe6HnwfWbkm8d57v8/edit?usp=sharing).
APAC timezone friendly meeting is held Tuesdays, 4PM pacific time. See
[OpenTelemetry calendar](https://github.com/open-telemetry/community#calendar).
-->

追加の注意を必要とする質問は、定例の仕様会議に持ち込むことができます。EUと米国のタイムゾーンフレンドリーな会議は、毎週火曜日の太平洋時間の午前8時に開催されています。会議ノートは[google doc](https://docs.google.com/document/d/1-bCYkN-DWJq4jw1ybaDZYYmx-WAe6HnwfWbkm8d57v8/edit?usp=sharing)に掲載されています。APACタイムゾーンフレンドリーミーティングは毎週火曜日、太平洋時間の午後4時に開催されます。[OpenTelemetryカレンダー](https://github.com/open-telemetry/community#calendar)をご覧ください。

<!--
Escalations to technical committee may be made over the
[e-mail](https://github.com/open-telemetry/community#tc-technical-committee).
Technical committee holds regular meetings, notes are held
[here](https://docs.google.com/document/d/17v2RMZlJZkgoPYHZhIFTVdDqQMIAH8kzo8Sl2kP3cbY/edit?usp=sharing).
-->

技術委員会へのエスカレーションは、[e-mail](https://github.com/open-telemetry/community#tc-technical-committee)で行うことができます。技術委員会では定例会議を開催しており、ノートはこちら(https://docs.google.com/document/d/17v2RMZlJZkgoPYHZhIFTVdDqQMIAH8kzo8Sl2kP3cbY/edit?usp=sharing)に掲載しています。


<!--
## Table of Contents
-->

## 目次

<!--
- [Overview](specification/overview.md)
- [Glossary](specification/glossary.md)
- [Versioning and stability for OpenTelemetry clients](specification/versioning-and-stability.md)
- [Library Guidelines](specification/library-guidelines.md)
  - [Package/Library Layout](specification/library-layout.md)
  - [General error handling guidelines](specification/error-handling.md)
- API Specification
  - [Baggage](specification/baggage/api.md)
    - [Propagators](specification/context/api-propagators.md)
  - [Tracing](specification/trace/api.md)
  - [Metrics](specification/metrics/api.md)
- SDK Specification
  - [Tracing](specification/trace/sdk.md)
  - [Resource](specification/resource/sdk.md)
  - [Configuration](specification/sdk-configuration.md)
- Data Specification
  - [Semantic Conventions](specification/overview.md#semantic-conventions)
  - [Protocol](specification/protocol/README.md)
- About the Project
  - [Timeline](#project-timeline)
  - [Notation Conventions and Compliance](#notation-conventions-and-compliance)
  - [Versioning the Specification](#versioning-the-specification)
  - [Acronym](#acronym)
  - [Contributions](#contributions)
  - [License](#license)
-->

- [概要](仕様/overview.md)
- [用語集](specification/glossary.md)
- [OpenTelemetryクライアントのバージョン管理と安定性](specification/versioning-and-stability.md)
- [ライブラリガイドライン](specification/library-guidelines.md)
  - [パッケージ/ライブラリレイアウト](specification/library-layout.md)
  - [一般的なエラー処理ガイドライン](specification/error-handling.md)
- API仕様
  - [手荷物](specification/baggage/api.md)
    - [Propagators](specification/context/api-propagators.md)
  - [トレース](specification/trace/api.md)
  - [メトリクス](specification/metrics/api.md)
- SDK仕様
  - [トレース](specification/trace/sdk.md)
  - [リソース](specification/resource/sdk.md)
  - [設定](specification/sdk-configuration.md)
- データ仕様
  - [意味規定](specification/overview.md#semantic-conventions)
  - [プロトコル](specification/protocol/README.md)
- プロジェクトについて
  - [タイムライン](#プロジェクトタイムライン)
  - [表記規則とコンプライアンス](#表記規則とコンプライアンス)
  - [仕様書のバージョン管理](#versioning-the-specification)
  - [頭字語](#略語)
  - [貢献](#貢献)
  - [ライセンス](#license)


<!--
## Project Timeline
-->

## プロジェクトのタイムライン

<!--
The current project status as well as information on notable past releases is found at
[the OpenTelemetry project page](https://opentelemetry.io/project-status/).
-->

現在のプロジェクトの状況や過去のリリース情報は [OpenTelemetryプロジェクトページ](https://opentelemetry.io/project-status/) にあります。

<!--
Information about current work and future development plans is found at the
[specification development milestones](https://github.com/open-telemetry/opentelemetry-specification/milestones).
-->

現在の作業内容や今後の開発計画に関する情報は、[仕様開発マイルストーン](https://github.com/open-telemetry/opentelemetry-specification/milestones)に掲載されています。

<!--
## Notation Conventions and Compliance
-->

## 表記規則とコンプライアンス

<!--
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in the [specification](./specification/overview.md) are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14) [[RFC2119](https://tools.ietf.org/html/rfc2119)] [[RFC8174](https://tools.ietf.org/html/rfc8174)] when, and only when, they appear in all capitals, as shown here.
-->

[仕様書](./specification/overview.md)のキーワード "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", "OPTIONAL "は、[BCP 14](https://tools.ietf.org/html/bcp14) [RFC2119](https://tools.ietf.org/html/rfc2119) [RFC8174](https://tools.ietf.org/html/rfc8174)で述べられているように解釈されるべきです。このドキュメント中に示されているように、これらの用語はすべて大文字で表示される場合に限ります。

<!--
An implementation of the [specification](./specification/overview.md) is not compliant if it fails to satisfy one or more of the "MUST", "MUST NOT", "REQUIRED", "SHALL", or "SHALL NOT" requirements defined in the [specification](./specification/overview.md).
Conversely, an implementation of the [specification](./specification/overview.md) is compliant if it satisfies all the "MUST", "MUST NOT", "REQUIRED", "SHALL", and "SHALL NOT" requirements defined in the [specification](./specification/overview.md).
-->

[仕様](./specification/overview.md)の実装は、[仕様](./specification/overview.md)で定義された "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT "のうち1つ以上の要件を満たさない場合、準拠していません。逆に、[仕様](./specification/overview.md)の実装は、[仕様](./specification/overview.md)で定義されている "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT "のすべての要件を満たしていれば、準拠しています。


<!--
## Versioning the Specification
-->

## 仕様書のバージョン管理

<!--
Changes to the [specification](./specification/overview.md) are versioned according to [Semantic Versioning 2.0](https://semver.org/spec/v2.0.0.html) and described in [CHANGELOG.md](CHANGELOG.md). Layout changes are not versioned. Specific implementations of the specification should specify which version they implement.
-->

[仕様](./specification/overview.md)への変更は、[Semantic Versioning 2.0](https://semver.org/spec/v2.0.0.html)に従ってバージョン管理され、[CHANGELOG.md](CHANGELOG.md)に記述されています。レイアウトの変更はバージョン管理されません。仕様の特定の実装は、どのバージョンを実装したものかを指定しなければなりません。

<!--
Changes to the change process itself are not currently versioned but may be independently versioned in the future.
-->

変更プロセス自体の変更は、現在はバージョン化されていませんが、将来的には独立してバージョン化される可能性があります。

<!--
## Acronym
-->

## 頭字語

<!--
The official acronym used by the OpenTelemetry project is "OTel".
-->

OpenTelemetryプロジェクトで使われている正式な頭文字は"OTel"です。

<!--
Please refrain from using "OT" in order to avoid confusion with the now deprecated "OpenTracing" project.
-->

現在は廃止された "OpenTracing" プロジェクトとの混同を避けるため、 "OT" の使用はお控えください。

<!--
## Contributions
-->

## 貢献

<!--
See [CONTRIBUTING.md](CONTRIBUTING.md) for details on contribution process.
-->

貢献プロセスの詳細については、[CONTRIBUTING.md](CONTRIBUTING.md)を参照してください。

<!--
## License
-->

## ライセンス

<!--
By contributing to OpenTelemetry Specification repository, you agree that your contributions will be licensed under its [Apache 2.0 License](https://github.com/open-telemetry/specification/blob/main/LICENSE).
-->

OpenTelemetry Specification リポジトリに貢献することで、あなたの貢献はその [Apache 2.0 License](https://github.com/open-telemetry/specification/blob/main/LICENSE) の下でライセンスされることに同意したことになります。

