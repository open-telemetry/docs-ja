<!--
# Tracing API
-->

# Tracing API

**Status**: [Stable, Feature-freeze](../document-status.md)

<details>
<summary>
目次
</summary>

<!--
* [Data types](#data-types)
  * [Time](#time)
    * [Timestamp](#timestamp)
    * [Duration](#duration)
* [TracerProvider](#tracerprovider)
  * [TracerProvider operations](#tracerprovider-operations)
* [Context Interaction](#context-interaction)
* [Tracer](#tracer)
  * [Tracer operations](#tracer-operations)
* [SpanContext](#spancontext)
  * [Retrieving the TraceId and SpanId](#retrieving-the-traceid-and-spanid)
  * [IsValid](#isvalid)
  * [IsRemote](#isremote)
* [Span](#span)
  * [Span creation](#span-creation)
    * [Determining the Parent Span from a Context](#determining-the-parent-span-from-a-context)
    * [Specifying Links](#specifying-links)
  * [Span operations](#span-operations)
    * [Get Context](#get-context)
    * [IsRecording](#isrecording)
    * [Set Attributes](#set-attributes)
    * [Add Events](#add-events)
    * [Set Status](#set-status)
    * [UpdateName](#updatename)
    * [End](#end)
    * [Record Exception](#record-exception)
  * [Span lifetime](#span-lifetime)
  * [Wrapping a SpanContext in a Span](#wrapping-a-spancontext-in-a-span)
* [SpanKind](#spankind)
* [Concurrency](#concurrency)
* [Included Propagators](#included-propagators)
-->

* [Data型](#Data型)
  * [Time](#time)
    * [Timestamp](#timestamp)
    * [Duration](#duration)
* [TracerProvider](#tracerprovider)
  * [TracerProviderの操作](#tracerproviderの操作)
* [Context Interaction](#context-interaction)
* [Tracer](#tracer)
  * [Tracerの操作](#tracerの操作)
* [SpanContext](#spancontext)
  * [TraceIdとSpanIdの取得](#traceIdとspanIdの取得)
  * [IsValid](#isvalid)
  * [IsRemote](#isremote)
* [Span](#span)
  * [spanの作成](#spanの作成)
    * [Contextから親のSpanを決定する](#contextから親のSpanを決定する)
    * [リンクの指定](#リンクの指定)
  * [Span操作](#span操作)
    * [Contextの取得](#Contextの取得)
    * [IsRecording](#isrecording)
    * [属性の設定](#属性の設定)
    * [イベントの追加](#イベントの追加)
    * [ステータス設定](#ステータス設定)
    * [UpdateName](#updatename)
    * [End](#end)
    * [例外の記録](#例外の記録)
  * [Span lifetime](#span-lifetime)
  * [SpanContextをSpanでラップする](#SpanContextをSpanでラップする)
* [SpanKind](#spankind)
* [並行性](#並行性)
* [含まれるPropagator](#含まれるPropagator)

</details>

<!--
The Tracing API consist of these main classes:
-->

Tracing APIは、以下の主要なクラスで構成されています:

<!--
- [`TracerProvider`](#tracerprovider) is the entry point of the API.
  It provides access to `Tracer`s.
- [`Tracer`](#tracer) is the class responsible for creating `Span`s.
- [`Span`](#span) is the API to trace an operation.
-->

- [`TracerProvider`](#tracerprovider)は、APIのエントリーポイントです。`Tracer`へのアクセスを提供します。
- [`Tracer`](#tracer)は `Span` を作成するためのクラスです。
- [`Span`](#span)は操作をトレースするためのAPIです。

<!--
## Data types
-->

## Data型

<!--
While languages and platforms have different ways of representing data,
this section defines some generic requirements for this API.
-->

言語やプラットフォームによってデータの表現方法は異なりますが、ここでは本APIの一般的な要件を定義します。

<!--
### Time
-->

### Time

<!--
OpenTelemetry can operate on time values up to nanosecond (ns) precision.
The representation of those values is language specific.
-->

OpenTelemetryは、ナノ秒(ns)の精度までの時間値を扱うことができます。これらの値の表現方法は言語によって異なります。

<!--
#### Timestamp
-->

#### Timestamp

<!--
A timestamp is the time elapsed since the Unix epoch.
-->

タイムスタンプとは、Unixのエポックからの経過時間のことです。

<!--
* The minimal precision is milliseconds.
* The maximal precision is nanoseconds.
-->

* 最小精度はミリ秒です。
* 最大精度はナノ秒です。

<!--
#### Duration
-->

#### Duration

<!--
A duration is the elapsed time between two events.
-->

Durationとは、2つの事象の間の経過時間のことです。

<!--
* The minimal precision is milliseconds.
* The maximal precision is nanoseconds.
-->

* 最小精度はミリ秒です。
* 最大精度はナノ秒です。

<!--
## TracerProvider
-->

## TracerProvider

<!--
`Tracer`s can be accessed with a `TracerProvider`.
-->

`Tracer`にアクセスするには、`TracerProvider`を使用します。

<!--
In implementations of the API, the `TracerProvider` is expected to be the
stateful object that holds any configuration.
-->

APIの実装では、`TracerProvider`は、あらゆる設定を保持するステートフルなオブジェクトであることが期待されます。

<!--
Normally, the `TracerProvider` is expected to be accessed from a central place.
Thus, the API SHOULD provide a way to set/register and access
a global default `TracerProvider`.
-->

通常、`TracerProvider`は中央の場所(XXX: どんな場所からも？)からアクセスされることが期待されます。そのため、APIはグローバルなデフォルトの`TracerProvider`を設定/登録し、アクセスする方法を提供すべきです(SHOULD)。

<!--
Notwithstanding any global `TracerProvider`, some applications may want to or
have to use multiple `TracerProvider` instances,
e.g. to have different configuration (like `SpanProcessor`s) for each
(and consequently for the `Tracer`s obtained from them),
or because its easier with dependency injection frameworks.
Thus, implementations of `TracerProvider` SHOULD allow creating an arbitrary
number of `TracerProvider` instances.
-->

グローバルな`TracerProvider`にも関わらず、アプリケーションによっては、複数の`TracerProvider`インスタンスを使用したい、または使用しなければならない場合があります。例えば、それぞれに異なる構成(`SpanProcessor`のような)を持つために、あるいは、依存性注入フレームワークでより簡単になるためです。したがって、`TracerProvider` の実装は、任意の数の`TracerProvider` インスタンスの作成を可能にするべきです(SHOULD)。

<!--
### TracerProvider operations
-->

### TracerProviderの操作

<!--
The `TracerProvider` MUST provide the following functions:
-->

`TracerProvider`は、以下の関数を提供しなければなりません:

<!--
- Get a `Tracer`
-->

- `Tracer`の取得

<!--
#### Get a Tracer
-->

#### Tracerの取得

<!--
This API MUST accept the following parameters:
-->

このAPIは、以下のパラメータを受け付けなければなりません(MUST)。

<!--
- `name` (required): This name must identify the [instrumentation library](../overview.md#instrumentation-libraries)
  (e.g. `io.opentelemetry.contrib.mongodb`).
  If an application or library has built-in OpenTelemetry instrumentation, both
  [Instrumented library](../glossary.md#instrumented-library) and
  [Instrumentation library](../glossary.md#instrumentation-library) may refer to the same library.
  In that scenario, the `name` denotes a module name or component name within that library
  or application.
  In case an invalid name (null or empty string) is specified, a working
  Tracer implementation MUST be returned as a fallback rather than returning
  null or throwing an exception, its `name` property SHOULD be set to an **empty** string,
  and a message reporting that the specified value is invalid SHOULD be logged.
  A library, implementing the OpenTelemetry API *may* also ignore this name and
  return a default instance for all calls, if it does not support "named"
  functionality (e.g. an implementation which is not even observability-related).
  A TracerProvider could also return a no-op Tracer here if application owners configure
  the SDK to suppress telemetry produced by this library.
- `version` (optional): Specifies the version of the instrumentation library (e.g. `1.0.0`).
-->

* `name` (必須)。この名前は、[計装ライブラリ](../overview.md#instrumentation-libraries)を特定する必要があります(例: `io.opentelemetry.contrib.mongodb`)。アプリケーションやライブラリにOpenTelemetryの計装機能が組み込まれている場合、[計装されるライブラリ](../glossary.md#instrumented-library)と[計装するライブラリ](../glossary.md#instrumentation-library)の両方が同じライブラリを参照してい
ることがあります。その場合、`name`は、そのライブラリやアプリケーション内のモジュール名やコンポーネント名を表します。無効な名前(NULLまたは空の文字列)が指定された場合、NULLを返したり例外をスロー
するのではなく、フォールバックとして動作するMeterの実装が返されなければならず(MUST)、その`name`プロパティは**空**文字列にすべき(SHOULD)で、指定された値が無効であることを報告するメッセージがログに記
録されるべきです(SHOULD)。OpenTelemetry APIを実装しているライブラリは、「名前付き」機能をサポートしていない場合(例:観測性に関係のない実装など)、この名前を無視して、すべての呼び出しに対してデフ
ォルトのインスタンスを返すことも*できます*。アプリケーションの所有者が、このライブラリで生成されるテレメトリを抑制するようにSDKを構成している場合、MeterProviderはここでno-op Meter(何もしないMeter)を返すこともできます。
* `バージョン`(任意)。計装ライブラリのバージョンを指定します(例:`1.0.0`)

<!--
It is unspecified whether or under which conditions the same or different
`Tracer` instances are returned from this functions.
-->

この関数から、同じまたは異なる `Tracer` インスタンスが返されるかどうか、またはどのような条件で返されるかは、定義されていません。

<!--
Implementations MUST NOT require users to repeatedly obtain a `Tracer` again
with the same name+version to pick up configuration changes.
This can be achieved either by allowing to work with an outdated configuration or
by ensuring that new configuration applies also to previously returned `Tracer`s.
-->

実装では、コンフィグレーションの変更を拾うために、同じ名前+バージョンの`Tracer`を繰り返し取得することをユーザーに要求してはなりません(MUST NOT)。これは、古いコンフィグレーションでの作業を許可するか、新しいコンフィグレーションを以前に返された `Tracer` にも適用できるるようにすることで実現できます。

<!--
Note: This could, for example, be implemented by storing any mutable
configuration in the `TracerProvider` and having `Tracer` implementation objects
have a reference to the `TracerProvider` from which they were obtained.
If configuration must be stored per-tracer (such as disabling a certain tracer),
the tracer could, for example, do a look-up with its name+version in a map in
the `TracerProvider`, or the `TracerProvider` could maintain a registry of all
returned `Tracer`s and actively update their configuration if it changes.
-->

注:これは例えば、`TracerProvider`に変更可能なコンフィギュレーションを保存し、`Tracer`の実装オブジェクトに、それらが取得された`TracerProvider`への参照を持たせることで実装できます。Tracerごとにコンフィギュレーションを保存しなければならない場合(特定のTracerを無効にするなど)、Tracerは、例えば、`TracerProvider`内のマップでその名前+バージョンで検索できます。あるいは、`TracerProvider`は、返されたすべての`Tracer`のレジストリを維持し、コンフィギュレーションが変更された場合、積極的に更新することができます。

<!--
## Context Interaction
-->

## Contextのインタラクション

<!--
This section defines all operations within the Tracing API that interact with the
[`Context`](../context/context.md).
-->

この章では、Tracing APIの中で、[`Context`](../context/context.md)とやり取りするすべての操作を定義します。

<!--
The API MUST provide the following functionality to interact with a `Context`
instance:
-->

APIは、`Context`インスタンスと対話するために、以下の機能を提供しなければなりません(MUST)。

<!--
- Extract the `Span` from a `Context` instance
- Insert the `Span` to a `Context` instance
-->

- `Context` インスタンスから `Span` を抽出する。
- `Span`を`Context`インスタンスに挿入する。

<!--
The functionality listed above is necessary because API users SHOULD NOT have
access to the [Context Key](../context/context.md#create-a-key) used by the Tracing API implementation.
-->

APIユーザーはTracing APIの実装で使用される[Context Key](../context/context.md#create-a-key)にアクセスすべきではない(SHOULD NOT)ため、上記の機能が必要となります。

<!--
If the language has support for implicitly propagated `Context` (see
[here](../context/context.md#optional-global-operations)), the API SHOULD also provide
the following functionality:
-->

言語が暗黙的に伝搬する`Context`をサポートしている場合([こちら](../context/context.md#optional-global-operations)を参照)、APIは以下の機能も提供すべきです(SHOULD)。

<!--
- Get the currently active span from the implicit context. This is equivalent to getting the implicit context, then extracting the `Span` from the context.
- Set the currently active span to the implicit context. This is equivalent to getting the implicit context, then inserting the `Span` to the context.
-->

- 暗黙のコンテキストから現在アクティブなSpanを取得します。これは、暗黙のコンテキストを取得して、そのコンテキストから `Span` を抽出することと同じです。
- 現在のアクティブなSpanを暗黙のコンテキストに設定します。これは、暗黙のコンテキストを取得して、そのコンテキストに `Span` を挿入することと同じです。

<!--
All the above functionalities operate solely on the context API, and they MAY be
exposed as either static methods on the trace module, or as static methods on a class
inside the trace module. This functionality SHOULD be fully implemented in the API when possible.
-->

上記の機能はすべてcontext API のみで動作し、Trace モジュールのスタティックメソッド、またはTraceモジュール内のクラスのスタティックメソッドとして公開しても構いません(MAY)。この機能は、可能であればAPIに完全に実装されるべきです(SHOULD)。

<!--
## Tracer
-->

## Tracer

<!--
The tracer is responsible for creating `Span`s.
-->

Tracerは「Span」の作成を担当しています。

<!--
Note that `Tracer`s should usually *not* be responsible for configuration.
This should be the responsibility of the `TracerProvider` instead.
-->

注意: `Tracer` は通常、設定を行うべきでは*ありません*。これは、代わりに `TracerProvider` が担うべきものです。

<!--
### Tracer operations
-->

### Tracerの操作

<!--
The `Tracer` MUST provide functions to:
-->

`Tracer`は以下の関数を提供しなければなりません:

<!--
- [Create a new `Span`](#span-creation) (see the section on `Span`)
-->

- [新しい `Span` を作成する](#spanの作成) (`Span` のセクションを参照)

<!--
## SpanContext
-->

## SpanContext

<!--
A `SpanContext` represents the portion of a `Span` which must be serialized and
propagated along side of a distributed context. `SpanContext`s are immutable.
-->

`SpanContext`は`Span`の一部を表し、分散したコンテキストに沿ってシリアライズされ、伝播される必要があります。`SpanContext` は不変です。

<!--
The OpenTelemetry `SpanContext` representation conforms to the [W3C TraceContext
specification](https://www.w3.org/TR/trace-context/). It contains two
identifiers - a `TraceId` and a `SpanId` - along with a set of common
`TraceFlags` and system-specific `TraceState` values.
-->

OpenTelemetryの`SpanContext`表現は[W3C TraceContext 仕様](https://www.w3.org/TR/trace-context/)に準拠しています。`TraceId`と`SpanId`の2つの識別子と、共通の`TraceFlags`とシステム固有の`TraceState`の値を含んでいます。

<!--
`TraceId` A valid trace identifier is a 16-byte array with at least one
non-zero byte.
-->

`TraceId` : 有効なTrace 識別子は、少なくとも1つの非ゼロバイトを持つ16バイトの配列です。

<!--
`SpanId` A valid span identifier is an 8-byte array with at least one non-zero
byte.
-->

`SpanId`: 有効なSpan識別子は、少なくとも1つの非ゼロバイトを持つ8バイトの配列です。

<!--
`TraceFlags` contain details about the trace. Unlike TraceState values,
TraceFlags are present in all traces. The current version of the specification
only supports a single flag called [sampled](https://www.w3.org/TR/trace-context/#sampled-flag).
-->

`TraceFlags` はトレースの詳細を含みます。TraceState の値とは異なり、TraceFlags はすべてのトレースに存在します。現在のバージョンの仕様では、[sampled](https://www.w3.org/TR/trace-context/#sampled-flag)という単一のフラグのみをサポートしています。

<!--
`TraceState` carries vendor-specific trace identification data, represented as a list
of key-value pairs. TraceState allows multiple tracing
systems to participate in the same trace. It is fully described in the [W3C Trace Context
specification](https://www.w3.org/TR/trace-context/#tracestate-header).
-->

`TraceState`は、ベンダー固有のトレース識別データを持ち、キーと値のペアのリストとして表されます。TraceState は、複数のトレースシステムが同じトレースに参加することを可能にします。これは、[W3C Trace Context 仕様](https://www.w3.org/TR/trace-context/#tracestate-header)で完全に説明されています。

<!--
The API MUST implement methods to create a `SpanContext`. These methods SHOULD be the only way to
create a `SpanContext`. This functionality MUST be fully implemented in the API, and SHOULD NOT be
overridable.
-->

APIは、`SpanContext`を作成するためのメソッドを実装しなければなりません(MUST)。これらのメソッドは、`SpanContext`を作成する唯一の方法であるべきです(SHOULD)。この機能はAPIに完全に実装されなければならず、オーバーライドできないようにすべきです(SHOULD NOT)。

<!--
### Retrieving the TraceId and SpanId
-->

### TraceIdとSpanIdの取得

<!--
The API MUST allow retrieving the `TraceId` and `SpanId` in the following forms:
-->

APIは、以下の形式で`TraceId`と`SpanId`を取得できなければなりません(MUST):

<!--
* Hex - returns the lowercase [hex encoded](https://tools.ietf.org/html/rfc4648#section-8)
`TraceId` (result MUST be a 32-hex-character lowercase string) or `SpanId`
(result MUST be a 16-hex-character lowercase string).
* Binary - returns the binary representation of the `TraceId` (result MUST be a
16-byte array) or `SpanId` (result MUST be an 8-byte array).
-->

* Hex - 小文字の [hex encoded](https://tools.ietf.org/html/rfc4648#section-8) `TraceId` (結果は 32-hex-character lowercase string でなければなりません(MUST))または `SpanId` (結果は 16-hex-character lowercase string でなければなりません(MUST))を返します。
* Binary - `TraceId`(結果は16バイトの配列でなければならない(MUST))または`SpanId`(結果は8バイトの配列でなければならない(MUST))のバイナリ表現を返します。

<!--
The API SHOULD NOT expose details about how they are internally stored.
-->

APIは、それらが内部的にどのように保存されているかについての詳細を公開すべきではありません(SHOULD NOT)。

<!--
### IsValid
-->

### IsValid

<!--
An API called `IsValid`, that returns a boolean value, which is `true` if the SpanContext has a
non-zero TraceID and a non-zero SpanID, MUST be provided.
-->

APIは、`IsValid`と呼ばれるAPIを提供しなければなりません(MUST)。この関数はSpanContextがゼロ以外のTrace IDとゼロ以外のSpan IDを持っている場合に`true`を返すブール値を返します。

<!--
### IsRemote
-->

### IsRemote

<!--
An API called `IsRemote`, that returns a boolean value, which is `true` if the SpanContext was
propagated from a remote parent, MUST be provided.
When extracting a `SpanContext` through the [Propagators API](../context/api-propagators.md#propagators-api),
`IsRemote` MUST return true, whereas for the SpanContext of any child spans it MUST return false.
-->

APIは、`IsRemote`と呼ばれる関数を提供しなければなりません(MUST)。このAPIは、Spanコンテキストがリモートの親から伝搬された場合に`true`を返すブール値を返します。[Propagators API](../context/api-propagators.md#propagators-api) を通じて `SpanContext` を抽出する際には、`IsRemote` は true を返さなければなりません(MUST)。一方、子Spanの SpanContext については false を返さなければなりません(MUST)。

<!--
### TraceState
-->

### TraceState

<!--
`TraceState` is a part of [`SpanContext`](./api.md#spancontext), represented by an immutable list of string key/value pairs and
formally defined by the [W3C Trace Context specification](https://www.w3.org/TR/trace-context/#tracestate-header).
Tracing API MUST provide at least the following operations on `TraceState`:
-->

`TraceState`は、[`SpanContext`](./api.md#spancontext)の一部で、文字列のキーと値のペアの不変のリストで表され、[W3C Trace Context 仕様](https://www.w3.org/TR/trace-context/#tracestate-header)で正式に定義されています。Tracing APIは、`TraceState`に対して、少なくとも以下の操作を提供しなければなりません。

<!--
* Get value for a given key
* Add a new key/value pair
* Update an existing value for a given key
* Delete a key/value pair
-->

* 与えられたキーに対する値の取得 
* 新しいキー/値・ペアの追加 
* 指定されたキーに対する既存の値の更新 
* キー/値のペアを削除

<!--
These operations MUST follow the rules described in the [W3C Trace Context specification](https://www.w3.org/TR/trace-context/#mutating-the-tracestate-field).
All mutating operations MUST return a new `TraceState` with the modifications applied.
`TraceState` MUST at all times be valid according to rules specified in [W3C Trace Context specification](https://www.w3.org/TR/trace-context/#tracestate-header-field-values).
Every mutating operations MUST validate input parameters.
If invalid value is passed the operation MUST NOT return `TraceState` containing invalid data
and MUST follow the [general error handling guidelines](../error-handling.md).
-->

これらの操作は、[W3C Trace Context 仕様](https://www.w3.org/TR/trace-context/#mutating-the-tracestate-field)に記載されているルールに従わなければなりません(MUST)。すべての変更操作は、変更が適用された新しい `TraceState` を返さなければなりません (MUST)。`TraceState`は、[W3C Trace Context 仕様](https://www.w3.org/TR/trace-context/#tracestate-header-field-values)で指定されたルールに従って、常に有効でなければなりません(MUST)。すべての変更操作は、入力パラメータを検証しなければなりません(MUST)。無効な値が渡された場合、操作は無効なデータを含む`TraceState`を返してはならず(MUST NOT)、[一般的なエラー処理のガイドライン](../error-handling.md)に従わなければなりません(MUST)。

<!--
Please note, since `SpanContext` is immutable, it is not possible to update `SpanContext` with a new `TraceState`.
Such changes then make sense only right before
[`SpanContext` propagation](../context/api-propagators.md)
or [telemetry data exporting](sdk.md#span-exporter).
In both cases, `Propagator`s and `SpanExporter`s may create a modified `TraceState` copy before serializing it to the wire.
-->

なお、`SpanContext` は不変なので、新しい `TraceState` で `SpanContext` を更新することはできないことに注意してください。このような変更は、[SpanContext` propagation](../context/api-propagators.md)や[Telemetry data exporting](sdk.md#span-exporter)の直前にのみ意味を持ちます。どちらの場合も、`Propagator`と`SpanExporter`は、ワイヤにシリアライズする前に、修正された`TraceState`のコピーを作成することができます。

<!--
## Span
-->

## Span

<!--
A `Span` represents a single operation within a trace. Spans can be nested to
form a trace tree. Each trace contains a root span, which typically describes
the entire operation and, optionally, one or more sub-spans for its sub-operations.
-->

`Span`は、Trace内の単一の操作を表します。SpanはネストしてTraceツリーを形成することができます。各TraceにはルートSpanが含まれ、通常はオペレーション全体を記述し、オプションでサブオペレーションのための1つまたは複数のサブSpanを記述します。

<!--
<a name="span-data-members"></a>
`Span`s encapsulate:
-->

<a name="span-data-members"></a> `Span` は以下の情報を持ちます:

<!--
- The span name
- An immutable [`SpanContext`](#spancontext) that uniquely identifies the
  `Span`
- A parent span in the form of a [`Span`](#span), [`SpanContext`](#spancontext),
  or null
- A [`SpanKind`](#spankind)
- A start timestamp
- An end timestamp
- [`Attributes`](../common/common.md#attributes)
- A list of [`Link`s](#specifying-links) to other `Span`s
- A list of timestamped [`Event`s](#add-events)
- A [`Status`](#set-status).
-->


- Span名
- 不変の[SpanContext`](#spancontext)で、`Span`を一意に識別します。
- 親となるSpanは、[Span`](#span)、[SpanContext`](#spancontext)、またはNULLのいずれかの形式で指定します。
- [`SpanKind`](#spankind)
- 開始タイムスタンプ
- 終了時のタイムスタンプ
- [`属性`](../common/common.md#属性)
- 他の `Span` への [Link`s](#specifying-link) のリスト
- タイムスタンプ付きの[`イベント`](#add-events)のリスト
- [`ステータス`](#set-status)のリスト

<!--
The _span name_ concisely identifies the work represented by the Span,
for example, an RPC method name, a function name,
or the name of a subtask or stage within a larger computation.
The span name SHOULD be the most general string that identifies a
(statistically) interesting _class of Spans_,
rather than individual Span instances while still being human-readable.
That is, "get_user" is a reasonable name, while "get_user/314159",
where "314159" is a user ID, is not a good name due to its high cardinality.
Generality SHOULD be prioritized over human-readability.
-->

_Span名_は、例えば、RPCメソッド名、関数名、より大きな計算の中のサブタスクやステージの名前など、Spanが表す作業を簡潔に識別します。Span名は、個々のSpanのインスタンスではなく、(統計的に)興味深い_Spanの種類_を識別する最も一般的な文字列であるべきであり、かつ人間が読めるものでなければなりません(SHOULD)。つまり、"get_user"は妥当な名前ですが、"get_user/314159 "は "314159"がユーザーIDであるため、カーディナリティが高くなり、良い名前ではありません。汎用性は人間が読みやすいことよりも優先されるべきです(SHOULD)。

<!--
For example, here are potential span names for an endpoint that gets a
hypothetical account information:
-->

例えば、仮想的なアカウント情報を取得するエンドポイントのSpan名の候補を以下に示します。

<!--
| Span Name         | Guidance     |
| ----------------- | ------------ |
| `get`             | Too general  |
| `get_account/42`  | Too specific |
| `get_account`     | Good, and account_id=42 would make a nice Span attribute |
| `get_account/{accountId}` | Also good (using the "HTTP route") |
-->

| Span 名           | 解説         |
| ----------------- | ------------ |
| `get`             | 一般的すぎ   |
| `get_account/42`  | 特定すぎ     |
| `get_account`     | 良い。それにaccount_id=42はいいSpan属性になります|
| `get_account/{accountId}` | これも良い("HTTP route"を使って) |

<!--
The `Span`'s start and end timestamps reflect the elapsed real time of the
operation.
-->

`Span`の開始と終了のタイムスタンプは、操作の経過した実時間を反映しています。

<!--
For example, if a span represents a request-response cycle (e.g. HTTP or an RPC),
the span should have a start time that corresponds to the start time of the
first sub-operation, and an end time of when the final sub-operation is complete.
This includes:
-->

例えば、HTTPやRPCなどのリクエスト・レスポンスサイクルを表す場合、最初のサブオペレーションの開始時刻に対応する開始時刻と、最後のサブオペレーションが完了したときの終了時刻を持つ必要があります。これには以下の情報が含まれます。

<!--
- receiving the data from the request
- parsing of the data (e.g. from a binary or json format)
- any middleware or additional processing logic
- business logic
- construction of the response
- sending of the response
-->

- リクエストから受け取ったデータ
- パースしたデータ(例:バイナリまたはjsonフォーマットから)
- ミドルウェアや追加の処理ロジック
- ビジネスロジック
- レスポンスの作成
- レスポンスの送信

<!--
Child spans (or in some cases events) may be created to represent
sub-operations which require more detailed observability. Child spans should
measure the timing of the respective sub-operation, and may add additional
attributes.
-->

より詳細な観測が必要なサブ操作を表すために、子Span(場合によってはイベント)を作成することができます。子Spanは、それぞれのサブオペレーションのタイミングを測定する必要があり、さらに属性を追加することもできます。

<!--
A `Span`'s start time SHOULD be set to the current time on [span
creation](#span-creation). After the `Span` is created, it SHOULD be possible to
change its name, set its `Attribute`s, add `Event`s, and set the `Status`. These
MUST NOT be changed after the `Span`'s end time has been set.
-->

`Span`の開始時間は[Spanの作成](#spanの作成)時の現在時刻に設定されるべきです(SHOULD)。`Span`が作成された後、その名前を変更したり、`属性`を設定したり、`イベント`を追加したり、`ステータス`を設定したりすることができるべきです(SHOULD)。これらは`Span`の終了時間が設定された後に変更してはいけません(MUST NOT)。

<!--
`Span`s are not meant to be used to propagate information within a process. To
prevent misuse, implementations SHOULD NOT provide access to a `Span`'s
attributes besides its `SpanContext`.
-->

`Span` はプロセス内で情報を伝達するために使用するものではありません。誤用を防ぐために、実装では `SpanContext` 以外の `Span` の属性へのアクセスを提供すべきではありません(SHOULD NOT)。

<!--
Vendors may implement the `Span` interface to effect vendor-specific logic.
However, alternative implementations MUST NOT allow callers to create `Span`s
directly. All `Span`s MUST be created via a `Tracer`.
-->

ベンダーはベンダー固有のロジックを実現するために`Span`インターフェースを実装することができます。しかし、他の実装では、呼び出し元が直接 `Span` を作成することを許可してはなりません(MUST NOT)。すべての `Span` は `Tracer` を介して作成されなければなりません(MUST)。

<!--
### Span Creation
-->

### Spanの作成

<!--
There MUST NOT be any API for creating a `Span` other than with a [`Tracer`](#tracer).
-->

`Span`を作成するためのAPIは、[`Tracer`](#tracer)以外にあってはなりません(MUST NOT)。

<!--
In languages with implicit `Context` propagation, `Span` creation MUST NOT
set the newly created `Span` as the active `Span` in the
[current `Context`](#context-interaction) by default, but this functionality
MAY be offered additionally as a separate operation.
-->

暗黙の `Context` 伝播を持つ言語では、`Span` の作成は、デフォルトでは、新しく作成された `Span` を [現在の`Context`](#context-interaction) のアクティブな `Span` として設定してはいけません (MUST NOT)。

<!--
The API MUST accept the following parameters:
-->

このAPIは、以下のパラメータを受け入れなければなりません(MUST):

<!--
- The span name. This is a required parameter.
- The parent `Context` or an indication that the new `Span` should be a root `Span`.
  The API MAY also have an option for implicitly using
  the current Context as parent as a default behavior.
  This API MUST NOT accept a `Span` or `SpanContext` as parent, only a full `Context`.

  The semantic parent of the Span MUST be determined according to the rules
  described in [Determining the Parent Span from a Context](#determining-the-parent-span-from-a-context).
- [`SpanKind`](#spankind), default to `SpanKind.Internal` if not specified.
- [`Attributes`](../common/common.md#attributes). Additionally,
  these attributes may be used to make a sampling decision as noted in [sampling
  description](sdk.md#sampling). An empty collection will be assumed if
  not specified.

  Whenever possible, users SHOULD set any already known attributes at span creation
  instead of calling `SetAttribute` later.
- `Link`s - an ordered sequence of Links, see API definition [here](#specifying-links).
- `Start timestamp`, default to current time. This argument SHOULD only be set
  when span creation time has already passed. If API is called at a moment of
  a Span logical start, API user MUST NOT explicitly set this argument.
-->

- Spanの名前。これは必須のパラメータです。

- 親の `Context` または、ルートの `Span` であることを示す新しい `Span`。このAPIは、デフォルトの動作として、暗黙的に現在のContextを親として使用するオプションを持っていてもかまいません。このAPIは親として `Span` や `SpanContext` を受け入れてはならず(MUST NOT)、完全な `Context` のみを受け入れます。

  Spanの意味上の親は、[Contextから親のSpanを決定する](#Contextから親のSpanを決定する)に記載されているルールに従って決定されなければなりません(MUST)。

- [`SpanKind`](#spankind)。 指定されない場合は `SpanKind.Internal` がデフォルトとなります。

- [`属性`](../common/common.md#属性). さらに、これらの属性は [sampling description](sdk.md#sampling) に記載されているように、サンプリングの決定に使用されることがあります。指定がない場合は、空のコレクションが想定されます。

  可能な限り、ユーザーは、後で `SetAttribute` を呼び出すのではなく、Spanの作成時に既に判明している属性を設定すべきです。
- `Link`` - 順番に並んだリンクの列。APIの定義[こちら](#specifying-links)を参照。
- `開始時間`, デフォルトは現在時刻。この引数は、Spanの作成時間が既に経過している場合にのみ設定されるべきです(SHOULD)。Spanの論理的な開始時点でAPIが呼び出された場合、APIユーザーはこの引数を明示的に設定してはなりません(MUST NOT)。

<!--
Each span has zero or one parent span and zero or more child spans, which
represent causally related operations. A tree of related spans comprises a
trace. A span is said to be a _root span_ if it does not have a parent. Each
trace includes a single root span, which is the shared ancestor of all other
spans in the trace. Implementations MUST provide an option to create a `Span` as
a root span, and MUST generate a new `TraceId` for each root span created.
For a Span with a parent, the `TraceId` MUST be the same as the parent.
Also, the child span MUST inherit all `TraceState` values of its parent by default.
-->

各Spanは、ゼロまたは1つの親Spanとゼロまたは複数の子Spanを持ち、これらは因果関係のある操作を表します。関連するSpanのツリーがTraceを構成する。あるSpanが親を持たない場合、そのSpanは _root span_ と呼ばれます。各Traceには、Trace内の他のすべてのSpanの共有先である、単一のルートSpanが含まれます。実装では、ルートSpanとして `Span` を作成するオプションを提供しなければならず(MUST)、作成された各ルートSpanに対して新しい `TraceId` を生成しなければなりません(MUST)。親を持つSpanの場合、`TraceId` は親と同じものでなければなりません(MUST)。また、子Spanはデフォルトで親のすべての `TraceState` 値を継承しなければなりません(MUST)。

<!--
A `Span` is said to have a _remote parent_ if it is the child of a `Span`
created in another process. Each propagators' deserialization must set
`IsRemote` to true on a parent `SpanContext` so `Span` creation knows if the
parent is remote.
-->

他のプロセスで作成された `Span` の子供である場合、`Span` は _remote parent_ を持つと言われます。各プロパゲータのデシリアライゼーションでは、親の `SpanContext` に対して `IsRemote` を true に設定し、`Span` の作成時に親がリモートであるかどうかを知る必要があります。

<!--
Any span that is created MUST also be ended.
This is the responsibility of the user.
API implementations MAY leak memory or other resources
(including, for example, CPU time for periodic work that iterates all spans)
if the user forgot to end the span.
-->

作成されたSpanは必ず終了しなければなりません(MUST)。これはユーザーの責任です。API の実装は、ユーザーがSpanの終了を忘れた場合、メモリやその他のリソース(すべてのSpanを反復する定期的な作業のための CPU 時間など)をリークしてもかまいません(MAY)。

<!--
#### Determining the Parent Span from a Context
-->

#### Contextから親のSpanを決定する

<!--
When a new `Span` is created from a `Context`, the `Context` may contain a `Span`
representing the currently active instance, and will be used as parent.
If there is no `Span` in the `Context`, the newly created `Span` will be a root span.
-->

コンテキスト `Context` から新しい `Span` を作成する場合、`Context` には現在のアクティブなインスタンスを表す `Span` が含まれていることがあり、これが親として使用されます。もし`Context`に`Span`がなければ、新しく作成された`Span`はルートSpanになります。

<!--
A `SpanContext` cannot be set as active in a `Context` directly, but by
[wrapping it into a Span](#wrapping-a-spancontext-in-a-span).
For example, a `Propagator` performing context extraction may need this.
-->

`SpanContext`は`Context`の中で直接アクティブに設定することはできませんが、[spanにラップする](#wrapping-a-spancontext-in-a-span)ことで可能になります。例えば、Contextの抽出を行う `Propagator` がこれを必要とするかもしれません。

<!--
#### Specifying links
-->

#### リンクの指定

<!--
During the `Span` creation user MUST have the ability to record links to other
`Span`s. Linked `Span`s can be from the same or a different trace. See [Links
description](../overview.md#links-between-spans). `Link`s cannot be added after
Span creation.
-->

`Span`の作成中に、ユーザーは他の`Span`へのリンクを記録する機能を持たなければなりません(MUST)。リンクされた`Span`は同じトレースのものでも、異なるトレースのものでも構いません。[リンクの説明](../overview.md#links-between-spans)を参照してください。リンクはSpan作成後に追加することはできません。

<!--
A `Link` is structurally defined by the following properties:
-->

`リンク`は、以下のプロパティによって構造的に定義されます:

<!--
- `SpanContext` of the `Span` to link to.
- Zero or more [`Attributes`](../common/common.md#attributes) further describing
  the link.
-->

- リンク先の `Span` の `SpanContext`
- リンクを記述する0個以上の[`属性`](../common/common.md#属性)。

<!--
The Span creation API MUST provide:
-->

Span作成APIは以下を提供しなければなりません(MUST):

<!--
- An API to record a single `Link` where the `Link` properties are passed as
  arguments. This MAY be called `AddLink`. This API takes the `SpanContext` of
  the `Span` to link to and optional `Attributes`, either as individual
  parameters or as an immutable object encapsulating them, whichever is most
  appropriate for the language. Implementations MAY ignore links with an
  [invalid](#isvalid) `SpanContext`.
-->

- 一つの `Link` を記録するためのAPI。`Link` のプロパティが引数として渡されます。これは `AddLink` と呼んでも構いません(MAY)。このAPIは、リンク先の `Span` の `SpanContext` とオプションの `Attributes` を、個別のパラメータとして、またはそれらをカプセル化した不変のオブジェクトとして、言語に応じて最適なものを受け取ります。実装では、[invalid](#isvalid) の `SpanContext` のリンクを無視してもかまいません(MAY)。

<!--
Links SHOULD preserve the order in which they're set.
-->

リンクは設定された順序を維持すべきです(SHOULD)。

<!--
### Span operations
-->

### Span操作

<!--
With the exception of the function to retrieve the `Span`'s `SpanContext` and
recording status, none of the below may be called after the `Span` is finished.
-->

Span`の`SpanContext`と記録状態を取得する関数を除いて、`Span`が終了した後に以下の関数を呼び出すことはできません。

<!--
#### Get Context
-->

#### Contextの取得

<!--
The Span interface MUST provide:
-->

Spanインタフェースは以下の機能を提供しなければなりません(MUST)。

<!--
- An API that returns the `SpanContext` for the given `Span`. The returned value
  may be used even after the `Span` is finished. The returned value MUST be the
  same for the entire Span lifetime. This MAY be called `GetContext`.
-->

- 与えられた `Span` に対応する `SpanContext` を返すAPI。返された値は、`Span`が終了した後でも使用することができます。返される値は、Spanのライフタイム全体で同じものでなければなりません(MUST)。これは `GetContext` と呼んでも構いません(MAY)。

<!--
#### IsRecording
-->

#### IsRecording

<!--
Returns true if this `Span` is recording information like events with the
`AddEvent` operation, attributes using `SetAttributes`, status with `SetStatus`,
etc.
-->

この `Span` が、`AddEvent` オペレーションによるイベント、`SetAttributes` による属性、`SetStatus` によるステータスなどの情報を記録している場合は、true を返します。

<!--
After a `Span` is ended, it usually becomes non-recording and thus
`IsRecording` SHOULD consequently return false for ended Spans.
Note: Streaming implementations, where it is not known if a span is ended,
are one expected case where `IsRecording` cannot change after ending a Span.
-->

Span`が終了すると、通常は記録されない状態になるため、終了したSpanに対しては、`IsRecording`は結果的にfalseを返すべきです(SHOULD)。注意: Spanが終了したかどうかが分からないストリーミングの実装は、Spanの終了後に `IsRecording` が変更できない想定されるケースの一つです。

<!--
`IsRecording` SHOULD NOT take any parameters.
-->

`IsRecording` はパラメータを取るべきではありません(SHOULD NOT)。

<!--
This flag SHOULD be used to avoid expensive computations of a Span attributes or
events in case when a Span is definitely not recorded. Note that any child
span's recording is determined independently from the value of this flag
(typically based on the `sampled` flag of a `TraceFlags` on
[SpanContext](#spancontext)).
-->

このフラグは、Spanが確実に記録されていない場合に、Spanの属性やイベントの高価な計算を避けるために使用されるべきです(SHOULD)。子Spanの記録は、このフラグの値とは無関係に決定されることに注意してください(通常、[SpanContext](#spancontext)の `TraceFlags` の `sampled` フラグに基づいて決定されます)。

<!--
This flag may be `true` despite the entire trace being sampled out. This
allows to record and process information about the individual Span without
sending it to the backend. An example of this scenario may be recording and
processing of all incoming requests for the processing and building of
SLA/SLO latency charts while sending only a subset - sampled spans - to the
backend. See also the [sampling section of SDK design](sdk.md#sampling).
-->

このフラグは、トレース全体がサンプルアウトされているにもかかわらず、`true`になることがあります。これにより、個々のSpanに関する情報をバックエンドに送らずに記録・処理することができます。このシナリオの例としては、SLA/SLOのレイテンシーチャートを処理・構築するために、すべての受信リクエストを記録・処理する一方で、サブセット(サンプリングされたSpan)のみをバックエンドに送信することが考えられます。[SDKデザインのサンプリングの章](sdk.md#sampling)も参照してください。

<!--
Users of the API should only access the `IsRecording` property when
instrumenting code and never access `SampledFlag` unless used in context
propagators.
-->

APIのユーザーは、コードを計装する際にのみ、`IsRecording`プロパティにアクセスし、Context Propagatorで使用する場合を除き、`SampledFlag`には決してアクセスしないでください。

<!--
#### Set Attributes
-->

#### 属性の設定

<!--
A `Span` MUST have the ability to set [`Attributes`](../common/common.md#attributes) associated with it.
-->

`Span`は、それに関連する[`Attributes`](../common/common.md#属性)を設定する能力を持たなければなりません(MUST)。

<!--
The Span interface MUST provide:
-->

Spanのインターフェースは以下のAPIを提供しなければなりません(MUST)。

<!--
- An API to set a single `Attribute` where the attribute properties are passed
  as arguments. This MAY be called `SetAttribute`. To avoid extra allocations some
  implementations may offer a separate API for each of the possible value types.
-->

- 一つの`Attribute`を設定するAPIで、属性のプロパティが引数として渡されます。これは`SetAttribute`と呼んでも構いません(MAY)。余分な割り当てを避けるために、いくつかの実装では、可能な値のタイプごとに個別のAPIを提供することがあります。

<!--
The Span interface MAY provide:
-->

Spanのインターフェースは、以下の機能を提供しても構いません(MAY):

<!--
- An API to set multiple `Attributes` at once, where the `Attributes` are passed in a
  single method call.
-->

- 複数の `Attribute` を一度に設定するためのAPI。`Attribute` は単一のメソッドコールで渡されます。

<!--
Setting an attribute with the same key as an existing attribute SHOULD overwrite
the existing attribute's value.
-->

既存の属性と同じキーを持つ属性を設定すると、既存の属性の値は上書きされるべきです(SHOULD)。

<!--
Note that the OpenTelemetry project documents certain ["standard
attributes"](semantic_conventions/README.md) that have prescribed semantic meanings.
-->

OpenTelemetryプロジェクトでは、セマンティックな意味を持つ特定の["標準属性"](semantic_conventions/README.md)を文書化していることに注意してください。

<!--
Note that [Samplers](sdk.md#sampler) can only consider information already
present during span creation. Any changes done later, including new or changed
attributes, cannot change their decisions.
-->

なお、[Sampler](sdk.md#sampler)は、Span作成時にすでに存在する情報しか考慮できません。新しい属性や変更された属性など、後から行われたいかなる変更も、その決定を変えることはできません。

<!--
#### Add Events
-->

#### イベントの追加

<!--
A `Span` MUST have the ability to add events. Events have a time associated
with the moment when they are added to the `Span`.
-->

`Span`はイベントを追加する機能を持たなければなりません(MUST)。イベントは、`Span`に追加された瞬間と関連付けられた時間を持ちます。

<!--
An `Event` is structurally defined by the following properties:
-->

`イベント`は、以下のプロパティによって構造的に定義されます。

<!--
- Name of the event.
- A timestamp for the event. Either the time at which the event was
  added or a custom timestamp provided by the user.
- Zero or more [`Attributes`](../common/common.md#attributes) further describing
  the event.
-->

- イベントの名前。
- イベントのタイムスタンプ。イベントが追加された時刻か、ユーザーが指定したカスタムタイムスタンプのいずれかです。
- イベントを説明するための0個以上の[`属性`](../common/common.md#属性)。

<!--
The Span interface MUST provide:
-->

Spanインタフェースは以下のAPIを提供しなければなりません(MUST):

<!--
- An API to record a single `Event` where the `Event` properties are passed as
  arguments. This MAY be called `AddEvent`.
  This API takes the name of the event, optional `Attributes` and an optional
  `Timestamp` which can be used to specify the time at which the event occurred,
  either as individual parameters or as an immutable object encapsulating them,
  whichever is most appropriate for the language. If no custom timestamp is
  provided by the user, the implementation automatically sets the time at which
  this API is called on the event.
-->

- 一つの `Event` を記録するための API で、`Event` のプロパティが引数として渡されます。これは `AddEvent` と呼んでも構いません(MAY)。この API は、イベントの名前、オプションの `Attributes` と、イベントが発生した時間を指定するために使用できるオプションの `Timestamp` を、個々のパラメータとして、またはそれらをカプセル化した不変のオブジェクトとして、言語に応じて最も適切なものを受け取ります。ユーザーからカスタムのタイムスタンプが提供されない場合、実装ではイベントでこのAPIが呼び出された時間が自動的に設定されます。

<!--
Events SHOULD preserve the order in which they are recorded.
This will typically match the ordering of the events' timestamps,
but events may be recorded out-of-order using custom timestamps.
-->

イベントは、記録された順番を保持すべきです(SHOULD)。これは通常、イベントのタイムスタンプの順序と一致しますが、カスタムのタイムスタンプを使用して、イベントを順不同に記録することができます。

<!--
Consumers should be aware that an event's timestamp might be before the start or
after the end of the span if custom timestamps were provided by the user for the
event or when starting or ending the span.
The specification does not require any normalization if provided timestamps are
out of range.
-->

利用者は、イベントのタイムスタンプがSpanの開始前や終了後になる可能性があることに注意する必要があります。この仕様では、提供されたタイムスタンプが範囲外であっても、正規化は必要ありません。

<!--
Note that the OpenTelemetry project documents certain ["standard event names and
keys"](semantic_conventions/README.md) which have prescribed semantic meanings.
-->

OpenTelemetryプロジェクトでは、セマンティックな意味を持つ["標準イベント名とキー"](semantic_conventions/README.md)を規定していることに注意してください。

<!--
Note that [`RecordException`](#record-exception) is a specialized variant of
`AddEvent` for recording exception events.
-->

なお、[`RecordException`](#record-exception)は、例外イベントを記録するための `AddEvent` の特殊化したバリエーションです。

<!--
#### Set Status
-->

#### ステータス設定

<!--
Sets the `Status` of the `Span`. If used, this will override the default `Span`
status, which is `Unset`.
-->

`Span`の`ステータス`を設定します。これを使用すると、`Span`のデフォルトのステータスである`Unset`がオーバーライドされます。

<!--
`Status` is structurally defined by the following properties:
-->

`ステータス`は、以下のプロパティによって構造的に定義されます。

<!--
- `StatusCode`, one of the values listed below.
- Optional `Description` that provides a descriptive message of the `Status`.
  `Description` MUST only be used with the `Error` `StatusCode` value.
  An empty `Description` is equivalent with a not present one.
-->

- 以下に示す値の一つである `StatusCode`
- 任意の `Description` は、`ステータス` の説明的なメッセージを提供します。`Description` は `Error` `StatusCode` と共にのみ使用しなければなりません(MUST)。空の `Description` は、存在しないものと同じです。

<!--
`StatusCode` is one of the following values:
-->

`StatusCode`は以下のいずれかの値です。

<!--
- `Unset`
  - The default status.
- `Ok`
  - The operation has been validated by an Application developer or Operator to
    have completed successfully.
- `Error`
  - The operation contains an error.
-->

- `Unset`
  - デフォルトのステータス
- `Ok`
  - アプリケーション開発者やオペレーターによって、操作が正常に完了したことが確認されていることを示す
- `Error`
  - 操作にエラーが発生したことを示す

<!--
The Span interface MUST provide:
-->

Spanインタフェースは以下のAPIを提供しなければなりません(MUST)

<!--
- An API to set the `Status`. This SHOULD be called `SetStatus`. This API takes
  the `StatusCode`, and an optional `Description`, either as individual
  parameters or as an immutable object encapsulating them, whichever is most
  appropriate for the language. `Description` MUST be IGNORED for `StatusCode`
  `Ok` & `Unset` values.
-->

- `Status`を設定するためのAPI。これは `SetStatus` と呼ばれるべきです(SHOULD)。このAPIは `StatusCode` とオプションの `Description` を個別のパラメータとして、あるいはそれらをカプセル化した不変のオブジェクトとして、言語に応じて最適なものを受け取ります。`Description` は `StatusCode` の `Ok` と `Unset` の値に対しては無視されなければなりません(MUST)。

<!--
The status code SHOULD remain unset, except for the following circumstances:
-->

以下の状況を除き、`Status Code`は設定されないままであるべきです(SHOULD):

<!--
When the status is set to `Error` by Instrumentation Libraries, the status codes
SHOULD be documented and predictable. The status code should only be set to `Error`
according to the rules defined within the semantic conventions. For operations
not covered by the semantic conventions, Instrumentation Libraries SHOULD
publish their own conventions, including status codes.
-->

計装するライブラリでステータスが`Error`に設定される場合、そのステータスコードは文書化され、予測可能であるべきです。ステータスコードは、セマンティック規約で定義されているルールに従ってのみ `Error` に設定する必要があります。セマンティック規約でカバーされていない操作については、計装するライブラリは、ステータスコードを含む独自の規約を公開するべきです(SHOULD)。

<!--
Generally, Instrumentation Libraries SHOULD NOT set the status code to `Ok`,
unless explicitly configured to do so. Instrumention libraries SHOULD leave the
status code as `Unset` unless there is an error, as described above.
-->

一般に、計装するライブラリでは、明示的に構成されていない限り、ステータスコードを`Ok`に設定すべきではありません。軽装するライブラリは、上述のようにエラーが発生しない限り、ステータスコードを `Unset` のままにしておくべきです(SHOULD)。

<!--
Application developers and Operators may set the status code to `Ok`.
-->

アプリケーションの開発者やオペレータは、ステータスコードを `Ok` に設定することができます。

<!--
Analysis tools SHOULD respond to an `Ok` status by suppressing any errors they
would otherwise generate. For example, to suppress noisy errors such as 404s.
-->

解析ツールは、`Ok`ステータスに対して、それ以外の方法で生成されるエラーを抑制して対応すべきです(SHOULD)。例えば、404のようなノイズの多いエラーを抑制するためです。

<!--
Only the value of the last call will be recorded, and implementations are free
to ignore previous calls.
-->

最後の呼び出しの値だけが記録されるので、実装は以前の呼び出しを自由に無視することができます。

<!--
#### UpdateName
-->

#### UpdateName

<!--
Updates the `Span` name. Upon this update, any sampling behavior based on `Span`
name will depend on the implementation.
-->

`Span` 名を更新します。この更新に伴い、`Span`名に基づいたサンプリング動作は、実装に依存することになります。

<!--
Note that [Samplers](sdk.md#sampler) can only consider information already
present during span creation. Any changes done later, including updated span
name, cannot change their decisions.
-->

[Sampler](sdk.md#sampler)は、Span作成時にすでに存在する情報しか考慮できないことに注意してください。後から行われた変更(Span名の更新など)は、その決定を変えることはできません。

<!--
Alternatives for the name update may be late `Span` creation, when Span is
started with the explicit timestamp from the past at the moment where the final
`Span` name is known, or reporting a `Span` with the desired name as a child
`Span`.
-->

名前の更新には、最終的な `Span` の名前が判明した時点で、過去からの明示的なタイムスタンプで Span を開始するという、遅い `Span` の作成や、希望の名前を持つ `Span` を子 `Span` として報告するという方法があります。

<!--
Required parameters:
-->

必須パラメータ:

<!--
- The new **span name**, which supersedes whatever was passed in when the
  `Span` was started
-->

- 新しい **Span名**。`Span` の起動時に渡されたものよりも優先されます。

<!--
#### End
-->

#### End

<!--
Signals that the operation described by this span has
now (or at the time optionally specified) ended.
-->

このSpanで記述された操作が現在(または任意に指定された時間)で終了したことを示すSignalです。

<!--
Implementations SHOULD ignore all subsequent calls to `End` and any other Span methods,
i.e. the Span becomes non-recording by being ended
(there might be exceptions when Tracer is streaming events
and has no mutable state associated with the `Span`).
-->

実装は、その後の `End` やその他のSpanメソッドの呼び出しをすべて無視すべきです(SHOULD)。つまり、Spanは終了することで記録されない状態になります(Tracer がイベントをストリーミングしていて、`Span` に関連付けられた変更可能な状態がない場合は例外になる(XXX:例外が発生する？)かもしれません)。

<!--
Language SIGs MAY provide methods other than `End` in the API that also end the
span to support language-specific features like `with` statements in Python.
However, all API implementations of such methods MUST internally call the `End`
method and be documented to do so.
-->

Language SIG は、Python の `with` 文のような言語固有の機能をサポートするために、Spanを終了させる `End` 以外のメソッドを API で提供しても構いません(MAY)。ただし、そのようなメソッドのすべての API 実装は、内部的に `End` メソッドを呼び出し、その挙動がドキュメント化されていなければなりません(MUST)。

<!--
`End` MUST NOT have any effects on child spans.
Those may still be running and can be ended later.
-->

`End` は子Spanに影響を与えてはなりません(MUST NOT)。これらのSpanはまだ実行されている可能性があり、後で終了させることができます。

<!--
`End` MUST NOT inactivate the `Span` in any `Context` it is active in.
It MUST still be possible to use an ended span as parent via a Context it is
contained in. Also, any mechanisms for putting the Span into a Context MUST
still work after the Span was ended.
-->

`End` は、それがアクティブであるどの `Context` においても `Span` を非アクティブにしてはなりません (MUST NOT)。終了したSpanを、それが含まれているContextを介して親として使用することは可能でなければなりません(MUST)。また、SpanをContextに入れるためのメカニズムは、Spanが終了した後も機能しなければなりません(MUST)。

<!--
Parameters:
-->

パラメータ:

<!--
- (Optional) Timestamp to explicitly set the end timestamp.
  If omitted, this MUST be treated equivalent to passing the current time.
-->

- (任意) 終了タイムスタンプを明示的に設定するタイムスタンプ。省略された場合、これは現在の時刻を渡すのと同等に扱われなければなりません(MUST)

<!--
Expect this operation to be called in the "hot path" of production
applications. It needs to be designed to complete fast, if not immediately.
This operation itself MUST NOT perform blocking I/O on the calling thread.
Any locking used needs be minimized and SHOULD be removed entirely if
possible. Some downstream SpanProcessors and subsequent SpanExporters called
from this operation may be used for testing, proof-of-concept ideas, or
debugging and may not be designed for production use themselves. They are not
in the scope of this requirement and recommendation.
-->

この操作は、プロダクション・アプリケーションの「ホットパス」と呼ばれることが予想されます。即座にとはいかないまでも、素早く完了するように設計する必要があります。この操作自体は、呼び出したスレッドでブロッキングI/Oを実行してはいけません(MUST NOT)。使用されるいかなるロックも最小限にする必要があり(SHOULD)、可能であれば完全に除去すべきです。この操作から呼び出されるいくつかの下流のSpanProcessorとそれに続くSpanExporterは、テスト、概念実証のアイデア、またはデバッグのために使用されるかもしれませんが、それ自体は製品使用のために設計されていないかもしれません。それらは、この要求と勧告の範囲外です。

<!--
#### Record Exception
-->

#### 例外の記録

<!--
To facilitate recording an exception languages SHOULD provide a
`RecordException` method if the language uses exceptions.
This is a specialized variant of [`AddEvent`](#add-events),
so for anything not specified here, the same requirements as for `AddEvent` apply.
-->

言語が例外を使用している場合、例外の記録を容易にするために、対応する言語の実装は `RecordException` メソッドを提供するべきです(SHOULD)。これは、[`AddEvent`](#add-events)を特殊化したもので、ここで指定されていないものについては、`AddEvent`と同じ要件が適用されます。

<!--
The signature of the method is to be determined by each language
and can be overloaded as appropriate.
The method MUST record an exception as an `Event` with the conventions outlined in
the [exception semantic conventions](semantic_conventions/exceptions.md) document.
The minimum required argument SHOULD be no more than only an exception object.
-->

メソッドのシグネチャは各言語で決定され、必要に応じてオーバーロードすることができます。このメソッドは、[例外セマンティック規約](semantic_conventions/exceptions.md)ドキュメントに概説されている規約に従って、`Event`として例外を記録しなければなりません(MUST)。最小限必要な引数は、例外オブジェクトのみであるべきです(SHOULD)。

<!--
If `RecordException` is provided, the method MUST accept an optional parameter
to provide any additional event attributes
(this SHOULD be done in the same way as for the `AddEvent` method).
If attributes with the same name would be generated by the method already,
the additional attributes take precedence.
-->

`RecordException`が提供される場合、メソッドは追加のイベント属性を提供するためのオプションのパラメータを受け入れなければなりません(これは`AddEvent`メソッドと同じ方法で行われるべきです)。同じ名前の属性が既にそのメソッドで生成されている場合は、追加の属性が優先されます。

<!--
Note: `RecordException` may be seen as a variant of `AddEvent` with
additional exception-specific parameters and all other parameters being optional
(because they have defaults from the exception semantic convention).
-->

注意: `RecordException` は、`AddEvent` に例外特有のパラメータを追加し、その他のすべてのパラメータをオプションとしたものと考えることができます(例外のセマンティックな規約に基づいてデフォルト値が設定されているため)．

<!--
### Span lifetime
-->

### Span ライフタイム

<!--
Span lifetime represents the process of recording the start and the end
timestamps to the Span object:
-->

Spanのライフタイムは、開始と終了のタイムスタンプをSpanオブジェクトに記録するプロセスを表します。

<!--
- The start time is recorded when the Span is created.
- The end time needs to be recorded when the operation is ended.
-->

- 開始時間は、Spanが作成されたときに記録されます。
- 終了時間は、操作が終了したときに記録する必要があります。

<!--
Start and end time as well as Event's timestamps MUST be recorded at a time of a
calling of corresponding API.
-->

開始時刻、終了時刻、およびイベントのタイムスタンプは、対応するAPIを呼び出した時点で記録されなければなりません(MUST)。

<!--
### Wrapping a SpanContext in a Span
-->

### SpanContextをSpanでラップする

<!--
The API MUST provide an operation for wrapping a `SpanContext` with an object
implementing the `Span` interface. This is done in order to expose a `SpanContext`
as a `Span` in operations such as in-process `Span` propagation.
-->

APIは、`Span`インターフェイスを実装したオブジェクトで`SpanContext`をラップする操作を提供しなければなりません(MUST)。これは、`SpanContext`を`Span`として公開するために、プロセス内の`Span`伝搬などの操作で行われます。

<!--
If a new type is required for supporting this operation, it SHOULD NOT be exposed
publicly if possible (e.g. by only exposing a function that returns something
with the Span interface type). If a new type is required to be publicly exposed,
it SHOULD be named `NonRecordingSpan`.
-->

この操作をサポートするために新しい型が必要な場合は、可能であれば公開すべきではありません(SHOULD NOT)(例えば、Spanのインターフェイスの型を持つものを返す関数のみを公開するなど)。新しい型を公開する必要がある場合は、`NonRecordingSpan`という名前にすべきです(SHOULD)。

<!--
The behavior is defined as follows:
-->

この動作は次のように定義されています。

<!--
- `GetContext` MUST return the wrapped `SpanContext`.
- `IsRecording` MUST return `false` to signal that events, attributes and other elements
  are not being recorded, i.e. they are being dropped.
-->

- `GetContext` は、ラップされた `SpanContext` を返さなければなりません(MUST)。
- `IsRecording` は、イベントや属性、その他の要素が記録されていないこと、つまりドロップされていることを示すために、`false` を返さなければなりません (MUST)。

<!--
The remaining functionality of `Span` MUST be defined as no-op operations.
Note: This includes `End`, so as an exception from the general rule,
it is not required (or even helpful) to end such a Span.
-->

`Span`の残りの機能は、no-op操作として定義されなければなりません(MUST)。注意: これには `End` も含まれており、一般的なルールの例外として、このようなSpanを終了することは必須ではありません(あるいは有用でもありません)。

<!--
This functionality MUST be fully implemented in the API, and SHOULD NOT be overridable.
-->

この機能はAPIに完全に実装されなければなりません(MUST)。また、オーバーライドできないようにすべきです(SHOULD NOT)。

<!--
## SpanKind
-->

## SpanKind

<!--
`SpanKind` describes the relationship between the Span, its parents,
and its children in a Trace.  `SpanKind` describes two independent
properties that benefit tracing systems during analysis.
-->

`SpanKind` は、Trace内のSpan、その親、子の関係を記述します。 `SpanKind` は、解析時にトレースシステムに利益をもたらす2つの独立したプロパティを記述します。

<!--
The first property described by `SpanKind` reflects whether the Span
is a remote child or parent.  Spans with a remote parent are
interesting because they are sources of external load.  Spans with a
remote child are interesting because they reflect a non-local system
dependency.
-->

`SpanKind`で記述される最初のプロパティは、Spanがリモートの子か親かを反映しています。 リモートの親を持つSpanは、外部からの負荷の源となるので興味深いです(XXX:いい訳が欲しい)。 リモートの子を持つSpanは、非ローカルなシステムの依存関係を反映しているので興味深いです(XXX:いい訳が欲しい)。

<!--
The second property described by `SpanKind` reflects whether a child
Span represents a synchronous call.  When a child span is synchronous,
the parent is expected to wait for it to complete under ordinary
circumstances.  It can be useful for tracing systems to know this
property, since synchronous Spans may contribute to the overall trace
latency. Asynchronous scenarios can be remote or local.
-->

`SpanKind`に記述されている2つ目のプロパティは、子Spanが同期呼び出しを表しているかどうかを反映しています。 子Spanが同期的である場合、親は通常の状況下でその完了を待つことが期待されます。 同期的なSpanは全体的なTraceのレイテンシーに影響を与える可能性があるため、トレースシステムにとってこのプロパティを知ることは有用です。非同期のシナリオは、リモートまたはローカルになります。

<!--
In order for `SpanKind` to be meaningful, callers SHOULD arrange that
a single Span does not serve more than one purpose.  For example, a
server-side span SHOULD NOT be used directly as the parent of another
remote span.  As a simple guideline, instrumentation should create a
new Span prior to extracting and serializing the SpanContext for a
remote call.
-->

`SpanKind`に意味を持たせるために、呼び出し側は1つのSpanが複数の目的を果たさないように調整すべきです(SHOULD)。 例えば、サーバーサイドのSpanを、別のリモートSpanの親として直接使用すべきではありません(SHOULD NOT)。 簡単なガイドラインとして、計装は、リモートコールのSpanContextを抽出してシリアライズする前に、新しいSpanを作成するべきです。

<!--
These are the possible SpanKinds:
-->

これらは、可能なSpanKindです:

<!--
* `SERVER` Indicates that the span covers server-side handling of a
  synchronous RPC or other remote request.  This span is the child of
  a remote `CLIENT` span that was expected to wait for a response.
* `CLIENT` Indicates that the span describes a synchronous request to
  some remote service.  This span is the parent of a remote `SERVER`
  span and waits for its response.
* `PRODUCER` Indicates that the span describes the parent of an
  asynchronous request.  This parent span is expected to end before
  the corresponding child `CONSUMER` span, possibly even before the
  child span starts. In messaging scenarios with batching, tracing
  individual messages requires a new `PRODUCER` span per message to
  be created.
* `CONSUMER` Indicates that the span describes the child of an
  asynchronous `PRODUCER` request.
* `INTERNAL` Default value. Indicates that the span represents an
  internal operation within an application, as opposed to an
  operations with remote parents or children.
-->

* `SERVER` : このSpanが、同期RPCやその他のリモートリクエストのサーバーサイドの処理をカバーしていることを示します。このSpanは、応答を待つことが期待されるリモートの `CLIENT` Spanの子です。
* `CLIENT` : このSpanが、あるリモートサービスへの同期的なリクエストを記述していることを示します。このSpanは、リモートの `SERVER` Spanの親であり、その応答を待ちます。
* `PRODUCER` : Spanが非同期リクエストの親を記述していることを示します。この親のSpanは、対応する子の `CONSUMER` Spanの前に終了することが予想され、場合によっては子のSpanが始まる前に終了することもあります。バッチ処理を伴うメッセージングシナリオでは、個々のメッセージをトレースするには、メッセージごとに新しい `PRODUCER` Spanを作成する必要があります。
* `CONSUMER` : Spanが非同期の `PRODUCER` リクエストの子を記述していることを示します。
* `INTERNAL` : デフォルト値です。Spanが、リモートの親や子を持つ操作ではなく、アプリケーション内の内部操作であることを示します。

<!--
To summarize the interpretation of these kinds:
-->

これらの種類をまとめると以下となります。

<!--
| `SpanKind` | Synchronous | Asynchronous | Remote Incoming | Remote Outgoing |
|--|--|--|--|--|
| `CLIENT` | yes | | | yes |
| `SERVER` | yes | | yes | |
| `PRODUCER` | | yes | | maybe |
| `CONSUMER` | | yes | maybe | |
| `INTERNAL` | | | | |
-->

| `SpanKind` | 同期 | 非同期 | Remote Incoming | Remote Outgoing |
|--|--|--|--|--|
| `CLIENT` | yes | | | yes |
| `SERVER` | yes | | yes | |
| `PRODUCER` | | yes | | maybe |
| `CONSUMER` | | yes | maybe | |
| `INTERNAL` | | | | |

<!--
## Concurrency
-->

## 並行性

<!--
For languages which support concurrent execution the Tracing APIs provide
specific guarantees and safeties. Not all of API functions are safe to
be called concurrently.
-->

同時実行をサポートする言語では、Tracing APIは特定の保証と安全性を提供します。すべてのAPI関数が同時に呼び出されても安全というわけではありません。

<!--
**TracerProvider** - all methods are safe to be called concurrently.
-->

**TracerProvider** - すべてのメソッドは、同時に呼び出されても安全です。

<!--
**Tracer** - all methods are safe to be called concurrently.
-->

**Tracer** - すべてのメソッドは、同時に呼び出されても安全です。

<!--
**Span** - All methods of Span are safe to be called concurrently.
-->

**Span** - Spanのすべてのメソッドは、同時に呼び出されても安全です。

<!--
**Event** - Events are immutable and safe to be used concurrently.
-->

**Event** - イベントは不変であり、同時に使用しても安全です。

<!--
**Link** - Links are immutable and safe to be used concurrently.
-->

**Link** - リンクは不変であり、同時に使用しても安全です。

<!--
## Included Propagators
-->

## 含まれるPropagator

<!--
See [Propagators Distribution](../context/api-propagators.md#propagators-distribution)
for how propagators are to be distributed.
-->

プロパゲータの配布方法については、[Propagators Distribution](../context/api-propagators.md#propagators-distribution)を参照してください。

<!--
## Behavior of the API in the absence of an installed SDK
-->

## SDKがインストールされていない場合のAPIの動作について

<!--
In general, in the absence of an installed SDK, the Trace API is a "no-op" API.
This means that operations on a Tracer, or on Spans, should have no side effects and do nothing. However, there
is one important exception to this general rule, and that is related to propagation of a `SpanContext`:
The API MUST create a [non-recording Span](#wrapping-a-spancontext-in-a-span) with the `SpanContext`
that is in the `Span` in the parent `Context` (whether explicitly given or implicit current) or,
if the parent is a non-recording Span (which it usually always is if no SDK is present),
it MAY return the parent Span back from the creation method.
If the parent `Context` contains no `Span`, an empty non-recording Span MUST be returned instead
(i.e., having a `SpanContext` with all-zero Span and Trace IDs, empty Tracestate, and unsampled TraceFlags).
This means that a `SpanContext` that has been provided by a configured `Propagator`
will be propagated through to any child span and ultimately also `Inject`,
but that no new `SpanContext`s will be created.
-->

一般的に、SDK がインストールされていない場合、Trace API は「no-op」API です。これは、TracerやSpanに対する操作は、副作用がなく、何もしないことを意味します。しかし、この一般的なルールには重要な例外があり、それは `SpanContext` の伝播に関するものです。APIは、親の`Context`(明示的に与えられたものであれ、暗黙の了解であるものであれ)の`Span`の中にある`SpanContext`で、[non-recording Span](#wrapping-a-spancontext-in-a-span)を作成しなければなりません(MUST)。また、親が非記録的なSpanである場合(SDKが存在しない場合、通常は常にそうです)、作成メソッドから親のSpanを返してもかまいません(MAY)。親の `Context` に `Span` が含まれていない場合は、代わりに空の非記録的な Span を返さなければなりません(MUST)(つまり、すべてゼロの Span と Trace ID、空の Tracestate、サンプルされていない TraceFlags を持つ `SpanContext` を持つこと)。つまり、設定された `Propagator` によって提供された `SpanContext` は、すべての子Spanに伝搬され、最終的には `Inject` にも伝搬されますが、新しい `SpanContext` は作成されません。