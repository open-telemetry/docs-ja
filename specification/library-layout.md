# OpenTelemetry プロジェクトのパッケージレイアウト

<!--
This documentation serves to document the "look and feel" of a basic layout for OpenTelemetry
projects. This package layout is intentionally generic and it doesn't try to impose a language
specific package structure.
-->

この文書ではOpenTelemetryプロジェクトの基本的な"look and feel(見た目)"のレイアウトについて説明します。このパッケージレイアウトは言語に依存しないよう、意図的に一般的な構造にしてあります。


## API パッケージ

<!--
Here is a proposed generic package structure for OpenTelemetry API package.
-->

OpenTelemetryのAPIパッケージ関する一般的な構造を示します。


<!-- A typical top-level directory layout: -->

典型的なトップレベルのディレクトリ構造は以下のとおりです:

```
api
   ├── context
   │   └── propagation
   ├── metrics
   ├── trace
   │   └── propagation
   ├── correlationcontext
   │   └── propagation
   ├── internal
   └── logs
```

<!--
> Use of lowercase, CamelCase or Snake Case (stylized as snake_case) names depends on the language.
-->

> 小文字で、CamelCaseかsnake_caseを使います。どちらを使うかは言語によって異なります。

### `/context`

<!--
This directory describes the API that provides in-process context propagation.
-->

このディレクトリにはプロセス内のコンテキスト伝搬に関するAPIを提供します。

### [/metrics](./metrics/api.md)

<!--
This directory describes the Metrics API that can be used to record application metrics.
-->

このディレクトリにはアプリケーションのメトリックを保存するためのメトリックAPIを提供します。

### [/correlationcontext](correlationcontext/api.md)

<!--
This directory describes the CorrelationContext API that can be used to manage context propagation
and metrics-related labeling.
->

このディレクトリにはコンテキストの伝搬とメトリックに関連するラベリングに関するCorrelationContext APIを含みます。


### [/trace](trace/api.md)

<!--
This API consist of a few main classes:
-->

このAPIは2つのメインクラスを提供します。

<!--
- `Tracer` is used for all operations. See [Tracer](trace/api.md#tracer) section.
- `Span` is a mutable object storing information about the current operation
   execution. See [Span](trace/api.md#span) section.
-->

- `Tracer` はすべての操作で使われます。 [Tracer](trace/api.md#tracer) セクションを参照してください。
- `Span` は現在の操作・実行についての情報を保持する改変可能なオブジェクトです。 [Span](trace/api.md#span) セクションを参照してください。

### `/internal` (_省略可能_)

<!--
Private application and library code.
-->

非公開のアプリケーションとライブラリのコードです。

### `/logs` (_将来実装_)

<!--
> TODO: logs operations
-->

> TODO: ログ操作

## SDK パッケージ

<!--
Here is a proposed generic package structure for OpenTelemetry SDK package.
-->

OpenTelemtry SDKパッケージの一般的なパッケージ構造について説明します。

<!--
A typical top-level directory layout:
-->

典型的なトップレベルのディレクトリ構造は以下のとおりです:

```
sdk
   ├── context
   ├── metrics
   ├── resource
   ├── trace
   ├── correlationcontext
   ├── internal
   └── logs
```

<!--
> Use of lowercase, CamelCase or Snake Case (stylized as snake_case) names depends on the language.
-->

> 小文字で、CamelCaseかsnake_caseを使います。どちらを使うかは言語によって異なります。

### `/sdk/context`
<!--
This directory describes the SDK implementation for api/context.
-->

このディレクトリは api/context のSDK実装を提供します。

### `/sdk/metrics`

<!--
This directory describes the SDK implementation for api/metrics.
-->

このディレクトリは api/metrics のSDK実装を提供します。


### [/sdk/resource](resource/sdk.md)

<!--
The resource directory primarily defines a type [Resource](overview.md#resources) that captures
information about the entity for which stats or traces are recorded. For example, metrics exposed
by a Kubernetes container can be linked to a resource that specifies the cluster, namespace, pod,
and container name.
-->

このリソースディレクトリはどのStatsあるいはTraceが記録されたかに関するエンティティに関する情報を記録する [Resource](overview.md#resources) 型を定義します。
例えばKubernetesクラスターで提供されているmetricはそのクラスター、namespace、pod、あるいはコンテナ名などで使われているリソースと関連があります。


### `/sdk/correlationcontext`

### [/sdk/trace](trace/sdk.md)

<!--
This directory describes the SDK implementation for api/trace.
-->

このディレクトリは api/trace のSDK実装を提供します。

### `/sdk/internal` (_省略可_)

<!--
Private application and library code.
-->

非公開のアプリケーションとライブラリのコードです。

### `/sdk/logs` (_将来実装_)

<!--
> TODO: logs operations
-->

> TODO: ログ操作
