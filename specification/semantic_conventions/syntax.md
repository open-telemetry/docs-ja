<!--
# Semantic Convention YAML Language
-->

# セマンティック規約YAML言語

<!--
First, the syntax with a pseudo [EBNF](https://en.wikipedia.org/wiki/Extended_Backus-Naur_form) grammar is presented.
Then, the semantic of each field is described.
-->

まず、擬似的な[EBNF](https://en.wikipedia.org/wiki/Extended_Backus-Naur_form)文法を用いた構文を示します。次に、各フィールドの意味を記述します。

<!--
## Syntax
-->

## 文法

<!--
All attributes are lower case.
-->

すべての属性は小文字です。
```bnf
groups ::= semconv
       | semconv groups

semconv ::= id brief [note] [prefix] [extends] [span_kind] attributes [constraints]

id    ::= string
brief ::= string
note  ::= string

prefix ::= string

# extends MUST point to an existing semconv id
extends ::= string

span_kind ::= "client"
          |   "server"
          |   "producer"
          |   "consumer"
          |   "internal"

attributes ::= (id type brief examples | ref [brief] [examples]) [required] [note]

# ref MUST point to an existing attribute id
ref ::= id

type ::= "string"
     |   "number"
     |   "boolean"
     |   "string[]"
     |   "number[]"
     |   "boolean[]"
     |   enum

enum ::= [allow_custom_values] members

allow_custom_values := boolean

members ::= member {member}

member ::= id value [brief] [note]

required ::= "always"
         |   "conditional" <condition>

examples ::= <example_value> {<example_value>}

constraints ::= constraint {constraint}

constraint ::= any_of
           |   include

any_of ::= id {id}

include ::= id

```

<!--
## セマンティック
-->

## Semantics

<!--
### Groups
-->

### グループ

<!--
Groups contain the list of semantic conventions and it is the root node of each yaml file.
-->

グループにはセマンティック規約のリストが含まれており、各YAMLファイルのルートノードとなります。

<!--
### Semantic Convention
-->

### セマンティック規約

<!--
The field `semconv` represents a semantic convention and it is made by:
-->

フィールド `semconv` はセマンティック規約を表し、下記を用いて作成されます:

<!--
- `id`, string that uniquely identifies the semantic convention.
- `brief`, string, a brief description of the semantic convention.
- `note`, optional string, a more elaborate description of the semantic convention.
    It defaults to an empty string.
- `prefix`, optional string, prefix for the attributes for this semantic convention.
    It defaults to an empty string.
- `extends`, optional string, reference another semantic convention `id`.
    It inherits the prefix, constraints, and all attributes defined in the specified semantic convention.
- `span_kind`, optional enum, specifies the kind of the span.
- `attributes`, list of attributes that belong to the semantic convention.
- `constraints`, optional list, additional constraints (See later). It defaults to an empty list.
-->

- `id`, セマンティック規約を一意に識別する文字列
- `brief`, 文字列、セマンティック規約の簡単な説明
- `note`, オプションの文字列で、セマンティック規約のより詳細な説明
    デフォルトは空の文字列です
- `prefix`, オプションの文字列で、このセマンティック規約の属性の接頭辞
    デフォルトは空文字列です
- `extends`, オプションの文字列, 別のセマンティック規約 `id` を参照します
    指定されたセマンティック規約で定義された接頭辞、制約、すべての属性を継承します
- `span_kind`, オプションの enum, Spanの種類を指定します
- `attributes`, セマンティック規約に属する属性のリスト
- `constraints`, オプションのリストで、追加の制約を指定します (後述)。デフォルトは空のリストです

<!--
### Attributes
-->

### 属性

<!--
An attribute is defined by:
-->

属性は次のように定義されています:

<!--
- `id`, string that uniquely identifies the attribute.
- `type`, either a string literal denoting the type or an enum definition (See later).
   The accepted strings literals are:
  * "string": String attributes.
  * "number": Numeric attributes.
  * "boolean": Boolean attributes.
  * "string[]": Array of strings attributes.
  * "number[]": Array of numbers attributes.
  * "boolean[]": Array of booleans attributes.
- `ref`, optional string, reference an existing attribute, see later.
- `required`, optional, specifies if the attribute is mandatory.
    Can be "always", or "conditional". When omitted, the attribute is not required.
    When set to "conditional",the string provided as `<condition>` MUST specify
    the conditions under which the attribute is required.
- `brief`, string, brief description of the attribute.
- `note`, optional string, additional notes to the attribute. It defaults to an empty string.
- `examples`, sequence/dictionary of example values for the attribute.
   They are optional for boolean and enum attributes.
   Example values must be of the same type of the attribute.
   If only a single example is provided, it can directly be reported without encapsulating it into a sequence/dictionary.
-->

- `id`, 属性を一意に識別する文字列。
- `type`, 型を表す文字列リテラルか、列挙型の定義(後述)。受け入れられる文字列リテラルは以下の通りです。
  * "string": 文字列属性。
  * "number": 数値属性。
  * "boolean": 真偽値属性。
  * "string[]": 文字列属性の配列。
  * "number[]": 数値属性の配列。
  * "boolean[]": 真偽値属性の配列。
- `ref`: 任意で設定する文字列で、既存の属性を参照します(後述)。
- `required`, 任意で設定する属性, この属性が必須かどうかを指定します。"alwaysまたは"conditional"と指定することができます。省略した場合、その属性は必須ではないという意味です。
    "conditional "に設定されている場合、`<condition>`として与えられる文字列は、属性が要求される条件を指定しなければなりません(MUST)。
- `brief`, 文字列, 属性の簡単な説明。
- `note`, 任意で設定する文字列。属性への追加の注意事項を指定します。デフォルトは空の文字列です。
- `examples`, 属性の値の例のシーケンス/辞書。これらはboolean属性やenum属性ではオプションです。 exampleの値は、属性と同じ型でなければなりません。単一の例のみが提供された場合、それをシーケンス/辞書にカプセル化せずに直接報告することができます。


<!--
Examples for setting the `examples` field:
-->

`examples` フィールドを設定するための例を示す。

<!--
A single example value for a string attribute. All the following three representations are equivalent:
-->

文字列属性のための単一の例示的な値。以下の3つの表現はすべて同等です。

<!--
```yaml
examples: 'this is a single string'
```
-->

```yaml
examples: 'これは単一の文字列です'
```

あるいは

<!--
```yaml
examples: ['this is a single string']
```
-->

```yaml
examples: ['これは単一の文字列です']
```

あるいは

<!--
```yaml
examples:
   - 'this is a single string'
```
-->

```yaml
examples:
   - 'これは単一の文字列です'
```

<!--
Attention, the following will throw a type mismatch error because a string type as example value is expected and not an array of string:
-->

注意：以下は文字列の配列ではなく文字列型が想定されているため、型の不一致エラーが発生します。

```yaml
examples:
   - ['これはエラーです']

examples: [['これはエラーです']]
```

<!--
Multiple example values for a string attribute:
-->

文字列属性の複数の値の例:


```yaml
examples: ['this is a single string', 'this is another one']
```

あるいは

```yaml
examples:
   - 'this is a single string'
   - 'this is another one'
```

<!--
A single example value for an array of strings attribute:
-->

文字列属性の配列のための単一の値の例:

```yaml
examples: ['first element of first array', 'second element of first array']
```

or

```yaml
examples:
   - ['first element of first array', 'second element of first array']
```

<!--
Attention, the following will throw a type mismatch error because an array of strings as type for the example values is expected and not a string:
-->

注意：例の値の型は文字列ではなく文字列の配列を想定しているため、以下のような場合は型の不一致エラーが発生します。
```yaml
examples: 'this is an error'
```

Multiple example values for an array of string attribute:

```yaml
examples: [ ['first element of first array', 'second element of first array'], ['first element of second array', 'second element of second array'] ]
```

or

```yaml
examples:
   - ['first element of first array', 'second element of first array']
   - ['first element of second array', 'second element of second array']
```

<!--
### Ref
-->

### Ref

<!--
`ref` MUST have an id of an existing attribute. When it is set, `id` and `type` MUST NOT be present.
`ref` is useful for specifying that an existing attribute of another semantic convention is part of
the current semantic convention and inherit its `brief`, `note`, and `example` values. However, if these
fields are present in the current attribute definition, they override the inherited values.
-->

`ref` は既存の属性のidを持たなければなりません(MUST)。refが設定されている場合、`id` と `type` は存在してはなりません(MUST NOT)。`ref` は、他のセマンティック規約の既存の属性が現在のセマンティック規約の一部であり、その `brief`, `note`, `example` の値を継承していることを指定するのに便利です。しかし、これらのフィールドが現在の属性定義に存在する場合は、継承された値を上書きします。

<!--
### Type
-->

### 型

<!--
An attribute type can either be a string, number, boolean, array of strings, array of numbers,
array of booleans, or an enumeration. If it is an enumeration, additional fields are required:
-->

属性の型は、文字列、数値、真偽値、文字列の配列、数値の配列、真偽値の配列、または列挙型のいずれかです。列挙型の場合は、追加のフィールドが必要です。

<!--
- `allow_custom_values`, optional boolean, set to false to not accept values
     other than the specified members. It defaults to `true`.
- `members`, list of enum entries.
-->

- `allow_custom_values`: 任意で指定する boolean 型で、指定したメンバ以外の値を受け付けないようにする場合は false を設定します。デフォルトは `true` です。
- `members`, 列挙型のリスト

<!--
An enum entry has the following fields:
-->

列挙型には以下のフィールドがあります:

<!--
- `id`, string that uniquely identifies the enum entry.
- `value`, string, number, or boolean, value of the enum entry.
- `brief`, optional string, brief description of the enum entry value. It defaults to the value of `id`.
- `note`, optional string, longer description. It defaults to an empty string.
-->

- `id`, 列挙型を一意に識別する文字列。
- `value`, 文字列、数値、あるいは真偽値で、列挙型の値。
- `brief`, オプションの文字列で、列挙項目の値の簡単な説明を記述します。デフォルトは `id` の値です。
- `note`, オプションの文字列。デフォルトは空文字列です。

<!--
### Constraints
-->

### 制約(Constraints)

<!--
Allow to define additional requirements on the semantic convention.
Currently, it supports `any_of` and `include`.
-->

セマンティック規約に関する追加の要件を定義できるようにします。現在は `any_of` と `include` をサポートしています。

<!--
#### Any Of
-->

#### Any Of

<!--
`any_of` accepts a list of sequences. Each sequence contains a list of attribute ids that are required.
`any_of` enforces that all attributes of at least one of the sequences are set.
-->

`any_of` はシーケンスのリストを受け付けます。各シーケンスには必要な属性IDのリストが含まれます。`any_of` は、少なくとも1つのシーケンスのすべての属性が設定されていることを強制します。

<!--
#### Include
-->

#### Include

<!--
`include` accepts a semantic conventions `id`. It includes as part of this semantic convention all constraints
and required attributes that are not already defined in the current semantic convention.
-->

`include`はセマンティック規約`id`を受け入れます。現在のセマンティック規約で定義されていないすべての制約と必須属性を、このセマンティック規約の一部として含みます。

