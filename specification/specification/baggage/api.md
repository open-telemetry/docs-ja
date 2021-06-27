<!--
# Baggage API
-->

# Baggage API

<!--
**Status**: [Stable, Feature-freeze](../document-status.md)
-->

**Status**: [Stable, Feature-freeze](../document-status.md)

<details>
<summary>
目次
</summary>

<!--
- [Overview](#overview)
- [Operations](#operations)
  - [Get Value](#get-value)
  - [Get All Values](#get-all-values)
  - [Set Value](#set-value)
  - [Remove Value](#remove-value)
- [Context Interaction](#context-interaction)
  - [Clear Baggage in the Context](#clear-baggage-in-the-context)
- [Propagation](#propagation)
- [Conflict Resolution](#conflict-resolution)
-->

- [概要](#概要)
- [操作](#操作)
  - [値の取得](#値の取得)
  - [すべての値の取得](#すべての値の取得)
  - [値の設定](#値の設定)
  - [値の削除](#値の削除)
- [コンテキストのインタラクション](#コンテキストのインタラクション)
  - [Context内のBaggageを削除](#Context内のBaggageを削除)
- [伝搬](#伝搬)
- [コンフリクトの解決](#コンフリクトの解決)

</details>

<!--
## Overview
-->

## 概要

<!--
`Baggage` is used to annotate telemetry, adding context and information to
metrics, traces, and logs. It is a set of name/value pairs describing
user-defined properties. Each name in `Baggage` MUST be associated with
exactly one value.
-->

`Baggage` は、テレメトリにアノテーションを付けて、Metrics, Trace, Logにコンテキストや情報を追加するために使用します。これは、ユーザー定義のプロパティを記述する名前と値のペアのセットです。`Baggage`の各名前は、正確に1つの値と関連付けられなければなりません(MUST)。

<!--
The Baggage API consists of:
-->

Baggage APIは以下のように構成されています:

<!--
- the `Baggage`
- functions to interact with the `Baggage` in a `Context`
-->

- `Baggage`
- `Context` 内の `Baggage` を操作するための関数

<!--
The functions described here are one way to approach interacting with the
`Baggage` via having struct/object that represents the entire Baggage content.
Depending on language idioms, a language API MAY implement these functions by
interacting with the baggage via the `Context` directly.
-->

ここで説明されている関数は、`Baggage`の内容全体を表す構造体/オブジェクトを介して `Baggage` と対話するための1つの方法です。言語イディオムによっては、言語APIは、`Context`を介してBaggageを直接扱うことで、これらの関数を実装しても構いません(MAY)。

<!--
The Baggage API MUST be fully functional in the absence of an installed SDK.
This is required in order to enable transparent cross-process Baggage
propagation. If a Baggage propagator is installed into the API, it will work
with or without an installed SDK.
-->

Baggage APIは、インストールされたSDKがなくても完全に機能しなければなりません(MUST)。これは、透過的なプロセス間のBaggage伝播を可能にするために必要です。もしバゲージプロパゲータがAPIにインストールされていれば、インストールされたSDKがあってもなくても動作します。

<!--
The `Baggage` container MUST be immutable, so that the containing `Context`
also remains immutable.
-->

`Baggage`コンテナは不変的(immutable)でなければなりません(MUST)。これにより、含まれる`Context`も不変的なものになります。

<!--
## Operations
-->

## 操作

<!--
### Get Value
-->

### 値の取得

<!--
To access the value for a name/value pair set by a prior event, the Baggage API
MUST provide a function that takes the name as input, and returns a value
associated with the given name, or null if the given name is not present.
-->

以前のイベントで設定された名前と値のペアの値にアクセスするために、Baggage APIは、名前を入力として受け取り、与えられた名前に関連する値、または与えられた名前が存在しない場合はnullを返す関数を提供しなければなりません(MUST)。


<!--
REQUIRED parameters:
-->

必須パラメータ:

<!--
`Name` the name to return the value for.
-->

`Name` 値を返す名前

<!--
### Get All Values
-->

### すべての値を取得

<!--
Returns the name/value pairs in the `Baggage`. The order of name/value pairs
MUST NOT be significant. Based on the language specifics, the returned
value can be either an immutable collection or an iterator on the immutable
collection of name/value pairs in the `Baggage`.
-->

その `Baggage`に含まれる名前と値のペアを返します。名前と値のペアの順序に意味があってはいけません(MUST NOT)。言語の仕様により、戻り値は不変のコレクションか、`Baggage`内の名前/値のペアの不変のコレクションのイテレータになります。

<!--
### Set Value
-->

### 値の設定

<!--
To record the value for a name/value pair, the Baggage API MUST provide a
function which takes a name, and a value as input. Returns a new `Baggage`
that contains the new value. Depending on language idioms, a language API MAY
implement these functions by using a `Builder` pattern and exposing a way to
construct a `Builder` from a `Baggage`.
-->

名前と値のペアの値を記録するために、Baggage APIは、名前と値を入力として受け取る関数を提供しなければなりません(MUST)。新しい値を含む新しい `Baggage` を返します。言語イディオムによっては、言語APIは `Builder` パターンを使用してこれらの関数を実装し、`Baggage` から `Builder` を構築する方法を公開してもかまいません。

<!--
REQUIRED parameters:
-->

必須パラメータ:

<!--
`Name` The name for which to set the value, of type string.
-->

`Name` 値を設定する名前で、文字列型

<!--
`Value` The value to set, of type string.
-->

`Value` 設定する値で、文字列型

<!--
OPTIONAL parameters:
-->

任意パラメータ:

<!--
`Metadata` Optional metadata associated with the name-value pair. This should be
an opaque wrapper for a string with no semantic meaning. Left opaque to allow
for future functionality.
-->

`Metadata` 名前と値のペアに関連する任意のメタデータ。これはセマンティックな意味を持たない文字列の不透明なラッパーでなければなりません。将来の機能性を考慮して、不透明なままにしてあります。
<!--
### Remove Value
-->

### 値の削除

<!--
To delete a name/value pair, the Baggage API MUST provide a function which
takes a name as input. Returns a new `Baggage` which no longer contains the
selected name. Depending on language idioms, a language API MAY
implement these functions by using a `Builder` pattern and exposing a way to
construct a `Builder` from a `Baggage`.
-->

名前と値のペアを削除するために、Baggage APIは名前を入力として受け取る関数を提供しなければなりません(MUST)。この関数は選択された名前を含まない、新しい `Baggage` を返します。言語イディオムによっては、言語APIは `Builder` パターンを使用してこれらの関数を実装し、`Baggage` から `Builder` を構築する方法を公開してもかまいません(MAY)。

<!--
REQUIRED parameters:
-->

必須パラメータ:

<!--
`Name` the name to remove.
-->

`Name` 削除する名前

<!--
## Context Interaction
-->

## コンテキストのインタラクション

<!--
This section defines all operations within the Baggage API that interact with
the [`Context`](../context/context.md).
-->

このセクションでは、[`Context`](../context/context.md)と相互作用するBaggage API内のすべての操作を定義します。

<!--
If an implementation of this API does not operate directly on the `Context`, it
MUST provide the following functionality to interact with a `Context` instance:
-->

API の実装が `Context` を直接操作しない場合は、`Context` インスタンスを扱うための以下の機能を提供しなければなりません (MUST)。

<!--
- Extract the `Baggage` from a `Context` instance
- Insert the `Baggage` to a `Context` instance
-->

- `Context` インスタンスから `Baggage` を抽出する
- `Context` インスタンスに`Baggage`を挿入する


<!--
The functionality listed above is necessary because API users SHOULD NOT have
access to the [Context Key](../context/context.md#create-a-key) used by the
Baggage API implementation.
-->

上記の機能が必要なのは、APIユーザが、Baggage APIの実装で使用される[Context Key](../context/context.md#create-a-key)にアクセスすべきではないからです(SHOULD NOT)。

<!--
If the language has support for implicitly propagated `Context` (see
[here](../context/context.md#optional-global-operations)), the API SHOULD also
provide the following functionality:
-->

言語が暗黙的に伝搬する`Context`をサポートしている場合([こちら](../context/context.md#optional-global-operations)を参照)、APIは以下の機能も提供すべきです(SHOULD)。

<!--
- Get the currently active `Baggage` from the implicit context. This is
equivalent to getting the implicit context, then extracting the `Baggage` from
the context.
- Set the currently active `Baggage` to the implicit context. This is equivalent
to getting the implicit context, then inserting the `Baggage` to the context.
-->

- 暗黙的なContextから現在アクティブな `Baggage` を取得します。これは、暗黙のContextを取得して、そのContextから `Baggage` を抽出することと同じです。

- 現在アクティブな `Baggage` を暗黙のContextに設定します。これは、暗黙のContextを取得してから、そのContextに `Baggage` を挿入することと同じです。

<!--
All the above functionalities operate solely on the context API, and they MAY be
exposed as static methods on the baggage module, as static methods on a class
inside the baggage module (it MAY be named `BaggageUtilities`), or on the
`Baggage` class. This functionality SHOULD be fully implemented in the API when
possible.
-->

上記のすべての機能は、Context API のみで動作し、Baggage モジュールのスタティックメソッド、Baggage モジュール内のクラス(名前を `BaggageUtilities` にしてもよい(MAY))のスタティックメソッド、または `Baggage` クラスのスタティックメソッドとして公開しても構いません(MAY)。この機能は可能であれば、APIに完全に実装すべきです(SHOULD)。

<!--
### Clear Baggage in the Context
-->

### Context内のBaggageを削除

<!--
To avoid sending any name/value pairs to an untrusted process, the Baggage API
MUST provide a way to remove all baggage entries from a context.
-->

信頼されていないプロセスに名前/値のペアを送ることを避けるために、Baggage APIは、コンテキストからすべてのBaggage エントリを削除する方法を提供しなければなりません(MUST)。

<!--
This functionality can be implemented by having the user set an empty `Baggage`
object/struct into the context, or by providing an API that takes a `Context` as
input, and returns a new `Context` with no `Baggage` associated.
-->

この機能は、ユーザが空の `Baggage` オブジェクト/構造体をContextに設定することで実装できます。また、`Context` を入力として受け取り、`Baggage` が関連付けられていない新しい `Context` を返す API を提供することもできます。

<!--
## Propagation
-->

## 伝搬

<!--
`Baggage` MAY be propagated across process boundaries or across any arbitrary
boundaries (process, $OTHER_BOUNDARY1, $OTHER_BOUNDARY2, etc) for various
reasons.
-->

`Baggage` は、さまざまな理由で、プロセスの境界や任意の境界 (プロセス、$OTHER_BOUNDARY1、$OTHER_BOUNDARY2 など) を越えて伝搬させてもかまいません (MAY)。

<!--
The API layer or an extension package MUST include the following `Propagator`s:
-->

APIレイヤーや拡張パッケージには、以下の `Propagator` が含まれていなければなりません(MUST)。

<!--
* A `TextMapPropagator` implementing the [W3C Baggage Specification](https://w3c.github.io/baggage).
-->

* [W3C Baggage 仕様](https://w3c.github.io/baggage)を実装した`TextMapPropagator`

<!--
See [Propagators Distribution](../context/api-propagators.md#propagators-distribution)
for how propagators are to be distributed.
-->

Propagatorの流通については、[Propagators Distribution](../context/api-propagators.md#propagators-distribution)を参照してください。

<!--
Note: The W3C baggage specification does not currently assign semantic meaning
to the optional metadata.
-->

注:W3C baggage仕様では、現在、オプションのメタデータにセマンティックな意味を付与していません。

<!--
On `extract`, the propagator should store all metadata as a single metadata instance per entry.
On `inject`, the propagator should append the metadata per the W3C specification format.
-->

`extract`では、Propagatorはすべてのメタデータをエントリごとに単一のメタデータ・インスタンスとして保存します。`inject`では、PropagatorはW3C仕様のフォーマットに従ってメタデータを追加します。

<!--
Refer to the API Propagators
[Operation](../context/api-propagators.md#operations) section for the
additional requirements these operations need to follow.
-->

API Propagatorsの[操作](../context/api-propagators.md#operations)のセクションを参照して、これらの操作が従うべき追加要件を確認してください。

<!--
## Conflict Resolution
-->

## コンフリクトの解決

<!--
If a new name/value pair is added and its name is the same as an existing name,
than the new pair MUST take precedence. The value is replaced with the added
value (regardless if it is locally generated or received from a remote peer).
-->

新しい名前/値のペアが追加され、その名前が既存の名前と同じである場合、新しいペアが優先されなければなりません(MUST)。値は追加された値で置き換えられます(ローカルに生成されたものか、リモートピアから受信したものかは関係ありません)。
