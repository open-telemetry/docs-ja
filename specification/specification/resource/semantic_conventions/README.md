<!--
# Resource Semantic Conventions
-->
# Resource セマンティック規約

**Status**: [Experimental](../../document-status.md)

<!--
This document defines standard attributes for resources. These attributes are typically used in the [Resource](../sdk.md) and are also recommended to be used anywhere else where there is a need to describe a resource in a consistent manner. The majority of these attributes are inherited from [OpenCensus Resource standard](https://github.com/census-instrumentation/opencensus-specs/blob/master/resource/StandardResources.md).
-->

この文書では、Resourceの標準属性を定義しています。これらの属性は通常 [Resource](../sdk.md) で使用され、Resourceを一貫した方法で記述する必要がある場合にはどこでも使用することが推奨されます。これらの属性の大部分は [OpenCensus Resource standard](https://github.com/census-instrumentation/opencensus-specs/blob/master/resource/StandardResources.md) から継承されています。

<!-- Re-generate TOC with `markdown-toc --no-first-h1 -i` -->

<!-- toc -->

<!--
- [TODOs](#todos)
- [Document Conventions](#document-conventions)
- [Semantic Attributes with SDK-provided Default Value](#semantic-attributes-with-sdk-provided-default-value)
- [Service](#service)
- [Telemetry SDK](#telemetry-sdk)
- [Compute Unit](#compute-unit)
- [Compute Instance](#compute-instance)
- [Environment](#environment)
- [Version attributes](#version-attributes)
- [Cloud-Provider-Specific Attributes](#cloud-provider-specific-attributes)
-->

- [TODO](#todo)
- [ドキュメント規約](#document-conventions)
- [SDKが提供するデフォルト値を持つセマンティック属性](#semantic-attributes-with-sdk-provided-default-value)
- [サービス](#service)
- [Telemetry SDK](#telemetry-sdk)
- [Compute Unit](#compute-unit)
- [Compute Instance](#compute-instance)
- [環境](#環境)
- [バージョン属性](#version-attributes)
- [クラウドプロバイダ固有の属性](#クラウドプロバイダ固有の属性)

<!-- tocstop -->

## TODO

<!--
* Add more compute units: AppEngine unit, etc.
* Add Device (mobile) and Web Browser.
* Decide if lower case strings only.
* Consider to add optional/required for each attribute and combination of attributes
  (e.g when supplying a k8s resource all k8s may be required).
-->

* 計算単位の追加。AppEngine単位など。
* デバイス(モバイル)とWebブラウザを追加する。
* 小文字の文字列のみにするかどうかを決める
* 各属性や属性の組み合わせについて、オプション/必須を追加することを検討
  (例: k8sリソースを供給する場合は、すべてのk8が必要になるかもしれません)。

<!--
## Document Conventions
-->

## ドキュメント規約

<!-- Attributes are grouped logically by the type of the concept that they described. Attributes in the same group have a common prefix that ends with a dot. For example all attributes that describe Kubernetes properties start with "k8s."
 -->

属性は、記述されたコンセプトの種類によって論理的にグループ化されています。同じグループの属性には、ドットで終わる共通のプレフィックスがあります。例えば、Kubernetesのプロパティを記述するすべての属性は、"k8s."で始まります。

<!-- Certain attribute groups in this document have a **Required** column. For these groups if any attribute from the particular group is present in the Resource then all attributes that are marked as Required MUST be also present in the Resource. However it is also valid if the entire attribute group is omitted (i.e. none of the attributes from the particular group are present even though some of them are marked as Required in this document).
-->

このドキュメントの特定の属性グループには、**Required**列があります。これらのグループでは、特定のグループの属性がリソースに存在する場合、Requiredとしてマークされたすべての属性がリソースにも存在しなければなりません(MUST)。ただし、属性グループ全体が省略されている場合も有効であす(つまり、このドキュメントでRequiredとマークされている属性があっても、特定のグループの属性が一つも存在していない場合)。


<!-- ## Semantic Attributes with SDK-provided Default Value
-->

## SDKが提供するデフォルト値を持つセマンティック属性

<!-- These are the the attributes which MUST be provided by the SDK
as specified in the [Resource SDK specification](../sdk.md#sdk-provided-resource-attributes):
-->

これらは、[Resource SDK specification](../sdk.md#sdk-provided-resource-attributes)で規定されている、SDKが提供しなければならない属性です。

- [`service.name`](#service)

<!-- ## Service
-->

## Service

**type:** `service`

<!-- **Description:** A service instance.
-->

**説明:** サービスのインスタンス

<!-- semconv service -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `service.name` | string | サービスの論理名。 [1] | `shoppingcart` | Yes |
| `service.namespace` | string | `service.name`の名前空間。 [2] | `Shop` | No |
| `service.instance.id` | string | サービスインスタンスを表すID文字列。 [3] | `627cc493-f310-47de-96bd-71410b7dec09` | No |
| `service.version` | string | サービスAPIまたは実装のバージョン文字列。 | `2.0.0` | No |

**[1]:** 水平方向にスケールされたサービスのすべてのインスタンスに対して同じ値を指定しなければなりません(MUST)。値が指定されていない場合、SDKは `unknown_service:` を [`process.executeable.name`](process.md#process) で連結したもの、例えば `unknown_service:bash` などにフォールバックしなければなりません(MUST)。もし `process.executeable.name` が利用できない場合は、`unknown_service` を設定しなければなりません(MUST)。

**[2]:** サービスのグループを区別するのに役立つ意味を持つ文字列値。例えば、サービスのグループを所有するチーム名などです。 `service.name` は同じ名前空間内で一意であることが期待されます。`service.namespace` がResourceに指定されていない場合、`service.name` は明示的な名前空間が定義されていないすべてのサービスに対して一意であることが期待されます (つまり、空の/未指定の名前空間は単に有効な名前空間の1つに過ぎません)。長さ0の名前空間文字列は、未指定の名前空間と同じとみなされます。

**[3]:** 同じ `service.namespace,service.name` のペアの各インスタンスに対して一意でなければなりません(言い換えれば、`service.namespace,service.name,service.instance.id` の一組はグローバルに一意でなければなりません(MUST)。IDは、同時に存在する同じサービスのインスタンスを区別するのに役立ちます(例えば、水平方向にスケーリングされたサービスのインスタンス)。ID は永続的で、サービスインスタンスの寿命の間は同じであることが望ましいですが、サービスの重要な寿命イベント(サービスの再起動など)の間に ID が変化しても構いません。サービスがこの属性の値として使用できる固有のユニークなIDを持たない場合、ランダムなバージョン1またはバージョン4のRFC 4122 UUIDを生成することが推奨されます(再現可能なUUIDを目指すサービスはバージョン5を使用することもできます。より多くの推奨事項についてはRFC 4122を参照してください)。
<!-- endsemconv -->

<!-- Note: `service.namespace` and `service.name` are not intended to be concatenated for the purpose of forming a single globally unique name for the service. For example the following 2 sets of attributes actually describe 2 different services (despite the fact that the concatenation would result in the same string):
-->

注意: `service.namespace` と `service.name` は、サービスのグローバルに一意な名前を形成する目的で連結することを意図していません。例えば、以下の2つの属性セットは、実際には2つの異なるサービスを表しています(連結すると同じ文字列になるにもかかわらず)。


```
# あるサービスを説明するリソース属性
namespace = Company.Shop
service.name = shoppingcart
```

```
# 別のサービスを記述するリソース属性の別のセット
namespace = Company
service.name = Shop.shoppingcart
```

<!-- ## Telemetry SDK
-->

## Telemetry SDK

**type:** `telemetry.sdk`

<!-- **Description:** The telemetry SDK used to capture data recorded by the instrumentation libraries.
-->

**説明:** 計装ライブラリで記録されたデータをキャプチャするために使用されるTelemetry SDKです。

<!-- The default OpenTelemetry SDK provided by the OpenTelemetry project MUST set `telemetry.sdk.name`
to the value `opentelemetry`.
-->

OpenTelemetryプロジェクトが提供するデフォルトのOpenTelemetry SDKは、`telemetry.sdk.name`に`opentelemetry`という値を設定しなければなりません(MUST)。

<!-- If another SDK, like a fork or a vendor-provided implementation, is used, this SDK MUST set the attribute
`telemetry.sdk.name` to the fully-qualified class or module name of this SDK's main entry point
or another suitable identifier depending on the language.
The identifier `opentelemetry` is reserved and MUST NOT be used in this case.
The identifier SHOULD be stable across different versions of an implementation.
-->

フォークやベンダーが提供する実装など、別のSDKを使用する場合、このSDKは、属性 `telemetry.sdk.name` に、このSDKのメインエントリーポイントの完全修飾クラスまたはモジュール名、あるいは言語に応じた別の適切な識別子を設定しなければなりません(MUST)。識別子 `opentelemetry` は予約済みで、この場合は使用してはいけません(MUST NOT)。識別子は、実装の異なるバージョン間で安定しているべきです(SHOULD)。

<!-- semconv telemetry -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `telemetry.sdk.name` | string | 上記で定義したテレメトリSDKの名前。 | `opentelemetry` | No |
| `telemetry.sdk.language` | string | テレメトリSDKの言語。 | `cpp` | No |
| `telemetry.sdk.version` | string | テレメトリSDKのバージョン文字列。 | `1.2.3` | No |
| `telemetry.auto.version` | string | 使用されている場合は、自動インストルメンテーションエージェントのバージョン文字列。 | `1.2.3` | No |

`telemetry.sdk.language` MUST be one of the following or, if none of the listed values apply, a custom value:

| Value  | Description |
|---|---|
| `cpp` | cpp |
| `dotnet` | dotnet |
| `erlang` | erlang |
| `go` | go |
| `java` | java |
| `nodejs` | nodejs |
| `php` | php |
| `python` | python |
| `ruby` | ruby |
| `webjs` | webjs |
<!-- endsemconv -->

<!-- ## Compute Unit
-->

## 計算機ユニット

<!-- Attributes defining a compute unit (e.g. Container, Process, Function as a Service):
-->

計算機ユニットを定義する属性(例:コンテナ、プロセス、Function as a Service)。

<!-- - [Container](./container.md)
- [Function as a Service](./faas.md)
- [Process](./process.md)
- [Web engine](./webengine.md)
-->

- [Container](./container.md)
- [Function as a Service](./faas.md)
- [Process](./process.md)
- [Web engine](./webengine.md)

<!-- ## Compute Instance
-->

## 計算機インスタンス

<!--
Attributes defining a computing instance (e.g. host):
-->

計算機インスタンスを定義する属性(例:ホスト)。

- [Host](./host.md)

<!--
## Environment
-->

## 環境

<!--
Attributes defining a running environment (e.g. Operating System, Cloud, Data Center, Deployment Service):
-->

実行環境を定義する属性(例:オペレーティングシステム、クラウド、データセンター、デプロイメントサービス)。

<!--
- [Operating System](./os.md)
- [Cloud](./cloud.md)
- Deployment:
  - [Deployment Environment](./deployment_environment.md)
  - [Kubernetes](./k8s.md)
-->

- [オペレーティングシステム](./os.md)
- [クラウド](./cloud.md)
- デプロイ:
  - [デプロイ環境](./deployment_environment.md)
  - [Kubernetes](./k8s.md)

<!--
## Version attributes
-->

## バージョン属性

<!-- Version attributes, such as `service.version`, are values of type `string`. They are
the exact version used to identify an artifact. This may be a semantic version, e.g., `1.2.3`, git hash, e.g.,
`8ae73a`, or an arbitrary version string, e.g., `0.1.2.20210101`, whatever was used when building the artifact.
-->

`service.version`のようなバージョン属性で、`string`型の値です。これは、アーティファクトの識別に使用される正確なバージョンです。これは、セマンティックバージョン(例:`1.2.3`)、gitハッシュ(例:`8ae73a`)、または任意のバージョン文字列(例:`0.1.2.20210101`)など、アーティファクトの構築時に使用されたものであれば何でも構いません。

<!--
## Cloud-Provider-Specific Attributes
-->

## クラウドプロバイダ固有の属性

<!-- Attributes that are only applicable to resources from a specific cloud provider. Currently, these
resources can only be defined for providers listed as a valid `cloud.provider` in
[Cloud](./cloud.md) and below. Provider-specific attributes all reside in the `cloud_provider` directory.
Valid cloud providers are:
-->

特定のクラウドプロバイダーからのリソースにのみ適用される属性です。現在、これらのリソースは、[Cloud](./cloud.md)以下で有効な `cloud.provider` としてリストアップされているプロバイダに対してのみ定義できます。プロバイダー固有の属性はすべて、`cloud_provider` ディレクトリにあります。有効なクラウドプロバイダーは以下の通りです。

- [Amazon Web Services](https://aws.amazon.com/) ([`aws`](cloud_provider/aws/README.md))
- [Google Cloud Platform](https://cloud.google.com/) (`gcp`)
- [Microsoft Azure](https://azure.microsoft.com/) (`azure`)
