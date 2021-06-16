<!--
# OpenTelemetry Logging Overview
-->

# OpenTelemetry Logging 概要

**Status**: [Experimental](../document-status.md)

<!--
* [Introduction](#introduction)
* [Limitations of non-OpenTelemetry Solutions](#limitations-of-non-opentelemetry-solutions)
* [OpenTelemetry Solution](#opentelemetry-solution)
* [Log Correlation](#log-correlation)
* [Events and Logs](#events-and-logs)
* [Legacy and Modern Log Sources](#legacy-and-modern-log-sources)
  * [System Logs](#system-logs)
  * [Infrastructure Logs](#infrastructure-logs)
  * [Third-party Application Logs](#third-party-application-logs)
  * [Legacy First-Party Applications Logs](#legacy-first-party-applications-logs)
  * [New First-Party Application Logs](#new-first-party-application-logs)
* [OpenTelemetry Collector](#opentelemetry-collector)
* [Auto-Instrumenting Existing Logging](#auto-instrumenting-existing-logging)
* [Trace Context in Legacy Formats](#trace-context-in-legacy-formats)
-->

* [はじめに](#はじめに)
* [OpenTelemetry以外のソリューションの限界](#OpenTelemetry以外のソリューションの限界)
* [OpenTelemetryの解決方法](#opentelemetryの解決方法)
* [ログの相関関係](#ログの相関関係)
* [イベントとログ](#イベントとログ)
* [レガシーとモダンなログソース](#レガシーとモダンなログソース)
  * [システムログ](#システムログ)
  * [インフラストラクチャ・ログ](#インフラストラクチャ・ログ)
  * [サードパーティアプリケーションのログ](#サードパーティアプリケーションのログ)
  * [レガシーなファーストパーティ・アプリケーションのログ](#レガシーなファーストパーティ・アプリケーションのログ)
  * [新しいファーストパーティ・アプリケーションのログ](#新しいファーストパーティ・アプリケーションのログ)
* [OpenTelemetry Collector](#opentelemetry-collector)
* [既存のログ機能に自動で計装を追加](#既存のログ機能に自動で計装を追加)
* [レガシーフォーマットでのTrace Context](#レガシーフォーマットでのTrace-Context)

<!--
## Introduction
-->

## はじめに

<!--
Of all telemetry signals logs have probably the biggest legacy. Most programming
languages have built-in logging capabilities or well-known, widely used logging
libraries.
-->

全てのテレメトリシグナルの中で、ログはおそらく最大の遺産です。ほとんどのプログラミング言語には、ロギング機能が組み込まれているか、よく知られているロギングライブラリが広く使われています。

<!--
For metrics and traces OpenTelemetry takes the approach of a clean-sheet design,
specifies a new API and provides full implementations of this API in multiple
languages.
-->

メトリックとトレースについては、OpenTelemetryはクリーンシートデザインのアプローチをとり、新しいAPIを指定し、このAPIの完全な実装を複数の言語で提供しています。

<!--
Our approach with logs is somewhat different. For OpenTelemetry to be
successful in logging space we need to support existing legacy of logs and
logging libraries, while offering improvements and better integration with the
rest of observability world where possible.
-->

私たちのログに対するアプローチは多少異なります。OpenTelemetryがログの分野で成功するためには、既存のログのレガシーとログライブラリをサポートする必要があります。

<!--
This is in essence the philosophy behind OpenTelemetry's logs support. We
embrace existing logging solutions and make sure OpenTelemetry works nicely with
existing logging libraries, log collection and processing solutions.
-->

これが、OpenTelemetryのログサポートの理念です。私たちは、既存のロギングソリューションを受け入れ、OpenTelemetryが既存のロギングライブラリ、ログ収集、処理ソリューションとうまく連携するようにしています。

<!--
## Limitations of non-OpenTelemetry Solutions
-->

## OpenTelemetry以外のソリューションの限界

<!--
Unfortunately existing logging solutions are currently weakly integrated with
the rest of the observability signals. Logs typically have limited support in
tracing and monitoring tools in the form of links that use available and often
incomplete correlation information (such as the time and origin attributes).
This correlation may be fragile because attributes are often added to logs,
traces and metrics via different means (e.g. using different collection agents).
There is no standardized way to include the information about the origin and
source of logs (such as the application and the location/infrastructure where
the application runs) that is uniform with traces and metrics and allows all
telemetry data to be fully correlated in a precise and robust manner.
-->

残念ながら、既存のロギングソリューションは、残りの観測可能なシグナルとの統合が弱いのが現状です。一般的にログは、利用可能でしばしば不完全な相関情報(時間や起源の属性など)を使用するリンクの形で、トレースおよび監視ツールを限定的にサポートしています。属性はログ、トレース、メトリックごとに異なる手段(異なる収集エージェントを使用するなど)で追加されることが多いため、この相関関係は脆弱な場合があります。トレースやメトリックと統一され、すべてのテレメトリデータを正確かつ堅牢な方法で完全に相関させることができる、ログの起源およびソース(アプリケーションおよびアプリケーションが実行されている場所/インフラなど)に関する情報を含めるための標準的な方法はありません。


<!--
Similarly, logs have no standardized way to propagate and record the request
execution context. In distributed systems this often results in a disjoint set
of logs collected from different components of the system.
-->

同様に、ログにはリクエストの実行状況を伝達し、記録する標準的な方法がありません。分散システムでは、システムの異なるコンポーネントから収集されたログのセットがバラバラになることがよくあります。

<!--
This is how a typical non-OpenTelemetry observability collection pipeline looks
like today:
-->

現在、OpenTelemetry以外の典型的なオブザーバビリティ収集パイプラインはこのようになっています。

<!--
![Separate Collection Diagram](img/separate-collection.png)
-->

![分断されたログ収集方法のダイアグラム](img/separate-collection.png)

<!--
There are often different libraries and different collection agents, using
different protocols and data models, with telemetry data ending up in separate
backends that don't know how to work well together.
-->

異なるプロトコルやデータモデルを使用した異なるライブラリや異なる収集エージェントが存在することが多く、テレメトリデータは、うまく連携する方法を知らない別々のバックエンドに送られます。

<!--
## OpenTelemetry Solution
-->

## OpenTelemetryの解決方法

<!--
Distributed tracing introduced the notion of request context propagation.
-->

分散型トレーシングでは、リクエスト・コンテキストの伝搬という概念が導入されました。

<!--
Fundamentally, though, nothing prevents the logs to adopt the same context
propagation concepts. If the recorded logs contained request context identifiers
(such as trace and span ids or user-defined baggage) it would result
in much richer correlation between logs and traces, as well as correlation
between logs emitted by different components of a distributed system. This would
make logs significantly more valuable in distributed systems.
-->

基本的には、しかし、ログが同じコンテキスト伝搬のコンセプトを採用することを妨げるものではありません。もし、記録されたログにリクエスト・コンテキストの識別子(トレースやSpanのID、ユーザー定義のバゲージなど)が含まれていれば、ログとトレースの間の相関性がより豊かになり、また、分散システムの異なるコンポーネントから出力されたログの間の相関性も高まります。これにより、分散システムにおけるログの価値が大幅に向上します。

<!--
This is one of the promising evolutionary directions for observability tools.
Standardizing log correlation with traces and metrics, adding support for
distributed context propagation for logs, unification of source attribution of
logs, traces and metrics will increase the individual and combined value of
observability information for legacy and modern systems. This is the vision of
OpenTelemetry's collection of logs, traces and metrics:
-->

これは、オブザーバビリティツールの有望な進化の方向性の一つです。トレースやメトリックとログの相関関係を標準化し、ログの分散型コンテキスト伝搬のサポートを追加し、ログ、トレース、メトリックのソース属性を統一することで、レガシーシステムとモダンシステムのオブザーバビリティ情報の個々の価値と総合的な価値を高めることができます。これがOpenTelemetryのログ、トレース、メトリックのコレクションのビジョンです。

<!--
![Unified Collection Diagram](img/unified-collection.png)
-->

![統一された収集のダイアグラム](img/unified-collection.png)

<!--
We emit logs, traces and metrics in a way that is compliant with OpenTelemetry
data models, send the data through OpenTelemetry Collector, where it can be
enriched and processed in a uniform manner. For example, Collector can add to
all telemetry data coming from a Kubernetes Pod several attributes that describe
the pod and it can be done automatically using
[k8sprocessor](https://pkg.go.dev/github.com/open-telemetry/opentelemetry-collector-contrib/processor/k8sprocessor?tab=doc)
without the need for the Application to do anything special. Most importantly
such enrichment is completely uniform for all 3 signals. The Collector
guarantees that logs, traces and metrics have precisely the same attribute names
and values describing the Kubernetes Pod that they come from. This enables exact
and unambiguous correlation of the signals by the Pod in the backend.
-->

OpenTelemetryのデータモデルに準拠した方法でログ、トレース、メトリックを出力し、OpenTelemetry Collectorを介してデータを送り、そこで統一的な方法で情報を豊かにしたり処理を行うことができます。例えば、CollectorはKubernetes Podから送られてくるすべてのテレメトリデータに、Podを説明するいくつかの属性を追加することができます。これはアプリケーションが特別なことをしなくても、[k8sprocessor](https://pkg.go.dev/github.com/open-telemetry/opentelemetry-collector-contrib/processor/k8sprocessor?tab=doc)を使って自動的に行うことができます。最も重要なことは、このような情報の強化は、3つのシグナルすべてにおいて完全に統一されているということです。Collectorは、ログ、トレース、メトリックが、それらの元となるKubernetes Podを記述する正確に同じ属性名と値を持つことを保証します。これにより、バックエンドのポッドによるシグナルの正確で曖昧さのない相関が可能になります。

<!--
For traces and metrics OpenTelemetry defines a new API that application
developers must use to emit traces and metrics.
-->

トレースやメトリックについて OpenTelemetryは、アプリケーション開発者がトレースやメトリックを発行するために使用しなければならない新しいAPIを定義しています。

<!--
For logs we did not take the same path. We realized that there is a much bigger
and more diverse legacy in logging space. There are many existing logging
libraries in different languages, each having their own API. Many programming
languages have established standards for using particular logging libraries. For
example in Java world there are several highly popular and widely used logging
libraries, such as Log4j or Logback.
-->

ログについては、同じ道を辿りませんでした。私たちは、ログの分野には、より大きく、より多様な遺産があることに気づきました。異なる言語で多くの既存のロギング・ライブラリがあり、それぞれが独自のAPIを持っています。多くのプログラミング言語は、特定のロギングライブラリを使用するための標準を確立しています。例えば、Javaの世界では、Log4jまたはLogbackのようないくつかの非常に人気があり、広く使用されるロギング・ライブラリがあります。

<!--
There are also countless existing prebuilt applications or systems that emit
logs in certain formats. Operators of such applications have no or limited
control on how the logs are emitted. OpenTelemetry needs to support these logs.
-->

また、特定のフォーマットでログを出力する既存のアプリケーションやシステムも無数にあります。このようなアプリケーションは、ログをどのように出力するかについては全く、あるいは限定的にしか制御できません。OpenTelemetryはこれらのログをサポートする必要があります。

<!--
Given the above state of the logging space we took the following approach:
-->

以上のようなログ分野の状況を踏まえて、私たちは次のようなアプローチをとりました。

<!--
- OpenTelemetry defines a [log data model](data-model.md). The purpose of the
  data model is to have a common understanding of what a log record is, what
  data needs to be recorded, transferred, stored and interpreted by a logging
  system.
-->

- OpenTelemetryでは、[log データモデル](data-model.md)を定義しています。データモデルの目的は、ログ記録とは何か、どのようなデータが記録され、転送され、保存され、ロギングシステムによって解釈される必要があるかについて、共通の理解を持つことです。

<!--
- Newly designed logging systems are expected to emit logs according to
  OpenTelemetry's log data model. More on this [later](#new-first-party-application-logs).
-->

- 新しく設計されたログシステムは、OpenTelemetryのログデータモデルに従ってログを出力することが期待されます。これについては[後述](#new-first-party-application-logs)を参照してください。

<!--
- Existing log formats can be
  [unambiguously mapped](data-model.md#appendix-a-example-mappings) to
  OpenTelemetry log data model. OpenTelemetry Collector can read such logs and
  translate them to OpenTelemetry log data model.
-->

- 既存のログフォーマットは、OpenTelemetryのログデータモデルに対して[曖昧さがなくマッピング](data-model.md#appendix-a-example-mappings)できます。OpenTelemetry Collectorはそのようなログを読み、OpenTelemetryログデータモデルに変換することができます。

<!--
- Existing applications or logging libraries can be modified to emit logs
  according to OpenTelemetry log data model. OpenTelemetry does not define a new
  logging API that application developers are expected to call. Instead we opt
  to make it easy to continue using the common logging libraries that already
  exist. OpenTelemetry provides guidance on how applications or logging
  libraries can be modified to become OpenTelemetry-compliant (link TBD). We
  also provide SDKs for some languages (link TBD) that make it easy to modify
  the existing logging libraries so that they emit OpenTelemetry-compliant logs.
-->

- 既存のアプリケーションやロギングライブラリを変更して、OpenTelemetryのログデータモデルに従ってログを出力することができます。OpenTelemetryは、アプリケーション開発者が呼び出すことを期待される新しいロギングAPIを定義しません。その代わりに、すでに存在する一般的なロギングライブラリを簡単に使い続けられるようにしています。OpenTelemetryは、アプリケーションやロギングライブラリをどのように修正してOpenTelemetryに準拠させるかについてのガイダンスを提供しています(リンク未定)。また、いくつかの言語用のSDK(リンク未定)を提供しています。これにより、既存のロギング・ライブラリを簡単に変更して、OpenTelemetry準拠のログを出力できるようになります。

<!--
This approach allows OpenTelemetry to read existing system and application logs,
provides a way for newly built application to emit rich, structured,
OpenTelemetry-compliant logs, and ensures that all logs are eventually
represented according to a uniform log data model on which the backends can
operate.
-->

このアプローチにより、OpenTelemetryは既存のシステムやアプリケーションのログを読み取ることができ、新しく構築されたアプリケーションがリッチで構造化されたOpenTelemetry準拠のログを出力する方法を提供します。また、すべてのログは最終的にバックエンドが操作できる統一されたログデータモデルに従って表現されます。

<!--
In the future OpenTelemetry may define a new logging API and provide
implementations for various languages (like we currently do for traces and
metrics), but it is not an immediate priority.
-->

将来的には、OpenTelemetryは(現在、トレースやメトリックのために行っているように)新しいロギングAPIを定義し、様々な言語のための実装を提供するかもしれませんが、それはすぐには優先されません。

<!--
Later in this document we will discuss in more details
[how various log sources are handled](#legacy-and-modern-log-sources) by
OpenTelemetry, but first we need to describe in more details an important
concept: the log correlation.
-->

このドキュメントの後半では、OpenTelemetryで[様々なログソースがどのように処理されるか](#レガシーとモダンなログソース)について詳しく説明しますが、その前に、重要なコンセプトであるログの相関関係について詳しく説明する必要があります。

<!--
## Log Correlation
-->

## ログの相関関係

<!--
Logs can be correlated with the rest of observability data in a few dimensions:
-->

ログは他のオブザーバビリティ・データといくつかの次元で相関しています:

<!--
- By the **time of execution**. Logs, traces and metrics can record the moment
  of time or the range of time the execution took place. This is the most basic
  form of correlation.
-->

- **実行された時間**によって。ログ、トレース、メトリックは、実行が行われた瞬間や時間の範囲を記録することができます。これは相関関係の最も基本的な形です。

<!--
- By the **execution context**, also known as the request context. It is a
  standard practice to record the execution context (trace and span ids as well
  as user-defined context) in the spans. OpenTelemetry extends this practice to
  logs where possible by including [TraceId](data-model.md#field-traceid) and
  [SpanId](data-model.md#field-spanid) in the log records. This allows to
  directly correlate logs and traces that correspond to the same execution
  context. It also allows to correlate logs from different components of a
  distributed system that participated in the particular request execution.
-->

- リクエスト・コンテキストとしても知られる**実行コンテキスト**によって。実行コンテキスト(トレースとSpanのID、およびユーザー定義のコンテキスト)をSpanに記録するのは標準的な慣習です。OpenTelemetry は、[TraceId](data-model.md#field-traceid) と [SpanId](data-model.md#field-spanid) をログレコードに含めることで、この慣習を可能な限りログに拡張しています。これにより、同じ実行コンテキストに対応するログとトレースを直接相関させることができます。また、特定のリクエストの実行に参加した分散システムの異なるコンポーネントからのログを相関させることもできます。

<!--
- By the **origin of the telemetry**, also known as the Resource context.
  OpenTelemetry traces and metrics contain information about the Resource they
  come from. We extend this practice to logs by including the
  [Resource](data-model.md#field-resource) in log records.
-->

- リソース・コンテキストとしても知られる、テレメトリの**オリジン**によって。OpenTelemetryのトレースとメトリックは、それらの元となるResourceに関する情報を含んでいます。私たちは、ログレコードに[Resource](data-model.md#field-resource)を含めることで、この慣習をログにも適用しています。

<!--
These 3 correlations can be the foundation of powerful navigational, filtering,
querying and analytical capabilities. OpenTelemetry aims to record and collects
logs in a manner that enables such correlations.
-->

これらの3つの相関関係は、強力なナビゲーション、フィルタリング、クエリー、分析機能の基礎となります。OpenTelemetryは、このような相関関係を可能にする方法でログを記録・収集することを目指しています。

<!--
## Events and Logs
-->

## イベントとログ

<!--
Wikipedia’s [definition of log file](https://en.wikipedia.org/wiki/Log_file):
-->

ウィキペディアの[ログファイルの定義](https://en.wikipedia.org/wiki/Log_file)は以下のとおりです。:

<!--
>In computing, a log file is a file that records either events that occur in an
>operating system or other software runs.
The notion of a log record used throughout OpenTelemetry is aligned with
Wikipedia’s definition. We claim that in the observability realm there is no
important distinction between log records and recorded events from a data
modeling perspective.
-->

>コンピュータにおいて、ログファイルとは、オペレーティングシステムや他のソフトウェアの実行中に発生したイベントを記録するファイルのことです。

OpenTelemetryで使用されているログ・レコードの概念は、Wikipediaの定義と一致しています。観測性の領域では、データモデリングの観点から、ログレコードと記録されたイベントの間に重要な区別はないと主張しています。

<!--
From OpenTelemetry's perspective Log Records and Events are different names for
the same concept.
-->

OpenTelemetryの視点では、ログレコードとイベントは同じコンセプトの異なる名称です。

<!--
Some products may want to make a distinction between Events collected from
certain sources and Logs collected from other sources. OpenTelemetry believes
that there is nothing inherently different between log records and events from
data modeling perspective, the differences are in the sources themselves. Thus
where it matters the products should make that distinction based on the source
of the data rather than attempt to arbitrarily categorize the data as events vs
logs.
-->

製品によっては、特定のソースから収集したイベントと、他のソースから収集したログを区別したい場合があります。OpenTelemetryは、データモデリングの観点からは、ログレコードとイベントの間に本質的な違いはなく、違いはソース自体にあると考えています。したがって、問題となる場所では、製品はデータをイベントとログの間で恣意的に分類しようとするのではなく、データのソースに基づいて区別すべきです。

<!--
## Legacy and Modern Log Sources
-->

## レガシーとモダンなログソース

<!--
It is important to distinguish several sorts of legacy and modern log sources.
Firstly, this directly affects how exactly we get access to these logs and how
we collect them. Secondly, we have varying levels of control over how these logs
are generated and whether we can amend the information that can be included in
the logs.
-->

いくつかの種類のレガシーログソースとモダンログソースを区別することが重要です。第一に、これはログへのアクセス方法やログの収集方法に直接影響します。第二に、ログの生成方法や、ログに含まれる情報を修正できるかどうかについて、様々なレベルでコントロールすることができます。

<!--
Below we list several categories of logs and describe what can be possibly done
for each category to have better experience in the observability solutions.
-->

以下では、いくつかのカテゴリーのログを挙げ、それぞれのカテゴリーについて、オブザーバビリティ・ソリューションでより良い経験をするために何ができるかを説明します。

<!--
### System Logs
-->

### システムログ

<!--
These are logs generated by the operating system and over which we have no
control. We cannot change the format or affect what information is included.
Examples of system format are Syslog and Windows Event Logs.
-->

これらはオペレーティングシステムによって生成されたログであり、コントロールすることはできません。フォーマットを変更したり、どのような情報が含まれるかに影響を与えることはできません。システムフォーマットの例としては、SyslogやWindowsイベントログなどがあります。

<!--
System logs are written at the host level (which may be physical, virtual or
containerized) and have a predefined format and content (note that applications
may also be able to write records to standard system logs: this case is covered
below in the [Third-Party Applications](#third-party-application-logs) section).
-->

システムログはホストレベル(物理的、仮想的、またはコンテナ化されたもの)で書き込まれ、あらかじめ定義されたフォーマットと内容を持っています(アプリケーションが標準のシステムログに記録を書き込める場合もあることに注意してください:このケースについては後述の[サードパーティアプリケーション](#サードパーティアプリケーションのログ)のセクションで説明します)。

<!--
System operations recorded in the logs can be a result of a request execution.
However system logs either do not include any data about the request context or
if included it is highly idiosyncratic and thus difficult to identify, parse and
use. This makes it nearly impossible to perform request context correlation for
system logs. However we can and should automatically enrich system logs with the
resource context - the information about the host that is available during
collection. This can include the host name, IP address, container or pod name,
etc. This information should be added to the Resource field of collected log
data.
-->

ログに記録されるシステム操作は、リクエスト実行の結果である可能性があります。しかし、システムログにはリクエスト・コンテキストに関するデータが含まれていないか、含まれていたとしても非常に特異なものであるため、識別、解析、利用が困難です。このため、システムログのリクエスト・コンテキストの相関関係を作ることはほぼ不可能です(XXX: perform request context correlationとは相関関係を作ること？) 。しかし、システムログにリソース・コンテキスト(収集時に利用可能なホストに関する情報)を自動的に追加することはできますし、そうすべきです。これには、ホスト名、IPアドレス、コンテナやポッドの名前などが含まれます。この情報は、収集したログデータのResourceフィールドに追加する必要があります。

<!--
OpenTelemetry Collector can read system logs (link TBD) and automatically enrich
them with Resource information using the
[resourcedetection](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/master/processor/resourcedetectionprocessor)
processor.
-->

OpenTelemetry Collectorは、システムログ(リンク未定)を読み、[resourcedetection](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/master/processor/resourcedetectionprocessor)プロセッサを使ってResource情報で自動的に情報を追加できます。

<!--
### Infrastructure Logs
-->

### インフラストラクチャ・ログ

<!--
These are logs generated by various infrastructure components, such as
Kubernetes events (if you are wondering why events are discussed in the context
of logs see [Events and Logs](#events-and-logs)). Like system logs, the
infrastructure logs lack a request context and can be enriched by the resource
context - information about the node, pod, container, etc.
-->

インフラストラクチャ・ログは、Kubernetesのイベントなど、さまざまなインフラストラクチャコンポーネントによって生成されるログです(なぜイベントがログの文脈で議論されるのか不思議に思われる方は、[イベントとログ](#イベントとログ)をご覧ください)。システムログと同様に、インフラログにはリクエスト・コンテキストがなく、リソース・コンテキスト(ノード、ポッド、コンテナなどに関する情報)によって充実させることができます。

<!--
OpenTelemetry Collector or other agents can be used to query logs from most
common infrastructure controllers.
-->

OpenTelemetry Collectorやその他のエージェントを使用して、一般的なインフラストラクチャコントローラのログを照会することができます。

<!--
### Third-party Application Logs
-->

### サードパーティアプリケーションのログ

<!--
Applications typically write logs to standard output, to files or other
specialized medium (e.g. Windows Event Logs for applications). These logs can be
in many different formats, spanning a spectrum along these variations:
-->

アプリケーションは通常、標準出力やファイル、その他の特殊な媒体にログを書き込みます(例:アプリケーションのWindowsイベントログ)。これらのログは、以下のような様々なフォーマットで出力されます。

<!--
- Free-form text formats with no easily automatable and reliable way to parse
  structured data from them.
-->

- 自由形式のテキストフォーマットで、構造化されたデータを簡単に自動化できる信頼性の高い方法がないもの。

<!--
- Better specified and sometimes customizable formats that can be parsed to
  extract structured data (such as Apache logs or RFC5424 Syslog).
-->

- 構造化されたデータ(ApacheログやRFC5424 Syslogなど)を抽出するために解析することができる、より指定された、時にはカスタマイズ可能なフォーマット。

<!--
- Formally structured formats (e.g. JSON files with well-defined schema or
  Windows Event Log).
-->

- 形式的に構造化されたフォーマット(明確に定義されたスキーマを持つJSONファイルやWindowsイベントログなど)。

<!--
The collection system needs to be able to discover most commonly used
applications and have parsers that can convert these logs into a structured
format. Like system and infrastructure logs, application logs often lack request
context but can be enriched by resource context, including the attributes that
describe the host and infrastructure as well as application-level attributes
(such as the application name, version, name of the database - if it is a DBMS,
etc).
-->

収集システムは、最も一般的に使用されているアプリケーションを発見でき、これらのログを構造化されたフォーマットに変換できるパーサーを備えている必要があります。システムログやインフラストラクチャ・ログと同様に、アプリケーションログはリクエストのコンテキストを欠いていることが多いのですが、アプリケーションレベルの属性(アプリケーション名、バージョン、データベース名(DBMSの場合)など)だけでなく、ホストやインフラを説明する属性を含むリソースコンテキストによって情報を充実させることができます。

<!--
OpenTelemetry recommends to collect application logs using Collector's
[filelog receiver](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/receiver/filelogreceiver).
Alternatively, another log collection agent, such as FluentBit, can collect
logs,
[then send](https://github.com/open-telemetry/opentelemetry-collector/tree/master/receiver/fluentforwardreceiver)
to OpenTelemetry Collector where the logs can be further processed and enriched.
-->

OpenTelemetryはCollectorの[filelog receiver](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/receiver/filelogreceiver)を使用してアプリケーションログを収集することを推奨します。

あるいは、FluentBitのような別のログ収集エージェントがログを収集し、OpenTelemetry Collectorに[送信](https://github.com/open-telemetry/opentelemetry-collector/tree/master/receiver/fluentforwardreceiver)して、そこでログをさらに処理して情報を充実させることもできます。

<!--
### Legacy First-Party Applications Logs
-->

### レガシーなファーストパーティ・アプリケーションのログ

<!--
These are applications that are created in-house. People tasked with setting up
log collection infrastructure sometimes are able to modify these applications to
alter how logs are written and what information is included in the logs. For
example, the application’s log formatters may be reconfigured to output json
instead of plain text and by doing so help improve the reliability of log
collection.
-->

ファーストパーティ・アプリケーションとは、社内で作成されたアプリケーションです。ログ収集のインフラを整える仕事をしている人は、これらのアプリケーションを修正して、ログの書き方やログに含まれる情報を変更できることがあります。例えば、アプリケーションのログフォーマッタを再設定して、プレーンテキストの代わりにjsonを出力することで、ログ収集の信頼性を向上させることができます。

<!--
More significant modifications to these applications can be done manually by
their developers, such as addition of the request context to every log
statement, however this is likely going to be vanishingly rare due to the effort
required.
-->

これらのアプリケーションのより重要な変更は、すべてのログステートメントにリクエスト・コンテキストを追加するなど、開発者が手動で行うことができますが、労力が必要なため実際に行われることはほとんどありません。


<!--
As opposed to manual efforts we have an interesting opportunity to "upgrade"
application logs in a less laborious way by providing full or semi
auto-instrumenting solutions that modify the logging libraries used by the
application to automatically output the request context such as the trace id or
span id with every log statement. The request context can be automatically
extracted from incoming requests if standard compliant request propagation is
used, e.g. via [W3C TraceContext](https://w3c.github.io/trace-context/). In
addition, the requests outgoing from the application may be injected with the
same request context data, thus resulting in context propagation through the
application and creating an opportunity to have full request context in logs
collected from all applications that can be instrumented in this manner.
-->

手動での作業とは対照的に、アプリケーションが使用するロギング・ライブラリを変更して、トレースIDやSpanIDなどのリクエスト・コンテキストをすべてのログ・ステートメントで自動的に出力する完全または半自動テレメトリ・ソリューションを提供することで、より手間のかからない方法でアプリケーション・ログを「アップグレード」するという興味深い機会があります。例えば、[W3C TraceContext](https://w3c.github.io/trace-context/)を経由して、標準に準拠したリクエスト伝搬が使用されている場合、リクエスト・コンテキストは、入ってくるリクエストから自動的に抽出することができます。さらに、アプリケーションから発信されるリクエストには、同じリクエスト・コンテキスト・データが注入されるかもしれません。その結果、アプリケーションを通じてコンテキストが伝搬され、この方法で計装できるすべてのアプリケーションから収集されたログに完全なリクエスト・コンテキストを持つ機会が生まれます。

<!--
Some logging libraries are designed to be extended in this manner relatively
easily. There is no need to actually modify the libraries, instead we can
implement "appender" or "exporter" components for such libraries and implement
the additional log record enrichment in these components.
-->

いくつかのロギングライブラリは、比較的簡単にこの方法で拡張できるように設計されています。実際にライブラリを変更する必要はありませんが、代わりにそのようなライブラリのための「appender」または「exporter」コンポーネントを実装し、これらのコンポーネントに追加のログ・レコードの充実化を実装することができます。

<!--
There are typically 2 ways to collect logs from these applications.
-->

これらのアプリケーションからログを収集するには、通常2つの方法があります。

<!--
#### Via File or Stdout Logs
-->

#### ファイル介して、あるいは標準出力ログを介して

<!--
The first approach, assuming the logs are written to files or to standard
output, requires ability to read file logs, tail then, work correctly when log
rotation is used, optionally also parse the logs to convert them into more
structured formats. Parings requires support for different parser types, which
can also be configured to parse custom formats as well as ability to add custom
parsers. Examples of common formats that parsers need to support are: CSV,
Common Log Format, Labeled Tab-separated Values (LTSV), Key/Value Pair format,
JSON, etc. To support this approach OpenTelemetry recommends to collect logs
using OpenTelemetry
[Collector](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/receiver/filelogreceiver).
-->

最初のアプローチでは、ログがファイルや標準出力に書き込まれていると仮定して、ファイルのログを読み込んで尾行し、ログローテーションが使用されている場合には正しく動作し、オプションでログを解析してより構造化されたフォーマットに変換する機能が必要となります。Parings(XXX:なんだろうこれは)では、さまざまな種類のパーサーをサポートし、カスタムフォーマットを解析するように設定したり、カスタムパーサーを追加したりすることができます。パーサーがサポートする必要のある一般的なフォーマットの例は以下のとおりです。CSV、Common Log Format、Labeled Tab-separated Values(LTSV)、Key/Value Pairフォーマット、JSONなどです。このアプローチをサポートするために、OpenTelemetryはOpenTelemetry [Collector](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/receiver/filelogreceiver)を使ってログを収集することを推奨します。

<!--
![Application to File Logs](img/app-to-file-logs-fb.png)
-->

![アプリケーションからファイルログ](img/app-to-file-logs-fb.png)

<!--
Alternatively, if the Collector does not have the necessary file reading and
parsing capabilities, another log collection agent, such as FluentBit can
collect the logs,
[then send the logs](https://github.com/open-telemetry/opentelemetry-collector/tree/master/receiver/fluentforwardreceiver)
to OpenTelemetry Collector.
-->

あるいは、Collectorが必要なファイルの読み取りと解析の機能を持っていない場合は、FluentBitなどの別のログ収集エージェントがログを収集し、OpenTelemetry Collectorにログを送信することもできます(https://github.com/open-telemetry/opentelemetry-collector/tree/master/receiver/fluentforwardreceiver)。


<!--
The benefit of this approach is that how logs are produced and where they are
written by the application requires no or minimal changes. The downside is that
it requires the often non-trivial log file reading and parsing functionality.
Parsing may also be not reliable if the output format is not well-defined.
-->

この方法の利点は、アプリケーションがログをどのように生成し、どこに書き込むかについて、変更が不要または最小限で済むことです。デメリットは、ログファイルの読み込みと解析の機能が必要になることです。また、出力フォーマットが明確に定義されていない場合、解析の信頼性が低くなる可能性があります。

<!--
#### Direct to Collector
-->

#### Collectorに直接

<!--
The second approach is to modify the application so that the logs are output via
a network protocol, e.g. via
[OTLP](https://github.com/open-telemetry/opentelemetry-proto/blob/master/opentelemetry/proto/logs/v1/logs.proto).
The most convenient way to achieve this is to provide addons or extensions to
the commonly used logging libraries. The addons implement sending over such
network protocols, which would then typically require small, localized changes
to the application code to change the logging target.
-->

2つ目の方法は、アプリケーションを変更して、ログがネットワーク・プロトコル、例えば[OTLP](https://github.com/open-telemetry/opentelemetry-proto/blob/master/opentelemetry/proto/logs/v1/logs.proto)を介して出力されるようにすることです。これを実現する最も便利な方法は、一般的に使用されているロギングライブラリのアドオンや拡張機能を提供することです。アドオンは、そのようなネットワーク・プロトコルを介した送信を実装しており、その場合、ロギング・ターゲットを変更するためには、通常、アプリケーション・コードに小さな局所的変更が必要となります。

<!--
![Application to Collector](img/app-to-otelcol.png)
-->

![アプリケーションからCollector](img/app-to-otelcol.png)

<!--
The application logs will be also enriched by the resource context, similarly to
how it is done for third-party applications and so will potentially have full
correlation information across all context dimensions.
-->

アプリケーションのログは、サードパーティのアプリケーションの場合と同様に、リソース・コンテキストによって強化されるため、すべてのコンテキストの次元にわたって完全な相関情報を持つ可能性があります。

<!--
The downside of this approach is that the simplicity of having the logs in a
local file is lost (e.g. ability to easily inspect the log file locally) and
requires a full buy-in in OpenTelemetry's logging approach. This approach also
only works if the destination that the logs need to be delivered is able to
receive logs via the network protocol that OpenTelemetry can send in.
-->

このアプローチの欠点は、ログをローカルファイルに保存することのシンプルさが失われることであり(例:ログファイルをローカルで簡単に検査することができる)、OpenTelemetryのログ取得アプローチに全面的な賛同を得る必要があります。また、この方法は、ログの配信先がOpenTelemetryが送信できるネットワークプロトコルでログを受信できる場合にのみ機能します。

<!--
The benefits of this approach is that it emits the logs in well-defined, formal,
highly structured format, removes all complexity associated with file logs, such
as parsers, log tailing and rotation. It also enables the possibility to send
logs directly to the logging backend without using a log collection agent.
-->

このアプローチの利点は、明確に定義された正式な高度に構造化されたフォーマットでログを出力し、パーサー、ログテーリング、ローテーションなど、ファイルログに関連する複雑さをすべて取り除くことができることです。また、ログ収集エージェントを使用せずに、ログを直接ログバックエンドに送信することも可能です。

<!--
To facilitate both approaches described above OpenTelemetry provides SDKs, which
can be used together with existing logging libraries and which automatically
inject the request context in the emitted logs and provide an easy way to send
the logs via OTLP. These SDKs do not require application developers to modify
each logging statement in the source code and instead require the developer to
enable the OpenTelemetry SDK's logging support at the application startup. After
that the SDK intercepts all emitted logs and modifies the emitting behavior as
configured.
-->

この両方のアプローチを促進するために、OpenTelemetryは、既存のロギング・ライブラリと一緒に使用できるSDKを提供しています。このSDKは、出力されるログにリクエスト・コンテキストを自動的に注入し、OTLP経由でログを送信する簡単な方法を提供します。これらのSDKは、アプリケーション開発者がソースコード内の各ロギングステートメントを変更する必要はなく、代わりに開発者がアプリケーションの起動時にOpenTelemetry SDKのロギングサポートを有効にする必要があります。その後、SDKは送信されたすべてのログを傍受し、設定に応じて送信動作を変更します。

<!--
### New First-Party Application Logs
-->

### 新しいファーストパーティ・アプリケーションのログ

<!--
These are greenfield developments. OpenTelemetry provides recommendations and
best practices about how to emit logs (along with traces and metrics) from these
applications. For applicable languages and frameworks the auto-instrumentation
or simple configuration of a logging library to use an OpenTelemetry appender or
extension will still be the easiest way to emit context-enriched logs. As
already described earlier we provide extensions to some popular logging
libraries languages to support the manual instrumentation cases. The extensions
will support the inclusion of the request context in the logs and allow to send
logs using OTLP protocol to the backend or to the Collector, bypassing the need
to have the logs represented as text files. Emitted logs are automatically
augmented by application-specific resource context (e.g. process id, programming
language, logging library name and version, etc). Full correlation across all
context dimensions will be available for these logs.
-->

これらはグリーンフィールド開発(訳注: スクラッチからの開発)です。OpenTelemetryは、これらのアプリケーションからログを(トレースやメトリックスと一緒に)出力する方法について、推奨事項やベストプラクティスを提供しています。該当する言語やフレームワークでは、自動計測や、OpenTelemetryのAppenderや拡張機能を使用するためのロギングライブラリの簡単な設定が、コンテキストに富んだログを出力する最も簡単な方法となります。すでに説明したように、私たちはいくつかの人気のあるロギングライブラリ言語に、手動での計測ケースをサポートするための拡張機能を提供しています。この拡張機能は、ログにリクエスト・コンテキストを含めることをサポートし、OTLPプロトコルを使用してログをバックエンドやコレクターに送信することができ、ログをテキストファイルとして表現する必要がありません。送信されたログは、アプリケーション固有のリソースコンテキスト(プロセスID、プログラミング言語、ロギングライブラリの名前とバージョンなど)によって自動的に拡張されます。これらのログでは、すべてのコンテキスト次元にわたる完全な相関関係が得られます。

<!--
As noted earlier OpenTelemetry does not currently define a new logging API or
create new user-facing logging libraries. Our initial goal is to enhance
existing popular logging libraries as needed. This is how a typical new
application uses OpenTelemetry API, SDK and the existing log libraries:
-->

先に述べたように、OpenTelemetryは現在のところ、新しいロギングAPIを定義したり、ユーザー向けの新しいロギングライブラリを作成したりしません。私たちの最初の目標は、必要に応じて既存の一般的なログライブラリを強化することです。典型的な新しいアプリケーションが、OpenTelemetry API、SDK、既存のログライブラリーをどのように使用するかを説明します。

<!--
![Application, API, SDK Diagram](img/application-api-sdk.png)
-->

![アプリケーション、API、SDKダイアグラム](img/application-api-sdk.png)

<!--
## OpenTelemetry Collector
-->

## OpenTelemetry Collector

<!--
To enable log collection according to this specification we use OpenTelemetry
Collector.
-->

この仕様に従ったログ収集を可能にするために、OpenTelemetry Collectorを使用します。

<!--
The following functionality exists to enable log collection:
-->

ログ収集を可能にするために以下の機能があります。

<!--
- Support for log data type and log pipelines based on the
  [log data model](data-model.md). This includes processors such as
  [attributesprocessor](https://github.com/open-telemetry/opentelemetry-collector/tree/main/processor/attributesprocessor)
  that can operate on log data.
-->

- [ログデータモデル](data-model.md)に基づくログデータタイプとログパイプラインのサポート。これには、ログデータを操作できる[attributesprocessor](https://github.com/open-telemetry/opentelemetry-collector/tree/main/processor/attributesprocessor)などのプロセッサが含まれます。

<!--
- Ability to read logs from text files, tail the files, understand common log
  rotation schemes, watch directories for log file creation, ability to
  checkpoint file positions and resume reading from checkpoints. This ability is
  implemented by using Collector's
  [filelog receiver](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/receiver/filelogreceiver)
  or using an externally running agent (such as FluentBit).
-->

- テキストファイルからのログ読み込み、ファイルの更新検知処理、一般的なログローテーション方式の理解、ログファイル作成のためのディレクトリの監視、ファイル位置のチェックポイント、チェックポイントからの読み込み再開などの機能。これら機能は、Collectorの[filelog receiver](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/receiver/filelogreceiver)を使用するか、外部で動作するエージェント(FluentBitなど)を使用して実装されています。

<!--
- Ability to parse logs in common text formats and to allow end users to
  customize parsing formats and add custom parsers as needed. Collector's
  [parsers](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/receiver/filelogreceiver#operators)
  or parsing in the external agent is used for this.
-->

- 一般的なテキスト形式でログを解析し、エンドユーザーが必要に応じてパース形式をカスタマイズしたり、カスタムパーサーを追加したりできること。コレクターの[パーサー](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/receiver/filelogreceiver#operators)や外部エージェントでのパーシングがこれに使われます。

<!--
- Ability to send logs via common network protocols for logs, such as Syslog, or
  vendor-specific log formats. Collector contains exporters that directly
  implement this ability.
-->

- Syslog などのログ用の一般的なネットワークプロトコル、またはベンダー固有のログフォーマットでログを送信する機能。Collectorには、この機能を直接実装するExporterが含まれています。

<!--
## Auto-Instrumenting Existing Logging
-->

## 既存のログ機能に自動で計装を追加

<!--
We can provide auto-instrumentation for most popular logging libraries. The
auto-instrumented logging statements will do the following:
-->

私たちは、ほとんどの一般的なロギングライブラリに対する自動計装を提供することができます。自動計装されたログに関する機能は以下のようになります。

<!--
- Read incoming request context (this is part of broader instrumentation that
  auto-instrumenting libraries perform).
-->

- 受信したリクエストのコンテキストを読み取る(これは、自動計装ライブラリが実行する広範な計装の一部です)。

<!--
- Configure logging libraries to use trace id and span id fields from request
  context as logging context and automatically include them in all logged
  statements.
-->

- リクエスト・コンテキストのトレースIDとSpanIDフィールドをロギング・コンテキストとして使用し、自動的にすべてのロギングされたステートメントにそれらを含めるように、ロギング・ライブラリを構成します。

<!--
This is possible to do for certain languages (e.g. in Java) and we can reuse
[existing open-source libraries](https://docs.datadoghq.com/tracing/connect_logs_and_traces/java/?tab=log4j2)
that do this.
-->

これは、特定の言語(例えばJava)では可能であり、これを実現している[既存のオープンソース・ライブラリ](https://docs.datadoghq.com/tracing/connect_logs_and_traces/java/?tab=log4j2)を再利用することができます。

<!--
A further optional modification would be to auto-instrument loggers to send logs
directly to the backend via OTLP instead or in addition to writing to a file or
standard output.
-->

さらにオプションとして、機器のロガーがファイルや標準出力への書き込みに代えて、OTLPを介してバックエンドに直接ログを送信するように変更することもできます。

<!--
## Trace Context in Legacy Formats
-->

## レガシーフォーマットでのTrace Context

<!--
Earlier we briefly mentioned that it is possible to modify existing applications
so that they include the Request Context information in the emitted logs.
-->

先ほど、既存のアプリケーションを改造して、リクエスト・コンテキストの情報をログに含めることができると簡単に説明しました。

<!--
[OTEP0114](https://github.com/open-telemetry/oteps/pull/114) defines how the
trace context should be recorded in logs. To summarize, the following field
names should be used in legacy formats:
-->

[OTEP0114](https://github.com/open-telemetry/oteps/pull/114)では、トレースコンテキストをログにどのように記録すべきかを定義しています。要約すると、レガシーフォーマットでは以下のフィールド名を使用する必要があります。

<!--
- "trace_id" for [TraceId](data-model.md#field-traceid), hex-encoded.
- "span_id" for [SpanId](data-model.md#field-spanid), hex-encoded.
- "trace_flags" for [trace flags](data-model.md#field-traceflags), formatted
  according to W3C traceflags format.
-->

- "trace_id" [TraceId](data-model.md#field-traceid) のことで、16 進数で表現されます。
- "span_id" は [SpanId](data-model.md#field-spanid) のことで、16 進数で表現されます。
- "trace_flags" は [trace flags](data-model.md#field-traceflags) のことで、W3C traceflags フォーマットにしたがって表現されます。

<!--
All 3 fields are optional (see the [data model](data-model.md) for details of
which combination of fields is considered valid).
-->

3つのフィールドはすべて任意です(どの組み合わせが有効かについては、[データモデル](data-model.md)を参照してください)。

<!--
### Syslog RFC5424
-->

### Syslog RFC5424

<!--
Trace id, span id and traceflags SHOULD be recorded via SD-ID "opentelemetry".
-->

トレースID、SpanID、およびトレースフラグは、SD-ID「opentelemetry」を介して記録されるべきです(SHOULD)。

<!--
For example:
-->

例:

<!--
```
[opentelemetry trace_id="102981ABCD2901" span_id="abcdef1010" trace_flags="01"]
```
-->

```
[opentelemetry trace_id="102981ABCD2901" span_id="abcdef1010" trace_flags="01"]
```

<!--
### Plain Text Formats
-->

### プレーンテキスト形式

<!--
The fields should be recorded according to the customary approach used for a
particular format (e.g. field:value format for LTSV). For example:
-->

フィールドは、特定のフォーマットに使用されている慣習的な方法(例:LTSVのフィールド:値のフォーマット)に従って記録する必要があります。例えば、以下のようになります。

<!--
```
host:192.168.0.1<TAB>trace_id:102981ABCD2901<TAB>span_id:abcdef1010<TAB>time:[01/Jan/2010:10:11:23 -0400]<TAB>req:GET /health HTTP/1.0<TAB>status:200
```
-->

```
host:192.168.0.1<TAB>trace_id:102981ABCD2901<TAB>span_id:abcdef1010<TAB>time:[01/Jan/2010:10:11:23 -0400]<TAB>req:GET /health HTTP/1.0<TAB>status:200
```

<!--
### JSON Formats
-->

### JSON 形式

<!--
The fields should be recorded as top-level fields in the JSON structure. For example:
-->

フィールドは、JSON構造のトップレベルのフィールドとして記録する必要があります。例えば、以下のようになります。

<!--
```json
{
  "timestamp":1581385157.14429,
  "body":"Incoming request",
  "trace_id":"102981ABCD2901",
  "span_id":"abcdef1010"
}
```
-->

```json
{
  "timestamp":1581385157.14429,
  "body":"Incoming request",
  "trace_id":"102981ABCD2901",
  "span_id":"abcdef1010"
}

<!--
### 他の構造化形式
-->

### Other Structured Formats

<!--
The fields should be recorded as top-level structured attributes of the log
record as it is customary for the particular format.
-->

フィールドは、特定のフォーマットに慣例的に、ログレコードのトップレベルの構造化属性として記録されるべきです。

