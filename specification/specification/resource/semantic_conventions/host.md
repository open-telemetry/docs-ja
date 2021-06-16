<!--
# Host
-->

# ホスト

**Status**: [Experimental](../../document-status.md)

**type:** `host`

<!--
**Description:** A host is defined as a general computing instance.
-->

**Description:** ホストとは、一般的なコンピューティングインスタンスと定義されます。

<!-- semconv host -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `host.id` | string | 一意のホスト ID。クラウドの場合、これはクラウドプロバイダーによって割り当てられた instance_id である必要があります。 | `opentelemetry-test` | No |
| `host.name` | string | ホストの名前。Unix システムでは、hostname コマンドが返すもの、完全修飾されたホスト名、あるいはユーザが指定した別の名前が含まれます。 | `opentelemetry-test` | No |
| `host.type` | string | ホストの種類。クラウドの場合は、マシンタイプである必要があります。 | `n1-standard-1` | No |
| `host.arch` | string | ホストシステムが動いているCPUアーキテクチャ | `amd64` | No |
| `host.image.name` | string | ホストがインスタンス化されたVMイメージまたはOSのインストール名。 | `infra-ami-eks-worker-node-7d4ec78312`; `CentOS-8-x86_64-1905` | No |
| `host.image.id` | string | VM イメージ ID。クラウドの場合、この値はプロバイダのものです。 | `ami-07b06b442921831e5` | No |
| `host.image.version` | string | [バージョン属性](README.md#version-attributes)で定義されているVMイメージのバージョン文字列。 | `0.1` | No |

`host.arch` MUST be one of the following or, if none of the listed values apply, a custom value:

| Value  | Description |
|---|---|
| `amd64` | AMD64 |
| `arm32` | ARM32 |
| `arm64` | ARM64 |
| `ia64` | Itanium |
| `ppc32` | 32-bit PowerPC |
| `ppc64` | 64-bit PowerPC |
| `x86` | 32-bit x86 |
<!-- endsemconv -->
