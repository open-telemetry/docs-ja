# Cloud

**Status**: [Experimental](../../document-status.md)

**type:** `cloud`

**Description:** A cloud infrastructure (e.g. GCP, Azure, AWS).

<!-- semconv cloud -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `cloud.provider` | string | クラウドプロバイダの名前 | `gcp` | No |
| `cloud.account.id` | string | リソースが割り当てられているクラウドのアカウントID | `111111111111`; `opentelemetry` | No |
| `cloud.region` | string | リソースが動作している地理的な地域。[AWSリージョン](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/)、[Azureリージョン](https://azure.microsoft.com/en-us/global-infrastructure/geographies/)、[Google Cloudリージョン](https://cloud.google.com/about/locations)など、利用可能なリージョンについては、プロバイダーのドキュメントを参照してください。 | `us-central1`; `us-east-1` | No |
| `cloud.availability_zone` | string | クラウドのリージョンは、可用性を高めるために、ゾーンと呼ばれる複数の隔離された場所を持つことが多いです。アベイラビリティゾーンは、リソースが稼働しているゾーンを表します。 [1] | `us-east-1c` | No |
| `cloud.platform` | string | 使用中のクラウドプラットフォームのリソース [2] | `aws_ec2`; `azure_vm`; `gcp_compute_engine` | No |

**[1]:** Google Cloudでは、アベイラビリティー・ゾーンを"Zone"と呼んでいます。

**[2]:** サービスのプレフィックスは `cloud.provider` で指定されたものと一致するべきです(SHOULD)。

`cloud.provider` MUST be one of the following or, if none of the listed values apply, a custom value:

| Value  | Description |
|---|---|
| `aws` | Amazon Web Services |
| `azure` | Microsoft Azure |
| `gcp` | Google Cloud Platform |

`cloud.platform` MUST be one of the following or, if none of the listed values apply, a custom value:

| Value  | Description |
|---|---|
| `aws_ec2` | AWS Elastic Compute Cloud |
| `aws_ecs` | AWS Elastic Container Service |
| `aws_eks` | AWS Elastic Kubernetes Service |
| `aws_lambda` | AWS Lambda |
| `aws_elastic_beanstalk` | AWS Elastic Beanstalk |
| `azure_vm` | Azure Virtual Machines |
| `azure_container_instances` | Azure Container Instances |
| `azure_aks` | Azure Kubernetes Service |
| `azure_functions` | Azure Functions |
| `azure_app_service` | Azure App Service |
| `gcp_compute_engine` | Google Cloud Compute Engine (GCE) |
| `gcp_cloud_run` | Google Cloud Run |
| `gcp_kubernetes_engine` | Google Cloud Kubernetes Engine (GKE) |
| `gcp_cloud_functions` | Google Cloud Functions (GCF) |
| `gcp_app_engine` | Google Cloud App Engine (GAE) |
<!-- endsemconv -->
