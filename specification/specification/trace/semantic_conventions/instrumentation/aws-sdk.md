<!--
# Semantic conventions for AWS SDK
-->

# AWS SDKのセマンティック規約


**Status**: [Experimental](../../../document-status.md)

<!--
This document defines semantic conventions to apply when instrumenting the AWS SDK. They map request or response
parameters in AWS SDK API calls to attributes on a Span. The conventions have been collected over time based
on feedback from AWS users of tracing and will continue to increase as new interesting conventions
are found.
-->

このドキュメントでは、AWS SDKを計装する際に適用するセマンティック規約を定義しています。これらは、AWS SDKのAPIコールにおけるリクエストやレスポンスのパラメータを、Spanの属性にマッピングします。この規約は、AWSのトレースユーザーからのフィードバックに基づいて時間をかけて収集されたものであり、新たに興味深い規約が発見された場合には、今後も増加していく予定です。

<!--
Some descriptions are also provided for populating general OpenTelemetry semantic conventions based on these APIs.
-->

また、これらのAPIに基づいてOpenTelemetryの一般的なセマンティック規約を作成するための説明もあります。

<!--
## Common Attributes
-->

## 共通の属性

<!--
The span name MUST be of the format `Service.Operation` as per the AWS HTTP API, e.g., `DynamoDB.GetItem`,
`S3.ListBuckets`. This is equivalent to concatenating `rpc.service` and `rpc.method` with `.` and consistent
with the naming guidelines for RPC client spans.
-->

Spanの名前は、AWS HTTP APIのように、`Service.Operation`というフォーマットでなければなりません(MUST)(例:`DynamoDB.GetItem`、`S3.ListBuckets`など)。これは `rpc.service` と `rpc.method` を `.` で連結したものと同じで、RPC クライアントSpanのネーミングガイドラインと一致しています。


<!-- semconv aws -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| [`rpc.method`](../rpc.md) | string | AWS SDKから返された、リクエストに対応するオペレーションの名前 | `GetItem`; `PutItem` | No |
| [`rpc.service`](../rpc.md) | string | AWS SDKから返される、リクエスト先のサービス名 | `DynamoDB`; `S3` | No |
| [`rpc.system`](../rpc.md) | string | `aws-api` の値 | `aws-api` | Yes |
<!-- endsemconv -->

## DynamoDB

### 共通の属性

<!--
These attributes are filled in for all DynamoDB request types.
-->

これらの属性は、すべてのDynamoDBリクエストタイプに記入されます。


<!-- semconv dynamodb.all -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| [`db.system`](../database.md) | string | `dynamodb`の値 | `dynamodb` | Yes |
<!-- endsemconv -->

### DynamoDB.BatchGetItem

<!-- semconv dynamodb.batchgetitem -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `aws.dynamodb.consumed_capacity` | string[] | `ConsumedCapacity`レスポンスフィールドの各項目をJSONでシリアライズした値 | `[{ "CapacityUnits": number, "GlobalSecondaryIndexes": { "string" : { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number } }, "LocalSecondaryIndexes": { "string" : { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number } }, "ReadCapacityUnits": number, "Table": { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number }, "TableName": "string", "WriteCapacityUnits": number }]` | No |
| `aws.dynamodb.table_names` | string[] | RequestItems` オブジェクトフィールドのキーです。 | `[Users, Cats]` | No |
<!-- endsemconv -->

### DynamoDB.BatchWriteItem

<!-- semconv dynamodb.batchwriteitem -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `aws.dynamodb.consumed_capacity` | string[] | `ConsumedCapacity`レスポンスフィールドの各項目をJSONでシリアライズした値 | `[{ "CapacityUnits": number, "GlobalSecondaryIndexes": { "string" : { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number } }, "LocalSecondaryIndexes": { "string" : { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number } }, "ReadCapacityUnits": number, "Table": { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number }, "TableName": "string", "WriteCapacityUnits": number }]` | No |
| `aws.dynamodb.item_collection_metrics` | string | `ItemCollectionMetrics` レスポンスフィールドの JSON 形式の値 | `{ "string" : [ { "ItemCollectionKey": { "string" : { "B": blob, "BOOL": boolean, "BS": [ blob ], "L": [ "AttributeValue" ], "M": { "string" : "AttributeValue" }, "N": "string", "NS": [ "string" ], "NULL": boolean, "S": "string", "SS": [ "string" ] } }, "SizeEstimateRangeGB": [ number ] } ] }` | No |
| `aws.dynamodb.table_names` | string[] | RequestItems` オブジェクトフィールドのキーです。 | `[Users, Cats]` | No |
<!-- endsemconv -->

