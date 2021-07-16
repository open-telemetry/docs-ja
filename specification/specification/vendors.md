<!--
# Vendors
-->

# ベンダー

<details>
<summary>目次</summary>

<!--
* [Abstract](#abstract)
* [Supports OpenTelemetry](#supports-opentelemetry)
* [Implements OpenTelemetry](#implements-opentelemetry)
* [Qualifications](#qualifications)
-->

* [概要](#概要)
* [Supports OpenTelemetry](#supports-opentelemetry)
* [Implements OpenTelemetry](#implements-opentelemetry)
* [資格](#資格)


</details>

<!--
## Abstract
-->

## 概要

<!--
The OpenTelemetry project consists of both a
[specification](https://github.com/open-telemetry/opentelemetry-specification)
for the API, SDK, protocol and semantic conventions, as well as an
implementation of each for a number of languages. The default SDK implementation
is [highly configurable](sdk-configuration.md) and extendable, for example
through [Span Processors](trace/sdk.md#span-processor), to allow for additional
logic needed by particular vendors to be added without having to implement a
custom SDK. By not requiring a custom SDK means for most languages a user will
already find an implementation to use and if not they'll have a well documented
specification to follow for implementing in a new language.
-->

OpenTelemetryプロジェクトは、API、SDK、プロトコル、セマンティックな規約の[仕様書](https://github.com/open-telemetry/opentelemetry-specification)と、いくつかの言語のためのそれぞれの実装の両方から構成されています。デフォルトのSDKの実装は、[高度な設定が可能](sdk-configuration.md)で拡張可能であり、例えば[Span Processor](trace/sdk.md#span-processor)を介して、特定のベンダーが必要とする追加ロジックを、カスタムSDKを実装することなく追加することができます。カスタムSDKを必要としないということは、ほとんどの言語において、ユーザーはすでに使用する実装を見つけることができ、もしそうでない場合は、新しい言語で実装するための十分に文書化された仕様に従うことができるということです。

<!--
The goal is for users to be able to easily switch between vendors while also
ensuring that any language with an OpenTelemetry SDK implementation is able to
work with any vendor who claims support for OpenTelemetry.
-->

その目的は、ユーザーがベンダーを簡単に切り替えられるようにすることと、OpenTelemetry SDKを実装した言語が、OpenTelemetryのサポートを表明しているどのベンダーでも動作することを保証することです。

<!--
This document will explain what is required of a vendor to be considered to
"Support OpenTelemetry" or "Implements OpenTelemetry".
-->

このドキュメントでは、ベンダーが「OpenTelemetryをサポート(Supports OpenTelemetry)」または「OpenTelemetryを実装(Implements OpenTelemetry)」しているとみなされるために必要なことを説明します。

<!--
## Supports OpenTelemetry
-->

## Supports OpenTelemetry

<!--
"Supports OpenTelemetry" means the vendor must accept the output of the default
SDK through one of two mechanisms:
-->

"Supports OpenTelemetry"とは、ベンダーが2つのメカニズムのうちの1つを通して、デフォルトのSDKの出力を受け入れる必要があることを意味します:

<!--
- By providing an exporter for the [OpenTelemetry Collector](https://github.com/open-telemetry/opentelemetry-collector/) and / or the OpenTelemetry SDKs
- By building a receiver for the [OpenTelemetry protocol](https://github.com/open-telemetry/opentelemetry-proto)
-->

- [OpenTelemetry Collector](https://github.com/open-telemetry/opentelemetry-collector/)および/またはOpenTelemetry SDK用のエクスポーターを提供する方法
- [OpenTelemetryプロトコル](https://github.com/open-telemetry/opentelemetry-proto)用のレシーバーを構築する方法


<!--
## Implements OpenTelemetry
-->

## Implements OpenTelemetry

<!--
A vendor with a custom SDK implementation will be listed as "Implements
OpenTelemetry". If the custom SDK is optional then the vendor can be listed as
"Supports OpenTelemetry".
-->

カスタムSDKを実装しているベンダーは「Implements OpenTelemetry」と表示されます。カスタムSDKが任意である場合、そのベンダーは「Supports OpenTelemetry」と表示されます。

<!--
## Qualifications
-->

## 資格

<!--
A vendor can qualify their support for OpenTelemetry with the type of telemetry
they support. For example, a vendor that accepts the OpenTelemetry protocol
exports for metrics only will be listed as "Supports OpenTelemetry Metrics" or
one that implements a custom SDK only for tracing will be listed as "Implements
OpenTelemetry Tracing".
-->

ベンダーは、サポートしているテレメトリの種類でOpenTelemetryのサポートという資格を得ることができます。例えば、OpenTelemetryプロトコルのエクスポートをメトリックのみで受け入れているベンダーは「Supports OpenTelemetry Metrics」と表示され、トレーシングのためだけにカスタムSDKを実装しているベンダーは「Implements OpenTelemetry Tracing」と表示されます。

