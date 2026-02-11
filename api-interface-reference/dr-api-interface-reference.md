# Ramen DR API Interface Reference

This document describes the API surface and status details you need to build an interface (UI, CLI, or external orchestrator) to **configure** and **orchestrate** Disaster Recovery with Ramen.

## Important: No REST API

Ramen does **not** expose its own REST API. All configuration and orchestration is done through the **Kubernetes API** using **Custom Resource Definitions (CRDs)**. Your interface must:

- Talk to the **Kubernetes API server** (on the hub cluster for DRPolicy, DRCluster, DRPlacementControl; on each managed cluster for VolumeReplicationGroup).
- Use standard Kubernetes **HTTP verbs** and **paths** for these custom resources.

## API Group and Version

| Property | Value |
| :------- | :---- |
| **API Group** | `ramendr.openshift.io` |
| **API Version** | `v1alpha1` |
| **Base path** (cluster-scoped) | `/apis/ramendr.openshift.io/v1alpha1/{resource}` |
| **Base path** (namespaced) | `/apis/ramendr.openshift.io/v1alpha1/namespaces/{namespace}/{resource}` |

## Kubernetes “Endpoints” and Verbs

All resources are accessed via the Kubernetes API. Typical usage:

| Operation | HTTP verb | Path pattern (examples) |
| :-------- | :-------- | :------------------------ |
| **List (cluster-scoped)** | GET | `/apis/ramendr.openshift.io/v1alpha1/drpolicies` |
| **List (namespaced)** | GET | `/apis/ramendr.openshift.io/v1alpha1/namespaces/{ns}/drplacementcontrols` |
| **Get** | GET | `.../drpolicies/{name}` or `.../namespaces/{ns}/drplacementcontrols/{name}` |
| **Create** | POST | `.../drpolicies` or `.../namespaces/{ns}/drplacementcontrols` |
| **Update** | PUT/PATCH | Same as Get, with body |
| **Delete** | DELETE | Same as Get |
| **Watch** | GET | Same as List with `?watch=1` |

Use **watch** to get real-time status updates for dashboards and automation.

---

## Resources Overview

| Resource | Scope | Where it lives | Purpose |
| :------- | :---- | :------------- | :------ |
| **DRPolicy** | Cluster | Hub | DR topology: clusters, replication, S3 |
| **DRCluster** | Cluster | Hub | Per-cluster DR config (S3 profile, fencing, region) |
| **DRPlacementControl** | Namespaced | Hub | Per-application DR: protect, relocate, failover |
| **VolumeReplicationGroup** | Namespaced | Managed clusters | Volume replication state (created by Ramen from DRPC) |
| **RamenConfig** | ConfigMap | Hub / each cluster | Operator config (S3 profiles, etc.); not a CRD |

For a **DR configuration and orchestration** interface, you primarily need **DRPolicy**, **DRCluster**, and **DRPlacementControl**. **VolumeReplicationGroup** is needed for detailed per-workload and per-PVC status if you show it (often on managed clusters).

---

## 1. DRPolicy

**Purpose**: Define DR topology (two clusters), replication schedule, and class selectors.

### 1.1 DRPolicy Endpoints (Kubernetes API)

- List: `GET /apis/ramendr.openshift.io/v1alpha1/drpolicies`
- Get: `GET /apis/ramendr.openshift.io/v1alpha1/drpolicies/{name}`
- Create: `POST /apis/ramendr.openshift.io/v1alpha1/drpolicies`
- Update: `PUT` or `PATCH` on the same URL as Get
- Delete: `DELETE` (same as Get)

### 1.2 DRPolicy Spec (Configuration)

| Field | Type | Required | Description |
| :---- | :--- | :------- | :---------- |
| `drClusters` | []string | Yes | Exactly 2 cluster names (DRCluster names) |
| `schedulingInterval` | string | Yes | Replication interval, e.g. `5m`, `1h`, `1d` |
| `replicationClassSelector` | LabelSelector | No | Select VolumeReplicationClass |
| `volumeSnapshotClassSelector` | LabelSelector | No | Select VolumeSnapshotClass (e.g. VolSync) |
| `volumeGroupSnapshotClassSelector` | LabelSelector | No | Select VolumeGroupSnapshotClass |

