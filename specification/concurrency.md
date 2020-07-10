<!--
# Concurrency and Thread-Safety of OpenTelemetry API
-->

# OpenTelemetry APIにおける並行性とスレッドセーフ

<!--
For languages which support concurrent execution the OpenTelemetry APIs provide
specific guarantees and safeties. Not all of API functions are safe to
be called concurrently. Function and method documentation must explicitly
specify whether it is safe or no to make concurrent calls and in what
situations.
-->

並行実行をサポートする言語については、OpenTelemetry APIは限定された保証と安全性を提供しています。すべてのAPI関数が同時に呼び出されても安全というわけではありません。関数やメソッドのドキュメントでは、どのような状況での同時呼び出しが安全か否かを明示的に指定しなければなりません。

<!--
The following are general recommendations of concurrent call safety of
specific subsets of the API.
-->

以下は、APIの特定のサブセットの同時呼び出しの安全性に関する一般的な推奨事項です。

<!--
**Tracer** - all methods are safe to be called concurrently.
-->

**Tracer** - すべてのメソッドが同時に呼ばれても安全です。

<!--
**SpanBuilder** - It is not safe to concurrently call any methods of the
same SpanBuilder instance. Different instances of SpanBuilder can be safely
used concurrently by different threads/coroutines, provided that no single
SpanBuilder is used by more than one thread/coroutine.
-->

**SpanBuilder** - 同じ SpanBuilder インスタンスのメソッドを同時に呼び出すことは安全ではありません。1 つの SpanBuilder が複数のスレッド/コルーチンから使用されない場合であれば、異なるSpanBuilderのインスタンスを異なるスレッド/コルーチンが同時に使用しても安全です。

<!--
**Span** - All methods of Span are safe to be called concurrently.
-->

**Span** - すべてのメソッドが同時に呼ばれても安全です。

<!--
**Link** - Links are immutable and is safe to be used concurrently. Lazy
initialized links must be implemented to be safe to be called concurrently.
-->

**Link** - リンクはイミュータブルであり、同時に使用しても安全です。遅延初期化されたリンクは、並行に呼び出されても安全なように実装する必要があります(MUST)。
