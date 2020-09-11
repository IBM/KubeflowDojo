## Kubeflow Deployment on OpenShift Container Platform

This guide describes how to deploy Kubeflow on OpenShift clusters. There are two approaches to deploy Kubeflow, either through the `kfctl` CLI or the Kubeflow operator. Choose the one that works for you.

* Deploy with `kfctl` CLI

  For users who want to install the latest supported Kubeflow on Openshift, follow the [Kubeflow on OpenShift](https://www.kubeflow.org/docs/openshift/) guide on Kubeflow document website. This will install Kubeflow with Kubeflow [Pipelines](https://github.com/kubeflow/pipelines).

  For users who rather want to run [Kubeflow Pipelines with Tekton](https://github.com/kubeflow/kfp-tekton), a KfDef configuration file is provided in [manifests](manifests) directory. Users should follow the [KFP-Tekton install instructions on OpenShift in this link](manifests/README.md) to deploy.

* Deploy with the Kubeflow Operator

  Kubeflow can also be deployed through the Kubeflow Operator. There are also two approaches to install the Kubeflow Operator and deploy Kubeflow, either install manually with command line or install through OpenShift web console.
  
  ***Approach #1:*** Users can follow the [Installing the Kubeflow Operator with kustomize and kubectl](https://www.kubeflow.org/docs/operator/install-operator/#2-installing-the-kubeflow-operator-with-kustomize-and-kubectl) guide on Kubeflow document website to install the operator manually. Once the operator is installed, follow the [Install Kubeflow](https://www.kubeflow.org/docs/operator/install-kubeflow/) guide to deploy Kubeflow. Users can set `KFDEF_URL` to either `https://raw.githubusercontent.com/opendatahub-io/manifests/v1.0-branch-openshift/kfdef/kfctl_openshift.yaml` for Kubeflow install with Kubeflow Pipelines or `https://raw.githubusercontent.com/IBM/KubeflowDojo/master/OpenShift/manifests/kfctl_openshift_tekton.v1.1.0.yaml` for Kubeflow install with Tekton Pipelines.

  ***Approach #2:*** Since OpenShift Container Platform 4.x, the OpenShift web console integrates with the Operators catalog where users can search for operators and install with several simple clicks. Users can follow the guide in this [link](operator/README.md) to add the Kubeflow Operator to the Operators catalog on the OpenShift cluster and install the operator then deploy the Kubeflow from the OpenShift web console. 
