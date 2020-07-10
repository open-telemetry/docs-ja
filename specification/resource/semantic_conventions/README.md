<!--
# Resource Semantic Conventions
-->

# Resource のセマンティック規約

<!--
This document defines standard attributes for resources. These attributes are typically used in the [Resource](../sdk.md) and are also recommended to be used anywhere else where there is a need to describe a resource in a consistent manner. The majority of these attributes are inherited from
[OpenCensus Resource standard](https://github.com/census-instrumentation/opencensus-specs/blob/master/resource/StandardResources.md).
-->

この文書では、Resourceの標準的属性を定義します。これらの属性は通常、[Resource](../sdk.md)で使用され、また一貫した方法で記述する必要がある場合には、他の場所でも使用することが推奨されます。これらの属性の大部分は [OpenCensus Resource standard](https://github.com/census-instrumentation/opencensus-specs/blob/master/resource/StandardResources.md) から継承されています。

<!--
- [Service](#service)
- [Telemetry SDK](#telemetry-sdk)
- [Compute Unit](#compute-unit)
  * [Container](#container)
  * [Function as a Service](#function-as-a-service)
- [Deployment Service](#deployment-service)
  * [Kubernetes](#kubernetes)
- [Compute Instance](#compute-instance)
  * [Host](#host)
- [Environment](#environment)
  * [Cloud](#cloud)
-->

- [Service](#service)
- [テレメトリーSDK](#テレメトリーsdk)
- [計算単位(Compute Unit)](#計算単位-compute-unit)
  * [コンテナ](#コンテナ)
  * [Function as a Service](#function-as-a-service)
- [Deploymentサービス](#Deploymentサービス)
  * [Kubernetes](#kubernetes)
- [Compute Instance](#compute-instance)
  * [Host](#host)
- [Environment](#environment)
  * [Cloud](#cloud)

<!-- tocstop -->

<!--
## TODOs
-->

## TODO

<!--
* Add more compute units: Process, AppEngine unit, etc.
* Add Device (mobile) and Web Browser.
* Decide if lower case strings only.
* Consider to add optional/required for each attribute and combination of attributes
  (e.g when supplying a k8s resource all k8s may be required).
-->

* 計算単位の追加: プロセス、AppEngineユニットなど
* デバイス(mobile)とWebブラウザの追加
* 小文字のみの文字列の場合の処理を決める
* 各属性や属性の組み合わせについて、オプション/必須を追加することを検討する（例：k8s Resourceを提供する場合には、すべてのk8sが必要になるかもしれません）

<!--
### Document Conventions
-->

### 文書規約

<!--
Attributes are grouped logically by the type of the concept that they described. Attributes in the same group have a common prefix that ends with a dot. For example all attributes that describe Kubernetes properties start with "k8s."
-->

属性は、それらが記述する概念の種類によって論理的にグループ化されます。同じグループ内の属性は、ドット(`.`)で終わる共通の接頭辞を持っています。例えば、Kubernetesのプロパティを記述するすべての属性は "k8s."で始まります。

<!--
Certain attribute groups in this document have a **Required** column. For these groups if any attribute from the particular group is present in the Resource then all attributes that are marked as Required MUST be also present in the Resource. However it is also valid if the entire attribute group is omitted (i.e. none of the attributes from the particular group are present even though some of them are marked as Required in this document).
-->

この文書中のいくつかの属性グループは **Required** カラムを持っています。これらの属性がResourceに存在する場合、Requiredとマークされている全ての属性がResourceに存在する必要があります(MUST)。しかし、属性グループ全体が省略されている場合も有効です（つまり、このドキュメントでRequiredとマークされている属性があるにもかかわらず、特定のグループの属性が存在しない場合）。(???ちょっと訳が取れていない)

<!--
## Service
-->

## Service

<!--
**type:** `service`
-->

**タイプ:** `service`

<!--
**Description:** A service instance.
-->

**説明:** serviceのインスタンス

<!--
| Attribute  | Description  | Example  | Required? |
|---|---|---|---|
| service.name | Logical name of the service. <br/> MUST be the same for all instances of horizontally scaled services. | `shoppingcart` | Yes |
| service.namespace | A namespace for `service.name`.<br/>A string value having a meaning that helps to distinguish a group of services, for example the team name that owns a group of services. `service.name` is expected to be unique within the same namespace. The field is optional. If `service.namespace` is not specified in the Resource then `service.name` is expected to be unique for all services that have no explicit namespace defined (so the empty/unspecified namespace is simply one more valid namespace). Zero-length namespace string is assumed equal to unspecified namespace. | `Shop` | No |
| service.instance.id | The string ID of the service instance. <br/>MUST be unique for each instance of the same `service.namespace,service.name` pair (in other words `service.namespace,service.name,service.id` triplet MUST be globally unique). The ID helps to distinguish instances of the same service that exist at the same time (e.g. instances of a horizontally scaled service). It is preferable for the ID to be persistent and stay the same for the lifetime of the service instance, however it is acceptable that the ID is ephemeral and changes during important lifetime events for the service (e.g. service restarts). If the service has no inherent unique ID that can be used as the value of this attribute it is recommended to generate a random Version 1 or Version 4 RFC 4122 UUID (services aiming for reproducible UUIDs may also use Version 5, see RFC 4122 for more recommendations). | `627cc493-f310-47de-96bd-71410b7dec09` | Yes |
| service.version | The version string of the service API or implementation as defined in [Version Attributes](#version-attributes). | `semver:2.0.0` | No |
-->

| 属性  | 説明  | 例  | Required? |
|---|---|---|---|
| service.name | サービスの論理的な名前。<br/> 水平方向にスケールされたserviceのすべてのインスタンスで同じでなければなりません(MUST) | `shoppingcart` | Yes |
| service.namespace | `service.name`の名前空間。<br/>serviceのグループを区別するのに役立つ意味を持つ文字列の値です。例えば、サービスのグループを所有するチーム名などです。`service.name` は同じ名前空間内で一意であることが期待されます。このフィールドはオプションです。Resourceで `service.namespace` が指定されていない場合は、明示的な名前空間が定義されていないすべてのサービスに対して `service.name` が一意であることが期待されます (つまり、空あるいは指定されていない名前空間は単に有効な名前空間の1つに過ぎません)。長さ0の名前空間文字列は、指定されていない名前空間と同じとみなされます | `Shop` | No |
| service.instance.id | Serviceインスタンスの文字列 ID。 <br/>同じ `service.namespace,service.name` のペアのインスタンスごとに一意でなければなりません(MUST) （言い換えれば `service.namespace,service.name,service.id` の３つのセットはグローバルに一意でなければなりません(MUST)）。ID は、同時に存在する同じサービスのインスタンスを区別するのに役立ちます（例: 水平方向にスケールされたサービスのインスタンス）。ID は永続的で、サービスインスタンスが生きている間は同じままであることが望ましいですが、サービスの重要なライフタイムイベント（サービスの再起動など）の間に ID が変化してしまうような、一時的なものであっても構いません。サービスがこの属性の値として使用できる固有の一意のIDを持たない場合、ランダムなバージョン1またはバージョン4のRFC 4122 UUIDを生成することが推奨されます（再現可能なUUIDを目指すサービスはバージョン5を使用することもできます。より多くの推奨事項についてはRFC 4122を参照してください）。 | `627cc493-f310-47de-96bd-71410b7dec09` | Yes |
| service.version | [バージョン属性](#バージョン属性)で定義されているサービスAPIまたは実装のバージョン文字列。 | `semver:2.0.0` | No |

<!--
Note: `service.namespace` and `service.name` are not intended to be concatenated for the purpose of forming a single globally unique name for the service. For example the following 2 sets of attributes actually describe 2 different services (despite the fact that the concatenation would result in the same string):
-->

注意: `service.namespace` と `service.name` は、グローバルに一意な単一の名前を形成するために連結することを意図していません。例えば、以下の2つの属性セットは、連結すると同じ文字列になりますが、実際には2つの異なるサービスを記述しています

<!--
```
# Resource attributes that describes a service.
namespace = Company.Shop
service.name = shoppingcart
```
-->

```
# serviceを記述するResource属性
namespace = Company.Shop
service.name = shoppingcart
```

<!--
```
# Another set of resource attributes that describe a different service.
namespace = Company
service.name = Shop.shoppingcart
```
-->

```
# 違うserviceを記述する別のResource属性
namespace = Company
service.name = Shop.shoppingcart
```
<!--
## Telemetry SDK
-->

## テレメトリー SDK

<!--
**type:** `telemetry.sdk`
-->

**タイプ:** `telemetry.sdk`

<!--
**Description:** The telemetry SDK used to capture data recorded by the instrumentation libraries.
-->

**説明:** 計装ライブラリで記録されたデータをキャプチャするために使用されるテレメトリーSDKです。

<!--
The default OpenTelemetry SDK provided by the OpenTelemetry project MUST set `telemetry.sdk.name`
to the value `opentelemetry`.
-->

OpenTelemetryプロジェクトが提供するデフォルトのOpenTelemetry SDKは、`telemetry.sdk.name` を値 `opentelemetry` に設定してください(MUST)。

<!--
If another SDK, like a fork or a vendor-provided implementation, is used, this SDK MUST set the attribute
`telemetry.sdk.name` to the fully-qualified class or module name of this SDK's main entry point
or another suitable identifier depending on the language.
The identifier `opentelemetry` is reserved and MUST NOT be used in this case.
The identifier SHOULD be stable across different versions of an implementation.
-->

別のSDK(フォークした場合やベンダ提供の実装など)を使用する場合、このSDKは属性 `telemetry.sdk.name` を、このSDKのメインエントリーポイントの完全修飾クラスまたはモジュール名、または言語に応じた別の適切な識別子に設定しなければなりません(MUST)。識別子 `opentelemetry` は予約済みであり、この場合には使用してはいけません(MUST NOT)。識別子は、実装の異なるバージョン間で安定しているべきです(SHOULD)。

<!--
| Attribute  | Description  | Example  | Required? |
|---|---|---|---|
| telemetry.sdk.name | The name of the telemetry SDK as defined above. | `opentelemetry` | No |
| telemetry.sdk.language | The language of the telemetry SDK.<br/> One of the following values MUST be used, if one applies: "cpp", "dotnet", "erlang", "go", "java", "nodejs", "php", "python", "ruby", "webjs" | `java` | No |
| telemetry.sdk.version | The version string of the telemetry SDK as defined in [Version Attributes](#version-attributes). | `semver:1.2.3` | No |
-->

| 属性  | 説明  | 例  | Required? |
|---|---|---|---|
| telemetry.sdk.name | 上記で定義したテレメトリSDKの名前です。 | `opentelemetry` | No |
| telemetry.sdk.language | テレメトリーSDKの言語<br/> 以下のいずれかの値を使用しなければなりません(MUST)。"cpp", "dotnet", "erlang", "go", "java", "nodejs", "php", "python", "ruby", "webjs" | `java` | No |
| telemetry.sdk.version | [バージョン属性](#バージョン属性)で定義されているテレメトリーSDKのバージョン文字列です。 | `semver:1.2.3` | No |


<!--
## Compute Unit
-->

## 計算単位(Compute Unit)

<!--
Attributes defining a compute unit (e.g. Container, Process, Function as a Service).
-->

計算単位を定義する属性(例: コンテナ, プロセス, Function as a Service)。

<!--
### Container
-->

### コンテナ

<!--
**type:** `container`
-->

**タイプ:** `container`

<!--
**Description:** A container instance.
-->

**説明:** コンテナインスタンス

<!--
| Attribute  | Description  | Example  |
|---|---|---|
| container.name | Container name. | `opentelemetry-autoconf` |
| container.image.name | Name of the image the container was built on. | `gcr.io/opentelemetry/operator` |
| container.image.tag | Container image tag. | `0.1` |
-->

| 属性  | 説明  | 例 |
|---|---|---|
| container.name | コンテナ名 | `opentelemetry-autoconf` |
| container.image.name | コンテナが構築されたイメージの名前。| `gcr.io/opentelemetry/operator` |
| container.image.tag | コンテナイメージのタグ| `0.1` |

<!--
### Function as a Service
-->

### Function as a Service

<!--
**type:** `faas`
-->

**タイプ:** `faas`

<!--
**Description:** A serverless instance.
-->

**説明:** サーバーレスのインスタンス

<!--
| Label  | Description  | Example  | Required |
|---|---|---|--|
| faas.name | The name of the function being executed. | `my-function` | Yes |
| faas.id | The unique name of the function being executed. <br /> For example, in AWS Lambda this field corresponds to the [ARN] value, in GCP to the URI of the resource, and in Azure to the [FunctionDirectory] field. | `arn:aws:lambda:us-west-2:123456789012:function:my-function` | Yes |
| faas.version | The version string of the function being executed as defined in [Version Attributes](#version-attributes). | `semver:2.0.0` | No |
| faas.instance | The execution environment ID as a string. | `my-function:instance-0001` | No |
-->

| Label  | 説明  | 例  | Required? |
|---|---|---|--|
| faas.name | 実行される関数の名前 | `my-function` | Yes |
| faas.id | 実行される関数の一意の名前。 <br /> 例えば、AWS Lambdaではこのフィールドは[ARN]の値に、GCPではリソースのURIに、Azureでは[FunctionDirectory]フィールドに対応しています。| `arn:aws:lambda:us-west-2:123456789012:function:my-function` | Yes |
| faas.version | [バージョン属性](#バージョン属性)で定義されている、実行中の関数のバージョン文字列。 | `semver:2.0.0` | No |
| faas.instance | 実行環境IDの文字列 | `my-function:instance-0001` | No |


<!--
Note: The resource attribute `faas.instance` differs from the span attribute `faas.execution`. For more information see the [Semantic conventions for FaaS spans](../../trace/semantic_conventions/faas.md#difference-between-execution-and-instance).
-->

注意: Resource 属性 `faas.instance` はSpan 属性 `faas.execution` とは異なります。詳細は [Semantic conventions for FaaS spans] (.../.../trace/semantic_conventions/faas.md#ifference-between-execution-and-instance) を参照してください。

<!--
[ARN]:https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html
[FunctionDirectory]: https://github.com/Azure/azure-functions-host/wiki/Retrieving-information-about-the-currently-running-function
-->

[ARN]:https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html
[FunctionDirectory]: https://github.com/Azure/azure-functions-host/wiki/Retrieving-information-about-the-currently-running-function

<!--
## Deployment Service
-->

## Deploymentサービス

<!--
Attributes defining a deployment service (e.g. Kubernetes).
-->

Deploymentサービス（Kubernetes など）を定義する属性。

<!--
### Kubernetes
-->

### Kubernetes

<!--
**type:** `k8s`
-->

**タイプ:** `k8s`

<!--
**Description:** A Kubernetes resource.
-->

**説明:** Kubernetesのresource

<!--
| Attribute  | Description  | Example  |
|---|---|---|
| k8s.cluster.name | The name of the cluster that the pod is running in. | `opentelemetry-cluster` |
| k8s.namespace.name | The name of the namespace that the pod is running in. | `default` |
| k8s.pod.name | The name of the pod. | `opentelemetry-pod-autoconf` |
| k8s.deployment.name | The name of the deployment. | `opentelemetry` |
-->

| 属性  | 説明  | 例  |
|---|---|---|
| k8s.cluster.name | Podが稼働しているクラスタの名前。 | `opentelemetry-cluster` |
| k8s.namespace.name | Podが動作している名前空間の名前。 | `default` |
| k8s.pod.name | Podの名前 | `opentelemetry-pod-autoconf` |
| k8s.deployment.name | Deploymentの名前 | `opentelemetry` |

<!--
## Compute Instance
-->

## コンピューティングインスタンス

<!--
Attributes defining a computing instance (e.g. host).
-->

コンピューティングインスタンスを定義する属性（ホストなど）。

<!--
### Host
-->

### ホスト

<!--
**type:** `host`
-->

**タイプ:** `host`

<!--
**Description:** A host is defined as a general computing instance.
-->

**説明:** ホストは一般的なコンピューティングインスタンスとして定義されます。

<!--
| Attribute  | Description  | Example  |
|---|---|---|
| host.hostname | Hostname of the host.<br/> It contains what the `hostname` command returns on the host machine. | `opentelemetry-test` |
| host.id | Unique host id.<br/> For Cloud this must be the instance_id assigned by the cloud provider | `opentelemetry-test` |
| host.name | Name of the host.<br/> It may contain what `hostname` returns on Unix systems, the fully qualified, or a name specified by the user. | `opentelemetry-test` |
| host.type | Type of host.<br/> For Cloud this must be the machine type.| `n1-standard-1` |
| host.image.name | Name of the VM image or OS install the host was instantiated from. | `infra-ami-eks-worker-node-7d4ec78312`, `CentOS-8-x86_64-1905` |
| host.image.id | VM image id. For Cloud, this value is from the provider. | `ami-07b06b442921831e5` |
| host.image.version | The version string of the VM image as defined in [Version Attributes](#version-attributes). | `0.1` |
-->

| 属性  | 説明  | 例  |
|---|---|---|
| host.hostname | ホストのホスト名<br/> ホストマシン上で `hostname` コマンドが返す内容を格納します。 | `opentelemetry-test` |
| host.id | 一意なホストID。<br/>   クラウドの場合、これはクラウドプロバイダーによって割り当てられた instance_id でなければなりません(MUST)。 | `opentelemetry-test` |
| host.name |ホストの名前<br/> Unixシステムでは `hostname` が返すもの、完全修飾されたもの、あるいはユーザが指定した名前などです。 | `opentelemetry-test` |
| host.type | ホストの種類 <br/> クラウドの場合、これはマシンタイプでなければなりません(MUST)。| `n1-standard-1` |
| host.image.name | ホストがインスタンス化されたVMイメージまたはOSのインストール名。 | `infra-ami-eks-worker-node-7d4ec78312`, `CentOS-8-x86_64-1905` |
| host.image.id | VM イメージ ID。クラウドの場合、この値はプロバイダから得られます | `ami-07b06b442921831e5` |
| host.image.version | [バージョン属性](#バージョン属性)で定義されているVMイメージのバージョン文字列。| `0.1` |

<!--
## Environment
-->

## 環境

<!--
Attributes defining a running environment (e.g. Cloud, Data Center).
-->

実行環境（クラウド、データセンターなど）を定義する属性。

<!--
### Cloud
-->

### クラウド

<!--
**type:** `cloud`
-->

**タイプ:** `cloud`

<!--
**Description:** A cloud infrastructure (e.g. GCP, Azure, AWS).
-->

**説明:** クラウドインフラ (例: GCP, Azure, AWS)

<!--
| Attribute  | Description  | Example  |
|---|---|---|
| cloud.provider | Name of the cloud provider.<br/> Example values are aws, azure, gcp. | `gcp` |
| cloud.account.id | The cloud account id used to identify different entities. | `opentelemetry` |
| cloud.region | A specific geographical location where different entities can run | `us-central1` |
| cloud.zone | Zones are a sub set of the region connected through low-latency links.<br/> In aws it is called availability-zone. | `us-central1-a` |
-->

| 属性  | 説明  | 例  |
|---|---|---|
| cloud.provider | クラウドプロバイダーの名前<br/> 例えばaws、 azure、 gcpです | `gcp` |
| cloud.account.id | 異なるエンティティを識別するために使用されるクラウドアカウントID。 | `opentelemetry` |
| cloud.region | 異なるエンティティが実行できる特定の地理的な場所 | `us-central1` |
| cloud.zone | ゾーンは、低遅延リンクを介して接続された領域のサブセットです。<br/> awsではavailability-zoneです。 | `us-central1-a` |

<!--
## Version Attributes
-->

## バージョン属性

<!--
Version attributes such as `service.version` and `library.version` are values of type `string`,
with naming schemas hinting at the type of a version, such as the following:
-->

`service.version` や `library.version` と言ったバージョン属性は文字列です。以下のようにバージョンの型を示す命名スキーマを持ちます。

<!--
- `semver:1.2.3` (a semantic version)
- `git:8ae73a` (a git sha hash)
- `0.0.4.2.20190921` (a untyped version)
-->

- `semver:1.2.3` (セマンティックバージョニング)
- `git:8ae73a` (gitのSHAハッシュ)
- `0.0.4.2.20190921` (種類が未指定のバージョン)

<!--
The type and version value MUST be separated by a colon character `:`.
-->

タイプとバージョンの値はコロン文字 `:` で区切らなければなりません(MUST)。
