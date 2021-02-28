# Container

**Status**: [Experimental](../../document-status.md)

**type:** `container`

**Description:** A container instance.

<!-- semconv container -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `container.name` | string | コンテナ名 | `opentelemetry-autoconf` | No |
| `container.id` | string | コンテナID。通常はUUIDで、例えば[Dockerコンテナの識別子](https://docs.docker.com/engine/reference/run/#container-identification)]に使用されます。UUIDが略されている場合があります。 | `a3bf90e006b2` | No |
| `container.image.name` | string | コンテナイメージの名前。 | `gcr.io/opentelemetry/operator` | No |
| `container.image.tag` | string | コンテナイメージタグ。 | `0.1` | No |
<!-- endsemconv -->
