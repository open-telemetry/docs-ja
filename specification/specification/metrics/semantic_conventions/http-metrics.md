<!--
# General
-->

# 一般

**Status**: [Experimental](../../document-status.md)

<!--
The conventions described in this section are HTTP specific. When HTTP operations occur,
metric events about those operations will be generated and reported to provide insight into the
operations. By adding HTTP labels to metric events it allows for finely tuned filtering.
-->

このセクションで説明する規約は、HTTP固有のものです。HTTPの操作が発生すると、その操作に関するメトリックイベントが生成され、報告され、操作内容を知ることができます。メトリックイベントにHTTPラベルを追加することで、きめ細かなフィルタリングが可能になります。

<!--
**Disclaimer:** These are initial HTTP metric instruments and labels but more may be added in the future.
-->

**免責事項:** これらは初期のHTTPメトリックInstrumentsとラベルですが、今後さらに追加される可能性があります。

<!--
## Metric Instruments
-->

## Metric Instruments

<!--
The following metric instruments MUST be used to describe HTTP operations. They MUST be of the specified
type and units.
-->

HTTPの操作を記述するために、以下のInstrumentsを使用しなければなりません(MUST)。これらは、指定されたタイプと単位でなければなりません(MUST)。

<!--
### HTTP Server
-->

### HTTP Server

<!--
Below is a table of HTTP server metric instruments.
-->

以下は、HTTPサーバーのMetric Instrumentsの表です。

<!--
| Name                          | Instrument        | Units        | Description |
|-------------------------------|-------------------|--------------|-------------|
| `http.server.duration`        | ValueRecorder     | milliseconds | measures the duration of the inbound HTTP request |
| `http.server.active_requests` | UpDownSumObserver | requests     | measures the number of concurrent HTTP requests that are currently in-flight |
-->

| Name                          | Instrument        | Units        | Description |
|-------------------------------|-------------------|--------------|-------------|
| `http.server.duration`        | ValueRecorder     | milliseconds | 受信したHTTPリクエストの持続時間 |
| `http.server.active_requests` | UpDownSumObserver | requests     | 現在進行中(in-flight)のHTTPリクエストの同時実行数 |

<!--
### HTTP Client
-->

### HTTP Client

<!--
Below is a table of HTTP client metric instruments.
-->

以下は、HTTPクライアントのMetric Instrumentsの表です。

<!--
| Name                   | Instrument    | Units        | Description |
|------------------------|---------------|--------------|-------------|
| `http.client.duration` | ValueRecorder | milliseconds | measure the duration of the outbound HTTP request |
-->

| Name                   | Instrument    | Units        | Description |
|------------------------|---------------|--------------|-------------|
| `http.client.duration` | ValueRecorder | milliseconds | 送信したHTTPリクエストの持続時間を測定 |

<!--
## Labels
-->

## Labels

<!--
Below is a table of the labels that SHOULD be included on `duration` metric events
and whether they should be on server, client, or both types of HTTP metric events:
-->

以下は、`duration`メトリックイベントに含めるべきラベルと、HTTPメトリックイベントのサーバー、クライアント、または両方のタイプに含めるべきラベルの表です。