### DynamoDB.CreateTable

<!-- semconv dynamodb.createtable -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `aws.dynamodb.global_secondary_indexes` | string[] | リクエストフィールド `GlobalSecondaryIndexes`の各項目のJSON形式の値 | `[{ "IndexName": "string", "KeySchema": [ { "AttributeName": "string", "KeyType": "string" } ], "Projection": { "NonKeyAttributes": [ "string" ], "ProjectionType": "string" }, "ProvisionedThroughput": { "ReadCapacityUnits": number, "WriteCapacityUnits": number } }]` | No |
| `aws.dynamodb.local_secondary_indexes` | string[] | `LocalSecondaryIndexes`リクエストフィールドの各項目のJSON形式の値 | `[{ "IndexArn": "string", "IndexName": "string", "IndexSizeBytes": number, "ItemCount": number, "KeySchema": [ { "AttributeName": "string", "KeyType": "string" } ], "Projection": { "NonKeyAttributes": [ "string" ], "ProjectionType": "string" } }]` | No |
| `aws.dynamodb.consumed_capacity` | string[] | `ConsumedCapacity`レスポンスフィールドの各項目をJSONでシリアライズした値 | `[{ "CapacityUnits": number, "GlobalSecondaryIndexes": { "string" : { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number } }, "LocalSecondaryIndexes": { "string" : { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number } }, "ReadCapacityUnits": number, "Table": { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number }, "TableName": "string", "WriteCapacityUnits": number }]` | No |
| `aws.dynamodb.item_collection_metrics` | string | `ItemCollectionMetrics` レスポンスフィールドの JSON 形式の値 | `{ "string" : [ { "ItemCollectionKey": { "string" : { "B": blob, "BOOL": boolean, "BS": [ blob ], "L": [ "AttributeValue" ], "M": { "string" : "AttributeValue" }, "N": "string", "NS": [ "string" ], "NULL": boolean, "S": "string", "SS": [ "string" ] } }, "SizeEstimateRangeGB": [ number ] } ] }` | No |
| `aws.dynamodb.provisioned_read_capacity` | double | `ProvisionedThroughput.ReadCapacityUnits`リクエストパラメーターの値 | `1.0`; `2.0` | No |
| `aws.dynamodb.provisioned_write_capacity` | double | `ProvisionedThroughput.WriteCapacityUnits`リクエストパラメーターの値 | `1.0`; `2.0` | No |
| `aws.dynamodb.table_names` | string[] | TableNameリクエストパラメータの値を持つ1要素の配列 | `[Users]` | No |
<!-- endsemconv -->

### DynamoDB.DeleteItem

<!-- semconv dynamodb.deleteitem -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `aws.dynamodb.consumed_capacity` | string[] | `ConsumedCapacity`レスポンスフィールドの各項目をJSONでシリアライズした値 | `[{ "CapacityUnits": number, "GlobalSecondaryIndexes": { "string" : { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number } }, "LocalSecondaryIndexes": { "string" : { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number } }, "ReadCapacityUnits": number, "Table": { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number }, "TableName": "string", "WriteCapacityUnits": number }]` | No |
| `aws.dynamodb.item_collection_metrics` | string | `ItemCollectionMetrics` レスポンスフィールドの JSON 形式の値 | `{ "string" : [ { "ItemCollectionKey": { "string" : { "B": blob, "BOOL": boolean, "BS": [ blob ], "L": [ "AttributeValue" ], "M": { "string" : "AttributeValue" }, "N": "string", "NS": [ "string" ], "NULL": boolean, "S": "string", "SS": [ "string" ] } }, "SizeEstimateRangeGB": [ number ] } ] }` | No |
| `aws.dynamodb.table_names` | string[] | TableNameリクエストパラメータの値を持つ1要素の配列 | `[Users]` | No |
<!-- endsemconv -->

### DynamoDB.DeleteTable

<!-- semconv dynamodb.deletetable -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `aws.dynamodb.table_names` | string[] | TableNameリクエストパラメータの値を持つ1要素の配列 | `[Users]` | No |
<!-- endsemconv -->

### DynamoDB.DescribeTable

<!-- semconv dynamodb.describetable -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `aws.dynamodb.table_names` | string[] | TableNameリクエストパラメータの値を持つ1要素の配列 | `[Users]` | No |
<!-- endsemconv -->

### DynamoDB.GetItem

