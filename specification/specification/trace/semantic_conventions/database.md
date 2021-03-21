# Semantic conventions for database client calls

**Status**: [Experimental](../../document-status.md)

<!-- Re-generate TOC with `markdown-toc --no-first-h1 -i` -->

<!-- toc -->

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

<!-- tocstop -->

**Span kind:** MUST always be `CLIENT`.

The **span name** SHOULD be set to a low cardinality value representing the statement executed on the database.
It MAY be a stored procedure name (without arguments), DB statement without variable arguments, operation name, etc.
Since SQL statements may have very high cardinality even without arguments, SQL spans SHOULD be named the
following way, unless the statement is known to be of low cardinality:
`<db.operation> <db.name>.<db.sql.table>`, provided that `db.operation` and `db.sql.table` are available.
If `db.sql.table` is not available due to its semantics, the span SHOULD be named `<db.operation> <db.name>`.
It is not recommended to attempt any client-side parsing of `db.statement` just to get these properties,
they should only be used if the library being instrumented already provides them.
When it's otherwise impossible to get any meaningful span name, `db.name` or the tech-specific database name MAY be used.

## Connection-level attributes

These attributes will usually be the same for all operations performed over the same database connection.
Some database systems may allow a connection to switch to a different `db.user`, for example, and other database systems may not even have the concept of a connection at all.

