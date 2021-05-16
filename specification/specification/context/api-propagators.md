<!--
# Propagators API
-->

# Propagator API

**Status**: [Stable, Feature-Freeze](../document-status.md)

<details>
<summary>
目次
</summary>

<!--
- [Overview](#overview)
- [Propagator Types](#propagator-types)
  * [Carrier](#carrier)
  * [Operations](#operations)
    + [Inject](#inject)
    + [Extract](#extract)
- [TextMap Propagator](#textmap-propagator)
  * [Fields](#fields)
  * [TextMap Inject](#textmap-inject)
    + [Setter argument](#setter-argument)
      - [Set](#set)
  * [TextMap Extract](#textmap-extract)
    + [Getter argument](#getter-argument)
      - [Keys](#keys)
      - [Get](#get)
- [InjectorsとExtractorsを別々のインターフェースに](#InjectorsとExtractorsを別々のインターフェースに)
- [Composite Propagator](#composite-propagator)
  * [Create a Composite Propagator](#create-a-composite-propagator)
  * [Composite Extract](#composite-extract)
  * [Composite Inject](#composite-inject)
- [Global Propagators](#global-propagators)
  * [Get Global Propagator](#get-global-propagator)
  * [Set Global Propagator](#set-global-propagator)
- [Propagators Distribution](#propagators-distribution)
  * [B3 Requirements](#b3-requirements)
    + [B3 Extract](#b3-extract)
    + [B3 Inject](#b3-inject)
-->

- [概要](#概要)
- [Propagatorの種類](#Propagatorの種類)
  * [キャリア](#キャリア)
  * [操作](#操作)
    + [注入(Inject)](#注入_inject)
    + [抽出(Extract)](#抽出_extract)
- [TextMap Propagator](#textmap-propagator)
  * [フィールド](#フィールド)
  * [TextMap Inject](#textmap-inject)
    + [Setter引数](#setter引数)
      - [Set](#set)
  * [TextMap Extract](#textmap-extract)
    + [Getter引数](#getter引数)
      - [Keys](#keys)
      - [Get](#get)
- [Injectors and Extractors as Separate Interfaces](#injectors-and-extractors-as-separate-interfaces)
- [複合Propagator](#複合Propagator)
  * [Create a Composite Propagator](#create-a-composite-propagator)
  * [Composite Extract](#composite-extract)
  * [Composite Inject](#composite-inject)
- [Global Propagators](#global-propagators)
  * [Get Global Propagator](#get-global-propagator)
  * [Set Global Propagator](#set-global-propagator)
- [Propagators Distribution](#propagators-distribution)
  * [B3 Requirements](#b3-requirements)
    + [B3 Extract](#b3-extract)
    + [B3 Inject](#b3-inject)

</details>

<!--
## Overview
-->

## 概要

<!--
Cross-cutting concerns send their state to the next process using
`Propagator`s, which are defined as objects used to read and write
context data to and from messages exchanged by the applications.
Each concern creates a set of `Propagator`s for every supported
`Propagator` type.
-->

横断的な関心事は、アプリケーションによって交換されるメッセージとの間でコンテキストデータを読み書きするために使用されるオブジェクトとして定義されている `Propagator` を使用して、次のプロセスに状態を送信します。各コンサーンは、サポートされているすべての `Propagator` タイプの `Propagator` のセットを作成します。

<!--
`Propagator`s leverage the `Context` to inject and extract data for each
cross-cutting concern, such as traces and `Baggage`.
-->

`Propagator`は、`Context`を活用して、Traceや`Baggage`などの横断的な関心事にデータを注入・抽出します。

<!--
Propagation is usually implemented via a cooperation of library-specific request
interceptors and `Propagators`, where the interceptors detect incoming and outgoing requests and use the `Propagator`'s extract and inject operations respectively.
-->

伝搬は通常、ライブラリ固有のリクエストインターセプターと `Propagator` の連携によって実装されます。インターセプターは受信および送信されるリクエストを検出し、`Propagator` の 注入 および 抽出操作をそれぞれ使用します。

<!--
The Propagators API is expected to be leveraged by users writing
instrumentation libraries.
-->

Propagators APIは、計装ライブラリを作成するユーザーが活用することを想定しています。

<!--
## Propagator Types
-->

## Propagatorの種類

<!--
A `Propagator` type defines the restrictions imposed by a specific transport
and is bound to a data type, in order to propagate in-band context data across process boundaries.
-->

`Propagator` タイプは、受信したContextのデータをプロセスの境界を越えて伝搬させるために、特定のトランスポートによって課される制限を定義し、データ型にバインドされます。

<!--
The Propagators API currently defines one `Propagator` type:
-->

Propagators APIでは現在、1つの `Propagator` タイプが定義されています。

<!--
- `TextMapPropagator` is a type that inject values into and extracts values
  from carriers as string key/value pairs.
-->

- `TextMapPropagator` は、文字列のキーと値のペアとしてキャリアに値を注入したり、キャリアから値を抽出したりします。

<!--
A binary `Propagator` type will be added in the future (see [#437](https://github.com/open-telemetry/opentelemetry-specification/issues/437)).
-->

将来的には、バイナリの`Propagator` 型が追加される予定です([#437](https://github.com/open-telemetry/opentelemetry-specification/issues/437)参照)。

<!--
### Carrier
-->

### キャリア

<!--
A carrier is the medium used by `Propagator`s to read values from and write values to.
Each specific `Propagator` type defines its expected carrier type, such as a string map
or a byte array.
-->

キャリアとは `Propagator` が値を読み書きする際に使用する媒体のことです。特定の `Propagator` 型は文字列のマップやバイト配列のような期待されるキャリアタイプを定義します。

<!--
Carriers used at [Inject](#inject) are expected to be mutable.
-->

[Inject](#inject)で使用されるキャリアは変更可能であることが期待されます。

<!--
### Operations
-->

### 操作

<!--
`Propagator`s MUST define `Inject` and `Extract` operations, in order to write
values to and read values from carriers respectively. Each `Propagator` type MUST define the specific carrier type
and MAY define additional parameters.
-->

`Propagator` は、キャリアに値を書き込んだり、キャリアから値を読み込んだりするために、`Inject` と `Extract` という操作を定義しなければなりません (MUST)。各 `Propagator` タイプは、特定のキャリアタイプを定義しなければなりません (MUST)。また、追加のパラメータを定義しても構いません (MAY)。

<!--
#### Inject
-->

#### 注入(Inject)

<!--
Injects the value into a carrier. For example, into the headers of an HTTP request.
-->

値をキャリアに注入します。例えば、HTTPリクエストのヘッダーに注入します。

<!--
Required arguments:
-->

必須引数:

<!--
- A `Context`. The Propagator MUST retrieve the appropriate value from the `Context` first, such as
`SpanContext`, `Baggage` or another cross-cutting concern context.
- The carrier that holds the propagation fields. For example, an outgoing message or HTTP request.
-->

- `Context`。Propagator は最初に `Context` から適切な値を取得しなければなりません (例: `SpanContext`, `Baggage` または他の横断的な関心事のContextなど)。
- 伝搬フィールドを保持するキャリア。例えば、送信メッセージやHTTPリクエストなどです。

<!--
#### Extract
-->

#### 抽出(Extract)

<!--
Extracts the value from an incoming request. For example, from the headers of an HTTP request.
-->

受信したリクエストから値を抽出します。例えば、HTTPリクエストのヘッダーから値を抽出します。

<!--
If a value can not be parsed from the carrier, for a cross-cutting concern,
the implementation MUST NOT throw an exception and MUST NOT store a new value in the `Context`,
in order to preserve any previously existing valid value.
-->

キャリアから値を抽出できない場合、横断的な関心事のために、実装は例外を投げてはならず(MUST NOT)、`Context`に新しい値を保存してはなりません(MUST NOT)。これは以前に存在した有効な値を保存するためです。

<!--
Required arguments:
-->

必須引数:

<!--
- A `Context`.
- The carrier that holds the propagation fields. For example, an incoming message or http response.
-->

- `Context`
- 伝搬フィールドを保持するキャリア。例えば、受信メッセージやhttpのレスポンスなどです。

<!--
Returns a new `Context` derived from the `Context` passed as argument,
containing the extracted value, which can be a `SpanContext`,
`Baggage` or another cross-cutting concern context.
-->

抽出された値を含む、引数として渡された `Context` から派生した新しい `Context` を返します。`Context` は `SpanContext` や `Baggage` などの横断的な関心事のContextになります。

<!--
## TextMap Propagator
-->

## TextMap Propagator

<!--
`TextMapPropagator` performs the injection and extraction of a cross-cutting concern
value as string key/values pairs into carriers that travel in-band across process boundaries.
-->

`TextMapPropagator` は、横断的な関心事内の値を、文字列のキーと値のペアとして、プロセスの境界を越えて内部(in-band)で移動するキャリアに注入したり抽出したりします。

<!--
The carrier of propagated data on both the client (injector) and server (extractor) side is
usually an HTTP request.
-->

クライアント(Injector)とサーバー(Extractor)の両方に伝搬するデータのキャリアは、通常、HTTPリクエストです。

<!--
In order to increase compatibility, the key/value pairs MUST only consist of US-ASCII characters
that make up valid HTTP header fields as per [RFC 7230](https://tools.ietf.org/html/rfc7230#section-3.2).
-->

互換性を高めるために、キーと値のペアは、[RFC 7230](https://tools.ietf.org/html/rfc7230#section-3.2)に準拠した有効なHTTPヘッダーフィールドを構成するUS-ASCII文字のみで構成されなければなりません(MUST)。

<!--
`Getter` and `Setter` are optional helper components used for extraction and injection respectively,
and are defined as separate objects from the carrier to avoid runtime allocations,
by removing the need for additional interface-implementing-objects wrapping the carrier in order
to access its contents.
-->
`Getter`と`Setter`は、それぞれ抽出と注入に使用される任意のヘルパーコンポーネントで、ランタイムの割り当てを避けるために、キャリアとは別のオブジェクトとして定義されています。これは、キャリアのコンテンツにアクセスするために、キャリアをラップする追加のインターフェース実装オブジェクトの必要性を排除するためです。(XXX: 一文が長すぎて自信なし)

<!--
`Getter` and `Setter` MUST be stateless and allowed to be saved as constants, in order to effectively
avoid runtime allocations.
-->

実行時の割り当てを効果的に回避するために、`Getter`と`Setter`はステートレスで、定数として保存することができなければなりません(MUST)。

<!--
### Fields
-->

### フィールド

<!--
The predefined propagation fields. If your carrier is reused, you should delete the fields here
before calling [inject](#inject).
-->

定義済みの伝搬フィールドです。キャリアを再利用する場合は、[Inject](#inject)を呼ぶ前に、ここのフィールドを削除する必要があります。

<!--
Fields are defined as string keys identifying format-specific components in a carrier.
-->

フィールドは、キャリア内のフォーマット固有のコンポーネントを識別する文字列キーとして定義されます。

<!--
For example, if the carrier is a single-use or immutable request object, you don't need to
clear fields as they couldn't have been set before. If it is a mutable, retryable object,
successive calls should clear these fields first.
-->

例えば、キャリアが一回だけ使用または不変のリクエストオブジェクトの場合、フィールドは以前に設定されたことがないのでクリアする必要はありません。改変可能(mutable)でリトライ可能なオブジェクトであれば、連続した呼び出しではまずこれらのフィールドをクリアする必要があります。

<!--
The use cases of this are:
-->

使用例:

<!--
- allow pre-allocation of fields, especially in systems like gRPC Metadata
- allow a single-pass over an iterator
-->

- 特にgRPC Metadataのようなシステムでは、フィールドの事前割り当てを可能にします。
- イテレータのシングルパスを可能にします。

<!--
Returns list of fields that will be used by the `TextMapPropagator`.
-->

返り値として、`TextMapPropagator`で使用されるフィールドのリストを返します。

<!--
Observe that some `Propagator`s may define, besides the returned values, additional fields with
variable names. To get a full list of fields for a specific carrier object, use the
[Keys](#keys) operation.
-->

なお、`Propagator`の中には、返り値の他に、変数名で追加のフィールドを定義するものもあります。特定のキャリアオブジェクトのフィールドの全リストを取得するには、[Keys](#keys)操作を使用します。

<!--
### TextMap Inject
-->

### TextMap Inject

<!--
Injects the value into a carrier. The required arguments are the same as defined by
the base [Inject](#inject) operation.
-->

値をキャリアに注入します。必要な引数は、基本の[Inject](#inject)操作で定義されているものと同じです。

<!--
Optional arguments:
-->

任意の引数:

<!--
- A `Setter` to set a propagation key/value pair. Propagators MAY invoke it multiple times in order to set multiple pairs.
  This is an additional argument that languages are free to define to help inject data into the carrier.
-->

- プロパゲーションのキーと値のペアを設定する `Setter`。Propagatorは、複数のペアを設定するために、これを複数回起動しても構いません(MAY)。これは、キャリアにデータを注入するために、言語が自由に定義できる追加の引数です。

<!--
#### Setter argument
-->

#### Setter引数

<!--
Setter is an argument in `Inject` that sets values into given fields.
-->

Setterは `Inject` の引数で、与えられたフィールドに値を設定します。

<!--
`Setter` allows a `TextMapPropagator` to set propagated fields into a carrier.
-->

`Setter` は、`TextMapPropagator` が伝搬したフィールドをキャリアに設定することを可能にします。

<!--
One of the ways to implement it is `Setter` class with `Set` method as described below.
-->

これを実装する方法の一つとして、以下で説明する`Setter`クラスの`Set`メソッドがあります。

<!--
##### Set
-->

##### Set

<!--
Replaces a propagated field with the given value.
-->

伝播されたフィールドを、与えられた値で置き換える。

<!--
Required arguments:
-->

必須引数:

<!--
- the carrier holding the propagation fields. For example, an outgoing message or an HTTP request.
- the key of the field.
- the value of the field.
-->

- 伝搬フィールドを保持するキャリア。例えば、送信メッセージやHTTPリクエストなどです。
- フィールドのキー
- フィールドの値

<!--
The implementation SHOULD preserve casing (e.g. it should not transform `Content-Type` to `content-type`) if the used protocol is case insensitive, otherwise it MUST preserve casing.
-->

実装は、使用されるプロトコルが大文字小文字を区別しない場合は、大文字小文字をそのまま保持するべきです(SHOULD)(例えば、`Content-Type`を`content-type`に変換すべきではありません)。使用されるプロトコルが大文字小文字を区別する場合、大文字小文字をそのまま保持しなければなりません(MUST)。

<!--
### TextMap Extract
-->

### TextMap Extract

<!--
Extracts the value from an incoming request. The required arguments are the same as defined by
the base [Extract](#extract) operation.
-->

受信したリクエストから値を抽出します。必要な引数は、基本の[Extract](#extract)操作で定義されているものと同じです。

<!--
Optional arguments:
-->

任意の引数:

<!--
- A `Getter` invoked for each propagation key to get. This is an additional
  argument that languages are free to define to help extract data from the carrier.
-->

- 取得する伝搬キーごとに呼び出される`Getter`。これは、キャリアからデータを抽出するために、言語が自由に定義できる追加の引数です。

<!--
Returns a new `Context` derived from the `Context` passed as argument.
-->

引数として渡された `Context` から派生した新しい `Context` を返します。

<!--
#### Getter argument
-->

#### Getter 引数

<!--
Getter is an argument in `Extract` that get value from given field
-->

Getterは `Extract` の引数で、与えられたフィールドから値を取得します。

<!--
`Getter` allows a `TextMapPropagator` to read propagated fields from a carrier.
-->

`Getter`は、`TextMapPropagator`がキャリアから伝搬したフィールドを読み取ります。

<!--
One of the ways to implement it is `Getter` class with `Get` and `Keys` methods
as described below. Languages may decide on alternative implementations and
expose corresponding methods as delegates or other ways.
-->

これを実装する方法の一つとして、以下に示すような `Get` と `Keys` のメソッドを持つ `Getter` クラスがあります．言語によっては、別の実装方法を決めて、対応するメソッドをデリゲートやその他の方法で公開することもできます。

<!--
##### Keys
-->

##### Keys

<!--
The `Keys` function MUST return the list of all the keys in the carrier.
-->

`Keys`関数は、キャリアのすべてのキーのリストを返さなければなりません(MUST)。

<!--
Required arguments:
-->

必須引数:

<!--
- The carrier of the propagation fields, such as an HTTP request.
-->

- HTTPリクエストのような、伝搬フィールドのキャリア

<!--
The `Keys` function can be called by `Propagator`s which are using variable key names in order to
iterate over all the keys in the specified carrier.
-->

`Keys`関数は、変数のキー名を使っている`Propagator`が、指定されたキャリアのすべてのキーを反復して調べるために呼び出すことができます。

<!--
For example, it can be used to detect all keys following the `uberctx-{user-defined-key}` pattern, as defined by the
[Jaeger Propagation Format](https://www.jaegertracing.io/docs/1.18/client-libraries/#baggage).
-->

例えば、[Jaeger Propagation Format](https://www.jaegertracing.io/docs/1.18/client-libraries/#baggage)で定義されている`uberctx-{user-defined-key}`パターンに従うすべてのキーを検出するのに使用できます。

<!--
##### Get
-->

##### Get

<!--
The Get function MUST return the first value of the given propagation key or return null if the key doesn't exist.
-->

Get関数は、与えられた伝搬キーの最初の値を返すか、キーが存在しない場合はnullを返さなければなりません(MUST)。

<!--
Required arguments:
-->

必須引数:

<!--
- the carrier of propagation fields, such as an HTTP request.
- the key of the field.
-->

- HTTPリクエストのような伝搬フィールドのキャリア
- フィールドのキー

<!--
The Get function is responsible for handling case sensitivity. If the getter is intended to work with a HTTP request object, the getter MUST be case insensitive.
-->

大文字小文字の区別は、Get関数が責任を持ちます。GetterがHTTPリクエストオブジェクトを扱うことを意図している場合、Getterは大文字と小文字を区別しないものでなければなりません(MUST)。

<!--
## Injectors and Extractors as Separate Interfaces
-->

## InjectorsとExtractorsを別々のインターフェースに分ける

<!--
Languages can choose to implement a `Propagator` type as a single object
exposing `Inject` and `Extract` methods, or they can opt to divide the
responsibilities further into individual `Injector`s and `Extractor`s. A
`Propagator` can be implemented by composing individual `Injector`s and
`Extractors`.
-->

言語は `Propagator` タイプを `Inject` と `Extract` メソッドを公開する単一のオブジェクトとして実装するか、あるいはさらに個々の `Injector` と `Extractor` に責任を分割するかを選ぶことができます。`Propagator`は個々の `Injector` と `Extractor` を組み合わせて実装できます。

<!--
## Composite Propagator
-->

## 複合Propagator

<!--
Implementations MUST offer a facility to group multiple `Propagator`s
from different cross-cutting concerns in order to leverage them as a
single entity.
-->

実装は、異なる横断的な関心事からの複数の`Propagator`をグループ化して、単一のエンティティとして活用するための機能を提供しなければなりません(MUST)。

<!--
A composite propagator can be built from a list of propagators, or a list of
injectors and extractors. The resulting composite `Propagator` will invoke the `Propagator`s, `Injector`s, or `Extractor`s, in the order they were specified.
-->

複合PropagatorはPropagatorのリスト、またはインジェクタとエクストラクタのリストから構築できます。合成された`Propagator`は指定された順に`Propagator`、`Injector`、`Extractor`を起動します。

<!--
Each composite `Propagator` will implement a specific `Propagator` type, such
as `TextMapPropagator`, as different `Propagator` types will likely operate on different
data types.
-->

それぞれの複合`Propagator`は、`TextMapPropagator`のような特定の`Propagator`型を実装します。なぜなら、異なる`Propagator`型は異なるデータ型を操作する可能性があるからです。

<!--
There MUST be functions to accomplish the following operations.
-->

以下の操作を行うための機能がなければなりません(MUST)。

<!--
- Create a composite propagator
- Extract from a composite propagator
- Inject into a composite propagator
-->

- 複合Propagatorの作成
- 複合Propagatorからの抽出
- 複合Propagatorへの注入

<!--
### Create a Composite Propagator
-->

### 複合Propagatorの作成

<!--
Required arguments:
-->

必須引数:

<!--
- A list of `Propagator`s or a list of `Injector`s and `Extractor`s.
-->

- `Propagator`のリスト、または`Injector`と`Extractor`のリスト

<!--
Returns a new composite `Propagator` with the specified `Propagator`s.
-->

指定された `Propagator` を含む新しい `複合Propagator` を返します。

<!--
### Composite Extract
-->

### 複合Propagatorからの抽出

<!--
Required arguments:
-->

必須引数:

<!--
- A `Context`.
- The carrier that holds propagation fields.

If the `TextMapPropagator`'s `Extract` implementation accepts the optional `Getter` argument, the following arguments are REQUIRED, otherwise they are OPTIONAL:

- The instance of `Getter` invoked for each propagation key to get.
-->

- `Context`
- 伝播フィールドを保持するキャリア

`TextMapPropagator`の`Extract`実装がオプションの`Getter`引数を受け入れる場合、以下の引数は必須(REQUIRED)であり、そうでない場合はオプション(OPTIONAL)です。

- 取得する伝搬キーごとに呼び出される`Getter`のインスタンス

<!--
### Composite Inject
-->

### 複合Propagatorへの注入

<!--
Required arguments:
-->

必須引数:

<!--
- A `Context`.
- The carrier that holds propagation fields.

If the `TextMapPropagator`'s `Inject` implementation accepts the optional `Setter` argument, the following arguments are REQUIRED, otherwise they are OPTIONAL:

- The `Setter` to set a propagation key/value pair. Propagators MAY invoke it multiple times in order to set multiple pairs.
-->

- `Context`
- 伝播フィールドを保持するキャリア

`TextMapPropagator`の`Inject`実装がオプションの`Setter`引数を受け入れる場合、以下の引数は必須(REQUIRED)であり、そうでない場合はオプション(OPTIONAL)です。

- プロパゲーションのキーと値のペアを設定するための `Setter`。Propagatorは複数のペアを設定するために、複数回起動してもかまいません(MAY)。

<!--
## Global Propagators
-->

## グローバルPropagator

<!--
The OpenTelemetry API MUST provide a way to obtain a propagator for each
supported `Propagator` type. Instrumentation libraries SHOULD call propagators
to extract and inject the context on all remote calls. Propagators, depending on
the language, MAY be set up using various dependency injection techniques or
available as global accessors.
-->

OpenTelemetry APIは、サポートされている各 `Propagator` タイプのプロパゲータを取得する方法を提供しなければなりません(MUST)。計装ライブラリは、すべてのリモートコールでPropagatorを呼び出してContextを抽出・注入すべき(SHOULD)です。Propagatorは、言語によっては、様々な依存性注入技術を用いて設定してもかまいません(MAY)し、グローバルアクセサとして利用してもかまいません(MAY)。

<!--
**Note:** It is a discouraged practice, but certain instrumentation libraries
might use proprietary context propagation protocols or be hardcoded to use a
specific one. In such cases, instrumentation libraries MAY choose not to use the
API-provided propagators and instead hardcode the context extraction and injection
logic.
-->

**注釈:** これは推奨されない方法ですが、特定の計装ライブラリは、独自のContext伝搬プロトコルを使用したり、特定のプロトコルを使用するようにハードコードされている場合があります。このような場合、計装ライブラリは、APIが提供するPropagatorを使用せず、代わりにContext抽出とインジェクション・ロジックをハードコードしてもかまいません(MAY)。

<!--
The OpenTelemetry API MUST use no-op propagators unless explicitly configured
otherwise. Context propagation may be used for various telemetry signals -
traces, metrics, logging and more. Therefore, context propagation MAY be enabled
for any of them independently. For instance, a span exporter may be left
unconfigured, although the trace context propagation was configured to enrich logs or metrics.
-->

OpenTelemetry API は、他に明示的に設定されていない限り、no-op プロパゲータを使用しなければなりません(MUST)。Contextの伝搬は、Trace、Metrics、Loggingなど、様々なテレメトリシグナルに使用できます。したがって、Contextの伝搬はそれらのどれかに対して独立して有効にしても構いません(MAY)。例えば、ログやメトリクスを豊かにするためにトレースのコンテキスト伝播が設定されているにもかかわらず、Span Exporterが未設定のままになっている場合があります。(XXX)

<!--
Platforms such as ASP.NET may pre-configure out-of-the-box
propagators. If pre-configured, `Propagator`s SHOULD default to a composite
`Propagator` containing the W3C Trace Context Propagator and the Baggage
`Propagator` specified in the [Baggage API](../baggage/api.md#propagation).
These platforms MUST also allow pre-configured propagators to be disabled or overridden.
-->

ASP.NETなどのプラットフォームでは、すぐに使えるPropagatorを事前に設定できます。事前に設定されている場合、`Propagator`のデフォルトは、W3C Trace Context Propagatorと、[Baggage API](../baggage/api.md#propagation)で指定されたBaggage `Propagator`を含む複合`Propagator`にすべきです。これらのプラットフォームでは、事前に設定されたプロパゲータを無効にしたり、オーバーライドしたりすることもできなければなりません(MUST)。

<!--
### Get Global Propagator
-->

### Global Propagatorの取得

<!--
This method MUST exist for each supported `Propagator` type.
-->

このメソッドは、サポートされている各 `Propagator` タイプに対して存在しなければなりません (MUST)。

<!--
Returns a global `Propagator`. This usually will be composite instance.
-->

グローバルな `Propagator` を返します。これは通常、複合インスタンスとなります。

<!--
### Set Global Propagator
-->

### Global Propagatorの設定

<!--
This method MUST exist for each supported `Propagator` type.
-->

このメソッドは、サポートされている各 `Propagator` タイプに対して存在しなければなりません (MUST)。

<!--
Sets the global `Propagator` instance.
-->

グローバルな `Propagator` を設定します。

<!--
Required parameters:
-->

必須パラメータ:

<!--
- A `Propagator`. This usually will be a composite instance.
-->

- `Propagator`。これは通常、複合インスタンスとなります。

<!--
## Propagators Distribution
-->

## Propagatorの配布

<!--
The official list of propagators that MUST be maintained by the OpenTelemetry
organization and MUST be distributed as OpenTelemetry extension packages:
-->

The official list of propagators that MUST be maintained by the OpenTelemetry organization and MUST be distributed as OpenTelemetry extension packages:

Propagatorの公式リストはOpenTelemetry組織によって維持されなければなりません(MUST)し、、OpenTelemetry拡張パッケージとして配布されなければなりません(MUST)。


<!--
* [W3C TraceContext](https://www.w3.org/TR/trace-context/). MAY alternatively
  be distributed as part of the OpenTelemetry API.
* [W3C Baggage](https://w3c.github.io/baggage). MAY alternatively
  be distributed as part of the OpenTelemetry API.
* [B3](https://github.com/openzipkin/b3-propagation).
* [Jaeger](https://www.jaegertracing.io/docs/latest/client-libraries/#propagation-format).
-->

* [W3C TraceContext](https://www.w3.org/TR/trace-context/). 代わりにOpenTelemetry APIの一部として配布しても構いません(MAY)
* [W3C Baggage](https://w3c.github.io/baggage). 代わりにOpenTelemetry APIの一部として配布しても構いません(MAY)
* [B3](https://github.com/openzipkin/b3-propagation)
* [Jaeger](https://www.jaegertracing.io/docs/latest/client-libraries/#propagation-format)

<!--
This is a list of additional propagators that MAY be maintained and distributed
as OpenTelemetry extension packages:
-->

これは、OpenTelemetryの拡張パッケージとして維持・配布してもよい(MAY)、追加のプロパゲータのリストです。

<!--
* [OT Trace](https://github.com/opentracing?q=basic&type=&language=). Propagation format
  used by the OpenTracing Basic Tracers. It MUST NOT use `OpenTracing` in the resulting
  propagator name as it is not widely adopted format in the OpenTracing ecosystem.
-->

* [OT Trace](https://github.com/opentracing?q=basic&type=&language=)。OpenTracing Basic Tracers で使用されるプロパゲーションフォーマットです。OpenTracing のエコシステムでは広く採用されていないフォーマットであるため、結果として得られるプロパゲータ名に `OpenTracing` を使用してはいけません (MUST NOT)。

<!--
Additional `Propagator`s implementing vendor-specific protocols such as AWS
X-Ray trace header protocol MUST NOT be maintained or distributed as part of
the Core OpenTelemetry repositories.
-->

AWS X-Rayトレースヘッダプロトコルのようなベンダー固有のプロトコルを実装する追加の`Propagator`は、Core OpenTelemetryのリポジトリの一部として維持・配布してはなりません(MUST NOT)。

<!--
### B3 Requirements
-->

### B3 要件

<!--
B3 has both single and multi-header encodings. It also has semantics that do not
map directly to OpenTelemetry such as a debug trace flag, and allowing spans
from both sides of request to share the same id. To maximize compatibility
between OpenTelemetry and Zipkin implementations, the following guidelines have
been established for B3 context propagation.
-->

B3には、シングルヘッダーとマルチヘッダーの両方のエンコーディングがあります。また、デバッグトレースフラグや、リクエストの両サイドからのSpanが同じIDを共有することを許可するなど、OpenTelemetryには直接対応しないセマンティクスを持っています。OpenTelemetry と Zipkin の実装間の互換性を最大限に高めるために、B3 Contextの伝搬には以下のガイドラインが設定されています。

<!--
#### B3 Extract
-->

#### B3 抽出

<!--
When extracting B3, propagators:
-->

B3を抽出する際、Propagatorは:


<!--
* MUST attempt to extract B3 encoded using single and multi-header
  formats. The single-header variant takes precedence over
  the multi-header version.
* MUST preserve a debug trace flag, if received, and propagate
  it with subsequent requests. Additionally, an OpenTelemetry implementation
  MUST set the sampled trace flag when the debug flag is set.
* MUST NOT reuse `X-B3-SpanId` as the id for the server-side span.
-->

* シングルヘッダーとマルチヘッダーでエンコードされたB3の抽出を試みなければなりません(MUST)。シングルヘッダー形式はマルチヘッダー形式よりも優先されます。
* デバッグフラグを保持しなければなりません(MUST)し、設定されている場合は後続のリクエストにも伝搬させなければなりません。さらに、OpenTelemetryの実装ではデバッグフラグが設定されたときにサンプルトレースフラグを設定しなければなりません(MUST)。
* サーバー側のSpanの ID として、`X-B3-SpanId` を再利用してはいけません (MUST NOT)。

<!--
#### B3 Inject
-->

#### B3 注入

<!--
When injecting B3, propagators:
-->

B3を注入する際、Propagatorは:

<!--
* MUST default to injecting B3 using the single-header format
* MUST provide configuration to change the default injection format to B3
  multi-header
* MUST NOT propagate `X-B3-ParentSpanId` as OpenTelemetry does not support
  reusing the same id for both sides of a request.
-->

* デフォルトでは、シングルヘッダー形式を使用してB3を注入しなければならりません(MUST)。
* デフォルトの注入フォーマットをB3マルチヘッダーに変更するための設定を提供しなければなりません(MUST)。
* OpenTelemetry は同じ ID をリクエストの両側で再利用することをサポートしていないため、`X-B3-ParentSpanId` を伝播してはなりません(MUST NOT)

<!--
#### Fields
-->

#### フィールド

<!--
Fields MUST return the header names that correspond to the configured format,
i.e., the headers used for the inject operation.
-->

フィールドは、設定されたフォーマットに対応するヘッダー名、すなわち注入操作に使用されるヘッダーを返さなければなりません(MUST)。

<!--
#### Configuration
-->

#### 設定

<!--
| Option    | Extract Order | Inject Format | Specification     |
|-----------|---------------|---------------| ------------------|
| B3 Single | Single, Multi | Single        | [Link][b3-single] |
| B3 Multi  | Single, Multi | Multi         | [Link][b3-multi]  |
-->

| Option    | Extract Order | Inject フォーマット | 仕様     |
|-----------|---------------|---------------| ------------------|
| B3 Single | Single, Multi | Single        | [Link][b3-single] |
| B3 Multi  | Single, Multi | Multi         | [Link][b3-multi]  |

<!--
[b3-single]: https://github.com/openzipkin/b3-propagation#single-header
[b3-multi]: https://github.com/openzipkin/b3-propagation#multiple-headers
-->

[b3-single]: https://github.com/openzipkin/b3-propagation#single-header
[b3-multi]: https://github.com/openzipkin/b3-propagation#multiple-headers
