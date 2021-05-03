<!--
# Default SDK Configuration
-->

# SDKのデフォルト設定

<details>
<summary>目次</summary>


<!--
* [Abstract](#abstract)
* [Configuration Interface](#configuration-interface)
-->

* [概要](#概要)
* [設定インターフェース](#設定インターフェース)


</details>

<!--
## Abstract
-->

## 概要

<!--
The default Open Telemetry SDK (hereafter referred to as "The SDK")
is highly configurable. This specification outlines the mechanisms by
which the SDK can be configured. It does
not attempt to specify the details of what can be configured.
-->

デフォルトのOpen Telemetry SDK(以下、SDK)は高度な設定が可能です。本仕様書では、SDKを設定するためのメカニズムの概要を説明しています。設定できる内容の詳細を規定するものではありません。

<!--
## Configuration Interface
-->

## 設定インターフェース

<!--
### Programmatic
-->

### プログラム的

<!--
The SDK MUST provide a programmatic interface for all configuration.
This interface SHOULD be written in the language of the SDK itself.
All other configuration mechanisms SHOULD be built on top of this interface.
-->

SDKは、すべての設定のためのプログラム的なインターフェースを提供しなければなりません(MUST)。このインターフェイスは、SDK自体の言語で書かれるべきです(SHOULD)。他のすべての設定メカニズムは、このインターフェースの上に構築されるべきです(SHOULD)。

<!--
An example of this programmatic interface is accepting a well-defined
struct on an SDK builder class. From that, one could build a CLI that accepts a
file (YAML, JSON, TOML, ...) and then transforms into that well-defined struct
consumable by the programatic interface.
-->

このプログラム・インターフェースの例として、SDKビルダー・クラスで定義された構造体を受け入れることが挙げられます。そこから、ファイル(YAML、JSON、TOMLなど)を受け取り、プログラム・インターフェースで消費可能な定義済みの構造体に変換するCLIを構築することができます。


<!--
### Other Mechanisms
-->

### 他のメカニズム

<!--
Additional configuration mechanisms SHOULD be provided in whatever
language/format/style is idiomatic for the language of the SDK. The
SDK can include as many configuration mechanisms as appropriate.
-->

追加の設定メカニズムは、SDKの言語に適した言語/フォーマット/スタイルで提供されるべきです(SHOULD)。SDKは適切な数の設定メカニズムを含むことができます。
