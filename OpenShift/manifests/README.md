## Deploy Kubeflow Pipelines with Tekton backend on OpenShift Container Platform

### Prepare OpenShift cluster environment

* Uninstall OpenShift Pipelines

  If the tech-preview OpenShift Pipelines is installed on your OpenShift Container Platform 4.3+ cluster, remove it from your cluster before process further. The version of Tektoncd used by the OpenShift Pipelines is v0.13.0. It won't work with the Kubeflow kfp-tekton project, which requires a minimum Tekton version of v0.14
  
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

* Download `kfctl`

  Follow these steps to download the `kfctl` binary from the kfctl project's release [page](https://github.com/kubeflow/kfctl/releases/tag/v1.1.0).

  ```shell
  wget https://github.com/kubeflow/kfctl/releases/download/v1.1.0/kfctl_v1.1.0-0-g9a3621e_$(uname | tr '[:upper:]' '[:lower:]').tar.gz
  tar zxvf kfctl_v1.1.0-0-g9a3621e_$(uname | tr '[:upper:]' '[:lower:]').tar.gz
  chmod +x kfctl
  mv kfctl /usr/local/bin
  ```

### Deploy Kubeflow Pipelines with Tekton

There are single-user and multi-user options to deploy Kubeflow. Choose one that works for your use case.

* Single-user

  Use this [KfDef configuration](./kfctl_tekton_openshift_minimal.v1.1.0.yaml) file to deploy the minimal required components for single-user Kubeflow with Tekton pipeline.

  ```shell
  export KFDEF_DIR=<path_to_kfdef>
  mkdir -p ${KFDEF_DIR}
  cd ${KFDEF_DIR}
  export CONFIG_URI=https://raw.githubusercontent.com/IBM/KubeflowDojo/master/OpenShift/manifests/kfctl_tekton_openshift_minimal.v1.1.0.yaml
  kfctl apply -V -f ${CONFIG_URI}
  ```

* Multi-user
  
  Coming soon.

### Set up routes to Kubeflow Pipelines and Tekton Pipelines dashboards

Run with following command to expose the dashboards.

```shell
oc expose svc ml-pipeline-ui -n kubeflow
kfp_ui="http://"$(oc get routes -n kubeflow|grep pipeline-ui|awk '{print $2}')
oc expose svc tekton-dashboard -n tekton-pipelines
tekton_ui="http://"$(oc get routes -n tekton-pipelines|grep dashboard|awk '{print $2}')
```

`$kfp_ui` is the url for the Kubeflow Pipelines UI and `$tekton_ui` is the url for the Tekton Dashboard.
