<!--
# Performance Benchmark of OpenTelemetry API
-->

# OpenTelemetry APIのパフォーマンス・ベンチマーク

<!--
This document describes common performance benchmark guidelines on how to
measure and report the performance of OpenTelemetry SDKs.
-->

このドキュメントでは、OpenTelemetry SDKのパフォーマンスをどのように測定・報告するかについて、一般的なパフォーマンスベンチマークのガイドラインを説明しています。

<!--
The goal of this benchmark is to provide a tool to get the basic performance
overhead of the OpenTelemetry SDK for given events throughput on the target
platform.
-->

このベンチマークの目的は、ターゲットプラットフォーム上で与えられたイベントのスループットに対するOpenTelemetry SDKの基本的なパフォーマンスオーバーヘッドを得るためのツールを提供することです。

<!--
## Benchmark Configuration
-->

## ベンチマークの設定

<!--
### Span Configuration
-->

### Span設定

<!--
- No parent `Span` or parent `SpanContext`.
- Default Span [Kind](./trace/api.md#spankind) and
  [Status](./trace/api.md#set-status).
- Associated to a [resource](overview.md#resources) with attributes
  `service.name`, `service.version` and 10 characters string value for each
  attribute, and attribute `service.instance.id` with a unique UUID. See
  [Service](./resource/semantic_conventions/README.md#service) for details.
- 1 [attribute](./common/common.md#attributes) with a signed 64-bit integer
  value.
- 1 [event](./trace/api.md#add-events) without any attributes.
- The `AlwaysOn` sampler should be enabled.
- Each `Span` is created and immediately ended.
-->

- 親の `Span` や親の `SpanContext` は含まないこと。
- デフォルトのSpanは[Kind](./trace/api.md#spankind)と[Status](./trace/api.md#set-status)とすること。
- 属性 `service.name`, `service.version` と各属性に10文字の文字列値、属性 `service.instance.id` にはユニークなUUIDが設定されている [resource](overview.md#resources) に関連付けること。詳細は[Service](./resource/semantic_conventions/README.md#service)を参照してください。
- 1 [attribute](./common/common.md#attributes)には、64ビットの符号付き整数値を指定すること。
- 1 [event](./trace/api.md#add-events)には属性を含まないこと。
- Samplerの`AlwaysOn`を有効にすること。
- 各 `Span` は作成され、すぐに終了すること。

<!--
### Measurement Configuration
-->

### Measurement 設定

<!--
For the languages with bootstrap cost like JIT compilation, a warm-up phase is
recommended to take place before the measurement, which runs under the same
`Span` [configuration](#span-configuration).
-->

JITコンパイルのようなブートストラップコストを持つ言語では、測定の前に、同じ`Span`[設定](#span設定)の下で実行されるウォームアップフェーズを行うことが推奨されています。

<!--
## Throughput Measurement
-->

## スループット測定

<!--
### Create Spans
-->

### Spanの作成

<!--
Number of spans which could be created and exported via OTLP exporter in 1
second per logical core and average number over all logical cores, with each
span containing 10 attributes, and each attribute containing two 20 characters
strings, one as attribute name the other as value.
-->

1秒でOTLP Exporterを使って作成・エクスポートできたSpanの数を、論理コアごとと全論理コアの平均数で表した数値。各Spanには10個の属性が含まれ、各属性には2つの20文字の文字列(1つは属性名、もう1つは値)を含めます。

<!--
## Instrumentation Cost
-->

## 計装コスト

<!--
### CPU Usage Measurement
-->

### CPU利用率測定

<!--
With given number of span throughput specified by user, or 10,000 spans per
second as default if user does not input the number, measure and report the CPU
usage for SDK with both default configured simple and batching span processors
together with OTLP exporter. The benchmark should create an out-of-process OTLP
receiver which listens on the exporting target or adopts existing OTLP exporter
which runs out-of-process, responds with success status immediately and drops
the data. The collector should not add significant CPU overhead to the
measurement. Because the benchmark does not include user processing logic, the
total CPU consumption of benchmark program could be considered as approximation
of SDK's CPU consumption.
-->

ユーザーによって指定された任意のSpan・スループット数、またはユーザーが数を入力しない場合はデフォルトとして毎秒10,000のSpanを使用して、OTLP Exporterと共にデフォルトで構成されたシンプルなSpan・プロセッサーとバッチ・Span・プロセッサーの両方を備えたSDKのCPU使用率を測定し、報告します。ベンチマークは、エクスポートするターゲット上でListenするout-of-processのOTLPレシーバーを作成するか、out-of-processで実行され、直ちに成功ステータスで応答し、データをドロップする既存のOTLPエクスポーターを採用する必要があります。コレクタは、測定に大きなCPUオーバーヘッドを加えるべきではありません。ベンチマークにはユーザーの処理ロジックが含まれていないため、ベンチマークプログラムの総CPU消費量は、SDKのCPU消費量の近似値とみなすことができます。

<!--
The total running time for one test iteration is suggested to be at least 15
seconds. The average and peak CPU usage should be reported.
-->

1回のテストの繰り返しにかかる総実行時間は、少なくとも15秒であることが推奨されます。また、平均およびピーク時のCPU使用率を報告する必要があります。

<!--
### Memory Usage Measurement
-->

### メモリ利用率測定

<!--
Measure dynamic memory consumption, e.g. heap, for the same scenario as above
CPU Usage section with 15 seconds duration.
-->

上記の「CPU使用率」と同じシナリオで、15秒間の動的メモリ消費量(ヒープなど)を測定します。

<!--
## Report
-->

## 報告

<!--
### Report Format
-->

### 報告形式

<!--
All the numbers above should be measured multiple times (suggest 10 times at
least) and reported.
-->

上記の数値はすべて複数回(最低でも10回を推奨)測定し、報告する必要があります。