### 1.3 DRPolicy Status (For Your Interface)

| Field | Type | Description |
| :---- | :--- | :---------- |
| `conditions` | []Condition | e.g. `Validated` |
| `status.async.peerClasses` | []PeerClass | Resolved async replication peer classes (Regional DR) |
| `status.sync.peerClasses` | []PeerClass | Resolved sync replication peer classes (Metro DR) |

Use **conditions** to show validation/errors; use **peerClasses** to show which storage classes are available for DR.

---

## 2. DRCluster

**Purpose**: Per-managed-cluster DR configuration (S3 profile, region, fencing).

### 2.1 DRCluster Endpoints

- List: `GET /apis/ramendr.openshift.io/v1alpha1/drclusters`
- Get: `GET /apis/ramendr.openshift.io/v1alpha1/drclusters/{name}`
- Create / Update / Delete: same patterns as DRPolicy.

### 2.2 DRCluster Spec (Configuration)

| Field | Type | Required | Description |
| :---- | :--- | :------- | :---------- |
| `s3ProfileName` | string | Yes | S3 profile name from RamenConfig (for PV metadata) |
| `region` | string | No | Region; clusters in same region are Metro DR peers |
| `clusterFence` | string | No | `Unfenced` \| `Fenced` \| `ManuallyFenced` \| `ManuallyUnfenced` |
| `cidrs` | []string | No | Node CIDRs used for fencing |

### 2.3 DRCluster Status (For Your Interface)

| Field | Type | Description |
| :---- | :--- | :---------- |
| `phase` | string | `Available`, `Starting`, `Fencing`, `Fenced`, `Unfencing`, `Unfenced` |
| `conditions` | []Condition | e.g. `Validated`, `Clean`, `Fenced` |
| `maintenanceModes` | []ClusterMaintenanceMode | Storage maintenance mode state per provisioner/target |

Use **phase** and **conditions** for cluster readiness and fencing state in the UI.

---

## 3. DRPlacementControl (DRPC)

**Purpose**: Per-application DR – enable protection, relocate, or failover. This is the main resource your interface will **configure** and **orchestrate**.

### 3.1 DRPC Endpoints

- List: `GET /apis/ramendr.openshift.io/v1alpha1/namespaces/{namespace}/drplacementcontrols`
- Get: `GET /apis/ramendr.openshift.io/v1alpha1/namespaces/{namespace}/drplacementcontrols/{name}`
- Create: `POST /apis/ramendr.openshift.io/v1alpha1/namespaces/{namespace}/drplacementcontrols`
- Update: `PUT` or `PATCH` (use PATCH for partial updates, e.g. only `spec.action` and `spec.failoverCluster`)
- Delete: `DELETE` (same as Get)
- Watch: `GET .../drplacementcontrols?watch=1` (for live status)

### 3.2 DRPC Spec (Configuration and Orchestration)

| Field | Type | Required | Description |
| :---- | :--- | :------- | :---------- |
| `placementRef` | ObjectReference | Yes | Reference to Placement or PlacementRule (immutable) |
| `drPolicyRef` | ObjectReference | Yes | Reference to DRPolicy (immutable) |
| `pvcSelector` | LabelSelector | Yes | Labels to select PVCs to protect (immutable) |
| `preferredCluster` | string | No | Preferred cluster to run the app (initial or after relocate) |
| `failoverCluster` | string | No | Target cluster for **failover**; set when `action=Failover` |
| `action` | string | No | `Failover` \| `Relocate` – triggers the operation |
| `protectedNamespaces` | []string | No | Additional namespaces to protect (unmanaged resources) |
| `kubeObjectProtection` | object | No | Kube object capture/recovery (recipe, interval, selector) |
| `volSyncSpec` | object | No | VolSync-specific config (mover, TLS, etc.) |

**Orchestration (what to set from your UI):**

- **Enable protection**: Create DRPC with `placementRef`, `drPolicyRef`, `pvcSelector`, and optionally `preferredCluster`.
- **Relocate (planned migration)**: Set `spec.action = "Relocate"` and `spec.preferredCluster = <target-cluster>`.
- **Failover (disaster recovery)**: Set `spec.action = "Failover"` and `spec.failoverCluster = <target-cluster>`.
- **Clear action**: After operation completes, you can set `spec.action = ""` and clear `spec.failoverCluster` when applicable.

