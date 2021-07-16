<!--
# Semantic Conventions for Runtime Environment Metrics
-->

# 実行環境メトリックのセマンティック規約

**Status**: [Experimental](../../document-status.md)

<!--
This document includes semantic conventions for runtime environment level
metrics in OpenTelemetry. Also consider the [general
metric](README.md#general-metric-semantic-conventions), [system
metrics](system-metrics.md) and [OS Process metrics](process-metrics.md)
semantic conventions when instrumenting runtime environments.
-->

このドキュメントには、OpenTelemetryにおけるランタイム環境レベルのメトリックのセマンティック規約が含まれています。また、ランタイム環境を計装する際には、[一般的なメトリック](README.md#general-metric-semantic-conventions)、[システム・メトリック](system-metric.md)、[OSプロセス・メトリック](process-metric.md)のセマンティック規約も考慮してください。

<!-- Re-generate TOC with `markdown-toc --no-first-h1 -i` -->

<!-- toc -->

<!--
- [Metric Instruments](#metric-instruments)
  * [Runtime Environment Specific Metrics - `runtime.{environment}.`](#runtime-environment-specific-metrics---runtimeenvironment)
-->

- [Metric計装](#Metric計装)
  * [ランタイム環境固有のMetrics - `runtime.{environment}.`](#ランタイム環境固有のMetrics---runtimeenvironment)

<!-- tocstop -->

<!--
## Metric Instruments
-->

## Metric計装

<!--
Runtime environments vary widely in their terminology, implementation, and
relative values for a given metric. For example, Go and Python are both
garbage collected languages, but comparing heap usage between the Go and
CPython runtimes directly is not meaningful. For this reason, this document
does not propose any standard top-level runtime metric instruments. See [OTEP
108](https://github.com/open-telemetry/oteps/pull/108/files) for additional
discussion.
-->

ランタイム環境は、その用語、実装、および特定の指標に対する相対的な値が大きく異なります。例えば、GoとPythonはどちらもガベージコレクション言語ですが、GoランタイムとCPythonランタイムの間でヒープ使用量を直接比較することは意味がありません。このような理由から、本ドキュメントでは、標準的なトップレベルのランタイムメトリック機器を提案しません。追加の議論については[OTEP 108](https://github.com/open-telemetry/oteps/pull/108/files)を参照してください。

<!--
### Runtime Environment Specific Metrics - `runtime.{environment}.`
-->

### ランタイム環境固有のMetrics - `runtime.{environment}.`

<!--
Metrics specific to a certain runtime environment should be prefixed with
`runtime.{environment}.` and follow the semantic conventions outlined in
[general metric semantic
conventions](README.md#general-metric-semantic-conventions). Authors of
runtime instrumentations are responsible for the choice of `{environment}` to
avoid ambiguity when interpreting a metric's name or values.
-->

特定のランタイム環境に特化したメトリックは、プレフィックスとして `runtime.{environment}.` を付け、[一般的なMetricセマンティック規約](README.md#general-metric-semantic-conventions)に概説されているセマンティック規約に従います。ランタイム計装の作者は、メトリックの名前や値を解釈する際の曖昧さを避けるために、`{environment}`の選択に責任を負います。

<!--
For example, some programming languages have multiple runtime environments
that vary significantly in their implementation, like [Python which has many
implementations](https://wiki.python.org/moin/PythonImplementations). For
such languages, consider using specific `{environment}` prefixes to avoid
ambiguity, like `runtime.cpython.` and `runtime.pypy.`.
-->

例えば、プログラミング言語の中には、[多くの実装を持つPython](https://wiki.python.org/moin/PythonImplementations)のように、実装が大きく異なる複数の実行環境を持つものがあります。このような言語では、`runtime.cpython.`や`runtime.pypy.`のように、曖昧さを避けるために特定の`{environment}`接頭辞を使用することを検討してください。

<!--
There are other dimensions even within a given runtime environment to
consider, for example pthreads vs green thread implementations.
-->

例えば、pthreadとグリーンスレッドの実装など、あるランタイム環境の中でも別の次元を考慮する必要があります。

