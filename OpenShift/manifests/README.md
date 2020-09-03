## Deploy Kubeflow Pipelines with Tekton backend on OpenShift Container Platform

- [Deploy Kubeflow Pipelines with Tekton backend on OpenShift Container Platform](#deploy-kubeflow-pipelines-with-tekton-backend-on-openshift-container-platform)
  - [Prepare OpenShift cluster environment](#prepare-openshift-cluster-environment)
  - [Deploy Kubeflow Pipelines with Tekton backend](#deploy-kubeflow-pipelines-with-tekton-backend)
    - [1. Leverage OpenShift Pipelines](#1-leverage-openshift-pipelines)
    - [2. Include Tekton Pipelines as part of deployment](#2-include-tekton-pipelines-as-part-of-deployment)
  - [Set up routes to Kubeflow Pipelines and Tekton Pipelines dashboards](#set-up-routes-to-kubeflow-pipelines-and-tekton-pipelines-dashboards)
  - [Update configmap when running with OpenShift Pipelines](#update-configmap-when-running-with-openshift-pipelines)

### Prepare OpenShift cluster environment

* Install Tekton Pipelines CLI

  Follow this [link](https://github.com/tektoncd/cli) to install Tekton Pipelines CLI.

  ```shell
  # Get the tar.gz
  curl -LO https://github.com/tektoncd/cli/releases/download/v0.12.0/tkn_0.12.0_$(uname -sm|awk '{print $1"_"$2}').tar.gz
  # Extract tkn to your PATH (e.g. /usr/local/bin)
  sudo tar xvzf tkn_0.12.0_$(uname -sm|awk '{print $1"_"$2}').tar.gz -C /usr/local/bin tkn
  ```

* Check OpenShift Pipelines

  Depending on how the OpenShift Container Platform is configured and installed, the [OpenShift Pipelines](https://docs.openshift.com/container-platform/4.4/pipelines/understanding-openshift-pipelines.html) may already exist on your cluster. Or your cluster may have [Tekton Pipelines](https://github.com/tektoncd/pipeline) installed previously for other uses.

  To verfiy, run

  ```shell
  tkn version
  ```

  If the `Pipeline version` in the output is `unknown` or greater than `v0.14.0`, then continue to next step.
  
  Otherwise, the existing version won't work with the Kubeflow kfp-tekton project, which requires a minimum Tekton version of v0.14.0. Remove it from your cluster before process further.
  
* Set up default StorageClass

  A default storageclass is required to deploy Kubeflow. To check if your cluster has a default storageclass, run

  ```shell
  oc get storageclass
  NAME                                 PROVISIONER                     AGE
  rook-ceph-block-internal (default)   rook-ceph.rbd.csi.ceph.com      27h
  rook-ceph-cephfs-internal            rook-ceph.cephfs.csi.ceph.com   27h
  rook-ceph-delete-bucket-internal     ceph.rook.io/bucket             27h
  ```

  The default storageclass should have the **`(default)`** attached to its name. To make a storageclass the default storageclass for the cluster, run

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

### Deploy Kubeflow Pipelines with Tekton backend

As explained in the [Prepare OpenShift cluster environment](#prepare-openshift-cluster-environment) section, your cluster may have pre-installed with OpenShift Pipelines product. Kubeflow Pipelines can leverage the OpenShift Pipelines as the Tekton backend. On the other hand, you can choose to install the Tekton Pipelines as part of the Kubeflow Pipelines deployment. Choose one of the approaches feasible to your cluster.

#### 1. Leverage OpenShift Pipelines

Choose [kfctl_openshift_pipelines.v1.1.0.yaml](./kfctl_openshift_pipelines.v1.1.0.yaml) to deploy the minimal required components for single-user Kubeflow with Tekton backend. Run

```shell
export KFDEF_DIR=<path_to_kfdef>
mkdir -p ${KFDEF_DIR}
cd ${KFDEF_DIR}
export CONFIG_URI=https://raw.githubusercontent.com/IBM/KubeflowDojo/master/OpenShift/manifests/kfctl_openshift_pipelines.v1.1.0.yaml
kfctl apply -V -f ${CONFIG_URI}
```

#### 2. Include Tekton Pipelines as part of deployment

Choose [kfctl_tekton_openshift_minimal.v1.1.0.yaml](./kfctl_tekton_openshift_minimal.v1.1.0.yaml) to deploy the minimal required components for single-user Kubeflow with Tekton backend. Run

```shell
export KFDEF_DIR=<path_to_kfdef>
mkdir -p ${KFDEF_DIR}
cd ${KFDEF_DIR}
export CONFIG_URI=https://raw.githubusercontent.com/IBM/KubeflowDojo/master/OpenShift/manifests/kfctl_tekton_openshift_minimal.v1.1.0.yaml
kfctl apply -V -f ${CONFIG_URI}
```

### Set up routes to Kubeflow Pipelines and Tekton Pipelines dashboards

Run with following command to expose the dashboards.

```shell
oc expose svc ml-pipeline-ui -n kubeflow
kfp_ui="http://"$(oc get routes -n kubeflow|grep pipeline-ui|awk '{print $2}')
oc expose svc tekton-dashboard -n tekton-pipelines
tekton_ui="http://"$(oc get routes -n tekton-pipelines|grep dashboard|awk '{print $2}')
```

`$kfp_ui` is the url for the Kubeflow Pipelines UI and `$tekton_ui` is the url for the Tekton Dashboard.

### Update configmap when running with OpenShift Pipelines

If you choose to deploy Kubeflow Pipelines with Tekton backend using OpenShift Pipelines product, supported via this KfDef Configuration [kfctl_openshift_pipelines.v1.1.0.yaml](./kfctl_openshift_pipelines.v1.1.0.yaml), you need to update the following configmap to support the use cases where users use `$HOME` variable in their containers when running pipelines.

```shell
TEKTON_PIPELINES_NAMESPACE=openshift-pipelines
cat <<EOF |oc apply -f - -n $TEKTON_PIPELINES_NAMESPACE
apiVersion: v1
kind: ConfigMap
metadata:
  name: feature-flags
data:
  disable-affinity-assistant: "false"
  disable-home-env-overwrite: "true"
  disable-working-directory-overwrite: "true"
  running-in-environment-with-injected-sidecars: "true"
EOF
oc rollout restart deployment/tekton-pipelines-controller -n $TEKTON_PIPELINES_NAMESPACE
```

Note: change **`TEKTON_PIPELINES_NAMESPACE`** to the namespace where Tekton pipelines is installed on your cluster.
