<!--
# Semantic conventions for HTTP spans
-->

# HTTP Spanのセマンティック規約

<!--
This document defines semantic conventions for HTTP client and server Spans.
They can be used for http and https schemes
and various HTTP versions like 1.1, 2 and SPDY.
-->

このドキュメントでは、HTTPクライアントとサーバのSpanのセマンティック規約を定義しています。これらは、httpとhttps スキーム、さらに 1.1, 2, SPDY のような様々な HTTP のバージョンで使用できます。

<!-- Re-generate TOC with `markdown-toc --no-first-h1 -i` -->

<!-- toc -->

<!--
- [Name](#name)
- [Status](#status)
- [Common Attributes](#common-attributes)
- [HTTP client](#http-client)
- [HTTP server](#http-server)
  * [HTTP server definitions](#http-server-definitions)
  * [HTTP Server semantic conventions](#http-server-semantic-conventions)
- [HTTP client-server example](#http-client-server-example)
-->

- [名前](#名前)
- [Status](#Status)
- [共通属性](#共通属性)
- [HTTPクライアント](#httpクライアント)
- [HTTPサーバー](#httpサーバー)
  * [HTTPサーバー定義](#HTTPサーバー定義)
  * [HTTPサーバーセマンティック規約](#HTTPサーバーセマンティック規約)
- [HTTPクライアント・サーバー例](#HTTPクライアント・サーバー例)

<!-- tocstop -->

<!--
## Name
-->

## 名前

<!--
HTTP spans MUST follow the overall [guidelines for span names](../api.md#span).
Many REST APIs encode parameters into URI path, e.g. `/api/users/123` where `123`
is a user id, which creates high cardinality value space not suitable for span
names. In case of HTTP servers, these endpoints are often mapped by the server
frameworks to more concise _HTTP routes_, e.g. `/api/users/{user_id}`, which are
recommended as the low cardinality span names. However, the same approach usually
does not work for HTTP client spans, especially when instrumentation is provided
by a lower-level middleware that is not aware of the specifics of how the URIs
are formed. Therefore, HTTP client spans SHOULD be using conservative, low
cardinality names formed from the available parameters of an HTTP request,
such as `"HTTP {METHOD_NAME}"`. Instrumentation MUST NOT default to using URI
path as span name, but MAY provide hooks to allow custom logic to override the
default span name.
-->


HTTP Spanは、全体的な[Span名のガイドライン](../api.md#span)に従わなければなりません(MUST)。多くのREST APIでは、パラメータをURIパスにエンコードしています。例えば、`/api/users/123`のように、`123`はユーザIDであり、Span名には適していない高いカーディナリティの値空間が生成されます。HTTPサーバの場合、これらのエンドポイントはサーバフレームワークによって、例えば `/api/users/{user_id}` のようなより簡潔な _HTTPルート_ にマップされることがよくあります。このようなカーディナリティの低いSpan名が推奨されます。しかしながら、URIがどのように形成されているかの詳細を認識していない低レベルのミドルウェアによって計装が提供されている場合は特に、同じアプローチは通常HTTPクライアントSpanでは機能しません。したがって、HTTPクライアントSpanは、`"HTTP {METHOD_NAME}"`のようなHTTPリクエストの利用可能なパラメータから形成される、保守的でカーディナリティの低い名前を使用するべきです(SHOULD)。テレメトリー、Span名として URI パスを使用することをデフォルトにしてはいけません(MUST NOT)が、カスタム・ロジックがデフォルトのSpan名をオーバーライドできるようにフックを提供しても構いません(MAY)。

<!--
## Status
-->

## Status

<!--
Implementations MUST set the [span status](../api.md#status) if the HTTP communication failed
or an HTTP error status code is returned (e.g. above 3xx).
-->

HTTP通信に失敗した場合や、HTTPエラーのステータスコードが返された場合(例：3xx以上)には、[Span Status](./api.md#status)を設定しなければなりません(MUST)。

<!--
In the case of an HTTP redirect, the request should normally be considered successful,
unless the client aborts following redirects due to hitting some limit (redirect loop).
If following a (chain of) redirect(s) successfully, the status should be set according to the result of the final HTTP request.
-->

HTTP リダイレクトの場合、クライアントが何らかの制限(リダイレクトループ)にぶつかったためにリダイレクトの後続を中断しない限り、リクエストは通常は成功したとみなされるべきです。リダイレクト(の連鎖)に成功した場合、Statusは最終的な HTTP リクエストの結果に応じて設定されなければなりません。

<!--
Don't set the span status description if the reason can be inferred from `http.status_code` and `http.status_text`.
-->

`http.status_code` と `http.status_text` から理由が推測できる場合は、Spanのステータス記述を設定しないようにします。

<!--
| HTTP code               | Span status code      |
|-------------------------|-----------------------|
| 100...299               | `Ok`                  |
| 3xx redirect codes      | `DeadlineExceeded` in case of loop (see above) [1], otherwise `Ok` |
| 401 Unauthorized ⚠      | `Unauthenticated` ⚠ (Unauthorized actually means unauthenticated according to [RFC 7235][rfc-unauthorized])  |
| 403 Forbidden           | `PermissionDenied`    |
| 404 Not Found           | `NotFound`            |
| 429 Too Many Requests   | `ResourceExhausted`   |
| Other 4xx code          | `InvalidArgument` [1] |
| 501 Not Implemented     | `Unimplemented`       |
| 503 Service Unavailable | `Unavailable`         |
| 504 Gateway Timeout     | `DeadlineExceeded`    |
| Other 5xx code          | `Internal` [1]   |
| Any status code the client fails to interpret (e.g., 093 or 573) | `Unknown` |
-->

| HTTP code               | Span status コード      |
|-------------------------|-----------------------|
| 100...299               | `Ok`                  |
| 3xx redirect codes      | ループ(上記参照)の場合[1] `DeadlineExceeded` それ以外は `Ok` |
| 401 Unauthorized ⚠      | `Unauthenticated` ⚠ (Unauthorized は、実際には [RFC 7235][rfc-unauthorized] によると認証されていないことを意味します)  |
| 403 Forbidden           | `PermissionDenied`    |
| 404 Not Found           | `NotFound`            |
| 429 Too Many Requests   | `ResourceExhausted`   |
| 他の 4xx code           | `InvalidArgument` [1] |
| 501 Not Implemented     | `Unimplemented`       |
| 503 Service Unavailable | `Unavailable`         |
| 504 Gateway Timeout     | `DeadlineExceeded`    |
| 他の 5xx code           | `Internal` [1]   |
| クライアントが解釈に失敗したステータスコード(093や573など) | `Unknown` |


<!--
Note that the items marked with [1] are different from the mapping defined in the [OpenCensus semantic conventions][oc-http-status].
-->

[1]の項目は、[OpenCensus セマンティック規約][oc-http-status]で定義されているマッピングとは異なるので注意が必要です。

<!--

-->

[oc-http-status]: https://github.com/census-instrumentation/opencensus-specs/blob/master/trace/HTTP.md#mapping-from-http-status-codes-to-trace-status-codes
[rfc-unauthorized]: https://tools.ietf.org/html/rfc7235#section-3.1

<!--
## Common Attributes
-->

## 共通属性

<!--
| Attribute name | Notes and examples                                           | Required? |
| :------------- | :----------------------------------------------------------- | --------- |
| `http.method` | HTTP request method. E.g. `"GET"`. | Yes |
| `http.url` | Full HTTP request URL in the form `scheme://host[:port]/path?query[#fragment]`. Usually the fragment is not transmitted over HTTP, but if it is known, it should be included nevertheless. | Defined later. |
| `http.target` | The full request target as passed in a [HTTP request line][] or equivalent, e.g. `"/path/12314/?q=ddds#123"`. | Defined later. |
| `http.host` | The value of the [HTTP host header][]. When the header is empty or not present, this attribute should be the same. | Defined later. |
| `http.scheme` | The URI scheme identifying the used protocol: `"http"` or `"https"` | Defined later. |
| `http.status_code` | [HTTP response status code][]. E.g. `200` (integer) | If and only if one was received/sent. |
| `http.status_text` | [HTTP reason phrase][]. E.g. `"OK"` | No |
| `http.flavor` | Kind of HTTP protocol used: `"1.0"`, `"1.1"`, `"2"`, `"SPDY"` or `"QUIC"`. |  No |
| `http.user_agent` | Value of the HTTP [User-Agent][] header sent by the client. | No |
-->

| 属性名 | 説明と例                                           | Required? |
| :------------- | :----------------------------------------------------------- | --------- |
| `http.method` | HTTP リクエストのメソッド。例: `"GET"` | Yes |
| `http.url` | HTTPリクエストの完全なURLを `scheme://host[:port]/path?query[#fragment]` 形式で指定します。通常、フラグメントはHTTPでは送信されませんが、それが分かっている場合は、それにもかかわらず含まれるべきです | 後で定義 |
| `http.target` | [HTTPリクエスト行][]で渡される完全なリクエストターゲット。または同等のもの、例えば `"/path/12314/?q=ddds#123"` | 後で定義 |
| `http.host` | [HTTP HOSTヘッダ][]の値。ヘッダが空であったり存在しない場合、この属性は同じでなければなりません(???これはなにと同じ？空の場合は空のままにしろということだろうか) | 後で定義 |
| `http.scheme` | 使用するプロトコルを識別する URI スキーム: `"http"` or `"https"` | 後で定義 |
| `http.status_code` | [HTTP レスポンスステータスコード][]。例えば `200` (Integer) | 受信/送信された場合に限ります |
| `http.status_text` | [HTTP レスポンスのフレーズ][]。例えば `"OK"` | No |
| `http.flavor` | 使われたHTTPプロトコルの種類: `"1.0"`, `"1.1"`, `"2"`, `"SPDY"`, `"QUIC"` |  No |
| `http.user_agent` | クライアントによって送信されたHTTP [User-Agent][]ヘッダの値  | No |
<!--
It is recommended to also use the general [network attributes][], especially `net.peer.ip`. If `net.transport` is not specified, it can be assumed to be `IP.TCP` except if `http.flavor` is `QUIC`, in which case `IP.UDP` is assumed.
-->

一般的な[ネットワーク属性][]、特に `net.peer.ip` を利用することを推奨します。`net.transport` が指定されない場合は、`http.flavor` が `QUIC` の場合を除き、`IP.TCP` とみなすことができます。

<!--
[network attributes]: span-general.md#general-network-connection-attributes
[HTTP response status code]: https://tools.ietf.org/html/rfc7231#section-6
[HTTP reason phrase]: https://tools.ietf.org/html/rfc7230#section-3.1.2
[User-Agent]: https://tools.ietf.org/html/rfc7231#section-5.5.3
-->

[network attributes]: span-general.md#general-network-connection-attributes
[HTTP response status code]: https://tools.ietf.org/html/rfc7231#section-6
[HTTP reason phrase]: https://tools.ietf.org/html/rfc7230#section-3.1.2
[User-Agent]: https://tools.ietf.org/html/rfc7231#section-5.5.3

<!--
## HTTP client
-->

## HTTPクライアント

<!--
This span type represents an outbound HTTP request.
-->

このSpanの種類は、送信の HTTP リクエストを表します。

<!--
For an HTTP client span, `SpanKind` MUST be `Client`.
-->

HTTPクライアントのSpanの場合、`SpanKind` は `Client` でなければなりません(MUST)。

<!--
If set, `http.url` must be the originally requested URL,
before any HTTP-redirects that may happen when executing the request.
-->

設定されている場合、`http.url` は、リクエストを実行する際に発生する可能性のある HTTP リダイレクトの前の、最初にリクエストされた URL でなければなりません。

<!--
One of the following sets of attributes is required (in order of usual preference unless for a particular web client/framework it is known that some other set is preferable for some reason; all strings must be non-empty):
-->

以下の属性セットのいずれかが必要です(特定のウェブクライアント/フレームワークでは何らかの理由で他のセットが好ましいことがわかっている場合を除き、通常の優先順位の順に並べます; すべての文字列は空でなければなりません)。

<!--
* `http.url`
* `http.scheme`, `http.host`, `http.target`
* `http.scheme`, `net.peer.name`, `net.peer.port`, `http.target`
* `http.scheme`, `net.peer.ip`, `net.peer.port`, `http.target`
-->

* `http.url`
* `http.scheme`, `http.host`, `http.target`
* `http.scheme`, `net.peer.name`, `net.peer.port`, `http.target`
* `http.scheme`, `net.peer.ip`, `net.peer.port`, `http.target`

<!--
Note that in some cases `http.host` might be different
from the `net.peer.name`
used to look up the `net.peer.ip` that is actually connected to.
In that case it is strongly recommended to set the `net.peer.name` attribute in addition to `http.host`.
-->

場合によっては、`http.host` が実際に接続されている `net.peer.ip` を調べるための `net.peer.name` とは異なる場合があることに注意してください。その場合は、`http.host` の他に `net.peer.name` 属性を設定することを強く推奨します。

<!--
For status, the following special cases have canonical error codes assigned:
-->

Statusについては、以下の特殊なケースには標準的なエラーコードが割り当てられています。

<!--
| Client error                | Trace status code  |
|-----------------------------|--------------------|
| DNS resolution failed       | `Unknown`     |
| Request cancelled by caller | `Cancelled`        |
| URL cannot be parsed        | `InvalidArgument`  |
| Request timed out           | `DeadlineExceeded` |
-->

| クライアントのエラー          | Trace statusコード |
|-----------------------------|--------------------|
| DNS 解決が失敗した           | `Unknown`     |
| 呼び出し側によってリクエストがキャンセルされた | `Cancelled`        |
| URLがパースできない           | `InvalidArgument`  |
| リクエストタイムアウト        | `DeadlineExceeded` |


<!--
This is not meant to be an exhaustive list
but if there is no clear mapping for some error conditions,
instrumentation developers are encouraged to use `Unknown`
and open a PR or issue in the specification repository.
-->

これは網羅的なリストではありませんが、いくつかのエラー条件に対する明確なマッピングがない場合は、計装の開発者は `Unknown` を使用し、仕様リポジトリで PR やIssueを作ることをお勧めします。

<!--
## HTTP server
-->

## HTTPサーバー

<!--
To understand the attributes defined in this section, it is helpful to read the "Definitions" subsection.
-->

このセクションで定義されている属性を理解するには、「HTTPサーバー定義」の項を読むと分かりやすいです。

<!--
### HTTP server definitions
-->

### HTTPサーバー定義

<!--
This section gives a short summary of some concepts
in web server configuration and web app deployment
that are relevant to tracing.
-->

この項では、Traceに関連するWebサーバ設定とWebアプリの開発におけるいくつかの概念を簡単にまとめています。

<!--
Usually, on a physical host, reachable by one or multiple IP addresses, a single HTTP listener process runs.
If multiple processes are running, they must listen on distinct TCP/UDP ports so that the OS can route incoming TCP/UDP packets to the right one.
-->

通常、1つまたは複数のIPアドレスで到達可能な物理ホスト上では、単一のHTTPリスナープロセスが実行されます。複数のプロセスが実行されている場合、OS が受信 TCP/UDP パケットを適切なものにルーティングできるように、異なる TCP/UDP ポートでListenする必要があります。

<!--
Within a single server process, there can be multiple **virtual hosts**.
The [HTTP host header][] (in combination with a port number) is normally used to determine to which of them to route incoming HTTP requests.
-->

単一のサーバプロセス内には、複数の**バーチャルホスト**が存在することがあります。[HTTP ホストヘッダ][] (ポート番号との組み合わせ) は、通常、受信 HTTP リクエストをどのバーチャルホストにルーティングするかを決めるために使用されます。

<!--
The host header value that matches some virtual host is called the virtual hosts's **server name**. If there are multiple aliases for the virtual host, one of them (often the first one listed in the configuration) is called the **primary server name**. See for example, the Apache [`ServerName`][ap-sn] or NGINX [`server_name`][nx-sn] directive or the CGI specification on `SERVER_NAME` ([RFC 3875][rfc-servername]).
In practice the HTTP host header is often ignored when just a single virtual host is configured for the IP.
-->

いくつかのバーチャルホストにマッチするホストヘッダーの値は、バーチャルホストの **サーバ名** と呼ばれます。バーチャルホストに複数のエイリアスがある場合、そのうちの1つ(多くの場合、設定ファイルで最初に指定されているもの)を **primary server name** と呼びます。例えば、Apache [`ServerName`][ap-sn] 、 NGINX [`server_name`][nx-sn] ディレクティブや `SERVER_NAME` ([RFC 3875][rfc-servername]) の CGI 仕様を参照してください。実際には、単一のバーチャルホストがそのIPに設定されているだけの場合、HTTPホストヘッダは無視されることが多いです。

<!--
Within a single virtual host, some servers support the concepts of an **HTTP application**
(for example in Java, the Servlet JSR defines an application as
"a collection of servlets, HTML pages, classes, and other resources that make up a complete application on a Web server"
-- SRV.9 in [JSR 53][];
in a deployment of a Python application to Apache, the application would be the [PEP 3333][] conformant callable that is configured using the
[`WSGIScriptAlias` directive][modwsgisetup] of `mod_wsgi`).
-->

単一のバーチャルホスト内では、いくつかのサーバは**HTTPアプリケーション**の概念をサポートしています(例えば、Javaでは、Servlet JSRはアプリケーションを「Webサーバ上の完全なアプリケーションを構成するサーブレット、HTMLページ、クラス、およびその他のリソースの集合」と[JSR 53][]のSRV.9で定義しています。PythonアプリケーションをApacheにデプロイする場合、アプリケーションは、`mod_wsgi`の[`WSGIScriptAlias`ディレクティブ][modwsgisetup]を使用して設定された[PEP 3333][]準拠の呼び出し可能なものになります)。

<!--
An application can be "mounted" under some **application root**
(also know as *[context root][]* *[context prefix][]*, or *[document base][]*)
which is a fixed path prefix of the URL that determines to which application a request is routed
(e.g., the server could be configured to route all requests that go to an URL path starting with `/webshop/`
at a particular virtual host
to the `com.example.webshop` web application).
-->

アプリケーションは、リクエストをどのアプリケーションにルーティングするかを決定するURLの固定パスのプレフィックスである**アプリケーションルート**(*[context root][]*、*[context prefix][]*あるいは *[document base][]*としても知られています)の下に"マウント"することができます(例えば、特定のバーチャルホストの`/webshop/`で始まるURLパスに向かうすべてのリクエストを`com.example.webshop`のウェブアプリケーションにルーティングするようにサーバを設定することができます)。

<!--
Some servers allow to bind the same HTTP application to multiple `(virtual host, application root)` pairs.
-->

サーバによっては、同じHTTPアプリケーションを複数の `(バーチャルホスト、アプリケーションルート)` ペアにバインドすることができます。

<!--
> TODO: Find way to trace HTTP application and application root ([opentelemetry/opentelementry-specification#335][])
-->

> TODO: HTTPアプリケーションとアプリケーションルートをトレースする方法を見つける ([opentelemetry/opentelementry-specification#335][])

<!--
[PEP 3333]: https://www.python.org/dev/peps/pep-3333/
[modwsgisetup]: https://modwsgi.readthedocs.io/en/develop/user-guides/quick-configuration-guide.html
[context root]: https://docs.jboss.org/jbossas/guides/webguide/r2/en/html/ch06.html
[context prefix]: https://marc.info/?l=apache-cvs&m=130928191414740
[document base]: http://tomcat.apache.org/tomcat-5.5-doc/config/context.html
[rfc-servername]: https://tools.ietf.org/html/rfc3875#section-4.1.14
[ap-sn]: https://httpd.apache.org/docs/2.4/mod/core.html#servername
[nx-sn]: http://nginx.org/en/docs/http/ngx_http_core_module.html#server_name
[JSR 53]: https://jcp.org/aboutJava/communityprocess/maintenance/jsr053/index2.html
[opentelemetry/opentelementry-specification#335]: https://github.com/open-telemetry/opentelemetry-specification/issues/335
-->

[PEP 3333]: https://www.python.org/dev/peps/pep-3333/
[modwsgisetup]: https://modwsgi.readthedocs.io/en/develop/user-guides/quick-configuration-guide.html
[context root]: https://docs.jboss.org/jbossas/guides/webguide/r2/en/html/ch06.html
[context prefix]: https://marc.info/?l=apache-cvs&m=130928191414740
[document base]: http://tomcat.apache.org/tomcat-5.5-doc/config/context.html
[rfc-servername]: https://tools.ietf.org/html/rfc3875#section-4.1.14
[ap-sn]: https://httpd.apache.org/docs/2.4/mod/core.html#servername
[nx-sn]: http://nginx.org/en/docs/http/ngx_http_core_module.html#server_name
[JSR 53]: https://jcp.org/aboutJava/communityprocess/maintenance/jsr053/index2.html
[opentelemetry/opentelementry-specification#335]: https://github.com/open-telemetry/opentelemetry-specification/issues/335

<!--
### HTTP Server semantic conventions
-->

### HTTPサーバーセマンティック規約

<!--
This span type represents an inbound HTTP request.
-->

このSpanの種類は、受信の HTTP リクエストを表します。

<!--
For an HTTP server span, `SpanKind` MUST be `Server`.
-->

HTTPサーバのSpanの場合、`SpanKind` は `Server` でなければなりません(MUST)。

<!--
Given an inbound request for a route (e.g. `"/users/:userID?"`) the `name` attribute of the span SHOULD be set to this route.
If the route does not include the application root, it SHOULD be prepended to the span name.
-->

Route(例: `"/users/:userID?"`)に対する受信リクエストがあった場合、Spanの `name` 属性はこのRouteに設定されるべきです(SHOULD)。Routeにアプリケーションルートが含まれていない場合は、Span名の前に付加されるべきです(SHOULD)。

<!--
If the route cannot be determined, the `name` attribute MUST be set as defined in the general semantic conventions for HTTP.
-->

Routeを決定できない場合、HTTPの一般的なセマンティック規約で定義されているように `name` 属性を設定しなければなりません(MUST)。

<!--
| Attribute name | Notes and examples                                           | Required? |
| :------------- | :----------------------------------------------------------- | --------- |
| `http.server_name` | The primary server name of the matched virtual host. This should be obtained via configuration. If no such configuration can be obtained, this attribute MUST NOT be set ( `net.host.name` should be used instead). | [1] |
| `http.route` | The matched route (path template). (TODO: Define whether to prepend application root) E.g. `"/users/:userID?"`. | No |
| `http.client_ip` | The IP address of the original client behind all proxies, if known (e.g. from [X-Forwarded-For][]). Note that this is not necessarily the same as `net.peer.ip`, which would identify the network-level peer, which may be a proxy. | No |
-->

| 属性名 | 説明と例                                           | Required? |
| :------------- | :----------------------------------------------------------- | --------- |
| `http.server_name` | 一致したバーチャルホストのPrimary Server名。これは設定から取得しなければなりません。そのような設定ができない場合、この属性は設定してはいけません(MUST NOT) (代わりに `net.host.name` を使用すべきです(SHOULD)) | [1] |
| `http.route` | 一致したRoute(パステンプレート)。(TODO: アプリケーションルートを前置するかどうかの定義)。例 `"/users/:userID?"` | No |
| `http.client_ip` | 取得できる場合は、すべてのプロキシの背後にある元のクライアントのIPアドレス( [X-Forwarded-For][]など)。これはネットワークレベルのピアを識別する `net.peer.ip` と必ずしも同じではないことに注意してください | No |

<!--
[HTTP request line]: https://tools.ietf.org/html/rfc7230#section-3.1.1
[HTTP host header]: https://tools.ietf.org/html/rfc7230#section-5.4
[X-Forwarded-For]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-For
-->

[HTTP request line]: https://tools.ietf.org/html/rfc7230#section-3.1.1
[HTTP host header]: https://tools.ietf.org/html/rfc7230#section-5.4
[X-Forwarded-For]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-For

<!--
**[1]**: `http.url` is usually not readily available on the server side but would have to be assembled in a cumbersome and sometimes lossy process from other information (see e.g. <https://github.com/open-telemetry/opentelemetry-python/pull/148>).
It is thus preferred to supply the raw data that *is* available.
Namely, one of the following sets is required (in order of usual preference unless for a particular web server/framework it is known that some other set is preferable for some reason; all strings must be non-empty):
-->

**[1]**: 通常、`http.url` はサーバ側では利用できませんが、他の情報から面倒で時には損失の大きいプロセスで組み立てなければなりません(例えば <https://github.com/open-telemetry/opentelemetry-python/pull/148> を参照)。したがって、利用可能な生データを提供すること *が* 好ましいです(??? *is* をどう表現するか)。すなわち、以下のセットのいずれかが必要です (特定のウェブサーバ/フレームワークのために何らかの理由で他のセットが好ましいことが知られていない限り、通常の優先順位の順です; すべての文字列は空でなければなりません)。

<!--
* `http.scheme`, `http.host`, `http.target`
* `http.scheme`, `http.server_name`, `net.host.port`, `http.target`
* `http.scheme`, `net.host.name`, `net.host.port`, `http.target`
* `http.url`
-->

* `http.scheme`, `http.host`, `http.target`
* `http.scheme`, `http.server_name`, `net.host.port`, `http.target`
* `http.scheme`, `net.host.name`, `net.host.port`, `http.target`
* `http.url`

<!--
Of course, more than the required attributes can be supplied, but this is recommended only if they cannot be inferred from the sent ones.
For example, `http.server_name` has shown great value in practice, as bogus HTTP Host headers occur often in the wild.
-->

もちろん、必要以上の属性を指定することもできますが、送信された属性から推測できない場合にのみ推奨されます。例えば、HTTP ホストヘッダが偽装されていることがよくあるため、 `http.server_name` は実際には非常に価値のあるものです。

<!--
It is strongly recommended to set `http.server_name` to allow associating requests with some logical server entity.
-->

リクエストを論理的なサーバ実体に関連付けるために、`http.server_name` を設定することを強く推奨します。

<!--
## HTTP client-server example
-->

## HTTPクライアント・サーバーの例

<!--
As an example, if a browser request for `https://example.com:8080/webshop/articles/4?s=1` is invoked from a host with IP 192.0.2.4, we may have the following Span on the client side:
-->

例えば、IP 192.0.2.4のホストから `https://example.com:8080/webshop/articles/4?s=1` へのブラウザリクエストが呼び出された場合、クライアント側では以下のようになります。

<!--
Span name: `/webshop/articles/4` (NOTE: This is subject to change, see [open-telemetry/opentelemetry-specification#270][].)
-->

Span 名: `/webshop/articles/4` (注意: 変更の可能性があります。[open-telemetry/opentelemetry-specification#270][] 参照)

<!--
[open-telemetry/opentelemetry-specification#270]: https://github.com/open-telemetry/opentelemetry-specification/issues/270
-->

[open-telemetry/opentelemetry-specification#270]: https://github.com/open-telemetry/opentelemetry-specification/issues/270

<!--
|   Attribute name   |                                       Value             |
| :----------------- | :-------------------------------------------------------|
| `http.method`      | `"GET"`                                                 |
| `http.flavor`      | `"1.1"`                                                 |
| `http.url`         | `"https://example.com:8080/webshop/articles/4?s=1"`     |
| `net.peer.ip`      | `"192.0.2.5"`                                           |
| `http.status_code` | `200`                                                   |
| `http.status_text` | `"OK"`                                                  |
-->

|   属性名        |                                       値                  |
| :----------------- | :-------------------------------------------------------|
| `http.method`      | `"GET"`                                                 |
| `http.flavor`      | `"1.1"`                                                 |
| `http.url`         | `"https://example.com:8080/webshop/articles/4?s=1"`     |
| `net.peer.ip`      | `"192.0.2.5"`                                           |
| `http.status_code` | `200`                                                   |
| `http.status_text` | `"OK"`                                                  |

<!--
The corresponding server Span may look like this:
-->

対応するサーバのSpanは以下のようになります。

<!--
Span name: `/webshop/articles/:article_id`.
-->

Span 名: `/webshop/articles/:article_id`

<!--
|   Attribute name   |                      Value                      |
| :----------------- | :---------------------------------------------- |
| `http.method`      | `"GET"`                                         |
| `http.flavor`      | `"1.1"`                                         |
| `http.target`      | `"/webshop/articles/4?s=1"`                     |
| `http.host`        | `"example.com:8080"`                            |
| `http.server_name` | `"example.com"`                                 |
| `net.host.port`    | `8080`                                          |
| `http.scheme`      | `"https"`                                       |
| `http.route`       | `"/webshop/articles/:article_id"`               |
| `http.status_code` | `200`                                           |
| `http.status_text` | `"OK"`                                          |
| `http.client_ip`   | `"192.0.2.4"`                                   |
| `net.peer.ip`      | `"192.0.2.5"` (the client goes through a proxy) |
| `http.user_agent`  | `"Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:72.0) Gecko/20100101 Firefox/72.0"`                               |
-->

|   属性名        |                                       値                  |
| :----------------- | :---------------------------------------------- |
| `http.method`      | `"GET"`                                         |
| `http.flavor`      | `"1.1"`                                         |
| `http.target`      | `"/webshop/articles/4?s=1"`                     |
| `http.host`        | `"example.com:8080"`                            |
| `http.server_name` | `"example.com"`                                 |
| `net.host.port`    | `8080`                                          |
| `http.scheme`      | `"https"`                                       |
| `http.route`       | `"/webshop/articles/:article_id"`               |
| `http.status_code` | `200`                                           |
| `http.status_text` | `"OK"`                                          |
| `http.client_ip`   | `"192.0.2.4"`                                   |
| `net.peer.ip`      | `"192.0.2.5"` (the client goes through a proxy) |
| `http.user_agent`  | `"Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:72.0) Gecko/20100101 Firefox/72.0"`                               |

<!--
Note that following the recommendations above, `http.url` is not set in the above example.
If set, it would be
`"https://example.com:8080/webshop/articles/4?s=1"`
but due to `http.scheme`, `http.host` and `http.target` being set, it would be redundant.
As explained above, these separate values are preferred but if for some reason the URL is available but the other values are not,
URL can replace `http.scheme`, `http.host` and `http.target`.
-->

上記の推奨事項に従って、上記の例では `http.url` は設定されていないことに注意してください。設定されていれば `"https://example.com:8080/webshop/articles/4?s=1"` となりますが、`http.scheme`、 `http.host`、 `http.target` の３つが設定されているため、重複した値になってしまいます。上で説明したように、これらの値は別々に設定するのが望ましいですが、何らかの理由でURLが利用できるのに他の値が利用できない場合は、URLを `http.scheme`、 `http.host`、 `http.target` に置き換えることができます。

