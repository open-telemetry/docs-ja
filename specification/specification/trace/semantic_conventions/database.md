<!-- 
# Semantic conventions for database client calls
-->

# データベース・クライアントの呼び出しに関するセマンティック規約

**Status**: [Experimental](../../document-status.md)

<!-- Re-generate TOC with `markdown-toc --no-first-h1 -i` -->

<!-- toc -->

<!-- 
- [Semantic conventions for database client calls](#semantic-conventions-for-database-client-calls)
  - [Connection-level attributes](#connection-level-attributes)
    - [Notes and well-known identifiers for `db.system`](#notes-and-well-known-identifiers-for-dbsystem)
    - [Connection-level attributes for specific technologies](#connection-level-attributes-for-specific-technologies)
  - [Call-level attributes](#call-level-attributes)
    - [Call-level attributes for specific technologies](#call-level-attributes-for-specific-technologies)
      - [Cassandra](#cassandra)
  - [Examples](#examples)
    - [MySQL](#mysql)
    - [Redis](#redis)
    - [MongoDB](#mongodb)
-->

- [Semantic conventions for database client calls](#semantic-conventions-for-database-client-calls)
  - [コネクションレベルの属性](#コネクションレベルの属性)
    - [`db.system`の注意とよく知られた識別子](#db.systemの注意とよく知られた識別子)
    - [特定の技術に対するコネクションレベルの属性](#特定の技術に対するコネクションレベルの属性)
  - [呼び出しレベルの属性](#呼び出しレベルの属性)
    - [特定の技術に対する呼び出しレベルの属性](#特定の技術に対する呼び出しレベルの属性)
      - [Cassandra](#cassandra)
  - [例](#例)
    - [MySQL](#mysql)
    - [Redis](#redis)
    - [MongoDB](#mongodb)

<!-- tocstop -->

<!--
**Span kind:** MUST always be `CLIENT`. 
-->

**Span kind:** 常に `CLIENT` でなければなりません(MUST)

<!-- 
The **span name** SHOULD be set to a low cardinality value representing the statement executed on the database.
It MAY be a stored procedure name (without arguments), DB statement without variable arguments, operation name, etc.
Since SQL statements may have very high cardinality even without arguments, SQL spans SHOULD be named the
following way, unless the statement is known to be of low cardinality:
`<db.operation> <db.name>.<db.sql.table>`, provided that `db.operation` and `db.sql.table` are available.
If `db.sql.table` is not available due to its semantics, the span SHOULD be named `<db.operation> <db.name>`.
It is not recommended to attempt any client-side parsing of `db.statement` just to get these properties,
they should only be used if the library being instrumented already provides them.
When it's otherwise impossible to get any meaningful span name, `db.name` or the tech-specific database name MAY be used.
-->

**span name**には、データベース上で実行されたステートメントを表すカーディナリティの低い値を設定すべきです(SHOULD)。これは、ストアドプロシージャ名(引数なし)、変数引数なしのDB文、操作名などであってもかまいません(MAY)。SQL文は引数がなくても非常に高いカーディナリティを持つ可能性があるので、SQLSpanは、文が低いカーディナリティであることが分かっている場合を除き、次のように命名されるべきです: `<db.operation> <db.name>.<db.sql.table>`(ただし、`db.operation`と`db.sql.table`が利用可能な場合)。もし `db.sql.table` がそのセマンティクスのために利用できない場合は、`<db.operation> <db.name>` という名前にすべきです(SHOULD)。これらのプロパティを取得するために、クライアントサイドで `db.statement` の解析を試みることは推奨されません。これらのプロパティは、計装対象のライブラリが既に提供している場合にのみ使用する必要があります。意味のあるSpan名を得ることができない場合には、`db.name` または技術固有のデータベース名を使用しても構いません(MAY)。

<!--
## Connection-level attributes
-->

## コネクションレベルの属性

<!--
These attributes will usually be the same for all operations performed over the same database connection.
Some database systems may allow a connection to switch to a different `db.user`, for example, and other database systems may not even have the concept of a connection at all.
-->

これらの属性は通常、同じデータベース接続で実行されるすべての操作で同じになります。データベースシステムによっては、例えば、接続を別の`db.user`に切り替えることができる場合もありますし、接続という概念が全くないデータベースシステムもあります。

<!-- semconv db(tag=connection-level) -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `db.system` | string | 使用しているデータベース管理システム(DBMS)製品の識別子。よく知られた識別子のリストは以下を参照してください。 | `other_sql` | Yes |
| `db.connection_string` | string | データベースへの接続に使用される接続文字列です。埋め込まれた資格情報を削除することを推奨します。 | `Server=(localdb)\v11.0;Integrated Security=true;` | No |
| `db.user` | string | データベースにアクセスするためのユーザー名。 | `readonly_user`; `reporting_user` | No |
| [`net.peer.ip`](span-general.md) | string | 相手のリモートアドレス(IPv4ではドット10進数、IPv6では[RFC5952](https://tools.ietf.org/html/rfc5952) | `127.0.0.1` | 下記参照 |
| [`net.peer.name`](span-general.md) | string | リモートのホスト名あるいは類似の文字列。下記注釈参照 | `example.com` | 下記参照 |
| [`net.peer.port`](span-general.md) | int | リモートのポート番号 | `80`; `8080`; `443` | このDBMSのデフォルトポート以外のポートを使用している場合は必須。 |
| [`net.transport`](span-general.md) | string | Transport protocol used. 下記注釈参照。 | `ip_tcp` | 一般的に推奨、プロセス中のデータベース(`"inproc"`)には必須。 |

**Additional attribute requirements:** At least one of the following sets of attributes is required:

* [`net.peer.name`](span-general.md)
* [`net.peer.ip`](span-general.md)

`db.system` MUST be one of the following or, if none of the listed values apply, a custom value:

| Value  | Description |
|---|---|
| `other_sql` | 他のSQLデータベース。フォールバックのみ。注意事項を参照してください。 |
| `mssql` | Microsoft SQL Server |
| `mysql` | MySQL |
| `oracle` | Oracle Database |
| `db2` | IBM Db2 |
| `postgresql` | PostgreSQL |
| `redshift` | Amazon Redshift |
| `hive` | Apache Hive |
| `cloudscape` | Cloudscape |
| `hsqldb` | HyperSQL DataBase |
| `progress` | Progress Database |
| `maxdb` | SAP MaxDB |
| `hanadb` | SAP HANA |
| `ingres` | Ingres |
| `firstsql` | FirstSQL |
| `edb` | EnterpriseDB |
| `cache` | InterSystems Caché |
| `adabas` | Adabas (Adaptable Database System) |
| `firebird` | Firebird |
| `derby` | Apache Derby |
| `filemaker` | FileMaker |
| `informix` | Informix |
| `instantdb` | InstantDB |
| `interbase` | InterBase |
| `mariadb` | MariaDB |
| `netezza` | Netezza |
| `pervasive` | Pervasive PSQL |
| `pointbase` | PointBase |
| `sqlite` | SQLite |
| `sybase` | Sybase |
| `teradata` | Teradata |
| `vertica` | Vertica |
| `h2` | H2 |
| `coldfusion` | ColdFusion IMQ |
| `cassandra` | Apache Cassandra |
| `hbase` | Apache HBase |
| `mongodb` | MongoDB |
| `redis` | Redis |
| `couchbase` | Couchbase |
| `couchdb` | CouchDB |
| `cosmosdb` | Microsoft Azure Cosmos DB |
| `dynamodb` | Amazon DynamoDB |
| `neo4j` | Neo4j |
| `geode` | Apache Geode |
| `elasticsearch` | Elasticsearch |
<!-- endsemconv -->

<!--
### Notes and well-known identifiers for `db.system`
-->

### `db.system`の注意とよく知られた識別子

<!-- The list above is a non-exhaustive list of well-known identifiers to be specified for `db.system`.
-->

上記のリストは、`db.system`に指定すべき有名な識別子の非網羅的なリストです。

<!--
If a value defined in this list applies to the DBMS to which the request is sent, this value MUST be used.
If no value defined in this list is suitable, a custom value MUST be provided.
This custom value MUST be the name of the DBMS in lowercase and without a version number to stay consistent with existing identifiers.
-->

このリストに定義されている値が、リクエストが送信されたDBMSに適用される場合、この値を使用しなければなりません(MUST)。このリストで定義された値が適切でない場合は、カスタム値を提供しなければなりません(MUST)。このカスタム値は、既存の識別子との整合性を保つために、小文字でバージョン番号を含まないDBMSの名前でなければなりません(MUST)。

<!--
It is encouraged to open a PR towards this specification to add missing values to the list, especially when instrumentations for those missing databases are written.
This allows multiple instrumentations for the same database to be aligned and eases analyzing for backends.
-->

足りてない値をリストに追加するために、この仕様に向けてPRを行うことが推奨されます。特に、これらの足りていない値を持つデータベースのための計装が作成された場合には、そのようにしてください。これにより、同じデータベースに対する複数の計装を揃えることができ、バックエンドの分析が容易になります。

<!--
The value `other_sql` is intended as a fallback and MUST only be used if the DBMS is known to be SQL-compliant but the concrete product is not known to the instrumentation.
If the concrete DBMS is known to the instrumentation, its specific identifier MUST be used.
-->

値 `other_sql` はフォールバックを目的としており、DBMS が SQL に準拠していることはわかっているが、具体的な製品が計装に知られていない場合にのみ使用しなければなりません(MUST)。具体的なDBMSが計装に知られている場合は、その具体的な識別子を使用しなければなりません(MUST)。

<!--
Back ends could, for example, use the provided identifier to determine the appropriate SQL dialect for parsing the `db.statement`.
-->

バックエンドは、例えば、提供された識別子を使って、`db.statement`を解析するための適切なSQL方言(SQL dialect)を決定することができます。

<!--
When additional attributes are added that only apply to a specific DBMS, its identifier SHOULD be used as a namespace in the attribute key as for the attributes in the sections below.
-->

特定のDBMSにのみ適用される属性が追加された場合、その識別子は、以下のセクションの属性と同様に、属性キーの名前空間として使用されるべきです(SHOULD)。

<!--
### Connection-level attributes for specific technologies
-->

### 特定の技術に対するコネクションレベルの属性

<!-- semconv db.mssql(tag=connection-level-tech-specific,remove_constraints) -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `db.jdbc.driver_classname` | string | 接続に使用する[Java Database Connectivity (JDBC)](https://docs.oracle.com/javase/8/docs/technotes/guides/jdbc/)ドライバの完全修飾クラス名。 | `org.postgresql.Driver`; `com.microsoft.sqlserver.jdbc.SQLServerDriver` | No |
| `db.mssql.instance_name` | string | 接続しているMicrosoft SQL Serverの[インスタンス名](https://docs.microsoft.com/en-us/sql/connect/jdbc/building-the-connection-url?view=sql-server-ver15)です。この名前は、名前付きインスタンスのポートを決定するために使用されます。 [1] | `MSSQLSERVER` | No |

**[1]:** `db.mssql.instance_name`を設定する場合、`net.peer.port`は必須ではなくなりました(ただし、非標準の場合は推奨)
<!-- endsemconv -->

## 呼び出しレベルの属性

<!-- These attributes may be different for each operation performed, even if the same connection is used for multiple operations.
Usually only one `db.name` will be used per connection though.
-->

These attributes may be different for each operation performed, even if the same connection is used for multiple operations. Usually only one `db.name` will be used per connection though.

<!-- semconv db(tag=call-level,remove_constraints) -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `db.name` | string | もし[その技術特有の属性](#call-level-attributes-for-specific-technologies)が定義されていない場合、この属性はアクセスされているデータベースの名前を報告するために使用されます。データベースを切り替えるコマンドのために、これは(コマンドが失敗した場合でも)ターゲットデータベースに設定しなければなりません。 [1] | `customers`; `main` | 該当し、他に適切な属性が定義されていない場合は必須 |
| `db.statement` | string | 実行されているデータベース文。 [2] | `SELECT * FROM wuser_table`; `SET mykey "WuValue"` | 該当する場合で、Metricsの設定で明示的に無効化されていない場合は必須 |
| `db.operation` | string | 実行する操作の名前。例えば、`findAndModify` のような [MongoDB コマンド名](https://docs.mongodb.com/manual/reference/command/#database-operations) や SQL キーワード。 [3] | `findAndModify`; `HMSET`; `SELECT` | Required, if `db.statement` is not applicable. |

**[1]:** 一部のSQLデータベースでは、使用するデータベース名を「スキーマ名」と呼びます。

**[2]:** この値は、機密情報を除外するためにサニタイズされている場合があります。

**[3]:** これをSQLキーワードに設定する場合、このプロパティを取得するためだけに `db.statement` のクライアント側の解析を試みることは推奨されません。しかし、測定対象のライブラリから操作名が提供されている場合は設定する必要があります。SQL文に曖昧な操作があったり、複数の操作を実行したりする場合は、この値を省略しても構いません。
<!-- endsemconv -->

<!-- For **Redis**, the value provided for `db.statement` SHOULD correspond to the syntax of the Redis CLI.
If, for example, the [`HMSET` command][] is invoked, `"HMSET myhash field1 'Hello' field2 'World'"` would be a suitable value for `db.statement`.
-->

**Redis**の場合、`db.statement`に指定される値は、Redis CLIの構文に対応するべきです(SHOULD)。例えば、[`HMSET`コマンド][]が起動された場合、`"HMSET myhash field1 'Hello' field2 'World'"`が`db.statement`の適切な値となります。

<!-- [`HMSET` command]: https://redis.io/commands/hmset
-->

[`HMSET`コマンド]: https://redis.io/commands/hmset

<!-- In **CouchDB**, `db.operation` should be set to the HTTP method + the target REST route according to the API reference documentation.
For example, when retrieving a document, `db.operation` would be set to (literally, i.e., without replacing the placeholders with concrete values): [`GET /{db}/{docid}`][CouchDB get doc].
-->

**CouchDB**では、`db.operation`に、APIリファレンスドキュメントに記載されているHTTPメソッド+対象となるRESTルートを設定する必要があります。例えば、ドキュメントを取得する場合、`db.operation`は以下のように設定されます(文字通り、つまりプレースホルダーを具体的な値に置き換えずに、です)。[`GET /{db}/{docid}`][CouchDB get doc].

<!--
[CouchDB get doc]: http://docs.couchdb.org/en/stable/api/document/common.html#get--db-docid
-->

[CouchDB get doc]: http://docs.couchdb.org/en/stable/api/document/common.html#get--db-docid


<!--
### Call-level attributes for specific technologies
-->

### 特定の技術に対する呼び出しレベルの属性

<!-- semconv db.tech(tag=call-level-tech-specific) -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `db.hbase.namespace` | string | アクセスされている[HBase 名前空間](https://hbase.apache.org/book.html#_namespace)です。一般的な `db.name` 属性の代わりに使用します。 | `default` | Yes |
| `db.redis.database_index` | int | [`SELECT`コマンド](https://redis.io/commands/select)で使われる、アクセスするデータベースのインデックスを整数で指定します。一般的な `db.name` 属性の代わりに使用されます。 | `0`; `1`; `15` | デフォルトのデータベース(`0`)以外の場合は必須です。 |
| `db.mongodb.collection` | string | `db.name`で指定されたデータベース内でアクセスされるコレクションです。 | `customers`; `products` | Yes |
| `db.sql.table` | string | 操作の対象となる主テーブルの名前で、該当する場合スキーマ名も含まれます。 [1] | `public.users`; `customers` | 使用可能であれば推奨されます。 |

**[1]:** このプロパティを取得するために、クライアントサイドで `db.statement` の解析を行うことはお勧めできませんが、計装対象のライブラリで提供されている場合には設定する必要があります。匿名のテーブルや複数のテーブルを操作する場合には、この値を設定してはいけません(MUST NOT)。
<!-- endsemconv -->

#### Cassandra

<!--
Separated for clarity.
-->

明確にするために分離しました。

<!-- semconv db.tech(tag=call-level-tech-specific-cassandra) -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `db.cassandra.keyspace` | string | アクセスするキースペースの名前です。一般的な `db.name` 属性の代わりに使用されます。 | `mykeyspace` | Yes |
| `db.cassandra.page_size` | int | ページングに使用されるフェッチサイズ、つまり一度に何行を返すかを指定します。 | `5000` | No |
| `db.cassandra.consistency_level` | string | クエリの一貫性レベル。[CQL](https://docs.datastax.com/en/cassandra-oss/3.0/cassandra/dml/dmlConfigConsistency.html)の整合性値に基づいています。 | `all` | No |
| `db.cassandra.table` | string | 操作の対象となる主テーブルの名前で、(該当する場合)スキーマ名も含まれます。 [1] | `mytable` | Recommended if available. |
| `db.cassandra.idempotence` | boolean | クエリがべき等であるかどうか。 |  | No |
| `db.cassandra.speculative_execution_count` | int | クエリが投機的に実行された回数です。クエリが投機的に実行されなかった場合は、設定されないか、または `0` となります。 | `0`; `2` | No |
| `db.cassandra.coordinator.id` | string | クエリのコーディネーションノードのIDです。 | `be13faa2-8574-4d71-926d-27f16cf8a7af` | No |
| `db.cassandra.coordinator.dc` | string | クエリのコーディネーションノードのデータセンターです。 | `us-west-2` | No |

**[1]:** これは、db.sql.table属性を反映していますが、sqlではなくcassandraを参照しています。この属性を取得するためだけにクライアントサイドで `db.statement` の解析を行うことは推奨されませんが、計装対象のライブラリで提供されている場合は設定する必要があります。操作が匿名のテーブルや複数のテーブルに対して行われる場合、この値は設定してはいけません(MUST NOT)。
<!-- endsemconv -->

## 例

### MySQL

| Key | Value |
| :---------------------- | :----------------------------------------------------------- |
| Span name               | `"SELECT ShopDb.orders"` |
| `db.system`             | `"mysql"` |
| `db.connection_string`  | `"Server=shopdb.example.com;Database=ShopDb;Uid=billing_user;TableCache=true;UseCompression=True;MinimumPoolSize=10;MaximumPoolSize=50;"` |
| `db.user`               | `"billing_user"` |
| `net.peer.name`         | `"shopdb.example.com"` |
| `net.peer.ip`           | `"192.0.2.12"` |
| `net.peer.port`         | `3306` |
| `net.transport`         | `"IP.TCP"` |
| `db.name`               | `"ShopDb"` |
| `db.statement`          | `"SELECT * FROM orders WHERE order_id = 'o4711'"` |
| `db.operation`          | `"SELECT"` |
| `db.sql.table`          | `"orders"` |

### Redis

<!-- In this example, Redis is connected using a unix domain socket and therefore the connection string and `net.peer.ip` are left out.
Furthermore, `db.name` is not specified as there is no database name in Redis and `db.redis.database_index` is set instead.
-->

この例では、Redisはunixドメインソケットを使って接続しているので、接続文字列と`net.peer.ip`は省略しています。また、Redisにはデータベース名が存在しないため、`db.name`を指定せず、代わりに`db.redis.database_index`を設定しています。

| Key | Value |
| :------------------------ | :-------------------------------------------- |
| Span name                 | `"HMSET myhash"` |
| `db.system`               | `"redis"` |
| `db.connection_string`    | not set |
| `db.user`                 | not set |
| `net.peer.name`           | `"/tmp/redis.sock"` |
| `net.transport`           | `"Unix"` |
| `db.name`                 | not set |
| `db.statement`            | `"HMSET myhash field1 'Hello' field2 'World"` |
| `db.operation`            | not set |
| `db.redis.database_index` | `15` |

### MongoDB

| Key | Value |
| :---------------------- | :----------------------------------------------------------- |
| Span name               | `"products.findAndModify"` |
| `db.system`             | `"mongodb"` |
| `db.connection_string`  | not set |
| `db.user`               | `"the_user"` |
| `net.peer.name`         | `"mongodb0.example.com"` |
| `net.peer.ip`           | `"192.0.2.14"` |
| `net.peer.port`         | `27017` |
| `net.transport`         | `"IP.TCP"` |
| `db.name`               | `"shopDb"` |
| `db.statement`          | not set |
| `db.operation`          | `"findAndModify"` |
| `db.mongodb.collection` | `"products"` |
