<!--
# General attributes
-->
# 一般的な属性

**Status**: [Experimental](../../document-status.md)

<!--
The attributes described in this section are not specific to a particular operation but rather generic.
They may be used in any Span they apply to.
Particular operations may refer to or require some of these attributes.
-->

このセクションで説明する属性は、特定のオペレーションに特化したものではなく、一般的なものです。これらの属性は、適用されるあらゆるSpanで使用することができます。特定の操作では、これらの属性の一部を参照したり、必要とする場合があります。

<!-- Re-generate TOC with `markdown-toc --no-first-h1 -i` -->

<!-- toc -->

<!--
- [General network connection attributes](#general-network-connection-attributes)
  * [`net.transport` attribute](#nettransport-attribute)
  * [`net.*.name` attributes](#netname-attributes)
- [General remote service attributes](#general-remote-service-attributes)
- [General identity attributes](#general-identity-attributes)
- [General thread attributes](#general-thread-attributes)
- [Source Code Attributes](#source-code-attributes)
-->

- [一般的なネットワーク接続属性](#一般的なネットワーク接続属性)
  * [`net.transport` 属性](#nettransport-属性)
  * [`net.*.name` 属性](#netname-属性)
- [一般的なリモートサービスの属性](#一般的なリモートサービスの属性)
- [一般的な識別子属性](#一般的な識別子属性)
- [一般的なスレッドの属性](#一般的なスレッドの属性)
- [ソースコードの属性](#ソースコードの属性)

<!-- tocstop -->

<!--
## General network connection attributes
-->

## 一般的なネットワーク接続属性


<!--
These attributes may be used for any network related operation.
The `net.peer.*` attributes describe properties of the remote end of the network connection
(usually the transport-layer peer, e.g. the node to which a TCP connection was established),
while the `net.host.*` properties describe the local end.
In an ideal situation, not accounting for proxies, multiple IP addresses or host names,
the `net.peer.*` properties of a client are equal to the `net.host.*` properties of the server and vice versa.
-->

これらの属性は、ネットワークに関連するあらゆる操作に使用できます。`net.peer.*` 属性は、ネットワーク接続のリモート側(通常はトランスポート層のピアで、TCP接続が確立されたノードなど)のプロパティを記述し、`net.host.*` 属性は、ローカル側のプロパティを記述します。理想的な状況では、プロキシや複数のIPアドレス、ホスト名などは考慮されません。クライアントの `net.peer.*` プロパティは、サーバの `net.host.*` プロパティと等しく、その逆もまた同様です。

<a name="nettransport-attribute">

<!-- semconv network -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `net.transport` | string | Transport protocol used. 下記注釈参照。 | `ip_tcp` | No |
| `net.peer.ip` | string | 相手のリモートアドレス(IPv4ではドット10進数、IPv6では[RFC5952](https://tools.ietf.org/html/rfc5952) | `127.0.0.1` | No |
| `net.peer.port` | int | リモートのポート番号 | `80`; `8080`; `443` | No |
| `net.peer.name` | string | リモートのホスト名あるいは類似の文字列。下記注釈参照 | `example.com` | No |
| `net.host.ip` | string | `net.peer.ip`と同じですが、ホストのIPを指定します。マルチIPのホストの場合に便利です。 | `192.168.0.1` | No |
| `net.host.port` | int | `net.peer.port`と同じですが、ホストのポートを指定します。 | `35555` | No |
| `net.host.name` | string | ローカルのホスト名あるいは類似の文字列 下記注釈参照 | `localhost` | No |

`net.transport` MUST be one of the following:

| Value  | Description |
|---|---|
| `ip_tcp` | ip_tcp |
| `ip_udp` | ip_udp |
| `ip` | 他の IP-based プロトコル |
| `unix` | Unix Domain socket. 下記参照 |
| `pipe` | Named あるいは anonymous pipe. 下記注釈参照 |
| `inproc` | In-process communication. [1] |
| `other` | その他 (non IP-based). |

**[1]:** 通常、ネットワークの属性が期待される場合に、「実際の」ネットワークプロトコルを使用しないin-processの通信のみが存在することを示します。通常、この場合、他のネットワーク属性はすべて省くことができます。

<!-- endsemconv -->

<!--
For `Unix` and `pipe`, since the connection goes over the file system instead of being directly to a known peer, `net.peer.name` is the only attribute that usually makes sense (see description of `net.peer.name` below).
-->

`Unix` や `pipe` では、既知のピアに直接接続するのではなく、ファイルシステムを介して接続するため、`net.peer.name` が通常意味を持つ唯一の属性となります (後述の `net.peer.name` の説明を参照)。

<a name="net.name"></a>

<!--
### `net.*.name` attributes
-->

### `net.*.name` 属性

<!--
For IP-based communication, the name should be a DNS host name.
For `net.peer.name`, this should be the name that was used to look up the IP address that was connected to
(i.e., matching `net.peer.ip` if that one is set; e.g., `"example.com"` if connecting to an URL `https://example.com/foo`).
If only the IP address but no host name is available, reverse-lookup of the IP may optionally be used to obtain it.
`net.host.name` should be the host name of the local host,
preferably the one that the peer used to connect for the current operation.
If that is not known, a public hostname should be preferred over a private one. However, in that case it may be redundant with information already contained in resources and may be left out.
It will usually not make sense to use reverse-lookup to obtain `net.host.name`, as that would result in static information that is better stored as resource information.
-->

IPベースの通信の場合、名前はDNSのホスト名でなければなりません。`net.peer.name`には、接続先のIPアドレスを調べるのに使用した名前を指定してください(`net.peer.ip`が設定されている場合はそれに一致します)。IPアドレスのみでホスト名がない場合は、オプションでIPの逆引きを使用して取得することができます。net.host.name`には、ローカルホストのホスト名、できれば相手が現在の操作で接続に使用したホスト名を指定してください。それがわからない場合は、プライベートなホスト名よりもパブリックなホスト名を使用することをお勧めします。ただし、その場合は、リソースにすでに含まれている情報と重複する可能性があるため、省略することができます。通常、`net.host.name`を取得するために逆引きを使用することは意味がありません。それは、リソース情報として保存することが望ましい静的な情報になってしまうからです。

<!--
If `net.transport` is `"unix"` or `"pipe"`, the absolute path to the file representing it should be used as `net.peer.name` (`net.host.name` doesn't make sense in that context).
If there is no such file (e.g., anonymous pipe),
the name should explicitly be set to the empty string to distinguish it from the case where the name is just unknown or not covered by the instrumentation.
-->

`net.transport` が `"unix"` または `"pipe"` の場合、それを表すファイルの絶対パスを `net.peer.name` として使用する必要があります (この文脈では `net.host.name` は意味を持ちません)。そのようなファイルがない場合(例:anonymous pipe)は、名前が不明であったり、計装の対象外であったりする場合と区別するために、名前に空の文字列を明示的に設定する必要があります。

<!--
## General remote service attributes
-->

## 一般的なリモートサービスの属性

<!--
This attribute may be used for any operation that accesses some remote service.
Users can define what the name of a service is based on their particular semantics in their distributed system.
Instrumentations SHOULD provide a way for users to configure this name.
-->

この属性は、あるリモートサービスにアクセスするすべての操作に使用することができます。ユーザーは、分散システムにおける特定のセマンティクスに基づいて、サービスの名前を定義できます。計装は、ユーザーがこの名前を設定する方法を提供すべきです(SHOULD)。

<!-- semconv peer -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `peer.service` | string | リモートサービスの[`service.name`](../../resource/semantic_conventions/README.md#service)を指定します。リモートサービスの実際の `service.name` リソース属性があれば、それと等しくすべきです(SHOULD)。 | `AuthTokenCache` | No |
<!-- endsemconv -->

<!--
Examples of `peer.service` that users may specify:
-->

ユーザーが指定できる `peer.service` の例です。

<!--
- A Redis cache of auth tokens as `peer.service="AuthTokenCache"`.
- A gRPC service `rpc.service="io.opentelemetry.AuthService"` may be hosted in both a gateway, `peer.service="ExternalApiService"` and a backend, `peer.service="AuthService"`.
-->

- Redisによる認証トークンのキャッシュは `peer.service="AuthTokenCache"` となります。
- gRPCサービス `rpc.service="io.opentelemetry.AuthService"` は、ゲートウェイである `peer.service="ExternalApiService"` とバックエンドである `peer.service="AuthService"` の両方でホストすることができます。

<!--
## General identity attributes
-->

## 一般的なアイデンティティ属性

<!--
These attributes may be used for any operation with an authenticated and/or authorized enduser.
-->

これらの属性は、認証および/または認可されたエンドユーザーとのあらゆる操作に使用できます。

<!-- semconv identity -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `enduser.id` | string | システム外からのインバウンドリクエストのアクセストークンまたは[Authorization](https://tools.ietf.org/html/rfc7235#section-4.2)ヘッダーから抽出したユーザー名またはclient_id。 | `username` | No |
| `enduser.role` | string | トークンやアプリケーションのセキュリティコンテキストから抽出された、クライアントがリクエストを行う際の実際の役割/想定される役割。 | `admin` | No |
| `enduser.scope` | string | トークンやアプリケーションのセキュリティコンテキストから抽出された、クライアントが現在所有しているスコープや付与された権限。この値は、[OAuth 2.0 Access Token](https://tools.ietf.org/html/rfc6749#section-3.3)に関連付けられたスコープ、または[SAML 2.0 Assertion](http://docs.oasis-open.org/security/saml/Post2.0/sstc-saml-tech-overview-2.0.html)の属性値に由来するものです。 | `read:message, write:files` | No |
<!-- endsemconv -->

<!--
These attributes describe the authenticated user driving the user agent making requests to the instrumented
system. It is expected this information would be propagated unchanged from node-to-node within the system
using the Baggage mechanism. These attributes should not be used to record system-to-system
authentication attributes.
-->

これらの属性は、計装されたシステムへのリクエストを行うユーザー・エージェントを駆動する認証済みユーザーを表しています。この情報は、Baggageの機構を使用して、システム内のノード間で変更されずに伝搬されることが期待されます。これらの属性は、システム間の認証属性を記録するために使用すべきではありません。

<!--
Examples of where the `enduser.id` value is extracted from:
-->

`enduser.id` の値がどこから抽出されるかの例:

| 認証プロトコル | フィールド名あるいは説明            |
| :---------------------- | :------------------------------ |
| [HTTP Basic/Digest Authentication] | `username`               |
| [OAuth 2.0 Bearer Token] | [OAuth 2.0 Client Credentials Grant]フローでは、`client_id`から[OAuth 2.0 Client Identifier]の値を、不透明なトークンを使用する他のフローでは、get token infoレスポンスから`subject`または`username`を取得します。 |
| [OpenID Connect 1.0 IDToken] | `sub` |
| [SAML 2.0 Assertion] | `urn:oasis:names:tc:SAML:2.0:assertion:Subject` |
| [Kerberos] | `PrincipalName` |

| Framework               | Field or description            |
| :---------------------- | :------------------------------ |
| [JavaEE/JakartaEE Servlet] | `javax.servlet.http.HttpServletRequest.getUserPrincipal()` |
| [Windows Communication Foundation] | `ServiceSecurityContext.Current.PrimaryIdentity` |

[Authorization]: https://tools.ietf.org/html/rfc7235#section-4.2
[OAuth 2.0 Access Token]: https://tools.ietf.org/html/rfc6749#section-3.3
[SAML 2.0 Assertion]: http://docs.oasis-open.org/security/saml/Post2.0/sstc-saml-tech-overview-2.0.html
[HTTP Basic/Digest Authentication]: https://tools.ietf.org/html/rfc2617
[OAuth 2.0 Bearer Token]: https://tools.ietf.org/html/rfc6750
[OAuth 2.0 Client Identifier]: https://tools.ietf.org/html/rfc6749#section-2.2
[OAuth 2.0 Client Credentials Grant]: https://tools.ietf.org/html/rfc6749#section-4.4
[OpenID Connect 1.0 IDToken]: https://openid.net/specs/openid-connect-core-1_0.html#IDToken
[Kerberos]: https://tools.ietf.org/html/rfc4120
[JavaEE/JakartaEE Servlet]: https://jakarta.ee/specifications/platform/8/apidocs/javax/servlet/http/HttpServletRequest.html
[Windows Communication Foundation]: https://docs.microsoft.com/en-us/dotnet/api/system.servicemodel.servicesecuritycontext?view=netframework-4.8

<!--
Given the sensitive nature of this information, SDKs and exporters SHOULD drop these attributes by
default and then provide a configuration parameter to turn on retention for use cases where the
information is required and would not violate any policies or regulations.
-->

この情報の機密性を考慮すると、SDKとエクスポーターは、これらの属性をデフォルトで削除し、情報が必要なユースケースでは保持をオンにする設定パラメーターを提供すべきです(SHOULD)。
情報が必要で、ポリシーや規制に違反しないユースケースでは、保持をオンにするための設定パラメータを提供すべきです。

<!--
## General thread attributes
-->

## 一般的なスレッドの属性

<!--
These attributes may be used for any operation to store information about
a thread that started a span.
-->

これらの属性は、Spanを開始したスレッドに関する情報を保存するためのあらゆる操作に使用できます。


<!-- semconv thread -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `thread.id` | int | 現在の「管理された」スレッドID(OSのスレッドIDとは異なります)。 | `42` | No |
| `thread.name` | string | 現在のスレッド名 | `main` | No |
<!-- endsemconv -->

<!--
Examples of where `thread.id` and `thread.name` can be extracted from:
-->

`thread.id`と`thread.name`を抽出することができる例:

| Launguage or platform | `thread.id`                            | `thread.name`                      |
|-----------------------|----------------------------------------|------------------------------------|
| JVM                   | `Thread.currentThread().getId()`       | `Thread.currentThread().getName()` |
| .Net                  | `Thread.CurrentThread.ManagedThreadId` | `Thread.CurrentThread.Name`        |
| Python                | `threading.current_thread().ident`     | `threading.current_thread().name`  |
| Ruby                  |                                        | `Thread.current.name`              |
| C++                   | `std::this_thread::get_id()`             |                                    |
| Erlang               | `erlang:system_info(scheduler_id)` |                                  |

<!--
## Source Code Attributes
-->

## ソースコードの属性


<!--
Often a span is closely tied to a certain unit of code that is logically responsible for handling
the operation that the span describes (usually the method that starts the span).
For an HTTP server span, this would be the function that handles the incoming request, for example.
The attributes listed below allow to report this unit of code and therefore to provide more context
about the span.
-->

多くの場合、Spanは、そのSpanが記述している操作を論理的に処理する責任を持つ、特定のコードユニットと密接に結びついています(通常は、Spanを開始するメソッド)。HTTPサーバーの場合、これは例えば、受信したリクエストを処理する関数となります。以下の属性は、このコードのユニットを報告し、その結果、Spanに関するより多くのコンテキストを提供することができます。

<!-- semconv code -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `code.function` | string | メソッド名、関数名、またはそれに相当するもの(通常はコードユニット名の最右端)。 | `serveRequest` | No |
| `code.namespace` | string | `code.function`が定義されている「名前空間」です。通常は、修飾されたクラスやモジュールの名前で、 `code.namespace` + 何かの区切り文字 + `code.function` がコードユニットの一意な識別子になります。 | `com.example.MyHttpService` | No |
| `code.filepath` | string | コードユニットを可能な限り一意に識別するためのソースコードファイル名(絶対ファイルパスが望ましい)。 | `/usr/local/MyApplication/content_root/app/index.php` | No |
| `code.lineno` | int | `code.filepath` の中で、操作を表すのに最適な行番号です。これは `code.function` で指定されたコードユニット内を指すべきです。 | `42` | No |
<!-- endsemconv -->
