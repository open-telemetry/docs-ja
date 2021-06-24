# Kubernetes

**Status**: [Experimental](../../document-status.md)

<!--
Useful resources to understand Kubernetes objects and metadata:
-->

Kubernetesのオブジェクトやメタデータを理解するのに役立つリソースです。

<!--
* [Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
* [Names and UIDs](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/).
* [Pods](https://kubernetes.io/docs/concepts/workloads/pods/)
* [Controllers](https://kubernetes.io/docs/concepts/workloads/controllers/)
-->

* [Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
* [Names and UIDs](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/).
* [Pods](https://kubernetes.io/docs/concepts/workloads/pods/)
* [Controllers](https://kubernetes.io/docs/concepts/workloads/controllers/)

<!-- The "name" of a Kubernetes object is unique for that type of object within a
"namespace" and only at a specific moment of time (names can be reused over
time). The "uid" is unique across your whole cluster, and very likely across
time. Because of this it is recommended to always set the UID for every
Kubernetes object, but "name" is usually more user friendly so can be also set.
-->

Kubernetesオブジェクトの "name" は、 "namespace" 内のその種類のオブジェクトに対して一意であり、特定の瞬間にのみ有効です(名前は時間の経過とともに再利用できます)。"uid" は、クラスタ全体で一意であり、時間を超えても一意である可能性が高いです。このため、すべてのKubernetesオブジェクトにUIDを設定することが推奨されていますが、通常は "name" の方がユーザーフレンドリーなので、こちらも設定可能です。

<!--
## Cluster
-->

## クラスター

**type:** `k8s.cluster`

<!-- **Description:** A Kubernetes Cluster.
-->

**説明:** kubernetes Cluster

<!-- semconv k8s.cluster -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `k8s.cluster.name` | string | クラスターの名前。 | `opentelemetry-cluster` | No |
<!-- endsemconv -->

<!--
## Node
-->

## Node

**type:** `k8s.node`

<!--
**Description:** A Kubernetes Node.
-->

**説明:** Kubernetes Node

<!-- semconv k8s.node -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `k8s.node.name` | string | Nodeの名前 | `node-1` | No |
| `k8s.node.uid` | string | NodeのUID | `1eb3a0c6-0477-4080-a9cb-0cb7db65c6a2` | No |
<!-- endsemconv -->

<!--
## Namespace
-->

## Namespace

<!--
Namespaces provide a scope for names. Names of objects need to be unique within
a namespace, but not across namespaces.
-->

Namespaceは、nameの範囲を提供します。オブジェクトの名前は、namespace内では一意である必要がありますが、namespace間では一意ではありません。

**type:** `k8s.namespace`

<!--
**Description:** A Kubernetes Namespace.
-->

**説明:** Kubernetes Namespace

<!-- semconv k8s.namespace -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `k8s.namespace.name` | string | Podが動作しているnamespaceの名前 | `default` | No |
<!-- endsemconv -->

<!--
## Pod
-->

## Pod

<!-- The smallest and simplest Kubernetes object. A Pod represents a set of running
containers on your cluster.
-->

最小かつ最もシンプルなKubernetesのオブジェクト。Podは、クラスタ上で稼働するコンテナのセットを表します。

**type:** `k8s.pod`

<!--
**Description:** A Kubernetes Pod object.
-->

**説明:** Kubernetes Pod オブジェクト

<!-- semconv k8s.pod -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `k8s.pod.uid` | string | PodのUID | `275ecb36-5aa8-4c2a-9c47-d8bb681b9aff` | No |
| `k8s.pod.name` | string | Podの名前 | `opentelemetry-pod-autoconf` | No |
<!-- endsemconv -->

<!--
## Container
-->

## Container

<!--
A container specification in a Pod template. This type is intended to be used to
capture information such as name of a container in a Pod template which is different
from the name of the running container.
-->

Podテンプレート内のコンテナの仕様です。このタイプは、実行中のコンテナの名前とは異なるPodテンプレート内のコンテナの名前などの情報を取得するために使用することを目的としています。

<!--
Note: This type is different from [container](./container.md), which corresponds
to a running container.
-->

注:このタイプは、実行中のコンテナに対応する[container](./container.md)とは異なります。

**type:** `k8s.container`

<!--
**Description:** A container in a [PodTemplate](https://kubernetes.io/docs/concepts/workloads/pods/#pod-templates).
-->

**説明:** [PodTemplate](https://kubernetes.io/docs/concepts/workloads/pods/#pod-templates)のコンテナです。

<!-- semconv k8s.container -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `k8s.container.name` | string | PodTemplate の中のコンテナの名前 | `redis` | No |
<!-- endsemconv -->

<!--
## ReplicaSet
-->

## ReplicaSet

<!--
A ReplicaSet’s purpose is to maintain a stable set of replica Pods running at
any given time.
-->

ReplicaSetの目的は、常に安定したレプリカPodのセットを実行することです。

**type:** `k8s.replicaset`

<!--
**Description:** A Kubernetes ReplicaSet object.
-->

**説明:** Kubernetes ReplicaSet オブジェクト

<!-- semconv k8s.replicaset -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `k8s.replicaset.uid` | string | ReplicaSetのUID | `275ecb36-5aa8-4c2a-9c47-d8bb681b9aff` | No |
| `k8s.replicaset.name` | string | ReplicaSetの名前 | `opentelemetry` | No |
<!-- endsemconv -->

<!--
## Deployment
-->

## Deployment

<!--
An API object that manages a replicated application, typically by running Pods
with no local state. Each replica is represented by a Pod, and the Pods are
distributed among the nodes of a cluster.
-->

複製されたアプリケーションを管理するAPIオブジェクトで、通常、ローカルステートを持たないPodを実行します。各レプリカはPodで表現され、Podはクラスタのノードに分散されます。

**type:** `k8s.deployment`

<!--
**Description:** A Kubernetes Deployment object.
-->

**説明:** Kubernetes Deployment オブジェクト

<!-- semconv k8s.deployment -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `k8s.deployment.uid` | string | DeploymentのUID | `275ecb36-5aa8-4c2a-9c47-d8bb681b9aff` | No |
| `k8s.deployment.name` | string | Deploymentの名前 | `opentelemetry` | No |
<!-- endsemconv -->

<!--
## StatefulSet
-->

## StatefulSet

<!--
Manages the deployment and scaling of a set of Pods, and provides guarantees
about the ordering and uniqueness of these Pods.
-->

一連のPodのデプロイとスケーリングを管理し、これらのPodの順序と一意性に関する保証を提供します。

**type:** `k8s.statefulset`

<!--
**Description:** A Kubernetes StatefulSet object.
-->

**説明:** Kubernetes StatefulSet オブジェクト


<!-- semconv k8s.statefulset -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `k8s.statefulset.uid` | string | StatefulSetのUID | `275ecb36-5aa8-4c2a-9c47-d8bb681b9aff` | No |
| `k8s.statefulset.name` | string | StatefulSetの名前 | `opentelemetry` | No |
<!-- endsemconv -->

<!--
## DaemonSet
-->

## DaemonSet

<!--
A DaemonSet ensures that all (or some) Nodes run a copy of a Pod.
-->

DaemonSetは、すべての(または一部の)NodeがPodのコピーを実行することを保証します。

<!--
**type:** `k8s.daemonset`
-->

**type:** `k8s.daemonset`

<!--
**Description:** A Kubernetes DaemonSet object.
-->

**説明:** Kubernetes DaemonSet オブジェクト


<!-- semconv k8s.daemonset -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `k8s.daemonset.uid` | string | DaemonSetのUID | `275ecb36-5aa8-4c2a-9c47-d8bb681b9aff` | No |
| `k8s.daemonset.name` | string | DaemonSetの名前 | `opentelemetry` | No |
<!-- endsemconv -->

<!--
## Job
-->

## Job

<!--
A Job creates one or more Pods and ensures that a specified number of them
successfully terminate.
-->

ジョブは、1つまたは複数のPodを作成し、そのうちの指定された数のPodが正常に終了することを保証します。

<!--
**type:** `k8s.job`
-->

**type:** `k8s.job`

<!--
**Description:** A Kubernetes Job object.
-->

**説明:** Kubernetes Job オブジェクト

<!-- semconv k8s.job -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `k8s.job.uid` | string | JobのUID | `275ecb36-5aa8-4c2a-9c47-d8bb681b9aff` | No |
| `k8s.job.name` | string | Jobの名前 | `opentelemetry` | No |
<!-- endsemconv -->

<!--
## CronJob
-->

## CronJob

<!--
A CronJob creates Jobs on a repeating schedule.
-->

CronJobは、繰り返しのスケジュールでジョブを作成します。

<!--
**type:** `k8s.cronjob`
-->

**type:** `k8s.cronjob`

<!--
**Description:** A Kubernetes CronJob object.
-->

**例:** Kubernetes CronJobオブジェクト

<!-- semconv k8s.cronjob -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `k8s.cronjob.uid` | string | CronJobのUID | `275ecb36-5aa8-4c2a-9c47-d8bb681b9aff` | No |
| `k8s.cronjob.name` | string | CronJobの名前 | `opentelemetry` | No |
<!-- endsemconv -->
