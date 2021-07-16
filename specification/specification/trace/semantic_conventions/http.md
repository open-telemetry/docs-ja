<!--
# Semantic conventions for HTTP spans
-->

# HTTP Spanのセマンティック規約

<!--
**Status**: [Experimental](../../document-status.md)
-->

**Status**: [Experimental](../../document-status.md)

<!--
This document defines semantic conventions for HTTP client and server Spans.
They can be used for http and https schemes
and various HTTP versions like 1.1, 2 and SPDY.
-->

本ドキュメントでは、HTTPクライアントとサーバーのSpanの意味的な規約を定義しています。これらは、httpやhttpsのスキームや、1.1、2、SPDYなどの様々なHTTPバージョンに使用できます。

<!-- Re-generate TOC with `markdown-toc --no-first-h1 -i` -->

<!-- toc -->

<!--
- [Semantic conventions for HTTP spans](#semantic-conventions-for-http-spans)
  - [Name](#name)
  - [Status](#status)
  - [Common Attributes](#common-attributes)
  - [HTTP client](#http-client)
  - [HTTP server](#http-server)
    - [HTTP server definitions](#http-server-definitions)
    - [HTTP Server semantic conventions](#http-server-semantic-conventions)
  - [HTTP client-server example](#http-client-server-example)
-->

- [HTTPSpanのセマンティック規約](#HTTPSpanのセマンティック規約)
  - [名前](#名前)
  - [Status](#status)
  - [共通の属性](#共通の属性)
  - [HTTP client](#http-client)
  - [HTTP server](#http-server)
    - [HTTP serverの定義](#HTTP-serverの定義)
    - [HTTPサーバのセマンティック規約](#HTTPサーバのセマンティック規約)
  - [HTTPクライアント・サーバーの例](#HTTPクライアント・サーバーの例)

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

HTTPのSpanは、全体的な[Span名のガイドライン](../api.md#span)に従わなければなりません。多くのREST APIでは、パラメータをURIパスにエンコードしています。例えば、`/api/users/123`(`123`はユーザーID)のように、Span名には適さない高いカーディナリティの値空間を作り出しています。HTTPサーバの場合、これらのエンドポイントは、サーバフレームワークによって、より簡潔な _HTTP route_、例えば、`/api/users/{user_id}`にマッピングされることが多く、これは、低いカーディナリティのSpan名として推奨されます。しかし、同じアプローチは通常、HTTPクライアントSpanでは機能しません。特に、URIがどのように形成されるかの詳細を認識していない低レベルのミドルウェアによって計装が提供される場合はそうです。したがって、HTTPクライアントSpanは、`"HTTP {METHOD_NAME}"`のような、HTTPリクエストの利用可能なパラメータから形成された、保守的でカーディナリティの低い名前を使用すべきです(SHOULD)。計装では、デフォルトでURIパスをSpan名として使用してはならないが、カスタムロジックでデフォルトのSpan名をオーバーライドできるフックを提供しても構いません(MAY)。

<!--
## Status
-->

## Status

<!--
[Span Status](../api.md#set-status) MUST be left unset if HTTP status code was in the
1xx, 2xx or 3xx ranges, unless there was another error (e.g., network error receiving
the response body; or 3xx codes with max redirects exceeded), in which case status
MUST be set to `Error`.
-->

HTTPステータスコードが1xx、2xx、3xxの範囲にある場合は、別のエラー(レスポンスボディを受信するネットワークエラー、最大リダイレクト数を超えた3xxコードなど)が発生しない限り、[Span Status](../api.md#set-status)は設定しないままにしなければなりません(その場合(別のエラーが発生した場合)、statusは`Error`に設定しなければなりません(MUST))。

<!--
For HTTP status codes in the 4xx and 5xx ranges, as well as any other code the client
failed to interpret, status MUST be set to `Error`.
-->

4xx や 5xx の HTTP ステータスコードや、クライアントが解釈に失敗したその他のコードについては、status を `Error` に設定しなければなりません (MUST)。

<!--
Don't set the span status description if the reason can be inferred from `http.status_code`.
-->

理由が `http.status_code` から推測できる場合は、Spanの"status description"を設定しないでください。

<!--
## Common Attributes
-->

## 共通の属性

<!-- semconv http -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `http.method` | string | HTTPリクエストメソッド | `GET`; `POST`; `HEAD` | Yes |
| `http.url` | string | `scheme://host[:port]/path?query[#fragment]`という形式の完全なHTTPリクエストURLです。 通常、このフラグメントはHTTPでは送信されませんが、知られている場合は、それにもかかわらず含める必要があります。 [1] | `https://www.foo.bar/search?q=OpenTelemetry#SemConv` | No |
| `http.target` | string | HTTPリクエスト行またはそれに相当するもので渡される、完全なリクエストターゲット。 | `/path/12314/?q=ddds#123` | No |
| `http.host` | string | [HTTP host ヘッダー](https://tools.ietf.org/html/rfc7230#section-5.4)の値です。 ヘッダーが空の場合、または存在しない場合、この属性は同じである必要があります。 | `www.example.org` | No |
| `http.scheme` | string | 使用するプロトコルを示すURIスキーム | `http`; `https` | No |
| `http.status_code` | int | [HTTP レスポンス status code](https://tools.ietf.org/html/rfc7231#section-6). | `200` | 受信/送信された場合に限ります。 |
| `http.flavor` | string | 使われているHTTP protocolの種類 [2] | `1.0` | No |
| `http.user_agent` | string | クライアントが送信した[HTTP User-Agent](https://tools.ietf.org/html/rfc7231#section-5.5.3)ヘッダの値。 | `CERN-LineMode/2.15 libwww/2.17b3` | No |
| `http.request_content_length` | int | リクエストのペイロードボディのサイズ(バイト)です。これは、ヘッダーを除いて転送されたバイト数で、[Content-Length](https://tools.ietf.org/html/rfc7230#section-3.3.2)ヘッダーとして表示されることが多いですが、必ずしもそうではありません。トランスポートエンコーディングを使用するリクエストでは、これは圧縮されたサイズでなければなりません。 | `3495` | No |
| `http.request_content_length_uncompressed` | int | トランスポートデコーディング後の、圧縮されていないリクエストペイロードボディのサイズです。トランスポートエンコーディングを使用しない場合は設定されません。 | `5493` | No |
| `http.response_content_length` | int | レスポンスペイロードボディのサイズ(バイト)です。これは、ヘッダーを除いて転送されたバイト数で、[Content-Length](https://tools.ietf.org/html/rfc7230#section-3.3.2)ヘッダーとして表示されることが多いですが、必ずしもそうではありません。トランスポートエンコーディングを使用するリクエストでは、これは圧縮されたサイズでなければなりません。 | `3495` | No |
| `http.response_content_length_uncompressed` | int | トランスポートデコーディング後の、圧縮されていないレスポンスペイロードボディのサイズです。トランスポートエンコーディングを使用しない場合は設定されません。 | `5493` | No |

**[1]:** `http.url` MUST NOT contain credentials passed via URL in form of `https://username:password@www.example.com/`. In such case the attribute's value should be `https://www.example.com/`.

**[2]:** `net.transport` が指定されていない場合は、`IP.TCP`と仮定できます。ただし、`http.flavor` が `QUIC` の場合は、`IP.UDP` を想定しています。

`http.flavor` MUST be one of the following or, if none of the listed values apply, a custom value:

| Value  | Description |
|---|---|
| `1.0` | HTTP 1.0 |
| `1.1` | HTTP 1.1 |
| `2.0` | HTTP 2 |
| `SPDY` | SPDY protocol. |
| `QUIC` | QUIC protocol. |
<!-- endsemconv -->

<!--
It is recommended to also use the general [network attributes][], especially `net.peer.ip`. If `net.transport` is not specified, it can be assumed to be `IP.TCP` except if `http.flavor` is `QUIC`, in which case `IP.UDP` is assumed.
-->

また、一般的な [ネットワーク属性][] 、特に `net.peer.ip` を使用することを推奨します。ただし、`http.flavor` が `QUIC` の場合は `IP.UDP` を想定しています。

<!--
[network attributes]: span-general.md#general-network-connection-attributes
-->

[ネットワーク属性]: span-general.md#general-network-connection-attributes

<!--
## HTTP client
-->

## HTTP client

<!--
This span type represents an outbound HTTP request.
-->

このSpanタイプは、アウトバウンドのHTTPリクエストを表します。

<!--
For an HTTP client span, `SpanKind` MUST be `Client`.
-->

HTTPクライアントのSpanの場合、`SpanKind`は`Client`でなければなりません(MUST)。

<!--
If set, `http.url` must be the originally requested URL,
before any HTTP-redirects that may happen when executing the request.
-->

設定されている場合、`http.url`は、リクエストを実行する際に発生する可能性のあるHTTPリダイレクトの前の、最初にリクエストされたURLでなければなりません。

<!-- semconv http.client -->

**Additional attribute requirements:** At least one of the following sets of attributes is required:

* `http.url`
* `http.scheme`, `http.host`, `http.target`
* `http.scheme`, [`net.peer.name`](span-general.md), [`net.peer.port`](span-general.md), `http.target`
* `http.scheme`, [`net.peer.ip`](span-general.md), [`net.peer.port`](span-general.md), `http.target`
<!-- endsemconv -->

<!--
Note that in some cases `http.host` might be different
from the `net.peer.name`
used to look up the `net.peer.ip` that is actually connected to.
In that case it is strongly recommended to set the `net.peer.name` attribute in addition to `http.host`.
-->

場合によっては、`http.host`が、実際に接続されている`net.peer.ip`を検索する際に使用される`net.peer.name`と異なることがありますのでご注意ください。そのような場合には、`http.host`に加えて、`net.peer.name`属性を設定することを強く勧めします。

<!--
## HTTP server
-->

## HTTP server

<!--
To understand the attributes defined in this section, it is helpful to read the "Definitions" subsection.
-->

このセクションで定義されている属性を理解するためには、「定義」のサブセクションを読むことが役立ちます。

<!--
### HTTP server definitions
-->

### HTTP serverの定義

<!--
This section gives a short summary of some concepts
in web server configuration and web app deployment
that are relevant to tracing.
-->

このセクションでは、トレースに関連するウェブサーバーの設定とウェブアプリのデプロイメントにおけるいくつかのコンセプトについて簡単にまとめています。

<!--
Usually, on a physical host, reachable by one or multiple IP addresses, a single HTTP listener process runs.
If multiple processes are running, they must listen on distinct TCP/UDP ports so that the OS can route incoming TCP/UDP packets to the right one.
-->

通常、1つまたは複数のIPアドレスで到達可能な物理的なホスト上で、1つのHTTPリスナープロセスが実行されます。複数のプロセスが動作している場合は、OSが受信したTCP/UDPパケットを適切なプロセスにルーティングできるように、それぞれのプロセスが異なるTCP/UDPポートをリッスンする必要があります。

<!--
Within a single server process, there can be multiple **virtual hosts**.
The [HTTP host header][] (in combination with a port number) is normally used to determine to which of them to route incoming HTTP requests.
-->

1つのサーバープロセスの中に、複数の **バーチャルホスト** が存在することがあります。[HTTPホストヘッダ][](ポート番号との組み合わせ)は、通常、受信したHTTPリクエストをどのホストに転送するかを決定するために使用されます。

<!--
The host header value that matches some virtual host is called the virtual hosts's **server name**. If there are multiple aliases for the virtual host, one of them (often the first one listed in the configuration) is called the **primary server name**. See for example, the Apache [`ServerName`][ap-sn] or NGINX [`server_name`][nx-sn] directive or the CGI specification on `SERVER_NAME` ([RFC 3875][rfc-servername]).
In practice the HTTP host header is often ignored when just a single virtual host is configured for the IP.
-->

あるバーチャルホストにマッチするホストヘッダの値を、そのバーチャルホストの**サーバー名**と呼びます。バーチャルホストに複数のエイリアスがある場合、そのうちの1つ(多くの場合、設定に記載されている最初のもの)を **primary server name** と呼びます。例えば、Apache [`ServerName`][ap-sn] や NGINX [`server_name`][nx-sn] のディレクティブや、`SERVER_NAME` に関する CGI の仕様 ([RFC 3875][rfc-servername])を参照してください。実際には、IPにバーチャルホストが1つだけ設定されている場合、HTTPホストヘッダは無視されることが多いです。

<!--
Within a single virtual host, some servers support the concepts of an **HTTP application**
(for example in Java, the Servlet JSR defines an application as
"a collection of servlets, HTML pages, classes, and other resources that make up a complete application on a Web server"
-- SRV.9 in [JSR 53][];
in a deployment of a Python application to Apache, the application would be the [PEP 3333][] conformant callable that is configured using the
[`WSGIScriptAlias` directive][modwsgisetup] of `mod_wsgi`).
-->

1つのバーチャルホストの中で、**HTTPアプリケーション**の概念をサポートしているサーバもあります(例えばJavaでは、Servlet JSRがアプリケーションを「Webサーバ上の完全なアプリケーションを構成するサーブレット、HTMLページ、クラス、およびその他のリソースの集合体」と定義しています--SRV.9の[JSR 53][]で定義されています。Apache への Python アプリケーションのデプロイメントでは、アプリケーションは `mod_wsgi` の [`WSGIScriptAlias` ディレクティブ][modwsgisetup] を使用して設定された [PEP 3333][] 準拠の callable となります)。

<!--
An application can be "mounted" under some **application root**
(also know as *[context root][]* *[context prefix][]*, or *[document base][]*)
which is a fixed path prefix of the URL that determines to which application a request is routed
(e.g., the server could be configured to route all requests that go to an URL path starting with `/webshop/`
at a particular virtual host
to the `com.example.webshop` web application).
-->

アプリケーションは、ある**アプリケーションルート**(*[コンテキストルート][]* *[コンテキストプレフィックス][]* または *[ドキュメントベース][]*とも呼ばれます)の下に「マウント」することができます。これは、リクエストがどのアプリケーションにルーティングされるかを決定するURLの固定パスプレフィックスです(例えば、特定のバーチャルホストの`/webshop/`で始まるURLパスに向かうすべてのリクエストを、`com.example.webshop`というウェブアプリケーションにルーティングするようにサーバーを設定することができます)。

<!--
Some servers allow to bind the same HTTP application to multiple `(virtual host, application root)` pairs.
-->

サーバーによっては、同じHTTPアプリケーションを複数の`(バーチャルホスト、アプリケーションルート)`ペアにバインドすることができます。

<!--
> TODO: Find way to trace HTTP application and application root ([opentelemetry/opentelementry-specification#335][])
-->

> TODO: HTTPアプリケーションとアプリケーションルートを追跡する方法を探す ([opentelemetry/opentelementry-specification#335][])

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

### HTTPサーバのセマンティック規約

<!--
This span type represents an inbound HTTP request.
-->

このSpanタイプは、インバウンドのHTTPリクエストを表します。


<!--
For an HTTP server span, `SpanKind` MUST be `Server`.
-->

HTTPサーバーSpanの場合、`SpanKind`は`Server`でなければなりません(MUST)。

<!--
Given an inbound request for a route (e.g. `"/users/:userID?"`) the `name` attribute of the span SHOULD be set to this route.
If the route does not include the application root, it SHOULD be prepended to the span name.
-->

Route(例:`"/users/:userID?"`)に対するインバウンドリクエストがあった場合、Spanの `name` 属性にはこのRouteが設定されるべきです(SHOULD)。Routeにアプリケーションのrootが含まれていない場合は、Spanの名前の前にRouteが付加されるべきです(SHOULD)。

<!--
If the route cannot be determined, the `name` attribute MUST be set as defined in the general semantic conventions for HTTP.
-->

Routeが特定できない場合は、HTTPの一般的なセマンティック規約で定義されているように、`name`属性を設定しなければなりません(MUST)。

<!-- semconv http.server -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `http.server_name` | string | 一致したバーチャルホストのプライマリサーバー名です。これは設定で取得する必要があります。 そのような設定が得られない場合は、この属性を設定してはいけません(MUST NOT)(`net.host.name`を代わりに使用する必要があります)。 [1] | `example.com` | See below |
| `http.route` | string | マッチしたルート(パステンプレート) | `/users/:userID?` | No |
| `http.client_ip` | string | すべてのプロキシの背後にあるオリジナルのクライアントのIPアドレス(既知の場合)(例:[X-Forwarded-For](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-For)より) [2] | `83.164.160.102` | No |

**[1]:** `http.url`は通常、サーバー側では容易に入手できませんが、他の情報から煩雑で時には損失を伴うプロセスで組み立てなければなりません(例:open-telemetry/opentelemetry-python/pull/148を参照)。 そのため、利用可能な生データを提供することが望ましいです。

**[2]:** これは必ずしも`net.peer.ip`と同じではありません。`net.peer.ip`はネットワークレベルのピアを特定するもので、プロキシである可能性もあります。

**Additional attribute requirements:** At least one of the following sets of attributes is required:

* `http.scheme`, `http.host`, `http.target`
* `http.scheme`, `http.server_name`, [`net.host.port`](span-general.md), `http.target`
* `http.scheme`, [`net.host.name`](span-general.md), [`net.host.port`](span-general.md), `http.target`
* `http.url`
<!-- endsemconv -->

<!--
Of course, more than the required attributes can be supplied, but this is recommended only if they cannot be inferred from the sent ones.
For example, `http.server_name` has shown great value in practice, as bogus HTTP Host headers occur often in the wild.
-->

もちろん、必要な属性以外のものを指定することもできますが、これは送信された属性から推測できない場合にのみ推奨されます。例えば、`http.server_name`は、実運用では偽のHTTP Hostヘッダーが頻繁に発生することから、実際には大きな価値を示しています。

<!--
It is strongly recommended to set `http.server_name` to allow associating requests with some logical server entity.
-->

`http.server_name`を設定して、リクエストを何らかの論理的なサーバーエンティティに関連付けることを強く推奨します。

<!--
## HTTP client-server example
-->

## HTTPクライアント・サーバーの例

<!--
As an example, if a browser request for `https://example.com:8080/webshop/articles/4?s=1` is invoked from a host with IP 192.0.2.4, we may have the following Span on the client side:
-->

例えば、IP 192.0.2.4のホストからブラウザで`https://example.com:8080/webshop/articles/4?s=1`のリクエストが呼び出された場合、クライアント側では以下のようなSpanになります。

<!--
Span name: `/webshop/articles/4` (NOTE: This is subject to change, see [open-telemetry/opentelemetry-specification#270][].)
-->

Span名: `/webshop/articles/4` (注意: これは変更される可能性があります。[open-telemetry/opentelemetry-specification#270][]を参照してください。)

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
-->

|   Attribute name   |                                       Value             |
| :----------------- | :-------------------------------------------------------|
| `http.method`      | `"GET"`                                                 |
| `http.flavor`      | `"1.1"`                                                 |
| `http.url`         | `"https://example.com:8080/webshop/articles/4?s=1"`     |
| `net.peer.ip`      | `"192.0.2.5"`                                           |
| `http.status_code` | `200`                                                   |

<!--
The corresponding server Span may look like this:
-->

対応するサーバーSpanは以下のようになります。

<!--
Span name: `/webshop/articles/:article_id`.
-->

Span name: `/webshop/articles/:article_id`.

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
| `http.client_ip`   | `"192.0.2.4"`                                   |
| `net.peer.ip`      | `"192.0.2.5"` (the client goes through a proxy) |
| `http.user_agent`  | `"Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:72.0) Gecko/20100101 Firefox/72.0"`                               |
-->

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

上記の推奨事項に従うと、上記の例では `http.url` が設定されていないことに注意してください。もし設定されていれば、`"https://example.com:8080/webshop/articles/4?s=1"`となりますが、`http.scheme`、`http.host`、`http.target`が設定されているため、冗長になってしまいます。上記のように、これらの独立した値が望ましいのですが、何らかの理由でURLが利用できても他の値が利用できない場合には、URLが`http.scheme`、`http.host`、`http.target`の代わりになります。
