# OpenTelemetry でのエラー処理

<!--
OpenTelemetry generates telemetry data to help users monitor application code. In most cases, the work that the library performs is not essential from the perspective of application business logic. We assume that users would prefer to lose telemetry data rather than have the library significantly change the behavior of the instrumented application.
-->

<!-- weはわたしたち? -->
<!-- would prefer to A rather than Bは、AもBもマイナスでどちらがましか？という意味なので選択すると訳しました -->

OpenTelemetryは、ユーザーがアプリケーションコードを監視するのに役立つテレメトリーデータを生成します。ほとんどの場合、ライブラリが実行する作業はアプリケーションのビジネスロジックの観点からは本質的なものではありません。ライブラリーが計測するアプリケーションのふるまいを大幅に変更するよりは、テレメトリーデータを失うほうを選択するとわたしたちは想定しています。

<!--
OpenTelemetry may be enabled via platform extensibility mechanisms, or dynamically loaded at runtime. This makes the use of the library non-obvious for end users, and may even be outside of the application developer's control. This makes for some unique requirements with respect to error handling.
-->

OpenTelemetryは、プラットフォームの拡張性メカニズムを介して有効にすることもできますし、実行時に動的にロードすることもできます。これによりエンドユーザにとってはライブラリーの使用が非自明ではなくなり、アプリケーション開発者の制御外になることさえあります。このためエラー処理に関していくつかのユニークな要件があります。

<!--
## Basic error handling principles
-->

## 基本的なエラー処理の原則

<!--
OpenTelemetry implementations MUST NOT throw unhandled exceptions at run time.
-->

OpenTelemetryの実装は、実行時にハンドルされない例外を投げてはなりません(MUST NOT)。

<!--
1. API methods MUST NOT throw unhandled exceptions when used incorrectly by end users.
   The API and SDK SHOULD provide safe defaults for missing or invalid arguments.
   For instance, a name like `empty` may be used if the user passes in `null` as the span name argument during `Span` construction.
-->

1. APIメソッドはエンドユーザーが誤って使用した場合にハンドルされない例外を投げてはなりません(MUST NOT)。APIとSDKは、欠落しているか無効な引数に対して安全なデフォルト値を提供する必要があります(SHOULD)。例えば、Spanの構築中にユーザーがSpan名の引数として`null`を渡した場合、`empty`のような名前を使用してもよいです。

<!--
2. The API or SDK may _fail fast_ and cause the application to fail on initialization, e.g. because of a bad user config or environment, but MUST NOT cause the application to fail later at run time, e.g. due to dynamic config settings received from the Collector.
-->

2. APIまたはSDKは初期化時にフェイルしてアプリケーションを _早い段階でフェイル_ させてもよいですが（例えば、正しくないユーザー設定や環境により）、アプリケーションを実行時にフェイルさせてはいけません（例えば、Collectorからの動的な設定値を受けとった場合）。

<!--
3. The SDK MUST NOT throw unhandled exceptions for errors in their own operations.
   For example, an exporter should not throw an exception when it cannot reach the endpoint to which it sends telemetry data.
-->

3. SDK は独自の操作でエラーが発生した場合ハンドルされない例外を投げてはなりません (MUST NOT)。例えばExporterは、テレメトリーデータを送信するエンドポイントに到達できない場合に例外を投げてはいけません。


<!--
## Guidance
-->

## ガイダンス

<!--
1. API methods that accept external callbacks MUST handle all errors.
-->

1. 外部コールバックを受け入れるAPIメソッドは、すべてのエラーを処理しなければなりません(MUST)。

<!--
2. Background tasks (e.g. threads, asynchronous tasks, and spawned processes) should run in the context of a global error handler to ensure that exceptions do not affect the end user application.
-->

2. バックグラウンドタスク(スレッド、非同期タスク、spawnされたプロセスなど)は、例外がエンドユーザーアプリケーションに影響を与えないように、グローバルエラーハンドラのコンテキスト内で実行されなければなりません。

<!--
3. Long-running background tasks should not fail permanently in response to internal errors.
   In general, internal exceptions should only affect the execution context of the request that caused the exception.
-->

3. 長時間実行中のバックグラウンドタスクは、内部エラーの影響により恒久的に失敗してはいけません。
   一般的に、内部例外は例外の原因となったリクエストの実行コンテキストにのみ影響を与えるべきです。

<!--
4. Internal error handling should follow language-specific conventions.
   In general, developers should minimize the scope of error handlers and add special processing for expected exceptions.
-->

4. 内部エラー処理は言語固有の規約に従うべきです。
   一般的に、開発者はエラーハンドラの範囲を最小限にし、予想される例外のために特別な処理を追加すべきです。

<!--
5. Beware external callbacks and overrideable interfaces: Expect them to throw.
-->

