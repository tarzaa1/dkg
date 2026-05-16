# KubeGraph — Graph Schema

This document is the source of truth for the graph model. Both `kubeinsights` (what it watches and publishes) and `kubegrapher` (what it parses and merges) must implement against this schema.

---

## Conventions

- Every node carries `cluster_id` (set to the Kafka topic name) and `last_seen` (UTC ISO timestamp of the last merge).
- UIDs used for `MERGE` are denoted **[merge key]** — these must be stable and unique.
- Relationships are directed unless noted otherwise.
- All relationships are created with `MERGE` (idempotent).

---

## Node Types

### Cluster
Root anchor — one per cluster.

| Property | Type | Notes |
|---|---|---|
| `id` | string | **[merge key]** — Kafka topic name |
| `name` | string | Human-readable cluster name |
| `last_seen` | string | UTC ISO timestamp |

---

### Namespace
Organizational layer between Cluster and workloads. Previously absent — all workload nodes must carry an `IN_NAMESPACE` edge.

| Property | Type | Notes |
|---|---|---|
| `id` | string | **[merge key]** — `cluster_id/namespace_name` |
| `name` | string | |
| `cluster_id` | string | |
| `uid` | string | k8s UID |
| `status` | string | Active \| Terminating |
| `last_seen` | string | UTC ISO timestamp |

**Relationships:**
- `BELONGS_TO` → `Cluster`

---

### K8sNode
A physical or virtual machine in the cluster.

| Property | Type | Notes |
|---|---|---|
| `id` | string | **[merge key]** — k8s UID |
| `name` | string | |
| `cluster_id` | string | |
| `hostname` | string | |
| `internal_ip` | string | |
| `os` | string | |
| `kernel_version` | string | |
| `container_runtime` | string | |
| `kubelet_version` | string | |
| `allocatable_cpu` | string | |
| `allocatable_memory` | string | |
| `allocatable_storage` | string | |
| `usage_cpu` | float | Set by metrics worker |
| `usage_memory` | float | Set by metrics worker |
| `creation_timestamp` | string | |
| `last_seen` | string | UTC ISO timestamp |

**Relationships:**
- `BELONGS_TO` → `Cluster`
- `HAS_LABEL` → `Label`
- `HAS_ANNOTATION` → `Annotation`
- `HAS_TAINT` → `Taint`
- `STORES_IMAGE` → `Image`

---

### Pod
A running workload unit.

| Property | Type | Notes |
|---|---|---|
| `id` | string | **[merge key]** — k8s UID |
| `name` | string | |
| `cluster_id` | string | Previously missing — now required |
| `namespace` | string | |
| `node_name` | string | |
| `phase` | string | Pending \| Running \| Succeeded \| Failed \| Unknown |
| `host_ip` | string | |
| `pod_ip` | string | |
| `qos_class` | string | |
| `restart_policy` | string | |
| `service_account_name` | string | |
| `start_time` | string | |
| `creation_timestamp` | string | |
| `last_seen` | string | UTC ISO timestamp |

**Relationships:**
- `IN_NAMESPACE` → `Namespace`
- `SCHEDULED_ON` → `K8sNode` (matched by k8s UID, not name)
- `MANAGED_BY` → `ReplicaSet` \| `StatefulSet` \| `DaemonSet` (via ownerReferences)
- `RUNS_CONTAINER` → `Container`
- `HAS_LABEL` → `Label`
- `HAS_ANNOTATION` → `Annotation`
- `RUNS_AS` → `ServiceAccount`
- `MOUNTS` → `PersistentVolumeClaim`

---

### Container

| Property | Type | Notes |
|---|---|---|
| `id` | string | **[merge key]** — pod UID + container name |
| `name` | string | |
| `cluster_id` | string | Previously missing — now required |
| `image` | string | |
| `image_pull_policy` | string | |
| `request_cpu` | string | |
| `request_memory` | string | |
| `limit_cpu` | string | |
| `limit_memory` | string | |
| `usage_cpu` | float | Set by metrics worker |
| `usage_memory` | float | Set by metrics worker |
| `ready` | bool | |
| `restart_count` | int | |
| `last_seen` | string | UTC ISO timestamp |

**Relationships:**
- `INSTANTIATES_IMAGE` → `Image`
- `CONFIGMAP_REF` → `ConfigMap`

---

### Deployment

| Property | Type | Notes |
|---|---|---|
| `id` | string | **[merge key]** — k8s UID |
| `name` | string | |
| `cluster_id` | string | Previously missing — now required |
| `namespace` | string | |
| `replicas` | int | |
| `ready_replicas` | int | |
| `available_replicas` | int | |
| `strategy` | string | |
| `last_seen` | string | UTC ISO timestamp |

**Relationships:**
- `IN_NAMESPACE` → `Namespace`
- `HAS_LABEL` → `Label`
- `HAS_ANNOTATION` → `Annotation`

---

### ReplicaSet