<!--
| Name               | Type                | Recommended       | Notes and examples |
|--------------------|---------------------|-------------------|--------------------|
| `http.method`      | `client` & `server` | Yes               | The HTTP request method. E.g. `"GET"` |
| `http.host`        | `client` & `server` | see [label alternatives](#label-alternatives) | The value of the [HTTP host header][]. When the header is empty or not present, this label should be the same. |
| `http.scheme`      | `client` & `server` | see [label alternatives](#label-alternatives) | The URI scheme identifying the used protocol in lowercase: `"http"` or `"https"` |
| `http.status_code` | `client` & `server` | Optional          | [HTTP response status code][]. E.g. `200` (String) |
| `http.flavor`      | `client` & `server` | Optional          | Kind of HTTP protocol used: `"1.0"`, `"1.1"`, `"2"`, `"SPDY"` or `"QUIC"`. |
| `net.peer.name`    | `client`            | see [1] in [label alternatives](#label-alternatives) | See [general network connection attributes](../../trace/semantic_conventions/span-general.md#general-network-connection-attributes) |
| `net.peer.port`    | `client`            | see [1] in [label alternatives](#label-alternatives) | See [general network connection attributes](../../trace/semantic_conventions/span-general.md#general-network-connection-attributes) |
| `net.peer.ip`      | `client`            | see [1] in [label alternatives](#label-alternatives) | See [general network connection attributes](../../trace/semantic_conventions/span-general.md#general-network-connection-attributes) |
| `http.server_name` | `server`            | see [2] in [label alternatives](#label-alternatives) | The primary server name of the matched virtual host. This should be obtained via configuration. If no such configuration can be obtained, this label MUST NOT be set ( `net.host.name` should be used instead). |
| `net.host.name`    | `server`            | see [2] in [label alternatives](#label-alternatives) | See [general network connection attributes](../../trace/semantic_conventions/span-general.md#general-network-connection-attributes) |
| `net.host.port`    | `server`            | see [2] in [label alternatives](#label-alternatives) | See [general network connection attributes](../../trace/semantic_conventions/span-general.md#general-network-connection-attributes) |
-->

| Name               | Type                | Recommended       | Notes and examples |
|--------------------|---------------------|-------------------|--------------------|
| `http.method`      | `client` & `server` | Yes               | HTTPリクエストメソッド。例: `"GET"` |
| `http.host`        | `client` & `server` | [label alternatives](#label-alternatives)を参照 | [HTTPホストヘッダ][]の値です。ヘッダーが空であったり、存在しない場合、このラベルは同じものでなければなりません |
| `http.scheme`      | `client` & `server` | [label alternatives](#label-alternatives)を参照 | 使用するプロトコルを示すURIスキームを小文字で表記します。`"http"`または `"https"`となります |
| `http.status_code` | `client` & `server` | Optional          | [HTTPレスポンスステータスコード][]。例:`200`(文字列) |
| `http.flavor`      | `client` & `server` | Optional          | 使用するHTTPプロトコルの種類。`"1.0"`, `"1.1"`, `"2"`, `"SPDY"`, `"QUIC"` のいずれかになります |
| `net.peer.name`    | `client`            | [label alternatives](#label-alternatives)の[1]を参照 | [一般的なネットワーク接続属性](./../trace/semantic_conventions/span-general.md#general-network-connection-attributes)を参照 |
| `net.peer.port`    | `client`            | [label alternatives](#label-alternatives)の[1]を参照 | [一般的なネットワーク接続属性](./../trace/semantic_conventions/span-general.md#general-network-connection-attributes)を参照 |
| `net.peer.ip`      | `client`            | [label alternatives](#label-alternatives)の[1]を参照 | [一般的なネットワーク接続属性](./../trace/semantic_conventions/span-general.md#general-network-connection-attributes)を参照 |
| `http.server_name` | `server`            | [label alternatives](#label-alternatives)の[2]を参照 | 一致したバーチャルホストのプライマリサーバー名です。これは設定で得られるはずです。そのような設定が得られない場合は、このラベルを設定してはいけません(MUST NOT)(`net.host.name`を代わりに使うべきです) |
| `net.host.name`    | `server`            | [label alternatives](#label-alternatives)の[2]を参照 | [一般的なネットワーク接続属性](./../trace/semantic_conventions/span-general.md#general-network-connection-attributes)を参照 |
| `net.host.port`    | `server`            | [label alternatives](#label-alternatives)の[2]を参照 | [一般的なネットワーク接続属性](./../trace/semantic_conventions/span-general.md#general-network-connection-attributes)を参照 |

<!--
The following labels SHOULD be included in the `http.server.active_requests` observation:
-->

以下のラベルは `http.server.active_requests` の観測に含まれるべきです(SHOULD)。

<!--
| Name               | Recommended | Notes and examples |
|--------------------|-------------|--------------------|
| `http.method`      | Yes         | The HTTP request method. E.g. `"GET"` |
| `http.host`        | see [label alternatives](#label-alternatives) | The value of the [HTTP host header][]. When the header is empty or not present, this label should be the same |
| `http.scheme`      | see [label alternatives](#label-alternatives) | The URI scheme identifying the used protocol in lowercase: `"http"` or `"https"` |
| `http.flavor`      | Optional    | Kind of HTTP protocol used: `"1.0"`, `"1.1"`, `"2"`, `"SPDY"` or `"QUIC"` |
| `http.server_name` | see [2] in [label alternatives](#label-alternatives) | The primary server name of the matched virtual host. This should be obtained via configuration. If no such configuration can be obtained, this label MUST NOT be set ( `net.host.name` should be used instead). |
-->

| Name               | Recommended | Notes and examples |
|--------------------|-------------|--------------------|
| `http.method`      | Yes         | HTTPリクエストメソッド。例:`"GET"`。 |
| `http.host`        | [label alternatives](#label-alternatives)を参照 | [HTTPホストヘッダ][]の値です。ヘッダーが空であったり、存在しない場合、このラベルは同じものでなければなりません |
| `http.scheme`      | [label alternatives](#label-alternatives)を参照 | 使用するプロトコルを示すURIスキームを小文字で表記します。`"http"`または `"https"`となります |
| `http.flavor`      | Optional    | 使用するHTTPプロトコルの種類。`"1.0"`, `"1.1"`, `"2"`, `"SPDY"`, `"QUIC"` のいずれかになります |
| `http.server_name` | see [2] in [label alternatives](#label-alternatives) | 一致したバーチャルホストのプライマリサーバー名です。これは設定で得られるはずです。そのような設定が得られない場合は、このラベルを設定してはいけません(MUST NOT)(`net.host.name`を代わりに使うべきです)。 |

<!--
[HTTP host header]: https://tools.ietf.org/html/rfc7230#section-5.4
[HTTP response status code]: https://tools.ietf.org/html/rfc7231#section-6
[HTTP reason phrase]: https://tools.ietf.org/html/rfc7230#section-3.1.2
-->

[HTTPホストヘッダ]: https://tools.ietf.org/html/rfc7230#section-5.4
[HTTPレスポンスステータスコード]: https://tools.ietf.org/html/rfc7231#section-6
[HTTP reason phrase]: https://tools.ietf.org/html/rfc7230#section-3.1.2

<!--
### Parameterized labels
-->

### パラメータ化されたラベル

<!--
To avoid high cardinality the following labels SHOULD substitute any parameters when added as labels to http metric events as described below:
-->

カーディナリティが高くなるのを避けるため、以下のラベルは、以下のようにhttpメトリックイベントにラベルとして追加された場合、任意のパラメータに代わるべきです(SHOULD)。

<!--
| Label name        | Type                | Recommended |  Notes and examples |
|-------------------|---------------------|-------------|---------------------|
|`http.url`         | `client` & `server` | see [label alternatives](#label-alternatives) | The originally requested URL |
|`http.target`      | `client` & `server` | see [label alternatives](#label-alternatives) | The full request target as passed in a [HTTP request line][] or equivalent, e.g. `"/path/{id}/?q={}"`. |
-->

| Label name        | Type                | Recommended |  Notes and examples |
|-------------------|---------------------|-------------|---------------------|
|`http.url`         | `client` & `server` | [label alternatives](#label-alternatives)を参照 | リクエストされたオリジナルのURL |
|`http.target`      | `client` & `server` | [label alternatives](#label-alternatives)を参照 | [HTTPリクエストライン][]またはそれに相当するもので、例えば`"/path/{id}/?q={}"`のように渡される完全なリクエストターゲットです。 |

<!--
[HTTP request line]: https://tools.ietf.org/html/rfc7230#section-3.1.1
-->

[HTTPリクエストライン]: https://tools.ietf.org/html/rfc7230#section-3.1.1

<!--
Many REST APIs encode parameters into the URI path, e.g. `/api/users/123` where `123`
is a user id, which creates high cardinality value space not suitable for labels on metric events.
In case of HTTP servers, these endpoints are often mapped by the server
frameworks to more concise _HTTP routes_, e.g. `/api/users/{user_id}`, which are
recommended as the low cardinality label values. However, the same approach usually
does not work for HTTP client labels, especially when instrumentation is provided
by a lower-level middleware that is not aware of the specifics of how the URIs
are formed. Therefore, HTTP client labels SHOULD be using conservative, low
cardinality names formed from the available parameters of an HTTP request,
such as `"HTTP {METHOD_NAME}"`. These labels MUST NOT default to using URI
path.
-->

多くのREST APIは、パラメータをURIパスにエンコードしています。例えば、`/api/users/123`(`123`はユーザID)のように、高いカーディナリティの値空間を作りますが、これはメトリックイベントのラベルには適していません。HTTPサーバの場合、これらのエンドポイントは、サーバフレームワークによって、より簡潔な_HTTP route_、例えば`/api/users/{user_id}`にマッピングされることが多く、これは低いカーディナリティのラベル値として推奨されます。しかし、同じアプローチは通常、HTTPクライアントラベルでは機能しません。特に、URIがどのように形成されるかの詳細を認識していない低レベルのミドルウェアによって計装が提供される場合はそうです。したがって、HTTPクライアントラベルは、`"HTTP {METHOD_NAME}"`のような、HTTPリクエストの利用可能なパラメータから形成された、保守的で低いカーディナリティの名前を使用するべきです(SHOULD)。これらのラベルは、デフォルトでURIパスを使用してはなりません(MUST NOT)。

<!--
### Label alternatives
-->

### Label alternatives

<!--
**[1]** For client metric labels, one of the following sets of labels is RECOMMENDED (in order of usual preference unless for a particular web client/framework it is known that some other set is preferable for some reason; all strings must be non-empty):
-->

**[1]** クライアントのメトリックラベルについては、以下のラベルセットのいずれかが推奨されます(RECOMMENDED)(特定のWebクライアント/フレームワークについて、何らかの理由で他のセットが望ましいことがわかっている場合を除き、通常の優先順とします)。

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
**[2]** For server metric labels, `http.url` is usually not readily available on the server side but would have to be assembled in a cumbersome and sometimes lossy process from other information (see e.g. <https://github.com/open-telemetry/opentelemetry-python/pull/148>).
It is thus preferred to supply the raw data that *is* available.
Namely, one of the following sets is RECOMMENDED (in order of usual preference unless for a particular web server/framework it is known that some other set is preferable for some reason; all strings must be non-empty):
-->

**[2]** サーバーのメトリックラベルの場合、`http.url`は通常、サーバー側ではすぐには入手できず、他の情報から煩雑で時には損失の大きいプロセスで組み立てなければなりません(例えば、<https://github.com/open-telemetry/opentelemetry-python/pull/148>を参照)。したがって、利用可能な生データを提供することが望ましいです。すなわち、以下のセットのうちの1つが推奨されます(RECOMMENDED)(特定のウェブサーバ/フレームワークについて、何らかの理由で他のセットが望ましいことが知られている場合を除き、通常の優先順位の順で、すべての文字列は空であってはなりません)。

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

