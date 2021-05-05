<!--
# OpenTelemetry Environment Variable Specification
-->

# OpenTelemetry環境変数の仕様

<!--
**Status**: [Mixed](document-status.md)
-->

**Status**: [Mixed](document-status.md)

<!--
The goal of this specification is to unify the environment variable names between different OpenTelemetry SDK implementations. SDKs MAY choose to allow configuration via the environment variables in this specification, but are not required to. If they do, they SHOULD use the names listed in this document.
-->

この仕様の目的は、異なるOpenTelemetry SDKの実装間で環境変数名を統一することです。SDKはこの仕様の環境変数を使った設定を許可してもよい(MAY)が、必須ではありません。その場合は、このドキュメントに記載されている名前を使用するべきです(SHOULD)。

<!--
## Special configuration types
-->

## 特殊な設定タイプ

<!--
**Status**: [Stable](document-status.md)
-->

**Status**: [Stable](document-status.md)

<!--
### Numeric value
-->

### 数値

<!--
If an SDK chooses to support an integer-valued environment variable, it SHOULD support nonnegative values between 0 and 2³¹ − 1 (inclusive). Individual SDKs MAY choose to support a larger range of values.
-->

SDKが整数値の環境変数をサポートすることを選択した場合、0から2³¹ - 1(Inclusive)の間の非負の値をサポートするべきです(SHOULD)。個々のSDKは、より広い範囲の値をサポートしてもかまいません(MAY)。

<!--
### Enum value
-->

### Enum 値

<!--
For variables which accept a known value out of a set, i.e., an enum value, SDK implementations MAY support additional values not listed here.
For variables accepting an enum value, if the user provides a value the SDK does not recognize, the SDK MUST generate a warning and gracefully ignore the setting.
-->

セットから既知の値を受け入れる変数(enum値)の場合、SDKの実装はここに記載されていない追加の値をサポートしても構いません(MAY)。enum値を受け入れる変数については、SDKが認識できない値をユーザーが提供した場合、SDKは警告を生成し、その設定を無視しなければなりません(MUST)。

<!--
### Duration
-->

### 期間(Duration)

<!--
Any value that represents a duration, for example a timeout, MUST be an integer representing a number of
milliseconds. The value is non-negative - if a negative value is provided, the SDK MUST generate a warning,
gracefully ignore the setting and use the default value if it is defined.
-->

タイムアウトなど、継続時間を表す値は、ミリ秒数を表す整数でなければなりません(MUST)。値は非負の値です。負の値が提供された場合、SDKは警告を生成し、設定を無視し、デフォルト値が定義されている場合はそれを使用しなければなりません(MUST)。

<!--
For example, the value `12000` indicates 12000 milliseconds, i.e., 12 seconds.
-->

例えば、`12000`という値は、12000ミリ秒、つまり12秒を表しています。

<!--
## General SDK Configuration
-->

## 一般的なSDKの設定

<!--
**Status**: [Stable](document-status.md)
-->

**Status**: [Stable](document-status.md)

