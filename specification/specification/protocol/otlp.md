<!--
# OpenTelemetry Protocol Specification
-->

# OpenTelemetryプロトコル仕様

<!--
**Status**: [Mixed](../document-status.md)
-->

**Status**: [Mixed](../document-status.md)

<!--
OpenTelemetry Protocol (OTLP) specification describes the encoding, transport,
and delivery mechanism of telemetry data between telemetry sources, intermediate
nodes such as collectors and telemetry backends.
-->

OpenTelemetry Protocol(OTLP)仕様は、テレメトリソース、コレクターなどの中間ノード、およびテレメトリバックエンド間のテレメトリデータのエンコーディング、トランスポート、および配信メカニズムを記述したものです。

<details>
<summary>
目次
</summary>

<!--
- [Signals Maturity Level](#signals-maturity-level)
- [Protocol Details](#protocol-details)
  * [OTLP/gRPC](#otlpgrpc)
    + [OTLP/gRPC Concurrent Requests](#otlpgrpc-concurrent-requests)
    + [OTLP/gRPC Response](#otlpgrpc-response)
    + [OTLP/gRPC Throttling](#otlpgrpc-throttling)
    + [OTLP/gRPC Service and Protobuf Definitions](#otlpgrpc-service-and-protobuf-definitions)
    + [OTLP/gRPC Default Port](#otlpgrpc-default-port)
  * [OTLP/HTTP](#otlphttp)
    + [OTLP/HTTP Request](#otlphttp-request)
    + [OTLP/HTTP Response](#otlphttp-response)
      - [Success](#success)
      - [Failures](#failures)
      - [Bad Data](#bad-data)
      - [OTLP/HTTP Throttling](#otlphttp-throttling)
      - [All Other Responses](#all-other-responses)
    + [OTLP/HTTP Connection](#otlphttp-connection)
    + [OTLP/HTTP Concurrent Requests](#otlphttp-concurrent-requests)
    + [OTLP/HTTP Default Port](#otlphttp-default-port)
- [Implementation Recommendations](#implementation-recommendations)
  * [Multi-Destination Exporting](#multi-destination-exporting)
- [Known Limitations](#known-limitations)
  * [Request Acknowledgements](#request-acknowledgements)
    + [Duplicate Data](#duplicate-data)
  * [Partial Success](#partial-success)
- [Future Versions and Interoperability](#future-versions-and-interoperability)
- [Glossary](#glossary)
- [References](#references)
-->

- [シグナルの成熟度](#シグナルの成熟度)
- [プロトコル詳細](#プロトコル詳細)
  * [OTLP/gRPC](#otlpgrpc)
    + [OTLP/gRPC 同時リクエスト](#otlpgrpc-同時リクエスト)
    + [OTLP/gRPC レスポンス](#otlpgrpc-レスポンス)
    + [OTLP/gRPC スロットリング](#otlpgrpc-スロットリング)
    + [OTLP/gRPC サービスとProtobufの定義](#otlpgrpc-サービスとProtobufの定義)
    + [OTLP/gRPC デフォルトポート](#otlpgrpc-default-port)
  * [OTLP/HTTP](#otlphttp)
    + [OTLP/HTTP リクエスト](#otlphttp-リクエスト)
    + [OTLP/HTTP レスポンス](#otlphttp-レスポンス)
      - [成功](#成功)
      - [失敗](#失敗)
      - [悪いデータ](#悪いデータ)
      - [OTLP/HTTP スロットリング](#otlphttp-スロットリング)
      - [All Other レスポンス](#all-other-レスポンス)
    + [OTLP/HTTP 接続](#otlphttp-接続)
    + [OTLP/HTTP 同時リクエスト](#otlphttp-同時リクエスト)
    + [OTLP/HTTP デフォルトポート](#otlphttp-デフォルトポート)
- [実装に関する推奨事項](#実装に関する推奨事項)
  * [複数の宛先へのエクスポート](#複数の宛先へのエクスポート)
- [既知の制限事項](#既知の制限事項)
  * [リクエストの確認応答](#リクエストの確認応答)
    + [重複したデータ](#重複したデータ)
  * [部分的な成功](#部分的な成功)
- [将来のバージョンと相互運用性](#将来のバージョンと相互運用性)
- [用語集](#用語集)
- [参考文献](#参考文献)

</details>

<!--
OTLP is a general-purpose telemetry data delivery protocol designed in the scope
of OpenTelemetry project.
-->

OTLPは、OpenTelemetryプロジェクトの範囲内で設計された、汎用のテレメトリデータ配信プロトコルです。

<!--
## Signals Maturity Level
-->

## シグナルの成熟度

<!--
Each signal has different support and stability in OTLP, described through its
own maturity level, which in turn applies to **all** the OTLP Transports listed below.
-->

それぞれのシグナルは、OTLPにおいて異なるサポートと安定性を持ち、それぞれの成熟度レベルによって説明されます。また、この成熟度レベルは、以下に示すOTLPトランスポートの**すべて**に適用されます。

<!--
* Tracing: **Stable**
* Metrics: **Beta**
* Logs: **Alpha**
-->

* Tracing: **Stable**
* Metrics: **Beta**
* Logs: **Alpha**

<!--
See [OTLP Maturity Level](https://github.com/open-telemetry/opentelemetry-proto#maturity-level).
-->

[OTLP成熟度](https://github.com/open-telemetry/opentelemetry-proto#maturity-level)をご参照ください。

<!--
## Protocol Details
-->

## プロトコル詳細

<!--
OTLP defines the encoding of telemetry data and the protocol used to exchange
data between the client and the server.
-->

OTLPは、テレメトリデータのエンコーディングと、クライアントとサーバー間のデータ交換に使用されるプロトコルを定義しています。

<!--
This specification defines how OTLP is implemented over
[gRPC](https://grpc.io/) and HTTP 1.1 transports and specifies
[Protocol Buffers schema](https://developers.google.com/protocol-buffers/docs/overview)
that is used for the payloads.
-->

本ドキュメントは、OTLPが[gRPC](https://grpc.io/)とHTTP 1.1のトランスポート上でどのように実装されるかを定義し、ペイロードに使用される[Protocol Buffersのスキーマ](https://developers.google.com/protocol-buffers/docs/overview)を規定しています。

<!--
OTLP is a request/response style protocols: the clients send requests, the
server replies with corresponding responses. This document defines one requests
and response type: `Export`.
-->

OTLPは、クライアントがリクエストを送信し、サーバがそれに対応するレスポンスを返す、リクエスト/レスポンス形式のプロトコルです。このドキュメントでは、1つのリクエストとレスポンスタイプを定義します: `Export`

<!--
### OTLP/gRPC
-->

### OTLP/gRPC

<!--
**Status**: [Stable](../document-status.md)
-->

**Status**: [Stable](../document-status.md)

<!--
After establishing the underlying gRPC transport the client starts sending
telemetry data using unary requests using
[Export*ServiceRequest](https://github.com/open-telemetry/opentelemetry-proto)
messages (`ExportTraceServiceRequest` for traces, `ExportMetricsServiceRequest`
for metrics, `ExportLogsServiceRequest` for logs). The client continuously sends
a sequence of requests to the server and expects to receive a response to each
request:
-->

基礎となるgRPCトランスポートを確立した後、クライアントは[Export*ServiceRequest](https://github.com/open-telemetry/opentelemetry-proto)メッセージを使ったunaryリクエストでテレメトリデータの送信を開始します(トレースは`ExportTraceServiceRequest`、メトリックは`ExportMetricsServiceRequest`、ログは`ExportLogsServiceRequest`)。クライアントは連続したリクエストをサーバーに送信し、各リクエストに対するレスポンスを受け取ることを期待します。

<!--
![Request-Response](img/otlp-request-response.png)
-->

![リクエスト-レスポンス](img/otlp-request-response.png)

<!--
_Note: this protocol is concerned with reliability of delivery between one pair
of client/server nodes and aims to ensure that no data is lost in-transit
between the client and the server. Many telemetry collection systems have
intermediary nodes that the data must travel across until reaching the final
destination (e.g. application -> agent -> collector -> backend). End-to-end
delivery guarantees in such systems is outside of the scope of OTLP. The
acknowledgements described in this protocol happen between a single
client/server pair and do not span intermediary nodes in multi-hop delivery
paths._
-->

注:このプロトコルは、一対のクライアント/サーバーノード間の配信の信頼性に関するもので、クライアントとサーバー間の移動中にデータが失われないことを目的としています。多くのテレメトリ収集システムには、最終目的地に到達するまでデータが通過しなければならない中間ノードがあります(例:アプリケーション -> エージェント -> コレクター -> バックエンド)。このようなシステムにおけるエンドツーエンドの配信保証は、OTLPの範囲外です。このプロトコルに記載されている確認応答は、単一のクライアント/サーバーペアの間で行われ、マルチホップ配信パスの中間ノードをまたぐことはありません。

<!--
#### OTLP/gRPC Concurrent Requests
-->

#### OTLP/gRPC 同時リクエスト

<!--
After sending the request the client MAY wait until the response is received
from the server. In that case there will be at most only one request in flight
that is not yet acknowledged by the server.
-->

リクエストを送信した後、クライアントはサーバーからのレスポンスを受信するまで待ってもかまいません(MAY)。その場合、サーバーからまだ承認されていない飛行中(in-flight)のリクエストは、最大でも1つだけです。

<!--
![Unary](img/otlp-sequential.png)
-->

![Unary](img/otlp-sequential.png)

<!--
Sequential operation is recommended when simplicity of implementation is
desirable and when the client and the server are connected via very low-latency
network, such as for example when the client is an instrumented application and
the server is an OpenTelemetry Collector running as a local daemon (agent).
-->

クライアントが計装済みアプリケーションで、サーバーがローカル・デーモン(Agent)として動作するOpenTelemetry Collectorである場合など、実装を簡単にしたい場合や、クライアントとサーバーが非常に低いレイテンシーのネットワークで接続されている場合には、シーケンシャルな動作が推奨されます。

<!--
The implementations that need to achieve high throughput SHOULD support
concurrent Unary calls to achieve higher throughput. The client SHOULD send new
requests without waiting for the response to the earlier sent requests,
essentially creating a pipeline of requests that are currently in flight that
are not acknowledged.
-->

高いスループットを達成する必要のある実装では、より高いスループットを達成するために、同時のUnaryコールをサポートするべきです(SHOULD)。クライアントは、以前に送信したリクエストに対するレスポンスを待たずに、新しいリクエストを送信するべきです(SHOULD)。本質的には、現在飛行中(in-flight)で確認されていないリクエストのパイプラインを作成することになります。

<!--
![Concurrent](img/otlp-concurrent.png)
-->

![同時](img/otlp-concurrent.png)

<!--
The number of concurrent requests SHOULD be configurable.
-->

同時リクエストの数は設定可能であるべきです(SHOULD)。

<!--
The maximum achievable throughput is
`max_concurrent_requests * max_request_size / (network_latency + server_response_time)`.
For example if the request can contain at most 100 spans, network roundtrip
latency is 200ms and server response time is 300 ms, then the maximum achievable
throughput with one concurrent request is `100 spans / (200ms+300ms)` or 200
spans per second. It is easy to see that in high latency networks or when the
server response time is high to achieve good throughput the requests need to be
very big or a lot concurrent requests must be done.
-->

達成可能な最大スループットは、`max_concurrent_requests * max_request_size / (network_latency + server_response_time)`となります。例えば、リクエストに含まれるSpanが最大100個で、ネットワークの往復遅延が200ms、サーバーの応答時間が300msの場合、1つの同時リクエストで達成可能な最大スループットは、`100Span / (200ms+300ms)`または1秒あたり200Spanとなります。高レイテンシーのネットワークやサーバの応答時間が長い場合、良いスループットを得るためには、リクエストが非常に大きくなるか、多くの同時リクエストを行う必要があることは容易に理解できます。

<!--
If the client is shutting down (e.g. when the containing process wants to exit)
the client will optionally wait until all pending acknowledgements are received
or until an implementation specific timeout expires. This ensures reliable
delivery of telemetry data. The client implementation SHOULD expose an option to
turn on and off the waiting during shutdown.
-->

クライアントがシャットダウンする場合(内部プロセスが終了する場合など)、クライアントは、保留中のすべての確認応答を受信するまで、または実装固有のタイムアウトが終了するまで、待機するかもしれません。これにより、テレメトリデータの確実な配信が保証されます。クライアントの実装は、シャットダウン時の待機をオン/オフする設定を公開すべきです(SHOULD)。

<!--
If the client is unable to deliver a certain request (e.g. a timer expired while
waiting for acknowledgements) the client SHOULD record the fact that the data
was not delivered.
-->

クライアントがあるリクエストを配信できなかった場合(例:確認応答を待っている間にタイマーが切れた)、クライアントはデータが配信されなかったという事実を記録すべきです(SHOULD)。

<!--
#### OTLP/gRPC Response
-->

#### OTLP/gRPC レスポンス

<!--
The server may respond with either a success or an error to the requests.
-->

サーバーは、リクエストに対して成功またはエラーのいずれかで応答します。

<!--
The success response indicates telemetry data is successfully processed by the
server. If the server receives an empty request (a request that does not carry
any telemetry data) the server SHOULD respond with success.
-->

successレスポンスは、テレメトリデータがサーバーで正常に処理されたことを示します。サーバーが空のリクエスト(テレメトリデータを持たないリクエスト)を受信した場合、サーバーはsuccessと応答するべきです(SHOULD)。

<!--
Success response is returned via
[Export*ServiceResponse](https://github.com/open-telemetry/opentelemetry-proto)
message (`ExportTraceServiceResponse` for traces, `ExportMetricsServiceResponse`
for metrics, `ExportLogsServiceResponse` for logs).
-->

成功した場合は、[Export*ServiceResponse](https://github.com/open-telemetry/opentelemetry-proto)メッセージ(Traceの場合は`ExportTraceServiceResponse`、Metricsの場合は`ExportMetricsServiceResponse`、Logの場合は`ExportLogsServiceResponse`)で返されます。

<!--
When an error is returned by the server it falls into 2 broad categories:
retryable and not-retryable:
-->

サーバーからエラーが返された場合、大きく分けて「再試行可能」と「再試行不可能」の2つのカテゴリーに分かれます。

<!--
- Retryable errors indicate that processing of telemetry data failed and the
  client SHOULD record the error and may retry exporting the same data. This can
  happen when the server is temporarily unable to process the data.
-->

- Retryableエラーは、テレメトリデータの処理に失敗したことを示します。クライアントはエラーを記録すべきであり(SHOULD)、同じデータのエクスポートを再試行することができます。これは、サーバーが一時的にデータを処理できなかった場合に発生します。

<!--
- Not-retryable errors indicate that processing of telemetry data failed and the
  client MUST NOT retry sending the same telemetry data. The telemetry data MUST
  be dropped. This can happen, for example, when the request contains bad data
  and cannot be deserialized or otherwise processed by the server. The client
  SHOULD maintain a counter of such dropped data.
-->

- Not-retryableエラーは、テレメトリデータの処理が失敗したことを示し、クライアントは同じテレメトリデータの送信を再試行してはなりません(MUST NOT)。テレメトリデータはドロップしなければなりません(MUST)。これは例えば、リクエストに不正なデータが含まれていて、サーバーでデシリアライズやその他の処理ができない場合に起こり得ます。クライアントは、そのようなドロップされたデータのカウンタを維持するべきです(SHOULD)。

<!--
The server MUST indicate retryable errors using code
[Unavailable](https://godoc.org/google.golang.org/grpc/codes) and MAY supply
additional
[details via status](https://godoc.org/google.golang.org/grpc/status#Status.WithDetails)
using
[RetryInfo](https://github.com/googleapis/googleapis/blob/6a8c7914d1b79bd832b5157a09a9332e8cbd16d4/google/rpc/error_details.proto#L40)
containing 0 value of RetryDelay. Here is a sample Go code to illustrate:
-->

サーバーは、再試行可能なエラーをコード[Unavailable](https://godoc.org/google.golang.org/grpc/codes)を使って示さなければならず(MUST)、0の値のRetryDelayを含む[RetryInfo](https://github.com/googleapis/googleapis/blob/6a8c7914d1b79bd832b5157a09a9332e8cbd16d4/google/rpc/error_details.proto#L40)を使って、追加の[details via status](https://godoc.org/google.golang.org/grpc/status#Status.WithDetails)を提供しても構いません(MAY)。ここでは、サンプルのGoコードを紹介します。

```go
  // サーバー側での実装
  st, err := status.New(codes.Unavailable, "Server is unavailable").
    WithDetails(&errdetails.RetryInfo{RetryDelay: &duration.Duration{Seconds: 0}})
  if err != nil {
    log.Fatal(err)
  }

  return st.Err()
```

<!--
To indicate not-retryable errors the server is recommended to use code
[InvalidArgument](https://godoc.org/google.golang.org/grpc/codes) and MAY supply
additional
[details via status](https://godoc.org/google.golang.org/grpc/status#Status.WithDetails)
using
[BadRequest](https://github.com/googleapis/googleapis/blob/6a8c7914d1b79bd832b5157a09a9332e8cbd16d4/google/rpc/error_details.proto#L119).
Other gRPC status code may be used if it is more appropriate. Here is a sample
Go code to illustrate:
-->

リトライできないエラーを示すために、サーバーは[InvalidArgument](https://godoc.org/google.golang.org/grpc/codes)というコードを使用することが推奨されます。また、[BadRequest](https://github.com/googleapis/googleapis/blob/6a8c7914d1b79bd832b5157a09a9332e8cbd16d4/google/rpc/error_details.proto#L119)を使用して、[details via status](https://godoc.org/google.golang.org/grpc/status#Status.WithDetails)を追加で提供しても構いません(MAY)。より適切な場合は、他のgRPCステータスコードを使用することもできます。ここでは、サンプルのGoコードを紹介します。

```go
  // サーバー側での実装
  st, err := status.New(codes.InvalidArgument, "Invalid Argument").
    WithDetails(&errdetails.BadRequest{})
  if err != nil {
    log.Fatal(err)
  }

  return st.Err()
```

<!--
The server MAY use other gRPC codes to indicate retryable and not-retryable
errors if those other gRPC codes are more appropriate for a particular erroneous
situation. The client SHOULD interpret gRPC status codes as retryable or
not-retryable according to the following table:
-->

サーバーは、特定のエラー状況に他のgRPCコードの方が適している場合、再試行可能および再試行不可能なエラーを示すために他のgRPCコードを使用しても構いません(MAY)。クライアントは、以下の表に従って、gRPCステータスコードを再試行可能または非再試行可能と解釈すべきです(SHOULD)。

<!--
|gRPC Code|Retryable?|
|---------|----------|
|CANCELLED|Yes|
|UNKNOWN|No|
|INVALID_ARGUMENT|No|
|DEADLINE_EXCEEDED|Yes|
|NOT_FOUND|No|
|ALREADY_EXISTS|No|
|PERMISSION_DENIED|No|
|UNAUTHENTICATED|No|
|RESOURCE_EXHAUSTED|Yes|
|FAILED_PRECONDITION|No|
|ABORTED|Yes|
|OUT_OF_RANGE|Yes|
|UNIMPLEMENTED|No|
|INTERNAL|No|
|UNAVAILABLE|Yes|
|DATA_LOSS|Yes|
-->

|gRPC Code|再実行可能?|
|---------|----------|
|CANCELLED|Yes|
|UNKNOWN|No|
|INVALID_ARGUMENT|No|
|DEADLINE_EXCEEDED|Yes|
|NOT_FOUND|No|
|ALREADY_EXISTS|No|
|PERMISSION_DENIED|No|
|UNAUTHENTICATED|No|
|RESOURCE_EXHAUSTED|Yes|
|FAILED_PRECONDITION|No|
|ABORTED|Yes|
|OUT_OF_RANGE|Yes|
|UNIMPLEMENTED|No|
|INTERNAL|No|
|UNAVAILABLE|Yes|
|DATA_LOSS|Yes|

<!--
When retrying, the client SHOULD implement an exponential backoff strategy. An
exception to this is the Throttling case explained below, which provides
explicit instructions about retrying interval.
-->

再試行の際、クライアントは指数関数的なバックオフ戦略を実装すべきです(SHOULD)。ただし、以下に説明するスロットリングの場合は例外で、再試行の間隔について明示的な指示があります。

<!--
#### OTLP/gRPC Throttling
-->

#### OTLP/gRPC スロットリング

<!--
OTLP allows backpressure signalling.
-->

OTLPではバックプレッシャー・シグナリングが可能です。

<!--
If the server is unable to keep up with the pace of data it receives from the
client then it SHOULD signal that fact to the client. The client MUST then
throttle itself to avoid overwhelming the server.
-->

サーバーがクライアントから受け取るデータのペースに追いつけない場合は、その事実をクライアントに通知すべきです(SHOULD)。クライアントは、サーバーに負担をかけないように自らをスロットルしなければなりません。

<!--
To signal backpressure when using gRPC transport, the server MUST return an
error with code [Unavailable](https://godoc.org/google.golang.org/grpc/codes)
and MAY supply additional
[details via status](https://godoc.org/google.golang.org/grpc/status#Status.WithDetails)
using
[RetryInfo](https://github.com/googleapis/googleapis/blob/6a8c7914d1b79bd832b5157a09a9332e8cbd16d4/google/rpc/error_details.proto#L40).
Here is a sample Go code to illustrate:
-->

gRPCトランスポートを使用しているときにバックプレッシャーを知らせるために、サーバーはコード[Unavailable](https://godoc.org/google.golang.org/grpc/codes)でエラーを返さなければならず(MUST)、[RetryInfo](https://github.com/googleapis/googleapis/blob/6a8c7914d1b79bd832b5157a09a9332e8cbd16d4/google/rpc/error_details.proto#L40)を使用して追加の[details via status](https://godoc.org/google.golang.org/grpc/status#Status.WithDetails)を提供しても構いません(MAY)s。ここでは、サンプルのGoコードを紹介します。

```go
  // サーバー側での実装
  st, err := status.New(codes.Unavailable, "Server is unavailable").
    WithDetails(&errdetails.RetryInfo{RetryDelay: &duration.Duration{Seconds: 30}})
  if err != nil {
    log.Fatal(err)
  }

  return st.Err()

  ...

  // クライアント側での実装
  st := status.Convert(err)
  for _, detail := range st.Details() {
    switch t := detail.(type) {
    case *errdetails.RetryInfo:
      if t.RetryDelay.Seconds > 0 || t.RetryDelay.Nanos > 0 {
        // リトライする前に待ちます
      }
    }
  }
```
<!--
When the client receives this signal it SHOULD follow the recommendations
outlined in documentation for
[RetryInfo](https://github.com/googleapis/googleapis/blob/6a8c7914d1b79bd832b5157a09a9332e8cbd16d4/google/rpc/error_details.proto#L40):
-->

このレスポンスを受け取ったクライアントは、[RetryInfo](https://github.com/googleapis/googleapis/blob/6a8c7914d1b79bd832b5157a09a9332e8cbd16d4/google/rpc/error_details.proto#L40)のドキュメントに記載されている推奨事項に従うべきです(SHOULD)。

<!--
```
// Describes when the clients can retry a failed request. Clients could ignore
// the recommendation here or retry when this information is missing from error
// responses.
//
// It's always recommended that clients should use exponential backoff when
// retrying.
//
// Clients should wait until `retry_delay` amount of time has passed since
// receiving the error response before retrying.  If retrying requests also
// fail, clients should use an exponential backoff scheme to gradually increase
// the delay between retries based on `retry_delay`, until either a maximum
// number of retires have been reached or a maximum retry delay cap has been
// reached.
```
-->

```
// クライアントが失敗したリクエストを再試行するタイミングを記述します。クライアントはここでの推奨事項を無視することもできますし、エラーレスポンスにこの情報が含まれていない場合に再試行することもできます。クライアントが再試行する際には、常に指数関数的なバックオフを使用することが推奨されます。クライアントは、エラーレスポンスを受信してから `retry_delay` の時間が経過するまで待ってから再試行を行うべきです。再試行のリクエストが失敗した場合、クライアントは指数バックオフ方式を使用して、最大の再試行回数に達するか、最大再試行遅延時間に達するまで、`retry_delay`に基づいて再試行間の遅延時間を徐々に増加させる必要があります。
```

<!--
The value of `retry_delay` is determined by the server and is implementation
dependant. The server SHOULD choose a `retry_delay` value that is big enough to
give the server time to recover, yet is not too big to cause the client to drop
data while it is throttled.
-->

`retry_delay`の値はサーバが決定し、実装に依存します。サーバは、サーバが回復するための時間を確保するのに十分な大きさでありながら、スロットルされている間にクライアントがデータを落とす原因にならない大きさの `retry_delay` 値を選択すべきです (SHOULD)。

<!--
#### OTLP/gRPC Service and Protobuf Definitions
-->

#### OTLP/gRPC サービスとProtobufの定義

<!--
gRPC service definitions
[are here](https://github.com/open-telemetry/opentelemetry-proto/tree/master/opentelemetry/proto/collector).
-->

gRPCサービス定義は[ここで定義されています](https://github.com/open-telemetry/opentelemetry-proto/tree/master/opentelemetry/proto/collector)。

<!--
Protobuf definitions for requests and responses
[are here](https://github.com/open-telemetry/opentelemetry-proto/tree/master/opentelemetry/proto).
-->

リクエストとレスポンスのためのProtobuf定義は[ここで定義されています](https://github.com/open-telemetry/opentelemetry-proto/tree/master/opentelemetry/proto)。

<!--
Please make sure to check the proto version and
[maturity level](https://github.com/open-telemetry/opentelemetry-proto/#maturity-level).
Schemas for different signals may be at different maturity level - some stable,
some in beta.
-->

protoのバージョンと[成熟度](https://github.com/open-telemetry/opentelemetry-proto/#maturity-level)を必ず確認してください。シグナルごとのスキーマは、安定版やベータ版など、成熟度が異なる場合があります。

<!--
#### OTLP/gRPC Default Port
-->

#### OTLP/gRPC デフォルトポート

<!--
The default network port for OTLP/gRPC is 4317.
-->

OTLP/gRPCのデフォルトのネットワークポートは4317です。

<!--
### OTLP/HTTP
-->

### OTLP/HTTP

<!--
**Binary Format Status**: [Stable](../document-status.md)
**JSON Format Status**: [Experimental](../document-status.md)
-->

**Binary Format Status**: [Stable](../document-status.md)
**JSON Format Status**: [Experimental](../document-status.md)

<!--
OTLP/HTTP uses Protobuf payloads encoded either in binary format or in JSON
format. The Protobuf schema of the messages is the same for OTLP/HTTP and
OTLP/gRPC.
-->

OTLP/HTTP は、バイナリ形式または JSON 形式でエンコードされた Protobuf ペイロードを使用します。メッセージのProtobufスキーマは、OTLP/HTTPとOTLP/gRPCで同じです。

<!--
OTLP/HTTP uses HTTP POST requests to send telemetry data from clients to
servers. Implementations MAY use HTTP/1.1 or HTTP/2 transports. Implementations
that use HTTP/2 transport SHOULD fallback to HTTP/1.1 transport if HTTP/2
connection cannot be established.
-->

OTLP/HTTP は、クライアントからサーバへテレメトリデータを送信するために HTTP POST リクエストを使用します。実装では、HTTP/1.1 または HTTP/2 トランスポートを使用しても構いません(MAY)。HTTP/2 トランスポートを使用する実装は、HTTP/2 接続が確立できない場合、HTTP/1.1 トランスポートにフォールバックするべきです (SHOULD)。

<!--
#### OTLP/HTTP Request
-->

#### OTLP/HTTP リクエスト

<!--
Telemetry data is sent via HTTP POST request. The body of the POST request is a
payload either in binary-encoded Protobuf format or in JSON-encoded Protobuf
format.
-->

テレメトリデータは、HTTP POSTリクエストで送信されます。POSTリクエストのボディは、バイナリエンコードされたProtobuf形式またはJSONエンコードされたProtobuf形式のペイロードです。

<!--
The default URL path for requests that carry trace data is `/v1/traces` (for
example the full URL when connecting to "example.com" server will be
`https://example.com/v1/traces`). The request body is a Protobuf-encoded
`ExportTraceServiceRequest` message.
-->

トレースデータを含むリクエストのデフォルトのURLパスは`/v1/traces`です(例えば、"example.com" サーバーに接続する場合の完全なURLは`https://example.com/v1/traces`になります)。リクエストボディは、Protobufでエンコードされた`ExportTraceServiceRequest`メッセージです。

<!--
The default URL path for requests that carry metric data is `/v1/metrics` and
the request body is a Protobuf-encoded `ExportMetricsServiceRequest` message.
-->

メトリックデータを扱うリクエストのデフォルトのURLパスは`/v1/metrics`で、リクエストボディはProtobufでエンコードされた`ExportMetricsServiceRequest`メッセージとなります。

<!--
The default URL path for requests that carry log data is `/v1/logs` and the
request body is a Protobuf-encoded `ExportLogsServiceRequest` message.
-->

ログデータを扱うリクエストのデフォルトのURLパスは `/v1/logs` で、リクエストボディはProtobufでエンコードされた `ExportLogsServiceRequest` メッセージです。

<!--
The client MUST set "Content-Type: application/x-protobuf" request header when
sending binary-encoded Protobuf or "Content-Type: application/json" request
header when sending JSON encoded Protobuf payload.
-->

クライアントは、バイナリエンコードされたProtobufを送信する場合は"Content-Type: application/x-protobuf"リクエストヘッダーを、JSONエンコードされたProtobufペイロードを送信する場合は"Content-Type: application/json"リクエストヘッダーを設定しなければなりません(MUST)。

<!--
The client MAY gzip the content and in that case MUST include
"Content-Encoding: gzip" request header. The client MAY include
"Accept-Encoding: gzip" request header if it can receive gzip-encoded responses.
-->

クライアントはコンテンツをgzip圧縮してもよく(MAY)、その場合は"Content-Encoding: gzip" リクエストヘッダーを含めなければなりません(MUST)。クライアントは、gzipエンコードされたレスポンスを受信できる場合、"Accept-Encoding: gzip"リクエストヘッダーを含めても構いません(MAY)。

<!--
Non-default URL paths for requests MAY be configured on the client and server
sides.
-->

リクエスト用のデフォルト以外のURLパスを、クライアント側とサーバー側で設定しても構いません(MAY)。

<!--
JSON-encoded Protobuf payloads use proto3 standard defined
[JSON Mapping](https://developers.google.com/protocol-buffers/docs/proto3#json)
for mapping between Protobuf and JSON, with one deviation from that mapping: the
`trace_id` and `span_id` byte arrays are represented as
[case-insensitive hex-encoded strings](https://tools.ietf.org/html/rfc4648#section-8),
they are not base64-encoded like it is defined in the standard
[JSON Mapping](https://developers.google.com/protocol-buffers/docs/proto3#json).
The hex encoding is used for `trace_id` and `span_id` fields in all OTLP
Protobuf messages, e.g. the `Span`, `Link`, `LogRecord`, etc. messages.
-->

JSONエンコードされたProtobufペイロードは、ProtobufとJSONの間のマッピングにproto3標準で定義された[JSONマッピング](https://developers.google.com/protocol-buffers/docs/proto3#json)を使用していますが、そのマッピングから1つだけ逸脱している点があります。`trace_id`と`span_id`のバイト配列は、[大文字小文字を区別しない16進エンコード文字列](https://tools.ietf.org/html/rfc4648#section-8)として表され、標準の[JSONマッピング](https://developers.google.com/protocol-buffers/docs/proto3#json)で定義されているようなbase64エンコードはされません。16進エンコーディングは、すべてのOTLP Protobufメッセージ(`Span`, `Link`, `LogRecord` など)の`trace_id`と`span_id`フィールドに使用されます。

<!--
Note that according to [Protobuf specs](
https://developers.google.com/protocol-buffers/docs/proto3#json) 64-bit integer
numbers in JSON-encoded payloads are encoded as decimal strings, and either
numbers or strings are accepted when decoding.
-->

[Protobuf specs](https://developers.google.com/protocol-buffers/docs/proto3#json)によると、JSONエンコードされたペイロード内の64ビット整数値は10進数の文字列としてエンコードされ、デコード時には数字か文字列のどちらかが受け入れられることに気をつけてください。

<!--
#### OTLP/HTTP Response
-->

  #### OTLP/HTTP レスポンス

<!--
Response body MUST be the appropriate serialized Protobuf message (see below for
the specific message to use in the Success and Failure cases).
-->

レスポンスのボディは、適切にシリアライズされたProtobufメッセージでなければなりません(MUST)(SuccessおよびFailureのケースで使用する特定のメッセージについては、以下を参照)。

<!--
The server MUST set "Content-Type: application/x-protobuf" header if the
response body is binary-encoded Protobuf payload. The server MUST set
"Content-Type: application/json" if the response is JSON-encoded Protobuf
payload. The server MUST use the same "Content-Type" in the response as it
received in the request.
-->

レスポンスのボディがバイナリエンコードされた Protobuf ペイロードの場合、サーバーは "Content-Type: application/x-protobuf"ヘッダーを設定しなければなりません(MUST)。レスポンスのボディが JSON エンコードされた Protobuf ペイロードである場合、サーバーは "Content-Type: application/json "を設定しなければなりません(MUST)。サーバーは、リクエストで受け取ったものと同じ"Content-Type"をレスポンスで使用しなければなりません(MUST)。

<!--
If the request header "Accept-Encoding: gzip" is present in the request the
server MAY gzip-encode the response and set "Content-Encoding: gzip" response
header.
-->

リクエストの中に"Accept-Encoding: gzip"というリクエストヘッダーがある場合、サーバーはレスポンスをgzipエンコードし、"Content-Encoding: gzip"というレスポンスヘッダーを設定しても構いません(MAY)。

<!--
##### Success
-->

##### 成功

<!--
On success the server MUST respond with `HTTP 200 OK`. Response body MUST be
Protobuf-encoded `ExportTraceServiceResponse` message for traces,
`ExportMetricsServiceResponse` message for metrics and
`ExportLogsServiceResponse` message for logs.
-->

成功した場合、サーバーは `HTTP 200 OK` で応答しなければなりません(MUST)。レスポンスのボディには、Traceについては Protobuf エンコードされた `ExportTraceServiceResponse` メッセージ、Metricsについては `ExportMetricsServiceResponse` メッセージ、Logについては `ExportLogsServiceResponse` メッセージを含めなければなりません(MUST)。

<!--
The server SHOULD respond with success no sooner than after successfully
decoding and validating the request.
-->

サーバーは、リクエストのデコードと検証に成功した後、すぐに成功を応答すべきです(SHOULD)。

<!--
##### Failures
-->

##### 失敗

<!--
If the processing of the request fails the server MUST respond with appropriate
`HTTP 4xx` or `HTTP 5xx` status code. See sections below for more details about
specific failure cases and HTTP status codes that should be used.
-->

リクエストの処理に失敗した場合、サーバは適切な `HTTP 4xx` または `HTTP 5xx` のステータスコードで応答しなければなりません (MUST)。具体的な失敗事例や、使用すべきHTTPステータスコードの詳細については、以下のセクションを参照してください。

<!--
Response body for all `HTTP 4xx` and `HTTP 5xx` responses MUST be a
Protobuf-encoded
[Status](https://godoc.org/google.golang.org/genproto/googleapis/rpc/status#Status)
message that describes the problem.
-->

すべての `HTTP 4xx` および `HTTP 5xx` レスポンスのボディは、発生した問題を説明する Protobuf エンコードされた [Status](https://godoc.org/google.golang.org/genproto/googleapis/rpc/status#Status) メッセージでなければなりません (MUST)。

<!--
This specification does not use `Status.code` field and the server MAY omit
`Status.code` field. The clients are not expected to alter their behavior based
on `Status.code` field but MAY record it for troubleshooting purposes.
-->

本仕様では `Status.code` フィールドを使用せず、サーバーは `Status.code` フィールドを省略しても構いません(MAY)。クライアントは `Status.code` フィールドに基づいて動作を変更することは期待されていませんが、トラブルシューティングのために記録しても構いません(MAY)。

<!--
The `Status.message` field SHOULD contain a developer-facing error message as
defined in `Status` message schema.
-->

`Status.message` フィールドには、`Status` メッセージスキーマで定義されている開発者向けのエラーメッセージを含めるべきです(SHOULD)。

<!--
The server MAY include `Status.details` field with additional details. Read
below about what this field can contain in each specific failure case.
-->

サーバーは追加の詳細情報として `Status.details` フィールドを含めても構いません(MAY)。それぞれの失敗事例でこのフィールドに何を含めることができるかについては、以下を参照してください。

<!--
##### Bad Data
-->

##### 悪いデータ

<!--
If the processing of the request fails because the request contains data that
cannot be decoded or is otherwise invalid and such failure is permanent then the
server MUST respond with `HTTP 400 Bad Request`. The `Status.details` field in
the response SHOULD contain a
[BadRequest](https://github.com/googleapis/googleapis/blob/d14bf59a446c14ef16e9931ebfc8e63ab549bf07/google/rpc/error_details.proto#L166)
that describes the bad data.
-->

リクエストにデコードできないデータや無効なデータが含まれているためにリクエストの処理が失敗し、その失敗が永続する場合、サーバーは `HTTP 400 Bad Request` で応答しなければなりません(MUST)。レスポンスの `Status.details` フィールドには、不正なデータを記述した [BadRequest](https://github.com/googleapis/googleapis/blob/d14bf59a446c14ef16e9931ebfc8e63ab549bf07/google/rpc/error_details.proto#L166) を含めるべきです (SHOULD)。

<!--
The client MUST NOT retry the request when it receives `HTTP 400 Bad Request`
response.
-->

クライアントは、`HTTP 400 Bad Request` レスポンスを受け取ったときに、リクエストを再試行してはいけません (MUST NOT)。

<!--
##### OTLP/HTTP Throttling
-->

##### OTLP/HTTP スロットリング

<!--
If the server receives more requests than the client is allowed or the server is
overloaded the server SHOULD respond with `HTTP 429 Too Many Requests` or
`HTTP 503 Service Unavailable` and MAY include
["Retry-After"](https://tools.ietf.org/html/rfc7231#section-7.1.3) header with a
recommended time interval in seconds to wait before retrying.
-->

クライアントに許可した以上のリクエストをサーバーが受信した場合や、サーバーが過負荷状態になった場合、サーバーは `HTTP 429 Too Many Requests` または `HTTP 503 Service Unavailable` で応答すべき(SHOULD)であり、再試行するまでの推奨時間を秒単位で示す ["Retry-After"](https://tools.ietf.org/html/rfc7231#section-7.1.3) ヘッダーを含めても構いません(MAY)。

<!--
The client SHOULD honour the waiting interval specified in "Retry-After" header
if it is present. If the client receives `HTTP 429` or `HTTP 503` response and
"Retry-After" header is not present in the response then the client SHOULD
implement an exponential backoff strategy between retries.
-->

クライアントは、"Retry-After"ヘッダで指定された待ち時間が存在する場合には、それに従うべきです(SHOULD)。クライアントが `HTTP 429` または `HTTP 503` レスポンスを受信し、そのレスポンスに "Retry-After" ヘッダが存在しない場合、クライアントは再試行の間に指数関数的なバックオフ戦略を実装するべきです (SHOULD)。

<!--
##### All Other Responses
-->

##### その他のレスポンス

<!--
All other HTTP responses that are not explicitly listed in this document should
be treated according to HTTP specification.
-->

このドキュメントに明示的に記載されていない他のすべてのHTTPレスポンスは、HTTPの仕様に従って扱われるべきです。

<!--
If the server disconnects without returning a response the client SHOULD retry
and send the same request. The client SHOULD implement an exponential backoff
strategy between retries to avoid overwhelming the server.
-->

サーバーが応答を返さずに切断した場合、クライアントは再試行して同じリクエストを送信すべきです(SHOULD)。クライアントは、サーバーに負担をかけないように、再試行の間に指数関数的なバックオフ戦略を実装すべきです(SHOULD)。

<!--
#### OTLP/HTTP Connection
-->

#### OTLP/HTTP 接続

<!--
If the client is unable to connect to the server the client SHOULD retry the
connection using exponential backoff strategy between retries. The interval
between retries must have a random jitter.
-->

クライアントがサーバーに接続できない場合、クライアントは再試行の間に指数関数的バックオフ戦略を使用して接続を再試行すべきです(SHOULD)。再試行の間隔は、ランダムなジッターを持たなければなりません。

<!--
The client SHOULD keep the connection alive between requests.
-->

クライアントは、リクエスト間で接続を維持すべきです(SHOULD)。

<!--
Server implementations SHOULD accept OTLP/HTTP with binary-encoded Protobuf
payload and OTLP/HTTP with JSON-encoded Protobuf payload requests on the same
port and multiplex the requests to the corresponding payload decoder based on
the "Content-Type" request header.
-->

サーバの実装は、バイナリエンコードされたProtobufペイロードを持つOTLP/HTTPと、JSONエンコードされたProtobufペイロードを持つOTLP/HTTPのリクエストを同じポートで受け入れ、"Content-Type"リクエストヘッダに基づいて、対応するペイロードデコーダにリクエストを多重化(multiplex)するべきです(SHOULD)。

<!--
Server implementations MAY accept OTLP/gRPC and OTLP/HTTP requests on the same
port and multiplex the connections to the corresponding transport handler based
on the "Content-Type" request header.
-->

サーバの実装では、同じポートでOTLP/gRPCとOTLP/HTTPのリクエストを受け付け、"Content-Type"リクエストヘッダに基づいて、対応するトランスポートハンドラへの接続を多重化しても構いません。

<!--
#### OTLP/HTTP Concurrent Requests
-->

#### OTLP/HTTP 同時リクエスト

<!--
To achieve higher total throughput the client MAY send requests using several
parallel HTTP connections. In that case the maximum number of parallel
connections SHOULD be configurable.
-->

より高いトータルスループットを実現するために、クライアントは複数の並列HTTPコネクションを使用してリクエストを送信しても構いません(MAY)。その場合、並列接続の最大数は設定可能であるべきです(SHOULD)。

<!--
#### OTLP/HTTP Default Port
-->

#### OTLP/HTTP デフォルトポート

<!--
The default network port for OTLP/HTTP is 4317.
-->

OTLP/HTTPのデフォルトのネットワークポートは4317です。

<!--
## Implementation Recommendations
-->

## 実装に関する推奨事項

<!--
### Multi-Destination Exporting
-->

### 複数の宛先へのエクスポート

<!--
When the telemetry data from one client must be sent to more than one
destination server there is an additional complication that must be accounted
for. When one of the servers acknowledges the data and the other server does not
(yet) acknowledges the client needs to make a decision about how to move
forward.
-->

1つのクライアントからのテレメトリデータを複数の送信先サーバーに送信しなければならない場合、さらに複雑な問題を考慮しなければなりません。一方のサーバーがデータを確認し、他方のサーバーが(まだ)確認していない場合、クライアントはどのように処理を進めるべきかを決定する必要があります。

<!--
In such situation the the client SHOULD implement queuing, acknowledgement
handling and retrying logic per destination. This ensures that servers do not
block each other. The queues SHOULD reference shared, immutable data to be sent,
thus minimizing the memory overhead caused by having multiple queues.
-->

このような場合、クライアントはデスティネーションごとにキューイング、確認応答の処理、リトライのロジックを実装すべきです(SHOULD)。これにより、サーバ同士が互いにブロックしないことが保証されます。キューは、送信される共有された不変のデータを参照するべきであり(SHOULD)、その結果、複数のキューを持つことによるメモリのオーバーヘッドを最小限に抑えることができます。

<!--
![Multi-Destination Exporting](img/otlp-multi-destination.png)
-->

![複数の宛先へのエクスポート](img/otlp-multi-destination.png)

<!--
This ensures that all destination servers receive the data regardless of their
speed of reception (within the available limits imposed by the size of the
client-side queue).
-->

これにより、すべての宛先サーバーが、受信速度に関係なく(クライアント側のキューのサイズによって課せられる利用可能な制限の範囲内で)データを受け取ることができます。

<!--
## Known Limitations
-->

## 既知の制限事項

<!--
### Request Acknowledgements
-->

### リクエストの確認応答

<!--
#### Duplicate Data
-->

#### 重複したデータ

<!--
In edge cases (e.g. on reconnections, network interruptions, etc) the client has
no way of knowing if recently sent data was delivered if no acknowledgement was
received yet. The client will typically choose to re-send such data to guarantee
delivery, which may result in duplicate data on the server side. This is a
deliberate choice and is considered to be the right tradeoff for telemetry data.
-->

エッジケース(再接続時、ネットワークの中断時など)では、クライアントは、確認応答がまだ受信されていない場合、最近送信されたデータが配信されたかどうかを知る方法がありません。クライアントは通常、配信を保証するためにそのようなデータを再送信することを選択しますが、その結果、サーバー側でデータが重複することがあります。これは意図的な選択であり、テレメトリデータにとっては適切なトレードオフであると考えられます。

<!--
### Partial Success
-->

### 部分的な成功

<!--
The protocol does not attempt to communicate partial reception success from the
server to the client (i.e. when part of the data can be received by the server
and part of it cannot). Attempting to do so would complicate the protocol and
implementations significantly and is left out as a possible future area of work.
-->

本プロトコルでは、サーバからクライアントへの部分的な受信成功の伝達は試みていません(つまり、データの一部がサーバで受信でき、一部が受信できない場合)。このような通信を行うと、プロトコルや実装が著しく複雑になるため、将来の課題として残されています。

<!--
## Future Versions and Interoperability
-->

## 将来のバージョンと相互運用性

<!--
OTLP will evolve and change over time. Future versions of OTLP must be designed
and implemented in a way that ensures that clients and servers that implement
different versions of OTLP can interoperate and exchange telemetry data. Old
clients must be able to talk to new servers and vice versa. If new versions of
OTLP introduce new functionality that cannot be understood and supported by
nodes implementing the old versions of OTLP the protocol must regress to the
lowest common denominator from functional perspective.
-->

OTLPは時間とともに進化し、変化していきます。OTLPの将来のバージョンは、OTLPの異なるバージョンを実装しているクライアントとサーバーが相互運用し、テレメトリデータを交換できることを保証する方法で設計・実装されなければなりません。古いクライアントは新しいサーバと通信できなければなりませんし、その逆もまた然りです。もしOTLPの新バージョンが、旧バージョンのOTLPを実装しているノードでは理解・サポートできない新機能を導入した場合、プロトコルは機能的観点から最小公倍数に回帰しなければなりません。

<!--
When possible the interoperability MUST be ensured between all versions of
OTLP that are not declared obsolete.
-->

可能な限り、廃止宣言されていないOTLPのすべてのバージョン間で相互運用性を確保しなければなりません(MUST)。

<!--
OTLP does not use explicit protocol version numbering. OTLP's interoperability
of clients and servers of different versions is based on the following concepts:
-->

OTLPでは、明示的なプロトコルのバージョン番号を使用していません。OTLPの異なるバージョンのクライアントやサーバの相互運用性は、以下の概念に基づいています。

<!--
1. OTLP (current and future versions) defines a set of capabilities, some of
   which are mandatory, others are optional. Clients and servers must implement
   mandatory capabilities and can choose implement only a subset of optional
   capabilities.
-->

1. OTLP(現在および将来のバージョン)では、一連の機能を定義していますが、その中には必須のものと任意のものがあります。クライアントとサーバーは、必須の機能を実装する必要があり、任意の機能のサブセットから一部のみを選択して実装することができます。

<!--
2. For minor changes to the protocol future versions and extension of OTLP are
   encouraged to use the ability of Protobufs to evolve message schema in
   backwards compatible manner. Newer versions of OTLP may add new fields to
   messages that will be ignored by clients and servers that do not understand
   these fields. In many cases careful design of such schema changes and correct
   choice of default values for new fields is enough to ensure interoperability
   of different versions without nodes explicitly detecting that their peer node
   has different capabilities.
-->

2. プロトコルのマイナーチェンジについては、OTLPの将来のバージョンと拡張は、下位互換性のある方法でメッセージスキーマを進化させるProtobufの機能を使用することが推奨されます。OTLPの新しいバージョンでは、メッセージに新しいフィールドを追加することがありますが、これらのフィールドを理解しないクライアントやサーバーでは無視されます。多くの場合、このようなスキーマの変更を慎重に設計し、新しいフィールドのデフォルト値を正しく選択することは、ノードが相手のノードが異なる能力を持っていることを明示的に検出することなく、異なるバージョンの相互運用性を確保するのに十分です。

<!--
3. More significant changes must be explicitly defined as new optional
   capabilities in future OTEPs. Such capabilities SHOULD be discovered by
   client and server implementations after establishing the underlying
   transport. The exact discovery mechanism SHOULD be described in future OTEPs
   which define the new capabilities and typically can be implemented by making
   a discovery request/response message exchange from the client to server. The
   mandatory capabilities defined by this specification are implied and do not
   require a discovery. The implementation which supports a new, optional
   capability MUST adjust its behavior to match the expectation of a peer that
   does not support a particular capability.
-->

3. より重要な変更は、将来のOTEPで新しいオプション機能として明示的に定義しなければなりません。そのような能力は、基礎となるトランスポートを確立した後に、クライアントとサーバーの実装によって発見されるべきです(SHOULD)。正確な発見メカニズムは、新しい能力を定義する将来のOTEPに記述されるべき(SHOULD)であり、通常、クライアントからサーバーへの発見要求/応答メッセージ交換を行うことで実装できます。本仕様で定義された必須機能は暗黙の了解であり、発見を必要としません。新しい任意の機能をサポートする実装は、特定の機能をサポートしていない接続対象の期待に合うように、その動作を調整しなければなりません(MUST)。

<!--
## Glossary
-->

## 用語集

<!--
There are 2 parties involved in telemetry data exchange. In this document the
party that is the source of telemetry data is called the `Client`, the party
that is the destination of telemetry data is called the `Server`.
-->

テレメトリデータの交換には2つの当事者が存在します。このドキュメントでは、テレメトリデータの送信元となる側を「クライアント」と呼び、テレメトリデータの送信先対象となる側を「サーバー」と呼びます。

<!--
![Client-Server](img/otlp-client-server.png)
-->

![クライアント・サーバー](img/otlp-client-server.png)

<!--
Examples of a Client are instrumented applications or sending side of telemetry
collectors, examples of Servers are telemetry backends or receiving side of
telemetry collectors (so a Collector is typically both a Client and a Server
depending on which side you look from).
-->

クライアントの例としては、計装済みアプリケーションやテレメトリコレクターの送信側が挙げられ、サーバーの例としては、テレメトリ・バックエンドやテレメトリ・コレクターの受信側が挙げられます(したがって、コレクターは、どちらの側から見るかによって、一般的にクライアントとサーバーの両方になります)。

<!--
Both the Client and the Server are also a `Node`. This term is used in the
document when referring to either one.
-->

クライアントもサーバーも「Node」です。本ドキュメントでは、どちらか一方を指すときにこの用語を使用しています。

<!--
## References
-->

## 参考文献

<!--
- [OTEP 0035](https://github.com/open-telemetry/oteps/blob/main/text/0035-opentelemetry-protocol.md) OpenTelemetry Protocol Specification
- [OTEP 0099](https://github.com/open-telemetry/oteps/blob/main/text/0099-otlp-http.md) OTLP/HTTP: HTTP Transport Extension for OTLP
- [OTEP 0122](https://github.com/open-telemetry/oteps/blob/main/text/0122-otlp-http-json.md) OTLP: JSON Encoding for OTLP/HTTP
-->

- [OTEP 0035](https://github.com/open-telemetry/oteps/blob/main/text/0035-opentelemetry-protocol.md) OpenTelemetry Protocol Specification
- [OTEP 0099](https://github.com/open-telemetry/oteps/blob/main/text/0099-otlp-http.md) OTLP/HTTP: HTTP Transport Extension for OTLP
- [OTEP 0122](https://github.com/open-telemetry/oteps/blob/main/text/0122-otlp-http-json.md) OTLP: JSON Encoding for OTLP/HTTP