### 3.3 DRPC Status (Critical for Your Interface)

| Field | Type | Description |
| :---- | :--- | :---------- |
| `phase` | string | High-level state (see below) |
| `progression` | string | Fine-grained step (see below) |
| `preferredDecision` | { clusterName, clusterNamespace } | Current cluster where workload is placed |
| `conditions` | []Condition | Available, PeerReady, Protected (see below) |
| `actionStartTime` | time | When current action started |
| `actionDuration` | duration | Duration of current action |
| `lastUpdateTime` | time | Last status update |
| `lastGroupSyncTime` | time | Last successful sync of all PVCs |
| `lastGroupSyncDuration` | duration | Last sync duration |
| `lastGroupSyncBytes` | int64 | Bytes transferred in last sync |
| `lastKubeObjectProtectionTime` | time | Last kube object capture |
| `resourceConditions` | VRGConditions | Aggregated VRG conditions from managed cluster |

### DRPC Phase (status.phase)

| Value | Meaning |
| :---- | :------ |
| (empty) | Initial / not yet started |
| `WaitForUser` | Waiting for user action (e.g. after hub recovery) |
| `Initiating` | Action is being prepared |
| `Deploying` | Initial deployment in progress |
| `Deployed` | Initial deployment done |
| `FailingOver` | Failover in progress |
| `FailedOver` | Failover completed |
| `Relocating` | Relocation in progress |
| `Relocated` | Relocation completed |
| `Deleting` | DRPC is being deleted |

### DRPC Progression (status.progression)

Use for progress bars or step indicators. Examples:

- `CreatingMW`, `WaitForReadiness`, `CheckingFailoverPrerequisites`, `FailingOverToCluster`, `WaitingForResourceRestore`, `UpdatedPlacement`, `EnsuringVolSyncSetup`, `Completed`, `Paused`, `Deleted`, etc.

Full set is in `api/v1alpha1/drplacementcontrol_types.go` (ProgressionStatus constants).

### DRPC Conditions (status.conditions)

Standard Kubernetes conditions; check `type` and `status` (True/False).

| Type | Meaning |
| :--- | :------ |
| **Available** | Whether the cluster in `preferredDecision` is ready for the workload. Reason can be `Progressing`, `Success`, `Paused`, etc. |
| **PeerReady** | Whether a peer cluster is ready for failover/relocate. Reason e.g. `Success`, `NotStarted`, `Paused`. |
| **Protected** | Whether the workload is protected. Reason: `Protected`, `Progressing`, `Error`, `Unknown`. |

Use **Message** for user-facing text; use **Reason** for programmatic handling.

---

## 4. VolumeReplicationGroup (VRG)

**Purpose**: Per-cluster volume replication state. Usually **created and updated by Ramen** from the DRPC; your interface may **read** it for detailed status (e.g. per-PVC sync status).

### 4.1 Where VRG Lives

VRGs are on **managed clusters**, not the hub. To show VRG status from a hub-based UI you need either OCM ManagedClusterView (current Ramen design) or direct API access to managed clusters.

### 4.2 VRG Endpoints (on each managed cluster)

- List: `GET /apis/ramendr.openshift.io/v1alpha1/namespaces/{namespace}/volumereplicationgroups`
- Get: `GET /apis/ramendr.openshift.io/v1alpha1/namespaces/{namespace}/volumereplicationgroups/{name}`

### 4.3 VRG Spec (Reference Only – Usually Managed by Ramen)

- `pvcSelector`, `replicationState` (primary/secondary), `s3Profiles`, `async`/`sync`/`volSync`, etc.

### 4.4 VRG Status (For Your Interface)

| Field | Type | Description |
| :---- | :--- | :---------- |
| `state` | string | `Primary` \| `Secondary` \| `Unknown` |
| `conditions` | []Condition | Summary conditions (see VRG conditions below) |
| `protectedPVCs` | []ProtectedPVC | Per-PVC status and conditions |
| `lastGroupSyncTime` | time | Last successful sync of all PVCs |
| `lastGroupSyncDuration` | duration | Last sync duration |
| `lastGroupSyncBytes` | int64 | Bytes in last sync |
| `kubeObjectProtection` | object | Capture to recover from (if kube object protection enabled) |
| `prepareForFinalSyncComplete` | bool | Relocation: final sync prepared |
| `finalSyncComplete` | bool | Relocation: final sync done |

