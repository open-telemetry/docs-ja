<!--
# OpenTelemetry to Zipkin Transformation
-->

# OpenTelemetry から Zipkin への変換

<!--
**Status**: [Stable](../../document-status.md)
-->

**Status**: [Stable](../../document-status.md)

<!--
This document defines the transformation between OpenTelemetry and Zipkin Spans.
The generic transformation [rules specified here](non-otlp.md) also apply. If a
particular generic transformation rule and the rule in this document contradict
then the rule in this document MUST be used.
-->

このドキュメントでは、OpenTelemetryとZipkin Spansの間の変換を定義しています。一般的な変換ルールである[ここで指定されたルール](non-otlp.md)も適用されます。特定の汎用変換ルールとこの文書のルールが矛盾する場合は、このドキュメントのルールを使用しなければなりません(MUST)。

<!--
Zipkin's v2 API is defined in the
[zipkin.proto](https://github.com/openzipkin/zipkin-api/blob/master/zipkin.proto)
-->

Zipkin の v2 API は [zipkin.proto](https://github.com/openzipkin/zipkin-api/blob/master/zipkin.proto)で定義されています。

<!--
## Summary
-->

## 要約

<!--
The following table summarizes the major transformations between OpenTelemetry
and Zipkin.
-->

以下の表は、OpenTelemetryとZipkinの間の主な変換点をまとめたものです。

<!--
| OpenTelemetry            | Zipkin           | Notes                                                                                         |
| ------------------------ | ---------------- | --------------------------------------------------------------------------------------------- |
| Span.TraceId             | Span.trace_id    |                                                                                               |
| Span.ParentId            | Span.parent_id   |                                                                                               |
| Span.SpanId              | Span.id          |                                                                                               |
| Span.TraceState          | TBD              | TBD                                                                                           |
| Span.Name                | Span.name        |                                                                                               |
| Span.Kind                | Span.kind        | See [SpanKind](#spankind) for values mapping                                                  |
| Span.StartTime           | Span.timestamp   | See [Unit of time](#unit-of-time)                                                             |
| Span.EndTime             | Span.duration    | Duration is calculated based on StartTime and EndTime. See also [Unit of time](#unit-of-time) |
| Span.Attributes          | Span.tags        | See [Attributes](../../common/common.md#attributes) for data types for the mapping.            |
| Span.DroppedAttributesCount| Span.tags | See [Dropped Attributes Count](non-otlp.md#dropped-attributes-count) for tag name to use. |
| Span.Events              | Span.annotations | See [Events](#events) for the mapping format.                                                 |
| Span.DroppedEventsCount | Span.tags | See [Dropped Events Count](non-otlp.md#dropped-events-count) for tag name to use. |
| Span.Links               | TBD              | TBD                                                                                           |
| Span.Status              | Add to Span.tags | See [Status](#status) for tag names to use.                                                   |
-->

| OpenTelemetry            | Zipkin           | 注意                                                                                          |
| ------------------------ | ---------------- | --------------------------------------------------------------------------------------------- |
| Span.TraceId             | Span.trace_id    |                                                                                               |
| Span.ParentId            | Span.parent_id   |                                                                                               |
| Span.SpanId              | Span.id          |                                                                                               |
| Span.TraceState          | TBD              | TBD                                                                                           |
| Span.Name                | Span.name        |                                                                                               |
| Span.Kind                | Span.kind        | 値のマッピングについては [SpanKind](#spankind)参照                                             |
| Span.StartTime           | Span.timestamp   | [時間の単位](#時間の単位) 参照                                                             |
| Span.EndTime             | Span.duration    | 期間(duration)はStartTimeとEndTimeから計算されます。[時間の単位](#時間の単位)を参照 |
| Span.Attributes          | Span.tags        | データ型のマッピングについては[Attributes](../../common/common.md#attributes) 参照            |
| Span.DroppedAttributesCount| Span.tags | 使用するタグ名は[ドロップされた属性数](nonotlp.md#ドロップされた属性数)を参照してください |
| Span.Events              | Span.annotations | マッピング形式については [イベント](#イベント) 参照                                                 |
| Span.DroppedEventsCount | Span.tags | 使用するタグ名は[ドロップされた Events 数](nonotlp.md#ドロップされた-Events-数)を参照してください |
| Span.Links               | TBD              | TBD                                                                                           |
| Span.Status              | Add to Span.tags | 使うタグ名については [Status](#status) 参照                                                   |

<!--
TBD : This is work in progress document and it is currently doesn't specify
mapping for these fields:
-->

TBD : これは現在進行中のドキュメントであり、現在のところこれらのフィールドのマッピングは指定されていません。

<!--
OpenTelemetry fields:
-->

OpenTelemetryのフィールド:

<!--
- Resource attributes
- Tracestate
- Links
- dropped links count
-->

- Resource attributes
- Tracestate
- Links
- dropped links count

<!--
Zipkin fields:
-->

Zipkinのフィールド:

<!--
- local_endpoint
- debug
- shared
-->

- local_endpoint
- debug
- shared

<!--
## Mappings
-->

## マッピング

<!--
This section discusses the details of the transformations between OpenTelemetry
and Zipkin.
-->

このセクションでは、OpenTelemetryとZipkinの間の変換の詳細について説明します。
<!--
### Service name
-->

### サービス名

<!--
Zipkin service name MUST be set to the value of the
[resource attribute](../../resource/semantic_conventions/README.md):
`service.name`. If no `service.name` is contained in a Span's Resource, it MUST be populated from the
[default](../../resource/sdk.md#sdk-provided-resource-attributes) `Resource`.
In Zipkin it is important that the service name is consistent
for all spans in a local root. Otherwise service graph and aggregations would
not work properly. OpenTelemetry doesn't provide this consistency guarantee.
Exporter may chose to override the value for service name based on a local root
span to improve Zipkin user experience.
-->

Zipkin のサービス名は、[resource attribute](../../resource/semantic_conventions/README.md): `service.name` の値に設定されなければなりません(MUST)。もし、SpanのResourceに `service.name` が含まれていない場合は、[default](../../resource/sdk.md#sdk-provided-resource-attributes) `Resource` から生成されなければなりません(MUST)。Zipkin では、サービス名がローカルルートの全てのSpanで一貫していることが重要です。そうでないと、サービスグラフやアグリゲーションが正しく動作しません。OpenTelemetry はこの一貫性を保証しません。Exporterは、Zipkin のユーザーエクスペリエンスを向上させるために、ローカルルートのSpanに基づいてサービス名の値を上書きしても構いません。

<!--
*Note*, the attribute `service.namespace` MUST NOT be used for the Zipkin
service name and should be sent as a Zipkin tag.
-->

*注意*: 属性 `service.namespace` は、Zipkin サービス名に使用してはならず(MUST NOT)、Zipkin タグとして送信しなければなりません。

<!--
### SpanKind
-->

### SpanKind

<!--
The following table lists all the `SpanKind` mappings between OpenTelemetry and
Zipkin.
-->

次の表は、OpenTelemetry と Zipkin の間のすべての `SpanKind` のマッピングの一覧です。

<!--
| OpenTelemetry | Zipkin | Note |
| ------------- | ------ | ---- |
| `SpanKind.CLIENT`|`SpanKind.CLIENT`||
| `SpanKind.SERVER`|`SpanKind.SERVER`||
| `SpanKind.CONSUMER`|`SpanKind.CONSUMER`||
| `SpanKind.PRODUCER`|`SpanKind.PRODUCER` ||
| `SpanKind.INTERNAL`|`null` |must be omitted (set to `null`)|
-->

| OpenTelemetry | Zipkin | 注意 |
| ------------- | ------ | ---- |
| `SpanKind.CLIENT`|`SpanKind.CLIENT`||
| `SpanKind.SERVER`|`SpanKind.SERVER`||
| `SpanKind.CONSUMER`|`SpanKind.CONSUMER`||
| `SpanKind.PRODUCER`|`SpanKind.PRODUCER` ||
| `SpanKind.INTERNAL`|`null` |省略されなければいけません (`null`に設定)|

<!--
### Remote endpoint
-->

### リモートエンドポイント

<!--
#### OTLP -> Zipkin
-->

#### OTLP -> Zipkin

<!--
If Zipkin `SpanKind` resolves to either `SpanKind.CLIENT` or `SpanKind.PRODUCER`
then the service SHOULD specify remote endpoint otherwise Zipkin won't treat the
Span as a dependency. `peer.service` is the preferred attribute but is not
always available. The following table lists the possible attributes for
`remoteEndpoint` by preferred ranking:
-->

Zipkin の `SpanKind` が `SpanKind.CLIENT` または `SpanKind.PRODUCER` に解決する場合、サービスはリモートエンドポイントを指定すべきです(SHOULD)。そうしないと Zipkin はSpanを依存関係として扱いません。`peer.service` が望ましい属性ですが、常に利用できるとは限りません。次の表は、`remoteEndpoint` に指定可能な属性を優先順位別にまとめたものです。

<!--
|Rank|Attribute Name|Reason|
|---|---|---|
|1|peer.service|[OpenTelemetry adopted attribute for remote service.](../semantic_conventions/span-general.md#general-remote-service-attributes)|
|2|net.peer.name|[OpenTelemetry adopted attribute for remote hostname, or similar.](../semantic_conventions/span-general.md#general-network-connection-attributes)|
|3|net.peer.ip & net.peer.port|[OpenTelemetry adopted attribute for remote address of the peer.](../semantic_conventions/span-general.md#general-network-connection-attributes)|
|4|peer.hostname|Remote hostname defined in OpenTracing specification.|
|5|peer.address|Remote address defined in OpenTracing specification.|
|6|http.host|Commonly used HTTP host header attribute for Http Spans.|
|7|db.name|Commonly used database name attribute for DB Spans.|
-->

|Rank|Attribute Name|Reason|
|---|---|---|
|1|peer.service|[OpenTelemetryが採用したリモートサービス用の属性](../semantic_conventions/span-general.md#general-remote-service-attributes)|
|2|net.peer.name|[OpenTelemetryが採用したリモートホスト名の属性、またはそれに類するもの](../semantic_conventions/span-general.md#general-network-connection-attributes)|
|3|net.peer.ip & net.peer.port|[OpenTelemetryが採用した、ピアのリモートアドレスの属性](../semantic_conventions/span-general.md#general-network-connection-attributes)|
|4|peer.hostname|OpenTracingの仕様で定義されているリモートホスト名|
|5|peer.address|OpenTracingの仕様で定義されているリモートアドレス|
|6|http.host|HTTP Spanでよく使われるHTTPホスト・ヘッダ属性|
|7|db.name|DB Spanでよく使われるデータベース名属性|

<!--
* Ranking should control the selection order. For example, `net.peer.name` (Rank
  2) should be selected before `http.host` (Rank 6).
* `net.peer.ip` can be used by itself as `remoteEndpoint` but should be combined
  with `net.peer.port` if it is also present.
-->

* ランキングは、選択順序を制御する必要があります。例えば、`net.peer.name` (ランク2) は `http.host` (ランク6) よりも先に選択されるべきです。
* `net.peer.ip` は単独で `remoteEndpoint` として使用することができますが、`net.peer.port` が存在する場合は、それと組み合わせる必要があります。

<!--
#### Zipkin -> OTLP
-->

#### Zipkin -> OTLP

<!--
When mapping from Zipkin to OTLP set `peer.service` tag from `remoteEndpoint`
unless there is a `peer.service` tag defined explicitly.
-->

Zipkin から OTLP にマッピングする際、`peer.service` タグが明示的に定義されていない限り、`remoteEndpoint` から `peer.service` タグを設定します。

<!--
### Attribute
-->

### 属性

<!--
OpenTelemetry Span `Attribute`(s) MUST be reported as `tags` to
Zipkin.
-->

OpenTelemetry Span `Attribute`(s)は、`tags`としてZipkinに報告されなければなりません(MUST)。

<!--
Some attributes defined in [semantic
convention](../semantic_conventions/README.md)
document maps to the strongly-typed fields of Zipkin spans.
-->

[セマンティック規約](../semantic_conventions/README.md)ドキュメントで定義されたいくつかの属性は、Zipkin spanのstrong-typedフィールドにマップされます。

<!--
Primitive types MUST be converted to string using en-US culture settings.
Boolean values MUST use lower case strings `"true"` and `"false"`.
-->

プリミティブ型は、en-US Locale 設定を用いて文字列に変換しなければなりません(MUST)。真偽値は小文字の文字列 `"true"` と `"false"` を使用しなければなりません(MUST)。

<!--
Array values MUST be serialized to string like a JSON list as mentioned in
[semantic conventions](../../overview.md#semantic-conventions).
-->

配列の値は、[セマンティック規約](./../overview.md#semantic-conventions)で述べられているように、JSONリストのように文字列にシリアライズされなければなりません(MUST)。

<!--
TBD: add examples
-->

TBD: 例を追加する

<!--
### Status
-->

### Status

<!--
This section overrides the
[generic Status mapping rule](non-otlp.md#span-status).
-->

このセクションは、[一般的なStatusのマッピングルール](non-otlp.md#span-status)よりも優先されます。

<!--
Span `Status` MUST be reported as a key-value pair in `tags` to Zipkin, unless it is `UNSET`.
In the latter case it MUST NOT be reported.
-->

Span `Status` は、`UNSET` でない限り、`tags` のキーと値のペアとして Zipkin に報告しなければなりません (MUST)。後者の場合は、報告してはいけません(MUST NOT)。

<!--
The following table defines the OpenTelemetry `Status` to Zipkin `tags` mapping.
-->

以下の表は、OpenTelemetry の `Status` から Zipkin の `tags` へのマッピングを定義しています。

<!--
| Status|Tag Key| Tag Value |
|--|--|--|
|Code | `otel.status_code` | Name of the code, either `OK` or `ERROR`. MUST NOT be set if the code is `UNSET`. |
|Description| `error` | Description of the `Status`. MUST be set if the code is `ERROR`, use an empty string if Description has no value. MUST NOT be set for `OK` and `UNSET` codes. |
-->

| Status|Tag キー| Tag の値 |
|--|--|--|
|Code | `otel.status_code` | コードの名前で、`OK`または`ERROR`です。コードが `UNSET` の場合、設定してはいけません (MUST NOT) |
|Description| `error` | `Status`の説明です。コードが `ERROR` の場合は設定しなければなりません (MUST)。コードが `OK` と `UNSET` の場合は設定してはいけません。|


<!--
Note: The `error` tag should only be set if `Status` is `Error`. If a boolean
version (`{"error":false}` or `{"error":"false"}`) is present, it SHOULD be
removed. Zipkin will treat any span with `error` sent as failed.
-->

注意: `error` タグは `Status` が `Error` の場合にのみ設定してください。真偽値バージョン (`{"error":false}` または `{"error": "false"}`) が存在する場合は、削除するべきです (SHOULD)。Zipkin は `error` が送られた全てのSpanを失敗したものとして扱います。

<!--
### Events
-->

### イベント

<!--
OpenTelemetry `Event` has an optional `Attribute`(s) which is not supported by
Zipkin. Events MUST be converted to the Annotations with the names which will
include attribute values like this:
-->

OpenTelemetry の `Event` には、オプションの `Attribute` がありますが、Zipkin ではサポートされていません。イベントは、以下のような属性値を含む名前のアノテーションに変換しなければなりません(MUST)。

<!--
```
"my-event-name": { "key1" : "value1", "key2": "value2" }
```
-->

```
"my-event-name": { "key1" : "value1", "key2": "value2" }
```

<!--
### Unit of Time
-->

### 時間の単位

<!--
Zipkin times like `timestamp`, `duration` and `annotation.timestamp` MUST be
reported in microseconds as whole numbers. For example, `duration`
of 1234 nanoseconds will be represented as 1 microsecond.
-->

`timestamp`, `duration`, `annotation.timestamp` などのZipkinの時間は、マイクロ秒単位の整数で報告しなければなりません(MUST)。例えば、`duration` の 1234 ナノ秒は、1 マイクロ秒として表されます。

<!--
## Request Payload
-->

## リクエストのペイロード

<!--
For performance considerations, Zipkin fields that can be absent SHOULD be
omitted from the payload when they are empty in the OpenTelemetry `Span`.
-->

パフォーマンスを考慮して、省略できるZipkinフィールドは、OpenTelemetryの`Span`の中で空である場合、ペイロードから省略されるべきです(SHOULD)。

<!--
For example, an OpenTelemetry `Span` without any `Event` should not have an
`annotations` field in the Zipkin payload.
-->

例えば、`Event` のない OpenTelemetry `Span` は、Zipkin のペイロードに `annotations` フィールドを持つべきではありません。

<!--
## Considerations for Legacy (v1) Format
-->

## レガシー(v1)フォーマットに関する注意事項

<!--
Zipkin's v2 [json](https://github.com/openzipkin/zipkin-api/blob/master/zipkin2-api.yaml) format was defined in 2017, followed up by a [protobuf](https://github.com/openzipkin/zipkin-api/blob/master/zipkin.proto) format in 2018.
-->

Zipkinのv2の[json](https://github.com/openzipkin/zipkin-api/blob/master/zipkin2-api.yaml)フォーマットは2017年に定義され、続いて2018年に[protobuf](https://github.com/openzipkin/zipkin-api/blob/master/zipkin.proto)フォーマットが定義されました。

<!--
Frameworks made before then use a more complex v1 [Thrift](https://github.com/openzipkin/zipkin-api/blob/master/thrift/zipkinCore.thrift) or [json](https://github.com/openzipkin/zipkin-api/blob/master/zipkin-api.yaml) format that notably differs in so far as it uses terminology such as Binary Annotation, and repeats endpoint information on each attribute.
-->

それ以前に作られたフレームワークは、より複雑なv1 [Thrift](https://github.com/openzipkin/zipkin-api/blob/master/thrift/zipkinCore.thrift)または[json](https://github.com/openzipkin/zipkin-api/blob/master/zipkin-api.yaml)フォーマットを使用しています。このフォーマットは、バイナリアノテーションなどの用語を使用し、各属性に対してエンドポイント情報を繰り返すという点で大きく異なっています。

<!--
Consider using [V1SpanConverter.java](https://github.com/openzipkin/zipkin/blob/master/zipkin/src/main/java/zipkin2/v1/V1SpanConverter.java) as a reference implementation for converting v1 model to OpenTelemetry.
-->

v1モデルをOpenTelemetryに変換するための参照実装として、[V1SpanConverter.java](https://github.com/openzipkin/zipkin/blob/master/zipkin/src/main/java/zipkin2/v1/V1SpanConverter.java)の使用を検討してください。

<!--
The span timestamp and duration were late additions to the V1 format. As in the code link above, it is possible to heuristically derive them from annotations.
-->

SpanのタイムスタンプとDurationは、V1フォーマットに後から追加されたものです。上記のコードリンクのように、アノテーションからヒューリスティックに導き出すことが可能です。