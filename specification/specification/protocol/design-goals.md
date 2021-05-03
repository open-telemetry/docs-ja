<!--
# Design Goals for OpenTelemetry Wire Protocol
-->

# OpenTelemetry Wireプロトコルのデザインゴール

<!--
We want to design a telemetry data exchange protocol that has the following characteristics:
-->

以下のような特徴を持つテレメトリデータ交換プロトコルを設計したいと考えています。

<!--
- Be suitable for use between all of the following node types: instrumented applications, telemetry backends, local agents, stand-alone collectors/forwarders.
-->

- 以下のすべてのタイプのノードでの使用に適していること: 計装されたアプリケーション、テレメトリ・バックエンド、ローカル・エージェント、スタンドアローン・コレクター/フォワーダーなど。

<!--
- Have high reliability of data delivery and clear visibility when the data cannot be delivered.
-->

- データ配信の信頼性が高く、データが配信されない場合も明確にわかること。

<!--
- Have low CPU usage for serialization and deserialization.
-->

- シリアライズとデシリアライズのCPU使用率が低いこと。

<!--
- Impose minimal pressure on memory manager, including pass-through scenarios, where deserialized data is short-lived and must be serialized as-is shortly after and where such short-lived data is created and discarded at high frequency (think telemetry data forwarders).
-->

- パススルーシナリオを含めて、メモリマネージャへの負荷を最小限に抑えること。デシリアライズされたデータが短命で、直後にそのままシリアライズする必要がある場合や、そのような短命のデータが高頻度で作成・破棄される場合(テレメトリデータフォワーダなど)を含みます。

<!--
- Support ability to efficiently modify deserialized data and serialize again to pass further. This is related but slightly different from the previous requirement.
-->

- デシリアライズされたデータを効率的に修正し、さらに渡すために再びシリアライズする機能をサポートすること。これは関連はしていますが、先ほどの要件とは少し異なります。

<!--
- Ensure high throughput (within the available bandwidth) in high latency networks (e.g. scenarios where telemetry source and the backend are separated by high latency network).
-->

- 高遅延ネットワークでの高スループット(利用可能な帯域幅)を確保すること。(例:テレメトリソースとバックエンドが高遅延ネットワークで分離されているシナリオ)

<!--
- Allow backpressure signalling.
-->

- バックプレッシャー・シグナリングを可能にすること

<!--
- Be load-balancer friendly (do not hinder re-balancing).
-->

- ロードバランサーフレンドリーであること(リバランシングの妨げにならないこと)

