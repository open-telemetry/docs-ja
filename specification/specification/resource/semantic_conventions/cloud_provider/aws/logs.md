# AWS Logs

**Status**: [Experimental](../../../../document-status.md)

**Type:** `aws.log`

**Description:** Log attributes for Amazon Web Services.

<!-- semconv aws.log -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `aws.log.group.names` | string[] | アプリケーションが書き込んでいるAWS log groupの名前。 [1] | `[/aws/lambda/my-function, opentelemetry-service]` | No |
| `aws.log.group.arns` | string[] | AWSロググループのAmazon Resource Name(ARN)。 [2] | `[arn:aws:logs:us-west-1:123456789012:log-group:/aws/my/group:*]` | No |
| `aws.log.stream.names` | string[] | アプリケーションが書き込んでいるAWSログストリームの名前。 | `[logs/main/10838bed-421f-43ef-870a-f43feacbbb5b]` | No |
| `aws.log.stream.arns` | string[] | AWSログストリームのARN。 [3] | `[arn:aws:logs:us-west-1:123456789012:log-group:/aws/my/group:log-stream:logs/main/10838bed-421f-43ef-870a-f43feacbbb5b]` | No |

**[1]:** マルチコンテナアプリケーションのように1つのアプリケーションにサイドカーコンテナがあり、それぞれがそれぞれのロググループに書き込むような場合があるため、複数のロググループをサポートする必要があります。

**[2]:** [ロググループ ARN 形式のドキュメント](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/iam-access-control-overview-cwl.html#CWL_ARN_Format) を参照してください。

**[3]:** [ログストリーム ARN 形式のドキュメント](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/iam-access-control-overview-cwl.html#CWL_ARN_Format)を参照してください。1つのロググループには複数のログストリームを含むことができるので、これらのARNは必然的にロググループとログストリームの両方を識別する必要があります。
<!-- endsemconv -->
