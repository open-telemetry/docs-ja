<!--
# General RPC conventions
-->

# 一般的なRPCの規約

**Status**: [Experimental](../../document-status.md)

<!--
The conventions described in this section are RPC specific. When RPC operations
occur, metric events about those operations will be generated and reported to
provide insight into those operations. By adding RPC labels to metric events
it allows for finely tuned filtering.
-->

このセクションで説明する規約は、RPC固有のものです。RPC操作が発生すると、その操作に関するメトリック・イベントが生成・報告され、その操作についての情報が提供されます。メトリック・イベントにRPCラベルを追加することで、きめ細かなフィルタリングが可能になります。

<!-- Re-generate TOC with `markdown-toc --no-first-h1 -i` -->

<!-- toc -->

<!--
- [Metric instruments](#metric-instruments)
  * [RPC Server](#rpc-server)
  * [RPC Client](#rpc-client)
- [Labels](#labels)
  * [Service name](#service-name)
- [gRPC conventions](#grpc-conventions)
-->

- [Metric instruments](#metric-instruments)
  * [RPC Server](#rpc-server)
  * [RPC Client](#rpc-client)
- [Labels](#labels)
  * [Service name](#service-name)
- [gRPC規約](#grpc規約)

<!-- tocstop -->

<!--
## Metric instruments
-->

## Metric instruments

<!--
The following metric instruments MUST be used to describe RPC operations. They
MUST be of the specified type and units.
-->

RPC操作の記述には、以下のInstrumentsを使用しなければなりません(MUST)。これらは、指定されたタイプと単位でなければなりません(MUST)。

<!--
*Note: RPC server and client metrics are split to allow correlation across client/server boundaries, e.g. Lining up an RPC method latency to determine if the server is responsible for latency the client is seeing.*
-->

*注:RPCサーバとクライアントのメトリックが分割され、クライアント/サーバの境界を越えた相関が可能になります。例えば、RPCメソッドのレイテンシーを並べて、クライアントが見ているレイテンシーの原因がサーバーにあるかどうかを判断できます。*

<!--
### RPC Server
-->

### RPC Server

<!--
Below is a table of RPC server metric instruments.
-->

以下は、RPCサーバーのInstrumentsの表です。

<!--
| Name                       | Instrument    | Units        | Description | Status | Streaming |
|----------------------------|---------------|--------------|-------------|--------|-----------|
| `rpc.server.duration`      | ValueRecorder | milliseconds | measures duration of inbound RPC | Recommended | N/A.  While streaming RPCs may record this metric as start-of-batch to end-of-batch, it's hard to interpret in practice. |
| `rpc.server.request.size`  | ValueRecorder | bytes        | measures size of RPC request messages (uncompressed) | Optional | Recorded per message in a streaming batch |
| `rpc.server.response.size` | ValueRecorder | bytes        | measures size of RPC response messages (uncompressed) | Optional | Recorded per response in a streaming batch |
| `rpc.server.requests_per_rpc` | ValueRecorder | count | measures the number of messages received per RPC.  Should be 1 for all non-streaming RPCs | Optional | Required |
| `rpc.server.responses_per_rpc` | ValueRecorder | count | measures the number of messages sent per RPC.  Should be 1 for all non-streaming RPCs | Optional | Required |
-->

| Name                       | Instrument    | 単位        | 説明 | Status | Streaming |
|----------------------------|---------------|--------------|-------------|--------|-----------|
| `rpc.server.duration`      | ValueRecorder | milliseconds | インバウンドRPCの継続期間 | Recommended | 該当なし。 ストリーミングRPCでは、この指標をバッチ開始からバッチ終了までとして記録することがありますが、実際には解釈が困難です |
| `rpc.server.request.size`  | ValueRecorder | bytes        | RPCリクエストメッセージのサイズ(無圧縮) | Optional | ストリーミングバッチでメッセージごとに記録 |
| `rpc.server.response.size` | ValueRecorder | bytes        | RPCレスポンスメッセージのサイズ(無圧縮) | Optional | ストリーミングバッチで応答ごとに記録 |
| `rpc.server.requests_per_rpc` | ValueRecorder | count | RPCごとに受信したメッセージの数。 ストリーミングを行わないRPCでは1とします。 | Optional | Required |
| `rpc.server.responses_per_rpc` | ValueRecorder | count | RPCごとに送信したメッセージの数。 ストリーミングを行わないRPCでは1とします。 | Optional | Required |

<!--
### RPC Client
-->

### RPC Client

<!--
Below is a table of RPC client metric instruments.  These apply to traditional
RPC usage, not streaming RPCs.
-->

以下は、RPCクライアントのInstrumentsの表です。 これらは、ストリーミングRPCではなく、従来のRPCの使用に適用されます。

<!--
| Name                       | Instrument    | Units        | Description | Status | Streaming |
|----------------------------|---------------|--------------|-------------|--------|-----------|
| `rpc.client.duration`      | ValueRecorder | milliseconds | measures duration of outbound RPC | Recommended | N/A.  While streaming RPCs may record this metric as start-of-batch to end-of-batch, it's hard to interpret in practice. |
| `rpc.client.request.size`  | ValueRecorder | bytes        | measures size of RPC request messages (uncompressed) | Optional | Recorded per message in a streaming batch |
| `rpc.client.response.size` | ValueRecorder | bytes        | measures size of RPC response messages (uncompressed) | Optional | Recorded per message in a streaming batch |
| `rpc.client.requests_per_rpc` | ValueRecorder | count | measures the number of messages received per RPC.  Should be 1 for all non-streaming RPCs | Optional | Required |
| `rpc.client.responses_per_rpc` | ValueRecorder | count | measures the number of messages sent per RPC.  Should be 1 for all non-streaming RPCs | Optional | Required |
-->

| Name                       | Instrument    | 単位        | 説明 | Status | Streaming |
|----------------------------|---------------|--------------|-------------|--------|-----------|
| `rpc.client.duration`      | ValueRecorder | milliseconds | アウトバウンドRPCの継続時間 | Recommended | 該当なし。 ストリーミングRPCでは、この指標をバッチ開始からバッチ終了までとして記録することがありますが、実際には解釈が困難です |
| `rpc.client.request.size`  | ValueRecorder | bytes        | RPCリクエストメッセージのサイズ(無圧縮) | Optional | ストリーミングバッチでメッセージごとに記録 |
| `rpc.client.response.size` | ValueRecorder | bytes        | RPCレスポンスメッセージのサイズ(無圧縮) | Optional | ストリーミングバッチで応答ごとに記録 |
| `rpc.client.requests_per_rpc` | ValueRecorder | count | RPCごとに受信したメッセージの数。 ストリーミングを行わないRPCでは1とします。 | Optional | Required |
| `rpc.client.responses_per_rpc` | ValueRecorder | count | RPCごとに送信したメッセージの数。 ストリーミングを行わないRPCでは1とします。 | Optional | Required |

<!--
## Labels
-->

## Labels

<!--
Below is a table of labels that SHOULD be included on metric events and whether
or not they should be on the server, client or both.
-->

以下は、メトリックイベントに含めるべきラベルと、そのラベルをサーバー、クライアント、またはその両方に付けるべきかどうかの表です。

<!-- semconv rpc -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| [`rpc.system`](../../trace/semantic_conventions/rpc.md) | string | リモーションシステムを識別する文字列 | `grpc`; `java_rmi`; `wcf` | Yes |
| [`rpc.service`](../../trace/semantic_conventions/rpc.md) | string | 呼び出されるサービスのフルネーム(該当する場合は、パッケージ名も含む) | `myservice.EchoService` | いいえ、しかし推奨です |
| [`rpc.method`](../../trace/semantic_conventions/rpc.md) | string | 呼び出されるメソッドの名前は、Span名の$method部分と等しくなければなりません。 | `exampleMethod` | いいえ、しかし推奨です |
| [`net.peer.ip`](../../trace/semantic_conventions/span-general.md) | string | 相手のリモートアドレス(IPv4ではドット10進数、IPv6では[RFC5952](https://tools.ietf.org/html/rfc5952) | `127.0.0.1` | See below |
| [`net.peer.name`](../../trace/semantic_conventions/span-general.md) | string | リモートのホスト名あるいは類似の文字列。下記注釈参照 | `example.com` | See below |
| [`net.peer.port`](../../trace/semantic_conventions/span-general.md) | int | リモートのポート番号 | `80`; `8080`; `443` | 下記参照 |
| [`net.transport`](../../trace/semantic_conventions/span-general.md) | string | Transport protocol used. 下記注釈参照。 | `ip_tcp` | 下記参照 |

**Additional attribute requirements:** At least one of the following sets of attributes is required:

* [`net.peer.ip`](../../trace/semantic_conventions/span-general.md)
* [`net.peer.name`](../../trace/semantic_conventions/span-general.md)
<!-- endsemconv -->

<!--
To avoid high cardinality, implementations should prefer the most stable of `net.peer.name` or
`net.peer.ip`, depending on expected deployment profile.  For many cloud applications, this is likely
`net.peer.name` as names can be recycled even across re-instantiation of a server with a different `ip`.
-->

カーディナリティが高くなるのを避けるために、実装では、想定されるデプロイメントプロファイルに応じて、`net.peer.name`または`net.peer.ip`のうち、最も安定したものを選択する必要があります。 多くのクラウドアプリケーションでは、おそらく `net.peer.name` でしょう。これは、異なる `ip` を持つサーバーが再起動されても、名前を再利用できるからです。

<!--
For client-side metrics `net.peer.port` is required if the connection is IP-based and the port is available (it describes the server port they are connecting to).
For server-side spans `net.peer.port` is optional (it describes the port the client is connecting from).
Furthermore, setting [net.transport][] is required for non-IP connection like named pipe bindings.
-->

クライアントサイドメトリックでは、接続がIPベースでポートが利用可能な場合、`net.peer.port`が必須となります(クライアントが接続しているサーバーのポートが記述されます)。サーバー側のSpanでは、`net.peer.port`はオプションです(クライアントが接続しているポートが記述されています)。さらに、名前付きパイプバインディングのような非IP接続の場合は、[net.transport][]の設定が必要です。

<!--
### Service name
-->

### Service name

<!--
On the server process receiving and handling the remote procedure call, the service name provided in `rpc.service` does not necessarily have to match the [`service.name`][] resource attribute.
One process can expose multiple RPC endpoints and thus have multiple RPC service names. From a deployment perspective, as expressed by the `service.*` resource attributes, it will be treated as one deployed service with one `service.name`.
-->

RPCを受信して処理するサーバプロセスでは、`rpc.service`で提供されるサービス名は必ずしも[`service.name`][]リソース属性と一致する必要はありません。1つのプロセスが複数のRPCエンドポイントを公開することで、複数のRPCサービス名を持つことができます。デプロイメントの観点からは、リソース属性の `service.*` で表されるように、1つの `service.name` を持つ1つのデプロイされたサービスとして扱われます。

<!--
[`service.name`]: ../../resource/semantic_conventions/README.md#service
-->

[`service.name`]: ../../resource/semantic_conventions/README.md#service

<!--
## gRPC conventions
-->

## gRPC規約

<!--
For remote procedure calls via [gRPC][], additional conventions are described in this section.
-->

[gRPC][]によるリモート・プロシージャ・コールでは、追加の規約がこのセクションに記載されています。

<!--
`rpc.system` MUST be set to `"grpc"`.
-->

`rpc.system` には `"grpc"` を設定しなければなりません(MUST)。

<!--
[gRPC]: https://grpc.io/
-->

[gRPC]: https://grpc.io/

