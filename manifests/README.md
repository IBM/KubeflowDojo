## Deploy Kubeflow with Tekton pipeline

### Prepare cluster environment

This is only required for deployments on OpenShift clusters. OpenShift Container Platform 4.3+ comes with the default OpenShift Pipelines, which is the Tektoncd v0.13.0. This version of Tektoncd won't work with the Kubeflow kfp-tekton project. So before procceed further, delete the OpenShift Pipelines from your cluster.

### Deploy Kubeflow with Tekton pipeline

Use this [KfDef configuration](./kfctl_ibm_tekton.v1.1.0.yaml) file to deploy the minimal required components for single-user Kubeflow with Tekton pipeline. Uncomment the `openshift-scc` application in the KfDef manifest for OpenShift Container Platforms.

|Application|Version|
|---|---|
|KfDef configuration|kustomize v3|
|Kubeflow|v1.1.0|
|Kubeflow Pipelines|v1.0.0|
|Tektoncd|v0.14.0|

Since Kubeflow central dashboard component is not part of this installation, access the Kubeflow Pipelines UI by forwarding the port on the native Kubernetes cluster or by creating a route on the OCP for the `ml-pipeline-ui` service in the `kubeflow` namespace.
