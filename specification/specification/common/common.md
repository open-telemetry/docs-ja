<!--
# Common specification concepts
-->

# 共通な仕様のコンセプト

**Status**: [Stable, Feature-freeze](../document-status.md)

<details>
<summary>
目次
</summary>

<!--
- [Attributes](#attributes)
-->

- [属性](#属性)

</details>

<!--
## Attributes
-->

## 属性

<!--
Attributes are a list of zero or more key-value pairs. An `Attribute` MUST have the following properties:
-->

属性は、0個以上のキーと値のペアのリストです。`アトリビュート`は以下のプロパティを持たなければなりません(MUST)。

<!--
- The attribute key, which MUST be a non-`null` and non-empty string.
- The attribute value, which is either:
  - A primitive type: string, boolean, double precision floating point (IEEE 754-1985) or signed 64 bit integer.
  - An array of primitive type values. The array MUST be homogeneous,
    i.e. it MUST NOT contain values of different types. For protocols that do
    not natively support array values such values SHOULD be represented as JSON strings.
-->

- 属性のキーは、非`null`かつ非空の文字列でなければなりません(MUST)。
- 属性の値は、下のいずれかの値です:
  - プリミティブタイプ:string、boolean、倍精度浮動小数点(IEEE 754-1985)、符号付き64ビット整数
  - プリミティブ型の値の配列。配列内はすべて同じ型でなければならない(MUST)。つまり、異なるタイプの値を含んではならない(MUST NOT)。配列値をネイティブにサポートしていないプロトコルでは、そのような値はJSON文字列として表現されるべきです(SHOULD)。

<!--
Attribute values expressing a numerical value of zero, an empty string, or an
empty array are considered meaningful and MUST be stored and passed on to
processors / exporters.
-->

ゼロの数値、空の文字列、または空の配列を表現する属性値は、意味があるとみなされ、保存されてProcessorやExporterに渡されなければなりません(MUST)。

<!--
Attribute values of `null` are not valid and attempting to set a `null` value is
undefined behavior.
-->

`null`の属性値は有効ではなく、`null`の値を設定しようとするのは未定義の動作です。

<!--
`null` values SHOULD NOT be allowed in arrays. However, if it is impossible to
make sure that no `null` values are accepted
(e.g. in languages that do not have appropriate compile-time type checking),
`null` values within arrays MUST be preserved as-is (i.e., passed on to span
processors / exporters as `null`). If exporters do not support exporting `null`
values, they MAY replace those values by 0, `false`, or empty strings.
This is required for map/dictionary structures represented as two arrays with
indices that are kept in sync (e.g., two attributes `header_keys` and `header_values`,
both containing an array of strings to represent a mapping
`header_keys[i] -> header_values[i]`).
-->

配列の中に `null` 値を入れてはいけません(SHOULD NOT)。しかし、`null` 値が受け入れられないことを確認するのが不可能な場合 (たとえば、コンパイル時に適切な型チェックを行わない言語の場合)、配列内の `null` 値はそのまま保存されなければなりません (つまり、`null` としてSpanProcessorやExporterに渡されます)。Exporterが `null` 値のエクスポートをサポートしていない場合には、それらの値を 0 や `false` 、あるいは空の文字列で置き換えてもかまいません(MAY)。これは、インデックスが同期している2つの配列として表現されるマップ/辞書構造に必要です(例えば、2つの属性 `header_keys` と `header_values` があり、どちらもマッピング `header_keys[i] -> header_values[i]` を表す文字列の配列を含んでいる場合)。

<!--
See [Attribute and Label Naming](attribute-and-label-naming.md) for naming guidelines.
-->

命名のガイドラインは、[属性とラベルの命名](attribute-and-label-naming.md)を参照してください。

