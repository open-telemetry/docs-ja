<!--
# Tracing SDK
-->

# Tracing SDK


**Status**: [Stable](../document-status.md)

<details>

<summary>目次</summary>

<!--
* [Tracer Provider](#tracer-provider)
* [Additional Span Interfaces](#additional-span-interfaces)
* [Sampling](#sampling)
* [Span Limits](#span-limits)
* [Id Generator](#id-generators)
* [Span Processor](#span-processor)
* [Span Exporter](#span-exporter)
-->

* [Tracer Provider](#tracer-provider)
* [追加のSpanインターフェイス](#追加のSpanインターフェイス)
* [サンプリング](#サンプリング)
* [Span Limits](#span-limits)
* [Id Generator](#id-generators)
* [Span Processor](#span-processor)
* [Span Exporter](#span-exporter)


</details>

<!--
## Tracer Provider
-->

## Tracer Provider

<!--
### Tracer Creation
-->

### Tracerの作成

<!--
New `Tracer` instances are always created through a `TracerProvider` (see
[API](api.md#tracerprovider)). The `name` and `version` arguments
supplied to the `TracerProvider` must be used to create an
[`InstrumentationLibrary`][otep-83] instance which is stored on the created
`Tracer`.
-->

新しい `Tracer` インスタンスは常に `TracerProvider` (参照: [API](api.md#tracerprovider)) を通して作成されます。`TracerProvider`に与えられる`name`と`version`の引数は、作成された`Tracer`に格納される[`InstrumentationLibrary`][otep-83]のインスタンスを作成するために使用されなければなりません。

<!--
Configuration (i.e., [SpanProcessors](#span-processor), [IdGenerator](#id-generators),
[SpanLimits](#span-limits) and [`Sampler`](#sampling)) MUST be managed solely by
the `TracerProvider` and it MUST provide some way to configure all of them that
are implemented in the SDK, at least when creating or initializing it.
-->

設定(すなわち、[SpanProcessors](#span-processor)、[IdGenerator](#id-generators)、[SpanLimits](#span-limits)、[`Sampler`](#sampling)です) は、`TracerProvider`によってのみ管理されなければならず、少なくとも作成時や初期化時には、SDKに実装されているそれらすべてを設定する何らかの方法を提供しなければなりません(MUST)。

<!--
The TracerProvider MAY provide methods to update the configuration. If
configuration is updated (e.g., adding a `SpanProcessor`),
the updated configuration MUST also apply to all already returned `Tracers`
(i.e. it MUST NOT matter whether a `Tracer` was obtained from the
`TracerProvider` before or after the configuration change).
Note: Implementation-wise, this could mean that `Tracer` instances have a
reference to their `TracerProvider` and access configuration only via this
reference.
-->

TracerProvider は設定を更新するためのメソッドを提供しても構いません(MAY)。設定が更新された場合(例:`SpanProcessor`の追加)、更新された設定はすでに返されたすべての`Tracer`にも適用されなければなりません(すなわち、設定の変更の前後に`TracerProvider`から`Tracer`を取得したかどうかは問題にしてはいけません(MUST NOT))。注:実装上、これは `Tracer` インスタンスがその `TracerProvider` への参照を持ち、この参照を介してのみ構成にアクセスすることを意味します。

<!--
### Shutdown
-->

### Shutdown

<!--
This method provides a way for provider to do any cleanup required.
-->

このメソッドは、Providerが必要なクリーンアップを行うための方法を提供します。

<!--
`Shutdown` MUST be called only once for each `TracerProvider` instance. After
the call to `Shutdown`, subsequent attempts to get a `Tracer` are not allowed. SDKs
SHOULD return a valid no-op Tracer for these calls, if possible.
-->

`Shutdown` は各 `TracerProvider` インスタンスに対して一度だけ呼び出さなければなりません(MUST)。`Shutdown` の呼び出し後、それ以降に `Tracer` を取得しようとすることはできません。SDKは可能であれば、これらの呼び出しに対して有効なno-opのTracerを返すべきです(SHOULD)。

<!--
`Shutdown` SHOULD provide a way to let the caller know whether it succeeded,
failed or timed out.
-->

`Shutdown` は成功したのか、失敗したのか、あるいはタイムアウトしたのかを呼び出し元に知らせる方法を提供すべきです(SHOULD)。

<!--
`Shutdown` SHOULD complete or abort within some timeout. `Shutdown` can be
implemented as a blocking API or an asynchronous API which notifies the caller
via a callback or an event. OpenTelemetry client authors can decide if they want to
make the shutdown timeout configurable.
-->

`Shutdown`は、あるタイムアウト内に完了または中止すべきです(SHOULD。`Shutdown`はブロッキングAPIとして実装することも、コールバックやイベントで呼び出し元に通知する非同期APIとして実装することもできます。OpenTelemetryクライアントの作者は、シャットダウンのタイムアウトを設定可能にするかどうかを決めることができます。

<!--
`Shutdown` MUST be implemented at least by invoking `Shutdown` within all internal processors.
-->

`Shutdown`は、少なくともすべての内部プロセッサ内で`Shutdown`を呼び出すことで実装しなければなりません(MUST)。

<!--
### ForceFlush
-->

### ForceFlush

<!--
This method provides a way for provider to immediately export all spans that have not yet been exported for all the internal processors.
-->

このメソッドは、プロバイダがすべての内部プロセッサに対してまだエクスポートされていないすべてのSpanを直ちにエクスポートする方法を提供します。

<!--
`ForceFlush` SHOULD provide a way to let the caller know whether it succeeded,
failed or timed out.
-->

`ForceFlush` は成功したのか、失敗したのか、あるいはタイムアウトしたのかを呼び出し側に知らせる方法を提供すべきです(SHOULD)。

<!--
`ForceFlush` SHOULD complete or abort within some timeout. `ForceFlush` can be
implemented as a blocking API or an asynchronous API which notifies the caller
via a callback or an event. OpenTelemetry client authors can decide if they want to
make the flush timeout configurable.
-->

`ForceFlush`はあるタイムアウト内に完了または中止すべきです(SHOULD)。`ForceFlush`はブロッキングAPIとして実装することも、コールバックやイベントで呼び出し元に通知する非同期APIとして実装することもできます。OpenTelemetryクライアントの作者は、フラッシュのタイムアウトを設定可能にするかどうかを決めることができます。

<!--
`ForceFlush` MUST invoke `ForceFlush` on all registered `SpanProcessors`.
-->

`ForceFlush` は、登録されたすべての `SpanProcessors` 上で `ForceFlush` を呼び出さなければなりません(MUST)。

<!--
## Additional Span Interfaces
-->

## 追加のSpanインターフェイス

<!--
The [API-level definition for Span's interface](api.md#span-operations)
only defines write-only access to the span.
This is good because instrumentations and applications are not meant to use the data
stored in a span for application logic.
However, the SDK needs to eventually read back the data in some locations.
Thus, the SDK specification defines sets of possible requirements for
`Span`-like parameters:
-->

[SpanのインターフェイスのAPIレベルの定義](api.md#span-operations)では、Spanへの書き込みのみのアクセスを定義しています。計装やアプリケーションは、Spanに格納されたデータをアプリケーションロジックに使用することを意図していないため、これは良いことです。しかし、SDKでは最終的にいくつかの場所のデータを読み返す必要があります。そこで、SDKの仕様では、`Span`のようなパラメータに対する可能な要件のセットを定義しています。

<!--
* **Readable span**: A function receiving this as argument MUST be able to
  access all information that was added to the span,
  as listed [in the API spec](api.md#span-data-members).
  In particular, it MUST also be able to access
  the `InstrumentationLibrary` and `Resource` information (implicitly)
  associated with the span.
  It must also be able to reliably determine whether the Span has ended
  (some languages might implement this by having an end timestamp of `null`,
  others might have an explicit `hasEnded` boolean).

  A function receiving this as argument might not be able to modify the Span.

  Note: Typically this will be implemented with a new interface or
  (immutable) value type.
  In some languages SpanProcessors may have a different readable span type
  than exporters (e.g. a `SpanData` type might contain an immutable snapshot and
  a `ReadableSpan` interface might read information directly from the same
  underlying data structure that the `Span` interface manipulates).
-->

* **読み取り可能なSpan**。これを引数として受け取る関数は、[API仕様書](api.md#span-data-members)に記載されているように、Spanに追加されたすべての情報にアクセスできなければなりません(MUST)。特に、Spanに関連付けられた `InstrumentationLibrary` と `Resource` の情報にも(暗黙のうちに)アクセスできなければなりません。また、Spanが終了したかどうかを確実に判断できなければなりません(言語によっては、終了タイムスタンプを `null` にすることで実装している場合もあるし、明示的に `hasEnded` という真偽値を設定している場合もあります)。

  これを引数として受け取る関数は、Spanを変更できないかもしれません。

  注:典型的には、これは新しいインターフェースや(不変の)値のタイプで実装されます。いくつかの言語では、Spanプロセッサは、エクスポートとは異なる読み取り可能なSpanタイプを持つかもしれません(例えば、`SpanData`タイプは、不変のスナップショットを含み、`ReadableSpan`インターフェースは、`Span`インターフェースが操作するのと同じ基礎的なデータ構造から直接情報を読み取るかもしれません)。

<!--
* **Read/write span**: A function receiving this as argument must have access to
  both the full span API as defined in the
  [API-level definition for span's interface](api.md#span-operations) and
  additionally must be able to retrieve all information that was added to the span
  (as with *readable span*).

  It MUST be possible for functions being called with this
  to somehow obtain the same `Span` instance and type
  that the [span creation API](api.md#span-creation) returned (or will return) to the user
  (for example, the `Span` could be one of the parameters passed to such a function,
  or a getter could be provided).
-->

* **読み取り/書き込みするSpan**。これを引数として受け取る関数は、[SpanのインターフェイスのAPIレベルの定義](api.md#span-operations)で定義されている完全なspan APIの両方にアクセスできなければならず、さらにspanに追加されたすべての情報を取得できなければなりません(*読み取り可能なSpan*の場合)。

  これで呼び出される関数は、[span 作成API](api.md#span-creation)がユーザーに返した(または返す予定の)のと同じ `Span` インスタンスとタイプを何らかの形で取得できるようにしなければなりません(MUST)(例えば、`Span` をそのような関数に渡されるパラメータの1つにしたり、getterを提供したりすることができます)。

<!--
## Sampling
-->

## サンプリング

<!--
Sampling is a mechanism to control the noise and overhead introduced by
OpenTelemetry by reducing the number of samples of traces collected and sent to
the backend.
-->

サンプリングとは、収集したトレースのサンプル数を減らしてバックエンドに送信することで、OpenTelemetryによってもたらされるノイズやオーバーヘッドを抑制する仕組みです。

<!--
Sampling may be implemented on different stages of a trace collection. The
earliest sampling could happen before the trace is actually created, and the
latest sampling could happen on the Collector which is out of process.
-->

サンプリングは、トレース収集のさまざまな段階で実施することができます。最も早いサンプリングは、トレースが実際に作成される前に行われ、最も遅いサンプリングは、プロセスが終了したコレクターで行われる可能性があります。

<!--
The OpenTelemetry API has two properties responsible for the data collection:
-->

OpenTelemetry APIには、データ収集を担当する2つのプロパティがあります。

<!--
* `IsRecording` field of a `Span`. If `false` the current `Span` discards all
  tracing data (attributes, events, status, etc.). Users can use this property
  to determine if collecting expensive trace data can be avoided. [Span
  Processor](#span-processor) MUST receive only those spans which have this
  field set to `true`. However, [Span Exporter](#span-exporter) SHOULD NOT
  receive them unless the `Sampled` flag was also set.
* `Sampled` flag in `TraceFlags` on `SpanContext`. This flag is propagated via
  the `SpanContext` to child Spans. For more details see the [W3C Trace Context
  specification](https://www.w3.org/TR/trace-context/#sampled-flag). This flag indicates that the `Span` has been
  `sampled` and will be exported. [Span Exporters](#span-exporter) MUST
  receive those spans which have `Sampled` flag set to true and they SHOULD NOT receive the ones
  that do not.
-->

* `Span`の`IsRecording`フィールド: `false`の場合、現在の`Span`はすべてのトレースデータ(属性、イベント、ステータスなど)を破棄します。ユーザーはこのプロパティを使用して、高価なトレースデータの収集を回避できるかどうかを判断できます。[Spanプロセッサ](#span-processor)は、このフィールドが`true`に設定されているSpanのみを受信しなければなりません(MUST)。しかし、[Span Exporter](#span-exporter)は、`Sampled`フラグも同時に設定されていない限り、これらのSpanを受信すべきではありません(SHOULD NOT)。 

* `SpanContext` の `TraceFlags` にある `Sampled` フラグ: このフラグは`SpanContext`を介して子のSpanに伝搬されます。詳細については、[W3C Trace Context specification](https://www.w3.org/TR/trace-context/#sampled-flag)を参照してください。このフラグは、`Span` が `sampled` され、エクスポートされることを示します。[Spanエクスポート](#span-exporter)は、`Sampled`フラグがtrueに設定されているSpanを受信しなければならず(MUST)、そうでないものを受信するべきではありません(SHOULD NOT)。

<!--
The flag combination `SampledFlag == false` and `IsRecording == true`
means that the current `Span` does record information, but most likely the child
`Span` will not.
-->

`SampledFlag == false`と`IsRecording == true`のフラグの組み合わせは、現在の`Span`は情報を記録しているが、子`Span`では記録しない可能性が高いことを意味します。

<!--
The flag combination `SampledFlag == true` and `IsRecording == false`
could cause gaps in the distributed trace, and because of this OpenTelemetry API
MUST NOT allow this combination.
-->

`SampledFlag == true`と`IsRecording == false`のフラグの組み合わせは、分散したトレースにギャップを生じさせる可能性があります。このため、OpenTelemetry APIはこの組み合わせを許可してはなりません(MUST NOT)。

<!--
<a name="recording-sampled-reaction-table"></a>
-->

<a name="recording-sampled-reaction-table"></a>

<!--
The following table summarizes the expected behavior for each combination of
`IsRecording` and `SampledFlag`.
-->

次の表は、`IsRecording`と`SampledFlag`の各組み合わせで期待される動作をまとめたものです。

<!--
| `IsRecording` | `Sampled` Flag | Span Processor receives Span? | Span Exporter receives Span? |
| ------------- | -------------- | ----------------------------- | ---------------------------- |
| true          | true           | true                          | true                         |
| true          | false          | true                          | false                        |
| false         | true           | Not allowed                   | Not allowed                  |
| false         | false          | false                         | false                        |
-->

| `IsRecording` | `Sampled` Flag | Span Processor はSpanを受信する? | Span Exporter はSpanを受信する? |
| ------------- | -------------- | ------------------------------- | ---------------------------- |
| true          | true           | true                            | true                         |
| true          | false          | true                            | false                        |
| false         | true           | 許可されていない                 | 許可されていない              |
| false         | false          | false                           | false                        |

<!--
The SDK defines the interface [`Sampler`](#sampler) as well as a set of
[built-in samplers](#built-in-samplers) and associates a `Sampler` with each [`TracerProvider`].
-->

SDKでは、[`Sampler`](#sampler)というインターフェースと、[組み込みsamplers](#built-in-samplers)のセットを定義し、各[`TracerProvider`]に`Sampler`を関連付けています。

<!--
### SDK Span creation
-->

### SDK Span 作成

<!--
When asked to create a Span, the SDK MUST act as if doing the following in order:
-->

Spanの作成を依頼された場合、SDKは以下を順に行うように行動しなければなりません(MUST)。

<!--
1. If there is a valid parent trace ID, use it. Otherwise generate a new trace ID
   (note: this must be done before calling `ShouldSample`, because it expects
   a valid trace ID as input).
2. Query the `Sampler`'s [`ShouldSample`](#shouldsample) method
   (Note that the [built-in `ParentBasedSampler`](#parentbased) can be used to
   use the sampling decision of the parent,
   translating a set SampledFlag to RECORD and an unset one to DROP).
3. Generate a new span ID for the `Span`, independently of the sampling decision.
   This is done so other components (such as logs or exception handling) can rely on
   a unique span ID, even if the `Span` is a non-recording instance.
4. Create a span depending on the decision returned by `ShouldSample`:
   see description of [`ShouldSample`'s](#shouldsample) return value below
   for how to set `IsRecording` and `Sampled` on the Span,
   and the [table above](#recording-sampled-reaction-table) on whether
   to pass the `Span` to `SpanProcessor`s.
   A non-recording span MAY be implemented using the same mechanism as when a
   `Span` is created without an SDK installed or as described in
   [wrapping a SpanContext in a Span](api.md#wrapping-a-spancontext-in-a-span).
-->

1. 有効な親のTraceIDがあればそれを使用する。そうでなければ、新しいTraceIDを生成します(注意: これは `ShouldSample` を呼ぶ前に行わなければなりません。有効なトレースIDが入力されるからです)。

2. `Sampler` の [`ShouldSample`](#shouldsample) メソッドに問い合わせます (なお、[組み込みの`ParentBasedSampler`](#parentbased) を使用すると、親のサンプリング決定を使用することができ、SampledFlag が設定されている場合は RECORD に、設定されていない場合は DROP に変換されます)。

3. サンプリングの決定とは別に、`Span` に対して新しいSpanID を生成する。 これは、他のコンポーネント(ログや例外処理など)が、たとえ `Span` が非記録的なインスタンスであっても、一意のSpanID に依存できるようにするために行われます。


4. `ShouldSample`によって返される決定に応じて、Spanを作成します。Spanに `IsRecording` と `Sampled` を設定する方法については、以下の [`ShouldSample` の](#shouldsample) 返却値の説明を、`Span` を `SpanProcessor` に渡すかどうかについては、上記の [表](#recording-sampled-reaction-table) を参照してください。 記録されないSpanは、SDKがインストールされていない状態で`Span`が作成されたときと同じメカニズム、もしくは[SpanContextをSpanでラップする](api.md#SpanContextをSpanでラップする)に記載されている方法で実装してもかまいません(MAY)。

<!--
### Sampler
-->

### Sampler

<!--
`Sampler` interface allows users to create custom samplers which will return a
sampling `SamplingResult` based on information that is typically available just
before the `Span` was created.
-->

`Sampler` インターフェースでは、`Span` が作成される直前に一般に入手可能な情報に基づいて、サンプリングの `SamplingResult` を返すカスタムのSamplerを作成することができます。

<!--
#### ShouldSample
-->

#### ShouldSample

<!--
Returns the sampling Decision for a `Span` to be created.
-->

作成される `Span` のサンプリング判定を返します。

<!--
**Required arguments:**
-->

**必要な引数:**

<!--
* [`Context`](../context/context.md) with parent `Span`.
  The Span's SpanContext may be invalid to indicate a root span.
* `TraceId` of the `Span` to be created.
  If the parent `SpanContext` contains a valid `TraceId`, they MUST always match.
* Name of the `Span` to be created.
* `SpanKind` of the `Span` to be created.
* Initial set of `Attributes` of the `Span` to be created.
* Collection of links that will be associated with the `Span` to be created.
  Typically useful for batch operations, see
  [Links Between Spans](../overview.md#links-between-spans).
-->

* 親の`Span`を持つ[`Context`](../context/context.md)。SpanのSpanContextは、ルートSpanを示すために、invalidな場合があります。
* 作成される `Span` の `TraceId`。親の `SpanContext` が有効な `TraceId` を含んでいる場合、それらは常に一致しなければなりません。
* 作成される `Span` の名前
* 作成される `Span` の `SpanKind`
* 作成される `Span` の `Attributes` の初期セット
* 作成する `Span` に関連付けられるリンクのコレクション。典型的にはバッチ操作に有用です。[Links Between Spans](../overview.md#links-between-spans)を参照してください。

<!--
**Return value:**
-->

**返り値:**

<!--
It produces an output called `SamplingResult` which contains:
-->

以下の情報を持つ`SamplingResult`という出力を生成します。

<!--
* A sampling `Decision`. One of the following enum values:
  * `DROP` - `IsRecording() == false`, span will not be recorded and all events and attributes
  will be dropped.
  * `RECORD_ONLY` - `IsRecording() == true`, but `Sampled` flag MUST NOT be set.
  * `RECORD_AND_SAMPLE` - `IsRecording() == true` AND `Sampled` flag MUST be set.
* A set of span Attributes that will also be added to the `Span`. The returned
object must be immutable (multiple calls may return different immutable objects).
* A `Tracestate` that will be associated with the `Span` through the new
  `SpanContext`.
  If the sampler returns an empty `Tracestate` here, the `Tracestate` will be cleared,
  so samplers SHOULD normally return the passed-in `Tracestate` if they do not intend
  to change it.
-->

* サンプリングするかどうかの`Decision`. 以下の列挙型の値のいずれかです。
  * `DROP` - `IsRecording() == false`, Spanは記録されず、すべてのイベントと属性はドロップされます。
  * `RECORD_ONLY` - `IsRecording() == true`, ただし `Sampled` フラグは設定してはいけません(MUST NOT)。
  * `RECORD_AND_SAMPLE` - `IsRecording() == true` かつ `Sampled` フラグを設定しなければなりません(MUST)。
* `Span` にも追加される span 属性のセット。返されるオブジェクトは、不変でなければなりません(複数回の呼び出しにより、異なる不変のオブジェクトが返される可能性があります)。
* 新しい `SpanContext` を通じて、`Span` に関連付けられる `Tracestate`
  Samplerがここで空の`Tracestate`を返すと`Tracestate`はクリアされるので、Samplerは通常、変更するつもりがなければ、渡された`Tracestate`を返すべきです(SHOULD)。

<!--
#### GetDescription
-->

#### GetDescription

<!--
Returns the sampler name or short description with the configuration. This may
be displayed on debug pages or in the logs. Example:
`"TraceIdRatioBased{0.000100}"`.
-->

設定されているSamplerの名前または短い説明を返します。これは、デバッグページやログに表示されることがあります。例: `"TraceIdRatioBased{0.000100}"`。

<!--
Description MUST NOT change over time and caller can cache the returned value.
-->

説明は時間とともに変化してはならず(MUST NOT)、呼び出し元は返された値をキャッシュすることができます。

<!--
### Built-in samplers
-->

### 組み込みsampler

<!--
OpenTelemetry supports a number of built-in samplers to choose from.
The default sampler is `ParentBased(root=AlwaysOn)`.
-->

OpenTelemetryはいくつかの組み込みSamplerをサポートしており、それらの中から選ぶことができます。デフォルトのSamplerは`ParentBased(root=AlwaysOn)`です。

<!--
#### AlwaysOn
-->

#### AlwaysOn

<!--
* Returns `RECORD_AND_SAMPLE` always.
* Description MUST be `AlwaysOnSampler`.
-->

* 常に `RECORD_AND_SAMPLE` を返します。
* Description は `AlwaysOnSampler` でなければなりません(MUST)。

<!--
#### AlwaysOff
-->

#### AlwaysOff

<!--
* Returns `DROP` always.
* Description MUST be `AlwaysOffSampler`.
-->

* 常に `DROP` を返します。
* Description は `AlwaysOffSampler` でなければなりません(MUST)。

<!--
#### TraceIdRatioBased
-->

#### TraceIdRatioBased

<!--
* The `TraceIdRatioBased` MUST ignore the parent `SampledFlag`. To respect the
parent `SampledFlag`, the `TraceIdRatioBased` should be used as a delegate of
the `ParentBased` sampler specified below.
* Description MUST return a string of the form `"TraceIdRatioBased{RATIO}"`
  with `RATIO` replaced with the Sampler instance's trace sampling ratio
  represented as a decimal number. The precision of the number SHOULD follow
  implementation language standards and SHOULD be high enough to identify when
  Samplers have different ratios. For example, if a TraceIdRatioBased Sampler
  had a sampling ratio of 1 to every 10,000 spans it COULD return
  `"TraceIdRatioBased{0.000100}"` as its description.
-->

* `TraceIdRatioBased` は、親の `SampledFlag` を無視しなければなりません(MUST)。親の `SampledFlag` を尊重するために、`TraceIdRatioBased` は下記の `ParentBased` Samplerの移譲先として使用する必要があります。

* Description は `"TraceIdRatioBased{RATIO}"` という形式の文字列を返さなければなりません(MUST)。`RATIO` はSamplerインスタンスのトレースサンプリング比率に置き換えられ、10進数で表されます。この数値の精度は、実装言語の標準に従うべき(SHOULD)であり、Samplerの比率が異なる場合に識別できる程度に高くすべきです(SHOULD)。例えば、TraceIdRatioBased Samplerが10,000Spanごとに1のサンプリング比率を持つ場合、その説明として `"TraceIdRatioBased{0.000100}"` を返すことができます(COULD)。

<!--
TODO: Add details about how the `TraceIdRatioBased` is implemented as a function
of the `TraceID`. [#1413](https://github.com/open-telemetry/opentelemetry-specification/issues/1413)
-->

TODO: `TraceIdRatioBased` が `TraceID` の関数としてどのように実装されているかの詳細を追加する。[#1413](https://github.com/open-telemetry/opentelemetry-specification/issues/1413)

<!--
##### Requirements for `TraceIdRatioBased` sampler algorithm
-->

##### `TraceIdRatioBased` Sampler アルゴリズムの要件

<!--
* The sampling algorithm MUST be deterministic. A trace identified by a given
  `TraceId` is sampled or not independent of language, time, etc. To achieve this,
  implementations MUST use a deterministic hash of the `TraceId` when computing
  the sampling decision. By ensuring this, running the sampler on any child `Span`
  will produce the same decision.
* A `TraceIdRatioBased` sampler with a given sampling rate MUST also sample all
  traces that any `TraceIdRatioBased` sampler with a lower sampling rate would
  sample. This is important when a backend system may want to run with a higher
  sampling rate than the frontend system, this way all frontend traces will
  still be sampled and extra traces will be sampled on the backend only.
* **WARNING:** Since the exact algorithm is not specified yet (see TODO above),
  there will probably be changes to it in any language SDK once it is, which
  would break code that relies on the algorithm results.
  Only the configuration and creation APIs can be considered stable.
  It is recommended to use this sampler algorithm only for root spans
  (in combination with [`ParentBased`](#parentbased)) because different language
  SDKs or even different versions of the same language SDKs may produce inconsistent
  results for the same input.
-->

* サンプリングアルゴリズムは決定論的(Deterministic)でなければなりません(MUST)。与えられた `TraceId` で識別されるTraceは、言語や時間などに依存せずにサンプリングされるかどうかが決まります。これを実現するために、実装はサンプリングの決定を計算する際に、`TraceId` の決定論的なハッシュを使用しなければなりません (MUST)。これを確実にすることで、どの子`Span`に対してもSamplerを実行しても同じ判定が得られます。


* あるサンプリングレートの `TraceIdRatioBased` Samplerは、より低いサンプリングレートの `TraceIdRatioBased` Samplerがサンプリングするであろうすべてのトレースもサンプリングしなければなりません (MUST)。これは、バックエンドシステムがフロントエンドシステムよりも高いサンプリングレートで動作させたい場合に重要です。この方法では、フロントエンドのトレースはすべてサンプリングされ、バックエンドでのみ追加のトレースがサンプリングされます。

* **警告:** 正確なアルゴリズムはまだ指定されていないため(上記のTODOを参照)、おそらくどの言語のSDKでも、アルゴリズムが変更された時点で、そのアルゴリズムの結果に依存するコードが壊れることになるでしょう。 安定していると言えるのは、設定と作成のAPIだけです。 異なる言語SDKや、同じ言語SDKの異なるバージョンであっても、同じ入力に対して矛盾した結果を出す可能性があるため、このSamplerアルゴリズムはルートSpanにのみ([`ParentBased`](#parentbased)と組み合わせて)使用することをお勧めします。

<!--
#### ParentBased
-->

#### ParentBased

<!--
* This is a composite sampler. `ParentBased` helps distinguished between the
following cases:
  * No parent (root span).
  * Remote parent (`SpanContext.IsRemote() == true`) with `SampledFlag` equals `true`
  * Remote parent (`SpanContext.IsRemote() == true`) with `SampledFlag` equals `false`
  * Local parent (`SpanContext.IsRemote() == false`) with `SampledFlag` equals `true`
  * Local parent (`SpanContext.IsRemote() == false`) with `SampledFlag` equals `false`
-->

* これは合成Samplerです。`ParentBased` は、次のようなケースを区別するのに役立ちます。
  * 親がいない(ルートSpan)
  * リモートの親 (`SpanContext.IsRemote() == true`) で `SampledFlag` が `true` と等しい場合
  * リモートの親 (`SpanContext.IsRemote() == true`) で `SampledFlag` が `false` と等しい場合
  * ローカルの親 (`SpanContext.IsRemote() == false`) で `SampledFlag` が `true` と等しい場合
  * ローカルの親 (`SpanContext.IsRemote() == false`) で `SampledFlag` が `false` に等しい場合

<!--
Required parameters:
-->

必須パラメータ:

<!--
* `root(Sampler)` - Sampler called for spans with no parent (root spans)
-->

* `root(Sampler)` - 親のいないSpan(ルートSpan)に対して呼び出されるSampler

<!--
Optional parameters:
-->

任意のパラメータ:

<!--
* `remoteParentSampled(Sampler)` (default: AlwaysOn)
* `remoteParentNotSampled(Sampler)` (default: AlwaysOff)
* `localParentSampled(Sampler)` (default: AlwaysOn)
* `localParentNotSampled(Sampler)` (default: AlwaysOff)
-->

* `remoteParentSampled(Sampler)` (デフォルト: AlwaysOn)
* `remoteParentNotSampled(Sampler)` (デフォルト: AlwaysOff)
* `localParentSampled(Sampler)` (デフォルト: AlwaysOn)
* `localParentNotSampled(Sampler)` (デフォルト: AlwaysOff)

<!--
|Parent| parent.isRemote() | parent.IsSampled()| Invoke sampler|
|--|--|--|--|
|absent| n/a | n/a |`root()`|
|present|true|true|`remoteParentSampled()`|
|present|true|false|`remoteParentNotSampled()`|
|present|false|true|`localParentSampled()`|
|present|false|false|`localParentNotSampled()`|
-->

|Parent| parent.isRemote() | parent.IsSampled()| Invoke sampler|
|--|--|--|--|
|absent| n/a | n/a |`root()`|
|present|true|true|`remoteParentSampled()`|
|present|true|false|`remoteParentNotSampled()`|
|present|false|true|`localParentSampled()`|
|present|false|false|`localParentNotSampled()`|

<!--
## Span Limits
-->

## Span Limits

<!--
Erroneous code can add unintended attributes, events, and links to a span. If
these collections are unbounded, they can quickly exhaust available memory,
resulting in crashes that are difficult to recover from safely.
-->

誤ったコードにより、意図しない属性、イベント、リンクがSpanに追加されてしまうことがあります。これらのコレクションが制限されていない場合、利用可能なメモリをあっという間に使い果たしてしまい、安全に回復するのが難しいクラッシュが発生してしまいます。

<!--
To protect against such errors, SDK Spans MAY discard attributes, links, and
events that would increase the number of elements of each collection beyond
the configured limit.
-->

このようなエラーを防ぐために、SDK Spansは、各コレクションの要素数が設定された上限を超えて増加するような属性、リンク、イベントを破棄しても構いません(MAY)。

<!--
If the SDK implements the limits above it MUST provide a way to change these
limits, via a configuration to the TracerProvider, by allowing users to
configure individual limits like in the Java example bellow.
-->

もしSDKが上記の制限を実装するならば、ユーザーが下記のJavaの例のように個々の制限を設定することで、TracerProviderへの設定を介してこれらの制限を変更する方法を提供しなければなりません(MUST)。

<!--
The name of the configuration options SHOULD be `AttributeCountLimit`,
`EventCountLimit` and `LinkCountLimit`. The options MAY be bundled in a class,
which then SHOULD be called `SpanLimits`. Implementations MAY provide additional
configuration such as `AttributePerEventCountLimit` and `AttributePerLinkCountLimit`.
-->

設定オプションの名前は、`AttributeCountLimit`、`EventCountLimit`、`LinkCountLimit`とすべきです(SHOULD)。オプションはクラスにまとめてもよく(MAY)、その場合は `SpanLimits` と呼ぶべきです(SHOULD)。実装では、`AttributePerEventCountLimit`や`AttributePerLinkCountLimit`などの追加設定を提供しても構いません(MAY)。

<!--
```java
public final class SpanLimits {
  SpanLimits(int attributeCountLimit, int linkCountLimit, int eventCountLimit);

  public int getAttributeCountLimit();

  public int getEventCountLimit();

  public int getLinkCountLimit();
}
```-->

```java
public final class SpanLimits {
  SpanLimits(int attributeCountLimit, int linkCountLimit, int eventCountLimit);

  public int getAttributeCountLimit();

  public int getEventCountLimit();

  public int getLinkCountLimit();
}
```

<!--
**Configurable parameters:**
-->

**設定可能なパラメータ:**

<!--
* `AttributeCountLimit` (Default=128) - Maximum allowed span attribute count;
* `EventCountLimit` (Default=128) - Maximum allowed span event count;
* `LinkCountLimit` (Default=128) - Maximum allowed span link count;
* `AttributePerEventCountLimit` (Default=128) - Maximum allowed attribute per span event count;
* `AttributePerLinkCountLimit` (Default=128) - Maximum allowed attribute per span link count;
-->

* `AttributeCountLimit` (Default=128) - 許容される最大のSpanのアトリビュート数
* `EventCountLimit` (Default=128) - 許容される最大のSpan イベント数
* `LinkCountLimit` (Default=128) - 許容される最大のSpan リンク数
* `AttributePerEventCountLimit` (Default=128) - Spanのイベント数あたりの、許容される最大のアトリビュート数
* `AttributePerLinkCountLimit` (Default=128) - Spanのリンク数あたりの、許容される最大のアトリビュート数

<!--
There SHOULD be a log emitted to indicate to the user that an attribute, event,
or link was discarded due to such a limit. To prevent excessive logging, the log
should not be emitted once per span, or per discarded attribute, event, or links.
-->

このような制限のために属性、イベント、またはリンクが破棄されたことをユーザに示すために、ログが発行されるべきです(SHOULD)。過剰なログを防ぐために、ログは1つのSpan、または廃棄された属性、イベント、リンクごとに1回発行されるべきではありません。

<!--
## Id Generators
-->

## Id Generators

<!--
The SDK MUST by default randomly generate both the `TraceId` and the `SpanId`.
-->

SDKはデフォルトで、`TraceId`と`SpanId`の両方をランダムに生成しなければなりません(MUST)。

<!--
The SDK MUST provide a mechanism for customizing the way IDs are generated for
both the `TraceId` and the `SpanId`.
-->

SDKは、`TraceId`と`SpanId`の両方に対して、IDを生成する方法をカスタマイズするためのメカニズムを提供しなければなりません(MUST)。

<!--
The SDK MAY provide this functionality by allowing custom implementations of
an interface like the java example below (name of the interface MAY be
`IdGenerator`, name of the methods MUST be consistent with
[SpanContext](./api.md#retrieving-the-traceid-and-spanid)), which provides
extension points for two methods, one to generate a `SpanId` and one for `TraceId`.
-->

SDKは、以下のJavaの例のようなインターフェースのカスタム実装を可能にすることで、この機能を提供しても構いません(インターフェースの名前は`IdGenerator`でも構いませんが(MAY)、メソッドの名前は[SpanContext](./api.md#retrieving-the-traceid-and-spanid)と一致しなければなりません(MUST))。

<!--
```java
public interface IdGenerator {
  byte[] generateSpanIdBytes();
  byte[] generateTraceIdBytes();
}
```
-->

```java
public interface IdGenerator {
  byte[] generateSpanIdBytes();
  byte[] generateTraceIdBytes();
}
```

<!--
Additional `IdGenerator` implementing vendor-specific protocols such as AWS
X-Ray trace id generator MUST NOT be maintained or distributed as part of the
Core OpenTelemetry repositories.
-->

AWS X-RayのトレースIDジェネレーターのようなベンダー固有のプロトコルを実装した追加の`IdGenerator`は、Core OpenTelemetryのリポジトリの一部として維持・配布してはいけません(MUST NOT)。

<!--
## Span processor
-->

## Span processor

<!--
Span processor is an interface which allows hooks for span start and end method
invocations. The span processors are invoked only when
[`IsRecording`](api.md#isrecording) is true.
-->

Spanプロセッサは、Spanの開始および終了メソッドの呼び出しのためのフックを可能にするインターフェースです。Spanプロセッサは、[`IsRecording`](api.md#isrecording)がtrueのときにのみ呼び出されます。

<!--
Built-in span processors are responsible for batching and conversion of spans to
exportable representation and passing batches to exporters.
-->

内蔵のSpanプロセッサは、Spanをバッチ処理してエクスポート可能な表現に変換し、バッチをExporterに渡す役割を果たします。

<!--
Span processors can be registered directly on SDK `TracerProvider` and they are
invoked in the same order as they were registered.
-->

Spanプロセッサは、SDKの `TracerProvider` で直接登録することができ、登録した順番に起動されます。

<!--
Each processor registered on `TracerProvider` is a start of pipeline that consist
of span processor and optional exporter. SDK MUST allow to end each pipeline with
individual exporter.
-->

`TracerProvider`に登録されている各プロセッサは、SpanプロセッサとオプションのExporterで構成されるパイプラインの開始点となります。SDKは、個々のExporterで各パイプラインを終了できるようにしなければなりません(MUST)。

<!--
SDK MUST allow users to implement and configure custom processors and decorate
built-in processors for advanced scenarios such as tagging or filtering.
-->

SDKは、ユーザーがカスタムプロセッサを実装・設定したり、タグ付けやフィルタリングなどの高度なシナリオのために組み込みのプロセッサを拡張できるようにしなければなりません(MUST)。

<!--
The following diagram shows `SpanProcessor`'s relationship to other components
in the SDK:
-->

以下の図は、`SpanProcessor`とSDKの他のコンポーネントとの関係を示しています。

```
  +-----+--------------+   +-------------------------+   +-------------------+
  |     |              |   |                         |   |                   |
  |     |              |   | Batching Span Processor |   |    SpanExporter   |
  |     |              +---> Simple Span Processor   +--->  (JaegerExporter) |
  |     |              |   |                         |   |                   |
  | SDK | Span.start() |   +-------------------------+   +-------------------+
  |     | Span.end()   |
  |     |              |
  |     |              |
  |     |              |
  |     |              |
  +-----+--------------+
```

<!--
### Interface definition
-->

### インターフェース定義

<!--
#### OnStart
-->

#### OnStart

<!--
`OnStart` is called when a span is started. This method is called synchronously
on the thread that started the span, therefore it should not block or throw
exceptions.
-->

`OnStart`はSpanが開始されるときに呼び出されます。このメソッドは、Spanを開始したスレッド上で同期的に呼び出されるので、ブロックしたり、例外を投げたりしてはいけません。

<!--
**Parameters:**
-->

**パラメータ:**

<!--
* `span` - a [read/write span object](#additional-span-interfaces) for the started span.
  It SHOULD be possible to keep a reference to this span object and updates to the span
  SHOULD be reflected in it.
  For example, this is useful for creating a SpanProcessor that periodically
  evaluates/prints information about all active span from a background thread.
* `parentContext` - the parent `Context` of the span that the SDK determined
  (the explicitly passed `Context`, the current `Context` or an empty `Context`
  if that was explicitly requested).
-->

* `span` - 開始されたSpanのための[read/write span object](#additional-span-interfaces)。このSpanオブジェクトへの参照を保持することが可能であるべき(SHOULD)で、Spanの更新はそれに反映されるべきです(SHOULD)。
  たとえば、バックグラウンドのスレッドからすべてのアクティブな span の情報を定期的に評価/印刷する SpanProcessor を作成する場合に便利です。

* `parentContext` - SDKが決定したSpanの親の `Context` (明示的に渡された `Context` か、現在の `Context` 、または明示的に要求された場合は空の `Context`)

<!--
**Returns:** `Void`
-->

**返り値:** `Void`

<!--
#### OnEnd(Span)
-->

#### OnEnd(Span)

<!--
`OnEnd` is called after a span is ended (i.e., the end timestamp is already set).
This method MUST be called synchronously within the [`Span.End()` API](api.md#end),
therefore it should not block or throw an exception.
-->

`OnEnd`は、Spanが終了した後(つまり、終了タイムスタンプがすでに設定された後)に呼び出されます。このメソッドは、[`Span.End()` API](api.md#end)内で同期的に呼び出されなければならず(MUST)、したがって、ブロックしたり、例外を投げたりしてはいけません。

<!--
**Parameters:**
-->

 **パラメータ:**

<!--
* `Span` - a [readable span object](#additional-span-interfaces) for the ended span.
  Note: Even if the passed Span may be technically writable,
  since it's already ended at this point, modifying it is not allowed.
-->

* `Span` - 終了したSpanの[readable span object](#additional-span-interfaces)。注意:渡されたSpanが技術的に書き込み可能であっても、この時点ですでに終了しているため、修正することはできません。

<!--
**Returns:** `Void`
-->

**返り値:** `Void`

<!--
#### Shutdown()
-->

#### Shutdown()

<!--
Shuts down the processor. Called when SDK is shut down. This is an opportunity
for processor to do any cleanup required.
-->

プロセッサをシャットダウンします。SDKがシャットダウンされたときに呼び出されます。これは、プロセッサが必要なクリーンアップを行う機会となります。

<!--
`Shutdown` SHOULD be called only once for each `SpanProcessor` instance. After
the call to `Shutdown`, subsequent calls to `OnStart`, `OnEnd`, or `ForceFlush`
are not allowed. SDKs SHOULD ignore these calls gracefully, if possible.
-->

`Shutdown` は各 `SpanProcessor` インスタンスに対して一度だけ呼び出されるべきです。`Shutdown` の呼び出しの後、それに続く `OnStart`, `OnEnd`, または `ForceFlush` の呼び出しは許されません。SDKは可能であれば、これらの呼び出しを尊重しつつ(gracefully)無視すべきです(SHOULD)。

<!--
`Shutdown` SHOULD provide a way to let the caller know whether it succeeded,
failed or timed out.
-->

`Shutdown` は成功したのか、失敗したのか、あるいはタイムアウトしたのかを呼び出し元に知らせる方法を提供すべきです(SHOULD)。

<!--
`Shutdown` MUST include the effects of `ForceFlush`.
-->

`Shutdown`には、`ForceFlush`の効果を含めなければなりません(MUST)。

<!--
`Shutdown` SHOULD complete or abort within some timeout. `Shutdown` can be
implemented as a blocking API or an asynchronous API which notifies the caller
via a callback or an event. OpenTelemetry client authors can decide if they want to
make the shutdown timeout configurable.
-->

`Shutdown`は、あるタイムアウト内に完了または中止すべきです(SHOULD)。`Shutdown`はブロッキングAPIとして実装することも、コールバックやイベントで呼び出し元に通知する非同期APIとして実装することもできます。OpenTelemetryクライアントの作者は、シャットダウンのタイムアウトを設定可能にするかどうかを決めることができます。

<!--
#### ForceFlush()
-->

#### ForceFlush()

<!--
This is a hint to ensure that any tasks associated with `Spans` for which the
`SpanProcessor` had already received events prior to the call to `ForceFlush` SHOULD
be completed as soon as possible, preferably before returning from this method.
-->

これは、`ForceFlush` を呼び出す前に `SpanProcessor` がすでにイベントを受信していた `Spans` に関連するすべてのタスクが可能な限り早く、できればこのメソッドから戻る前に完了すべきである(SHOULD)ことを保証するためのヒントです(XXX:ちょっと怪しい)。

<!--
In particular, if any `SpanProcessor` has any associated exporter, it SHOULD
try to call the exporter's `Export` with all spans for which this was not
already done and then invoke `ForceFlush` on it.
The [built-in SpanProcessors](#built-in-span-processors) MUST do so.
If a timeout is specified (see below), the SpanProcessor MUST prioritize honoring the timeout over
finishing all calls. It MAY skip or abort some or all Export or ForceFlush
calls it has made to achieve this goal.
-->

特に、任意の `SpanProcessor` が関連するエクスポーターを持っている場合、そのエクスポーターの `Export` を、まだ行われていない全てのSpanで呼び出し、その上で `ForceFlush` を呼び出すことを試みるべきです。[組み込みのSpanProcessors](#built-in-span-processors)がこれを行わなければなりません(MUST)。タイムアウトが指定されている場合(下記参照)、Spanプロセッサは、全ての呼び出しを終えるよりも、タイムアウトを尊重することを優先しなければなりません(MUST)。この目標を達成するために行った一部またはすべてのExportまたはForceFlushの呼び出しをスキップまたは中止しても構いません(MAY)。

<!--
`ForceFlush` SHOULD provide a way to let the caller know whether it succeeded,
failed or timed out.
-->

`ForceFlush` は成功したのか、失敗したのか、あるいはタイムアウトしたのかを呼び出し元に知らせる方法を提供すべきです(SHOULD)。

<!--
`ForceFlush` SHOULD only be called in cases where it is absolutely necessary,
such as when using some FaaS providers that may suspend the process after an
invocation, but before the `SpanProcessor` exports the completed spans.
-->

`ForceFlush` は、絶対に必要な場合にのみ呼び出すべきです(SHOULD)。例えば、一部の FaaS プロバイダーを使用している場合、呼び出し後、`SpanProcessor` が完了したSpanをエクスポートする前に、プロセスを中断する可能性があります。

<!--
`ForceFlush` SHOULD complete or abort within some timeout. `ForceFlush` can be
implemented as a blocking API or an asynchronous API which notifies the caller
via a callback or an event. OpenTelemetry client authors can decide if they want to
make the flush timeout configurable.
-->

`ForceFlush`はあるタイムアウト内に完了または中止すべきです(SHOULD)。`ForceFlush`はブロッキングAPIとして実装することも、コールバックやイベントで呼び出し元に通知する非同期APIとして実装することもできます。OpenTelemetryクライアントの作者は、フラッシュのタイムアウトを設定可能にするかどうかを決めることができます。

<!--
### Built-in span processors
-->

### 組み込み span processors

<!--
The standard OpenTelemetry SDK MUST implement both simple and batch processors,
as described below. Other common processing scenarios should be first considered
for implementation out-of-process in [OpenTelemetry Collector](../overview.md#collector)
-->

標準のOpenTelemetry SDKは、以下に説明するように、シンプルなプロセッサとバッチプロセッサの両方を実装しなければなりません(MUST)。その他の一般的な処理シナリオについては、まず[OpenTelemetry Collector](../overview.md#collector)にプロセス外(out-of-process)で実装することを検討すべきです。

<!--
#### Simple processor
-->

#### Simple processor

<!--
This is an implementation of `SpanProcessor` which passes finished spans
and passes the export-friendly span data representation to the configured
`SpanExporter`, as soon as they are finished.
-->

これは`SpanProcessor`の実装で、完成したSpanを渡し、構成された`SpanExporter`にエクスポートフレンドリーなSpanデータ表現を渡します。

<!--
**Configurable parameters:**
-->

**設定可能なパラメータ:**

<!--
* `exporter` - the exporter where the spans are pushed.
-->

* `exporter` - Spanがプッシュされるエクスポーター

<!--
#### Batching processor
-->

#### Batching processor

<!--
This is an implementation of the `SpanProcessor` which create batches of finished
spans and passes the export-friendly span data representations to the
configured `SpanExporter`.
-->

これは完成したSpanのバッチを作成する`SpanProcessor`の実装であり、構成された`SpanExporter`にエクスポートフレンドリーなSpanデータ表現を渡します。

<!--
**Configurable parameters:**
-->

**設定可能なパラメータ:**

<!--
* `exporter` - the exporter where the spans are pushed.
* `maxQueueSize` - the maximum queue size. After the size is reached spans are
  dropped. The default value is `2048`.
* `scheduledDelayMillis` - the delay interval in milliseconds between two
  consecutive exports. The default value is `5000`.
* `exportTimeoutMillis` - how long the export can run before it is cancelled.
  The default value is `30000`.
* `maxExportBatchSize` - the maximum batch size of every export. It must be
  smaller or equal to `maxQueueSize`. The default value is `512`.
-->

* `exporter` - Spanがプッシュされるエクスポーター
* `maxQueueSize` - キューの最大サイズ。このサイズに達すると、Spanは削除されます。デフォルトの値は `2048` です。
* `scheduledDelayMillis` - 2つの連続したエクスポートの間のミリ秒単位の遅延間隔。デフォルトの値は `5000` です。
* `exportTimeoutMillis` - エクスポートがキャンセルされるまでの実行可能時間。デフォルトの値は `30000` です。
* `maxExportBatchSize` - すべてのエクスポートの最大バッチサイズ。`maxQueueSize`と同じか、それ以下でなければなりません。デフォルトの値は `512` です。


<!--
## Span Exporter
-->

## Span Exporter

<!--
`Span Exporter` defines the interface that protocol-specific exporters must
implement so that they can be plugged into OpenTelemetry SDK and support sending
of telemetry data.
-->

`Span Exporter`では、OpenTelemetry SDKにプラグインしてテレメトリデータの送信をサポートするために、プロトコル固有のエクスポーターが実装しなければならないインターフェースを定義しています。

<!--
The goal of the interface is to minimize burden of implementation for
protocol-dependent telemetry exporters. The protocol exporter is expected to be
primarily a simple telemetry data encoder and transmitter.
-->

このインターフェースの目的は、プロトコルに依存するテレメトリ・エクスポーターの実装の負担を最小限にすることです。プロトコル・エクスポーターは、主に単純なテレメトリ・データのエンコーダーとトランスミッターであることが予想されます。

<!--
### Interface Definition
-->

### インターフェース定義

<!--
The exporter must support two functions: **Export** and **Shutdown**. In
strongly typed languages typically there will be 2 separate `Exporter`
interfaces, one that accepts spans (SpanExporter) and one that accepts metrics
(MetricsExporter).
-->

エクスポーターは2つの関数をサポートする必要があります。それは、**Export**と**Shutdown**です。強く型付けされた言語では、通常、2つの独立した`Exporter`インターフェースがあり、1つはSpanを受け入れるもの(SpanExporter)、もう1つはMetricを受け入れるもの(MetricsExporter)です。

<!--
#### `Export(batch)`
-->

#### `Export(batch)`

<!--
Exports a batch of [readable spans](#additional-span-interfaces).
Protocol exporters that will implement this
function are typically expected to serialize and transmit the data to the
destination.
-->

[読み込み可能なSpan](#additional-span-interfaces)のバッチをエクスポートします。この機能を実装するプロトコル・エクスポーターは、通常、データをシリアル化して宛先に送信することが期待されます。

<!--
Export() will never be called concurrently for the same exporter instance.
Export() can be called again only after the current call returns.
-->

Export()は、同じエクスポーターのインスタンスに対して同時に呼び出されることはありません。Export() が再び呼び出されるのは、現在の呼び出しが終了した後です。

<!--
Export() MUST NOT block indefinitely, there MUST be a reasonable upper limit
after which the call must time out with an error result (`Failure`).
-->

Export()は無限にブロックしてはいけません(MUST NOT)。合理的な上限があり、それを超えるとエラー結果(`Failure`)で呼び出しがタイムアウトしなければなりません(MUST)。

<!--
Any retry logic that is required by the exporter is the responsibility
of the exporter. The default SDK SHOULD NOT implement retry logic, as
the required logic is likely to depend heavily on the specific protocol
and backend the spans are being sent to.
-->

エクスポーターが必要とするリトライロジックは、エクスポーターの責任となります。デフォルトのSDKでは、リトライロジックを実装すべきではありません(SHOULD NOT)。必要なロジックは、Spanの送信先であるプロトコルやバックエンドに大きく依存する可能性があるからです。

<!--
**Parameters:**
-->

 **パラメータ:**

<!--
batch - a batch of [readable spans](#additional-span-interfaces). The exact data type of the batch is language
specific, typically it is some kind of list,
e.g. for spans in Java it will be typically `Collection<SpanData>`.
-->

batch - [読み込み可能なSpan](#additional-span-interfaces)のバッチです。バッチの正確なデータ型は言語によって異なりますが、通常は何らかのリストです。例えばJavaのSpanの場合は、通常は`Collection<SpanData>`となります。

<!--
**Returns:** ExportResult:
-->

**返り値:** ExportResult:

<!--
ExportResult is one of:
-->

ExportResult は次のうちのどちらかです。

<!--
* `Success` - The batch has been successfully exported.
  For protocol exporters this typically means that the data is sent over
  the wire and delivered to the destination server.
* `Failure` - exporting failed. The batch must be dropped. For example, this
  can happen when the batch contains bad data and cannot be serialized.
-->

* `Success` - バッチのエクスポートに成功しました。プロトコルエクスポーターの場合、これは通常、データがWireで送信され、宛先サーバーに配信されたことを意味します。
* `Failure` - エクスポートに失敗しました。バッチをドロップする必要があります。例えば、バッチに不良データが含まれていて、シリアライズできない場合に発生します。

<!--
Note: this result may be returned via an async mechanism or a callback, if that
is idiomatic for the language implementation.
-->

注:この結果は、言語の実装が適切であれば、非同期メカニズムやコールバックを介して返されることがあります。

<!--
#### `Shutdown()`
-->

#### `Shutdown()`

<!--
Shuts down the exporter. Called when SDK is shut down. This is an opportunity
for exporter to do any cleanup required.
-->

エクスポーターをシャットダウンします。SDKがシャットダウンされたときに呼び出されます。これは、エクスポーターが必要なクリーンアップを行う機会となります。

<!--
`Shutdown` should be called only once for each `Exporter` instance. After the
call to `Shutdown` subsequent calls to `Export` are not allowed and should
return a `Failure` result.
-->

`Shutdown`は各`Exporter`インスタンスに対して一度だけ呼び出す必要があります。`Shutdown` を呼び出した後に `Export` を呼び出すことはできず、結果として `Failure` を返す必要があります。

<!--
`Shutdown` should not block indefinitely (e.g. if it attempts to flush the data
and the destination is unavailable). OpenTelemetry client authors can decide if they
want to make the shutdown timeout configurable.
-->

`Shutdown` は無期限にブロックしてはいけません (例えば、データをフラッシュしようとして、宛先が利用できない場合など)。OpenTelemetryクライアントの作者は、シャットダウンのタイムアウトを設定可能にするかどうかを決めることができます。

<!--
#### `ForceFlush()`
-->

#### `ForceFlush()`

<!--
This is a hint to ensure that the export of any `Spans` the exporter has received prior to the
call to `ForceFlush` SHOULD be completed as soon as possible, preferably before
returning from this method.
-->

これは、エクスポーターが `ForceFlush` を呼び出す前に受け取った `Spans` のエクスポートをできるだけ早く、できればこのメソッドから戻る前に完了すべきである(SHOULD)ことを示すヒントです。

<!--
`ForceFlush` SHOULD provide a way to let the caller know whether it succeeded,
failed or timed out.
-->

`ForceFlush` は成功したのか、失敗したのか、あるいはタイムアウトしたのかを呼び出し側に知らせる方法を提供すべきです(SHOULD)。

<!--
`ForceFlush` SHOULD only be called in cases where it is absolutely necessary,
such as when using some FaaS providers that may suspend the process after an
invocation, but before the exporter exports the completed spans.
-->

`ForceFlush` は、絶対に必要な場合にのみ呼び出すべきです(SHOULD)。例えば、一部の FaaS プロバイダーを使用している場合には、呼び出した後、エクスポーターが完了したSpanをエクスポートする前に、プロセスを中断することがあります。

<!--
`ForceFlush` SHOULD complete or abort within some timeout. `ForceFlush` can be
implemented as a blocking API or an asynchronous API which notifies the caller
via a callback or an event. OpenTelemetry client authors can decide if they want to
make the flush timeout configurable.
-->

`ForceFlush`はあるタイムアウト内に完了または中止すべきです(SHOULD)。`ForceFlush`はブロッキングAPIとして実装することも、コールバックやイベントで呼び出し元に通知する非同期APIとして実装することもできます。OpenTelemetryクライアントの作者は、フラッシュのタイムアウトを設定可能にするかどうかを決めることができます。

<!--
### Further Language Specialization
-->

### さらなる言語の仕様

<!--
Based on the generic interface definition laid out above library authors must
define the exact interface for the particular language.
-->

上記の汎用的なインターフェースの定義に基づいて、ライブラリの作成者は、特定の言語のための正確なインターフェースを定義する必要があります。

<!--
Authors are encouraged to use efficient data structures on the interface
boundary that are well suited for fast serialization to wire formats by protocol
exporters and minimize the pressure on memory managers. The latter typically
requires understanding of how to optimize the rapidly-generated, short-lived
telemetry data structures to make life easier for the memory manager of the
specific language. General recommendation is to minimize the number of
allocations and use allocation arenas where possible, thus avoiding explosion of
allocation/deallocation/collection operations in the presence of high rate of
telemetry data generation.
-->

ライブラリ作成者は、プロトコル・エクスポータによるワイヤ・フォーマットへの高速シリアライゼーションに適しており、メモリ・マネージャへの負担を最小限に抑えることができる、効率的なデータ構造をインターフェイス境界に使用することが推奨されます。後者のためには、特定の言語のメモリマネージャの負担を軽減するために、急速に生成された短命のテレメトリデータ構造をどのように最適化するかを理解する必要があります。一般的な推奨事項は、割り当て数を最小限に抑え、可能な限り割り当て領域を使用することです。これにより、高頻度のテレメトリデータが生成された場合に、割り当て/割り当て解除/収集の操作が爆発的に増加するのを防ぐことができます。

<!--
#### Examples
-->

#### 例

<!--
These are examples on what the `Exporter` interface can look like in specific
languages. Examples are for illustration purposes only. OpenTelemetry client authors
are free to deviate from these provided that their design remain true to the
spirit of `Exporter` concept.
-->

これらは、特定の言語で `Exporter` インターフェースがどのように見えるかの例です。例は説明のためだけのものです。OpenTelemetryクライアントの作成者は、`Exporter`のコンセプトの精神に忠実なデザインであれば、これらから自由に変更することができます。

<!--
##### Go SpanExporter Interface
-->

##### Go SpanExporter Interface

<!--
```go
type SpanExporter interface {
    Export(batch []ExportableSpan) ExportResult
    Shutdown()
}

type ExportResult struct {
    Code         ExportResultCode
    WrappedError error
}

type ExportResultCode int

const (
    Success ExportResultCode = iota
    Failure
)
```

```go
type SpanExporter interface {
    Export(batch []ExportableSpan) ExportResult
    Shutdown()
}

type ExportResult struct {
    Code         ExportResultCode
    WrappedError error
}

type ExportResultCode int

const (
    Success ExportResultCode = iota
    Failure
)
```

<!--
##### Java SpanExporter Interface
-->

##### Java SpanExporter Interface

<!--
```java
public interface SpanExporter {
 public enum ResultCode {
   Success, Failure
 }

 ResultCode export(Collection<ExportableSpan> batch);
 void shutdown();
}
```
-->

```java
public interface SpanExporter {
 public enum ResultCode {
   Success, Failure
 }

 ResultCode export(Collection<ExportableSpan> batch);
 void shutdown();
}
```

<!--
[trace-flags]: https://www.w3.org/TR/trace-context/#trace-flags
[otep-83]: https://github.com/open-telemetry/oteps/blob/main/text/0083-component.md
-->

[trace-flags]: https://www.w3.org/TR/trace-context/#trace-flags
[otep-83]: https://github.com/open-telemetry/oteps/blob/main/text/0083-component.md

