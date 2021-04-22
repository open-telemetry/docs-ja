<!--
# OpenTelemetry Project Package Layout
-->

# OpenTelemetryプロジェクトのパッケージレイアウト

<!--
This documentation serves to document the "look and feel" of a basic layout for OpenTelemetry
projects. This package layout is intentionally generic and it doesn't try to impose a language
specific package structure.
-->

このドキュメントは、OpenTelemetryプロジェクトの基本的なレイアウトの「ルック＆フィール」を文書化したものです。このパッケージレイアウトは意図的に一般的なものになっており、言語固有のパッケージ構造を押し付けようとはしていません。

<!--
## API Package
-->

## API パッケージ

<!--
Here is a proposed generic package structure for OpenTelemetry API package.
-->

ここでは、OpenTelemetry APIパッケージの一般的なパッケージ構造を提案します。

<!--
A typical top-level directory layout:
-->

典型的なトップレベルディレクトリのレイアウト:

<!--
```
api
   ├── context
   │   └── propagation
   ├── metrics
   ├── trace
   │   └── propagation
   ├── baggage
   │   └── propagation
   ├── internal
   └── logs
```
-->

```
api
   ├── context
   │   └── propagation
   ├── metrics
   ├── trace
   │   └── propagation
   ├── baggage
   │   └── propagation
   ├── internal
   └── logs
```

<!--
> Use of lowercase, CamelCase or Snake Case (stylized as snake_case) names depends on the language.
-->

> 小文字、CamelCase、Snake Case(snake_caseとしてスタイリング)の名前の使用は、言語によって異なります。

<!--
### `/context`
-->

### `/context`

<!--
This directory describes the API that provides in-process context propagation.
-->

このディレクトリでは、プロセス中のコンテキスト伝播を提供するAPIについて説明します。

<!--
### [/metrics](./metrics/api.md)
-->

### [/metrics](./metrics/api.md)

<!--
This directory describes the Metrics API that can be used to record application metrics.
-->

このディレクトリでは、アプリケーション・メトリクスを記録するために使用できるMetrics APIについて説明しています。

<!--
### [/baggage](baggage/api.md)
-->

### [/baggage](baggage/api.md)

<!--
This directory describes the Baggage API that can be used to manage context propagation
and metrics-related labeling.
-->

このディレクトリでは、コンテキストプロパゲーションやMetrics関連のラベリングを管理するために使用できるBaggage APIについて説明しています。

<!--
### [/trace](trace/api.md)
-->

### [/trace](trace/api.md)

<!--
This API consist of a few main classes:
-->

このAPIは、いくつかの主要なクラスで構成されています。

<!--
- `Tracer` is used for all operations. See [Tracer](trace/api.md#tracer) section.
- `Span` is a mutable object storing information about the current operation
   execution. See [Span](trace/api.md#span) section.
-->

- すべての操作には `Tracer` が使われます。[Tracer](trace/api.md#tracer)のセクションを参照してください。
- `Span` は、現在のオペレーション実行に関する情報を格納する可変(mutable)オブジェクトです。[Span](trace/api.md#span)のセクションを参照してください。

<!--
### `/internal` (_Optional_)
-->

### `/internal` (_任意_)

<!--
Private application and library code.
-->

プライベートなアプリケーションやライブラリのコード。

<!--
### `/logs` (_In the future_)
-->

### `/logs` (_将来的に_)

<!--
> TODO: logs operations
-->

> TODO: ログ操作

<!--
## SDK Package
-->

## SDK パッケージ

<!--
Here is a proposed generic package structure for OpenTelemetry SDK package.
-->

ここでは、OpenTelemetry SDKパッケージの一般的なパッケージ構造を提案します。

<!--
A typical top-level directory layout:
-->

典型的なトップレベルディレクトリのレイアウト:

<!--
```
sdk
   ├── context
   ├── metrics
   ├── resource
   ├── trace
   ├── baggage
   ├── internal
   └── logs
```
-->

```
sdk
   ├── context
   ├── metrics
   ├── resource
   ├── trace
   ├── baggage
   ├── internal
   └── logs
```

<!--
> Use of lowercase, CamelCase or Snake Case (stylized as snake_case) names depends on the language.
-->

> 小文字、CamelCase、Snake Case(snake_caseとしてスタイリング)の名前の使用は、言語によって異なります。

<!--
### `/sdk/context`
-->

### `/sdk/context`

<!--
This directory describes the SDK implementation for api/context.
-->

api/contextのSDK実装について説明しているディレクトリです。

<!--
### `/sdk/metrics`
-->

### `/sdk/metrics`

<!--
This directory describes the SDK implementation for api/metrics.
-->

api/metricsのSDK実装について説明しているディレクトリです。

<!--
### [/sdk/resource](resource/sdk.md)
-->

### [/sdk/resource](resource/sdk.md)

<!--
The resource directory primarily defines a type [Resource](overview.md#resources) that captures
information about the entity for which stats or traces are recorded. For example, metrics exposed
by a Kubernetes container can be linked to a resource that specifies the cluster, namespace, pod,
and container name.
-->

リソースディレクトリでは主に、統計やTraceが記録されるエンティティに関する情報をキャプチャする[リソース](overview.md#resources)というタイプが定義されています。例えば、Kubernetesコンテナが公開するメトリクスは、クラスタ、名前空間、ポッド、コンテナ名を指定するリソースにリンクできます。

<!--
### `/sdk/baggage`
-->

### `/sdk/baggage`

<!--
### [/sdk/trace](trace/sdk.md)
-->

### [/sdk/trace](trace/sdk.md)

<!--
This directory describes the SDK implementation for api/trace.
-->

api/traceのSDK実装について説明しているディレクトリです。

<!--
### `/sdk/internal` (_Optional_)
-->

### `/sdk/internal` (_任意_)

<!--
Private application and library code.
-->

プライベートなアプリケーションやライブラリのコード。

<!--
### `/sdk/logs` (_In the future_)
-->

### `/sdk/logs` (_将来的に_)

<!--
> TODO: logs operations
-->

> TODO: ログ操作

