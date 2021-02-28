# Function as a Service

**Status**: [Experimental](../../document-status.md)

**type:** `faas`

**Description:** A serverless instance.

<!-- semconv faas_resource -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `faas.name` | string | 実行される関数名 | `my-function` | Yes |
| `faas.id` | string | 実行される関数の一意なID [1] | `arn:aws:lambda:us-west-2:123456789012:function:my-function` | Yes |
| `faas.version` | string | [バージョン属性](.../.../resource/semantic_conventions/README.md#version-attributes)で定義されている、実行される関数のバージョン文字列。 | `2.0.0` | No |
| `faas.instance` | string | 実行環境のID文字列。 | `my-function:instance-0001` | No |
| `faas.max_memory` | number | サーバーレス関数を実行する際のメモリをMiBで記述したもの [2] | `128` | No |

**[1]:** このフィールド例えば、AWS Lambdaではは[ARN](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html)の値に、GCPではリソースのURIに、Azureでは[FunctionDirectory](https://github.com/Azure/azure-functions-host/wiki/Retrieving-information-about-the-currently-running-function)の値に対応しています。

**[2]:** メモリが少なすぎるとJavaのAWS Lambda関数が正常に動作しなくなることがあるので、この属性を設定しておくことをお勧めします。 AWS Lambda上では、環境変数 `AWS_LAMBDA_FUNCTION_MEMORY_SIZE` がこの情報を提供します。
<!-- endsemconv -->

Note: The resource attribute `faas.instance` differs from the span attribute `faas.execution`. For more information see the [Semantic conventions for FaaS spans](../../trace/semantic_conventions/faas.md#difference-between-execution-and-instance).
