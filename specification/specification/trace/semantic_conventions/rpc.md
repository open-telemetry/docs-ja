<!--
# Semantic conventions for RPC spans
-->

# RPC Spanのセマンティック規約

**Status**: [Experimental](../../document-status.md)

<!--
This document defines how to describe remote procedure calls
(also called "remote method invocations" / "RMI") with spans.
-->

このドキュメントでは、リモートプロシージャコール(「リモートメソッド呼び出し」/「RMI」とも呼ばれる)をSpanで記述する方法を定義しています。

<!-- Re-generate TOC with `markdown-toc --no-first-h1 -i` -->

<!-- toc -->

<!--
- [Common remote procedure call conventions](#common-remote-procedure-call-conventions)
  * [Span name](#span-name)
  * [Attributes](#attributes)
    + [Service name](#service-name)
  * [Distinction from HTTP spans](#distinction-from-http-spans)
- [gRPC](#grpc)
  * [gRPC Attributes](#grpc-attributes)
  * [gRPC Status](#grpc-status)
  * [Events](#events)
-->

- [一般的なリモートプロシージャコールの規約](#一般的なリモートプロシージャコールの規約)
  * [Span名](#span名)
  * [属性](#属性)
    + [Service名](#service名)
  * [HTTP Spanとの差別化](#HTTP-Spanとの差別化)
- [gRPC](#grpc)
  * [gRPC属性](#grpc属性)
  * [gRPC Status](#grpc-status)
  * [Events](#events)


<!-- tocstop -->

<!--
## Common remote procedure call conventions
-->

## 一般的なリモートプロシージャコールの規約

<!--
A remote procedure calls is described by two separate spans, one on the client-side and one on the server-side.
-->

リモートプロシージャコールは、クライアント側とサーバー側の2つの独立したSpanで記述されます。

<!--
For outgoing requests, the `SpanKind` MUST be set to `CLIENT` and for incoming requests to `SERVER`.
-->

発信するリクエストでは、`SpanKind`には`CLIENT`を、着信するリクエストでは`SERVER`を設定しなければなりません(MUST)。

<!--
Remote procedure calls can only be represented with these semantic conventions, when the names of the called service and method are known and available.
-->

リモート・プロシージャ・コールは、呼び出されたサービスとメソッドの名前がわかっていて利用可能な場合にのみ、これらのセマンティック規約で表現することができます。

<!--
### Span name
-->

### Span name

<!--
The _span name_ MUST be the full RPC method name formatted as:
-->

_span name_は、以下のようにフォーマットされた完全なRPCメソッド名でなければなりません(MUST)。

```
$package.$service/$method
```

<!--
(where $service MUST NOT contain dots and $method MUST NOT contain slashes)
-->

(ここで、$serviceにはドットを含めてはならず、$methodにはスラッシュを含めてはなりません(MUST NOT))。

<!--
If there is no package name or if it is unknown, the `$package.` part (including the period) is omitted.
-->

パッケージ名がない場合や不明な場合は、`$package.`の部分(ピリオドを含む)が省略されます。

<!--
Examples of span names:
-->

Span名の例:

<!--
- `grpc.test.EchoService/Echo`
- `com.example.ExampleRmiService/exampleMethod`
- `MyCalcService.Calculator/Add` reported by the server and
  `MyServiceReference.ICalculator/Add` reported by the client for .NET WCF calls
- `MyServiceWithNoPackage/theMethod`
-->

- `grpc.test.EchoService/Echo`
- `com.example.ExampleRmiService/exampleMethod`
- .NETのWCFコールにおいて、サーバーから報告される`MyCalcService.Calculator/Add`と、クライアントから報告される`MyServiceReference.ICalculator/Add`の2種類があります。
- `MyServiceWithNoPackage/theMethod`


### 属性

<!-- semconv rpc -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `rpc.system` | string | リモーションシステムを識別する文字列 | `grpc`; `java_rmi`; `wcf` | Yes |
| `rpc.service` | string | 呼び出されるサービスのフルネーム(該当する場合は、パッケージ名も含む) | `myservice.EchoService` | いいえ、しかし推奨です |
| `rpc.method` | string | 呼び出されるメソッドの名前は、Span名の$method部分と等しくなければなりません。 | `exampleMethod` | いいえ、しかし推奨です |
| [`net.peer.ip`](span-general.md) | string | 相手のリモートアドレス(IPv4ではドット10進数、IPv6では[RFC5952](https://tools.ietf.org/html/rfc5952) | `127.0.0.1` | See below |
| [`net.peer.name`](span-general.md) | string | リモートのホスト名あるいは類似の文字列。下記注釈参照 | `example.com` | See below |
| [`net.peer.port`](span-general.md) | int | リモートのポート番号 | `80`; `8080`; `443` | 下記参照 |
| [`net.transport`](span-general.md) | string | Transport protocol used. 下記注釈参照。 | `ip_tcp` | 下記参照 |

**Additional attribute requirements:** At least one of the following sets of attributes is required:

* [`net.peer.ip`](span-general.md)
* [`net.peer.name`](span-general.md)
<!-- endsemconv -->

<!--
For client-side spans `net.peer.port` is required if the connection is IP-based and the port is available (it describes the server port they are connecting to).
For server-side spans `net.peer.port` is optional (it describes the port the client is connecting from).
Furthermore, setting [net.transport][] is required for non-IP connection like named pipe bindings.
-->

クライアント側のSpanでは、接続がIPベースでポートが利用可能な場合、`net.peer.port`が必要となります(接続先のサーバーのポートが記載されています)。

サーバーサイドのSpanでは、`net.peer.port`はオプションです(クライアントが接続するポートを記述します)。さらに、名前付きパイプバインディングのような非IP接続の場合は、[net.transport][]の設定が必要です。

<!--
#### Service name
-->

#### Service 名

<!--
On the server process receiving and handling the remote procedure call, the service name provided in `rpc.service` does not necessarily have to match the [`service.name`][] resource attribute.
One process can expose multiple RPC endpoints and thus have multiple RPC service names. From a deployment perspective, as expressed by the `service.*` resource attributes, it will be treated as one deployed service with one `service.name`.
Likewise, on clients sending RPC requests to a server, the service name provided in `rpc.service` does not have to match the [`peer.service`][] span attribute.
-->

リモートプロシージャコールを受信して処理するサーバプロセスでは、`rpc.service`で提供されるサービス名は必ずしも[`service.name`][]リソース属性と一致する必要はありません。1つのプロセスが複数のRPCエンドポイントを公開することで、複数のRPCサービス名を持つことができます。しかし、デプロイメントの観点からは、リソース属性の `service.*` で表されるように、1つの `service.name` を持つ1つのデプロイされたサービスとして扱われます。同様に、サーバーにRPCリクエストを送信するクライアントでは、`rpc.service`で提供されるサービス名は、[`peer.service`][] span属性と一致する必要はありません。

<!--
As an example, given a process deployed as `QuoteService`, this would be the name that goes into the `service.name` resource attribute which applies to the entire process.
This process could expose two RPC endpoints, one called `CurrencyQuotes` (= `rpc.service`) with a method called `getMeanRate` (= `rpc.method`) and the other endpoint called `StockQuotes`  (= `rpc.service`) with two methods `getCurrentBid` and `getLastClose` (= `rpc.method`).
In this example, spans representing client request should have their `peer.service` attribute set to `QuoteService` as well to match the server's `service.name` resource attribute.
Generally, a user SHOULD NOT set `peer.service` to a fully qualified RPC service name.
-->

例えば、`QuoteService`という名前でデプロイされたプロセスがあるとすると、これはプロセス全体に適用される`service.name`リソース属性に入る名前になります。このプロセスは、`CurrencyQuotes` (= `rpc.service`) という名前で、`getMeanRate` (= `rpc.method`) というメソッドを持つ2つのRPCエンドポイントを公開することができます。また、`StockQuotes` (= `rpc.service`) という名前で、`getCurrentBid` と `getLastClose` (= `rpc.method`) という2つのメソッドを持つエンドポイントを公開することができます。この例では、クライアントのリクエストを表すSpanは、サーバーの `service.name` リソース属性に合わせて、`peer.service` 属性を `QuoteService` に設定する必要があります。一般的に、ユーザーは `peer.service` に完全に修飾された RPC サービス名を設定すべきではありません (SHOULD NOT)。

<!--
[network attributes]: span-general.md#general-network-connection-attributes
[net.transport]: span-general.md#nettransport-attribute
[`service.name`]: ../../resource/semantic_conventions/README.md#service
[`peer.service`]: span-general.md#general-remote-service-attributes
-->

[network attributes]: span-general.md#general-network-connection-attributes
[net.transport]: span-general.md#nettransport-attribute
[`service.name`]: ../../resource/semantic_conventions/README.md#service
[`peer.service`]: span-general.md#general-remote-service-attributes

<!--
### Distinction from HTTP spans
-->

### HTTP Spanとの差別化

<!--
HTTP calls can generally be represented using just [HTTP spans](./http.md).
If they address a particular remote service and method known to the caller, i.e., when it is a remote procedure call transported over HTTP, the `rpc.*` attributes might be added additionally on that span, or in a separate RPC span that is a parent of the transporting HTTP call.
Note that _method_ in this context is about the called remote procedure and _not_ the HTTP verb (GET, POST, etc.).
-->

HTTPコールは、一般的に[HTTPSpan](./http.md)だけで表現できます。呼び出し側が知っている特定のリモートサービスやメソッドを扱う場合、つまり、HTTPで転送されるリモートプロシージャコールの場合は、`rpc.*`属性をそのSpanに追加したり、HTTPコールを転送する親である別のRPC Spanに追加したりすることができます。この文脈における _method_ は、呼び出されたリモートプロシージャに関するものであり、HTTP動詞(GET、POSTなど)に関するもの _ではない_ ことに注意してください。

<!--
## gRPC
-->

## gRPC

<!--
For remote procedure calls via [gRPC][], additional conventions are described in this section.
-->

[gRPC][]によるリモート・プロシージャ・コールに対しては、追加の規約がこのセクションに記載されています。

<!--
`rpc.system` MUST be set to `"grpc"`.
-->

`rpc.system` には `"grpc"` を設定しなければなりません(MUST)。

<!--
### gRPC Attributes
-->

### gRPC属性

<!-- semconv rpc.grpc -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `rpc.grpc.status_code` | int | gRPCリクエストの[numeric status code](https://github.com/grpc/grpc/blob/v1.33.2/doc/statuscodes.md)です | `0`; `1`; `16` | Yes |

`rpc.grpc.status_code` MUST be one of the following:

| Value  | Description |
|---|---|
| `0` | OK |
| `1` | CANCELLED |
| `2` | UNKNOWN |
| `3` | INVALID_ARGUMENT |
| `4` | DEADLINE_EXCEEDED |
| `5` | NOT_FOUND |
| `6` | ALREADY_EXISTS |
| `7` | PERMISSION_DENIED |
| `8` | RESOURCE_EXHAUSTED |
| `9` | FAILED_PRECONDITION |
| `10` | ABORTED |
| `11` | OUT_OF_RANGE |
| `12` | UNIMPLEMENTED |
| `13` | INTERNAL |
| `14` | UNAVAILABLE |
| `15` | DATA_LOSS |
| `16` | UNAUTHENTICATED |
<!-- endsemconv -->

[gRPC]: https://grpc.io/

<!--
### gRPC Status
-->

### gRPC Status

<!--
The [Span Status](../api.md#set-status) MUST be left unset for an `OK` gRPC status code, and set to `Error` for all others.
-->

[Span Status](../api.md#set-status)は、`OK`のgRPCステータスコードの場合は未設定にし、それ以外の場合は`Error`に設定しなければなりません(MUST)。

<!--
### Events
-->

### Events

<!--
In the lifetime of a gRPC stream, an event for each message sent/received on
client and server spans SHOULD be created with the following attributes:
-->

gRPC streamのライフタイムにおいて、クライアントとサーバーのSpanで送受信される各メッセージのイベントは、以下の属性で作成されるべきです(SHOULD):


```
-> [time],
    "name" = "message",
    "message.type" = "SENT",
    "message.id" = id
    "message.compressed_size" = <compressed size in bytes>,
    "message.uncompressed_size" = <uncompressed size in bytes>
```

```
-> [time],
    "name" = "message",
    "message.type" = "RECEIVED",
    "message.id" = id
    "message.compressed_size" = <compressed size in bytes>,
    "message.uncompressed_size" = <uncompressed size in bytes>
```

<!--
The `message.id` MUST be calculated as two different counters starting from `1`
one for sent messages and one for received message. This way we guarantee that
the values will be consistent between different implementations. In case of
unary calls only one sent and one received message will be recorded for both
client and server spans.
-->

`message.id` は、`1` から始まる 2つの異なるカウンターとして計算しなければなりません (MUST)。このようにして、異なる実装間で値の一貫性が保たれることを保証します。unary callの場合、クライアントとサーバーの両方のSpanで、1つの送信メッセージと1つの受信メッセージのみが記録されます。