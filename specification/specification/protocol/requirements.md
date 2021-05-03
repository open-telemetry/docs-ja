<!--
# OpenTelemetry Protocol Requirements
-->

# OpenTelemetry プロトコルの要求事項

<!--
This document will drive OpenTelemetry Protocol design and RFC.
-->

この文書は、OpenTelemetryプロトコルの設計とRFCを推進するものです。

<!--
## Goals
-->

## ゴール

<!--
See the goals of OpenTelemetry Protocol design [here](design-goals.md).
-->

OpenTelemetryプロトコル設計のゴールは[こちら](design-goals.md)をご覧ください。

<!--
## Vocabulary
-->

## 語彙

<!--
There are 2 parties involved in telemetry data exchange. In this document the party that is the source of telemetry data is called the Client, the party that is the destination of telemetry data is called the Server.
-->

テレメトリデータの交換には2つの当事者が存在します。このドキュメントでは、テレメトリデータの送信元側をクライアントと呼び、テレメトリデータの送信先側をサーバーと呼びます。


<!--
Examples of a Client are instrumented applications or sending side of telemetry collectors, examples of Servers are telemetry backends or receiving side of telemetry collectors (so a Collector is typically both a Client and a Server depending on which side you look from).
-->

クライアントの例としては、計装されたアプリケーションやテレメトリ・コレクターの送信側が挙げられ、サーバーの例としては、テレメトリ・バックエンドやテレメトリ・コレクターの受信側が挙げられます(したがって、コレクターは、どちらの側から見るかによって、一般的にクライアントとサーバーの両方になります)。

<!--
## Known Issues with Existing Protocols
-->

## 既存プロトコルの既知の問題点

<!--
Our experience with OpenCensus and other protocols has been that many of them have one or more of the following drawbacks:
-->

OpenCensusやその他のプロトコルを使用した経験から、多くのプロトコルは以下のような欠点があります。

<!--
- High CPU consumption for serialization and especially deserialization of received telemetry data.
- High and frequent CPU consumption by Garbage Collector.
- Lack of delivery guarantees for certain protocols (e.g. stream-based gRPC OpenCensus protocol) which makes troubleshooting of telemetry pipelines difficult.
- Not aware / not cooperating with load balancers resulting in potentially large imbalances in horizontally scaled backends.
- Support either traces or metrics but not both.
-->

- 受信したテレメトリデータのシリアライズ、特にデシリアライズのためのCPU消費が大きい
- ガベージ・コレクタによるCPU消費が大きく、頻繁に発生する
- 特定のプロトコル(例:ストリームベースのgRPC OpenCensusプロトコル)に対する配信保証がないため、テレメトリ・パイプラインのトラブルシューティングが困難になる
- ロードバランサーを認識していない、あるいは協力していないため、水平方向に拡張されたバックエンドで大きな不均衡が生じる可能性がある
- TraceまたはMetricsのいずれかをサポートするが、両方はサポートしない

<!--
Our goal is to avoid or mitigate these known issues in the new protocol.
-->

私たちの目標は、新しいプロトコルでこれらの既知の問題を回避または軽減することです。

<!--
## Requirements
-->

## 要求事項

<!--
The following are OpenTelemetry protocol requirements.
-->

以下は、OpenTelemetryプロトコルの要求事項です。

<!--
### Supported Node Types
-->

### サポートするノードの種類

<!--
The protocol must be suitable for use between all of the following node types: instrumented applications, telemetry backends, telemetry agents running as local daemons, stand-alone collector/forwarder services.
-->

プロトコルは、次のすべてのタイプのノード間での使用に適していなければなりません:計装済みアプリケーション、テレメトリ・バックエンド、ローカル・デーモンとして動作するテレメトリ・エージェント、スタンドアロンのコレクター/フォワーダーサービス。

<!--
### Supported Data Types
-->

### サポートするデータ型

<!--
The protocol must support traces and metrics as data types.
-->

プロトコルは、データタイプとしてTraceとMetricsをサポートしなければなりません。

<!--
### Reliability of Delivery
-->

### 配信の信頼性

<!--
The protocol must ensure reliable data delivery and clear visibility when the data cannot be delivered. This should be achieved by sending data acknowledgements from the Server to the Client.
-->

プロトコルは、信頼性の高いデータ配信と、データが配信されない場合の明確な視認性を確保する必要があります。これは、サーバーからクライアントへのデータ確認の送信によって達成されるべきです。

<!--
Note that acknowledgements alone are not sufficient to guarantee that: a) no data will be lost and b) no data will be duplicated. Acknowledgements can help to guarantee a) but not b). Guaranteeing both at the same is difficult. Because it is usually preferable for telemetry data to be duplicated than to lose it, we choose to guarantee that there are no data losses while potentially allowing duplicate data.
-->

