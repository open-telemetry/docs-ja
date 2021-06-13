<!--
# Metrics API
-->

# Metrics API

<!--
**Status**: [Experimental](../document-status.md)
-->

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

注意:この仕様は大きく変更される可能性があります。言語クライアントのメンテナに負担をかけないように、[OTEP](https://github.com/open-telemetry/oteps#opentelemetry-enhancement-proposal-otep)で明示的に伝えられない限り、OpenTelemetryクライアントが実装を開始することは推奨しません。

<!--
<!-- toc -->
-->

<!-- toc -->

<!--
- [Overview](#overview)
  * [Measurements](#measurements)
  * [Metric Instruments](#metric-instruments)
  * [Labels](#labels)
  * [Meter Interface](#meter-interface)
  * [Aggregations](#aggregations)
  * [Time](#time)
  * [Metric Event Format](#metric-event-format)
- [Meter provider](#meter-provider)
  * [Obtaining a Meter](#obtaining-a-meter)
  * [Global Meter provider](#global-meter-provider)
    + [Get the global MeterProvider](#get-the-global-meterprovider)
    + [Set the global MeterProvider](#set-the-global-meterprovider)
- [Instrument properties](#instrument-properties)
  * [Instrument naming requirements](#instrument-naming-requirements)
  * [Synchronous and asynchronous instruments compared](#synchronous-and-asynchronous-instruments-compared)
  * [Adding and grouping instruments compared](#adding-and-grouping-instruments-compared)
  * [Monotonic and non-monotonic instruments compared](#monotonic-and-non-monotonic-instruments-compared)
  * [Function names](#function-names)
- [The instruments](#the-instruments)
  * [Counter](#counter)
  * [UpDownCounter](#updowncounter)
  * [ValueRecorder](#valuerecorder)
  * [SumObserver](#sumobserver)
  * [UpDownSumObserver](#updownsumobserver)
  * [ValueObserver](#valueobserver)
  * [Interpretation](#interpretation)
  * [Constructors](#constructors)
- [Sets of labels](#sets-of-labels)
  * [Label performance](#label-performance)
  * [Option: Ordered labels](#option-ordered-labels)
- [Synchronous instrument details](#synchronous-instrument-details)
  * [Synchronous calling conventions](#synchronous-calling-conventions)
    + [Bound instrument calling convention](#bound-instrument-calling-convention)
    + [Direct instrument calling convention](#direct-instrument-calling-convention)
    + [RecordBatch calling convention](#recordbatch-calling-convention)
  * [Association with distributed context](#association-with-distributed-context)
    + [Baggage into metric labels](#baggage-into-metric-labels)
- [Asynchronous instrument details](#asynchronous-instrument-details)
  * [Asynchronous calling conventions](#asynchronous-calling-conventions)
    + [Single-instrument observer](#single-instrument-observer)
    + [Batch observer](#batch-observer)
  * [Asynchronous observations form a current set](#asynchronous-observations-form-a-current-set)
    + [Asynchronous instruments define moment-in-time ratios](#asynchronous-instruments-define-moment-in-time-ratios)
- [Concurrency](#concurrency)
- [Related OpenTelemetry work](#related-opentelemetry-work)
  * [Metric Views](#metric-views)
  * [OTLP Metric protocol](#otlp-metric-protocol)
  * [Metric SDK default implementation](#metric-sdk-default-implementation)
-->

- [概要](#概要)
  * [測定値](#測定値)
  * [Metric Instruments](#metric-instruments)
  * [ラベル](#ラベル)
  * [Meterインターフェース](#Meterインターフェース)
  * [集約(aggregation)](#集約-aggregation)
  * [時間](#時間)
  * [Metricイベントのフォーマット](#Metricイベントのフォーマット)
- [Meter Provider](#Meter-provider)
  * [Meterの取得](#meterの取得)
  * [グローバルMeter provider](#グローバルmeter-provider)
    + [グローバルMeterProviderの取得](#グローバルMeterProviderの取得)
    + [グローバルなMeterProviderの設定](#グローバルなMeterProviderの設定)
- [Instrumentプロパティ](#Instrumentプロパティ)
  * [Instrumentの名前の要件](#Instrumentの名前の要件)
  * [同期型と非同期型のInstrumentsの比較](#同期型と非同期型のInstrumentsの比較)
  * [加算とグループ化Instrumentsの比較](#加算とグループ化Instrumentsの比較)
  * [モノトニックと非モノトニックInstrumentsの比較](#モノトニックと非モノトニックInstrumentsの比較)
  * [関数名](#関数名)
- [Instrumens](#instrumens)
  * [Counter](#counter)
  * [UpDownCounter](#updowncounter)
  * [ValueRecorder](#valuerecorder)
  * [SumObserver](#sumobserver)
  * [UpDownSumObserver](#updownsumobserver)
  * [ValueObserver](#valueobserver)
  * [Interpretation](#interpretation)
  * [Constructors](#constructors)
- [ラベルセット](#ラベルセット)
  * [ラベルの性能](#ラベルの性能)
  * [Option: 順序付きラベル](#option-順序付きラベル)
- [同期型Instrumentsの詳細](#同期型Instrumentsの詳細)
  * [同期呼び出しの方式](#同期呼び出しの方式)
    + [バインドされたInstrumentsを呼び出す方式](#バインドされたInstrumentsを呼び出す方式)
    + [Instrumentsを直接呼び出す方式](#Instrumentsを直接呼び出す方式)
    + [RecordBatch呼び出し方式](#RecordBatch呼び出し方式)
  * [分散型コンテキストとの関連付け](#分散型コンテキストとの関連付け)
    + [MetricのラベルにBaggageを入れる](#MetricのラベルにBaggageを入れる)
- [非同期型Instrumentsの詳細](#非同期型Instrumentsの詳細)
  * [非同期型呼び出しの方式](#非同期型呼び出しの方式)
    + [Single-instrument observer](#single-instrument-observer)
    + [Batch observer](#batch-observer)
  * [Asynchronous observations form a current set](#asynchronous-observations-form-a-current-set)
    + [Asynchronous instruments define moment-in-time ratios](#asynchronous-instruments-define-moment-in-time-ratios)
- [Concurrency](#concurrency)
- [Related OpenTelemetry work](#related-opentelemetry-work)
  * [Metric Views](#metric-views)
  * [OTLP Metric protocol](#otlp-metric-protocol)
  * [Metric SDK default implementation](#metric-sdk-default-implementation)

<!-- tocstop -->

<!--
## Overview
-->

## 概要

<!--
The OpenTelemetry Metrics API supports capturing measurements about
the execution of a computer program at run time.  The Metrics API is
designed explicitly for processing raw measurements, generally with
the intent to produce continuous summaries of those measurements,
efficiently and simultaneously.  Hereafter, "the API" refers to the
OpenTelemetry Metrics API.
-->

OpenTelemetry Metrics APIは、コンピュータプログラムの実行に関する測定値を実行時に取得することをサポートします。Metrics APIは、生の測定値を処理するために明示的に設計されており、一般的には、それらの測定値の連続的なサマリーを効率的かつ同時に生成することを目的としています。以下、「API」はOpenTelemetry Metrics APIを指します。

<!--
The API provides functions for capturing raw measurements, through
several calling conventions that offer different levels of
performance.  Regardless of calling convention, we define a _metric
event_ as the logical thing that happens when a new measurement is
captured.  This moment of capture (at "run time") defines an implicit
timestamp, which is the wall time an SDK would read from a clock at
that moment.
-->

APIは、生の測定値をキャプチャするための関数を、異なるレベルのパフォーマンスを提供するいくつかの呼び出し方法によって提供します。呼び出し方法に関わらず、新しい測定値がキャプチャされたときに発生する論理的なものを_metric event(メトリックイベント)_と定義します。このキャプチャの瞬間("実行時")には、暗黙のタイムスタンプが定義されています。

<!--
The word "semantic" or "semantics" as used here refers to _how we give
meaning_ to metric events, as they take place under the API.  The term
is used extensively in this document to define and explain these API
functions and how we should interpret them.  As far as possible, the
terminology used here tries to convey the intended semantics, and a
_standard implementation_ will be described below to help us
understand their meaning.  Standard implementations perform
aggregation corresponding to the default interpretation for each kind
of metric event.
-->

ここで使われている「セマンティック」または「セマンティクス」という言葉は、APIの下で行われる計量イベントに「どのように意味を与えるか」を意味します。本ドキュメントでは、これらのAPI機能を定義・説明し、それらをどのように解釈すべきかを示すために、この用語を多用しています。ここで使用されている用語は、可能な限り、意図された意味を伝えるようにしており、その意味を理解するために、以下に _標準的な実装_ を説明します。標準的な実装では、メトリックイベントの種類ごとに、デフォルトの解釈に対応した集計を行います。

<!--
Monitoring and alerting systems commonly use the data provided through metric
events, after applying various [aggregations](#aggregations) and converting into
various exposition formats. However, we find that there are many other uses for
metric events, such as to record aggregated or raw measurements in tracing and
logging systems.  For this reason, [OpenTelemetry requires a separation of the
API from the SDK](../library-guidelines.md#requirements), so that different SDKs
can be configured at run time.
-->

モニタリングシステムやアラートシステムでは、メトリックイベントで提供されたデータに様々な[集約(Aggregation)](#aggregation)を施し、様々な表示形式に変換して使用するのが一般的です。しかし、トレースシステムやロギングシステムにおいて、集約された測定値や生の測定値を記録するなど、メトリックイベントには他にも多くの用途があることがわかります。このような理由から、[OpenTelemetryはAPIとSDKを分離することを要求しており](../library-guidelines.md#requirements)、ランタイムに異なるSDKを設定できるようにしています。

<!--
### Behavior of the API in the absence of an installed SDK
-->

### SDKがインストールされていない場合のAPIの動作について

<!--
In the absence of an installed Metrics SDK, the Metrics API MUST consist only
of no-ops. None of the calls on any part of the API can have any side effects
or do anything meaningful. Meters MUST return no-op implementations of any
instruments. From a user's perspective, calls to these should be ignored without raising errors
(i.e., *no* `null` references MUST be returned in languages where accessing these results in errors).
The API MUST NOT throw exceptions on any calls made to it.
-->

Metrics SDKがインストールされていない場合、Metrics APIはno-opsだけで構成されなければなりません(MUST)。APIのどの部分の呼び出しも、副作用を持つことも、意味のあることをすることもできません。メーターは、あらゆる楽器のno-op実装を返さなければなりません(MUST)。ユーザーの観点からすると、これらの呼び出しはエラーにならずに無視されるべきです(つまり、`null`参照にアクセスするとエラーになる言語では、`null`参照は*返してはいけません)。このAPIは、その呼び出しに対して例外を発生させてはなりません(MUST NOT)。

<!--
### Measurements
-->

### 測定値

<!--
The term _capture_ is used in this document to describe the action
performed when the user passes a measurement to the API.  The result
of a capture depends on the configured SDK, and if there is no SDK
installed, the default action is to do nothing in response to captured
events.  This usage is intended to convey that _anything can happen_
with the measurement, depending on the SDK, but implying that the user
has put effort into taking some kind of measurement.  For both
performance and semantic reasons, the API let users choose between two
kinds of measurement.
-->

このドキュメントでは、ユーザーが測定値をAPIに渡したときに実行されるアクションを表すために、_capture_という用語を使用しています。キャプチャーの結果は、構成されたSDKに依存します。SDKがインストールされていない場合、デフォルトのアクションは、キャプチャーされたイベントに対して何もしないことです。この使い方は、SDKに応じて測定結果に _何でも起こりうる_ ことを伝えるためのものですが、ユーザーが何らかの測定に力を入れていることを暗に示しています。パフォーマンスとセマンティックの両方の理由から、APIではユーザーが2種類の測定を選択できるようになっています。(XXX: ここの二種類とはどういうこと？)

<!--
The term _adding_ is used to specify a characteristic of some
measurements, meant to indicate that only the sum is considered useful
information.  These are measurements that you would naturally combine
using arithmetic addition, usually real quantities of something
(e.g., number of bytes).
-->

_加算(adding)_ という言葉は、いくつかの測定値の特徴を示すために使用され、その合計のみが有用な情報とみなされることを意味します。これらの測定値は、算術的な加算を使用して自然に結合されるもので、通常は何かの実際の量(例:バイト数)です。

<!--
Grouping measurements are used when the set of values, also known
as the population, is presumed to have useful information.  A
grouping measurement is either one that you would not naturally
combine using arithmetic addition (e.g., request latency), or it is a
measurement you would naturally add where the intention is to monitor
the distribution of values (e.g., queue size).  The median value is
considered useful information for grouping measurements.
-->

測定結果のグループ化は、値のセット(母集団とも呼ばれる)が有用な情報を持っていると推定される場合に使用されます。グループ化された測定値は、算術的な加算を使用して自然には結合されない測定値であるか(例:リクエストのレイテンシ)、または値の分布を監視する意図で自然に追加される測定値であるか(例:キュー・サイズ)のいずれかです。中央値は、測定値をグループ化するための有用な情報と考えられます。


<!--
Grouping instruments semantically capture more information than
adding instruments.  Grouping measurements are more expensive
than adding measurements, by this definition.  Users will choose
adding instruments except when they expect to get value from the
additional cost of information about individual values.  None of this
is to prevent an SDK from re-interpreting measurements based on
configuration.  Anything can happen with any kind of measurement.
-->

グループ化する計装は、計装を追加するよりも意味的に多くの情報を捉えます。この定義によれば、グループ化された測定値は、測定値を追加するよりも高価であるということになります。ユーザーは、個々の値に関する情報の追加コストから価値を得ることが期待される場合を除き、計装の追加を選択します。これらは、SDKが構成に基づいて測定値を再解釈することを妨げるものではありません。どんな種類の測定でも、何が起こっても不思議ではありません。


<!--
### Metric Instruments
-->

### Metric Instruments

<!--
A _metric instrument_ is a device for capturing raw measurements in
the API.  The standard instruments, listed in the table below, each have a dedicated
purpose.  The API purposefully avoids optional features that change
the semantic interpretation of an instrument; the API instead prefers
instruments that support a single method each with fixed interpretation.
-->

_Metric Instruments_は、APIで生の測定値を取り込むための装置です。以下の表に示す標準的なInstrumentsは、それぞれ専用の目的を持っています。APIは、Instrumentsの意味的な解釈を変えるようなオプション機能を意図的に避けています。その代わりに、APIは、固定された解釈を持つ単一のメソッドをサポートするInstrumentsを好んでいます。

<!--
All measurements captured by the API are associated with the
instrument used to make the measurement, thus giving the measurement its semantic properties.
Instruments are created and defined through calls to a `Meter` API,
which is the user-facing entry point to the SDK.
-->

APIによって取得されたすべての測定値は、その測定に使用されたInstrumentsに関連付けられ、その結果、測定値にセマンティックな特性が与えられます。Instrumentsは、SDKのユーザー向けエントリポイントである `Meter` APIを呼び出すことで作成および定義されます。

<!--
Instruments are classified in several ways that distinguish them from
one another.
-->

Instrumentsはいくつかの方法で分類されており、それらは互いに区別されています。

<!--
1. Synchronicity: A synchronous instrument is called by the user in a distributed [Context](../context/context.md) (i.e., with associated Span, Baggage, etc.). An asynchronous instrument is called by the SDK once per collection interval, lacking a Context.
2. Adding vs. Grouping: An adding instrument is one that records adding measurements, as opposed to a grouping instrument as described above.
3. Monotonicity: A monotonic instrument is an adding instrument, where the progression of sums is non-decreasing.  Monotonic instruments are useful for monitoring rate information.
-->

1. 同期性: 同期的なInstrumentsは、分散された[Context](../context/context.md)の中でユーザーによって呼び出されます(つまり、関連するSpan、Baggageなど)。非同期型のInstrumentsは、Contextを持たずに、収集間隔ごとに1回、SDKによって呼び出されます。

2. 加算vsグルーピング: 加算Instrumentsは、前述のグルーピングInstrumentsとは対照的に、加算の測定値を記録するものです。
3. Monotonicity: Monotonic Instrumentsとは加算Instrumentsのことで、加算の進行が非減少性(訳注: 減ることがなく、増加するのみ)であることが特徴です。Monotonic Instrumentsは、レート情報をモニターするのに便利です。

<!--
The metric instruments names are shown below along with whether they
are synchronous, adding, and/or monotonic.
-->

Metric Instrumentsの名称は、同期、加算、Monotonicのいずれかと合わせて以下の通りです。

<!--
| Name | Synchronous | Adding | Monotonic |
| ---- | ----------- | -------- | --------- |
| Counter           | Yes | Yes | Yes |
| UpDownCounter     | Yes | Yes | No  |
| ValueRecorder     | Yes | No  | No  |
| SumObserver       | No  | Yes | Yes |
| UpDownSumObserver | No  | Yes | No  |
| ValueObserver     | No  | No  | No  |
-->

| 名前 | Synchronous | Adding | Monotonic |
| ---- | ----------- | -------- | --------- |
| Counter           | Yes | Yes | Yes |
| UpDownCounter     | Yes | Yes | No  |
| ValueRecorder     | Yes | No  | No  |
| SumObserver       | No  | Yes | Yes |
| UpDownSumObserver | No  | Yes | No  |
| ValueObserver     | No  | No  | No  |

<!--
The synchronous instruments are useful for measurements that are
gathered in a distributed [Context](../context/context.md) (i.e., with associated Span, Baggage, etc.).  The asynchronous instruments are
useful when measurements are expensive, therefore should be gathered
periodically.  Read more [characteristics of synchronous and
asynchronous instruments](#synchronous-and-asynchronous-instruments-compared) below.
-->

同期型Instrumentsは、分散した[Context](../context/context.md)に集められた計測値(つまり、関連するSpanやBaggageなど)に有用です。非同期型Instrumentsは、測定の行為が高価であるため、定期的に収集する必要がある場合に便利です。詳しくは[同期型と非同期型のInstrumentsの比較](#同期型と非同期型のInstrumentsの比較)をご覧ください。


<!--
The synchronous and asynchronous adding instruments have a
significant difference: synchronous instruments are used to capture
changes in a sum, whereas asynchronous instruments are used to capture
sums directly.  Read more [characteristics of adding
instruments](#adding-and-grouping-instruments-compared) below.
-->

同期型Instrumentsと非同期型Instrumentsには大きな違いがあり、同期型Instrumentsは和の変化を捉えるのに対し、非同期型Instrumentsは和を直接捉えることができます。詳しくは以下の[加算Instrumentsの特徴](#加算とグループ化Instrumentsの比較)をご覧ください。

<!--
The monotonic adding instruments are significant because they support rate
calculations.  Read more information about [choosing metric
instruments](#monotonic-and-non-monotonic-instruments-compared) below.
-->

モノトニックInstrumentsはレート計算に対応していることに意義があります。[Instrumentsの選び方](#モノトニックと非モノトニックInstrumentsの比較)の詳細は以下をご覧ください。

<!--
An _instrument definition_ describes several properties of the
instrument, including its name and its kind.  The other properties of
a metric instrument are optional, including a description and the unit
of measurement.  An instrument definition is associated with the
data that it produces.
-->

_Instrumentsの定義_は、Instrumentsの名前や種類など、Instrumentsのいくつかのプロパティを記述します。Metric Instrumentsのその他のプロパティは、説明や測定単位などで任意です。Instrumentsの定義は、それが生成するデータに関連付けられています。

<!--
### Labels
-->

### ラベル

<!--
_Label_ is the term used to refer to a key-value attribute associated
with a metric event, similar to a [Span
attribute](../trace/api.md#span) in the tracing API.  Each label
categorizes the metric event, allowing events to be filtered and
grouped for analysis.
-->

_ラベル_は、メトリックイベントに関連付けられたキーバリュー属性を指す用語で、Tracing APIの[Span属性](../trace/api.md#span)と同様のものです。各ラベルはメトリックイベントを分類し、分析のためにイベントをフィルタリングしてグループ化できます。

<!--
Each of the instrument calling conventions (detailed below) accepts a
set of labels as an argument.  The set of labels is defined as a
unique mapping from key to value.  Typically, labels are passed to the
API in the form of a list of key:values, in which case the
specification dictates that duplicate entries for a key are resolved
by taking the last value to appear in the list.
-->

各Instrumentsの呼び出し規則(以下に詳述)は、引数としてラベルのセットを受け取ります。ラベルのセットは、キーから値へのユニークなマッピングとして定義されます。通常、ラベルはkey:valuesのリストの形でAPIに渡されます。この場合、仕様ではkeyの重複エントリはリストに現れた最後の値を取ることで解決されます。

<!--
Measurements by a synchronous instrument are commonly combined with
other measurements having exactly the same label set, which enables
significant optimizations.  Read more about [combining measurements
through aggregation](#aggregations) below.
-->

同期型Instrumentsによる測定値は、まったく同じラベル・セットを持つ他の測定値と一般的に結合され、大幅な最適化が可能になります。[集約による測定値の結合](#集約-aggregation)の詳細は以下をご覧ください。

<!--
### Meter Interface
-->

### Meter インターフェース

<!--
The API defines a `Meter` interface.  This interface consists of a set
of instrument constructors, and a facility for capturing batches of
measurements in a semantically atomic way.
-->

このAPIでは、`Meter`インターフェースを定義しています。このインターフェースは、Instrumentsのコンストラクタのセットと、意味的にアトミックな方法で測定値のバッチをキャプチャする機能で構成されています。


<!--
There is a global `Meter` instance available for use that facilitates
automatic instrumentation for third-party code.  Use of this instance
allows code to statically initialize its metric instruments, without
explicit dependency injection.  The global `Meter` instance acts as a
no-op implementation until the application initializes a global
`Meter` by installing an SDK either explicitly, through a service
provider interface, or other language-specific support.  Note that it
is not necessary to use the global instance: multiple instances of the
OpenTelemetry SDK may run simultaneously.
-->

サードパーティのコードの自動的な計装を促進するために、グローバルな`Meter`インスタンスが用意されています。このインスタンスを使用することで、コードは、明示的な依存関係の注入を行わずに、静的にInstrumentsを初期化することができます。グローバルな `Meter` インスタンスは、アプリケーションが SDK を明示的にインストールするか、サービスプロバイダインターフェイスを介してインストールするか、あるいはその他の言語固有のサポートによってグローバルな `Meter` を初期化するまでは、なにもしない実装として動作します。なお、グローバルインスタンスを使用する必要はないことに注意してください。また、OpenTelemetry SDKの複数のインスタンスを同時に実行できます。

<!--
As an obligatory step, the API requires the caller to provide the name of the
instrumenting library (optionally, the version) when obtaining a `Meter`
implementation.  The library name is meant to be used for identifying
instrumentation produced from that library, for such purposes as disabling
instrumentation, configuring aggregation, and applying sampling policies.  See
the specification on [TracerProvider](../trace/api.md#tracerprovider) for more
details.
-->

義務的なステップとして、APIでは、`Meter`の実装を取得する際に、呼び出し側が計装ライブラリの名前(オプションでバージョンも)を提供することを要求しています。ライブラリ名は、計装の無効化、集約の設定、サンプリング・ポリシーの適用などの目的で、そのライブラリから生成された計装を識別するために使用されることを意図しています。詳細は[TracerProvider]の仕様書(../trace/api.md#tracerprovider)を参照してください。

<!--
### Aggregations
-->

### 集約(Aggregation)

<!--
_Aggregation_ refers to the process of combining multiple measurements
into exact or estimated statistics about the measurements that took
place during an interval of time, during program execution.
-->

_集約(Aggregation)_とは、複数の測定値を組み合わせて、プログラム実行中のある一定期間に発生した測定値に関する正確なまたは推定される統計値を算出するプロセスのことです。

<!--
Each instrument specifies a default aggregation that is suited to the
semantics of the instrument, that serves to explain its properties and
give users an understanding of how it is meant to be used.
Instruments, in the absence of any configuration override, can be
expected to deliver a useful, economical aggregation out of the box.
-->

各Instrumentsは、そのInstrumentsのセマンティクスに適したデフォルトの集約方法を指定し、そのプロパティを説明し、どのように使用するかをユーザーに理解してもらいます。Instrumentsは、設定を変更しなくても、便利で経済的な集約機能をすぐに提供することが期待されます。

<!--
The adding instruments (`Counter`, `UpDownCounter`, `SumObserver`,
`UpDownSumObserver`) use a Sum aggregation by default.  Details about
computing a Sum aggregation vary, but from the user's perspective this
means they will be able to monitor the sum of values captured.  The
distinction between synchronous and asynchronous instruments is
crucial to specifying how exporters work, a topic that is covered in
the [SDK specification (WIP)](https://github.com/open-telemetry/opentelemetry-specification/pull/347).
-->

加算Instruments(`Counter`, `UpDownCounter`, `SumObserver`, `UpDownSumObserver`)は、デフォルトでSum集約を使用します。Sum集約の計算方法の詳細は様々ですが、ユーザーの視点では、取得した値の合計をモニターできることを意味します。同期型Instrumentsと非同期型Instrumentsの区別は、エクスポーターの動作を規定する上で非常に重要です。このトピックは[SDK specification (WIP)](https://github.com/open-telemetry/opentelemetry-specification/pull/347)でカバーされています。

<!--
The `ValueRecorder` instrument uses [TBD issue
636](https://github.com/open-telemetry/opentelemetry-specification/issues/636)
aggregation by default.
-->

`ValueRecorder` Instrumentsは、デフォルトで[TBD issue 636](https://github.com/open-telemetry/opentelemetry-specification/issues/636)の集約を使用します。

<!--
The `ValueObserver` instrument uses LastValue aggregation by default.
This aggregation keeps track of the last value that was observed and
its timestamp.
-->

`ValueObserver` Instrumentsは、デフォルトで最新値の集約を使用します。この集約は、観測された最後の値とそのタイムスタンプを記録します。

<!--
Other standard aggregations are available, especially for grouping
instruments, where we are generally interested in a variety of
different summaries, such as histograms, quantile summaries,
cardinality estimates, and other kinds of sketch data structure.
-->

他にも標準的な集計方法があり、特にグループ化されたInstrumentsでは、ヒストグラム、分位値の集計、カーディナリティの推定値、その他の種類のスケッチデータ構造など、様々な異なる集計方法が広く使用されてます。

<!--
The default OpenTelemetry SDK implements a [Views API (WIP)](https://github.com/open-telemetry/oteps/pull/89), which
supports configuring non-default aggregation behavior(s) on the level
of an individual instrument.  Even though OpenTelemetry SDKs can be
configured to treat instruments in non-standard ways, users are
expected to select instruments based on their semantic meaning, which
is explained using the default aggregation.
-->

デフォルトのOpenTelemetry SDKは[Views API (WIP)](https://github.com/open-telemetry/oteps/pull/89)を実装しており、個々のInstrumentsのレベルでデフォルト以外の集約動作を設定することをサポートしています。OpenTelemetry SDKは、Instrumentsを非標準的な方法で扱うように設定することができますが、ユーザーは、デフォルトの集約を使って説明される意味的な意味に基づいてInstrumentsを選択することが期待されます。

<!--
### Time
-->

### 時間

<!--
Time is a fundamental property of metric events, but not an explicit
one.  Users do not provide explicit timestamps for metric events.
SDKs are discouraged from capturing the current timestamp for each
event (by reading from a clock) unless there is a definite need for
high-precision timestamps calculated on every event.
-->

時間はメトリックイベントの基本的なプロパティですが、明示的なものではありません。ユーザーはメトリックイベントに明示的なタイムスタンプを提供しません。SDKは、すべてのイベントで計算された高精度のタイムスタンプの明確な必要性がない限り、各イベントの現在のタイムスタンプを(時計から読み取って)キャプチャすることは推奨されません。

<!--
This non-requirement stems from a common optimization in metrics
reporting, which is to configure metric data collection with a
relatively small period (e.g., 1 second) and use a single timestamp to
describe a batch of exported data, since the loss of precision is
insignificant when aggregating data across minutes or hours of data.
-->

これは、メトリクスデータの収集を比較的小さな期間(1秒など)で設定し、エクスポートされたデータのバッチを記述するために単一のタイムスタンプを使用するという、メトリクスレポートにおける一般的な最適化に由来するものです。これは、数分または数時間のデータを集約する際の精度の低下が重要ではないためです。

<!--
Aggregations are commonly computed over a series of events that fall
into a contiguous region of time, known as the collection interval.
Since the SDK controls the decision to start collection, it is possible to
collect aggregated metric data while only reading the clock once per
collection interval.  The default SDK takes this approach.
-->

集約は通常、収集間隔と呼ばれる連続した時間領域に含まれる一連のイベントを対象に計算されます。収集開始の判断はSDKが行うため、収集間隔ごとに1回だけ時計を読み取るだけで、集約されたメトリックデータを収集することができます。デフォルトのSDKではこの方法を採用しています。

<!--
Metric events produced with synchronous instruments happen at an
instant in time, thus fall into a collection interval where they are
aggregated together with other events from the same instrument and
label set.  Because events may happen simultaneously with one another,
the _most recent event_ is technically not well defined.
-->

同期型Instrumentsで生成されたメトリックイベントは、ある瞬間に発生するため、同じInstrumentsやラベルセットからの他のイベントと一緒に集約される収集区間に入ります。イベントは互いに同時に発生する可能性があるため、 _最新のイベント_ は技術的にうまく定義できません。

<!--
Asynchronous instruments allow the SDK to evaluate metric instruments
through observations made once per collection interval.  Because of this
coupling with collection (unlike synchronous instruments),
these instruments unambiguously define the most recent event.  We
define the _Last Value_ of an instrument and label set, with repect to
a moment in time, as the value that was measured during the most
recent collection interval.
-->

非同期型Instrumentsは、SDKが収集間隔ごとに1回行う観測によって、Metric Instrumentsを評価することができます。同期型Instrumentsとは異なり、このように収集と観測が結合しているため、これらのInstrumentsは最新のイベントを明確に定義します。我々はInstrumentsとラベルセットの _最新値(Last Value)_ を、ある時点を基準に、直近の収集インターバルで測定された値と定義しています。

<!--
Because metric events are implicitly timestamped, we could refer to a
series of metric events as a _time series_. However, we reserve the
use of this term for the SDK specification, to refer to parts of a
data format that express explicitly timestamped values, in a sequence,
resulting from an aggregation of raw measurements over time.
-->

メトリックイベントは暗黙的にタイムスタンプが付与されるため、一連のメトリックイベントを _時系列(time series)_ と呼ぶことができます。しかし、SDKの仕様では、この用語の使用を留保し、生の測定値を時間経過とともに集計した結果、明示的にタイムスタンプが付与された値を一連の流れで表現するデータフォーマットの部分を指すことにしました。

<!--
### Metric Event Format
-->

### Metricイベントのフォーマット

<!--
Metric events have the same logical representation, regardless of
instrument kind.  Metric events captured through any instrument
consist of:
-->

メトリックイベントは、Instrumentsの種類にかかわらず、同じ論理的表現を持っています。任意のInstrumentsで捕捉されたメトリックイベントは次のように構成されます。

<!--
- timestamp (implicit)
- instrument definition (name, kind, description, unit of measure)
- label set (keys and values)
- value (signed integer or floating point number)
- [resources](../resource/sdk.md) associated with the SDK at startup.
-->

- タイムスタンプ(暗黙的)
- Instrumentsの定義(名称、種類、説明、測定単位)
- ラベルセット (キーとバリュー)
- バリュー (符号付き整数または浮動小数点数)
- 起動時にSDKに関連付けられた[resources](../resource/sdk.md)

<!--
Synchronous events have one additional property, the distributed
[Context](../context/context.md) (containing Span, Baggage, etc.)
that was active at the time.
-->

同期イベントには、その時点でアクティブだった分散型の[Context](../context/context.md)(Span、Baggageなどを含む)というプロパティが1つ追加されます。

<!--
## Meter provider
-->

## Meter Provider

<!--
A concrete `MeterProvider` implementation can be obtained by initializing and
configuring an OpenTelemetry Metrics SDK.  This document does not
specify how to construct an SDK, only that they must implement the
`MeterProvider`.  Once configured, the application or library chooses
whether it will use a global instance of the `MeterProvider`
interface, or whether it will use dependency injection for greater
control over configuring the provider.
-->

具体的な`MeterProvider`の実装は、OpenTelemetry Metrics SDKを初期化して設定することで得られます。このドキュメントでは、SDKをどのように構築するかは規定しておらず、`MeterProvider`を実装しなければならないということだけを規定しています。設定が完了したら、アプリケーションやライブラリは、`MeterProvider`インターフェースのグローバルなインスタンスを使用するか、あるいは、依存性注入を使用してプロバイダの設定をより細かく制御するかを選択します。

<!--
### Obtaining a Meter
-->

### Meterの取得

<!--
New `Meter` instances can be created via a `MeterProvider` and its
`GetMeter(name, version)` method.  `MeterProvider`s are generally expected to
be used as singletons.  Implementations SHOULD provide a single global
default `MeterProvider`. The `GetMeter` method expects two string
arguments:
-->

新しい `Meter` インスタンスは `MeterProvider` とその `GetMeter(name, version)` メソッドによって生成できます。`MeterProvider` は一般的にシングルトンとして使用されることが期待されます。実装では、単一のグローバルなデフォルトの `MeterProvider` を提供すべきです(SHOULD)。`GetMeter`メソッドは2つの文字列の引数を取ります。

<!--
- `name` (required): This name must identify the instrumentation library (e.g. `io.opentelemetry.contrib.mongodb`)
  and *not* the instrumented library.
  In case an invalid name (null or empty string) is specified, a working default `Meter` implementation is returned as a fallback
  rather than returning null or throwing an exception.
  A `MeterProvider` could also return a no-op `Meter` here if application owners configure the SDK to suppress telemetry produced by this library.
- `version` (optional): Specifies the version of the instrumentation library (e.g. `1.0.0`).
-->

- `name` (必須)。この名前は、計装ライブラリ(例:`io.opentelemetry.contrib.mongodb`)を特定するものであり、計装ライブラリでは*ない*必要があります。無効な名前(NULLまたは空の文字列)が指定された場合には、NULLを返したり、例外をスローするのではなく、動作するデフォルトの`Meter`の実装がフォールバックとして返されます。アプリケーションのオーナーが、このライブラリから生成されるテレメトリを抑制するようにSDKを設定している場合、`MeterProvider` は、ここで何もしない `Meter` を返すこともできます。
- `version`(任意)。計装ライブラリのバージョンを指定します(例:`1.0.0`)

<!--
Each distinctly named `Meter` establishes a separate namespace for its
metric instruments, making it possible for multiple instrumentation
libraries to report the metrics with the same instrument name used by
other libraries.  The name of the `Meter` is explicitly not intended
to be used as part of the instrument name, as that would prevent
instrumentation libraries from capturing metrics by the same name.
-->

固有の名前を持つ`Meter`は、Metric Instrumentsのための個別の名前空間を確立し、複数の計装ライブラリが、他のライブラリが使用する同じInstruments名でメトリックを報告することを可能にします。計器ライブラリが同じ名前のメトリクスを捕捉できなくなるため、`Meter`の名前はInstruments名の一部として使用することは明示的に意図されていません。

<!--
### Global Meter provider
-->

### グローバルMeter provider

<!--
Use of a global instance may be seen as an anti-pattern in many
situations, but in most cases it is the correct pattern for telemetry
data, in order to combine telemetry data from inter-dependent
libraries _without use of dependency injection_.  OpenTelemetry
language APIs SHOULD offer a global instance for this reason.
Languages that offer a global instance MUST ensure that `Meter`
instances allocated through the global `MeterProvider` and instruments
allocated through those `Meter` instances have their initialization
deferred until the a global SDK is first initialized.
-->

グローバル・インスタンスの使用は、多くの状況でアンチパターンと見なされるかもしれませんが、ほとんどの場合、依存関係にあるライブラリからのテレメトリデータを、依存性注入を使用せずに組み合わせるためには、テレメトリデータにとって正しいパターンです。OpenTelemetry言語のAPIは、この理由からグローバル・インスタンスを提供すべきです(SHOULD)。グローバルなインスタンスを提供する言語は、グローバルな `MeterProvider` を通じて割り当てられた `Meter` インスタンスと、それらの `Meter` インスタンスを通じて割り当てられたInstrumentsが、グローバルな SDK が最初に初期化されるまで、その初期化が延期されるようにしなければなりません(MUST)。

<!--
#### Get the global MeterProvider
-->

#### グローバルMeterProviderの取得

<!--
Since the global `MeterProvider` is a singleton and supports a single
method, callers can obtain a global `Meter` using a global `GetMeter`
call.  For example, `global.GetMeter(name, version)` calls `GetMeter`
on the global `MeterProvider` and returns a named `Meter` instance.
-->

グローバルな `MeterProvider` はシングルトンであり、単一のメソッドをサポートしているので、呼び出し側はグローバルな `GetMeter` コールを使ってグローバルな `Meter` を取得することができる。例えば、`global.GetMeter(name, version)` は、グローバルな `MeterProvider` の `GetMeter` を呼び出し、名前付きの `Meter` インスタンスを返します。

<!--
#### Set the global MeterProvider
-->

#### グローバルなMeterProviderの設定

<!--
A global function installs a MeterProvider as the global SDK.  For
example, use `global.SetMeterProvider(MeterProvider)` to install the
SDK after it is initialized.
-->

グローバル関数はグローバルSDKとしてMeterProviderをインストールします。例えば、SDKを初期化した後にインストールするには、`global.SetMeterProvider(MeterProvider)`を使用してください。

<!--
## Instrument properties
-->

## Instrumentプロパティ

<!--
Because the API is separated from the SDK, the implementation
ultimately determines how metric events are handled.  Therefore, the
choice of instrument should be guided by semantics and the intended
interpretation.  The semantics of the individual instruments is
defined by several properties, detailed here, to assist with
instrument selection.
-->

APIはSDKから分離されているため、メトリックイベントをどのように処理するかは、最終的には実装が決定します。従って、Instrumentsの選択は、セマンティクスと意図した解釈に基づいて行う必要があります。個々のInstrumentsのセマンティクスはいくつかのプロパティで定義されており、ここではInstrumentsの選択に役立つ情報を提供します。

<!--
### Instrument naming requirements
-->

### Instrumentの名前の要件

<!--
Metric instruments are primarily defined by their name, which is how
we refer to them in external systems.  Metric instrument names conform
to the following syntax:
-->

Metric instrumentsは、主にその名前で定義され、外部システムでの参照方法となります。Metric Instrumentsの名前は次のような構文になっています。

<!--
1. They are non-empty strings
2. They are case-insensitive
3. The first character must be non-numeric, non-space, non-punctuation
4. Subsequent characters must belong to the alphanumeric characters, '\_', '.', and '-'.
-->

1. 空ではない文字列であること
2. 大文字小文字を区別しないこと
3. 最初の文字は、非数字、非スペース、非句読点でなければならない
4. 後続の文字は、英数字の'\'、.'、'-'に属するものとする

<!--
Metric instrument names belong to a namespace, established by the the
associated `Meter` instance.  `Meter` implementations MUST return an
error when multiple instruments are registered by the same name.
-->

Metric instrumentは、関連する `Meter` インスタンスによって確立される名前空間に属します。`Meter` の実装では、複数の計測器が同じ名前で登録されている場合、エラーを返さなければなりません(MUST)。

<!--
TODO: [The following paragraph is a placeholder for a more-detailed
document that is needed.](https://github.com/open-telemetry/opentelemetry-specification/issues/600)
-->

TODO: [以下の段落は、より詳細な文書が必要な場合のプレースホルダーです](https://github.com/open-telemetry/opentelemetry-specification/issues/600)

<!--
Metric instrument names SHOULD be semantically meaningful, independent
of the originating Meter name.  For example, when instrumenting an
http server library, "latency" is not an appropriate instrument name,
as it is too generic.  Instead, as an example, we should favor a name
like "http\_request\_latency", as it would inform the viewer of the
semantic meaning of the latency measurement.  Multiple instrumentation
libraries may be written to generate this metric.
-->

  Metric instrumentの名前は、元のMeterの名前とは別に、意味のあるものにすべきです(SHOULD)。たとえば、httpサーバーライブラリを計装する場合、「latency」は一般的すぎるため、適切なInstruments名ではありません。代わりに、例として「http\_request\_latency」のような名前を使用すると、視聴者にレイテンシー測定の意味を伝えることができます。このメトリクスを生成するために、複数の計装ライブラリを記述することができます。

<!--
### Synchronous and asynchronous instruments compared
-->

### 同期型と非同期型のInstrumentsの比較

<!--
Synchronous instruments are called inside a request, meaning they
have an associated distributed [Context](../context/context.md) (with Span, Baggage, etc.).  Multiple metric events may occur for a
synchronous instrument within a given collection interval.
-->

同期型Instrumentsは、リクエストの中で呼び出されます。つまり、関連する分散型の[Context](../context/context.md)を持っています(Span、Baggageなど)。1つの同期型Instrumentsに対して、所定の収集間隔内で複数のメトリックイベントが発生する可能性があります。

<!--
Asynchronous instruments are reported by a callback, once per
collection interval, and lack Context.  They are permitted to
report only one value per distinct label set per period.  If the
application observes multiple values for the same label set, in a
single callback, the last value is the only value kept.
-->

非同期型Instrumentsは、収集間隔ごとに1回、コールバックによって報告され、コンテキストはありません。非同期型Instrumentsは、収集間隔ごとに1回、コールバックによって報告されます。アプリケーションが1つのコールバックで同じラベルセットの複数の値を観測した場合、最後の値が唯一の値として保存されます。

<!--
To ensure that the definition of last value is consistent across
asynchronous instruments, the timestamp associated with asynchronous
events is fixed to the timestamp at the end of the interval in which
it was computed.  All asynchronous events are timestamped with the end
of the interval, which is the moment they become the last value
corresponding to the instrument and label set.  (For this reasons,
SDKs SHOULD run asynchronous instrument callbacks near the end of the
collection interval.)
-->

最終値の定義が非同期型Instrumentsで一貫しているように、非同期イベントに関連するタイムスタンプは、それが計算されたインターバルの終了時のタイムスタンプに固定されています。すべての非同期イベントのタイムスタンプはインターバル終了時のもので、これはInstrumentsとラベルセットに対応する最終値になった瞬間です。(この理由から、SDKは収集インターバルの終わり近くに非同期機器コールバックを実行するべきです)。

<!--
### Adding and grouping instruments compared
-->

### 加算とグループ化Instrumentsの比較

<!--
Adding instruments are used to capture information about a sum,
where, by definition, only the sum is of interest.  Individual events
are considered not meaningful for these instruments, the event count
is not computed.  This means, for example, that two `Counter` events
`Add(N)` and `Add(M)` are equivalent to one `Counter` event `Add(N +
M)`.  This is the case because `Counter` is synchronous, and
synchronous adding instruments are used to capture changes to a sum.
-->

加算Instrumentsは合計に関する情報を得るために使用され、定義上、合計のみが興味の対象となる。個々のイベントはこれらのInstrumentsでは意味がないと考えられ、イベントカウントは計算されません。これは、例えば、2つの`Counter`イベント `Add(N)`と`Add(M)`は、1つの`Counter`イベント `Add(N + M)`と同等であることを意味します。これは、`Counter` が同期的であり、同期型Instrumentsが和の変化を捉えるために使用されるためです。


<!--
Asynchronous, adding instruments (e.g., `SumObserver`) are used to
capture sums directly.  This means, for example, that in any sequence
of `SumObserver` observations for a given instrument and label set,
the Last Value defines the sum of the instrument.
-->

非同期で加算型のInstruments(例:`SumObserver`)は、合計値を直接捉えるために使用されます。これは例えば、任意のInstrumentsとラベル・セットに対する `SumObserver` 観測のあらゆるシーケンスにおいて、Last Value がInstrumentsの合計を定義することを意味します。

<!--
In both synchronous and asynchronous cases, the adding instruments
are inexpensively aggregated into a single number per collection interval
without loss of information.  This property makes adding instruments
higher performance, in general, than grouping instruments.
-->

同期の場合も非同期の場合も、加算Instrumentsは情報を失うことなく、収集間隔ごとに安価に1つの数値に集約されます。この特性により、一般的に、加算Instrumentsはグループ化Instrumentsよりも高性能となります。

<!--
Grouping instruments use a relatively inexpensive aggregation,
by default, compared with recording full data, but still more expensive aggregation than the
default for adding instruments (Sum).  Unlike adding instruments,
where only the sum is of interest by definition, grouping
instruments can be configured with even more expensive aggregators.
-->

グループ化されたInstrumentsは、フルデータを記録する場合に比べて比較的安価な集計をデフォルトで使用しますが、Instrumentsを追加する場合のデフォルト(Sum)よりはまだ高価な集計を使用します。加算Instrumentsでは、定義上、合計のみが対象となるのに対し、グループ化Instrumentsでは、さらに高価な集計Instrumentsを設定できます。

<!--
### Monotonic and non-monotonic instruments compared
-->

### モノトニックと非モノトニックInstrumentsの比較

<!--
Monotonicity applies only to adding instruments.  `Counter` and
`SumObserver` instruments are defined as monotonic because the sum
captured by either instrument is non-decreasing.  The `UpDown-`
variations of these two instruments are non-monotonic, meaning the sum
can increase, decrease, or remain constant without any guarantees.
-->

モノトニックは加算Instrumentsにのみ適用されます。`Counter`と`SumObserver`のInstrumentsは、どちらのInstrumentsでも合計が減少しないので、単調と定義されます。これらの2つのInstrumentsの `UpDown-` のバリエーションは非単調で、合計が保証なしに増加、減少、または一定になる可能性があることを意味しています。

<!--
Monotonic instruments are commonly used to capture information about a
sum that is meant to be monitored as a rate.  The Monotonic property
is defined by this API to refer to a non-decreasing sum.
Non-increasing sums are not considered a feature in the Metric API.
-->

単モノトニック Instrumentsは、一般的に、レートとしてモニターされることを意図した和に関する情報を取得するために使用されます。Monotonic プロパティは、この API で定義されており、減少しない和を指します。増加しない和は、Metric APIでは機能とみなされません。

<!--
### Function names
-->

### 関数名

<!--
Each instrument supports a single function, named to help convey the
instrument's semantics.
-->

各Instrumensは1つの関数をサポートしており、Instrumensのセマンティクスを伝えるために名前が付けられています。

<!--
Synchronous adding instruments support an `Add()` function,
signifying that they add to a sum and do not directly capture a sum.
-->

同期型は`Add()`関数をサポートしています。これは、和に加算することを意味し、和を直接取り込むことはありません。

<!--
Synchronous grouping instruments support a `Record()` function,
signifying that they capture individual events, not only a sum.
-->

同期型グループ化Instrumensは、 `Record()` 関数をサポートしており、これは、合計だけでなく個々のイベントを捕捉することを意味します。

<!--
Asynchronous instruments all support an `Observe()` function,
signifying that they capture only one value per measurement interval.
-->

非同期型Instrumensはすべて`Observe()`関数をサポートしており、これは計測間隔ごとに1つの値だけをキャプチャすることを意味します。

<!--
## The instruments
-->

## Instrumens

<!--
### Counter
-->

### Counter

<!--
`Counter` is the most common synchronous instrument.  This instrument
supports an `Add(increment)` function for reporting a sum, and is
restricted to non-negative increments.  The default aggregation is
`Sum`, as for any adding instrument.
-->

`Counter`は最も一般的な同期型のInstrumensです。このInstrumensは合計を報告するための `Add(increment)` 関数をサポートしており、非負の増分に制限されています。デフォルトの集計は、他のInstrumensと同様に `Sum` です。

<!--
Example uses for `Counter`:
-->

`Counter`の使用例:

<!--
- count the number of bytes received
- count the number of requests completed
- count the number of accounts created
- count the number of checkpoints run
- count the number of 5xx errors.
-->

- 受信したバイト数をカウント
- リクエスト完了数のカウント
- 作成されたアカウント数のカウント
- 実行されたチェックポイントのカウント
- 5xxエラーの数をカウント


<!--
These example instruments would be useful for monitoring the rate of
any of these quantities.  In these situations, it is usually more
convenient to report by how much a sum changes, as it happens, than to
calculate and report the sum on every measurement.
-->

これらのInstrumentsの例は、これらの量のいずれかの割合を監視するのに便利です。このような状況では、測定のたびに合計を計算して報告するよりも、合計がどのくらい変化したかを報告する方が通常は便利です。

<!--
### UpDownCounter
-->

### UpDownCounter

<!--
`UpDownCounter` is similar to `Counter` except that `Add(increment)`
supports negative increments.  This makes `UpDownCounter` not useful
for computing a rate aggregation.  It aggregates a `Sum`, only the sum
is non-monotonic.  It is generally useful for capturing changes in an
amount of resources used, or any quantity that rises and falls during a
request.
-->

`UpDownCounter`は`Counter`と似ていますが、`Add(increment)`が負の増分をサポートしている点が異なります。これにより、`UpDownCounter`はレート集計の計算には使えません。これは `Sum` を集約しますが、その合計は非単調です。これは一般的に、使用されたリソースの量や、リクエスト中に上昇したり下降したりする量の変化を捉えるのに便利です。

<!--
Example uses for `UpDownCounter`:
-->

`UpDownCounter`の使用例:

<!--
- count the number of active requests
- count memory in use by instrumenting `new` and `delete`
- count queue size by instrumenting `enqueue` and `dequeue`
- count semaphore `up` and `down` operations.
-->

- アクティブなリクエストのカウント
- `new` と `delete` に計装して使用中のメモリをカウントする
- `enqueue` と `dequeue` に計装して、キューのサイズをカウントする。
- セマフォの `up` と `down` の操作をカウントす

<!--
These example instruments would be useful for monitoring resource
levels across a group of processes.
-->

これらのInstrumentsの例は、プロセスのグループ全体のリソースレベルを監視するのに便利です。

<!--
### ValueRecorder
-->

### ValueRecorder

<!--
`ValueRecorder` is a grouping synchronous instrument useful for
recording any grouping number, positive or negative.  Values
captured by a `Record(value)` are treated as individual events
belonging to a distribution that is being summarized.  `ValueRecorder`
should be chosen either when capturing measurements that do not
contribute meaningfully to a sum, or when capturing numbers that are
adding in nature, but where the distribution of individual
increments is considered interesting.
-->

`ValueRecorder` は正負を問わず任意のグルーピング数を記録するのに便利なグルーピング同期型Instrumentsです。`Recourd(値)`によってキャプチャされた値は、要約されている分布に属する個々のイベントとして扱われます。`ValueRecorder` は、合計に意味を持たない測定値を記録する場合や、本質的には加算であるが個々の増分の分布が興味深いと思われる数値を記録する場合のいずれかに選択されるべきです。

<!--
One of the most common uses for `ValueRecorder` is to capture latency
measurements.  Latency measurements are not adding in the sense that
there is little need to know the latency-sum of all processed
requests.  We use a `ValueRecorder` instrument to capture latency
measurements typically because we are interested in knowing mean,
median, and other summary statistics about individual events.
-->

`ValueRecorder`の最も一般的な用途の一つは、レイテンシの測定値を取得することです。レイテンシ測定は、処理されたすべてのリクエストのレイテンシの合計を知る必要性がほとんどないという意味で、追加ではありません。平均値、中央値、および個々のイベントに関するその他の要約統計を知ることに興味があるので、一般的にはレイテンシの測定値を取得するために `ValueRecorder` Instrumentsを使用します。

<!--
The default aggregation for `ValueRecorder` computes the minimum and
maximum values, the sum of event values, and the count of events,
allowing the rate, the mean, and range of input values to be
monitored.
-->

`ValueRecorder`のデフォルトの集約は、最小値と最大値、イベント値の合計、イベントのカウントを計算し、入力値のレート、平均、範囲を監視することができます。

<!--
Example uses for `ValueRecorder` that are grouping:
-->

`ValueRecorder`のグループ化する使用例:

<!--
- capture any kind of timing information
- capture the acceleration experienced by a pilot
- capture nozzle pressure of a fuel injector
- capture the velocity of a MIDI key-press.
-->

- あらゆる種類のタイミング情報のキャプチャ
- パイロットの加速感をキャプチャ
- 燃料噴射装置のノズル圧のキャプチャ
- MIDIキーを押したときの速度をキャプチャ

<!--
Example _adding_ uses of `ValueRecorder` capture measurements that
are adding, but where we may have an interest in the distribution of
values and not only the sum:
-->

`ValueRecorder`の _加算(adding)_ 用途の例では、加算される測定値をキャプチャしますが、合計だけでなく値の分布にも興味があるかもしれません。

<!--
- capture a request size
- capture an account balance
- capture a queue length
- capture a number of board feet of lumber.
-->

- リクエストサイズのキャプチャ
- アカウント残高のキャプチャ
- 待ち行列の長さをキャプチャ
- 材木のボードフィート数をキャプチャ(訳注:ボードフィート(board-feet)とは、アメリカ合衆国およびカナダで用いられる材木用の体積の単位)

<!--
These examples show that although they are adding in nature,
choosing `ValueRecorder` as opposed to `Counter` or `UpDownCounter`
implies an interest in more than the sum.  If you did not care to
collect information about the distribution, you would have chosen one
of the adding instruments instead.  Using `ValueRecorder` makes
sense for capturing distributions that are likely to be important in
an observability setting.
-->

これらの例を見ると、それらは本質的には追加的なものですが、`Counter`や`UpDownCounter`ではなく`ValueRecorder`を選択することは、合計以上のものへの関心を意味しています。分布についての情報を収集することに関心がなければ、代わりに加算Instrumentsの一つを選んだことでしょう。`ValueRecorder`は、観測可能性の設定において重要であると思われる分布をキャプチャするために使われます。

<!--
Use these with caution because they naturally cost more than the use
of adding measurements.
-->

これらは、測定値を単に加算するよりも当然コストがかかるため、注意して使用してください。

<!--
### SumObserver
-->

### SumObserver

<!--
`SumObserver` is the asynchronous instrument corresponding to
`Counter`, used to capture a monotonic sum with `Observe(sum)`.  "Sum"
appears in the name to remind users that it is used to capture sums
directly.  Use a `SumObserver` to capture any value that starts at
zero and rises throughout the process lifetime and never falls.
-->

`SumObserver`は`Counter`に対応する非同期のInstrumentsで、`Observe(sum)`で単調な和を捉えるのに使用します。名前に "Sum"が含まれているのは、和を直接とらえるために使われることをユーザに思い出させるためです。ゼロから始まり、プロセスのライフタイムを通して上昇し、下降することのない任意の値をキャプチャするには、`SumObserver`を使用してください。

<!--
Example uses for `SumObserver`.
-->

`SumObserver`の使用例:

<!--
- capture process user/system CPU seconds
- capture the number of cache misses.
-->

- プロセスのユーザー/システムのCPU秒数をキャプチャする
- キャッシュミスの数をキャプチャする

<!--
A `SumObserver` is a good choice in situations where a measurement is
expensive to compute, such that it would be wasteful to compute on
every request.  For example, a system call is needed to capture
process CPU usage, therefore it should be done periodically, not on
each request.  A `SumObserver` is also a good choice in situations
where it would be impractical or wasteful to instrument individual
changes that comprise a sum.  For example, even though the number of
cache misses is a sum of individual cache-miss events, it would be too
expensive to synchronously capture each event using a `Counter`.
-->

`SumObserver`は、測定値の計算にコストがかかり、リクエストごとに計算するのは無駄になるような状況では良い選択です。例えば、プロセスのCPU使用率を把握するためにはシステムコールが必要であり、そのためにはリクエストごとではなく、定期的に行う必要があります。合計を構成する個々の変更を計測することが非現実的であったり、無駄であったりするような状況では、`SumObserver`も良い選択となります。例えば、キャッシュミスの数は個々のキャッシュミスイベントの合計であるにもかかわらず、`Counter`を使って各イベントを同期的にキャプチャするのはコストがかかりすぎます。

<!--
### UpDownSumObserver
-->

### UpDownSumObserver

<!--
`UpDownSumObserver` is the asynchronous instrument corresponding to
`UpDownCounter`, used to capture a non-monotonic count with
`Observe(sum)`.  "Sum" appears in the name to remind users that it is
used to capture sums directly.  Use a `UpDownSumObserver` to capture
any value that starts at zero and rises or falls throughout the
process lifetime.
-->

`UpDownSumObserver` は `UpDownCounter` に対応する非同期のInstrumentsで、`Observe(sum)` で非単調なカウントをキャプチャするのに使います。名前に "Sum"が付いているのは、和を直接とらえるために使われることをユーザに思い出させるためです。`UpDownSumObserver`を使って、ゼロから始まってプロセスのライフタイムを通じて上昇または下降するあらゆる値をキャプチャできます。

<!--
Example uses for `UpDownSumObserver`.
-->

`UpDownSumObserver`の使用例:

<!--
  - capture process heap size
  - capture number of active shards
  - capture number of requests started/completed
  - capture current queue size.
-->

- プロセスのヒープサイズのキャプチャ
- アクティブなシャード数のキャプチャ
- 開始/完了したリクエストの数をキャプチャする
- 現在のキューのサイズをキャプチャする

<!--
The same considerations mentioned for choosing `SumObserver` over the
synchronous `Counter` apply for choosing `UpDownSumObserver` over the
synchronous `UpDownCounter`.  If a measurement is expensive to
compute, or if the corresponding changes happen so frequently that it
would be impractical to instrument them, use a `UpDownSumObserver`.
-->

同期型の `Counter` よりも `SumObserver` を選択する際に述べたのと同じ考慮事項が、同期型の `UpDownCounter` よりも `UpDownSumObserver` を選択する際にも適用されます。測定値の計算にコストがかかる場合や、対応する変化があまりにも頻繁に起こり、それらを計測することが非現実的である場合には、`UpDownSumObserver`を使用してください。

<!--
### ValueObserver
-->

### ValueObserver

<!--
`ValueObserver` is the asynchronous instrument corresponding to
`ValueRecorder`, used to capture grouping measurements with
`Observe(value)`.  These instruments are especially useful for
capturing measurements that are expensive to compute, since it gives
the SDK control over how often they are evaluated.
-->

`ValueObserver` は `ValueRecorder` に対応する非同期型Instrumentsで、`Observe(value)` でグループ化された計測値をキャプチャするのに使われます． これらのInstrumentsは、SDKが評価の頻度をコントロールできるため、計算コストの高い計測値をキャプチャする際に特に役立ちます。

<!--
Example uses for `ValueObserver`:
-->

`ValueObserver`の使用例:

<!--
- capture CPU fan speed
- capture CPU temperature.
-->

- CPUファンの回転数をキャプチャ
- CPU温度をキャプチャ

<!--
Note that these examples use grouping measurements.  In the
`ValueRecorder` case above, example uses were given for capturing
synchronous adding measurements during a request (e.g.,
current queue size seen by a request).  In the asynchronous case,
however, how should users decide whether to use `ValueObserver` as
opposed to `UpDownSumObserver`?
-->

これらの例では、グループ化された測定値を使用していることに注意してください。上記の `ValueRecorder` のケースでは、リクエスト中に同期的に追加される測定値(例えば、リクエストによって見られる現在のキューのサイズ)をキャプチャするための使用例が示されました。しかし、非同期の場合では、ユーザは `UpDownSumObserver` と比較して `ValueObserver` を使用するかどうかをどのように決定すればよいのでしょうか？

<!--
Consider how to report the size of a queue asynchronously.  Both
`ValueObserver` and `UpDownSumObserver` logically apply in this case.
Asynchronous instruments capture only one measurement per interval, so
in this example the `UpDownSumObserver` reports a current sum, while the
`ValueObserver` reports a current sum (equal to the max and the min)
and a count equal to 1.  When there is no aggregation, these results
are equivalent.
-->

キューのサイズを非同期に報告する方法を考えてみましょう。この場合、論理的には `ValueObserver` と `UpDownSumObserver` の両方が適用されます。非同期のInstrumentsはインターバルごとに1つの計測値しか取得しませんので、この例では `UpDownSumObserver` は現在の合計を報告し、`ValueObserver` は現在の合計(最大値と最小値に等しい)と1に等しいカウントを報告します。集計が行われていない場合、これらの結果は同等です。

<!--
It may seem pointless to define a default aggregation when there is
exactly one data point.  The default aggregation is specified to apply
when performing spatial aggregation, meaning to combine measurements
across label sets or in a distributed setting.  Although a
`ValueObserver` observes one value per collection interval, the
default aggregation specifies how it will be aggregated with other
values, absent any other configuration.
-->

データポイントが1つだけの場合、デフォルトの集約を定義することは無意味に思えるかもしれません。デフォルトの集約は、空間的な集約を行う際に適用されるように指定されており、ラベルセットや分散型の設定で測定値を組み合わせることを意味します。`ValueObserver`は収集間隔ごとに1つの値を観測しますが、デフォルトの集約は、他の構成がなくても、他の値とどのように集約されるかを指定します。

<!--
Therefore, considering the choice between `ValueObserver` and
`UpDownSumObserver`, the recommendation is to choose the instrument
with the more-appropriate default aggregation.  If you are observing a
queue size across a group of machines and the only thing you want to
know is the aggregate queue size, use `SumObserver` because it
produces a sum, not a distribution.  If you are observing a queue size
across a group of machines and you are interested in knowing the
distribution of queue sizes across those machines, use
`ValueObserver`.
-->

したがって、`ValueObserver` と `UpDownSumObserver` のどちらを選択するかを考えると、より適切なデフォルトの集計方法を持つInstrumentsを選択することをお勧めします。マシングループ全体のキューサイズを観察していて、知りたいのがキューサイズの集計だけである場合には、`SumObserver`を使用してください。なぜなら、これは分布ではなく、合計を生成するからです。もし、マシンのグループ全体のキューサイズを観測していて、それらのマシン全体のキューサイズの分布を知りたい場合には、`ValueObserver` を使用してください。

<!--
### Interpretation
-->

### Interpretation

<!--
How are the instruments fundamentally different, and why are there
only three?  Why not one instrument?  Why not ten?
-->

根本的にInstrumentはどう違うのか、なぜ3つしかないのでしょうか。なぜ1つの楽器ではないのでしょうか？ なぜ10個ではないのでしょうか？

<!--
As we have seen, the instruments are categorized as to whether
they are synchronous, adding, and/or and monotonic.  This approach
gives each of the instruments unique semantics, in ways that
meaningfully improve the performance and interpretation of metric
events.
-->

これまで見てきたように、Instrumentsは同期(Synchronous)、加算(Adding)、単調(Monotonic)のいずれかで分類されています。このアプローチにより、それぞれのInstrumentsに一意な意味合いが与えられ、メトリックイベントのパフォーマンスと解釈が有意義に改善されます。

<!--
Establishing different kinds of instrument is important because in
most cases it allows the SDK to provide good default functionality
"out of the box", without requiring alternative behaviors to be
configured.  The choice of instrument determines not only the meaning
of the events but also the name of the function called by the user.
The function names--`Add()` for adding instruments, `Record()` for
grouping instruments, and `Observe()` for asynchronous
instruments--help convey the meaning of these actions.
-->

さまざまな種類のInstrumentsを設定することが重要なのは、ほとんどの場合、SDKが優れたデフォルト機能を「すぐに」提供し、代替の動作を設定する必要がないためです。Instrumentsの選択は、イベントの意味だけでなく、ユーザーが呼び出す関数の名前も決定します。関数名--Instrumentsの追加には`Add()`、Instrumentsのグループ化には`Record()`、非同期のInstrumentsには`Observe()`--は、これらのアクションの意味を伝えるのに役立ちます．

<!--
The properties and standard implementation described for the
individual instruments is summarized in the table below.
-->

各Instrumentsの特性と標準的な実装方法を以下の表にまとめました。

<!--
| **Name** | Instrument kind | Function(argument) | Default aggregation | Notes |
| ----------------------- | ----- | --------- | ------------- | --- |
| **Counter**             | Synchronous adding monotonic | Add(increment) | Sum | Per-request, part of a monotonic sum |
| **UpDownCounter**       | Synchronous adding | Add(increment) | Sum | Per-request, part of a non-monotonic sum |
| **ValueRecorder**       | Synchronous  | Record(value) | [TBD issue 636](https://github.com/open-telemetry/opentelemetry-specification/issues/636)  | Per-request, any grouping measurement |
| **SumObserver**         | Asynchronous adding monotonic | Observe(sum) | Sum | Per-interval, reporting a monotonic sum |
| **UpDownSumObserver**   | Asynchronous adding | Observe(sum) | Sum | Per-interval, reporting a non-monotonic sum |
| **ValueObserver**       | Asynchronous | Observe(value) | LastValue  | Per-interval, any grouping measurement |
-->

| **名前** | Instrumentの種類 | 関数(引数) | デフォルトの集計 | 注意 |
| ----------------------- | ----- | --------- | ------------- | --- |
| **Counter**             | Synchronous adding monotonic | Add(increment) | Sum | リクエストごと、monotonicの合計の一部 |
| **UpDownCounter**       | Synchronous adding | Add(increment) | Sum | リクエストごと、非monotonicの合計の一部 |
| **ValueRecorder**       | Synchronous  | Record(value) | [TBD issue 636](https://github.com/open-telemetry/opentelemetry-specification/issues/636)  | リクエストごと、任意のグループ化された測定 |
| **SumObserver**         | Asynchronous adding monotonic | Observe(sum) | Sum | インターバルごと、 monotonicの合計 |
| **UpDownSumObserver**   | Asynchronous adding | Observe(sum) | Sum | インターバルごと、非monotonicの合計 |
| **ValueObserver**       | Asynchronous | Observe(value) | LastValue  | インターバルごと、任意のグループ化された測定 |

<!--
### Constructors
-->

### コンストラクタ

<!--
The `Meter` interface supports functions to create new, registered
metric instruments.  Instrument constructors are named by adding a
`New-` prefix to the kind of instrument it constructs, with a
builder pattern, or some other idiomatic approach in the language.
-->

`Meter` インターフェースは、新しく登録されたMeter Instrumentsを作成する関数をサポートしています。Instrumentsのコンストラクタは、構築するInstrumentsの種類に`New-`という接頭辞を付けて、ビルダーパターンやその他の言語の慣用的なアプローチで命名されます．

<!--
There is at least one constructor representing each kind of instrument in
this specification (see [above](#metric-instruments)), and possibly
more as dictated by the language.  For example, if specializations are
provided for integer and floating pointer numbers, the OpenTelemetry
API would support 2 constructors per instrument kind.
-->

この仕様では、Instrumentsの種類ごとに少なくとも1つのコンストラクタが用意されています([上記](#metric-instruments)参照)が、言語によってはそれ以上の数になることもあります。例えば、整数と浮動小数点の数値に対する特殊化が提供されている場合、OpenTelemetry APIはInstrumentsの種類ごとに2つのコンストラクタをサポートすることになります。

<!--
Binding instruments to a single `Meter` instance has two benefits:
-->

Instrumentsを単一の `Meter` インスタンスにバインドすることには2つの利点があります:

<!--
1. Instruments can be exported from the zero state, prior to first use, without an explicit registration call
2. The library-name and version are implicitly associated with the metric event.
-->

1. Instrumentsは、最初に使用する前のゼロの状態から、明示的な登録コールなしでExportできる
2. ライブラリ名とバージョンがメトリックイベントに暗黙的に関連付けられている

<!--
Some existing metric systems support allocating metric instruments
statically and providing the equivalent of a `Meter` interface at the
time of use.  In one example, typical of statsd clients, existing code
may not be structured with a convenient place to store new metric
instruments.  Where this becomes a burden, it is recommended to use
the global `MeterProvider` to construct a static `Meter`, and to
construct and use globally-scoped metric instruments.
-->

既存のメトリックシステムの中には、Metric Instrumentsを静的に割り当て、使用時に `Meter` インターフェースと同等のものを提供することをサポートしているものがあります。statsdクライアントによく見られる例ですが、既存のコードでは、新しいMetric Instrumentsを格納するための便利な場所が構造化されていない場合があります。これが負担になる場合は、グローバルな `MeterProvider` を使用して静的な `Meter` を構築し、グローバルにスコープされた Metric instruments を構築して使用することが推奨されます。

<!--
The situation is similar for users of existing Prometheus clients, where
instruments can be allocated to the global `Registerer`.  
Such code may not have access to an appropriate `MeterProvider` or `Meter`
instance at the location where instruments are defined.
Where this becomes a burden, it is
recommended to use the global meter provider to construct a static
named `Meter`, to construct metric instruments.
-->

既存のPrometheusクライアントでは、グローバルな`Registerer`にInstrumentsを割り当てることができますが、このような状況は同様です。このようなコードでは、Instrumentsが定義されている場所で、適切な `MeterProvider` や `Meter` のインスタンスにアクセスできない可能性があります。これが負担になる場合には、グローバルなMeterProviderを使用して、静的な `Meter` という名前のMetric Instrumentsを構築することが推奨されます。

<!--
Applications are expected to construct long-lived instruments.
Instruments are considered permanent for the lifetime of a SDK, there
is no method to delete them.
-->

アプリケーションは、長期的なInstrumentsを構築することが期待されます。Instrumentsは、SDKの有効期間中は永久的なものとみなされ、削除する方法はありません。

<!--
## Sets of labels
-->

## ラベルセット

<!--
Semantically, a set of labels is a unique mapping from string key to
value.  Across the API, a set of labels MUST be passed in the same,
idiomatic form.  Common representations include an ordered list of
key:values, or a map of key:values.
-->

意味的には、ラベルセットは、文字列キーから値への一意なマッピングです。API全体で、ラベルセットは、同じ、慣用的な形式で渡されなければなりません(MUST)。一般的な表現としては、key:valueの順序付きリスト、またはkey:valueのマップが使われます。

<!--
When labels are passed as an ordered list of key:values, and there are
duplicate keys found, the last value in the list for any given key is
taken in order to form a unique mapping.
-->

ラベルがkey:valuesの順序付きリストとして渡され、その中に重複したキーがあった場合、一意なマッピングを形成するために、任意のキーに対するリストの最後の値が取られます。

<!--
The type of the label value is generally presumed to be a string by
exporters, although as a language-level decision, the label value type
could be any idiomatic type in that language that has a string
representation.
-->

Exporterはラベルの値の型を一般的に文字列と推定しますが、言語レベルの決定として、ラベルの値の型は、その言語で文字列表現が可能な慣用的な型とすることができます。

<!--
Users are not required to pre-declare the set of label keys that will
be used with metric instruments in the API.  Users can freely use any
set of labels for any metric event when calling the API.
-->

ユーザーは、Meteric Insrumentsで使用するラベルキーのセットを、APIで事前に宣言する必要はありません。ユーザーは、APIを呼び出す際に、任意のメトリックイベントに対して任意のラベルセットを自由に使用することができます。

<!--
### Label performance
-->

### ラベルの性能

<!--
Label handling can be a significant cost in the production of metric
data overall.
-->

ラベルの取り扱いは、メトリックデータ全体の構築において大きなコストとなります。

<!--
SDK support for in-process aggregation depends on the ability to find
an active record for an instrument, label set combination pair.  This
allows measurements to be combined.  Label handling costs can be
lowered through the use of bound synchronous instruments and
batch-reporting functions (`RecordBatch`, `BatchObserver`).
-->

In-processな集計のSDKサポートは、Instrumentとラベルセットの組み合わせペアのアクティブなレコードを見つける能力に依存します。これにより、測定値を組み合わせることができます。ラベル処理のコストは、バインドされた同期型Instrumentとバッチレポート機能(`RecordBatch`、`BatchObserver`)を使用することで削減できます。

<!--
### Option: Ordered labels
-->

### Option: 順序付きラベル

<!--
As a language-level decision, APIs MAY support label key ordering.  In this
case, the user may specify an ordered sequence of label keys, which is used to
create an unordered set of labels from a sequence of similarly ordered label
values.  For example:
-->

言語レベルの決定として、APIはラベルキーの順序付けをサポートしても構いません(MAY)。この場合、ユーザーはラベルキーの順序付けされたシーケンスを指定することができます。このシーケンスは、同様に順序付けされたラベル値のシーケンスから順序付けされていないラベルセットを作成するために使用されます。例えば、以下のようになります。

```golang

var rpcLabelKeys = OrderedLabelKeys("a", "b", "c")

for _, input := range stream {
    labels := rpcLabelKeys.Values(1, 2, 3)  // a=1, b=2, c=3

    // ...
}
```

<!--
This is specified as a language-optional feature because its safety, and
therefore its value as an input for monitoring, depends on the availability of
type-checking in the source language.  Passing unordered labels (i.e., a
mapping from keys to values) is considered the safer alternative.
-->

これは、その安全性、つまりモニタリング用の入力としての価値が、ソース言語での型チェックの有無に依存するため、言語の付加的機能として指定されています。順序付けされていないラベル(すなわち、キーから値へのマッピング)を渡すことは、より安全な選択肢と考えられます。

<!--
## Synchronous instrument details
-->

## 同期型Instrumentsの詳細

<!--
The following details are specified for synchronous instruments.
-->

同期型Instrumentsの場合、以下の詳細が規定されています。

<!--
### Synchronous calling conventions
-->

### 同期呼び出しの方式

<!--
The metrics API provides three semantically equivalent ways to capture
measurements using synchronous instruments:
-->

Metrics APIでは、同期型Instrumentsを使用して測定値をキャプチャするために、意味的に同等の3つの方法が用意されています。

<!--
- calling bound instruments, which have a pre-associated set of labels
- directly calling instruments, passing the associated set of labels
- batch recording measurements for multiple instruments using a single set of labels.
-->

- 事前に関連付けられたラベルセットを持っている、バインドされたInstrumentsを呼び出す
- 関連するラベルセットを渡して、Instrumentsを直接呼び出す
- 1つのラベルセットを使って複数のInstrumentsの測定値を一括記録する

<!--
All three methods generate equivalent metric events, but offer varying degrees
of performance and convenience.
-->

この3つの方法は、いずれも同等のメトリックイベントを生成しますが、性能や利便性には差があります。

<!--
The performance of the metric API depends on the work done to enter a
new measurement, which is typically dominated by the cost of handling
labels.  Bound instruments are the highest-performance calling
convention, because they can amortize the cost of handling labels
across many uses.  Recording multiple measurements via
`RecordBatch()`, another calling convention, is a good option for
improving performance, since the cost of handling labels is spread
across multiple measurements.  The direct calling convention is the
most convenient, but least performant calling convention for entering
measurements through the API.
-->

Metric APIの性能は、新しい測定値を入力するために行われる作業に依存しますが、これは通常、ラベルを処理するコストによって支配されます。バインドされたInstrumentsを呼び出す方式は、ラベル処理のコストを多くの呼び出しに渡って償却できるため、最も性能の高い呼び出し規約となっています。別の呼び出し規約である`RecordBatch()`を介して複数の測定値を記録することは、ラベル処理のコストが複数の測定値に分散されるので、パフォーマンスを向上させるための良いオプションです。直接呼び出し規約は、APIを介して測定値を入力するための最も便利な呼び出し規約ですが、パフォーマンスは最も低くなります。

<!--
#### Bound instrument calling convention
-->

#### バインドされたInstrumentsを呼び出す方式

<!--
In situations where performance is a requirement and a metric
instrument is repeatedly used with the same set of labels, the
developer may elect to use the _bound instrument_ calling convention
as an optimization.  For bound instruments to be a benefit, it
requires that a specific instrument will be re-used with specific
labels.  If an instrument will be used with the same labels more than
once, obtaining a bound instrument corresponding to the labels ensures
the highest performance available.
-->

パフォーマンスが要求され、Metric Instrumentsが同じラベルのセットで繰り返し使用される場合、開発者は最適化として_Bound Instrument(バインドされたInstruments)_ 呼び出し規約を使用することができます。Bound Instrumentを有効にするには、特定のInstrumentsが特定のラベルで再利用される必要があります。あるInstrumentsが同じラベルで複数回使用される場合、そのラベルに対応するbound instrumentを取得することで、最高のパフォーマンスが得られます。

<!--
To bind an instrument, use the `Bind(labels...)` method to return an
interface that supports the corresponding synchronous API (i.e.,
`Add()` or `Record()`).  Bound instruments are invoked without labels;
the corresponding metric event is associated with the labels that were
bound to the instrument.
-->

Instrumentsをバインドするには、`Bind(labels...)`メソッドを使用して、対応する同期API(`Add()`または`Record()`)をサポートするインターフェースを返します。Bound Instrumentは、ラベルなしで起動されます。対応するメトリックイベントは、Instrumentsにバインドされたラベルに関連付けられます。

<!--
As a consequence of their performance advantage, bound instruments
also consume resources in the SDK.  Bound instruments MUST support an
`Unbind()` method for users to indicate they are finished with the
binding and release the associated resources.  Note that `Unbind()`
does not imply deletion of a timeseries, it only permits the SDK to
forget the timeseries existed after there are no pending updates.
-->

パフォーマンス上の利点の結果として、Bound Instrumentは、SDKのリソースも消費します。Bound Instrumentは、ユーザーがバインディングの終了を示し、関連するリソースを解放するために、`Unbind()`メソッドをサポートしなければなりません(MUST)。`Unbind()`はタイムスケールの削除を意味するものではなく、保留中の更新がない後にTimeSerisの存在を忘れることをSDKに許可するだけであることに注意してください。

<!--
For example, to repeatedly update a counter with the same labels:
-->

例えば、同じラベルのカウンターを繰り返し更新する場合などです。

```golang
func (s *server) processStream(ctx context.Context) {

  // Bind() の返り値はBound Instrumentsです
  // (例: BoundInt64Counter)
  counter2 := s.instruments.counter2.Bind(
      kv.String("labelA", "..."),
      kv.String("labelB", "..."),
  )
  defer counter2.Unbind()

  for _, item := <-s.channel {
     // ... 他のコード

     // High-performance metric calling convention: Bound Instrumentsを使う
     counter2.Add(ctx, item.size())
  }
}
```

<!--
#### Direct instrument calling convention
-->

#### Instrumentsを直接呼び出す方式

<!--
When convenience is more important than performance, or when values
are not known ahead of time, users may elect to operate directly on
metric instruments, meaning to supply labels at the call site.  This
method offers the greatest convenience possible.
-->

性能よりも利便性を重視する場合や、事前に値が分からない場合には、呼び出し側でラベルを供給する意味で、Metric Instrumentsを直接操作することも可能です。この方法は、可能な限り最大の利便性を提供します。

<!--
For example, to update a single counter:
-->

例えば、1つのカウンターを更新する場合:

```golang
func (s *server) method(ctx context.Context) {
    // ... other work

    s.instruments.counter1.Add(ctx, 1,
        kv.String("labelA", "..."),
        kv.String("labelB", "..."),
        )
}
```

<!--
Direct calls are convenient because they do not require allocating and
storing a bound instrument.  They are appropriate for use in cases
where an instrument will be used rarely, or rarely used with the same
set of labels.  Unlike bound instruments, there is not a long-term
consumption of SDK resources when using the direct calling convention.
-->

直接呼び出し方式は、Bound Instrumentsの割り当てと保存が不要なので便利です。直接呼び出しは、Instrumentsの使用頻度が低い場合や、同じラベルセットを使用する機会が少ない場合に適しています。Bound Instrumentsとは異なり、直接呼び出し方式を使用してもSDKのリソースを長期的に消費することはありません。

<!--
#### RecordBatch calling convention
-->

#### RecordBatch呼び出し方式

<!--
There is one final API for entering measurements, which is like the
direct access calling convention but supports multiple simultaneous
measurements.  The use of the `RecordBatch` API supports entering
multiple measurements, implying a semantically atomic update to
several instruments.  Calls to `RecordBatch` amortize the cost of
label handling across multiple measurements.
-->

測定値を入力するための最後のAPIです。これは直接呼び出し方式のようなものですが、複数の同時測定をサポートしています。`RecordBatch` APIを使用することで、複数の測定値の入力をサポートし、複数のInstrumentsに対する意味的にアトミックな更新を意味します。`RecordBatch`の呼び出しは、複数の測定値に対するラベル処理のコストを償却します。

<!--
For example:
-->

例:


```golang
func (s *server) method(ctx context.Context) {
    // ... other work

    s.meter.RecordBatch(ctx, labels,
        s.instruments.counter.Measurement(1),
        s.instruments.updowncounter.Measurement(10),
        s.instruments.valuerecorder.Measurement(123.45),
    )
}
```

<!--
Another valid interface for recording batches uses a builder pattern:
-->

また、バッチを記録するための有効なインターフェースとして、ビルダーパターンがあります:

```java
    meter.RecordBatch(labels).
        put(s.instruments.counter, 1).
        put(s.instruments.updowncounter, 10).
        put(s.instruments.valuerecorder, 123.45).
        record();
```

<!--
Using the _record batch_ calling convention is semantically identical to
a sequence of direct calls, with the addition of atomicity.  Because
values are entered in a single call, the SDK is potentially able to
implement an atomic update, from the exporter's point of view, because
the SDK can enqueue a single bulk update, or take a lock only once,
for example.  Like the direct calling convention, there is not a
long-term consumption of SDK resources when using the batch calling
convention.
-->

_record batch_の呼び出し方式を使用することは、一連の直接呼び出しと意味的には同じですが、アトミック性が加わります。値は1回の呼び出しで入力されるため、SDKは1回の一括更新をエンキューしたり、ロックを1回だけ取得したりできるため、エクスポーターの観点からはアトミックな更新を実装できる可能性があります。直接呼び出し規約と同様に、バッチ呼び出し規約を使用しても、SDKリソースの長期的な消費はありません。

<!--
### Association with distributed context
-->

### 分散型コンテキストとの関連付け

<!--
Synchronous measurements are implicitly associated with the
distributed [Context](../context/context.md) at runtime, which may
include a Span and Baggage entries. The Metric SDK may use
this information in many ways, but one feature is of particular
interest in OpenTelemetry.
-->

同期測定は、実行時に分散された[Context](../context/context.md)と暗黙のうちに関連付けられます。これにはSpanとBaggageのエントリが含まれることがあります。Metric SDKはこの情報を様々な方法で使用することができますが、OpenTelemetryで特に注目されている機能があります。

<!--
#### Baggage into metric labels
-->

#### MetricのラベルにBaggageを入れる

<!--
Baggage is supported in OpenTelemetry as a means for
labels to propagate from one process to another in a distributed
computation.  Sometimes it is useful to aggregate metric data using
distributed baggage entries as metric labels.
-->

BaggageはOpenTelemetryでサポートされており、分散した計算の中でラベルをあるプロセスから別のプロセスに伝搬させるための手段となっています。分散したBaggageエントリをメトリックラベルとして使用してメトリックデータを集約することが有用な場合があります。

<!--
The use of Baggage must be explicitly configured, using
the [Views API (WIP)](https://github.com/open-telemetry/oteps/pull/89)
to select specific key baggage entries that should be applied as
labels.  The default SDK will not automatically use Baggage
labels in the export pipeline, since using Baggage labels
can be a significant expense.
-->

Baggageの使用は、[Views API (WIP)](https://github.com/open-telemetry/oteps/pull/89)を使って、ラベルとして適用すべき特定のキーのBaggageエントリを選択し、明示的に設定する必要があります。デフォルトのSDKでは、エクスポートパイプラインで自動的にBaggageラベルを使用しません。Baggageラベルの使用には大きなコストがかかるためです。

<!--
Configuring views for applying Baggage labels is a [work in
progress](https://github.com/open-telemetry/oteps/pull/89).
-->

Baggageラベルを適用するためのビューの設定は[work in progress](https://github.com/open-telemetry/oteps/pull/89)です。

<!--
## Asynchronous instrument details
-->

## 非同期型Instrumentsの詳細

<!--
The following details are specified for asynchronous instruments.
-->

非同期型Instrumentsについては、以下の内容が規定されています。

<!--
### Asynchronous calling conventions
-->

### 非同期型呼び出しの方式

<!--
The metrics API provides two semantically equivalent ways to capture
measurements using asynchronous instruments, either through
single-instrument callbacks or through multi-instrument batch
callbacks.
-->

Metric APIは、非同期型Instrumentsを使用して測定値を取得するために、単一Instrumentsのコールバックまたは複数Instrumentsのバッチ・コールバックの2つの意味上同等である方法を提供します。

<!--
Whether single or batch, asynchronous instruments must be observed
through only one callback.  The constructors return no-op instruments
for `null` observer callbacks.  It is considered an error when more
than one callback is specified for any asynchronous instrument.
-->

シングルでもバッチでも、非同期型Instrumentsは1つのコールバックでのみ観測する必要があります。コンストラクタは、`null`のオブザーバーコールバックに対しては、no-opのInstrumentsを返します。非同期型Instrumentsに複数のコールバックが指定されている場合は、エラーになります。

<!--
Instruments may not observe more than one value per distinct label set
per instrument.  When more than one value is observed for a single
instrument and label set, the last observed value is taken and earlier
values are discarded without error.
-->

Instrumentsは、1つのInstrumentsにつき、1つの異なるラベルセットにつき1つ以上の値を観測することはできません。1つのInstrumentsとラベルセットに対して複数の値が観測された場合、最後に観測された値が採用され、それ以前の値はエラーなく破棄されます。

<!--
#### Single-instrument observer
-->

#### 単一Instruments Observer

<!--
A single instrument callback is bound to one instrument.  Its
callback receives an `ObserverResult` with an `Observe(value,
labels...)` function.
-->

シングルInstrumentsコールバックは、1つのInstrumentsにバインドされます． このコールバックは、`Observe(value, labels...)` 関数を持つ `ObserverResult` を受け取ります。

```golang
func (s *server) registerObservers(.Context) {
     s.observer1 = s.meter.NewInt64SumObserver(
         "service_load_factor",
          metric.WithCallback(func(result metric.Float64ObserverResult) {
             for _, listener := range s.listeners {
                 result.Observe(
                     s.loadFactor(),
                     kv.String("name", server.name),
                     kv.String("port", listener.port),
                 )
             }
          }),
          metric.WithDescription("The load factor use for load balancing purposes"),
    )
}
```

<!--
#### Batch observer
-->

#### Batch observer

<!--
A `BatchObserver` callback supports observing multiple instruments in
one callback.  Its callback receives an `BatchObserverResult` with an
`Observe(labels, observations...)` function.
-->

`BatchObserver`コールバックは1つのコールバックで複数のInstrumentsを観測することをサポートします。このコールバックは、`Observe(labels, observations...)` 関数を含む `BatchObserverResult` を受け取ります。

<!--
An observation is returned by calling `Observation(value)`, on an
asynchronous instrument.
-->

観測値は、非同期型Instrumentsで`Observation(value)`を呼び出すことで返されます。


```golang
func (s *server) registerObservers(.Context) {
     batch := s.meter.NewBatchObserver(func (result BatchObserverResult) {
          result.Observe(
             []kv.KeyValue{
                 kv.String("name", server.name),
                 kv.String("port", listener.port),
             },
             s.observer1.Observation(value1),
             s.observer2.Observation(value2),
             s.observer3.Observation(value3),
          },
    )

     s.observer1 = batch.NewSumObserver(...)
     s.observer2 = batch.NewUpDownSumObserver(...)
     s.observer3 = batch.NewValueObserver(...)
}
```

<!--
### Asynchronous observations form a current set
-->

### 現在のセットからの非同期observations

<!--
Asynchronous instrument callbacks are permitted to observe one value
per instrument, per distinct label set, per callback invocation.  The
set of values recorded by one callback invocation represent a current
snapshot of the instrument; it is this set of values that defines the
Last Value for the instrument until the next collection interval.
-->

非同期のInstrumentsコールバックでは、Instrumentsごと、個別のラベルセットごと、コールバックの呼び出しごとに1つの値を観測することができます。1回のコールバック呼び出しで記録された値のセットは、Instrumentsの現在のスナップショットを表します。この値のセットが、次の収集インターバルまでのInstrumentsの最終値を定義します。

<!--
Asynchronous instruments are expected to record an observation for
every label set that it considers "current".  This means that
asynchronous callbacks are expected to observe a value, even when the
value has not changed since the last callback invocation.  To not
observe a label set implies that a value is no longer current.  The
Last Value becomes undefined, as it is no longer current, when it is
not observed during a collection interval.
-->

非同期型Instrumentsは、「現在」と思われるすべてのラベルセットに対して観測値を記録することが期待されます。つまり、非同期コールバックは、最後にコールバックを呼び出してから値が変化していない場合でも、値を観測することが期待されます。ラベルセットを観測しないということは、その値がもはや現在のものではないことを意味します。収集間隔中に観測されなかった場合、最新値はもはや現在のものではないため、定義されなくなります。

<!--
The definition of Last Value is possible for asynchronous instruments,
because their collection is coordinated by the SDK and because they
are expected to report all current values.  Another expression of this
property is that an SDK can keep just one collection interval worth of
observations in memory to lookup the current Last Value of any
instrument and label set.  In this way, asynchronous instruments
support querying current values, independent of the duration of a
collection interval, using data collected at a single point in time.
-->

非同期型Instrumentsの場合、収集はSDKによって調整され、すべての現在値を報告することが期待されているため、「最新値」の定義が可能です。この特性のもう一つの表現は、SDKが1つのコレクション・インターバルに相当するオブザベーションをメモリに保持するだけで、任意のInstrumentsとラベルセットの現在の最新値を検索できることです。このように、非同期型Instruments、1つの時点で収集されたデータを使って、収集間隔の長さに関係なく、現在の値を問い合わせることができます。

<!--
Recall that Last Value is not defined for synchronous instruments, and
it is precisely because there is not a well-defined notion of what is
"current".  To determine the "last-recorded" value for a synchronous
instrument could require inspecting multiple collection windows of
data, because there is no mechanism to ensure that a current value is
recorded during each interval.
-->

同期式Instrumentsでは最新値が定義されていないことを思い出してください。それはまさに、「現在」というものが明確に定義されていないからです。同期式Instrumentsの「最後に記録された」値を決定するには、複数のデータ収集ウィンドウを検査する必要があります。

<!--
#### Asynchronous instruments define moment-in-time ratios
-->

#### 非同期型Instrumentsは瞬間的な比率(moment-in-time ratio)を定義する

<!--
The notion of a current set developed for asynchronous instruments
above can be useful for monitoring ratios.  When the set of observed
values for an instrument add up to a whole, then each observation may
be divided by the sum of observed values from the same interval to
calculate its current relative contribution.  Current relative
contribution is defined in this way, independent of the collection
interval duration, thanks to the properties of asynchronous
instruments.
-->

上記の非同期型Instruments用に開発された現時点のセットの概念は、比率の監視にも役立ちます。あるInstrumentsの観測値のセットが全体になると、各観測値を同じインターバルの観測値の合計で割って、その現在の相対的な貢献度を計算することができます。現在の相対的な貢献度は、非同期型Instrumentsの特性のおかげで、収集間隔の持続時間とは無関係にこのように定義されます。

<!--
## Concurrency
-->

## 並行性

<!--
For languages which support concurrent execution the Metrics APIs provide
specific guarantees and safeties. Not all of API functions are safe to
be called concurrently.
-->

同時実行をサポートする言語では、特定のMetrics APIは保証と安全性を提供します。すべてのAPI関数が同時に呼び出されても安全というわけではありません。

<!--
**MeterProvider** - all methods are safe to be called concurrently.
-->

**MeterProvider** - すべてのメソッドが同時に呼び出されても安全です。

<!--
**Meter** - all methods are safe to be called concurrently.
-->

**Meter** - すべてのメソッドが同時に呼び出されても安全です。

<!--
**Instrument** - All methods of any Instrument are safe to be called concurrently.
-->

**Instrument** - 任意のInstrumentsのすべてのメソッドは、同時に呼び出しても安全です。

<!--
**Bound Instrument** - All methods of any Bound Instrument are safe to be called concurrently.
-->

**Bound Instrument** - 任意のBound Instrumentsのすべてのメソッドは、同時に呼び出しても安全です。

<!--
## Related OpenTelemetry work
-->

## 関連するOpenTelemetry仕様

<!--
Several ongoing efforts are underway as this specification is being
written.
-->

この仕様書の作成中にも、いくつかの継続的な取り組みが行われています。

<!--
### Metric Views
-->

### Metric Views

<!--
The API does not support configurable aggregations for metric
instruments.
-->

このAPIは、Metric Instrumentsに対する設定可能な集約をサポートしていません。

<!--
A _View API_ is defined as an interface to an SDK mechanism that
supports configuring aggregations, including which operator is applied
(sum, p99, last-value, etc.) and which dimensions are used.
-->

_View API_ とは、どの演算子を適用するか(sum、p99、last-valueなど)、どの次元を使用するかなど、集約の設定をサポートするSDKメカニズムへのインターフェースと定義されます。

<!--
See the [current issue discussion on this topic](https://github.com/open-telemetry/opentelemetry-specification/issues/466) and the [current OTEP draft](https://github.com/open-telemetry/oteps/pull/89).
-->

[このトピックに関する現在の問題の議論](https://github.com/open-telemetry/opentelemetry-specification/issues/466)と[現在のOTEPドラフト](https://github.com/open-telemetry/oteps/pull/89)を参照してください。

<!--
### OTLP Metric protocol
-->

### OTLP Metricプロトコル

<!--
The OTLP protocol is designed to export metric data in a memoryless
way, as documented above.  Several details of the protocol are being
worked out.  See the [current protocol](https://github.com/open-telemetry/opentelemetry-proto/blob/master/opentelemetry/proto/metrics/v1/metrics.proto).
-->

OTLPプロトコルは、上述したように、メモリを使わずメトリックデータをエクスポートするように設計されています。プロトコルの詳細については現在検討中で、[現在のドラフト](https://github.com/open-telemetry/opentelemetry-proto/blob/master/opentelemetry/proto/metrics/v1/metrics.proto)をご覧ください。

<!--
### Metric SDK default implementation
-->

### Metric SDKのデフォルト実装

<!--
The OpenTelemetry SDK includes default support for the metric API.  The specification for the default SDK is underway, see the [current draft](https://github.com/open-telemetry/opentelemetry-specification/pull/347).
-->

OpenTelemetry SDKは、Metric APIのデフォルトサポートを含んでいます。デフォルトSDKの仕様は現在進行中で、[現在のドラフト](https://github.com/open-telemetry/opentelemetry-specification/pull/347)をご覧ください。

