<!--
# General attributes
-->

# 一般的な属性

<!--
The attributes described in this section are not specific to a particular operation but rather generic.
They may be used in any Span they apply to.
Particular operations may refer to or require some of these attributes.
-->

このセクションで説明されている属性は、特定の操作に特化したものではなく、一般的なものです。これらの属性は、それらが適用されるどのSpanでも使用することができます。特定の操作は、これらの属性のいくつかを参照したり、必要としたりすることがあります。

<!-- Re-generate TOC with `markdown-toc --no-first-h1 -i` -->
<!-- toc -->

<!--
- [General network connection attributes](#general-network-connection-attributes)
- [General identity attributes](#general-identity-attributes)
-->

- [一般的なネットワークコネクション属性](#一般的なネットワークコネクション属性)
- [一般的な識別子属性](#一般的な識別子属性)


<!-- tocstop -->

<!--
## General network connection attributes
-->

## 一般的なネットワークコネクション属性

<!--
These attributes may be used for any network related operation.
The `net.peer.*` attributes describe properties of the remote end of the network connection
(usually the transport-layer peer, e.g. the node to which a TCP connection was established),
while the `net.host.*` properties describe the local end.
In an ideal situation, not accounting for proxies, multiple IP addresses or host names,
the `net.peer.*` properties of a client are equal to the `net.host.*` properties of the server and vice versa.
-->

これらの属性は、ネットワーク関連の操作に使用することができます。`net.peer.*` 属性はネットワーク接続のリモートエンド(通常はトランスポート層のピア、例えばTCP接続が確立されたノード)のプロパティを記述し、`net.host.*` プロパティはローカルエンドを記述します。理想的な状況では、プロキシや複数のIPアドレスやホスト名を考慮しない場合、クライアントの `net.peer.*` プロパティはサーバの `net.host.*` プロパティと等しくなり、その逆もまた然りです。

<!--
|  Attribute name  |                                 Notes and examples                                |
| :--------------- | :-------------------------------------------------------------------------------- |
| `net.transport` | Transport protocol used. See [note below](#net.transport).                         |
| `net.peer.ip`   | Remote address of the peer (dotted decimal for IPv4 or [RFC5952][] for IPv6)       |
| `net.peer.port` | Remote port number as an integer. E.g., `80`.                                      |
| `net.peer.name` | Remote hostname or similar, see [note below](#net.name).                           |
| `net.host.ip`   | Like `net.peer.ip` but for the host IP. Useful in case of a multi-IP host.         |
| `net.host.port` | Like `net.peer.port` but for the host port.                                        |
| `net.host.name` | Local hostname or similar, see [note below](#net.name).                            |
-->

|  属性名  |                                 説明と例                               |
| :--------------- | :--------------------------------------------------------------------------------|
| `net.transport` | 使用しているトランスポートプロトコル[以下の注意書きを参照](#net.transport)              |
| `net.peer.ip`   | ピアのリモートアドレス (`.` で区切られたIPv4あるいはIPv6には [RFC5952][])       |
| `net.peer.port` | リモートポート番号を整数で指定します。例：`80`                                         |
| `net.peer.name` | リモートホスト名またはそれに類するもの [以下の注意書きを参照](#net.name)                 |
| `net.host.ip`   | `net.peer.ip` のようなものですが、ホストの IP を指定します。複数のIPを持つホストの場合に便利です。 |
| `net.host.port` | `net.peer.port` のようなものですが、ホストのポートのためのものです                      |
| `net.host.name` | ローカルホスト名またはそれに類するもの[以下の注意書きを参照](#net.name)                 |

<!--
[RFC5952]: https://tools.ietf.org/html/rfc5952
-->

[RFC5952]: https://tools.ietf.org/html/rfc5952

<!--
<a name="net.transport"></a>
-->

<a name="net.transport"></a>

<!--
### `net.transport` attribute
-->

### `net.transport` 属性

<!--
This attribute should be set to the name of the transport layer protocol (or the relevant protocol below the "application protocol"). One of these strings should be used:
-->

この属性は、トランスポートレイヤープロトコル(または"application protocol"の下の関連するプロトコル)の名前に設定されるべきです(SHOULD)。これらの文字列のいずれかを使用しなければなりません(SHOULD)。

<!--
* `IP.TCP`
* `IP.UDP`
* `IP`: Another IP-based protocol.
* `Unix`: Unix Domain socket. See note below.
* `pipe`: Named or anonymous pipe. See note below.
* `inproc`: Signals that there is only in-process communication not using a "real" network protocol in cases where network attributes would normally be expected. Usually all other network attributes can be left out in that case.
* `other`: Something else (not IP-based).
-->

* `IP.TCP`
* `IP.UDP`
* `IP`: 他のIPベースのプロトコル
* `Unix`: Unix Domain socket。以下を参照のこと。
* `pipe`: Named あるいは anonymous pipe. 以下を参照のこと。
* `inproc`: ネットワーク属性が通常期待されるような「実際の」ネットワークプロトコルを使用しない。インプロセス通信のみを使うシグナルの場合(???あってるか自信なし)。通常、他のすべてのネットワーク属性は、この場合は除外することができます。
* `other`: その他 (IPベース以外)


<!--
For `Unix` and `pipe`, since the connection goes over the file system instead of being directly to a known peer, `net.peer.name` is the only attribute that usually makes sense (see description of `net.peer.name` below).
-->

`Unix` や `pipe` では、接続は既知の相手に直接接続するのではなくファイルシステムを経由するので、通常は `net.peer.name` が唯一の属性となります (後述の `net.peer.name` の説明を参照してください)。

<!--
<a name="net.name"></a>
-->

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

IPベースの通信の場合、名前はDNSホスト名でなければなりません(SHOULD)。`net.peer.name`の場合は、接続先のIPアドレスを調べるのに使われた名前の必要があります(SHOULD)(つまり、`net.peer.ip`が設定されていればそれにマッチする。例えば、URL `https://example.com/foo` に接続する場合は `"example.com"`)。IPアドレスのみでホスト名がない場合は、オプションでIPの逆引きを使用して取得しても構いません(MAY)。 `net.host.name` はローカルホストのホスト名で(SHOULD)、できればピアが現在の操作のために接続に使用したホスト名でなければなりません。もしそのホスト名が分からなければ、プライベートホスト名よりもパブリックホスト名の方が優先されます。 しかし、その場合、すでにリソースに含まれている情報と重複している可能性があり、取り残されてしまう可能性があります。通常、`net.host.name`を取得するために逆引きを使用することは意味がありません。これはリソース情報として保存される方が良い、静的な情報になってしまうからです。
<!--
If `net.transport` is `"unix"` or `"pipe"`, the absolute path to the file representing it should be used as `net.peer.name` (`net.host.name` doesn't make sense in that context).
If there is no such file (e.g., anonymous pipe),
the name should explicitly be set to the empty string to distinguish it from the case where the name is just unknown or not covered by the instrumentation.
-->

`net.transport` が `"unix"` または `"pipe"` の場合、それを表すファイルへの絶対パスは `net.peer.name` とするべきです(SHOULD) (この場合、`net.host.name` は意味をなさない)。そのようなファイルが存在しない場合(例えばanonymous pipe)は、名前が不明なだけの場合や計装でカバーされていない場合と区別するために、名前を明示的に空の文字列に設定する必要があります(SHOULD)。

<!--
## General identity attributes
-->

## 一般的な識別子属性

<!--
These attributes may be used for any operation with an authenticated and/or authorized enduser.
-->

これらの属性は、認証済みおよび/または認可されたエンドユーザとのあらゆる操作に使用することができます(MAY)。

<!--
|  Attribute name |                                 Notes and examples                                |
| :-------------- | :-------------------------------------------------------------------------------- |
| `enduser.id`    | Username or client_id extracted from the access token or [Authorization] header in the inbound request from outside the system.  |
| `enduser.role`  | Actual/assumed role the client is making the request under extracted from token or application security context. |
| `enduser.scope` | Scopes or granted authorities the client currently possesses extracted from token or application security context. The value would come from the scope associated with an [OAuth 2.0 Access Token] or an attribute value in a [SAML 2.0 Assertion]. |
-->

|  属性名 |                                 説明と例                                |
| :-------------- | :-------------------------------------------------------------------------------- |
| `enduser.id`    | システム外からのインバウンドリクエストのアクセストークン。あるいは[Authorization]ヘッダーから抽出されたユーザー名またはclient_id |
| `enduser.role`  | トークンまたはアプリケーションのセキュリティコンテキストから抽出された、クライアントがリクエストを行っている実際の/想定される役割 |
| `enduser.scope` | トークンまたはアプリケーションのセキュリティコンテキストから抽出された、クライアントが現在所有しているスコープまたは付与された権限。値は、[OAuth 2.0 Access Token]に関連付けられたスコープまたは[SAML 2.0 Assertion]の属性値から取得する |

<!--
These attributes describe the authenticated user driving the user agent making requests to the instrumented
system. It is expected this information would be propagated unchanged from node-to-node within the system
using the Correlation Context mechanism. These attributes should not be used to record system-to-system
authentication attributes.
-->

これらの属性は、計装されたシステムに要求を行うユーザーエージェントを実行する、認証されたユーザを示します。この情報は、Correlation Contextのメカニズムを使用して、システム内のノード間で変更されずに伝搬されることが期待されます。これらの属性は、システム間の認証属性を記録するために使用してはいけません(SHOULD NOT)。

<!--
Examples of where the `enduser.id` value is extracted from:
-->

`enduser.id` の値がどこから抽出されるかの例を示します。

<!--
| Authentication protocol | Field or description            |
| :---------------------- | :------------------------------ |
| [HTTP Basic/Digest Authentication] | `username`               |
| [OAuth 2.0 Bearer Token] | [OAuth 2.0 Client Identifier] value from `client_id` for the [OAuth 2.0 Client Credentials Grant] flow and `subject` or `username` from get token info response for other flows using opaque tokens. |
| [OpenID Connect 1.0 IDToken] | `sub` |
| [SAML 2.0 Assertion] | `urn:oasis:names:tc:SAML:2.0:assertion:Subject` |
| [Kerberos] | `PrincipalName` |
-->

| 認証プロトコル | フィールドまたは説明            |
| :---------------------- | :------------------------------ |
| [HTTP Basic/Digest Authentication] | `username`               |
| [OAuth 2.0 Bearer Token] | [OAuth 2.0 Client Credentials Grant]フローの場合は`client_id`から[OAuth 2.0 Client Identifier]の値を、その他のOpaqueトークンを使用するフローの場合はトークン情報を取得するレスポンスから `subject` または `username` を取得します |
| [OpenID Connect 1.0 IDToken] | `sub` |
| [SAML 2.0 Assertion] | `urn:oasis:names:tc:SAML:2.0:assertion:Subject` |
| [Kerberos] | `PrincipalName` |

<!--
| Framework               | Field or description            |
| :---------------------- | :------------------------------ |
| [JavaEE/JakartaEE Servlet] | `javax.servlet.http.HttpServletRequest.getUserPrincipal()` |
| [Windows Communication Foundation] | `ServiceSecurityContext.Current.PrimaryIdentity` |
-->

| フレームワーク               | フィールドまたは説明            |
| :---------------------- | :------------------------------ |
| [JavaEE/JakartaEE Servlet] | `javax.servlet.http.HttpServletRequest.getUserPrincipal()` |
| [Windows Communication Foundation] | `ServiceSecurityContext.Current.PrimaryIdentity` |


<!--
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
-->

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

この情報の機密性を考えると、SDKとExporterはデフォルトではこれらの属性を削除しておくべきです(SHOULD)。情報が必要でありいかなるポリシーや規制にも違反しないようなユースケースのために、保持を有効にするための設定パラメータを提供すべきです(SHOULD)。