| Property | Type | Notes |
|---|---|---|
| `id` | string | **[merge key]** — k8s UID |
| `name` | string | |
| `cluster_id` | string | Previously missing — now required |
| `namespace` | string | |
| `replicas` | int | |
| `ready_replicas` | int | |
| `last_seen` | string | UTC ISO timestamp |

**Relationships:**
- `IN_NAMESPACE` → `Namespace`
- `MANAGED_BY` → `Deployment`
- `HAS_LABEL` → `Label`
- `HAS_ANNOTATION` → `Annotation`

---

### StatefulSet
New — previously absent.

| Property | Type | Notes |
|---|---|---|
| `id` | string | **[merge key]** — k8s UID |
| `name` | string | |
| `cluster_id` | string | |
| `namespace` | string | |
| `replicas` | int | |
| `ready_replicas` | int | |
| `service_name` | string | Headless service name |
| `update_strategy` | string | |
| `last_seen` | string | UTC ISO timestamp |

**Relationships:**
- `IN_NAMESPACE` → `Namespace`
- `MANAGES` → `Pod`
- `HAS_LABEL` → `Label`
- `HAS_ANNOTATION` → `Annotation`

---

### DaemonSet
New — previously absent.

| Property | Type | Notes |
|---|---|---|
| `id` | string | **[merge key]** — k8s UID |
| `name` | string | |
| `cluster_id` | string | |
| `namespace` | string | |
| `desired_number_scheduled` | int | |
| `number_ready` | int | |
| `update_strategy` | string | |
| `last_seen` | string | UTC ISO timestamp |

**Relationships:**
- `IN_NAMESPACE` → `Namespace`
- `MANAGES` → `Pod`
- `HAS_LABEL` → `Label`
- `HAS_ANNOTATION` → `Annotation`

---

### Service

| Property | Type | Notes |
|---|---|---|
| `id` | string | **[merge key]** — k8s UID |
| `name` | string | |
| `cluster_id` | string | |
| `namespace` | string | |
| `type` | string | ClusterIP \| NodePort \| LoadBalancer \| ExternalName |
| `cluster_ip` | string | |
| `ports` | string | JSON |
| `selector` | string | JSON |
| `session_affinity` | string | |
| `last_seen` | string | UTC ISO timestamp |

**Relationships:**
- `IN_NAMESPACE` → `Namespace`
- `HAS_SELECTOR` → `Label`
- `HAS_LABEL` → `Label`
- `HAS_ANNOTATION` → `Annotation`
- `EXPOSES` → `Pod` (dynamically matched via selector labels)

---

### Ingress

| Property | Type | Notes |
|---|---|---|
| `id` | string | **[merge key]** — k8s UID |
| `name` | string | |
| `cluster_id` | string | |
| `namespace` | string | |
| `ingress_class` | string | |
| `service_names` | list | Extracted from rules |
| `last_seen` | string | UTC ISO timestamp |

**Relationships:**
- `IN_NAMESPACE` → `Namespace`
- `ROUTES_TO` → `Service`
- `HAS_ANNOTATION` → `Annotation`

---

### ServiceAccount
New — previously absent.

| Property | Type | Notes |
|---|---|---|
| `id` | string | **[merge key]** — k8s UID |
| `name` | string | |
| `cluster_id` | string | |
| `namespace` | string | |
| `last_seen` | string | UTC ISO timestamp |

**Relationships:**
- `IN_NAMESPACE` → `Namespace`

---

### PersistentVolumeClaim
New — previously absent.

| Property | Type | Notes |
|---|---|---|
| `id` | string | **[merge key]** — k8s UID |
| `name` | string | |
| `cluster_id` | string | |
| `namespace` | string | |
| `phase` | string | Pending \| Bound \| Lost |
| `access_modes` | string | JSON list |
| `storage_class_name` | string | |
| `request_storage` | string | |
| `volume_name` | string | Bound PV name |
| `last_seen` | string | UTC ISO timestamp |

**Relationships:**
- `IN_NAMESPACE` → `Namespace`
- `BOUND_TO` → `PersistentVolume`

---

### PersistentVolume
New — previously absent.

| Property | Type | Notes |
|---|---|---|
| `id` | string | **[merge key]** — k8s UID |
| `name` | string | |
| `cluster_id` | string | |
| `capacity_storage` | string | |
| `access_modes` | string | JSON list |
| `reclaim_policy` | string | |
| `status` | string | Available \| Bound \| Released \| Failed |
| `storage_class_name` | string | |
| `volume_mode` | string | Filesystem \| Block |
| `last_seen` | string | UTC ISO timestamp |

**Relationships:**
- `PROVISIONED_BY` → `StorageClass`

---

### StorageClass
New — previously absent.

