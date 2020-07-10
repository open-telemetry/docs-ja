<!--
# Semantic conventions for database client calls
-->

# データベースのクライアントに関するセマンティック規約

<!--
For database client call the `SpanKind` MUST be `Client`.
-->

データベースクライアントを呼び出す場合、`SpanKind` は `Client` でなければなりません(MUST)。

<!--
Span `name` should be set to low cardinality value representing the statement
executed on the database. It may be stored procedure name (without argument), sql
statement without variable arguments, etc. When it's impossible to get any
meaningful representation of the span `name`, it can be populated using the same
value as `db.instance`.
-->

Spanの`name` には、データベース上で実行された文を表すカーディナリティの低い値を設定する必要があります。(引数なしの)ストアドプロシージャ名、変数引数なしのSQL文などが考えられます。Spanの`name` として意味のある表現が得られない場合は、`db.instance` と同じ値を用いて入力することができます。

<!--
Note, Redis, Cassandra, HBase and other storage systems may reuse the same
attribute names.
-->

注意: Redis、Cassandra、HBaseなどのストレージシステムでは、同じ属性名を再利用することがあります。

<!--
| Attribute name | Notes and examples                                           | Required? |
| :------------- | :----------------------------------------------------------- | --------- |
| `db.type`      | Database type. For any SQL database, `"sql"`. For others, the lower-case database category, e.g. `"cassandra"`, `"hbase"`, or `"redis"`. | Yes       |
| `db.instance`  | Database instance name. E.g., In java, if the jdbc.url=`"jdbc:mysql://db.example.com:3306/customers"`, the instance name is `"customers"`. | Yes       |
| `db.statement` | A database statement for the given database type. Note, that the value may be sanitized to exclude sensitive information. E.g., for `db.type="sql"`, `"SELECT * FROM wuser_table"`; for `db.type="redis"`, `"SET mykey 'WuValue'"`. | Yes       |
| `db.user`      | Username for accessing database. E.g., `"readonly_user"` or `"reporting_user"` | No        |
| `db.url`       | JDBC substring like `"mysql://db.example.com:3306"`          | Yes      |
-->

| 属性名 | 説明と例                                       | Required? |
| :------------- | :----------------------------------------------------------- | --------- |
| `db.type`      | データベースの種類。任意のSQLデータベースについては、`"sql"`とします。その他の場合は、小文字のデータベースカテゴリ、例えば `"cassandra"`、 `"hbase"`、 `"redis"`です | Yes       |
| `db.instance`  | データベースのインスタンス名。例えば、javaでは jdbc.url=`"jdbc:mysql://db.example.com:3306/customers"`の場合、インスタンス名は`"customers"`となります。| Yes       |
| `db.statement` | 指定されたデータベースタイプのデータベースステートメント。この値は、機密情報を除外するためにサニタイズされている可能性があることに注意してください。例えば、`db.type="sql"`, `"SELECT * FROM wuser_table"`; for `db.type="redis"`, `"SET mykey 'WuValue''` | Yes       |
| `db.user`      | データベースにアクセスするためのユーザ名。例: `"readonly_user"` または `"reporting_user"` | No        |
| `db.url`       | `"mysql://db.example.com:3306"` のようなJDBC部分文字列          | Yes      |

<!--
Additionally at least one of `net.peer.name` or `net.peer.ip` from the [network attributes][] is required and `net.peer.port` is recommended.
-->

さらに、[ネットワーク属性][]の中の `net.peer.name` または `net.peer.ip` のうちどちらかが必要です。また、`net.peer.port` は推奨です。

<!--
[network attributes]: span-general.md#general-network-connection-attributes
-->

[ネットワーク属性]: span-general.md#general-network-connection-attributes

