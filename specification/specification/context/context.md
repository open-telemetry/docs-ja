<!--
# Context
-->

# Context

<!--
**Status**: [Stable, Feature-freeze](../document-status.md).
-->

**Status**: [Stable, Feature-freeze](../document-status.md).


<details> <summary> 目次 </summary>

<!--
- [Overview](#overview)
- [Create a key](#create-a-key)
- [Get value](#get-value)
- [Set value](#set-value)
- [Optional global operations](#optional-global-operations)
  - [Get current Context](#get-current-context)
  - [Attach Context](#attach-context)
  - [Detach Context](#detach-context)
-->

- [概要](#概要)
- [キーの作成](#キーの作成)
- [値の取得](#値の取得)
- [値の設定](#値の設定)
- [任意のグローバルな操作](#任意のグローバルな操作)
  - [現在のContextを取得](#現在のContextを取得)
  - [Contextのアタッチ](#Contextのアタッチ)
  - [Contextのデタッチ](#Contextのデタッチ)


</details>

<!--
## Overview
-->

## 概要

<!--
A `Context` is a propagation mechanism which carries execution-scoped values
across API boundaries and between logically associated execution units.
Cross-cutting concerns access their data in-process using the same shared
`Context` object.
-->

`Context`は、APIの境界を越えて、論理的に関連付けられた実行ユニット間で、実行スコープ付きの値を伝達する伝搬メカニズムです。横断的な関心事は、同じ共有された `Context` オブジェクトを使用してプロセス内のデータにアクセスします。

<!--
A `Context` MUST be immutable, and its write operations MUST
result in the creation of a new `Context` containing the original
values and the specified values updated.
-->

`Context` は不変でなければなりません(MUST)し、Contextへの書き込み操作では、元の値と指定された値が更新された新しい `Context` が作成されなければなりません (MUST)。

<!--
Languages are expected to use the single, widely used `Context` implementation
if one exists for them. In the cases where an extremely clear, pre-existing
option is not available, OpenTelemetry MUST provide its own `Context`
implementation. Depending on the language, its usage may be either explicit
or implicit.
-->

言語は、広く使われている単一の `Context` 実装があれば、それを使うことが期待されます。極めて明確な既存のオプションが利用できない場合には、OpenTelemetryは独自の`Context`実装を提供しなければなりません(MUST)。言語によっては、その使用方法は明示的なものにも暗黙的なものにもなる場合があります。

<!--
Users writing instrumentation in languages that use `Context` implicitly are
discouraged from using the `Context` API directly. In those cases, users will
manipulate `Context` through cross-cutting concerns APIs instead, in order to
perform operations such as setting trace or baggage entries for a specified
`Context`.
-->

`Context` を暗黙的に使用する言語で計装を実装するユーザは、`Context` API を直接使用することはお勧めできません。そのような場合には、指定した `Context` に対してTrace エントリやBaggageエントリを設定するなどの操作を行うために、代わりに横断的な関心事のAPI を使用して `Context` を操作することになります。

<!--
A `Context` is expected to have the following operations, with their
respective language differences:
-->

`Context`には、以下のような操作が想定されており、それぞれの言語で違いがあります:

<!--
## Create a key
-->

## キーの作成

<!--
Keys are used to allow cross-cutting concerns to control access to their local state.
They are unique such that other libraries which may use the same context
cannot accidentally use the same key. It is recommended that concerns mediate
data access via an API, rather than provide direct public access to their keys.
-->

キーは、横断的な関心事がローカルな状態へのアクセスを制御するために使用されます。キーは一意であるため、同じContextを使用する他のライブラリが誤って同じキーを使用することはありません。関係者は、自分のキーへの直接のパブリックアクセスを提供するのではなく、APIを介してデータアクセスを仲介することが推奨されます。

<!--
The API MUST accept the following parameter:
-->

このAPIは、以下のパラメータを受け入れなければなりません(MUST)。

<!--
- The key name. The key name exists for debugging purposes and does not uniquely identify the key. Multiple calls to `CreateKey` with the same name SHOULD NOT return the same value unless language constraints dictate otherwise. Different languages may impose different restrictions on the expected types, so this parameter remains an implementation detail.
-->

- キーの名前。キー名はデバッグのために存在するもので、キーを一意に識別するものではありません。同じ名前で `CreateKey` を複数回呼び出しても、言語上の制約がない限り、同じ値を返すべきではありません(SHOULD NOT)。言語によって、期待される型の制限が異なる場合があるので、このパラメータは実装上の詳細にとどまります。

<!--
The API MUST return an opaque object representing the newly created key.
-->

このAPIは、新しく作成されたキーを表す不透明なオブジェクトを返さなければなりません(MUST)。

<!--
## Get value
-->

## 値の取得

<!--
Concerns can access their local state in the current execution state
represented by a `Context`.
-->

横断的な関心事は、`Context`で表される現在の実行状態に保存されている、自分のローカルな状態にアクセスすることができます。

<!--
The API MUST accept the following parameters:
-->

このAPIは、以下のパラメータを受け入れなければなりません(MUST)。

<!--
- The `Context`.
- The key.
-->

- `Context`
- キー

<!--
The API MUST return the value in the `Context` for the specified key.
-->

このAPI は、指定されたキーの `Context` 内の値を返さなければなりません (MUST)。

<!--
## Set value
-->

## 値の設定

<!--
Concerns can record their local state in the current execution state
represented by a `Context`.
-->

横断的な関心事は、`Context`で表される現在の実行状態に、自分のローカルな状態を設定できます。

<!--
The API MUST accept the following parameters:
-->

このAPIは、以下のパラメータを受け入れなければなりません(MUST):

<!--
- The `Context`.
- The key.
- The value to be set.
-->

- `Context`
- キー
- 設定する値

<!--
The API MUST return a new `Context` containing the new value.
-->

このAPI は、新しい値を含む新しい `Context` を返さなければなりません。

<!--
## Optional Global operations
-->

## 任意のグローバルな操作

<!--
These operations are expected to only be implemented by languages
using `Context` implicitly, and thus are optional. These operations
SHOULD only be used to implement automatic scope switching and define
higher level APIs by SDK components and OpenTelemetry instrumentation libraries.
-->

これらの操作は、`Context` を暗黙のうちに使用する言語によってのみ実装されることが期待されており、したがって任意です。これらの操作は、SDKコンポーネントやOpenTelemetry 計装ライブラリによる自動スコープ切り替えの実装や、高レベルAPIの定義にのみ使用されるべき(SHOULD)です。

<!--
### Get current Context
-->

### 現在のContextを取得


<!--
The API MUST return the `Context` associated with the caller's current execution unit.
-->

このAPIは、呼び出し元の現在の実行ユニットに関連する`Context`を返さなければなりません(MUST)。

<!--
### Attach Context
-->

### Contextのアタッチ

<!--
Associates a `Context` with the caller's current execution unit.
-->

呼び出し元の現在の実行ユニットに `Context` を関連付けます。

<!--
The API MUST accept the following parameters:
-->

このAPIは、以下のパラメータを受け入れなければなりません(MUST)。

<!--
- The `Context`.
-->

- `Context`

<!--
The API MUST return a value that can be used as a `Token` to restore the previous
`Context`.
-->

このAPI は、以前の `Context` を復元するための `Token` として使用可能な値を返さなければなりません (MUST)。

<!--
Note that every call to this operation should result in a corresponding call to
[Detach Context](#detach-context).
-->

注意点として、この操作を行う際には、必ず[Contextのデタッチ](#Contextのデタッチ)を呼び出す必要があります。

<!--
### Detach Context
-->

### Contextのデタッチ

<!--
Resets the `Context` associated with the caller's current execution unit
to the value it had before attaching a specified `Context`.
-->

呼び出し元の現在の実行ユニットに関連付けられた `Context` を、指定された `Context` をアタッチする前の値にリセットします。

<!--
This operation is intended to help making sure the correct `Context`
is associated with the caller's current execution unit. Users can
rely on it to identify a wrong call order, i.e. trying to detach
a `Context` that is not the current instance. In this case the operation
can emit a signal to warn users of the wrong call order, such as logging
an error or returning an error value.
-->

この操作は、正しい `Context` が呼び出し元の現在の実行ユニットに関連付けられていることを確認するのに役立つことを目的としています。ユーザーはこの操作を頼りに、間違った呼び出し順序、つまり現在のインスタンスではない `Context` を切り離そうとしているかを調べられます。この場合、この操作は、エラーをログに記録したり、エラー値を返したりして、呼び出し順序が間違っていることをユーザーに警告するシグナルを発することができます。

<!--
The API MUST accept the following parameters:
-->

このAPIは、以下のパラメータを受け入れなければなりません(MUST):

<!--
- A `Token` that was returned by a previous call to attach a `Context`.
-->

- 以前に `Context` をアタッチした際に返された `Token`

<!--
The API MAY return a value used to check whether the operation
was successful or not.
-->

このAPI は、操作が成功したかどうかを確認するために使用する値を返してもかまいません(MAY)。