| Property | Type | Notes |
|---|---|---|
| `id` | string | **[merge key]** — `cluster_id/name` (StorageClasses are non-namespaced) |
| `name` | string | |
| `cluster_id` | string | |
| `provisioner` | string | e.g. `kubernetes.io/aws-ebs`, `rancher.io/local-path` |
| `reclaim_policy` | string | |
| `volume_binding_mode` | string | |
| `allow_volume_expansion` | bool | |
| `last_seen` | string | UTC ISO timestamp |

**Relationships:** none outgoing (leaf node)

---

### ConfigMap

| Property | Type | Notes |
|---|---|---|
| `id` | string | **[merge key]** — k8s UID |
| `name` | string | |
| `cluster_id` | string | Previously missing — now required |
| `namespace` | string | |
| `data` | string | JSON |
| `last_seen` | string | UTC ISO timestamp |

**Relationships:**
- `IN_NAMESPACE` → `Namespace`

---

### Image
Deduplicated across nodes.

| Property | Type | Notes |
|---|---|---|
| `id` | string | **[merge key]** — `name + sizeInBytes` |
| `name` | string | |
| `size_bytes` | int | |
| `last_seen` | string | UTC ISO timestamp |

---

### Label
Globally deduplicated.

| Property | Type | Notes |
|---|---|---|
| `id` | string | **[merge key]** — `key + value` |
| `key` | string | |
| `value` | string | |

---

### Annotation
Globally deduplicated.

| Property | Type | Notes |
|---|---|---|
| `id` | string | **[merge key]** — `key + value` |
| `key` | string | |
| `value` | string | |

---

### Taint

| Property | Type | Notes |
|---|---|---|
| `id` | string | **[merge key]** — `key + effect` |
| `key` | string | |
| `value` | string | |
| `effect` | string | NoSchedule \| PreferNoSchedule \| NoExecute |

---

## Complete Relationship List

| Relationship | From | To | Notes |
|---|---|---|---|
| `BELONGS_TO` | K8sNode, Namespace | Cluster | |
| `IN_NAMESPACE` | Pod, Deployment, ReplicaSet, StatefulSet, DaemonSet, Service, Ingress, ServiceAccount, PVC, ConfigMap | Namespace | New |
| `SCHEDULED_ON` | Pod | K8sNode | Match by k8s UID (not name) |
| `MANAGED_BY` | Pod | ReplicaSet \| StatefulSet \| DaemonSet | Via ownerReferences |
| `MANAGED_BY` | ReplicaSet | Deployment | |
| `MANAGES` | StatefulSet, DaemonSet | Pod | New |
| `RUNS_CONTAINER` | Pod | Container | |
| `RUNS_AS` | Pod | ServiceAccount | New |
| `MOUNTS` | Pod | PersistentVolumeClaim | New |
| `BOUND_TO` | PersistentVolumeClaim | PersistentVolume | New |
| `PROVISIONED_BY` | PersistentVolume | StorageClass | New |
| `INSTANTIATES_IMAGE` | Container | Image | |
| `CONFIGMAP_REF` | Container | ConfigMap | |
| `HAS_SELECTOR` | Service | Label | For pod matching |
| `EXPOSES` | Service | Pod | Dynamically matched |
| `ROUTES_TO` | Ingress | Service | |
| `HAS_LABEL` | Pod, Deployment, ReplicaSet, StatefulSet, DaemonSet, K8sNode, Service | Label | |
| `HAS_ANNOTATION` | Pod, Deployment, ReplicaSet, StatefulSet, DaemonSet, K8sNode, Service, Ingress | Annotation | |
| `HAS_TAINT` | K8sNode | Taint | |
| `STORES_IMAGE` | K8sNode | Image | |

---

## KubeInsights Event Actions

The Go watcher publishes JSON events with this envelope:

```json
{
  "id": "<uuid>",
  "timestamp": "<ISO string>",
  "action": "<action>",
  "kind": "<kind>",
  "body": { ... }
}
```

| Action | Kind | Notes |
|---|---|---|
| `Add` | `Cluster`, `Node`, `Namespace`, `Pod`, `Container`, `Deployment`, `ReplicaSet`, `StatefulSet`, `DaemonSet`, `Service`, `Ingress`, `ServiceAccount`, `ConfigMap`, `Image`, `PersistentVolumeClaim`, `PersistentVolume`, `StorageClass` | |
| `Delete` | `Pod`, `Node`, `Namespace` | Others implicitly orphaned |
| `Update` | `NodeMetrics`, `PodMetrics` | Polled every 2s |
| `Done` | — | Signals end of initial snapshot |

---

## Known Fixes (Existing Bugs)

These are issues in the current codebase that must be corrected as part of this work, not new features:

1. **`cluster_id` missing on `Pod`, `Deployment`, `ReplicaSet`, `Container`, `ConfigMap`** — all nodes must carry `cluster_id` for multi-cluster correctness.
2. **`SCHEDULED_ON` matches by node name** — must match by k8s UID to survive node replacement.
3. **No `last_seen` timestamps** — must be added to every node `MERGE`.
