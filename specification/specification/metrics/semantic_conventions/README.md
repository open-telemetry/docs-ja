<!--
# Metrics Semantic Conventions
-->

# Metrics セマンティック規約

**Status**: [Experimental](../../document-status.md)

<!-- Re-generate TOC with `markdown-toc --no-first-h1 -i` -->

<!-- toc -->

<!--
- [General Guidelines](#general-guidelines)
  * [Units](#units)
  * [Pluralization](#pluralization)
- [General Metric Semantic Conventions](#general-metric-semantic-conventions)
  * [Instrument Naming](#instrument-naming)
  * [Instrument Units](#instrument-units)
-->

- [General Guidelines](#general-guidelines)
  * [Units](#units)
  * [Pluralization](#pluralization)
- [General Metric Semantic Conventions](#general-metric-semantic-conventions)
  * [Instrument Naming](#instrument-naming)
  * [Instrument Units](#instrument-units)


<!-- tocstop -->

<!--
The following semantic conventions surrounding metrics are defined:
-->

メトリックにまつわる以下のセマンティック規約が定義されています。

<!--
* [HTTP Metrics](http-metrics.md): Semantic conventions and instruments for HTTP metrics.
* [System Metrics](system-metrics.md): Semantic conventions and instruments for standard system metrics.
* [Process Metrics](process-metrics.md): Semantic conventions and instruments for standard process metrics.
* [Runtime Environment Metrics](runtime-environment-metrics.md): Semantic conventions and instruments for runtime environment metrics.
-->

* [HTTP Metrics](http-metrics.md): HTTPメトリックのためのセマンティック規約と手段
* [System Metrics](system-metrics.md): 標準的なシステムメトリックのためのセマンティック規約と手段
* [Process Metrics](process-metrics.md): 標準的なプロセスメトリックのためのセマンティック規約と手段
* [Runtime Environment Metrics](runtime-environment-metrics.md): ランタイム環境メトリックのためのセマンティック規約と手段

<!--
Apart from semantic conventions for metrics and
[traces](../../trace/semantic_conventions/README.md), OpenTelemetry also
defines the concept of overarching [Resources](../../resource/sdk.md) with
their own [Resource Semantic
Conventions](../../resource/semantic_conventions/README.md).
-->

メトリックや[Traceのセマンティック規約](../../trace/semantic_conventions/README.md)とは別に、OpenTelemetryは独自の[Resource](../../resource/sdk.md)を持つ包括的な[Resourcesセマンティック規約](../../resource/semantic_conventions/README.md)の概念も定義しています。

<!--
## General Guidelines
-->

## 一般的なガイドライン

<!--
Metric names and labels exist within a single universe and a single
hierarchy. Metric names and labels MUST be considered within the universe of
all existing metric names. When defining new metric names and labels,
consider the prior art of existing standard metrics and metrics from
frameworks/libraries.
-->

メトリック名およびラベルは、単一のユニバースおよび単一の階層内に存在します。メトリック名とラベルは、既存のすべてのメトリック名の世界の中で考えなければなりません(MUST)。新しいメトリック名やラベルを定義する際には、既存の標準的なメトリックや、フレームワークやライブラリのメトリックなどの先行技術を考慮してください。

<!--
Associated metrics SHOULD be nested together in a hierarchy based on their
usage. Define a top-level hierarchy for common metric categories: for OS
metrics, like CPU and network; for app runtimes, like GC internals. Libraries
and frameworks should nest their metrics into a hierarchy as well. This aids
in discovery and adhoc comparison. This allows a user to find similar metrics
given a certain metric.
-->

関連づけられたMetricは、その使用方法に基づいて、階層的にネストされるべきです。CPUやネットワークなどのOSメトリック、GCインターナルなどのアプリのランタイムなど、共通のメトリックカテゴリのトップレベルの階層を定義します。ライブラリやフレームワークも同様に、メトリックを階層化する必要があります。これは発見とアドホックな比較に役に立ちます。これにより、ユーザーはあるメトリックが与えられたときに、類似したメトリックを見つけることができます。

<!--
The hierarchical structure of metrics defines the namespacing. Supporting
OpenTelemetry artifacts define the metric structures and hierarchies for some
categories of metrics, and these can assist decisions when creating future
metrics.
-->

メトリックの階層構造が名前空間を定義します。サポートするOpenTelemetryアーティファクトは、いくつかのカテゴリのメトリックの構造と階層を定義しており、これらは将来のメトリックを作成する際の判断材料となります。

<!--
Common labels SHOULD be consistently named. This aids in discoverability and
disambiguates similar labels to metric names.
-->

共通のラベルには一貫した名前を付けるべきです(SHOULD)。これは発見性を助け、類似したラベルのメトリック名から曖昧さを取り除きます。

<!--
["As a rule of thumb, **aggregations** over all the dimensions of a given
metric **SHOULD** be
meaningful,"](https://prometheus.io/docs/practices/naming/#metric-names) as
Prometheus recommends.
-->

Prometheusが推奨するように["経験則から言うと、ある指標のすべての次元での**集計**は**意味のあるものでなければなりません**](https://prometheus.io/docs/practices/naming/#metric-names)。

<!--
Semantic ambiguity SHOULD be avoided. Use prefixed metric names in cases
where similar metrics have significantly different implementations across the
breadth of all existing metrics. For example, every garbage collected runtime
has slightly different strategies and measures. Using a single set of metric
names for GC, not divided by the runtime, could create dissimilar comparisons
and confusion for end users. (For example, prefer `runtime.java.gc*` over
`runtime.gc.*`.) Measures of many operating system metrics are similarly
ambiguous.
-->

意味的な曖昧さは避けるべきです(SHOULD)。既存のすべてのメトリックの中で、類似したメトリックの実装が大きく異なる場合には、接頭辞付きのメトリック名を使用します。たとえば、すべてのガベージコレクションのランタイムは、わずかに異なる戦略と対策を持っています。ランタイムによって分けられていない、GCのための単一のメトリック名のセットを使用すると、異質な比較を生み出し、エンドユーザを混乱させる可能性があります。(例えば、`runtime.gc.*`よりも`runtime.java.gc*`の方が良いでしょう) 多くのオペレーティングシステムメトリックの尺度も同様に曖昧です。

<!--
### Units
-->

### 単位

<!--
Conventional metrics or metrics that have their units included in
OpenTelemetry metadata (e.g. `metric.WithUnit` in Go) SHOULD NOT include the
units in the metric name. Units may be included when it provides additional
meaning to the metric name. Metrics MUST, above all, be understandable and
usable.
-->

従来のメトリックや、OpenTelemetryのメタデータに単位が含まれているメトリック(例:Goの`metric.WithUnit`)は、メトリック名に単位を含めるべきではありません(SHOULD NOT)。単位が含まれるのは、それがメトリック名に追加の意味を与える場合です。メトリックは何よりも、理解できて使いやすいものでなければなりません(MUST)。

<!--
When building components that interoperate between OpenTelemetry and a system
using the OpenMetrics exposition format, use the
[OpenMetrics Guidelines](./openmetrics-guidelines.md).
-->

OpenTelemetryとOpenMetricsのexport フォーマットを使用するシステムとの間で相互運用するコンポーネントを構築する場合は、[OpenMetricsガイドライン](./openmetrics-guidelines.md)を使用してください。

<!--
### Pluralization
-->

### Pluralization(複数名化)

<!--
Metric names SHOULD NOT be pluralized, unless the value being recorded
represents discrete instances of a
[countable quantity](https://en.wikipedia.org/wiki/Count_noun).
Generally, the name SHOULD be pluralized only if the unit of the metric in
question is a non-unit (like `{faults}` or `{operations}`).
-->

記録される値が[数えられる量](https://en.wikipedia.org/wiki/Count_noun)の離散的なインスタンスを表す場合を除き、メトリック名は複数化されるべきではありません(SHOULD NOT)。一般的には、問題となっているメトリックの単位が(`{faults}`や`{operations}`のように)非単位である場合にのみ、名前を複数形にすべきです(SHOULD)。

<!--
Examples:
-->

例:

<!--
* `system.filesystem.utilization`, `http.server.duration`, and `system.cpu.time`
should not be pluralized, even if many data points are recorded.
* `system.paging.faults`, `system.disk.operations`, and `system.network.packets`
should be pluralized, even if only a single data point is recorded.
-->

* `system.filesystem.utilization`, `http.server.duration`, `system.cpu.time` は、たとえ多くのデータポイントが記録されていても、複数形にすべきではありません。
* `system.paging.faults`, `system.disk.operations`, `system.network.packets` は、たとえデータポイントがひとつしか記録されていなくても、複数形にすべきです。

<!--
## General Metric Semantic Conventions
-->

## 一般的なメトリックのセマンティック規約

<!--
The following semantic conventions aim to keep naming consistent. They
provide guidelines for most of the cases in this specification and should be
followed for other instruments not explicitly defined in this document.
-->

以下のセマンティック規約は、ネーミングの一貫性を保つことを目的としています。これは、本ドキュメントのほとんどのケースに対するガイドラインであり、本ドキュメントで明示的に定義されていないその他の機器についても従うべきものです。

<!--
### Instrument Naming
-->

### Instrumentの名前付け

<!--
- **limit** - an instrument that measures the constant, known total amount of
something should be called `entity.limit`. For example, `system.memory.limit`
for the total amount of memory on a system.
-->

- **limit** - あるものの一定の既知の総量を測定するInstrumentは、`entity.limit`と呼ぶべきです。例えば、システム上のメモリの総量は `system.memory.limit` とします。

<!--
- **usage** - an instrument that measures an amount used out of a known total
(**limit**) amount should be called `entity.usage`. For example,
`system.memory.usage` with label `state = used | cached | free | ...` for the
amount of memory in a each state. Where appropriate, the sum of **usage**
over all label values SHOULD be equal to the **limit**.
  A measure of the amount of an unlimited resource consumed is differentiated
  from **usage**.
-->

- **usage** - 既知の総量(**limit**)から使用された量を測定するInstrumentは、`entity.usage`と呼ぶべきです。例えば、各状態のメモリの量については、`system.memory.usage`に`state = used | cached | free | ...`というラベルを付けます。適切な場合には、すべてのラベル値の**usage**の合計は**limit**に等しくあるべきです。
  無限にある資源の消費量を示す指標は、**usage**とは区別されます。

<!--
- **utilization** - an instrument that measures the *fraction* of **usage**
out of its **limit** should be called `entity.utilization`. For example,
`system.memory.utilization` for the fraction of memory in use. Utilization
values are in the range `[0, 1]`.
-->

- **utilization** - **limit**を超えた**usage**の*fraction(割合)*を測定するInstrumentは、`entity.utilization`と呼ぶべきです。例えば、使用中のメモリの割合を表すには `system.memory.utilization` とします。利用率の値は `[0, 1]` の範囲になります。

<!--
- **time** - an instrument that measures passage of time should be called
`entity.time`. For example, `system.cpu.time` with label `state = idle | user
| system | ...`. **time** measurements are not necessarily wall time and can
be less than or greater than the real wall time between measurements.
  **time** instruments are a special case of **usage** metrics, where the
  **limit** can usually be calculated as the sum of **time** over all label
  values. **utilization** for time instruments can be derived automatically
  using metric event timestamps. For example, `system.cpu.utilization` is
  defined as the difference in `system.cpu.time` measurements divided by the
  elapsed time.
-->

- **time** - 時間の経過を測定するInstrumentは、`entity.time`と呼ばれるべきです。例えば、`system.cpu.time`には、`state = idle | user | system | ...`というラベルが付けられています。**time**の測定値は必ずしもwall clockではなく、測定間の実際のwall clockよりも小さくても大きくても構いません。
  **time**Instrumentは**usage**メトリックの特殊なケースであり、**limit**は通常、すべてのラベル値の**time**の合計として計算できます。例えば、`system.cpu.utilization`は、`system.cpu.time`の測定値の差を経過時間で割ったものと定義されます。

<!--
- **io** - an instrument that measures bidirectional data flow should be
called `entity.io` and have labels for direction. For example,
`system.network.io`.
-->

- **io** - 双方向のデータフローを測定するInstrumentは、 `entity.io` と呼ばれ、方向を示すラベルを持つべきです。例えば、`system.network.io`とします(XXX: direction書いてないのでは？)。

<!--
- Other instruments that do not fit the above descriptions may be named more
freely. For example, `system.paging.faults` and `system.network.packets`.
Units do not need to be specified in the names since they are included during
instrument creation, but can be added if there is ambiguity.
-->

- 上記の説明に当てはまらない他のInstrumentは、もっと自由に名前を付けても構いません。例えば、`system.paging.faults`や`system.network.packets`などです。単位はInstrumentの作成時に含まれるので、名前に指定する必要はありませんが、曖昧さがある場合は追加することができます。

<!--
### Instrument Units
-->

### Instrumentの単位

<!--
Units should follow the [UCUM](http://unitsofmeasure.org/ucum.html) (need
more clarification in
[#705](https://github.com/open-telemetry/opentelemetry-specification/issues/705)).
-->

単位は[UCUM](http://unitsofmeasure.org/ucum.html)に従うべきです([#705](https://github.com/open-telemetry/opentelemetry-specification/issues/705)でより明確にする必要があります)。

<!--
- Instruments for **utilization** metrics (that measure the fraction out of a
total) are dimensionless and SHOULD use the default unit `1` (the unity).
- Instruments that measure an integer count of something SHOULD use the
default unit `1` (the unity) and
[annotations](https://ucum.org/ucum.html#para-curly) with curly braces to
give additional meaning. For example `{packets}`, `{errors}`, `{faults}`,
etc.
-->

- **utilization**メトリック(全体の中での割合を測るもの)は無次元であり、デフォルトの単位である`1`(the unity)を使用すべきです。

- 何かの整数カウントを測定するInstrumentは、デフォルトの単位である`1`(the unity)と、追加の意味を与えるために中括弧付きの[annotations](https://ucum.org/ucum.html#para-curly)を使用すべきです。例えば、`{packets}`、`{errors}`、`{faults}`などです。

