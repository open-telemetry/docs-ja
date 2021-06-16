<!--
# Metrics Data Model
-->

# Metrics データモデル

**Status**: [Experimental](../document-status.md)

<!-- Re-generate TOC with `markdown-toc --no-first-h1 -i` -->

<!-- toc -->

<!-- tocstop -->

<!--
## Overview
-->

## 概要

<!--
The OpenTelemetry data model for metrics consists of a protocol specification
and semantic conventions for delivery of pre-aggregated metric timeseries data.
The data model is designed for importing data from existing systems and
exporting data into existing systems, as well as to support internal
OpenTelemetry use-cases for generating Metrics from streams of Spans or Logs.
-->

Metrics向けのOpenTelemetryデータモデルは、事前に集約されたMetricsの時系列データを配信するためのプロトコル仕様とセマンティック規約で構成されています。このデータモデルは、既存のシステムからデータをインポートしたり、既存のシステムにデータをエクスポートしたり、SpanやログのストリームからMetricsを生成するOpenTelemetry内部のユースケースをサポートするために設計されています。

<!--
Popular existing metrics data formats can be unambiguously translated into the
OpenTelemetry data model for metrics, without loss of semantics or fidelity.
Translation from the Prometheus and Statsd exposition formats is explicitly
specified.
-->

既存の一般的なメトリックデータ形式は、セマンティクスや正確さを損なうことなく、OpenTelemetryのMetricsデータモデルに明確に変換することができます。PrometheusやStatsdの出力フォーマットからの変換は、明示的に指定されています。

<!--
The data model specifies a number of semantics-preserving data transformations
for use on the collection path, supporting flexible system configuration. The
model supports reliability and statelessness controls, through the choice of
cumulative and delta transport. The model supports cost controls, through
spatial and temporal reaggregation.
-->

データモデルは、収集経路で使用するためのセマンティクスを保持したデータ変換を多数規定しており、柔軟なシステム構成をサポートしています。このモデルは、累積転送とデルタ転送の選択により、信頼性とステートレス性の制御をサポートします。このモデルは、空間的および時間的な再集計により、コスト管理をサポートします。

<!--
The OpenTelemetry collector is designed to accept metrics data in a number of
formats, transport data using the OpenTelemetry data model, and then export into
existing systems. The data model can be unambiguously translated into the
Prometheus Remote Write protocol without loss of features or semantics, through
well-defined translations of the data, including the ability to automatically
remove attributes and lower histogram resolution.
-->

OpenTelemetryコレクターは、様々なフォーマットのメトリックデータを受け入れ、OpenTelemetryデータモデルを使用してデータを転送し、既存のシステムにエクスポートするように設計されています。このデータモデルは、自動的に属性を削除したり、ヒストグラムの解像度を下げたりする機能を含む、データの明確な変換によって、機能やセマンティクスを失うことなく、Prometheus Remote Writeプロトコルに明確に変換することができます。

<!--
## Events → Data Stream → Timeseries
-->

## Events → Data Stream → Timeseries

<!--
The OTLP Metrics protocol is designed as a standard for transporting metric
data. To describe the intended use of this data and the associated semantic
meaning, OpenTelemetry metric data stream types will be linked into a framework
containing a higher-level model, about Metrics APIs and discrete input values,
and a lower-level model, defining the Timeseries and discrete output values.
The relationship between models is displayed in the diagram below.
-->

OTLP Metricsプロトコルは、メトリックデータを伝送するための標準として設計されています。このデータの使用目的と関連する意味を記述するために、OpenTelemetryのメトリックデータストリームは、Metrics APIおよび離散的な入力値に関する上位モデルと、Timeseriesおよび離散的な出力値を定義する下位モデルを含むフレームワークに結び付けられます。モデル間の関係は以下の図に示されています。

<!--
![Events → Data Stream → Timeseries Diagram](img/model-layers.png)
-->

![Events → Data Stream → Timeseries の図](img/model-layers.png)

<!--
This protocol was designed to meet the requirements of the OpenCensus Metrics
system, particularly to meet its concept of Metrics Views. Views are
accomplished in the OpenTelemetry Metrics data model through support for data
transformation on the collection path.
-->

このプロトコルは、OpenCensus Metricsシステムの要件、特にMetrics Viewのコンセプトを満たすように設計されています。ビューは、OpenTelemetry Metricsデータモデルにおいて、収集パス(XXX?)上のデータ変換をサポートすることで実現されます。

<!--
OpenTelemetry has identified three kinds of semantics-preserving Metric data
transformation that are useful in building metrics collection systems as ways of
controlling cost, reliability, and resource allocation. The OpenTelemetry
Metrics data model is designed to support these transformations both inside an
SDK as the data originates, or as a reprocessing stage inside the OpenTelemetry
collector. These transformations are:
-->

OpenTelemetryでは、コスト、信頼性、リソース配分を管理する方法としてメトリック収集システムを構築する際に有用な、3種類のセマンティクスを保持したメトリックデータ変換を定義しています。OpenTelemetry Metricsデータモデルは、データが発生したときにSDK内部で、あるいはOpenTelemetryコレクター内部の再処理段階で、これらの変換をサポートするように設計されています。これらの変換は以下の通りです。

<!--
1. Temporal reaggregation: Metrics that are collected at a high-frequency can be
   re-aggregated into longer intervals, allowing low-resolution timeseries to be
   pre-calculated or used in place of the original metric data.
2. Spatial reaggregation: Metrics that are produced with unwanted dimensions can
   be re-aggregated into metrics having fewer dimensions.
3. Delta-to-Cumulative: Metrics that are input and output with Delta temporality
   unburden the client from keeping high-cardinality state. The use of deltas
   allows downstream services to bear the cost of conversion into cumulative
   timeseries, or to forego the cost and calculate rates directly.
-->

1. 時間的な再集計: 高頻度で収集されたメトリックをより長い間隔で再集計することで、低解像度の時系列データを事前に計算したり、元のメトリックデータの代わりに使用したりすることができます。

2. 空間的な再集計: 不必要な次元で作成されたメトリックを、より少ない次元のメトリックに再集約することができます。

