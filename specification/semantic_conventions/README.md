<!--
# YAML Model for Semantic Conventions
-->

# セマンティック規約のためのYAMLモデル

<!--
The YAML descriptions of semantic convention contained in this directory are intended to
be used by the various OpenTelemetry language implementations to aid in automatic
generation of semantics-related code.
-->

このディレクトリに含まれるセマンティック規約のYAML記述は、セマンティック関連のコードの自動生成を支援するために、様々なOpenTelemetry言語の実装によって使用されることを意図しています。

<!--
## Generation
-->

## 世代

<!--
These YAML files are used by the make target `table-generation` to generate consistently
formattted Markdown tables for all semantic conventions in the specification. Run it from the root of this repository using the command
-->

これらのYAMLファイルはmakeターゲットの`table-generation`によって仕様のすべてのセマンティック規約のために、一貫してフォーマットされたMarkdownテーブルを生成するために使われます。このリポジトリの直下からコマンドを使って実行してください。


```
make table-generation
```

<!--
For more information, see the [semantic convention generator](https://github.com/open-telemetry/build-tools/tree/main/semantic-conventions)
in the OpenTelemetry build tools repository.
Using this build tool, it is also possible to generate code for use in OpenTelemetry
language projects.
-->

詳細については、OpenTelemetry ビルドツールのリポジトリにある [セマンティック規約ジェネレータ](https://github.com/open-telemetry/build-tools/tree/main/semantic-conventions) を参照してください。このビルドツールを使用すると、OpenTelemetry言語プロジェクトで使用するコードを生成することも可能です。

<!--
See also:
-->

こちらも参考:

<!--
* [Markdown Tables](https://github.com/open-telemetry/build-tools/tree/main/semantic-conventions#markdown-tables)
* [Code Generator](https://github.com/open-telemetry/build-tools/tree/main/semantic-conventions#code-generator)
-->

* [Markdownテーブル](https://github.com/open-telemetry/build-tools/tree/main/semantic-conventions#markdown-tables)
* [コード生成](https://github.com/open-telemetry/build-tools/tree/main/semantic-conventions#code-generator)

<!--
## Description of the model
-->

## モデルの説明

<!--
The fields and their expected values are presented in [syntax.md](./syntax.md).
-->

フィールドとその期待値は、[syntax.md] (./syntax.md)で示されています。