### 4.5 VRG Condition Types (status.conditions and protectedPVCs[].conditions)

| Type | Level | Meaning |
| :--- | :---- | :------ |
| **DataReady** | VRG + PVC | PV data ready (e.g. for failover/relocate) |
| **DataProtected** | VRG + PVC | PV data in sync with peer |
| **ClusterDataReady** | VRG | PV/PVC cluster data restored (S3) |
| **ClusterDataProtected** | VRG | PV cluster data uploaded to S3 |
| **KubeObjectsReady** | VRG | Kube objects restored (if enabled) |
| **NoClusterDataConflict** | VRG | No conflict between clusters |
| **AutoCleanup** | VRG | Post-DR cleanup state |
| **ReplicationSourceSetup** | PVC (VolSync) | VolSync source setup |
| **ReplicationDestinationSetup** | PVC (VolSync) | VolSync destination setup |
| **PVsRestored** | PVC (VolSync) | VolSync PVCs restored |
| **FinalSyncInProgress** | PVC (VolSync) | Final sync in progress |

Each condition has **status** (True/False/Unknown), **reason**, and **message** – use these for UI and automation.

---

## 5. RamenConfig (S3 and Operator Config)

RamenConfig is stored in a **ConfigMap**, not a CRD. S3 profiles are inside it.

- **Hub**: ConfigMap name `ramen-hub-operator-config`, key `ramen_manager_config.yaml`.
- **DR cluster**: ConfigMap name `ramen-dr-cluster-operator-config`, key `ramen_manager_config.yaml`.

To **configure S3** (for DR clusters and policies), you need to read/update this ConfigMap and the structure under `s3StoreProfiles`. Each profile has `s3ProfileName`, `s3Bucket`, `s3CompatibleEndpoint`, `s3Region`, `s3SecretRef`, and optional `caCertificates`, `veleroNamespaceSecretKeyRef`. Secrets use keys `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`.

---

## Summary: What You Need for a DR Interface

### To Configure DR

1. **DRPolicy**: Create/update/list – define the two clusters and replication (scheduling interval, class selectors).
2. **DRCluster**: Create/update/list – per-cluster S3 profile, region, fencing.
3. **RamenConfig** (ConfigMap): Read/update S3 profiles and operator settings.
4. **DRPlacementControl**: Create with `placementRef`, `drPolicyRef`, `pvcSelector`, `preferredCluster` to protect an application.

### To Orchestrate DR

1. **DRPlacementControl**:
   - **Relocate**: PATCH `spec.action = "Relocate"`, `spec.preferredCluster = <target>`.
   - **Failover**: PATCH `spec.action = "Failover"`, `spec.failoverCluster = <target>`.
   - **Clear**: PATCH `spec.action = ""`, clear `spec.failoverCluster` when appropriate.

### To Show Status

1. **DRPlacementControl**: Watch/list and display:
   - `status.phase` (high-level state)
   - `status.progression` (current step)
   - `status.preferredDecision.clusterName` (current cluster)
   - `status.conditions` (Available, PeerReady, Protected)
   - `status.actionStartTime`, `status.actionDuration`, `status.lastGroupSyncTime`, etc.
2. **DRPolicy**: `status.conditions`, `status.async.peerClasses`, `status.sync.peerClasses`.
3. **DRCluster**: `status.phase`, `status.conditions`, `status.maintenanceModes`.
4. **VolumeReplicationGroup** (optional, on managed clusters): `status.state`, `status.conditions`, `status.protectedPVCs`, sync times/bytes.

### Suggested Watch Targets

- `DRPlacementControl` in the relevant namespace(s) – for live DR state and progression.
- `DRPolicy` and `DRCluster` – for validation and cluster state (less frequent updates).

This gives you everything needed to implement a UI or orchestrator that configures DR (policies, clusters, applications) and runs relocate/failover while showing accurate status and progression.
