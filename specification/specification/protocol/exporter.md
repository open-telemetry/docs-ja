<!--
# OpenTelemetry Protocol Exporter
-->

# OpenTelemetry Protocol Exporter

**Status**: [Stable](../document-status.md)

<!--
This document specifies the configuration options available to the OpenTelemetry Protocol ([OTLP](https://github.com/open-telemetry/oteps/blob/main/text/0035-opentelemetry-protocol.md)) Exporter as well as the retry behavior.
-->

本ドキュメントは、OpenTelemetry Protocol ([OTLP](https://github.com/open-telemetry/oteps/blob/main/text/0035-opentelemetry-protocol.md)) Exporterで利用可能な設定オプションと、リトライの動作について規定しています。

<!--
## Configuration Options
-->

## 設定オプション

<!--
The following configuration options MUST be available to configure the OTLP exporter. Each configuration option MUST be overridable by a signal specific option.
-->

OTLP Exporterを構成するために、以下の構成オプションが利用可能でなければなりません(MUST)。また、各設定オプションは、Signal固有のオプションでオーバーライド可能でなければなりません(MUST)。

<!--
| Configuration Option | Description                                                  | Default           | Env variable                                                 |
| -------------------- | ------------------------------------------------------------ | ----------------- | ------------------------------------------------------------ |
| Endpoint             | Target to which the exporter is going to send spans or metrics. The endpoint MUST be a valid URL with scheme (http or https) and host, and MAY contain a port and path. A scheme of https indicates a secure connection. When using `OTEL_EXPORTER_OTLP_ENDPOINT` with OTLP/HTTP, exporters SHOULD follow the collector convention of appending the version and signal to the path (e.g. `v1/traces` or `v1/metrics`). The per-signal endpoint configuration options take precedence and can be used to override this behavior. See the [OTLP Specification][otlphttp-req] for more details. | `https://localhost:4317` | `OTEL_EXPORTER_OTLP_ENDPOINT` `OTEL_EXPORTER_OTLP_TRACES_ENDPOINT` `OTEL_EXPORTER_OTLP_METRICS_ENDPOINT` |
| Certificate File     | Path to certificate file for TLS credentials of gRPC client. Should only be used for a secure connection. | n/a               | `OTEL_EXPORTER_OTLP_CERTIFICATE` `OTEL_EXPORTER_OTLP_TRACES_CERTIFICATE` `OTEL_EXPORTER_OTLP_METRICS_CERTIFICATE` |
| Headers              | Key-value pairs to be used as headers associated with gRPC or HTTP requests. See [Specifying headers](./exporter.md#specifying-headers-via-environment-variables) for more details.                   | n/a               | `OTEL_EXPORTER_OTLP_HEADERS` `OTEL_EXPORTER_OTLP_TRACES_HEADERS` `OTEL_EXPORTER_OTLP_METRICS_HEADERS` |
| Compression          | Compression key for supported compression types. Supported compression: `gzip`| No value              | `OTEL_EXPORTER_OTLP_COMPRESSION` `OTEL_EXPORTER_OTLP_TRACES_COMPRESSION` `OTEL_EXPORTER_OTLP_METRICS_COMPRESSION` |
| Timeout              | Maximum time the OTLP exporter will wait for each batch export | 10s               | `OTEL_EXPORTER_OTLP_TIMEOUT` `OTEL_EXPORTER_OTLP_TRACES_TIMEOUT` `OTEL_EXPORTER_OTLP_METRICS_TIMEOUT` |
-->

| Configuration Option | 説明                                                  | Default           | 環境変数名                                                 |
| -------------------- | ------------------------------------------------------------ | ----------------- | ------------------------------------------------------------ |
| Endpoint             | ExporterがSpanやMetricsを送信する先のターゲット。エンドポイントは、スキーム(httpまたはhttps)とホストを持つ有効なURLでなければならず(MUST)、ポートとパスを含んでもかまいません(MAY)。スキームが https の場合は、安全な接続を示します。OTLP/HTTP で `OTEL_EXPORTER_OTLP_ENDPOINT` を使用する場合、エクスポートする側は、バージョンとシグナルをパスに追加するというコレクターの慣習に従うべきです(SHOULD) (例: `v1/traces` や `v1/metrics`)。Signalごとのエンドポイント設定オプションが優先され、この動作をオーバーライドするために使用することができます。詳しくは[OTLP仕様書][otlphttp-req]を参照してください。| `https://localhost:4317` | `OTEL_EXPORTER_OTLP_ENDPOINT` `OTEL_EXPORTER_OTLP_TRACES_ENDPOINT` `OTEL_EXPORTER_OTLP_METRICS_ENDPOINT` |
| Certificate File     | gRPCクライアントのTLS認証用証明書ファイルへのパス。安全な接続のためにのみ使用してください。| n/a               | `OTEL_EXPORTER_OTLP_CERTIFICATE` `OTEL_EXPORTER_OTLP_TRACES_CERTIFICATE` `OTEL_EXPORTER_OTLP_METRICS_CERTIFICATE` |
| Headers              | gRPCやHTTPリクエストに関連するヘッダーとして使用されるキーと値のペアです。詳しくは、[Specifying headers](./exporter.md#specifying-headers-via-environment-variables)をご覧ください。        | n/a               | `OTEL_EXPORTER_OTLP_HEADERS` `OTEL_EXPORTER_OTLP_TRACES_HEADERS` `OTEL_EXPORTER_OTLP_METRICS_HEADERS` |
| Compression          | サポートされている圧縮タイプの圧縮キーです。サポートされている圧縮形式:`gzip`。| No value              | `OTEL_EXPORTER_OTLP_COMPRESSION` `OTEL_EXPORTER_OTLP_TRACES_COMPRESSION` `OTEL_EXPORTER_OTLP_METRICS_COMPRESSION` |
| Timeout              | OTLPエクスポーターが各バッチのエクスポートで待機する最大時間 | 10s               | `OTEL_EXPORTER_OTLP_TIMEOUT` `OTEL_EXPORTER_OTLP_TRACES_TIMEOUT` `OTEL_EXPORTER_OTLP_METRICS_TIMEOUT` |

<!--
Supported values for `OTEL_EXPORTER_OTLP_*COMPRESSION` options:
-->

`OTEL_EXPORTER_OTLP_*COMPRESSION`オプションでサポートされる値:

<!--
- If the value is missing, then compression is disabled.
- `gzip` is the only specified compression method for now. Other options MAY be supported by language SDKs and should be documented for each particular language.
-->

- この値がない場合は、圧縮は無効です。
- 今のところ、`gzip`が唯一の圧縮方法として指定されています。その他のオプションは言語 SDK でサポートされていても構いません(MAY)。各言語のドキュメントを参照してください。

<!--
Example 1
-->

例 1

<!--
The following configuration sends all signals to the same collector:
-->

以下の構成では、すべてのSignalが同じコレクターに送られます。

<!--
```bash
export OTEL_EXPORTER_OTLP_ENDPOINT=http://collector:4317
```
-->

```bash
export OTEL_EXPORTER_OTLP_ENDPOINT=http://collector:4317
```

<!--
Example 2
-->

例 2

<!--
Traces and metrics are sent to different collectors:
-->

TraceやMetricsは、異なるCollectorに送られます。

```bash
export OTEL_EXPORTER_OTLP_TRACES_ENDPOINT=http://collector:4317

export OTEL_EXPORTER_OTLP_METRICS_ENDPOINT=https://collector.example.com/v1/metrics
```

<!--
### Specify Protocol
-->

### プロトコルの指定

<!--
Currently, OTLP has more than one transport protocol it can support, e.g.
`grpc`,  `http/json`, `http/protobuf`.   As of 1.0 of the specification, there
*is no specified default, or configuration via environment variables*.  We
reserve the following environment variables for configuration of protocols in
the future:
-->

現在、OTLPは、`grpc`、`http/json`、`http/protobuf`など、サポートできるトランスポートプロトコルが複数あります。仕様書の1.0の時点では、*指定されたデフォルトはなく、環境変数による設定もありません*。今後のプロトコルの設定のために、以下の環境変数を予約しています。

<!--
- `OTEL_EXPORTER_OTLP_PROTOCOL`
- `OTEL_EXPORTER_OTLP_TRACES_PROTOCOL`
- `OTEL_EXPORTER_OTLP_METRICS_PROTOCOL`
-->

- `OTEL_EXPORTER_OTLP_PROTOCOL`
- `OTEL_EXPORTER_OTLP_TRACES_PROTOCOL`
- `OTEL_EXPORTER_OTLP_METRICS_PROTOCOL`

<!--
SDKs have an unspecified default, if no configuration is provided.
-->

SDKは、設定が提供されていない場合、不特定のデフォルトを持っています。

<!--
### Specifying headers via environment variables
-->

### 環境変数によるヘッダーの指定

<!--
The `OTEL_EXPORTER_OTLP_HEADERS`, `OTEL_EXPORTER_OTLP_TRACES_HEADERS`, `OTEL_EXPORTER_OTLP_METRICS_HEADERS` environment variables will contain a list of key value pairs, and these are expected to be represented in a format matching to the [W3C Correlation-Context](https://github.com/w3c/baggage/blob/master/baggage/HTTP_HEADER_FORMAT.md), except that additional semi-colon delimited metadata is not supported, i.e.: key1=value1,key2=value2. All attribute values MUST be considered strings.
-->

環境変数 `OTEL_EXPORTER_OTLP_HEADERS`, `OTEL_EXPORTER_OTLP_TRACES_HEADERS`, `OTEL_EXPORTER_OTLP_METRICS_HEADERS` には、キーバリューペアのリストが格納され、これらは [W3C Correlation-Context](https://github.com/w3c/baggage/blob/master/baggage/HTTP_HEADER_FORMAT.md)に一致するフォーマットで表現されることが期待されます(例: key1=value1,key2=value2)。ただし、追加のセミコロンで区切られたメタデータはサポートされていません。すべての属性値は文字列とみなされなければなりません(MUST)。

<!--
## Retry
-->

## リトライ

<!--
Transient errors MUST be handled with a retry strategy. This retry strategy MUST implement an exponential back-off with jitter to avoid overwhelming the destination until the network is restored or the destination has recovered.
-->

一時的なエラーは、リトライ戦略で処理しなければなりません(MUST)。このリトライ戦略は、ネットワークが復旧するまで、あるいは送信先が回復するまで、送信先に負担をかけないように、ジッターを含む指数関数的なバックオフを実装しなければなりません(MUST)。

<!--
For OTLP/HTTP, the errors `408 (Request Timeout)` and `5xx (Server Errors)` are defined as transient, detailed information about erros can be found in the [HTTP failures section](otlp.md#failures). For the OTLP/gRPC, the full list of the gRPC retryable status codes can be found in the [gRPC response section](otlp.md#otlpgrpc-response).
-->

OTLP/HTTPでは、エラー`408 (Request Timeout)`と`5xx (Server Errors)`が一時的なエラーとして定義されています。エラーに関する詳細な情報は、[HTTP failures section](otlp.md#failures)に記載されています。OTLP/gRPCでは、gRPCのリトライ可能なステータスコードの一覧は、[gRPC response section](otlp.md#otlpgrpc-response)に記載されています。

<!--
[otlphttp-req]: otlp.md#otlphttp-request
-->

[otlphttp-req]: otlp.md#otlphttp-request

