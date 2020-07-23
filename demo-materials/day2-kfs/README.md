# KFServing Demo

This notebook showcases the different client and use cases for KFServing. It can run on the Kubeflow Jupyter Server or the local machine as long as
the remote endpoint is setup correctly in the initial cell.

Before running this notebook, please run the below command to relax the permission for Kubeflow pipeline since we will be using it to deploy Kubernetes
resources accross multiple namespaces.

```
kubectl create clusterrolebinding pipeline-runner-extend --clusterrole cluster-admin --serviceaccount=kubeflow:pipeline-runner
```
