# Cloud

**Status**: [Experimental](../../document-status.md)

**type:** `cloud`

**Description:** A cloud infrastructure (e.g. GCP, Azure, AWS).

<!-- semconv cloud -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `cloud.provider` | string | クラウドプロバイダの名前 | `gcp` | No |
| `cloud.account.id` | string | 異なるエンティティを識別するために使用されるクラウドアカウントID。 | `opentelemetry` | No |
| `cloud.region` | string | 異なるエンティティが実行可能な特定の地理的な場所。 | `us-central1` | No |
| `cloud.zone` | string | ゾーンは、低遅延リンクを介して接続されたRegionのサブセットです。 [1] | `us-central1-a` | No |
| `cloud.infrastructure_service` | string | 使用中のクラウド基盤リソース。 [2] | `aws_ec2`; `azure_vm`; `gcp_compute_engine` | No |

**[1]:** AWSでは、アベイラビリティゾーンと呼ばれます。

**[2]:** サービスのプレフィックスは `cloud.provider` で指定されたものと一致するべきです(SHOULD)。

`cloud.provider` MUST be one of the following or, if none of the listed values apply, a custom value:

| Value  | Description |
|---|---|
| `aws` | Amazon Web Services |
| `azure` | Microsoft Azure |
| `gcp` | Google Cloud Platform |

`cloud.infrastructure_service` MUST be one of the following or, if none of the listed values apply, a custom value:

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
| `gcp_compute_engine` | GCP Compute Engine |
| `gcp_cloud_run` | GCP Cloud Run |
| `gcp_gke` | Google Kubernetes Engine |
| `gcp_cloud_functions` | GCP Cloud Functions |
| `gcp_app_engine` | GCP App Engine |
<!-- endsemconv -->
