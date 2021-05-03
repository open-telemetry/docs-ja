<!--
# OpenTelemetry Client Design Principles
-->

# OpenTelemetryクライアントのデザイン原則

<!--
This document defines common principles that will help designers create OpenTelemetry clients that are easy to use, are uniform across all supported languages, yet allow enough flexibility for language-specific expressiveness.
-->

このドキュメントでは、設計者が、使いやすく、サポートされているすべての言語で統一されていて、かつ言語固有の表現力に十分な柔軟性を備えたOpenTelemetryクライアントを作成するのに役立つ共通なデザイン原則を定義しています。

<!--
OpenTelemetry clients are expected to provide full features out of the box and allow for innovation and experimentation through extensibility.
-->

OpenTelemetryのクライアントは、完全な機能をすぐに提供し、拡張性によって革新と実験を可能にすることが期待されています。

<!--
Please read the [overview](overview.md) first, to understand the fundamental architecture of OpenTelemetry.
-->

OpenTelemetryの基本的なアーキテクチャを理解するために、まず[概要](overview.md)をお読みください。

<!--
This document does not attempt to describe the details or functionality of the OpenTelemetry client API. For API specs see the [API specifications](../README.md).
-->

このドキュメントは、OpenTelemetryクライアントAPIの詳細や機能を説明しようとするものではありません。APIの仕様については、[API仕様](../README.md)を参照してください。

