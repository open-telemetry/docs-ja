<!--
# Messaging systems
-->

# メッセージングシステム

<!--
**Status**: [Experimental](../../document-status.md)
-->

**Status**: [Experimental](../../document-status.md)

<!-- Re-generate TOC with `markdown-toc --no-first-h1 -i` -->

<!-- toc -->

<!--
- [Definitions](#definitions)
  * [Destinations](#destinations)
  * [Message consumption](#message-consumption)
  * [Conversations](#conversations)
  * [Temporary destinations](#temporary-destinations)
- [Conventions](#conventions)
  * [Span name](#span-name)
  * [Span kind](#span-kind)
  * [Operation names](#operation-names)
- [Messaging attributes](#messaging-attributes)
  * [Attributes specific to certain messaging systems](#attributes-specific-to-certain-messaging-systems)
    + [RabbitMQ](#rabbitmq)
    + [Apache Kafka](#apache-kafka)
- [Examples](#examples)
  * [Topic with multiple consumers](#topic-with-multiple-consumers)
  * [Apache Kafka Example](#apache-kafka-example)
  * [Batch receiving](#batch-receiving)
  * [Batch processing](#batch-processing)
-->

- [定義](#定義)
  * [宛先](#宛先)
  * [メッセージの消費](#メッセージの消費)
  * [会話(Conversations)](#会話-conversations)
  * [一時的な宛先](#一時的な宛先)
- [規約](#規約)
  * [Span name](#span-name)
  * [Span kind](#span-kind)
  * [操作名](#操作名)
- [メッセージング属性](#メッセージング属性)
  * [特定のメッセージングシステムに特有の属性](#特定のメッセージングシステムに特有の属性)
    + [RabbitMQ](#rabbitmq)
    + [Apache Kafka](#apache-kafka)
- [例](#例)
  * [複数のConsumerがいるトピック](#複数のConsumerがいるトピック)
  * [Apache Kafkaの例](#apache-kafkaの例)
  * [バッチ受信](#バッチ受信)
  * [バッチ処理](#バッチ処理)


<!-- tocstop -->
<!--
## Definitions
-->

## 定義

<!--
Although messaging systems are not as standardized as, e.g., HTTP, it is assumed that the following definitions are applicable to most of them that have similar concepts at all (names borrowed mostly from JMS):
-->

メッセージングシステムは、HTTPなどのように標準化されていませんが、以下の定義は、似たような概念を持つほとんどのメッセージングシステムに適用可能であると想定されます(名前は主にJMSから借用しています)。

<!--
A *message* is an envelope with a potentially empty payload.
This envelope may offer the possibility to convey additional metadata, often in key/value form.
-->

*メッセージ*は、空のペイロードを持つ可能性のあるエンベロープです。このエンベロープには、追加のメタデータを伝える可能性があり、多くの場合、キー/バリュー形式になっています。

<!--
A message is sent by a message *producer* to:
-->

メッセージは、メッセージ *producer* から送られます。

<!--
* Physically: some message *broker* (which can be e.g., a single server, or a cluster, or a local process reached via IPC). The broker handles the actual delivery, re-delivery, persistence, etc. In some messaging systems the broker may be identical or co-located with (some) message consumers.
With Apache Kafka, the physical broker a message is written to depends on the number of partitions, and which broker is the *leader* of the partition the record is written to.
* Logically: some particular message *destination*.
-->

* 物理的: 何らかのメッセージブローカー(単一のサーバー、クラスター、IPCで到達するローカルプロセスなど)が存在します。ブローカーは、実際の配信、再配信、永続化などを行います。メッセージングシステムによっては、ブローカーはメッセージコンシューマと同一または同居している場合があります。Apache Kafkaでは、メッセージが書き込まれる物理的なブローカーは、パーティションの数と、レコードが書き込まれるパーティションの*リーダー*がどのブローカーであるかによって決まります。
* 論理的: 特定のメッセージの *destination*。

<!--
Messages can be delivered to 0, 1, or multiple consumers depending on the dispatching semantic of the protocol.
-->

メッセージは、プロトコルのディスパッチセマンティックに応じて、0、1、または複数のコンシューマに配信されます。

<!--
### Destinations
-->

### 宛先(Destrination)

<!--
A destination is usually identified by some name unique within the messaging system instance, which might look like a URL or a simple one-word identifier.
Traditional messaging, such as JMS, involves two kinds of destinations: *topic*s and *queue*s.
A message that is sent (the send-operation is often called "*publish*" in this context) to a *topic* is broadcasted to all consumers that have *subscribed* to the topic.
A message submitted to a queue is processed by a message *consumer* (usually exactly once although some message systems support a more performant at-least-once mode for messages with [idempotent][] processing).
-->

宛先は、通常、メッセージングシステムのインスタンス内でユニークな名前で識別され、URLやシンプルな1語の識別子のようなものです。JMSなどの従来のメッセージングでは、2種類の宛先があります。それは、*トピック*と*キュー*です。*トピック*に送信されたメッセージ(この文脈では、送信操作を「公開」と呼ぶことが多い)は、そのトピックを「購読」しているすべてのコンシューマにブロードキャストされます。キューに送信されたメッセージは、メッセージ *consumer* によって処理されます(通常はちょうど1回ですが、一部のメッセージ・システムでは、[べき等][] 処理によるメッセージに対して、よりパフォーマンスの高い at-least-once モードをサポートしています)。

<!--
In a messaging system such as Apache Kafka, all destinations are *topic*s.
Each record, or message, is sent to a single consumer per consumer group.
Consumer groups provide *deliver once* semantics for consumers of a topic within a group.
Whether a specific message is processed as if it was sent to a topic or queue entirely depends on the consumer groups and their composition.
For instance, there can be multiple consumer groups processing records from the same topic.
-->

Apache Kafkaのようなメッセージングシステムでは、すべての送信先は*トピック*となります。各レコード(メッセージ)は、コンシューマグループごとに1つのコンシューマに送信されます。コンシューマグループは、グループ内のトピックのコンシューマに対して、*deliver once*のセマンティクスを提供します。特定のメッセージが、あたかもトピックやキューに送られたかのように処理されるかどうかは、コンシューマ・グループとその構成に完全に依存します。たとえば、同じトピックのレコードを処理する複数のコンシューマグループが存在することがあります。

<!--
[idempotent]: https://en.wikipedia.org/wiki/Idempotence
-->

[べき等]: https://en.wikipedia.org/wiki/Idempotence

<!--
### Message consumption
-->

### Message consumption

<!--
The consumption of a message can happen in multiple steps.
First, the lower-level receiving of a message at a consumer, and then the logical processing of the message.
Often, the waiting for a message is not particularly interesting and hidden away in a framework that only invokes some handler function to process a message once one is received
(in the same way that the listening on a TCP port for an incoming HTTP message is not particularly interesting).
-->

メッセージの消費は、複数のステップで行われます。まず、コンシューマーでのメッセージの低レベルの受信、そしてメッセージの論理的な処理です。多くの場合、メッセージを待つことは特に面白いことではなく、フレームワークの中に隠れていて、メッセージを受信したら処理するためのハンドラー関数を呼び出すだけになっています(HTTPメッセージの着信をTCPポートで待ち受けることが特に面白いことではないのと同じように)。

<!--
### Conversations
-->

### 会話(Conversations)

<!--
In some messaging systems, a message can receive one or more reply messages that answers a particular other message that was sent earlier. All messages that are grouped together by such a reply-relationship are called a *conversation*.
The grouping usually happens through some sort of "In-Reply-To:" meta information or an explicit *conversation ID* (sometimes called *correlation ID*).
Sometimes a conversation can span multiple message destinations (e.g. initiated via a topic, continued on a temporary one-to-one queue).
-->

いくつかのメッセージングシステムでは、あるメッセージが、以前に送信された特定の他のメッセージに答える1つまたは複数の返信メッセージを受け取ることができます。このような返信関係によってグループ化されたすべてのメッセージを *会話(Conversations)* と呼びます。グループ化は通常、"In-Reply-To: "というメタ情報や、明示的な*会話ID*(*相関ID*と呼ばれることもある)によって行われます。時には、1つの会話が複数のメッセージ送信先にまたがることもあります(例えば、トピックを介して開始され、一時的に1対1のキューで継続される)。

<!--
### Temporary destinations
-->

### 一時的な宛先

<!--
Some messaging systems support the concept of *temporary destination* (often only temporary queues) that are established just for a particular set of communication partners (often one to one) or conversation.
Often such destinations are unnamed or have an auto-generated name.
-->

メッセージングシステムの中には、特定の通信相手(多くの場合、1対1)や会話のためだけに確立される*一時的な宛先*(多くの場合、一時的なキューのみ)の概念をサポートしているものがあります。多くの場合、そのような宛先には名前がついていないか、自動生成された名前がついています。

<!--
## Conventions
-->

## 規約

<!--
Given these definitions, the remainder of this section describes the semantic conventions for Spans describing interactions with messaging systems.
-->

これらの定義を踏まえて、このセクションの残りの部分では、メッセージングシステムとのインタラクションを記述するSpansのセマンティック規約について説明します。

<!--
### Span name
-->

### Span 名

<!--
The span name SHOULD be set to the message destination name and the operation being performed in the following format:
-->

Span名には、メッセージの宛先名と実行中の操作を、以下の形式で設定すべきです。

<!--
```
<destination name> <operation name>
```
-->

```
<宛先名> <操作名>
```

<!--
The destination name SHOULD only be used for the span name if it is known to be of low cardinality (cf. [general span name guidelines](../api.md#span)).
This can be assumed if it is statically derived from application code or configuration.
Wherever possible, the real destination names after resolving logical or aliased names SHOULD be used.
If the destination name is dynamic, such as a [conversation ID](#conversations) or a value obtained from a `Reply-To` header, it SHOULD NOT be used for the span name.
In these cases, an artificial destination name that best expresses the destination, or a generic, static fallback like `"(temporary)"` for [temporary destinations](#temporary-destinations) SHOULD be used instead.
-->

宛先名は、それが低いカーディナリティであることがわかっている場合にのみ、Span名に使用されるべきです(SHOULD)([general span name guidelines](../api.md#span)を参照)。これは、アプリケーションコードや構成から静的に派生したものである場合に想定されます。可能な限り、論理名やエイリアス名を解決した後の実際の宛先名を使用すべきです(SHOULD)。宛先名が[会話ID](#conversations)や`Reply-To`ヘッダーから得られる値のように動的なものである場合、それをSpan名に使用すべきではありません(SHOULD NOT)。このような場合には、宛先を最もよく表現する人工的な宛先名や、[一時的な宛先](#一時的な宛先)に対する`"(temporary)"`のような一般的で静的なフォールバックを代わりに使用すべきです(SHOULD)。

<!--
The values allowed for `<operation name>` are defined in the section [Operation names](#operation-names) below.
If the format above is used, the operation name MUST match the `messaging.operation` attribute defined for message consumer spans below.
-->

`<操作名>`に許される値は、以下の[操作名](#操作名)のセクションで定義されています。上記のフォーマットを使用する場合、操作名は以下のメッセージコンシューマーSpanで定義されている `messaging.operation` 属性と一致しなければなりません (MUST)。

<!--
Examples:
-->

例:

<!--
* `shop.orders send`
* `shop.orders receive`
* `shop.orders process`
* `print_jobs send`
* `topic with spaces process`
* `AuthenticationRequest-Conversations process`
* `(temporary) send` (`(temporary)` being a stable identifier for randomly generated, temporary destination names)
-->

* `shop.orders send`
* `shop.orders receive`
* `shop.orders process`
* `print_jobs send`
* `topic with spaces process`
* `AuthenticationRequest-Conversations process`
* `(temporary) send` (`(temporary)`はランダムに生成される一時的な宛先名の安定した識別子)

<!--
### Span kind
-->

### Span kind

<!--
A producer of a message should set the span kind to `PRODUCER` unless it synchronously waits for a response: then it should use `CLIENT`.
The processor of the message should set the kind to `CONSUMER`, unless it always sends back a reply that is directed to the producer of the message
(as opposed to e.g., a queue on which the producer happens to listen): then it should use `SERVER`.
-->

メッセージのプロデューサーは、応答を同期的に待つ場合を除き、Spanの種類を `PRODUCER` に設定する必要があります。その場合(XXX: 応答を同期的に待つ場合?)は `CLIENT` になります。
メッセージのプロセッサは、メッセージのプロデューサー(例えば、プロデューサーがたまたま聞いているキューではなく)に向けられた応答を常に送り返す場合を除き、種類を `CONSUMER` に設定する必要があります: その場合は `SERVER` を使用する必要があります。その場合(XXX: 応答を常に送り返す場合?)は `CLIENT` になります。

<!--
### Operation names
-->

### 操作名

<!--
The following operations related to messages are defined for these semantic conventions:
-->

これらのセマンティック規約には、以下のようなメッセージに関する操作が定義されています。

<!--
| Operation name | Description |
| -------------- | ----------- |
| `send`         | A message is sent to a destination by a message producer/client.       |
| `receive`      | A message is received from a destination by a message consumer/server. |
| `process`      | A message that was previously received from a destination is processed by a message consumer/server. |
-->

| 操作名 | 宛先 |
| -------------- | ----------- |
| `send`         | メッセージを、メッセージプロデューサ/クライアントによって宛先に送信する      |
| `receive`      | メッセージコンシューマー/サーバーが宛先からメッセージを受信する |
| `process`      | 宛先から事前に受信したメッセージを、メッセージコンシューマー/サーバーが処理する |

<!--
## Messaging attributes
-->

## メッセージング属性


<!-- semconv messaging -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `messaging.system` | string | メッセージングシステムを識別するための文字列 | `kafka`; `rabbitmq`; `activemq`; `AmazonSQS` | Yes |
| `messaging.destination` | string | メッセージの宛先名です。これは、Span名と同じかもしれませんが、それでも必要です。 | `MyQueue`; `MyTopic` | Yes |
| `messaging.destination_kind` | string | メッセージの宛先の種類 | `queue` | メッセージの宛先が `queue` または `topic` の場合にのみ必要です。 |
| `messaging.temp_destination` | boolean | メッセージの送信先が一時的なものである場合に真となる真偽値 |  | 欠落している場合は、偽であると見なされます |
| `messaging.protocol` | string | トランスポートプロトコルの名前 | `AMQP`; `MQTT` | No |
| `messaging.protocol_version` | string | トランスポートプロトコルのバージョン | `0.9.1` | No |
| `messaging.url` | string | 接続文字列 | `tibjmsnaming://localhost:7222`; `https://queue.amazonaws.com/80398EXAMPLE/MyQueue` | No |
| `messaging.message_id` | string | メッセージングシステムがメッセージの識別子として使用する値で、文字列で表されます。 | `452a7c7c7c7048c2f887f61572b18fc2` | No |
| `messaging.conversation_id` | string | メッセージが属する会話を識別する[会話ID](#conversations)を文字列で表したもの。"Correlation ID"と呼ばれることもあります。 | `MyConversationId` | No |
| `messaging.message_payload_size_bytes` | int | メッセージのペイロードの(圧縮されていない)サイズをバイト単位で示したものです。また、圧縮されたペイロード・サイズと非圧縮のペイロード・サイズのどちらが報告されるかが不明な場合にも、この属性を使用します。 | `2738` | No |
| `messaging.message_payload_compressed_size_bytes` | int | メッセージのペイロードの圧縮サイズ(バイト)です。 | `2048` | No |
| [`net.peer.ip`](span-general.md) | string | 相手のリモートアドレス(IPv4ではドット10進数、IPv6では[RFC5952](https://tools.ietf.org/html/rfc5952) | `127.0.0.1` | 利用可能な場合 |
| [`net.peer.name`](span-general.md) | string | リモートのホスト名あるいは類似の文字列。下記注釈参照 [1] | `example.com` | 利用可能な場合 |

**[1]:** これは、この特定のメッセージを送受信するブローカー(または他のネットワークレベルのピア)のIP/ホスト名でなければなりません。

`messaging.destination_kind` MUST be one of the following:

| Value  | Description |
|---|---|
| `queue` | メッセージはQueueに送られる |
| `topic` | メッセージはTopicに送られる |
<!-- endsemconv -->

<!--
Additionally `net.peer.port` from the [network attributes][] is recommended.
Furthermore, it is strongly recommended to add the [`net.transport`][] attribute and follow its guidelines, especially for in-process queueing systems (like [Hangfire][], for example).
These attributes should be set to the broker to which the message is sent/from which it is received.
-->

さらに、[ネットワーク属性][]の`net.peer.port`を推奨します。さらに、[net.transport`][]属性を追加し、特にプロセス中のキューイングシステム([Hangfire][]など)の場合は、そのガイドラインに従うことを強く推奨します。これらの属性には、メッセージの送信先/受信先のブローカーを設定する必要があります。

<!--
[network attributes]: span-general.md#general-network-connection-attributes
[`net.transport`]: span-general.md#nettransport-attribute
[Hangfire]: https://www.hangfire.io/
-->

[ネットワーク属性]: span-general.md#general-network-connection-attributes
[`net.transport`]: span-general.md#nettransport-attribute
[Hangfire]: https://www.hangfire.io/

<!--
For message consumers, the following additional attributes may be set:
-->

メッセージコンシューマーには、以下の追加属性を設定することができます。

<!-- semconv messaging.consumer -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `messaging.operation` | string | 上記の[操作名](#operation-names)セクションで定義されているメッセージ消費の種類を識別する文字列です。操作が "send"の場合、この属性は設定してはいけません(MUST NOT)。なぜなら、この場合、操作はspan kindから推測できるからです。 | `receive` | No |

`messaging.operation` MUST be one of the following:

| Value  | Description |
|---|---|
| `receive` | receive |
| `process` | process |
<!-- endsemconv -->

<!--
The _receive_ span is be used to track the time used for receiving the message(s), whereas the _process_ span(s) track the time for processing the message(s).
Note that one or multiple Spans with `messaging.operation` = `process` may often be the children of a Span with `messaging.operation` = `receive`.
The distinction between receiving and processing of messages is not always of particular interest or sometimes hidden away in a framework (see the [Message consumption](#message-consumption) section above) and therefore the attribute can be left out.
For batch receiving and processing (see the [Batch receiving](#batch-receiving) and [Batch processing](#batch-processing) examples below) in particular, the attribute SHOULD be set.
Even though in that case one might think that the processing span's kind should be `INTERNAL`, that kind MUST NOT be used.
Instead span kind should be set to either `CONSUMER` or `SERVER` according to the rules defined above.
-->

_receive_Spanはメッセージの受信にかかった時間を追跡し、_process_Spanはメッセージの処理にかかった時間を追跡するのに使います。なお、`messaging.operation` = `process` である1つまたは複数のSpanは、`messaging.operation` = `receive` のSpanの子であることが多いです。メッセージの受信と処理の区別は、必ずしも特別な関心事ではなく、フレームワークの中に隠されていることもあるので(上記の[メッセージ消費](#メッセージ消費)のセクションを参照)、この属性は省くことができます。特にバッチ受信やバッチ処理(後述の[バッチ受信](#バッチ受信)や[バッチ処理](#バッチ処理)の例を参照)では、この属性は設定されるべきです(SHOULD)。この場合、処理Spanのkindを`INTERNAL`にすべきだと考えるかもしれませんが、そのkindは使用してはいけません(MUST NOT)。代わりに、Spanのkindは、上記で定義されたルールに従って、`CONSUMER`または`SERVER`のいずれかに設定する必要があります。

<!--
### Attributes specific to certain messaging systems
-->

### 特定のメッセージングシステムに特有の属性

<!--
#### RabbitMQ
-->

#### RabbitMQ

<!--
In RabbitMQ, the destination is defined by an _exchange_ and a _routing key_.
`messaging.destination` MUST be set to the name of the exchange. This will be an empty string if the default exchange is used.
-->

RabbitMQでは、宛先は_exchange_と_routing key_で定義されます。`messaging.destination`には、Exchangeの名前を設定しなければなりません(MUST)。デフォルトの取引所を使用する場合は、空の文字列になります。


<!-- semconv messaging.rabbitmq -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `messaging.rabbitmq.routing_key` | string | RabbitMQのメッセージルーティングキー。 | `myKey` | Unless it is empty. |
<!-- endsemconv -->

<!--
#### Apache Kafka
-->

#### Apache Kafka

<!--
For Apache Kafka, the following additional attributes are defined:
-->

Apache Kafkaでは、以下の追加属性が定義されています。

<!-- semconv messaging.kafka -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `messaging.kafka.message_key` | string | Kafkaのメッセージキーは、同じメッセージが同じパーティションで処理されるようにグループ化するために使用されます。メッセージキーは、`messaging.message_id` とは異なり、一意ではありません。キーが `null` の場合、この属性は設定してはいけません (MUST NOT)。 [1] | `myKey` | No |
| `messaging.kafka.consumer_group` | string | メッセージを処理しているKafka Consumerグループの名前。Producerではなく、Consumerにのみ適用されます。 | `my-group` | No |
| `messaging.kafka.client_id` | string | メッセージを処理しているConsumerまたはProducerのClient Id。 | `client-5` | No |
| `messaging.kafka.partition` | int | メッセージの送信先となるパーティション。 | `2` | No |
| `messaging.kafka.tombstone` | boolean | メッセージがTombstoneである場合に真となる真偽値 |  | 欠落している場合は、偽であると見なされます |

**[1]:** キータイプが文字列でない場合は、その文字列表現を属性に与える必要があります。キーが曖昧でない正規の文字列形式を持たない場合、その値を含めてはいけません。
<!-- endsemconv -->
<!--
-->

<!--
For Apache Kafka producers, [`peer.service`](./span-general.md#general-remote-service-attributes) SHOULD be set to the name of the broker or service the message will be sent to.
The `service.name` of a Consumer's Resource SHOULD match the `peer.service` of the Producer, when the message is directly passed to another service.
If an intermediary broker is present, `service.name` and `peer.service` will not be the same.
-->

Apache Kafkaのプロデューサーの場合、[`peer.service`](./span-general.md#general-remote-service-attributes)には、メッセージの送信先となるブローカーやサービスの名前を設定すべきです(SHOULD)。メッセージが他のサービスに直接渡される場合、ConsumerのResourceの`service.name`は、Producerの`peer.service`と一致するべきです(SHOULD)。仲介ブローカーが存在する場合、`service.name`と`peer.service`は同じではありません。

<!--
## Examples
-->

## 例

<!--
### Topic with multiple consumers
-->

### 複数のConsumerがいるトピック

<!--
Given is a process P, that publishes a message to a topic T on messaging system MS, and two processes CA and CB, which both receive the message and process it.
-->

メッセージングシステムMS上のトピックTにメッセージを発行するプロセスPと、そのメッセージを受信して処理する2つのプロセスCAとCBがあるとします。

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
Process P:  | Span Prod1 |
--
Process CA:              | Span CA1 |
--
Process CB:                 | Span CB1 |
```

<!--
| Field or Attribute | Span Prod1 | Span CA1 | Span CB1 |
|-|-|-|-|
| Span name | `"T send"` | `"T process"` | `"T process"` |
| Parent |  | Span Prod1 | Span Prod1 |
| Links |  |  |  |
| SpanKind | `PRODUCER` | `CONSUMER` | `CONSUMER` |
| Status | `Ok` | `Ok` | `Ok` |
| `net.peer.name` | `"ms"` | `"ms"` | `"ms"` |
| `net.peer.port` | `1234` | `1234` | `1234` |
| `messaging.system` | `"rabbitmq"` | `"rabbitmq"` | `"rabbitmq"` |
| `messaging.destination` | `"T"` | `"T"` | `"T"` |
| `messaging.destination_kind` | `"topic"` | `"topic"` | `"topic"` |
| `messaging.operation` |  | `"process"` | `"process"` |
| `messaging.message_id` | `"a1"` | `"a1"`| `"a1"` |
-->

| フィールドまたは属性 | Span Prod1 | Span CA1 | Span CB1 |
|-|-|-|-|
| Span name | `"T send"` | `"T process"` | `"T process"` |
| Parent |  | Span Prod1 | Span Prod1 |
| Links |  |  |  |
| SpanKind | `PRODUCER` | `CONSUMER` | `CONSUMER` |
| Status | `Ok` | `Ok` | `Ok` |
| `net.peer.name` | `"ms"` | `"ms"` | `"ms"` |
| `net.peer.port` | `1234` | `1234` | `1234` |
| `messaging.system` | `"rabbitmq"` | `"rabbitmq"` | `"rabbitmq"` |
| `messaging.destination` | `"T"` | `"T"` | `"T"` |
| `messaging.destination_kind` | `"topic"` | `"topic"` | `"topic"` |
| `messaging.operation` |  | `"process"` | `"process"` |
| `messaging.message_id` | `"a1"` | `"a1"`| `"a1"` |

<!--
### Apache Kafka Example
-->

### Apache Kafkaの例

<!--
Given is a process P, that publishes a message to a topic T1 on Apache Kafka.
One process, CA, receives the message and publishes a new message to a topic T2 that is then received and processed by CB.
-->

プロセスPは、Apache Kafka上のトピックT1にメッセージを公開します。あるプロセスCAがそのメッセージを受信し、新しいメッセージをトピックT2にパブリッシュし、それをCBが受信して処理します。

<!--
```
Process P:  | Span Prod1 |
--
Process CA:              | Span Rcv1 |
                                | Span Proc1 |
                                  | Span Prod2 |
--
Process CB:                           | Span Rcv2 |
```
-->

```
Process P:  | Span Prod1 |
--
Process CA:              | Span Rcv1 |
                                | Span Proc1 |
                                  | Span Prod2 |
--
Process CB:                           | Span Rcv2 |
```

<!--
| Field or Attribute | Span Prod1 | Span Rcv1 | Span Proc1 | Span Prod2 | Span Rcv2
|-|-|-|-|-|-|
| Span name | `"T1 send"` | `"T1 receive"` | `"T1 process"` | `"T2 send"` | `"T2 receive`" |
| Parent |  | Span Prod1 | Span Rcv1 |  | Span Prod2 |
| Links |  |  | | Span Prod1 |  |
| SpanKind | `PRODUCER` | `CONSUMER` | `CONSUMER` | `PRODUCER` | `CONSUMER` |
| Status | `Ok` | `Ok` | `Ok` | `Ok` | `Ok` |
| `peer.service` | `"myKafka"` |  |  | `"myKafka"` |  |
| `service.name` |  | `"myConsumer1"` | `"myConsumer1"` |  | `"myConsumer2"` |
| `messaging.system` | `"kafka"` | `"kafka"` | `"kafka"` | `"kafka"` | `"kafka"` |
| `messaging.destination` | `"T1"` | `"T1"` | `"T1"` | `"T2"` | `"T2"` |
| `messaging.destination_kind` | `"topic"` | `"topic"` | `"topic"` | `"topic"` | `"topic"` |
| `messaging.operation` |  | `"receive"` | `"process"` |  | `"receive"` |
| `messaging.kafka.message_key` | `"myKey"` | `"myKey"` | `"myKey"` | `"anotherKey"` | `"anotherKey"` |
| `messaging.kafka.consumer_group` |  | `"my-group"` | `"my-group"` |  | `"another-group"` |
| `messaging.kafka.client_id` |  | `"5"` | `"5"` | `"5"` | `"8"` |
| `messaging.kafka.partition` |  | `"1"` | `"1"` |  | `"3"` |
-->

| フィールドまたは属性 | Span Prod1 | Span Rcv1 | Span Proc1 | Span Prod2 | Span Rcv2
|-|-|-|-|-|-|
| Span name | `"T1 send"` | `"T1 receive"` | `"T1 process"` | `"T2 send"` | `"T2 receive`" |
| Parent |  | Span Prod1 | Span Rcv1 |  | Span Prod2 |
| Links |  |  | | Span Prod1 |  |
| SpanKind | `PRODUCER` | `CONSUMER` | `CONSUMER` | `PRODUCER` | `CONSUMER` |
| Status | `Ok` | `Ok` | `Ok` | `Ok` | `Ok` |
| `peer.service` | `"myKafka"` |  |  | `"myKafka"` |  |
| `service.name` |  | `"myConsumer1"` | `"myConsumer1"` |  | `"myConsumer2"` |
| `messaging.system` | `"kafka"` | `"kafka"` | `"kafka"` | `"kafka"` | `"kafka"` |
| `messaging.destination` | `"T1"` | `"T1"` | `"T1"` | `"T2"` | `"T2"` |
| `messaging.destination_kind` | `"topic"` | `"topic"` | `"topic"` | `"topic"` | `"topic"` |
| `messaging.operation` |  | `"receive"` | `"process"` |  | `"receive"` |
| `messaging.kafka.message_key` | `"myKey"` | `"myKey"` | `"myKey"` | `"anotherKey"` | `"anotherKey"` |
| `messaging.kafka.consumer_group` |  | `"my-group"` | `"my-group"` |  | `"another-group"` |
| `messaging.kafka.client_id` |  | `"5"` | `"5"` | `"5"` | `"8"` |
| `messaging.kafka.partition` |  | `"1"` | `"1"` |  | `"3"` |

<!--
### Batch receiving
-->

### バッチ受信

<!--
Given is a process P, that sends two messages to a queue Q on messaging system MS, and a process C, which receives both of them in one batch (Span Recv1) and processes each message separately (Spans Proc1 and Proc2).
-->

メッセージングシステムMSのキューQに2つのメッセージを送信するプロセスPと、その2つのメッセージを1つのバッチで受信し(Span Recv1)、各メッセージを別々に処理するプロセスCです(Span Proc1、Proc2)。

<!--
Since a span can only have one parent and the propagated trace and span IDs are not known when the receiving span is started, the receiving span will have no parent and the processing spans are correlated with the producing spans using links.
-->

Spanは1つの親しか持つことができず、受信Spanの開始時には伝搬されたTraceとSpanのIDはわからないため、受信Spanは親を持たず、処理Spanはリンクを使用して生成Spanと関連づけられます。

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
Process P: | Span Prod1 | Span Prod2 |
--
Process C:                      | Span Recv1 |
                                        | Span Proc1 |
                                               | Span Proc2 |
```

<!--
| Field or Attribute | Span Prod1 | Span Prod2 | Span Recv1 | Span Proc1 | Span Proc2 |
|-|-|-|-|-|-|
| Span name | `"Q send"` | `"Q send"` | `"Q receive"` | `"Q process"` | `"Q process"` |
| Parent |  |  |  | Span Recv1 | Span Recv1 |
| Links |  |  |  | Span Prod1 | Span Prod2 |
| SpanKind | `PRODUCER` | `PRODUCER` | `CONSUMER` | `CONSUMER` | `CONSUMER` |
| Status | `Ok` | `Ok` | `Ok` | `Ok` | `Ok` |
| `net.peer.name` | `"ms"` | `"ms"` | `"ms"` | `"ms"` | `"ms"` |
| `net.peer.port` | `1234` | `1234` | `1234` | `1234` | `1234` |
| `messaging.system` | `"rabbitmq"` | `"rabbitmq"` | `"rabbitmq"` | `"rabbitmq"` | `"rabbitmq"` |
| `messaging.destination` | `"Q"` | `"Q"` | `"Q"` | `"Q"` | `"Q"` |
| `messaging.destination_kind` | `"queue"` | `"queue"` | `"queue"` | `"queue"` | `"queue"` |
| `messaging.operation` |  |  | `"receive"` | `"process"` | `"process"` |
| `messaging.message_id` | `"a1"` | `"a2"` | | `"a1"` | `"a2"` |
-->

| フィールドまたは属性 | Span Prod1 | Span Prod2 | Span Recv1 | Span Proc1 | Span Proc2 |
|-|-|-|-|-|-|
| Span name | `"Q send"` | `"Q send"` | `"Q receive"` | `"Q process"` | `"Q process"` |
| Parent |  |  |  | Span Recv1 | Span Recv1 |
| Links |  |  |  | Span Prod1 | Span Prod2 |
| SpanKind | `PRODUCER` | `PRODUCER` | `CONSUMER` | `CONSUMER` | `CONSUMER` |
| Status | `Ok` | `Ok` | `Ok` | `Ok` | `Ok` |
| `net.peer.name` | `"ms"` | `"ms"` | `"ms"` | `"ms"` | `"ms"` |
| `net.peer.port` | `1234` | `1234` | `1234` | `1234` | `1234` |
| `messaging.system` | `"rabbitmq"` | `"rabbitmq"` | `"rabbitmq"` | `"rabbitmq"` | `"rabbitmq"` |
| `messaging.destination` | `"Q"` | `"Q"` | `"Q"` | `"Q"` | `"Q"` |
| `messaging.destination_kind` | `"queue"` | `"queue"` | `"queue"` | `"queue"` | `"queue"` |
| `messaging.operation` |  |  | `"receive"` | `"process"` | `"process"` |
| `messaging.message_id` | `"a1"` | `"a2"` | | `"a1"` | `"a2"` |

<!--
### Batch processing
-->

### バッチ処理

<!--
Given is a process P, that sends two messages to a queue Q on messaging system MS, and a process C, which receives both of them separately (Span Recv1 and Recv2) and processes both messages in one batch (Span Proc1).
-->

メッセージング・システムMS上のキューQに2つのメッセージを送信するプロセスPと、その2つのメッセージを別々に受信し(Span Recv1とRecv2)、1つのバッチで2つのメッセージを処理するプロセスCです(Span Proc1)の例です。

<!--
Since each span can only have one parent, C3 should not choose a random parent out of C1 and C2, but rather rely on the implicitly selected parent as defined by the [tracing API spec](../api.md).
Similarly, only one value can be set as `message_id`, so C3 cannot report both `a1` and `a2` and therefore attribute is left out.
Depending on the implementation, the producing spans might still be available in the meta data of the messages and should be added to C3 as links.
The client library or application could also add the receiver span's SpanContext to the data structure it returns for each message. In this case, C3 could also add links to the receiver spans C1 and C2.
-->

各Spanは1つの親しか持つことができないので、C3はC1とC2の中からランダムに親を選ぶのではなく、[tracing API spec](../api.md)で定義されている暗黙的に選択された親に依存する必要があります。同様に、`message_id`として設定できる値は1つだけなので、C3は`a1`と`a2`の両方を報告することはできず、属性は省略されます。実装によっては、メッセージのメタデータの中にプロデュースSpanが残っている場合があるので、リンクとしてC3に追加する必要があります。また、クライアント・ライブラリやアプリケーションは、受信側SpanのSpanContextを、各メッセージに対して返すデータ構造に追加することもできます。この場合、C3はレシーバー・SpanC1とC2へのリンクも追加することもできます。

<!--
The status of the batch processing span is selected by the application. Depending on the semantics of the operation. A span status `Ok` could, for example, be set only if all messages or if just at least one were properly processed.
-->

バッチ処理のSpanの状態は、アプリケーションによって選択されます。操作のセマンティクスに応じて。例えば、すべてのメッセージが正しく処理された場合や、少なくとも1つのメッセージが正しく処理された場合にのみ、Spanのステータス`Ok`を設定することができます。

<!--
```
Process P: | Span Prod1 | Span Prod2 |
--
Process C:                              | Span Recv1 | Span Recv2 |
                                                                   | Span Proc1 |
```
-->

```
Process P: | Span Prod1 | Span Prod2 |
--
Process C:                              | Span Recv1 | Span Recv2 |
                                                                   | Span Proc1 |
```

<!--
| Field or Attribute | Span Prod1 | Span Prod2 | Span Recv1 | Span Recv2 | Span Proc1 |
|-|-|-|-|-|-|
| Span name | `"Q send"` | `"Q send"` | `"Q receive"` | `"Q receive"` | `"Q process"` |
| Parent |  |  | Span Prod1 | Span Prod2 |  |
| Links |  |  |  |  | Span Prod1 + Prod2 |
| SpanKind | `PRODUCER` | `PRODUCER` | `CONSUMER` | `CONSUMER` | `CONSUMER` |
| Status | `Ok` | `Ok` | `Ok` | `Ok` | `Ok` |
| `net.peer.name` | `"ms"` | `"ms"` | `"ms"` | `"ms"` | `"ms"` |
| `net.peer.port` | `1234` | `1234` | `1234` | `1234` | `1234` |
| `messaging.system` | `"rabbitmq"` | `"rabbitmq"` | `"rabbitmq"` | `"rabbitmq"` | `"rabbitmq"` |
| `messaging.destination` | `"Q"` | `"Q"` | `"Q"` | `"Q"` | `"Q"` |
| `messaging.destination_kind` | `"queue"` | `"queue"` | `"queue"` | `"queue"` | `"queue"` |
| `messaging.operation` |  |  | `"receive"` | `"receive"` | `"process"` |
| `messaging.message_id` | `"a1"` | `"a2"` | `"a1"` | `"a2"` | |
-->

| Field or Attribute | Span Prod1 | Span Prod2 | Span Recv1 | Span Recv2 | Span Proc1 |
|-|-|-|-|-|-|
| Span name | `"Q send"` | `"Q send"` | `"Q receive"` | `"Q receive"` | `"Q process"` |
| Parent |  |  | Span Prod1 | Span Prod2 |  |
| Links |  |  |  |  | Span Prod1 + Prod2 |
| SpanKind | `PRODUCER` | `PRODUCER` | `CONSUMER` | `CONSUMER` | `CONSUMER` |
| Status | `Ok` | `Ok` | `Ok` | `Ok` | `Ok` |
| `net.peer.name` | `"ms"` | `"ms"` | `"ms"` | `"ms"` | `"ms"` |
| `net.peer.port` | `1234` | `1234` | `1234` | `1234` | `1234` |
| `messaging.system` | `"rabbitmq"` | `"rabbitmq"` | `"rabbitmq"` | `"rabbitmq"` | `"rabbitmq"` |
| `messaging.destination` | `"Q"` | `"Q"` | `"Q"` | `"Q"` | `"Q"` |
| `messaging.destination_kind` | `"queue"` | `"queue"` | `"queue"` | `"queue"` | `"queue"` |
| `messaging.operation` |  |  | `"receive"` | `"receive"` | `"process"` |
| `messaging.message_id` | `"a1"` | `"a2"` | `"a1"` | `"a2"` | |

