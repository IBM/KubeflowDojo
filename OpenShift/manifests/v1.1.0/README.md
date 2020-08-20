## Deploy Kubeflow with Tekton pipeline on OpenShift Container Platform

### Prepare cluster environment

* Uninstall OpenShift Pipelines

  If the tech-preview OpenShift Pipelines is installed on your OpenShift Container Platform 4.3+ cluster, remove it from your cluster before process further. The version of Tektoncd used by the OpenShift Pipelines is v0.13.0. It won't work with the Kubeflow kfp-tekton project.
  
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

There are single-user and multi-user options to deploy Kubeflow. Choose one that works for your use case.

1. Single-user

Use this [KfDef configuration](./kfctl_tekton_openshift_minimal.v1.1.0.yaml) file to deploy the minimal required components for single-user Kubeflow with Tekton pipeline.

```shell
kfctl apply -V -f https://raw.githubusercontent.com/IBM/KubeflowDojo/master/manifests/openshift/kfctl_tekton_openshift_minimal.v1.1.0.yaml
```

2. Multi-user

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

### Versions

|Application|Version|
|---|---|
|KfDef configuration|kustomize v3|
|Kubeflow|v1.1.0|
|Kubeflow Pipelines|v1.0.0|
|Tektoncd|v0.14.0|
