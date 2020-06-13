<!--
# OpenTelemetry Protocol Requirements
-->

# OpenTelemetry プロトコルの要求仕様

<!--
This document will drive OpenTelemetry Protocol design and RFC.
-->

この文書にはOpenTelemetry プロトコルの設計とRFCを記します。

<!--
## Goals
-->

## ゴール

<!--
See the goals of OpenTelemetry Protocol design [here](design-goals.md).
-->

[OpenTelemetry プロトコルにおける設計のゴール](design-goals.md)を参照してください。

<!--
## Vocabulary
-->

## 用語の定義

<!--
There are 2 parties involved in telemetry data exchange. In this document the party that is the source of telemetry data is called the Client, the party that is the destination of telemetry data is called the Server.
-->

テレメトリーデータの交換には、2者が関与します。本文書では、テレメトリーデータの送信元をクライアントと呼び、テレメトリデータの送信先をサーバーと呼びます。

<!--
Examples of a Client are instrumented applications or sending side of telemetry collectors, examples of Servers are telemetry backends or receiving side of telemetry collectors (so a Collector is typically both a Client and a Server depending on which side you look from).
-->

クライアントの例としては、アプリケーションやテレメトリーCollectorの送信側があります。サーバーの例としては、テレメトリーバックエンドやテレメトリーCollectorの受信側があります(Collectorは通常クライアントとサーバーの両方を兼ねており、どちらの側から見るかによって呼び方が変わります)。

<!--
## Known Issues with Existing Protocols
-->

## 既存プロトコルにおける既知の問題

<!--
Our experience with OpenCensus and other protocols has been that many of them have one or more of the following drawbacks:
-->

OpenCensusや他のプロトコルの経験から、多くのプロトコルには以下のような欠点が1つ以上あります:

<!--
- High CPU consumption for serialization and especially deserialization of received telemetry data.
- High and frequent CPU consumption by Garbage Collector.
- Lack of delivery guarantees for certain protocols (e.g. stream-based gRPC OpenCensus protocol) which makes troubleshooting of telemetry pipelines difficult.
- Not aware / not cooperating with load balancers resulting in potentially large imbalances in horizontally scaled backends.
- Support either traces or metrics but not both.
-->

- シリアライズ、特に受信したテレメトリーデータのデシリアライズのためのCPU使用量が大きい
- ガベージコレクタによる高く、高頻度なCPU使用量
- 特定のプロトコル(ストリームベースの gRPC OpenCensus プロトコルなど)には配信保証がない。テレメトリーパイプラインのトラブルシューティングが困難
- ロードバランサーを無視、あるいは、協力していないため、水平方向にスケーリングされたバックエンドに大きな不均衡が生じる可能性がある
- トレースまたはメトリックのどちらかをサポートしているが、両方はサポートしていない

<!--
Our goal is to avoid or mitigate these known issues in the new protocol.
-->

私たちの目標は、新しいプロトコルにおけるこれらの既知の問題を回避したり、軽減したりすることです。

<!--
## Requirements
-->

## 要求仕様

<!--
The following are OpenTelemetry protocol requirements.
-->

以下にOpenTelemetryプロトコルの要求仕様を示します。

<!--
### Supported Node Types
-->

### サポートすべきノードの種類

<!--
The protocol must be suitable for use between all of the following node types: instrumented applications, telemetry backends, telemetry agents running as local daemons, stand-alone collector/forwarder services.
-->

以下のすべてのノードの種類をサポートすること(MUST): アプリケーションへの計装、テレメトリーバックエンド、ローカルエージェント、スタンドアローン collector/forwarder。

<!--
### Supported Data Types
-->

### サポートすべきデータの種類

<!--
The protocol must support traces and metrics as data types.
-->

トレースとメトリックをデータの種類としてサポートすること(MUST)。

<!--
### Reliability of Delivery
-->

### 配信の信頼性

<!--
The protocol must ensure reliable data delivery and clear visibility when the data cannot be delivered. This should be achieved by sending data acknowledgements from the Server to the Client.
-->

信頼性の高いデータ配信と、データが配信できない場合は明確な理由を出力すること(MUST)。これは、サーバーからクライアントにデータの確認応答を送信することで達成されなければなりません(SHOULD)。

<!--
Note that acknowledgements alone are not sufficient to guarantee that: a) no data will be lost and b) no data will be duplicated. Acknowledgements can help to guarantee a) but not b). Guaranteeing both at the same is difficult. Because it is usually preferable for telemetry data to be duplicated than to lose it, we choose to guarantee that there are no data losses while potentially allowing duplicate data.
-->

a) データが失われないこと、b) データが重複しないこと、を保証するには、確認応答(Ack)だけでは十分ではないことに注意してください。確認応答は a) を保証するのに役立ちますが、b) を保証するのには役立ちません。両方を同時に保証することは困難です。テレメトリーデータは通常、データを失うよりも重複する方が望ましいので、データが重複する可能性がある一方で、データの損失がないことを保証することを選択します。

