<!--
# OpenTracing Compatibility
-->

# OpenTracing 互換性

**Status**: [Experimental](../document-status.md).

<details>
<summary>Table of Contents</summary>

<!--
* [Abstract](#abstract)
* [Create an OpenTracing Tracer Shim](#create-an-opentracing-tracer-shim)
* [Tracer Shim](#tracer-shim)
  * [Inject](#inject)
  * [Extract](#extract)
* [OpenTelemetry Span and SpanContext Shim relationship](#opentelemetry-span-and-spancontext-shim-relationship)
* [Span Shim](#span-shim)
  * [Get Context](#get-context)
  * [Get Baggage Item](#get-baggage-item)
  * [Set Baggage Item](#set-baggage-item)
  * [Set Tag](#set-tag)
  * [Log](#log)
  * [Finish](#finish)
* [SpanContext Shim](#spancontext-shim)
  * [Get Baggage Items](#get-baggage-items)
* [ScopeManager Shim](#scopemanager-shim)
  * [Activate a Span](#activate-a-span)
  * [Get the active Span](#get-the-active-span)
-->

* [概要](#概要)
* [OpenTracing Tracer Shimの作成](#opentracing-tracer-shimの作成)
* [Tracer Shim](#tracer-shim)
  * [注入(Inject)](#注入-inject)
  * [抽出(Extract)](#抽出-extract)
* [OpenTelemetry SpanとSpanContext Shimの関係](#opentelemetry-spanとspancontext-shimの関係)
* [Span Shim](#span-shim)
  * [Contextの取得](#Contextの取得)
  * [Baggage Itemの取得](#baggage-itemの取得)
  * [Baggage Itemの設定](#baggage-itemの設定)
  * [Tagの設定](#Tagの設定)
  * [ログ](#ログ)
  * [Finish](#finish)
* [SpanContext Shim](#spancontext-shim)
  * [複数のBaggage Itemの取得](#複数のbaggage-itemの取得)
* [ScopeManager Shim](#scopemanager-shim)
  * [Spanの有効化](#Spanの有効化)
  * [有効なSpanの取得](#有効なSpanの取得)

</details>

<!--
## Abstract
-->

## 概要

<!--
The OpenTelemetry project aims to provide backwards compatibility with the
[OpenTracing](https://opentracing.io) project in order to ease migration of
instrumented codebases.
-->

OpenTelemetryプロジェクトは、[OpenTracing](https://opentracing.io)プロジェクトとの後方互換性を提供し、計装されたコードベースの移行を容易にすることを目的としています。

<!--
This functionality will be provided as a bridge layer implementing the
[OpenTracing API](https://github.com/opentracing/specification) using the
OpenTelemetry API. This layer MUST NOT rely on implementation specific details
of any SDK.
-->

この機能は、OpenTelemetry APIを用いて[OpenTracing API](https://github.com/opentracing/specification)を実装するブリッジレイヤーとして提供されます。このレイヤーは、どのSDKの実装上の詳細にも依存してはなりません。

<!--
More specifically, the intention is to allow OpenTracing instrumentation to be
recorded using OpenTelemetry. This Shim Layer MUST NOT publicly expose any
upstream OpenTelemetry API.
-->

より具体的には、OpenTracingの計装をOpenTelemetryで記録できるようにすることが目的です。この互換性レイヤーは、アップストリームのOpenTelemetry APIを公開してはなりません(MUST NOT)。

<!--
This functionality MUST be defined in its own OpenTracing Shim Layer, not in the
OpenTracing nor the OpenTelemetry API or SDK.
-->

この機能は、OpenTracingやOpenTelemetryのAPIやSDKではなく、独自のOpenTracing Shim レイヤーで定義されなければなりません(MUST)。

<!--
The OpenTracing Shim and the OpenTelemetry API/SDK are expected to be consumed
simultaneously in a running service, in order to ease migration from the former
to the latter.
-->

OpenTracing ShimとOpenTelemetry API/SDKは、前者から後者への移行を容易にするために、実行中のサービスで同時に利用されることが期待されています。

<!--
## Create an OpenTracing Tracer Shim
-->

## OpenTracing Tracer Shimの作成

<!--
This operation is used to create a new OpenTracing `Tracer`:
-->

この操作は、新しいOpenTracingの`Tracer`を作成するのに使われます。

<!--
This operation MUST accept the following parameters:
-->

この操作は、以下のパラメータを受け入れなければなりません(MUST):
<!--
- An OpenTelemetry `Tracer`, used to create `Span`s.
- OpenTelemetry `Propagator`s to be used to perform injection and extraction
  for the the OpenTracing `TextMap` and `HTTPHeaders` formats.
  If not specified, no `Propagator` values will be stored in the Shim, and
  the global OpenTelemetry `TextMap` propagator will be used for both OpenTracing
  `TextMap` and `HTTPHeaders` formats.
-->

- OpenTelemetryの `Tracer` で、`Span` を作成するのに使います。
- OpenTracingの `TextMap` と `HTTPHeaders` フォーマットの注入と抽出を行うために使用されるOpenTelemetryの `Propagator` です。指定されない場合、`Propagator` の値は Shim に保存されず、グローバルなOpenTelemetryの `TextMap` プロパゲータが OpenTracing の `TextMap` および `HTTPHeaders` フォーマットの両方に使用されます。

<!--
The API MUST return an OpenTracing `Tracer`.
-->

APIはOpenTracingの`Tracer`を返さなければなりません(MUST)。

<!--
```java
// Create a Tracer Shim relying on the global propagators.
createTracerShim(tracer);

// Create a Tracer Shim using:
// 1) TraceContext propagator for TextMap
// 2) Jaeger propagator for HttPHeaders.
createTracerShim(tracer, OTPropagatorsBuilder()
  .setTextMap(W3CTraceContextPropagator.getInstance())
  .setHttpHeaders(JaegerPropagator.getInstance())
  .build());
```
-->

```java
// グローバルプロパゲータに依存するTracer Shimを作成します。
createTracerShim(tracer);

// 以下を使用して、Tracer Shimを作成します。
// 1) TextMapのためのTraceContextプロパゲータ
// 2) HttpHeadersのJaegerプロパゲータ
createTracerShim(tracer, OTPropagatorsBuilder()
  .setTextMap(W3CTraceContextPropagator.getInstance())
  .setHttpHeaders(JaegerPropagator.getInstance())
  .build());
```

<!--
See OpenTracing Propagation
[Formats](https://github.com/opentracing/specification/blob/master/specification.md#extract-a-spancontext-from-a-carrier).
-->

OpenTracing Propagation [フォーマット](https://github.com/opentracing/specification/blob/master/specification.md#extract-a-spancontext-from-a-carrier)を参照してください。

<!--
## Tracer Shim
-->

## Tracer Shim

<!--
### Inject
-->

### 注入(Inject)

<!--
Parameters:
-->

パラメータ:

<!--
- A `SpanContext`.
- A `Format` descriptor.
- A carrier.
-->

- `SpanContext`.
- `Format` の記述子
- キャリア

<!--
Inject the underlying OpenTelemetry `Span` and `Bagagge` using either the explicitly
registered or the global OpenTelemetry `Propagator`s, as configured at construction time.
-->

構築時に設定されたとおり、明示的に登録された、あるいはグローバルなOpenTelemetryの`Propagator`を使って、基礎となるOpenTelemetryの`Span`と`Bagagge`を注入します。

<!--
- `TextMap` and `HttpHeaders` formats MUST use their explicitly specified `TextMapPropagator`,
  if any, or else use the global `TextMapPropagator`.
-->

- `TextMap` と `HttpHeaders` のフォーマットは、明示的に指定された `TextMapPropagator` があればそれを使わなければならない(MUST)し、なければグローバルな `TextMapPropagator` を使わなければなりません(MUST)。

<!--
Errors MAY be raised if the specified `Format` is not recognized, depending
on the specific OpenTracing Language API (e.g. Go and Python do, but Java may not).
-->

指定された `Format` が認識されない場合、特定の OpenTracing 言語 API に応じて、エラーを発生させてもかまいません(MAY) (例えば、Go と Python は認識されますが、Java は認識されない場合があります)。

<!--
### Extract
-->

### 抽出(Extract)

<!--
Parameters:
-->

パラメータ:

<!--
- A `Format` descriptor.
- A carrier.
-->

- Format`の記述子
- キャリア

<!--
Extract the underlying OpenTelemetry `Span` and `Bagagge` using either the explicitly
registered or the global OpenTelemetry `Propagator`s, as configured at construction time.
-->

構築時に設定されたように、明示的に登録された、あるいはグローバルなOpenTelemetryの`Propagator`を使って、基礎となっているOpenTelemetryの`Span`と`Bagagge`を抽出します。

<!--
- `TextMap` and `HttpHeaders` formats MUST use their explicitly specified `TextMapPropagator`,
  if any, or else use the global `TextMapPropagator`.
-->

- `TextMap` と `HttpHeaders` のフォーマットは、明示的に指定された `TextMapPropagator` があればそれを使わなければならない(MUST)し、なければグローバルな `TextMapPropagator` を使わなければなりません(MUST)。

<!--
Returns a `SpanContext` Shim with the underlying extracted OpenTelemetry
`Span` and `Baggage`. Errors MAY be raised if either the `Format` is not recognized
or no value could be extracted, depending on the specific OpenTracing Language API
(e.g. Go and Python do, but Java may not).
-->

基礎となる抽出されたOpenTelemetryの `Span` と `Baggage` を持つ `SpanContext` Shim を返します。特定の OpenTracing 言語 API に応じて、`Format` が認識されなかったり、値が抽出されなかったりした場合には、エラーを発生させてもかまいません(MAY) (例: Go と Python は認識されますが、Java は認識されない場合があります)。

<!--
## OpenTelemetry Span and SpanContext Shim relationship
-->

## OpenTelemetry SpanとSpanContext Shimの関係

<!--
OpenTracing `SpanContext`, just as OpenTelemetry `SpanContext`, MUST be
immutable, but it MUST also store `Baggage`. Hence, it MUST be replaced
every time baggage is updated through the OpenTracing
[Span Set Baggage Item](#set-baggage-item) operation. Special handling
MUST be done by the Shim layer in order to retain these invariants.
-->

OpenTracingの`SpanContext`は、OpenTelemetryの`SpanContext`と同様に、不変でなければなりません(MUST)が、`Baggage`も保存しなければなりません(MUST)。そのため、OpenTracingの[SpanによるBaggage Itemの設定](#set-baggage-item)操作でBaggageが更新されるたびに、置き換えられなければなりません(MUST)。これらの不変性を保持するためには、特別な処理がShimレイヤーによって行われなければなりません(MUST)。

<!--
Because of the previous requirement, a given OpenTelemetry `Span`
MUST be associated with ONE AND ONLY ONE `SpanContext` Shim object at all times
for ALL execution units, in order to keep any linked `Baggage` consistent
at all times. It MUST BE safe to get and set the associated `SpanContext` Shim
object for a specified OpenTelemetry `Span` from different execution units.
-->

前述の要件により、リンクされた `Baggage` の一貫性を常に保つために、すべての実行ユニットにおいて、指定された OpenTelemetry `Span` は"常に1つだけ"の `SpanContext` Shim オブジェクトに関連付けられなければなりません。指定された OpenTelemetry `Span` に対して、異なる実行ユニットから関連する `SpanContext` Shim オブジェクトを取得したり設定したりしても安全でなければなりません(MUST)。

<!--
An example showing the need for these requirements is having an OpenTracing `Span`
have its [Set Baggage Item](#set-baggage-item) operation called from two different
execution units (e.g. threads, coroutines), and afterwards have its
[Context](#get-context) fetched in order to
[iterate over its baggage values](#get-baggage-items).
-->

これらの要件の必要性を示す例として、OpenTracingの`Span`が2つの異なる実行ユニット(例：スレッドやコルーチン)から[Baggage Itemの設定](#set-baggage-item)操作を呼び出し、その後、[Context](#get-context)を取得して[Baggage values](#get-baggage-items)を反復処理することが挙げられます。

<!--
```java
// Thread A: New SpanContextShim and Baggage values are created.
openTracingSpan.setBaggageItem("1", "a")

// Thread B: New SpanContextShim and Baggage values are created again.
openTracingSpan.setBaggageItem("2", "b")

// Thread C: Up-to-date SpanContextShim and Bagggage values are retrieved.
for (Map.Entry<String, String> entry : openTracingSpan.context().baggageItems()) {
  ...
}
```
-->

```java
// スレッド A: 新しいSpanContextShimとBaggageの値が作成されます。
openTracingSpan.setBaggageItem("1", "a")

// スレッド B: 新しいSpanContextShimとBaggageの値が再び作成されます。
openTracingSpan.setBaggageItem("2", "b")

// スレッド C: 最新のSpanContextShimとBagggageの値を取得します。
for (Map.Entry<String, String> entry : openTracingSpan.context().baggageItems()) {
  ...
}
```

<!--
This is a simple graphical representation of the mentioned objects:
-->

これは、言及されたオブジェクトの簡単な図解です。

<!--
```
  Span Shim
  +- OpenTelemetry Span
  +- SpanContext Shim
        +- OpenTelemetry SpanContext
        +- OpenTelemetry Baggage
```
-->

```
  Span Shim
  +- OpenTelemetry Span
  +- SpanContext Shim
        +- OpenTelemetry SpanContext
        +- OpenTelemetry Baggage
```

<!--
The OpenTelemetry `Span` in the `Span` Shim object is used to get and set
its currently associated `SpanContext` Shim.
-->

`Span` ShimオブジェクトのOpenTelemetry `Span`は、現在関連付けられている`SpanContext` Shimの取得と設定に使用されます。

<!--
Managing this one-to-one relationship between an OpenTelemetry `Span` and
a `SpanContext` Shim object is an implementation detail. It can be implemented,
for example, with the help of a global synchronized dictionary, or with an
additional attribute in each OpenTelemetry `Span` object for dynamic languages.
-->

OpenTelemetryの`Span`と`SpanContext`のShimオブジェクトの間の1対1の関係を管理することは、実装の詳細です。例えば、グローバルな同期辞書を使って実装することもできますし、動的な言語の場合は各OpenTelemetry `Span` オブジェクトに追加の属性を持たせることもできます。

<!--
## Span Shim
-->

## Span Shim

<!--
The OpenTracing `Span` operations MUST be implemented using underlying OpenTelemetry `Span`
and `Baggage` values with the help of a `SpanContext` Shim object.
-->

OpenTracingの`Span`操作は、`SpanContext` Shimオブジェクトを使って、基礎となるOpenTelemetryの`Span`と`Baggage`の値を使って実装しなければなりません(MUST)。

<!--
The `Log` operations MUST be implemented using the OpenTelemetry
`Span`'s `Add Events` operations.
-->

`Log` 操作は、OpenTelemetry `Span` の `Add Events` 操作を使って実装しなければなりません(MUST)。

<!--
The `Set Tag` operations MUST be implemented using the OpenTelemetry
`Span`'s `Set Attributes` operations.
-->

`Set Tag`操作は、OpenTelemetry `Span` の「属性の設定」操作を使って実装しなければなりません(MUST)。

<!--
### Get Context
-->

### Contextの取得

<!--
Returns the [associated](#opentelemetry-span-and-spancontext-shim-relationship)
`SpanContext` Shim.
-->

[関連する](#opentelemetry-spanとspancontext-shimの関係) `SpanContext` Shim を返します。


<!--
### Get Baggage Item
-->

### Baggage Itemの取得

<!--
Parameters:
-->

パラメータ:

<!--
- The baggage key, a string.
-->

- 文字列のbaggageキー

<!--
Returns a value for the specified key in the OpenTelemetry `Baggage` of the
associated `SpanContext` Shim or null if none exists.
-->

関連する `SpanContext` Shim の OpenTelemetry `Baggage` 内の指定されたキーの値を返し、存在しない場合は null を返します。

<!--
This is accomplished by getting the
[associated](#opentelemetry-span-and-spancontext-shim-relationship)
`SpanContext` Shim and do a lookup for the specified key in the OpenTelemetry
`Baggage` instance.
-->

これは、[関連する](#opentelemetry-spanとspancontext-shimの関係) `SpanContext` Shim を取得して、OpenTelemetry `Baggage` インスタンスで指定されたキーの検索を行うことで実現されます。

<!--
```java
String getBaggageItem(String key) {
  getSpanContextShim().getBaggage().getEntryValue(key);
}
```
-->

```java
String getBaggageItem(String key) {
  getSpanContextShim().getBaggage().getEntryValue(key);
}
```

<!--
### Set Baggage Item
-->

### Baggage Itemの設定

<!--
Parameters:
-->

パラメータ:

<!--
- The baggage key, a string.
- The baggage value, a string.
-->

- baggage キー。文字列です
- baggage の値。文字列です。

<!--
Creates a new `SpanContext` Shim with a new OpenTelemetry `Baggage` containing
the specified `Baggage` key/value pair. The resulting `SpanContext` Shim is then
[associated](#opentelemetry-span-and-spancontext-shim-relationship) to the underlying
OpenTelemetry `Span`.
-->

指定された `Baggage` のキーと値のペアを含む新しい OpenTelemetry `Baggage` を持つ、新しい `SpanContext` Shim を作成します。作成された `SpanContext` Shim は、基礎となる OpenTelemetry `Span` に [関連付け](#opentelemetry-spanとspancontext-shimの関係) されます。

<!--
```java
void setBaggageItem(String key, String value) {
  SpanContextShim spanContextShim = getSpanContextShim();

  // Add value/key to the existing Baggage.
  Baggage newBaggage = spanContextShim.getBaggage().toBuilder()
    .put(key, value)
    .build();

  // Set a new SpanContext Shim object with the updated Baggage.
  setSpanContextShim(spanContextShim.newWithBaggage(newBaggage));
}
```
-->

```java
void setBaggageItem(String key, String value) {
  SpanContextShim spanContextShim = getSpanContextShim();

  // 既存のBaggageにkey/valueを加えます。
  Baggage newBaggage = spanContextShim.getBaggage().toBuilder()
    .put(key, value)
    .build();

  // 更新されたBaggageを持つ新しいSpanContext Shimオブジェクトを設定します。
  setSpanContextShim(spanContextShim.newWithBaggage(newBaggage));
}
```

<!--
### Set Tag
-->

### Tagの設定

<!--
Parameters:
-->

パラメータ:

<!--
- The tag key, a string.
- The tag value, which must be either a string, a boolean value, or a numeric type.
-->

- tag キー。文字列です。
- tag の値。これは、文字列、真偽値、数値のいずれかでなければなりません。

<!--
Calls `Set Attribute` on the underlying OpenTelemetry `Span` with the specified
key/value pair.
-->

指定されたキーと値のペアで、基礎となるOpenTelemetry `Span` の `Set Attribute` を呼び出します。

<!--
Certain values MUST be mapped from
[OpenTracing Span Tags](https://github.com/opentracing/specification/blob/master/semantic_conventions.md#standard-span-tags-and-log-fields)
to the respective OpenTelemetry `Attribute`:
-->

特定の値は[OpenTracing Span Tags](https://github.com/opentracing/specification/blob/master/semantic_conventions.md#standard-span-tags-and-log-fields)からそれぞれのOpenTelemetryの`Attribute`にマッピングされなければなりません(MUST)。

<!--
- `error` maps to [StatusCode](../trace/api.md##set-status):
  - `true` maps to `Error`.
  - `false` maps to `Ok`.
  - no value being set maps to `Unset`.
-->

- `error` は [StatusCode](../trace/api.md##set-status) に対応します。
  - `true` は `Error` に対応します。
  - `false` は `Ok` に対応します。
  - 設定されていない値 は `Unset` に対応します。

<!--
If the type of the specified value is not supported by the OTel API, the value
MUST be converted to a string.
-->

指定された値のタイプがOTel APIでサポートされていない場合、その値は文字列に変換されなければなりません(MUST)。

<!--
### Log
-->

### ログ

<!--
Parameters:
-->

パラメータ:

<!--
- A set of key/value pairs, where keys must be strings, and the values may have
  any type.
-->

- キーと値のペアのセットで、キーは文字列でなければならず、値は任意の型を持つことができます。

<!--
Calls `Add Events` on the underlying OpenTelemetry `Span` with the specified
key/value pair set.
-->

指定されたキーと値のペアセットを持つOpenTelemetryの`Span`に対して、`Add Events`を呼び出します。

<!--
The `Add Event`'s `name` parameter MUST be the value with the `event` key in
the pair set, or else fallback to use the `log` literal string.
-->

`Add Events`の `name` パラメータには、ペアセットの `event` キーの値を指定しなければなりません(MUST)。

<!--
If pair set contains a `event=error` entry, the values MUST be mapped from
[OpenTracing Log Fields](https://github.com/opentracing/specification/blob/master/semantic_conventionsmd#log-fields-table)
to an `Event` with the conventions outlined in the
[Exception semantic conventions](../trace/semantic_conventions/exceptions.md) document:
-->

ペアセットに `event=error` エントリが含まれている場合、その値は [OpenTracing ログフィールド](https://github.com/opentracing/specification/blob/master/semantic_conventionsmd#log-fields-table) から `Event` に [例外のセマンティック規約](../trace/semantic_conventions/exceptions.md) ドキュメントで概説されている規約に基づいてマッピングされなければなりません (MUST)。

<!--
- If an entry with `error.object` key exists and the value is a language-specific
  error object, a call to `RecordException(e)` is performed along the rest of
  the specified key/value pair set as additional event attributes.
- Else, a call to `AddEvent` is performed with `name` being set to `exception`,
  along the specified key/value pair set as additional event attributes,
  including mapping of the following key/value pairs:
  - `error.kind` maps to `exception.type`.
  - `message` maps to `exception.message`.
  - `stack` maps to `exception.stacktrace`.
-->

- `error.object`キーを持つエントリが存在し、その値が言語固有のエラーオブジェクトである場合、指定されたキーと値のペアの残りの部分が、追加のイベント属性として渡されて、`RecordException(e)`の呼び出しが実行されます。

- また、`name` を `exception` に設定し、`AddEvent` の呼び出しが実行されます。指定されたキー/値のペアを追加のイベント属性として渡されます。その時、以下のキー/値のペアのマッピングを含みます。
  - `error.kinds` は `exception.type` に対応します。
  - `message` は `exception.message` に対応します。
  - `stack` は `exception.stacktrace` に対応します。

<!--
If an explicit timestamp is specified, a conversion MUST be done to match the
OpenTracing and OpenTelemetry units.
-->

明示的なタイムスタンプが指定された場合、OpenTracingおよびOpenTelemetryの単位に合わせて変換が行われなければなりません(MUST)。

<!--
### Finish
-->

### Finish

<!--
Calls `End` on the underlying OpenTelemetry `Span`.
-->

基礎となるOpenTelemetryの`Span`に対して`End`を呼び出します。

<!--
If an explicit timestamp is specified, a conversion MUST be done to match the
OpenTracing and OpenTelemetry units.
-->

明示的なタイムスタンプが指定された場合、OpenTracingおよびOpenTelemetryの単位に合わせて変換が行われなければなりません(MUST)。

<!--
## SpanContext Shim
-->

## SpanContext Shim

<!--
`SpanContext` Shim MUST be immutable and MUST contain the associated
`SpanContext` and `Baggage` values.
-->

`SpanContext` Shim は不変でなければならず(MUST)、関連する `SpanContext` と `Baggage` の値を含まなければなりません(MUST)。

<!--
### Get Baggage Items
-->

### 複数のBaggage Itemの取得

<!--
Returns a dictionary, collection, or iterator (depending on the requirements of
the OpenTracing API for a specific language) backed by the associated OpenTelemetry
`Baggage` values.
-->

関連するOpenTelemetry `Baggage` の値でバックアップされた辞書、コレクション、またはイテレータ(特定の言語のOpenTracing APIの要件に依存)を返します。

<!--
## ScopeManager Shim
-->

## ScopeManager Shim

<!--
For OpenTracing languages implementing the `ScopeManager` interface, its operations
MUST be implemented using the OpenTelemetry Context Propagation API in order
to get and set active `Context` instances.
-->

`ScopeManager`インターフェースを実装しているOpenTracing言語では、アクティブな`Context`インスタンスを取得したり設定したりするために、OpenTelemetry Context Propagation APIを使ってその操作を実装しなければなりません(MUST)。

<!--
### Activate a Span
-->

### Spanの有効化

<!--
Parameters:
-->

パラメータ:

<!--
- A `Span`.
-->

- `Span`

<!--
Gets the [associated](#opentelemetry-span-and-spancontext-shim-relationship)
`SpanContext` Shim for the specified `Span` and puts its OpenTelemetry
`Span`, `Baggage` and `Span` Shim objects in a new `Context`,
which is then set as the currently active instance.
-->

指定された `Span` に対応する [関連する](#opentelemetry-spanとspancontext-shimの関係) `SpanContext` Shim を取得し、その OpenTelemetry `Span`, `Baggage`, `Span` Shim オブジェクトを新しい `Context` に配置し、それを現在のアクティブなインスタンスとして設定します。

<!--
```java
Scope activate(Span span) {
  SpanContextShim spanContextShim = getSpanContextShim(span);

  // Put the associated Span and Baggage in the used Context.
  Context context = Context.current()
    .withValue(spanContextShim.getSpan())
    .withValue(spanContextShim.getBaggage())
    .withValue((SpanShim)spanShim);

  // Set context as the current instance.
  return context.makeCurrent();
}
```
-->

```java
Scope activate(Span span) {
  SpanContextShim spanContextShim = getSpanContextShim(span);

  // Put the associated Span and Baggage in the used Context.
  Context context = Context.current()
    .withValue(spanContextShim.getSpan())
    .withValue(spanContextShim.getBaggage())
    .withValue((SpanShim)spanShim);

  // コンテキストを現在のインスタンスに設定します。
  return context.makeCurrent();
}
```

<!--
### Get the active Span
-->

### 有効なSpanの取得

<!--
Returns a `Span` Shim wrapping the currently active OpenTelemetry `Span`.
-->

現在アクティブなOpenTelemetryの`Span`をラップした`Span` Shimを返します。

<!--
If there are related OpenTelemetry `Span` and `Span` Shim objects in the
current `Context`, the `Span` Shim MUST be returned. Else, a new `Span` Shim
referencing the OpenTelemetry `Span` MUST be created and returned.
-->

現在の `Context` に関連する OpenTelemetry `Span` と `Span` Shim オブジェクトがある場合は、`Span` Shim を返さなければなりません(MUST)。そうでなければ、OpenTelemetry `Span` を参照する新しい `Span` Shim が作成され、返されなければなりません(MUST)。

<!--
The API MUST return null if no actual OpenTelemetry `Span` is set.
-->

実際のOpenTelemetryの`Span`が設定されていない場合、APIはnullを返さなければなりません(MUST)。

<!--
```java
Span active() {
  io.opentelemetry.api.trace.Span span = Span.fromContext(Context.current());
  SpanShim spanShim = SpanShim.fromContext(Context.current());

  // Span was activated through the Shim layer, re-use it.
  if (spanShim != null && spanShim.getSpan() == span) {
    return spanShim;
  }

  // Span was NOT activated through the Shim layer.
  new SpanShim(Span.current());
}
```
-->

```java
Span active() {
  io.opentelemetry.api.trace.Span span = Span.fromContext(Context.current());
  SpanShim spanShim = SpanShim.fromContext(Context.current());

  // Span was activated through the Shim layer, re-use it.
  if (spanShim != null && spanShim.getSpan() == span) {
    return spanShim;
  }

  // SpanはShim layerを介してアクティブにされたものではない。
  new SpanShim(Span.current());
}
```