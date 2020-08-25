## Deploy Kubeflow with Tekton pipeline

### Prepare cluster environment

* Set up default StorageClass

  A default storageclass is required to deploy Kubeflow. To check if your cluster has a default storageclass, run

  ```shell
  oc get storageclass
  NAME                                 PROVISIONER                     AGE
  rook-ceph-block-internal (default)   rook-ceph.rbd.csi.ceph.com      27h
  rook-ceph-cephfs-internal            rook-ceph.cephfs.csi.ceph.com   27h
  rook-ceph-delete-bucket-internal     ceph.rook.io/bucket             27h
  ```

  The default storageclass should have the `(default)` attached to its name. To make a storageclass the default storageclass for the cluster, run

  ```shell
  kubectl patch storageclass rook-ceph-block-internal -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
  ```

  Replace `rook-ceph-block-internal` with your desired storageclass.

  Make sure there is only one default storageclass. To unset a storageclass as default, run


  ```shell
  kubectl patch storageclass rook-ceph-block-internal -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'
  ```

  Replace `rook-ceph-block-internal` with your desired storageclass.

### Deploy Kubeflow with Tekton pipeline

Use this [KfDef configuration](./kfctl_ibm_dex_multi_user_tekton_V1.1.0.yaml) file to deploy the required components for multi-user Kubeflow with Tekton pipeline.