<!-- semconv db(tag=connection-level) -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `db.system` | string | 使用しているデータベース管理システム(DBMS)製品の識別子。よく知られた識別子のリストは以下を参照してください。 | `other_sql` | Yes |
| `db.connection_string` | string | データベースへの接続に使用される接続文字列です。埋め込まれた資格情報を削除することを推奨します。 | `Server=(localdb)\v11.0;Integrated Security=true;` | No |
| `db.user` | string | データベースにアクセスするためのユーザー名。 | `readonly_user`; `reporting_user` | No |
| [`net.peer.ip`](span-general.md) | string | 相手のリモートアドレス(IPv4ではドット10進数、IPv6では[RFC5952](https://tools.ietf.org/html/rfc5952) | `127.0.0.1` | 下記参照 |
| [`net.peer.name`](span-general.md) | string | リモートのホスト名あるいは類似の文字列。下記注釈参照 | `example.com` | 下記参照 |
| [`net.peer.port`](span-general.md) | number | リモートのポート番号 | `80`; `8080`; `443` | このDBMSのデフォルトポート以外のポートを使用している場合は必須。 |
| [`net.transport`](span-general.md) | string | Transport protocol used. 下記注釈参照。 | `IP.TCP` | 一般的に推奨、プロセス中のデータベース(`"inproc"`)には必須。 |

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
<!-- endsemconv -->

### Notes and well-known identifiers for `db.system`

The list above is a non-exhaustive list of well-known identifiers to be specified for `db.system`.

If a value defined in this list applies to the DBMS to which the request is sent, this value MUST be used.
If no value defined in this list is suitable, a custom value MUST be provided.
This custom value MUST be the name of the DBMS in lowercase and without a version number to stay consistent with existing identifiers.

It is encouraged to open a PR towards this specification to add missing values to the list, especially when instrumentations for those missing databases are written.
This allows multiple instrumentations for the same database to be aligned and eases analyzing for backends.

The value `other_sql` is intended as a fallback and MUST only be used if the DBMS is known to be SQL-compliant but the concrete product is not known to the instrumentation.
If the concrete DBMS is known to the instrumentation, its specific identifier MUST be used.

Back ends could, for example, use the provided identifier to determine the appropriate SQL dialect for parsing the `db.statement`.

When additional attributes are added that only apply to a specific DBMS, its identifier SHOULD be used as a namespace in the attribute key as for the attributes in the sections below.

### Connection-level attributes for specific technologies

<!-- semconv db.mssql(tag=connection-level-tech-specific,remove_constraints) -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `db.jdbc.driver_classname` | string | 接続に使用する[Java Database Connectivity (JDBC)](https://docs.oracle.com/javase/8/docs/technotes/guides/jdbc/)ドライバの完全修飾クラス名。 | `org.postgresql.Driver`; `com.microsoft.sqlserver.jdbc.SQLServerDriver` | No |
| `db.mssql.instance_name` | string | 接続しているMicrosoft SQL Serverの[インスタンス名](https://docs.microsoft.com/en-us/sql/connect/jdbc/building-the-connection-url?view=sql-server-ver15)です。この名前は、名前付きインスタンスのポートを決定するために使用されます。 [1] | `MSSQLSERVER` | No |

**[1]:** `db.mssql.instance_name`を設定する場合、`net.peer.port`は必須ではなくなりました(ただし、非標準の場合は推奨)
<!-- endsemconv -->

## Call-level attributes

These attributes may be different for each operation performed, even if the same connection is used for multiple operations.
Usually only one `db.name` will be used per connection though.

<!-- semconv db(tag=call-level,remove_constraints) -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `db.name` | string | もし[その技術特有の属性](#call-level-attributes-for-specific-technologies)が定義されていない場合、この属性はアクセスされているデータベースの名前を報告するために使用されます。データベースを切り替えるコマンドのために、これは(コマンドが失敗した場合でも)ターゲットデータベースに設定しなければなりません。 [1] | `customers`; `main` | 該当し、他に適切な属性が定義されていない場合は必須 |
| `db.statement` | string | 実行されているデータベース文。 [2] | `SELECT * FROM wuser_table`; `SET mykey "WuValue"` | 該当する場合は必須 |
| `db.operation` | string | 実行する操作の名前。例えば、`findAndModify` のような [MongoDB コマンド名](https://docs.mongodb.com/manual/reference/command/#database-operations) や SQL キーワード。 [3] | `findAndModify`; `HMSET`; `SELECT` | Required, if `db.statement` is not applicable. |

**[1]:** 一部のSQLデータベースでは、使用するデータベース名を「スキーマ名」と呼びます。

**[2]:** この値は、機密情報を除外するためにサニタイズされている場合があります。

**[3]:** これをSQLキーワードに設定する場合、このプロパティを取得するためだけに `db.statement` のクライアント側の解析を試みることは推奨されません。しかし、測定対象のライブラリから操作名が提供されている場合は設定する必要があります。SQL文に曖昧な操作があったり、複数の操作を実行したりする場合は、この値を省略しても構いません。
<!-- endsemconv -->

For **Redis**, the value provided for `db.statement` SHOULD correspond to the syntax of the Redis CLI.
If, for example, the [`HMSET` command][] is invoked, `"HMSET myhash field1 'Hello' field2 'World'"` would be a suitable value for `db.statement`.

[`HMSET` command]: https://redis.io/commands/hmset

In **CouchDB**, `db.operation` should be set to the HTTP method + the target REST route according to the API reference documentation.
For example, when retrieving a document, `db.operation` would be set to (literally, i.e., without replacing the placeholders with concrete values): [`GET /{db}/{docid}`][CouchDB get doc].

[CouchDB get doc]: http://docs.couchdb.org/en/stable/api/document/common.html#get--db-docid

### Call-level attributes for specific technologies

<!-- semconv db.tech(tag=call-level-tech-specific) -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `db.hbase.namespace` | string | アクセスされている[HBase 名前空間](https://hbase.apache.org/book.html#_namespace)です。一般的な `db.name` 属性の代わりに使用します。 | `default` | Yes |
| `db.redis.database_index` | number | [`SELECT`コマンド](https://redis.io/commands/select)で使われる、アクセスするデータベースのインデックスを整数で指定します。一般的な `db.name` 属性の代わりに使用されます。 | `0`; `1`; `15` | デフォルトのデータベース(`0`)以外の場合は必須です。 |
| `db.mongodb.collection` | string | `db.name`で指定されたデータベース内でアクセスされるコレクションです。 | `customers`; `products` | Yes |
| `db.sql.table` | string | 操作の対象となる主テーブルの名前で、該当する場合スキーマ名も含まれます。 [1] | `public.users`; `customers` | 使用可能であれば推奨されます。 |

**[1]:** このプロパティを取得するために、クライアントサイドで `db.statement` の解析を行うことはお勧めできませんが、計装対象のライブラリで提供されている場合には設定する必要があります。匿名のテーブルや複数のテーブルを操作する場合には、この値を設定してはいけません(MUST NOT)。
<!-- endsemconv -->

#### Cassandra

Separated for clarity.

<!-- semconv db.tech(tag=call-level-tech-specific-cassandra) -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `db.cassandra.keyspace` | string | アクセスするキースペースの名前です。一般的な `db.name` 属性の代わりに使用されます。 | `mykeyspace` | Yes |
| `db.cassandra.page_size` | number | ページングに使用されるフェッチサイズ、つまり一度に何行を返すかを指定します。 | `5000` | No |
| `db.cassandra.consistency_level` | string | クエリの一貫性レベル。[CQL](https://docs.datastax.com/en/cassandra-oss/3.0/cassandra/dml/dmlConfigConsistency.html)の整合性値に基づいています。 | `ALL` | No |
| `db.cassandra.table` | string | 操作の対象となる主テーブルの名前で、(該当する場合)スキーマ名も含まれます。 [1] | `mytable` | Recommended if available. |
| `db.cassandra.idempotence` | boolean | クエリがべき等であるかどうか。 |  | No |
| `db.cassandra.speculative_execution_count` | number | クエリが投機的に実行された回数です。クエリが投機的に実行されなかった場合は、設定されないか、または `0` となります。 | `0`; `2` | No |
| `db.cassandra.coordinator.id` | string | クエリのコーディネーションノードのIDです。 | `be13faa2-8574-4d71-926d-27f16cf8a7af` | No |
| `db.cassandra.coordinator.dc` | string | クエリのコーディネーションノードのデータセンターです。 | `us-west-2` | No |

**[1]:** これは、db.sql.table属性を反映していますが、sqlではなくcassandraを参照しています。この属性を取得するためだけにクライアントサイドで `db.statement` の解析を行うことは推奨されませんが、計装対象のライブラリで提供されている場合は設定する必要があります。操作が匿名のテーブルや複数のテーブルに対して行われる場合、この値は設定してはいけません(MUST NOT)。
<!-- endsemconv -->

## Examples

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

In this example, Redis is connected using a unix domain socket and therefore the connection string and `net.peer.ip` are left out.
Furthermore, `db.name` is not specified as there is no database name in Redis and `db.redis.database_index` is set instead.

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
