<!--
# Resource SDK
-->

# Resource SDK

<!--
A [Resource](../overview.md#resources) is an immutable representation of the entity producing
telemetry. For example, a process producing telemetry that is running in a
container on Kubernetes has a Pod name, it is in a namespace and possibly is
part of a Deployment which also has a name. All three of these attributes can be
included in the `Resource`.
-->

[Resource](./overview.md#resources)は、テレメトリーを生成するエンティティのイミュータブルな表現です。例えば、Kubernetes上のコンテナで実行されているテレメトリーを生成するプロセスはPod名を持ち、namespaceにあり、おそらく名前を持つデプロイメントの一部でもあります。これら3つの属性はすべて `Resource` に含めることができます。

<!--
The primary purpose of resources as a first-class concept in the SDK is
decoupling of discovery of resource information from exporters. This allows for
independent development and easy customization for users that need to integrate
with closed source environments. The SDK MUST allow for creation of `Resources` and
for associating them with telemetry.
-->

SDKにおける第一級の概念としてのResourceの主な目的は、Resource情報のディスカバリーをエクスポート先から切り離すことです。これにより、クローズドソース環境との統合を必要とするユーザーのために、独立した開発と容易なカスタマイズを可能にします。SDKは、`Resource`を作成し、それをテレメトリーに関連付けることができるようにしなければなりません(MUST)。

<!--
When used with distributed tracing, a resource can be associated with the
[TracerProvider](../trace/sdk.md#tracer-sdk) when it is created.
That association cannot be changed later.
When associated with a `TracerProvider`,
all `Span`s produced by any `Tracer` from the provider MUST be associated with this `Resource`.
-->

分散トレーシングで使用する場合、Resourceは作成時に[TracerProvider](./trace/sdk.md#tracer-sdk)と関連付けることができます。この関連付けは後で変更することはできません。`TracerProvider`と関連付けられている場合、そのプロバイダの `Tracer` が生成したすべての `Span` は、この `Resource` と関連付けらなければなりません(MUST)。

<!--
Analogous to distributed tracing, when used with metrics,
a resource can be associated with a `MeterProvider`.
When associated with a [`MeterProvider`](../metrics/api.md#meter-interface),
all metrics produced by any `Meter` from the provider will be
associated with this `Resource`.
-->

分散トレーシングと同様に、メトリックを使用する場合は、Resourceを `MeterProvider` に関連付けることができます。[`MeterProvider`](./metrics/api.md#meter-interface)に関連付けられている場合、プロバイダからの`Meter`によって生成されたすべてのメトリックは、この`Resource`に関連付けられます。

<!--
## Resource creation
-->

## Resource 生成

<!--
The SDK must support two ways to instantiate new resources. Those are:
-->

SDKは、新しいResourceをインスタンス化する2つの方法をサポートしなければなりません(MUST)。それは以下の2つです。

<!--
### Create
-->

### 生成(Create)

<!--
The interface MUST provide a way to create a new resource, from a collection of
attributes. Examples include a factory method or a constructor for a resource
object. A factory method is recommended to enable support for cached objects.
-->

このインターフェースは、属性の集合から新しいResourceを作成する方法を提供しなければなりません(MUST)。例えば、ファクトリーメソッドやResouceオブジェクトのコンストラクタなどがあります。ファクトリーメソッドは、キャッシュされたオブジェクトのサポートを可能にするために推奨されます。

<!--
Required parameters:
-->

必要なパラメータ:

<!--
- a collection of name/value attributes, where name is a string and value can be one
  of: string, int64, double, bool.
-->

- キー/値の属性ペアの配列。キーは文字列で、値は以下のうちのどれかです: 文字列、int64, double, bool

<!--
### Merge
-->

### マージ

<!--
The interface MUST provide a way for a primary resource and a
secondary resource to be merged into a new resource.
-->

このインターフェースは、プライマリResourceとセカンダリResourceを新しいResourceにマージする方法を提供しなければなりません(MUST)。

<!--
Note: This is intended to be utilized for merging of resources whose attributes
come from different sources,
such as environment variables, or metadata extracted from the host or container.
-->

注意: これは、環境変数やホストやコンテナから抽出されたメタデータなど、異なるソースからの属性を持つResourceをマージするために利用されることを意図しています。

<!--
The resulting resource MUST have all attributes that are on any of the two input resources.
Conflicts (i.e. a key for which attributes exist on both the primary and secondary resource)
MUST be handled as follows:
-->

結果として得られるResourceは、2つの入力Resourceのいずれかに存在するすべての属性を持つ必要があります(MUST)。競合(すなわち、プライマリResourceとセカンダリResourceの両方に属性が存在するキー)は以下のように処理しなければいけません。

<!--
* If the value on the primary resource is an empty string, the result has the value of the secondary resource.
* Otherwise, the value of the primary resource is used.
-->

* プライマリResourceの値が空文字列の場合、セカンダリResourceの値を使用します
* それ以外の場合は、プライマリResourceの値が使用されます

<!--
Attribute key namespacing SHOULD be used to prevent collisions across different
resource detection steps.
-->

異なるResource検出ステップ間の衝突を防ぐために、属性キーの名前空間を使用するべきです(SHOULD)。

<!--
Required parameters:
-->

必要なパラメータ:

<!--
- the primary resource whose attributes take precedence.
- the secondary resource whose attributes will be merged in.
-->

- 属性が優先されるプライマリResource
- 属性がマージされるセカンダリResource

<!--
### The empty resource
-->

### 空Resource

<!--
It is recommended, but not required, to provide a way to quickly create an empty
resource.
-->

空のResourceを素早く作成する方法を提供することが推奨されますが、必須ではありません。

<!--
Note that the OpenTelemetry project documents certain ["standard
attributes"](semantic_conventions/README.md) that have prescribed semantic meanings.
-->

OpenTelemetryプロジェクトでは、所定の意味を持つ["標準的な属性"](semantic_conventions/README.md)が文書化されていることに注意してください。

<!--
## Resource operations
-->

## Resource操作

<!--
Resources are immutable. Thus, in addition to resource creation,
only the following operations should be provided:
-->

Resourceはイミュータブルです。したがって、Resourceの作成に加えて、以下の操作のみを提供する必要があります。

<!--
### Retrieve attributes
-->

### 属性の取得

<!--
The SDK should provide a way to retrieve a read only collection of attributes
associated with a resource. The attributes should consist of the name and values,
both of which should be strings.
-->

SDKは、Resourceに関連付けられた属性の読み取り専用の配列を取得する方法を提供しなければなりません。属性は、名前と値から構成され、どちらも文字列でなければなりません。

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

属性を取得する際の最も一般的な操作は、それらの属性を列挙することです。そのため、特定のキーを持つ属性の値を素早く取得する方法などを設計するよりも、配列を高速に列挙するように最適化することをお勧めします。
