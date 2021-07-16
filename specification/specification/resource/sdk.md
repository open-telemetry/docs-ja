<!--
# Resource SDK
-->

# リソース SDK

<!--
**Status**: [Stable](../document-status.md)
-->

**Status**: [Stable](../document-status.md)

<!--
A [Resource](../overview.md#resources) is an immutable representation of the entity producing
telemetry as [Attributes](../common/common.md#attributes).
For example, a process producing telemetry that is running in a
container on Kubernetes has a Pod name, it is in a namespace and possibly is
part of a Deployment which also has a name. All three of these attributes can be
included in the `Resource`. Note that there are certain
["standard attributes"](semantic_conventions/README.md) that have prescribed meanings.
-->

[リソース](../overview.md#resources)は、テレメトリを生成するエンティティを[属性](../common/common.md#attributes)として不変的に表現したものです。例えば、Kubernetes上のコンテナで実行されているテレメトリを生成するプロセスは、Pod名を持ち、Namespace内にあり、おそらく名前を持つDeploymentの一部です。これらの3つの属性はすべて`リソース`に含めることができます。なお、意味が決まっている["標準属性"](semantic_conventions/README.md)があります。

<!--
The primary purpose of resources as a first-class concept in the SDK is
decoupling of discovery of resource information from exporters. This allows for
independent development and easy customization for users that need to integrate
with closed source environments. The SDK MUST allow for creation of `Resources` and
for associating them with telemetry.
-->

リソースをSDKのファーストクラスのコンセプトとした主な目的は、リソース情報の発見をエクスポーターから切り離すことです。これにより、独立した開発が可能になり、クローズドソース環境との統合を必要とするユーザーのためのカスタマイズが容易になります。SDKは、`リソース`の作成と、それをテレメトリに関連付けることを可能にしなければなりません(MUST)。

<!--
When used with distributed tracing, a resource can be associated with the
[TracerProvider](../trace/api.md#tracerprovider) when the TracerProvider is created.
That association cannot be changed later.
When associated with a `TracerProvider`,
all `Span`s produced by any `Tracer` from the provider MUST be associated with this `Resource`.
-->

分散トレーシングで使用する場合、TracerProviderが作成された時点で、リソースを[`TracerProvider`](../trace/api.md#tracerprovider)に関連付けることができます。その関連付けは後から変更することはできません。`TracerProvider`と関連付けられた場合、プロバイダからのあらゆる`Tracer`によって生成されたすべての`Span`は、この`リソース`と関連付けられなければなりません(MUST)。

<!--
Analogous to distributed tracing, when used with metrics,
a resource can be associated with a `MeterProvider`.
When associated with a [`MeterProvider`](../metrics/api.md#meter-interface),
all metrics produced by any `Meter` from the provider will be
associated with this `Resource`.
-->

分散トレーシングと同様に、Metricsを使用する場合、リソースは `MeterProvider` と関連付けることができます。[`MeterProvider`](../metrics/api.md#meter-interface)に関連付けられると、そのプロバイダからの任意の`Meter`によって生成されたすべてのメトリックは、この`リソース`に関連付けられます。

<!--
## SDK-provided resource attributes
-->

## SDKが提供する リソース属性

<!--
The SDK MUST provide access to a Resource with at least the attributes listed at
[Semantic Attributes with SDK-provided Default Value](semantic_conventions/README.md#semantic-attributes-with-sdk-provided-default-value).
This resource MUST be associated with a `TracerProvider` or `MeterProvider`
if another resource was not explicitly specified.
-->

SDKは、[Semantic Attributes with SDK-provided Default Value](semantic_conventions/README.md#semantic-attributes-with-sdk-provided-default-value)に記載されている属性を少なくとも持つResourceへのアクセスを提供しなければなりません(MUST)。他のリソースが明示的に指定されていない場合、このリソースは、`TracerProvider`または`MeterProvider`と関連付けられなければなりません(MUST)。

<!--
Note: This means that it is possible to create and associate a resource that
does not have all or any of the SDK-provided attributes present. However, that
does not happen by default. If a user wants to combine custom attributes with
the default resource, they can use [`Merge`](#merge) with their custom resource
or specify their attributes by implementing
[Custom resource detectors](#detecting-resource-information-from-the-environment)
instead of explicitly associating a resource.
-->

注:これは、SDKが提供する属性のすべてまたは一部が存在しないリソースを作成して関連付けることが可能であることを意味します。しかし、それはデフォルトでは起こりません。ユーザーがデフォルトのリソースにカスタム属性を組み合わせたい場合は、[`Merge`](#merge)をカスタムリソースに使用するか、リソースを明示的に関連付ける代わりに、[環境からリソース情報を検出する](#環境からリソース情報を検出する)を実装して属性を指定します。


<!--
## Resource creation
-->

## リソースの作成

<!--
The SDK must support two ways to instantiate new resources. Those are:
-->

SDKは、新しいリソースをインスタンス化する2つの方法をサポートする必要があります。それらは以下の２つです:


<!--
### Create
-->

### Create

<!--
The interface MUST provide a way to create a new resource, from [`Attributes`](../common/common.md#attributes).
Examples include a factory method or a constructor for a resource
object. A factory method is recommended to enable support for cached objects.
-->

インターフェースは、[`Attributes`](../common/common.md#attributes)から、新しいリソースを作成する方法を提供しなければなりません(MUST)。例えば、リソースオブジェクトのファクトリーメソッドやコンストラクタなどです。キャッシュされたオブジェクトのサポートを可能にするために、ファクトリーメソッドを推奨します。

<!--
Required parameters:
-->

必須パラメータ:

<!--
- [`Attributes`](../common/common.md#attributes)
-->

- [`Attributes`](../common/common.md#attributes)

<!--
### Merge
-->

### Merge

<!--
The interface MUST provide a way for an old resource and an
updating resource to be merged into a new resource.
-->

インターフェースは、古いリソースと更新中のリソースを新しいリソースに統合する方法を提供しなければなりません(MUST)。

<!--
Note: This is intended to be utilized for merging of resources whose attributes
come from different sources,
such as environment variables, or metadata extracted from the host or container.
-->

注:これは、環境変数や、ホストやコンテナから抽出されたメタデータなど、異なるソースから属性を得ているリソースをマージするために使用することを目的としています。

<!--
The resulting resource MUST have all attributes that are on any of the two input resources.
If a key exists on both the old and updating resource, the value of the updating
resource MUST be picked (even if the updated value is empty).
-->

結果として得られるリソースは、2つの入力リソースのいずれかに存在するすべての属性を持たなければなりません(MUST)。旧リソースと更新リソースの両方にキーが存在する場合、(更新値が空であっても)更新リソースの値が選ばれなければなりません(MUST)。

<!--
Required parameters:
-->

必須パラメータ:

<!--
- the old resource
- the updating resource whose attributes take precedence
-->

- 古いリソース
- 属性が優先される更新リソース

<!--
### The empty resource
-->

### 空リソース

<!--
It is recommended, but not required, to provide a way to quickly create an empty
resource.
-->

空のリソースを素早く作成する方法を提供することが推奨されていますが、必須ではありません。

<!--
### Detecting resource information from the environment
-->

### 環境からリソース情報を検出する

<!--
Custom resource detectors related to generic platforms (e.g. Docker, Kubernetes)
or vendor specific environments (e.g. EKS, AKS, GKE) MUST be implemented as
packages separate from the SDK.
-->

一般的なプラットフォーム(Docker、Kubernetesなど)やベンダー固有の環境(EKS、AKS、GKEなど)に関連するカスタムリソース検出器は、SDKとは別のパッケージとして実装しなければなりません(MUST)。

<!--
Resource detector packages MUST provide a method that returns a resource. This
can then be associated with `TracerProvider` or `MeterProvider` instances as
described above.
-->

リソース検出器パッケージは、リソースを返すメソッドを提供しなければなりません(MUST)。これにより、上述のように、`TracerProvider` や `MeterProvider` のインスタンスと関連付けられます。

<!--
Resource detector packages MAY detect resource information from multiple
possible sources and merge the result using the `Merge` operation described
above.
-->

リソース検出器パッケージは、複数のソースからリソース情報を検出し、その結果を前述の `Merge` 操作でマージしても構いません(MAY)。

<!--
Resource detection logic is expected to complete quickly since this code will be
run during application initialization. Errors should be handled as specified in
the [Error Handling
principles](../error-handling.md#basic-error-handling-principles). Note the
failure to detect any resource information MUST NOT be considered an error,
whereas an error that occurs during an attempt to detect resource information
SHOULD be considered an error.
-->

このコードはアプリケーションの初期化時に実行されるため、リソース検出ロジックは短時間で完了することが期待されます。エラーは、[エラー処理の原則](../error-handling.md#basic-error-handling-principles)で指定されているように処理されるべきです。なお、リソース情報の検出に失敗した場合はエラーとみなしてはいけません(MUST NOT)が、リソース情報を検出しようとした際に発生したエラーはエラーとみなすべきです(SHOULD)。

<!--
### Specifying resource information via an environment variable
-->

### 環境変数によるリソース情報の指定

<!--
The SDK MUST extract information from the `OTEL_RESOURCE_ATTRIBUTES` environment
variable and [merge](#merge) this, as the secondary resource, with any resource
information provided by the user, i.e. the user provided resource information
has higher priority.
-->

SDKは、環境変数`OTEL_RESOURCE_ATTRIBUTES`から情報を抽出し、これを二次リソースとして、ユーザーが提供したリソース情報と[マージ](#merge)しなければなりません(つまり、ユーザーが提供したリソース情報の方が優先されます)。

<!--
The `OTEL_RESOURCE_ATTRIBUTES` environment variable will contain of a list of
key value pairs, and these are expected to be represented in a format matching
to the [W3C Baggage](https://github.com/w3c/baggage/blob/fdc7a5c4f4a31ba2a36717541055e551c2b032e4/baggage/HTTP_HEADER_FORMAT.md#header-content),
except that additional semi-colon delimited metadata is not supported, i.e.:
`key1=value1,key2=value2`. All attribute values MUST be considered strings.
-->

環境変数 `OTEL_RESOURCE_ATTRIBUTES` には、キーと値のペアのリストが含まれます。これらは、セミコロンで区切られた追加のメタデータがサポートされていないことを除いて、[W3C Baggage](https://github.com/w3c/baggage/blob/fdc7a5c4f4a31ba2a36717541055e551c2b032e4/baggage/HTTP_HEADER_FORMAT.md#header-content)に沿ったフォーマットで表現されることが期待されています(例:`key1=value1,key2=value2`)。すべての属性値は文字列とみなされなければなりません(MUST)。

<!--
## Resource operations
-->

## リソース操作

<!--
Resources are immutable. Thus, in addition to resource creation,
only the following operations should be provided:
-->

リソースは不変(Immutable)です。したがって、リソースの作成に加えて、以下のような操作のみを提供する必要があります。

<!--
### Retrieve attributes
-->

### 属性の取得

<!--
The SDK should provide a way to retrieve a read only collection of attributes
associated with a resource.
-->

SDKは、リソースに関連付けられた属性の読み取り専用の属性集合(Collection)を取得する方法を提供する必要があります。

<!--
There is no need to guarantee the order of the attributes.
-->

属性の順番を保証する必要はありません。

<!--
The most common operation when retrieving attributes is to enumerate over them. As
such, it is recommended to optimize the resulting collection for fast
enumeration over other considerations such as a way to quickly retrieve a value
for a attribute with a specific key.
-->

属性を取得する際の最も一般的な操作は、その属性を列挙(enumerate)することです。そのため、特定のキーを持つ属性の値を素早く取得する方法などの他の考慮事項よりも、高速に列挙できるように結果の集合(Collection)を最適化することが推奨されます。

