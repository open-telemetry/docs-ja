<!--
# Error handling in OpenTelemetry
-->

# OpenTelemetryでのエラー処理

<!--
OpenTelemetry generates telemetry data to help users monitor application code.
In most cases, the work that the library performs is not essential from the perspective of application business logic.
We assume that users would prefer to lose telemetry data rather than have the library significantly change the behavior of the instrumented application.
-->

OpenTelemetryは、ユーザーがアプリケーションコードを監視するのに役立つテレメトリデータを生成します。ほとんどの場合、ライブラリが実行する作業は、アプリケーション・ビジネス・ロジックの観点からは本質的なものではありません。ユーザーは、ライブラリが計測されたアプリケーションの動作を大幅に変更するよりも、テレメトリデータを失うことを好むと想定しています。

<!--
OpenTelemetry may be enabled via platform extensibility mechanisms, or dynamically loaded at runtime.
This makes the use of the library non-obvious for end users, and may even be outside of the application developer's control.
This makes for some unique requirements with respect to error handling.
-->

OpenTelemetry は、プラットフォームの拡張メカニズムを介して有効にすることも、実行時に動的にロードすることもできます。これにより、エンドユーザにとってはライブラリの使用が目立たなくなり、アプリケーション開発者の制御外になることさえあります。このため、エラー処理に関していくつかのユニークな要件があります。

<!--
## Basic error handling principles
-->

## エラー処理の基本原理

<!--
OpenTelemetry implementations MUST NOT throw unhandled exceptions at run time.
-->

OpenTelemetry の実装は、実行時に未処理の例外を投げてはなりません(MUST NOT)。

<!--
1. API methods MUST NOT throw unhandled exceptions when used incorrectly by end users.
   The API and SDK SHOULD provide safe defaults for missing or invalid arguments.
   For instance, a name like `empty` may be used if the user passes in `null` as the span name argument during `Span` construction.
2. The API or SDK may _fail fast_ and cause the application to fail on initialization, e.g. because of a bad user config or environment, but MUST NOT cause the application to fail later at run time, e.g. due to dynamic config settings received from the Collector.
3. The SDK MUST NOT throw unhandled exceptions for errors in their own operations.
   For example, an exporter should not throw an exception when it cannot reach the endpoint to which it sends telemetry data.
-->

1. APIメソッドは、エンドユーザーが誤って使用した場合に、手の届かない例外を投げてはなりません(MUST NOT)。
   APIとSDKは、欠落した、または無効な引数に対する安全なデフォルトを提供すべきです(SHOULD)。
   例えば、`Span` の構築時にSpan名の引数に `null` を渡した場合、`empty` のような名前を使用することができます。
2. API または SDK は、初期化時にアプリケーションを高速で失敗(_fail fast_)させ、アプリケーションを失敗させる可能性があります(例:ユーザー設定や環境が悪いためなど。しかし、アプリケーションが実行時に後で失敗する原因になってはなりません(MUST NOT))
3. SDK は、独自の操作でエラーが発生した場合、処理されない例外をスローしてはなりません(MUST NOT)。
   例えば、エクスポーターは、テレメトリデータを送信するエンドポイントに到達できない場合に例外をスローしてはなりません。

<!--
## Guidance
-->

## ガイダンス

<!--
1. API methods that accept external callbacks MUST handle all errors.
2. Background tasks (e.g. threads, asynchronous tasks, and spawned processes) should run in the context of a global error handler to ensure that exceptions do not affect the end user application.
3. Long-running background tasks should not fail permanently in response to internal errors.
   In general, internal exceptions should only affect the execution context of the request that caused the exception.
4. Internal error handling should follow language-specific conventions.
   In general, developers should minimize the scope of error handlers and add special processing for expected exceptions.
5. Beware external callbacks and overrideable interfaces: Expect them to throw.
6. Beware to call any methods that wasn't explicitly provided by API and SDK users as a callbacks.
   Method `ToString` that SDK may decide to call on user object may be badly implemented and lead to stack overflow.
   It is common that the application never calls this method and this bad implementation would never be caught by an application owner.
