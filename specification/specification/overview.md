<!--
# Overview
-->

# 概要

<details>
<summary>
目次
</summary>

<!-- toc -->

<!--
- [OpenTelemetry Client Architecture](#opentelemetry-client-architecture)
  * [API](#api)
  * [SDK](#sdk)
  * [Semantic Conventions](#semantic-conventions)
  * [Contrib Packages](#contrib-packages)
  * [Versioning and Stability](#versioning-and-stability)
- [Tracing Signal](#tracing-signal)
  * [Traces](#traces)
  * [Spans](#spans)
  * [SpanContext](#spancontext)
  * [Links between spans](#links-between-spans)
- [Metric Signal](#metric-signal)
  * [Recording raw measurements](#recording-raw-measurements)
    + [Measure](#measure)
    + [Measurement](#measurement)
  * [Recording metrics with predefined aggregation](#recording-metrics-with-predefined-aggregation)
  * [Metrics data model and SDK](#metrics-data-model-and-sdk)
- [Log Signal](#log-signal)
  * [Data model](#data-model)
- [Baggage Signal](#baggage-signal)
- [Resources](#resources)
- [Context Propagation](#context-propagation)
- [Propagators](#propagators)
- [Collector](#collector)
- [Instrumentation Libraries](#instrumentation-libraries)
-->

- [OpenTelemetryクライアントアーキテクチャ](#OpenTelemetryクライアントアーキテクチャ)
  * [API](#api)
  * [SDK](#sdk)
  * [意味規定](#意味規定)
  * [Contribパッケージ](#Contribパッケージ)
  * [バージョン管理と安定性] (#バージョン管理と安定性)
- [Trace Signal](#trace-signal)
  * [Trace](#trace)
  * [Span](#Span)
  * [SpanContext](#spancontext)
  * [Span間のLink](#Span間のLink)
- [Metric Signal](#metric-signal)
  * [生のMeasurementを記録する](#recording-raw-measurements)
    + [Measure](#measure)
    + [Measurement](#measurement)
  * [Metricを事前に定義された集計で記録する](#recording-metrics-with-prefined-aggregation)
  * [MetricデータモデルとSDK](#metrics-data-model-and-sdk)
- [Log Signal](#log-signal)
  * [データモデル](#データモデル)
- [Baggage Signal](#bagagage-signal)
- [リソース](#リソース)
- [Context伝搬](#context伝搬)
- [Propagator](#propagator)
- [Collector](#Collector)
- [計装ライブラリ](#計装ライブラリ)

<!-- tocstop -->

</details>

<!--
This document provides an overview of the OpenTelemetry project and defines important fundamental terms.
-->

このドキュメントでは、OpenTelemetryプロジェクトの概要と重要な基本用語を定義しています。

<!--
Additional term definitions can be found in the [glossary](glossary.md).
-->

その他の用語の定義は、[用語集](glossary.md)に記載されています。

<!--
## OpenTelemetry Client Architecture
-->

## OpenTelemetry クライアントアーキテクチャ

<!--
![Cross cutting concerns](../internal/img/architecture.png)
-->

![横断的な関心事](../internal/img/architecture.png)

<!--
At the highest architectural level, OpenTelemetry clients are organized into [**signals**](glossary.md#signals).
Each signal provides a specialized form of observability. For example, tracing, metrics, and baggage are three separate signals.
Signals share a common subsystem – **context propagation** – but they function independently from each other.
-->

最高レベルのアーキテクチャでは、OpenTelemetryクライアントは[**Signal**](glossary.md#signals)に編成されています。各Signalは、観測性のそれぞれ特殊な形を提供します。例えば、Trace、Metrics、Baggageは3つの別々のSignalです。Signalは共通のサブシステム - **Context伝搬** - を共有していますが、それらは互いに独立して機能します。

<!--
Each signal provides a mechanism for software to describe itself. A codebase, such as web framework or a database client, takes a dependency on various signals in order to describe itself. OpenTelemetry instrumentation code can then be mixed into the other code within that codebase.
This makes OpenTelemetry a **cross-cutting concern** - a piece of software which is mixed into many other pieces of software in order to provide value. Cross-cutting concerns, by their very nature, violate a core design principle – separation of concerns. As a result, OpenTelemetry client design requires extra care and attention to avoid creating issues for the codebases which depend upon these cross-cutting APIs.
-->

それぞれのSignalは、ソフトウェアがそれ自身を記述するためのメカニズムを提供します。ウェブフレームワークやデータベースクライアントなどのコードベースは、それ自身を記述するために様々なSignalに依存しています。OpenTelemetry計装のコードは、そのコードベース内の他のコードに混ぜることができます。これにより、OpenTelemetry は **横断的な関心事** - 価値を提供するために、他の多くのソフトウェアに混合されるソフトウェアの一部になります。分野横断的な関心事は、その性質上、設計の基本原則である「関心事の分離」に違反しています。その結果、OpenTelemetry クライアントの設計では、これらの横断的な API に依存するコードベースに問題が発生しないように、細心の注意を払う必要があります。

<!--
OpenTelemetry clients are designed to separate the portion of each signal which must be imported as cross-cutting concerns from the portions which can be managed independently. OpenTelemetry clients are also designed to be an extensible framework.
To accomplish these goals, each signal consists of four types of packages: API, SDK, Semantic Conventions, and Contrib.
-->

OpenTelemetryクライアントは、各信号のうち、横断的な関心事としてインポートしなければならない部分と、独立して管理できる部分を分離するように設計されています。また、OpenTelemetryクライアントは、拡張可能なフレームワークとして設計されています。これらの目標を達成するために、各Signalは4種類のパッケージで構成されています。API、SDK、セマンティック規約、Contribです。


### API

<!--
API packages consist of the cross-cutting public interfaces used for instrumentation. Any portion of an OpenTelemetry client which is imported into third-party libraries and application code is considered part of the API.
-->

APIパッケージは、計測に使用される横断的なパブリックインターフェースで構成されています。サードパーティのライブラリやアプリケーションコードにインポートされるOpenTelemetryクライアントのすべての部分は、APIの一部とみなされます。

<!--
### SDK
-->

### SDK

<!--
The SDK is the implementation of the API provided by the OpenTelemetry project. Within an application, the SDK is installed and managed by the [application owner](glossary.md#application-owner).
Note that the SDK includes additional public interfaces which are not considered part of the API package, as they are not cross-cutting concerns. These public interfaces are defined as [constructors](glossary.md#constructors) and [plugin interfaces](glossary.md#sdk-plugins).
Application owners use the SDK constructors; [plugin authors](glossary.md#plugin-author) use the SDK plugin interfaces.
[Instrumentation authors](glossary.md#instrumentation-author) MUST NOT directly reference any SDK package of any kind, only the API.
-->

SDKはOpenTelemetryプロジェクトが提供するAPIの実装です。アプリケーション内では、SDKは[アプリケーションの所有者](glossary.md#application-owner)によってインストールされ、管理されます。SDKには、APIパッケージの一部とはみなされない追加のパブリックインターフェースが含まれていますが、これらは横断的な問題ではないため、注意してください。これらのパブリックインターフェースは [コンストラクタ](glossary.md#constructors) と [プラグインインターフェース](glossary.md#sdk-plugins) として定義されています。アプリケーションの所有者は SDK のコンストラクタを使用し、[プラグイン作者](glossary.md#plugin-author) は SDK のプラグイン インターフェースを使用します。[計装の作者](glossary.md#instrumentation-author)は、SDKパッケージを直接参照してはいけません。

<!--
### Semantic Conventions
-->

### 意味規定

<!--
The **Semantic Conventions** define the keys and values which describe commonly observed concepts, protocols, and operations used by applications.
-->

**意味規定**は、アプリケーションで使用される一般的な概念、プロトコル、操作を記述するキーと値を定義します。

<!--
* [Resource Conventions](resource/semantic_conventions/README.md)
* [Span Conventions](trace/semantic_conventions/README.md)
* [Metrics Conventions](metrics/semantic_conventions/README.md)
-->

* [リソース規約](resource/semantic_conventions/README.md)
* [Span規約](trace/semantic_conventions/README.md)
* [メトリック規約](metrics/semantic_conventions/README.md)

<!--
Both the collector and the client libraries SHOULD autogenerate semantic
convention keys and enum values into constants (or language idomatic
equivalent). Generated values shouldn't be distributed in stable packages
until semantic conventions are stable.
The [YAML](../semantic_conventions/README.md) files MUST be used as the
source of truth for generation. Each language implementation SHOULD
provide language-specific support to the
[code generator](https://github.com/open-telemetry/build-tools/tree/main/semantic-conventions#code-generator).
-->

コレクターとクライアントの両方のライブラリは、セマンティック規約のキーとenumの値を定数(または言語的に同等のもの)に自動生成すべきです(SHOULD)。生成された値は、セマンティック規約が安定するまで、安定したパッケージで配布すべきではありません。[YAML](../semantic_conventions/README.md)ファイルは生成のための信頼できる唯一の情報源(source of truth)として使用されなければなりません(MUST)。各言語の実装は、[コードジェネレータ](https://github.com/open-telemetry/build-tools/tree/main/semantic-conventions#code-generator)に言語固有のサポートを提供すべきです(SHOULD)。


<!--
### Contrib Packages
-->

### Contrib パッケージ

<!--
The OpenTelemetry project maintains integrations with popular OSS projects which have been identified as important for observing modern web services.
Example API integrations include instrumentation for web frameworks, database clients, and message queues.
Example SDK integrations include plugins for exporting telemetry to popular analysis tools and telemetry storage systems.
-->

OpenTelemetry プロジェクトは、最新のウェブサービスを観測するために重要な、人気のある OSS プロジェクトとの統合を維持しています。API統合の例には、Webフレームワーク、データベースクライアント、メッセージキューのための計装が含まれます。SDK の統合例には、一般的な分析ツールやテレメトリ・ストレージ・システムにテレメトリをエクスポートするためのプラグインが含まれています。

<!--
Some plugins, such as OTLP Exporters and TraceContext Propagators, are required by the OpenTelemetry specification. These required plugins are included as part of the SDK.
-->

OTLP Exporterや TraceContext Propagators などのいくつかのプラグインは、OpenTelemetry 仕様で必要とされています。これらの必要なプラグインは、SDK の一部として含まれています。

<!--
Plugins and instrumentation packages which are optional and separate from the SDK are referred to as **Contrib** packages.
**API Contrib** refers to packages which depend solely upon the API; **SDK Contrib** refers to packages which also depend upon the SDK.
-->

SDKとは別にオプションであるプラグインやインストルメンテーションパッケージは、**Contrib**パッケージと呼ばれています。**API Contrib**はAPIのみに依存するパッケージを指し、**SDK Contrib**はSDKにも依存するパッケージを指します。

<!--
The term Contrib specifically refers to the collection of plugins and instrumentation maintained by the OpenTelemetry project; it does not refer to third-party plugins hosted elsewhere.
-->

Contribという用語は、特にOpenTelemetryプロジェクトによって維持されているプラグインと計装のコレクションを指します。

<!--
### Versioning and Stability
-->

### バージョン管理と安定性

<!--
OpenTelemetry values stability and backwards compatibility. Please see the [versioning and stability guide](./versioning-and-stability.md) for details.
-->

OpenTelemetryは安定性と下位互換性を重視しています。詳しくは [バージョン管理と安定性ガイド] (./versioning-and-stability.md) を参照してください。

<!--
## Tracing Signal
-->

## Trace Signal

<!--
A distributed trace is a set of events, triggered as a result of a single
logical operation, consolidated across various components of an application. A
distributed trace contains events that cross process, network and security
boundaries. A distributed trace may be initiated when someone presses a button
to start an action on a website - in this example, the trace will represent
calls made between the downstream services that handled the chain of requests
initiated by this button being pressed.
-->

分散Traceは、単一の論理操作の結果としてトリガーされたイベントのセットで、アプリケーションの様々なコンポーネントに統合されています。分散Traceには、プロセス、ネットワーク、セキュリティの境界を越えたイベントが含まれます。この例では、ボタンが押されたことによって開始された一連のリクエストを処理するダウンストリームサービス間の呼び出しを表します。


<!--
### Traces
-->

### Trace

<!--
**Traces** in OpenTelemetry are defined implicitly by their **Spans**. In
particular, a **Trace** can be thought of as a directed acyclic graph (DAG) of
**Spans**, where the edges between **Spans** are defined as parent/child
relationship.
-->

OpenTelemetryの**Trace**は、**Span**によって暗黙的に定義されます。特に、**Trace**は**Spans**の有向非周期グラフ(DAG)と考えることができ、**Span**間のエッジは親子関係として定義されます。

<!--
For example, the following is an example **Trace** made up of 6 **Spans**:
-->

例えば、以下は、6つの**Span**で構成された**Trace**の例です。


<!--
``` Causal relationships between Spans in a single Trace
        [Span A]  ←←←(the root span)
            |
     +------+------+
     |             |
 [Span B]      [Span C] ←←←(Span C is a `child` of Span A)
     |             |
 [Span D]      +---+-------+
               |           |
           [Span E]    [Span F]
```
-->

``` 単一Trace内のSpan間の関係
        [Span A]  ←←←(root span)
            |
     +------+------+
     |             |
 [Span B]      [Span C] ←←←(Span C はSpan Aの`child`)
     |             |
 [Span D]      +---+-------+
               |           |
           [Span E]    [Span F]
```


<!--
Sometimes it's easier to visualize **Traces** with a time axis as in the diagram
below:
-->

下の図のように、**Trace**を時間軸で可視化した方が簡単な場合もあります。

<!--
``` Temporal relationships between Spans in a single Trace

––|–––––––|–––––––|–––––––|–––––––|–––––––|–––––––|–––––––|–> time

<!--
 [Span A···················································]
   [Span B··············································]
      [Span D··········································]
    [Span C········································]
         [Span E·······]        [Span F··]
```
-->

``` 単一Trace内のSpan間の時間的関係

––|–––––––|–––––––|–––––––|–––––––|–––––––|–––––––|–––––––|–> 時間

<!--
 [Span A···················································]
   [Span B··············································]
      [Span D··········································]
    [Span C········································]
         [Span E·······]        [Span F··]
```

<!--
### Spans
-->

### Span

<!--
A span represents an operation within a transaction. Each **Span** encapsulates
the following state:
-->

Spanは、トランザクション内の操作を表します。各**Span**は、以下の状態をカプセル化しています。

<!--
- An operation name
- A start and finish timestamp
- [**Attributes**](./common/common.md#attributes): A list of key-value pairs.
- A set of zero or more **Events**, each of which is itself a tuple (timestamp, name, [**Attributes**](./common/common.md#attributes)). The name must be strings.
- Parent's **Span** identifier.
- [**Links**](#links-between-spans) to zero or more causally-related **Spans**
  (via the **SpanContext** of those related **Spans**).
- **SpanContext** information required to reference a Span. See below.
-->

- 操作名
- 開始と終了のタイムスタンプ
- [**Attribute**] (./common/common.md#attributes): キーバリューペアのリスト
- 0個以上の**Event**のセット。それぞれはそれ自体がタプル(timestamp, name, [**Attributes**](./common/common/common.md#attributes))です。名前は文字列でなければなりません
- 親の **Span** 識別子
- 因果関係のある0以上の**Span**への[**Links**](#links-between-span) (関連する **Spans** の **SpanContext** を介して)
- Spanを参照するために必要な **SpanContext** 情報。以下を参照してください


<!--
### SpanContext
-->

### SpanContext

<!--
Represents all the information that identifies **Span** in the **Trace** and
MUST be propagated to child Spans and across process boundaries. A
**SpanContext** contains the tracing identifiers and the options that are
propagated from parent to child **Spans**.
-->

**Trace**内の**Span**を識別するすべての情報を表し、子Spanおよびプロセスの境界を越えて伝播されなければなりません(MUST)。**SpanContext**は、Traceの識別子と、親から子の**Span**に伝搬されるオプションが含まれています。


<!--
- **TraceId** is the identifier for a trace. It is worldwide unique with
  practically sufficient probability by being made as 16 randomly generated
  bytes. TraceId is used to group all spans for a specific trace together across
  all processes.
- **SpanId** is the identifier for a span. It is globally unique with
  practically sufficient probability by being made as 8 randomly generated
  bytes. When passed to a child Span this identifier becomes the parent span id
  for the child **Span**.
- **TraceFlags** represents the options for a trace. It is represented as 1
  byte (bitmap).
  - Sampling bit -  Bit to represent whether trace is sampled or not (mask
    `0x1`).
- **Tracestate** carries tracing-system specific context in a list of key value
  pairs. **Tracestate** allows different vendors propagate additional
  information and inter-operate with their legacy Id formats. For more details
  see [this](https://w3c.github.io/trace-context/#tracestate-field).
-->

- **TraceId** はTraceの識別子で、ランダムに生成された16バイトであり、実質的に十分な確率で世界的に一意であることを示しています。TraceId は、特定のTraceのすべてのSpanをすべてのプロセスを対象にしています。
- **SpanId** はSpanの識別子で、ランダムに生成された8バイトであり、実質的に十分な確率で世界的に一意であることを示しています。子Spanに渡されると、この識別子はその**子Span**に対する親Spanの ID になります。
- **TraceFlags** はTraceのオプションを表します。これは、1バイト(bitmap)です。
  - サンプリングビット - Traceがサンプリングされるかどうかを表すビット(`0x1`でmask)
- **Tracestate**は、Traceシステム固有のコンテキストをキーバリューペアのリストで保持します。**Tracestate**では、異なるベンダーが追加情報を伝搬します。これによりレガシーIDフォーマットとの相互運用が可能になります。詳細については[こちら](https://w3c.github.io/trace-context/#tracestate-field)を参照してください。

<!--
### Links between spans
-->

### Span間のリンク

<!--
A **Span** may be linked to zero or more other **Spans** (defined by
**SpanContext**) that are causally related. **Links** can point to
**Spans** inside a single **Trace** or across different **Traces**.
**Links** can be used to represent batched operations where a **Span** was
initiated by multiple initiating **Spans**, each representing a single incoming
item being processed in the batch.
-->

1つの**Span**は、因果関係のある0つ以上の他の**Span**(**SpanContext**によって定義されます)にリンクされています。**Link**は、1つの**Trace**内の**Span**、または異なる**Trace**間の**Span**を指し示すことができます。**Link**は、**Span**が複数の**Span**によって開始され、それぞれが一括で処理されている1つのSignalを表している場合に、一括での処理だということを表すために使用できます。

<!--
Another example of using a **Link** is to declare the relationship between
the originating and following trace. This can be used when a **Trace** enters trusted
boundaries of a service and service policy requires the generation of a new
Trace rather than trusting the incoming Trace context. The new linked Trace may
also represent a long running asynchronous data processing operation that was
initiated by one of many fast incoming requests.
-->

**Link**を使用するもう一つの例は、発信元のTraceと後続のTraceの関係を宣言することです。これは、**Trace**がサービスの信頼された境界に入り、サービスポリシーが受信Traceコンテキストを信頼するのではなく、新しいTraceの生成を必要とする場合に使用できます。新しくリンクされたTraceは、多くの高速なSignalリクエストのうちの1つによって開始された、長い間実行されている非同期データ処理操作を表すこともできます。

<!--
When using the scatter/gather (also called fork/join) pattern, the root
operation starts multiple downstream processing operations and all of them are
aggregated back in a single **Span**. This last **Span** is linked to many
operations it aggregates. All of them are the **Spans** from the same Trace. And
similar to the Parent field of a **Span**. It is recommended, however, to not
set parent of the **Span** in this scenario as semantically the parent field
represents a single parent scenario, in many cases the parent **Span** fully
encloses the child **Span**. This is not the case in scatter/gather and batch
scenarios.
-->

Scatter/Gather(Fork/Joinとも呼ばれる)パターンを使用する場合、ルート操作は複数の下流の処理操作を開始し、それらの操作はすべて1つの**Span**に集約されます。この最後の**Span**は、集約する多くの操作にリンクされています。これらはすべて、同じTraceからの**span**です。また、**Span**の親フィールドと似ています。しかし、このシナリオでは **Span** の親を設定しないことをお勧めします。多くの場合、親の **Span** は子の **Span** を完全に囲んでいます。


<!--
## Metric Signal
-->

## Metric Signal

<!--
OpenTelemetry allows to record raw measurements or metrics with predefined
aggregation and set of labels.
-->

OpenTelemetryでは、事前に定義された集約とラベルのセットで、生の測定値やメトリックを記録することができます。

<!--
Recording raw measurements using OpenTelemetry API allows to defer to end-user
the decision on what aggregation algorithm should be applied for this metric as
well as defining labels (dimensions). It will be used in client libraries like
gRPC to record raw measurements "server_latency" or "received_bytes". So end
user will decide what type of aggregated values should be collected out of these
raw measurements. It may be simple average or elaborate histogram calculation.
-->

OpenTelemetry APIを使用して生の測定値を記録することで、ラベル(次元)を定義するだけでなく、この測定値にどのような集約アルゴリズムを適用すべきかの決定をエンドユーザーに委ねることができます。これは、"server_latency"や"received_bytes"という生の測定値を記録するために、gRPCのようなクライアントライブラリで使用されます。そのため、エンドユーザはこれらの生の測定値からどのようなタイプの集約値を収集すべきかを決定します。単純な平均値であったり、手が込んだヒストグラム計算であったりします。

<!--
Recording of metrics with the pre-defined aggregation using OpenTelemetry API is
not less important. It allows to collect values like cpu and memory usage, or
simple metrics like "queue length".
-->

OpenTelemetry APIを使って、あらかじめ定義されたアグリゲーションを使ってメトリックを記録することも重要です。これにより、CPUやメモリ使用量のような値や、"キューの長さ"のような単純なメトリックを収集することができます。
<!--
### Recording raw measurements
-->

### 生の測定値の記録

<!--
The main classes used to record raw measurements are `Measure` and
`Measurement`. List of `Measurement`s alongside the additional context can be
recorded using OpenTelemetry API. So user may define to aggregate those
`Measurement`s and use the context passed alongside to define additional
dimensions of the resulting metric.
-->

生の測定値を記録するために使われる主なクラスは `Measure` と `Measurement` です。OpenTelemetry APIを使用して、追加のコンテキストとともに `Measurement` のリストを記録することができます。そのため、ユーザはこれらの `Measurement` を集約し、結果として得られるメトリックの追加次元を定義するために渡されたコンテキストを使用するように定義することができます。

<!--
#### Measure
-->

#### Measure

<!--
`Measure` describes the type of the individual values recorded by a library. It
defines a contract between the library exposing the measurements and an
application that will aggregate those individual measurements into a `Metric`.
`Measure` is identified by name, description and a unit of values.
-->

`Measure` はライブラリによって記録された個々の値の種類を記述します。これは、測定値を公開するライブラリと、それらの個々の測定値を集約して `Metric` にするアプリケーションとの間の契約を定義する。`Measure`は、名前、説明、値の単位によって識別されます。

<!--
#### Measurement
-->

#### Measurement

<!--
`Measurement` describes a single value to be collected for a `Measure`.
`Measurement` is an empty interface in API surface. This interface is defined in
SDK.
-->

`Measurement` は `Measure` のために収集する単一の値を記述します。`Measturement`はAPIサーフェイスの中では空のインタフェースです。このインタフェースはSDKで定義されています。

<!--
### Recording metrics with predefined aggregation
-->

### あらかじめ定義された集計でメトリックを記録する

<!--
The base class for all types of pre-aggregated metrics is called `Metric`. It
defines basic metric properties like a name and labels. Classes inheriting from
the `Metric` define their aggregation type as well as a structure of individual
measurements or Points. API defines the following types of pre-aggregated
metrics:
-->

すべてのタイプの事前集計済みメトリックの基底クラスは `Metric` と呼ばれます。このクラスは、名前やラベルなどの基本的なメトリックプロパティを定義します。`Metric`を継承するクラスは、個々の測定値やPointの構造だけでなく、集約型も定義します。APIでは、以下のようなタイプの集計済みメトリックを定義しています。

<!--
- Counter metric to report instantaneous measurement. Counter values can go
  up or stay the same, but can never go down. Counter values cannot be
  negative. There are two types of counter metric values - `double` and `long`.
- Gauge metric to report instantaneous measurement of a numeric value. Gauges can
  go both up and down. The gauges values can be negative. There are two types of
  gauge metric values - `double` and `long`.
-->

- 瞬間的な測定値を報告するためのカウンターメトリック。カウンターの値は上昇したり、同じままであったりすることができますが、決して下がることはできません。カウンターの値は負にすることはできません。カウンターのメトリック値には、`double`と`long`の2種類があります。
- ゲージメトリックは、数値の瞬間的な測定値を報告するためのものです。ゲージは上にも下にも行くことができます。ゲージの値がマイナスになることもあります。ゲージのメトリック値には、`double`と`long`の2種類があります。


<!--
API allows to construct the `Metric` of a chosen type. SDK defines the way to
query the current value of a `Metric` to be exported.
-->

APIでは、選択した種類の `Metric` を作成することができます。SDKでは、エクスポートする `Metric` の現在の値を問い合わせる方法を定義しています。

<!--
Every type of a `Metric` has it's API to record values to be aggregated. API
supports both - push and pull model of setting the `Metric` value.
-->

すべてのタイプの `Metric` には、集約する値を記録するための API があります。APIは、`Metric` の値を設定するためのプッシュモデルとプルモデルの両方をサポートしています。

<!--
### Metrics data model and SDK
-->

### メトリックデータモデルとSDK

<!--
Metrics data model is [specified here](metrics/datamodel.md) and is based on
[metrics.proto](https://github.com/open-telemetry/opentelemetry-proto/blob/master/opentelemetry/proto/metrics/v1/metrics.proto).
This data model defines three semantics: An Event model used by the API, an
in-flight data model used by the SDK and OTLP, and a TimeSeries model which
denotes how exporters should interpret the in-flight model.
Different exporters have different capabilities (e.g. which data types are
supported) and different constraints (e.g. which characters are allowed in label
keys). Metrics is intended to be a superset of what's possible, not a lowest
common denominator that's supported everywhere. All exporters consume data from
Metrics Data Model via a Metric Producer interface defined in OpenTelemetry SDK.
-->

Metricsのデータモデルは[ここで指定](metrics/datamodel.md)されており、[metrics.proto](https://github.com/open-telemetry/opentelemetry-proto/blob/master/opentelemetry/proto/metrics/v1/metrics.proto)に基づいています。このデータモデルは3つのセマンティクスを定義しています。APIで使用されるイベントモデル、SDKとOTLPで使用されるインフライトデータモデル、そしてエクスポーターがインフライトのモデルをどのように解釈すべきかを示すTimeSeriesモデルです。エクスポーターによって、機能(例:どのデータタイプがサポートされているかなど)と、異なる制約(例:どのような文字れがラベルキーに許可されているか)が異なります。`Metrics`は、可能なもののスーパーセットであることを意図しており、サポートされている最小公倍数ではありません。すべてのエクスポータの機能は、OpenTelemetry SDK で定義された Metric Producer インターフェイスを介して Metricデータモデルからデータを消費します。

<!--
Because of this, Metrics puts minimal constraints on the data (e.g. which
characters are allowed in keys), and code dealing with Metrics should avoid
validation and sanitization of the Metrics data. Instead, pass the data to the
backend, rely on the backend to perform validation, and pass back any errors
from the backend.
-->

このため、Metrics はデータに最小限の制約(キーに許可されている文字など)を課し、Metrics を扱うコードは Metrics データの検証とサニタイズを避けるべきです。その代わりに、バックエンドにデータを渡し、バックエンドに頼って検証を行い、バックエンドからのエラーはすべてパスバックしてください。

<!--
See [Metrics Data Model Specification](metrics/datamodel.md) for more
information.
-->

詳細は、[Metrics データモデル仕様](metrics/datamodel.md)を参照してください。

<!--
## Log Signal
-->

## Log Signal

<!--
### Data model
-->

### Data model

<!--
[Log Data Model](logs/data-model.md) defines how logs and events are understood by
OpenTelemetry.
-->

[Log Data Model](logs/data-model.md)は、OpenTelemetryがログやイベントをどのように理解するかを定義します。

<!--
## Baggage Signal
-->

## Baggage Signal

<!--
In addition to trace propagation, OpenTelemetry provides a simple mechanism for propagating
name/value pairs, called `Baggage`. `Baggage` is intended for
indexing observability events in one service with attributes provided by a prior service in
the same transaction. This helps to establish a causal relationship between these events.
-->

Traceの伝播に加えて、OpenTelemetryは名前と値のペアを伝播するためのシンプルなメカニズムを提供します。`Baggage` は、あるサービスの観測可能イベントを、同じトランザクション内で先行するサービスが提供する属性とインデックス化することを目的としています。これにより、これらのイベント間の因果関係を確立するのに役立ちます。

<!--
While `Baggage` can be used to prototype other cross-cutting concerns, this mechanism is primarily intended
to convey values for the OpenTelemetry observability systems.
-->

`Baggage` は他の横断的な関心事をプロトタイプ化するために使うことができますが、このメカニズムは主に OpenTelemetry の観測システムの値を伝えることを目的としています。

<!--
These values can be consumed from `Baggage` and used as additional dimensions for metrics,
or additional context for logs and traces. Some examples:
-->

これらの値は `Baggage` から消費され、メトリックの追加ディメンジョンとして、あるいはログやトレースの追加コンテキストとして使用することができます。いくつかの例を示します。

<!--
- a web service can benefit from including context around what service has sent the request
- a SaaS provider can include context about the API user or token that is responsible for that request
- determining that a particular browser version is associated with a failure in an image processing service
-->

- Webサービスでは、どのサービスがリクエストを送信したかのコンテキストを含められます
- SaaS プロバイダは、そのリクエストを担当する API ユーザーまたはトークンに関するコンテキストを含められます
- 特定のブラウザのバージョンが画像処理サービスの障害に関連しているかどうかの判断できます


<!--
For backward compatibility with OpenTracing, Baggage is propagated as `Baggage` when
using the OpenTracing bridge. New concerns with different criteria should consider creating a new
cross-cutting concern to cover their use-case; they may benefit from the W3C encoding format but
use a new HTTP header to convey data throughout a distributed trace.
-->

OpenTracingとの下位互換性のために、BaggageはOpenTracingブリッジを使用する際に`Baggage`として伝搬されます。異なる基準を持つ新しい懸念事項は、そのユースケースをカバーするために新しい横断的な懸念事項を作成することを検討すべきです。

<!--
## Resources
-->

## リソース

<!--
`Resource` captures information about the entity for which telemetry is
recorded. For example, metrics exposed by a Kubernetes container can be linked
to a resource that specifies the cluster, namespace, pod, and container name.
-->

`リソース`は、テレメトリが記録されるエンティティに関する情報をキャプチャします。例えば、Kubernetesコンテナが公開するメトリックは、クラスタ、ネームスペース、ポッド、コンテナ名を指定するリソースにリンクできます。

<!--
`Resource` may capture an entire hierarchy of entity identification. It may
describe the host in the cloud and specific container or an application running
in the process.
-->

`リソース`はエンティティ識別の階層全体を捉えることができます。これは、クラウド上のホストや特定のコンテナ、あるいはプロセス内で実行されているアプリケーションを記述することができます。

<!--
Note, that some of the process identification information can be associated with
telemetry automatically by OpenTelemetry SDK or specific exporter. See
OpenTelemetry
[proto](https://github.com/open-telemetry/opentelemetry-proto/blob/a46c815aa5e85a52deb6cb35b8bc182fb3ca86a0/src/opentelemetry/proto/agent/common/v1/common.proto#L28-L96)
for an example.
-->

プロセス識別情報の一部は、OpenTelemetry SDKまたは特定のエクスポータによって自動的にテレメトリと関連付けることができることに注意してください。例として OpenTelemetry [proto](https://github.com/open-telemetry/opentelemetry-proto/blob/a46c815aa5e85a52deb6cb35b8bc182fb3ca86a0/src/opentelemetry/proto/agent/common/v1/common.proto#L28-L96) を参照してください。

<!--
## Context Propagation
-->

## Context 伝搬

<!--
All of OpenTelemetry cross-cutting concerns, such as traces and metrics,
share an underlying `Context` mechanism for storing state and
accessing data across the lifespan of a distributed transaction.
-->

`Trace`や`Metric`などの OpenTelemetry の横断的な関心事はすべて、分散トランザクションのライフSpanにわたって状態を保存し、データにアクセスするための基盤となる `Context` メカニズムを共有しています。

<!--
See the [Context](context/context.md)
-->

[Context](context/context.md)を参照してください。

<!--
## Propagators
-->

## Propagator

<!--
OpenTelemetry uses `Propagators` to serialize and deserialize cross-cutting concern values
such as `Span`s (usually only the `SpanContext` portion) and `Baggage`. Different `Propagator` types define the restrictions
imposed by a specific transport and bound to a data type.
-->

OpenTelemetry は `Propagator` を使用して、`Span` (通常は `SpanContext` の部分のみ) や `Baggage` などの横断的な関心事の値をシリアライズしたり、デシリアライズしたりします。異なる `Propagator` の種類は、特定のトランスポートによって課せられる制限を定義し、データ型にバインドします。

<!--
The Propagators API currently defines one `Propagator` type:
-->

Propagators APIは現在1つの `Propagator` タイプを定義しています。

<!--
- `TextMapPropagator` injects values into and extracts values from carriers as text.
-->

- `TextMapPropagator` はキャリアに値を注入したり、キャリアから値をテキストとして抽出したりする

<!--
## Collector
-->

## コレクター(Collector)

<!--
The OpenTelemetry collector is a set of components that can collect traces,
metrics and eventually other telemetry data (e.g. logs) from processes
instrumented by OpenTelementry or other monitoring/tracing libraries (Jaeger,
Prometheus, etc.), do aggregation and smart sampling, and export traces and
metrics to one or more monitoring/tracing backends. The collector will allow to
enrich and transform collected telemetry (e.g. add additional attributes or
scrub personal information).
-->

OpenTelemetry コレクターは、OpenTelemetry や他のモニタリング/トレースライブラリ (Jaeger, Prometheus など) によって計測されたプロセスからTraceやMetric、最終的には他のテレメトリデータ (ログなど) を収集し、集約やスマートサンプリングを行い、TraceやMetricを 1 つまたは複数のモニタリング/トレースバックエンドにエクスポートすることができるコンポーネントのセットです。コレクターは、収集したテレメトリに情報を追加したり、変換したりすることができます(例:属性の追加や個人情報のスクラブ)。

<!--
The OpenTelemetry collector has two primary modes of operation: Agent (a daemon
running locally with the application) and Collector (a standalone running
service).
-->

OpenTelemetry コレクターには、主に 2 つの動作モードがあります。エージェント (アプリケーションと共にローカルで実行されるデーモン) とコレクター (スタンドアロンで実行されるサービス) です。

<!--
Read more at OpenTelemetry Service [Long-term
Vision](https://github.com/open-telemetry/opentelemetry-collector/blob/master/docs/vision.md).
-->

詳細はOpenTelemetry Service [長期ビジョン](https://github.com/open-telemetry/opentelemetry-collector/blob/master/docs/vision.md)を参照してください。

<!--
## Instrumentation Libraries
-->

## 計装ライブラリ

<!--
See [Instrumentation Library](glossary.md#instrumentation-library)
-->

[計装ライブラリ](glossary.md#instrumentation-library)を参照してください。

<!--
The inspiration of the project is to make every library and application
observable out of the box by having them call OpenTelemetry API directly. However,
many libraries will not have such integration, and as such there is a need for
a separate library which would inject such calls, using mechanisms such as
wrapping interfaces, subscribing to library-specific callbacks, or translating
existing telemetry into the OpenTelemetry model.
-->

このプロジェクトの考えは、OpenTelemetry API を直接呼び出すことで、すべてのライブラリやアプリケーションをすぐに観測可能にすることです。しかし、多くのライブラリはそのような統合を持っていないでしょう。そのため、インターフェースのラップ、ライブラリ固有のコールバックのサブスクライブ、既存のテレメトリをOpenTelemetryモデルに変換するなどのメカニズムを使って、そのような呼び出しを注入する、別のライブラリが必要になります。

<!--
A library that enables OpenTelemetry observability for another library is called
an [Instrumentation Library](glossary.md#instrumentation-library).
-->

OpenTelemetryの観測を別のライブラリで可能にするライブラリを[計装ライブラリ(Instrumentation Library)](glossary.md#instrumentation-library)と呼びます。

<!--
An instrumentation library should be named to follow any naming conventions of
the instrumented library (e.g. 'middleware' for a web framework).
-->

計装ライブラリは、計装ライブラリの命名規則に従うように命名されなければなりません (例: ウェブフレームワークの'ミドルウェア'のように)。

<!--
If there is no established name, the recommendation is to prefix packages
with "opentelemetry-instrumentation", followed by the instrumented library
name itself. Examples include:
-->

確立された名前がない場合は、パッケージの前に "opentelemetry-instrumentation" を付け、その後に計装されたライブラリ名そのものを付けることをお勧めします。例としては、以下のようなものがあります。

<!--
* opentelemetry-instrumentation-flask (Python)
* @opentelemetry/instrumentation-grpc (Javascript)
-->

* opentelemetry-instrumentation-flask (Python)
* @opentelemetry/instrumentation-grpc (Javascript)

