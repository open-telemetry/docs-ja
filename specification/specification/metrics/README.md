<!--
# OpenTelemetry Metrics
-->

# OpenTelemetry Metrics

**Status**: [Experimental](../document-status.md)

**Owner:**

* [Reiley Yang](https://github.com/reyang)

**Domain Experts:**

* [Bogdan Brutu](https://github.com/bogdandrutu)
* [Josh Suereth](https://github.com/jsuereth)
* [Joshua MacDonald](https://github.com/jmacd)

<!--
Note: this specification is subject to major changes. To avoid thrusting
language client maintainers, we don't recommend OpenTelemetry clients to start
the implementation unless explicitly communicated via
[OTEP](https://github.com/open-telemetry/oteps#opentelemetry-enhancement-proposal-otep).
-->

注意:この仕様は大きく変更される可能性があります。言語クライアントのメンテナに負担をかけないように、[OTEP](https://github.com/open-telemetry/oteps#opentelemetry-enhancement-proposal-otep)で明示的に伝えられない限り、OpenTelemetryのクライアントが実装を開始することは推奨しません。

<details>
<summary>
Table of Contents
</summary>

<!--
* [Overview](#overview)
  * [Design Goals](#design-goals)
  * [Concepts](#concepts)
  * [API](#api)
  * [SDK](#sdk)
* [Specifications](#specifications)
  * [Metrics API](./api.md)
  * Metrics SDK (not available yet)
  * [Metrics Data Model and Protocol](datamodel.md)
  * [Semantic Conventions](./semantic_conventions/README.md)
-->

* [概要](#概要)
  * [デザインゴール](#デザインゴール)
  * [コンセプト](#コンセプト)
  * [API](#api)
  * [SDK](#sdk)
* [仕様](#仕様)
  * [Metrics API](./api.md)
  * Metrics SDK (未定義)
  * [Metrics データモデルとプロトコル](datamodel.md)
  * [セマンティック規約](./semantic_conventions/README.md)

</details>

<!--
## Overview
-->

## 概要

<!--
### Design Goals
-->

### デザインゴール

<!--
Given there are many well-established metrics solutions that exist today, it is
important to understand the goals of OpenTelemetry’s metrics effort:
-->

現在、多くの確立されたメトリクス・ソリューションが存在することを考えると、OpenTelemetryのメトリクスの取り組みの目標を理解することは重要です。

<!--
* **Being able to connect metrics to other signals**. For example, metrics and
  traces can be correlated via exemplars, and metrics dimensions can be enriched
  via [Baggage](../baggage/api.md) and [Context](../context/context.md).
  Additionally, [Resource](../resource/sdk.md) can be applied to
  [logs](../overview.md#log-signal)/[metrics](../overview.md#metric-signal)/[traces](../overview.md#tracing-signal)
  in a consistent way.
-->

* **メトリクスを他のシグナルに結びつけることができること**。例えば、メトリクスとトレースはExamplarを介して相関させることができ、メトリクスのディメンションは[Baggage](../baggage/api.md)や[Context](../context/context.md)を介して情報を追加させることができます。さらに、[Resource](../resource/sdk.md)は、[logs](../overview.md#log-signal)/[metrics](../overview.md#metric-signal)/[traces](../overview.md#tracing-signal)に一貫した方法で適用することができます。

<!--
* **Providing a path for [OpenCensus](https://opencensus.io/) customers to
  migrate to OpenTelemetry**. This was the original goal of OpenTelemetry -
  converging OpenCensus and OpenTracing. We will focus on providing the
  semantics and capability, instead of doing a 1-1 mapping of the APIs.
-->

* **[OpenCensus](https://opencensus.io/)のユーザーがOpenTelemetryに移行するための道筋を提供すること**。これはOpenTelemetryの当初の目標であった、OpenCensusとOpenTracingの収束です。私たちは、APIの1-1マッピングを行うのではなく、セマンティクスと機能を提供することに注力します。

<!--
* **Working with existing metrics instrumentation protocols and standards**. The
  minimum goal is to provide full support for
  [Prometheus](https://prometheus.io/) and
  [StatsD](https://github.com/statsd/statsd) - users should be able to use
  OpenTelemetry clients and [Collector](../overview.md#collector) to collect and
  export metrics, with the ability to achieve the same functionality as their
  native clients.
-->

* **既存のメトリクス計測プロトコルや標準規格との連携**。最低限の目標は、[Prometheus](https://prometheus.io/)と[StatsD](https://github.com/statsd/statsd)を完全にサポートすることです。ユーザーは、OpenTelemetryクライアントと[Collector](../overview.md#collector)を使ってメトリクスを収集し、エクスポートすることができ、ネイティブクライアントと同じ機能を実現することができます。

<!--
### Concepts
-->

### コンセプト

<!--
#### API
-->

#### API

<!--
The **OpenTelemetry Metrics API** ("the API" hereafter) serves two purposes:
-->

**OpenTelemetry Metrics API**(以下、「API」)は、2つの目的を持っています。

<!--
* Capturing raw measurements efficiently and simultaneously.
* Decoupling the instrumentation from the [SDK](#sdk), allowing the SDK to be
  specified/included in the application.
-->

* 生の測定値を効率的かつ同時に取得する。
* 計測器と[SDK](#sdk)を切り離し、アプリケーションでSDKを指定/インクルードすることが可能。

<!--
When no [SDK](#sdk) is explicitly included/enabled in the application, no telemetry data
will be collected. Please refer to the overall [OpenTelemetry
API](../overview.md#api) concept and [API and Minimal
Implementation](../library-guidelines.md#api-and-minimal-implementation) for
more information.
-->

アプリケーションに[SDK](#sdk)が明示的に含まれていない/有効になっていない場合、テレメトリデータは収集されません。詳細については、全体的な[OpenTelemetry API](../overview.md#api)のコンセプトと、[APIと最小実装](../library-guidelines.md#APIと最小実装)を参照してください。

<!--
#### SDK
-->

#### SDK

<!--
The **OpenTelemetry Metrics SDK** ("the SDK" hereafter) implements the API,
providing functionality and extensibility such as configuration, aggregation,
processors and exporters.
-->

**OpenTelemetry Metrics SDK**(以下、SDK)は、APIを実装し、設定、集約、プロセッサー、エクスポーターなどの機能と拡張性を提供します。

<!--
OpenTelemetry requires a [separation of the API from the
SDK](../library-guidelines.md#requirements), so that different SDKs can be
configured at run time. Please refer to the overall [OpenTelemetry
SDK](../overview.md#sdk) concept concept for more information.
-->

OpenTelemetryでは、実行時に異なるSDKを設定できるように、[SDKからAPIの分離](../library-guidelines.md#requirements)を要求しています。詳細については、全体的な[OpenTelemetry SDK](../overview.md#sdk)のコンセプト概念をご参照ください。

<!--
#### Programming Model
-->

#### プログラミングモデル

<!--
```text
+------------------+
| MeterProvider    |
|   Meter A        |                 +-----------+  +------------+  +----------+
|     Instrument X | Measurements... |           |  |            |  |          |
|     Instrument Y +-----------------> Processor +--> Aggregator +--> Exporter +--> Another process
|   Meter B        |                 |           |  |            |  |          |
|     Instrument Z |                 +-----------+  +------------+  +----------+
|     ...          |
|   ...            |
+------------------+
```
-->

```text
+------------------+
| MeterProvider    |
|   Meter A        |                 +-----------+  +------------+  +----------+
|     Instrument X | Measurements... |           |  |            |  |          |
|     Instrument Y +-----------------> Processor +--> Aggregator +--> Exporter +--> Another process
|   Meter B        |                 |           |  |            |  |          |
|     Instrument Z |                 +-----------+  +------------+  +----------+
|     ...          |
|   ...            |
+------------------+
```

<!--
## Specifications
-->

## 仕様

<!--
* [Metrics API](./api.md)
* Metrics SDK (not available yet)
* [Metrics Data Model and Protocol](datamodel.md)
* [Semantic Conventions](./semantic_conventions/README.md)
-->

* [Metrics API](./api.md)
* Metrics SDK (未定義)
* [Metrics データモデルとプロトコル](datamodel.md)
* [セマンティック規約](./semantic_conventions/README.md)

<!--
## References
-->

## 参照

<!--
* Scenarios for Metrics API/SDK Prototyping ([OTEP 146](https://github.com/open-telemetry/oteps/blob/main/text/metrics/0146-metrics-prototype-scenarios.md))
-->

* メトリクスAPI/SDKプロトタイピングのためのシナリオ ([OTEP 146](https://github.com/open-telemetry/oteps/blob/main/text/metrics/0146-metrics-prototype-scenarios.md))

