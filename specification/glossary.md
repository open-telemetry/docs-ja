<!--
# Glossary
-->

# 用語集

<!--
Below are a list of some OpenTelemetry specific terms that are used across this
specification.
-->

以下に、この仕様で使用されているOpenTelemetry特有の用語のリストを示します。

<!--
## Common
-->

## 共通

<!--
### Telemetry SDK
-->

### テレメトリー SDK

<!--
Denotes the library that implements the *OpenTelemetry API*.
-->

*OpenTelemetry API*を実装したライブラリを示します。

<!--
See [Library Guidelines](library-guidelines.md#sdk-implementation) and
[Library resource semantic conventions](resource/semantic_conventions/README.md#telemetry-sdk)
-->

[言語ライブラリの設計原則](library-guidelines.md#sdk-implementation) と [ライブラリResourceのセマンティック規約](resource/semantic_conventions/README.md#telemetry-sdk)を参照してください。

<!--
<a name="instrumented_library"></a>
-->

<a name="instrumented_library"></a>

<!--
### Instrumented Library
-->

### 計装済みライブラリ

<!--
Denotes the library for which the telemetry signals (traces, metrics, logs) are gathered.
-->

テレメトリー(トレース、メトリクス、ログ)を収集するライブラリを示します。

<!--
The calls to the OpenTelemetry API can be done either by the Instrumented Library itself,
or by another [Instrumenting Library](#instrumenting_library).
-->

OpenTelemetry APIの呼び出しは、計装済みライブラリ自身、または別の[計装ライブラリ](#instrumenting_library)から行うことができます。

<!--
Example: `org.mongodb.client`.
-->

例: `org.mongodb.client`。

<!--
<a name="instrumenting_library"></a>
-->

<a name="instrumenting_library"></a>

<!--
### Instrumenting Library
-->

### 計装ライブラリ

<!--
Denotes the library that provides the instrumentation for a given [Instrumented Library](#instrumented_library).
*Instrumented Library* and *Instrumenting Library* may be the same library
if it has built-in OpenTelemetry instrumentation.
-->

与えられた[計装済みライブラリ](#instrumented_library)の計装を提供するライブラリを示します。OpenTelemetryの計装が組み込まれている場合、*計装済みライブラリ*と*計装ライブラリ*は同じライブラリである可能性があります。

<!--
Example: `io.opentelemetry.contrib.mongodb`.
-->

例: `io.opentelemetry.contrib.mongodb`。

<!--
Synonyms: *Instrumentation Library*, *Integration*.
-->

同義語: *Instrumentation Library*, *Integration*。

<!--
<a name="name"></a>
-->

<a name="name"></a>

<!--
### Tracer Name / Meter Name
-->

### Tracer 名 / Meter 名

<!--
This refers to the `name` and (optional) `version` arguments specified when
creating a new `Tracer` or `Meter` (see [Obtaining a Tracer](trace/api.md#obtaining-a-tracer)/[Obtaining a Meter](metrics/api.md#meter-interface)). It identifies the [Instrumenting Library](#instrumenting_library).
-->

これは、新しい `Tracer` や `Meter` を作成する際に指定される `name` と (オプションの) `version` の引数を参照します ( [Tracerの取得](trace/api.md#obtaining-a-tracer)/[Meterの取得](metrics/api.md#meter-interface) を参照してください)。[計装ライブラリ](#instrumenting_library)を識別します。

<!--
## Logs
-->

## ログ

<!--
### Log Record
-->

### ログレコード

<!--
A recording of an event. Typically the record includes a timestamp indicating
when the event happened as well as other data that describes what happened,
where it happened, etc.
-->

ある一イベントのレコード。一般的にレコードには、イベントがいつ起こったかを示すタイムスタンプと、何が起こったか、どこで起こったかなどを説明する他のデータが含まれています。

<!--
Also known as Log Entry.
-->

ログエントリーとも呼ばれます。

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

ログレコードの集合を参照するために使われることもあります。曖昧な場合がありますが、人々は単一の `ログレコード` を参照するために `ログ`という用語を使うこともあるので、この用語は慎重に使うべきです。

<!--
### Embedded Log
-->

### 埋め込みログ

<!--
`Log Records` embedded inside a [Span](trace/api.md#span)
object, in the [Events](trace/api.md#add-events) list.
-->

[Event](trace/api.md#add-events)リストの中の[Span](trace/api.md#span)オブジェクトの中に埋め込まれている`ログレコード`を示します。

<!--
### Standalone Log
-->

### スタンドアローンログ

<!--
`Log Records` that are not embedded inside a `Span` and are recorded elsewhere.
-->

`Span`の中に埋め込まれておらず、他の場所に記録されているログレコードを示します。

<!--
### Log Attributes
-->

### ログ属性

<!--
Key/value pairs contained in a `Log Record`.
-->

`ログレコード`に含まれるキーと値のペアです

<!--
### Structured Logs
-->

### 構造化ログ

<!--
Logs that are recorded in a format which has a well-defined structure that allows
to differentiate between different elements of a Log Record (e.g. the Timestamp,
the Attributes, etc). The _Syslog protocol_ ([RFC 5425](https://tools.ietf.org/html/rfc5424)),
for example, defines a `structured-data` format.
-->

ログレコードの異なる要素(例えば、タイムスタンプや属性など)を区別することができるような、明確に定義された構造を持つフォーマットで記録されるログ。例えば、_Syslog protocol_ ([RFC 5425](https://tools.ietf.org/html/rfc5424) は `構造化されたデータ` 形式を定義しています。

<!--
### Flat File Logs
-->

### フラットファイルログ

<!--
Logs recorded in text files, often one line per log record (although multiline
records are possible too). There is no common industry agreement whether
logs written to text files in more structured formats (e.g. JSON files)
are considered Flat File Logs or not. Where such distinction is important it is
recommended to call it out specifically.
-->

ログはテキストファイルに記録され、多くの場合、ログレコードごとに1行ずつ記録されます(複数行記録も可能ですが)。より構造化された形式(例：JSON ファイル)のテキストファイルに書き込まれたログがフラットファイルログとみなされるかどうかについては、業界で共通の合意はありません。このような区別が重要な場合には、具体的に呼び分けることをお勧めします。