7. Whenever API call returns values that is expected to be non-`null` value - in case of error in processing logic - SDK MUST return a "no-op" or any other "default" object that was (_ideally_) pre-allocated and readily available.
   This way API call sites will not crash on attempts to access methods and properties of a `null` objects.
-->

1. 外部コールバックを受け入れるAPIメソッドは、すべてのエラーを処理しなければなりません(MUST)。
2. バックグラウンドタスク(スレッド、非同期タスク、spwanされたプロセスなど)は、例外がエンドユーザーアプリケーションに影響を与えないように、グローバルエラーハンドラのコンテキストで実行されなければなりません。
3. 長時間実行中のバックグラウンドタスクは、内部エラーに応答して永続的な失敗を起こしてはいけません。
   一般的に、内部例外は例外の原因となったリクエストの実行コンテキストにのみ影響を与えるべきです。
4. 内部エラー処理は言語固有の規約に従うべきです。
   一般的に、開発者はエラーハンドラの範囲を最小限にし、予想される例外のために特別な処理を追加すべきです。
5. 外部のコールバックやオーバーライド可能なインタフェースに注意してください。スローすることを期待してください。
6. APIやSDKのユーザーが明示的に提供していないメソッドをコールバックとして呼び出すことに注意してください。
   SDKがユーザオブジェクト上で呼び出すことにしたメソッド `ToString` は、実装が悪いとスタックオーバーフローを起こす可能性があります。
   アプリケーションがこのメソッドを呼び出さないことはよくあることで、この実装の悪さがアプリケーションのオーナーに見つかることはありません。
7. APIコールが`非null`値であると予想される値を返す場合(処理ロジックに誤りがあった場合)、SDKは事前に(_ideally_)割り当てられていてすぐに利用可能な "no-op"またはその他の "default"オブジェクトを返さなければなりません(MUST)。
   これにより、APIコールサイトが `null` オブジェクトのメソッドやプロパティにアクセスしようとしてもクラッシュすることはありません。

<!--
## Error handling and performance
-->

## エラー処理とパフォーマンス

<!--
Error handling and extensive input validation may cause performance degradation, especially on dynamic languages where the input object types are not guaranteed in compile time.
Runtime type checks will impact performance and are error prone, exceptions may occur despite the best effort.
-->

エラー処理と広範な入力の検証は、特にコンパイル時に入力オブジェクトの型が保証されていない動的な言語では、パフォーマンスの低下を引き起こす可能性があります。ランタイムの型チェックはパフォーマンスに影響を与え、エラーが発生しやすく、最善の努力にもかかわらず例外が発生する可能性があります。

<!--
It is recommended to have a global exception handling logic that will guarantee that exceptions are not leaking to the user code.
And make a reasonable trade off of the SDK performance and fullness of type checks that will provide a better on-error behavior and SDK errors troubleshooting.
-->

例外がユーザーコードに漏れないことを保証するグローバル例外処理ロジックを持つことが推奨されます。そして、SDKのパフォーマンス、より良いエラー時の動作とSDKエラーのトラブルシューティング方法を提供する型チェックの充実度のトレードオフについて、合理的なバランスを取ってください。

<!--
## Self-diagnostics
-->

## 自己診断

<!--
All OpenTelemetry libraries -- the API, SDK, exporters, instrumentations, etc. -- are encouraged to expose self-troubleshooting metrics, spans, and other telemetry that can be easily enabled and filtered out by default.
-->

すべての OpenTelemetry ライブラリ(API、SDK、エクスポート、計装など)は、自己診断メトリクス、Span、およびその他のテレメトリを公開することが推奨されています。

<!--
One good example of such telemetry is a `Span` exporter that indicates how much time exporters spend uploading telemetry.
Another example may be a metric exposed by a `SpanProcessor` that describes the current queue size of telemetry data to be uploaded.
-->

