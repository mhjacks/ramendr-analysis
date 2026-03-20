# RamenDR Glossary

## Ramen CRDs (API group: ramendr.openshift.io/v1alpha1)

| Abbrev | Full Name | Where | Purpose |
|--------|-----------|-------|---------|
| **DRPC** | DRPlacementControl | Hub | Top-level DR orchestration CR. Binds an application (via Placement) to a DRPolicy. Drives failover/relocate. |
| **VRG** | VolumeReplicationGroup | Managed cluster | Groups PVCs for protection. Creates VolumeReplication or VolumeGroupReplication under the hood. Manages kube object backup via Velero. |
| **DRPolicy** | DRPolicy | Hub | Defines DR scheduling interval, replication class selectors, and pairs two DRClusters. |
| **DRCluster** | DRCluster | Hub | Represents a managed cluster in DR context: CIDRs, fencing state, region, S3 profile. |
| **DRClusterConfig** | DRClusterConfig | Managed cluster | Cluster identity, storage provider replication schedules. |
| **PVRGLIST** | ProtectedVolumeReplicationGroupList | Managed cluster | List of VRGs stored in S3 for discovery. |
| **MaintenanceMode** | MaintenanceMode | Managed cluster | Puts storage into maintenance mode for failover. |
| **RGS** | ReplicationGroupSource | Managed cluster | VolSync source for volume group snapshot replication. |
| **RGD** | ReplicationGroupDestination | Managed cluster | VolSync destination for volume group snapshot replication. |

## CSI-Addons CRDs (API group: replication.storage.openshift.io/v1alpha1)

| Abbrev | Full Name | Purpose |
|--------|-----------|---------|
| **VR** | VolumeReplication | Replicates a single PVC. Spec includes `replicationState: primary/secondary` and references a VRC. |
| **VRC** | VolumeReplicationClass | Defines replication parameters: provisioner, mirroring mode (snapshot/journal), scheduling interval. |
| **VGR** | VolumeGroupReplication | Replicates a group of PVCs via label selector (`source.selector`). Controller auto-creates VGRC and per-volume VRs. Requires CSI Addons v0.13+. |
| **VGRC** | VolumeGroupReplicationClass | Defines group replication parameters. |
| **VGRContent** | VolumeGroupReplicationContent | Tracks the backend volume group replication state. Created by VGR controller. |
| **VG** | VolumeGroup | Groups volumes via CSI CreateVolumeGroup gRPC. Used with Method 2 (VolumeGroupEnableReplication). Not in official release. |
| **VGClass** | VolumeGroupClass | Defines volume group parameters. |
| **VGContent** | VolumeGroupContent | Tracks backend volume group. |
| **NetworkFence** | NetworkFence | CSI-addon for network fencing — blocks I/O from a fenced cluster to storage. |
| **CSIAddonsNode** | CSIAddonsNode | Registers a CSI driver's addons capabilities (replication, fencing, etc.). |

## Kubernetes / Storage CRDs

| Abbrev | Full Name | Purpose |
|--------|-----------|---------|
| **PVC** | PersistentVolumeClaim | Request for storage. Binds to a PV. |
| **PV** | PersistentVolume | Actual storage volume provisioned by CSI driver. |
| **SC** | StorageClass | Defines storage provisioner and parameters (e.g. `rook-ceph-block`). |
| **VSC** | VolumeSnapshotClass | Defines snapshot provisioner parameters. |
| **VS** | VolumeSnapshot | Point-in-time snapshot of a PV. |
| **VSContent** | VolumeSnapshotContent | Backend snapshot object. |

## OCM / ACM CRDs

| Abbrev | Full Name | Purpose |
|--------|-----------|---------|
| **Placement** | Placement | OCM scheduling primitive — selects which managed cluster(s) to deploy to. |
| **PlacementDecision** | PlacementDecision | Result of Placement evaluation — lists selected clusters. |
| **MW** | ManifestWork | OCM mechanism to deploy resources from hub to managed cluster. |
| **MCV** | ManagedClusterView | OCM mechanism to read resources on managed cluster from hub. |
| **ManagedCluster** | ManagedCluster | Represents a cluster registered with OCM hub. |
| **MCS** | ManagedClusterSet | Groups managed clusters. |
| **KAC** | KlusterletAddonConfig | Configures ACM add-ons on a managed cluster. |
| **PlacementRule** | PlacementRule | Legacy OCM placement (deprecated, replaced by Placement). |
| **PlacementBinding** | PlacementBinding | Binds a Policy to a Placement for policy-based governance. |

