<!--
# Versioning and stability for OpenTelemetry clients
-->

# OpenTelemetryクライアントのバージョニングと安定性

<!--
**Status**: [Stable](document-status.md)
-->

**Status**: [Stable](document-status.md)

<!-- Re-generate TOC with `markdown-toc --no-first-h1 -i` -->

<!-- toc -->

<!--
- [Design goals](#design-goals)
- [Signal lifecycle](#signal-lifecycle)
  * [Experimental](#experimental)
  * [Stable](#stable)
    + [API Stability](#api-stability)
    + [SDK Stability](#sdk-stability)
    + [Contrib Stability](#contrib-stability)
    + [NOT DEFINED: Telemetry Stability](#not-defined-telemetry-stability)
    + [NOT DEFINED: Semantic Conventions Stability](#not-defined-semantic-conventions-stability)
  * [Deprecated](#deprecated)
  * [Removed](#removed)
  * [A note on replacing signals](#a-note-on-replacing-signals)
- [Version numbers](#version-numbers)
  * [Major version bump](#major-version-bump)
  * [Minor version bump](#minor-version-bump)
  * [Patch version bump](#patch-version-bump)
- [Long Term Support](#long-term-support)
  * [API support](#api-support)
  * [SDK Support](#sdk-support)
  * [Contrib Support](#contrib-support)
- [OpenTelemetry GA](#opentelemetry-ga)
-->

- [デザインゴール](#デザインゴール)
- [シグナルライフサイクル](#シグナルライフサイクル)
  * [実験的(Experimental)](#実験的Experimental)
  * [安定的(stable)](#安定的stable)
    + [API 安定性](#api-安定性)
    + [SDK 安定性](#sdk-安定性)
    + [Contrib 安定性](#contrib-安定性)
    + [NOT DEFINED: Telemetry Stability](#not-defined-telemetry-stability)
    + [NOT DEFINED: Semantic Conventions Stability](#not-defined-semantic-conventions-stability)
  * [非推奨(deprecated)](#非推奨deprecated)
  * [削除(removed)](#削除removed)
  * [シグナルの置き換えに関する注意点](#シグナルの置き換えに関する注意点)
- [バージョン番号](#バージョン番号)
  * [メジャーバージョンアップ](#メジャーバージョンアップ)
  * [マイナーバージョンアップ](#マイナーバージョンアップ)
  * [パッチバージョンアップ](#パッチバージョンアップ)
- [長期サポート(LTS)](#長期サポートlts)
  * [API サポート](#api-サポート)
  * [SDK サポート](#sdk-サポート)
  * [Contrib サポート](#contrib-サポート)
- [OpenTelemetry GA](#opentelemetry-ga)


<!-- tocstop -->

<!--
This document defines the stability guarantees offered by the OpenTelemetry clients, along with the rules and procedures for meeting those guarantees.
-->

このドキュメントでは、OpenTelemetryのクライアントが提供する安定性の保証と、その保証を満たすためのルールや手順を定義しています。

<!--
In this document, the terms "OpenTelemetry" and "language implementations" both specifically refer to the OpenTelemetry clients.
These terms do not refer to the specification or the Collector in this document.
-->

このドキュメントでは、「OpenTelemetry」および「言語実装」という用語は、いずれもOpenTelemetryクライアントを具体的に指しています。これらの用語は、このドキュメントの仕様書やCollectorを指すものではありません。


<!--
Each language implementation MUST take these versioning and stability requirements, and produce a language-specific document which details how these requirements will be met.
This document SHALL be placed in the root of each repo and named `VERSIONING`.
-->

各言語の実装は、これらのバージョニングと安定性の要件を受けて、これらの要件をどのように満たすかを詳細に説明した言語固有のドキュメントを作成しなければなりません (MUST)。このドキュメントは、各リポジトリのルートに置かれ、`VERSIONING`という名前が付けられなければなりません(SHALL)。

<!--
## Design goals
-->

## デザインゴール

<!--
Versioning and stability procedures are designed to meet the following goals.
-->

バージョニングと安定性の手順は、以下の目標を達成するために設計されています。

<!--
**Ensure that application owners stay up to date with the latest release of the SDK.**
We want all users to stay up to date with the latest version of the OpenTelemetry SDK.
We do not want to create hard breaks in support, of any kind, which leave users stranded on older versions.
It MUST always be possible to upgrade to the latest minor version of the OpenTelemetry SDK, without creating compilation or runtime errors.
-->

**アプリケーションの所有者がSDKの最新リリースを維持することを保証します**。 私たちは、すべてのユーザーがOpenTelemetry SDKの最新バージョンを維持することを望みます。私たちは、ユーザーが古いバージョンに取り残されるような、いかなる種類のサポートも断絶することを望んでいません。コンパイルエラーやランタイムエラーを起こすことなく、常にOpenTelemetry SDKの最新のマイナーバージョンにアップグレードできるようにしなければなりません(MUST)。

<!--
**Never create a dependency conflict between packages which rely on different versions of OpenTelemetry. Avoid breaking all stable public APIs.**
Backwards compatibility is a strict requirement.
Instrumentation APIs cannot create a version conflict, ever. Otherwise, the OpenTelemetry API cannot be embedded in widely shared libraries, such as web frameworks.
Code written against older versions of the API MUST work with all newer versions of the API.
Transitive dependencies of the API cannot create a version conflict. The OpenTelemetry API cannot depend on a particular package if there is any chance that any library or application may require a different, incompatible version of that package.
A library that imports the OpenTelemetry API should never become incompatible with other libraries due to a version conflict in one of OpenTelemetry's dependencies.
Theoretically, APIs can be deprecated and eventually removed, but this is a process measured in years and we have no plans to do so.
-->

**異なるバージョンのOpenTelemetryに依存しているパッケージ間で依存関係の衝突を起こさないようにします。すべての安定した公開APIを壊さないようにします**。 後方互換性は厳格な要件です。計装APIは、絶対にバージョンの衝突を起こしてはいけません。そうしないと、OpenTelemetry APIを、Webフレームワークなどの広く共有されているライブラリに組み込むことができません。古いバージョンのAPIに対して書かれたコードは、新しいバージョンのAPIのすべてで動作しなければなりません(MUST)。API の推移的な依存関係によって、バージョンの競合が発生してはいけません。OpenTelemetry APIは、ライブラリやアプリケーションがそのパッケージの互換性のない別のバージョンを必要とする可能性がある場合、特定のパッケージに依存してはいけません。OpenTelemetry APIをインポートしているライブラリは、OpenTelemetryの依存関係のいずれかのバージョンが競合することで、他のライブラリとの互換性を失ってはいけません。理論的には、APIは非推奨となり、最終的には削除される可能性がありますが、これは数年単位のプロセスであり、私たちはそのような計画を持っていません。

<!--
**Allow for multiple levels of package stability within the same release of an OpenTelemetry component.**
Provide maintainers a clear process for developing new, experimental [signals](glossary.md#signals) alongside stable signals.
Different packages within the same release may have different levels of stability.
This means that an implementation wishing to release stable tracing today MUST ensure that experimental metrics are factored out in such a way that breaking changes to metrics API do not destabilize the trace API packages.
-->

**OpenTelemetryコンポーネントの同一リリース内で、複数のレベルのパッケージの安定性を許容します**。 メンテナに対して、安定したシグナルと並行して新しい実験的な[シグナル](glossary.md#signals)を開発するための明確なプロセスを提供します。同じリリースの中でも、パッケージによって安定性のレベルが異なる場合があります。
これは、安定したTraceをリリースしたいと考えている実装は、実験的なMetricsがMetrics APIへの重大な変更がトレースAPIパッケージを不安定にしないような方法で取り除かれていることを保証しなければなりません(MUST)

<!--
## Signal lifecycle
-->

## シグナルライフサイクル

<!--
The development of each signal follows a lifecycle: experimental, stable, deprecated, removed.
-->

各シグナルの開発は、実験的(experimental)、安定的(stable)、非推奨(deprecated)、削除(removed)というライフサイクルに沿って行われます。

<!--
The infographic below shows an example of the lifecycle of an API component.
-->

以下の図は、APIコンポーネントのライフサイクルの例を示しています。

<!--
![API Lifecycle](../internal/img/api-lifecycle.png)
-->

![API ライフサイクル](../internal/img/api-lifecycle.png)

<!--
### Experimental
-->

### 実験的(Experimental)

<!--
Signals start as **experimental**, which covers alpha, beta, and release candidate versions of the signal.
While signals are experimental, breaking changes and performance issues MAY occur.
Components SHOULD NOT be expected to be feature-complete.
In some cases, the experiment MAY be discarded and removed entirely.
Long-term dependencies SHOULD NOT be taken against experimental signals.
-->

シグナルは**実験的(experimental)**から開始され、シグナルのアルファ版、ベータ版、リリース候補版が対象となります。シグナルが実験的なものである間は、破壊的な変更や性能上の問題が発生する可能性があります。コンポーネントは機能的に完全であることを期待すべきではありません。場合によっては、実験を破棄して完全に削除してもかまいません(MAY)。実験的なシグナルに対して長期的な依存関係を取るべきではありません(SHOULD NOT)。

<!--
OpenTelemetry clients MUST be designed in a manner that allows experimental signals to be created without breaking the stability guarantees of existing signals.
-->

OpenTelemetryクライアントは、既存のシグナルの安定性保証を破ることなく実験的なシグナルを作成できるように設計されなければなりません(MUST)。

<!--
OpenTelemetry clients MUST NOT be designed in a manner that breaks existing users when a signal transitions from experimental to stable. This would punish users of the release candidate, and hinder adoption.
-->

OpenTelemetryのクライアントは、シグナルが実験的(experimental)から安定的(stable)に移行するときに既存のユーザーを壊すような方法を設計してはいけません(MUST NOT)。これはリリース候補のユーザーを罰することになり、採用の妨げになります。

<!--
Terms which denote stability, such as "experimental," MUST NOT be used as part of a directory or import name.
Package **version numbers** MAY include a suffix, such as -alpha, -beta, -rc, or -experimental, to differentiate stable and experimental packages.
-->

"experimental"のような安定性を示す用語は、ディレクトリ名やインポート名の一部として使ってはいけません(MUST NOT)。パッケージの **バージョン番号** には、安定版と実験版を区別するために、-alpha, -beta, -rc, -experimental などの接尾辞を付けてもかまいません(MAY)。

<!--
### Stable
-->

### 安定的(stable)

<!--
Once an experimental signal has gone through rigorous beta testing, it MAY transition to **stable**.
Long-term dependencies MAY now be taken against this signal.
-->

実験的なシグナルは厳格なベータテストを経て、 **安定** に移行してもかまいません(MAY)。このシグナルに対する長期的な依存関係を構築してもかまいません(MAY)。

<!--
All signal components MAY become stable together, or MAY transition to stability component-by-component. The API MUST become stable before the other components.
-->

すべてのシグナルコンポーネントは一緒に安定してもかまいませんし(MAY)、コンポーネントごとに安定に移行してもかまいません(MAY)。APIは他のコンポーネントよりも先に安定しなければなりません(MUST)。

<!--
Once a signal component is marked as stable, the following rules MUST apply until the end of that signal’s existence.
-->

シグナルコンポーネントが安定(stable)しているとマークされると、そのシグナルの存在が終了するまで、以下のルールが適用されなければなりません(MUST)。

<!--
#### API Stability
-->

#### API 安定性

<!--
Backward-incompatible changes to API packages MUST NOT be made unless the major version number is incremented.
All existing API calls MUST continue to compile and function against all future minor versions of the same major version.
-->

APIパッケージへの後方互換性のない変更は、メジャーバージョン番号が増加しない限り行ってはなりません(MUST NOT)。既存のすべてのAPIコールは、同じメジャーバージョンの将来のすべてのマイナーバージョンに対して、引き続きコンパイルして機能しなければなりません(MUST)。

<!--
Languages which ship binary artifacts SHOULD offer [ABI compatibility](glossary.md#abi-compatibility) for API packages.
-->

バイナリにコンパイルする言語は、APIパッケージに対して[ABI互換性](glossary.md#abi-compatibility)を提供すべきです(SHOULD)。

<!--
#### SDK Stability
-->

#### SDK 安定性

<!--
Public portions of SDK packages MUST remain backwards compatible.
There are two categories of public features: **plugin interfaces** and **constructors**.
Examples of plugins include the SpanProcessor, Exporter, and Sampler interfaces.
Examples of constructors include configuration objects, environment variables, and SDK builders.
-->

SDKパッケージの公開部分は、後方互換性を保たなければなりません(MUST)。公開機能には2つのカテゴリがあります。**プラグインインターフェース**と**Constructor**です。プラグインの例としては、SpanProcessor、Exporter、Samplerの各インターフェースがあります。Constructorの例としては、設定オブジェクト、環境変数、SDKビルダーなどがあります。

<!--
Languages which ship binary artifacts SHOULD offer [ABI compatibility](glossary.md#abi-compatibility) for SDK packages.
-->

バイナリにコンパイルする言語は、SDKパッケージに対して[ABI互換性](glossary.md#abi-compatibility)を提供すべきです(SHOULD)。

<!--
#### Contrib Stability
-->

#### Contrib 安定性

<!--
**NOTE: Until telemetry stability is defined, Contrib instrumentation MUST NOT be marked as stable. See below.**
-->

**注:テレメトリの安定性が定義されるまでは、Contribの計装は安定しているとマークしてはいけません。以下を参照してください。**

<!--
Plugins, instrumentation, and other contrib packages SHOULD be kept up to date and compatible with the latest versions of the API, SDK, and Semantic Conventions.
If a release of the API, SDK, or Semantic Conventions contains changes which are relevant to a contrib package, that package SHOULD be updated and released in a timely fashion.
The goal is to ensure users can update to the latest version of OpenTelemetry, and not be held back by the plugins that they depend on.
-->

プラグイン、計装、その他のcontribパッケージは、API、SDK、セマンティック規約の最新バージョンとの互換性を保つべきです(SHOULD)。API、SDK、セマンティック規約のリリースに、contribパッケージに関連する変更が含まれている場合、そのパッケージはタイムリーに更新され、リリースされるべきです(SHOULD)。その目的は、ユーザーがOpenTelemetryの最新バージョンにアップデートできるようにすることであり、依存しているプラグインによって妨げられることがないようにすることです。

<!--
Public portions of contrib packages (constructors, configuration, interfaces) SHOULD remain backwards compatible.
-->

contribパッケージの公開部分(コンストラクタ、設定、インタフェース)は、後方互換性を維持すべきです(SHOULD)。

<!--
Languages which ship binary artifacts SHOULD offer [ABI compatibility](glossary.md#abi-compatibility) for contrib packages.
-->

バイナリにコンパイルする言語は、contribパッケージに対して[ABI互換性](glossary.md#abi-compatibility)を提供すべきです(SHOULD)。

<!--
**Exception:** Contrib packages MAY break stability when a required downstream dependency breaks stability.
For example, a database integration may break stability if the required database client breaks stability.
However, it is strongly RECOMMENDED that older contrib packages remain stable.
A new, incompatible version of an integration SHOULD be released as a separate contrib package, rather than break the existing contrib package.
-->

**例外:** Contribパッケージは、必要な下流の依存関係が安定性を失う場合、安定性を失っても構いません(MAY)。例えば、必要なデータベースクライアントが安定性を失うと、データベースインテグレーションが安定性を失う可能性があります。しかし、古いcontribパッケージは安定していることが強く推奨されます(RECOMMENDED)。新しい、互換性のないバージョンの統合は、既存のcontribパッケージを壊すのではなく、別のcontribパッケージとしてリリースすべきです(SHOULD)。

<!--
#### NOT DEFINED: Telemetry Stability
-->

#### 未定義: テレメトリ安定性

<!--
**Telemetry stability guarantees are TBD.**
-->

**テレメトリ安定性はTBDです**

<!--
Changes to telemetry produced by OpenTelemetry instrumentation SHOULD avoid breaking analysis tools, such as dashboards and alerts.
However, it is not clear at this time what type of instrumentation changes (for example, adding additional spans and labels) would actually cause a breaking change.
-->

OpenTelemetry計装によって生成されるテレメトリへの変更は、ダッシュボードやアラートなどの分析ツールの破損を避けるべきです(SHOULD)。しかし、現時点では、どのようなタイプの計測器の変更(例えば、追加のSpanやラベルの追加)が、実際に壊れた変更を引き起こすかは明らかではありません。

<!--
#### NOT DEFINED: Semantic Conventions Stability
-->

#### 未定義: セマンティック規約安定性

<!--
Telemetry stability, including semantic conventions, is not currently defined. The following practices are recommended.
-->

セマンティック規約を含むテレメトリの安定性は、現在のところ定義されていません。以下のプラクティスが推奨されます。

<!--
Semantic Conventions SHOULD NOT be removed once they are added.
New conventions MAY be added to replace usage of older conventions, but the older conventions SHOULD NOT be removed.
Older conventions SHOULD be marked as deprecated when they are replaced by newer conventions.
-->

一度追加されたセマンティック規約は削除されるべきではありません(SHOULD NOT)。古い規約の使用を置き換えるために新しい規約を追加しても構いません(MAY)が、古い規約は削除されるべきではありません(SHOULD NOT)。古い規約が新しい規約に置き換えられた場合は、deprecatedとマークされるべきです(SHOULD)。

<!--
### Deprecated
-->

### 非推奨(deprecated)

<!--
Signals MAY eventually be replaced. When this happens, they are marked as deprecated.
-->

シグナルはいずれ置き換えられてもかまいません(MAY)。そのような場合は、deprecatedとマークされます。

<!--
Signals SHALL only be marked as deprecated when the replacement becomes stable.
Deprecated code MUST abide by the same support guarantees as stable code.
-->

シグナルは、代替品が安定してきたときにのみdeprecatedとマークされなければなりません(SHALL)。非推奨のコードは、安定版のコードと同じサポート保証に従わなければなりません(MUST)。

<!--
### Removed
-->

### 削除(removed)

<!--
Support is ended by the removal of a signal from the release.
The release MUST make a major version bump when this happens.
-->

サポートの終了は、リリースからシグナルが削除されることによって行われます。このような場合、そのリリースはメジャーバージョンに移行しなければなりません(MUST)。

<!--
### A note on replacing signals
-->

### シグナルの置き換えに関する注意点

<!--
Note that we currently have no plans for creating a major version of OpenTelemetry past v1.0.
-->

現在のところ、v1.0以降のOpenTelemetryのメジャーバージョンを作成する予定はないことに留意してください。

<!--
For clarity, it is still possible to create new, backwards incompatible versions of existing signals without actually moving to v2.0 and breaking support.
-->

明確にするために、実際にv2.0に移行してサポートを打ち切らなくても、既存のシグナルの下位互換性のない新しいバージョンを作ることは可能です。

<!--
For example, imagine we develop a new, better tracing API - let's call it AwesomeTrace.
We will never mutate the current tracing API into AwesomeTrace.
Instead, AwesomeTrace would be added as an entirely new signal which coexists and interoperates with the current tracing signal.
This would make adding AwesomeTrace a minor version bump, *not* v2.0.
v2.0 would mark the end of support for current tracing, not the addition of AwesomeTrace.
And we don't want to ever end that support, if we can help it.
-->

例えば、新しい、より良いTrace APIを開発したとしましょう。現在のTrace API を AwesomeTrace に変更することはありません。代わりに、AwesomeTrace は現在のトレースシグナルと共存・相互運用する全く新しいシグナルとして追加されます。これにより、AwesomeTrace の追加は v2.0 ではなく、マイナーバージョンアップとなります。v2.0 は現在のトレーシングのサポート終了を意味し、AwesomeTrace の追加ではありません。できればサポートを終了させたくはありません。

<!--
This is not actually a theoretical example.
OpenTelemetry already supports two tracing APIs: OpenTelemetry and OpenTracing.
We invented a new tracing API, but continue to support the old one.
-->

実はこれは理論上の例ではありません。OpenTelemetryはすでに2つのトレーシングAPIをサポートしています。OpenTelemetryとOpenTracingです。私たちは新しいトレーシングAPIを発明しましたが、古いAPIは引き続きサポートしています。

<!--
## Version numbers
-->

## バージョン番号

<!--
OpenTelemetry clients follow [Semantic Versioning 2.0.0](https://semver.org/spec/v2.0.0.html), with the following clarifications.
-->

OpenTelemetryのクライアントは、[Semantic Versioning 2.0.0](https://semver.org/spec/v2.0.0.html)に従っており、以下の点が明確になっています。

<!--
OpenTelemetry clients have four components: API, SDK, Semantic Conventions, and Contrib.
-->

OpenTelemetryのクライアントには4つのコンポーネントがあります。API、SDK、セマンティック規約、そしてContribです。

<!--
For the purposes of versioning, all code within a component MUST treated as if it were part of a single package, and versioned with the same version number,
except for Contrib, which may be a collection of packages versioned separately.
-->

バージョニングの目的ために、コンポーネント内のすべてのコードは、1つのパッケージの一部であるかのように扱われ、同じバージョン番号でバージョン管理されなければなりません(MUST)。ただし、Contribは別個にバージョン管理されたパッケージの集合体である可能性があります。

<!--
* All stable API packages MUST version together, across all signals.
Stable signals MUST NOT have separate version numbers.
There is one version number that applies to all signals that are included in the API release that is labeled with that particular version number.
* SDK packages for all signals MUST version together, across all signals.
Signals MUST NOT have separate version numbers.
There is one version number that applies to all signals that are included in the SDK release that is labeled with that particular version number.
* Semantic Conventions are a single package with a single version number.
* Each contrib package MAY have it's own version number.
* The API, SDK, Semantic Conventions, and contrib components have independent version numbers.
For example, the latest version of `opentelemetry-python-api` MAY be at v1.2.3 while the latest version of `opentelemetry-python-sdk` is at v2.3.1.
* Different language implementations have independent version numbers.
For example, it is fine to have `opentelemetry-python-api` at v1.2.8 when `opentelemetry-java-api` is at v1.3.2.
* Language implementations have version numbers which are independent of the specification they implement.
For example, it is fine for v1.8.2 of `opentelemetry-python-api` to implement v1.1.1 of the specification.
-->

* すべての安定した API パッケージは、すべてのシグナルで一緒にバージョン管理しなければなりません(MUST)。安定したシグナルは、別々のバージョン番号を持ってはいけません(MUST NOT)。特定のバージョン番号でラベル付けされたAPIリリースに含まれるすべてのシグナルに適用されるバージョン番号は1つです。
* すべてのシグナルのSDKパッケージは、すべてのシグナルにわたって一緒にバージョン付けされなければなりません(MUST)。すべてのシグナルのSDKパッケージは、すべてのシグナルにわたって一緒にバージョン管理しなければなりません(MUST NOT)。特定のバージョン番号でラベル付けされたSDKリリースに含まれるすべてのシグナルに適用される1つのバージョン番号があります。
* セマンティック規約は、1つのバージョン番号を持つ1つのパッケージです。
* 各contribパッケージは、それぞれのバージョン番号を持っていてもかまいません(MAY)。
* API、SDK、Semantic Conventions、および contrib の各コンポーネントは、それぞれ独立したバージョン番号を持っています。例えば、`opentelemetry-python-api` の最新バージョンは v1.2.3 で、`opentelemetry-python-sdk` の最新バージョンは v2.3.1 であってもかまいません(MAY)。
* 言語の実装によって、バージョン番号が異なります。たとえば、`opentelemetry-java-api` の最新バージョンが v1.3.2 なのに、`opentelemetry-python-api` の最新バージョンが v1.2.8 であっても構いません。
* 言語の実装には、実装している仕様とは独立したバージョン番号があります。たとえば、`opentelemetry-python-api` の v1.8.2 が仕様の v1.1.1 を実装していても問題ありません。


<!--
**Exception:** in some languages, package managers may react poorly to experimental packages having a version higher than 0.X.
In these cases, experimental signals MAY version independently from stable signals, in order to retain a 0.X version number.
When a signal becomes stable, the version MUST be bumped to match the other stable signals in the release.
-->

**例外:** 言語によっては、実験的なパッケージのバージョンが 0.X よりも高いと、パッケージマネージャの反応が悪くなることがあります。このような場合、実験的なシグナルは、0.X のバージョン番号を維持するために、安定したシグナルとは別にバージョンを上げてもかまいません(MAY)。シグナルが安定版になったときには、そのリリースに含まれる他の安定版シグナルに合わせてバージョンを上げなければなりません(MUST)。

<!--
### Major version bump
-->

### メジャーバージョンアップ

<!--
Major version bumps MUST occur when there is a breaking change to a stable interface, the removal of a deprecated signal, or a drop in support for a language or runtime version.
Major version bumps SHOULD NOT occur for changes which do not result in a drop in support of some form.
-->

メジャーバージョンアップは、安定したインターフェースへの破壊的な変更、非推奨のシグナルの削除、言語やランタイムのバージョンのサポート低下などがあった場合に発生しなければなりません(MUST)。メジャーバージョンアップは、何らかの形でサポートの低下をもたらさない変更の場合には発生すべきではありません(SHOULD NOT)。

<!--
### Minor version bump
-->

### マイナーバージョンアップ

<!--
Most changes to OpenTelemetry clients result in a minor version bump.
-->

OpenTelemetryクライアントのほとんどの変更は、マイナーバージョンの変更になります。

<!--
* New backward-compatible functionality added to any component.
* Breaking changes to internal SDK components.
* Breaking changes to experimental signals.
* New experimental signals are added.
* Experimental signals become stable.
* Stable signals are deprecated.
-->

* 任意のコンポーネントに追加される下位互換性のある新機能
* 内部のSDKコンポーネントの変更
* 実験的なシグナルの変更
* 新しい実験的なシグナルの追加
* 実験的なシグナルを安定に移行
* 安定したシグナルの非推奨(deprecated)

<!--
### Patch version bump
-->

### パッチバージョンアップ

<!--
Patch versions make no changes which would require recompilation or potentially break application code.
The following are examples of patch fixes.
-->

パッチバージョンでは、再コンパイルが必要となるような変更や、アプリケーションコードが壊れる可能性のある変更は行われません。以下にパッチ修正の例を示します。


<!--
* Bug fixes which don't require minor version bump per rules above.
* Security fixes.
* Documentation.
-->

* マイナーバージョンへの移行を必要としないバグフィックス
* セキュリティ修正
* ドキュメンテーション

<!--
Currently, the OpenTelemetry project does NOT have plans to backport bug and security fixes to prior minor versions of the SDK.
Security and bug fixes MAY only be applied to the latest minor version.
We are committed to making it feasible for end users to stay up to date with the latest version of the OpenTelemetry SDK.
-->

現在、OpenTelemetryプロジェクトでは、バグやセキュリティの修正をSDKの以前のマイナーバージョンにバックポートする予定はありません。セキュリティやバグの修正は、最新のマイナーバージョンにのみ適用してもかまいません。私たちは、エンドユーザーの皆様がOpenTelemetry SDKの最新バージョンをご利用いただけるように努めてまいります。

<!--
## Long Term Support
-->

## 長期サポート(LTS)

<!--
![long term support](../internal/img/long-term-support.png)
-->

![long term support](../internal/img/long-term-support.png)

<!--
### API support
-->

### API サポート

<!--
Major versions of the API MUST be supported for a minimum of **three years** after the release of the next major API version.
API support is defined as follows.
-->

APIのメジャーバージョンは、次のメジャーバージョンのAPIがリリースされた後、最低でも**3年間**サポートされなければなりません(MUST)。APIサポートの定義は以下の通りです。

<!--
* API stability, as defined above, MUST be maintained.
-->

* APIの安定性は、上記の通り、維持しなければなりません(MUST)

<!--
* A version of the SDK which supports the latest minor version of the last major version of the API will continue to be maintained during LTS.
Bug and security fixes MUST be backported. Additional feature development is NOT RECOMMENDED.
-->

* 最後のメジャーバージョンのAPIの最新マイナーバージョンをサポートしているSDKのバージョンは、LTS期間中も維持されます。バグやセキュリティの修正はバックポートされなければなりません(MUST)。追加機能の開発は推奨されません(NOT RECOMMENDED)。

<!--
* Contrib packages available when the API is versioned MUST continue to be maintained for the duration of LTS.
Bug and security fixes will be backported.
Additional feature development is NOT RECOMMENDED.
-->

* APIがバージョン管理されているときに利用可能なContribパッケージは、LTSの期間中、継続してメンテナンスされなければなりません(MUST)。バグやセキュリティの修正はバックポートされます。追加機能の開発は推奨されません(NOT RECOMMENDED)。

<!--
### SDK Support
-->

### SDK サポート

<!--
SDK stability, as defined above, will be maintained for a minimum of **one year** after the release of the next major SDK version.
-->

上記で定義されたSDKの安定性は、次のメジャーバージョンのSDKがリリースされてから最低でも**1年間**維持されます。

<!--
### Contrib Support
-->

### Contrib サポート

<!--
Contrib stability, as defined above, will be maintained for a minimum of **one year** after the release of the next major version of a contrib package.
-->

上記で定義されたContribの安定性は、contribパッケージの次のメジャーバージョンがリリースされてから最低**1年間維持されます。

<!--
## OpenTelemetry GA
-->

## OpenTelemetry GA

<!--
The term “OpenTelemetry GA” refers to the point at which OpenTracing and OpenCensus will be fully deprecated.
The **minimum requirements** for declaring GA are as followed.
-->

「OpenTelemetry GA」とは、OpenTracingとOpenCensusが完全に非推奨となる時点を指します。GAを宣言するための**最低限の要件**は以下の通りです。

<!--
* A stable version of both tracing and metrics MUST be released in at least four languages.
* CI/CD, performance, and integration tests MUST be implemented for these languages.
-->

* TraceとMetricsの両方の安定版が、少なくとも4つの言語でリリースされなければなりません(MUST)
* CI/CD、パフォーマンス、統合テストがこれらの言語で実装されていなければなりません
