<!--
# OpenTelemetry to Jaeger Transformation
-->

# OpenTelemetry から Jaeger への変換

<!--
**Status**: [Stable](../../document-status.md)
-->

**Status**: [Stable](../../document-status.md)

<!--
This document defines the transformation between OpenTelemetry and Jaeger Spans.
The generic transformation [rules specified here](non-otlp.md) also apply. If a
particular generic transformation rule and the rule in this document contradict
then the rule in this document MUST be used.
-->

このドキュメントでは、OpenTelemetryとJaeger Spansの間の変換を定義しています。Jaegerは2つの形式のSpanを受け入れます。

このドキュメントでは、OpenTelemetryとJaeger Spanの間の変換を定義しています。一般的な[ここで指定された変換ルール](non-otlp.md)も適用されます。特定の汎用変換ルールと本文書のルールが矛盾する場合は、本ドキュメントのルールを使用しなければなりません(MUST)。

<!--
* Thrift `Batch`, defined in [jaeger-idl/.../jaeger.thrift](https://github.com/jaegertracing/jaeger-idl/blob/master/thrift/jaeger.thrift), accepted via UDP or HTTP
* Protobuf `Batch`, defined in [jaeger-idl/.../model.proto](https://github.com/jaegertracing/jaeger-idl/blob/master/proto/api_v2/model.proto), accepted via gRPC
-->

* Thrift `Batch`, [jaeger-idl/.../jaeger.thrift](https://github.com/jaegertracing/jaeger-idl/blob/master/thrift/jaeger.thrift) で定義されており、UDP または HTTP で受け付けられます。
* Protobuf `Batch`, [jaeger-idl/.../model.proto](https://github.com/jaegertracing/jaeger-idl/blob/master/proto/api_v2/model.proto) で定義されており、gRPC で受け付けられます。

<!--
See also:
-->

参照:

<!--
* [Jaeger APIs](https://www.jaegertracing.io/docs/latest/apis/)
* [Reference implementation of this translation in the OpenTelemetry Collector](https://github.com/open-telemetry/opentelemetry-collector/blob/master/translator/trace/jaeger/traces_to_jaegerproto.go)
-->

* [Jaeger APIs](https://www.jaegertracing.io/docs/latest/apis/)
* [OpenTelemetry Collectorにおけるこの翻訳のリファレンス実装](https://github.com/open-telemetry/opentelemetry-collector/blob/master/translator/trace/jaeger/traces_to_jaegerproto.go)

<!--
## Summary
-->

## 要約

<!--
The following table summarizes the major transformations between OpenTelemetry
and Jaeger.
-->

以下の表は、OpenTelemetryとJaegerの間の主な変換点をまとめたものです。

<!--
| OpenTelemetry            | Jaeger Thrift    | Jaeger Proto     | Notes |
| ------------------------ | ---------------- | ---------------- | ----- |
| Span.TraceId             | Span.traceIdLow/High | Span.trace_id | See [IDs](#ids)     |
| Span.ParentId            | Span.parentSpanId | as SpanReference | See [Parent ID](#parent-id)     |
| Span.SpanId              | Span.spanId       | Span.span_id     |      |
| Span.TraceState          | TBD               | TBD              |      |
| Span.Name                | Span.operationName | Span.operation_name |  |
| Span.Kind                | Span.tags["span.kind"] | same | See [SpanKind](#spankind) for values mapping |
| Span.StartTime           | Span.startTime | Span.start_time | See [Unit of time](#unit-of-time) |
| Span.EndTime             | Span.duration | same | Calculated as EndTime - StartTime. See also [Unit of time](#unit-of-time) |
| Span.Attributes          | Span.tags | same | See [Attributes](#attributes) for data types for the mapping.            |
| Span.DroppedAttributesCount| Add to Span.tags | same | See [Dropped Attributes Count](non-otlp.md#dropped-attributes-count) for tag name to use. |
| Span.Events              | Span.logs | same | See [Events](#events) for the mapping format. |
| Span.Links               | Span.references | same | See [Links](#links) |
| Span.Status              | Add to Span.tags | same | See [Status](#status) for tag names to use. |
-->

| OpenTelemetry            | Jaeger Thrift    | Jaeger Proto     | Notes |
| ------------------------ | ---------------- | ---------------- | ----- |
| Span.TraceId             | Span.traceIdLow/High | Span.trace_id | [ID](#id) 参照    |
| Span.ParentId            | Span.parentSpanId | as SpanReference | [Parent ID](#parent-id) 参照    |
| Span.SpanId              | Span.spanId       | Span.span_id     |      |
| Span.TraceState          | TBD               | TBD              |      |
| Span.Name                | Span.operationName | Span.operation_name |  |
| Span.Kind                | Span.tags["span.kind"] | same | 値の参照については [SpanKind](#spankind) 参照 |
| Span.StartTime           | Span.startTime | Span.start_time | [時間の単位](#時間の単位) 参照|
| Span.EndTime             | Span.duration | same | EndTime - StartTimeで計算されます。[時間の単位](#時間の単位) 参照 |
| Span.Attributes          | Span.tags | same | データ型のマッピングについては [属性(Attribute)](#属性-attribute) 参照            |
| Span.DroppedAttributesCount| Span.tags に追加 | same | 使用するタグ名は[Dropped Attributes Count](nonotlp.md#dropped-attributes-count)を参照してください |
| Span.Events              | Span.logs | same | マッピングの形式については [イベント](#イベント) 参照|
| Span.DroppedEventsCount | Span.tags に追加 | same | 使用するタグ名は[Dropped Events Count](nonotlp.md#dropped-events-count)を参照してください |
| Span.Links               | Span.references | same | [Links](#links) 参照 |
| Span.Status              | Add to Span.tags | same | タグ名の使い方は [Status](#status) 参照 |

<!--
## Mappings
-->

## マッピング

<!--
This section discusses the details of the transformations between OpenTelemetry
and Jaeger.
-->

このセクションでは、OpenTelemetryとJaegerの間の変換の詳細について説明します。

<!--
### Resource
-->

### リソース

<!--
OpenTelemetry resources MUST be mapped to Jaeger's `Span.Process` tags. Multiple resources can exist for a
single process and exporters need to handle this case accordingly.
-->

OpenTelemetryのリソースは、Jaegerの`Span.Process`タグにマッピングされなければなりません(MUST)。1つのプロセスに複数のリソースが存在する可能性があるので、Exporterはその場合でも適宜処理する必要があります。

<!--
Critically, Jaeger backend depends on `Span.Process.ServiceName` to identify the service
that produced the spans. That field MUST be populated from the `service.name` attribute
of the [`service` resource](../../resource/semantic_conventions/README.md#service).
If no `service.name` is contained in a Span's Resource, that field MUST be populated from the
[default](../../resource/sdk.md#sdk-provided-resource-attributes) `Resource`.
-->

重要なことは、Jaegerのバックエンドは、Spanを生成したサービスを識別するために、`Span.Process.ServiceName`に依存しているということです。このフィールドは、[`service` resource](../../resource/semantic_conventions/README.md#service)の `service.name` 属性から生成されなければなりません(MUST)。Spanのリソースに `service.name` が含まれていない場合、そのフィールドは [default](../../resource/sdk.md#sdk-provided-resource-attributes) `Resource` から入力されなければなりません(MUST)。


<!--
### IDs
-->

### ID

<!--
Trace and span IDs in Jaeger are random sequences of bytes. However, Thrift model
represents IDs using `i64` type, or in case of a 128-bit wide Trace ID as two `i64`
fields `traceIdLow` and `traceIdHigh`. The bytes MUST be converted to/from unsigned
ints using Big Endian byte order, e.g. `[0x10, 0x00, 0x00, 0x00] == 268435456`.
The unsigned ints MUST be converted to `i64` by re-interpreting the existing
64bit value as signed `i64`. For example (in Go):
-->

JaegerのTrace IDやSpan IDは、ランダムなバイト列です。しかし、Thriftモデルでは、`i64`型を用いてIDを表現しています。128ビット幅のTrace IDの場合は、2つの`i64`フィールド `traceIdLow`と`traceIdHigh`で表現しています。例えば、`[0x10, 0x00, 0x00, 0x00] == 268435456`のように、ビッグエンディアンのバイトオーダーを使って、バイトを符号なし整数に変換する必要があります。符号なし整数は、既存の64ビット値を符号付きの`i64`として再解釈することで、`i64`に変換しなければなりません(MUST)。以下にGoの場合の例を示します:

<!--
```go
var (
    id       []byte = []byte{0xFF, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00}
    unsigned uint64 = binary.BigEndian.Uint64(id)
    signed   int64  = int64(unsigned)
)
fmt.Println("unsigned:", unsigned)
fmt.Println("  signed:", signed)
// Output:
// unsigned: 18374686479671623680
//   signed: -72057594037927936
```
-->

```go
var (
    id       []byte = []byte{0xFF, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00}
    unsigned uint64 = binary.BigEndian.Uint64(id)
    signed   int64  = int64(unsigned)
)
fmt.Println("unsigned:", unsigned)
fmt.Println("  signed:", signed)
// Output:
// unsigned: 18374686479671623680
//   signed: -72057594037927936
```

<!--
### Parent ID
-->

### Parent ID

<!--
Jaeger Thrift format allows capturing parent ID in a top-level Span field.
Jaeger Proto format does not support parent ID field; instead the parent
MUST be recorded as a `SpanReference` of type `CHILD_OF`, e.g.:
-->

Jaeger Thriftフォーマットでは、トップレベルのSpanフィールドに親のIDを記録することができます。Jaeger Protoフォーマットは親のIDフィールドをサポートしていません。代わりに、親は `CHILD_OF` タイプの `SpanReference` として記録されなければなりません(MUST)。例:

<!--
```python
    SpanReference(
        ref_type=opentracing.CHILD_OF,
        trace_id=span.context.trace_id,
        span_id=parent_id,
    )
```
-->

```python
    SpanReference(
        ref_type=opentracing.CHILD_OF,
        trace_id=span.context.trace_id,
        span_id=parent_id,
    )
```

<!--
This span reference MUST be the first in the list of references.
-->

このSpan referenceは、リファレンスのリストの最初のものでなければなりません。

<!--
### SpanKind
-->

### SpanKind

<!--
OpenTelemetry `SpanKind` field MUST be encoded as `span.kind` tag in Jaeger span,
except for `SpanKind.INTERNAL`, which SHOULD NOT be translated to a tag.
-->

OpenTelemetryの`SpanKind`フィールドは、Jaeger spanの`span.kind`タグとしてエンコードされなければなりません(MUST)。(ただし、`SpanKind.INTERNAL`はタグに変換すべきではありません(SHOULD NOT))。

<!--
| OpenTelemetry | Jaeger |
| ------------- | ------ |
| `SpanKind.CLIENT`|`"client"`|
| `SpanKind.SERVER`|`"server"`|
| `SpanKind.CONSUMER`|`"consumer"`|
| `SpanKind.PRODUCER`|`"producer"` |
| `SpanKind.INTERNAL`| do not add `span.kind` tag |
-->

| OpenTelemetry | Jaeger |
| ------------- | ------ |
| `SpanKind.CLIENT`|`"client"`|
| `SpanKind.SERVER`|`"server"`|
| `SpanKind.CONSUMER`|`"consumer"`|
| `SpanKind.PRODUCER`|`"producer"` |
| `SpanKind.INTERNAL`| `span.kind` タグを追加してはいけません |

<!--
### Unit of time
-->

### 時間の単位

<!--
In Jaeger Thrift format the timestamps and durations MUST be represented in
microseconds (since epoch for timestamps). If the original value in OpenTelemetry
is expressed in nanoseconds, it MUST be rounded or truncated to microseconds.
-->

Jaeger Thriftフォーマットでは、タイムスタンプと持続時間はマイクロ秒で表現されなければなりません(MUST)(タイムスタンプの場合はsince epoch)。OpenTelemetryの元の値がナノ秒で表現されている場合は、マイクロ秒に丸められるか切り捨てられなければなりません(MUST)。

<!--
In Jaeger Proto format the timestamps and durations MUST be represented
with nanosecond precision using `google.protobuf.Timestamp` and
`google.protobuf.Duration` types.
-->

Jaeger Protoフォーマットでは、タイムスタンプと期間(duration)は、`google.protobuf.Timestamp`と`google.protobuf.Duration`タイプを使って、ナノ秒の精度で表現しなければなりません(MUST)。

<!--
### Status
-->

### Status

<!--
The Status is recorded as Span tags. See [Status](non-otlp.md#span-status) for
tag names to use.
-->

StatusはSpanタグとして記録されます。使用するタグ名は[Status](nonotlp.md#span-status)を参照してください。

<!--
#### Error flag
-->

#### Error フラグ

<!--
When Span `Status` is set to `ERROR`, an `error` span tag MUST be added with the
Boolean value of `true`. The added `error` tag MAY override any previous value.
-->

Spanの `Status` が `ERROR` に設定されている場合、`error` のSpan Tagを真偽値 `true` で追加しなければなりません (MUST)。追加された `error` タグは、以前の値をすべて上書きしてもかまいません(MAY)。

<!--
### Attributes
-->

### 属性(Attribute)

<!--
OpenTelemetry Span `Attribute`(s) MUST be reported as `tags` to Jaeger.
-->

OpenTelemetry Span `Attribute`(s) MUST be reported as `tags` to Jaeger.

OpenTelemetry Spanの `Attribute` は、 `tags` としてJaegerに報告しなければなりません(MUST)。

<!--
Primitive types MUST be represented by the corresponding types of Jaeger tags.
-->

プリミティブ型は、対応する型のJaeger tagで表現されなければなりません(MUST)。

<!--
Array values MUST be serialized to string like a JSON list as mentioned in
[semantic conventions](../../overview.md#semantic-conventions).
-->

配列の値は、[セマンティック規約](./../overview.md#semantic-conventions)で述べられているように、JSONリストのように文字列にシリアライズされなければなりません(MUST)。

<!--
### Links
-->

### Links

<!--
OpenTelemetry `Link`(s) MUST be converted to `SpanReference`(s) in Jaeger,
using `FOLLOWS_FROM` reference type. The Link's attributes cannot be represented
in Jaeger explicitly. The exporter MAY additionally convert `Link`(s) to span `Log`(s):
-->

OpenTelemetryの`Link`(s)は、Jaegerの`SpanReference`(s)に、`FOLLOWS_FROM`参照型を使って変換しなければなりません(MUST)。リンクの属性はJaegerでは明示的に表現できません。Exporterはさらに、`Link`(s)をspan `Log`(s)に変換しても構いません(MAY)。

<!--
* use Span start time as the timestamp of the Log
* set Log tag `event=link`
* set Log tags `trace_id` and `span_id` from the respective `SpanContext`'s fields
* store `Link`'s attributes as Log tags
-->

* Spanの開始時間をログのタイムスタンプとして使用する
* Log tag `event=link` を設定する
* Log tag `trace_id` と `span_id` を、それぞれの `SpanContext` のフィールドから設定する
* `Link` の属性をLog tagとして保存する

<!--
Span references generated from `Link`(s) MUST be added _after_ the span reference
generated from [Parent ID](#parent-id), if any.
-->

リンクから生成されたSpanの参照は、[Parent ID](#parent-id)から生成されたSpanの参照があれば、その後に追加しなければなりません(MUST)。

<!--
### Events
-->

### イベント

<!--
Events MUST be converted to Jaeger Logs. OpenTelemetry Event's `time_unix_nano` and `attributes` fields map directly to Jaeger Log's `timestamp` and `fields` fields. Jaeger Log has no direct equivalent for OpenTelemetry Event's `name` field but OpenTracing semantic conventions specify some special attribute names [here](https://github.com/opentracing/specification/blob/master/semantic_conventions.md#log-fields-table). OpenTelemetry Event's `name` field should be added to Jaeger Log's `fields` map as follows:
-->

イベントはJaeger Logに変換しなければなりません(MUST)。OpenTelemetry Eventの`time_unix_nano`と`attributes`フィールドは、Jaeger Logの`timestamp`と`fields`フィールドに直接マッピングされます。Jaeger Logには、OpenTelemetry Eventの`name`フィールドに直接対応するものはありませんが、OpenTracingのセマンティック規約では、いくつかの特別な属性名を指定しています [ここ](https://github.com/opentracing/specification/blob/master/semantic_conventions.md#log-fields-table)で示すようにいくつかの特別な属性名を指定しています。OpenTelemetry Eventの`name`フィールドは、以下のようにJaeger Logの`fields`マップに以下のように追加する必要があります。

<!--
| OpenTelemetry Event Field | Jaeger Attribute |
| -------------------------- | ----------------- |
| `name`|`event`|
-->

| OpenTelemetry Event Field | Jaeger Attribute |
| -------------------------- | ----------------- |
| `name`|`event`|

<!--
* If OpenTelemetry Event contains an attributes with the key `event`, it should take precedence over Event's `name` field.
-->

* OpenTelemetry Event が `event` というキーを持つ属性を含んでいる場合、それは Event の `name` フィールドよりも優先されます。

