# Prepare for Kubeflow Dojo

[Kubeflow](https://github.com/kubeflow) is community driven open source mega project to make the deployment of AI/ML stacks on cloud native platforms, such as Kubernetes and OpenShift Container Platform. Read the [document](http://kubeflow.org) to learn more about Kubeflow. For anyone who is interested in the **Kubeflow** project and would like to contribute, browsing through the [`community`](https://github.com/kubeflow/community) project is a good start. You will get the info of the community behind the Kubeflow, study the architecture and design and involve in the community discussion.

This Kubeflow live and recorded dojo will help you understand the project deeper by presenting the architecture design and code walkthrough of major components of the Kubeflow project. You will also participate in several handson sessions to get a head start messing with the project.

However, before anyone comes to the dojo materials and thinks to contribute, he should also prepare himself with a list of things.

1. ***Kubernetes***: A broad knowledge of cloud computing and Kubernetes is must. To help this, you may watch the dojo for Kubernetes project with the [IBM internal link](https://w3.ibm.com/developer/docs/open-source/kubernetes/) or [external link](https://video.ibm.com/embed/recorded/126773520).

Most importantly, for anyone interesting in Kubeflow project the access to a Kuberenetes cluster is a must. The minimal cluster configuration for running most applications of Kubeflow is **8 vcpu, 16gb memory and at least 50gb of disk for docker registry**.

Kubeflow deployment has been tested with Kubernetes up to 1.18 release. However, there are known issues with certain applications on latest Kuberenetes release. So we recommend run Kubeflow on Kubernetes 1.16 release.

Note that during the live session for this dojo, few IBM Cloud Kubernetes clusters will be provided for hands on workshops. But the amount of the clusters is very limited and the clusters will be deleted once the workshop ends.

Alternatively, just for the purpose of this dojo, a one-node [`minikube`](https://kubernetes.io/docs/tutorials/hello-minikube/) cluster running on a laptop is enough. If you need to create a minikube cluster before attending the dojo presentation, follow the [minikube-setup.md](minikube-setup.md) to set up a `minikube` cluster.

2. ***git*** and ***github***: The Kubeflow project source code is hosted on [github](https://github.com/kubeflow). Code delivery is through `git` command. You may watch the presentation with the [IBM internal link](https://w3.ibm.com/developer/docs/open-source/general-open-source/) or [external link for git](https://video.ibm.com/embed/recorded/126773542) and [external link for pull requests](https://video.ibm.com/embed/recorded/126773518).

3. ***golang***: The main source codes are either in `golang` or `python` for the Kubeflow project. To learn more on the language, go to [IBM internal link](https://w3.ibm.com/developer/docs/open-source/general-open-source/) or [external link](https://video.ibm.com/embed/recorded/126773543).

4. ***(Optional) Istio*** and ***KNative***:Istio is a service mesh. In Kubeflow project, istio provides ingress and egress gateways. Follow [IBM internal link](https://w3.ibm.com/developer/docs/open-source/istio/) or [external link](https://video.ibm.com/embed/recorded/126773530) to learn more about istio. KNative is cloud native platform to deploy and manage modern serverless workloads. Together with kfserving in Kubeflow, the serverless inferencing becomes simple. Follow the [IBM internal link](https://w3.ibm.com/developer/docs/open-source/knative/) and [external link](https://video.ibm.com/embed/recorded/126773537) to learn more about knative.

There are dozens of projects around **Kubeflow**, including some of the most active. Please browse through them.

* [kubeflow](https://github.com/kubeflow/kubeflow) the main anchor project
* [pipelines](https://github.com/kubeflow/pipelines) the ML pipelines
* [manifests](https://github.com/kubeflow/manifests) the manifest repoository for Kubeflow applications
* [kfp-tekton](https://github.com/kubeflow/kfp-tekton) the pipeline compiler to transform a Kubeflow Pipeline DSL to Tekton yaml manifest
* [katib](https://github.com/kubeflow/katib) the repository for hyperparameter tuning
* [kfserving](https://github.com/kubeflow/kfserving) the serverless inferencing on Kubernetes
* [kfctl](https://github.com/kubeflow/kfctl) the control plane to deploy and maintain Kubeflow
* [website](https://github.com/kubeflow/website) the document
* and more