<!--
Duplicates can typically happen in edge cases (e.g. on reconnections, network interruptions, etc) when the client has no way of knowing if last sent data was delivered. In these cases the client will usually choose to re-send the data to guarantee the delivery which in turn may result in duplicate data on the server side.
-->

重複は通常、クライアントが最後に送信したデータが受信されたかどうかを知る方法がないエッジケース(再接続やネットワークの中断など)で発生する可能性があります。このような場合、クライアントは通常、受信されたことを保証するためにデータを再送信しますが、その結果、サーバ側でデータが重複する可能性があります。

<!--
_To avoid having duplicates the client and the server could track sent and delivered items using uniquely identifying ids. The exact mechanism for tracking the ids and performing data de-duplication may be defined at the layer above the protocol layer and is outside the scope of this document._
-->

_重複を避けるために、クライアントとサーバは一意に識別されるIDを使用して、送信されたアイテムと受信されたアイテムを追跡できます。IDを追跡してデータの重複排除を行うための正確なメカニズムはプロトコル層より上の層で定義されるため、この文書の範囲外です_

<!--
For this reason we have slightly relaxed requirements and consider duplicate data acceptable in rare cases.
-->

このため、要求仕様を若干緩和し、まれにデータの重複が起こることを許容します。

<!--
Note: this protocol is concerned with reliability of delivery between one pair of client/server nodes and aims to ensure that no data is lost in-transit between the client and the server. Many telemetry collection systems have multiple nodes that the data must travel across until reaching the final destination (e.g. application -> agent -> collector -> backend). End-to-end delivery guarantees in such systems is outside of the scope for this document. The acknowledgements described in this protocol happen between a single client/server pair and do not span multiple nodes in multi-hop delivery paths.
-->

注意: このプロトコルは、一対のクライアント/サーバ間の送信の信頼性に着目しており、クライアントとサーバ間の転送中にデータが失われることがないことを目的としています。多くのテレメトリー収集システムには、最終目的地に到達するまでには複数のノードをデータが移動しなければないけません(例: アプリケーション -> エージェント -> collector -> バックエンド)。このようなシステムにおけるエンドツーエンドのデータ配信の保証はこの文書の範囲外です。このプロトコルで説明されている確認応答は、一対のクライアントとサーバの間で行われ、マルチホップで配信されたとしても、複数のノードにまたがることはありません。

<!--
### Throughput
-->

### スループット

<!--
The protocol must ensure high throughput in high latency networks when the client and the server are not in the same data center.
-->

クライアントとサーバが同じデータセンターに存在しない場合に、高遅延ネットワークで高いスループットを確保すること(MUST)。

<!--
This requirement may rule out half-duplex protocols. The throughput of half-duplex protocols is highly dependent on network roundtrip time and request size. To achieve good throughput request sizes may be too large to be practical.
-->

この要求仕様は、半二重プロトコルでは達成できない可能性があります。半二重プロトコルのスループットは、ネットワークのラウンドトリップ時間とリクエストのサイズに大きく依存します。良いスループットを達成するためには、リクエストサイズが大きすぎて実用的ではないかもしれません。

<!--
### Compression
-->

### 圧縮

<!--
The protocol must achieve high compression ratios for telemetry data. The protocol design must consider batching of telemetry data and grouping of similar data (both can help to achieve better compression using common compression algorithms).
-->

テレメトリーデータを高い効率で圧縮できること(MUST)。プロトコルの設計では、テレメトリーデータのバッチ化と類似データのグループ化を考慮する必要があります(MUST)(どちらも、標準的な圧縮アルゴリズムを使用してより良い圧縮を達成するのに役立ちます)。


<!--
### Encryption
-->

### 暗号化

<!--
Industry standard encryption (e.g. TLS/HTTPS) must be supported.
-->

業界標準の暗号化(例: TLS/HTTPS)をサポートする必要があります(MUST)。

<!--
### Backpressure Signalling and Throttling
-->

### バックプレッシャーシグナリングとスロットリング

<!--
The protocol must allow backpressure signalling.
-->

バックプレッシャーシグナリングを使用可能にすること(MUST)。

<!--
If the server is unable to keep up with the pace of data it receives from the client then it must be able to signal that fact to the client. The client may then throttle itself to avoid overwhelming the server.
-->

サーバがクライアントから受け取るデータのペースについていけない場合は、そのことをクライアントに知らせることができる必要があります(MUST)。クライアントは、サーバを溢れさせないように送信ペースを抑えることができます。

