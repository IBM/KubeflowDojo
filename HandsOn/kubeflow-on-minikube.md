* Prereq

  - [Kubernetes](https://w3.ibm.com/developer/docs/open-source/kubernetes/)
  - Kubernetes cluster
  - golang
  - git

Note: if you are running `Docker Desktop` which may come with the option to start a one node Kubernetes, you can use that as well for the following hands on exercise.

* Install `kubectl` and `minikube` following this [link](https://kubernetes.io/docs/tasks/tools/install-minikube/)

  - Install `kubectl`

  ```shell
  curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.15.6/bin/darwin/amd64/kubectl
  chmod +x ./kubectl
  sudo mv ./kubectl /usr/local/bin/kubectl
  ```

  - Install `minikube`

  ```shell
  brew install minikube
  ```

  - Start `minikube`

  ```shell
  minikube start --driver=docker --memory=8192MB --cpus=6 --disk-size=30000mb --kubernetes-version=v1.15.6
  ```

  - Access dashboard

  ```shell
  minikube dashboard&
  ```
  
* Install `go` following this [link](https://golang.org/dl/)

  After install, do following to set up the environment
  
  ```shell
  mkdir -p $HOME/go/src
  export GOPATH=$HOME/go
  ```

* Clone and build `kfctl`

  - Clone

  ```shell
  cd $GOPATH/src
  mkdir -p github.com/kubeflow
  cd github.com/kubeflow
  git clone https://github.com/kubeflow/kfctl.git
  ```

  - Build

  ```shell
  cd kfctl
  make build
  export PATH=$PWD/bin:$PATH
  ```

* Deploy Kubeflow with `kfctl` CLI

```shell
cd $HOME
mkdir kfdef
cd kfdef
kfctl apply -V -f https://raw.githubusercontent.com/kubeflow/manifests/master/kfdef/kfctl_k8s_istio.yaml
```

* Check the deployment

```shell
kubectl get ns
kubectl get pods -n istio-system
kubectl get pods -n kubeflow
```

Wait until all pods and services are up and running in the `kubeflow` namespace than continue to the next steps.

* Access Kubeflow dashboard

  - Port forward `istio-ingressgateway`

  ```shell
  kubectl port-forward --namespace istio-system $(kubectl get pod --namespace istio-system --selector="app=istio-ingressgateway" --output jsonpath='{.items[0].metadata.name}') 8080:80&
  ```

  - Access the dashboard through `localhost:8080`

  - Follow the instructions to have the profile namespace created

  - Run a pipeline tutorial

* `kfctl` source code walkthrough with VSC

  - Install `Go` extension
  - Open the folder to `$GOPATH/src/github.com/kubeflow/kfctl`
  - Run/Debug test from the IDE
  - Debug within VSC

* Deploy Kubeflow with the operator

  - Delete the current Kubeflow

  ```shell
  kfctl delete -f kfctl_k8s_istio.yaml

  kubectl delete mutatingwebhookconfigurations admission-webhook-mutating-webhook-configuration
  kubectl delete mutatingwebhookconfigurations inferenceservice.serving.kubeflow.org
  kubectl delete mutatingwebhookconfigurations istio-sidecar-injector
  kubectl delete mutatingwebhookconfigurations katib-mutating-webhook-config
  kubectl delete mutatingwebhookconfigurations mutating-webhook-configurations
  ```

  - Follow the [instructions](https://github.com/kubeflow/kfctl/blob/master/operator.md) to deploy the Kubeflow operator

  ```shell
  cd $GOPATH/src/github.com/kubeflow/kfctl
  export OPERATOR_NAMESPACE=operators
  kubectl create ns ${OPERATOR_NAMESPACE}

  cd deploy/
  kustomize edit set namespace ${OPERATOR_NAMESPACE}
  kustomize build | kubectl apply -f -
  ```

  - Check `${OPERATOR_NAMESPACE}` for the operator

  ```shell
  kubectl get pods -n ${OPERATOR_NAMESPACE}
  ```

  - Deploy Kubeflow with the operator

  ```shell
  cd $HOME/kfdef
  rm -rf .cache kustomize
  # wget https://raw.githubusercontent.com/kubeflow/manifests/master/kfdef/kfctl_k8s_istio.yaml
  sed -i '' '/metadata:/a\'$'\n\  ''name: kubeflow\'$'\n' kfctl_k8s_istio.yaml

  KUBEFLOW_NAMESPACE=kubeflow
  kubectl create ns ${KUBEFLOW_NAMESPACE}
  kubectl create -f kfctl_k8s_istio.yaml -n ${KUBEFLOW_NAMESPACE}
  ```

  - Watch the progress

  ```shell
  kubectl logs deployment/kubeflow-operator -n ${OPERATOR_NAMESPACE} -f
  ```

* Install `tekton pipelines`

  - Deploy `tekton pipelines`

  ```shell
  kubectl apply -f https://storage.googleapis.com/tekton-releases/pipeline/previous/v0.11.3/release.yaml
  kubectl patch cm feature-flags -n tekton-pipelines \
    -p '{"data":{"disable-home-env-overwrite":"true","disable-working-directory-overwrite":"true"}}'
  ```

  - Install CLI follow the [instructions](https://github.com/tektoncd/cli#installing-tkn)

  - Install `tekton dashboard`

  ```shell
  kubectl apply --filename https://github.com/tektoncd/dashboard/releases/download/v0.6.1/tekton-dashboard-release.yaml
  kubectl patch svc tekton-dashboard -n tekton-pipelines --type='json' -p '[{"op":"replace","path":"/spec/type","value":"NodePort"}]'
  ```

  - Access `tekton dashboard`

  ```shell
  kubectl port-forward -n tekton-pipelines svc/tekton-dashboard 9097:9097&
  ```

  To access, from browser [`http://localhost:9097`](http://localhost:9097).
