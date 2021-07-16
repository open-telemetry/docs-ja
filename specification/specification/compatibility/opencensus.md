<!--
# OpenCensus Compatibility
-->

# OpenCensus との互換性

**Status**: [Experimental](../document-status.md), Unless otherwise specified.

<!--
## Abstract
-->

## 概要

<!--
The OpenTelemetry project aims to provide backwards compatibility with the
[OpenCensus](https://opencensus.io) project in order to ease migration of
instrumented codebases.
-->

OpenTelemetryプロジェクトは、[OpenCensus](https://opencensus.io)プロジェクトとの下位互換性を提供することで、計測対象のコードベースの移行を容易にすることを目的としています。

<!--
This functionality will be provided as a bridge layer implementing the
[OpenCensus API](https://github.com/census-instrumentation/opencensus-specs)
using the OpenTelemetry API. This layer MUST NOT rely on implementation specific
details of the OpenTelemetry SDK.
-->

この機能は、[OpenCensus API](https://github.com/census-instrumentation/opencensus-specs)をOpenTelemetry APIで実装するブリッジ・レイヤーとして提供されます。このレイヤーは、OpenTelemetry SDKの実装特有の詳細に依存してはなりません(MUST NOT)。

<!--
More specifically, the intention is to allow OpenCensus instrumentation to be
recorded using OpenTelemetry. This Shim Layer MUST NOT publicly expose any
upstream OpenTelemetry API.
-->

より具体的には、OpenCensusの計装をOpenTelemetryで記録できるようにすることを意図しています。この互換性レイヤーは、どんなアップストリームのOpenTelemetry APIも公開してはなりません(MUST NOT)。

<!--
The OpenCensus Shim and the OpenTelemetry API/SDK are expected to be consumed
simultaneously in a running service, in order to ease migration from the former
to the latter.  It is expected that application owners will begin the migration
process towards OpenTelemetry via the shim and adding new telemetry information
via OpenTelemetry.  Slowly, libraries and integrations will also migrate
towards opentelemetry until the shim is no longer necessary.
-->

OpenCensus Shim(互換性レイヤー) と OpenTelemetry API/SDK は、前者から後者への移行を容易にするために、実行中のサービスで同時に使用されることが期待されています。 アプリケーション・オーナーは、Shimを介してOpenTelemetryへの移行プロセスを開始し、OpenTelemetryを介して新しいテレメトリ情報を追加することが予想されます。 徐々に、ライブラリやインテグレーションも、Shimが不要になるまで、OpenTelemetryに移行していくでしょう。

<!--
For example, an application may have traces today of the following variety:
-->

例えば、あるアプリケーションには、今日、次のような種類のTraceがあるかもしれません。

<!--
```
|-- Application - Configured OpenCensus --------------------------------- |
    |--  gRPC -> Using OpenCensus to generate Trace A  --------- |
      |--  Application -> Using OpenCensus to generate a sub Trace B-- |
```
-->

```
|-- Application - OpenCensusの設定 -------------------------------------- |
    |--  gRPC -> Trace Aを生成するためにOpenCensusを使う  --------- |
      |--  Application -> Sub Trace Bを生成するためにOpenCensusを使う-- |
```

<!--
In this case, the application should be able to update its outer layer to
OpenTelemetry, without having to wait for all downstream dependencies to
have updated to OpenTelemetry (or deal with incompatibilities therein). The
Application also doesn't need to rewrite any of its own instrumentation.
-->

この場合、アプリケーションは、その外側のレイヤーを 下流の依存関係がすべてOpenTelemetryに更新されるのを待つことなく、OpenTelemetryに更新できなければなりません。下流の依存関係がOpenTelemetryにアップデートされるのを待つ必要はありません。アプリケーションはまた、自身の計装を書き換える必要もありません。

<!--
```
|-- Application - Configured Otel w/ OpenCensus Shim ------------------- |
    |--  gRPC -> Using OpenCensus to generate Trace A  --------- |
      |--  Application -> Using OpenCensus to generate a sub Trace B-- |
```
-->

```
|-- Application - Otel設定 と OpenCensus Shim ------------------- |
    |--  gRPC -> Trace Aを生成するためにOpenCensusを使う  --------- |
      |--  Application -> Sub Trace Bを生成するためにOpenCensusを使う-- |
```

<!--
Next, the application can update its own instrumentation in a piecemeal fashion:
-->

次に、アプリケーションは自身の計装を断片的に更新することができます。

<!--
```
|-- Application - Configured Otel w/ OpenCensus Shim ---------------------- |
    |--  gRPC -> Using OpenCensus to generate Trace A  --------- |
      |--  Application -> Using OpenTelemetry to generate a sub Trace B-- |
```
-->

```
|-- Application - Otel設定 と OpenCensus Shim ------------------- |
    |--  gRPC -> Trace Aを生成するためにOpenCensusを使う  --------- |
      |--  Application -> Sub Trace Bを生成するためにOpenTelemetryを使う-- |
```

<!--
> This layer of Otel -> OpenCensus -> Otel tracing can be thought of as the
> "OpenTelemetry sandwich" problem, and is the key motivating factor for
> this specification.
-->

> このOtel -> OpenCensus -> Otelのトレースの層は、「OpenTelemetryサンドイッチ」問題と考えることができ、この仕様の重要な動機となっています。

<!--
Finally, the Application would update all usages of OpenCensus to OpenTelemetry.
-->

最後に、アプリケーションは、OpenCensus を使用しているすべてのアプリケーションを OpenTelemetry に更新します。

<!--
```
|-- Application - Configured Otel standalone ----------------------------- |
    |--  gRPC -> Using Otel to generate Trace A  --------- |
      |--  Application -> Using OpenTelemetry to generate a sub Trace B-- |
```
-->

```
|-- Application - Otel のみの設定 ----------------------------- |
    |--  gRPC -> Trace Aを生成するためにOpenTelemetryを使う  --------- |
      |--  Application -> Sub Trace Bを生成するためにOpenTelemetryを使う-- |
```

<!--
OpenCensus supports two primary types of telemetry: Traces and Stats (Metrics).
Compatibility for these is defined separately.
-->

OpenCensusは、主に2種類のテレメトリをサポートしています。Traces と Stats (Metrics) です。これらの互換性は別途定義されています。

<!--
> The overridding philosophy for compatibility is that OpenCensus instrumented
> libraries and applications need make *no change* to their API usage in order
> to use OpenTelemetry. All changes should be solely configuration / setup.
-->

> 互換性のための最優先の哲学は、OpenCensusの計装ライブラリやアプリケーションがOpenTelemetryを使用するために、そのAPIの使用方法を*変更する必要がない*ことです。すべての変更は、設定と構築のみであるべきです。

<!--
## Goals
-->

## ゴール

<!--
OpenTelemetry<->OpenCensus compatibility has the following goals:
-->

OpenTelemetry<->OpenCensusの互換性は次のような目標があります:

<!--
1. OpenCensus has no hard dependency on OpenTelemetry
2. Minimal changes to OpenCensus for implementation
3. Easy for users to use, ideally no change to their code
-->

1. OpenCensusは、OpenTelemetryへのハードな依存がない
2. OpenCensusへの実装は最小限の変更で済む
3. ユーザーにとって使いやすく、理想的にはコードを変更しないで済む

<!--
Additionally, for tracing there are the following goals:
-->

さらに、Traceには以下のような目標があります。

<!--
1. Maintain parent-child span relationship between applications and libraries
2. Maintain span link relationships between applications and libraries
-->

1. アプリケーションとライブラリの親子関係の維持
2. アプリケーションとライブラリの間のSpan-Link関係の維持

<!--
## Trace
-->

## Trace

**Status**: [Experimental, Feature Freeze](../document-status.md)

<!--
OpenTelemetry will provide an OpenCensus-Trace-Shim component that can be
added as a dependency to ensure compatibility with OpenCensus.
-->

OpenTelemetryは、OpenCensusとの互換性を確保するために、依存関係として追加することができるOpenCensus-Trace-Shimコンポーネントを提供します。

<!--
This component MUST be an optional dependency.
-->

このコンポーネントは、任意の依存関係でなければなりません(MUST)。

<!--
### Creating Spans in OpenCensus
-->

### OpenCensusでのSpanの作成

<!--
When the shim is in place, all OpenCensus Spans MUST be sent through an
OpenTelemetry `Tracer` as specified for the OpenTelemetry API.
-->

Shimが設置されている場合、すべてのOpenCensus Spanは、OpenTelemetry APIで指定されているように、OpenTelemetry `Tracer`を介して送信されなければなりません(MUST)。

<!--
This mechanism SHOULD be seamless to the user in languages that allow discovery
and auto injection of dependencies.
-->

このメカニズムは、依存関係の発見と自動注入が可能な言語では、ユーザーにとってシームレスであるべきです(SHOULD)。

<!--
### Methods on Spans
-->

### Spanのメソッド

<!--
All specified methods in OpenCensus will delegate to the underlying `Span` of
OpenTelemetry.
-->

OpenCensus で指定されたすべてのメソッドは、OpenTelemetry の基礎である `Span` にデリゲートされます。

<!--
#### Known Incompatibilities
-->

#### 既知の非互換性

<!--
Below are listed known incompatibilities between OpenTelemetry and OpenCensus
specifications.   Applications leveraging unspecified behavior from OpenCensus
that *is* specified incompatibly within OpenTelemetry are not eligble for
using the OpenCensus <-> OpenTelemetry bridge.
-->

以下に、OpenTelemetry と OpenCensus の仕様間の既知の非互換性を示します。OpenTelemetry内で非互換な仕様となっている、OpenCensusの仕様外の動作を利用しているアプリケーションは、OpenCensus <-> OpenTelemetryブリッジを使用する資格がありません。

<!--
1. In OpenCensus, there is [no specification](https://github.com/census-instrumentation/opencensus-specs/blob/master/trace/Span.md#span-creation)
   when parent spans can be specified on a child span.   OpenTelemetry specifies
   that [parent spans must be specified during span creation](../trace/api.md#span-creation).
   This leads to some issues with OpenCensus APIs that allowed flexible
   specification of parent spans post-initialization.
2. Links added to spans after the spans are created. This is [not supported in
   OpenTelemetry](../trace/api.md#specifying-links), therefore OpenCensus spans
   that have links added to them after creation will be mapped to OpenTelemetry
   spans without the links.
3. OpenTelemetry specifies that samplers are
   [attached to an entire Trace provider](../trace/sdk.md#sampling)
   while [OpenCensus allows custom samplers per span](https://github.com/census-instrumentation/opencensus-specs/blob/master/trace/Sampling.md#how-can-users-control-the-sampler-that-is-used-for-sampling).
4. TraceFlags in both OpenCensus and OpenTelemetry only specify the single
   `sampled` flag ([OpenTelemetry](../trace/api.md#spancontext),
   [OpenCensus](https://github.com/census-instrumentation/opencensus-specs/blob/master/trace/TraceConfig.md#traceparams)).
   Some OpenCensus APIs support "debug" and "defer" tracing flags in additon to
   "sampled".  In this case, the OpenCensus bridge will do its best to support
   and translate unspecified flags into the closest OpenTelemetry equivalent.
-->

1. OpenCensusでは、子Spanに親Spanを指定できるタイミングが[仕様上未定義](https://github.com/census-instrumentation/opencensus-specs/blob/master/trace/Span.md#span-creation)となっています。OpenTelemetryでは、[親SpanはSpan作成時に指定しなければならない](../trace/api.md#span-creation)と規定されています。 このため、初期化後に親Spanを柔軟に指定できるOpenCensusのAPIでは、いくつかの問題が発生します。

2. Spanが作成された後にSpanに追加されたリンク。これは[OpenTelemetryではサポートされていない](../trace/api.md#specifying-links)ので、作成後にリンクが追加されたOpenCensusのSpanは、リンクのないOpenTelemetryのSpanにマッピングされます。

3. OpenTelemetryでは、Samplerは[Traceプロバイダ全体に付けられる](../trace/sdk.md#sampling)と規定されていますが、OpenCensusでは[SpanごとにカスタムSamplerを設定可能](https://github.com/census-instrumentation/opencensus-specs/blob/master/trace/Sampling.md#how-can-users-control-the-sampler-that-is-used-for-sampling)となっています。 

4. OpenCensus と OpenTelemetry の両方の TraceFlags は、単一の `sampled` フラグのみを指定します ([OpenTelemetry](../trace/api.md#spancontext), [OpenCensus](https://github.com/census-instrumentation/opencensus-specs/blob/master/trace/TraceConfig.md#traceparams))。 OpenCensus API の中には、"sampled" に加えて、"debug" や "defer" といったトレースフラグをサポートしているものもあります。 この場合、OpenCensus ブリッジは、指定されていないフラグをサポートし、最も近い OpenTelemetry と同等のものに変換するように最善を尽くします。

<!--
### Context Propagation
-->

### Context 伝搬

<!--
The shim will provide an OpenCensus `PropagationComponent` implementation which
maps OpenCenus binary and text propagation to OpenTelemetry context.
-->

Shimは、OpenCenusのバイナリやテキストの伝搬をOpenTelemetryのContextにマッピングするOpenCensusの`PropagationComponent`の実装を提供します。

<!--
#### Text Context
-->

#### Text Context

<!--
This adapter MUST use an OpenTelemetry `TextMapPropagator` to implement the
OpenCensus `TextFormat`.
-->

このアダプタは、OpenCensus の `TextFormat` を実装するために、OpenTelemetry の `TextMapPropagator` を使用しなければなりません(MUST)。

<!--
This adapter SHOULD use configured OpenTelemetry `TextMapPropagator` on the
OpenTelemetry `TraceProvider` for text format propagation.
-->

このアダプタは、OpenTelemetry `TraceProvider` 上で設定された OpenTelemetry `TextMapPropagator` を使用して、テキスト形式の伝搬を行うべきです(SHOULD)。

<!--
This adapter MUST provide a default `W3CTraceContextPropagator`.  If
OpenTelemetry defines a global TextMapPropogator, OpenCensus SHOULD use this
for OpenCensus `traceContextFormat` propagation.
-->

このアダプタは、デフォルトの `W3CTraceContextPropagator` を提供しなければなりません (MUST)。 OpenTelemetry がグローバルな TextMapPropogator を定義している場合、OpenCensus は OpenCensus の `traceContextFormat` 伝播のためにこれを使用すべきです (SHOULD)。

<!--
#### B3 Context
-->

#### B3 Context

<!--
This adapter SHOULD use a contributed OpenTelemetry `B3Propagator` for the
B3 text format.
-->

このアダプタは、B3テキストフォーマットのために、コントリビュートされたOpenTelemetry `B3Propagator`を使用すべきです(SHOULD)。

<!--
#### OpenCensus Binary Context
-->

#### OpenCensus バイナリ Context

<!--
This adapter MUST provide an implementation of OpenCensus `BinaryPropogator` to
write OpenCensus binary format using OpenTelemetry's context.  This
implementation may be drawn from OpenCensus if applicable.
-->

このアダプタは、OpenTelemetry のコンテキストを使用して OpenCensus のバイナリ形式を書き込むために、OpenCensus の `BinaryPropogator` の実装を提供しなければなりません。 この実装は、必要に応じて OpenCensus から引用できます。

<!--
### Resources
-->

### Resources

<!--
Note: resources appear not to be usable in the "API" section of OpenCensus.
-->

注:OpenCensusの"API"セクションでは、リソースは使用できないようです。

<!--
### Semantic Convention Mappings
-->

### Semantic 規約のマッピング

<!--
Where possible, the tracing shim should provide mappings of labels defined
within the OpenTelemetry semantic convetions.  
-->

可能な限り、TraceのShimはOpenTelemetryのセマンティック規約で定義されたラベルのマッピングを提供する必要があります。

<!--
> The principle is to ensure OpenTelemetry exporters, which use these semantic
> conventions, are likely to export the correct data.
-->

> この原則は、これらのセマンティック規約を使用しているOpenTelemetryエクスポーターが、正しいデータをエクスポートできるようにすることです。

<!--
#### HTTP Attributes
-->

#### HTTP 属性

<!--
OpenCensus specifies the following [HTTP Attributes](https://github.com/census-instrumentation/opencensus-specs/blob/master/trace/HTTP.md#attributes):
-->

OpenCensusでは、以下の[HTTP 属性](https://github.com/census-instrumentation/opencensus-specs/blob/master/trace/HTTP.md#attributes)を規定しています。

<!--
| OpenCensus Name    | OpenTelemetry Name | Comments             |
| ------------------ | ------------------ |----------------------|
| `http.host`        | `http.host`        |                      |
| `http.method`      | `http.method`      |                      |
| `http.user_agent`  | `http.user_agent`  |                      |
| `http.status_code` | `http.status_code` |                      |
| `http.url`         | `http.url`         |                      |
| `http.path`        | `http.target`      | key-name change only |
| `http.route`       | N/A                | Pass through ok      |
-->

| OpenCensus 名      | OpenTelemetry 名   | コメント              |
| ------------------ | ------------------ |----------------------|
| `http.host`        | `http.host`        |                      |
| `http.method`      | `http.method`      |                      |
| `http.user_agent`  | `http.user_agent`  |                      |
| `http.status_code` | `http.status_code` |                      |
| `http.url`         | `http.url`         |                      |
| `http.path`        | `http.target`      | key-name change only |
| `http.route`       | N/A                | Pass through ok      |

<!--
## Metrics / Stats
-->

## Metrics / Stats

<!--
Metric compatibility with OpenCensus remains unspecified as the OpenTelemetry
metrics specification solidifies for GA.   Once GA on metrics is declared,
this section will be filled out.
-->

OpenCensusとのMetricsの互換性は、OpenTelemetryのMetrics仕様がGAに向けて固まっていく中で、まだ特定されていません。Metricsに関するGAが宣言されれば、このセクションは記入されます。

<!--
> Philosophically, this should follow the same principles as Trace.
> Specifically: Labels/Metric names should be converted to OTel semantic
> conventions, All API surface area should map to the closest relevant OTel
> API and no SDK usage of OpenCensus will be compatible.
-->

> 哲学的には、これはTraceと同じ原則に従うべきです。具体的には ラベル/メトリック名は、OTelのセマンティック規約に変換されるべきであり、すべてのAPIは、最も近い関連するOTelのAPIにマッピングされるべきであり、OpenCensusのSDKの使用は互換性がありません。
