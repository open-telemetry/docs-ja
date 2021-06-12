<!--
# Metrics API
-->

# Metrics API

**Status**: [Experimental](../document-status.md)

**Owner:**

* [Reiley Yang](https://github.com/reyang)

**Domain Experts:**

* [Bogdan Brutu](https://github.com/bogdandrutu)
* [Josh Suereth](https://github.com/jsuereth)
* [Joshua MacDonald](https://github.com/jmacd)

<!--
Note: this specification is subject to major changes. To avoid thrusting
language client maintainers, we don't recommend OpenTelemetry clients to start
the implementation unless explicitly communicated via
[OTEP](https://github.com/open-telemetry/oteps#opentelemetry-enhancement-proposal-otep).
-->

注意:この仕様は大きく変更される可能性があります。言語クライアントのメンテナに負担をかけないように、[OTEP](https://github.com/open-telemetry/oteps#opentelemetry-enhancement-proposal-otep)で明示
的に伝えられない限り、OpenTelemetryのクライアントが実装を開始することは推奨しません。

<details>
<summary>
目次
</summary>

<!--
* [Overview](#overview)
* [MeterProvider](#meterprovider)
  * [MeterProvider operations](#meterprovider-operations)
* [Meter](#meter)
  * [Meter operations](#meter-operations)
* [Instrument](#instrument)
  * [Counter](#counter)
    * [Counter creation](#counter-creation)
    * [Counter operations](#counter-operations)
  * [Asynchronous Counter](#asynchronous-counter)
    * [Asynchronous Counter creation](#asynchronous-counter-creation)
    * [Asynchronous Counter operations](#asynchronous-counter-operations)
  * [Asynchronous Gauge](#asynchronous-gauge)
    * [Asynchronous Gauge creation](#asynchronous-gauge-creation)
    * [Asynchronous Gauge operations](#asynchronous-gauge-operations)
  * [Histogram](#histogram)
    * [Histogram creation](#histogram-creation)
    * [Histogram operations](#histogram-operations)
  * [UpDownCounter](#updowncounter)
    * [UpDownCounter creation](#updowncounter-creation)
    * [UpDownCounter operations](#updowncounter-operations)
  * [Asynchronous UpDownCounter](#asynchronous-updowncounter)
    * [Asynchronous UpDownCounter creation](#asynchronous-updowncounter-creation)
    * [Asynchronous UpDownCounter operations](#asynchronous-updowncounter-operations)
* [Measurement](#measurement)
-->

* [概要](#概要)
* [MeterProvider](#meterprovider)
  * [MeterProviderの操作](#meterproviderの操作)
* [Meter](#meter)
  * [Meterの操作](#meterの操作)
* [Instrument](#instrument)
  * [Counter](#counter)
    * [Counterの作成](#counterの作成)
    * [Counterの操作](#counterの操作)
  * [非同期Counter](#非同期counter)
    * [非同期Counterの作成](#非同期counterの作成)
    * [非同期Counterの操作](#a非同期counterの操作)
  * [非同期Gauge](#非同期gauge)
    * [非同期Gaugeの作成](#非同期gaugeの作成)
    * [非同期Gaugeの操作](#非同期gaugeの操作)
  * [Histogram](#histogram)
    * [Histogramの作成](#histogramの作成)
    * [Histogramの操作](#histogramの操作)
  * [UpDownCounter](#updowncounter)
    * [UpDownCounterの作成](#updowncounterの作成)
    * [UpDownCounterの操作](#updowncounterの操作)
  * [非同期UpDownCounter](#asynchronous-updowncounter)
    * [非同期UpDownCounterの作成](#非同期updowncounterの作成)
    * [非同期UpDownCounterの操作](#非同期updowncounterの操作)
* [Measurement](#measurement)


</details>

<!--
## Overview
-->

## 概要

<!--
The Metrics API consists of these main components:
-->

Metrics APIは、以下の主要コンポーネントで構成されています。

<!--
* [MeterProvider](#meterprovider) is the entry point of the API. It provides
  access to `Meters`.
* [Meter](#meter) is the class responsible for creating `Instruments`.
* [Instrument](#instrument) is responsible for reporting
  [Measurements](#measurement).
-->

* [MeterProvider](#meterprovider)は、APIのエントリーポイントです。`Meter`へのアクセスを提供します。
* [Meter](#meter)は、`Instrument` の作成を担当するクラスです。
* [instrument](#instrument)は、[Measurements](#measurement)を報告する担当です。

<!--
Here is an example of the object hierarchy inside a process instrumented with
the metrics API:
-->

ここでは、メトリクスAPIで計測されたプロセス内のオブジェクト階層の例を示します。

<!--
```text
+-- MeterProvider(default)
    |
    +-- Meter(name='io.opentelemetry.runtime', version='1.0.0')
    |   |
    |   +-- Instrument<Asynchronous Gauge, int>(name='cpython.gc', attributes=['generation'], unit='kB')
    |   |
    |   +-- instruments...
    |
    +-- Meter(name='io.opentelemetry.contrib.mongodb.client', version='2.3.0')
        |
        +-- Instrument<Counter, int>(name='client.exception', attributes=['type'], unit='1')
        |
        +-- Instrument<Histogram, double>(name='client.duration', attributes=['net.peer.host', 'net.peer.port'], unit='ms')
        |
        +-- instruments...

+-- MeterProvider(custom)
    |
    +-- Meter(name='bank.payment', version='23.3.5')
        |
        +-- instruments...
```
-->

```text
+-- MeterProvider(default)
    |
    +-- Meter(name='io.opentelemetry.runtime', version='1.0.0')
    |   |
    |   +-- Instrument<Asynchronous Gauge, int>(name='cpython.gc', attributes=['generation'], unit='kB')
    |   |
    |   +-- instruments...
    |
    +-- Meter(name='io.opentelemetry.contrib.mongodb.client', version='2.3.0')
        |
        +-- Instrument<Counter, int>(name='client.exception', attributes=['type'], unit='1')
        |
        +-- Instrument<Histogram, double>(name='client.duration', attributes=['net.peer.host', 'net.peer.port'], unit='ms')
        |
        +-- instruments...

+-- MeterProvider(custom)
    |
    +-- Meter(name='bank.payment', version='23.3.5')
        |
        +-- instruments...
```

<!--
## MeterProvider
-->

## MeterProvider

<!--
`Meter`s can be accessed with a `MeterProvider`.
-->

`Meter`にアクセスするには`MeterProvider`を使用します。

<!--
In implementations of the API, the `MeterProvider` is expected to be the
stateful object that holds any configuration.
-->

APIの実装では、`MeterProvider`が、あらゆる設定を保持するステートフルなオブジェクトであることが期待されます。

<!--
Normally, the `MeterProvider` is expected to be accessed from a central place.
Thus, the API SHOULD provide a way to set/register and access a global default
`MeterProvider`.
-->

通常、`MeterProvider`は中央の場所(XXX: central placeの意味は？)からアクセスされることが期待されます。したがって、APIはグローバルなデフォルトの`MeterProvider`を設定/登録し、それにアクセスする方法を提供すべきです(SHOULD)。

<!--
Notwithstanding any global `MeterProvider`, some applications may want to or
have to use multiple `MeterProvider` instances, e.g. to have different
configuration for each, or because its easier with dependency injection
frameworks. Thus, implementations of `MeterProvider` SHOULD allow creating an
arbitrary number of `MeterProvider` instances.
-->

グローバルな `MeterProvider` にかかわらず、アプリケーションによっては、複数の `MeterProvider` インスタンスを使用したい、または使用しなければならない場合があります。例えば、それぞれに異なる構成を持たせたい場合や、依存性注入フレームワークを使用した方が簡単な場合などです。したがって、`MeterProvider`の実装は、任意の数の`MeterProvider`インスタンスの作成を可能にすべきです(SHOULD)。

<!--
### MeterProvider operations
-->

### MeterProvider 操作

<!--
The `MeterProvider` MUST provide the following functions:
-->

`MeterProvider`は、以下の機能を提供しなければなりません(MUST)。

<!--
* Get a `Meter`
-->

* `Meter`の取得

<!--
#### Get a Meter
-->

#### Meterの取得(Get)

<!--
This API MUST accept the following parameters:
-->

このAPIは、以下のパラメータを受け付けなければなりません。

<!--
* `name` (required): This name must identify the [instrumentation
  library](../overview.md#instrumentation-libraries) (e.g.
  `io.opentelemetry.contrib.mongodb`). If an application or library has built-in
  OpenTelemetry instrumentation, both [Instrumented
  library](../glossary.md#instrumented-library) and [Instrumentation
  library](../glossary.md#instrumentation-library) may refer to the same
  library. In that scenario, the `name` denotes a module name or component name
  within that library or application. In case an invalid name (null or empty
  string) is specified, a working Meter implementation MUST be returned as a
  fallback rather than returning null or throwing an exception, its `name`
  property SHOULD keep the original invalid value, and a message reporting that
  the specified value is invalid SHOULD be logged. A library, implementing the
  OpenTelemetry API *may* also ignore this name and return a default instance
  for all calls, if it does not support "named" functionality (e.g. an
  implementation which is not even observability-related). A MeterProvider could
  also return a no-op Meter here if application owners configure the SDK to
  suppress telemetry produced by this library.
* `version` (optional): Specifies the version of the instrumentation library
  (e.g. `1.0.0`).
-->

* `name` (必須)。この名前は、[計装ライブラリ](../overview.md#instrumentation-libraries)を特定する必要があります(例: `io.opentelemetry.contrib.mongodb`)。アプリケーションやライブラリにOpenTelemetryの計装機能が組み込まれている場合、[計装されるライブラリ](../glossary.md#instrumented-library)と[計装するライブラリ](../glossary.md#instrumentation-library)の両方が同じライブラリを参照していることがあります。その場合、`name`は、そのライブラリやアプリケーション内のモジュール名やコンポーネント名を表します。無効な名前(NULLまたは空の文字列)が指定された場合、NULLを返したり例外をスローするのではなく、フォールバックとして動作するMeterの実装が返されなければならず(MUST)、その`name`プロパティは元の無効な値を維持すべきで、指定された値が無効であることを報告するメッセージがログに記録されるべきです(SHOULD)。OpenTelemetry APIを実装しているライブラリは、「名前付き」機能をサポートしていない場合(例:観測性に関係のない実装など)、この名前を無視して、すべての呼び出しに対してデフォルトのインスタンスを返すことも*できます*。アプリケーションの所有者が、このライブラリで生成されるテレメトリを抑制するようにSDKを構成している場合、MeterProviderはここでno-op Meter(何もしないMeter)を返すこともできます。
* `バージョン`(任意)。計装ライブラリのバージョンを指定します(例:`1.0.0`)

<!--
It is unspecified whether or under which conditions the same or different
`Meter` instances are returned from this functions.
-->

この関数から、同じまたは異なる `Meter` インスタンスが返されるかどうか、またはどのような条件で返されるかは、定義されていません。

<!--
Implementations MUST NOT require users to repeatedly obtain a `Meter` again with
the same name+version to pick up configuration changes. This can be achieved
either by allowing to work with an outdated configuration or by ensuring that
new configuration applies also to previously returned `Meter`s.
-->

実装では、構成の変更を反映させるために、同じ名前とバージョンの `Meter` を繰り返し取得することをユーザーに要求してはなりません (MUST NOT)。これは、古い設定での作業を可能にするか、新しい設定が以前に返された `Meter` にも適用されるようにすることで実現できます。

<!--
Note: This could, for example, be implemented by storing any mutable
configuration in the `MeterProvider` and having `Meter` implementation objects
have a reference to the `MeterProvider` from which they were obtained. If
configuration must be stored per-meter (such as disabling a certain meter), the
meter could, for example, do a look-up with its name+version in a map in the
`MeterProvider`, or the `MeterProvider` could maintain a registry of all
returned `Meter`s and actively update their configuration if it changes.
-->

注意: これは例えば、変更可能な設定をすべて `MeterProvider` に保存し、`Meter` の実装オブジェクトに、取得元の `MeterProvider` への参照を持たせることで実装できます。設定をMeterごとに保存する必要がある場合 (特定のMeterを無効にするなど)、Meterは、たとえば、`MeterProvider`のマップで名前とバージョンを検索することができます。あるいは、`MeterProvider`は、返されたすべての`Meter`のレジストリを維持し、設定が変更された場合には積極的に更新することができます。

<!--
## Meter
-->

## Meter

<!--
The meter is responsible for creating [Instruments](#instrument).
-->

Meterは[Instrument](#instrument)の作成を担当しています。

<!--
Note: `Meter` SHOULD NOT be responsible for the configuration. This should be
the responsibility of the `MeterProvider` instead.
-->

注意: `Meter` は設定を行うべきではありません(SHOULD NOT)。これは `MeterProvider` が担当するべきです。

<!--
### Meter operations
-->

### Meter operations

<!--
The `Meter` MUST provide functions to create new [Instruments](#instrument):
-->

`Meter`は、新しい[Instrument](#instrument)を作成するための関数を提供しなければなりません(MUST)。

<!--
* [Create a new Counter](#counter-creation) (see the section on `Counter`)
-->

* [Counterの作成](#Counterの作成) (`Counter` のセクションを参照)

<!--
## Instrument
-->

## Instrument

<!--
Instruments are used to report [Measurements](#measurement). Each Instrument
will have the following information:
-->

Instrumentは、[測定値](#measurement)を報告するために使用されます。各Instrumentには以下の情報があります。

<!--
* The `name` of the Instrument
* The `kind` of the Instrument - whether it is a [Counter](#counter) or other
  instruments, whether it is synchronous or asynchronous
* An optional `unit of measure`
* An optional `description`
-->

* Instrumentの`name`
* Instrumentの`種類` - [Counter](#counter)であるか、その他のInstrumentであるか、同期であるか非同期であるか。
* 任意の `計測単位`
* 任意の `説明文`

<!--
Instruments are associated with the Meter during creation, and are identified by
the name:
-->

Instrumentは、作成時にMeterに関連付けられ、名前で識別されます。

<!--
* Meter implementations MUST return an error when multiple Instruments are
  registered under the same Meter instance using the same name.
* Different Meters MUST be treated as separate namespaces. The names of the
  Instruments under one Meter SHOULD NOT interfere with Instruments under
  another Meter.
-->

* Meter の実装では、同じ Meter インスタンスに同じ名前で複数の Instrument が登録されている場合、エラーを返さなければなりません(MUST)。
* 異なるMeterは別の名前空間として扱わなければなりません(MUST)。 あるメーターの機器の名前が、別のメーターの機器と干渉してはいけません(SHOULD NOT)。

<!--
<a name="instrument-naming-rule"></a>
-->

<a name="instrument-naming-rule"></a>

<!--
Instrument names MUST conform to the following syntax (described using the
[Augmented Backus-Naur Form](https://tools.ietf.org/html/rfc5234)):
-->

Instrument名は、以下の構文に準拠しなければなりません([Augmented Backus-Naur Form](https://tools.ietf.org/html/rfc5234)を使用して記述します)。

<!--
```abnf
instrument-name = ALPHA 0*62 ("_" / "." / "-" / ALPHA / DIGIT)

ALPHA = %x41-5A / %x61-7A; A-Z / a-z
DIGIT = %x30-39 ; 0-9
```
-->

```abnf
instrument-name = ALPHA 0*62 ("_" / "." / "-" / ALPHA / DIGIT)

ALPHA = %x41-5A / %x61-7A; A-Z / a-z
DIGIT = %x30-39 ; 0-9
```

<!--
* They are not null or empty strings.
* They are case-insensitive, ASCII strings.
* The first character must be an alphabetic character.
* Subsequent characters must belong to the alphanumeric characters, '_', '.',
  and '-'.
* They can have a maximum length of 63 characters.
-->

* nullや空の文字列ではないこと
* 大文字小文字を区別しないASCII文字列であること
* 最初の文字はアルファベットでなければならない
* 後続の文字は英数字である'_', '.', '-'でなければならない
* 文字列の長さは最大で63文字まで

<!--
<a name="instrument-unit"></a>
-->

<a name="instrument-unit"></a>

<!--
The `unit` is an optional string provided by the author of the instrument. It
SHOULD be treated as an oqaque string from the API and SDK (e.g. the SDK is not
expected to validate the unit of measurement, or perform the unit conversion).
-->

`unit`は、Instrumentの作者が提供するオプションの文字列です。これは、APIやSDKからはOpaque文字列として扱われるべきです(SHOULD)(例えば、SDKが測定単位を検証したり、単位変換を行ったりすることは期待されていません)。

<!--
* If the `unit` is not provided or the `unit` is null, the API and SDK MUST make
  sure that the behavior is the same as an empty `unit` string.
* It MUST be case-sensitive (e.g. `kb` and `kB` are different units), ASCII
  string.
* It can have a maximum length of 63 characters. The number 63 is chosen to
  allow the unit strings (includig the `\0` terminator on certain language
  runtimes) to be stored and compared as 8-bytes integers when performance is
  critical.
-->

* `unit` が提供されない場合や `unit` が NULL の場合、API および SDK は空の `unit` 文字列と同じ動作をするようにしなければなりません (MUST)。
* 大文字小文字を区別しASCII文字列でなければなりません(MUST)(例:`kb`と`kB`は異なる単位)
* 最大で63文字の長さになります。63という数字は、パフォーマンスが重視される場合に、単位文字列(特定の言語ランタイムでは `\0` というターミネーターを含む)を8バイトの整数として保存し、比較できるようにするために選ばれています。

<!--
<a name="instrument-description"></a>
-->

<a name="instrument-description"></a>

<!--
The `description` is an optional free-form text provided by the author of the
instrument. It MUST be treated as an oqaque string from the API and SDK.
-->

`description`は、Instrumentの作者が提供するオプションの自由形式テキストです。APIやSDKでは、Opaque文字列として扱われなければなりません(MUST)。

<!--
* If the `description` is not provided or the `description` is null, the API and
  SDK MUST make sure that the behavior is the same as an empty `description`
  string.
* It MUST support [BMP (Unicode Plane
  0)](https://en.wikipedia.org/wiki/Plane_(Unicode)#Basic_Multilingual_Plane),
  which is basically only the first three bytes of UTF-8 (or `utf8mb3`).
  Individual language clients can decide if they want to support more Unicode
  [Planes](https://en.wikipedia.org/wiki/Plane_(Unicode)).
* It MUST support at least 1023 characters. Individual language clients can
  decide if they want to support more.
-->

* `description` が提供されていないか、`description` が NULL の場合、API と SDK は、空の `description` 文字列と同じ動作をするようにしなければなりません (MUST)。
* [BMP (Unicode Plane 0)](https://en.wikipedia.org/wiki/Plane_(Unicode)#Basic_Multilingual_Plane)をサポートしなければなりません(MUST)。これは基本的にUTF-8(または`utf8mb3`)の最初の3バイトだけです。個々の言語のクライアントは、より多くのUnicode [Plane](https://en.wikipedia.org/wiki/Plane_(Unicode))をサポートしたいかどうかを決めることができます。
* 最低でも1023文字をサポートしなければなりません(MUST)。それ以上の文字数をサポートするかどうかは、個々の言語のクライアントが決めることができます。

<!--
### Counter
-->

### Counter

<!--
`Counter` is a synchronous Instrument which supports non-negative increments.
-->

`Counter`は、非負の増分をサポートする同期型のInstrumentです。

<!--
Example uses for `Counter`:
-->

`Counter`の使用例です。

<!--
* count the number of bytes received
* count the number of requests completed
* count the number of accounts created
* count the number of checkpoints run
* count the number of HTTP 5xx errors
-->

* 受信したバイト数のカウント
* 完了したリクエストの数をカウント
* 作成されたアカウント数のカウント
* チェックポイントの実行数をカウント
* HTTP 5xxエラーの数をカウント

<!--
#### Counter creation
-->

#### Counterの作成

<!--
There MUST NOT be any API for creating a `Counter` other than with a
[`Meter`](#meter). This MAY be called `CreateCounter`. If strong type is
desired, the client can decide the language idomatic name(s), for example
`CreateUInt64Counter`, `CreateDoubleCounter`, `CreateCounter<UInt64>`,
`CreateCounter<double>`.
-->

`Counter`を作成するためのAPIは、[`Meter`](#meter)以外にあってはなりません(MUST NOT)。これは `CreateCounter` と呼んでも構いません(MAY)。強い型が必要な場合は、クライアントは言語の慣用的な名前を決めることができます。例えば、`CreateUInt64Counter`, `CreateDoubleCounter`, `CreateCounter<UInt64>`, `CreateCounter<double>`などです。

<!--
The API MUST accept the following parameters:
-->

APIは、以下のパラメータを受け入れなければなりません(MUST):

<!--
* The `name` of the Instrument, following the [instrument naming
  rule](#instrument-naming-rule).
* An optional `unit of measure`, following the [instrument unit
  rule](#instrument-unit).
* An optional `description`, following the [instrument description
  rule](#instrument-description).
-->

* [Instrument命名規則](#Instrument命名規則)に従った、Instrumentの `名前`
* [Instrument単位規則](#Instrument単位規則)に従った、任意の `測定単位`
* [Instrument説明規則](#Instrument説明規則)に従った、任意の `説明`

<!--
Here are some examples that individual language client might consider:
-->

ここでは、個々の言語のクライアントが考慮すべきいくつかの例を紹介します:

```python
# Python

exception_counter = meter.create_counter(name="exceptions", description="number of exceptions caught", value_type=int)
```

```csharp
// C#

var counterExceptions = meter.CreateCounter<UInt64>("exceptions", description="number of exceptions caught");

readonly struct PowerConsumption
{
    [HighCardinality]
    string customer;
};

var counterPowerUsed = meter.CreateCounter<double, PowerConsumption>("power_consumption", unit="kWh");
```

<!--
#### Counter operations
-->

#### Counterの操作

<!--
##### Add
-->

##### Add

<!--
Increment the Counter by a fixed amount.
-->

カウンターを一定量だけ増加させる。

<!--
This API SHOULD NOT return a value (it MAY return a dummy value if required by
certain programming languages or systems, for example `null`, `undefined`).
-->

このAPIは、値を返すべきではありません(SHOULD NOT)(特定のプログラミング言語やシステムで必要とされる場合は、`null`や`undefined`などのダミーの値を返してもよい(MAY))。

<!--
Required parameters:
-->

必要なパラメータ:

<!--
* Optional [attributes](../common/common.md#attributes).
* The increment amount, which MUST be a non-negative numeric value.
-->

* オプションの [attributes](../common/common.md#attributes)
* インクリメント量は、非負の数値でなければなりません(MUST)

<!--
The client MAY decide to allow flexible
[attributes](../common/common.md#attributes) to be passed in as arguments. If
the attribute names and types are provided during the [counter
creation](#counter-creation), the client MAY allow attribute values to be passed
in using a more efficient way (e.g. strong typed struct allocated on the
callstack, tuple). The API MUST allow callers to provide flexible attributes at
invocation time rather than having to register all the possible attribute names
during the instrument creation. Here are some examples that individual language
client might consider:
-->

クライアントは柔軟な[attributes](../common/common.md#attributes)を引数として渡すことを許可してもかまいません(MAY)。属性名と型が [Counterの作成](#counter-creation) の際に提供される場合、クライアントは属性値をより効率的な方法(例えば、コールスタック上に割り当てられた強力な型付き構造体、タプル)で渡すことを許可してもかまいません(MAY)。APIは、呼び出し側が楽器作成時に可能な属性名をすべて登録するのではなく、呼び出し時に柔軟な属性を提供できるようにしなければなりません(MUST)。以下は、個々の言語クライアントが考慮すべき例です。
```python
# Python

exception_counter.Add(1, {"exception_type": "IOError", "handled_by_user": True})
exception_counter.Add(1, exception_type="IOError", handled_by_user=True})
```

```csharp
// C#

counterExceptions.Add(1, ("exception_type", "FileLoadException"), ("handled_by_user", true));

counterPowerUsed.Add(13.5, new PowerConsumption { customer = "Tom" });
counterPowerUsed.Add(200, new PowerConsumption { customer = "Jerry" }, ("is_green_energy", true));
```

<!--
### Asynchronous Counter
-->

### Asynchronous Counter

<!--
Asynchronous Counter is an asynchronous Instrument which reports
[monotonically](https://wikipedia.org/wiki/Monotonic_function) increasing
value(s) when the instrument is being observed.
-->

非同期Counter は非同期のInstrumentで、Instrumentが観察されているときに[単調に(モノトニック)](https://wikipedia.org/wiki/Monotonic_function)増加する値を報告します．

<!--
Example uses for Asynchronous Counter:
-->

Asynchronous Counterの使用例です。

<!--
* [CPU time](https://wikipedia.org/wiki/CPU_time), which could be reported for
  each thread, each process or the entire system. For example "the CPU time for
  process A running in user mode, measured in seconds".
* The number of [page faults](https://wikipedia.org/wiki/Page_fault) for each
  process.
-->

* [CPU時間](https://wikipedia.org/wiki/CPU_time) これは、各スレッド、各プロセス、またはシステム全体について報告することができます。例えば、「ユーザーモードで実行されているプロセスAのCPU時間(秒単位)」などです。
* 各プロセスの[ページフォルト](https://wikipedia.org/wiki/Page_fault)の数

<!--
#### Asynchronous Counter creation
-->

#### 非同期Counterの作成

<!--
There MUST NOT be any API for creating an Asynchronous Counter other than with a
[`Meter`](#meter). This MAY be called `CreateObservableCounter`. If strong type
is desired, the client can decide the language idomatic name(s), for example
`CreateUInt64ObservableCounter`, `CreateDoubleObservableCounter`,
`CreateObservableCounter<UInt64>`, `CreateObservableCounter<double>`.
-->

非同期Counterを作成するためのAPIは、[`Meter`](#meter)以外にあってはなりません(MUST NOT)。これは、`CreateObservableCounter`と呼んでも構いません(MAY)。強い型が必要な場合は、クライアントがその言語で慣用的な名前を決めることができます。例えば、`CreateUInt64ObservableCounter`, `CreateDoubleObservableCounter`, `CreateObservableCounter<UInt64>`, `CreateObservableCounter<double>`などです。

<!--
It is highly recommended that implementations use the name `ObservableCounter`
(or any language idiomatic variation, e.g. `observable_counter`) unless there is
a strong reason not to do so. Please note that the name has nothing to do with
[asynchronous
pattern](https://en.wikipedia.org/wiki/Asynchronous_method_invocation) and
[observer pattern](https://en.wikipedia.org/wiki/Observer_pattern).
-->

特に理由がない限り、`ObservableCounter`(または、`observable_counter`のような言語の慣用的なバリエーション)という名前を使用することを強く推奨します。この名前は[非同期パターン](https://en.wikipedia.org/wiki/Asynchronous_method_invocation)や[observerパターン](https://en.wikipedia.org/wiki/Observer_pattern)とは何の関係もないことに注意してください。



<!--
The API MUST accept the following parameters:
-->

APIは、以下のパラメータを受け入れなければなりません(MUST)。

<!--
* The `name` of the Instrument, following the [instrument naming
  rule](#instrument-naming-rule).
* An optional `unit of measure`, following the [instrument unit
  rule](#instrument-unit).
* An optional `description`, following the [instrument description
  rule](#instrument-description).
* A `callback` function.
-->

* [Instrument命名規則](#Instrument命名規則)に従った、Instrumentの `名前`
* [Instrument単位規則](#Instrument単位規則)に従った、任意の `測定単位`
* [Instrument説明規則](#Instrument説明規則)に従った、任意の `説明`
* `callback` 関数

<!--
The `callback` function is responsible for reporting the
[Measurement](#measurement)s. It will only be called when the Meter is being
observed. Individual language client SHOULD define whether this callback
function needs to be reentrant safe / thread safe or not.
-->

`callback`関数は、[Measurement](#measurement)の報告を行います。この関数は、Meterが観察されているときにのみ呼び出されます。各言語のクライアントは、このコールバック関数がリエントラントセーフ/スレッドセーフである必要があるかどうかを定義すべきです(SHOULD)。

<!--
Note: Unlike [Counter.Add()](#add) which takes the increment/delta value, the
callback function reports the absolute value of the counter. To determine the
reported rate the counter is changing, the difference between successive
measurements is used.
-->

注意:インクリメント/デルタ値を取得する[Counter.Add()](#add)とは異なり、コールバック関数はカウンタの絶対値を報告します。報告されたカウンタの変化率を判断するには、連続した測定値の差を使用します。

<!--
The callback function SHOULD NOT take indefinite amount of time. If multiple
independent SDKs coexist in a running process, they MUST invoke the callback
function(s) independently.
-->

コールバック関数は、不定の時間(XXX: 良い訳が欲しい)を要してはなりません(SHOULD NOT)。実行中のプロセスに複数の独立したSDKが共存している場合、それらのSDKは独立してコールバック関数を呼び出さなければなりません(MUST)。

<!--
Individual language client can decide what is the idomatic approach. Here are
some examples:
-->

どのようなアプローチをとるかは、各言語クライアント自身が決められます。いくつかの例を紹介します。


<!--
* Return a list (or tuple, generator, enumerator, etc.) of `Measurement`s.
* Use an observer argument to allow individual `Measurement`s to be reported.
-->

* `Measurement`のリスト(またはタプル、ジェネレーター、enumeratorなど)を返す
* 個々の `Measurement` を報告できるようにするには、observer 引数を使用する

<!--
User code is recommended not to provide more than one `Measurement` with the
same `attributes` in a single callback. If it happens, the
[SDK](./README.md#sdk) can decide how to handle it. For example, during the
callback invocation if two measurements `value=1, attributes={pid:4 bitness:64}`
and `value=2, attributes={pid:4, bitness:64}` are reported, the SDK can decide
to simply let them pass through (so the downstream consumer can handle
duplication), drop the entire data, pick the last one, or something else. The
API must treat observations from a single callback as logically taking place at
a single instant, such that when recorded, observations from a single callback
MUST be reported with identical timestamps.
-->

ユーザーコードでは、1つのコールバックで同じ `attributes` を持つ複数の `Measurement` を提供しないことが推奨されます。このような事態が発生した場合、[SDK](./README.md#sdk)はその処理方法を決定することができます。たとえば、コールバックの呼び出し中に、2つの測定値 `value=1, attributes={pid:4 bitness:64}` と `value=2, attributes={pid:4, bitness:64}` が報告された場合、SDKはそれらを単に通過させる(下流の消費者が重複を処理できるように)、データ全体をドロップする、最後の1つを選ぶ、あるいはそれ以外、などを決定することができます。APIは、単一のコールバックからの観測を、論理的には単一の瞬間に行われたものとして扱わなければなりません。そのため、記録される場合、単一のコールバックからの観測は同一のタイムスタンプで報告されなければなりません(MUST)。

<!--
The API SHOULD provide some way to pass `state` to the callback. Individual
language client can decide what is the idomatic approach (e.g. it could be an
additional parameter to the callback function, or captured by the lambda
closure, or something else).
-->

APIは、`state`をコールバックに渡すための何らかの方法を提供すべきです(SHOULD)。どのような方法が適切なのかは、各言語のクライアントが決めることができます (例えば、コールバック関数の追加パラメータにするとか、ラムダクロージャでキャプチャするとか、他の方法が考えられます)。

<!--
Here are some examples that individual language client might consider:
-->

ここでは、個々の言語のクライアントが考慮すべきいくつかの例を紹介します。

```python
# Python

def pf_callback():
    # 注:現実の世界では、これらはオペレーティングシステムから取得されます。
    return (
        (8,        ("pid", 0),   ("bitness", 64)),
        (37741921, ("pid", 4),   ("bitness", 64)),
        (10465,    ("pid", 880), ("bitness", 32)),
    )
meter.create_observable_counter(name="PF", description="process page faults", pf_callback)
```

```python
# Python

def pf_callback(result):
    # 注:現実の世界では、これらはオペレーティングシステムから取得されます。
    result.Observe(8,        ("pid", 0),   ("bitness", 64))
    result.Observe(37741921, ("pid", 4),   ("bitness", 64))
    result.Observe(10465,    ("pid", 880), ("bitness", 32))

meter.create_observable_counter(name="PF", description="process page faults", pf_callback)
```

```csharp
// C#

// 1つの値のみが報告されるシンプルなシナリオ

interface IAtomicClock
{
    UInt64 GetCaesiumOscillates();
}

IAtomicClock clock = AtomicClock.Connect();

meter.CreateObservableCounter<UInt64>("caesium_oscillates", () => clock.GetCaesiumOscillates());
```

<!--
#### Asynchronous Counter operations
-->

#### 非同期Counterの操作

<!--
Asynchronous Counter is only intended for an asynchronous scenario. The only
operation is provided by the `callback`, which is registered during the
[Asynchronous Counter creation](#asynchronous-counter-creation).
-->

非同期Counterは、非同期的なシナリオのみを想定しています。唯一の操作は、[非同期Counterの作成](#非同期Counterの作成)の際に登録される`callback`によって提供されます。

<!--
### Histogram
-->

### Histogram

<!--
`Histogram` is a synchronous Instrument which can be used to report arbitrary
values that are likely to be statistically meaningful. It is intended for
statistics such as histograms, summaries, and percentile.
-->

`Histogram`は、統計的に意味があると思われる任意の値を報告するのに使える同期型のInstrumentです．これはヒストグラム、サマリー、パーセンタイルなどの統計を目的としています．

<!--
Example uses for `Histogram`:
-->

`Histogram`の使用例:

<!--
* the request duration
* the size of the response payload
-->

* リクエスト期間
* レスポンスペイロードのサイズ

<!--
#### Histogram creation
-->

#### Histogramの作成

<!--
There MUST NOT be any API for creating a `Histogram` other than with a
[`Meter`](#meter). This MAY be called `CreateHistogram`. If strong type is
desired, the client can decide the language idomatic name(s), for example
`CreateUInt64Histogram`, `CreateDoubleHistogram`, `CreateHistogram<UInt64>`,
`CreateHistogram<double>`.
-->

`Histogram`を作成するためのAPIは、[`Meter`](#meter)以外にあってはなりません(MUST NOT)。これは`CreateHistogram`と呼んでも構いません(MAY)。強力な型が必要な場合は、クライアントがその言語で慣用的な名前を決めることができます。例えば、`CreateUInt64Histogram`、`CreateDoubleHistogram`、`CreateHistogram<UInt64>`、`CreateHistogram<double>`などです。

<!--
The API MUST accept the following parameters:
-->

APIは、以下のパラメータを受け入れなければなりません(MUST)。

<!--
* The `name` of the Instrument, following the [instrument naming
  rule](#instrument-naming-rule).
* An optional `unit of measure`, following the [instrument unit
  rule](#instrument-unit).
* An optional `description`, following the [instrument description
  rule](#instrument-description).
-->

* [Instrumentの命名のルール](#instrument-naming-rule)に従った、Instrumentsの `name`
* [Instrumentの単位のルール](#instrument-unit)に従った、任意の `unit of measure`
* [Instrumentの説明のルール](#instrument-description)に従った任意の`description`

<!--
Here are some examples that individual language client might consider:
-->

ここでは、個々の言語のクライアントが考慮すべきいくつかの例を紹介します。

```python
# Python

http_server_duration = meter.create_histogram(
    name="http.server.duration",
    description="measures the duration of the inbound HTTP request",
    unit="milliseconds",
    value_type=float)
```

```csharp
// C#

var httpServerDuration = meter.CreateHistogram<double>(
    "http.server.duration",
    description: "measures the duration of the inbound HTTP request",
    unit: "milliseconds"
    );
```

<!--
#### Histogram operations
-->

#### Histogramの操作

<!--
##### Record
-->

##### Record

<!--
Updates the statistics with the specified amount.
-->

指定した量の統計情報を更新します。

<!--
This API SHOULD NOT return a value (it MAY return a dummy value if required by
certain programming languages or systems, for example `null`, `undefined`).
-->

このAPIは、値を返すべきではありません(SHOULD NOT)(特定のプログラミング言語やシステムで必要とされる場合は、`null`や`undefined`などのダミーの値を返しても構いません(MAY))。

<!--
Parameters:
-->

引数:

<!--
* The amount of the `Measurement`.
* Optional [attributes](../common/common.md#attributes).
-->

* `Measurement` の量
* 任意の [属性](../common/common.md#attributes)

<!--
The client MAY decide to allow flexible
[attributes](../common/common.md#attributes) to be passed in as individual
arguments. The client MAY allow attribute values to be passed in using a more
efficient way (e.g. strong typed struct allocated on the callstack, tuple). Here
are some examples that individual language client might consider:
-->

クライアントはフレキシブルな[属性](../common/common.md#attributes)を個々の引数として渡すことを許可してもかまいません(MAY)。クライアントは属性値をより効率的な方法で渡すことを許可してもかまいません(例:コールスタックに割り当てられた強力な型付き構造体やタプルなど)。以下は、個々の言語のクライアントが考慮すべきいくつかの例です。

```python
# Python

http_server_duration.Record(50, {"http.method": "POST", "http.scheme": "https"})
http_server_duration.Record(100, http_method="GET", http_scheme="http"})
```

```csharp
// C#

httpServerDuration.Record(50, ("http.method", "POST"), ("http.scheme", "https"));
httpServerDuration.Record(100, new HttpRequestAttributes { method = "GET", scheme = "http" });
```

<!--
### Asynchronous Gauge
-->

### 非同期Gauge

<!--
Asynchronous Gauge is an asynchronous Instrument which reports non-additive
value(s) (_e.g. the room temperature - it makes no sense to report the
temperature value from multiple rooms and sum them up_) when the instrument is
being observed.
-->

非同期Gaugeは非加算値(_例:部屋の温度 - 複数の部屋の温度値を報告して合計するのは意味がない_)を、このInstrumentが観測されているときに報告する非同期型のInstrumentです。

<!--
Note: if the values are additive (_e.g. the process heap size - it makes sense
to report the heap size from multiple processes and sum them up, so we get the
total heap usage_), use [Asynchronous Counter](#asynchronous-counter) or
[Asynchronous UpDownCounter](#asynchronous-updowncounter).
-->

注意:値が加算される場合(_例:プロセスのヒープサイズ-複数のプロセスのヒープサイズを報告し、それらを合計することでヒープ使用量の合計を得ることに意味があります_)、[非同期Counter](#非同期counter)または[非同期UpDownCounter](#非同期updowncounter)を使用してください。

<!--
Example uses for Asynchronous Gauge:
-->

非同期型Gaugeの使用例

<!--
* the current room temperature
* the CPU fan speed
-->

* 現在の部屋の温度
* CPUファンの回転数

<!--
#### Asynchronous Gauge creation
-->

#### 非同期Gaugeの作成

<!--
TODO
-->

TODO

<!--
#### Asynchronous Gauge operations
-->

#### 非同期Gaugeの操作

<!--
Asynchronous Gauge is only intended for an asynchronous scenario. The only
operation is provided by the `callback`, which is registered during the
[Asynchronous Gauge creation](#asynchronous-gauge-creation).
-->

非同期Gaugeは、非同期的なシナリオのみを想定しています。唯一の操作は、[非同期Gaugeの作成](#非同期Gaugeの作成)の際に登録される`callback`によって行われます。

<!--
### UpDownCounter
-->

### UpDownCounter

<!--
`UpDownCounter` is a synchronous Instrument which supports increments and
decrements.
-->

`UpDownCounter`は加算と減算をサポートする同期型のInstrumentです。

<!--
Note: if the value grows
[monotonically](https://wikipedia.org/wiki/Monotonic_function), use
[Counter](#counter) instead.
-->

注意:値が[モノトニックに](https://wikipedia.org/wiki/Monotonic_function)成長する場合は、代わりに[Counter](#counter)を使用してください。

<!--
Example uses for `UpDownCounter`:
-->

`UpDownCounter`の使用例です。

<!--
* the number of active requests
* the number of items in a queue
-->

* アクティブなリクエストの数
* キュー内のアイテム数

<!--
An `UpDownCounter` is intended for scenarios where the absolute values are not
pre-calculated, or fetching the "current value" requires extra effort. If the
pre-calculated value is already available or fetching the snapshot of the
"current value" is straightforward, use [Asynchronous
UpDownCounter](#asynchronous-updowncounter) instead.
-->

`UpDownCounter`は、絶対値が事前に計算されていない場合や、「現在の値」を取得するのに余分な手間がかかる場合を想定しています。事前に計算された値が既に利用可能であったり、"現在の値"のスナップショットを取得するのが簡単な場合は、代わりに[非同期UpDownCounter](#非同期updowncounter)を使用してください。

<!--
Taking the **the size of a collection** as an example, almost all the language
runtimes would provide APIs for retrieving the size of a collection, whether the
size is internally maintained or calculated on the fly. If the intention is to
report the size that can be retrieved from these APIs, use [Asynchronous
UpDownCounter](#asynchronous-updowncounter).
-->

**コレクションのサイズ**を取得する場合を例にとると、ほとんどすべての言語ランタイムは、コレクションのサイズを取得するためのAPIを提供していますが、そのサイズが内部的に維持されているか、その場で計算されているかは問いません。これらのAPIから取得できるサイズを報告したい場合は、[非同期UpDownCounter](#非同期updowncounter)を使用してください。


```python
# Python
items = []

meter.create_observable_up_down_counter(
    name="store.inventory",
    description="the number of the items available",
    callback=lambda result: result.Observe(len(items)))
```

<!--
There are cases when the runtime APIs won't provide sufficient information, e.g.
reporting the number of items in a concurrent bag by the "color" and "material"
properties.
-->

ランタイムAPIでは十分な情報が得られない場合があります。例えば、「色」と「素材」のプロパティによって、同時使用のバッグのアイテム数を報告することができます。

<!--
| Color    | Material     | Count |
| -------- | -----------  | ----- |
| Red      | Aluminum     | 1     |
| Red      | Steel        | 2     |
| Blue     | Aluminum     | 0     |
| Blue     | Steel        | 5     |
| Yellow   | Aluminum     | 0     |
| Yellow   | Steel        | 3     |
-->

| 色       | 素材         | 数     |
| -------- | -----------  | ----- |
| 赤       | アルミニウム  | 1     |
| 赤       | 鉄            | 2     |
| 青       | アルミニウム  | 0     |
| 青       | 鉄            | 5     |
| 黄色     | アルミニウム | 0     |
| 黄色     | 鉄            | 3     |

```python
# Python
items_counter = meter.create_up_down_counter(
    name="store.inventory",
    description="the number of the items available")

def restock_item(color, material):
    inventory.add_item(color=color, material=material)
    items_counter.add(1, {"color": color, "material": material})
    return true

def sell_item(color, material):
    succeeded = inventory.take_item(color=color, material=material)
    if succeeded:
        items_counter.add(-1, {"color": color, "material": material})
    return succeeded
```

<!--
#### UpDownCounter creation
-->

#### UpDownCounterの作成

<!--
There MUST NOT be any API for creating an `UpDownCounter` other than with a
[`Meter`](#meter). This MAY be called `CreateUpDownCounter`. If strong type is
desired, the client can decide the language idomatic name(s), for example
`CreateInt64UpDownCounter`, `CreateDoubleUpDownCounter`,
`CreateUpDownCounter<Int64>`, `CreateUpDownCounter<double>`.
-->

`UpDownCounter`を作成するためのAPIは、[`Meter`](#meter)以外にあってはなりません(MUST NOT)。これを `CreateUpDownCounter` と呼んでも構いません(MAY)。強い型が必要な場合は、クライアントはその言語で慣用的な名前を決めることができます。例えば、`CreateInt64UpDownCounter`、`CreateDoubleUpDownCounter`、`CreateUpDownCounter<Int64>`、`CreateUpDownCounter<double>`などです。

<!--
The API MUST accept the following parameters:
-->

APIは、以下の引数を受け入れなければなりません(MUST)。

<!--
* The `name` of the Instrument, following the [instrument naming
  rule](#instrument-naming-rule).
* An optional `unit of measure`, following the [instrument unit
  rule](#instrument-unit).
* An optional `description`, following the [instrument description
  rule](#instrument-description).
-->

* [Instrumentの命名のルール](#instrument-naming-rule)に従った、Instrumentsの `name`
* [Instrumentの単位のルール](#instrument-unit)に従った、任意の `unit of measure`
* [Instrumentの説明のルール](#instrument-description)に従った任意の`description`

<!--
Here are some examples that individual language client might consider:
-->

ここでは、個々の言語のクライアントが考慮すべきいくつかの例を紹介します。

```python
# Python
customers_in_store = meter.create_up_down_counter(
    name="grocery.customers",
    description="measures the current customers in the grocery store",
    value_type=int)
```

```csharp
// C#
var customersInStore = meter.CreateUpDownCounter<int>(
    "grocery.customers",
    description: "measures the current customers in the grocery store",
    );
```

<!--
#### UpDownCounter operations
-->

#### UpDownCounterの操作

<!--
##### Add
-->

##### Add

<!--
Increment or decrement the UpDownCounter by a fixed amount.
-->

UpDownCounterを一定量だけ加算または減算します。

<!--
This API SHOULD NOT return a value (it MAY return a dummy value if required by
certain programming languages or systems, for example `null`, `undefined`).
-->

このAPIは、値を返すべきではありません(SHOULD NOT)(特定のプログラミング言語やシステムで必要とされる場合は、`null`や`undefined`などのダミーの値を返しても構いません(MAY))。

<!--
Parameters:
-->

引数:

<!--
* The amount to be added, can be positive, negative or zero.
* Optional [attributes](../common/common.md#attributes).
-->

* 加算される量: プラス、マイナス、ゼロのいずれかです。
* 任意で[属性](../common/common.md#attributes)

<!--
The client MAY decide to allow flexible
[attributes](../common/common.md#attributes) to be passed in as individual
arguments. The client MAY allow attribute values to be passed in using a more
efficient way (e.g. strong typed struct allocated on the callstack, tuple). Here
are some examples that individual language client might consider:
-->

クライアントはフレキシブルな[属性](../common/common.md#attributes)を個々の引数として渡すことを許可してもかまいません(MAY)。クライアントは属性値をより効率的な方法で渡すことを許可してもかまいません(例:コールスタックに割り当てられた強力な型付き構造体、タプル)。以下は、個々の言語のクライアントが考慮すべきいくつかの例です。

```python
# Python
customers_in_store.Add(1, {"account.type": "commercial"})
customers_in_store.Add(-1, account_type="residential")
```

```csharp
// C#
customersInStore.Add(1, ("account.type", "commercial"));
customersInStore.Add(-1, new Account { Type = "residential" });
```

<!--
### Asynchronous UpDownCounter
-->

### 非同期UpDownCounter

<!--
Asynchronous UpDownCounter is an asynchronous Instrument which reports additive
value(s) (_e.g. the process heap size - it makes sense to report the heap size
from multiple processes and sum them up, so we get the total heap usage_) when
the instrument is being observed.
-->

非同期UpDownCounterは非同期のInstrumentで、Instrumentが観測されているときに加算値(_例:プロセスのヒープ・サイズ-複数のプロセスのヒープ・サイズを報告し、それらを合計することでヒープの使用量の合計を得ることは理にかなっています_)を報告します。

<!--
Note: if the value grows
[monotonically](https://wikipedia.org/wiki/Monotonic_function), use
[Asynchronous Counter](#asynchronous-counter) instead; if the value is
non-additive, use [Asynchronous Gauge](#asynchronous-gauge) instead.
-->

注意:値が[モノトニックに](https://wikipedia.org/wiki/Monotonic_function)成長する場合は、代わりに[非同期Counter](#非同期Counter)を、値が非加算である場合は、代わりに[非同期Gauge](#非同期Gauge)を使用してください。

<!--
Example uses for Asynchronous UpDownCounter:
-->

非同期UpDownCounterの使用例:

<!--
* the process heap size
* the approximate number of items in a lock-free circular buffer
-->

* プロセスのヒープ・サイズ
* ロックフリーなリングバッファのおおよそのアイテム数

<!--
#### Asynchronous UpDownCounter creation
-->

#### 非同期UpDownCounterの作成

<!--
TODO
-->

TODO

<!--
#### Asynchronous UpDownCounter operations
-->

#### 非同期UpDownCounterの操作

<!--
Asynchronous UpDownCounter is only intended for an asynchronous scenario. The
only operation is provided by the `callback`, which is registered during the
[Asynchronous UpDownCounter creation](#asynchronous-updowncounter-creation).
-->

非同期UpDownCounterは非同期的なシナリオのみを想定しています。唯一の操作は、[非同期UpDownCounterの作成](#非同期UpDownCounterの作成)の際に登録される`callback`によって提供されます。

<!--
## Measurement
-->

## Measurement

<!--
A `Measurement` represents a data point reported via the metrics API to the SDK.
Please refer to the [Metrics Programming Model](./README.md#programming-model)
for the interaction between the API and SDK.
-->

`Measurement`とは、Metrics APIを介してSDKに報告されるデータポイントを表します。APIとSDKの間のやり取りについては、[Metricsプログラミングモデル](./README.md#programming-model)を参照してください。

<!--
`Measurement`s encapsulate:
-->

`Measurement`は以下の情報をカプセル化します:

<!--
* A value
* [`Attributes`](../common/common.md#attributes)
-->

* 値
* [`属性`](../common/common.md#attributes)

<!--
## Compatibility
-->

## 互換性

<!--
All the metrics components SHOULD allow new APIs to be added to existing
components without introducing breaking changes.
-->

すべてのメトリクス・コンポーネントは、既存のコンポーネントに新しいAPIを追加しても、壊れるような変更を加えずに済むようにすべきです(SHOULD)。

<!--
All the metrics APIs SHOULD allow optional parameter(s) to be added to existing
APIs without introducing breaking changes.
-->

すべてのメトリクスAPIは、既存のAPIに壊れるような変更を加えることなく、オプションのパラメータを追加できるようにすべきです(SHOULD)。

<!--
## Concurrency
-->

## Concurrency

<!--
For languages which support concurrent execution the Metrics APIs provide
specific guarantees and safeties.
-->

同時実行をサポートする言語では、Metrics APIが特定の保証と安全性を提供します。

<!--
**MeterProvider** - all methods are safe to be called concurrently.
-->

**MeterProvider** - すべてのメソッドは、同時に呼び出しても安全です。

<!--
**Meter** - all methods are safe to be called concurrently.
-->

**Meter** - すべてのメソッドは、同時に呼び出しても安全です。

<!--
**Instrument** - All methods of any Instrument are safe to be called
concurrently.
-->

**Instrument** - 任意のInstrumentのすべてのメソッドは、同時に呼び出しても安全です。

