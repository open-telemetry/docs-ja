<!--
# Default SDK Configuration
-->

# デフォルト SDK 設定

<!--
<details>
<summary>Table of Contents</summary>
-->

<details>
<summary>
目次
</summary>

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

デフォルトの Open Telemetry SDK(以下、「SDK」と呼ぶ)は高度に設定可能です。本仕様書は、SDKが可能な設定方法を概説するものです。設定可能な内容の詳細を指定するものではありません。

<!--
## Configuration Interface
-->

## 設定インターフェース

<!--
### Programmatic
-->

### プログラムからの設定

<!--
The SDK MUST provide a programmatic interface for all configuration.
This interface SHOULD be written in the language of the SDK itself.
All other configuration mechanisms SHOULD be built on top of this interface.
-->

SDKは、すべての設定のために、プログラムから呼び出せるインターフェースを提供しなければなりません(MUST)。このインターフェースはSDK自体の言語で書かれるべきです(SHOULD)。他のすべての設定方法は、このインターフェースの上に構築されるべきです(SHOULD)。

<!--
An example of this programmatic interface is accepting a well-defined
struct on an SDK builder class. From that, one could build a CLI that accepts a
file (YAML, JSON, TOML, ...) and then transforms into that well-defined struct
consumable by the programatic interface.
-->

このプログラム的インターフェースの例としては、SDKビルダークラスで定義済みの構造体を受け入れることが挙げられます。これにより、ファイル (YAML, JSON, TOML, ....) を受け入れて、プログラムから呼び出せる定義済み構造体に変換する CLI を構築することができます。

<!--
### Other Mechanisms
-->

### 他の設定方法

<!--
Additional configuration mechanisms SHOULD be provided in whatever
language/format/style is idiomatic for the language of the SDK. The
SDK can include as many configuration mechanisms as appropriate.
-->

追加の設定方法は、SDKの言語に適した言語/フォーマット/スタイルで提供されるべきです(SHOULD)。SDKは、適切なだけ多くの設定メカニズムを含めることができます。
