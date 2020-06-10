<!--
# Overview
-->

# 概要

<!--
## Distributed Tracing
-->

## 分散トレーシング

<!--
A distributed trace is a set of events, triggered as a result of a single
logical operation, consolidated across various components of an application. A
distributed trace contains events that cross process, network and security
boundaries. A distributed trace may be initiated when someone presses a button
to start an action on a website - in this example, the trace will represent
calls made between the downstream services that handled the chain of requests
initiated by this button being pressed.
-->

分散トレーシングは、一つのアプリケーション中の様々なコンポーネントをまたいで行われた、論理的に単一の操作の結果によって起こされた一連のイベントです。
分散トレーシングには、プロセス、ネットワーク、セキュリティの境界を越えたイベントが含まれます。例えば、分散トレーシングは誰かがWebサイトのボタンを押すことで開始されます。この例において、Traceはボタンが押されたことで開始される一連のリクエストを処理するダウンストリームのサービスを表現しています。

### Trace

<!--
**Traces** in OpenTelemetry are defined implicitly by their **Spans**. In
particular, a **Trace** can be thought of as a directed acyclic graph (DAG) of
**Spans**, where the edges between **Spans** are defined as parent/child
relationship.
-->

OpenTelemetryの **Trace** は、 **Span** によって暗黙的に定義されます。 特に、 **Trace** は **Span** の有向非巡回グラフ(DAG)と考えることができ、**Span** 間のエッジは親子関係として定義されます。

<!--
For example, the following is an example **Trace** made up of 6 **Spans**:
-->

例えば、以下は、6つの **Span** で構成された **Trace** の例です。


```
単一TraceにおけるSpan間の因果関係

        [Span A]  ←←←(大本のspan)
            |
     +------+------+
     |             |
 [Span B]      [Span C] ←←←(Span C は Span A の `子`)
     |             |
 [Span D]      +---+-------+
               |           |
           [Span E]    [Span F]
```

<!--
Sometimes it's easier to visualize **Traces** with a time axis as in the diagram
below:
-->

下の図のように、**Trace** を時間軸で可視化した方が簡単な場合もあります。

```
単一TraceにおけるSpan間の時間的関係

––|–––––––|–––––––|–––––––|–––––––|–––––––|–––––––|–––––––|–> 時間

 [Span A···················································]
   [Span B··············································]
      [Span D··········································]
    [Span C········································]
         [Span E·······]        [Span F··]
```

### Span

<!--
Each **Span** encapsulates the following state:
-->

各 **Span** は、以下の状態をカプセル化しています。

<!--
- An operation name
- A start and finish timestamp
- A set of zero or more key:value **Attributes**. The keys must be strings. The
  values may be strings, bools, or numeric types.
- A set of zero or more **Events**, each of which is itself a key:value map
  paired with a timestamp. The keys must be strings, though the values may be of
  the same types as Span **Attributes**.
