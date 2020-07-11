<!--
# Semantic conventions for FaaS spans
-->

# FaaS Spanのセマンティック規約

<!--
This document defines how to describe an instance of a function that runs without provisioning or managing of servers (also known as serverless) with spans.
-->

この文書では、サーバのプロビジョニングや管理を行わずに動作する機能(サーバレスとも呼ばれる)のインスタンスをSpanで記述する方法を定義しています。


<!-- Re-generate TOC with `markdown-toc --no-first-h1 -i` -->


<!-- toc -->

<!--
- [General Attributes](#general-attributes)
  * [Difference between execution and instance](#difference-between-execution-and-instance)
- [Function Trigger Type](#function-trigger-type)
  * [Datasource](#datasource)
  * [HTTP](#http)
  * [PubSub](#pubsub)
  * [Timer](#timer)
  * [Other](#other)
-->

- [一般的な属性](#一般的な属性)
  * [実行とインスタンスの違い](#実行とインスタンスの違い)
- [関数トリガーの種類](#関数トリガーの種類)
  * [Datasource](#datasource)
  * [HTTP](#http)
  * [PubSub](#pubsub)
  * [Timer](#timer)
  * [Other](#other)

<!-- tocstop -->

<!--
## General Attributes
-->

## 一般的な属性

<!--
Span `name` should be set to the function name being executed. Depending on the value of the `faas.trigger` attribute, additional attributes MUST be set. For example, an `http` trigger SHOULD follow the [HTTP Server semantic conventions](http.md#http-server-semantic-conventions). For more information, refer to the [Function Trigger Type](#function-trigger-type) section.
-->

Span `name` には、実行される関数名を設定しなければなりません(SHOULD)。 `faas.trigger` 属性の値に応じて、追加の属性を設定しなければなりません(MUST)。例えば、`http`のトリガは、[HTTPサーバーセマンティック規約](http.md#HTTPサーバーセマンティック規約)に従うべきです(SHOULD)。詳細は、[関数トリガーの種類](#関数トリガーの種類)のセクションを参照してください。

<!--
If Spans following this convention are produced, a Resource of type `faas` MUST exist following the [Resource semantic convention](../../resource/semantic_conventions/README.md#function-as-a-service).
-->

この規約に従ったSpanが生成された場合、`faas` 型のResourceは [Resourceのセマンティック規約](./.../resource/semantic_conventions/README.md#function-as-a-service) に従った形で存在しなければなりません(MUST)。

<!--
| Attribute name  | Notes  and examples  | Required? |
|---|---|--|
| `faas.trigger` | Type of the trigger on which the function is executed. <br > It SHOULD be one of the following strings: "datasource", "http", "pubsub", "timer", or "other". | Yes |
| `faas.execution` | String containing the execution id of the function. E.g. `af9d5aa4-a685-4c5f-a22b-444f80b3cc28` | No |
-->

| 属性名 | 説明と例                             | Required? |
|---|---|--|
| `faas.trigger` | 関数が実行されるトリガーの型。<br> 以下の文字列のいずれかであるべきです(SHOULD): "datasource"、 "http"、 "pubsub"、 "timer"、  "other". | Yes |
| `faas.execution` | 関数の実行IDを含む文字列。例えば `af9d5aa4-a685-4c5f-a22b-444f80b3cc28` | No |

<!--
### Difference between execution and instance
-->

### 実行とインスタンスの違い

<!--
For performance reasons (e.g. [AWS lambda], or [Azure functions]), FaaS providers allocate an execution environment for a single instance of a function that is used to serve multiple requests.
Developers exploit this fact to solve the **cold start** issue, caching expensive resource computations between different function executions.
Furthermore, FaaS providers encourage this behavior, e.g. [Google functions].
This field MAY be set to help correlate function executions that belong to the same execution environment.
The span attribute `faas.execution` differs from the resource attribute `faas.instance` in the following:
-->


パフォーマンス上の理由([AWS lambda]や[Azure functions]など)から、FaaSプロバイダーは、単一インスタンスに対して複数のリクエストの実行環境を割り当てます。開発者はこの事実を利用して **コールドスタート** の問題を解決し、異なる関数の実行間で高額なリソース計算をキャッシュしています。さらに、例えば[Google functions]などのFaaSプロバイダーはこの動作を奨励しています。このフィールドは、同じ実行環境に属する関数の実行を相関させるために設定しても構いません(MAY)。Span属性 `faas.execution` はリソース属性 `faas.instance` とは以下のように異なります。

<!--
- `faas.execution` refers to the current request ID handled by the function;
- `faas.instance` refers to the execution environment ID of the function.
-->

- `faas.execution` は関数が現在処理しているリクエストIDを参照します。
- `faas.instance` は関数の実行環境IDを示します。

<!--
[AWS lambda]: https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html
[Azure functions]: https://docs.microsoft.com/en-us/azure/azure-functions/manage-connections#static-clients
[Google functions]: https://cloud.google.com/functions/docs/concepts/exec#function_scope_versus_global_scope
-->

[AWS lambda]: https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html
[Azure functions]: https://docs.microsoft.com/en-us/azure/azure-functions/manage-connections#static-clients
[Google functions]: https://cloud.google.com/functions/docs/concepts/exec#function_scope_versus_global_scope

<!--
## Function Trigger Type
-->

## 関数トリガーの種類

<!--
This section describes how to handle the span creation and additional attributes based on the value of the attribute `faas.trigger`.
-->

ここでは、属性 `faas.trigger` の値に基づいて、Spanの作成と追加属性を処理する方法を説明します。

<!--
### Datasource
-->

### Datasource

<!--
A datasource function is triggered as a response to some data source operation such as a database or filesystem read/write.
For `faas` spans with trigger `datasource`, it is recommended to set the following attributes.
-->

Datasource関数は、データベースやファイルシステムの読み書きなどのデータソース操作に対する応答としてトリガされます。トリガー `datasource` を持つ `faas` Spanには、以下の属性を設定することを推奨します。


<!--
| Attribute name  | Notes  and examples  | Required? |
|---|---|--|
| `faas.document.collection` | The name of the source on which the operation was perfomed. For example, in Cloud Storage or S3 corresponds to the bucket name, and in Cosmos DB to the database name. | Yes |
| `faas.document.operation`  | Describes the type of the operation that was performed on the data.<br /> It SHOULD be one of the following strings: "insert", "edit", "delete". | Yes |
| `faas.document.time`       | A string containing the time when the data was accessed in the [ISO 8601] format expressed in [UTC]. E.g. `"2020-01-23T13:47:06Z"` | Yes |
| `faas.document.name`       | The document name/table subjected to the operation.<br /> For example, in Cloud Storage or S3 is the name of the file, and in Cosmos DB the table name.  | No |
-->

| 属性名 | 説明と例                                           | Required? |
|---|---|--|
| `faas.document.collection` | 操作が実行されたソースの名前。例えば、Cloud StorageやS3ではバケット名、Cosmos DBではデータベース名に対応します | Yes |
| `faas.document.operation`  | データに対して実行された操作の種類を記述します。<br /> 以下の文字列のいずれかであるべきです(SHOULD): "insert", "edit", "delete" | Yes |
| `faas.document.time`       | データにアクセスした時刻を含む文字列を[ISO 8601]形式で[UTC]で表現したもの。例: `"2020-01-23T13:47:06Z" `| Yes |
| `faas.document.name`       | 操作の対象となる文書名/テーブル<br /> 例えば、Cloud StorageやS3ではファイル名、Cosmos DBではテーブル名です。 | No |

<!--
### HTTP
-->

### HTTP

<!--
The function responsibility is to provide an answer to an inbound HTTP request. The `faas` span SHOULD follow the recommendations described in the [HTTP Server semantic conventions](http.md#http-server-semantic-conventions).
-->

この関数の責任は、受信HTTPリクエストに対する応答を提供することです。 `faas` Spanは、[HTTPサーバーセマンティック規約](http.md#HTTPサーバーセマンティック規約)で説明されている推奨事項に従うべきです(SHOULD)。

<!--
### PubSub
-->

### PubSub

<!--
A function is set to be executed when messages are sent to a messaging system.
In this case, multiple messages could be batch and forwarded at once to the same function execution.
Therefore, a different root span of type `faas` MUST be created for each message processed by the function, following the [Messaging systems semantic conventions](messaging.md).
This way, it is possible to correlate each individual message with its execution sender.
-->

この関数はメッセージングシステムにメッセージが送信されたときに実行されます。この場合、複数のメッセージを一括して同じ関数の実行に転送することができます。したがって、この関数が処理する各メッセージに対して、[メッセージングシステムのセマンティック規約](messaging.md)に従って、異なるrootを持つ`faas`型のSpan(??? この理解で正しいか)を作成しなければなりません(MUST)。このようにして、個々のメッセージをその実行送信者と関連付けることが可能になります。

<!--
### Timer
-->

### Timer

<!--
A function is scheduled to be executed regularly. The following additional attributes are recommended.
-->

この関数は定期的に実行されるようにスケジュールされています。以下の追加属性を推奨します。

<!--
| Attribute name  | Notes  and examples  | Required? |
|---|---|--|
| `faas.time` | A string containing the function invocation time in the [ISO 8601] format expressed in [UTC]. E.g. `"2020-01-23T13:47:06Z"`| Yes |
| `faas.cron` | A string containing the schedule period as [Cron Expression]. E.g. `"0/5 * * * ? *"`| No |
-->

| 属性名 | 説明と例                                           | Required? |
|---|---|--|
| `faas.time` | 関数の呼び出し時刻を含む文字列で、[ISO 8601]形式で[UTC]で表される。例: `"2020-01-23T13:47:06Z" `.| Yes |
| `faas.cron` | [Cron 記法]のようにスケジュール期間を含む文字列。例えば `"0/5 * * * ? *"`| No |

<!--
[Cron Expression]: https://docs.oracle.com/cd/E12058_01/doc/doc.1014/e12030/cron_expressions.htm
[ISO 8601]: https://www.iso.org/iso-8601-date-and-time-format.html
[UTC]: https://www.w3.org/TR/NOTE-datetime
-->

[Cron 記法]: https://docs.oracle.com/cd/E12058_01/doc/doc.1014/e12030/cron_expressions.htm
[ISO 8601]: https://www.iso.org/iso-8601-date-and-time-format.html
[UTC]: https://www.w3.org/TR/NOTE-datetime

<!--
### Other
-->

### Other

<!--
Function as a Service offers such flexibility that it is not possible to fully cover with semantic conventions.
When a function does not satisfy any of the aforementioned cases, a span MUST set the attribute `faas.trigger` to `"other"`.
In this case, it is responsibility of the framework or instrumentation library to define the most appropriate attributes.
-->

Function as a Serviceは、セマンティックな規約ではカバーしきれないほどの柔軟性を提供しています。関数が前述のいずれの場合も満たさない場合、Spanは属性 `faas.trigger` を `"other"` に設定しなければなりません(MUST)。この場合、最も適切な属性を定義するのはフレームワークや計装ライブラリの責任です。
