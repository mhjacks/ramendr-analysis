# Example of a successfully updated ManagedClusterView CR for a VolumeReplicationGroup CR

```yaml
apiVersion: view.open-cluster-management.io/v1beta1
kind: ManagedClusterView
metadata:
  annotations:
    drplacementcontrol.ramendr.openshift.io/drpc-name: nick-deploy-busybox
    drplacementcontrol.ramendr.openshift.io/drpc-namespace: ramen-ops
  creationTimestamp: "2026-02-17T20:03:18Z"
  generation: 1
  labels:
    ramendr.openshift.io/created-by-ramen: "true"
  name: nick-deploy-busybox-ramen-ops-vrg-mcv
  namespace: dr1
  resourceVersion: "4615"
  uid: 946e8c82-0416-4668-8c77-42f6dacde278
spec:
  scope:
    apiGroup: ramendr.openshift.io
    kind: VolumeReplicationGroup
    name: nick-deploy-busybox
    namespace: ramen-ops
    version: v1alpha1
status:
  conditions:
    - lastTransitionTime: "2026-02-17T20:03:48Z"
      message: Watching resources successfully
      reason: GetResourceProcessing
      status: "True"
      type: Processing
  result:
    apiVersion: ramendr.openshift.io/v1alpha1
    kind: VolumeReplicationGroup
    metadata:
      annotations:
        drplacementcontrol.ramendr.openshift.io/destination-cluster: dr1
        drplacementcontrol.ramendr.openshift.io/do-not-delete-pvc: ""
        drplacementcontrol.ramendr.openshift.io/drpc-uid: af2dde75-6859-4fd9-b890-e44bb7e7bed8
        drplacementcontrol.ramendr.openshift.io/is-submariner-enabled: ""
        drplacementcontrol.ramendr.openshift.io/use-volsync-for-pvc-protection: ""
      creationTimestamp: "2026-02-17T20:03:18Z"
      finalizers:
        - volumereplicationgroups.ramendr.openshift.io/vrg-protection
      generation: 1
      labels:
        ramendr.openshift.io/created-by-ramen: "true"
      name: nick-deploy-busybox
      namespace: ramen-ops
      ownerReferences:
        - apiVersion: work.open-cluster-management.io/v1
          kind: AppliedManifestWork
          name: af81e999aa30f9b69139adc20e85db6b68ebb1aea1fa72fa83fe906f82c43e37-nick-deploy-busybox-ramen-ops-vrg-mw
          uid: 3caabc86-dc49-4a1e-b237-818d2fb2f5f8
      resourceVersion: "6050"
      uid: 815ce5e7-4bf5-447a-b23d-70c597408014
    spec:
      async:
        replicationClassSelector: {}
        schedulingInterval: 1m
        volumeGroupSnapshotClassSelector: {}
        volumeSnapshotClassSelector: {}
      kubeObjectProtection:
        kubeObjectSelector:
          matchExpressions:
            - key: appname
              operator: In
              values:
                - nick-busybox
      protectedNamespaces:
        - test-nick-busybox
      pvcSelector:
        matchLabels:
          appname: nick-busybox
      replicationState: primary
      s3Profiles:
        - minio-on-dr1
        - minio-on-dr2
      volSync:
        disabled: true
    status:
      conditions:
        - lastTransitionTime: "2026-02-17T20:03:19Z"
          message: PVCs in the VolumeReplicationGroup are ready for use
          observedGeneration: 1
          reason: Ready
          status: "True"
          type: DataReady
        - lastTransitionTime: "2026-02-17T20:03:18Z"
          message: VolumeReplicationGroup is replicating
          observedGeneration: 1
          reason: Replicating
          status: "False"
          type: DataProtected
        - lastTransitionTime: "2026-02-17T20:03:18Z"
          message: Nothing to restore
          observedGeneration: 1
          reason: Restored
          status: "True"
          type: ClusterDataReady
        - lastTransitionTime: "2026-02-17T20:03:23Z"
          message: Cluster data of all PVs are protected. VRG object protected. Kube objects protected
          observedGeneration: 1
          reason: Uploaded
          status: "True"
          type: ClusterDataProtected
        - lastTransitionTime: "2026-02-17T20:03:19Z"
          message: Kube objects restored
          observedGeneration: 1
          reason: KubeObjectsRestored
          status: "True"
          type: KubeObjectsReady
        - lastTransitionTime: "2026-02-17T20:03:18Z"
          message: No PVC conflict detected for VolumeSync scheme. No resource conflict
          observedGeneration: 1
          reason: NoConflictDetected
          status: "True"
          type: NoClusterDataConflict
      kubeObjectProtection:
        captureToRecoverFrom:
          endTime: "2026-02-17T20:03:23Z"
          number: 1
          startGeneration: 1
          startTime: "2026-02-17T20:03:22Z"
      lastGroupSyncBytes: 81920
      lastGroupSyncDuration: 0s
      lastGroupSyncTime: "2026-02-17T20:05:00Z"
      lastUpdateTime: "2026-02-17T20:06:19Z"
      observedGeneration: 1
      protectedPVCs:
        - accessModes:
            - ReadWriteOnce
          conditions:
            - lastTransitionTime: "2026-02-17T20:03:19Z"
              message: PVC in the VolumeReplicationGroup is ready for use
              observedGeneration: 1
              reason: Ready
              status: "True"
              type: DataReady
            - lastTransitionTime: "2026-02-17T20:03:18Z"
              message: PV cluster data already protected for PVC nick-busybox-pvc
              observedGeneration: 1
              reason: Uploaded
              status: "True"
              type: ClusterDataProtected
            - lastTransitionTime: "2026-02-17T20:03:19Z"
              message: PVC in the VolumeReplicationGroup is ready for use
              observedGeneration: 1
              reason: Replicating
              status: "False"
              type: DataProtected
          csiProvisioner: rook-ceph.rbd.csi.ceph.com
          labels:
            appname: nick-busybox
            ramendr.openshift.io/owner-name: nick-deploy-busybox
            ramendr.openshift.io/owner-namespace-name: ramen-ops
          lastSyncBytes: 81920
          lastSyncDuration: 0s
          lastSyncTime: "2026-02-17T20:05:00Z"
          name: nick-busybox-pvc
          namespace: test-nick-busybox
          replicationID:
            id: ""
          resources:
            requests:
              storage: 1Gi
          storageClassName: rook-ceph-block
          storageID:
            id: ""
          volumeMode: Filesystem
      state: Primary
```
