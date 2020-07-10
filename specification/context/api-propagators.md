<!--
# Propagators API
-->

# Propagator API

<!--
<details>
<summary>
Table of Contents
</summary>
-->

<details>
<summary>
目次
</summary>

<!--
- [Propagator API](#propagator-api)
  - [概要](#概要)
  - [HTTP テキストフォーマット](#http-テキストフォーマット)
    - [フィールド](#フィールド)
    - [注入(inject)](#注入inject)
      - [Setter引数](#setter引数)
        - [Set](#set)
    - [抽出(Extract)](#抽出extract)
      - [Getter 引数](#getter-引数)
        - [Get](#get)
  - [Injectors and Extractors as Separate Interfaces](#injectors-and-extractors-as-separate-interfaces)
  - [Composite Propagator](#composite-propagator)
    - [Create a Composite Propagator](#create-a-composite-propagator)
    - [Extract](#extract)
    - [Inject](#inject)
  - [Global Propagators](#global-propagators)
    - [Get Global Propagator](#get-global-propagator)
    - [Set Global Propagator](#set-global-propagator)
-->

- [Propagator API](#propagator-api)
  - [概要](#概要)
  - [HTTP テキストフォーマット](#http-テキストフォーマット)
    - [フィールド](#フィールド)
    - [注入(inject)](#注入inject)
      - [Setter引数](#setter引数)
        - [Set](#set)
    - [抽出(Extract)](#抽出extract)
      - [Getter 引数](#getter-引数)
        - [Get](#get)
  - [別々のインターフェースとしての注入と抽出](#別々のインターフェースとしての注入と抽出)
  - [合成Propagator](#合成Propagator)
    - [合成Propagatorの作成](#合成Propagatorの作成)
    - [合成Propagatorからの抽出](#合成Propagatorからの抽出)
    - [合成Propagatorへの注入](#合成Propagatorへの注入)
  - [グローバル Propagator](#グローバル-Propagator)
    - [グローバル Propagatorの取得](#グローバル-Propagatorの取得)
    - [グローバル Propagatorのセット](#グローバル-Propagatorのセット)

<!--
</details>
-->

</details>

<!--
## Overview
-->

## 概要

<!--
Cross-cutting concerns send their state to the next process using
`Propagator`s, which are defined as objects used to read and write
context data to and from messages exchanged by the applications.
Each concern creates a set of `Propagator`s for every supported `Format`.
-->

関連するコンポーネントは自分自身の状態を `Propagator` を使って次のプロセスに送信します。`Propagator`はアプリケーションが交換するメッセージとの間でコンテキストデータを読み書きするためのオブジェクトとして定義されています。各関連するコンポーネントは、サポートされている `Format`全部に対して `Propagator` のセットを作成します。

<!--
Propagators leverage the `Context` to inject and extract data for each
cross-cutting concern, such as traces and correlation context.
-->

Propagatorは `Context` を利用して、関連するコンポーネントそれぞれに対してTraceやCorrelationContextなどのデータを注入したり抽出したりします。

<!--
The Propagators API is expected to be leveraged by users writing
instrumentation libraries.
-->

Propagator APIは計装ライブラリを書いているユーザーが活用することが期待されています。

<!--
The Propagators API currently consists of one `Format`:
-->

Propagator APIは現在1つの `Format` から構成されています。

<!--
- `HTTPTextFormat` is used to inject values into and extract values from carriers as text that travel
  in-band across process boundaries.
-->

- `HTTPTextFormat` はプロセスの境界を越えてインバンドで移動するキャリアに、関連するコンポーネントの値をテキストとして注入・抽出することに使われます。
<!--
A binary `Format` will be added in the future.
-->

将来的にはバイナリの `Format` が追加される予定です。

<!--
## HTTP Text Format
-->

## HTTP テキストフォーマット

<!--
`HTTPTextFormat` is a formatter that injects and extracts a cross-cutting concern
value as text into carriers that travel in-band across process boundaries.
-->

`HTTP テキストフォーマット` は、プロセスの境界を越えてインバンドで移動するキャリアに、関連するコンポーネントの値をテキストとして注入・抽出するフォーマッタです。

<!--
Encoding is expected to conform to the HTTP Header Field semantics. Values are often encoded as
RPC/HTTP request headers.
-->

エンコードは HTTP ヘッダーフィールドのセマンティクスに準拠することが期待されます。値はしばしばRPC/HTTPリクエストヘッダとしてエンコードされます。

<!--
The carrier of propagated data on both the client (injector) and server (extractor) side is
usually an http request. Propagation is usually implemented via library-specific request
interceptors, where the client-side injects values and the server-side extracts them.
-->

クライアント側(注入側)とサーバ側(抽出側)の両方で伝搬されるデータのキャリアは、通常はHTTPリクエストです。伝搬は通常、ライブラリ固有のリクエストインターセプターを介して実装され、 クライアント側が値を注入し、サーバ側が値を抽出します。


<!--
`HTTPTextFormat` MUST expose the APIs that injects values into carriers,
and extracts values from carriers.
-->

`HTTPTextFormat` はキャリアに値を注入し、キャリアから値を抽出するAPIを公開しなければなりません(MUST)。

<!--
### Fields
-->

### フィールド

<!--
The propagation fields defined. If your carrier is reused, you should delete the fields here
before calling [inject](#inject).
-->

定義されている伝搬フィールドです。キャリアを再利用する場合は、[注入(inject)](#注入inject)を呼び出す前にこのフィールドを削除する必要があります。

<!--
For example, if the carrier is a single-use or immutable request object, you don't need to
clear fields as they couldn't have been set before. If it is a mutable, retryable object,
successive calls should clear these fields first.
-->

例えば、キャリアが一回のみ使われる、またはイミュータブルなリクエストオブジェクトであれば、フィールドを削除する必要はありません。もし、変更可能で何度も使われる可能性があるオブジェクトであれば、連続した呼び出しは最初にこれらのフィールドをクリアしなければなりません。

<!--
The use cases of this are:
-->

ユースケースを以下に示します。

<!--
- allow pre-allocation of fields, especially in systems like gRPC Metadata
- allow a single-pass over an iterator
-->

- 特に gRPC のメタデータのようなシステムでフィールドの事前割り当てを可能にする
- 変更されないため、一回フィールド全体を走査するだけで良くなる

<!--
Returns list of fields that will be used by this formatter.
-->

このフォーマッタが使用するフィールドのリストを返します。(??? この文章唐突なんだけど)

<!--
### Inject
-->

### 注入(inject)

<!--
Injects the value downstream. For example, as http headers.
-->

例えばhttpヘッダとして、次に渡すプロセスのために値を注入します。

<!--
Required arguments:
-->

必要な引数:

<!--
- A `Context`. The Propagator MUST retrieve the appropriate value from the `Context` first, such as `SpanContext`, `CorrelationContext` or another cross-cutting concern context. For languages supporting current `Context` state, this argument is OPTIONAL, defaulting to the current `Context` instance.
- the carrier that holds propagation fields. For example, an outgoing message or http request.
- the `Setter` invoked for each propagation key to add or remove.
-->

- `Context`: Propagator は最初に `SpanContext` や `CorrelationContext` あるいは他の関連するコンポーネントの `Context` から適切な値を取得しなければなりません(MUST)。現在の `Context` をサポートする言語では、この引数はオプションであり、デフォルトは現在の `Context` インスタンスになります。
- 伝搬用のフィールドを保持するキャリア: 例えば、発信メッセージやHTTPリクエストなどです。
- 追加または削除する伝搬キーごとに呼び出される `Setter`

<!--
#### Setter argument
-->

#### Setter引数

<!--
Setter is an argument in `Inject` that sets value into given field.
-->

Setterは指定されたフィールドに値をセットする `Inject` のための引数です。


<!--
`Setter` allows a `HTTPTextFormat` to set propagated fields into a carrier.
-->

`Setter` は 伝搬されたフィールドをキャリアに`HTTPTextFormat` を設定します。

<!--
`Setter` MUST be stateless and allowed to be saved as a constant to avoid runtime allocations. One of the ways to implement it is `Setter` class with `Set` method as described below.
-->

`Setter` はステートレスでなければならず、実行時の割り当てを避けるために定数として保存できなければなりません(MUST)。これを実装する方法の一つは、以下のように `Set` メソッドを持つ `Setter` クラスです。

<!--
##### Set
-->

##### Set

<!--
Replaces a propagated field with the given value.
-->

伝搬されたフィールドを指定された値に置き換えます。

<!--
Required arguments:
-->

必要な引数:

<!--
- the carrier holds propagation fields. For example, an outgoing message or http request.
- the key of the field.
- the value of the field.
-->

- 伝搬されるフィールドを保持しているキャリア。例えば、送信メッセージや HTTP リクエストなどです。
- そのフィールドのキー
- そのフィールドの値

<!--
The implemenation SHOULD preserve casing (e.g. it should not transform `Content-Type` to `content-type`) if the used protocol is case insensitive, otherwise it MUST preserve casing.
-->

使用されるプロトコルが大文字小文字を区別しない場合、実装は大文字小文字を保存するべきです(SHOULD) (例: `Content-Type` を `content-type` に変換してはならない) 。区別する場合は、大文字小文字を保持しなければなりません(MUST)。

<!--
### Extract
-->

### 抽出(Extract)

<!--
Extracts the value from an incoming request. For example, from the headers of an HTTP request.
-->

受信したリクエストから値を抽出します。たとえば、HTTP リクエストのヘッダからです。

<!--
If a value can not be parsed from the carrier for a cross-cutting concern,
the implementation MUST NOT throw an exception. It MUST store a value in the `Context`
that the implementation can recognize as a null or empty value.
-->

関連するコンポーネントのためにキャリアから値を解析できない場合、実装は例外を投げてはいけません(MUST NOT)。実装がnullまたは空の値として認識できる値を `Context` に格納する必要があります(MUST)。

<!--
Required arguments:
-->

必要な引数:

<!--
- A `Context`. For languages supporting current `Context` state this argument is OPTIONAL, defaulting to the current `Context` instance.
- the carrier holds propagation fields. For example, an outgoing message or http request.
- the instance of `Getter` invoked for each propagation key to get.
-->

- `Context`. 現在の `Context` をサポートする言語では、この引数はオプションであり、デフォルトは現在の `Context` インスタンスになります。
- 伝搬されるフィールドを保持しているキャリア。例えば、送信メッセージや http リクエストなどです。
- 各伝搬キーを取得するために呼び出される `Getter` のインスタンス

<!--
Returns a new `Context` derived from the `Context` passed as argument,
containing the extracted value, which can be a `SpanContext`,
`CorrelationContext` or another cross-cutting concern context.
-->

引数として渡された `Context` から派生した新しい `Context` を返します。この `Context` は `SpanContext` や `CorrelationContext` あるいは関連するコンポーネントから展開された値を含んでいる場合があります。

<!--
If the extracted value is a `SpanContext`, its `IsRemote` property MUST be set to true.
-->

もし展開された値が `SpanContext` ならば、 `IsRemote` プロパティはtrueにセットされている必要があります(MUST)。

<!--
#### Getter argument
-->

#### Getter 引数

<!--
Getter is an argument in `Extract` that get value from given field
-->

Getter は `Extract` の引数で、与えられたフィールドから値を取得します。

<!--
`Getter` allows a `HttpTextFormat` to read propagated fields from a carrier.
-->

`Getter` は `HttpTextFormat` がキャリアから伝搬されたフィールドを読み込むことを可能にする。

<!--
`Getter` MUST be stateless and allowed to be saved as a constant to avoid runtime allocations. One of the ways to implement it is `Getter` class with `Get` method as described below.
-->

`Getter` はステートレスでなければならず、実行時のメモリ割り当てを避けるために定数として保存される必要があります(MUST)。これを実装する方法の一つとして、以下に説明するように `Get` メソッドを持つ `Getter` クラスがあります。

<!--
##### Get
-->

##### Get

<!--
The Get function MUST return the first value of the given propagation key or return null if the key doesn't exist.
-->

Get関数は、与えられた伝搬キーの最初の値。あるいは、キーが存在しない場合はNULLを返さなければなりません(MUST)。

<!--
Required arguments:
-->

必要な引数:

<!--
- the carrier of propagation fields, such as an HTTP request.
- the key of the field.
-->

- HTTPリクエストなどの伝搬フィールドのキャリア
- フィールドのキー

<!--
The Get function is responsible for handling case sensitivity. If the getter is intended to work with a HTTP request object, the getter MUST be case insensitive. To improve compatibility with other text-based protocols, text `Format` implementions MUST ensure to always use the canonical casing for their attributes. NOTE: Cannonical casing for HTTP headers is usually title case (e.g. `Content-Type` instead of `content-type`).
-->

Get関数は、大文字小文字の区別を処理する役割を担っています。GetterがHTTPリクエストオブジェクトで動作することを意図している場合、 Getterは大文字小文字を区別する必要があります(MUST)。他のテキストベースのプロトコルとの互換性を向上させるために、テキスト `Format` の実装は、その属性は常にCanonical Case(???ここでのcanonocal caseとはどういう意味？)である必要があります(MUST)。注意: HTTPヘッダのキャノニカルなケーシングは通常タイトルケース(先頭が大文字)です(例えば、`content-type`の代わりに`Content-Type`など)。

<!--
## Injectors and Extractors as Separate Interfaces
-->

## 別々のインターフェースとしての注入と抽出

<!--
Languages can choose to implement a `Propagator` for a format as a single object
exposing `Inject` and `Extract` methods, or they can opt to divide the
responsibilities further into individual `Injector`s and `Extractor`s. A
`Propagator` can be implemented by composing individual `Injector`s and
`Extractors`.
-->

言語は、`Inject` と `Extract` メソッドを公開する単一のオブジェクトとして `Propagator` を実装するか、あるいは個々の `Injector` と `Extractor` にさらに責任を分解することができます。Propagatorは、個々の `Injector` と `Extractor` を組み合わせて実装できます。

<!--
## Composite Propagator
-->

## 合成Propagator(Composite Propagator)

<!--
Implementations MUST offer a facility to group multiple `Propagator`s
from different cross-cutting concerns in order to leverage them as a
single entity.
-->

実装は、単一のエンティティとして活用するために、異なる関連するコンポーネントから複数の `Propagator` をグループ化する機能を提供しなければなりません(MUST)。

<!--
A composite propagator can be built from a list of propagators, or a list of
injectors and extractors. The resulting composite `Propagator` will invoke the `Propagator`s, `Injector`s, or `Extractor`s, in the order they were specified.
-->

合成PropagatorはPropagatorのリスト、またはInjectorとExtractorのリストから構築できます。結果として得られる合成Propagatorは、指定された順に `Propagator`, `Injector`, `Extractor` を呼び出します。

<!--
Each composite `Propagator` will be bound to a specific `Format`, such
as `HttpTextFormat`, as different `Format`s will likely operate on different
data types.
There MUST be functions to accomplish the following operations.
-->

それぞれの合成 `Propagator` は `HttpTextFormat` のようなそれぞれ異なるデータを扱う `Format` と結び付けられます。合成 `Propagator`は以下の操作を行うための関数を持つ必要があります(MUST)。

<!--
- Create a composite propagator
- Extract from a composite propagator
- Inject into a composite propagator
-->

- 合成Propagatorの作成
- 合成Propagatorからの抽出
- 合成Propagatorへの注入

<!--
### Create a Composite Propagator
-->

### 合成Propagatorの作成

<!--
Required arguments:
-->

必要な引数:

<!--
- A list of `Propagator`s or a list of `Injector`s and `Extractor`s.
-->

- `Propagator` のリスト、または `Injector` と `Extractor` のリスト。

<!--
Returns a new composite `Propagator` with the specified `Propagator`s.
-->

指定された `Propagator` を用いた新しい合成 `Propagator` を返します。

<!--
### Extract
-->

### 合成Propagatorからの抽出

<!--
Required arguments:
-->

必要な引数:

<!--
- A `Context`.
- The carrier that holds propagation fields.
- The instance of `Getter` invoked for each propagation key to get.
-->

- `Context`
- 伝搬されるフィールドを保持しているキャリア
- 各伝搬キーを取得するために呼び出される `Getter` のインスタンス

<!--
### Inject
-->

### 合成Propagatorへの注入

<!--
Required arguments:
-->

必要な引数:

<!--
- A `Context`.
- The carrier that holds propagation fields.
- The `Setter` invoked for each propagation key to add or remove.
-->

- `Context`
- 伝搬されるフィールドを保持しているキャリア
- 各伝搬キーを取得するために呼び出される `Getter` のインスタンス
- 追加または削除する伝搬キーごとに呼び出される `Setter`

<!--
## Global Propagators
-->

## グローバル Propagator

<!--
Implementations MAY provide global `Propagator`s for
each supported `Format`.
-->

サポートされている`Format`ごとにグローバルな`Propagator`を実装しても構いません(MAY)。

<!--
If offered, the global `Propagator`s should default to a composite `Propagator`
containing W3C Trace Context and Correlation Context `Propagator`s,
in order to provide propagation even in the presence of no-op
OpenTelemetry implementations.
-->

もし実装された場合、OpenTelemetryの実装が存在しない場合でも伝搬を提供するために、グローバル `Propagator` は、W3Cトレースコンテキストと `Correlation Context` の `Propagator` を含む合成 `Propagator` はデフォルトに設定されるべきです(SHOULD)。

<!--
### Get Global Propagator
-->

### グローバル Propagatorの取得

<!--
This method MUST exist for each supported `Format`.
-->

このメソッドはサポートされている`Format`ごとに存在しなければなりません(MUST)。

<!--
Returns a global `Propagator`. This usually will be composite instance.
-->

グローバル `Propagator` を返す。これは通常、合成インスタンスになります。

<!--
### Set Global Propagator
-->

### グローバル Propagatorのセット

<!--
This method MUST exist for each supported `Format`.
-->

このメソッドはサポートされている`Format`ごとに存在しなければなりません(MUST)。

<!--
Sets the global `Propagator` instance.
-->

グローバル `Propagator` をセットします。

<!--
Required parameters:
-->

必要なパラメータ:

<!--
- A `Propagator`. This usually will be a composite instance.
-->

- `Propagator`。これは通常、合成インスタンスになります。
