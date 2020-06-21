<!--
# Design Goals for OpenTelemetry Wire Protocol
-->

# OpenTelemetryのワイヤプロトコルにおける設計のゴール

<!--
We want to design a telemetry data exchange protocol that has the following characteristics:
-->

私達はテレメトリーデータ交換プロトコルを以下の特徴を持つように設計したいと考えています。

<!--
- Be suitable for use between all of the following node types: instrumented applications, telemetry backends, local agents, stand-alone collectors/forwarders.
-->

- 以下のすべてのノードの種類に適していること: アプリケーションへの計装、テレメトリーバックエンド、ローカルエージェント、スタンドアローン Collector/Forwarder。

<!--
- Have high reliability of data delivery and clear visibility when the data cannot be delivered.
-->

- データ配信の信頼性が高く、データが配信できない場合の理由が明確であること。

<!--
- Have low CPU usage for serialization and deserialization.
-->

- シリアライズとデシリアライズに使うCPU使用量が少ないこと。

<!--
- Impose minimal pressure on memory manager, including pass-through scenarios, where deserialized data is short-lived and must be serialized as-is shortly after and where such short-lived data is created and discarded at high frequency (think telemetry data forwarders).
-->

- デシリアライズされたデータが短命で、その後すぐにそのままシリアライズされなければならないようなパススルーのシナリオや、そのような短命のデータが高頻度で作成・破棄されるような場合(テレメトリーデータの転送を考えてみてください)を含めて、メモリ使用量を最小限に抑えること。

<!--
- Support ability to efficiently modify deserialized data and serialize again to pass further. This is related but slightly different from the previous requirement.
-->

- デシリアライズされたデータを修正し、さらに別なところに渡すために再度シリアライズする効率的な機能をサポートすること。関連はありますが、上記の要件とは少し異なります。

<!--
- Ensure high throughput (within the available bandwidth) in high latency networks (e.g. scenarios where telemetry source and the backend are separated by high latency network).
-->

- 高遅延ネットワークにおいて、(利用可能な帯域幅の範囲内で)高いスループットを確保すること (例: テレメトリー元とバックエンドが高遅延ネットワークによって分離されているシナリオ)

<!--
- Allow backpressure signalling.
-->

- バックプレッシャーのシグナリングをサポートすること

<!--
- Be load-balancer friendly (do not hinder re-balancing).
-->

- ロードバランサーに優しいこと (リバランスを妨げないこと)