<!-- semconv dynamodb.getitem -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `aws.dynamodb.consistent_read` | boolean | `ConsistentRead` リクエストパラメーターの値 |  | No |
| `aws.dynamodb.consumed_capacity` | string[] | `ConsumedCapacity`レスポンスフィールドの各項目をJSONでシリアライズした値 | `[{ "CapacityUnits": number, "GlobalSecondaryIndexes": { "string" : { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number } }, "LocalSecondaryIndexes": { "string" : { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number } }, "ReadCapacityUnits": number, "Table": { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number }, "TableName": "string", "WriteCapacityUnits": number }]` | No |
| `aws.dynamodb.projection` | string | `ProjectionExpression` リクエストパラメータの値 | `Title`; `Title, Price, Color`; `Title, Description, RelatedItems, ProductReviews` | No |
| `aws.dynamodb.table_names` | string[] | TableNameリクエストパラメータの値を持つ1要素の配列 | `[Users]` | No |
<!-- endsemconv -->

### DynamoDB.ListTables

<!-- semconv dynamodb.listtables -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `aws.dynamodb.exclusive_start_table` | string | `ExclusiveStartTableName`のリクエストパラメータの値 | `Users`; `CatsTable` | No |
| `aws.dynamodb.table_count` | int | `TableNames` レスポンスパラメーターのアイテム数 | `20` | No |
| `aws.dynamodb.limit` | int | `Limit` リクエストパラメータの値 | `10` | No |
<!-- endsemconv -->

### DynamoDB.PutItem

