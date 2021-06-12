# Operating System

**Status**: [Experimental](../../document-status.md)

**type:** `os`

<!--
**Description**: The operating system (OS) on which the process represented by this resource is running.
-->

**Description**: このリソースで表されるプロセスが実行されているオペレーティング・システム(OS)。

<!--
In case of virtualized environments, this is the operating system as it is observed by the process, i.e., the virtualized guest rather than the underlying host.
-->

仮想化環境の場合、これはプロセスによって観察されるオペレーティングシステムであり、基礎となるホストではなく仮想化されたゲストです。

<!-- semconv os -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `os.type` | string | The operating system type. | `windows` | Yes |
| `os.description` | string | 人間が読める (解析を意図していない) OS のバージョン情報で、例えば `ver` や `lsb_release -a` コマンドで得られるような文字列です。 | `Microsoft Windows [Version 10.0.18363.778]`; `Ubuntu 18.04.1 LTS` | No |

`os.type` MUST be one of the following or, if none of the listed values apply, a custom value:

| Value  | Description |
|---|---|
| `windows` | Microsoft Windows |
| `linux` | Linux |
| `darwin` | Apple Darwin |
| `freebsd` | FreeBSD |
| `netbsd` | NetBSD |
| `openbsd` | OpenBSD |
| `dragonflybsd` | DragonFly BSD |
| `hpux` | HP-UX (Hewlett Packard Unix) |
| `aix` | AIX (Advanced Interactive eXecutive) |
| `solaris` | Oracle Solaris |
| `z_os` | IBM z/OS |
<!-- endsemconv -->