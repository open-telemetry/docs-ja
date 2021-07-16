<!--
# Semantic Conventions for Exceptions
-->

# 例外のセマンティック規約

**Status**: [Experimental](../../document-status.md)

<!--
This document defines semantic conventions for recording application
exceptions.
-->

このドキュメントでは、アプリケーションの例外を記録するためのセマンティック規約を定義しています。

<!-- toc -->

<!--
- [Recording an Exception](#recording-an-exception)
- [Attributes](#attributes)
  - [Stacktrace Representation](#stacktrace-representation)
-->

- [例外の記録](#例外の記録)
- [属性](#属性)
  - [スタックトレースの表現](#スタックトレースの表現)

<!-- tocstop -->

<!--
## Recording an Exception
-->

## 例外の記録

<!--
An exception SHOULD be recorded as an `Event` on the span during which it occurred.
The name of the event MUST be `"exception"`.
-->

例外は、それが発生したSpan上の `イベント` として記録されるべきです (SHOULD)。イベントの名前は `"exception"` でなければなりません。

<a name="exception-end-example"></a>

<!--
A typical template for an auto-instrumentation implementing this semantic convention
using an [API-provided `recordException` method](../api.md#record-exception)
could look like this (pseudo-Java):
-->

APIで提供される[`recordException`メソッド](../api.md#record-exception)を使って、このセマンティック規約を実装した自動計装実装の典型的なテンプレートは、以下のようになります(擬似Java)。

```java
Span span = myTracer.startSpan(/*...*/);
try {
  // Code that does the actual work which the Span represents
} catch (Throwable e) {
  span.recordException(e, Attributes.of("exception.escaped", true));
  throw e;
} finally {
  span.end();
}
```

<!--
## Attributes
-->

## 属性

<!--
The table below indicates which attributes should be added to the `Event` and
their types.
-->

以下の表は、`Event`に追加すべき属性とそのタイプを示しています。


<!-- semconv exception -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `exception.type` | string | 例外のタイプ(該当する場合は、完全修飾クラス名)。例外の動的な型をサポートする言語では、静的型よりも動的型を優先すべきです。 | `java.net.ConnectException`; `OSError` | See below |
| `exception.message` | string | 例外のメッセージ | `Division by zero`; `Can't convert 'int' object to str implicitly` | See below |
| `exception.stacktrace` | string | スタックトレースは、言語ランタイムの自然な表現の文字列です。この表現は各言語のSIGが決定し、文書化することになっています。 | `Exception in thread "main" java.lang.RuntimeException: Test exception\n at com.example.GenerateTrace.methodB(GenerateTrace.java:13)\n at com.example.GenerateTrace.methodA(GenerateTrace.java:9)\n at com.example.GenerateTrace.main(GenerateTrace.java:5)` | No |
| `exception.escaped` | boolean | 例外がSpanの範囲を逸脱していることが分かっている場所で例外イベントが記録された場合、trueに設定するべきです(SHOULD)。 [1] |  | No |

**[1]:** 例外が論理的に"飛行中(in flight)"である間にSpanが終了した場合、例外はSpanの範囲から脱出(または離脱)したとみなされます。これは、言語によっては実際に"飛行中"かもしれませんが(例:Pythonで例外がContext managerの`__exit__`メソッドに渡された場合)、ほとんどの言語では例外を記録した時点で捕捉されるのが普通です。
例外が投げられた時点で、その例外がSpanのスコープから外れるかどうかを判断することは通常できません。しかし、[上記の例](#exception-end-example)のように、Spanを終了する直前にアクティブな例外があるかどうかをチェックすれば、例外がエスケープされるかどうかを知ることは簡単です。
これによると、`exception.escaped` 属性が設定されていなかったり、false に設定されていたりしても、例外がSpanのスコープを抜け出す可能性があります。これは、例外が抜け出すかどうかがはっきりしない時点でイベントが記録されている可能性があるからです。

**Additional attribute requirements:** At least one of the following sets of attributes is required:

* `exception.type`
* `exception.message`
<!-- endsemconv -->

<!--
### Stacktrace Representation
-->

### スタックトレースの表現

<!--
The table below, adapted from [Google Cloud][gcp-error-reporting], includes
possible representations of stacktraces in various languages. The table is not
meant to be a recommendation for any particular language, although SIGs are free
to adopt them if they see fit.
-->

以下の表は、[Google Cloud][gcp-error-reporting]からの引用で、様々な言語でのスタックトレースの表現を含んでいます。この表は、特定の言語を推奨するものではありませんが、SIGが適切と判断した場合には自由に採用することができます．

<!--
| Language   | Format                                                              |
| ---------- | ------------------------------------------------------------------- |
| C#         | the return value of [Exception.ToString()][csharp-stacktrace]       |
| Go         | the return value of [runtime.Stack][go-stacktrace]                  |
| Java       | the contents of [Throwable.printStackTrace()][java-stacktrace]      |
| Javascript | the return value of [error.stack][js-stacktrace] as returned by V8  |
| Python     | the return value of [traceback.format_exc()][python-stacktrace]     |
| Ruby       | the return value of [Exception.full_message][ruby-full-message]     |
-->

| 言語   | Format                                                              |
| ---------- | ------------------------------------------------------------------- |
| C#         | [Exception.ToString()][csharp-stacktrace]の戻り値      |
| Go         | [runtime.Stack][go-stacktrace]の戻り値                  |
| Java       | [Throwable.printStackTrace()][java-stacktrace]の内容      |
| Javascript | V8で返される[error.stack][js-stacktrace]の戻り値  |
| Python     | [traceback.format_exc()][python-stacktrace]の戻り値     |
| Ruby       | [Exception.full_message][ruby-full-message]の戻り値     |

<!--
Backends can use the language specified methodology for generating a stacktrace
combined with platform information from the
[telemetry sdk resource][telemetry-sdk-resource] in order to extract more fine
grained information from a stacktrace, if necessary.
-->

バックエンドは、必要に応じてStacktraceからより詳細な情報を抽出するために、[telemetry sdk resource][telemetry-sdk-resource]からのプラットフォーム情報と組み合わせて、スタックトレースを生成するために言語で指定された方法を使用することができます。

<!--
[gcp-error-reporting]: https://cloud.google.com/error-reporting/reference/rest/v1beta1/projects.events/report
[java-stacktrace]: https://docs.oracle.com/javase/7/docs/api/java/lang/Throwable.html#printStackTrace%28%29
[python-stacktrace]: https://docs.python.org/3/library/traceback.html#traceback.format_exc
[js-stacktrace]: https://v8.dev/docs/stack-trace-api
[ruby-full-message]: https://ruby-doc.org/core-2.7.1/Exception.html#method-i-full_message
[csharp-stacktrace]: https://docs.microsoft.com/en-us/dotnet/api/system.exception.tostring
[go-stacktrace]: https://golang.org/pkg/runtime/debug/#Stack
[telemetry-sdk-resource]:../../resource/semantic_conventions/README.md#telemetry-sdk
-->

[gcp-error-reporting]: https://cloud.google.com/error-reporting/reference/rest/v1beta1/projects.events/report
[java-stacktrace]: https://docs.oracle.com/javase/7/docs/api/java/lang/Throwable.html#printStackTrace%28%29
[python-stacktrace]: https://docs.python.org/3/library/traceback.html#traceback.format_exc
[js-stacktrace]: https://v8.dev/docs/stack-trace-api
[ruby-full-message]: https://ruby-doc.org/core-2.7.1/Exception.html#method-i-full_message
[csharp-stacktrace]: https://docs.microsoft.com/en-us/dotnet/api/system.exception.tostring
[go-stacktrace]: https://golang.org/pkg/runtime/debug/#Stack
[telemetry-sdk-resource]:../../resource/semantic_conventions/README.md#telemetry-sdk
