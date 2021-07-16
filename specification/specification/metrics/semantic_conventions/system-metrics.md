<!--
# Semantic Conventions for System Metrics
-->

# システムメトリックのためのセマンティック規約

<!--
**Status**: [Experimental](../../document-status.md)
-->

**Status**: [Experimental](../../document-status.md)

<!--
This document describes instruments and labels for common system level
metrics in OpenTelemetry. Consider the [general metric semantic
conventions](README.md#general-metric-semantic-conventions) when creating
instruments not explicitly defined in the specification.
-->

本ドキュメントでは、OpenTelemetryの一般的なシステム・レベル・メトリックのInstrumentsとラベルについて説明しています。この仕様書で明示的に定義されていないInstrumentsを作成する場合は、[general metric semantic conventions](README.md#general-metric-semantic-conventions)を考慮してください。

<!-- Re-generate TOC with `markdown-toc --no-first-h1 -i` -->

<!-- toc -->

<!--
- [Metric Instruments](#metric-instruments)
  * [`system.cpu.` - Processor metrics](#systemcpu---processor-metrics)
  * [`system.memory.` - Memory metrics](#systemmemory---memory-metrics)
  * [`system.paging.` - Paging/swap metrics](#systempaging---pagingswap-metrics)
  * [`system.disk.` - Disk controller metrics](#systemdisk---disk-controller-metrics)
  * [`system.filesystem.` - Filesystem metrics](#systemfilesystem---filesystem-metrics)
  * [`system.network.` - Network metrics](#systemnetwork---network-metrics)
  * [`system.processes.` - Aggregate system process metrics](#systemprocesses---aggregate-system-process-metrics)
  * [`system.{os}.` - OS Specific System Metrics](#systemos---os-specific-system-metrics)
-->

- [メトリックInstruments](#メトリックInstruments)
  * [`system.cpu.` - Processor メトリック](#systemcpu---processor-メトリック)
  * [`system.memory.` - Memory メトリック](#systemmemory---memory-メトリック)
  * [`system.paging.` - Paging/swap メトリック](#systempaging---pagingswap-メトリック)
  * [`system.disk.` - Disk controller メトリック](#systemdisk---disk-controller-メトリック)
  * [`system.filesystem.` - Filesystem メトリック](#systemfilesystem---filesystem-メトリック)
  * [`system.network.` - Network メトリック](#systemnetwork---network-メトリック)
  * [`system.processes.` - Aggregate system process メトリック](#systemprocesses---aggregate-system-process-メトリック)
  * [`system.{os}.` - OS Specific System メトリック](#systemos---os-specific-system-メトリック)

<!-- tocstop -->

<!--
## Metric Instruments
-->

## メトリックInstruments

<!--
### `system.cpu.` - Processor metrics
-->

### `system.cpu.` - プロセッサのメトリック

<!--
**Description:** System level processor metrics.
-->

**説明:** システムレベルのプロセッサのメトリック

<!--
| Name                   | Description | Units | Instrument Type | Value Type | Label Key(s) | Label Values                        |
| ---------------------- | ----------- | ----- | --------------- | ---------- | ------------ | ----------------------------------- |
| system.cpu.time        |             | s     | SumObserver     | Double     | state        | idle, user, system, interrupt, etc. |
|                        |             |       |                 |            | cpu          | CPU number [0..n-1]                 |
| system.cpu.utilization |             | 1     | ValueObserver   | Double     | state        | idle, user, system, interrupt, etc. |
|                        |             |       |                 |            | cpu          | CPU number (0..n)                   |
-->

| Name                   | Description | Units | Instrument Type | Value Type | Label Key(s) | Label Values                        |
| ---------------------- | ----------- | ----- | --------------- | ---------- | ------------ | ----------------------------------- |
| system.cpu.time        |             | s     | SumObserver     | Double     | state        | idle, user, system, interrupt, etc. |
|                        |             |       |                 |            | cpu          | CPU number [0..n-1]                 |
| system.cpu.utilization |             | 1     | ValueObserver   | Double     | state        | idle, user, system, interrupt, etc. |
|                        |             |       |                 |            | cpu          | CPU number (0..n)                   |

<!--
### `system.memory.` - Memory metrics
-->

### `system.memory.` - メモリのメトリック


<!--
**Description:** System level memory metrics. This does not include [paging/swap
memory](#systempaging---pagingswap-metrics).
-->

**説明:** システムレベルのメモリ・メトリックです。[paging/swap memory](#systempaging--pagingswap-metrics)は含まれません。

<!--
| Name                      | Description | Units | Instrument Type   | Value Type | Label Key | Label Values             |
| ------------------------- | ----------- | ----- | ----------------- | ---------- | --------- | ------------------------ |
| system.memory.usage       |             | By    | UpDownSumObserver | Int64      | state     | used, free, cached, etc. |
| system.memory.utilization |             | 1     | ValueObserver     | Double     | state     | used, free, cached, etc. |
-->

| Name                      | Description | Units | Instrument Type   | Value Type | Label Key | Label Values             |
| ------------------------- | ----------- | ----- | ----------------- | ---------- | --------- | ------------------------ |
| system.memory.usage       |             | By    | UpDownSumObserver | Int64      | state     | used, free, cached, etc. |
| system.memory.utilization |             | 1     | ValueObserver     | Double     | state     | used, free, cached, etc. |


<!--
### `system.paging.` - Paging/swap metrics
-->

### `system.paging.` - Paging/swap メトリック


<!--
**Description:** System level paging/swap memory metrics.
-->

**説明:** システムレベルの paging/swap メモリのメトリックです。

<!--
| Name                      | Description                         | Units        | Instrument Type   | Value Type | Label Key | Label Values |
| ------------------------- | ----------------------------------- | ------------ | ----------------- | ---------- | --------- | ------------ |
| system.paging.usage       | Unix swap or windows pagefile usage | By           | UpDownSumObserver | Int64      | state     | used, free   |
| system.paging.utilization |                                     | 1            | ValueObserver     | Double     | state     | used, free   |
| system.paging.faults      |                                     | {faults}     | SumObserver       | Int64      | type      | major, minor |
| system.paging.operations  |                                     | {operations} | SumObserver       | Int64      | type      | major, minor |
|                           |                                     |              |                   |            | direction | in, out      |
-->

| Name                      | Description                         | Units        | Instrument Type   | Value Type | Label Key | Label Values |
| ------------------------- | ----------------------------------- | ------------ | ----------------- | ---------- | --------- | ------------ |
| system.paging.usage       | Unix swap or windows pagefile usage | By           | UpDownSumObserver | Int64      | state     | used, free   |
| system.paging.utilization |                                     | 1            | ValueObserver     | Double     | state     | used, free   |
| system.paging.faults      |                                     | {faults}     | SumObserver       | Int64      | type      | major, minor |
| system.paging.operations  |                                     | {operations} | SumObserver       | Int64      | type      | major, minor |
|                           |                                     |              |                   |            | direction | in, out      |


<!--
### `system.disk.` - Disk controller metrics
-->

### `system.disk.` - ディスクコントローラーのメトリック


<!--
**Description:** System level disk performance metrics.
-->

**説明:** システムレベルのディスク性能のメトリック


| Name                                                      | Description                                     | Units        | Instrument Type | Value Type | Label Key | Label Values |
| --------------------------------------------------------- | ----------------------------------------------- | ------------ | --------------- | ---------- | --------- | ------------ |
| system.disk.io<!--notlink-->                              |                                                 | By           | SumObserver     | Int64      | device    | (identifier) |
|                                                           |                                                 |              |                 |            | direction | read, write  |
| system.disk.operations                                    |                                                 | {operations} | SumObserver     | Int64      | device    | (identifier) |
|                                                           |                                                 |              |                 |            | direction | read, write  |
| system.disk.io_time<sup>[1](#io_time)</sup>               | Time disk spent activated                       | s            | SumObserver     | Double     | device    | (identifier) |
| system.disk.operation_time<sup>[2](#operation_time)</sup> | Sum of the time each operation took to complete | s            | SumObserver     | Double     | device    | (identifier) |
|                                                           |                                                 |              |                 |            | direction | read, write  |
| system.disk.merged                                        |                                                 | {operations} | SumObserver     | Int64      | device    | (identifier) |
|                                                           |                                                 |              |                 |            | direction | read, write  |

<!--
<sup><a name="io_time">1</a></sup> The real elapsed time ("wall clock")
used in the I/O path (time from operations running in parallel are not
counted). Measured as:
-->

<sup><a name="io_time">1</a></sup> I/Oパスで使用される実際の経過時間(「ウォールクロック」)です(並行して実行される操作の時間はカウントされません)。以下の項目で測定されます:


<!--
- Linux: Field 13 from
[procfs-diskstats](https://www.kernel.org/doc/Documentation/ABI/testing/procfs-diskstats)
- Windows: The complement of ["Disk\% Idle
Time"](https://docs.microsoft.com/en-us/archive/blogs/askcore/windows-performance-monitor-disk-counters-explained#windows-performance-monitor-disk-counters-explained:~:text=%25%20Idle%20Time,Idle\)%20to%200%20(meaning%20always%20busy).)
performance counter: `uptime * (100 - "Disk\% Idle Time") / 100`
-->

- Linux: [procfs-diskstats](https://www.kernel.org/doc/Documentation/ABI/testing/procfs-diskstats)の13番目のフィールド
- Windows: ["Disk\% Idle Time"](https://docs.microsoft.com/en-us/archive/blogs/askcore/windows-performance-monitor-disk-counters-explained#windows-performance-monitor-disk-counters-explained:~:text=%25%20Idle%20Time,Idle)%20to%200%20(mean%20always%20busy))のパフォーマンスカウンタを補完します。: `uptime * (100 - "Disk\% Idle Time") / 100`.

<!--
<sup><a name="operation_time">2</a></sup> Because it is the sum of time each
request took, parallel-issued requests each contribute to make the count
grow. Measured as:
-->

<sup><a name="operation_time">2</a></sup> 各リクエストにかかった時間の合計なので、並行して発行されたリクエストは、それぞれが貢献してカウントが大きくなります。以下の項目で測定されます:

<!--
- Linux: Fields 7 & 11 from
[procfs-diskstats](https://www.kernel.org/doc/Documentation/ABI/testing/procfs-diskstats)
- Windows: "Avg. Disk sec/Read" perf counter multiplied by "Disk Reads/sec"
perf counter (similar for Writes)
-->

- Linux: [procfs-diskstats](https://www.kernel.org/doc/Documentation/ABI/testing/procfs-diskstats)の7と11番目のフィールド
- Windows: "Avg. Disk sec/Read" パフォーマンスカウンタに "Disk Reads/sec" パフォーマンスカウンタを乗じたもの。ディスク秒/読込」パフォーマンスカウンタに「ディスク読込/秒」パフォーマンスカウンタを乗じたもの(書込についても同様)。


<!--
### `system.filesystem.` - Filesystem metrics
-->

### `system.filesystem.` - ファイルシステムのメトリック


<!--
**Description:** System level filesystem metrics.
-->

**説明:** システムレベルのファイルシステムのメトリック

| Name                          | Description | Units | Instrument Type   | Value Type | Label Key  | Label Values         |
| ----------------------------- | ----------- | ----- | ----------------- | ---------- | ---------- | -------------------- |
| system.filesystem.usage       |             | By    | UpDownSumObserver | Int64      | device     | (identifier)         |
|                               |             |       |                   |            | state      | used, free, reserved |
|                               |             |       |                   |            | type       | ext4, tmpfs, etc.    |
|                               |             |       |                   |            | mode       | rw, ro, etc.         |
|                               |             |       |                   |            | mountpoint | (path)               |
| system.filesystem.utilization |             | 1     | ValueObserver     | Double     | device     | (identifier)         |
|                               |             |       |                   |            | state      | used, free, reserved |
|                               |             |       |                   |            | type       | ext4, tmpfs, etc.    |
|                               |             |       |                   |            | mode       | rw, ro, etc.         |
|                               |             |       |                   |            | mountpoint | (path)               |

<!--
### `system.network.` - Network metrics
-->

### `system.network.` - ネットワークのメトリック

<!--
**Description:** System level network metrics.
-->

**説明:** システムレベルのネットワークのメトリック

| Name                                           | Description                                                                   | Units         | Instrument Type   | Value Type | Label Key | Label Values                                                                                   |
| ---------------------------------------------- | ----------------------------------------------------------------------------- | ------------- | ----------------- | ---------- | --------- | ---------------------------------------------------------------------------------------------- |
| system.network.dropped<sup>[1](#dropped)</sup> | Count of packets that are dropped or discarded even though there was no error | {packets}     | SumObserver       | Int64      | device    | (identifier)                                                                                   |
|                                                |                                                                               |               |                   |            | direction | transmit, receive                                                                              |
| system.network.packets                         |                                                                               | {packets}     | SumObserver       | Int64      | device    | (identifier)                                                                                   |
|                                                |                                                                               |               |                   |            | direction | transmit, receive                                                                              |
| system.network.errors<sup>[2](#errors)</sup>   | Count of network errors detected                                              | {errors}      | SumObserver       | Int64      | device    | (identifier)                                                                                   |
|                                                |                                                                               |               |                   |            | direction | transmit, receive                                                                              |
| system<!--notlink-->.network.io                |                                                                               | By            | SumObserver       | Int64      | device    | (identifier)                                                                                   |
|                                                |                                                                               |               |                   |            | direction | transmit, receive                                                                              |
| system.network.connections                     |                                                                               | {connections} | UpDownSumObserver | Int64      | device    | (identifier)                                                                                   |
|                                                |                                                                               |               |                   |            | protocol  | tcp, udp, [etc.](https://en.wikipedia.org/wiki/Transport_layer#Protocols)                      |
|                                                |                                                                               |               |                   |            | state     | [e.g. for tcp](https://en.wikipedia.org/wiki/Transmission_Control_Protocol#Protocol_operation) |

<!--
<sup><a name="dropped">1</a></sup> Measured as:
-->

<sup><a name="dropped">1</a></sup> 以下の項目で測定されます:

<!--
- Linux: the `drop` column in `/proc/dev/net`
([source](https://web.archive.org/web/20180321091318/http://www.onlamp.com/pub/a/linux/2000/11/16/LinuxAdmin.html)).
- Windows:
[`InDiscards`/`OutDiscards`](https://docs.microsoft.com/en-us/windows/win32/api/netioapi/ns-netioapi-mib_if_row2)
from
[`GetIfEntry2`](https://docs.microsoft.com/en-us/windows/win32/api/netioapi/nf-netioapi-getifentry2).
-->

- Linux: `/proc/dev/net` の `drop` カラム ([source](https://web.archive.org/web/20180321091318/http://www.onlamp.com/pub/a/linux/2000/11/16/LinuxAdmin.html)).
- Windows: [`GetIfEntry2`](https://docs.microsoft.com/en-us/windows/win32/api/netioapi/nf-netioapi-getifentry2)の[InDiscards`/`OutDiscards`](https://docs.microsoft.com/en-us/windows/win32/api/netioapi/ns-netioapi-mib_if_row2)


<!--
<sup><a name="errors">2</a></sup> Measured as:
-->

<sup><a name="errors">2</a></sup> 以下で測定されます:

<!--
- Linux: the `errs` column in `/proc/dev/net`
([source](https://web.archive.org/web/20180321091318/http://www.onlamp.com/pub/a/linux/2000/11/16/LinuxAdmin.html)).
- Windows:
[`InErrors`/`OutErrors`](https://docs.microsoft.com/en-us/windows/win32/api/netioapi/ns-netioapi-mib_if_row2)
from
[`GetIfEntry2`](https://docs.microsoft.com/en-us/windows/win32/api/netioapi/nf-netioapi-getifentry2).
-->

- Linux: `/proc/dev/net` の `errs` カラム ([source](https://web.archive.org/web/20180321091318/http://www.onlamp.com/pub/a/linux/2000/11/16/LinuxAdmin.html)).
- Windows: [`GetIfEntry2`](https://docs.microsoft.com/en-us/windows/win32/api/netioapi/nf-netioapi-getifentry2)の[InErrors`/`OutErrors`](https://docs.microsoft.com/en-us/windows/win32/api/netioapi/ns-netioapi-mib_if_row2)


<!--
### `system.processes.` - Aggregate system process metrics
-->

### `system.processes.` - システム・プロセス・メトリックの集計

<!--
**Description:** System level aggregate process metrics. For metrics at the
individual process level, see [process metrics](process-metrics.md).
-->

**説明:** システムレベルの集約されたプロセスメトリックです。個々のプロセスレベルのメトリックについては、[process metrics](process-metrics.md)を参照してください。

| Name                     | Description                                               | Units       | Instrument Type   | Value Type | Label Key | Label Values                                                                                   |
| ------------------------ | --------------------------------------------------------- | ----------- | ----------------- | ---------- | --------- | ---------------------------------------------------------------------------------------------- |
| system.processes.count   | Total number of processes in each state                   | {processes} | UpDownSumObserver | Int64      | status    | running, sleeping, [etc.](https://man7.org/linux/man-pages/man1/ps.1.html#PROCESS_STATE_CODES) |
| system.processes.created | Total number of processes created over uptime of the host | {processes} | SumObserver       | Int64      | -         | -                                                                                              |

<!--
### `system.{os}.` - OS Specific System Metrics
-->

### `system.{os}.` - OS固有のシステムメトリック

<!--
Instrument names for system level metrics that have different and conflicting
meaning across multiple OSes should be prefixed with `system.{os}.` and
follow the hierarchies listed above for different entities like CPU, memory,
and network.
-->

複数のOS間で意味が異なり、矛盾するシステムレベルメトリックのInstrumentsは、先頭に `system.{os}.` を付け、CPU、メモリ、ネットワークなどの異なるエンティティに対する上述した階層に従うものとします。

<!--
For example, [UNIX load
average](https://en.wikipedia.org/wiki/Load_(computing)) over a given
interval is not well standardized and its value across different UNIX like
OSes may vary despite being under similar load:
-->

例えば、ある区間の[UNIX Load Average](https://en.wikipedia.org/wiki/Load_(computing))は、標準化されておらず、UNIX系OSでも異なるOSの場合、同じような負荷がかかっているにもかかわらず、その値が異なる場合があります。

<!--
> Without getting into the vagaries of every Unix-like operating system in
existence, the load average more or less represents the average number of
processes that are in the running (using the CPU) or runnable (waiting for
the CPU) states. One notable exception exists: Linux includes processes in
uninterruptible sleep states, typically waiting for some I/O activity to
complete. This can markedly increase the load average on Linux systems.
-->

> 現存するすべてのUnix系OSの違いを考慮する必要はありませんが、ロードアベレージは、実行中(CPUを使用している状態)または実行可能(CPUを待っている状態)のプロセスの平均数を多かれ少なかれ表しています。ただし、例外があります。Linuxには、I/Oアクティビティの完了を待つなど、中断できないスリープ状態のプロセスがあります。これにより、Linuxシステムのロードアベレージが大幅に上昇します。

<!--
([source of
quote](https://github.com/torvalds/linux/blob/e4cbce4d131753eca271d9d67f58c6377f27ad21/kernel/sched/loadavg.c#L11-L18),
[linux source
code](https://github.com/torvalds/linux/blob/e4cbce4d131753eca271d9d67f58c6377f27ad21/kernel/sched/loadavg.c#L11-L18))
-->

([引用元](https://github.com/torvalds/linux/blob/e4cbce4d131753eca271d9d67f58c6377f27ad21/kernel/sched/loadavg.c#L11-L18), [linux source code](https://github.com/torvalds/linux/blob/e4cbce4d131753eca271d9d67f58c6377f27ad21/kernel/sched/loadavg.c#L11-L18))

<!--
An instrument for load average over 1 minute on Linux could be named
`system.linux.cpu.load_1m`, reusing the `cpu` name proposed above and having
an `{os}` prefix to split this metric across OSes.
-->

Linuxにおける1分間のロードアベレージのInstrumentsは、名前を `system.linux.cpu.load_1m` とし、上で提案した `cpu` という名前を再利用し、このメトリックをOS間で分割するために `{os}` というプレフィックスを付けることができます。

