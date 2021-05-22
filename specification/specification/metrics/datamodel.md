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

既存の一般的なメトリクスデータ形式は、セマンティクスや正確さを損なうことなく、OpenTelemetryのMetricsデータモデルに明確に変換することができます。PrometheusやStatsdの出力フォーマットからの変換は、明示的に指定されています。

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

OpenTelemetryコレクターは、様々なフォーマットのメトリクスデータを受け入れ、OpenTelemetryデータモデルを使用してデータを転送し、既存のシステムにエクスポートするように設計されています。このデータモデルは、自動的に属性を削除したり、ヒストグラムの解像度を下げたりする機能を含む、データの明確な変換によって、機能やセマンティクスを失うことなく、Prometheus Remote Writeプロトコルに明確に変換することができます。

<!--
## Events → Data → Timeseries
-->

## Events → Data → Timeseries

<!--
The OTLP Metrics protocol is designed as a standard for transporting metric
data. To describe the intended use of this data and the associated semantic
meaning, OpenTelemetry metric data types will be linked into a framework
containing a higher-level model, about Metrics APIs and discrete input values,
and a lower-level model, defining the Timeseries and discrete output values.
The relationship between models is displayed in the diagram below.
-->

OTLP Metricsプロトコルは、メトリックデータを伝送するための標準として設計されています。このデータの使用目的と関連する意味を記述するために、OpenTelemetryのメトリックデータタイプは、Metrics APIおよび離散的な入力値に関する上位モデルと、Timeseriesおよび離散的な出力値を定義する下位モデルを含むフレームワークに結び付けられます。モデル間の関係は以下の図に示されています。

<!--
![Events  → Data → Timeseries Diagram](img/model-layers.png)
-->

![Events  → Data → Timeseries の図](img/model-layers.png)

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

OpenTelemetryでは、コスト、信頼性、リソース配分を管理する方法としてメトリクス収集システムを構築する際に有用な、3種類のセマンティクスを保持したメトリクスデータ変換を定義しています。OpenTelemetry Metricsデータモデルは、データが発生したときにSDK内部で、あるいはOpenTelemetryコレクター内部の再処理段階で、これらの変換をサポートするように設計されています。これらの変換は以下の通りです。

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

1. 時間的な再集計: 高頻度で収集されたメトリクスをより長い間隔で再集計することで、低解像度の時系列データを事前に計算したり、元のメトリクスデータの代わりに使用したりすることができます。

2. 空間的な再集計: 不必要な次元で作成されたメトリクスを、より少ない次元のメトリクスに再集約することができます。

3. デルタから累積値へ: デルタの時間性で入出力されるメトリクスは、クライアントが高いカーディナリティの状態を維持する負担を軽減します。デルタを使用することで、下流のサービスが累積時系列への変換コストを負担したり、コストを省いて直接レートを計算したりすることができます。

