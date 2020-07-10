<!--
# Performance and Blocking of OpenTelemetry API
-->

# OpenTelemetry APIでのパフォーマンスとブロッキング

<!--
This document defines common principles that will help designers create language libraries that are safe to use.
-->

この文書では、設計者が安全に使用できる言語ライブラリを作成するための共通の原則を定義しています。

<!--
## Key principles
-->

## 重要な原則

<!--
Here are the key principles:
-->

重要な原則を示します:

<!--
- **Library should not block end-user application by default.**
- **Library should not consume unbounded memory resource.**
-->

- **ライブラリはデフォルトでエンドユーザーのアプリケーションをブロックすべきではありません(SHOULD NOT)**
- **ライブラリは束縛されていないメモリリソースを消費すべきではありません(SHOULD NOT)**

<!--
Although there are inevitable overhead to achieve monitoring, API should not degrade the end-user application as possible. So that it should not block the end-user application nor consume too much memory resource.
-->

モニタリングを実現するためには避けられないオーバーヘッドがありますが、APIはできるだけエンドユーザアプリケーションを劣化させないようにしなければなりません。そのため、エンドユーザーアプリケーションをブロックしたり、多くのメモリリソースを消費したりすることがないようにする必要があります。

<!--
See also [Concurrency and Thread-Safety](concurrency.md) if the implementation supports concurrency.
-->

実装が同時実行をサポートしている場合は、[平行性とスレッドセーフ](concurrency.md)も参照してください。

<!--
### Tradeoff between non-blocking and memory consumption
-->

### ノンブロッキングとメモリ消費量のトレードオフ

<!--
Incomplete asynchronous I/O tasks or background tasks may consume memory to preserve their state. In such a case, there is a tradeoff between dropping some tasks to prevent memory starvation and keeping all tasks to prevent information loss.
-->

実行途中の非同期I/Oタスクやバックグラウンドタスクは、状態を維持するためにメモリを消費することがあります。このような場合、メモリ不足を防ぐためにいくつかのタスクを削除するか、情報の損失を防ぐためにすべてのタスクを維持するかのトレードオフがあります。

<!--
If there is such tradeoff in language library, it should provide the following options to end-user:
-->

言語ライブラリにこのようなトレードオフがある場合、エンドユーザに以下のオプションを提供すべきです(SHOULD)。

<!--
- **Prevent information loss**: Preserve all information but possible to consume many resources
- **Prevent blocking**: Dropping some information under overwhelming load and show warning log to inform when information loss starts and when recovered
  - Should provide option to change threshold of the dropping
  - Better to provide metric that represents effective sampling ratio
  - Language library might provide this option for Logging
-->

- **情報の損失を防ぐ**: すべての情報を保存するが、多くのリソースを消費する可能性がある
- **ブロッキングをしない**: 圧倒的な負荷がかかっているときに情報を落とし、情報の損失が始まったときと復旧したときに警告ログを表示してユーザーに知らせるようにする
  - 損失のしきい値を変更するオプションを提供すべきです(SHOULD)
  - 効果的なサンプリング比を表す指標を提供するとより良くなります
  - 言語ライブラリはこのオプションを提供しているかもしれません

<!--
### End-user application should be aware of the size of logs
-->

### エンドユーザーアプリケーションは、ログのサイズを認識する必要があります

<!--
Logging could consume much memory by default if the end-user application emits too many logs. This default behavior is intended to preserve logs rather than dropping it. To make resource usage bounded, the end-user should consider reducing logs that are passed to the exporters.
-->

エンドユーザーアプリケーションがあまりにも多くのログを送出すると、ログはデフォルトで多くのメモリを消費する可能性があります。このデフォルトの動作は、ログをドロップするのではなく、保存することを目的としています。リソースの使用量を制限するために、エンドユーザーはエクスポート先に渡されるログを減らすことを検討すべきです。

<!--
Therefore, the language library should provide a way to filter logs to capture by OpenTelemetry. End-user applications may want to log so much into log file or stdout (or somewhere else) but not want to send all of the logs to OpenTelemetry exporters.
-->

したがって、言語ライブラリはOpenTelemetryでキャプチャするログをフィルタリングする方法を提供すべきです。エンドユーザーアプリケーションは、ログファイルや標準出力(またはどこか他の場所)に多くのログを記録したいけれど、OpenTelemetryのエクスポート先にすべてのログを送りたくないかもしれません。

<!--
In a documentation of the language library, it is a good idea to point out that too many logs consume many resources by default then guide how to filter logs.
-->

言語ライブラリのドキュメントで、あまりにも多くのログがデフォルトで多くのリソースを消費していることを指摘し、ログをフィルタリングする方法を説明するのは良いアイデアです。

<!--
### Shutdown and explicit flushing could block
-->

### シャットダウンと明示的なフラッシュはブロックされる可能性があります

<!--
The language library could block the end-user application when it shut down. On shutdown, it has to flush data to prevent information loss. The language library should support user-configurable timeout if it blocks on shut down.
-->

言語ライブラリは、エンドユーザーアプリケーションがシャットダウンするときにブロックする可能性があります。シャットダウン時には、情報の損失を防ぐためにデータをフラッシュしなければなりません。シャットダウン時にブロックする場合、言語ライブラリはユーザが設定可能なタイムアウトをサポートする必要があります(SHOULD)。

<!--
If the language library supports an explicit flush operation, it could block also. But should support a configurable timeout.
-->

言語ライブラリが明示的なフラッシュ操作をサポートしている場合は、それもブロックすることができます。しかし、設定可能なタイムアウトをサポートしている必要があります(SHOULD)。

<!--
## Documentation
-->

## ドキュメント

<!--
If language specific implementation has special characteristics that are not described in this document, such characteristics should be documented.
-->

言語固有の実装がこの文書に記載されていない特殊な特性を持つ場合は，そのような特性を文書化するべきです(SHOULD)。
