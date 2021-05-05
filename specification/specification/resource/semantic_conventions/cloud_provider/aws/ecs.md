# AWS ECS

**Status**: [Experimental](../../../../document-status.md)

**type:** `aws.ecs`

**Description:** AWS Elastic Container Service(ECS)が使用するリソースです。

<!-- semconv aws.ecs -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `aws.ecs.container.arn` | string | [ECSコンテナインスタンス](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_instances.html)のAmazon Resource Name(ARN)。| `arn:aws:ecs:us-west-1:123456789123:container/32624152-9086-4f0e-acae-1a75b14fe4d9` | No |
| `aws.ecs.cluster.arn` | string | [ECSクラスタ](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/clusters.html)のARN。| `arn:aws:ecs:us-west-2:123456789123:cluster/my-cluster` | No |
| `aws.ecs.launchtype` | string | ECSタスクの[起動タイプ](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/launch_types.html)。| `ec2`; `fargate` | No |
| `aws.ecs.task.arn` | string | [ECSタスク定義](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definitions.html)のARN。| `arn:aws:ecs:us-west-1:123456789123:task/10838bed-421f-43ef-870a-f43feacbbb5b` | No |
| `aws.ecs.task.family` | string | このタスク定義が属するタスク定義ファミリ。| `opentelemetry-family` | No |

`aws.ecs.launchtype` MUST be one of the following:

| Value  | Description |
|---|---|
| `ec2` | ec2 |
| `fargate` | fargate |
<!-- endsemconv -->