3. デルタから累積値へ: デルタの時間性で入出力されるメトリックは、クライアントが高いカーディナリティの状態を維持する負担を軽減します。デルタを使用することで、下流のサービスが累積時系列への変換コストを負担したり、コストを省いて直接レートを計算したりすることができます。

<!--
OpenTelemetry Metrics data streams are designed so that these transformations
can be applied automatically to streams of the same type, subject to conditions
outlined below. Every OTLP data stream has an intrinsic
[decomposable aggregate function](https://en.wikipedia.org/wiki/Aggregate_function#Decomposable_aggregate_functions)
making it semantically well-defined to merge data points across both temporal
and spatial dimensions. Every OTLP data point also has two meaningful timestamps
which, combined with intrinsic aggregation, make it possible to carry out the
standard metric data transformations for each of the model’s basic points while
ensuring that the result carries the intended meaning.
-->

OpenTelemetry Metricsのデータストリームは、以下に概説する条件のもと、これらの変換が同じタイプのストリームに自動的に適用できるように設計されています。すべてのOTLPデータストリームは、固有の[分解可能な集約関数](https://en.wikipedia.org/wiki/Aggregate_function#Decomposable_aggregate_functions)を持っており、時間的・空間的な次元を超えてデータポイントをマージすることが意味的に明確に定義されています。また、すべてのOTLPデータポイントは、2つの意味のあるタイムスタンプを持っています。このタイムスタンプと固有の集約機能を組み合わせることで、モデルの各基本ポイントに対して、意図した意味を持つ結果を確保しながら、標準的なメトリックデータ変換を行うことが可能になります。

<!--
As in OpenCensus Metrics, metrics data can be transformed into one or more
Views, just by selecting the aggregation interval and the desired dimensions.
One stream of OTLP data can be transformed into multiple timeseries outputs by
configuring different Views, and the required Views processing may be applied
inside the SDK or by an external collector.
-->

OpenCensus Metricsと同様に、メトリックデータは、集計間隔と必要な次元を選択するだけで、1つまたは複数のViewに変換することができます。1つのOTLPデータは、異なるViewを設定することで、複数のTimeseriesデータに変換することができ、必要なViewの処理はSDK内でも、外部のコレクターでも行うことができます。

<!--
### Example Use-cases
-->

### ユースケースの例

<!--
The metric data model is designed around a series of "core" use cases.  While
this list is not exhaustive, it is meant to be representative of the scope and
breadth of OTel metrics usage.
-->

Metricsデータモデルは、一連の「コア」ユースケースを中心に設計されています。 このリストはすべてを網羅しているわけではありませんが、OTel Metricsが対象とする領域を示すことを目的としています。

<!--
1. OTel SDK exports 10 second resolution to a single OTel collector, using
  cumulative temporality for a stateful client, stateless server:
    - Collector passes-through original data to an OTLP destination
    - Collector re-aggregates into longer intervals without changing dimensions
    - Collector re-aggregates into several distinct views, each with a subset of
      the available dimensions, outputs to the same destination
2. OTel SDK exports 10 second resolution to a single OTel collector, using delta
  temporality for a stateless client, stateful server:
    - Collector re-aggregates into 60 second resolution
    - Collector converts delta to cumulative temporality
3. OTel SDK exports both 10 seconds resolution (e.g. CPU, request latency) and
  15 minutes resolution (e.g. room temperature) to a single OTel Collector.
  The collector exports streams upstream with or without aggregation.
4. A number of OTel SDKs running locally each exports 10 second resolution, each
  reports to a single (local) OTel collector.
    - Collector re-aggregates into 60 second resolution
    - Collector re-aggregates to eliminate the identity of individual SDKs (e.g.,
      distinct `service.instance.id` values)
    - Collector outputs to an OTLP destination
5. Pool of OTel collectors receive OTLP and export Prometheus Remote Write
    - Collector joins service discovery with metric resources
    - Collector computes “up”, staleness marker
    - Collector applies a distinct external label
6. OTel collector receives Statsd and exports OTLP
    - With delta temporality: stateless collector
    - With cumulative temporality: stateful collector
7. OTel SDK exports directly to 3P backend
-->

1. OTel SDKが、ステートフルなクライアントとステートレスなサーバの累積的な時間性を利用して、10秒の解像度を1つのOTel Collectorにエクスポートする
    - Collectorは元のデータをOTLPの宛先にパススルーする
    - Collectorは次元を変えずに、より長い間隔に再集約する
    - Collectorは利用可能なディメンションのサブセットを持つ、複数の異なるビューに再集約し、同じ宛先に出力する
2. OTel SDKが、ステートレスなクライアントとステートフルなサーバのためにdeltae temporality(XXX:時間差分？)を使用して、10秒の解像度を単一のOTel Collectorにエクスポートする
    - Collectorは60秒の解像度に再集約する
    - Collectorはデルタを累積時間性に変換する
3. OTel SDKが、10秒単位の解像度(CPU、リクエストのレイテンシなど)と15分単位の解像度(室温など)の両方を1つのOTel Collectorにエクスポートします。Collectorは、集約の有無にかかわらず、ストリームをアップストリームにエクスポートする
4. ローカルで動作している複数のOTel SDKがそれぞれ10秒の解像度をエクスポートし、それぞれが1つの(ローカルの)OTel Collectorにレポートする
    - Collectorは60秒の解像度に再集約する
    - Collectorは個々のSDKのアイデンティティを排除するために再集約します(例:個別の`service.instance.id`値)
    - CollectorはOTLPの宛先に出力する
5. OTel CollectorのプールがOTLPを受信し、Prometheus Remote Writeをエクスポートする
    - Collectorがメトリックリソースを使ったサービスディスカバリーに参加する
    - Collectorが "up"を計算し、staleness marker(訳注: Prometheus用語)とする
    - Collectorが明確な外部ラベルを適用する
6. OTel CollectorがStatsdを受信し、OTLPをエクスポートします
    - 時間差分(XXX: Delta Temporarlity)の場合:ステートレスCollector
    - 累積時間性の場合:ステートフルCollector
7. OTel SDKが3Pバックエンドに直接エクスポートする


<!--
These are considered the "core" use-cases used to analyze tradeoffs and design
decisions within the metrics data model.
-->

これらは、Metricsデータモデル内のトレードオフや設計上の決定を分析するために使用される「コア」ユースケースと考えられます。

<!--
### Out of Scope Use-cases
-->

### 対象外のユースケース

<!--
The metrics data model is NOT designed to be a perfect rosetta stone of metrics.
Here are a set of use cases that, while won't be outright unsupported, are not
in scope for key design decisions:
-->

Metricsデータモデルは、メトリックの完璧なロゼッタ・ストーンになるようには設計されていません。ここでは、完全にサポートされていないわけではありませんが、重要な設計上の決定事項には含まれていないユースケースのセットを紹介します。

<!--
- Using OTLP as an intermediary format between two non-compatible formats
  - Importing [statsd](https://github.com/statsd/statsd) => Prometheus PRW
  - Importing [collectd](https://collectd.org/wiki/index.php/Binary_protocol#:~:text=The%20binary%20protocol%20is%20the,some%20documentation%20to%20reimplement%20it)
    => Prometheus PRW
  - Importing Prometheus endpoint scrape => [statsd push | collectd | opencensus]
  - Importing OpenCensus "oca" => any non OC or OTel format
- TODO: define others.
-->

- 互換性のない2つのフォーマットの間の中間フォーマットとしてOTLPを使用する
  - インポート [statsd](https://github.com/statsd/statsd) => Prometheus PRW
  - インポート [collectd](https://collectd.org/wiki/index.php/Binary_protocol#:~:text=The%20binary%20protocol%20is%20the,some%20documentation%20to%20reimplement%20it)
    => Prometheus PRW
  - Prometheus エンドポイントのスクレイプをインポートする => [statsd push | collectd | opencensus] 
  - OpenCensus のインポート "oca" => OC または OTel 以外の任意のフォーマット
- TODO: その他の定義。


<!--
## Model Details
-->

## Model の詳細

<!--
OpenTelemetry fragments metrics into three interacting models:
-->

OpenTelemetryは、Metricsを3つの相互作用するモデルに分けています。

<!--
- An Event model, representing how instrumentation reports metric data.
- A Timeseries model, representing how backends store metric data.
- A Metric Stream model, defining the *O*pen*T*e*L*emetry *P*rotocol (OTLP)
  representing how metric data streams are manipulated and transmitted between
  the Event model and the Timeseries storage.
-->

- Eventモデル: 計装によるメトリックデータの報告方法を表します
- TimeSeriesモデル: バックエンドがどのようにメトリックデータを保存するかを表します
- メトリックz/ストリームモデル: イベントモデルとTimeseriesストレージの間でメトリックデータストリームがどのように操作され、転送されるかを表すOTLP(*O*pen*T*e*L*emetry *P*rotocol)を定義します。

<!--
### Event Model
-->

### Event モデル

<!--
The event model is where recording of data happens. Its foundation is made of
[Instruments](api.md), which are used to record data observations via events.
These raw events are then transformed in some fashion before being sent to some
other system.  OpenTelemetry metrics are designed such that the same instrument
and events can be used in different ways to generate metric streams.
-->

イベントモデルは、データの記録を行う場所です。イベントモデルの基盤は、イベントを介してデータ観測を記録するために使用される[Instruments](api.md)でできています。これらの生のイベントは、他のシステムに送られる前に何らかの方法で変換されます。OpenTelemetryのMetricsは、同じInstrumentとイベントを異なる方法で使用してメトリックストリームを生成できるように設計されています。

<!--
![Events → Streams](img/model-event-layer.png)
-->

![Events → Streams](img/model-event-layer.png)

<!--
Even though observation events could be reported directly to a backend, in
practice this would be infeasible due to the sheer volume of data used in
observability systems, and the limited amount of network/cpu telemetry
collection resources available for telemetry collection purposes. The best
example of this is the Histogram metric where raw events are recorded in a
compressed format rather than individual timeseries.
-->

観測イベントをバックエンドに直接報告することができたとしても、実際には、観測システムで使用されるデータ量が膨大であることや、テレメトリ収集の目的で利用できるネットワーク/CPUのテレメトリ収集リソースが限られていることから、実現できないでしょう。その最たる例が、個々のTimeseriesではなく生のイベントが圧縮された形で記録されるヒストグラム・メトリックです。

<!--
> Note: The above picture shows how one instrument can transform events into
> more than one type of metric stream. There are caveats and nuances for when
> and how to do this.  Instrument and metric configuration are outlined
> in the [metrics API specification](api.md).
-->

> 注:上記の図は、1つの機器でイベントを複数の種類のメトリック・ストリームに変換する方法を示しています。これをいつ、どのように行うかについては、注意点やニュアンスがあります。InstrumentsとMetricの設定については、[Metric API 仕様](api.md)に概要が記載されています。

<!--
While OpenTelemetry provides flexibility in how instruments can be transformed
into metric streams, the instruments are defined such that a reasonable default
mapping can be provided. The exact
[OpenTelemetry instruments](api.md##metric-instruments) are more fully
detailed in the API specification.
-->

OpenTelemetryでは、Instrumentsをメトリック・ストリームに変換する方法に柔軟性を持たせています。Instrumentsは、合理的なデフォルトのマッピングが提供できるように定義されています。正確な[OpenTelemetry instruments](api.md##metric-instruments)の詳細は、API仕様に記載されています。

<!--
In the Event model, the primary data are (instrument, number) points, originally
observed in real time or on demand (for the synchronous and asynchronous cases,
respectively).
-->

イベントモデルでは、主なデータは(Instruments, number)点であり、元々はリアルタイムまたはオンデマンドで観測されたものです(同期および非同期の場合のそれぞれに)。

<!--
### Timeseries Model
-->

### Timeseries モデル

<!--
In this low-level metrics data model, a Timeseries is defined by an entity
consisting of several metadata properties:
-->

この低レベルのメトリックデータモデルでは、Timeseriesはいくつかのメタデータプロパティで構成されるエンティティで定義されます。

<!--
- Metric name
- Label set
- Kind of point (integer, floating point, etc)
- Unit of measurement
-->

- メトリック名
- ラベルセット
- 点の種類(整数、浮動小数点、その他)
- 測定単位

<!--
The primary data of each timeseries are ordered (timestamp, value) points, for
three value types:
-->

各Timeseriesの主なデータは、3つの値のタイプに対応した順序(タイムスタンプ、値)のポイントです。

<!--
1. Counter (Monotonic, cumulative)
2. Gauge
3. Histogram
-->

1. カウンター(モノトニック、累積)
2. ゲージ
3. ヒストグラム

<!--
This model may be viewed as an idealization of
[Prometheus Remote Write](https://docs.google.com/document/d/1LPhVRSFkGNSuU1fBd81ulhsCPR4hkSZyyBj1SZ8fWOM/edit#heading=h.3p42p5s8n0ui).
Like that protocol, we are additionally concerned with knowing when a point
value is defined, as compared with being implicitly or explicitly absent. A
metric stream of delta data points defines time-interval values, not
point-in-time values.  To precisely define presence and absence of data requires
further development of the correspondence between these models.
-->

このモデルは、[Prometheus Remote Write](https://docs.google.com/document/d/1LPhVRSFkGNSuU1fBd81ulhsCPR4hkSZyyBj1SZ8fWOM/edit#heading=h.3p42p5s8n0ui)の理想化と見なすことができます。このプロトコルのように、我々は、暗黙的または明示的に存在しないことと比較して、ポイント値がいつ定義されるかを知ることにも関心があります。delta data points(XXX: 差分?)のメトリック・ストリームは、ポイントインタイム値ではなく、タイムインターバル値を定義します。 データの存在と不在を正確に定義するには、これらのモデルの対応関係をさらに発展させる必要があります。

<!--
### OpenTelemetry Protocol data model
-->

### OpenTelemetry Protocol データモデル

<!--
The OpenTelmetry protocol data model is composed of Metric data streams. These
streams are in turn composed of metric data points. Metric data streams
can be converted directly into Timeseries, and share the same identity
characteristics for a Timeseries. A metric stream is identified by:
-->

OpenTelmetryプロトコルのデータモデルは、メトリック・データ・ストリームで構成されています。これらのストリームは、順にメトリックデータポイントで構成されます。メトリック・データ・ストリームは、Timeseriesに直接変換することができ、Timeseriesと同じ識別特性を持っています。メトリック・ストリームは以下の方法で識別されます。

<!--
- The originating `Resource`
- The metric stream's `name`.
- The attached `Attribute`s
- The metric stream's point kind.
-->

- 発信元の `Resource`
- メトリック・ストリームの `名前`
- 添付された `Attribute`
- メトリック・ストリームの`ポイントの種類`

<!--
It is possible (and likely) that more than one metric stream is created per
`Instrument` in the event model.
-->

イベントモデルでは、`Instrument`ごとに複数のメトリック・ストリームが作成される可能性があります。

<!--
__Note: The same `Resource`, `name` and `Attribute`s but differing point kind
coming out of an OpenTelemetry SDK is considered an "error state" that should
be handled by an SDK.__
-->

__注:OpenTelemetry SDKから出力される、同じ`Resource`、`name`、`Attribute`でもポイントの種類が異なるものは、SDKが処理すべき「エラー状態」とみなされます。__

<!--
A metric stream can use one of three basic point kinds, all of
which satisfy the requirements above, meaning they define a decomposable
aggregate function (also known as a “natural merge” function) for points of the
same kind. <sup>[1](#otlpdatapointfn)</sup>
-->

メトリック・ストリームでは、3つの基本的なポイントの種類のうちの1つを使用することができます。これらのポイントはすべて上記の要件を満たしており、同じ種類のポイントに対する分解可能な集約関数(「ナチュラルマージ」関数としても知られています)を定義していることになります。<sup>[1](#otlpdatapointfn)</sup> <sup>[1](#otlpdatapointfn)

<!--
The basic point kinds are:
-->

基本的なポイントの種類は以下のとおりです。

<!--
1. [Sum](https://github.com/open-telemetry/opentelemetry-proto/blob/main/opentelemetry/proto/metrics/v1/metrics.proto#L200)
2. [Gauge](https://github.com/open-telemetry/opentelemetry-proto/blob/main/opentelemetry/proto/metrics/v1/metrics.proto#L170)
3. [Histogram](https://github.com/open-telemetry/opentelemetry-proto/blob/main/opentelemetry/proto/metrics/v1/metrics.proto#L228)
-->

1. [Sum](https://github.com/open-telemetry/opentelemetry-proto/blob/main/opentelemetry/proto/metrics/v1/metrics.proto#L200)
2. [Gauge](https://github.com/open-telemetry/opentelemetry-proto/blob/main/opentelemetry/proto/metrics/v1/metrics.proto#L170)
3. [Histogram](https://github.com/open-telemetry/opentelemetry-proto/blob/main/opentelemetry/proto/metrics/v1/metrics.proto#L228)

<!--
Comparing the OTLP Metric Data Stream and Timeseries data models, OTLP does
not map 1:1 from its point types into timeseries points. In OTLP, a Sum point
can represent a monotonic count or a non-monotonic count. This means an OTLP Sum
is either translated into a Timeseries Counter, when the sum is monotonic, or
a Gauge when the sum is not monotonic.
-->

OTLのメトリック・データ・ストリームとTimeseriesデータモデルを比較すると、OTLP はポイントタイプをTimeseriesポイントに1:1でマッピングしていません。OTLPのSumは、単調なカウントと非単調なカウントを表すことができます。つまり、OTLPのSumは、和が単調な場合はTimeseriesカウンターに変換され、和が単調でない場合はゲージに変換されます。

<!--
![Stream → Timeseries](img/model-layers-stream.png)
-->

![Stream → Timeseries](img/model-layers-stream.png)

<!--
Specifically, in OpenTelemetry Sums always have an aggregate function where
you can combine via addition. So, for non-monotonic sums in OpenTelemetry we
can aggregate (naturally) via addition.  In the timeseries model, you cannot
assume that any particular Gauge is a sum, so the default aggregation would not
be addition.
-->

具体的には、OpenTelemetryのSumは常に、加算によって結合できる集約関数を持っています。つまり、OpenTelemetryでは、非単調な和については、(当然ながら)加算によって集約することができます。 Timeseriesモデルでは、特定のゲージが和であると仮定することはできないので、デフォルトの集約は加算ではありません。

<!--
In addition to the core point kinds used in OTLP, there are also data types
designed for compatibility with existing metric formats.
-->

OTLPで使用されているコアポイントの種類に加えて、既存のメトリックフォーマットとの互換性を考慮したデータタイプもあります。

<!--
- [Summary](#summary-legacy)
-->

- [まとめ](#summary-legacy)

<!--
## Metric points
-->

## Metric points

<!--
### Sums
-->

### Sums

<!--
[Sum](https://github.com/open-telemetry/opentelemetry-proto/blob/main/opentelemetry/proto/metrics/v1/metrics.proto#L202)s
in OTLP consist of the following:
-->

OTLPの[Sum](https://github.com/open-telemetry/opentelemetry-proto/blob/main/opentelemetry/proto/metrics/v1/metrics.proto#L202)の構成は以下の通りです。

<!--
- An *Aggregation Temporality* of delta or cumulative.
- A flag denoting whether the Sum is
  [monotonic](https://en.wikipedia.org/wiki/Monotonic_function). In this case of
  metrics, this means the sum is nominally increasing, which we assume without
  loss of generality.
  - For delta monotonic sums, this means the reader should expect non-negative
    values.
  - For cumulative monotonic sums, this means the reader should expect values
    that are not less than the previous value.
- A set of data points, each containing:
  - An independent set of Attribute name-value pairs.
  - A time window (of `(start, end]`) time for which the Sum was calculated.
    - The time interval is inclusive of the end time.
    - Times are specified in Value is UNIX Epoch time in nanoseconds since
      `00:00:00 UTC on 1 January 1970`
-->

- delta または cumulative の *Aggregation Temporality*
- Sumが[monotonic](https://en.wikipedia.org/wiki/Monotonic_function)であるかどうかを示すフラグ。このメトリックの場合、これは和が名目上増加していることを意味し、一般性を損なわない範囲でこれを仮定します。
   - デルタ単調和の場合、読み込み側は非負の値を期待することになります。
   - 累積単調和の場合は、前の値を下回らない値を期待することになります。
- データポイントのセット。それぞれが以下の情報を含みます。
  - 独立したAttributeの名前と値のペアのセット
  - Sumが計算されたタイムウィンドウ(`(start, end]`)
    - 時間間隔は終了時間を含みます
    - 時間の指定は、「1970年1月1日00:00:00 UTC」からのナノ秒単位のUNIXエポックタイムです

<!--
The aggregation temporality is used to understand the context in which the sum
was calculated. When the aggregation temporality is "delta", we expect to have
no overlap in time windows for metric streams, e.g.
-->

集約の時間性は、Sumが計算されたコンテキストを理解するために使用されます。アグリゲーション・テンポラリティ(XXX:???)が "delta"の場合、例えばメトリック・ストリームのタイム・ウィンドウが重複しないことを期待しています。

<!--
![Delta Sum](img/model-delta-sum.png)
-->

![Delta Sum](img/model-delta-sum.png)

<!--
Contrast with cumulative aggregation temporality where we expect to report the
full sum since 'start' (where usually start means a process/application start):
-->

これは、「開始」(通常、開始とはプロセス/アプリケーションの開始を意味する)以降の全合計を報告することを期待する累積的な集計の時間性とは対照的です。

<!--
![Cumulative Sum](img/model-cumulative-sum.png)
-->

![Cumulative Sum](img/model-cumulative-sum.png)

<!--
There are various tradeoffs between using Delta vs. Cumulative aggregation, in
various use cases, e.g.:
-->

デルタ型と累積型の集計方法には、様々なユースケースにおいて、様々なトレードオフがあります。例えば以下のようなものです:

<!--
- Detecting process restarts
- Calculating rates
- Push vs. Pull based metric reporting
-->

- プロセス再起動の検出
- レートの計算
- プッシュ型とプル型の指標レポート

<!--
OTLP supports both models, and allows APIs, SDKs and users to determine the
best tradeoff for their use case.
-->

OTLPは両方のモデルをサポートしており、API、SDK、ユーザーはそれぞれのユースケースに最適なトレードオフを決定することができます。

<!--
### Gauge
-->

### ゲージ

<!--
Pending
-->

Pending

<!--
### Histogram
-->

### Histogram

<!--
Pending
-->

Pending

<!--
### Summary (Legacy)
-->

### まとめ (Legacy)

<!--
[Summary](https://github.com/open-telemetry/opentelemetry-proto/blob/main/opentelemetry/proto/metrics/v1/metrics.proto#L244)
metric data points convey quantile summaries, e.g. What is the 99-th percentile
latency of my HTTP server.  Unlike other point types in OpenTelemetry, Summary
points cannot always be merged in a meaningful way. This point type is not
recommended for new applications and exists for compatibility with other
formats.
-->

[Summary](https://github.com/open-telemetry/opentelemetry-proto/blob/main/opentelemetry/proto/metrics/v1/metrics.proto#L244) metricのデータポイントは、分位値のサマリーを伝えます。例えば、「私のHTTPサーバーの99パーセンタイルのレイテンシーは何か」などです。 OpenTelemetryの他のポイントタイプとは異なり、Summaryポイントは常に意味のある方法でマージすることができません。このポイントタイプは新しいアプリケーションには推奨されておらず、他のフォーマットとの互換性のために存在しています。

<!--
## Single-Writer
-->

## Single-Writer

<!--
All metric data streams within OTLP must have one logical writer.  This means,
conceptually, that any Timeseries created from the Protocol must have one
originating source of truth.  In practical terms, this implies the following:
-->

OTLP内のすべてのメトリック・データ・ストリームは、1つの論理的書き込みを持たなければなりません。 これは概念的には、プロトコルから作成されたTimeseriesは、信頼できる唯一の情報源(source of truth)を持たなければならないことを意味します。 現実的には、これは以下のことを意味します。

<!--
- All metric data streams produced by OTel SDKs must be globally uniquely
  produced and free from duplicates.   All metric data streams can be uniquely
  identified in some way.
- Aggregations of metric streams must only be written from a single logical
  source.
  __Note: This implies aggregated metric streams must reach one destination__.
-->

- OTel SDKが生成するすべてのメトリック・データ・ストリームは、グローバルに一意に生成され、重複がないものでなければなりません。すべてのメトリック・データ・ストリームは、何らかの方法で一意に識別できます。
- メトリック・ストリームの集約は、単一の論理的ソースからのみ書き込まれなければなりません。注:これは、集約されたメトリック・ストリームが1つの宛先に到達しなければならないことを意味します__。

<!--
In systems, there is the possibility of multiple writers sending data for the
same metric stream (duplication).  For example, if an SDK implementation fails
to find uniquely identifying Resource attributes for a component, then all
instances of that component could be reporting metrics as if they are from the
same resource.  In this case, metrics will be reported at inconsistent time
intervals.  For metrics like cumulative sums, this could cause issues where
pairs of points appear to reset the cumulative sum leading to unusable metrics.
-->

システムでは、複数のライターが同じメトリック・ストリームに対してデータを送信する可能性があります(duplication)。 例えば、SDKの実装で、コンポーネントを一意に識別するResource属性を見つけられなかった場合、そのコンポーネントのすべてのインスタンスが、あたかも同じリソースからのものであるかのようにメトリックを報告してしまう可能性があります。 この場合、メトリックは矛盾した時間間隔で報告されます。 累積和のようなメトリックでは、ポイントのペアが累積和をリセットしているように見える問題が発生し、使用できないメトリックになる可能性があります。

<!--
Multiple writers for a metric stream is considered an error state, or
misbehaving system. Receivers SHOULD presume a single writer was intended and
eliminate overlap / deduplicate.
-->

1つのメトリック・ストリームに対して複数の書き込み側が存在する場合は、エラー状態または誤動作しているシステムとみなされます。受信側は、単一の書き込みが意図されていたと推定し、重複を排除するべきです(SHOULD)。

<!--
Note: Identity is an important concept in most metrics systems.  For example,
[Prometheus directly calls out uniqueness](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#metric_relabel_configs):
-->

注:同一性は、ほとんどのメトリックシステムにおいて重要な概念である。 例えば、[Prometheusは一意性を直接呼びます](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#metric_relabel_configs)。

<!--
> Care must be taken with `labeldrop` and `labelkeep` to ensure that metrics
> are still uniquely labeled once the labels are removed.
-->

> `labeldrop`と`labelkeep`では、ラベルが取り除かれた後も、Metricsが一意にラベル付けされていることを保証するために注意しなければなりません。

<!--
For OTLP, the Single-Writer principle grants a way to reason over error
scenarios and take corrective actions.  Additionally, it ensures that
well-behaved systems can perform metric stream manipulation without undesired
degradation or loss of visibility.
-->

OTLPでは、単一書き込み者の原則により、エラーシナリオを推論し、修正措置を取ることができます。 さらに、お行儀の良いシステムが、望ましくない劣化や視認性の低下を招くことなく、メトリック…ストリームの操作を行えることを保証します。

<!--
## Temporality
-->

## Temporality

<!--
Every OTLP point has two associated timestamps. For OTLP Sum and Histogram
points, the two timestamps indicate when the point was reset and when the sum
was captured. For OTLP Gauge points, the two timestamps indicate when the
measurement was taken and when it was reported as being still the last value.
-->

すべてのOTLPポイントには2つの関連するタイムスタンプを持ちます。OTLP SumとHistogramポイントの場合、2つのタイムスタンプはポイントがリセットされた時とSumがキャプチャされた時を示しています。OTLP Gaugeポイントの場合、2つのタイムスタンプは、測定が行われた時と、それがまだ最後の値であると報告された時を示しています。

<!--
The notion of temporality refers to a configuration choice made in the system
as a whole, indicating whether reported values incorporate previous
measurements, or not.
-->

時間性の概念は、システム全体で行われる設定の選択を意味し、報告された値に以前の測定値を組み込むかどうかを示します。

<!--
- *Cumulative temporality* means that successive data points repeat the starting
  timestamp. For example, from start time T0, cumulative data points cover time
  ranges (T<sub>0</sub>, T<sub>1</sub>), (T<sub>0</sub>, T<sub>2</sub>),
  (T<sub>0</sub>, T<sub>3</sub>), and so on.
- *Delta temporality* means that successive data points advance the starting
  timestamp. For example, from start time T0, delta data points cover time
  ranges (T<sub>0</sub>, T<sub>1</sub>), (T<sub>1</sub>, T<sub>2</sub>),
  (T<sub>2</sub>, T<sub>3</sub>), and so on.
-->

- *累積時間性(Cumulative temporality)*とは、連続したデータポイントが開始時のタイムスタンプを繰り返すことを意味します。例えば、開始時刻T0から、累積データポイントは、(T<sub>0</sub>, T<sub>1</sub>)、(T<sub>0</sub>, T<sub>2</sub>)、(T<sub>0</sub>, T<sub>3</sub>)などの時間範囲をカバーします。

- *デルタ時間性(Delta temporality)*とは、連続したデータポイントが開始タイムスタンプを進めることを意味します。例えば、開始時刻T0から、デルタデータポイントは、(T<sub>0</sub>, T<sub>1</sub>)、(T<sub>1</sub>, T<sub>2</sub>)、(T<sub>2</sub>, T<sub>3</sub>)などの時間範囲をカバーします。

<!--
The use of cumulative temporality for monotonic sums is common, exemplified by
Prometheus. Systems based in cumulative monotonic sums are naturally simpler, in
terms of the cost of adding reliability. When collection fails intermittently,
gaps in the data are naturally averaged from cumulative measurements.
Cumulative data requires the sender to remember all previous measurements, an
“up-front” memory cost proportional to cardinality.
-->

Prometheusに代表されるように、モノトニック・サムに累積時間性を用いることは一般的です。累積モノトニック・サムに基づくシステムは、信頼性を高めるためのコストという点で、当然ながらよりシンプルです。収集が断続的に失敗する場合、データのギャップは累積測定から自然に平均化されます。累積データは、送信者が以前のすべての測定値を記憶しておく必要があります。これは、カーディナリティに比例した「アップフロント」メモリコストです。(XXX:事前に推測できる、という意味かな)

<!--
The use of delta temporality for metric sums is also common, exemplified by
Statsd. There is a connection between OpenTelemetry tracing, in which a Span
event commonly is translated into two metric events (a 1-count and a timing
measurement). Delta temporality enables sampling and supports shifting the cost
of cardinality outside of the process.
-->

メトリックの合計に対するデルタ時間性の使用も一般的であり、Statsdがその例です。OpenTelemetryのTraceには、Spanイベントが一般的に2つのメトリックイベント(1カウントとタイミング測定)に変換されるという関連性があります。デルタ時間性はサンプリングを可能にし、カーディナリティのコストをプロセスの外に移すことをサポートします。

<!--
## Overlap
-->

## Overlap

<!--
Overlap occurs when more than one metric data point occurs for a data stream
within a time window.   This is particularly problematic for data points meant
to represent an entire time window, e.g. a Histogram reporting population
density of collected metric data points for a time window.  If two of these show
up with overlapping time windows, how do backends handle this situation?
-->

オーバーラップは、1つのデータストリームにおいて、1つのタイムウィンドウ内で複数のメトリック・データポイントが発生した場合に発生します。  これは、時間ウィンドウ全体を表すデータポイントの場合に特に問題となります。たとえば、時間ウィンドウ内で収集したメトリック・データポイントの密度を報告するHistogramなどです。 このようなデータポイントが2つ発生し、時間ウィンドウが重なっている場合、バックエンドはこの状況をどのように処理するのでしょうか。

<!--
We define three principles for handling overlap:
-->

オーバーラップを処理するための3つの原則を定義しています。

<!--
- Resolution (correction via dropping points)
- Observability (allowing the data to flow to backends)
- Interpolation (correction via data manipulation)
-->

- 解決(Resolution): (ドロップポイントによる補正)
- 観測(Observability): (バックエンドへのデータの流し込み)
- 補間(Interpolation): (データ操作による補正)

<!--
### Overlap resolution
-->

### Overlapの解決

<!--
When more than one process writes the same metric data stream, OTLP data points
may appear to overlap. This condition typically results from misconfiguration, but
can also result from running identical processes (indicative of operating system
or SDK bugs, like missing
[process attributes](../resource/semantic_conventions/process.md)). When there
are overlapping points, receivers SHOULD eliminate points so that there are no
overlaps. Which data to select in overlapping cases is not specified.
-->

複数のプロセスが同じメトリックデータストリームを書き込むと、OTLPのデータポイントが重なって見えることがあります。この状態は一般的に設定ミスに起因しますが、同一のプロセスを実行した場合にも発生する可能性があります([process attributes]の欠落(../resource/semantic_conventions/process.md)など、OSやSDKのバグを示しています)。重複するポイントがある場合、受信側は重複がないようにポイントを排除すべきです(SHOULD)。重複する場合にどのデータを選択するかは規定しません。

<!--
### Overlap observability
-->

### Overlapの観測

<!--
OpenTelemetry collectors SHOULD export telemetry when they observe overlapping
points in data streams, so that the user can monitor for erroneous
configurations.
-->

OpenTelemetry collctorは、データストリームに重複するポイントを観測したときにテレメトリをエクスポートするべきです(SHOULD)。これによりユーザーは誤った構成を監視することができます。

<!--
### Overlap interpolation
-->

### Overlapの補間

<!--
When one process starts just as another exits, the appearance of overlapping
points may be expected. In this case, OpenTelemetry collectors SHOULD modify
points at the change-over using interpolation for Sum data points, to reduce
gaps to zero width in these cases, without any overlap.
-->

1 つのプロセスが開始され、別のプロセスが終了するとき、重複するポイントの出現が予想されます。この場合、OpenTelemetry collectorは、Sumデータポイントの補間を使用して切り替え時にポイントを修正し、ギャップをゼロ幅にして、オーバーラップが発生しないようにすべきです(SHOULD)。

<!--
## Resources
-->

## Resources

<!--
Pending
-->

Pending

<!--
## Temporal Alignment
-->

## Temporal Alignment

<!--
Pending
-->

Pending

<!--
## External Labels
-->

## External Labels

<!--
Pending
-->

Pending

<!--
## Stream Manipulations
-->

## Stream操作

<!--
Pending introduction.
-->

Pending introduction.

<!--
### Sums: Delta-to-Cumulative
-->

### Sums: デルタから累積への変換

<!--
While OpenTelemetry (and some metric backends) allows both Delta and Cumulative
sums to be reported, the timeseries model we target does not support delta
counters.  To this end, converting from delta to cumulative needs to be defined
so that backends can use this mechanism.
-->

OpenTelemetry(およびいくつかのメトリックバックエンド)では、デルタ値と累積値の両方を報告することができますが、私たちが対象としているTimesriesモデルはデルタカウンターをサポートしていません。 このため、バックエンドがこのメカニズムを使用できるように、デルタから累積への変換を定義する必要があります。

<!--
> Note: This is not the only possible Delta to Cumulative algorithm.  It is
> just one possible implementation that fits the OTel Data Model.
-->

> 注:これは、唯一可能なデルタから累積に変換するアルゴリズムではありません。 これは、OTelデータモデルに適合する可能な実装の一つに過ぎません。

<!--
Converting from delta points to cumulative point is inherently a stateful
operation.  To successfully translate, we need all incoming delta points to
reach one destination which can keep the current counter state and generate
a new cumulative stream of data (see [single writer princple](#single-writer)).
-->

デルタポイントから累積ポイントへの変換は、本質的にステートフルな操作です。 変換を成功させるためには、入力されるすべてのデルタポイントが、現在のカウンタの状態を維持し、新しい累積データのストリームを生成できる1つの目的地に到達する必要があります([単一書き込み者の例](#single-writer)を参照)。

<!--
The algorithm is scheduled out as follows:
-->

このアルゴリズムは、以下のようにスケジュールされています:

<!--
- Upon receiving the first Delta point for a given counter we set up the
  following:
  - A new counter which stores the cumulative sum, set to the initial counter.
  - A start time that aligns with the start time of the first point.
  - A "last seen" time that aligns with the time of the first point.
- Upon receiving future Delta points, we do the following:
  - If the next point aligns with the expected next-time window
    (see [detecting delta restarts](#sums-detecting-alignment-issues))
    - Update the "last seen" time to align with the time of the current point.
    - Add the current value to the cumulative counter
    - Output a new cumulative point with the original start time and current
      last seen time and count.
  - if the current point precedes the start time, then drop this point.
    Note: there are algorithms which can deal with late arriving points.
  - if the next point does NOT align with the expected next-time window, then
    reset the counter following the same steps performed as if the current point
    was the first point seen.
-->

- あるカウンターの最初のデルタポイントを受け取ると、次のように設定します。
  - 最初のカウンターに設定された、累積の合計を保存する新しいカウンター
  - 最初のポイントの開始時刻に合わせた開始時刻
  - 最初のポイントの時間と一致する「最後に見た(last seen)」時間
- 将来のデルタポイントを受け取る際には、以下の動作を行います。
  - 次のポイントが、予想される次の時間のウィンドウと一致した場合([deltaの再起動の検出](#sums-アライメント問題の検出)を参照)。
    - 「最後に見た時間」を現在のポイントの時間に合わせて更新する
    - 累積カウンターに現在の値を追加する 
    - 元の開始時刻と現在の最後に見た時刻とカウントを持つ新しい累積ポイントを出力する
  - 現在のポイントが開始時刻より前にある場合は、このポイントを削除します。注意:遅れて到着したポイントに対応できるアルゴリズムがあります。
  - 次のポイントが予想される次の時間のウィンドウと一致していない場合、現在のポイントが最初に見られたポイントである場合に実行される同じステップに従って、カウンタをリセットする。

<!--
#### Sums: detecting alignment issues
-->

#### Sums: アライメント問題の検出

<!--
When the next delta sum reported for a given metric stream does not align with
where we expect it, one of several things could have occurred:
-->

ある指標のストリームで報告された次のデルタサムが、期待する値と一致しない場合、いくつかの原因が考えられます。

<!--
- the process reporting metrics was rebooted, leading to a new reporting
  interval for the metric.
- A Single-Writer principle violation where multiple processes are reporting the
  same metric stream.
- There was a lost data point, or dropped information.
-->

- メトリックを報告するプロセスが再起動されたため、メトリックの報告間隔が新しくなった
- 複数のプロセスが同じメトリック・ストリームを報告している場合に、単一書き込み者の原則に違反している。
- データポイントが失われたり、情報が取りこぼされたりした。

<!--
In all of these scenarios we do our best to give any cumulative metric knowledge
that some data was lost, and reset the counter.
-->

いずれの場合も、データが失われたことを累積的に知ることができるように最善を尽くし、カウンターをリセットしています。

<!--
We detect alignment via two mechanisms:
-->

アライメントの検出には2つの方法があります。

<!--
- If the incoming delta time interval has significant overlap with the previous
  time interval, we must assume a violation of the single-writer principle.
- If the incoming delta time interval has a significant gap from the last seen
  time, we assume some kind of reboot/restart and reset the cumulative counter.
-->

- 入力されたデルタ時間間隔が前回の時間間隔と大きく重なっている場合、単一書き込み者の原則に違反していると仮定しなければならない。
- 入ってくるデルタ時間間隔が最後に見た時間から大きくずれている場合は、何らかの再起動/再スタートを想定し、累積カウンターをリセットする。

<!--
#### Sums: Missing Timestamps
-->

#### Sums: タイムスタンプの欠落

<!--
One degenerate case for the delta-to-cumulative algorithm is when timestamps
are missing from metric data points. While this shouldn't be the case when
using OpenTelemetry generated metrics, it can occur when adapting other metric
formats, e.g.
[StatsD counts](https://github.com/statsd/statsd/blob/master/docs/metric_types.md#counting).
-->

デルタから累積への変換アルゴリズムの退化したケースとして、メトリック・データポイントからタイムスタンプが失われている場合があります。OpenTelemetryが生成したメトリックを使用する際にはこのようなケースはないはずですが、[StatsD counts](https://github.com/statsd/statsd/blob/master/docs/metric_types.md#counting)などの他のメトリック形式を適応する際には発生する可能性があります。

<!--
In this scenario, the algorithm listed above would reset the cumulative sum on
every data point due to not being able to deterimine alignment or point overlap.
For comparison, see the simple logic used in
[statsd sums](https://github.com/statsd/statsd/blob/master/stats.js#L281)
where all points are added, and lost points are ignored.
-->

このシナリオでは、上記のアルゴリズムでは、アライメントやポイントのオーバーラップを判別できないため、すべてのデータポイントの累積和をリセットしてしまいます。比較のために、[statsd sums](https://github.com/statsd/statsd/blob/master/stats.js#L281)で使用されている、すべてのポイントが加算され、失われたポイントが無視されるシンプルなロジックを参照してください。


<!--
## Footnotes
-->

## 脚注

<!--
<a name="otlpdatapointfn">[1]</a>: OTLP supports data point kinds that do not
satisfy these conditions; they are well-defined but do not support standard
metric data transformations.
-->

<a name="otlpdatapointfn">[1]</a>: OTLPはこれらの条件を満たさないデータポイントの種類をサポートしています。それらは明示的に定義されていますが、標準的なメトリックデータ変換をサポートしていません。

