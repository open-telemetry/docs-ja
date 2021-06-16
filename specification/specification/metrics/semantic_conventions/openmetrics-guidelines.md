<!--
# Guidance for Interoperating with OpenMetrics
-->

# OpenMetricsとの相互運用のためのガイダンス

**Status**: [Experimental](../../document-status.md)

<!--
**Note:** This document is work in progress and will be updated as the
OpenMetrics specification further develops.
-->

**このドキュメントは現在進行中であり、OpenMetricsの仕様がさらに発展するにつれて更新される予定です。

<!--
## Overview
-->

## 概要

<!--
OpenTelemetry will need to interoperate with systems using the OpenMetrics or
Prometheus exposition format in two ways:
-->

OpenTelemetryは、2つの方法でOpenMetricsまたはPrometheusエクスポートフォーマットを使用するシステムと相互運用する必要があります。

<!--
* OpenTelemetry may need to accept and propagate metrics expressed in
  the OpenMetrics exposition format, and export them to downstream systems
  (including OpenTelemetry Collector(s), zPages, vendor backends, etc...)
* OpenTelemetry may need to expose OpenTelemetry generated metrics in the
  OpenMetrics exposition format.
-->

* OpenTelemetryは、OpenMetrics形式で表現されたメトリックを受け入れて伝播させ、下流のシステム(OpenTelemetry Collector、zPages、ベンダーのバックエンドなど)にエクスポートする必要があるかもしれません。
* OpenTelemetryでは、OpenTelemetryで生成したメトリックをOpenMetricsの公開フォーマットで公開する必要があるかもしれません。

<!--
### OpenMetrics to OpenTelemetry
-->

### OpenMetrics から OpenTelemetry へ

<!--
The OpenTelemetry collector implements a Prometheus receiver, which reads
metrics in the OpenMetrics exposition format. For more information, refer to the
[Prometheus Receiver Design Document](https://github.com/open-telemetry/opentelemetry-collector/blob/master/receiver/prometheusreceiver/DESIGN.md).
-->

OpenTelemetryコレクターは、Prometheusレシーバーを実装しており、OpenMetricsエクスポートフォーマットのメトリックを読み取ります。詳細については、[Prometheus Receiver Design Document](https://github.com/open-telemetry/opentelemetry-collector/blob/master/receiver/prometheusreceiver/DESIGN.md)を参照してください。

<!--
### OpenTelemetry to OpenMetrics
-->

### OpenTelemetry から OpenMetrics へ

<!--
#### Name and Label Keys
-->

#### Name と Label キー

<!--
Exposting OpenTelemetry metrics in the OpenMetrics format is primarily
problematic for metric and label naming; the OpenMetrics exposition format
expressly forbids some characters that are allowed in OpenTelemetry.
-->

OpenTelemetryのメトリックをOpenMetricsフォーマットで公開することは、主にメトリックとラベルの命名に関して問題があります。OpenMetricsの公開フォーマットでは、OpenTelemetryで許可されているいくつかの文字が明示的に禁止されています。

<!--
When converting OpenTelemetry metric events to the OpenMetrics exposition
format, the name field and all label keys MUST be sanitized by replacing
every character that is not a letter or a digit with an underscore.
-->

OpenTelemetryのメトリックイベントをOpenMetricsの公開フォーマットに変換する際には、nameフィールドおよびすべてのラベルキーを、文字または数字ではないすべての文字をアンダースコアに置き換えることでサニタイズしなければなりません(MUST)。

<!--
Example pseudocode:
-->

疑似コードの例:

<!--
```ruby
def sanitize(name)
    return name.sub(/\W/, '_')
```
-->

```ruby
def sanitize(name)
    return name.sub(/\W/, '_')
```

<!--
See also [Metric names and labels](https://prometheus.io/docs/concepts/data_model/#metric-names-and-labels)
in the Prometheus data model documentation.
-->

Prometheusデータモデル・ドキュメントの[Metric名とラベル](https://prometheus.io/docs/concepts/data_model/#metric-names-and-labels)も参照してください。

<!--
OpenMetrics does not allow metric names to begin with a digit. OpenTelemetry's
[instrument naming requirements](../api.md#instrument-naming-requirements) also
require that the first character of a metric instrument is non-numeric.
-->

OpenMetricsでは、メトリック名が数字で始まることを認めていません。OpenTelemetryの[instrument naming requirements](../api.md#instrument-naming-requirements)でも、Metric instrumentsの最初の文字が数字でないことを要求しています。

<!--
If a metric event is generated in OpenTelemetry that does not conform to this
specification, the name of the resulting OpenMetrics metric MAY be prepended
with an underscore.
-->

本仕様に準拠しないメトリックイベントがOpenTelemetryで生成された場合、結果として得られるOpenMetricsメトリックの名前の前にアンダースコアを付けても構いません(MAY)。