5. 外部のコールバックやオーバーライド可能なインタフェースに注意してください。それらは例外をスローすることを予期してください。

<!--
6. Beware to call any methods that wasn't explicitly provided by API and SDK users as a callbacks.
   Method `ToString` that SDK may decide to call on user object may be badly implemented and lead to stack overflow.
   It is common that the application never calls this method and this bad implementation would never be caught by an application owner.
-->

6. APIやSDKのユーザーが明示的に提供していないメソッドをコールバックとして呼び出すことに注意してください。
   SDKがユーザオブジェクト上で呼び出すことにしたメソッド `ToString` は、実装が悪いとスタックオーバーフローを起こす可能性があります。
   アプリケーションがこのメソッドを呼び出さないことはよくあることで、この実装の悪さがアプリケーションのオーナーに見つかることはまずありません。

<!--
7. Whenever API call returns values that is expected to be non-`null` value - in case of error in processing logic - SDK MUST return a "no-op" or any other "default" object that was (_ideally_) pre-allocated and readily available.
   This way API call sites will not crash on attempts to access methods and properties of a `null` objects.
-->

7. APIコールが、処理ロジックでエラーが発生した場合に、非 `null` 値であることが期待される値を返すときは常に、SDKは "no-op"、または( _理想的には_ ) 事前に割り当てられすぐに利用可能な他の "デフォルト" オブジェクトを返さなければなりません(MUST)。
   これにより、APIコールサイトは `null` オブジェクトのメソッドやプロパティにアクセスしようとしてクラッシュすることがありません。

<!--
## Error handling and performance
-->

## エラー処理とパフォーマンス

<!--
Error handling and extensive input validation may cause performance degradation, especially on dynamic languages where the input object types are not guaranteed in compile time.
Runtime type checks will impact performance and are error prone, exceptions may occur despite the best effort.
-->

エラー処理と広範な入力の検証は、特にコンパイル時に入力オブジェクトの型が保証されていない動的な言語では、パフォーマンスの低下を引き起こす可能性があります。
実行時の型チェックはパフォーマンスに影響を与え、エラーが発生しやすく、最善の努力にもかかわらず例外が発生する可能性があります。

<!--
It is recommended to have a global exception handling logic that will guarantee that exceptions are not leaking to the user code.
And make a reasonable trade off of the SDK performance and fullness of type checks that will provide a better on-error behavior and SDK errors troubleshooting.
-->

例外がユーザーコードに漏れないことを保証するグローバルな例外処理ロジックを持つことが推奨されます。
そして、より良いエラー発生時の動作とSDKエラーのトラブルシューティングを提供するために、SDKのパフォーマンスと型チェックの充実度を合理的にトレードオフしてください。

<!--
## Self-diagnostics
-->

## 自己診断

<!--
All OpenTelemetry libraries -- the API, SDK, exporters, instrumentation adapters, etc. -- are encouraged to expose self-troubleshooting metrics, spans, and other telemetry that can be easily enabled and filtered out by default.
-->

すべての OpenTelemetry ライブラリー（API、SDK、Exporters、計装器アダプタなど）は、自己診断のために簡単に有効化できデフォルトで除外設定されている、Metrics、Span、およびその他のテレメトリーを公開することを推奨します。

<!--
One good example of such telemetry is a `Span` exporter that indicates how much time exporters spend uploading telemetry.
Another example may be a metric exposed by a `SpanProcessor` that describes the current queue size of telemetry data to be uploaded.
-->

このようなテレメトリーの良い例としては、Exportersがテレメトリーのアップロードに要した時間を示す `Span` Exporterがあります。
もう一つの例は、アップロードされるテレメトリーデータの現在のキューの長さを示す `SpanProcessor`によって公開されるメトリックがあります。

<!--
Whenever the library suppresses an error that would otherwise have been exposed to the user, the library SHOULD log the error using language-specific conventions.
SDKs MAY expose callbacks to allow end users to handle self-diagnostics separately from application code.
-->

抑制しなければユーザーに公開されていたであろうエラーをライブラリーが抑制するときは常に、ライブラリーは言語固有の規約を使用してエラーをログに記録するべきです(SHOULD)。
SDKは、エンドユーザーがアプリケーションコードとは別に自己診断をハンドルできるように、コールバックを公開することができます(MAY)。

<!--
## Exceptions to the rule
-->

## ルールの例外

<!--
SDK authors MAY supply settings that allow end users to change the library's default error handling behavior.
Application developers may want to run with strict error handling in a staging environment to catch invalid uses of the API, or malformed config.
-->

SDKの作者は、エンドユーザーがライブラリーのデフォルトのエラー処理の動作を変更できるように設定を提供することができます(MAY)。
アプリケーション開発者は、APIの不正な使用や不正な設定を捉えるために、ステージング環境で厳格なエラー処理を実行したいと思うかもしれません。