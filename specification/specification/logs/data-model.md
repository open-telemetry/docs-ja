<!--
# Log Data Model
-->

# Log データモデル

<!--
**Status**: [Experimental](../document-status.md)
-->

**Status**: [Experimental](../document-status.md)

<!--
* [Design Notes](#design-notes)
  * [Requirements](#requirements)
  * [Field Kinds](#field-kinds)
* [Log and Event Record Definition](#log-and-event-record-definition)
  * [Field: `Timestamp`](#field-timestamp)
  * [Trace Context Fields](#trace-context-fields)
  * [Field: `TraceId`](#field-traceid)
  * [Field: `SpanId`](#field-spanid)
  * [Field: `TraceFlags`](#field-traceflags)
  * [Severity Fields](#severity-fields)
  * [Field: `SeverityText`](#field-severitytext)
  * [Field: `SeverityNumber`](#field-severitynumber)
  * [Mapping of `SeverityNumber`](#mapping-of-severitynumber)
  * [Reverse Mapping](#reverse-mapping)
  * [Error Semantics](#error-semantics)
  * [Displaying Severity](#displaying-severity)
  * [Comparing Severity](#comparing-severity)
  * [Field: `Name`](#field-name)
  * [Field: `Body`](#field-body)
  * [Field: `Resource`](#field-resource)
  * [Field: `Attributes`](#field-attributes)
* [Example Log Records](#example-log-records)
* [Appendix A. Example Mappings](#appendix-a-example-mappings)
  * [RFC5424 Syslog](#rfc5424-syslog)
  * [Windows Event Log](#windows-event-log)
  * [SignalFx Events](#signalfx-events)
  * [Splunk HEC](#splunk-hec)
  * [Log4j](#log4j)
  * [Zap](#zap)
  * [Apache HTTP Server access log](#apache-http-server-access-log)
  * [CloudTrail Log Event](#cloudtrail-log-event)
  * [Google Cloud Logging](#google-cloud-logging)
* [Elastic Common Schema](#elastic-common-schema)
* [Appendix B: `SeverityNumber` example mappings](#appendix-b-severitynumber-example-mappings)
* [References](#references)
-->

* [設計上の注意](#設計上の注意)
  * [要求仕様](#要求仕様)
  * [フィールドの種類](#フィールドの種類)
* [ログとイベントレコードの定義](#ログとイベントレコードの定義)
  * [フィールド: `Timestamp`](#フィールドtimestamp)
  * [Trace Context Fields](#trace-context-fields)
  * [フィールド: `TraceId`](#フィールドtraceid)
  * [フィールド: `SpanId`](#フィールドspanid)
  * [フィールド: `TraceFlags`](#フィールドtraceflags)
  * [深刻度フィールド](#深刻度フィールド)
  * [フィールド: `SeverityText`](#フィールドseveritytext)
  * [フィールド: `SeverityNumber`](#フィールドseveritynumber)
  * [`SeverityNumber`のマッピング](#SeverityNumberのマッピング)
  * [逆マッピング](#逆マッピング)
  * [エラーのセマンティクス](#エラーのセマンティクス)
  * [深刻度の表示](#深刻度の表示)
  * [深刻度の比較](#深刻度の比較)
  * [フィールド: `Name`](#フィールドname)
  * [フィールド: `Body`](#フィールドbody)
  * [フィールド: `Resource`](#フィールドresource)
  * [フィールド: `Attributes`](#フィールドattributes)
* [ログレコード例](#ログレコード例)
* [付録A マッピング例](#付録A-マッピング例)
  * [RFC5424 Syslog](#rfc5424-syslog)
  * [Windows Event Log](#windows-event-log)
  * [SignalFx Events](#signalfx-events)
  * [Splunk HEC](#splunk-hec)
  * [Log4j](#log4j)
  * [Zap](#zap)
  * [Apache HTTP Server access log](#apache-http-server-access-log)
  * [CloudTrail Log Event](#cloudtrail-log-event)
  * [Google Cloud Logging](#google-cloud-logging)
* [Elastic Common Schema](#elastic-common-schema)
* [付録B `SeverityNumber`マッピング例](#appendixb-severitynumberマッピング例)
* [参考文献](#参考文献)

<!--
This is a data model and semantic conventions that allow to represent logs from
various sources: application log files, machine generated events, system logs,
etc. Existing log formats can be unambiguously mapped to this data model.
Reverse mapping from this data model is also possible to the extent that the
target log format has equivalent capabilities.
-->

これは、アプリケーションのログファイル、マシンが生成したイベント、システムログなど、さまざまなソースからのログを表現するためのデータモデルとセマンティック規約です。既存のログフォーマットは、このデータモデルに明確にマッピングすることができます。また、対象となるログフォーマットが同等の機能を持っている場合には、このデータモデルからの逆マッピングも可能です。

<!--
The purpose of the data model is to have a common understanding of what a log
record is, what data needs to be recorded, transferred, stored and interpreted
by a logging system.
-->

このデータモデルの目的は、ログ記録とは何か、どのようなデータが記録され、転送され、保存され、ロギングシステムによって解釈される必要があるかについて、共通の理解を持つことです。

<!--
This proposal defines a data model for
[Standalone Logs](../glossary.md#standalone-log).
-->

本提案は、[スタンドアローン・ログ](../glossary.md#standalone-log)のデータモデルを定義するものです。

<!--
## Design Notes
-->

## 設計上の注意

<!--
### Requirements
-->

### 要求仕様

<!--
The Data Model was designed to satisfy the following requirements:
-->

このデータモデルは、以下の要件を満たすように設計されています:


<!--
- It should be possible to unambiguously map existing log formats to this Data
  Model. Translating log data from an arbitrary log format to this Data Model
  and back should ideally result in identical data.
-->

- 既存のログフォーマットをこのデータモデルに明確にマッピングすることが可能であること。任意のログフォーマットからこのデータモデルにログデータを変換して戻すと、理想的には同一のデータになります。

<!--
- Mappings of other log formats to this Data Model should be semantically
  meaningful. The Data Model must preserve the semantics of particular elements
  of existing log formats.
-->

- 他のログフォーマットの本データモデルへのマッピングは、セマンティクス的に意味のあるものでなければなりません。このデータモデルは、既存のログフォーマットの特定の要素のセマンティクスを維持しなければなりません。

<!--
- Translating log data from an arbitrary log format A to this Data Model and
  then translating from the Data Model to another log format B ideally must
  result in a meaningful translation of log data that is no worse than a
  reasonable direct translation from log format A to log format B.
-->

- 任意のログフォーマットAからこのデータモデルにログデータを変換し、さらにこのデータモデルから別のログフォーマットBに変換する場合、理想的には、ログフォーマットAからログフォーマットBへの合理的な直接変換よりも悪くはならない、意味のあるログデータの変換にならなければなりません。

<!--
- It should be possible to efficiently represent the Data Model in concrete
  implementations that require the data to be stored or transmitted. We
  primarily care about 2 aspects of efficiency: CPU usage for
  serialization/deserialization and space requirements in serialized form. This
  is an indirect requirement that is affected by the specific representation of
  the Data Model rather than the Data Model itself, but is still useful to keep
  in mind.
-->

- データの保存や送信を必要とする具体的な実装において、データモデルを効率的に表現できるようにする必要があります。私たちは主に2つの効率性に注目しています。それは、シリアライズ/デシリアライズのためのCPU使用率と、シリアライズされた形での必要サイズです。これは、データモデル自体ではなく、データモデルの特定の表現に影響される間接的な要件ですが、留意しておくと便利です。

<!--
The Data Model aims to successfully represent 3 sorts of logs and events:
-->

このデータモデルは、以下の3種類のログやイベントをうまく表現することを目的としています。:

<!--
- System Formats. These are logs and events generated by the operating system
  and over which we have no control - we cannot change the format or affect what
  information is included (unless the data is generated by an application which
  we can modify). An example of system format is Syslog.
-->

- システムフォーマット。オペレーティングシステムによって生成されたログやイベントで、私たちがコントロールすることはできません。フォーマットを変更したり、含まれる情報に影響を与えたりすることはできません(私たちが変更できるアプリケーションによってデータが生成されている場合を除く)。システムフォーマットの例として、Syslogがあります。

<!--
- Third-party Applications. These are generated by third-party applications. We
  may have certain control over what information is included, e.g. customize the
  format. An example is Apache log file.
-->

- サードパーティ・アプリケーション。これらは、第三者のアプリケーションによって生成されます。私たちは、どのような情報が含まれるか、例えばフォーマットをカスタマイズするなど、一定のコントロールが可能です。例としては、Apacheのログファイルがあります。

<!--
- First-party Applications. These are applications that we develop and we have
  some control over how the logs and events are generated and what information
  we include in the logs. We can likely modify the source code of the
  application if needed.
-->

- ファーストパーティ・アプリケーション。これらは、私たちが開発したアプリケーションで、ログやイベントの生成方法やログに含める情報をある程度コントロールすることができます。また、必要に応じてアプリケーションのソースコードを変更することも可能です。

<!--
### Field Kinds
-->

### フィールドの種類

<!--
This Data Model defines a logical model for a log record (irrespective of the
physical format and encoding of the record). Each record contains 2 kinds of
fields:
-->

このデータモデルは、(レコードの物理的なフォーマットやエンコーディングに関係なく)ログレコードの論理モデルを定義しています。各レコードには2種類のフィールドがあります。

<!--
- Named top-level fields of specific type and meaning.
-->

- 特定の型と意味を持つ、名前付きのトップレベルフィールド

<!--
- Fields stored in the key/value pair lists, which can contain arbitrary values
  of different types. The keys and values for well-known fields follow semantic
  conventions for key names and possible values that allow all parties that work
  with the field to have the same interpretation of the data. See references to
  semantic conventions for `Resource` and `Attributes` fields and examples in
  [Appendix A](#appendix-a-example-mappings).
-->

- キーと値のペアのリストに格納されているフィールドで、異なるタイプの任意の値を含むことができます。よく知られているフィールドのキーと値は、そのフィールドを扱うすべての関係者がデータを同じように解釈できるように、キー名と取りうる値はセマンティック規約に従っています。[付録A](#付録A-マッピング例)の `Resource` と `Attributes` フィールドのセマンティック規約と例について参照してください。

<!--
The reasons for having these 2 kinds of fields are:
-->

この2種類のフィールドを用意した理由は以下のとおりです:

<!--
- Ability to efficiently represent named top-level fields, which are almost
  always present (e.g. when using encodings like Protocol Buffers where fields
  are enumerated but not named on the wire).
  
- Ability to enforce types of named fields, which is very useful for compiled
  languages with type checks.
-->

- ほとんどの場合、存在する名前付きのトップレベルフィールドを効率的に表現することができます(例えば、フィールドが列挙されているがワイヤ上では名前が付けられていないProtocol Bufferのようなエンコーディングを使用する場合)。

- 名前付きフィールドの型を強制する機能。これは型チェックを行うコンパイル済み言語では非常に便利です。

<!--
- Flexibility to represent less frequent data via key/value pair lists. This
  includes well-known data that has standardized semantics as well as arbitrary
  custom data that the application may want to include in the logs.
-->

- 頻繁ではないデータを、キー/バリューペアのリストで柔軟に表現することができます。これには、標準化されたセマンティクスを持つよく知られたデータや、アプリケーションがログに含めたい任意のカスタムデータが含まれます。

<!--
When designing this data model we followed the following reasoning to make a
decision about when to use a top-level named field:
-->

このデータモデルを設計する際、トップレベルの名前付きフィールドをいつ使用するかについて、次のような理由に従って決定しました。

<!--
- The field needs to be either mandatory for all records or be frequently
  present in well-known log and event formats (such as `Timestamp`) or is
  expected to be often present in log records in upcoming logging systems (such
  as `TraceId`).
-->

- このフィールドは、すべてのレコードに必須であるか、よく知られているログやイベントのフォーマットに頻繁に存在するか(`Timestamp`など)、今後のログシステムのログレコードに頻繁に存在することが予想されるか(`TraceId`など)のいずれかである必要があること。

<!--
- The field’s semantics must be the same for all known log and event formats and
  can be mapped directly and unambiguously to this data model.
  
Both of the above conditions were required to give the field a place in the
top-level structure of the record.
-->

- フィールドのセマンティクスは、すべての既知のログおよびイベントフォーマットで同じでなければならず、このデータモデルに直接かつ明確にマッピングすることができること。


レコードのトップレベル構造の中でフィールドに場所を与えるためには、上記の両方の条件が必要でした。

<!--
## Log and Event Record Definition
-->

## ログとイベントレコードの定義

<!--
Note: below we use type `any`, which can be a scalar value (number, string or
boolean), or an array or map of values. Arbitrary deep nesting of values for
arrays and maps is allowed (essentially allow to represent an equivalent of a
JSON object).
-->

注:以下では、スカラー値(数値、文字列、ブーリアン)や、値の配列やマップを表す `any` 型を使用しています。配列やマップでは、値の任意の深い入れ子が可能です(基本的には、JSONオブジェクトと同等のものを表すことができます)。

<!--
[Appendix A](#appendix-a-example-mappings) contains many examples that show how
existing log formats map to the fields defined below. If there are questions
about the meaning of the field reviewing the examples may be helpful.
-->

[付録A](#付録A-マッピング例)には、既存のログフォーマットが以下に定義されたフィールドにどのようにマッピングされるかを示す多くの例が含まれています。フィールドの意味について疑問がある場合は、例を確認するとよいでしょう。

<!--
Here is the list of fields in a log record:
-->

ここでは、ログレコードのフィールドの一覧を示します。

<!--
Field Name   |Description
---------------|--------------------------------------------
Timestamp  |Time when the event occurred.
TraceId    |Request trace id.
SpanId   |Request span id.
TraceFlags   |W3C trace flag.
SeverityText |The severity text (also known as log level).
SeverityNumber |Numerical value of the severity.
Name     |Short event identifier.
Body     |The body of the log record.
Resource   |Describes the source of the log.
Attributes   |Additional information about the event.
-->

フィールド名    |説明
---------------|--------------------------------------------
Timestamp      |イベントが発生した時間
TraceId        |リクエストのTraceID
SpanId         |リクエストのSpanID
TraceFlags     |W3C trace フラグ
SeverityText   |深刻度のテキスト(ログレベルとも呼ばれます)
SeverityNumber |深刻度の数値。
Name           |イベントの短い識別子
Body           |ログレコードの中身
Resource       |ログの発生元の説明
Attributes     |このイベントの追加情報

<!--
Below is the detailed description of each field.
-->

以下、各項目の詳細を説明します。

<!--
### Field: `Timestamp`
-->

### フィールド: `Timestamp`

<!--
Type: Timestamp, uint64 nanoseconds since Unix epoch.
-->

型: タイムスタンプ、Unixエポックからのuint64ナノ秒。

<!--
Description: Time when the event occurred measured by the origin clock. This
field is optional, it may be missing if the timestamp is unknown.
-->

説明: イベントが発生した時刻を原点の時計(XXX: origin clock)で計測したもの。このフィールドは任意で、タイムスタンプが不明な場合は欠落することがあります。

<!--
### Trace Context Fields
-->

### Trace Context フィールド

<!--
#### Field: `TraceId`
-->

#### フィールド: `TraceId`

<!--
Type: byte sequence.
-->

型: バイト配列

<!--
Description: Request trace id as defined in
[W3C Trace Context](https://www.w3.org/TR/trace-context/#trace-id). Can be set
for logs that are part of request processing and have an assigned trace id. This
field is optional.
-->

説明: [W3C Trace Context](https://www.w3.org/TR/trace-context/#trace-id)で定義されている、Request Trace ID。リクエスト処理の一部であり、TraceIDが割り当てられているログに設定できます。このフィールドは任意です。

<!--
#### Field: `SpanId`
-->

#### フィールド: `SpanId`

<!--
Type: byte sequence.
-->

型: バイト配列

<!--
Description: Span id. Can be set for logs that are part of a particular
processing span. If SpanId is present TraceId SHOULD be also present. This field
is optional.
-->

説明: SpanId。特定の処理Spanの一部であるログに設定できます。SpanIdが存在する場合、TraceIdも存在すべきです(SHOULD)。このフィールドは任意です。

<!--
#### Field: `TraceFlags`
-->

#### フィールド: `TraceFlags`

<!--
Type: byte.
-->

型: バイト

<!--
Description: Trace flag as defined in
[W3C Trace Context](https://www.w3.org/TR/trace-context/#trace-flags)
specification. At the time of writing the specification defines one flag - the
SAMPLED flag. This field is optional.
-->

説明: [W3C Trace Context](https://www.w3.org/TR/trace-context/#trace-flags)仕様で定義されているトレースフラグです。現時点では、この仕様では、SAMPLEDフラグという1つのフラグが定義されています。このフィールドは任意です。

<!--
### Severity Fields
-->

### 深刻度フィールド

<!--
#### Field: `SeverityText`
-->

#### フィールド: `SeverityText`

<!--
Type: string.
-->

型: 文字列

<!--
Description: severity text (also known as log level). This is the original
string representation of the severity as it is known at the source. If this
field is missing and `SeverityNumber` is present then the short name that
corresponds to the `SeverityNumber` may be used as a substitution. This field is
optional.
-->

説明: 深刻度テキスト(ログレベルとも呼ばれます)です。これは、ソースで知られている深刻度のオリジナルの文字列表現です。このフィールドがなく `SeverityNumber` がある場合は、`SeverityNumber` に対応する短い名前が代用として使われることがあります。このフィールドは任意です。

<!--
#### Field: `SeverityNumber`
-->

#### フィールド: `SeverityNumber`

<!--
型: number.
-->

型: Number

<!--
Description: numerical value of the severity, normalized to values described in
this document. This field is optional.
-->

説明: このドキュメントに記載されている値に正規化された、深刻度の数値。このフィールドは任意です。

<!--
`SeverityNumber` is an integer number. Smaller numerical values correspond to
less severe events (such as debug events), larger numerical values correspond to
more severe events (such as errors and critical events). The following table
defines the meaning of `SeverityNumber` value:
-->

`SeverityNumber`は整数値です。小さい値はそれほど深刻ではないイベント(デバッグイベントなど)に対応し、大きい値はより深刻なイベント(エラーやクリティカルイベントなど)に対応します。以下の表は、`SeverityNumber`の値の意味を定義しています。

<!--
SeverityNumber range|Range name|Meaning
--------------------|----------|-------
1-4       |TRACE   |A fine-grained debugging event. Typically disabled in default configurations.
5-8       |DEBUG   |A debugging event.
9-12      |INFO  |An informational event. Indicates that an event happened.
13-16     |WARN  |A warning event. Not an error but is likely more important than an informational event.
17-20     |ERROR   |An error event. Something went wrong.
21-24     |FATAL   |A fatal error such as application or system crash.
-->

SeverityNumberの範囲|範囲の名前|意味
--------------------|----------|-------
1-4       |TRACE    |細かく設定されたデバッギングイベント。通常、デフォルトの設定では無効になっています。
5-8       |DEBUG    |デバッグイベント。
9-12      |INFO     |情報を提供するイベント。イベントが発生したことを示す。
13-16     |WARN     |警告イベント。エラーではありませんが、情報提供イベントよりも重要であると思われます。
17-20     |ERROR    |エラーイベント。何かが間違っていることを示します
21-24     |FATAL    |アプリケーションやシステムのクラッシュなどの致命的なエラーが発生した場合。

<!--
Smaller numerical values in each range represent less important (less severe)
events. Larger numerical values in each range represent more important (more
severe) events. For example `SeverityNumber=17` describes an error that is less
critical than an error with `SeverityNumber=20`.
-->

各範囲の数値が小さいほど、重要度が低い(深刻度が低い)事象を表しています。各範囲内の大きな数値は、より重要な(より深刻な)イベントを表します。例えば、`SeverityNumber=17`は、`SeverityNumber=20`のエラーよりも重要度が低いエラーを表しています。

<!--
#### Mapping of `SeverityNumber`
-->

#### `SeverityNumber`のマッピング

<!--
Mappings from existing logging systems and formats (or **source format** for
short) must define how severity (or log level) of that particular format
corresponds to `SeverityNumber` of this data model based on the meaning given
for each range in the above table.
-->

既存のログシステムやフォーマット(略して**ソースフォーマット**)からのマッピングでは、そのフォーマットの深刻度(またはログレベル)がこのデータモデルの`SeverityNumber`にどのように対応するかを、上記の表の各範囲に与えられた意味に基づいて定義する必要があります。

<!--
If the source format has more than one severity that matches a single range in
this table then the severities of the source format must be assigned numerical
values from that range according to how severe (important) the source severity
is.
-->

ソースフォーマットの深刻度が、この表の1つの範囲に一致するものが複数ある場合、ソースフォーマットの深刻度は、ソースの深刻度の度合い(重要度)に応じて、その範囲から数値を割り当てる必要があります。

<!--
For example if the source format defines "Error" and "Critical" as error events
and "Critical" is a more important and more severe situation then we can choose
the following `SeverityNumber` values for the mapping: "Error"->17,
"Critical"->18.
-->

例えば、ソースフォーマットでエラーイベントとして "Error "と "Critical "が定義されていて、"Critical "がより重要でより深刻な状況である場合、マッピングには以下の`SeverityNumber`の値を選択することができます。"Error"->17, "Critical"->18

<!--
If the source format has only a single severity that matches the meaning of the
range then it is recommended to assign that severity the smallest value of the
range.
-->

ソースフォーマットに範囲の意味と一致する深刻度が1つしかない場合は、その深刻度に範囲の最小値を割り当てることをお勧めします。

<!--
For example if the source format has an "Informational" log level and no other
log levels with similar meaning then it is recommended to use
`SeverityNumber=9` for "Informational".
-->

例えば、ソースフォーマットに "Informational"というログレベルがあり、同様の意味を持つ他のログレベルがない場合、"Informational "には`SeverityNumber=9`を使用することが推奨されます。

<!--
Source formats that do not define a concept of severity or log level MAY omit
`SeverityNumber` and `SeverityText` fields. Backend and UI may represent log
records with missing severity information distinctly or may interpret log
records with missing `SeverityNumber` and `SeverityText` fields as if the
`SeverityNumber` was set equal to INFO (numeric value of 9).
-->

深刻度やログレベルの概念を定義していないソースフォーマットは、`SeverityNumber` と `SeverityText` フィールドを省略しても構いません(MAY)。バックエンドと UI は、深刻度の情報が欠落したログレコードを区別して表現したり、`SeverityNumber` と `SeverityText` フィールドが欠落したログレコードを、`SeverityNumber` が INFO (数値 9) に設定されているかのように解釈することができます。

<!--
#### Reverse Mapping
-->

#### 逆マッピング

<!--
When performing a reverse mapping from `SeverityNumber` to a specific format
and the `SeverityNumber` has no corresponding mapping entry for that format
then it is recommended to choose the target severity that is in the same
severity range and is closest numerically.
-->

`SeverityNumber`から特定のフォーマットへの逆マッピングを行う際に、`SeverityNumber`にそのフォーマットに対応するマッピングエントリがない場合は、同じ深刻度の範囲にあり、かつ数値的に最も近いターゲットの深刻度を選択することをお勧めします。

<!--
For example Zap has only one severity in the INFO range, called "Info". When
doing reverse mapping all `SeverityNumber` values in INFO range (numeric 9-12)
will be mapped to Zap’s "Info" level.
-->

例えば、ZapにはINFOの範囲に「Info」という1つの深刻度しかありません。逆マッピングを行うと、INFO範囲(数字の9～12)のすべての`SeverityNumber`値がZapの「Info」レベルにマッピングされます。

<!--
#### Error Semantics
-->

#### エラーのセマンティクス

<!--
If `SeverityNumber` is present and has a value of ERROR (numeric 17) or higher
then it is an indication that the log record represents an erroneous situation.
It is up to the reader of this value to make a decision on how to use this fact
(e.g. UIs may display such errors in a different color or have a feature to find
all erroneous log records).
-->

`SeverityNumber`が存在し、ERROR(数字の17)以上の値を持っている場合、それはエラーが起きている状況をログレコードが表していることを示しています。この事実をどのように使用するかは、この値の読み手次第です(例えば、UIはそのようなエラーを別の色で表示したり、すべてのエラーを示すログレコードを見つける機能を持つことができます)。

<!--
If the log record represents an erroneous event and the source format does not
define a severity or log level concept then it is recommended to set
`SeverityNumber` to ERROR (numeric 17) during the mapping process. If the log
record represents a non-erroneous event the `SeverityNumber` field may be
omitted or may be set to any numeric value less than ERROR (numeric 17). The
recommended value in this case is INFO (numeric 9). See
[Appendix B](#appendix-b-severitynumber-example-mappings) for more mapping
examples.
-->

ログレコードがエラーイベントを表し、ソースフォーマットが重大度やログレベルの概念を定義していない場合は、マッピングプロセス中に `SeverityNumber` を ERROR (numeric 17) に設定することが推奨されます。ログレコードがエラーではないイベントを表している場合、`SeverityNumber`フィールドは省略できるか、ERROR(数値17)よりも小さい任意の数値に設定することができます。この場合の推奨値はINFO(数字の9)です。その他のマッピング例については、[付録B](#appendix-b-severitynumber-example-mappings)を参照してください。

<!--
#### Displaying Severity
-->

#### 深刻度の表示

<!--
The following table defines the recommended short name for each
`SeverityNumber` value. The short name can be used for example for representing
the `SeverityNumber` in the UI:
-->

次の表では、各 `SeverityNumber` の値に推奨される短縮名を定義しています。この短縮名は、例えば、UIで`SeverityNumber`を表現する際に使用できます。

<!--
SeverityNumber|Short Name
--------------|----------
1     |TRACE
2     |TRACE2
3     |TRACE3
4     |TRACE4
5     |DEBUG
6     |DEBUG2
7     |DEBUG3
8     |DEBUG4
9     |INFO
10    |INFO2
11    |INFO3
12    |INFO4
13    |WARN
14    |WARN2
15    |WARN3
16    |WARN4
17    |ERROR
18    |ERROR2
19    |ERROR3
20    |ERROR4
21    |FATAL
22    |FATAL2
23    |FATAL3
24    |FATAL4
-->

深刻度の数字|短縮名
--------------|----------
1             |TRACE
2             |TRACE2
3             |TRACE3
4             |TRACE4
5             |DEBUG
6             |DEBUG2
7             |DEBUG3
8             |DEBUG4
9             |INFO
10            |INFO2
11            |INFO3
12            |INFO4
13            |WARN
14            |WARN2
15            |WARN3
16            |WARN4
17            |ERROR
18            |ERROR2
19            |ERROR3
20            |ERROR4
21            |FATAL
22            |FATAL2
23            |FATAL3
24            |FATAL4
<!--
When an individual log record is displayed it is recommended to show both
`SeverityText` and `SeverityNumber` values. A recommended combined string in
this case begins with the short name followed by `SeverityText` in parenthesis.
-->

個々のログレコードを表示する際には、`SeverityText`と`SeverityNumber`の両方の値を表示することが推奨されます。この場合の推奨される結合文字列は、短縮名で始まり、括弧内に `SeverityText` が続きます。

<!--
For example "Informational" Syslog record will be displayed as **INFO
(Informational)**. When for a particular log record the `SeverityNumber` is
defined but the `SeverityText` is missing it is recommended to only show the
short name, e.g. **INFO**.
-->

例えば、"Informational "Syslogレコードは**INFO (Informational)**と表示されます。特定のログレコードについて、`SeverityNumber`は定義されているが、`SeverityText`がない場合は、**INFO**のように短い名前だけを表示することをお勧めします。

<!--
When drop down lists (or other UI elements that are intended to represent the
possible set of values) are used for representing the severity it is preferable
to display the short name in such UI elements.
-->

重大度を表すためにドロップダウンリスト(または可能な値のセットを表すことを意図した他のUI要素)を使用する場合、そのようなUI要素では短い名前を表示することが望ましいです。

<!--
For example a dropdown list of severities that allows filtering log records by
severities is likely to be more usable if it contains the short names of
`SeverityNumber` (and thus has a limited upper bound of elements) compared to a
dropdown list, which lists all distinct `SeverityText` values that are known to
the system (which can be a large number of elements, often differing only in
capitalization or abbreviated, e.g. "Info" vs "Information").
-->

例えば、ログレコードを深刻度でフィルタリングするための深刻度のドロップダウンリストは、システムに知られているすべての異なる `SeverityText` 値をリストアップするドロップダウンリスト (これは多数の要素になる可能性があり、しばしば大文字と小文字が異なるだけだったり、「Info」と「Information」のように省略されたりします) と比較して、`SeverityNumber` の短縮名を含む (したがって、要素の上限は限られています) 場合には、より使いやすくなります。

<!--
#### Comparing Severity
-->

#### 深刻度の比較

<!--
In the contexts where severity participates in less-than / greater-than
comparisons `SeverityNumber` field should be used. `SeverityNumber` can be
compared to another `SeverityNumber` or to numbers in the 1..24 range (or to the
corresponding short names).
-->

深刻度が小なり/大なりの比較に関与する文脈では、`SeverityNumber`フィールドを使用する必要があります。SeverityNumber` は別の `SeverityNumber` や 1 ～ 24 の範囲の数字 (または対応する短縮名) と比較することができます。

<!--
### Field: `Name`
-->

### フィールド: `Name`

<!--
型: string.
-->

型: 文字列

<!--
Description: Short event identifier that does not contain varying parts.
`Name` describes what happened (e.g. "ProcessStarted"). Recommended to be
no longer than 50 characters. Not guaranteed to be unique in any way. Typically
used for filtering and grouping purposes in backends. This field is optional.
-->

説明: 変化する部分を含まない、短いイベント識別子です。`Name`は何が起こったかを表します(例:"ProcessStarted")。50文字以下であることを推奨します。どのような場合でも、一意であることは保証されません。通常、バックエンドでのフィルタリングやグループ化の目的で使用されます。このフィールドはオプションです。

<!--
### Field: `Body`
-->

### フィールド: `Body`

<!--
型: any.
-->

型: any

<!--
Description: A value containing the body of the log record (see the description
of `any` type above). Can be for example a human-readable string message
(including multi-line) describing the event in a free form or it can be a
structured data composed of arrays and maps of other values. Can vary for each
occurrence of the event coming from the same source. This field is optional.
-->

説明: ログレコードの本文を含む値です(上記の `any` タイプの説明を参照)。例えば、イベントを自由な形式で記述した人間が読める文字列メッセージ(複数行を含む)や、他の値の配列やマップで構成された構造化データにすることができます。同じソースからのイベントの発生ごとに異なることがあります。このフィールドはオプションです。

<!--
### Field: `Resource`
-->

### フィールド: `Resource`

<!--
型: key/value pair list.
-->

型: key/value pair list.

<!--
Description: Describes the source of the log, aka
[resource](../overview.md#resources).
"key" of each pair is a `string` and "value" is of `any` type. Multiple
occurrences of events coming from the same event source can happen across time
and they all have the same value of `Resource`. Can contain for example
information about the application that emits the record or about the
infrastructure where the application runs. Data formats that represent this data
model may be designed in a manner that allows the `Resource` field to be
recorded only once per batch of log records that come from the same source.
SHOULD follow OpenTelemetry
[semantic conventions for Resources](../resource/semantic_conventions/README.md).
This field is optional.
-->

説明: ログのソースを記述します。[resource](../overview.md#resources)といいます。各ペアの "key"は `文字列` で、"value"は `any` の型です。同じイベントソースからの複数のイベントが時間を超えて発生することがありますが、それらはすべて `Resource` の同じ値を持ちます。例えば、レコードを発行するアプリケーションに関する情報や、アプリケーションが動作するインフラに関する情報などを含むことができます。このデータモデルを表すデータフォーマットは、`Resource`フィールドが、同じソースから来るログレコードのバッチごとに一度だけ記録されるように設計されているかもしれません。OpenTelemetry [semantic conventions for Resources](../resource/semantic_conventions/README.md)に従うべきです(SHOULD)。このフィールドは任意です。

<!--
### Field: `Attributes`
-->

### フィールド: `Attributes`

<!--
型: key/value pair list.
-->

型: キー/バリューペアの配列

<!--
Description: Additional information about the specific event occurrence. "key"
of each pair is a `string` and "value" is of `any` type. Unlike the `Resource`
field, which is fixed for a particular source, `Attributes` can vary for each
occurrence of the event coming from the same source. Can contain information
about the request context (other than TraceId/SpanId). SHOULD follow
OpenTelemetry
[semantic conventions for Attributes](../trace/semantic_conventions/README.md).
This field is optional.
-->

説明: 特定のイベントの発生に関する追加情報です。各ペアの "key" は `文字列` で、"value" は `any` 型です。特定のソースに対して固定されている `Resource` フィールドとは異なり、`Attributes` は同じソースから来るイベントの各発生に対して変化させることができます。(TraceId/SpanId以外の)リクエストコンテキストに関する情報を含むことができます。OpenTelemetry [semantic conventions for Attributes](../trace/semantic_conventions/README.md)に従うべきです(SHOULD)。このフィールドは任意です。

<!--
## Example Log Records
-->

## ログレコードの例

<!--
Below are examples that show one possible representation of log records in JSON.
These are just examples to help understand the data model. Don’t treat the
examples as _the_ way to represent this data model in JSON.
-->

以下は、JSONでのログレコードの表現の可能性を示す例です。これらは、データモデルを理解するための単なる例です。これらの例は、データモデルを理解するための一例であり、このデータモデルをJSONで表現するための _唯一の_方法として(XXX:このtheはどういう意味？)扱わないでください。

<!--
This document does not define the actual encoding and format of the log record
representation. Format definitions will be done in separate OTEPs (e.g. the log
records may be represented as msgpack, JSON, Protocol Buffer messages, etc).
-->

本ドキュメントでは、ログレコード表現の実際のエンコーディングやフォーマットは定義しません。フォーマットの定義は別のOTEPで行います(例:ログレコードはmsgpack、JSON、Protocol Bufferメッセージなどで表現されます)。

<!--
Example 1
-->

例 1

<!--
```javascript
{
  "Timestamp": 1586960586000, // JSON needs to make a decision about
          // how to represent nanoseconds.
  "Attributes": {
  "http.status_code": 500,
  "http.url": "http://example.com",
  "my.custom.application.tag": "hello",
  },
  "Resource": {
  "service.name": "donut_shop",
  "service.version": "2.0.0",
  "k8s.pod.uid": "1138528c-c36e-11e9-a1a7-42010a800198",
  },
  "TraceId": "f4dbb3edd765f620", // this is a byte sequence
           // (hex-encoded in JSON)
  "SpanId": "43222c2d51a7abe3",
  "SeverityText": "INFO",
  "SeverityNumber": 9,
  "Body": "20200415T072306-0700 INFO I like donuts"
}
```
-->

```javascript
{
  "Timestamp": 1586960586000, // JSON needs to make a decision about
          // how to represent nanoseconds.
  "Attributes": {
  "http.status_code": 500,
  "http.url": "http://example.com",
  "my.custom.application.tag": "hello",
  },
  "Resource": {
  "service.name": "donut_shop",
  "service.version": "2.0.0",
  "k8s.pod.uid": "1138528c-c36e-11e9-a1a7-42010a800198",
  },
  "TraceId": "f4dbb3edd765f620", // this is a byte sequence
           // (hex-encoded in JSON)
  "SpanId": "43222c2d51a7abe3",
  "SeverityText": "INFO",
  "SeverityNumber": 9,
  "Body": "20200415T072306-0700 INFO I like donuts"
}
```

<!--
Example 2
-->

例 2

<!--
```javascript
{
  "Timestamp": 1586960586000,
  ...
  "Body": {
  "i": "am",
  "an": "event",
  "of": {
  "some": "complexity"
  }
  }
}
```
-->

```javascript
{
  "Timestamp": 1586960586000,
  ...
  "Body": {
  "i": "am",
  "an": "event",
  "of": {
  "some": "complexity"
  }
  }
}
```

<!--
Example 3
-->

例 3

<!--
```javascript
{
 "Timestamp": 1586960586000,
 "Attributes":{
  "http.scheme":"https",
  "http.host":"donut.mycie.com",
  "http.target":"/order",
  "http.method":"post",
  "http.status_code":500,
  "http.flavor":"1.1",
  "http.user_agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.149 Safari/537.36",
 }
}
```
-->

```javascript
{
 "Timestamp": 1586960586000,
 "Attributes":{
  "http.scheme":"https",
  "http.host":"donut.mycie.com",
  "http.target":"/order",
  "http.method":"post",
  "http.status_code":500,
  "http.flavor":"1.1",
  "http.user_agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.149 Safari/537.36",
 }
}
```

<!--
## Appendix A. Example Mappings
-->

## 付録A マッピング例

<!--
This section contains examples of mapping of other events and logs formats to
this data model.
-->

ここでは、他のイベントやログのフォーマットをこのデータモデルにマッピングした例を紹介します。

<!--
### RFC5424 Syslog
-->

### RFC5424 Syslog

<!--
<table>
  <tr>
  <td>Property</td>
  <td>型</td>
  <td>説明</td>
  <td>統一データモデルのフィールドへのマッピング</td>
  </tr>
  <tr>
  <td>TIMESTAMP</td>
  <td>Timestamp</td>
  <td>Time when an event occurred measured by the origin clock.</td>
  <td>Timestamp</td>
  </tr>
  <tr>
  <td>SEVERITY</td>
  <td>enum</td>
  <td>Defines the importance of the event. Example: `Debug`</td>
  <td>Severity</td>
  </tr>
  <tr>
  <td>FACILITY</td>
  <td>enum</td>
  <td>Describes where the event originated. A predefined list of Unix processes. Part of event source identity. Example: `mail system`</td>
  <td>Attributes["syslog.facility"]</td>
  </tr>
  <tr>
  <td>VERSION</td>
  <td>number</td>
  <td>Meta: protocol version, orthogonal to the event.</td>
  <td>Attributes["syslog.version"]</td>
  </tr>
  <tr>
  <td>HOSTNAME</td>
  <td>文字列</td>
  <td>Describes the location where the event originated. Possible values are FQDN, IP address, etc.</td>
  <td>Resource["host.hostname"]</td>
  </tr>
  <tr>
  <td>APP-NAME</td>
  <td>文字列</td>
  <td>User-defined app name. Part of event source identity.</td>
  <td>Resource["service.name"]</td>
  </tr>
  <tr>
  <td>PROCID</td>
  <td>文字列</td>
  <td>Not well defined. May be used as a meta field for protocol operation purposes or may be part of event source identity.</td>
  <td>Attributes["syslog.procid"]</td>
  </tr>
  <tr>
  <td>MSGID</td>
  <td>文字列</td>
  <td>Defines the type of the event. Part of event source identity. Example: "TCPIN"</td>
  <td>Name</td>
  </tr>
  <tr>
  <td>STRUCTURED-DATA</td>
  <td>array of maps of string to string</td>
  <td>A variety of use cases depending on the SDID:
Can describe event source identity
Can include data that describes particular occurrence of the event.
Can be meta-information, e.g. quality of timestamp value.</td>
  <td>SDID origin.swVersion map to Resource["service.version"]

SDID origin.ip map to attribute[net.host.ip"]

Rest of SDIDs -> Attributes["syslog.*"]</td>
  </tr>
  <tr>
    <td>MSG</td>
    <td>文字列</td>
    <td>Free-form text message about the event. Typically human readable.</td>
    <td>Body</td>
  </tr>
</table>
-->

<table>
  <tr>
  <td>Property</td>
  <td>型</td>
  <td>説明</td>
  <td>統一データモデルのフィールドへのマッピング</td>
  </tr>
  <tr>
  <td>TIMESTAMP</td>
  <td>Timestamp</td>
  <td>ある事象が発生した時刻を原点の時計で測定したもの</td>
  <td>Timestamp</td>
  </tr>
  <tr>
  <td>SEVERITY</td>
  <td>enum</td>
  <td>イベントの重要性を定義します。例: `Debug`</td>
  <td>Severity</td>
  </tr>
  <tr>
  <td>FACILITY</td>
  <td>enum</td>
  <td>イベントが発生した場所を示します。Unix プロセスの定義済みのリストです。イベントソースの識別子の一部です。例:`mail system`</td>
  <td>Attributes["syslog.facility"]</td>
  </tr>
  <tr>
  <td>VERSION</td>
  <td>number</td>
  <td>Meta:イベントとは直交するプロトコルバージョン</td>
  <td>Attributes["syslog.version"]</td>
  </tr>
  <tr>
  <td>HOSTNAME</td>
  <td>文字列</td>
  <td>イベントが発生した場所を記述します。設定可能な値は、FQDN、IPアドレスなどです</td>
  <td>Resource["host.hostname"]</td>
  </tr>
  <tr>
  <td>APP-NAME</td>
  <td>文字列</td>
  <td>ユーザー定義のアプリ名。イベントソースの識別子の一部です</td>
  <td>Resource["service.name"]</td>
  </tr>
  <tr>
  <td>PROCID</td>
  <td>文字列</td>
  <td>十分に定義されていなません。プロトコル運用のためのメタフィールドとして使用されるか、イベントソースの識別子の一部となる可能性があります</td>
  <td>Attributes["syslog.procid"]</td>
  </tr>
  <tr>
  <td>MSGID</td>
  <td>文字列</td>
  <td>イベントのタイプを定義します。イベントソースの識別子の一部。例 "TCPIN"</td>
  <td>Name</td>
  </tr>
  <tr>
  <td>STRUCTURED-DATA</td>
  <td>文字列から文字列へのMapの配列</td>
  <td>SDIDに応じて様々な使用例があります:イベントソースの識別子を記述できます。イベントの特定の発生を説明するデータを含むことができます。例えば、タイムスタンプの値の質などのメタ情報が含まれます。
<td>SDID origin.swVersion は Resource["service.version"]にマップされます。

SDID origin.ip は attribute[net.host.ip"]にマップされます

その他のSDIDs -> Attributes["syslog.*"]</td>
  </tr>
  <tr>
    <td>MSG</td>
    <td>文字列</td>
    <td>イベントに関する自由形式のテキストメッセージ。一般的には人間が読めるものです</td>
    <td>Body</td>
  </tr>
</table>

<!--
### Windows Event Log
-->

### Windows イベントログ

<!--
<table>
  <tr>
  <td>Property</td>
  <td>型</td>
  <td>説明</td>
  <td>統一データモデルのフィールドへのマッピング</td>
  </tr>
  <tr>
  <td>TimeCreated</td>
  <td>Timestamp</td>
  <td>The time stamp that identifies when the event was logged.</td>
  <td>Timestamp</td>
  </tr>
  <tr>
  <td>Level</td>
  <td>enum</td>
  <td>Contains the severity level of the event.</td>
  <td>Severity</td>
  </tr>
  <tr>
  <td>Computer</td>
  <td>文字列</td>
  <td>The name of the computer on which the event occurred.</td>
  <td>Resource["host.hostname"]</td>
  </tr>
  <tr>
  <td>EventID</td>
  <td>uint</td>
  <td>The identifier that the provider used to identify the event.</td>
  <td>Name</td>
  </tr>
  <tr>
  <td>Message</td>
  <td>文字列</td>
  <td>The message string.</td>
  <td>Body</td>
  </tr>
  <tr>
  <td>Rest of the fields.</td>
  <td>any</td>
  <td>All other fields in the event.</td>
  <td>Attributes["winlog.*"]</td>
  </tr>
</table>
-->

<table>
  <tr>
  <td>Property</td>
  <td>型</td>
  <td>説明</td>
  <td>統一データモデルのフィールドへのマッピング</td>
  </tr>
  <tr>
  <td>TimeCreated</td>
  <td>Timestamp</td>
  <td>イベントがいつ記録されたかを示すタイムスタンプ</td>
  <td>Timestamp</td>
  </tr>
  <tr>
  <td>Level</td>
  <td>enum</td>
  <td>イベントの深刻度を表示します</td>
  <td>Severity</td>
  </tr>
  <tr>
  <td>Computer</td>
  <td>文字列</td>
  <td>イベントが発生したコンピューターの名前</td>
  <td>Resource["host.hostname"]</td>
  </tr>
  <tr>
  <td>EventID</td>
  <td>uint</td>
  <td>プロバイダがイベントの識別に使用した識別子</td>
  <td>Name</td>
  </tr>
  <tr>
  <td>Message</td>
  <td>文字列</td>
  <td>メッセージの文字列</td>
  <td>Body</td>
  </tr>
  <tr>
  <td>フィールドの残り</td>
  <td>any</td>
  <td>イベントの他のすべてのフィールド</td>
  <td>Attributes["winlog.*"]</td>
  </tr>
</table>


<!--
### SignalFx Events
-->

### SignalFx イベント

<!--
<table>
  <tr>
  <td>フィールド</td>
  <td>型</td>
  <td>説明</td>
  <td>統一データモデルのフィールドへのマッピング</td>
  </tr>
  <tr>
  <td>Timestamp</td>
  <td>Timestamp</td>
  <td>Time when the event occurred measured by the origin clock.</td>
  <td>Timestamp</td>
  </tr>
  <tr>
  <td>EventType</td>
  <td>文字列</td>
  <td>Short machine understandable string describing the event type. SignalFx specific concept. Non-namespaced. Example: k8s Event Reason field.</td>
  <td>Name</td>
  </tr>
  <tr>
  <td>Category</td>
  <td>enum</td>
  <td>Describes where the event originated and why. SignalFx specific concept. Example: AGENT. </td>
  <td>Attributes["com.splunk.signalfx.event_category"]</td>
  </tr>
  <tr>
  <td>Dimensions</td>
  <td>map of string to string</td>
  <td>Helps to define the identity of the event source together with EventType and Category. Multiple occurrences of events coming from the same event source can happen across time and they all have the value of Dimensions. </td>
  <td>Resource</td>
  </tr>
  <tr>
  <td>Properties</td>
  <td>map of string to any</td>
  <td>Additional information about the specific event occurrence. Unlike Dimensions which are fixed for a particular event source, Properties can have different values for each occurrence of the event coming from the same event source.</td>
  <td>Attributes</td>
  </tr>
</table>
-->
<table>
  <tr>
  <td>フィールド</td>
  <td>型</td>
  <td>説明</td>
  <td>統一データモデルのフィールドへのマッピング</td>
  </tr>
  <tr>
  <td>Timestamp</td>
  <td>Timestamp</td>
  <td>イベントが発生した時間を原点の時計で計測したもの</td>
  <td>Timestamp</td>
  </tr>
  <tr>
  <td>EventType</td>
  <td>文字列</td>
  <td>イベントタイプを説明する、機械で理解できる短い文字列。SignalFx特有の概念です。ネームスペースなし。例: k8s Event Reason Field</td>
  <td>Name</td>
  </tr>
  <tr>
  <td>Category</td>
  <td>enum</td>
  <td>イベントがどこで発生し、なぜ発生したのかを説明します。SignalFx固有のコンセプト。例: AGENT</td>
  <td>Attributes["com.splunk.signalfx.event_category"]</td>
  </tr>
  <tr>
  <td>Dimensions</td>
  <td>文字列-文字列のMap</td>
  <td>EventTypeやCategoryとともに、イベントソースの識別子を定義するのに役立ちます。同じイベントソースから来るイベントの複数の発生は、時間を超えて起こる可能性があり、それらはすべてその同じDimensionsの値を持っています</td>
  <td>Resource</td>
  </tr>
  <tr>
  <td>Properties</td>
  <td>文字列-文字列のMap</td>
  <td>特定のイベント発生に関する追加情報。特定のイベントソースに対して固定されているDimensionsとは異なり、Propertiesは同じイベントソースから来るイベントの各発生に対して異なる値を持つことができます</td>
  <td>Attributes</td>
  </tr>
</table>
<!--
### Splunk HEC
-->

### Splunk HEC

<!--
<table>
  <tr>
  <td>フィールド</td>
  <td>型</td>
  <td>説明</td>
  <td>統一データモデルのフィールドへのマッピング</td>
  </tr>
  <tr>
  <td>time</td>
  <td>numeric, string</td>
  <td>The event time in epoch time format, in seconds.</td>
  <td>Timestamp</td>
  </tr>
  <tr>
  <td>host</td>
  <td>文字列</td>
  <td>The host value to assign to the event data. This is typically the host name of the client that you are sending data from.</td>
  <td>Resource["host.hostname"]</td>
  </tr>
  <tr>
  <td>source</td>
  <td>文字列</td>
  <td>The source value to assign to the event data. For example, if you are sending data from an app you are developing, you could set this key to the name of the app.</td>
  <td>Resource["service.name"]</td>
  </tr>
  <tr>
  <td>sourcetype</td>
  <td>文字列</td>
  <td>The sourcetype value to assign to the event data.</td>
  <td>Attributes["source.type"]</td>
  </tr>
  <tr>
  <td>event</td>
  <td>any</td>
  <td>The JSON representation of the raw body of the event. It can be a string, number, string array, number array, JSON object, or a JSON array.</td>
  <td>Body</td>
  </tr>
  <tr>
  <td>fields</td>
  <td>Map of any</td>
  <td>Specifies a JSON object that contains explicit custom fields.</td>
  <td>Attributes</td>
  </tr>
  <tr>
  <td>index</td>
  <td>文字列</td>
  <td>The name of the index by which the event data is to be indexed. The index you specify here must be within the list of allowed indexes if the token has the indexes parameter set.</td>
  <td>TBD, most like will go to attributes</td>
  </tr>
</table>
-->

<table>
  <tr>
  <td>フィールド</td>
  <td>型</td>
  <td>説明</td>
  <td>統一データモデルのフィールドへのマッピング</td>
  </tr>
  <tr>
  <td>time</td>
  <td>numeric, 文字列</td>
  <td>エポックタイム形式のイベント時間(単位:秒)</td>
  <td>Timestamp</td>
  </tr>
  <tr>
  <td>host</td>
  <td>文字列</td>
  <td>イベントデータに割り当てるホスト値です。これは通常、データを送信するクライアントのホスト名となります</td>
  <td>Resource["host.hostname"]</td>
  </tr>
  <tr>
  <td>source</td>
  <td>文字列</td>
  <td>イベントデータに割り当てるソース値です。例えば、開発中のアプリからデータを送信する場合は、このキーにアプリの名前を設定します</td>
  <td>Resource["service.name"]</td>
  </tr>
  <tr>
  <td>sourcetype</td>
  <td>文字列</td>
  <td>イベントデータに割り当てるsourcetypeの値</td>
  <td>Attributes["source.type"]</td>
  </tr>
  <tr>
  <td>event</td>
  <td>any</td>
  <td>イベントの生のボディのJSON表現です。文字列、数値、文字列配列、数値配列、JSON オブジェクト、JSON 配列のいずれかになります</td>
  <td>Body</td>
  </tr>
  <tr>
  <td>fields</td>
  <td>Map of any</td>
  <td>明示的なカスタムフィールドを含むJSONオブジェクトを指定します</td>
  <td>Attributes</td>
  </tr>
  <tr>
  <td>index</td>
  <td>文字列</td>
  <td>イベントデータのインデックスを作成するためのインデックス名です。ここで指定するインデックスは、トークンにindexesパラメータが設定されている場合、許可されるインデックスのリスト内になければなりません</td>
  <td>未定ですが、ほとんどの場合は属性になります</td>
  </tr>
</table>

<!--
### Log4j
-->

### Log4j

<!--
<table>
  <tr>
  <td>フィールド</td>
  <td>型</td>
  <td>説明</td>
  <td>統一データモデルのフィールドへのマッピング</td>
  </tr>
  <tr>
  <td>Instant</td>
  <td>Timestamp</td>
  <td>Time when an event occurred measured by the origin clock.</td>
  <td>Timestamp</td>
  </tr>
  <tr>
  <td>Level</td>
  <td>enum</td>
  <td>Log level.</td>
  <td>Severity</td>
  </tr>
  <tr>
  <td>Message</td>
  <td>文字列</td>
  <td>Human readable message.</td>
  <td>Body</td>
  </tr>
  <tr>
  <td>All other fields</td>
  <td>any</td>
  <td>Structured data.</td>
  <td>Attributes</td>
  </tr>
</table>
-->


<table>
  <tr>
  <td>フィールド</td>
  <td>型</td>
  <td>説明</td>
  <td>統一データモデルのフィールドへのマッピング</td>
  </tr>
  <tr>
  <td>Instant</td>
  <td>Timestamp</td>
  <td>ある事象が発生した時刻を原点の時計で測定したもの</td>
  <td>Timestamp</td>
  </tr>
  <tr>
  <td>Level</td>
  <td>enum</td>
  <td>ログレベル</td>
  <td>Severity</td>
  </tr>
  <tr>
  <td>Message</td>
  <td>文字列</td>
  <td>人間が読めるメッセージ</td>
  <td>Body</td>
  </tr>
  <tr>
  <td>その他のフィールド</td>
  <td>any</td>
  <td>構造化データ</td>
  <td>Attributes</td>
  </tr>
</table>

<!--
### Zap
-->

### Zap

<!--
<table>
  <tr>
  <td>フィールド</td>
  <td>型</td>
  <td>説明</td>
  <td>統一データモデルのフィールドへのマッピング</td>
  </tr>
  <tr>
  <td>ts</td>
  <td>Timestamp</td>
  <td>Time when an event occurred measured by the origin clock.</td>
  <td>Timestamp</td>
  </tr>
  <tr>
  <td>level</td>
  <td>enum</td>
  <td>Logging level.</td>
  <td>Severity</td>
  </tr>
  <tr>
  <td>caller</td>
  <td>文字列</td>
  <td>Calling function's filename and line number.
</td>
  <td>Attributes, key=TBD</td>
  </tr>
  <tr>
  <td>msg</td>
  <td>文字列</td>
  <td>Human readable message.</td>
  <td>Body</td>
  </tr>
  <tr>
  <td>All other fields</td>
  <td>any</td>
  <td>Structured data.</td>
  <td>Attributes</td>
  </tr>
</table>
-->


<table>
  <tr>
  <td>フィールド</td>
  <td>型</td>
  <td>説明</td>
  <td>統一データモデルのフィールドへのマッピング</td>
  </tr>
  <tr>
  <td>ts</td>
  <td>Timestamp</td>
  <td>ある事象が発生した時刻を原点の時計で測定したもの</td>
  <td>Timestamp</td>
  </tr>
  <tr>
  <td>level</td>
  <td>enum</td>
  <td>ログレベル</td>
  <td>Severity</td>
  </tr>
  <tr>
  <td>caller</td>
  <td>文字列</td>
  <td>呼び出した関数のファイル名と行番号
</td>
  <td>Attributes, key=TBD</td>
  </tr>
  <tr>
  <td>msg</td>
  <td>文字列</td>
  <td>人間が読めるメッセージ</td>
  <td>Body</td>
  </tr>
  <tr>
  <td>その他のフィールド</td>
  <td>any</td>
  <td>構造化データ</td>
  <td>Attributes</td>
  </tr>
</table>
<!--
### Apache HTTP Server access log
-->

### Apache HTTP Server access log

<!--
<table>
  <tr>
  <td>フィールド</td>
  <td>型</td>
  <td>説明</td>
  <td>統一データモデルのフィールドへのマッピング</td>
  </tr>
  <tr>
  <td>%t</td>
  <td>Timestamp</td>
  <td>Time when an event occurred measured by the origin clock.</td>
  <td>Timestamp</td>
  </tr>
  <tr>
  <td>%a</td>
  <td>文字列</td>
  <td>Client IP</td>
  <td>Attributes["net.peer.ip"]</td>
  </tr>
  <tr>
  <td>%A</td>
  <td>文字列</td>
  <td>Server IP</td>
  <td>Attributes["net.host.ip"]</td>
  </tr>
  <tr>
  <td>%h</td>
  <td>文字列</td>
  <td>Remote hostname. </td>
  <td>Attributes["net.peer.name"]</td>
  </tr>
  <tr>
  <td>%m</td>
  <td>文字列</td>
  <td>The request method.</td>
  <td>Attributes["http.method"]</td>
  </tr>
  <tr>
  <td>%v,%p,%U,%q</td>
  <td>文字列</td>
  <td>Multiple fields that can be composed into URL.</td>
  <td>Attributes["http.url"]</td>
  </tr>
  <tr>
  <td>%>s</td>
  <td>文字列</td>
  <td>Response status.</td>
  <td>Attributes["http.status_code"]</td>
  </tr>
  <tr>
  <td>All other fields</td>
  <td>any</td>
  <td>Structured data.</td>
  <td>Attributes, key=TBD</td>
  </tr>
</table>
-->

<table>
  <tr>
  <td>フィールド</td>
  <td>型</td>
  <td>説明</td>
  <td>統一データモデルのフィールドへのマッピング</td>
  </tr>
  <tr>
  <td>%t</td>
  <td>Timestamp</td>
  <td>ある事象が発生した時刻を原点の時計で測定したもの</td>
  <td>Timestamp</td>
  </tr>
  <tr>
  <td>%a</td>
  <td>文字列</td>
  <td>クライアント IP</td>
  <td>Attributes["net.peer.ip"]</td>
  </tr>
  <tr>
  <td>%A</td>
  <td>文字列</td>
  <td>サーバー IP</td>
  <td>Attributes["net.host.ip"]</td>
  </tr>
  <tr>
  <td>%h</td>
  <td>文字列</td>
  <td>リモートのホスト名 </td>
  <td>Attributes["net.peer.name"]</td>
  </tr>
  <tr>
  <td>%m</td>
  <td>文字列</td>
  <td>リクエストメソッド</td>
  <td>Attributes["http.method"]</td>
  </tr>
  <tr>
  <td>%v,%p,%U,%q</td>
  <td>文字列</td>
  <td>URLを構成できる複数のフィールド</td>
  <td>Attributes["http.url"]</td>
  </tr>
  <tr>
  <td>%>s</td>
  <td>文字列</td>
  <td>レスポンスステータス</td>
  <td>Attributes["http.status_code"]</td>
  </tr>
  <tr>
  <td>その他のフィールド</td>
  <td>any</td>
  <td>構造化データ</td>
  <td>Attributes, key=TBD</td>
  </tr>
</table>
<!--
### CloudTrail Log Event
-->

### CloudTrail Log Event

<!--
<table>
  <tr>
  <td>フィールド</td>
  <td>型</td>
  <td>説明</td>
  <td>統一データモデルのフィールドへのマッピング</td>
  </tr>
  <tr>
  <td>eventTime</td>
  <td>文字列</td>
  <td>The date and time the request was made, in coordinated universal time (UTC).</td>
  <td>Timestamp</td>
  </tr>
  <tr>
  <td>eventSource</td>
  <td>文字列</td>
  <td>The service that the request was made to. This name is typically a short form of the service name without spaces plus .amazonaws.com.</td>
  <td>Resource["service.name"]?</td>
  </tr>
  <tr>
  <td>awsRegion</td>
  <td>文字列</td>
  <td>The AWS region that the request was made to, such as us-east-2.</td>
  <td>Resource["cloud.region"]</td>
  </tr>
  <tr>
  <td>sourceIPAddress</td>
  <td>文字列</td>
  <td>The IP address that the request was made from.</td>
  <td>Resource["net.peer.ip"] or Resource["net.host.ip"]? TBD</td>
  </tr>
  <tr>
  <td>errorCode</td>
  <td>文字列</td>
  <td>The AWS service error if the request returns an error.</td>
  <td>Name</td>
  </tr>
  <tr>
  <td>errorMessage</td>
  <td>文字列</td>
  <td>If the request returns an error, the description of the error.</td>
  <td>Body</td>
  </tr>
  <tr>
  <td>All other fields</td>
  <td>*</td>
  <td></td>
  <td>Attributes["cloudtrail.*"]</td>
  </tr>
</table>
-->

<table>
  <tr>
  <td>フィールド</td>
  <td>型</td>
  <td>説明</td>
  <td>統一データモデルのフィールドへのマッピング</td>
  </tr>
  <tr>
  <td>eventTime</td>
  <td>文字列</td>
  <td>リクエストが行われた日時を、協定世界時(UTC)で表したもの</td>
  <td>Timestamp</td>
  </tr>
  <tr>
  <td>eventSource</td>
  <td>文字列</td>
  <td>リクエストが行われたサービスです。この名前は通常、スペースを除いたサービス名の短縮形に.amazonaws.comを加えたものです</td>
  <td>Resource["service.name"]?</td>
  </tr>
  <tr>
  <td>awsRegion</td>
  <td>文字列</td>
  <td>リクエストが行われたAWSのリージョン(us-east-2など)</td>
  <td>Resource["cloud.region"]</td>
  </tr>
  <tr>
  <td>sourceIPAddress</td>
  <td>文字列</td>
  <td>リクエストが行われたIPアドレス</td>
  <td>Resource["net.peer.ip"] or Resource["net.host.ip"]? TBD</td>
  </tr>
  <tr>
  <td>errorCode</td>
  <td>文字列</td>
  <td>リクエストがエラーを返した場合のAWSサービスのエラー</td>
  <td>Name</td>
  </tr>
  <tr>
  <td>errorMessage</td>
  <td>文字列</td>
  <td>リクエストがエラーを返した場合は、そのエラーの説明</td>
  <td>Body</td>
  </tr>
  <tr>
  <td>その他のフィールド</td>
  <td>*</td>
  <td></td>
  <td>Attributes["cloudtrail.*"]</td>
  </tr>
</table>
<!--
### Google Cloud Logging
-->

### Google Cloud Logging

<!--
Field    | Type     | Description               | Maps to Unified Model Field
-----------------|--------------------| ------------------------------------------------------- | ---------------------------
timestamp    | string     | The time the event described by the log entry occurred. | Timestamp
resource   | MonitoredResource  | The monitored resource that produced this log entry.  | Resource
log_name   | string     | The URL-encoded LOG_ID suffix of the log_name field identifies which log stream this entry belongs to. | Name
json_payload   | google.protobuf.Struct | The log entry payload, represented as a structure that is expressed as a JSON object. | Body
proto_payload  | google.protobuf.Any | The log entry payload, represented as a protocol buffer. | Body
text_payload   | string     | The log entry payload, represented as a Unicode string (UTF-8). | Body
severity   | LogSeverity    | The severity of the log entry.          | Severity
trace    | string     | The trace associated with the log entry, if any.    | TraceId
span_id    | string     | The span ID within the trace associated with the log entry. | SpanId
labels     | map<string,string> | A set of user-defined (key, value) data that provides additional information about the log entry. | Attributes
http_request   | HttpRequest    | The HTTP request associated with the log entry, if any. | Attributes["google.httpRequest"]
All other fields |        |                   | Attributes["google.*"]
-->

Field    | Type     | Description               | Maps to Unified Model Field
-----------------|--------------------| ------------------------------------------------------- | ---------------------------
timestamp    | string     | ログエントリに記載されているイベントが発生した時間 | Timestamp
resource   | MonitoredResource  | このログエントリを生成したモニターリソース  | Resource
log_name   | string     | log_nameフィールドのURLエンコードされたLOG_IDサフィックスは、このエントリーがどのログストリームに属しているかを識別します。| Name
json_payload   | google.protobuf.Struct | ログエントリのペイロードで、JSONオブジェクトとして表現される構造体 | Body
proto_payload  | google.protobuf.Any | ログエントリのペイロードで、Protocol Bufferとして表されます | Body
text_payload   | string     | ログエントリのペイロードで、ユニコード文字列(UTF-8)で表されます| Body
severity   | LogSeverity    | ログエントリの深刻度。         | Severity
trace    | string     | ログエントリに関連するトレースがあれば、それを表示します    | TraceId
span_id    | string     | ログエントリに関連するトレース内のSpanID | SpanId
labels     | map<string,string> | ログエントリに関する追加情報を提供する、ユーザー定義の(キー、値)データのセットログエントリに関連するHTTPリクエスト(ある場合) | Attributes
http_request   | HttpRequest    | ログエントリに関連するHTTPリクエスト(ある場合) | Attributes["google.httpRequest"]
All other fields |        |                   | Attributes["google.*"]

<!--
## Elastic Common Schema
-->

## Elastic Common Schema


<table>
  <tr>
    <td>フィールド</td>
    <td>型</td>
    <td>説明</td>
    <td>統一データモデルのフィールドへのマッピング</td>
  </tr>
  <tr>
    <td>@timestamp</td>
    <td>datetime</td>
    <td>イベントが記録された時間</td>
    <td>timestamp</td>
  </tr>
  <tr>
    <td>message</td>
    <td>文字列</td>
    <td>任意の種類のメッセージ</td>
    <td>body</td>
  </tr>
  <tr>
    <td>labels</td>
    <td>key/value</td>
    <td>イベントに関連した恣意的なラベル</td>
    <td>attributes[*]</td>
  </tr>
  <tr>
    <td>tags</td>
    <td>文字列の配列</td>
    <td>イベントに関連する値のリスト</td>
    <td>?</td>
  </tr>
  <tr>
    <td>trace.id</td>
    <td>文字列</td>
    <td>Trace ID</td>
    <td>TraceId</td>
  </tr>
  <tr>
    <td>span.id*</td>
    <td>文字列</td>
    <td>Span ID</td>
    <td>SpanId</td>
  </tr>
  <tr>
    <td>agent.ephemeral_id</td>
    <td>文字列</td>
    <td>Agentによって作成されたEphemeral ID</td>
    <td>**resource</td>
  </tr>
  <tr>
    <td>agent.id</td>
    <td>文字列</td>
    <td>Agentによって作成された一意な識別子</td>
    <td>**resource</td>
  </tr>
  <tr>
    <td>agent.name</td>
    <td>文字列</td>
    <td>Agentに与えられた名前</td>
    <td>resource["telemetry.sdk.name"]</td>
  </tr>
  <tr>
    <td>agent.type</td>
    <td>文字列</td>
    <td>Agentの種類</td>
    <td>resource["telemetry.sdk.language"]</td>
  </tr>
  <tr>
    <td>agent.version</td>
    <td>文字列</td>
    <td>Agentのバージョン</td>
    <td>resource["telemetry.sdk.version"]</td>
  </tr>
  <tr>
    <td>source.ip, client.ip</td>
    <td>文字列</td>
    <td>リクエストが行われたIPアドレス</td>
    <td>attributes["net.peer.ip"] あるいは attributes["net.host.ip"]</td>
  </tr>
  <tr>
    <td>cloud.account.id</td>
    <td>文字列</td>
    <td>指定されたクラウドのアカウントのID</td>
    <td>resource["cloud.account.id"]</td>
  </tr>
  <tr>
    <td>cloud.availability_zone</td>
    <td>文字列</td>
    <td>このホストが稼働しているアベイラビリティー・ゾーン</td>
    <td>resource["cloud.zone"]</td>
  </tr>
  <tr>
    <td>cloud.instance.id</td>
    <td>文字列</td>
    <td>ホストマシンのInstance ID</td>
    <td>**resource</td>
  </tr>
  <tr>
    <td>cloud.instance.name</td>
    <td>文字列</td>
    <td>ホストマシンのインスタンス名</td>
    <td>**resource</td>
  </tr>
  <tr>
    <td>cloud.machine.type</td>
    <td>文字列</td>
    <td>ホストマシンのマシンタイプ</td>
    <td>**resource</td>
  </tr>
  <tr>
    <td>cloud.provider</td>
    <td>文字列</td>
    <td>クラウドプロバイダーの名前。aws、azure、gcp、digitaloceanなどが例示されています</td>
    <td>resource["cloud.provider"]</td>
  </tr>
  <tr>
    <td>cloud.region</td>
    <td>文字列</td>
    <td>このホストが動作しているリージョン</td>
    <td>resource["cloud.region"]</td>
  </tr>
  <tr>
    <td>cloud.image.id*</td>
    <td>文字列</td>
    <td></td>
    <td>resource["host.image.name"]</td>
  </tr>
  <tr>
    <td>container.id</td>
    <td>文字列</td>
    <td>一意なコンテナのID</td>
    <td>resource["container.id"]</td>
  </tr>
  <tr>
    <td>container.image.name</td>
    <td>文字列</td>
    <td>コンテナが構築されたイメージの名前</td>
    <td>resource["container.image.name"]</td>
  </tr>
  <tr>
    <td>container.image.tag</td>
    <td>文字列の配列</td>
    <td>コンテナイメージタグ</td>
    <td>**resource</td>
  </tr>
  <tr>
    <td>container.labels</td>
    <td>key/value</td>
    <td>コンテナイメージラベル</td>
    <td>attributes[*]</td>
  </tr>
  <tr>
    <td>container.name</td>
    <td>文字列</td>
    <td>コンテナ名</td>
    <td>resource["container.name"]</td>
  </tr>
  <tr>
    <td>container.runtime</td>
    <td>文字列</td>
    <td>このコンテナを管理するランタイム。例: "docker"</td>
    <td>**resource</td>
  </tr>
  <tr>
    <td>destination.address</td>
    <td>文字列</td>
    <td>イベントの宛先アドレス</td>
    <td>attributes["destination.address"]</td>
  </tr>
  <tr>
    <td>error.code</td>
    <td>文字列</td>
    <td>エラーの内容を表すエラーコード</td>
    <td>attributes["error.code"]</td>
  </tr>
  <tr>
    <td>error.id</td>
    <td>文字列</td>
    <td>エラーの一意な識別子</td>
    <td>attributes["error.id"]</td>
  </tr>
  <tr>
    <td>error.message</td>
    <td>文字列</td>
    <td>エラーメッセージ</td>
    <td>attributes["error.message"]</td>
  </tr>
  <tr>
    <td>error.stack_trace</td>
    <td>文字列</td>
    <td>このエラーのスタックトレースをプレーンテキストで表示したもの</td>
    <td>attributes["error.stack_trace]</td>
  </tr>
  <tr>
    <td>host.architecture</td>
    <td>文字列</td>
    <td>OSのアーキテクチャ</td>
    <td>**resource</td>
  </tr>
  <tr>
    <td>host.domain</td>
    <td>文字列</td>
    <td>ホストが属しているドメイン名。

例えば、Windowsの場合、これはホストのActive DirectoryドメインまたはNetBIOSドメイン名になります。Linuxの場合は、ホストのLDAPプロバイダーのドメインになります。</td>

<td>**resource</td>
  </tr>
  <tr>
    <td>host.hostname</td>
    <td>文字列</td>
    <td>ホスト名。

通常は、ホストマシン上でhostnameコマンドが返す内容を含んでいます</td>

<td>resource["host.hostname"]</td>

  </tr>
  <tr>
    <td>host.id</td>
    <td>文字列</td>
    <td>固有のホストID</td>
    <td>resource["host.id"]</td>
  </tr>
  <tr>
    <td>host.ip</td>
    <td>文字列の配列</td>
    <td>ホストIP</td>
    <td>resource["host.ip"]</td>
  </tr>
  <tr>
    <td>host.mac</td>
    <td>文字列の配列</td>
    <td>ホストのMACアドレス</td>
    <td>resource["host.mac"]</td>
  </tr>
  <tr>
    <td>host.name</td>
    <td>文字列</td>
    <td>ホスト名。

これには、Unixシステムで返されるホスト名、完全修飾されたもの、またはユーザーが指定した名前が含まれます</td>

<td>resource["host.name"]</td>

  </tr>
  <tr>
    <td>host.type</td>
    <td>文字列</td>
    <td>ホストタイプ</td>
    <td>resource["host.type"]</td>
  </tr>
  <tr>
    <td>host.uptime</td>
    <td>文字列</td>
    <td>ホストが起動してからの秒数</td>
    <td>?</td>
  </tr>
  <tr>
    <td>service.ephemeral_id

</td>
    <td>文字列</td>
    <td>このサービスの一過性の識別子</td>
    <td>**resource</td>
  </tr>
  <tr>
    <td>service.id</td>
    <td>文字列</td>
    <td>実行中のサービスの一意の識別子。サービスが多くのノードで構成されている場合、service.idはすべてのノードで同じでなければなりません</td>
    <td>**resource</td>
  </tr>
  <tr>
    <td>service.name</td>
    <td>文字列</td>
    <td>データが収集されるサービスの名称</td>
    <td>resource["service.name"]</td>
  </tr>
  <tr>
    <td>service.node.name</td>
    <td>文字列</td>
    <td>そのサービスを提供する特定のノードの名前</td>
    <td>resource["service.instance.id"]</td>
  </tr>
  <tr>
    <td>service.state</td>
    <td>文字列</td>
    <td>サービスの現在ステータス</td>
    <td>attributes["service.state"]</td>
  </tr>
  <tr>
    <td>service.type</td>
    <td>文字列</td>
    <td>データが収集されるサービスの種類</td>
    <td>**resource</td>
  </tr>
  <tr>
    <td>service.version</td>
    <td>文字列</td>
    <td>データが収集されたサービスのバージョン</td>
    <td>resource["service.version"]</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
</table>

<!--
\* Not yet formalized into ECS.
-->

\* ECSにはまだ正式に組み込まれていません。

<!--
\*\* A resource that doesn’t exist in the
[OpenTelemetry resource semantic convention](../resource/semantic_conventions/README.md).
-->

\*\* [OpenTelemetry resource semantic convention](../resource/semantic_conventions/README.md)に存在しないリソースです。

<!--
This is a selection of the most relevant fields. See
[for the full reference](https://www.elastic.co/guide/en/ecs/current/ecs-field-reference.html)
for an exhaustive list.
-->

これは、最も関連性の高い分野の選択です。網羅的なリストは[for the full reference](https://www.elastic.co/guide/en/ecs/current/ecs-field-reference.html)をご覧ください。

<!--
## Appendix B: `SeverityNumber` example mappings
-->

## 付録B:`SeverityNumber`マッピング例

<!--
|Syslog   |WinEvtLog  |Log4j |Zap |java.util.logging|SeverityNumber|
|-------------|-----------|------|------|-----------------|--------------|
|     |     |TRACE |  | FINEST    |TRACE   |
|Debug    |Verbose  |DEBUG |Debug | FINER     |DEBUG   |
|     |     |  |  | FINE    |DEBUG2    |
|     |     |  |  | CONFIG    |DEBUG3    |
|Informational|Information|INFO  |Info  | INFO    |INFO    |
|Notice   |     |  |  |       |INFO2   |
|Warning  |Warning  |WARN  |Warn  | WARNING   |WARN    |
|Error    |Error  |ERROR |Error | SEVERE    |ERROR   |
|Critical   |Critical |  |Dpanic|       |ERROR2    |
|Emergency  |     |  |Panic |       |ERROR3    |
|Alert    |     |FATAL |Fatal |       |FATAL   |
-->

|Syslog   |WinEvtLog  |Log4j |Zap |java.util.logging|SeverityNumber|
|-------------|-----------|------|------|-----------------|--------------|
|     |     |TRACE |  | FINEST    |TRACE   |
|Debug    |Verbose  |DEBUG |Debug | FINER     |DEBUG   |
|     |     |  |  | FINE    |DEBUG2    |
|     |     |  |  | CONFIG    |DEBUG3    |
|Informational|Information|INFO  |Info  | INFO    |INFO    |
|Notice   |     |  |  |       |INFO2   |
|Warning  |Warning  |WARN  |Warn  | WARNING   |WARN    |
|Error    |Error  |ERROR |Error | SEVERE    |ERROR   |
|Critical   |Critical |  |Dpanic|       |ERROR2    |
|Emergency  |     |  |Panic |       |ERROR3    |
|Alert    |     |FATAL |Fatal |       |FATAL   |

<!--
## References
-->

## 参考文献


- Log Data Model [OTEP 0097](https://github.com/open-telemetry/oteps/blob/main/text/logs/0097-log-data-model.md)

- [Draft discussion of Data Model](https://docs.google.com/document/d/1ix9_4TQO3o-qyeyNhcOmqAc1MTyr-wnXxxsdWgCMn9c/edit#)

- [Discussion of Severity field](https://docs.google.com/document/d/1WQDz1jF0yKBXe3OibXWfy3g6lor9SvjZ4xT-8uuDCiA/edit#)