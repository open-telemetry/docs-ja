<!--
# AWS Semantic Conventions
-->

# AWS セマンティック規約

<!--
**Status**: [Experimental](../../../../document-status.md)
-->

**Status**: [Experimental](../../../../document-status.md)

<!--
This directory defines standards for resource attributes that only apply to Amazon
Web Services (AWS) resources. If an attribute could apply to resources from more than one cloud
provider (like account ID, operating system, etc), it belongs in the parent
`semantic_conventions` directory.
-->

このディレクトリでは、Amazon Web Services (AWS)のリソースにのみ適用されるリソース属性の標準を定義しています。もしある属性が複数のクラウドプロバイダーのリソースに適用される可能性がある場合(アカウントIDやオペレーティングシステムなど)、その属性は親ディレクトリである `semantic_conventions` に属します。

<!--
## Generic AWS Attributes
-->

## 一般的な AWS 属性

<!--
Attributes that relate to AWS or use AWS-specific terminology, but are used by several
services within AWS or are abstracted away from any particular service:
-->

AWSに関連する属性、またはAWS固有の用語を使用しているが、AWS内の複数のサービスで使用されていたり、特定のサービスから抽象化されていたりする属性。

<!--
- [AWS Logs](./logs.md)
-->

- [AWS Logs](./logs.md)

<!--
## Services
-->

## サービス

<!--
Attributes that relate to an individual AWS service:
-->

個々のAWSサービスに関連する属性。

<!--
- [Elastic Container Service](./ecs.md)
-->

- [Elastic Container Service](./ecs.md)

