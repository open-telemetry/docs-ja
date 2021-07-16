<!--
# Performance and Blocking of OpenTelemetry API
-->

# OpenTelemetry APIのパフォーマンスとブロッキング

<!--
This document defines common principles that will help designers create OpenTelemetry clients that are safe to use.
-->

このドキュメントは、設計者が安全に使用できるOpenTelemetryクライアントを作成するのに役立つ共通原則を定義しています。

<!--
## Key principles
-->

## 主要な原則

<!--
Here are the key principles:
-->

ここで、主要な原則を紹介します:

<!--
- **Library should not block end-user application by default.**
- **Library should not consume unbounded memory resource.**
-->

- **ライブラリーがエンドユーザーのアプリケーションをデフォルトでブロックしないこと**
- **ライブラリが制限のないメモリリソースを消費しないこと**

<!--
Although there are inevitable overhead to achieve monitoring, API should not degrade the end-user application as possible. So that it should not block the end-user application nor consume too much memory resource.
-->

モニタリングを実現するためにはオーバーヘッドが避けられませんが、APIは可能な限りエンドユーザー・アプリケーションを劣化させないようにすべきです。そのため、エンドユーザー・アプリケーションをブロックしたり、メモリ・リソースを過剰に消費したりしてはいけません。

<!--
See also [Concurrency and Thread-Safety](library-guidelines.md#concurrency-and-thread-safety) if the implementation supports concurrency.
-->

実装が同時実行をサポートしている場合は、[並行処理とスレッドセーフ](library-guidelines.md#concurrency-and-thread-safety)も参照してください。

<!--
### Tradeoff between non-blocking and memory consumption
-->

### ノンブロッキングとメモリ消費のトレードオフ

<!--
Incomplete asynchronous I/O tasks or background tasks may consume memory to preserve their state. In such a case, there is a tradeoff between dropping some tasks to prevent memory starvation and keeping all tasks to prevent information loss.
-->

不完全な非同期I/Oタスクやバックグラウンドタスクは、その状態を維持するためにメモリを消費することがあります。このような場合、メモリ不足を防ぐために一部のタスクを削除することと、情報損失を防ぐためにすべてのタスクを保持することはトレードオフの関係にあります。

<!--
If there is such tradeoff in OpenTelemetry client, it should provide the following options to end-user:
-->

OpenTelemetryクライアントにそのようなトレードオフがある場合、エンドユーザーに次のようなオプションを提供する必要があります。

<!--
- **Prevent information loss**: Preserve all information but possible to consume many resources
- **Prevent blocking**: Dropping some information under overwhelming load and show warning log to inform when information loss starts and when recovered
  - Should provide option to change threshold of the dropping
  - Better to provide metric that represents effective sampling ratio
  - OpenTelemetry client might provide this option for Logging
-->

- **情報を損失しない**: すべての情報を保持するが、多くのリソースを消費する可能性がある。
- **ブロッキングしない**: 負荷が大きい場合一部の情報を消失させ、情報消失の開始時と回復時に警告ログを表示する。
  - ドロッピングの閾値を変更するオプションを提供すべき
  - 効果的なサンプリング比率を示す指標を提供するとよい
  - OpenTelemetryクライアントは、ロギングのためにこのオプションを提供するかもしれません

<!--
### End-user application should be aware of the size of logs
-->

### エンドユーザーのアプリケーションは、ログのサイズに注意する必要がある

<!--
Logging could consume much memory by default if the end-user application emits too many logs. This default behavior is intended to preserve logs rather than dropping it. To make resource usage bounded, the end-user should consider reducing logs that are passed to the exporters.
-->

エンドユーザーのアプリケーションがあまりにも多くのログを発する場合、ログはデフォルトで多くのメモリを消費する可能性があります。このデフォルトの動作は、ログを落とすのではなく保存することを目的としています。リソースの使用量を制限するために、エンドユーザーはエクスポート先に渡されるログを減らすことを検討する必要があります。

<!--
Therefore, the OpenTelemetry client should provide a way to filter logs to capture by OpenTelemetry. End-user applications may want to log so much into log file or stdout (or somewhere else) but not want to send all of the logs to OpenTelemetry exporters.
-->

したがって、OpenTelemetryクライアントは、OpenTelemetryでキャプチャするログをフィルタリングする方法を提供する必要があります。エンドユーザーのアプリケーションは、ログファイルや標準出力(あるいは他の場所)に多くのログを記録したいが、OpenTelemetry エクスポーターにすべてのログを送りたくないという場合があります。

<!--
In a documentation of the OpenTelemetry client, it is a good idea to point out that too many logs consume many resources by default then guide how to filter logs.
-->

OpenTelemetryクライアントのドキュメントでは、デフォルトではログが多すぎて多くのリソースを消費することを指摘した上で、ログをフィルタリングする方法を案内するのがよいでしょう。

<!--
### Shutdown and explicit flushing could block
-->

### シャットダウンと明示的なフラッシュにより、ブロックを可能とする

<!--
The OpenTelemetry client could block the end-user application when it shut down. On shutdown, it has to flush data to prevent information loss. The OpenTelemetry client should support user-configurable timeout if it blocks on shut down.
-->

OpenTelemetryクライアントは、シャットダウン時にエンドユーザーのアプリケーションをブロックする可能性があります。シャットダウン時には、情報の損失を防ぐためにデータをフラッシュしなければなりません。OpenTelemetryクライアントは、シャットダウン時にブロックする場合、ユーザーが設定できるタイムアウトをサポートする必要があります。

<!--
If the OpenTelemetry client supports an explicit flush operation, it could block also. But should support a configurable timeout.
-->

OpenTelemetryクライアントが明示的なフラッシュ操作をサポートしていれば、ブロックすることも可能です。しかし、設定可能なタイムアウトをサポートする必要があります。

<!--
## Documentation
-->

## ドキュメンテーション

<!--
If language specific implementation has special characteristics that are not described in this document, such characteristics should be documented.
-->

言語固有の実装で、本文書に記載されていない特別な特性がある場合は、その特性を文書化する必要があります。