- Parent's **Span** identifier.
- [**Links**](#links-between-spans) to zero or more causally-related **Spans**
  (via the **SpanContext** of those related **Spans**).
- **SpanContext** identification of a Span. See below.
-->

- 操作名
- 開始と終了のタイムスタンプ
- 0個以上の key:value ペアの **Attribute**。キーは文字列でなければいけません。値は文字列、Boolあるいは数字です
- 0個以上の **Event**。それぞれのEventはkey:valueペアとタイムスタンプを持ちます。キーは文字列でなければいけません。値はSpanの **Attribute** と同じ型です
- 親 **Span** 識別子
- 0個以上の何らかの関係がある **Span** への [**Link**](#span間のlink) (関連する **Span** の **SpanContext** を通じてLinkします)
- Spanの **SpanContext** 識別子。次の項を参照してください

### SpanContext

<!--
Represents all the information that identifies **Span** in the **Trace** and
MUST be propagated to child Spans and across process boundaries. A
**SpanContext** contains the tracing identifiers and the options that are
propagated from parent to child **Spans**.
-->

SpanContxtは **Trace** 内の **Span** を識別するすべての情報を表し、子Spanおよびプロセスの境界を越えて伝播されなければなりません(MUST)。 **SpanContext**には、Traceの識別子と、親から子の**Span**に伝搬されるオプションが含まれています。

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

- **TraceId**はTraceの識別子です。ランダムに生成された16バイトで構成され、実質的に十分な確率でグローバルに一意です。TraceId は、特定のTraceのすべてのSpanをすべてのプロセスにまたがってグループ化するために使用されます。
- **SpanId** はSpanの識別子です。ランダムに生成された 8 バイトで構成され、実質的に十分な確率でグローバルに一意です。SpanIdを子 **Span** に渡すと、その子Spanの親Span IDになります。
- **TraceFlags** は、Traceのオプションを表します。1 バイト(ビットマップ)で表されます。
  - サンプリングビット - Traceがサンプリングされるかどうかを表すビット (マスク `0x1`)
- **Tracestate**は、トレースシステム固有のコンテキストをキー値のペアのリストに格納します。**Tracestate**は、異なるベンダーが追加情報を伝搬し、それらのレガシーIDフォーマットとの相互運用を可能にします。詳細は [こちら](https://w3c.github.io/trace-context/#tracestate-field) を参照してください。

<!--
### Links between spans
-->

### Span間のLink

<!--
A **Span** may be linked to zero or more other **Spans** (defined by
**SpanContext**) that are causally related. **Links** can point to
**SpanContexts** inside a single **Trace** or across different **Traces**.
**Links** can be used to represent batched operations where a **Span** was
initiated by multiple initiating **Spans**, each representing a single incoming
item being processed in the batch.
-->

1つの**Span**は、因果関係のある0個以上の他の**Span** (**SpanContext**によって定義されます) にLinkすることができます。**Link**は、単一の**Trace**内の**SpanContext**、または異なる**Trace**間の**SpanContext**を指すことができます。**Link**は、**Span**が複数の**Span**によって開始され、それぞれがバッチで処理されている1つのアイテムを表している場合に、バッチ処理を表すために使用できます。(???incoming itemがなにを示しているか分からない)

<!--
Another example of using a **Link** is to declare the relationship between
the originating and following trace. This can be used when a **Trace** enters trusted
boundaries of a service and service policy requires the generation of a new
Trace rather than trusting the incoming Trace context. The new linked Trace may
also represent a long running asynchronous data processing operation that was
initiated by one of many fast incoming requests.
-->

**Link**を使用するもう一つの例は、開始元のTraceと後続のTraceの関係を宣言することです。これは、**Trace**がサービスの信頼された境界に入り、サービスポリシーが受信トレースコンテキストを信頼するのではなく、新しいTraceの生成を必要とする場合に使用できます。新しくLinkされたTraceは、多くの高速なリクエストのうちの1つによって開始された、長い間実行されている非同期データ処理操作を表すこともできます。

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

scatter/gather(fork/joinとも呼ばれます)パターンを使用する場合、大本の操作から複数の下流の処理操作が開始され、それらの操作はすべて1つの**Span**に集約されます。この最後の**Span**は、集約する多くの操作にLinkされています。これらはすべて、同じTraceからの**span**です。また、**Span**の親フィールドと似ています。しかし、このシナリオでは親フィールドは一つの親を意味的に表すシナリオのため、**Span** の親を設定しないことをお勧めします。多くの場合親の **Span** は子の **Span** を完全に含んでいるおり、scatter/gatherやバッチのシナリオではこのようなことはありません。(???ちょっと文章構造が難解)

<!--
## Metrics
-->

## メトリック

<!--
OpenTelemetry allows to record raw measurements or metrics with predefined
aggregation and set of labels.
-->

OpenTelemetryでは、事前に定義された集約とラベルのセットにより、生の測定値やメトリックを記録することができます。

<!--
Recording raw measurements using OpenTelemetry API allows to defer to end-user
the decision on what aggregation algorithm should be applied for this metric as
well as defining labels (dimensions). It will be used in client libraries like
gRPC to record raw measurements "server_latency" or "received_bytes". So end
user will decide what type of aggregated values should be collected out of these
raw measurements. It may be simple average or elaborate histogram calculation.
-->

OpenTelemetry APIを使用して生の測定値を記録することで、ラベル(次元)を定義するだけでなく、このメトリックにどのような集約アルゴリズムを適用すべきかの決定をエンドユーザーに委ねることができます。これはgRPCのようなクライアントライブラリで使用され、生の測定値 "server_latency"や "received_bytes"を記録するために使用されます。そのため、エンドユーザーは、これらの生の測定値からどのような種類の集計値を収集すべきかを決定します。単純な平均値であったり、精巧なヒストグラム計算であったりします。

<!--
Recording of metrics with the pre-defined aggregation using OpenTelemetry API is
not less important. It allows to collect values like cpu and memory usage, or
simple metrics like "queue length".
-->

OpenTelemetry APIを使用してあらかじめ定義された集計でメトリックを記録することは、それほど重要ではありません。OpenTelemetry APIにより、CPUやメモリ使用量のような値や、あるいは"キューの長さ"のような単純なメトリックを収集することができます。

<!--
### Recording raw measurements
-->

### 生の測定値を記録

<!--
The main classes used to record raw measurements are `Measure` and
`Measurement`. List of `Measurement`s alongside the additional context can be
recorded using OpenTelemetry API. So user may define to aggregate those
`Measurement`s and use the context passed alongside to define additional
dimensions of the resulting metric.
-->

生の測定値を記録するためによく使われるクラスは `Measure` と `Measurement`です。OpenTelemetry APIを使用して、追加のコンテキストと一緒に`Measurement`のリストを記録することができます。そのため、ユーザーはこれらの`Measurement`を集約するように定義し、その際に渡されるコンテキストを使用して、結果として得られるメトリックの追加の次元を定義することができます。

#### Measure

<!--
`Measure` describes the type of the individual values recorded by a library. It
defines a contract between the library exposing the measurements and an
application that will aggregate those individual measurements into a `Metric`.
`Measure` is identified by name, description and a unit of values.
-->

`Measure` はライブラリによって記録される個々の値の種類を記述します。これは、測定値を公開するライブラリと、それらの個々の測定値を `Metric` に集約するアプリケーションとの間の契約を定義します。`Measure` は、名前、説明、値の単位によって識別されます。

#### Measurement

<!--
`Measurement` describes a single value to be collected for a `Measure`.
`Measurement` is an empty interface in API surface. This interface is defined in
SDK.
-->

`Measurement` は `Measure` のために収集する単一の値を表します。 `Measurement` はAPIに対する空のインターフェイスです(???ここでのSurfaceとはどういう意味か)。このインタフェースはSDKで定義されています。

<!--
### Recording metrics with predefined aggregation
-->

### 事前定義された手法でメトリックを集計し、記録する

<!--
The base class for all types of pre-aggregated metrics is called `Metric`. It
defines basic metric properties like a name and labels. Classes inheriting from
the `Metric` define their aggregation type as well as a structure of individual
measurements or Points. API defines the following types of pre-aggregated
metrics:
-->

事前定義された手法で集計したメトリックの基底クラスは `Metric` と呼ばれます。 `Metric` は名前やラベルなどの基本的なメトリックのプロパティを定義します。`Metric` から継承されるクラスは、集計の種類と個々の測定値あるいは Point の構造を定義します。APIでは以下の事前定義された集計手法が定義されています。

<!--
- Counter metric to report instantaneous measurement. Counter values can go
  up or stay the same, but can never go down. Counter values cannot be
  negative. There are two types of counter metric values - `double` and `long`.
- Gauge metric to report instantaneous measurement of a numeric value. Gauges can
  go both up and down. The gauges values can be negative. There are two types of
  gauge metric values - `double` and `long`.
-->

- 瞬間的なメトリックを表すためのカウンターメトリック。カウンターの値は上がることもあれば変わらないこともありますが、決して下がることはありません。カウンターの値はマイナスにはなりません。カウンターの値には、`double` と `long` の二種類の型があります。
- 数値の瞬間的なメトリックを表すためのゲージメトリック。ゲージは上がることも下がることもあります。また、マイナスになることもあります。ゲージの値には、 `double` と `long` の二種類の型があります。

<!--
API allows to construct the `Metric` of a chosen type. SDK defines the way to
query the current value of a `Metric` to be exported.
-->

APIでは、選んだ型の `Metric` を使えます。SDKは、送信される `Metric` の現在値を問い合わせる方法を定義します。

<!--
Every type of a `Metric` has it's API to record values to be aggregated. API
supports both - push and pull model of setting the `Metric` value.
-->

すべての種類の `Metric` には、集約する値を記録するAPIがあります。APIは `Metric` の値を設定する方法としてプッシュ型とプル型の両方をサポートしています。

<!--
### Metrics data model and SDK
-->

### メトリックのデータモデルとSDK


<!--
Metrics data model is defined in SDK and is based on
[metrics.proto](https://github.com/open-telemetry/opentelemetry-proto/blob/master/opentelemetry/proto/metrics/v1/metrics.proto).
This data model is used by all the OpenTelemetry exporters as an input.
Different exporters have different capabilities (e.g. which data types are
supported) and different constraints (e.g. which characters are allowed in label
keys). Metrics is intended to be a superset of what's possible, not a lowest
common denominator that's supported everywhere. All exporters consume data from
Metrics Data Model via a Metric Producer interface defined in OpenTelemetry SDK.
-->

メトリックのデータモデルは、[metrics.proto](https://github.com/open-telemetry/opentelemetry-proto/blob/master/opentelemetry/proto/metrics/v1/metrics.proto) に基づいてSDKで定義されています。このデータモデルはすべてのOpenTelemetryのExporterに対する入力として使われます。Exporterごとに、サポートされているデータ型などの異なる機能と、ラベルキーで使用可能な文字種類などの異なる制約があります。メトリックは、可能なことのスーパーセットであることを意図しており、どこでもサポートされている最低の共通分母ではありません。すべてのExporterは、OpenTelemetry SDK で定義された Metric Producer インターフェースを介して Metrics Data Model からデータを取得します。

<!--
Because of this, Metrics puts minimal constraints on the data (e.g. which
characters are allowed in keys), and code dealing with Metrics should avoid
validation and sanitization of the Metrics data. Instead, pass the data to the
backend, rely on the backend to perform validation, and pass back any errors
from the backend.
-->

このため、メトリックはデータにキーで許可される文字などの最小限の制約を課すだけにし、Metrics を扱うコードは Metrics データのバリデーションとサニタイズを避けるべきです(SHOULD)。その代わりに、バックエンドにデータを渡し、バックエンドによる検証を信頼し、バックエンドからのエラーはすべて送り返してください。(??? pass backの訳が不明)

## CorrelationContext

<!--
In addition to trace propagation, OpenTelemetry provides a simple mechanism for propagating
name/value pairs, called `CorrelationContext`. `CorrelationContext` is intended for
indexing observability events in one service with attributes provided by a prior service in
the same transaction. This helps to establish a causal relationship between these events.
-->

Traceの伝播に加えて、OpenTelemetryは名前と値のペアを伝播するための `CorrelationContext` と呼ばれるシンプルなメカニズムを提供します。`CorrelationContext` は、あるサービスの観測可能イベントを、同じトランザクション内で先行するサービスが提供する属性を含めてインデックス化することを目的としています。これにより、イベントの間に相関関係を構築できます。

<!--
The `CorrelationContext` implements the editor's draft of the [W3C Correlation-Context specification](https://w3c.github.io/correlation-context/).
While `CorrelationContext` can be used to prototype other cross-cutting concerns, this mechanism is primarily intended
to convey values for the OpenTelemetry observability systems.
-->

`CorrelationContext` はドラフト状態の [W3C Correlation-Context 仕様](https://w3c.github.io/correlation-context/)を実装しています。`CorrelationContext` は他の横断的な関心事をプロトタイプ化するために使うことができますが、このメカニズムは主にOpenTelemetryの観測システムのために値を伝えることを意図しています。(???プロトタイプの意味、observability systemsの訳語)

<!--
These values can be consumed from `CorrelationContext` and used as additional dimensions for metrics,
or additional context for logs and traces. Some examples:
-->

これらの値は `CorrelationContext` から消費され、メトリックの追加次元として、あるいはログやTraceの追加コンテキストとして使用することができます。以下に例をあげます:

<!--
- a web service can benefit from including context around what service has sent the request
- a SaaS provider can include context about the API user or token that is responsible for that request
- determining that a particular browser version is associated with a failure in an image processing service
-->

- Webサービスはどのサービスがそのリクエストを送ったのかという周辺のコンテキストを含むことで便利になります
- SaaSプロバイダーは、そのリクエストを担当する API ユーザーまたはトークンに関するコンテキストを含めることができます
- 特定のブラウザのバージョンが画像処理サービスの障害に関連しているかどうかの判断に使えます

<!--
For backward compatibility with OpenTracing, Baggage is propagated as `CorrelationContext` when
using the OpenTracing bridge. New concerns with different criteria should consider creating a new
cross-cutting concern to cover their use-case; they may benefit from the W3C encoding format but
use a new HTTP header to convey data throughout a distributed trace.
-->

OpenTracingとの下位互換性のために、OpenTracingブリッジを使用する際にBaggageが`CorrelationContext`として伝搬されます。異なる基準を持つ新しい関心事は、そのユースケースをカバーするために、新しい横断的な関心事を作成することを検討するべきです; W3Cエンコーディングフォーマットの恩恵はあるかもしれませんが、分散トレーシング全体でデータを伝えるために新しいHTTPヘッダーを使用します。

## Resources

<!--
`Resource` captures information about the entity for which telemetry is
recorded. For example, metrics exposed by a Kubernetes container can be linked
to a resource that specifies the cluster, namespace, pod, and container name.
-->

`Resource`は、どのテレメトリーが記録されるかエンティティに関する情報を記録します。例えば、Kubernetesコンテナが公開するメトリックは、クラスター、ネームスペース、ポッド、コンテナ名を指定するリソースにリンクすることができます。

<!--
`Resource` may capture an entire hierarchy of entity identification. It may
describe the host in the cloud and specific container or an application running
in the process.
-->

`Resource` はエンティティ識別子の階層構造全体を記録することができます。クラウド上のホストや特定のコンテナ、あるいはプロセス内で実行されているアプリケーションを表現することができます。

<!--
Note, that some of the process identification information can be associated with
telemetry automatically by OpenTelemetry SDK or specific exporter. See
OpenTelemetry
[proto](https://github.com/open-telemetry/opentelemetry-proto/blob/a46c815aa5e85a52deb6cb35b8bc182fb3ca86a0/src/opentelemetry/proto/agent/common/v1/common.proto#L28-L96)
for an example.
-->

プロセスの識別子情報の一部はOpenTelemetry SDKあるいは特定のExporterによって自動的にテレメトリーに関連付けられることに注意してください。具体的な例はOpenTelemetry [protoファイル](https://github.com/open-telemetry/opentelemetry-proto/blob/a46c815aa5e85a52deb6cb35b8bc182fb3ca86a0/src/opentelemetry/proto/agent/common/v1/common.proto#L28-L96)を参照してください。


<!--
## Context Propagation
-->

## Context 伝搬

<!--
All of OpenTelemetry cross-cutting concerns, such as traces and metrics,
share an underlying `Context` mechanism for storing state and
accessing data across the lifespan of a distributed transaction.
-->

Traceやメトリックなどの OpenTelemetry の横断的な関心事はすべて、分散トランザクション全体にわたって状態を保存し、データにアクセスするための基盤となる `Context` メカニズムを共有しています。

<!--
See the [Context](context/context.md)
-->

[Context](context/context.md) を参照してください。

## Propagators

<!--
OpenTelemetry uses `Propagators` to serialize and deserialize cross-cutting concern values
such as `SpanContext` and `CorrelationContext` into a `Format`. Currently there is one
type of propagator:
-->

OpenTelmetryは、`SpanContext` と `CorrelationContext` といった横断的な関心事を `Format` にシリアライズあるいはデシリアライズするために `Propagator` を使います。現時点では Propagator は一つだけです。

<!--
- `HTTPTextFormat` which is used to inject and extract a value as text into carriers that travel
  in-band across process boundaries.
-->

- `HTTPTextFormat` は、プロセスを越えてインバンドで移動するキャリアに文字列として値を注入したり抽出したりするために使用されます (??? carrierとは？in-bandとは？)

## Collector

<!--
The OpenTelemetry collector is a set of components that can collect traces,
metrics and eventually other telemetry data (e.g. logs) from processes
instrumented by OpenTelementry or other monitoring/tracing libraries (Jaeger,
Prometheus, etc.), do aggregation and smart sampling, and export traces and
metrics to one or more monitoring/tracing backends. The collector will allow to
enrich and transform collected telemetry (e.g. add additional attributes or
scrub personal information).
-->

OpenTelemetry Collectorは、OpenTelementryやJaeger、Prometheusなど他のモニタリング/トレースライブラリによって計装されたプロセスから、トレース、メトリック、そして最終的には他のテレメトリーデータ (例: ログ) を収集することができる一連のコンポーネントです。Collectorは集計、賢いサンプリング、トレースとメトリックを一つ以上のモニタリング/トレースバックエンドに対して送信することを行います。Collectorは収集したテレメトリーに対して、属性を付け加えたり個人情報を削除するなど、情報の付加や変換を行えます。

<!--
The OpenTelemetry collector has two primary modes of operation: Agent (a daemon
running locally with the application) and Collector (a standalone running
service).
-->

OpenTelemetry Collectorには、主な動作モードが2つあります。Agent (アプリケーションと一緒にローカルで実行されるデーモン) とCollector (スタンドアロンで実行されるサービス) です。

<!--
Read more at OpenTelemetry Service [Long-term
Vision](https://github.com/open-telemetry/opentelemetry-collector/blob/master/docs/vision.md).
-->

OpenTelemetry Serviceの[長期ビジョン](https://github.com/open-telemetry/opentelemetry-collector/blob/master/docs/vision.md)も参照してください。

<!--
## Instrumentation adapters
-->

## 計装用アダプター

<!--
The inspiration of the project is to make every library and application
manageable out of the box by instrumenting it with OpenTelemetry. However on the
way to this goal there will be a need to enable instrumentation by plugging
instrumentation adapters into the library of choice. These adapters can be
wrapping library APIs, subscribing to the library-specific callbacks or
translating telemetry exposed in other formats into OpenTelemetry model.
-->

このプロジェクトの創造性(???inspirationの訳は？)は、OpenTelemetryを使って計測することで、すべてのライブラリやアプリケーションをすぐに管理可能な状態にすることです。しかし、この目標を実現するには選んだライブラリに計装用アダプターを接続して計装を可能にする必要があります。計装用アダプターはライブラリAPIをラップしたり、ライブラリ固有のコールバックを購読したり、他の形式で公開されているテレメトリーをOpenTelemetryモデルに変換したりすることができます。

<!--
Instrumentation adapters may be called different names. It is often referred as
plugin, collector or auto-collector, telemetry module, bridge, etc. It is always
recommended to follow the library and language standards. For instance, if
instrumentation adapter is implemented as "log appender" - it will probably be
called an `appender`, not an instrumentation adapter. However if there is no
established name - the recommendation is to call packages "Instrumentation
Adapter" or simply "Adapter".
-->

計装用アダプターは違う名前で呼ばれることがあります。プラグイン、コレクターあるいは自動コレクター、テレメトリーモジュール、ブリッジなどです。ライブラリと言語の標準に従うことが望ましいです。例えばもし計装用アダプターが "log appender" として実装されているのであれば、計装用アダプターではなく "appender" という名前で呼ばれるでしょう。しかし、もしも確固とした名前がない場合、 推奨されるのは、"計装用アダプター"あるいは単に"アダプター"と呼ぶことです。

<!--
## Code injecting adapters
-->

## コード注入アダプタ

<!--
TODO: fill out as a result of SIG discussion.
-->

TODO: SIGの議論の結果として記入します。

<!--
## Semantic Conventions
-->

## 意味規則 (???Semantic Conventionsの訳として良いか？)

<!--
OpenTelemetry defines standard names and values of Resource attributes and
Span attributes.
-->

OpenTelemetryは標準的な名前、ResourceのAttributeとSpanのAttributeについて定義しています。

<!--
* [Resource Conventions](resource/semantic_conventions/README.md)
* [Span Conventions](trace/semantic_conventions/README.md)
* [Metrics Conventions](metrics/semantic_conventions/README.md)
-->

* [Resource に関する意味規則](resource/semantic_conventions/README.md)
* [Span に関する意味規則](trace/semantic_conventions/README.md)
* [Metrics に関する意味規則](metrics/semantic_conventions/README.md)

<!--
The type of the attribute SHOULD be specified in the semantic convention
for that attribute. Array values are allowed for attributes. For
protocols that do not natively support array values such values MUST be
represented as JSON strings.
-->

これらのAttributeの型は、そのAttributeの意味規定で指定されるべきです(SHOULD)。Attributeには配列型を使えます。配列値をネイティブにサポートしていないプロトコルでは、そのような値は JSON 文字列として表現されなければなりません(MUST)。