このようなテレメトリの良い例の一つは、エクスポータがテレメトリのアップロードにどれくらいの時間を費やしているかを示す`Span` エクスポータです。もう一つの例は、`SpanProcessor`によって公開されるメトリックで、アップロードされるテレメトリデータの現在のキューサイズを公開しています。

<!--
Whenever the library suppresses an error that would otherwise have been exposed to the user, the library SHOULD log the error using language-specific conventions.
SDKs MAY expose callbacks to allow end users to handle self-diagnostics separately from application code.
-->

ユーザーに公開されているエラーをライブラリが抑制するときはいつでも、ライブラリは言語固有の規約を使用してエラーをログに記録するべきです(SHOULD)。SDKは、エンドユーザーがアプリケーションコードとは別に自己診断を処理できるように、コールバックを公開しても構いません(MAY)。

<!--
## Configuring Error Handlers
-->

## エラーハンドラの設定

<!--
SDK implementations MUST allow end users to change the library's default error handling behavior for relevant errors.
Application developers may want to run with strict error handling in a staging environment to catch invalid uses of the API, or malformed config.
Note that configuring a custom error handler in this way is the only exception to the basic error handling principles outlined above.
The mechanism by which end users set or register a custom error handler should follow language-specific conventions.
-->

SDK の実装では、関連するエラーに対するライブラリのデフォルトのエラー処理動作をエンドユーザーが変更できるようにしなければなりません(MUST)。アプリケーション開発者は、APIの無効な使用や不正な設定をキャッチするために、ステージング環境で厳格なエラー処理を実行したいと思うかもしれません。この方法でカスタムエラーハンドラを設定することは、上記で概説した基本的なエラー処理の原則に対する唯一の例外であることに注意してください。エンドユーザーがカスタムエラーハンドラを設定または登録するメカニズムは、言語固有の規約に従うべきです。

<!--
### Examples
-->

### 例

<!--
These are examples of how end users might register custom error handlers.
Examples are for illustration purposes only. OpenTelemetry client authors
are free to deviate from these provided that their design matches the requirements outlined above.
-->

これらは、エンドユーザーがカスタム・エラー・ハンドラを登録する方法の例です。例は説明のみを目的としています。OpenTelemetryクライアントの作成者は、そのデザインが上記の要件に合致していれば、これらの例から自由に逸脱できます。


#### Go

```go
// 基本的なエラー処理のインターフェース
type ErrorHandler interface {
  Handle(err error)
}

func Handler() ErrorHandler
func SetHandler(handler ErrorHandler)
```

```go
// カスタムエラーハンドラーを登録
type IgnoreExporterErrorsHandler struct{}

func (IgnoreExporterErrorsHandler) Handle(err error) {
    switch err.(type) {
    case *SpanExporterError:
    default:
        fmt.Println(err)
    }
}

func main() {
    // 他の設定…
    opentelemetrysdk.SetHandler(IgnoreExporterErrorsHandler{})
}

```

##### Java

<!--
OpenTelemetry Java uses [java.util.logging](https://docs.oracle.com/javase/7/docs/api/java/util/logging/package-summary.html)
to output and handle all logs, including errors. Custom handlers and filters can be registered both in code and using the Java logging configuration file.
-->

OpenTelemetry Javaは、[java.util.logging](https://docs.oracle.com/javase/7/docs/api/java/util/logging/package-summary.html)を使用して、エラーを含むすべてのログを出力し、処理します。カスタムハンドラやフィルタは、コード内でも、Javaのロギング設定ファイルを使っても登録することができます。


```properties
## すべてのエラーログを出力しない
io.opentelemetry.level = OFF
```

```java
// エクスポーターから来たエラーをログ出力しないようにするカスタムフィルターを作成
public class IgnoreExportErrorsFilter implements Filter {

 public boolean isLoggable(LogRecord record) {
    return !record.getMessage().contains("Exception thrown by the export");
 }
}
```

```properties
## カスタムフィルターをBatchSpanProcessorに登録
io.opentelemetry.sdk.trace.export.BatchSpanProcessor = io.opentelemetry.extensions.logging.IgnoreExportErrorsFilter
```