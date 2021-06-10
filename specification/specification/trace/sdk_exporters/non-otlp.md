<!--
# OpenTelemetry Transformation to non-OTLP Formats
-->

# 非OTLPフォーマットへのOpenTelemetry変換

<!--
**Status**: [Stable](../../document-status.md)
-->

**Status**: [Stable](../../document-status.md)

<!--
All OpenTelemetry concepts and span data recorded using OpenTelemetry API can be
directly and precisely represented using corresponding messages and fields of
OTLP format. However, for other formats this is not always the case. Sometimes a
format will not have a native way to represent a particular OpenTelemetry
concept or a field of a concept.
-->

すべてのOpenTelemetryコンセプトとOpenTelemetry APIを使って記録されたSpanデータは、OTLPフォーマットの対応するメッセージとフィールドを使って直接かつ正確に表現することができます。しかし、他のフォーマットについては、必ずしもそうではありません。あるフォーマットが、特定のOpenTelemetryコンセプトやコンセプトのフィールドを表現するネイティブな方法を持っていないこともあります。

<!--
This document defines the transformation between OpenTelemetry and formats other
than OTLP, for OpenTelemetry fields and concepts that have no direct semantic
equivalent in those other formats.
-->

このドキュメントでは、OTLP以外のフォーマットとOpenTelemetryの間の変換を定義しています。OpenTelemetryのフィールドやコンセプトは、他のフォーマットでは直接の意味的な互換性がありません。

<!--
Note: when a format has a direct semantic equivalent for a particular field or
concept then the recommendation in this document MUST be ignored.
-->

注:あるフォーマットが、特定のフィールドやコンセプトに対して意味的に等価なフィールド直接的に持っている場合、この文書の推奨事項は無視されなければなりません(MUST)。

<!--
See also additional specific transformation rules for [Jaeger](jaeger.md) and
[Zipkin](zipkin.md). The specific rules for Jaeger and Zipkin take precedence
over the generic rules defined in this document.
-->

[Jaeger](jaeger.md)と[Zipkin](zipkin.md)の特定の変換ルールも追加で参照してください。JaegerとZipkinの固有のルールは、このドキュメントで定義された一般的なルールよりも優先されます。

<!--
## Mappings
-->

## マッピング

<!--
### InstrumentationLibrary
-->

### InstrumentationLibrary

<!--
OpenTelemetry `InstrumentationLibrary`'s fields MUST be reported as key-value
pairs associated with the Span using the following mapping:
-->

OpenTelemetry `InstrumentationLibrary` のフィールドは、以下のマッピングを使用して、Spanに関連付けられたキーと値のペアとして報告されなければなりません(MUST)。

<!--
| OpenTelemetry InstrumentationLibrary Field | non-OTLP Key |
| ------------------- | --- |
| `InstrumentationLibrary.name`|`otel.library.name`|
| `InstrumentationLibrary.version`|`otel.library.version`|
-->

| OpenTelemetry InstrumentationLibrary フィールド | 非OTLPのキー |
| ------------------- | --- |
| `InstrumentationLibrary.name`|`otel.library.name`|
| `InstrumentationLibrary.version`|`otel.library.version`|

<!--
### Span Status
-->

### Span Status

<!--
Span `Status` MUST be reported as key-value pairs associated with the Span,
unless the `Status` is `UNSET`. In the latter case it MUST NOT be reported.
-->

Spanの `Status` は、 `Status` が `UNSET` でない限り、Spanに関連付けられたキー・バリュー・ペアとして報告されなければなりません(MUST)。後者の場合は、報告してはいけません(MUST NOT)。

<!--
The following table defines the OpenTelemetry `Status`'s mapping to Span's
key-value pairs:
-->

以下の表は、OpenTelemetryの`Status`とSpanのキーと値のペアのマッピングを定義しています。

<!--
|OpenTelemetry Status Field|non-OTLP Key|non-OTLP Value|
|--|--|--|
|Code | `otel.status_code` | Name of the code, either `OK` or `ERROR`. MUST NOT be set if the code is `UNSET`. |
|Description | `otel.status_description` | Description of the `Status` if it has a value otherwise not set. |
-->

|OpenTelemetry Status フィールド|非OTLPのキー|非OTLPの値|
|--|--|--|
|Code | `otel.status_code` | このコードの名前は、`OK`または`ERROR`です。コードが `UNSET` の場合、設定してはいけません (MUST NOT) |
|Description | `otel.status_description` | `Status`に値がある場合はその説明、ない場合は設定されていません |

<!--
### Dropped Attributes Count
-->

### ドロップされた属性数

<!--
OpenTelemetry Span's dropped attributes count MUST be reported as a key-value
pair associated with the Span. Similarly, Span Event's dropped attributes count
MUST be reported as a key-value pair associated with the Span Event. In both
cases the key name MUST be `otel.dropped_attributes_count`.
-->

OpenTelemetry Span のドロップされた属性数は、Span に関連付けられたキー・バリュー・ペアとして報告されなければなりません(MUST)。同様に、Spanイベントのドロップされた属性数は、Spanイベントに関連付けられたキー・バリュー・ペアとして報告されなければなりません(MUST)。どちらの場合も、キー名は `otel.dropped_attributes_count` としなければなりません(MUST)。

<!--
This key-value pair should only be recorded when it contains a non-zero value.
-->

このキー・バリュー・ペアは、ゼロ以外の値が含まれている場合にのみ記録されます。

<!--
### Dropped Events Count
-->

### ドロップされた Events 数

<!--
OpenTelemetry Span's dropped events count MUST be reported as a key-value pair
associated with the Span. The key name MUST be `otel.dropped_events_count`.
-->

OpenTelemetry Spanのドロップされたイベント数は、Spanに関連付けられたキー・バリュー・ペアとして報告されなければなりません(MUST)。キーの名前は `otel.dropped_events_count` でなければなりません(MUST)。

<!--
This key-value pair should only be recorded when it contains a non-zero value.
-->

このキー・バリュー・ペアは、ゼロ以外の値が含まれている場合にのみ記録されます。

