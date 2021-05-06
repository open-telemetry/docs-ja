<!--
# Trace Semantic Conventions
-->

# Trace セマンティック規約

<!--
**Status**: [Experimental](../../document-status.md)
-->

**Status**: [Experimental](../../document-status.md)

<!--
In OpenTelemetry spans can be created freely and it’s up to the implementor to
annotate them with attributes specific to the represented operation. Spans
represent specific operations in and between systems. Some of these operations
represent calls that use well-known protocols like HTTP or database calls.
Depending on the protocol and the type of operation, additional information
is needed to represent and analyze a span correctly in monitoring systems. It is
also important to unify how this attribution is made in different languages.
This way, the operator will not need to learn specifics of a language and
telemetry collected from polyglot (multi-language) micro-service environments
can still be easily correlated and cross-analyzed.
-->

OpenTelemetryでは、Spanは自由に作成することができ、表現される操作に特有の属性でSpanをアノテートするのは実装者の責任です。Spanは、システム内およびシステム間の特定の操作を表します。これらの操作の中には、HTTPやデータベースコールなどのよく知られたプロトコルを使用するコールを表すものがあります。プロトコルや操作の種類によっては、監視システムでSpanを正しく表現し、分析するための追加情報が必要になります。また、これらの属性を異なる言語で統一することも重要です。こうすることで、オペレータは言語の詳細を学ぶ必要がなくなり、ポリグロット(多言語)のマイクロサービス環境から収集されたテレメトリは、依然として容易に相関関係を保ち、相互に分析することができます。

<!--
The following semantic conventions for spans are defined:
-->

Spanのセマンティック規約は以下のように定義されています。

<!--
* [General](span-general.md): General semantic attributes that may be used in describing different kinds of operations.
* [HTTP](http.md): Spans for HTTP client and server.
* [Database](database.md): Spans for SQL and NoSQL client calls.
* [RPC/RMI](rpc.md): Spans for remote procedure calls (e.g., gRPC).
* [Messaging](messaging.md): Spans for interaction with messaging systems (queues, publish/subscribe, etc.).
* [FaaS](faas.md): Spans for Function as a Service (e.g., AWS Lambda).
* [Exceptions](exceptions.md): Attributes for recording exceptions associated with a span.
-->

* [General](span-general.md): 異なる種類の操作を記述する際に使用される可能性のある一般的なセマンティック属性
* [HTTP](http.md): HTTPクライアントとサーバーのSpan
* [Database](database.md): SQLとNoSQLのクライアントコールに対応したSpan
* [RPC/RMI](rpc.md): リモートプロシージャコール(例:gRPC)のSpan
* [Messaging](messaging.md): メッセージングシステム(キュー、パブリッシュ/サブスクライブなど)とのやりとりのためのSpan
* [FaaS](faas.md): Function as a Service(例:AWS Lambda)のためのSpan
* [Exceptions](exceptions.md): Spanに関連する例外を記録するための属性

<!--
The following library-specific semantic conventions are defined:
-->

以下のライブラリ固有のセマンティック・コンベンションが定義されています。

<!--
* [AWS Lambda](instrumentation/aws-lambda.md): AWS Lambda
* [AWS SDK](instrumentation/aws-sdk.md): AWS SDK
-->

* [AWS Lambda](instrumentation/aws-lambda.md): AWS Lambda 
* [AWS SDK](instrumentation/aws-sdk.md): AWS SDK

<!--
Apart from semantic conventions for traces and [metrics](../../metrics/semantic_conventions/README.md),
OpenTelemetry also defines the concept of overarching [Resources](../../resource/sdk.md) with their own
[Resource Semantic Conventions](../../resource/semantic_conventions/README.md).
-->

Traceや[metricsのセマンティック規約](../../metrics/semantic_conventions/README.md)とは別に、OpenTelemetryは独自の[Resourceセマンティック規約](../../resource/sdk.md)を持つ包括的な[Resources]の概念も定義しています。
