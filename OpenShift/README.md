## Kubeflow Deployment on OpenShift Container Platform

There are two approaches to deploy Kubeflow on OpenShift clusters, either through the `kfctl` CLI or the Kubeflow operator. This directory contains the KfDef configurations and install instructions for both approaches. Choose the one that works for you.

* Deploy with `kfctl` CLI

  Follow these steps to download the `kfctl` binary from the kfctl project's release [page](https://github.com/kubeflow/kfctl/releases/tag/v1.1.0).

  ```shell
  wget https://github.com/kubeflow/kfctl/releases/download/v1.1.0/kfctl_v1.1.0-0-g9a3621e_$(uname | tr '[:upper:]' '[:lower:]').tar.gz
  tar zxvf kfctl_v1.1.0-0-g9a3621e_$(uname | tr '[:upper:]' '[:lower:]').tar.gz
  chmod +x kfctl
  mv kfctl /usr/local/bin
  ```

  The [manifests](manifests) directory maintains the manifests for Kubeflow v1.0.0 and v1.1.0.
  
  The KfDef configuration [file](manifests/v1.0.0/kfctl_openshift.v1.0.0.yaml) for v1.0.0 is a copied from the Open Data Hub [manifest](https://github.com/opendatahub-io/manifests/blob/v1.0-branch-openshift/kfdef/kfctl_openshift.yaml). This will deploy Kubeflow with many supported applications hosted in Kubeflow manifests [repo](https://github.com/kubeflow/manifests), including Kubeflow [Pipelines](https://github.com/kubeflow/pipelines) v0.2.5, Istio, Jupyter Notebook, PyTorch operator, Tensorflow operator and more.

  The KfDef configuration [file](manifests/v1.1.0/kfctl_tekton_openshift_minimal.v1.1.0.yaml) for v1.1.0 integrates the [Tekton pipelines](https://github.com/kubeflow/kfp-tekton) with Kubeflow. It is a minimal set of manifests to run ML pipelines with Tekton on Kubeflow. It also includes a couple of applications, such as Spark operator and Jupyter Notebook. More applications will be added once verified.

  Once you choose which version of Kubeflow to deploy, run

  ```shell
  export KFDEF_DIR=<path_to_kfdef>
  mkdir -p ${KFDEF_DIR}
  cd ${KFDEF_DIR}
  export CONFIG_URI=https://raw.githubusercontent.com/IBM/KubeflowDojo/master/OpenShift/manifests/v1.1.0/kfctl_tekton_openshift_minimal.v1.1.0.yaml
  kfctl apply -V -f ${CONFIG_URI}
  ```

  Change the `CONFIG_URI` value if choosing the other KfDef manifest.

  Refer to documents in each sub directory of [manifests](manifests) for more detailed install instructions.

* Deploy with the Kubeflow Operator

Kubeflow can also be deployed through the Kubeflow Operator. We are working on adding the Kubeflow Operator to the Operators catalog and to be seen from the OpenShift Container Platform's Web Console. Before that is done, please follow the [document](operator/README.md) in [operator](operator) directory to manually add the operator to the catalog and then deploy Kubeflow.