<!--
If the underlying transport is a stream that has its own flow control mechanism then the backpressure could be applied by delaying the reading of data from the server’s endpoint which could then be signalled to the client via underlying flow-control. However this approach makes it difficult for the client to distinguish server overloading from network delays (due to e.g. network losses). Such distinction is important for [observability reasons](https://github.com/open-telemetry/opentelemetry-service/pull/188). Because of this it is required for the protocol to allow to explicitly and clearly signal backpressure from the server to the client without relying on implicit signalling using underlying flow-control mechanisms.
-->

使用しているトランスポートが独自のフロー制御メカニズムを持つストリームである場合、サーバのエンドポイントからのデータの読み取りを遅らせることで、トランスポートのフロー制御を介してクライアントに通知して、バックプレッシャーを実現できます。しかし、この方法では、クライアントはサーバの過負荷と(例えばネットワーク切断などによる)ネットワークの遅延を区別することが困難になります。この区別は、[観測性の理由](https://github.com/open-telemetry/opentelemetry-service/pull/188)のために重要です。このため、プロトコルには、トランスポートのフロー制御メカニズムを使用した暗黙のシグナリングに頼ることなく、サーバからクライアントへのバックプレッシャーを明示的かつ明確にシグナリングできるようにすることが求められます。

<!--
The backpressure signal should include a hint to the client about desirable reduced rate of data.
-->

バックプレッシャーのシグナルには、データの望ましい低減率に関するクライアントへのヒントを含むことが求められます(SHOULD)。

<!--
### Serialization Performance
-->

### シリアライズ性能

<!--
The protocol must have fast data serialization and deserialization characteristics.
-->

高速なデータのシリアライズとデシリアライズができること(MUST)。

<!--
Ideally it must also support very fast pass-through mode (when no modifications to the data are needed), fast “augmenting” or “tagging” of data and partial inspection of data (e.g. check for presence of specific tag). These requirements help to create fast Agents and Collectors.
-->

理想的には、非常に高速なパススルーモード(データへの変更が必要ない場合)、データの高速な「付加」または「タグ付け」、データの部分的な検証(特定のタグの存在をチェックするなど)もサポートしていなければなりません(MUST)。これらの要件は、高速なAgentとCollectorの作成に役立ちます。

<!--
### Memory Usage Profile
-->

### メモリ使用率のプロファイル

<!--
The protocol must impose minimal pressure on memory manager, including pass-through scenarios, when deserialized data is short-lived and must be serialized as-is shortly after and when such short-lived data is created and discarded at high frequency (think telemetry data forwarders).
-->

デシリアライズされたデータが短命で、その後すぐにそのままシリアライズされなければならないようなパススルーのシナリオや、そのような短命のデータが高頻度で作成されて破棄されるような場合(テレメトリデータフォワーダを考えてみてください)を含めて、メモリ使用量を最小限に抑えること。(MUST)。

<!--
The implementation of telemetry protocol must aim to minimize the number of memory allocations and dealocations performed during serialization and deserialization and aim to minimize the pressure on Garbage Collection (for GC languages).
-->

テレメトリープロトコルの実装は、シリアライズとデシリアライズの間に実行されるメモリの割当と解放を最小化し、(GC言語のための)ガーベッジコレクションを最小化する必要があります(MUST)。

<!--
### Level 7 Load Balancer Friendly
-->

### Level 7 ロードバランサーとの親和性

<!--
The protocol must allow Level 7 load balancers such as Envoy to re-balance the traffic for each batch of telemetry data. The traffic should not get pinned by a load balancer to one server for the entire duration of telemetry data sending, thus potentially leading to imbalanced load of servers located behind the load balancer.
-->

Envoy のようなLevel 7 のロードバランサーが、テレメトリーデータの各バッチごとにトラフィックをリバランスできるようにすること(MUST)。トラフィックは、テレメトリーデータ送信の間全体において、ロードバランサーによって1台のサーバーに固定されないようにしなければいけません(SHOULD)。固定されていると、ロードバランサーの後ろにあるサーバーの負荷が不均衡になる可能性があります。

<!--
### Backwards Compatibility
-->

### 下位互換性

<!--
The protocol should be possible to evolve over time. It should be possible for nodes that implement different versions of OpenTelemetry protocol to interoperate (while possibly regressing to the lowest common denominator from functional perspective).
-->

時間の経過とともに進化することが可能でなければなりません(SHOULD)。OpenTelemetryプロトコルの異なるバージョンを実装したノードは(機能的な観点からは最小公約数に落ちてしまう可能性もあるが)相互運用できるようにすべきです(SHOULD)。

<!--
### General Requirements
-->

### 全体における要求事項

<!--
The protocol must use well-known, mature encoding and transport mechanisms with ubiquitous availability of implementations in wide selection of languages that are supported by OpenTelemetry.
-->

OpenTelemetry がサポートしている幅広い言語で実装が利用できるように、よく知られ、成熟したエンコーディングとトランスポートメカニズムを使用すること(MUST)。