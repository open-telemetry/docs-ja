<!--
# Attribute and Label Naming
-->

# 属性とラベルの命名

<!--
**Status**: [Experimental](../document-status.md)
-->

**Status**: [Experimental](../document-status.md)

<details>
<summary>
目次
</summary>


<!--
- [Name Pluralization Guidelines](#name-pluralization-guidelines)
- [Recommendations for OpenTelemetry Authors](#recommendations-for-opentelemetry-authors)
- [Recommendations for Application Developers](#recommendations-for-application-developers)
-->

- [名前の複数化に関するガイドライン](#名前の複数化に関するガイドライン)
- [Recommendations for OpenTelemetry Authors](#recommendations-for-opentelemetry-authors)
- [Recommendations for Application Developers](#recommendations-for-application-developers)


</details>

<!--
_This section applies to Resource, Span and Log attribute names (also known as
the "attribute keys") and to keys of Metric labels. For brevity within this
section when we use the term "name" without an adjective it is implied to mean
"attribute name or metric label key"._
-->

_このセクションは、Resource、Span、Logの属性名(「属性キー」としても知られている)とメトリック・ラベルのキーに適用されます。このセクションを簡潔にするために、形容詞なしで「名前」という用語を使用する場合、それは「属性名またはメトリック・ラベル・キー」を意味します。

<!--
Every name MUST be a valid Unicode sequence.
-->

すべての名前は、有効なUnicode配列でなければなりません(MUST)。

<!--
_Note: we merely require that the names are represented as Unicode sequences.
This specification does not define how exactly the Unicode sequences are
encoded. The encoding can vary from one programming language to another and from
one wire format to another. Use the idiomatic way to represent Unicode in the
particular programming language or wire format._
-->

_注：ここでは、単に名前がUnicode配列として表現されていることを要求しています。この仕様では、Unicode配列をどのようにエンコードするかは定義していません。プログラミング言語やワイヤ・フォーマットによって、エンコードの仕方は異なります。プログラミング言語やワイヤフォーマットごとに、Unicodeを表現するための慣用的な方法を使用してください。

<!--
Names SHOULD follow these rules:
-->

名前は以下のルールに従うべきです(SHOULD):

<!--
- Use namespacing to avoid name clashes. Delimit the namespaces using a dot
  character. For example `service.version` denotes the service version where
  `service` is the namespace and `version` is an attribute in that namespace.
- Namespaces can be nested. For example `telemetry.sdk` is a namespace inside
  top-level `telemetry` namespace and `telemetry.sdk.name` is an attribute
  inside `telemetry.sdk` namespace.
  Note: the fact that an entity is located within another entity is typically
  not a sufficient reason for forming nested namespaces. The purpose of a
  namespace is to avoid name clashes, not to indicate entity hierarchies. This
  purpose should primarily drive the decision about forming nested namespaces.
- For each multi-word dot-delimited component of the attribute name separate the
  words by underscores (i.e. use snake_case). For example `http.status_code`
  denotes the status code in the http namespace.
- Names SHOULD NOT coincide with namespaces. For example if
  `service.instance.id` is an attribute name then it is no longer valid to have
  an attribute named `service.instance` because `service.instance` is already a
  namespace. Because of this rule be careful when choosing names: every existing
  name prohibits existence of an equally named namespace in the future, and vice
  versa: any existing namespace prohibits existence of an equally named
  attribute or label key in the future.
-->

- 名前の衝突を避けるために、名前空間を使用します。名前空間の区切りにはドット文字を使用します。例えば、`service.version` はサービスのバージョンを表し、`service` は名前空間で、`version` はその名前空間の属性です。
- 名前空間は入れ子にすることができます。例えば、`telemetry.sdk` はトップレベルの `telemetry` 名前空間の中の名前空間で、`telemetry.sdk.name` は `telemetry.sdk` 名前空間の中の属性です。  注意: あるエンティティが別のエンティティの中にあるという事実は、通常、入れ子の名前空間を形成する十分な理由にはなりません。名前空間の目的は、名前の衝突を避けることであり、エンティティの階層を示すことではありません。ネストしたネームスペースを形成するかどうかは、主にこの目的に基づいて決定されるべきです。
- 属性名の複数の単語をドットで区切った場合は、アンダースコアで単語を区切ります(例：snake_caseを使用)。例えば、`http.status_code`は、http名前空間のステータスコードを表します。
- 名前は名前空間と一致するべきではありません(SHOULD NOT)。例えば、`service.instance.id`が属性名である場合、`service.instance`という名前の属性はもはや有効ではありません。なぜなら、`service.instance`はすでに名前空間だからです。このルールにより、名前を選択する際には注意が必要です。既存の名前は将来的に同じ名前の名前空間の存在を禁止し、逆に既存の名前空間は将来的に同じ名前の属性やラベルキーの存在を禁止します。

<!--
## Name Pluralization guidelines
-->

## 名前の複数化に関するガイドライン

<!--
- When an attribute represents a single entity, the attribute name SHOULD be singular.
  Examples: `host.name`, `db.user`, `container.id`.
- When attribute can represent multiple entities, the attribute name SHOULD be pluralized
  and the value type SHOULD be an array. E.g. `process.command_args` might include multiple
  values: the executable name and command arguments.
- When an attribute represents a measurement,
  [Metric Name Pluralization Guidelines](../metrics/semantic_conventions/README.md#pluralization)
  SHOULD be followed for the attribute name.
-->

- 属性が単一のエンティティを表す場合、属性名は単数形であるべきです(SHOULD)。例 host.name`, `db.user`, `container.id`。
- 属性が複数のエンティティを表すことができる場合、属性名は複数化されるべき(SHOULD)で、値の型は配列であるべきです(SHOULD)。例えば、`process.command_args` は実行ファイル名とコマンド引数という複数の値を含むかもしれません。
- 属性がMeasurementを表す場合、属性名は[Metric名の複数化ガイドライン](../metrics/semantic_conventions/README.md#pluralization)に従うべきです(SHOULD)。

<!--
## Recommendations for OpenTelemetry Authors
-->

## OpenTelemetryの作者への推奨事項

<!--
- All names that are part of OpenTelemetry semantic conventions SHOULD be part
  of a namespace.
- When coming up with a new semantic convention make sure to check existing
  namespaces for
  [Resources](../resource/semantic_conventions/README.md),
  [Spans](../trace/semantic_conventions/README.md),
  and
  [Metrics](../metrics/semantic_conventions/README.md)
  to see if the new name fits.
- When a new namespace is necessary consider whether it should be a top-level
  namespace (e.g. `service`) or a nested namespace (e.g. `service.instance`).
- Semantic conventions exist for four areas: for Resource, Span and Log
  attribute names as well as Metric label keys. In addition, for spans we have
  two more areas: Event and Link attribute names. Identical namespaces or names
  in all these areas MUST have identical meanings. For example the `http.method`
  span attribute name denotes exactly the same concept as the `http.method`
  metric label, has the same data type and the same set of possible values (in
  both cases it records the value of the HTTP protocol's request method as a
  string).
- Semantic conventions MUST limit names to printable Basic Latin characters
  (more precisely to
  [U+0021 .. U+007E](https://en.wikipedia.org/wiki/Basic_Latin_(Unicode_block)#Table_of_characters)
  subset of Unicode code points). It is recommended to further limit names to
  the following Unicode code points: Latin alphabet, Numeric, Underscore, Dot
  (as namespace delimiter).
-->

- OpenTelemetryのセマンティック規約の一部であるすべての名前は、名前空間の一部であるべきです(SHOULD)。
- 新しいセマンティック規約を考え出すときには、既存の名前空間である[Resources](../resource/semantic_conventions/README.md)、[Span](../trace/semantic_conventions/README.md)、[Metrics](../metrics/semantic_conventions/README.md)をチェックして、新しい名前が適合するかどうかを確認してください。
- 新しい名前空間が必要になったときには、トップレベルの名前空間(例：`service`)にするか、ネストした名前空間(例：`service.instance`)にするかを検討してください

- Resource、Span、Logの属性名とMetricラベルキーの4つの領域でセマンティック規約が存在します。また、Spanについては、さらに2つの領域があります。EventとLinkの属性名です。これらすべての領域で同一の名前空間または名前は、同一の意味を持たなければなりません(MUST)。例えば、`http.method`というSpan 属性名は、`http.method`というMetric・ラベルと全く同じ概念を表しており、同じデータ・タイプ、同じ値のセットを持っています(どちらの場合も、HTTPプロトコルのリクエスト・メソッドの値を文字列として記録します)。
- セマンティック規約では、名前を印刷可能な基本ラテン文字(より正確には、[U+0021 .. U+007E](https://en.wikipedia.org/wiki/Basic_Latin_(Unicode_block)#Table_of_characters)のUnicodeコードポイントのサブセット)に制限しなければなりません(MUST)。さらに、名前を以下のUnicodeコードポイントに限定することを推奨します。ラテン・アルファベット、数字、アンダースコア、ドット(名前空間の区切り文字として)。

<!--
## Recommendations for Application Developers
-->

## アプリケーション開発者に対する推奨事項

<!--
As an application developer when you need to record an attribute or a label
first consult existing semantic conventions for
[Resources](../resource/semantic_conventions/README.md),
[Spans](../trace/semantic_conventions/README.md),
and
[Metrics](../metrics/semantic_conventions/README.md).
If an appropriate name does not exists you will need to come up with a new name.
To do that consider a few options:
-->

アプリケーション開発者として、属性やラベルを記録する必要があるときは、まず[Resource](../resource/semantic_conventions/README.md)、[Span](../trace/semantic_conventions/README.md)、[Metrics](../metrics/semantic_conventions/README.md)の既存のセマンティック規約を参照してください。適切な名前が存在しない場合は、新しい名前を考える必要があります。そのためには、いくつかの事項を検討します。

<!--
- The name is specific to your company and may be possibly used outside the
  company as well. To avoid clashes with names introduced by other companies (in
  a distributed system that uses applications from multiple vendors) it is
  recommended to prefix the new name by your company's reverse domain name, e.g.
  `com.acme.shopname`.
- The name is specific to your application that will be used internally only. If
  you already have an internal company process that helps you to ensure no name
  clashes happen then feel free to follow it. Otherwise it is recommended to
  prefix the attribute name or label key by your application name, provided that
  the application name is reasonably unique within your organization (e.g.
  `myuniquemapapp.longitude` is likely fine). Make sure the application name
  does not clash with an existing semantic convention namespace.
- The name may be generally applicable to applications in the industry. In that
  case consider submitting a proposal to this specification to add a new name to
  the semantic conventions, and if necessary also to add a new namespace.
-->

- この名前はあなたの会社に固有のものであり、会社の外でも使用される可能性がある場合、他の会社が導入した名前との衝突を避けるために(複数のベンダーのアプリケーションを使用する分散型システムの場合)、新しい名前の前にあなたの会社のリバースドメイン名(例：`com.acme.shopname`)を付けることをお勧めします。
- この名前は、社内でのみ使用される、あなたのアプリケーションに固有のものの場合、もし、名前の衝突が起こらないようにするための社内プロセスが既にあるのであれば、それに従っていただいて構いません。そうでない場合は、属性名やラベルキーの前にアプリケーション名を付けることをお勧めします。ただし、アプリケーション名が組織内で適度にユニークであることが条件です(例：`myuniquemapapp.longitude`なら問題ないでしょう)。アプリケーション名が既存のセマンティック規約の名前空間と衝突しないようにしてください。
- この名前は、その業界のアプリケーションに一般的に適用できるかもしれなません。その場合は、セマンティック規約に新しい名前を追加し、必要に応じて新しい名前空間を追加する提案を本仕様書に提出することを検討してください。

<!--
It is recommended to limit names to printable Basic Latin characters
(more precisely to
[U+0021 .. U+007E](https://en.wikipedia.org/wiki/Basic_Latin_(Unicode_block)#Table_of_characters)
subset of Unicode code points).
-->

名前は印刷可能な基本ラテン文字(正確には [U+0021 .. U+007E](https://en.wikipedia.org/wiki/Basic_Latin_(Unicode_block)#Table_of_characters) サブセットの Unicode コードポイント)に限定することを推奨します。

