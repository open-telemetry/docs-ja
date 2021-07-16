<!--
# Semantic conventions for FaaS spans
-->

# FaaSSpanのセマンティック規約

<!--
**Status**: [Experimental](../../document-status.md)
-->

**Status**: [Experimental](../../document-status.md)

<!--
This document defines how to describe an instance of a function that runs without provisioning
or managing of servers (also known as serverless functions or Function as a Service (FaaS)) with spans.
-->

本ドキュメントでは、サーバーのプロビジョニングや管理を行わずに動作する機能(サーバーレス機能やFaaS(Function as a Service)とも呼ばれる)のインスタンスをSpanで記述する方法を定義しています。

<!-- Re-generate TOC with `markdown-toc --no-first-h1 -i` -->

<!-- toc -->

<!--
- [General Attributes](#general-attributes)
  * [Difference between execution and instance](#difference-between-execution-and-instance)
- [Incoming Invocations](#incoming-invocations)
- [Outgoing Invocations](#outgoing-invocations)
- [Function Trigger Type](#function-trigger-type)
  * [Datasource](#datasource)
  * [HTTP](#http)
  * [PubSub](#pubsub)
  * [Timer](#timer)
  * [Other](#other)
- [Example](#example)
-->

- [一般的な属性](#一般的な属性)
  * [実行とインスタンスの違い](#実行とインスタンスの違い)
- [Incoming Invocations](#incoming-invocations)
- [Outgoing Invocations](#outgoing-invocations)
- [関数トリガーの種類](#関数トリガーの種類)
  * [Datasource](#datasource)
  * [HTTP](#http)
  * [PubSub](#pubsub)
  * [タイマー](#タイマー)
  * [その他](#その他)
- [例](#例)

<!-- tocstop -->

<!--
## General Attributes
-->

## 一般的な属性

<!--
Span `name` should be set to the function name being executed. Depending on the value of the `faas.trigger` attribute, additional attributes MUST be set. For example, an `http` trigger SHOULD follow the [HTTP Server semantic conventions](http.md#http-server-semantic-conventions). For more information, refer to the [Function Trigger Type](#function-trigger-type) section.
-->

Span `name` には、実行される関数名を設定する必要があります。`faas.trigger` 属性の値に応じて、追加の属性を設定しなければなりません。例えば、`http`のトリガーは、[HTTPサーバのセマンティック規約](http.md#HTTPサーバのセマンティック規約)に従うべきです(SHOULD)。詳細については、[関数トリガーの種類](#関数トリガーの種類)のセクションを参照してください。

<!--
If Spans following this convention are produced, a Resource of type `faas` MUST exist following the [Resource semantic convention](../../resource/semantic_conventions/faas.md#function-as-a-service).
-->

この規約に従ったSpanが生成された場合、[Resource semantic convention](./../resource/semantic_conventions/faas.md#function-as-a-service)に従った`faas`タイプのResourceが存在しなければなりません(MUST)。

<!-- semconv faas_span -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `faas.trigger` | string | 関数が実行されるトリガーのタイプ。 | `datasource` | Conditional [1] |
| `faas.execution` | string | 現在実行されている関数の実行ID。 | `af9d5aa4-a685-4c5f-a22b-444f80b3cc28` | No |

**[1]:** FaaSインスタンスでは、着信時にfaas.trigger属性を設定しなければなりません(MUST)。FaaSインスタンスを起動しているクライアントは、クライアントに知られている場合、発信される起動に`faas.trigger`を設定しなければなりません(MUST)。例えば、トランスポートレイヤーがFaaSクライアントフレームワークで抽象化されていて、その設定にアクセスできない場合は、この限りではありません。

`faas.trigger` MUST be one of the following:

| Value  | Description |
|---|---|
| `datasource` | データベースやファイルシステムの読み取り/書き込みなど、データソースの操作に対する応答の関数。 |
| `http` | 受信したHTTPリクエストへの応答の関数。 |
| `pubsub` | メッセージングシステムにメッセージが送信されたときに実行される設定の関数。 |
| `timer` | 定期的に実行されるように予定されている関数。 |
| `other` | 他に該当するものがない関数 |
<!-- endsemconv -->
<!--
### Function Name
-->

### 関数名

<!--
There are 2 locations where the function's name can be recorded: the span name and the
[`faas.name` Resource attribute](../../resource/semantic_conventions/faas.md#function-as-a-service).
-->

関数の名前を記録する場所は、Span名と[`faas.name` リソース属性](./../resource/semantic_conventions/faas.md#function-as-a-service)の2つです。

<!--
It is guaranteed that if `faas.name` attribute is present it will contain the
function name, since it is defined in the semantic convention strictly for that
purpose. It is also highly likely that Span name will contain the function name
(e.g. for Span displaying purposes), but it is not guaranteed (since it is a
weaker "SHOULD" requirement). Consumers that needs such guarantee can use
`faas.name` attribute as the source.
-->

これは、`faas.name`属性が存在する場合、関数名が含まれていることが保証されています。また、Span name に関数名が含まれている可能性も高いですが(例:Span の表示のため)、これは保証されていません(より弱い「SHOULD」要件であるため)。このような保証を必要とする消費者は、`faas.name`属性をソースとして使用することができます。

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

パフォーマンス上の理由から(例:[AWS lambda]や[Azure functions])、FaaSプロバイダーは、複数のリクエストに対応する関数の単一のインスタンスに実行環境を割り当てます。開発者はこの事実を利用して、**コールドスタート**の問題を解決し、高価なリソースの計算を異なる関数の実行間でキャッシュします。さらに、FaaSプロバイダーはこの動作を推奨しています(例:[Google functions])。このフィールドは、同じ実行環境に属する関数の実行を相関させるために設定してもよいものとします。span 属性の `faas.execution` は、resource 属性の `faas.instance` と以下の点で異なります。

<!--
- `faas.execution` refers to the current request ID handled by the function;
- `faas.instance` refers to the execution environment ID of the function.
-->

- `faas.execution` は、その関数が扱う現在のリクエストIDを指します。
- `faas.instance` は、その関数の実行環境の ID を指します。

<!--
[AWS lambda]: https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html
[Azure functions]: https://docs.microsoft.com/en-us/azure/azure-functions/manage-connections#static-clients
[Google functions]: https://cloud.google.com/functions/docs/concepts/exec#function_scope_versus_global_scope
-->

[AWS lambda]: https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html
[Azure functions]: https://docs.microsoft.com/en-us/azure/azure-functions/manage-connections#static-clients
[Google functions]: https://cloud.google.com/functions/docs/concepts/exec#function_scope_versus_global_scope

<!--
## Incoming Invocations
-->

## Incoming呼び出し

<!--
This section describes incoming FaaS invocations as they are reported by the FaaS instance itself.
-->

このセクションでは、FaaSインスタンス自身から報告されるFaaSの呼び出しについて説明します。

<!--
For incoming FaaS spans, the span kind MUST be `Server`.-->

受信するFaaSSpanの場合、Spanの種類は`Server`でなければなりません(MUST)。


<!-- semconv faas_span.in -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `faas.coldstart` | boolean | サーバーレス機能が初めて実行された場合(コールドスタート)に真となる真偽値。 |  | No |
<!-- endsemconv -->
<!--
## Outgoing Invocations
-->

## Outgoing呼び出し

<!--
This section describes outgoing FaaS invocations as they are reported by a client calling a FaaS instance.
-->

このセクションでは、FaaSインスタンスを呼び出すクライアントから報告される、発信されるFaaSの呼び出しについて説明します。

<!--
For outgoing FaaS spans, the span kind MUST be `Client`.
-->

発信するFaaSSpanの場合、Spanの種類は「Client」でなければなりません(MUST)。

<!--
The values reported by the client for the attributes listed below SHOULD be equal to
the corresponding [FaaS resource attributes][] and [Cloud resource attributes][],
which the invoked FaaS instance reports about itself, if it's instrumented.-->

以下の属性についてクライアントが報告する値は、計装されている場合、呼び出されたFaaSインスタンスが自身について報告する、対応する[FaaSリソース属性][]および[クラウドリソース属性][]と同じであるべきです。

<!-- semconv faas_span.out -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `faas.invoked_name` | string | 呼び出された関数の名前です。 [1] | `my-function` | Yes |
| `faas.invoked_provider` | string | 呼び出された関数のクラウドプロバイダー。 [2] | `aws` | Yes |
| `faas.invoked_region` | string | 呼び出された関数が動いているリージョン [3] | `eu-central-1` | Conditional [4] |

**[1]:** 呼び出された関数のリソース属性 `faas.name` と同じ値にすべきです(SHOULD)。

**[2]:** 呼び出された関数のリソース属性である `cloud.provider` と同じ値にすべきです(SHOULD)。

**[3]:** 呼び出された関数のリソース属性である `cloud.region` と同じ値にすべきです(SHOULD)。

**[4]:** AWSやGCPなどの一部のクラウドプロバイダーでは、関数を一意に識別するためにその関数がホストされているリージョンが不可欠な情報であり、またエンドポイントの一部でもあります。リージョンは呼び出されるエンドポイントの一部であるため、クライアントは常にリージョンを知っています。このような場合には、`faas.invoked_region` を適宜設定しなければなりません(MUST)。リージョンがクライアントに知られていない場合や、呼び出された関数を識別するのに必要ない場合は、`faas.invoked_region`の設定は任意です。

`faas.invoked_provider` MUST be one of the following or, if none of the listed values apply, a custom value:

| Value  | Description |
|---|---|
| `aws` | Amazon Web Services |
| `azure` | Microsoft Azure |
| `gcp` | Google Cloud Platform |
<!-- endsemconv -->

[FaaSリソース属性]: ../../resource/semantic_conventions/faas.md
[クラウドリソース属性]: ../../resource/semantic_conventions/cloud.md

<!--
## Function Trigger Type
-->

## 関数トリガーの種類

<!--
This section describes how to handle the span creation and additional attributes based on the value of the attribute `faas.trigger`.
-->

ここでは、`faas.trigger`という属性の値に基づいて、Spanの作成や追加属性を処理する方法を説明します。

<!--
### Datasource
-->

### データソース

<!--
A datasource function is triggered as a response to some data source operation such as a database or filesystem read/write.
For `faas` spans with trigger `datasource`, it is recommended to set the following attributes.-->

データソース関数は、データベースやファイルシステムの読み書きなど、何らかのデータソース操作に対する応答としてトリガされます。トリガーが `datasource` である `faas` Spanについては、以下の属性を設定することが推奨されます。

<!-- semconv faas_span.datasource -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `faas.document.collection` | string | トリガー操作が行われたソースの名前。例えば、Cloud StorageやS3であればバケット名、Cosmos DBであればデータベース名に対応します。 | `myBucketName`; `myDbName` | Yes |
| `faas.document.operation` | string | データに対して行われた操作の種類を記述します。 | `insert` | Yes |
| `faas.document.time` | string | データにアクセスした時刻を[ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html)形式で、[UTC](https://www.w3.org/TR/NOTE-datetime)で表した文字列。 | `2020-01-23T13:47:06Z` | Yes |
| `faas.document.name` | string | 操作の対象となるドキュメント名/テーブル名。例えば、クラウドストレージやS3ではファイル名、Cosmos DBではテーブル名となります。 | `myFile.txt`; `myTableName` | No |

`faas.document.operation` MUST be one of the following or, if none of the listed values apply, a custom value:

| Value  | Description |
|---|---|
| `insert` | 新しいオブジェクトが作成されたとき |
| `edit` | オブジェクトが変更されたとき |
| `delete` | オブジェクトが削除されたとき |
<!-- endsemconv -->
<!--
### HTTP
-->

### HTTP

<!--
The function responsibility is to provide an answer to an inbound HTTP request. The `faas` span SHOULD follow the recommendations described in the [HTTP Server semantic conventions](http.md#http-server-semantic-conventions).
-->

この関数の責任は、インバウンドの HTTP リクエストに対する返答を提供することです。この`faas`Spanは、[HTTPサーバのセマンティック規約](http.md#HTTPサーバのセマンティック規約)に記載されている推奨事項に従うべきです。

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

メッセージングシステムにメッセージが送られてきたときに、実行されるように設定されている関数です。この場合、複数のメッセージがバッチ処理され、同じ関数の実行に一度に転送される可能性があります。そのため、[メッセージングシステムのセマンティック規約](messaging.md)に従って、関数が処理する各メッセージに対して、`faas`型の異なるルートSpanを作成しなければなりません(MUST)。このようにして、個々のメッセージとその実行送信者を関連付けることができます。

<!--
### Timer
-->

### タイマー

<!--
A function is scheduled to be executed regularly. The following additional attributes are recommended.
-->

関数は、定期的に実行されるようにスケジュールされています。以下の追加属性を推奨します。

<!-- semconv faas_span.timer -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `faas.time` | string | [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html)形式で、[UTC](https://www.w3.org/TR/NOTE-datetime)で表現された関数の起動時間を含む文字列です | `2020-01-23T13:47:06Z` | Yes |
| `faas.cron` | string | スケジュール期間を[Cron Expression](https://docs.oracle.com/cd/E12058_01/doc/doc.1014/e12030/cron_expressions.htm)として含む文字列。 | `0/5 * * * ? *` | No |
<!-- endsemconv -->
<!--
### Other
-->

### その他

<!--
Function as a Service offers such flexibility that it is not possible to fully cover with semantic conventions.
When a function does not satisfy any of the aforementioned cases, a span MUST set the attribute `faas.trigger` to `"other"`.
In this case, it is responsibility of the framework or instrumentation library to define the most appropriate attributes.
-->

Function as a Serviceは非常に柔軟性が高いため、セマンティック規約で完全にカバーすることはできません。関数が前述のケースのいずれも満たさない場合、Spanは属性 `faas.trigger` を `"other"` に設定しなければなりません(MUST)。この場合、最も適切な属性を定義するのは、フレームワークや計装ライブラリの責任となります。

<!--
## Example
-->

## 例

<!--
This example shows the FaaS attributes for a (non-FaaS) process hosted on Google Cloud Platform (Span A with kind `Client`), which invokes a Lambda function called "my-lambda-function" in Amazon Web Services (Span B with kind `Server`).-->

この例では、Google Cloud Platformにホストされた(FaaSではない)プロセス(SpanA、種類は`Client`)が、Amazon Web Servicesの "my-lambda-function "というLambda関数を呼び出す場合(SpanB、種類は`Server`)のFaaS属性を示しています。


<!--
| Attribute Kind | Attribute               | Span A (Client, GCP)   | Span B (Server, AWS Lambda) |
| -------------- | ----------------------- | ---------------------- | -- |
| Resource       | `cloud.provider`        | `"gcp"`                | `"aws"` |
| Resource       | `cloud.region`          | `"europe-west3"`       | `"eu-central-1"` |
| Span           | `faas.invoked_name`     | `"my-lambda-function"` | n/a |
| Span           | `faas.invoked_provider` | `"aws"`                | n/a |
| Span           | `faas.invoked_region`   | `"eu-central-1"`       | n/a |
| Span           | `faas.trigger`          | n/a                    | `"http"` |
| Span           | `faas.execution`        | n/a                    | `"af9d5aa4-a685-4c5f-a22b-444f80b3cc28"` |
| Span           | `faas.coldstart`        | n/a                    | `true` |
| Resource       | `faas.name`             | n/a                    | `"my-lambda-function"` |
| Resource       | `faas.id`               | n/a                    | `"arn:aws:lambda:us-west-2:123456789012:function:my-lambda-function"` |
| Resource       | `faas.version`          | n/a                    | `"semver:2.0.0"` |
| Resource       | `faas.instance`         | n/a                    | `"my-lambda-function:instance-0001"` |
-->

| 属性の種類      | 属性                    | Span A (Client, GCP)   | Span B (Server, AWS Lambda) |
| -------------- | ----------------------- | ---------------------- | -- |
| Resource       | `cloud.provider`        | `"gcp"`                | `"aws"` |
| Resource       | `cloud.region`          | `"europe-west3"`       | `"eu-central-1"` |
| Span           | `faas.invoked_name`     | `"my-lambda-function"` | n/a |
| Span           | `faas.invoked_provider` | `"aws"`                | n/a |
| Span           | `faas.invoked_region`   | `"eu-central-1"`       | n/a |
| Span           | `faas.trigger`          | n/a                    | `"http"` |
| Span           | `faas.execution`        | n/a                    | `"af9d5aa4-a685-4c5f-a22b-444f80b3cc28"` |
| Span           | `faas.coldstart`        | n/a                    | `true` |
| Resource       | `faas.name`             | n/a                    | `"my-lambda-function"` |
| Resource       | `faas.id`               | n/a                    | `"arn:aws:lambda:us-west-2:123456789012:function:my-lambda-function"` |
| Resource       | `faas.version`          | n/a                    | `"semver:2.0.0"` |
| Resource       | `faas.instance`         | n/a                    | `"my-lambda-function:instance-0001"` |