<!--
| Name                     | Description                                       | Default                           | Notes                               |
| ------------------------ | ------------------------------------------------- | --------------------------------- | ----------------------------------- |
| OTEL_RESOURCE_ATTRIBUTES | Key-value pairs to be used as resource attributes |                                   | See [Resource SDK](./resource/sdk.md#specifying-resource-information-via-an-environment-variable) for more details. |
| OTEL_LOG_LEVEL           | Log level used by the SDK logger                  | "info"                            |                                     |
| OTEL_PROPAGATORS         | Propagators to be used as a comma separated list  | "tracecontext,baggage"            | Values MUST be deduplicated in order to register a `Propagator` only once. |
| OTEL_TRACES_SAMPLER       | Sampler to be used for traces                     | "parentbased_always_on"                       | See [Sampling](./trace/sdk.md#sampling) |
| OTEL_TRACES_SAMPLER_ARG   | String value to be used as the sampler argument   |                                   | The specified value will only be used if OTEL_TRACES_SAMPLER is set. Each Sampler type defines its own expected input, if any. Invalid or unrecognized input MUST be logged and MUST be otherwise ignored, i.e. the SDK MUST behave as if OTEL_TRACES_SAMPLER_ARG is not set.  |
-->

| Name                     | Description                                       | Default                           | Notes                               |
| ------------------------ | ------------------------------------------------- | --------------------------------- | ----------------------------------- |
| OTEL_RESOURCE_ATTRIBUTES | リソースの属性として使用されるキーバリューペア |                                   | 詳細は[リソースSDK](./resource/sdk.md#specifying-resource-information-via-an-environment-variable)参照 |
| OTEL_LOG_LEVEL           | SDKロガーが使用するログレベル                 | "info"                            |                                     |
| OTEL_PROPAGATORS         | コンマで区切られたリストとして使用されるPropagetor  | "tracecontext,baggage"            | `Propagator`を一度だけ登録するためには、値を重複排除しなければなりません(MUST) |
| OTEL_TRACES_SAMPLER       | トレースに使用するサンプラー                | "parentbased_always_on"                       | [サンプリング](./trace/sdk.md#sampling)参照 |
| OTEL_TRACES_SAMPLER_ARG   | サンプラー引数として使用する文字列値   |                                   | 指定された値は、OTEL_TRACES_SAMPLERが設定されている場合にのみ使用されます。サンプラータイプごとに、期待される入力があればそれを定義します。無効または認識できない入力はログに記録されなければならず(MUST)、それ以外は無視されなければなりません(MUST)、つまり、SDKはOTEL_TRACES_SAMPLER_ARGが設定されていないかのように動作しなければなりません(MUST)。|

<!--
Known values for OTEL_PROPAGATORS are:
-->

OTEL_PROPAGATORSの既知の値は以下の通りです。

<!--
- `"tracecontext"`: [W3C Trace Context](https://www.w3.org/TR/trace-context/)
- `"baggage"`: [W3C Baggage](https://www.w3.org/TR/baggage/)
- `"b3"`: [B3 Single](https://github.com/openzipkin/b3-propagation#single-header)
- `"b3multi"`: [B3 Multi](https://github.com/openzipkin/b3-propagation#multiple-headers)
- `"jaeger"`: [Jaeger](https://www.jaegertracing.io/docs/1.21/client-libraries/#propagation-format)
- `"xray"`: [AWS X-Ray](https://docs.aws.amazon.com/xray/latest/devguide/xray-concepts.html#xray-concepts-tracingheader) (_third party_)
- `"ottrace"`: [OT Trace](https://github.com/opentracing?q=basic&type=&language=) (_third party_)
-->

- `"tracecontext"`: [W3C Trace Context](https://www.w3.org/TR/trace-context/)
- `"baggage"`: [W3C Baggage](https://www.w3.org/TR/baggage/)
- `"b3"`: [B3 Single](https://github.com/openzipkin/b3-propagation#single-header)
- `"b3multi"`: [B3 Multi](https://github.com/openzipkin/b3-propagation#multiple-headers)
- `"jaeger"`: [Jaeger](https://www.jaegertracing.io/docs/1.21/client-libraries/#propagation-format)
- `"xray"`: [AWS X-Ray](https://docs.aws.amazon.com/xray/latest/devguide/xray-concepts.html#xray-concepts-tracingheader) (_サードパーティー_)
- `"ottrace"`: [OT Trace](https://github.com/opentracing?q=basic&type=&language=) (_サードパーティー_)

<!--
Known values for `OTEL_TRACES_SAMPLER` are:
-->

OTEL_TRACES_SAMPLER`の既知の値は次のとおりです。

<!--
- `"always_on"`: `AlwaysOnSampler`
- `"always_off"`: `AlwaysOffSampler`
- `"traceidratio"`: `TraceIdRatioBased`
- `"parentbased_always_on"`: `ParentBased(root=AlwaysOnSampler)`
- `"parentbased_always_off"`: `ParentBased(root=AlwaysOffSampler)`
- `"parentbased_traceidratio"`: `ParentBased(root=TraceIdRatioBased)`
-->

- `"always_on"`: `AlwaysOnSampler`
- `"always_off"`: `AlwaysOffSampler`
- `"traceidratio"`: `TraceIdRatioBased`
- `"parentbased_always_on"`: `ParentBased(root=AlwaysOnSampler)`
- `"parentbased_always_off"`: `ParentBased(root=AlwaysOffSampler)`
- `"parentbased_traceidratio"`: `ParentBased(root=TraceIdRatioBased)`

<!--
Depending on the value of `OTEL_TRACES_SAMPLER`, `OTEL_TRACES_SAMPLER_ARG` may be set as follows:
-->

`OTEL_TRACES_SAMPLER`の値に応じて、`OTEL_TRACES_SAMPLER_ARG`を以下のように設定することができます。

<!--
- For `traceidratio` and `parentbased_traceidratio` samplers: Sampling probability, a number in the [0..1] range, e.g. "0.25". Default is 1.0 if unset.
-->

- `traceidratio` と `parentbased_traceidratio` のサンプラー用: "0.25"のように、[0..1]の範囲の数値です。設定されていない場合のデフォルトは1.0です。

<!--
## Batch Span Processor
-->

## バッチSpan Processor

<!--
**Status**: [Stable](document-status.md)
-->

**Status**: [Stable](document-status.md)

<!--
| Name                           | Description                                    | Default | Notes                                                 |
| ------------------------------ | ---------------------------------------------- | ------- | ----------------------------------------------------- |
| OTEL_BSP_SCHEDULE_DELAY        | Delay interval between two consecutive exports | 5000    |                                                       |
| OTEL_BSP_EXPORT_TIMEOUT        | Maximum allowed time to export data            | 30000   |                                                       |
| OTEL_BSP_MAX_QUEUE_SIZE        | Maximum queue size                             | 2048    |                                                       |
| OTEL_BSP_MAX_EXPORT_BATCH_SIZE | Maximum batch size                             | 512     | Must be less than or equal to OTEL_BSP_MAX_QUEUE_SIZE |
-->

| Name                           | Description                                    | Default | Notes                                                 |
| ------------------------------ | ---------------------------------------------- | ------- | ----------------------------------------------------- |
| OTEL_BSP_SCHEDULE_DELAY        | 連続する2つのExportの間の遅延時間 | 5000    |                                                       |
| OTEL_BSP_EXPORT_TIMEOUT        | データをexportする最大タイムアウト            | 30000   |                                                       |
| OTEL_BSP_MAX_QUEUE_SIZE        | 最大キューサイズ                             | 2048    |                                                       |
| OTEL_BSP_MAX_EXPORT_BATCH_SIZE | 最大バッチサイズ                             | 512     | OTEL_BSP_MAX_QUEUE_SIZEであること |

<!--
## Span Collection Limits
-->

## Span Collectionの制限

<!--
**Status**: [Stable](document-status.md)
-->

**Status**: [Stable](document-status.md)

<!--
See the SDK [Span Limits](trace/sdk.md#span-limits) section for the definition of the limits.
-->

制限の定義については、SDK [Span Limits](trace/sdk.md#span-limits)のセクションを参照してください。

<!--
| Name                            | Description                          | Default | Notes |
| ------------------------------- | ------------------------------------ | ------- | ----- |
| OTEL_SPAN_ATTRIBUTE_COUNT_LIMIT | Maximum allowed span attribute count | 128     |       |
| OTEL_SPAN_EVENT_COUNT_LIMIT     | Maximum allowed span event count     | 128     |       |
| OTEL_SPAN_LINK_COUNT_LIMIT      | Maximum allowed span link count      | 128     |       |
-->

| Name                            | 説明                                 | デフォルト | 注意 |
| ------------------------------- | ------------------------------------ | ---------- | ----- |
| OTEL_SPAN_ATTRIBUTE_COUNT_LIMIT | 最大Span属性数 | 128        |       |
| OTEL_SPAN_EVENT_COUNT_LIMIT     | 最大Spanイベント数     | 128        |       |
| OTEL_SPAN_LINK_COUNT_LIMIT      | 最大Span Link数      | 128        |       |

<!--
## OTLP Exporter
-->

## OTLP Exporter

<!--
See [OpenTelemetry Protocol Exporter Configuration Options](./protocol/exporter.md).
-->

[OpenTelemetry Protocol Exporter Configuration Options](./protocol/exporter.md)を参照してください。

<!--
## Jaeger Exporter
-->

## Jaeger Exporter

<!--
**Status**: [Stable](document-status.md)
-->

**Status**: [Stable](document-status.md)

| Name                            | Description                                       | Default                                                                                          |
| ------------------------------- | ------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| OTEL_EXPORTER_JAEGER_AGENT_HOST | Jaeger agentが使うホスト名                     | "localhost"                                                                                      |
| OTEL_EXPORTER_JAEGER_AGENT_PORT | Jaeger agentが使うポート                         | 6832                                                                                             |
| OTEL_EXPORTER_JAEGER_ENDPOINT   | Jaeger tracesが使うHTTP エンドポイント                   | <!-- markdown-link-check-disable --> "http://localhost:14250"<!-- markdown-link-check-enable --> |
| OTEL_EXPORTER_JAEGER_USER       | HTTP BASIC認証で使うユーザー名 | -                                                                                                |
| OTEL_EXPORTER_JAEGER_PASSWORD   | HTTP BASIC認証で使うパスワード | -                                                                                                |


<!--
## Zipkin Exporter
-->

## Zipkin Exporter

<!--
**Status**: [Stable](document-status.md)
-->

**Status**: [Stable](document-status.md)


| Name                          | 説明                | Default                                                                                                      |
| ----------------------------- | -------------------------- | ------------------------------------------------------------------------------------------------------------ |
| OTEL_EXPORTER_ZIPKIN_ENDPOINT | Endpoint for Zipkin traces | <!-- markdown-link-check-disable --> "http://localhost:9411/api/v2/spans"<!-- markdown-link-check-enable --> |

<!--
Addtionally, the following environment variables are reserved for future
usage in Zipkin Exporter configuration:
-->

さらに、以下の環境変数は、将来 Zipkin Exporter の設定で使用するために予約されています。

<!--
- `OTEL_EXPORTER_ZIPKIN_PROTOCOL`
-->

- `OTEL_EXPORTER_ZIPKIN_PROTOCOL`

<!--
This will be used to specify whether or not the exporter uses v1 or v2, json,
thrift or protobuf.  As of 1.0 of the specification, there
*is no specified default, or configuration via environment variables*.
-->

これは、エクスポーターがv1とv2、json、thrift、protobufのいずれを使用するかを指定するために使用されます。仕様書の1.0時点では、*デフォルトの指定はなく、環境変数による設定もありません*。

<!--
## Prometheus Exporter
-->

## Prometheus Exporter

<!--
**Status**: [Experimental](document-status.md)
-->

**Status**: [Experimental](document-status.md)

<!--
| Name                          | Description                     | Default                      |
| ----------------------------- | --------------------------------| ---------------------------- |
| OTEL_EXPORTER_PROMETHEUS_HOST | Host used by the Prometheus exporter | All addresses: "0.0.0.0"|
| OTEL_EXPORTER_PROMETHEUS_PORT | Port used by the Prometheus exporter | 9464                    |
-->

| Name                          | Description                     | Default                      |
| ----------------------------- | --------------------------------| ---------------------------- |
| OTEL_EXPORTER_PROMETHEUS_HOST | Prometheus exporterで使うホスト | All addresses: "0.0.0.0"|
| OTEL_EXPORTER_PROMETHEUS_PORT | Prometheus exporterで使うポート | 9464                    |

<!--
## Exporter Selection
-->

## Exporter 選択

<!--
**Status**: [Stable](document-status.md)
-->

**Status**: [Stable](document-status.md)

<!--
We define environment variables for setting a single exporter per signal.
-->

シグナルごとに1つのエクスポーターを設定するための環境変数を定義します。

<!--
| Name          | Description                                                                  | Default |
| ------------- | ---------------------------------------------------------------------------- | ------- |
| OTEL_TRACES_EXPORTER | Trace exporter to be used | "otlp"  |
| OTEL_METRICS_EXPORTER | Metrics exporter to be used | "otlp"  |
-->

| Name          | Description                                                                  | Default |
| ------------- | ---------------------------------------------------------------------------- | ------- |
| OTEL_TRACES_EXPORTER | 使用するTrace Exporter | "otlp"  |
| OTEL_METRICS_EXPORTER | 使用するMetrics exporter | "otlp"  |

<!--
Known values for OTEL_TRACES_EXPORTER are:
-->

OTEL_TRACES_EXPORTERの既知の値は以下のとおりです。

<!--
- `"otlp"`: [OTLP](./protocol/otlp.md)
- `"jaeger"`: [Jaeger gRPC](https://www.jaegertracing.io/docs/1.21/apis/#protobuf-via-grpc-stable)
- `"zipkin"`: [Zipkin](https://zipkin.io/zipkin-api/) (Defaults to [protobuf](https://github.com/openzipkin/zipkin-api/blob/master/zipkin.proto) format)
-->

- `"otlp"`: [OTLP](./protocol/otlp.md)
- `"jaeger"`: [Jaeger gRPC](https://www.jaegertracing.io/docs/1.21/apis/#protobuf-via-grpc-stable)
- `"zipkin"`: [Zipkin](https://zipkin.io/zipkin-api/) (Defaults to [protobuf](https://github.com/openzipkin/zipkin-api/blob/master/zipkin.proto) format)

<!--
Known values for OTEL_METRICS_EXPORTER are:
-->

OTEL_METRICS_EXPORTERの既知の値は以下のとおりです。

<!--
- `"otlp"`: [OTLP](./protocol/otlp.md)
- `"prometheus"`: [Prometheus](https://github.com/prometheus/docs/blob/master/content/docs/instrumenting/exposition_formats.md)
-->

- `"otlp"`: [OTLP](./protocol/otlp.md)
- `"prometheus"`: [Prometheus](https://github.com/prometheus/docs/blob/master/content/docs/instrumenting/exposition_formats.md)

<!--
## Language Specific Environment Variables
-->

## 言語固有の環境変数

<!--
To ensure consistent naming across projects, this specification recommends that language specific environment variables are formed using the following convention:
-->

本仕様書では、プロジェクト間で一貫した名称を使用するために、言語固有の環境変数を次のような規則で形成することを推奨しています。

<!--
```
OTEL_{LANGUAGE}_{FEATURE}
```
-->

```
OTEL_{LANGUAGE}_{FEATURE}
```


