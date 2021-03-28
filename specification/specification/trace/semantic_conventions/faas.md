# Semantic conventions for FaaS spans

**Status**: [Experimental](../../document-status.md)

This document defines how to describe an instance of a function that runs without provisioning
or managing of servers (also known as serverless functions or Function as a Service (FaaS)) with spans.

<!-- Re-generate TOC with `markdown-toc --no-first-h1 -i` -->

<!-- toc -->

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

<!-- tocstop -->

<!--
## General Attributes
-->

## 一般属性

<!--
Span `name` should be set to the function name being executed. Depending on the value of the `faas.trigger` attribute, additional attributes MUST be set. For example, an `http` trigger SHOULD follow the [HTTP Server semantic conventions](http.md#http-server-semantic-conventions). For more information, refer to the [Function Trigger Type](#function-trigger-type) section.
-->

Span `name` には、実行される関数名を設定する必要があります。 `fas.trigger` 属性の値に応じて、追加の属性を設定しなければなりません(MUST)。例えば、`http`のトリガーは、[HTTP Server セマンティク規約](http.md#http-server-semantic-conventions)に従うべきです(SHOULD)。詳細については、[関数トリガタイプ](#function-trigger-type)のセクションを参照してください。

<!--
If Spans following this convention are produced, a Resource of type `faas` MUST exist following the [Resource semantic convention](../../resource/semantic_conventions/faas.md#function-as-a-service).
-->

この規約に従ったSpanが生成された場合、[Resource セマンティク規約](./../resource/semantic_conventions/faas.md#function-as-a-service)に従った`faas`タイプのResourceが存在しなければなりません(MUST)。

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

### Function Name

There are 2 locations where the function's name can be recorded: the span name and the
[`faas.name` Resource attribute](../../resource/semantic_conventions/faas.md#function-as-a-service).

It is guaranteed that if `faas.name` attribute is present it will contain the
function name, since it is defined in the semantic convention strictly for that
purpose. It is also highly likely that Span name will contain the function name
(e.g. for Span displaying purposes), but it is not guaranteed (since it is a
weaker "SHOULD" requirement). Consumers that needs such guarantee can use
`faas.name` attribute as the source.

### Difference between execution and instance

For performance reasons (e.g. [AWS lambda], or [Azure functions]), FaaS providers allocate an execution environment for a single instance of a function that is used to serve multiple requests.
Developers exploit this fact to solve the **cold start** issue, caching expensive resource computations between different function executions.
Furthermore, FaaS providers encourage this behavior, e.g. [Google functions].
This field MAY be set to help correlate function executions that belong to the same execution environment.
The span attribute `faas.execution` differs from the resource attribute `faas.instance` in the following:

- `faas.execution` refers to the current request ID handled by the function;
- `faas.instance` refers to the execution environment ID of the function.

[AWS lambda]: https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html
[Azure functions]: https://docs.microsoft.com/en-us/azure/azure-functions/manage-connections#static-clients
[Google functions]: https://cloud.google.com/functions/docs/concepts/exec#function_scope_versus_global_scope

## Incoming Invocations

This section describes incoming FaaS invocations as they are reported by the FaaS instance itself.

For incoming FaaS spans, the span kind MUST be `Server`.

<!-- semconv faas_span.in -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `faas.coldstart` | boolean | サーバーレス機能が初めて実行された場合(コールドスタート)に真となる真偽値。 |  | No |
<!-- endsemconv -->

## Outgoing Invocations

This section describes outgoing FaaS invocations as they are reported by a client calling a FaaS instance.

For outgoing FaaS spans, the span kind MUST be `Client`.

The values reported by the client for the attributes listed below SHOULD be equal to
the corresponding [FaaS resource attributes][] and [Cloud resource attributes][],
which the invoked FaaS instance reports about itself, if it's instrumented.

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
| `azure` | Amazon Web Services |
| `gcp` | Google Cloud Platform |
<!-- endsemconv -->

[FaaS resource attributes]: ../../resource/semantic_conventions/faas.md
[Cloud resource attributes]: ../../resource/semantic_conventions/cloud.md

## Function Trigger Type

This section describes how to handle the span creation and additional attributes based on the value of the attribute `faas.trigger`.

### Datasource

A datasource function is triggered as a response to some data source operation such as a database or filesystem read/write.
For `faas` spans with trigger `datasource`, it is recommended to set the following attributes.

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

### HTTP

The function responsibility is to provide an answer to an inbound HTTP request. The `faas` span SHOULD follow the recommendations described in the [HTTP Server semantic conventions](http.md#http-server-semantic-conventions).

### PubSub

A function is set to be executed when messages are sent to a messaging system.
In this case, multiple messages could be batch and forwarded at once to the same function execution.
Therefore, a different root span of type `faas` MUST be created for each message processed by the function, following the [Messaging systems semantic conventions](messaging.md).
This way, it is possible to correlate each individual message with its execution sender.

### Timer

A function is scheduled to be executed regularly. The following additional attributes are recommended.

<!-- semconv faas_span.timer -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `faas.time` | string | [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html)形式で、[UTC](https://www.w3.org/TR/NOTE-datetime)で表現された関数の起動時間を含む文字列です | `2020-01-23T13:47:06Z` | Yes |
| `faas.cron` | string | スケジュール期間を[Cron Expression](https://docs.oracle.com/cd/E12058_01/doc/doc.1014/e12030/cron_expressions.htm)として含む文字列。 | `0/5 * * * ? *` | No |
<!-- endsemconv -->

### Other

Function as a Service offers such flexibility that it is not possible to fully cover with semantic conventions.
When a function does not satisfy any of the aforementioned cases, a span MUST set the attribute `faas.trigger` to `"other"`.
In this case, it is responsibility of the framework or instrumentation library to define the most appropriate attributes.

## Example

This example shows the FaaS attributes for a (non-FaaS) process hosted on Google Cloud Platform (Span A with kind `Client`), which invokes a Lambda function called "my-lambda-function" in Amazon Web Services (Span B with kind `Server`).

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