確認応答(Ack)だけでは、a) データが失われないこと、b) データが複製されないことを保証するのに十分ではないことに注意してください。確認応答はa)を保証するのに役立ちますが、b)は保証できません。両方を同時に保証することは困難です。通常、テレメトリデータは失われるよりも複製される方が望ましいので、データの損失がないことを保証しつつ、データの複製を許容することを選択しました。


<!--
Duplicates can typically happen in edge cases (e.g. on reconnections, network interruptions, etc) when the client has no way of knowing if last sent data was delivered. In these cases the client will usually choose to re-send the data to guarantee the delivery which in turn may result in duplicate data on the server side.
-->

重複は、最後に送信されたデータが配信されたかどうかをクライアントが知る方法がないエッジケース(再接続、ネットワークの中断など)でよく起こります。このような場合、クライアントは通常、配信を保証するためにデータの再送信を選択しますが、その結果、サーバー側でデータが重複する可能性があります。

<!--
_To avoid having duplicates the client and the server could track sent and delivered items using uniquely identifying ids. The exact mechanism for tracking the ids and performing data de-duplication may be defined at the layer above the protocol layer and is outside the scope of this document._
-->

_重複を避けるために、クライアントとサーバーは、一意に識別できるIDを使って送信および配信されたアイテムを追跡できます。IDを追跡し、データの重複排除を行うための正確なメカニズムは、プロトコル層より上の層で定義される可能性があり、本ドキュメントの範囲外です。_

<!--
For this reason we have slightly relaxed requirements and consider duplicate data acceptable in rare cases.
-->

このような理由から、私たちは要件を若干緩和し、まれなケースではデータの重複を許容しています。

<!--
Note: this protocol is concerned with reliability of delivery between one pair of client/server nodes and aims to ensure that no data is lost in-transit between the client and the server. Many telemetry collection systems have multiple nodes that the data must travel across until reaching the final destination (e.g. application -> agent -> collector -> backend). End-to-end delivery guarantees in such systems is outside of the scope for this document. The acknowledgements described in this protocol happen between a single client/server pair and do not span multiple nodes in multi-hop delivery paths.
-->

注:このプロトコルは、一対のクライアント/サーバーのノード間の配信の信頼性に関するもので、クライアントとサーバー間の移動中にデータが失われないことを目的としています。多くのテレメトリ収集システムでは、データが最終目的地に到達するまでに、複数のノードを経由しなければなりません(例:アプリケーション -> エージェント -> コレクター -> バックエンド)。このようなシステムでのエンドツーエンドの配信保証は、本ドキュメントの範囲外です。このプロトコルに記載されている確認応答は、単一のクライアント/サーバーペアの間で行われ、マルチホップ配信パスの複数のノードにまたがることはありません。

<!--
### Throughput
-->

### スループット

<!--
The protocol must ensure high throughput in high latency networks when the client and the server are not in the same data center.
-->

このプロトコルは、クライアントとサーバーが同じデータセンターにない場合、高レイテンシーのネットワークにおいて高いスループットを確保する必要があります。

<!--
This requirement may rule out half-duplex protocols. The throughput of half-duplex protocols is highly dependent on network roundtrip time and request size. To achieve good throughput request sizes may be too large to be practical.
-->

この要件は、半二重プロトコルを除外する可能性があります。半二重プロトコルのスループットは、ネットワークのラウンドトリップタイムとリクエストサイズに大きく依存します。良好なスループットを得るためには、リクエスト・サイズが大きすぎて実用的ではないかもしれません。

<!--
### Compression
-->

### 圧縮

<!--
The protocol must achieve high compression ratios for telemetry data. The protocol design must consider batching of telemetry data and grouping of similar data (both can help to achieve better compression using common compression algorithms).
-->

プロトコルは、テレメトリデータの高い圧縮率を達成する必要があります。プロトコルの設計では、テレメトリデータのバッチ処理と類似データのグループ化を考慮する必要があります(どちらも一般的な圧縮アルゴリズムを使用してより良い圧縮を実現するのに役立ちます)。

<!--
### Encryption
-->

### 暗号化

<!--
Industry standard encryption (e.g. TLS/HTTPS) must be supported.
-->

業界標準の暗号化(TLS/HTTPSなど)に対応していること。

<!--
### Backpressure Signalling and Throttling
-->

### バックプレッシャー・シグナリングとスロットリング

<!--
The protocol must allow backpressure signalling.
-->

プロトコルはバックプレッシャー・シグナルを許可する必要があります。

<!--
If the server is unable to keep up with the pace of data it receives from the client then it must be able to signal that fact to the client. The client may then throttle itself to avoid overwhelming the server.
-->

もしサーバーがクライアントから受け取るデータのペースに追いつけない場合は、その事実をクライアントに知らせることができなければなりません。そうすればクライアントは、サーバーに負担をかけないように自分でスロットルできます。

