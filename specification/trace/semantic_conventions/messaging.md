<!--
# Messaging systems
-->

# メッセージングシステム

<!-- Re-generate TOC with `markdown-toc --no-first-h1 -i` -->

<!-- toc -->

<!--
- [Definitions](#definitions)
- [Conventions](#conventions)
- [Messaging attributes](#messaging-attributes)
- [Examples](#examples)
-->

- [定義](#定義)
- [規約](#規約)
- [メッセージング属性](#メッセージング属性)
- [例](#例)

<!-- tocstop -->

<!--
## Definitions
-->

## 定義

<!--
Although messaging systems are not as standardized as, e.g., HTTP, it is assumed that the following definitions are applicable to most of them that have similar concepts at all (names borrowed mostly from JMS):
-->

メッセージングシステムはHTTPなどのように標準化されているわけではありません。以下の定義は、全て似たような概念を持つもの(名前の多くはJMSからの借用)がほとんどであると想定しています。

<!--
A *message* usually consists of headers (or properties, or meta information) and an optional body. It is sent by a single message *producer* to:
-->

*メッセージ*は通常、ヘッダ(またはプロパティ、メタ情報)とオプションのボディで構成されます。これは、単一のメッセージ *producer* によって以下の宛先に送信されます。

<!--
* Physically: some message *broker* (which can be e.g., a single server, or a cluster, or a local process reached via IPC). The broker handles the actual routing, delivery, re-delivery, persistence, etc. In some messaging systems the broker may be identical or co-located with (some) message consumers.
* Logically: some particular message *destination*.
-->

* 物理的には: *ブローカー* (単一のサーバ、クラスタ、IPC経由で到達するローカルプロセスなど)。ブローカーは、実際のルーティング、配信、再配信、永続性などを処理します。一部のメッセージングシステムでは、ブローカーは(一部の)メッセージコンシューマーと同一または同居している場合があります。
* 論理的には: 特定のメッセージの *宛先*

<!--
A destination is usually identified by some name unique within the messaging system instance, which might look like an URL or a simple one-word identifier.
Two kinds of destinations are distinguished: *topic*s and *queue*s.
A message that is sent (the send-operation is often called "*publish*" in this context) to a *topic* is broadcasted to all *subscribers* of the topic.
A message submitted to a queue is processed by a message *consumer* (usually exactly once although some message systems support a more performant at-least-once mode for messages with [idempotent][] processing).
-->

宛先は通常、URLや簡単な一語からなる識別子のような、メッセージングシステムのインスタンス内で一意な名前で識別されます。2種類の宛先が区別されます。*トピック*と*キュー*です。*トピック*に送信されたメッセージ(この文脈では送信操作を「*publish*」と呼ぶことが多い)は、そのトピックのすべての*サブスクライバ*にブロードキャストされます。キューに送信されたメッセージは、メッセージ*コンシューマー*によって処理されます(ただし、メッセージシステムによっては、[冪等性][]処理を持つメッセージに対して、よりパフォーマンスの高いAt least onceモードをサポートするものもありますが、通常は正確に一度だけです)。

<!--
The consumption of a message can happen in multiple steps.
First, the lower-level receiving of a message at a consumer, and then the logical processing of the message.
Often, the waiting for a message is not particularly interesting and hidden away in a framework that only invokes some handler function to process a message once one is received
(in the same way that the listening on a TCP port for an incoming HTTP message is not particularly interesting).
However, in a synchronous conversation, the wait time for a message is important.
-->

メッセージの消費は、複数のステップで起こり得ます。まず、コンシューマーでのメッセージの下位レベルの受信、そしてメッセージの論理的な処理です。多くの場合、メッセージの待ち時間は特に興味がなく、メッセージを受信してから処理するハンドラ関数を呼び出すだけのフレームワークの中に隠されています (着信 HTTP メッセージの TCP ポートでのリスニングが特に興味がないのと同じように)。しかし、同期通信では、メッセージの待ち時間は重要です。

<!--
In some messaging systems, a message can receive a reply message that answers a particular other message that was sent earlier. All messages that are grouped together by such a reply-relationship are called a *conversation*. The grouping usually happens through some sort of "In-Reply-To:" meta information or an explicit conversation ID. Sometimes a conversation can span multiple message destinations (e.g. initiated via a topic, continued on a temporary one-to-one queue).
-->

一部のメッセージングシステムでは、メッセージは、以前に送信された特定の他のメッセージに回答する返信メッセージを受信することができます。このような返信関係によってグループ化されたすべてのメッセージは、*Conversation*と呼ばれます。このグループ化は通常、ある種の「In-Reply-To:」メタ情報または明示的なConversation IDを介して行われます。会話は、複数のメッセージの宛先にまたがることもあります(例: トピックを介して開始され、一時的な1対1のキューで継続されます)。

<!--
Some messaging systems support the concept of *temporary destination* (often only temporary queues) that are established just for a particular set of communication partners (often one to one) or conversation. Often such destinations are unnamed or have an auto-generated name.
-->

いくつかのメッセージングシステムは、特定の通信相手のセット(多くの場合、1対1)や会話のためだけに確立された *一時的な宛先* (多くの場合、一時的なキューのみ)の概念をサポートしています。多くの場合、このような宛先には名前が付けられていなかったり、自動生成された名前が付けられていたりします。

<!--
[idempotent]: https://en.wikipedia.org/wiki/Idempotence
-->

[冪等性]: https://en.wikipedia.org/wiki/Idempotence

<!--
## Conventions
-->

## 規約

<!--
Given these definitions, the remainder of this section describes the semantic conventions that shall be followed for Spans describing interactions with messaging systems.
-->

これらの定義を前提に、本項の残りの部分では、メッセージングシステムとの相互作用を記述するSpanのために従うべきセマンティック規則について説明します。

<!--
**Span name:** The span name should usually be set to the message destination name.
The conversation ID should be used instead when it is expected to have lower cardinality.
In particular, the conversation ID must be used if the message destination is unnamed or temporary unless multiple conversations can be combined to a logical destination of lower cardinality.
-->

**Span名:** Span名は通常、メッセージの宛先名を設定すべきです(SHOULD)。カーディナリティが低くなることが予想される場合には、代わりにConversation IDを使用するべきです(SHOULD)。特に、メッセージの宛先が無名の場合や一時的な場合は、複数の会話を低カーディナリティの論理的な宛先に結合することができない限り、Conversation IDを使用しなければなりません(MUST)。

<!--
**Span kind:** A producer of a message should set the span kind to `PRODUCER` unless it synchronously waits for a response: then it should use `CLIENT`.
The processor of the message should set the kind to `CONSUMER`, unless it always sends back a reply that is directed to the producer of the message
(as opposed to e.g., a queue on which the producer happens to listen): then it should use `SERVER`.
-->

**Span kind:** メッセージのプロデューサーは、同期的に応答を待たない限り、Span Kindを `PRODUCER` に設定するべきです(SHOULD)。メッセージの処理者は、メッセージのプロデューサーに向けられた返答のSpan Kindを常に `CONSUMER` に設定するべきです(SHOULD)。ただし、メッセージのプロデューサーに直接向けられた応答を送り返す(例えば、プロデューサーがたまたまListenしているQueueではなく)場合は除きます。この場合は `SERVER` を使うべきです(SHOULD)。

<!--
## Messaging attributes
-->

## メッセージング属性

<!--
| Attribute name |                          Notes and examples                            | Required? |
| -------------- | ---------------------------------------------------------------------- | --------- |
| `messaging.system` | A string identifying the messaging system such as `kafka`, `rabbitmq` or `activemq`. | Yes |
| `messaging.destination` | The message destination name, e.g. `MyQueue` or `MyTopic`. This might be equal to the span name but is required nevertheless. | Yes |
| `messaging.destination_kind` | The kind of message destination: Either `queue` or `topic`. | Yes, if either of them applies. |
| `messaging.temp_destination` | A boolean that is `true` if the message destination is temporary. | If temporary (assumed to be `false` if missing). |
| `messaging.protocol` | The name of the transport protocol such as `AMQP` or `MQTT`. | No |
| `messaging.protocol_version` | The version of the transport protocol such as `0.9.1`. | No |
| `messaging.url` | Connection string such as `tibjmsnaming://localhost:7222` or `https://queue.amazonaws.com/80398EXAMPLE/MyQueue`. | No |
| `messaging.message_id` | A value used by the messaging system as an identifier for the message, represented as a string. | No |
| `messaging.conversation_id` | A value identifying the conversation to which the message belongs, represented as a string. Sometimes called "Correlation ID". | No |
| `messaging.message_payload_size_bytes` | The (uncompressed) size of the message payload in bytes. Also use this attribute if it is unknown whether the compressed or uncompressed payload size is reported. | No |
| `messaging.message_payload_compressed_size_bytes` | The compressed size of the message payload in bytes. | No |
-->

| 属性名 | 説明と例                                           | Required? |
| -------------- | ---------------------------------------------------------------------- | --------- |
| `messaging.system` | `kafka`、 `rabbitmq`、 `activemq` などのメッセージングシステムを識別する文字列 | Yes |
| `messaging.destination` | メッセージの送信先の名前、例えば `MyQueue` や `MyTopic` など。これはSpan名と同じかもしれませんが、必須です | Yes |
| `messaging.destination_kind` | メッセージの送信先の種類。`queue` または `topic` のいずれか | どちらかに当てはまるのであればYes |
| `messaging.temp_destination` | メッセージの宛先が一時的なものである場合に `true` となるブール値 | 一時的な場合(ない場合は `false` とする) |
| `messaging.protocol` | `AMQP` や `MQTT` などのトランスポートプロトコルの名前 | No |
| `messaging.protocol_version` | `0.9.1`のようなトランスポートプロトコルのバージョン | No |
| `messaging.url` | `tibjmsnaming://localhost:7222` や `https://queue.amazonaws.com/80398EXAMPLE/MyQueue` のような接続文字列 | No |
| `messaging.message_id` | メッセージングシステムがメッセージの識別子として使用する値で、文字列で表されます | No |
| `messaging.conversation_id` | メッセージが属する会話を識別する値で、文字列で表されます。"Correlation ID"と呼ばれることもあります | No |
| `messaging.message_payload_size_bytes` | メッセージのペイロードの(非圧縮の)サイズをバイト単位で指定します。また、ペイロードのサイズが圧縮されたものか非圧縮のものかが不明な場合は、この属性を使用します | No |
| `messaging.message_payload_compressed_size_bytes` | メッセージペイロードの圧縮サイズをバイト単位で指定します | No |

<!--
Additionally at least one of `net.peer.name` or `net.peer.ip` from the [network attributes][] is required and `net.peer.port` is recommended.
Furthermore, it is strongly recommended to add the [`net.transport`][] attribute and follow its guidelines, especially for in-process queueing systems (like [Hangfire][], for example).
These attributes should be set to the broker to which the message is sent/from which it is received.
-->


さらに、[ネットワーク属性][]の中の `net.peer.name` または `net.peer.ip` のうち少なくとも1つが必要です。また、`net.peer.port` は推奨です。さらに、特にインプロセスキューイングシステム(例えば、[Hangfire][]のような)のために、[`net.transport`][]属性を追加し、そのガイドラインに従うことを強く推奨します。これらの属性は、メッセージの送信先/受信先のブローカに設定するべきです(SHOULD)。

<!--
[network attributes]: span-general.md#general-network-connection-attributes
[`net.transport`]: span-general.md#nettransport-attribute
[Hangfire]: https://www.hangfire.io/
-->

[network attributes]: span-general.md#general-network-connection-attributes
[`net.transport`]: span-general.md#nettransport-attribute
[Hangfire]: https://www.hangfire.io/

<!--
For message consumers, the following additional attributes may be set:
-->

メッセージコンシューマーには、以下の追加属性を設定できます(MAY):

<!--
| Attribute name |                          Notes and examples                            | Required? |
| -------------- | ---------------------------------------------------------------------- | --------- |
| `messaging.operation` | A string identifying which part and kind of message consumption this span describes: either `receive` or `process`. (If the operation is `send`, this attribute must not be set: the operation can be inferred from the span kind in that case.) | No |
-->

| 属性名 | 説明と例                                           | Required? |
| -------------- | ---------------------------------------------------------------------- | --------- |
| `messaging.operation` | このSpanが記述するメッセージ消費の種類と部分を識別する文字列。(操作が `send` の場合は、この属性を設定してはいけません(MUST NOT)。その場合は、Span Kindから操作を推測することができます)| No |

<!--
The _receive_ span is be used to track the time used for receiving the message(s), whereas the _process_ span(s) track the time for processing the message(s).
Note that one or multiple Spans with `messaging.operation` = `process` may often be the children of a Span with `messaging.operation` = `receive`.
Even though in that case one might think that the processing span's kind should be `INTERNAL`, that kind MUST NOT be used.
Instead span kind should be set to either `CONSUMER` or `SERVER` according to the rules defined above.
-->

_receive_ Spanはメッセージの受信に使用された時間を追跡するために使用され、 _process_ Spanはメッセージの処理に使用された時間を追跡するために使用されます。`messaging.operation` = `process` の 1 つまたは複数のSpanは、`messaging.operation` = `receceive` のSpanの子であることが多いことに注意してください。この場合、process SpanのKindを `INTERNAL` にすべきだと思うかもしれませんが、そのKindは使用してはいけません(MUST NOT)。その代わりに、上で定義したルールに従って、Span Kindを `CONSUMER` か `SERVER` のいずれかに設定しなければなりません(SHOULD)。

<!--
### Attributes specific to certain messaging systems
-->

### 特定のメッセージングシステムに固有の属性

<!--
#### RabbitMQ
-->

#### RabbitMQ

<!--
In RabbitMQ, the destination is defined by an _exchange_ and a _routing key_.
`messaging.destination` MUST be set to the name of the exchange. This will be an empty string if the default exchange is used.
The routing key MUST be provided to the attribute `messaging.rabbitmq.routing_key`, unless it is empty.
-->

RabbitMQでは、宛先は _exchange_ と _routing key_ で定義されます。`messaging.destination` には交換先の名前を設定しなければなりません(MUST)。デフォルトのブローカーが使われている場合、これは空の文字列になります。ルーティングキーは、空でない限り `messaging.rabbitmq.routing_key` 属性に指定しなければなりません(MUST)。

<!--
## Examples
-->

## 例

<!--
### Topic with multiple consumers
-->

### 複数のコンシューマーがあるTopic

<!--
Given is a process P, that publishes a message to a topic T on messaging system MS, and two processes CA and CB, which both receive the message and process it.
-->

メッセージングシステムMS上のトピックTにメッセージを公開するプロセスPと、メッセージを受信して処理する2つのプロセスCAとCBがあるとします。

<!--
```
Process P:  | Span Prod1 |
--
Process CA:              | Span CA1 |
--
Process CB:                 | Span CB1 |
```
-->

```
プロセス P:  | Span Prod1 |
--
プロセス CA:              | Span CA1 |
--
プロセス CB:                 | Span CB1 |
```

<!--
| Field or Attribute | Span Prod1 | Span CA1 | Span CB1 |
|-|-|-|-|
| Name | `"T"` | `"T"` | `"T"` |
| Parent |  | Span Prod1 | Span Prod1 |
| Links |  |  |  |
| SpanKind | `PRODUCER` | `CONSUMER` | `CONSUMER` |
| Status | `Ok` | `Ok` | `Ok` |
| `net.peer.name` | `"ms"` | `"ms"` | `"ms"` |
| `net.peer.port` | `1234` | `1234` | `1234` |
| `messaging.system` | `"kafka"` | `"kafka"` | `"kafka"` |
| `messaging.destination` | `"T"` | `"T"` | `"T"` |
| `messaging.destination_kind` | `"topic"` | `"topic"` | `"topic"` |
| `messaging.operation` |  | `"process"` | `"process"` |
| `messaging.message_id` | `"a1"` | `"a1"`| `"a1"` |
-->

| フィールドまたは属性 | Span Prod1 | Span CA1 | Span CB1 |
|-|-|-|-|
| Name | `"T"` | `"T"` | `"T"` |
| Parent |  | Span Prod1 | Span Prod1 |
| Links |  |  |  |
| SpanKind | `PRODUCER` | `CONSUMER` | `CONSUMER` |
| Status | `Ok` | `Ok` | `Ok` |
| `net.peer.name` | `"ms"` | `"ms"` | `"ms"` |
| `net.peer.port` | `1234` | `1234` | `1234` |
| `messaging.system` | `"kafka"` | `"kafka"` | `"kafka"` |
| `messaging.destination` | `"T"` | `"T"` | `"T"` |
| `messaging.destination_kind` | `"topic"` | `"topic"` | `"topic"` |
| `messaging.operation` |  | `"process"` | `"process"` |
| `messaging.message_id` | `"a1"` | `"a1"`| `"a1"` |

<!--
### Batch receiving
-->

### 一括受信

<!--
Given is a process P, that sends two messages to a queue Q on messaging system MS, and a process C, which receives both of them in one batch (Span Recv1) and processes each message separately (Spans Proc1 and Proc2).
-->

メッセージングシステムMS上のキューQに2つのメッセージを送信するプロセスPと、その2つのメッセージを一括で受信し(Span Recv1)、それぞれのメッセージを個別に処理するプロセスC(Span Proc1, Proc2)があるとします。

<!--
Since a span can only have one parent and the propagated trace and span IDs are not known when the receiving span is started, the receiving span will have no parent and the processing spans are correlated with the producing spans using links.
-->

1つのSpanが1つの親しか持てず、受信Spanが開始された時点では伝搬されたTraceとSpan IDがわからないため、受信Spanは親を持たず、処理SpanはLinkを使って生成Spanと相関関係(Correlation)を持つことになります。

<!--
```
Process P: | Span Prod1 | Span Prod2 |
--
Process C:                      | Span Recv1 |
                                        | Span Proc1 |
                                               | Span Proc2 |
```
-->

```
プロセス P: | Span Prod1 | Span Prod2 |
--
プロセス C:                      | Span Recv1 |
                                        | Span Proc1 |
                                               | Span Proc2 |
```

<!--
| Field or Attribute | Span Prod1 | Span Prod2 | Span Recv1 | Span Proc1 | Span Proc2 |
|-|-|-|-|-|-|
| Name | `"Q"` | `"Q"` | `"Q"` | `"Q"` | `"Q"` |
| Parent |  |  |  | Span Recv1 | Span Recv1 |
| Links |  |  |  | Span Prod1 | Span Prod2 |
| SpanKind | `PRODUCER` | `PRODUCER` | `CONSUMER` | `CONSUMER` | `CONSUMER` |
| Status | `Ok` | `Ok` | `Ok` | `Ok` | `Ok` |
| `net.peer.name` | `"ms"` | `"ms"` | `"ms"` | `"ms"` | `"ms"` |
| `net.peer.port` | `1234` | `1234` | `1234` | `1234` | `1234` |
| `messaging.system` | `"kafka"` | `"kafka"` | `"kafka"` | `"kafka"` | `"kafka"` |
| `messaging.destination` | `"Q"` | `"Q"` | `"Q"` | `"Q"` | `"Q"` |
| `messaging.destination_kind` | `"queue"` | `"queue"` | `"queue"` | `"queue"` | `"queue"` |
| `messaging.operation` |  |  | `"receive"` | `"process"` | `"process"` |
| `messaging.message_id` | `"a1"` | `"a2"` | | `"a1"` | `"a2"` |
-->

| フィールドまたは属性 | Span Prod1 | Span Prod2 | Span Recv1 | Span Proc1 | Span Proc2 |
|-|-|-|-|-|-|
| Name | `"Q"` | `"Q"` | `"Q"` | `"Q"` | `"Q"` |
| Parent |  |  |  | Span Recv1 | Span Recv1 |
| Links |  |  |  | Span Prod1 | Span Prod2 |
| SpanKind | `PRODUCER` | `PRODUCER` | `CONSUMER` | `CONSUMER` | `CONSUMER` |
| Status | `Ok` | `Ok` | `Ok` | `Ok` | `Ok` |
| `net.peer.name` | `"ms"` | `"ms"` | `"ms"` | `"ms"` | `"ms"` |
| `net.peer.port` | `1234` | `1234` | `1234` | `1234` | `1234` |
| `messaging.system` | `"kafka"` | `"kafka"` | `"kafka"` | `"kafka"` | `"kafka"` |
| `messaging.destination` | `"Q"` | `"Q"` | `"Q"` | `"Q"` | `"Q"` |
| `messaging.destination_kind` | `"queue"` | `"queue"` | `"queue"` | `"queue"` | `"queue"` |
| `messaging.operation` |  |  | `"receive"` | `"process"` | `"process"` |
| `messaging.message_id` | `"a1"` | `"a2"` | | `"a1"` | `"a2"` |

<!--
### Batch processing
-->

### 一括処理

<!--
Given is a process P, that sends two messages to a queue Q on messaging system MS, and a process C, which receives both of them separately (Span Recv1 and Recv2) and processes both messages in one batch (Span Proc1).
-->

メッセージングシステムMS上のキューQに2つのメッセージを送信するプロセスPと、その両方を別々に受信し(Span Recv1とRecv2)、両方のメッセージを一括で処理するプロセスC(Span Proc1)があるとすると、以下のようになります。

<!--
Since each span can only have one parent, C3 should not choose a random parent out of C1 and C2, but rather rely on the implicitly selected parent as defined by the [tracing API spec](../api.md).
Similarly, only one value can be set as `message_id`, so C3 cannot report both `a1` and `a2` and therefore attribute is left out.
Depending on the implementation, the producing spans might still be available in the meta data of the messages and should be added to C3 as links.
The client library or application could also add the receiver span's span context to the data structure it returns for each message. In this case, C3 could also add links to the receiver spans C1 and C2.
-->

各Spanは1つの親しか持てないので、C3はC1とC2の中からランダムな親を選ぶのではなく、[tracing API 仕様](./api.md)で定義されているように、暗黙のうちに選択された親に頼るべきです。
同様に、`message_id`には1つの値しか設定できないので、C3は`a1`と`a2`の両方を報告することができず、そのために属性は除外されています。実装にもよりますが、メッセージのメタデータの中でSpanを生成することができるかもしれませんし、C3にLinkとして追加するべきです(SHOULD)。クライアントライブラリやアプリケーションは、各メッセージに対して返すデータ構造に、受信SpanのSpan Contextを追加することもできます。この場合、C3 は、受信Span C1 および C2 へのLinkを追加することもできます。

<!--
The status of the batch processing span is selected by the application. Depending on the semantics of the operation. A span status `Ok` could, for example, be set only if all messages or if just at least one were properly processed.
-->

一括処理SpanのStatusは、アプリケーションによって選択されます。操作のセマンティクスに依存します。例えば、SpanのStatus `Ok` は、すべてのメッセージが適切に処理された場合のみ、あるいは少なくとも1つのメッセージだけが適切に処理された場合のみ設定することができます。

<!--
```
Process P: | Span Prod1 | Span Prod2 |
--
Process C:                              | Span Recv1 | Span Recv2 |
                                                                   | Span Proc1 |
```
-->

```
プロセス P: | Span Prod1 | Span Prod2 |
--
プロセス C:                              | Span Recv1 | Span Recv2 |
                                                                   | Span Proc1 |
```

<!--
| Field or Attribute | Span Prod1 | Span Prod2 | Span Recv1 | Span Recv2 | Span Proc1 |
|-|-|-|-|-|-|
| Name | `"Q"` | `"Q"` | `"Q"` | `"Q"` | `"Q"` |
| Parent |  |  | Span Prod1 | Span Prod2 |  |
| Links |  |  |  |  | Span Prod1 + Prod2 |
| SpanKind | `PRODUCER` | `PRODUCER` | `CONSUMER` | `CONSUMER` | `CONSUMER` |
| Status | `Ok` | `Ok` | `Ok` | `Ok` | `Ok` |
| `net.peer.name` | `"ms"` | `"ms"` | `"ms"` | `"ms"` | `"ms"` |
| `net.peer.port` | `1234` | `1234` | `1234` | `1234` | `1234` |
| `messaging.system` | `"kafka"` | `"kafka"` | `"kafka"` | `"kafka"` | `"kafka"` |
| `messaging.destination` | `"Q"` | `"Q"` | `"Q"` | `"Q"` | `"Q"` |
| `messaging.destination_kind` | `"queue"` | `"queue"` | `"queue"` | `"queue"` | `"queue"` |
| `messaging.operation` |  |  | `"receive"` | `"receive"` | `"process"` |
| `messaging.message_id` | `"a1"` | `"a2"` | `"a1"` | `"a2"` | |
-->

| フィールドまたは属性 | Span Prod1 | Span Prod2 | Span Recv1 | Span Recv2 | Span Proc1 |
|-|-|-|-|-|-|
| Name | `"Q"` | `"Q"` | `"Q"` | `"Q"` | `"Q"` |
| Parent |  |  | Span Prod1 | Span Prod2 |  |
| Links |  |  |  |  | Span Prod1 + Prod2 |
| SpanKind | `PRODUCER` | `PRODUCER` | `CONSUMER` | `CONSUMER` | `CONSUMER` |
| Status | `Ok` | `Ok` | `Ok` | `Ok` | `Ok` |
| `net.peer.name` | `"ms"` | `"ms"` | `"ms"` | `"ms"` | `"ms"` |
| `net.peer.port` | `1234` | `1234` | `1234` | `1234` | `1234` |
| `messaging.system` | `"kafka"` | `"kafka"` | `"kafka"` | `"kafka"` | `"kafka"` |
| `messaging.destination` | `"Q"` | `"Q"` | `"Q"` | `"Q"` | `"Q"` |
| `messaging.destination_kind` | `"queue"` | `"queue"` | `"queue"` | `"queue"` | `"queue"` |
| `messaging.operation` |  |  | `"receive"` | `"receive"` | `"process"` |
| `messaging.message_id` | `"a1"` | `"a2"` | `"a1"` | `"a2"` | |
