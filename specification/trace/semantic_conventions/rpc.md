<!--
# Semantic conventions for RPC spans
-->

# RPC Spanのセマンティック規約

<!--
This document defines how to describe remote procedure calls
(also called "remote method invocations" / "RMI") with spans.
-->

このドキュメントでは、リモートプロシージャコール(リモートメソッド呼び出し/RMIとも呼ばれる)をSpanで記述する方法を定義しています。

<!-- Re-generate TOC with `markdown-toc --no-first-h1 -i` -->

<!-- toc -->

<!--
- [gRPC](#grpc)
  * [Attributes](#attributes)
  * [Status](#status)
  * [Events](#events)
-->

- [gRPC](#grpc)
  * [属性](#属性)
  * [ステータス](#ステータス)
  * [イベント](#イベント)

<!-- tocstop -->

<!--
## gRPC
-->

## gRPC

<!--
Implementations MUST create a span, when the gRPC call starts, one for
client-side and one for server-side. Outgoing requests should be a span `kind`
of `CLIENT` and incoming requests should be a span `kind` of `SERVER`.
-->

実装では、gRPC呼び出しの開始時に、クライアント側とサーバ側のSpanを作成しなければなりません(MUST)。外部へのリクエストは `CLIENT` の `kind` というSpanで、外部からのリクエストは `SERVER` の `kind` というSpanでなければなりません。

<!--
Span `name` MUST be full gRPC method name formatted as:
-->

Span `name` は完全な gRPC メソッド名でなければなりません (MUST):

<!--
```
$package.$service/$method
```
-->

```
$package.$service/$method
```

<!--
Examples of span name: `grpc.test.EchoService/Echo`.
-->

Span 名の例: `grpc.test.EchoService/Echo`

<!--
### Attributes
-->

### 属性

<!--
| Attribute name | Notes and examples                                           | Required? |
| -------------- | ------------------------------------------------------------ | --------- |
| `rpc.service`  | The service name, must be equal to the $service part in the span name. | Yes |
| `net.peer.ip`  | See [network attributes][]. | See below |
| `net.peer.name`  | See [network attributes][]. | See below |
| `net.peer.port`  | See [network attributes][]. | See below |
-->

| 属性名 | 説明と例                                       | Required? |
| -------------- | ------------------------------------------------------------ | --------- |
| `rpc.service`  | サービス名です。Span名の中の $service の部分と同じでなければなりません。| Yes |
| `net.peer.ip`  | [ネットワーク属性][]を参照 | 以下を参照 |
| `net.peer.name`  | [ネットワーク属性][]を参照 | 以下を参照 |
| `net.peer.port`  | [ネットワーク属性][]を参照 | 以下を参照 |

<!--
At least one of [network attributes][] `net.peer.name` or `net.peer.ip` is required.
For client-side spans `net.peer.port` is required (it describes the server port they are connecting to).
For server-side spans `net.peer.port` is optional (it describes the port the client is connecting from).
-->

[ネットワーク属性][]、`net.peer.name` または `net.peer.ip` のうち少なくとも一つが必要です。クライアント側のSpanでは `net.peer.port` が必須です(接続先のサーバのポートを記述する)。サーバ側のSpanでは `net.peer.port` はオプションです (クライアントが接続しているポートを記述します)。

<!--
[network attributes]: span-general.md#general-network-connection-attributes
-->

[ネットワーク属性]: span-general.md#general-network-connection-attributes

<!--
### Status
-->

### ステータス

<!--
Implementations MUST set status which MUST be the same as the gRPC client/server
status. The mapping between gRPC canonical codes and OpenTelemetry status codes
is 1:1 as OpenTelemetry canonical codes is just a snapshot of grpc codes which
can be found [here](https://github.com/grpc/grpc-go/blob/master/codes/codes.go).
-->

実装では、gRPC クライアント/サーバのステータスと同じステータスでなければなりません(MUST)。gRPCの正規コードとOpenTelemetryのステータスコードの間のマッピングは1:1で、OpenTelemetryの正規コードは、[ここ] (https://github.com/grpc/grpc-go/blob/master/codes/codes.go)で見つけられるgrpcのコードのスナップショットにすぎません。

<!--
### Events
-->

### イベント

<!--
In the lifetime of a gRPC stream, an event for each message sent/received on
client and server spans SHOULD be created with the following attributes:
-->

gRPCストリームの有効期間中、クライアントとサーバのSpanで送受信される各メッセージのイベントは、以下の属性で作成されるべきです(SHOULD)。

<!--
```
-> [time],
    "name" = "message",
    "message.type" = "SENT",
    "message.id" = id
    "message.compressed_size" = <compressed size in bytes>,
    "message.uncompressed_size" = <uncompressed size in bytes>
```
-->

```
-> [time],
    "name" = "message",
    "message.type" = "SENT",
    "message.id" = id
    "message.compressed_size" = <bytesで数えた圧縮後の容量>,
    "message.uncompressed_size" = <bytesで数えた非圧縮時の容量>
```

<!--
```
-> [time],
    "name" = "message",
    "message.type" = "RECEIVED",
    "message.id" = id
    "message.compressed_size" = <compressed size in bytes>,
    "message.uncompressed_size" = <uncompressed size in bytes>
```
-->

```
-> [time],
    "name" = "message",
    "message.type" = "RECEIVED",
    "message.id" = id
    "message.compressed_size" = <bytesで数えた圧縮後の容量>,
    "message.uncompressed_size" = <bytesで数えた非圧縮時の容量>
```

<!--
The `message.id` MUST be calculated as two different counters starting from `1`
one for sent messages and one for received message. This way we guarantee that
the values will be consistent between different implementations. In case of
unary calls only one sent and one received message will be recorded for both
client and server spans.
-->

`message.id` は `1` から始まる2つの異なるカウンタとして計算されなければいけません(MUST)。この方法で、異なる実装間で値が一貫していることを保証します。unary callの場合は、クライアントとサーバの両方のSpanに対して、送信されたメッセージと受信されたメッセージのうちの1つだけが記録されます。
