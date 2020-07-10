<!-- # Trace Semantic Conventions -->

# Traceのセマンティック規約

<!-- In OpenTelemetry spans can be created freely and it’s up to the implementor to
annotate them with attributes specific to the represented operation. Spans
represent specific operations in and between systems. Some of these operations
represent calls that use well-known protocols like HTTP or database calls.
Depending on the protocol and the type of operation, additional information
is needed to represent and analyze a span correctly in monitoring systems. It is
also important to unify how this attribution is made in different languages.
This way, the operator will not need to learn specifics of a language and
telemetry collected from polyglot (multi-language) micro-service environments
can still be easily correlated and cross-analyzed. -->

OpenTelemetryでは実装者は自由にSpanを作成でき、また表現された操作に固有の属性でアノテーションをつけることができます。 Spanはシステム中やシステム間の特定の操作を表します。ある操作はHTTPやデータベース呼び出しのようなよく知られたプロトコルを表します。モニタリングシステムでSpanを正しく表したり分析するためにどんな付加情報が必要になるかは、どのようなプロトコルか、またどのようなタイプの操作か次第で変わります。また異なる言語においてどのようにその属性が作られたかを統一することが重要になります。それによって作業者は複数の言語やテレメトリーを学ぶことなしにポリグリット（多言語）なマイクロサービス環境を相互比較したり横断的に分析したりできるのです。

<!-- The following semantic conventions for spans are defined: -->

ここでは以下のようなセマンティック規約が定義されています。

<!-- * [General](span-general.md): General semantic attributes that may be used in describing different kinds of operations.
* [HTTP](http.md): Spans for HTTP client and server.
* [Database](database.md): Spans for SQL and NoSQL client calls.
* [RPC/RMI](rpc.md): Spans for remote procedure calls (e.g., gRPC).
* [Messaging](messaging.md): Spans for interaction with messaging systems (queues, publish/subscribe, etc.).
* [FaaS](faas.md): Spans for Function as a Service (e.g., AWS Lambda). -->

* [General](span-general.md): 異なる種類の操作を記述する際に使われる一般的なセマンティック属性
* [HTTP](http.md): HTTPクライアント/サーバーにおけるSpan
* [Database](database.md): SQLやNoSQLのクライアント呼び出しのSpan
* [RPC/RMI](rpc.md): gRPCなどのリモートプロシージャコールのSpan
* [Messaging](messaging.md): キューイングシステムやpub/subなどのインタラクティブメッセージシステムのSpan
* [FaaS](faas.md): AWS LambdaなどのFunction as a ServiceのSpan

<!-- Apart from semantic conventions for traces and [metrics](../../metrics/semantic_conventions/README.md),
OpenTelemetry also defines the concept of overarching [Resources](../../resource/sdk.md) with their own
[Resource Semantic Conventions](../../resource/semantic_conventions/README.md). -->

traceや[metrics](../../metrics/semantic_conventions/README.md)のセマンティック規約の他に、OpenTelemetryでは [Resources](../../resource/sdk.md) の包括的なコンセプトも[Resource のセマンティック規約](../../resource/semantic_conventions/README.md) と共に定義されています。