<!--
If the underlying transport is a stream that has its own flow control mechanism then the backpressure could be applied by delaying the reading of data from the server’s endpoint which could then be signalled to the client via underlying flow-control. However this approach makes it difficult for the client to distinguish server overloading from network delays (due to e.g. network losses). Such distinction is important for [observability reasons](https://github.com/open-telemetry/opentelemetry-service/pull/188). Because of this it is required for the protocol to allow to explicitly and clearly signal backpressure from the server to the client without relying on implicit signalling using underlying flow-control mechanisms.
-->

もし下層のトランスポートが独自のフロー制御メカニズムを持つストリームである場合、サーバーのエンドポイントからのデータ読み取りを遅らせることでバックプレッシャーをかけ、下層のフロー制御を介してクライアントに通知することができます。しかし、この方法では、クライアントがサーバーの過負荷と(ネットワークの損失などによる)ネットワークの遅延とを区別することが困難になります。このような区別は、[observability reasons](https://github.com/open-telemetry/opentelemetry-service/pull/188)のために重要です。このため、プロトコルには、基盤となるフロー制御メカニズムを用いた暗黙のシグナリングに頼ることなく、サーバーからクライアントへのバックプレッシャーを明示的かつ明確にシグナリングできることが求められます。


<!--
The backpressure signal should include a hint to the client about desirable reduced rate of data.
-->

バックプレッシャー・シグナルには、データレートの低下が望ましいことを示すクライアントへのヒントが含まれている必要があります。

<!--
### Serialization Performance
-->

### シリアライズ性能

<!--
The protocol must have fast data serialization and deserialization characteristics.
-->

このプロトコルは、データのシリアライズとデシリアライズが高速であることが必要です。

<!--
Ideally it must also support very fast pass-through mode (when no modifications to the data are needed), fast “augmenting” or “tagging” of data and partial inspection of data (e.g. check for presence of specific tag). These requirements help to create fast Agents and Collectors.
-->

理想的には、非常に高速なパススルーモード(データの修正が必要ない場合)、データの高速な「補強」または「タグ付け」、データの部分的な検査(特定のタグの存在を確認するなど)もサポートしなければなりません。これらの要件を満たすことで、高速なAgentとCollectorが実現します。


<!--
### Memory Usage Profile
-->

### メモリ使用量プロファイル

<!--
The protocol must impose minimal pressure on memory manager, including pass-through scenarios, when deserialized data is short-lived and must be serialized as-is shortly after and when such short-lived data is created and discarded at high frequency (think telemetry data forwarders).
-->

このプロトコルは、パススルーシナリオを含めて、メモリマネージャへの負担を最小限にしなければなりません。パススルーシナリオとは、シリアライズされたデータが短命で、直後にそのままシリアライズしなければならない場合や、そのような短命のデータが高頻度で作成され、破棄されるというシナリオです(テレメトリデータ・フォワーダを考えてみてください)。

<!--
The implementation of telemetry protocol must aim to minimize the number of memory allocations and dealocations performed during serialization and deserialization and aim to minimize the pressure on Garbage Collection (for GC languages).
-->

テレメトリプロトコルの実装では、シリアライズとデシリアライズの際に実行されるメモリの割り当てと解放の数を最小限にし、ガベージコレクション(GC言語の場合)への負担を最小限にすることを目的としなければなりません。

<!--
### Level 7 Load Balancer Friendly
-->

### レベル7ロードバランサーフレンドリー

<!--
The protocol must allow Level 7 load balancers such as Envoy to re-balance the traffic for each batch of telemetry data. The traffic should not get pinned by a load balancer to one server for the entire duration of telemetry data sending, thus potentially leading to imbalanced load of servers located behind the load balancer.
-->

プロトコルは、Envoyのようなレベル7のロードバランサーが、テレメトリデータのバッチごとにトラフィックをリバランスさせることを可能にしなければなりません。テレメトリデータの送信期間中、ロードバランサーによってトラフィックが1つのサーバーに固定されることがあってはなりません。この場合、ロードバランサーの背後にあるサーバーの負荷が不均衡になる可能性があります。

<!--
### Backwards Compatibility
-->

### 後方互換性

<!--
The protocol should be possible to evolve over time. It should be possible for nodes that implement different versions of OpenTelemetry protocol to interoperate (while possibly regressing to the lowest common denominator from functional perspective).
-->

プロトコルは時間の経過とともに進化することが可能であること。異なるバージョンのOpenTelemetryプロトコルを実装したノードが相互運用できるようにすべきです(ただし、機能面では最小の共通機能セットに退化する可能性があります)。


<!--
### General Requirements
-->

### 一般的な要件

<!--
The protocol must use well-known, mature encoding and transport mechanisms with ubiquitous availability of implementations in wide selection of languages that are supported by OpenTelemetry.
-->

The protocol must use well-known, mature encoding and transport mechanisms with ubiquitous availability of implementations in wide selection of languages that are supported by OpenTelemetry.

