<!--
# Tracing API
-->

# トレーシング API

<details>
<summary>
目次
</summary>

* [Data types](#data-types)
  * [Time](#time)
    * [Timestamp](#timestamp)
    * [Duration](#duration)
* [Tracer](#tracer)
  * [Obtaining a tracer](#obtaining-a-tracer)
  * [Tracer operations](#tracer-operations)
* [SpanContext](#spancontext)
* [Span](#span)
  * [Span creation](#span-creation)
    * [Determining the Parent Span from a Context](#determining-the-parent-span-from-a-context)
    * [Add Links](#add-links)
  * [Span operations](#span-operations)
    * [Get Context](#get-context)
    * [IsRecording](#isrecording)
    * [Set Attributes](#set-attributes)
    * [Add Events](#add-events)
    * [Set Status](#set-status)
    * [UpdateName](#updatename)
    * [End](#end)
  * [Span lifetime](#span-lifetime)
* [Status](#status)
  * [StatusCanonicalCode](#statuscanonicalcode)
  * [Status creation](#status-creation)
  * [GetCanonicalCode](#getcanonicalcode)
  * [GetDescription](#getdescription)
  * [GetIsOk](#getisok)
* [SpanKind](#spankind)

</details>

<!--
Tracing API consist of a few main classes:

- `Tracer` is used for all operations. See [Tracer](#tracer) section.
- `Span` is a mutable object storing information about the current operation
   execution. See [Span](#span) section.
-->

トレーシングAPIは、いくつかのメインクラスで成り立っています:

- `Tracer` はすべての操作に使われます。[Tracer](#tracer)節を参照。
- `Span` は現在の操作の実行についての情報を格納する可変オブジェクトです。[Span](#span)節を参照。

<!--
## Data types

While languages and platforms have different ways of representing data,
this section defines some generic requirements for this API.
-->

## データ型

言語やプラットフォームによってデータの表現方法は異なりますが、
この節では、このAPIの一般的な要件を定義します。

<!--
### Time

OpenTelemetry can operate on time values up to nanosecond (ns) precision.
The representation of those values is language specific.
-->

### Time

OpenTelemetryでは、ナノ秒（ns）の精度までの時間の値を操作することができます。
これらの値の表現は言語固有のものです。

<!--
#### Timestamp

A timestamp is the time elapsed since the Unix epoch.

* The minimal precision is milliseconds.
* The maximal precision is nanoseconds.
-->

#### Timestamp

TimestampはUnixエポックからの経過時間です。

* 最低精度はミリ秒です。
* 最高精度はナノ秒です。

<!--
#### Duration

A duration is the elapsed time between two events.

* The minimal precision is milliseconds.
* The maximal precision is nanoseconds.
-->

#### Duration

Durationは、2つのイベント間の経過時間です。

* 最低精度はミリ秒です。
* 最高精度はナノ秒です。

<!--
## Tracer

The OpenTelemetry library achieves in-process context propagation of `Span`s by
way of the `Tracer`.

The `Tracer` is responsible for tracking the currently active `Span`, and
exposes functions for creating and activating new `Span`s. The `Tracer` is
configured with `Propagator`s which support transferring span context across
process boundaries.
-->

## Tracer

OpenTelemetryライブラリは`Tracer`を用いて、`Span`のプロセス内部でのContext伝搬を実現します。

`Tracer`は現在アクティブな`Span`を追跡する責任を持ち、新しい`Span`を作成したりアクティブにしたりするための関数を公開します。
`Tracer`は`Propagator`で構成されており、プロセスの境界を越えてSpanのContextを転送することをサポートしています。

<!--
### Obtaining a Tracer

New `Tracer` instances can be created via a `TracerProvider` and its `getTracer`
function. This function expects two string arguments:

`TracerProvider`s are generally expected to be used as singletons. Implementations
SHOULD provide a single global default `TracerProvider`.

Some applications may use multiple `TracerProvider` instances, e.g. to provide
different settings (e.g. `SpanProcessor`s) to each of those instances and -
in further consequence - to the `Tracer` instances created by them.

- `name` (required): This name must identify the instrumentation library (also
  referred to as integration, e.g. `io.opentelemetry.contrib.mongodb`) and *not*
  the instrumented library.
  In case an invalid name (null or empty string) is specified, a working
  default Tracer implementation as a fallback is returned rather than returning
  null or throwing an exception.
  A library, implementing the OpenTelemetry API *may* also ignore this name and
  return a default instance for all calls, if it does not support "named"
  functionality (e.g. an implementation which is not even observability-related).
  A TracerProvider could also return a no-op Tracer here if application owners configure
  the SDK to suppress telemetry produced by this library.
- `version` (optional): Specifies the version of the instrumentation library
  (e.g. `semver:1.0.0`).

Implementations might require the user to specify configuration properties at
`TracerProvider` creation time, or rely on external configuration, e.g. when using the
provider pattern.
-->

### Tracerの取得

新しい`Tracer`インスタンスは`TracerProvider`とその関数`getTracer`を介して作成することができます。
この関数は2つの文字列の引数を取ります:

`TracerProvider`は一般的にシングルトンとして使用されることが期待されます。
実装は、単一でグローバルなデフォルトの `TracerProvider` を提供すべきです(SHOULD)。

アプリケーションによっては、複数の `TracerProvider` インスタンスを使用することがあります。
例えば、それぞれのインスタンスに異なる設定(例えば`SpanProcessor`)を提供したり、
その結果として、それらのインスタンスによって生成された`Tracer`インスタンスを提供したりするためです。

- `name` (必須): nameは、計装されるライブラリではなく、計装するライブラリ (インテグレーションとも呼ばれます。
  `io.opentelemetry.contrib.mongodb` など) を識別しなければなりません。
  無効な名前 (nullまたは空文字列) が指定された場合は、nullを返したり例外をスローせず、フォールバックとして作業用であるデフォルトのTracerの実装を返します。
  OpenTelemetry APIを実装しているライブラリが「名前付き」機能をサポートしていない場合（例えば、オブザーバビリティに関係のない実装など）、
  この名前を無視して、すべての呼び出しに対してデフォルトのインスタンスを返すかもしれません。
  アプリケーションの所有者がこのライブラリによって生成されたテレメトリーを抑制するようにSDKを設定している場合、TracerProviderは何もしないTracerを返すこともできます。
- `version` (オプション): 計装ライブラリのバージョンを指定します（例: `semver:1.0.0`）。

実装では、`TracerProvider`の作成時にユーザが設定プロパティを指定することを要求したり、
プロバイダパターンを使用する場合など、外部設定に依存したりすることがあります。

<!--
#### Runtimes with multiple deployments/applications

Runtimes that support multiple deployments or applications might need to
provide a different `TracerProvider` instance to each deployment. To support this,
the global `TracerProvider` registry may delegate calls to create new instances of
`TracerProvider` to a separate `Provider` component, and the runtime may include
its own `Provider` implementation which returns a different `TracerProvider` for
each deployment.

`Provider` instances are registered with the API via some language-specific
mechanism, for instance the `ServiceLoader` class in Java.
-->

#### 複数のデプロイメント/アプリケーションを持つランタイム

複数のデプロイメントやアプリケーションをサポートするランタイムは、
各デプロイメントに異なる`TracerProvider`インスタンスを提供する必要があるかもしれません。
これをサポートするために、グローバルの`TracerProvider`レジストリは、
呼び出しを`TracerProvider`の新しいインスタンスを作成する委譲し、`Provider`コンポーネントを分離することもでき、
ランタイムは各デプロイメントごとに異なる`TracerProvider`を返す独自の`Provider`実装を含めることができます。

`Provider`インスタンスは、Javaの`ServiceLoader`クラスなど、言語固有のメカニズムを介してAPIに登録されます。

<!--
### Tracer operations

The `Tracer` MUST provide functions to:

- Create a new `Span`

The `Tracer` SHOULD provide methods to:

- Get the currently active `Span`
- Make a given `Span` as active

The `Tracer` MUST internally leverage the `Context` in order to get and set the
current `Span` state and how `Span`s are passed across process boundaries.

When getting the current span, the `Tracer` MUST return a placeholder `Span`
with an invalid `SpanContext` if there is no currently active `Span`.

When creating a new `Span`, the `Tracer` MUST allow the caller to specify the
new `Span`'s parent in the form of a `Span` or `SpanContext`. The `Tracer`
SHOULD create each new `Span` as a child of its active `Span` unless an
explicit parent is provided or the option to create a span without a parent is
selected, or the current active `Span` is invalid.

The `Tracer` SHOULD provide a way to update its active `Span` and MAY provide
convenience functions to manage a `Span`'s lifetime and the scope in which a
`Span` is active. When an active `Span` is made inactive, the previously-active
`Span` SHOULD be made active. A `Span` maybe finished (i.e. have a non-null end
time) but still active. A `Span` may be active on one thread after it has been
made inactive on another.
-->

### Tracerの操作

`Tracer` は以下の関数を提供しなければなりません(MUST):

- 新しい `Span` を作成する

`Tracer` は以下のメソッドを提供する必要があります(SHOULD):

- 現在アクティブな `Span` を取得する
- 与えられた `Span` をアクティブにする

Tracerは内部的に`Context`を利用して、現在の`Span`の状態と、どのように`Span`がプロセスの境界を超えるるかについて、取得および設定しなければなりません(MUST)。

現在のSpanを取得する際、現在アクティブな `Span` が存在しない場合は、`Tracer`は無効な`SpanContext`とともに、一時的な`Span`を返さなければなりません(MUST)。

新しい`Span`を作成する際、`Tracer`は、呼び出し元が新しい`Span`の親を`Span`または`SpanContext`の形式で指定できるようにしなければなりません(MUST)。
`Tracer`は、明示的な親が与えられていたり、親を指定せずにSpanを作成するオプションが選択されたり、現在アクティブな`Span`が無効である以外の場合には、
新しい`Span`または`SpanContext`をそれぞれアクティブな`Span`の子として作成する必要があります(SHOULD)。

`Tracer`はアクティブな`Span`を更新する方法を提供する必要があり(SHOULD)、
`Span`をアクティブにして、`Span`のライフタイムとスコープを管理するための便利な関数を提供することもできます(MAY)。
アクティブな`Span`が非アクティブになる際、それまでアクティブだった`Span`はアクティブになる必要があります(SHOULD)。
ある`Span`は終了している（つまり、nullではない終了時刻を持つ）が、まだアクティブかもしれません。
ある`Span`は、別のスレッドで非アクティブになったあとで、一つのスレッドでアクティブになることもできます。

<!--
## SpanContext

A `SpanContext` represents the portion of a `Span` which must be serialized and
propagated along side of a distributed context. `SpanContext`s are immutable.
`SpanContext` MUST be a final (sealed) class.

The OpenTelemetry `SpanContext` representation conforms to the [w3c TraceContext
specification](https://www.w3.org/TR/trace-context/). It contains two
identifiers - a `TraceId` and a `SpanId` - along with a set of common
`TraceFlags` and system-specific `TraceState` values.

`TraceId` A valid trace identifier is a 16-byte array with at least one
non-zero byte.

`SpanId` A valid span identifier is an 8-byte array with at least one non-zero
byte.

`TraceFlags` contain details about the trace. Unlike Tracestate values,
TraceFlags are present in all traces. Currently, the only `TraceFlags` is a
boolean `sampled`
[flag](https://www.w3.org/TR/trace-context/#trace-flags).

`Tracestate` carries system-specific configuration data, represented as a list
of key-value pairs. TraceState allows multiple tracing systems to participate in
the same trace.

`IsValid` is a boolean flag which returns true if the SpanContext has a non-zero
TraceID and a non-zero SpanID.

`IsRemote` is a boolean flag which returns true if the SpanContext was propagated
from a remote parent.
When creating children from remote spans, their IsRemote flag MUST be set to false.

Please review the W3C specification for details on the [Tracestate
field](https://www.w3.org/TR/trace-context/#tracestate-field).
-->


## SpanContext

`SpanContext`は`Span`の一部分であり、分散コンテキストと一緒にシリアライズされ、伝搬されなければなりません。
`SpanContext`はイミュータブルです。`SpanContext` はfinal (sealed)クラスでなければなりません(MUST)。

OpenTelemetryの`SpanContext`表現は[w3c TraceContext 仕様](https://www.w3.org/TR/trace-context/)に準拠しています。
これには2つの識別子`TraceId`と`SpanId`と、共通の`TraceFlags`値とシステム固有の`TraceState`値のセットが含まれています。

`TraceId` 有効なTrace識別子は、16バイトの配列で、少なくとも1つは0以外のバイトです。

`SpanId` 有効なスパン識別子は、8バイトの配列で、少なくとも1つの0以外のバイトです。


****************
`TraceFlags` はトレースに関する詳細を格納する。
Tracestate の値とは異なります。TraceFlags はすべてのトレースに存在します。
現在、唯一の `TraceFlags` は ブール値 `sampled` フラグ](https://www.w3.org/TR/trace-context/#trace-flags)。

`Tracestate` はシステム固有の設定データを保持します。キーと値のペアの TraceState は、複数のトレースシステムが 同じトレースを使用しています。

`IsValid` は、スパンコンテキストが 0 以外の値の TraceID と、0 以外の SpanID を指定します。

`IsRemote` は、スパンコンテキストが伝播された場合に真を返すブール値フラグです。の子をリモートの親から作成します。
リモートスパンから子を作成する場合、その子のIsRemoteフラグをfalseに設定しなければなりません(MUST)。

W3Cの仕様について、詳細は[Tracestate field](https://www.w3.org/TR/trace-context/#tracestate-field)を参照してください。

<!--
## Span

A `Span` represents a single operation within a trace. Spans can be nested to
form a trace tree. Each trace contains a root span, which typically describes
the end-to-end latency and, optionally, one or more sub-spans for its
sub-operations.

`Span`s encapsulate:

- The span name
- An immutable [`SpanContext`](#spancontext) that uniquely identifies the
  `Span`
- A parent span in the form of a [`Span`](#span), [`SpanContext`](#spancontext),
  or null
- A [`SpanKind`](#spankind)
- A start timestamp
- An end timestamp
- An ordered mapping of [`Attribute`s](#set-attributes)
- A list of [`Link`s](#add-links) to other `Span`s
- A list of timestamped [`Event`s](#add-events)
- A [`Status`](#set-status).

The _span name_ is a human-readable string which concisely identifies the work
represented by the Span, for example, an RPC method name, a function name,
or the name of a subtask or stage within a larger computation. The span name
should be the most general string that identifies a (statistically) interesting
_class of Spans_, rather than individual Span instances. That is, "get_user" is
a reasonable name, while "get_user/314159", where "314159" is a user ID, is not
a good name due to its high cardinality.

For example, here are potential span names for an endpoint that gets a
hypothetical account information:

| Span Name         | Guidance     |
| ----------------- | ------------ |
| `get`             | Too general  |
| `get_account/42`  | Too specific |
| `get_account`     | Good, and account_id=42 would make a nice Span attribute |
| `get_account/{accountId}` | Also good (using the "HTTP route") |

The `Span`'s start and end timestamps reflect the elapsed real time of the
operation. A `Span`'s start time SHOULD be set to the current time on [span
creation](#span-creation). After the `Span` is created, it SHOULD be possible to
change the its name, set its `Attribute`s, and add `Link`s and `Event`s. These
MUST NOT be changed after the `Span`'s end time has been set.

`Span`s are not meant to be used to propagate information within a process. To
prevent misuse, implementations SHOULD NOT provide access to a `Span`'s
attributes besides its `SpanContext`.

Vendors may implement the `Span` interface to effect vendor-specific logic.
However, alternative implementations MUST NOT allow callers to create `Span`s
directly. All `Span`s MUST be created via a `Tracer`.
-->

## Span

`Span`はTrace内の単一の操作を表します。Spanは入れ子にして、Traceのツリーを形成できます。
各TraceにはルートSpanが含まれており、通常はエンドツーエンドのレイテンシと、オプションで、サブオペレーションのための1つ以上のサブSpanを表しています。

`Span`は次のようにカプセル化されます:

- Span名
- `Span`をユニークに特定する、不変の [`SpanContext`](#spancontext)
- [`Span`](#span), [`SpanContext`](#spancontext)のペアの形で指定される親Span（nullの場合もあります）
- [`SpanKind`](#spankind)
- 開始タイムスタンプ
- 終了タイムスタンプ
- [`Attribute`](#set-attributes)の順序付きマッピング
- 他の`Span`への[`Link`](#add-links)のリスト
- タイムスタンプ付きの[`Event`](#add-events) のリスト
- [`Status`](#set-status)

_Span名_ は人間が読める文字列で、Spanが行う作業を簡潔に識別するためのものです。
例えば、RPCメソッド名、関数名、サブタスク名や大規模な計算の中でのステージの名前です。
Span名は、個別のSpanインススタンスを表すよりも、 _Spanのクラス_ を（統計的に）識別できるような、最も一般的な文字列である必要があります。
つまり、 "get_user" は妥当な名前であり、"314159"をユーザーIDとしたときの "get_user/314159" のような名前はカーディナリティが高く、良い名前ではありません。

例えば、仮のアカウント情報を取得するためのエンドポイントに対する潜在的なSpan名は:

| スパン名                   | ガイダンス                                                              |
| ------------------------- | ---------------------------------------------------------------------- |
| `get`                     | 一般的すぎます。                                                         |
| `get_account/42`          | 限定しすぎています。                                                      |
| `get_account`             | 良い名前です。Span Attributeとして account_id=42 を指定とするとよさそうです。 |
| `get_account/{accountId}` | これも良い名前です。（"HTTP route"を使う)                                  |

`Span`の開始と終了のタイムスタンプには、操作の実際の経過時間が反映されます。
`Span`の開始時間は[Span作成](#span-creation)時点での時刻がセットされる必要があります(SHOULD)。
`Span`が作成されたあと、Span名は変更、複数の`Attribute`の設定、`Link`および`Event`の追加が可能である必要があります(SHOULD)。
`Span`の終了時間がセットされたあとは、これらは変更してはいけません(MUST NOT)。

`Span`はプロセス内で情報を伝播するために使用することを意図していません。
誤用を防ぐために、実装は`Span`の`SpanContext`の他、Attributeへのアクセスを提供するべきではありません(SHOULD NOT)。

ベンダーは`Span`インタフェースを実装して、ベンダー独自のロジックを実現することもできます。
しかし、代替の実装は呼び出し元が直接`Span`を作成することを許容してはいけません(MUST NOT)。
すべての`Span`は`Tracer`を通じて作成しなければなりません(MUST)。

<!--
### Span Creation

Implementations MUST provide a way to create `Span`s via a `Tracer`. By default,
the currently active `Span` is set as the new `Span`'s parent. The `Tracer`
MAY provide other default options for newly created `Span`s.

`Span` creation MUST NOT set the newly created `Span` as the currently
active `Span` by default, but this functionality MAY be offered additionally
as a separate operation.

The API MUST accept the following parameters:

- The span name. This is a required parameter.
- The parent `Span` or a `Context` containing a parent `Span` or `SpanContext`,
  and whether the new `Span` should be a root `Span`. API MAY also have an
  option for implicit parenting from the current context as a default behavior.
  See [Determining the Parent Span from a Context](#determining-the-parent-span-from-a-context)
  for guidance on `Span` parenting from explicit and implicit `Context`s.
- [`SpanKind`](#spankind), default to `SpanKind.Internal` if not specified.
- `Attribute`s - A collection of key-value pairs, with the same semantics as
  the ones settable with [Span::SetAttributes](#set-attributes). Additionally,
  these attributes may be used to make a sampling decision as noted in [sampling
  description](sdk.md#sampling). An empty collection will be assumed if
  not specified.

  Whenever possible, users SHOULD set any already known attributes at span creation
  instead of calling `SetAttribute` later.

- `Link`s - see API definition [here](#add-links). Empty list will be assumed if
  not specified.
- `Start timestamp`, default to current time. This argument SHOULD only be set
  when span creation time has already passed. If API is called at a moment of
  a Span logical start, API user MUST not explicitly set this argument.

Each span has zero or one parent span and zero or more child spans, which
represent causally related operations. A tree of related spans comprises a
trace. A span is said to be a _root span_ if it does not have a parent. Each
trace includes a single root span, which is the shared ancestor of all other
spans in the trace. Implementations MUST provide an option to create a `Span` as
a root span, and MUST generate a new `TraceId` for each root span created.
For a Span with a parent, the `TraceId` MUST be the same as the parent.
Also, the child span MUST inherit all `TraceState` values of its parent by default.

A `Span` is said to have a _remote parent_ if it is the child of a `Span`
created in another process. Each propagators' deserialization must set
`IsRemote` to true on a parent `SpanContext` so `Span` creation knows if the
parent is remote.
-->

### Spanの作成

実装は、`Tracer`を使って`Span`を作成する方法を提供しなければなりません(MUST)。
デフォルトでは、現在のアクティブな`Span`が新しい`Span`の親に設定されます。
`Tracer`は、新しく作成された`Span`に他のデフォルトオプションを提供することもできます(MAY)。

`Span`の作成は、新しく作成された`Span`を現在のアクティブな`Span`として設定してはいけません(MUST NOT)が、
この機能は別の操作として追加で提供することはできます(MAY)。

APIは、以下のパラメータを受け取らなければなりません(MUST)。

- Spa名。これは必須のパラメータです。
- 親の`Span`、または親の`Span`か`SpanContext`を含む`Context`。および、新しい`Span`がルートの`Span`であるべきかどうか。
  APIは現在のコンテキストから暗黙的に親化(??? `parenting`の良い訳がほしい)ときの、デフォルトの動作のためのオプションを取ることもできます(MAY)。
  明示的及び暗黙的な`Context`からの`Span`の親化に関するガイダンスは、  [Contextからの親Spanの決定](#determining-theparent-span-from-a-context)を参照してください。
- [`SpanKind`](#spankind)。指定されていない場合はデフォルトで`SpanKind.Internal`となります。
- 複数の`Attribute` - キーと値のペアのコレクションで、[Span::SetAttributes](#set-attributes)で設定可能なものと同じセマンティクスを持ちます。
  さらに、それらのAttributeは、[サンプリングの説明](sdk.md#sampling)で示されているように、サンプリングの説明を付けるために使うこともできます。
  指定されていない場合、空のコレクションとして扱われます。

  可能な限り、ユーザーは、後から`SetAttribute`を呼び出すのではなく、Spanのの作成時に既に知ることができるAttributeを設定すべきです(SHOULD)。

- 複数の`Link` - [APIの定義](#add-links)を参照してください。
  指定されていない場合、空のリストとして扱われます。
- `Start timestamp`は、デフォルトは現在の時刻です。
  この引数は、Spanの作成時間がすでに経過している場合にのみ設定されるべきです(SHOULD)。
  Spanが論理的に開始する瞬間にAPIが呼ばれた場合、APIのユーザーはこの引数を明示的に設定してはなりません(MUST NOT)。(??? 原文ではMUST notになってて扱いがこまる)

因果関係のある操作を表現するために、各Spanは、0または1個の親Spanと、0個以上の子Spanを持ちます。
関連するSpanのツリーはTraceを含みます。
親のないSpanは _ルートSpan_ と呼ばれます。
各Traceには単一のルートSpanが含まれており、Trace内のその他すべてのSpanの共有の先祖になります。
実装は`Span`をルートSpanとして作成するオプションを提供しなければならず(MUST)、各ルートSpanに対して新しい`TraceId`を生成しなければなりません(MUST)。
親を持つSpanは、`TraceId`は親と同じものでなければなりません(MUST)。
また、子スパンはデフォルトで親の`TraceState`値をすべて継承しなければなりません(MUST)。

ある`Span`が別のプロセス内で作られた`Span`の子である場合、その `Span` は _リモート親_ (??? remote parentの訳！)を持っていると言われます。
各Propagatorのデシリアライズでは、親の`SpanContext`の`IsRemote`をtrueに設定することで、`Span`作成で親がリモートであることを示すことができます。


#### Determining the Parent Span from a Context

When a new `Span` is created from a `Context`, the `Context` may contain:

- A current `Span`
- An extracted `SpanContext`
- A current `Span` and an extracted `SpanContext`
- Neither a current `Span` nor an extracted `Span` context

The parent should be selected in the following order of precedence:

- Use the current `Span`, if available.
- Use the extracted `SpanContext`, if available.
- There is no parent. Create a root `Span`.

#### Add Links

During the `Span` creation user MUST have the ability to record links to other `Span`s. Linked
`Span`s can be from the same or a different trace. See [Links
description](../overview.md#links-between-spans).

A `Link` is defined by the following properties:

- (Required) `SpanContext` of the `Span` to link to.
- (Optional) One or more `Attribute`s with the same restrictions as defined for
  [Span Attributes](#set-attributes).

The `Link` SHOULD be an immutable type.

The Span creation API should provide:

- An API to record a single `Link` where the `Link` properties are passed as
  arguments. This MAY be called `AddLink`.
- An API to record a single `Link` whose attributes or attribute values are
  lazily constructed, with the intention of avoiding unnecessary work if a link
  is unused. If the language supports overloads then this SHOULD be called
  `AddLink` otherwise `AddLazyLink` MAY be considered. In some languages, it might
  be easier to defer `Link` or attribute creation entirely by providing a wrapping
  class or function that returns a `Link` or formatted attributes. When providing
  a wrapping class or function it SHOULD be named `LinkFormatter`.

Links SHOULD preserve the order in which they're set.

### Span operations

With the exception of the function to retrieve the `Span`'s `SpanContext` and
recording status, none of the below may be called after the `Span` is finished.

#### Get Context

The Span interface MUST provide:

- An API that returns the `SpanContext` for the given `Span`. The returned value
  may be used even after the `Span` is finished. The returned value MUST be the
  same for the entire Span lifetime. This MAY be called `GetContext`.

#### IsRecording

Returns true if this `Span` is recording information like events with the
`AddEvent` operation, attributes using `SetAttributes`, status with `SetStatus`,
etc.

There should be no parameter.

This flag SHOULD be used to avoid expensive computations of a Span attributes or
events in case when a Span is definitely not recorded. Note that any child
span's recording is determined independently from the value of this flag
(typically based on the `sampled` flag of a `TraceFlag` on
[SpanContext](#spancontext)).

This flag may be `true` despite the entire trace being sampled out. This
allows to record and process information about the individual Span without
sending it to the backend. An example of this scenario may be recording and
processing of all incoming requests for the processing and building of
SLA/SLO latency charts while sending only a subset - sampled spans - to the
backend. See also the [sampling section of SDK design](sdk.md#sampling).

Users of the API should only access the `IsRecording` property when
instrumenting code and never access `SampledFlag` unless used in context
propagators.

#### Set Attributes

A `Span` MUST have the ability to set attributes associated with it.

An `Attribute` is defined by the following properties:

- (Required) The attribute key, which MUST be a non-`null` and non-empty string.
- (Required) The attribute value, which is either:
  - A primitive type: string, boolean or numeric.
  - An array of primitive type values. The array MUST be homogeneous,
    i.e. it MUST NOT contain values of different types.

The Span interface MUST provide:

- An API to set a single `Attribute` where the attribute properties are passed
  as arguments. This MAY be called `SetAttribute`. To avoid extra allocations some
  implementations may offer a separate API for each of the possible value types.

Attributes SHOULD preserve the order in which they're set. Setting an attribute
with the same key as an existing attribute SHOULD overwrite the existing
attribute's value.

Attribute values expressing a numerical value of zero or an empty string are
considered meaningful and MUST be stored and passed on to span processors / exporters.
Attribute values of `null` are considered to be not set and get discarded as if
that `SetAttribute` call had never been made.
As an exception to this, if overwriting of values is supported, this results in
clearing the previous value and dropping the attribute key from the set of attributes.

`null` values within arrays MUST be preserved as-is (i.e., passed on to span
processors / exporters as `null`). If exporters do not support exporting `null`
values, they MAY replace those values by 0, `false`, or empty strings.
This is required for map/dictionary structures represented as two arrays with
indices that are kept in sync (e.g., two attributes `header_keys` and `header_values`,
both containing an array of strings to represent a mapping
`header_keys[i] -> header_values[i]`).

Note that the OpenTelemetry project documents certain ["standard
attributes"](semantic_conventions/README.md) that have prescribed semantic meanings.

#### Add Events

A `Span` MUST have the ability to add events. Events have a time associated
with the moment when they are added to the `Span`.

An `Event` is defined by the following properties:

- (Required) Name of the event.
- (Optional) One or more `Attribute`s with the same restrictions as defined for
  [Span Attributes](#set-attributes).
- (Optional) Timestamp for the event.

The `Event` SHOULD be an immutable type.

The Span interface MUST provide:

- An API to record a single `Event` where the `Event` properties are passed as
  arguments. This MAY be called `AddEvent`.
- An API to record a single `Event` whose attributes or attribute values are
  lazily constructed, with the intention of avoiding unnecessary work if an event
  is unused. If the language supports overloads then this SHOULD be called
  `AddEvent` otherwise `AddLazyEvent` MAY be considered. In some languages, it
  might be easier to defer `Event` or attribute creation entirely by providing a
  wrapping class or function that returns an `Event` or formatted attributes. When
  providing a wrapping class or function it SHOULD be named `EventFormatter`.

Events SHOULD preserve the order in which they're set. This will typically match
the ordering of the events' timestamps.

Note that the OpenTelemetry project documents certain ["standard event names and
keys"](semantic_conventions/README.md) which have prescribed semantic meanings.

#### Set Status

Sets the [`Status`](#status) of the `Span`. If used, this will override the
default `Span` status, which is `OK`.

Only the value of the last call will be recorded, and implementations are free
to ignore previous calls.

The Span interface MUST provide:

- An API to set the `Status` where the new status is the only argument. This
  SHOULD be called `SetStatus`.

#### UpdateName

Updates the `Span` name. Upon this update, any sampling behavior based on `Span`
name will depend on the implementation.

It is highly discouraged to update the name of a `Span` after its creation.
`Span` name is often used to group, filter and identify the logical groups of
spans. And often, filtering logic will be implemented before the `Span` creation
for performance reasons. Thus the name update may interfere with this logic.

The function name is called `UpdateName` to differentiate this function from the
regular property setter. It emphasizes that this operation signifies a major
change for a `Span` and may lead to re-calculation of sampling or filtering
decisions made previously depending on the implementation.

Alternatives for the name update may be late `Span` creation, when Span is
started with the explicit timestamp from the past at the moment where the final
`Span` name is known, or reporting a `Span` with the desired name as a child
`Span`.

Required parameters:

- The new **span name**, which supersedes whatever was passed in when the
  `Span` was started

#### End

Finish the `Span`. This call will take the current timestamp to set as `Span`'s
end time. Implementations MUST ignore all subsequent calls to `End` (there might
be exceptions when Tracer is streaming event and has no mutable state associated
with the `Span`).

Call to `End` of a `Span` MUST not have any effects on child spans. Those may
still be running and can be ended later.

Parameters:

- (Optional) Timestamp to explicitly set the end timestamp

This API MUST be non-blocking.

### Span lifetime

Span lifetime represents the process of recording the start and the end
timestamps to the Span object:

- The start time is recorded when the Span is created.
- The end time needs to be recorded when the operation is ended.

Start and end time as well as Event's timestamps MUST be recorded at a time of a
calling of corresponding API.

## Status

`Status` interface represents the status of a finished `Span`. It's composed of
a canonical code in conjunction with an optional descriptive message.

### StatusCanonicalCode

`StatusCanonicalCode` represents the canonical set of status codes of a finished
`Span`, following the [Standard GRPC
codes](https://github.com/grpc/grpc/blob/master/doc/statuscodes.md):

- `Ok`
  - The operation completed successfully.
- `Cancelled`
  - The operation was cancelled (typically by the caller).
- `Unknown`
  - An unknown error.
- `InvalidArgument`
  - Client specified an invalid argument. Note that this differs from
    `FailedPrecondition`. `InvalidArgument` indicates arguments that are problematic
    regardless of the state of the system.
- `DeadlineExceeded`
  - Deadline expired before operation could complete. For operations that change the
    state of the system, this error may be returned even if the operation has
    completed successfully.
- `NotFound`
  - Some requested entity (e.g., file or directory) was not found.
- `AlreadyExists`
  - Some entity that we attempted to create (e.g., file or directory) already exists.
- `PermissionDenied`
  - The caller does not have permission to execute the specified operation.
    `PermissionDenied` must not be used if the caller cannot be identified (use
    `Unauthenticated1` instead for those errors).
- `ResourceExhausted`
  - Some resource has been exhausted, perhaps a per-user quota, or perhaps the
    entire file system is out of space.
- `FailedPrecondition`
  - Operation was rejected because the system is not in a state required for the
    operation's execution.
- `Aborted`
  - The operation was aborted, typically due to a concurrency issue like sequencer
    check failures, transaction aborts, etc.
- `OutOfRange`
  - Operation was attempted past the valid range. E.g., seeking or reading past end
    of file. Unlike `InvalidArgument`, this error indicates a problem that may be
    fixed if the system state changes.
- `Unimplemented`
  - Operation is not implemented or not supported/enabled in this service.
- `Internal`
  - Internal errors. Means some invariants expected by underlying system has been
    broken.
- `Unavailable`
  - The service is currently unavailable. This is a most likely a transient
    condition and may be corrected by retrying with a backoff.
- `DataLoss`
  - Unrecoverable data loss or corruption.
- `Unauthenticated`
  - The request does not have valid authentication credentials for the operation.

### Status creation

API MUST provide a way to create a new `Status`.

Required parameters

- `StatusCanonicalCode` of this `Status`.

Optional parameters

- Description of this `Status`.

### GetCanonicalCode

Returns the `StatusCanonicalCode` of this `Status`.

### GetDescription

Returns the description of this `Status`.
Languages should follow their usual conventions on whether to return `null` or an empty string here if no description was given.

### GetIsOk

Returns true if the canonical code of this `Status` is `Ok`, otherwise false.

## SpanKind

`SpanKind` describes the relationship between the Span, its parents,
and its children in a Trace.  `SpanKind` describes two independent
properties that benefit tracing systems during analysis.

The first property described by `SpanKind` reflects whether the Span
is a remote child or parent.  Spans with a remote parent are
interesting because they are sources of external load.  Spans with a
remote child are interesting because they reflect a non-local system
dependency.

The second property described by `SpanKind` reflects whether a child
Span represents a synchronous call.  When a child span is synchronous,
the parent is expected to wait for it to complete under ordinary
circumstances.  It can be useful for tracing systems to know this
property, since synchronous Spans may contribute to the overall trace
latency. Asynchronous scenarios can be remote or local.

In order for `SpanKind` to be meaningful, callers should arrange that
a single Span does not serve more than one purpose.  For example, a
server-side span should not be used directly as the parent of another
remote span.  As a simple guideline, instrumentation should create a
new Span prior to extracting and serializing the span context for a
remote call.

These are the possible SpanKinds:

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

To summarize the interpretation of these kinds:

| `SpanKind` | Synchronous | Asynchronous | Remote Incoming | Remote Outgoing |
|--|--|--|--|--|
| `CLIENT` | yes | | | yes |
| `SERVER` | yes | | yes | |
| `PRODUCER` | | yes | | maybe |
| `CONSUMER` | | yes | maybe | |
| `INTERNAL` | | | | |
