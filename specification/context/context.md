# Context

<details>
<summary>
目次
</summary>

- [概要](#概要)
- [キーの作成](#キーの作成)
- [値の取得](#値の取得)
- [値の設定](#値の設定)
- [他の操作(実装は任意)](#他の操作実装は任意)
  - [現在のContextを取得](#現在のContextを取得)
  - [Contextを付与](#Contextを付与)
  - [Contextを除去](#Contextを除去)

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

`Context` は、論理的に関連がある実行ユニット間でAPIの境界を越えて、実行時の値を伝達する伝搬メカニズムです。関連するコンポーネント(???Cross-cutting concerns)は、同じ共有されている `Context` オブジェクトを使ってプロセス中のデータにアクセスします。

<!--
A `Context` MUST be immutable, and its write operations MUST
result in the creation of a new `Context` containing the original
values and the specified values updated.
-->

`Context` は不変でなければならず(MUST)、Contextへの書き込みは元の値と指定され更新された値を持つ新しい `Context` を生成しなければなりません(MUST)。

<!--
Languages are expected to use the single, widely used `Context` implementation
if one exists for them. In the cases where an extremely clear, pre-existing
option is not available, OpenTelemetry MUST provide its own `Context`
implementation. Depending on the language, its usage may be either explicit
or implicit.
-->

実装言語に広く使われている `Context` 実装が存在する場合には、それを使うことが望ましいです。明確な既存の選択肢がない場合、OpenTelemetry は独自の `Context` 実装を提供しなければなりません(MUST)。言語によっては、その使用法が明示的か暗黙的かのどちらかになります。

<!--
Users writing instrumentation in languages that use `Context` implicitly are
discouraged from using the `Context` API directly. In those cases, users will
manipulate `Context` through cross-cutting concerns APIs instead, in order to
perform operations such as setting trace or correlation context values for
a specified `Context`.
-->

暗黙的に `Context` を使用する言語では、`Context` API を直接使用しないようにしてください。このような場合、指定された `Context` に対してTraceやCorrelation Contextを設定するなどの操作を行うために、関連するコンポーネント(???Cross-cutting concerns)がAPIを介して `Context` を操作することになります。

<!--
A `Context` is expected to have the following operations, with their
respective language differences:
-->

`Context`は、それぞれの言語による差異はありますが、以下の操作を持つことが期待されます。

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

キーは、関連するコンポーネント(???Cross-cutting concerns)がそのローカルに持つ状態へのアクセスを制御できるようにするために使用します。同じContextを使用する可能性のある他のライブラリが誤って同じキーを使用することができないように、キーは一意です。関連するコンポーネント(???Cross-cutting concerns)は、鍵への直接的なパブリックアクセスを提供するのではなく、APIを介したデータアクセスを推奨します。

<!--
The API MUST accept the following parameters:
-->

APIは以下のパラメータを受け付ける必要があります(MUST):

<!--
- The key name. The key name exists for debugging purposes and does not uniquely identify the key. Multiple calls to `CreateKey` with the same name SHOULD NOT return the same value unless language constraints dictate otherwise. Different languages may impose different restrictions on the expected types, so this parameter remains an implementation detail.
-->

- キーの名前: このキーの名前はデバッグのためのもので、キーを一意に特定するものではありません。同じ名前で `CreateKey` を複数回呼び出しても、言語上の制約がない限り、同じ値を返すべきではありません(SHOULD NOT)。言語によっては、期待される型に異なる制限が課せられることがあるので、このパラメータの詳細は実装依存になります。

<!--
The API MUST return an opaque object representing the newly created key.
-->

API は、新しく作成されたキーを表すopaqueなオブジェクトを返す必要があります(MUST)。


<!--
## Get value
-->

## 値の取得

<!--
Concerns can access their local state in the current execution state
represented by a `Context`.
-->

関連するコンポーネント(???Concern)がローカルの状態を、`Context`で表される現在実行中の状態から取得します。

<!--
The API MUST accept the following parameters:
-->

APIは以下のパラメータを受け付ける必要があります(MUST):


<!--
- The `Context`.
- The key.
-->

- `Context`
- キー

<!--
The API MUST return the value in the `Context` for the specified key.
-->

APIは指定されたキーの `Context` が持つ値を返す必要があります(MUST)。

<!--
## Set value
-->

## 値の設定

<!--
Concerns can record their local state in the current execution state
represented by a `Context`.
-->

関連するコンポーネント(???Concern)はローカルの状態を、`Context`で表される現在実行中の状態に設定します。



<!--
The API MUST accept the following parameters:
-->

APIは以下のパラメータを受け付ける必要があります(MUST):

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

APIは新しい値を含んでいる新しい `Context` を返す必要があります(MUST)。


<!--
## Optional Global operations
-->

## 他の操作(実装は任意)

<!--
These operations are expected to only be implemented by languages
using `Context` implicitly, and thus are optional. These operations
SHOULD only be used to implement automatic scope switching and define
higher level APIs by SDK components and OpenTelemetry instrumentation libraries.
-->

これらの操作は、暗黙的に `Context` を使用している言語でのみ実装されることが期待され、従って、任意の操作です。これらの操作は、自動スコープ切り替えの実装と、SDKコンポーネントやOpenTelemetry計装ライブラリによる高位APIの定義にのみ使用されるべきです(SHOULD)。

<!--
### Get current Context
-->

### 現在のContextを取得

<!--
The API MUST return the `Context` associated with the caller's current execution unit.
-->

APIは、呼び出し元の現在の実行単位に関連付けられた `Context` を返す必要があります(MUST)。

### Contextを付与

<!--
Associates a `Context` with the caller's current execution unit.
-->

`Context` を呼び出し元の現在の実行単位に関連付けます。

<!--
The API MUST accept the following parameters:
-->

APIは以下のパラメータを受け付ける必要があります(MUST):

<!--
- The `Context`.
-->

- `Context`

<!--
The API MUST return a value that can be used as a `Token` to restore the previous
`Context`.
-->

APIは、前回の `Context` を復元するための `Token` として利用できる値を返す必要があります(MUST)。

<!--
Note that every call to this operation should result in a corresponding call to
[Detach Context](#detach-context).
-->

この操作を行うたびに、対応する [Contextを除去](#Contextを除去) が呼び出されることに注意してください。

### Contextを除去

<!--
Resets the `Context` associated with the caller's current execution unit
to the value it had before attaching a specified `Context`.
-->

呼び出し元の現在の実行単位に関連付けられた `Context` を、指定された `Context` を付与する前の値にリセットします。

<!--
This operation is intended to help making sure the correct `Context`
is associated with the caller's current execution unit. Users can
rely on it to identify a wrong call order, i.e. trying to detach
a `Context` that is not the current instance. In this case the operation
can emit a signal to warn users of the wrong call order, such as logging
an error or returning an error value.
-->

この操作は、正しい `Context` が呼び出し元の現在の実行ユニットに関連付けられていることを確認するために使うことを意図しています。ユーザはこの操作で、間違った呼び出し順をしている、つまり現在のインスタンスではない `Context` を切り離すこと、かどうかを確認できます。間違った呼び出し順をしている時は、この操作はエラーをログに出力したりエラー値を返したりするなど、誤った呼び出し順をユーザに警告するためのシグナルを発することができます。

<!--
The API MUST accept the following parameters:
-->

APIは以下のパラメータを受け付ける必要があります(MUST):

<!--
- A `Token` that was returned by a previous call to attach a `Context`.
-->

- `Context`を付与するための前の呼び出しで返された `Token`

<!--
The API MAY return a value used to check whether the operation
was successful or not.
-->

APIは、操作が成功したかを確認するための値を返しても構いません(MAY)。