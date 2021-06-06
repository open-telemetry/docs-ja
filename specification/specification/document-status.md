<!--
# Definitions of Document Statuses
-->

# 文書ステータスの定義

<!--
Specification documents (files) may explicitly define a "Status", typically
shown immediately after the document title. When present, the "Status" applies
to the individual document only and not to the entire specification or any other
documents. The following table describes what the statuses mean.
-->

仕様書(ファイル)は、"ステータス"を明示的に定義することができます。この場合、"ステータス"は個々の文書にのみ適用され、仕様書全体や他の文書には適用されません。以下の表では、ステータスの意味を説明します。

<!--
## Lifecycle status
-->

## ライフサイクル ステータス

<!--
The support guarantees and allowed changes are governed by the lifecycle of the document.Lifecycle stages are defined in the [Versioning and Stability](versioning-and-stability.md) document.
-->

サポート保証と許可された変更は、ドキュメントのライフサイクルによって管理されています。

<!--
|Status              |Explanation|
|--------------------|-----------|
|No explicit "Status"|Equivalent to Experimental.|
|Experimental        |Breaking changes are allowed.|
|Stable              |Breaking changes are no longer allowed. See [stability guarantees](versioning-and-stability.md#stable) for details.|
|Deprecated          |Changes are no longer allowed, except for editorial changes.|
-->

|ステータス              |説明|
|-----------------------|-----------|
|明示的な"ステータス"なし|実験的と同じ|
|Experimental           |破壊的な変更が許可されている|
|Stable                 |破壊的な変更はできない。詳細は [安定保証](versioning-and-stability.md#stable) を参照してください|
|Deprecated             |編集上の変更を除き、変更はできない|

<!--
## Feature freeze
-->

## フィーチャーフリーズ

<!--
In addition to the statuses above, documents may be marked as `Feature-freeze`. These documents are not currently accepting new feature requests, to allow the Technical Committee time to focus on other areas of the specification. Editorial changes are still accepted. Changes that address production issues with existing features are still accepted.
-->

上記のステータスに加えて、文書には `Feature-freeze` というマークが付けられていることがあります。これらの文書は、技術委員会が仕様の他の分野に集中する時間を確保するため、現在は新しい機能要求を受け付けていない、ということを示します。編集上の変更はまだ受け付けています。既存機能の本番時に起きる問題に対処する変更は引き続き受け付けられています。

<!--
Feature freeze is separate from a lifecycle status. The lifecycle represents the support requirements for the document, feature freeze only indicates the current focus of the specification community. The feature freeze label may be applied to a document at any lifecycle stage. By definition, deprecated documents have a feature freeze in place.
-->

フィーチャーフリーズはライフサイクルの状態とは別物です。ライフサイクルは文書のサポート要件を表し、フィーチャフリーズは仕様コミュニティの現在取り組んでいる箇所を示すだけです。フィーチャーフリーズラベルは、どのライフサイクル段階でも文書に適用することができます。定義により、deprecatedの文書には、フィーチャフリーズが適用されます。

<!--
## Mixed
-->

## 混合 (`Mixed`)

<!--
Some documents have individual sections with different statuses. These documents are marked with the status `Mixed` at the top, for clarity.
-->

いくつかの文書には、異なるステータスを持つ個別のセクションがあります。これらの文書は、わかりやすくするために、上部に `Mixed` というステータスでマークされています。
