<!--
# Correlations API
-->

# Correlations API

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
- [Overview](#overview)
  - [CorrelationContext](#correlationcontext)
  - [Get correlations](#get-correlations)
  - [Get correlation](#get-correlation)
  - [Set correlation](#set-correlation)
  - [Remove correlation](#remove-correlation)
  - [Clear correlations](#clear-correlations)
- [CorrelationContext Propagation](#correlationcontext-propagation)
- [Conflict Resolution](#conflict-resolution)
-->

- [概要](#概要)
  - [CorrelationContext](#correlationcontext)
  - [複数のCorrelationの取得](#複数のCorrelationの取得)
  - [Correlationの取得](#Correlationの取得)
  - [Correlationの設定](#Correlationの設定)
  - [Correlationの削除](#Correlationの削除)
  - [Correlationのクリア](#Correlationのクリア)
- [CorrelationContextの伝搬](#CorrelationContextの伝搬)
- [コンフリクトの解決](#コンフリクトの解決)

</details>

<!--
## Overview
-->

## 概要

<!--
The Correlations API consists of:
-->

Correlations APIは以下から成り立っています:

<!--
- the `CorrelationContext`
- functions to interact with the `CorrelationContext` in a `Context`
-->

- `CorrelationContext`
- `Context` 内の `CorrelationContext` と対話するための関数

<!--
### CorrelationContext
-->

### CorrelationContext

<!--
`CorrelationContext` is used to annotate telemetry, adding context and information to metrics, traces, and logs.
It is an abstract data type represented by a set of name/value pairs describing user-defined properties.
Each name in `CorrelationContext` MUST be associated with exactly one value.
`CorrelationContext` MUST be serialized according to the editor's draft of the [W3C Correlation Context](https://w3c.github.io/correlation-context/)
specification.
-->

`CorrelationContext` は、テレメトリーに注釈を付けるために使用され、メトリック、トレース、ログにContextと情報を追加します。これは、ユーザー定義のプロパティを記述する名前と値のペアのセットで表される抽象的なデータ型です。 `CorrelationContext` の各名前は正確に1つの値に関連付けなければなりません(MUST)。`CorrelationContext` は、[W3C Correlation Context](https://w3c.github.io/correlation-context/) 仕様のエディタードラフトに従ってシリアライズされなければなりません(MUST)。

<!--
### Get correlations
-->

### 複数のCorrelationの取得

<!--
Returns the name/value pairs in the `CorrelationContext`. The order of name/value pairs MUST NOT be
significant. Based on the language specification, the returned value can be
either an immutable collection or an immutable iterator to the collection of
name/value pairs in the `CorrelationContext`.
-->

`CorrelationContext`の名前と値のペアを返します。名前/値のペアの順序は、有意であってはいけません(MUST NOT)。言語仕様に基づいて、返される値はイミュータブルのコレクションか、`CorrelationContext`内の名前と値のペアのコレクションに対するイミュータブルなイテレータのどちらかになります。

<!--
OPTIONAL parameters:
-->

オプションのパラメータ:

<!--
`Context` the context containing the `CorrelationContext` from which to get the correlations.
-->

- `Context`: Correlationを取得するための `CorrelationContext` を含む `Context`。

<!--
### Get correlation
-->

### Correlationの取得

<!--
To access the value for a name/value pair by a prior event, the Correlations API
SHALL provide a function that takes a context and a name as input, and returns a
value. Returns the value associated with the given name, or null
if the given name is not present.
-->

過去のイベントによる名前と値のペアの値にアクセスするには、Correlations APIはContextと名前を入力とし、 値を返す関数を提供さなければいけません(SHALL)。指定された名前に関連付けられた値を返すか、指定された名前が存在しない場合は null を返します。

<!--
REQUIRED parameters:
-->

必要なパラメータ:

<!--
`Name` the name to return the value for.
-->

- `Name`: 値を返す名前

<!--
OPTIONAL parameters:
-->

オプションのパラメータ:

<!--
`Context` the context containing the `CorrelationContext` from which to get the correlation.
-->

- `Context`: Correlationを取得するための `CorrelationContext` を含む `Context`。

<!--
### Set correlation
-->

### Correlationの設定

<!--
To record the value for a name/value pair, the Correlations API SHALL provide a function which
takes a context, a name, and a value as input. Returns a new `Context` which
contains a `CorrelationContext` with the new value.
-->

名前と値のペアの値を記録するために、Correlations APIはContext、名前、値を入力として受け取る関数を提供することになります(SHALL)。戻り値は、新しい値を含む `CorrelationContext` を含む新しい `Context` です。

<!--
REQUIRED parameters:
-->

必要なパラメータ:

<!--
`Name` the name for which to set the value.
-->

- `Name`: 値をセットする名前
<!--
`Value` the value to set.
-->

- `Value`: セットする値

<!--
OPTIONAL parameters:
-->

オプションのパラメータ:

<!--
`Context` the context containing the `CorrelationContext` in which to set the correlation.
-->

- `Context`: Correlationをセットするための `CorrelationContext` を含む `Context`。

<!--
### Remove correlation
-->

### Correlationの削除

<!--
To delete a name/value pair, the Correlations API SHALL provide a function which takes a context
and a name as input. Returns a new `Context` which no longer contains the selected name.
-->

名前と値のペアを削除するために、Correlations APIはContextと名前を入力として受け取る関数を提供する必要があります(SHALL)。選択した名前を含まない新しい `Context` を返します。

<!--
REQUIRED parameters:
-->

必要なパラメータ:

<!--
`Name` the name to remove.
-->

- `Name`: 削除する名前

<!--
OPTIONAL parameters:
-->

オプションのパラメータ:

<!--
`Context` the context containing the `CorrelationContext` from which to remove the correlation.
-->

- `Context`: Correlationを削除するための `CorrelationContext` を含む `Context`。

<!--
### Clear correlations
-->

### Correlationのクリア

<!--
To avoid sending any name/value pairs to an untrusted process, the Correlations API SHALL provide
a function to remove all Correlations from a context. Returns a new `Context`
with no correlations.
-->

信頼されていないプロセスに名前と値のペアを送信しないようにするために、Correlations API はContextからすべてのCorrelationを削除する関数を提供する必要があります(SHALL)。Correlationがない新しい `Context` を返します。

<!--
OPTIONAL parameters:
-->

オプションのパラメータ:

<!--
`Context` the context containing the `CorrelationContext` from which to remove all correlations.
-->

- `Context`: Correlationをすべて削除するための `CorrelationContext` を含む `Context`。

<!--
## CorrelationContext Propagation
-->

## CorrelationContextの伝搬

<!--
`CorrelationContext` MAY be propagated across process boundaries or across any arbitrary boundaries
(process, $OTHER_BOUNDARY1, $OTHER_BOUNDARY2, etc) for various reasons.
-->

様々な理由により、`CorrelationContext`はプロセスの境界を越えて、あるいは任意の境界(プロセス、$OTHER_BOUNDARY1、$OTHER_BOUNDARY2など)を越えて伝搬することがあります(MAY)。

<!--
## Conflict Resolution
-->

## コンフリクトの解決

<!--
If a new name/value pair is added and its name is the same as an existing name, than the new pair MUST take precedence. The value
is replaced with the added value (regardless if it is locally generated or received from a remote peer).
-->

新しい名前/値のペアが追加され、その名前が既存の名前と同じ場合、新しいペアを優先しなければなりません(MUST)。値は追加された値に置き換えられます(ローカルで生成されたものかリモートピアから受信したものかは関係ありません)。
