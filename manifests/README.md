## Deploy Kubeflow with Tekton pipeline

### Prepare cluster environment

This is only required for deployments on OpenShift clusters. OpenShift Container Platform 4.3+ comes with the default OpenShift Pipelines, which is the Tektoncd v0.13.0. This version of Tektoncd won't work with the Kubeflow kfp-tekton project. So before procceed further, delete the OpenShift Pipelines from your cluster.

### Deploy Kubeflow with Tekton pipeline

Use this [KfDef configuration](./kfctl_ibm_dex_multi_user_tekton_V1.1.0.yaml) file to deploy the minimal required components for multi-user Kubeflow with Tekton pipeline. Uncomment the `openshift-scc` application in the KfDef manifest for OpenShift Container Platforms.

|Application|Version|
|---|---|
|KfDef configuration|kustomize v3|
|Kubeflow|v1.1.0|
|Kubeflow Pipelines|v1.0.0|
|Tektoncd|v0.14.0|