## Platform / Infrastructure

| Abbrev | Full Name | What It Is |
|--------|-----------|------------|
| **ACM** | Advanced Cluster Management | Red Hat product built on OCM. Multi-cluster management for OpenShift. |
| **OCM** | Open Cluster Management | Upstream multi-cluster orchestration project. Hub-spoke model. |
| **ODF** | OpenShift Data Foundation | Red Hat storage platform (Ceph-based) for OpenShift. |
| **OCP** | OpenShift Container Platform | Red Hat's Kubernetes distribution. |
| **CSI** | Container Storage Interface | Standard interface between Kubernetes and storage drivers. |
| **CSI-Addons** | CSI Addons | Extensions to CSI spec for replication, fencing, reclaim space, etc. |
| **VolSync** | VolSync | Rsync-based asynchronous volume replication (alternative to CSI replication). |
| **Velero** | Velero | Kubernetes backup/restore tool. Used by VRG for kube object protection. |
| **OADP** | OpenShift API for Data Protection | Red Hat's Velero distribution for OpenShift. |
| **RBD** | RADOS Block Device | Ceph block storage. Used for cross-cluster mirroring. |
| **CephFS** | Ceph File System | Ceph filesystem. Uses VolSync for replication (no native cross-cluster mirror). |
| **Rook** | Rook | Kubernetes operator that deploys and manages Ceph clusters. |
| **Submariner** | Submariner | Cross-cluster networking (L3 connectivity between managed clusters). |
| **ArgoCD** | ArgoCD | GitOps continuous delivery tool. Deploys applications via ApplicationSets. |
| **Hive** | Hive | OpenShift cluster provisioning operator (creates ClusterDeployments). |
| **MinIO** | MinIO | S3-compatible object storage (used for Velero backend in dev/test). |
| **OLM** | Operator Lifecycle Manager | Manages operator installation and upgrades on OpenShift. |
| **CNV** | Container-native Virtualization | OpenShift Virtualization (KubeVirt-based). |
| **OTS** | Object Transport System | Proposed abstraction to replace OCM's ManifestWork/MCV (from ramendr-analysis). |

## DR Operations

| Term | Meaning |
|------|---------|
| **Failover** | Unplanned DR: primary cluster is down, promote secondary to primary. |
| **Relocate** | Planned DR: graceful migration of workload from one cluster to another. |
| **Failback** | Return workload to original cluster after failover. |
| **Promote** | Make a secondary volume the new primary (accept writes). |
| **Demote** | Make a primary volume secondary (stop writes, become read-only replica). |
| **Resync** | Re-synchronize a secondary volume with the primary after promotion/demotion. |
| **Fencing** | Block I/O from a cluster to storage (prevents split-brain). Uses NetworkFence CRD. |
| **RPO** | Recovery Point Objective — maximum acceptable data loss (time). |
| **RTO** | Recovery Time Objective — maximum acceptable downtime. |
| **Regional DR** | DR across geographically separated clusters (async replication). |
| **Metro DR** | DR across nearby clusters (sync replication or stretched storage). |

## Ramen Architecture

| Term | Meaning |
|------|---------|
| **Hub operator** | Ramen controller on the OCM hub. Manages DRPolicy, DRCluster, DRPC. |
| **DR cluster operator** | Ramen controller on each managed cluster. Manages VRG, DRClusterConfig, MaintenanceMode. |
| **RamenConfig** | ConfigMap-based config: `ramenControllerType` (dr-hub or dr-cluster), S3 profiles, VolSync settings. |
| **drenv** | Python-based test environment tool for multi-cluster setup (minikube, Lima). |
| **Recipe** | Vendor-defined workflow for capture/recovery ordering. |
| **Discovered apps** | Applications discovered via Velero (not deployed via GitOps). |

## Testing

| Term | Meaning |
|------|---------|
| **envtest** | controller-runtime's in-process API server for unit/integration tests. |
| **basic-test** | Python-based DR flow test (deploy, enable, failover, relocate, disable, undeploy). |
| **E2E** | Go-based end-to-end DR tests with deployers (AppSet, Subscription, DiscoveredApp). |
| **drenv** | Python environment manager creating minikube/Lima clusters with addons. |
| **CSI replication tests** | Nadav's bash-based test framework testing raw VR/VGR operations on two clusters. |

