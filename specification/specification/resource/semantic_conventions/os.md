# Operating System

**Status**: [Experimental](../../document-status.md)

**type:** `os`

**Description**: The operating system (OS) on which the process represented by this resource is running.

In case of virtualized environments, this is the operating system as it is observed by the process, i.e., the virtualized guest rather than the underlying host.

<!-- semconv os -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `os.type` | string | The operating system type. | `WINDOWS` | Yes |
| `os.description` | string | 人間が読める (解析を意図していない) OS のバージョン情報で、例えば `ver` や `lsb_release -a` コマンドで得られるような文字列です。 | `Microsoft Windows [Version 10.0.18363.778]`; `Ubuntu 18.04.1 LTS` | No |

`os.type` MUST be one of the following or, if none of the listed values apply, a custom value:

| Value  | Description |
|---|---|
| `WINDOWS` | Microsoft Windows |
| `LINUX` | Linux |
| `DARWIN` | Apple Darwin |
| `FREEBSD` | FreeBSD |
| `NETBSD` | NetBSD |
| `OPENBSD` | OpenBSD |
| `DRAGONFLYBSD` | DragonFly BSD |
| `HPUX` | HP-UX (Hewlett Packard Unix) |
| `AIX` | AIX (Advanced Interactive eXecutive) |
| `SOLARIS` | Oracle Solaris |
| `ZOS` | IBM z/OS |
<!-- endsemconv -->