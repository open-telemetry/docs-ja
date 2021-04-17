<!--
# Glossary
-->

# 用語集

<!--
This document defines some terms that are used across this specification.
-->

このドキュメントでは、本仕様書で使用されているいくつかの用語を定義します。(訳注: 元ドキュメントを理解しやすくするため、このドキュメントでは用語はなるべく英語のままとしています)

<!--
Some other fundamental terms are documented in the [overview document](overview.md).
-->

その他の基本的な用語については、[概要](overview.md)に記載されています。

<!-- Re-generate TOC with `markdown-toc --no-first-h1 -i` -->

<!-- toc -->

<!--
- [User Roles](#user-roles)
  * [Application Owner](#application-owner)
  * [Library Author](#library-author)
  * [Instrumentation Author](#instrumentation-author)
  * [Plugin Author](#plugin-author)
- [Common](#common)
  * [シグナル](#シグナル)
  * [Packages](#packages)
  * [ABI Compatibility](#abi-compatibility)
  * [In-band and Out-of-band Data](#in-band-and-out-of-band-data)
  * [Manual Instrumentation](#manual-instrumentation)
  * [Automatic Instrumentation](#automatic-instrumentation)
  * [Telemetry SDK](#telemetry-sdk)
  * [Constructors](#constructors)
  * [SDK Plugins](#sdk-plugins)
  * [Exporter Library](#exporter-library)
  * [Instrumented Library](#instrumented-library)
  * [Instrumentation Library](#instrumentation-library)
  * [Tracer Name / Meter Name](#tracer-name--meter-name)
- [Logs](#logs)
  * [Log Record](#log-record)
  * [Log](#log)
  * [Embedded Log](#embedded-log)
  * [Standalone Log](#standalone-log)
  * [Log Attributes](#log-attributes)
  * [Structured Logs](#structured-logs)
  * [Flat File Logs](#flat-file-logs)
-->

- [ユーザーロール](#ユーザーロール)
  * [Application Owner](#application-owner)
  * [Library Author](#library-author)
  * [Instrumentation Author](#instrumentation-author)
  * [Plugin Author](#plugin-author)
- [共通](#共通)
  * [シグナル](#シグナル)
  * [パッケージ](#パッケージ)
  * [ABI互換性](#ABI互換性)
  * [In-band and Out-of-band Data](#in-band-and-out-of-band-data)
  * [Manual Instrumentation](#manual-instrumentation)
  * [Automatic Instrumentation](#automatic-instrumentation)
  * [Telemetry SDK](#telemetry-sdk)
  * [Constructors](#constructors)
  * [SDK Plugins](#sdk-plugins)
  * [Exporter Library](#exporter-library)
  * [Instrumented Library](#instrumented-library)
  * [Instrumentation Library](#instrumentation-library)
  * [Tracer Name / Meter Name](#tracer-name--meter-name)
- [ログ](#ログ)
  * [Log Record](#log-record)
  * [ログ](#ログ)
  * [Embedded Log](#embedded-log)
  * [Standalone Log](#standalone-log)
  * [Log Attributes](#log-attributes)
  * [Structured Logs(構造化ログ)](#structured-logs(構造化ログ))
  * [Flat File Logs](#flat-file-logs)

<!-- tocstop -->

<!--
## User Roles
-->

## ユーザーロール

<!--
### Application Owner
-->

### Application Owner

<!--
The maintainer of an application or service, responsible for configuring and managing the lifecycle of the OpenTelemetry SDK.
-->

アプリケーションやサービスのメンテナで、OpenTelemetry SDKの設定やライフサイクルの管理を担当します。


<!--
### Library Author
-->

### Library Author

<!--
The maintainer of a shared library which is depended upon by many applications, and targeted by OpenTelemetry instrumentation.
-->

OpenTelemetryの計装対象となる、多くのアプリケーションから使用される共有ライブラリのメンテナです。

<!--
### Instrumentation Author
-->

### Instrumentation Author

<!--
The maintainer of OpenTelemetry instrumentation written against the OpenTelemetry API.
This may be instrumentation written within application code, within a shared library, or within an instrumentation library.
-->


OpenTelemetry APIに対して書かれたOpenTelemetry計装のメンテナです。これは、アプリケーション・コード内、共有ライブラリ内、あるいは計装ライブラリ内に書かれた計装です。


<!--
### Plugin Author
-->

### Plugin Author

<!--
The maintainer of an OpenTelemetry SDK Plugin, written against OpenTelemetry SDK plugin interfaces.
-->

OpenTelemetry SDKプラグインのインターフェイスに対して書かれたOpenTelemetry SDKプラグインのメンテナです。

<!--
## Common
-->

## 共通

<!--
### Signals
-->

### シグナル

<!--
OpenTelemetry is structured around signals, or categories of telemetry.
Metrics, logs, traces, and baggage are examples of signals.
Each signal represents a coherent, stand-alone set of functionality.
Each signal follows a separate lifecycle, defining its current stability level.
-->

OpenTelemetryはシグナル、つまり遠隔測定のカテゴリーを中心に構成されています。Metrics、Log、Trace、Baggageなどがシグナルの例です。各シグナルは、一貫した独立した機能のセットを表しています。各シグナルは独立したライフサイクルをたどり、現在の安定性レベルを定義します。

<!--
### Packages
-->

### パッケージ

<!--
In this specification, the term **package** describes a set of code which represents a single dependency, which may be imported into a program independently from other packages.
This concept may map to a different term in some languages, such as "module."
Note that in some languages, the term "package" refers to a different concept.
-->

本仕様書では、**パッケージ**という用語は、他のパッケージとは独立してプログラムにインポートすることができる、単一の依存関係を表すコードのセットを表しています。この概念は、言語によっては、"モジュール"のような別の用語に対応している場合があります。なお、言語によっては、「パッケージ」という言葉が別の概念を指すことがあります。

<!--
### ABI Compatibility
-->

### ABI互換性

<!--
An ABI (application binary interface) is an interface which defines interactions between software components at the machine code level, for example between an application executable and a compiled binary of a shared object library. ABI compatibility means that a new compiled version of a library may be correctly linked to a target executable without the need for that executable to be recompiled.
-->

ABI(アプリケーション・バイナリ・インターフェース)とは、ソフトウェアコンポーネント間の相互作用をマシンコードレベルで定義するインターフェースのことで、例えば、アプリケーションの実行ファイルと共有オブジェクト・ライブラリのコンパイル済みバイナリの間で使用されます。ABIの互換性とは、新しいバージョンのライブラリをコンパイルした際に、実行ファイルを再コンパイルすることなく、対象となる実行ファイルに正しくリンクできることを意味します。

<!--
ABI compatibility is important for some languages, especially those which provide a form of machine code. For other languages, ABI compatibility may not be a relevant requirement.
-->

ABI互換性は、いくつかの言語、特にマシンコードの形式を提供する言語にとって重要です。その他の言語では、ABI互換性は重要な要件ではないかもしれません。

<!--
<a name="in-band"></a>
<a name="out-of-band"></a>
-->

<!--
### In-band and Out-of-band Data
-->

### インバンドとアウトオブバンドのデータ

<!--
> In telecommunications, **in-band signaling** is the sending of control information within the same band or channel used for data such as voice or video. This is in contrast to **out-of-band signaling** which is sent over a different channel, or even over a separate network ([Wikipedia](https://en.wikipedia.org/wiki/In-band_signaling)).
-->

> 通信分野において、**インバンドシグナリング**とは、音声やビデオなどのデータに使用されているのと同じ帯域やチャンネル内で制御情報を送信することである。これは、別のチャンネル、あるいは別のネットワークを介して送信される**アウトオブバンドシグナリング**とは対照的である([Wikipedia](https://en.wikipedia.org/wiki/In-band_signaling))。

<!--
In OpenTelemetry we refer to **in-band data** as data that is passed
between components of a distributed system as part of business messages,
for example, when trace or baggages are included
in the HTTP requests in the form of HTTP headers.
Such data usually does not contain the telemetry,
but is used to correlate and join the telemetry produced by various components.
The telemetry itself is referred to as **out-of-band data**:
it is transmitted from applications via dedicated messages,
usually asynchronously by background routines
rather than from the critical path of the business logic.
Metrics, logs, and traces exported to telemetry backends are examples of out-of-band data.
-->

OpenTelemetryでは、分散システムのコンポーネント間でビジネスメッセージの一部として受け渡されるデータを**インバンドデータ**と呼んでいます。例えば、HTTPリクエストにHTTPヘッダーの形でTraceやBaggageが含まれている場合などがこれにあたります。このようなデータは、通常テレメトリを含んでいませんが、様々なコンポーネントによって生成されたテレメトリを関連付け、結合するために使用されます。テレメトリ自体は、**アウトオブバンドデータ**と呼びます。テレメトリは、専用メッセージを介してアプリケーションから送信され、通常、ビジネスロジックのクリティカルパスからではなく、バックグラウンドルーチンによって非同期的に送信されます。Metrics、Log、テレメトリ・バックエンドにエクスポートされたTraceなどが帯域外データの例です。

<!--
### Manual Instrumentation
-->

### Manual Instrumentation

<!--
Coding against the OpenTelemetry API such as the [Tracing API](trace/api.md), [Metrics API](metrics/api.md), or others to collect telemetry from end-user code or shared frameworks (e.g. MongoDB, Redis, etc.).
-->

[Tracing API](trace/api.md)、[Metrics API](metrics/api.md)などのOpenTelemetry APIを使ったコーディングで、エンドユーザーのコードや共有フレームワーク(MongoDB、Redisなど)からテレメトリデータを収集できます。

<!--
### Automatic Instrumentation
-->

### Automatic Instrumentation

<!--
Refers to telemetry collection methods that do not require the end-user to write or access application code to use the OpenTelemetry APIs. Methods vary by programming language, and examples include bytecode injection or monkey patching.
-->

OpenTelemetry APIを使用するために、エンドユーザーがアプリケーションコードを書いたりアクセスしたりする必要のないテレメトリ収集方法を指します。方法はプログラミング言語によって異なり、例としてはバイトコードインジェクションやモンキーパッチなどが挙げられます。

<!--
Synonym: *Auto-instrumentation*.
-->

同義語: *Auto-instrumentation*.

<!--
### Telemetry SDK
-->

### Telemetry SDK

<!--
Denotes the library that implements the *OpenTelemetry API*.
-->

*OpenTelemetry API*を実装したライブラリを示します。

<!--
See [Library Guidelines](library-guidelines.md#sdk-implementation) and
[Library resource semantic conventions](resource/semantic_conventions/README.md#telemetry-sdk).
-->

[ライブラリガイドライン](library-guidelines.md#sdk-implementation)と[ライブラリリソースセマンティック規約](resource/semantic_conventions/README.md#telemetry-sdk)を参照してください。

<!--
### Constructors
-->

### Constructors

<!--
Constructors are public code used by Application Owners to initialize and configure the OpenTelemetry SDK and contrib packages. Examples of constructors include configuration objects, environment variables, and builders.
-->

コンストラクタは、アプリケーションオーナーがOpenTelemetry SDKやcontribパッケージの初期化や設定に使用するパブリックコードです。コンストラクタの例としては、設定オブジェクト、環境変数、およびビルダーがあります。

<!--
### SDK Plugins
-->

### SDK Plugins

<!--
Plugins are libraries which extend the OpenTelemetry SDK. Examples of plugin interfaces are the `SpanProcessor`, `Exporter`, and `Sampler` interfaces.
-->

プラグインはOpenTelemetry SDKを拡張するライブラリです。プラグインのインターフェースの例としては、`SpanProcessor`, `Exporter`, `Sampler` の各インターフェースがあります。

<!--
### Exporter Library
-->

### Exporter Library

<!--
Exporters are SDK Plugins which implement the `Exporter` interface, and emit telemetry to consumers.
-->

エクスポーターは、`Exporter`インターフェースを実装したSDKプラグインで、コンシューマーにテレメトリを送信します。

<!--
### Instrumented Library
-->

### Instrumented Library

<!--
Denotes the library for which the telemetry signals (traces, metrics, logs) are gathered.
-->

テレメトリ信号(Trace、Metrics、Log)が収集されるライブラリを示します。

<!--
The calls to the OpenTelemetry API can be done either by the Instrumented Library itself,
or by another [Instrumentation Library](#instrumentation-library).
-->

OpenTelemetry APIの呼び出しは、Instrumented Library自身が行うことも、別の[Instrumentation Library](#instrumentation-library)が行うこともできます。


<!--
Example: `org.mongodb.client`.
-->

例: `org.mongodb.client`

<!--
### Instrumentation Library
-->

### Instrumentation Library

<!--
Denotes the library that provides the instrumentation for a given [Instrumented Library](#instrumented-library).
*Instrumented Library* and *Instrumentation Library* may be the same library
if it has built-in OpenTelemetry instrumentation.
-->

ある[Instrumented Library](#instrumented-library)の計装を提供するライブラリを示します。OpenTelemetryの計装が組み込まれている場合、*Instrumented Library*と*Instrumentation Library*は同じライブラリである可能性があります。

<!--
See [Overview](overview.md#instrumentation-libraries) for a more detailed definition and naming guidelines.
-->

より詳細な定義と命名のガイドラインについては、[概要](overview.md#instrumentation-libraries)を参照してください。

<!--
Example: `io.opentelemetry.contrib.mongodb`.
-->

例: `io.opentelemetry.contrib.mongodb`

<!--
Synonyms: *Instrumenting Library*.
-->

同義語: *Instrumenting Library*

<!--
### Tracer Name / Meter Name
-->

### Tracer Name / Meter Name

<!--
This refers to the `name` and (optional) `version` arguments specified when
creating a new `Tracer` or `Meter` (see [Obtaining a Tracer](trace/api.md#tracerprovider)/[Obtaining a Meter](metrics/api.md#meter-interface)).
The name/version pair identifies the [Instrumentation Library](#instrumentation-library).
-->

これは、新しい `Tracer` や `Meter` を作成するときに指定された `name` と (任意の) `version` の引数を参照しています (参照: [Obtaining a Tracer](trace/api.md#tracerprovider)/[Obtaining a Meter](metrics/api.md#meter-interface))。名前とバージョンのペアは、[Instrumentation Library](#instrumentation-library)を識別します。

<!--
## Logs
-->

## ログ

<!--
### Log Record
-->

### Log Record

<!--
A recording of an event. Typically the record includes a timestamp indicating
when the event happened as well as other data that describes what happened,
where it happened, etc.
-->

あるイベントを記録したもの。一般的に、この記録には、イベントが起こった時間を示すタイムスタンプや、何が起こったか、どこで起こったかなどを説明するその他のデータが含まれます。


<!--
Synonyms: *Log Entry*.
-->

同義語: *Log Entry*

<!--
### Log
-->

### ログ

<!--
Sometimes used to refer to a collection of Log Records. May be ambiguous, since
people also sometimes use `Log` to refer to a single `Log Record`, thus this
term should be used carefully and in the context where ambiguity is possible
additional qualifiers should be used (e.g. `Log Record`).
-->

ログレコードの集合体を指す場合に使われることがあります。そのため、この用語は慎重に使用する必要があります。また、曖昧さが生じる可能性がある場合には、追加の修飾語を使用する必要があります (例: `Log Record`)。

<!--
### Embedded Log
-->

### Embedded Log

<!--
`Log Records` embedded inside a [Span](trace/api.md#span)
object, in the [Events](trace/api.md#add-events) list.
-->

[Events](trace/api.md#add-events)リストの中にある、[Span](trace/api.md#span)オブジェクトの中に埋め込まれた`Log Records`です。

<!--
### Standalone Log
-->

### Standalone Log

<!--
`Log Records` that are not embedded inside a `Span` and are recorded elsewhere.
-->

`Log Records` that are not embedded inside a `Span` and are recorded elsewhere.

`Span`の中に組み込まれておらず、別の場所に記録されている、`Log Record` です。

<!--
### Log Attributes
-->

### Log Attributes

<!--
Key/value pairs contained in a `Log Record`.
-->

`Log Record`に含まれるキーと値のペアです。

<!--
### Structured Logs
-->

### Structured Logs(構造化ログ)

<!--
Logs that are recorded in a format which has a well-defined structure that allows
to differentiate between different elements of a Log Record (e.g. the Timestamp,
the Attributes, etc). The _Syslog protocol_ ([RFC 5425](https://tools.ietf.org/html/rfc5424)),
for example, defines a `structured-data` format.
-->

ログレコードの各要素(タイムスタンプ、属性など)を区別できるような、明確な構造を持ったフォーマットで記録されたログを示します。例えば、_Syslogプロトコル_([RFC 5425](https://tools.ietf.org/html/rfc5424))では、`構造化データ` のフォーマットを定義しています。


<!--
### Flat File Logs
-->

### Flat File Logs

<!--
Logs recorded in text files, often one line per log record (although multiline
records are possible too). There is no common industry agreement whether
logs written to text files in more structured formats (e.g. JSON files)
are considered Flat File Logs or not. Where such distinction is important it is
recommended to call it out specifically.
-->

ログはテキストファイルに記録され、多くの場合、1つのログレコードにつき1行です(ただし、複数行の記録も可能です)。より構造化されたフォーマット(例：JSONファイル)でテキストファイルに書き込まれたログをフラットファイルログとみなすかどうかについては、業界で共通の合意はありません。このような区別が重要な場合には、それを明確に示すことをお勧めします。

