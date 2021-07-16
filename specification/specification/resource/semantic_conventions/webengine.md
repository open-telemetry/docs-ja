# Webengine

**type:** `webengine`

<!--
**Description:** Resource describing the packaged software running the application code. Web engines are typically executed using process.runtime.
-->

**Description:** アプリケーションコードを実行するパッケージソフトウェアを記述するリソース。Webエンジンは、通常、process.runtimeを使用して実行されます。

<!-- semconv webengine_resource -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `webengine.name` | string | Webエンジンの名前 | `WildFly` | Yes |
| `webengine.version` | string | Webエンジンのバージョン | `21.0.0` | No |
| `webengine.description` | string | Webエンジンの追加説明(詳細なバージョンとエディション情報など) | `WildFly Full 21.0.0.Final (WildFly Core 13.0.1.Final) - 2.2.2.Final` | No |
<!-- endsemconv -->

<!--
Information describing the web engine SHOULD be captured using the values acquired from the API provided by the web engine, preferably during runtime, to avoid maintenance burden on engine version upgrades. As an example - Java engines are often but not always packaged as application servers. For Java application servers supporting Servlet API the required information MAY be captured by invoking `ServletContext.getServerInfo()` during runtime and parsing the result.
-->

Webエンジンの情報は、Webエンジンが提供するAPIから取得した値を用いて取得すべきであり、エンジンのバージョンアップに伴うメンテナンスの負担を避けるために、できればランタイム中に取得すべきです(SHOULD)。例として、Javaエンジンはアプリケーションサーバーとしてパッケージ化されていることが多いですが、必ずしもそうではありません。Servlet APIをサポートするJavaアプリケーションサーバの場合、必要な情報は、実行時に`ServletContext.getServerInfo()`を呼び出し、その結果を解析することで取得しても構いません(MAY)。

<!--
A resource can be attributed to at most one web engine.
-->

1つのリソースは、最大で1つのウェブエンジンに帰属します。

<!--
The situations where there are multiple candidates, it is up to instrumentation library authors to choose the web engine. To illustrate, let's look at a Python application using Apache HTTP Server with mod_wsgi as the server and Django as the web framework. In this situation:
-->

複数の候補がある場合は、計装するライブラリの作者がウェブエンジンを選択することになります。例として、サーバーにmod_wsgiを使用したApache HTTP Server、WebフレームワークにDjangoを使用したPythonアプリケーションを見てみましょう。この状況では、

<!--
* Either Apache HTTP Server or `mod_wsgi` MAY be chosen as `webengine`, depending on the decision made by the instrumentation authors.
* Django SHOULD NOT be set as an `webengine` as the required information is already available in insrumentation library and setting this into `webengine` would duplicate the information.
-->

* 計装の作成者の判断により、Apache HTTP Server または `mod_wsgi` のいずれかを `webengine` として選択しても構いません(MAY)。
* Django は `webengine` に設定すべきではありません (SHOULD NOT) 。必要な情報はすでに計装のライブラリにあり、これを `webengine` に設定すると情報が重複してしまうからです。