<!--
_Note to OpenTelemetry client Authors:_ OpenTelemetry specification, API and SDK implementation guidelines are work in progress. If you notice incomplete or missing information, contradictions, inconsistent styling and other defects please let specification writers know by creating an issue in this repository or posting in [Gitter](https://gitter.im/open-telemetry/opentelemetry-specification). As implementors of the specification you will often have valuable insights into how the specification can be improved. The Specification SIG and members of Technical Committee highly value your opinion and welcome your feedback.
-->

_OpenTelemetryクライアント作成者への注意事項:_ OpenTelemetryの仕様、API、SDKの実装ガイドラインは、現在進行中です。不完全な情報や欠落している情報、矛盾点、一貫性のないスタイル、その他の不具合にお気づきの場合は、このリポジトリに課題を作成するか、[Gitter](https://gitter.im/open-telemetry/opentelemetry-specification)に投稿して、仕様書作成者にお知らせください。仕様書の実装者であるあなたは、仕様書をどのように改善すればよいかについて、しばしば貴重な洞察力を持っています。仕様書SIGと技術委員会のメンバーは、あなたの意見を高く評価し、あなたのフィードバックを歓迎します。

<!--
## Requirements
-->

## 要件

<!--
1. The OpenTelemetry API must be well-defined and clearly decoupled from the implementation. This allows end users to consume API only without also consuming the implementation (see points 2 and 3 for why it is important).
-->

1. OpenTelemetryのAPIは明確に定義され、実装から明確に切り離されていなければなりません。これにより、エンドユーザーは実装を使うのではなく、APIのみを使うことができます(重要な理由についてはポイント2と3を参照)。

<!--
2. Third party libraries and frameworks that add instrumentation to their code will have a dependency only on the API of OpenTelemetry client. The developers of third party libraries and frameworks do not care (and cannot know) what specific implementation of OpenTelemetry is used in the final application.
-->

2. コードに計装を追加するサードパーティのライブラリやフレームワークは、OpenTelemetryクライアントのAPIにのみ依存することになります。サードパーティのライブラリやフレームワークの開発者は、最終的なアプリケーションで使用されるOpenTelemetryの特定の実装を気にしません(し、知ることもできません)。

<!--
3. The developers of the final application normally decide how to configure OpenTelemetry SDK and what extensions to use. They should be also free to choose to not use any OpenTelemetry implementation at all, even though the application and/or its libraries are already instrumented.  The rationale is that third-party libraries and frameworks which are instrumented with OpenTelemetry must still be fully usable in the applications which do not want to use OpenTelemetry (so this removes the need for framework developers to have "instrumented" and "non-instrumented" versions of their framework).
-->

3. 最終的なアプリケーションの開発者は、通常、OpenTelemetry SDKをどのように設定するか、どの拡張機能を使用するかを決定します。また、アプリケーションやそのライブラリがすでに計装されている場合でも、OpenTelemetryの実装を一切使用しないという選択も自由にできるはずです。 その理由は、OpenTelemetryで計装されたサードパーティのライブラリやフレームワークは、OpenTelemetryを使用したくないアプリケーションでも、完全に使用可能でなければならないからです(そのため、フレームワーク開発者が自分のフレームワークに「計装されたバージョン」と「計装されていないバージョン」を用意する必要がなくなります)。

<!--
4. The SDK must be clearly separated into wire protocol-independent parts that implement common logic (e.g. batching, tag enrichment by process information, etc.) and protocol-dependent telemetry exporters. Telemetry exporters must contain minimal functionality, thus enabling vendors to easily add support for their specific protocol.
-->

4. SDKは、共通のロジック(例:バッチ処理、プロセス情報によるタグの改良など)を実装するワイヤープロトコルに依存しない部分と、プロトコルに依存するテレメトリ・エクスポーターに明確に分離されていなければなりません。テレメトリ・エクスポータには最小限の機能しか含まれていないため、ベンダーは特定のプロトコルのサポートを容易に追加することができます。

<!--
5. The SDK implementation should include the following exporters:
    - OTLP.
    - Jaeger.
    - Zipkin.
    - Prometheus.
    - Standard output (or logging) to use for debugging and testing as well as an input for the various log proxy tools.
    - In-memory (mock) exporter that accumulates telemetry data in the local memory and allows to inspect it (useful for e.g. unit tests).
-->

5. SDKの実装には、以下のエクスポーターを含める必要があります:
    - OTLP
    - Jaeger
    - Zipkin
    - Prometheus
    - 標準的な出力(またはロギング)。デバッグやテストに使用するためでもあり、様々なログプロキシツールの入力でもあります
    - テレメトリデータをローカルメモリに蓄積し、検査できる、インメモリ(モック)エクスポーター(ユニットテストなどに便利です)


<!--
    Note: some of these support multiple protocols (e.g. gRPC, Thrift, etc). The exact list of protocols to implement in the exporters is TBD.
-->

    注:これらの中には、複数のプロトコルをサポートしているものがあります(例:gRPC、Thriftなど)。エクスポーターに実装するプロトコルの正確なリストは未定です。

<!--
    Other vendor-specific exporters (exporters that implement vendor protocols) should not be included in OpenTelemetry clients and should be placed elsewhere (the exact approach for storing and maintaining vendor-specific exporters will be defined in the future).
-->

    その他のベンダー固有のエクスポーター(ベンダー独自のプロトコルを実装するエクスポーター)は、OpenTelemetryクライアントに含めるべきではなく、別の場所に置くべきです(ベンダー固有のエクスポーターを保存・管理するための正確な方法は、将来的に定義される予定です)。

<!--
## OpenTelemetry Client Generic Design
-->

## OpenTelemetryクライアントの一般的な設計

<!--
Here is a generic design for an OpenTelemetry client (arrows indicate calls):
-->

ここでは、OpenTelemetryクライアントの一般的な設計を紹介します(矢印はコールを示します)。

<!--
![OpenTelemetry client Design Diagram](../internal/img/library-design.png)
-->

![OpenTelemetry クライアント 設計ダイアグラム](../internal/img/library-design.png)

<!--
### Expected Usage
-->

### 期待される使い方

<!--
The OpenTelemetry client is composed of 4 types of [packages](glossary.md#packages): API packages, SDK packages, a Semantic Conventions package, and plugin packages.
The API and the SDK are split into multiple packages, based on signal type (e.g. one for api-trace, one for api-metric, one for sdk-trace, one for sdk-metric) is considered an implementation detail as long as the API artifact(s) stay separate from the SDK artifact(s).
-->

OpenTelemetryクライアントは、4種類の[パッケージ](glossary.md#packages)で構成されています。APIパッケージ、SDKパッケージ、セマンティック規約パッケージ、そしてプラグインパッケージです。APIとSDKはシグナルタイプに応じて複数のパッケージに分割されています(例:api-trace用のパッケージ、api-metric用のパッケージ、sdk-trace用のパッケージ、sdk-metric用のパッケージ)が、APIの成果物とSDKの成果物が分離されている限り、実装の詳細は考慮されません。

<!--
Libraries, frameworks, and applications that want to be instrumented with OpenTelemetry take a dependency only on the API packages. The developers of these third-party libraries will make calls to the API to produce telemetry data.
-->

OpenTelemetry での計装を希望するライブラリ、フレームワーク、およびアプリケーションは、API パッケージにのみ依存します。これらのサードパーティライブラリの開発者は、APIを呼び出してテレメトリデータを生成します。

<!--
Applications that use third-party libraries that are instrumented with OpenTelemetry API control whether or not to install the SDK and generate telemetry data. When the SDK is not installed, the API calls should be no-ops which generate minimal overhead.
-->

OpenTelemetry APIで計装されたサードパーティのライブラリを使用するアプリケーションは、SDKをインストールしてテレメトリデータを生成するかどうかを制御します。SDKがインストールされていない場合、APIコールは最小限のオーバーヘッドを発生させるno-opでなければなりません。

<!--
In order to enable telemetry the application must take a dependency on the OpenTelemetry SDK. The application must also configure exporters and other plugins so that telemetry can be correctly generated and delivered to their analysis tool(s) of choice. The details of how plugins are enabled and configured are language specific.
-->

テレメトリを有効にするためには、アプリケーションは OpenTelemetry SDK への依存を取る必要があります。また、アプリケーションは、テレメトリが正しく生成され、選択した解析ツールに配信されるように、エクスポーターやその他のプラグインを設定しなければなりません。プラグインがどのように有効化され、設定されるかの詳細は、言語によって異なります。

<!--
### API and Minimal Implementation
-->

### APIと最小限の実装

<!--
The API package is a self-sufficient dependency, in the sense that if the end-user application or a third-party library depends only on it and does not plug a full SDK implementation then the application will still build and run without failing, although no telemetry data will be actually delivered to a telemetry backend.
-->

APIパッケージは自給自足の依存関係にあります。エンドユーザーのアプリケーションやサードパーティのライブラリがAPIパッケージのみに依存し、完全なSDKの実装を使用しない場合、アプリケーションは失敗することなくビルド、実行されますが、テレメトリデータは実際にはテレメトリ・バックエンドに配信されません。

<!--
This self-sufficiency is achieved the following way.
-->

この自給自足を実現するには、次のような方法があります。

<!--
The API dependency contains a minimal implementation of the API. When no other implementation is explicitly included in the application no telemetry data will be collected. Here is what active components look like in this case:
-->

APIの依存関係には、APIの最小限の実装が含まれています。アプリケーションに他の実装が明示的に含まれていない場合、テレメトリデータは収集されません。この場合のアクティブなコンポーネントは次のようになります。

<!--
![Minimal Operation Diagram](../internal/img/library-minimal.png)
-->

![最小限操作のダイアグラム](../internal/img/library-minimal.png)

<!--
It is important that values returned from this minimal implementation of API are valid and do not require the caller to perform extra checks (e.g. createSpan() method should not fail and should return a valid non-null Span object). The caller should not need to know and worry about the fact that minimal implementation is in effect. This minimizes the boilerplate and error handling in the instrumented code.
-->

このAPIの最小限の実装から返される値が有効であり、呼び出し側が余分なチェックを行う必要がないことが重要です(例えば、createSpan()メソッドは失敗してはならず、有効なnullではないSpanオブジェクトを返さなければなりません)。呼び出し側は、最小限の実装が実際に使えるかどうかを知り、心配する必要はありません。これにより、計装コードの定型文やエラー処理を最小限に抑えることができます。

<!--
It is also important that minimal implementation incurs as little performance penalty as possible, so that third-party frameworks and libraries that are instrumented with OpenTelemetry impose negligible overheads to users of such libraries that do not want to use OpenTelemetry too.
-->

また、OpenTelemetryを搭載したサードパーティのフレームワークやライブラリが、OpenTelemetryの使用を望まないライブラリのユーザーに無視できる程度のオーバーヘッドを与えるように、最小限の実装でパフォーマンス上のペナルティを可能な限り少なくすることも重要です。

<!--
### SDK Implementation
-->

### SDK 実装

<!--
SDK implementation is a separate (optional) dependency. When it is plugged in it substitutes the minimal implementation that is included in the API package (exact substitution mechanism is language dependent).
-->

SDKの実装は、独立した(任意の)依存関係にあります。これが使用されると、APIパッケージに含まれる最小限の実装に置き換わります(正確な置換メカニズムは言語によって異なります)。

<!--
SDK implements core functionality that is required for translating API calls into telemetry data that is ready for exporting. Here is how OpenTelemetry components look like when SDK is enabled:
-->

SDKは、APIコールをエクスポート可能なテレメトリデータに変換するために必要なコア機能を実装しています。以下は、SDKを有効にした場合のOpenTelemetryコンポーネントの外観です。

<!--
![Full Operation Diagram](../internal/img/library-full.png)
-->

![完全な操作のダイアグラム](../internal/img/library-full.png)

<!--
SDK defines an [Exporter interface](trace/sdk.md#span-exporter). Protocol-specific exporters that are responsible for sending telemetry data to backends must implement this interface.
-->

SDKでは、[Exporter]インターフェース(trace/sdk.md#span-exporter)を定義しています。バックエンドへのテレメトリデータの送信を担当するプロトコル固有のExporterは、このインターフェイスを実装する必要があります。

<!--
SDK also includes optional helper exporters that can be composed for additional functionality if needed.
-->

また、SDKには任意で使えるヘルパーエクスポーターが用意されており、必要に応じてそれらを組み合わせて機能を追加することができます。

<!--
Library designers need to define the language-specific `Exporter` interface based on [this generic specification](trace/sdk.md#span-exporter).
-->

ライブラリの設計者は、[一般的な仕様](trace/sdk.md#span-exporter)に基づいて、言語固有の`Exporter`インターフェースを定義する必要があります。

<!--
#### Protocol Exporters
-->

#### プロトコル・エクスポーター

<!--
Telemetry backend vendors are expected to implement [Exporter interface](trace/sdk.md#span-exporter). Data received via Export() function should be serialized and sent to the backend in a vendor-specific way.
-->

テレメトリ・バックエンドのベンダーは、[Exporter interface](trace/sdk.md#span-exporter)を実装することが期待されています。Export()関数で受け取ったデータは、ベンダー固有の方法でシリアライズされ、バックエンドに送信されなければなりません。

<!--
Vendors are encouraged to keep protocol-specific exporters as simple as possible and achieve desirable additional functionality such as queuing and retrying using helpers provided by SDK.
-->

ベンダーは、エクスポーターのプロトコル固有部分を可能な限りシンプルに保ち、キューイングやリトライなどの望ましい追加機能はSDKが提供するヘルパーを使って実現することが推奨されます。

<!--
End users should be given the flexibility of making many of the decisions regarding the queuing, retrying, tagging, batching functionality that make the most sense for their application. For example, if an application's telemetry data must be delivered to a remote backend that has no guaranteed availability the end user may choose to use a persistent local queue and an `Exporter` to retry sending on failures. As opposed to that for an application that sends telemetry to a locally running Agent daemon, the end user may prefer to have a simpler exporting configuration without retrying or queueing.
-->

エンドユーザーには、アプリケーションにとって最も意味のあるキューイング、リトライ、タグ付け、バッチ機能に関する多くの決定を行う柔軟性が与えられるべきです。例えば、アプリケーションのテレメトリデータを、可用性が保証されていないリモートのバックエンドに配信しなければならない場合、エンドユーザーは、永続的なローカルキューと、失敗時に送信を再試行する`Exporter`の使用を選択することができます。一方、ローカルで動作するAgentデーモンにテレメトリを送信するアプリケーションの場合は、エンドユーザーは再試行やキューを使用しない、よりシンプルなエクスポーターを使う構成を好むかもしれません。

<!--
If additional exporters for the sdk are provided as separate libraries, the
name of the library should be prefixed with the terms "OpenTelemetry" and "Exporter" in accordance with the naming conventions of the respective technology.
-->

SDK用の追加のエクスポーターが個別のライブラリーとして提供される場合は、それぞれの技術の命名規則に従った上で、ライブラリーの名前の前に "OpenTelemetry "と "Exporter "の用語を付ける必要があります。

<!--
For example:
-->

例:

<!--
- Python and Java: opentelemetry-exporter-jaeger
- Javascript: @opentelemetry/exporter-jeager
-->

- Python や Java: opentelemetry-exporter-jaeger
- Javascript: @opentelemetry/exporter-jeager

<!--
#### Resource Detection
-->

#### リソース検出

<!--
Cloud vendors are encouraged to provide packages to detect resource information from the environment. These MUST be implemented outside of the SDK. See [Resource SDK](./resource/sdk.md#detecting-resource-information-from-the-environment) for more details.
-->

クラウドベンダーは、環境からリソース情報を検出するためのパッケージを提供することが推奨されます。これらはSDKの外で実装されなければなりません(MUST)。詳細は[リソースSDK](./resource/sdk.md#detecting-resource-information-from-the-environment)を参照してください。

<!--
### Alternative Implementations
-->

### 別実装

<!--
The end-user application may decide to take a dependency on alternative implementation.
-->

エンドユーザーのアプリケーションは、代替の実装を使うことを決定できます。

<!--
SDK provides flexibility and extensibility that may be used by many implementations. Before developing an alternative implementation, please, review extensibility points provided by OpenTelemetry.
-->

SDKは柔軟性と拡張性を備えており、多くの実装に利用することができます。代替実装を開発する前に、OpenTelemetryが提供する拡張性のポイントを確認してください。

<!--
An example use-case for alternate implementations is automated testing. A mock implementation can be plugged in during automated tests. For example, it can store all generated telemetry data in memory and provide a capability to inspect this stored data. This will allow the tests to verify that the telemetry is generated correctly. OpenTelemetry client authors are encouraged to provide such a mock implementation.
-->

代替実装の使用例として、自動テストがあります。自動化されたテストの際に、モックの実装を差し込むことができます。例えば、生成されたすべてのテレメトリデータをメモリに格納し、この格納されたデータを検査する機能を提供できます。これにより、テストでテレメトリが正しく生成されているかどうかを検証できます。OpenTelemetryクライアントの作者は、そのようなモック実装を提供することが推奨されます。

<!--
Note that mocking is also possible by using SDK and a Mock `Exporter` without needing to swap out the entire SDK.
-->

なお、SDKとMock `Exporter`を使うことで、SDK全体を入れ替えることなく、モックを作ることができます。

<!--
The mocking approach chosen will depend on the testing goals and at which point exactly it is desirable to intercept the telemetry data path during the test.
-->

どのようなモックアップ手法を選択するかは、テストの目的や、テスト中にどの時点でテレメトリ・データ・パスをインターセプトすることが望ましいかによって異なります。

<!--
### Version Labeling
-->

### バージョン表示

<!--
API and SDK packages must use semantic version numbering. API package version number and SDK package version number are decoupled and can be different (and they both can be also different from the Specification version number that they implement). API and SDK packages MUST be labeled with their own version number.
-->

APIおよびSDKパッケージは、セマンティックバージョン番号を使用しなければなりません。APIパッケージのバージョン番号とSDKパッケージのバージョン番号は切り離されており、異なっていても構いません(また、それらは両方とも、それらが実装する仕様のバージョン番号とは異なることができます)。APIパッケージとSDKパッケージは、それぞれのバージョン番号でラベル付けされなければなりません(MUST)。


<!--
This decoupling of version numbers allows OpenTelemetry client authors to make API and SDK package releases independently without the need to coordinate and match version numbers with the Specification.
-->

このようにバージョン番号が切り離されているため、OpenTelemetryクライアントの作者は、仕様書とのバージョン番号の調整や一致を必要とせずに、APIやSDKのパッケージを独立してリリースすることができます。

<!--
Because API and SDK package version numbers are not coupled, every API and SDK package release MUST clearly mention the Specification version number that they implement. In addition, if a particular version of SDK package is only compatible with a specific version of API package, then this compatibility information must be also published by OpenTelemetry client authors. OpenTelemetry client authors MUST include this information in the release notes. For example, the SDK package release notes may say: "SDK 0.3.4, use with API 0.1.0, implements OpenTelemetry Specification 0.1.0".
-->

APIとSDKパッケージのバージョン番号は連動していないため、すべてのAPIとSDKパッケージのリリースは、それらが実装している仕様のバージョン番号を明確に言及しなければなりません(MUST)。さらに、特定のバージョンのSDKパッケージが特定のバージョンのAPIパッケージとしか互換性がない場合は、この互換性情報もOpenTelemetryクライアントの作成者が公開しなければなりません。OpenTelemetryクライアントの作成者は、この情報をリリースノートに含めなければなりません(MUST)。例えば、SDKパッケージのリリースノートには次のように書かれているかもしれません。: "SDK 0.3.4, use with API 0.1.0, implements OpenTelemetry Specification 0.1.0".

<!--
_TODO: How should third-party library authors who use OpenTelemetry for instrumentation guide their end users to find the correct SDK package?_
-->

_TODO: OpenTelemetryを計装に使用するサードパーティのライブラリ作者は、エンドユーザーが正しいSDKパッケージを見つけられるよう、どのように案内すべきでしょうか?_

<!--
### Performance and Blocking
-->

### パフォーマンスとブロッキング

<!--
See the [Performance and Blocking](performance.md) specification for
guidelines on the performance expectations that API implementations should meet, strategies for meeting these expectations, and a description of how implementations should document their behavior under load.
-->

APIの実装が満たすべきパフォーマンスのガイドライン、これらの期待に応えるための戦略、実装が負荷時の動作をどのように文書化すべきかについては、[パフォーマンスとブロッキング](performance.md)仕様を参照してください。

<!--
### Concurrency and Thread-Safety
-->

### 並行性とスレッド安全性

<!--
Please refer to individual API specification for guidelines on what concurrency
safeties should API implementations provide and how they should be documented:
-->

APIの実装が提供すべき同時実行安全性とその文書化のガイドラインについては、個々のAPI仕様を参照してください。

<!--
* [Metrics API](./metrics/api.md#concurrency)
* [Tracing API](./trace/api.md#concurrency)
-->

* [Metrics API](./metrics/api.md#concurrency)
* [Tracing API](./trace/api.md#concurrency)

