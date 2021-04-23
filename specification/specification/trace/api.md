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

トレーシングAPIは、いくつかのメインクラスから成り立っています:

- `Tracer` はすべての操作に使われます。[Tracer](#tracer)節を参照してください。
- `Span` は現在の操作の実行についての情報を格納するミュータブルなオブジェクトです。[Span](#span)節を参照してください。

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

### 時間

OpenTelemetryでは、ナノ秒(ns)の精度までの時間の値を操作することができます。
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
実装は、単一でグローバルなデフォルトの `TracerProvider` を提供する必要があります(SHOULD)。

アプリケーションによっては、複数の `TracerProvider` インスタンスを使用することがあります。
例えば、それぞれのインスタンスに異なる設定(例えば`SpanProcessor`)を提供したり、
その結果として、それらのインスタンスによって生成された`Tracer`インスタンスを提供したりするためです。

- `name` (必須): nameは、計装されるライブラリ *ではなく* 、計装するライブラリ
  (`io.opentelemetry.contrib.mongodb` などの、インテグレーションとも呼ばれるもの) を識別しなければなりません。
  無効な名前 (nullまたは空文字列) が指定された場合は、nullを返したり例外をスローせず、
  フォールバックとして作業用であるデフォルトのTracerの実装を返します。
  OpenTelemetry APIを実装しているライブラリが「名前付き」機能をサポートしていない場合(例えばオブザーバビリティに関係のない実装など)、
  この名前を無視して、すべての呼び出しに対してデフォルトのインスタンスを返す *かもしれません* 。
  アプリケーションの所有者がこのライブラリによって生成されたテレメトリを抑制するようにSDKを設定している場合、
  TracerProviderは何もしないTracerを返すこともできます。
- `version` (オプション): 計装ライブラリのバージョンを指定します(例: `semver:1.0.0`)。

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
呼び出しを`TracerProvider`の新しいインスタンスを作成する関数呼び出しに委譲しすることによって`Provider`コンポーネントを分離することもでき、
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

`Tracer` は可能な限り、以下のメソッドを提供する必要があります(SHOULD):

- 現在アクティブな `Span` を取得する
- 与えられた `Span` をアクティブにする

Tracerは内部的に`Context`を利用して、現在の`Span`の状態と、どのように`Span`が
プロセスの境界を超えるかについて、取得および設定しなければなりません(MUST)。

現在のSpanを取得する際、現在アクティブな `Span` が存在しない場合は、
`Tracer`は無効な`SpanContext`とともに、一時的な`Span`を返さなければなりません(MUST)。

新しい`Span`を作成する際、`Tracer`は、呼び出し元が新しい`Span`の親を、
`Span`または`SpanContext`の形式で指定できるようにしなければなりません(MUST)。
`Tracer`は、明示的な親が与えられていたり、親を指定せずにSpanを作成するオプションが選択されたり、
現在アクティブな`Span`が無効である以外の場合には、
新しい`Span`または`SpanContext`をそれぞれアクティブな`Span`の子として作成する必要があります(SHOULD)。

`Tracer`はアクティブな`Span`を更新する方法を提供する必要があり(SHOULD)、
`Span`をアクティブにして、`Span`のライフタイムとスコープを管理するための便利な関数を提供することもできます(MAY)。
アクティブな`Span`が非アクティブになる際、それまでアクティブだった`Span`はアクティブになる必要があります(SHOULD)。
ある`Span`は終了している(つまり、nullではない終了時刻を持つ)が、まだアクティブかもしれません。
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

`TraceId`として有効なTrace識別子は16バイトの配列で、少なくとも1つは0以外のバイトです。

`SpanId`として有効なSpan識別子は8バイトの配列で、少なくとも1つの0以外のバイトです。

`TraceFlags`は、トレースに関する詳細を格納します。
Tracestate値と異なり、TraceFlagsはすべてのトレースに存在します。
現在、`TraceFlags`は真偽値`sampled`[フラグ](https://www.w3.org/TR/trace-context/#trace-flags)のみを持っています。

`Tracestate`は、システム固有の設定データをキーと値のペアの配列として保持します。
TraceStateにより、複数のトレースシステムが同じトレースを扱えるようになります。

`IsValid`は真偽値で、SpanContextが0ではないTraceIDと0以外のSpanIDを持っている場合にtrueを返します。

`IsRemote`は真偽値で、SpanContextがリモートの親から伝搬された場合にtrueを返します。
リモートのSpanから子を作成する場合、その子のIsRemoteフラグはfalseにセットされなければなりません(MUST)。

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
各TraceにはルートSpanが含まれており、通常はエンドツーエンドのレイテンシーと、オプションで、
サブオペレーションのための1つ以上のサブSpanを表しています。

`Span`は次のようにカプセル化されます:

- Span名
- `Span`をユニークに特定する、イミュータブルな [`SpanContext`](#spancontext)
- [`Span`](#span), [`SpanContext`](#spancontext)のペアの形で指定される親Span(nullの場合もあります)
- [`SpanKind`](#spankind)
- 開始タイムスタンプ
- 終了タイムスタンプ
- [`Attribute`](#set-attributes)の順序付きマッピング
- 他の`Span`への[`Link`](#add-links)のリスト
- タイムスタンプ付きの[`Event`](#add-events) のリスト
- [`Status`](#set-status)

_Span名_ は人間が読める文字列で、Spanが行う作業を簡潔に識別するためのものです。
例えば、RPCメソッド名、関数名、サブタスク名や大規模な計算の中でのステージの名前です。
Span名は、個別のSpanインススタンスを表すよりも、 _Spanのクラス_ を(統計的に)識別できるような、
最も一般的な文字列である必要があります。
つまり、 "get_user" は妥当な名前であり、"314159"をユーザーIDとしたときの"get_user/314159"
のような名前はカーディナリティが高く、良い名前ではありません。

例えば、仮のアカウント情報を取得するためのエンドポイントに対するSpan名は以下が考えられます:

| Span名                   | ガイダンス                                                              |
| ------------------------- | ---------------------------------------------------------------------- |
| `get`                     | 一般的すぎます。                                                         |
| `get_account/42`          | 限定しすぎています。                                                      |
| `get_account`             | 良い名前です。Span Attributeとして account_id=42 を指定とするとよさそうです。 |
| `get_account/{accountId}` | これも良い名前です。("HTTP route"を使う *)                               |

(* 訳注:
WEBアプリケーションフレームワークにおけるルーティング設定のようなものを使います。
フレームワークによっては`get_account/:accountId`のように表現することもあるものです。
つまり、`{accountId}`部分は文字列`{accountId}`そのものであり、`42`などの実際のIDを名前が埋め込まれるわけではありません。)

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

- Span名。これは必須のパラメータです。
- 親の`Span`、または親の`Span`か`SpanContext`を含む`Context`。
  および、新しい`Span`がルートの`Span`であるべきかどうか。
  APIは現在のコンテキストから暗黙的に親へと変換するときの、
  デフォルトの動作のためのオプションを取ることもできます(MAY)。
  明示的及び暗黙的な`Context`から`Span`の親への変換に関しては、
  [Contextからの親Spanの決定](#determining-theparent-span-from-a-context)を参照してください。
- [`SpanKind`](#spankind)。指定されていない場合はデフォルトで`SpanKind.Internal`となります。
- 複数の`Attribute` - キーと値のペアの配列で、[Span::SetAttributes](#set-attributes)で
  設定可能なものと同じセマンティクスを持ちます。
  さらに、それらのAttributeは、[サンプリングの説明](sdk.md#sampling)で示されているように、
  サンプリングの説明を付けるために使うこともできます。
  指定されていない場合、空のコレクションとして扱われます。

  可能な限り、ユーザーは、後から`SetAttribute`を呼び出すのではなく、
  Spanの作成時に既に知ることができるAttributeをセットする必要があります(SHOULD)。

- 複数の`Link` - [APIの定義](#add-links)を参照してください。
  指定されていない場合、空のリストとして扱われます。
- `Start timestamp`は、デフォルトは現在の時刻です。
  この引数は、Spanの作成時間がすでに経過している場合にのみセットされる必要があります(SHOULD)。
  Spanが論理的に開始する瞬間にAPIが呼ばれた場合、APIのユーザーはこの引数を明示的に設定してはなりません(MUST NOT)。

因果関係のある操作を表現するために、各Spanは、0または1個の親Spanと、0個以上の子Spanを持ちます。
関連するSpanのツリーはTraceを含みます。
親のないSpanは _ルートSpan_ と呼ばれます。
各Traceには単一のルートSpanが含まれており、Trace内のその他すべてのSpanの共有の先祖になります。
実装は`Span`をルートSpanとして作成するオプションを提供しなければならず(MUST)、
各ルートSpanに対して新しい`TraceId`を生成しなければなりません(MUST)。
親を持つSpanは、`TraceId`は親と同じものでなければなりません(MUST)。
また、子Spanはデフォルトで親の`TraceState`値をすべて継承しなければなりません(MUST)。

ある`Span`が別のプロセス内で作られた`Span`の子である場合、
その `Span` は _リモートの親_ を持っていると呼ばれます。
各Propagatorのデシリアライズでは、親の`SpanContext`の`IsRemote`をtrueに設定することで、
`Span`作成で親がリモートであることを示すことができます。

<!--
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
-->

#### Contextから親Spanを決定する

新しい`Span`が`Context`から作成された場合、その`Context`には次のものを含むことができます:

- 現在の`Span`
- 抽出された`SpanContext`
- 現在の`Span`と抽出された `SpanContext`
- 現在の`Span`および抽出された `SpanContext`のどちらも含まない

親は、以下の優先順位で選択する必要があります:

- 可能であれば、現在の`Span`を使う
- 可能であれば、抽出した`SpanContext`を使う
- 親がない場合、ルート`Span`を作成する

<!--
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
-->

#### Linkの追加

`Span`を作成する際には、ユーザーが他の `Span`とのLinkを記録する機能を持たなければなりません(MUST)。
Linkされた`Span`は同じTraceから来ている場合もあれば、異なるTraceから来ている場合もあります。
[Linkの説明](./overview.md#links-between-spans)を参照ください。

`Link`は以下のプロパティで定義されます:

- (必須)Link先の`Span`の`SpanContext`
- (オプション)[Span Attribute](#set-attributes)で定義されているものと同じ制約を持つ、1つ以上の`Attribute`

`Link`はイミュータブルな型である必要があります(SHOULD)。

Span作成APIは、以下のものを提供しなければなりません:

- `link`のプロパティを引数として受け取る、単一の`Link`を記録するAPI。
  このAPIは`AddLink`とも呼ぶことができます(MAY)。
- AttributeやAttributeの値を遅延構築する、単一の`Link`を記録するAPI。
  これは、Linkが使用されていない場合に不要な作業を回避することを目的としています。
  言語がオーバーロードをサポートしている場合にはこのAPIは`AddLink`と呼ぶ必要がありますが(SHOULD)、
  オーバーロードがサポートされていなければ`AddLazyLink`と呼ぶこともできます(MAY)。
  言語によっては、`Link`やフォーマットされたAttributeを返すラッピングクラスや関数を提供することで、
  `Link`や`Attribute`の作成を完全に遅延させる方が簡単な場合があります。
  ラッピングクラスや関数を提供する場合、`LinkFormatter`と名付けられる必要があります(SHOULD)。

Linkは設定された順番を保持する必要があります(SHOULD)。

<!--
### Span operations

With the exception of the function to retrieve the `Span`'s `SpanContext` and
recording status, none of the below may be called after the `Span` is finished.
-->

### Spanの操作

`Span`の`SpanContext`と記録状態を取得する関数を除いて、
以下のいずれも`Span`が終了したあとに呼び出すことはできません。

<!--
#### Get Context

The Span interface MUST provide:

- An API that returns the `SpanContext` for the given `Span`. The returned value
  may be used even after the `Span` is finished. The returned value MUST be the
  same for the entire Span lifetime. This MAY be called `GetContext`.
-->

#### Contextの取得

Spanは次のインタフェースを提供しなければなりません(MUST):

- 与えられた`Span`の`SpanContext`を返すAPI。
  返却値は`Span`が終了した後でも使うことができます。
  返却値はSpanのライフタイム全体に渡って同じものでなければなりません(MUST)。
  `GetContext`という呼び出しにすることもできます(MAY)。

<!--
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
-->

#### IsRecording

`Span`が、`AddEvent`の操作や、`SetAttributes`を用いたAttribute、
`SetStatus`を用いたStatusの操作等のEvent情報を記録している場合に、Trueを返します。

引数を取ってはいけません

このフラグは、Spanが記録されないことが明白な場合に、SpanのAttributeやEventの計算コストの
発生を避けるために使われる必要があります(SHOULD)。
任意の子Spanの記録は、このフラグ値とは独立して決定されることに注意してください
(通常、`SpanContext`にある`TraceFlag`の`sampled`フラグに基づきます)。

<!--
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
-->

#### Attributeのセット

`Span`は、自身に関連付けられるAttributeをセットできなければなりません(MUST)。

`Attribute`は次のプロパティによって定義されます:

- (必須)Attributeのキー。`null`や空文字ではない文字列でなければなりません(MUST)。
- (必須)Attributeの値。次のどちらかになります:
  - プリミティブ型: 文字列、真偽値、数値
  - プリミティブ型の配列: 配列は均一でなければなりません(MUST)。
    つまり、異なる型の値を含んではなりません(MUST NOT)。

Spanは次のインタフェースを提供しなければなりません(MUST):

- Attributeのプロパティを引数として受け取る、単一の`Attribute`をセットするAPI。
  このAPIは`SetAttribute`と呼ぶこともできます(MAY)。
  実装によっては、余分な割り当てを防ぐために、可能な値の型ごとに別のAPIを提供することもできます。

Attributeはセットされた順序を保持する必要があります(SHOULD)。
既存のAttributeと同じキーを持つAttributeをセットする場合、
既存のAttributeの値を上書きする必要があります(SHOULD)。

ゼロの整数または空文字列を表すAttributeの値は意味がある値とみなされるため、
保持され、Span Processor/Exporterに渡さなければなりません(MUST)。
`null`の値を持つAttributeは、セットされていないものとみなされており、
値が取得されたとしても、一度も`SetAttribute`されていないように扱われます。
例外として、値の上書きがサポートされている場合には、
結果として過去の値をクリアし、Attributeの集合からAttributeのキーを削除することになります。

配列の中の`null`値は、そのまま保持されなければなりません(MUST)
(つまり、Span Processer/Exporterに`null`として渡されます)。
Exporterが`null`値のエクスポートをサポートしていない場合、
その値は0、`false`、空文字に置き換えることもできます(MAY)。
マップやディクショナリ構造を表現するために、2つの配列をインデックスを使って同期させる
(つまり、2つのAttributeとして`header_keys`と`header_values`という2つの文字列配列を使って、
`header_keys[i] -> header_values[i]`というマッピングを表現する)ときに必要となります。

OpenTelemetryプロジェクトは、特定の["標準的なAttriubte"](semantic_conventions/README.md)に対して、
所定の意味を持たせています。

<!--
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
-->

#### Eventの追加

`Span`はeventを追加できる能力を持たなければなりません(MUST)。
Eventは`Span`に追加された瞬間に関連付けられた時間を持ちます。

`Event`は次のプロパティを持ちます:

- (必須)Event名
- (オプション)[Span Attributes](#set-attributes)で定義されたものと同じ制約を持つ、1個以上の`Attribute`
- (オプション)Eventのタイムスタンプ

`Event`はイミュータブル型である必要があります(SHOULD)。

Spanインタフェースは次のものを提供しなければなりません(MUST):
- `Event`のプロパティを引数として受け取る、単一の`Event`を記録するAPI。`AddEvent`と呼ぶこともできます(MAY)。
- Attributeやその値を遅延構築する、単一の`Event`を記録するAPI。
  これは、Eventが使用されていない場合に不要な作業を回避することを目的としています。
  言語がオーバーロードをサポートしている場合にはこのAPIは`AddEvent`と呼ぶ必要がありますが(SHOULD)、
  オーバーロードがサポートされていなければ`AddLazyEvent`と呼ぶこともできます(MAY)。
  言語によっては、`Event`やフォーマットされたAttributeを返すラッピングクラスや関数を提供することで、
  `Event`や`Attribute`の作成を完全に遅延させる方が簡単な場合があります。
  ラッピングクラスや関数を提供する場合、`EventFormatter`と名付けられる必要があります(SHOULD)。

Eventは設定された順番を保持する必要があります(SHOULD)。
これは通常、Eventのタイムスタンプの順序と対応します。

OpenTelemetryプロジェクトでは、所定の意味を持たせている、
特定の["標準のEvent名とKey"](semantic_conventions/README.md)について文書化してますので、注意してください。


<!--
#### Set Status

Sets the [`Status`](#status) of the `Span`. If used, this will override the
default `Span` status, which is `OK`.

Only the value of the last call will be recorded, and implementations are free
to ignore previous calls.

The Span interface MUST provide:

- An API to set the `Status` where the new status is the only argument. This
  SHOULD be called `SetStatus`.
-->

#### Statusのセット

`Span`の[`Statsus`](#status)をセットします。
使われた場合、デフォルトの`Span` Statusである`OK`を上書きします。

最後の呼び出しの値だけが記録され、実装はそれ以前の呼び出しを無視することができます。

Spanは次のインタフェースを提供しなければなりません(MUST):

- 新しいStatusのみを引数として受け取る、`Status`をセットするAPI。
  このAPIは`SetStatus`と呼ぶ必要があります(SHOULD)。

<!--
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
-->

#### 名前の更新

`Span`名を更新します。
この更新により、`Span`名に基づくすべてのサンプリングの動作は、実装に依存することになります。

作成後に`Span`名を更新することは、強くお勧めしません。
`Span`名は通常、論理的なSpanのグループをグループ化したり、フィルタリングしたり、識別するために使われます。
そして多くの場合、パフォーマンス上の都合により、フィルターのロジックは`Span`作成の前に実装されます。
したがって、名前の更新はロジックに干渉する可能性があります。

関数名は`UpdateName`と呼ばれ、通常のプロパティのセッターとは区別させています。
この操作は、`Span`の主要な変更を意味しており、実装に応じて以前に行われた
サンプリングまたはフィルタリングの決定の再計算につながる可能性があることを強調しています。

名前の更新の代替案は、`Span`の遅延作成です。
`Span`名がわかった時点、もしくは子`Span`として名前をレポートするような時点で、
過去の明示的にタイムスタンプを与えてSpanをスタートさせます。

必要なパラメーター:

- 新しい **Span名**、これは`Span`が開始されたときに渡されたものよりも優先されます。

<!--
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
-->

#### End

`Span`を終了させます。この呼び出しにより、`Span`の終了時刻として現在時刻がセットされます。
実装は、それ以降の`End`の呼び出しをすべて無視しなければなりません(MUST)
(Tracerがイベントのストリーミングをしていて、`Span`に関連したミュータブルな状態を持つときは、例外になるかもしれません)。

`Span`の`End`の呼び出しは、子Spanに影響を与えてはなりません(MUST NOT)。
子Spanは実行中であり、後で終了するかもしれません。

パラメータ:

- (オプショナル)明示的に終了時刻をセットする場合のタイムスタンプ

このAPIはノンブロッキングでなければなりません(MUST)。

<!--
### Span lifetime

Span lifetime represents the process of recording the start and the end
timestamps to the Span object:

- The start time is recorded when the Span is created.
- The end time needs to be recorded when the operation is ended.

Start and end time as well as Event's timestamps MUST be recorded at a time of a
calling of corresponding API.
-->

### Spanのライフタイム

Spanのライフタイムは、Spanオブジェクトに対して開始と終了のタイムスタンプを記録するプロセスを表します。

- 開始時刻はSpanが作成されたときに記録されます。
- 終了時刻は操作が終了したときに記録される必要があります。

開始時刻と終了時刻はEventのタイムスタンプと同様に、関連するAPIの呼び出しの時刻を記録されなければなりません(MUST)

<!--
## Status

`Status` interface represents the status of a finished `Span`. It's composed of
a canonical code in conjunction with an optional descriptive message.
-->

## Status

`Status`インタフェースは終了した`Span`の状態を表します。
これは、正規化されたコードとオプションの説明メッセージの組み合わせで構成されています。

<!--
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
-->

### StatusCanonicalCode

`StatusCanonicalCode`は終了した`Span`に関する正規化されたステータスコードの集合を表し、
[Standard GRPC codes](https://github.com/grpc/grpc/blob/master/doc/statuscodes.md)に従っています:


- `Ok`
  - 操作は正常に終了しました。
- `Cancelled`
  - 操作は(主に呼び出し元よって)キャンセルされました。
- `Unknown`
  - 未知のエラー。
- `InvalidArgument`
  - クライアントは無効な引数を指定しました。`FailedPrecondition`とは異なることに注意してください。
    `InvalidArgument`はシステムの状態に関係なく、問題のある引数を示します。
- `DeadlineExceeded`
  - 操作が完了する前に期限が切れました。
    システムの状態を変更する操作の場合、操作が正常に完了しても、このエラーが返されることがあります。
- `NotFound`
  - リクエストされたエンティティ(ファイルやディレクトリなど)が見つかりません。
- `AlreadyExists`
  - 作成しようとしたエンティティ(ファイルやディレクトリなど)はすでに存在しています。
- `PermissionDenied`
  - 呼び出し元は指定した操作を実行するための権限を持っていません。
    `PermissionDenied`は呼び出し元が識別できないときは使ってはいけません
    (そのようなときは替わりに`Unahtenticated`を使いましょう)。
- `ResourceExhausted`
  - リソース(ユーザーごとの割り当てやファイルシステム全体の空き容量など)が枯渇しています。
- `FailedPrecondition`
  - システムは操作を実行するための必要な状態ではないため、操作が拒否されました。
- `Aborted`
  - 操作が中断されました。通常は、トランザクションの中断やシーケンサーチェックの失敗などの並行処理の問題が原因になります。
- `OutOfRange`
  - 有効な範囲を超えた操作(ファイル終端を超えた走査や読み取りなど)が試行されました。
    `InvalidArgument`と違い、システムの状態が変わった場合、解消されるかもしれません。
- `Unimplemented`
  - 操作はこのサービスにおいて実装されていないか、サポートされていない、または有効になっていません。
- `Internal`
  - 内部エラー。基礎となるシステムによって期待されるいくつかの不変条件が壊れていることを意味します。
- `Unavailable`
  - サービスは現在利用できません。
    これはおそらく一時的な状態であり、バックオフを使って再試行することで修正できることもできます。
- `DataLoss`
  - 復旧不可能なデータの損失または破損。
- `Unauthenticated`
  - リクエストは、操作に対する有効な認証情報がありません。

<!--
### Status creation

API MUST provide a way to create a new `Status`.

Required parameters

- `StatusCanonicalCode` of this `Status`.

Optional parameters

- Description of this `Status`.
-->

### Statusの作成

APIは、新しい`Status`を作成する手段を提供しなくてはなりません(MUST)。

必要なパラメータ

- その`Status`の`StatusCanonicalCode`

オプションのパラメータ

- その`Status`の説明

<!--
### GetCanonicalCode

Returns the `StatusCanonicalCode` of this `Status`.
-->

# GetCanonicalCode

その`Status`の`StatusCanonicalCode`を返します。

<!--
### GetDescription

Returns the description of this `Status`.
Languages should follow their usual conventions on whether to return `null` or an empty string here if no description was given.
-->

### GetDescription

その`Status`の説明を返します。
説明が与えられていない場合に`null`を返すか空の文字列を返すかについては、言語の通常の慣習に従う必要があります。

<!--
### GetIsOk

Returns true if the canonical code of this `Status` is `Ok`, otherwise false.
-->

### GetIsOk

その`Status`の正規化コードが`Ok`の場合にtrueを返し、それ以外の場合にfalseを返します。

<!--
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
-->

## SpanKind

`SpanKind`は、Trace内での親子関係を記述します。
`SpanKind`は、トレーシングシステムで分析の際に利点となる、2つの独立したプロパティを記述します。

`SpanKind`が記述する1つめのプロパティは、Spanがリモートの子または親かを反映します。
リモートの親を持つSpanの興味深い部分は、外部負荷の発生源となるからです。
リモートの子を持つSpanの興味深い部分は、ローカルシステムではない依存が反映されているからです。

`SpanKind`が記述する2つめのプロパティは、子Spanが非同期呼び出しかどうかを反映します。
子Spanが非同期の場合、親は通常の状況下で完了するのを待つよう期待されています。
非同期Spanはトレースレイテンシー全体に貢献することがあるため、このプロパティを知ることは、トレースシステムにとって有用です。
非同期のシナリオはリモートとローカルどちらもありえます。

`SpanKind`が意味のあるものにするために、呼び出し元は単一のSpanは1つ以上の目的のサービスを提供しないよう、調整する必要があります。
例えば、サーバーサイドSpanは、他のリモートSpanの親として直接使ってはいけません。
簡単なガイドラインとしては、計装はリモートコール用のSpanContextを抽出したりシリアライズしたりする前に、新しいSpanを作成する必要があります。

使用できるSpanKindの値は次の通りです:

* `SERVER`は、そのSpanが非同期のRPCやその他のリモートリクエストなどの、サーバーサイドでの処理をカバーしていることを示します。
  このSpanは、リモートの`CLIENT`としてレスポンスを待っていると思われるSpanの子になります。
* `CLIENT`は、そのSpanがリモートサービスへ非同期にリクエストしていることを示します。
  このSpanはリモートの`SERVER` Spanの親であり、レスポンスを待ちます。
* `PRODUCER`は、そのSpanが非同期のリクエストの親であることを示します。
  この親Spanは、関連する子の`CUNSUMER` Spanの前、もしくは、場合によっては子Spanが開始する前に終了していることが期待されます。
  メッセージのバッチ処理でのシナリオでは、個々のメッセージは、メッセージの単位で新しい`PRODUCER` Spanが作成される必要があります。
* `CONSUMER`は、そのSpanが非同期の`PRODUCER`リクエストの子であることを示します。
* `INTERNAL`は、デフォルトの値であり、リモート操作の親や子とは対照的に、そのSpanがアプリケーションの内部操作であることを表します。

これらの種類の解釈をまとめると以下のようになります:

| `SpanKind` | 同期 | 非同期 | リモート受信 | リモート発信 |
|--|--|--|--|--|
| `CLIENT` | yes | | | yes |
| `SERVER` | yes | | yes | |
| `PRODUCER` | | yes | | maybe |
| `CONSUMER` | | yes | maybe | |
| `INTERNAL` | | | | |