<!--
OpenTelemetry Metrics data points are designed so that these transformations can
be applied automatically to points of the same type, subject to conditions
outlined below. Every OTLP data point has an intrinsic
[decomposable aggregate function](https://en.wikipedia.org/wiki/Aggregate_function#Decomposable_aggregate_functions)
making it semantically well-defined to merge data points across both temporal
and spatial dimensions. Every OTLP data point also has two meaningful timestamps
which, combined with intrinsic aggregation, make it possible to carry out the
standard metric data transformations for each of the model’s basic points while
ensuring that the result carries the intended meaning.
-->

OpenTelemetry Metricsのデータポイントは、以下に概説する条件のもと、これらの変換が同じタイプのポイントに自動的に適用できるように設計されています。すべてのOTLPデータポイントは、固有の[分解可能な集約関数](https://en.wikipedia.org/wiki/Aggregate_function#Decomposable_aggregate_functions)を持っており、時間的・空間的な次元を超えてデータポイントをマージすることが意味的に明確に定義されています。また、すべてのOTLPデータポイントは、2つの意味のあるタイムスタンプを持っています。このタイムスタンプと固有の集約機能を組み合わせることで、モデルの各基本ポイントに対して、意図した意味を持つ結果を確保しながら、標準的なメトリックデータ変換を行うことが可能になります。

<!--
As in OpenCensus Metrics, metrics data can be transformed into one or more
Views, just by selecting the aggregation interval and the desired dimensions.
One stream of OTLP data can be transformed into multiple timeseries outputs by
configuring different Views, and the required Views processing may be applied
inside the SDK or by an external collector.
-->

OpenCensus Metricsと同様に、メトリクスデータは、集計間隔と必要な次元を選択するだけで、1つまたは複数のViewに変換することができます。1つのOTLPデータは、異なるViewを設定することで、複数の時系列データに変換することができ、必要なViewの処理はSDK内でも、外部のコレクターでも行うことができます。

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

Metricsデータモデルは、メトリクスの完璧なロゼッタ・ストーンになるようには設計されていません。ここでは、完全にサポートされていないわけではありませんが、重要な設計上の決定事項には含まれていないユースケースのセットを紹介します。

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
- A TimeSeries model, representing how backends store metric data.
- The *O*pen*T*e*L*emetry *P*rotocol (OTLP) data model representing how metrics
  are manipulated and transmitted between the Event model and the TimeSeries
  storage.
-->

- Eventモデル: 計装によるメトリックデータの報告方法を表します
- TimeSeriesモデル: バックエンドがどのようにメトリックデータを保存するかを表します
- O*pen*T*e*L*emetry *P*rotocol(OTLP)データモデル: メトリクスがどのように操作され、EventモデルとTimeSeriesストレージの間で送信されるかを表します。

<!--
### Event Model
-->

### Event モデル

<!--
This specification uses as its foundation a
[Metrics API consisting of 6 model instruments](api.md), each having distinct
semantics, that were prototyped in several OpenTelemetry SDKs between July 2019
and June 2020. The model instruments and their specific use-cases are meant to
anchor our understanding of the OpenTelemetry data model and are divided into
three categories:
-->

この仕様は、2019年7月から2020年6月の間にいくつかのOpenTelemetry SDKで試作された、それぞれが異なるセマンティクスを持つ[6つのモデル計装からなるMetrics API](api.md)を基礎として使用しています。モデル計装とその具体的なユースケースは、OpenTelemetryのデータモデルに対する理解を深めるためのもので、3つのカテゴリーに分かれています。

<!--
- Synchronous vs. Asynchronous. The act of calling a Metrics API in a
  synchronous context means the application/library calls the SDK, typically having
  associated trace context and baggage; an Asynchronous instrument is called at
  collection time, through a callback, and lacks context.
- Adding vs. Grouping. Whereas adding instruments express a sum, grouping
  instruments characterize a group of measurements. The numbers passed to adding
  instruments define division, in the algebraic sense, while the numbers passed
  to grouping instruments are generally not. Adding instrument values are always
  parts of a sum, while grouping instrument values are individual measurements.
- Monotonic vs. Non-Monotonic. The adding instruments are categorized by whether
  the derivative of the quantity they express is non-negative. Monotonic
  instruments are primarily useful for monitoring a rate value, whereas
  non-monotonic instruments are primarily useful for monitoring a total value.
-->

- 同期型と非同期型: 同期的なコンテキストでMetrics APIを呼び出す行為は、アプリケーション/ライブラリがSDKを呼び出すことを意味し、通常、関連するトレースコンテキストとBaggageを持ちます。非同期的な計装は、コレクション時にコールバックを介して呼び出され、コンテキストを持ちません。
- 加算とグルーピング: 加算Instrumentが合計を表すのに対し、グルーピングInstrumentは測定値のグループを特徴付けます。加算Instrumentに渡される数値は、代数的な意味での分割を定義しますが、グルーピングInstrumentに渡される数値は一般的に定義されません。加算Instrumentの値は常に和の一部ですが、グルーピングInstrumentの値は個々の測定値です。
- モノトニックなものと非モノトニックなもの: 加算Instrumentは、それが表現する量の微分が非負であるかどうかによって分類されます。モノトニックなInstrumentは主にレート値のモニタリングに役立ち、非ものとニックなInstrumentは主に合計値のモニタリングに役立ちます。

<!--
In the Event model, the primary data are (instrument, number) points, originally
observed in real time or on demand (for the synchronous and asynchronous cases,
respectively). The instruments and model use-cases will be described in greater
detail as we link the event model with the other two.
-->

イベントモデルでは、主なデータは(Instrument, 数)点であり、元々はリアルタイムまたはオンデマンドで観測されたものです(それぞれ同期型と非同期型の場合)。Instrumentとモデルの使用例については、イベントモデルを他の2つのモデルとリンクさせながら、より詳細に説明します。

<!--
### Timeseries Model
-->

### Timeseries モデル

<!--
In this low-level metrics data model, a Timeseries is defined by an entity
consisting of several metadata properties:
-->

この低レベルのメトリクスデータモデルでは、タイムスケールはいくつかのメタデータプロパティで構成されるエンティティで定義されます。

<!--
- Metric name and description
- Label set
- Kind of point (integer, floating point, etc)
- Unit of measurement
-->

- メトリック名と説明
- ラベルセット
- 点の種類(整数、浮動小数点、その他)
- 測定単位

<!--
The primary data of each timeseries are ordered (timestamp, value) points, for
three value types:
-->

各タイムスケールの主なデータは、3つの値のタイプに対応した順序(タイムスタンプ、値)のポイントです。

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

このモデルは、[Prometheus Remote Write](https://docs.google.com/document/d/1LPhVRSFkGNSuU1fBd81ulhsCPR4hkSZyyBj1SZ8fWOM/edit#heading=h.3p42p5s8n0ui)の理想化と見なすことができます。このプロトコルのように、我々は、暗黙的または明示的に存在しないことと比較して、ポイント値がいつ定義されるかを知ることにも関心があります。delta data points(XXX: 差分?)のメトリックストリームは、ポイントインタイム値ではなく、タイムインターバル値を定義します。 データの存在と不在を正確に定義するには、これらのモデルの対応関係をさらに発展させる必要があります。

<!--
### OpenTelemetry Protocol data model
-->

### OpenTelemetry Protocol データモデル

<!--
The OpenTelemetry data model for metrics includes four basic point kinds, all of
which satisfy the requirements above, meaning they define a decomposable
aggregate function (also known as a “natural merge” function) for points of the
same kind. <sup>[1](#otlpdatapointfn)</sup>
-->

OpenTelemetryのメトリクスのデータモデルには、4つの基本的なポイントの種類があり、それらはすべて上記の要件を満たし、同じ種類のポイントのための分解可能な集約関数(「ナチュラル・マージ」関数としても知られています)を定義しています。<sup>[1](#otlpdatapointfn)</sup> <sup>[1](#otlpdatapointfn)

<!--
The basic point kinds are:
-->

基本的なポイントの種類は以下のとおりです:

<!--
1. Monotonic Sum
2. Non-Monotonic Sum
3. Gauge
4. Histogram
-->

1. モノトニックな和
2. 非モノトニックな和
3. ゲージ
4. ヒストグラム

<!--
Comparing the OpenTelemetry and Timeseries data models, OTLP carries an
additional kind of point. Whereas an OTLP Monotonic Sum point translates into a
Timeseries Counter point, and an OTLP Histogram point translates into a
Timeseries Histogram point, there are two OTLP data points that become Gauges
in the Timeseries model: the OTLP Non-Monotonic Sum point and OTLP Gauge point.
-->

OpenTelemetryとTimeseriesのデータモデルを比較すると、OTLPには追加の種類のポイントがあります。OTLPモノトニックな和のポイントが時系列カウンターポイントに、OTLPヒストグラムポイントが時系列ヒストグラムポイントに変換されるのに対し、時系列モデルでゲージになるOTLPデータポイントは、OTLP非モノトニックな和のポイントとOTLPゲージポイントの2つです。

<!--
The two points that become Gauges in the Timeseries model are distinguished by
their built in aggregate function, meaning they define re-aggregation
differently. Sum points combine using addition, while Gauge points combine into
histograms.
-->

時系列モデルでゲージとなる2つのポイントは、内蔵された集計関数によって区別され、再集計の定義が異なります。和点は加算で結合し、ゲージ点はヒストグラムで結合します。

<!--
## Single-Writer
-->

## Single-Writer

<!--
Pending
-->

保留

<!--
## Temporarily
-->

## Temporarily

<!--
Pending
-->

保留

<!--
## Resources
-->

## Resources

<!--
Pending
-->

保留

<!--
## Temporal Alignment
-->

## Temporal Alignment

<!--
Pending
-->

保留

<!--
## External Labels
-->

## External Labels

<!--
Pending
-->

保留

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