<!-- semconv dynamodb.putitem -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `aws.dynamodb.consumed_capacity` | string[] | `ConsumedCapacity`レスポンスフィールドの各項目をJSONでシリアライズした値 | `[{ "CapacityUnits": number, "GlobalSecondaryIndexes": { "string" : { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number } }, "LocalSecondaryIndexes": { "string" : { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number } }, "ReadCapacityUnits": number, "Table": { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number }, "TableName": "string", "WriteCapacityUnits": number }]` | No |
| `aws.dynamodb.item_collection_metrics` | string | `ItemCollectionMetrics` レスポンスフィールドの JSON 形式の値 | `{ "string" : [ { "ItemCollectionKey": { "string" : { "B": blob, "BOOL": boolean, "BS": [ blob ], "L": [ "AttributeValue" ], "M": { "string" : "AttributeValue" }, "N": "string", "NS": [ "string" ], "NULL": boolean, "S": "string", "SS": [ "string" ] } }, "SizeEstimateRangeGB": [ number ] } ] }` | No |
| `aws.dynamodb.table_names` | string[] | RequestItems` オブジェクトフィールドのキーです。 | `[Users, Cats]` | No |
<!-- endsemconv -->

### DynamoDB.Query

<!-- semconv dynamodb.query -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `aws.dynamodb.scan_forward` | boolean | `ScanIndexForward` リクエストパラメーターの値 |  | No |
| `aws.dynamodb.attributes_to_get` | string[] | `AttributesToGet` リクエストパラメータの値 | `[lives, id]` | No |
| `aws.dynamodb.consistent_read` | boolean | `ConsistentRead` リクエストパラメーターの値 |  | No |
| `aws.dynamodb.consumed_capacity` | string[] | `ConsumedCapacity`レスポンスフィールドの各項目をJSONでシリアライズした値 | `[{ "CapacityUnits": number, "GlobalSecondaryIndexes": { "string" : { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number } }, "LocalSecondaryIndexes": { "string" : { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number } }, "ReadCapacityUnits": number, "Table": { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number }, "TableName": "string", "WriteCapacityUnits": number }]` | No |
| `aws.dynamodb.index_name` | string | `IndexName` リクエストパラメータの値 | `name_to_group` | No |
| `aws.dynamodb.limit` | int | `Limit` リクエストパラメータの値 | `10` | No |
| `aws.dynamodb.projection` | string | `ProjectionExpression` リクエストパラメータの値 | `Title`; `Title, Price, Color`; `Title, Description, RelatedItems, ProductReviews` | No |
| `aws.dynamodb.select` | string | `Select` リクエストパラメータの値 | `ALL_ATTRIBUTES`; `COUNT` | No |
| `aws.dynamodb.table_names` | string[] | TableNameリクエストパラメータの値を持つ1要素の配列 | `[Users]` | No |
<!-- endsemconv -->

### DynamoDB.Scan

<!-- semconv dynamodb.scan -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `aws.dynamodb.segment` | int | `Segment` リクエストパラメータの値 | `10` | No |
| `aws.dynamodb.total_segments` | int | `TotalSegments` リクエストパラメータの値 | `100` | No |
| `aws.dynamodb.count` | int | `Count` レスポンスパラメータの値 | `10` | No |
| `aws.dynamodb.scanned_count` | int | `ScannedCount` レスポンスパラメータの値 | `50` | No |
| `aws.dynamodb.attributes_to_get` | string[] | `AttributesToGet` リクエストパラメータの値 | `[lives, id]` | No |
| `aws.dynamodb.consistent_read` | boolean | `ConsistentRead` リクエストパラメーターの値 |  | No |
| `aws.dynamodb.consumed_capacity` | string[] | `ConsumedCapacity`レスポンスフィールドの各項目をJSONでシリアライズした値 | `[{ "CapacityUnits": number, "GlobalSecondaryIndexes": { "string" : { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number } }, "LocalSecondaryIndexes": { "string" : { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number } }, "ReadCapacityUnits": number, "Table": { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number }, "TableName": "string", "WriteCapacityUnits": number }]` | No |
| `aws.dynamodb.index_name` | string | `IndexName` リクエストパラメータの値 | `name_to_group` | No |
| `aws.dynamodb.limit` | int | `Limit` リクエストパラメータの値 | `10` | No |
| `aws.dynamodb.projection` | string | `ProjectionExpression` リクエストパラメータの値 | `Title`; `Title, Price, Color`; `Title, Description, RelatedItems, ProductReviews` | No |
| `aws.dynamodb.select` | string | `Select` リクエストパラメータの値 | `ALL_ATTRIBUTES`; `COUNT` | No |
| `aws.dynamodb.table_names` | string[] | TableNameリクエストパラメータの値を持つ1要素の配列 | `[Users]` | No |
<!-- endsemconv -->

### DynamoDB.UpdateItem

<!-- semconv dynamodb.updateitem -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `aws.dynamodb.consumed_capacity` | string[] | `ConsumedCapacity`レスポンスフィールドの各項目をJSONでシリアライズした値 | `[{ "CapacityUnits": number, "GlobalSecondaryIndexes": { "string" : { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number } }, "LocalSecondaryIndexes": { "string" : { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number } }, "ReadCapacityUnits": number, "Table": { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number }, "TableName": "string", "WriteCapacityUnits": number }]` | No |
| `aws.dynamodb.item_collection_metrics` | string | `ItemCollectionMetrics` レスポンスフィールドの JSON 形式の値 | `{ "string" : [ { "ItemCollectionKey": { "string" : { "B": blob, "BOOL": boolean, "BS": [ blob ], "L": [ "AttributeValue" ], "M": { "string" : "AttributeValue" }, "N": "string", "NS": [ "string" ], "NULL": boolean, "S": "string", "SS": [ "string" ] } }, "SizeEstimateRangeGB": [ number ] } ] }` | No |
| `aws.dynamodb.table_names` | string[] | TableNameリクエストパラメータの値を持つ1要素の配列 | `[Users]` | No |
<!-- endsemconv -->

### DynamoDB.UpdateTable

<!-- semconv dynamodb.updatetable -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `aws.dynamodb.attribute_definitions` | string[] | `AttributeDefinitions`のリクエストフィールドの各項目のJSON形式の値 | `[{ "AttributeName": "string", "AttributeType": "string" }]` | No |
| `aws.dynamodb.global_secondary_index_updates` | string[] | `GlobalSecondaryIndexUpdates`リクエストフィールドの各項目のJSON形式の値 | `[{ "Create": { "IndexName": "string", "KeySchema": [ { "AttributeName": "string", "KeyType": "string" } ], "Projection": { "NonKeyAttributes": [ "string" ], "ProjectionType": "string" }, "ProvisionedThroughput": { "ReadCapacityUnits": number, "WriteCapacityUnits": number } }]` | No |
| `aws.dynamodb.consumed_capacity` | string[] | `ConsumedCapacity`レスポンスフィールドの各項目をJSONでシリアライズした値 | `[{ "CapacityUnits": number, "GlobalSecondaryIndexes": { "string" : { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number } }, "LocalSecondaryIndexes": { "string" : { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number } }, "ReadCapacityUnits": number, "Table": { "CapacityUnits": number, "ReadCapacityUnits": number, "WriteCapacityUnits": number }, "TableName": "string", "WriteCapacityUnits": number }]` | No |
| `aws.dynamodb.provisioned_read_capacity` | double | `ProvisionedThroughput.ReadCapacityUnits`リクエストパラメーターの値 | `1.0`; `2.0` | No |
| `aws.dynamodb.provisioned_write_capacity` | double | `ProvisionedThroughput.WriteCapacityUnits`リクエストパラメーターの値 | `1.0`; `2.0` | No |
| `aws.dynamodb.table_names` | string[] | TableNameリクエストパラメータの値を持つ1要素の配列 | `[Users]` | No |
<!-- endsemconv -->
