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

  Note:

  1. if `minikube` version is older than `v1.8.0`, replace `--driver` with `--vm-driver`.
  2. if you have to change the cpu resources for the cluster, make sure you have at least 4 cpus for the cluster. Also on MacOS, make sure that the cpu resource for your Docker Desktop has at least two more cpus. E.g. if the cpu resource in Docker Desktop is 6, then the `--cpus` for the above command should be changed to 4.
  <p>

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

* Download the `kfctl`
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
kfctl apply -V -f https://raw.githubusercontent.com/IBM/KubeflowDojo/master/manifests/kfctl_ibm_tekton.yaml
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

* Delete the current Kubeflow

  ```shell
  kfctl delete -f kfctl_ibm_tekton.yaml

  kubectl delete mutatingwebhookconfigurations --all
  kubectl delete validatingwebhookconfigurations --all
  ```

* Deploy Kubeflow with the operator

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
  wget https://raw.githubusercontent.com/IBM/KubeflowDojo/master/manifests/kfctl_ibm_tekton.yaml
  sed -i '' '/metadata:/a\'$'\n\  ''name: kubeflow\'$'\n' kfctl_ibm_tekton.yaml

  KUBEFLOW_NAMESPACE=kubeflow
  kubectl create ns ${KUBEFLOW_NAMESPACE}
  kubectl create -f kfctl_ibm_tekton.yaml -n ${KUBEFLOW_NAMESPACE}
  ```

  - Watch the progress

  ```shell
  kubectl logs deployment/kubeflow-operator -n ${OPERATOR_NAMESPACE} -f
  ```

* Use `tekton pipelines`

  - Install CLI follow the [instructions](https://github.com/tektoncd/cli#installing-tkn)

  - Access `tekton dashboard`

  ```shell
  kubectl port-forward -n tekton-pipelines svc/tekton-dashboard 9097:9097&
  ```

  To access, from browser [`http://localhost:9097`](http://localhost:9097).

* `kfctl` source code walkthrough with VSC

  - Install `Go` extension
  - Open the folder to `$GOPATH/src/github.com/kubeflow/kfctl`
  - Run/Debug test from the IDE
  - Debug within VSC

## Misc tasks

### Enable LoadBalancer for Istio-ingressgateway

To enable a dedicated LoadBalancer IP for Kubeflow, run:
```shell
kubectl patch svc istio-ingressgateway -n istio-system -p '{"spec": {"type": "LoadBalancer"}}'
```

### Remove namespaces stuck with finalizers

```shell
kubectl proxy&
pid=$!
nss="kubeflow istio-system knative-serving cert-manager"
for ns in $nss; do
  kubectl delete ns $ns --force --grace-period=0
  kubectl get namespace $ns -o json |jq '.spec = {"finalizers":[]}' >temp.json
  curl -k -H "Content-Type: application/json" -X PUT --data-binary @temp.json 127.0.0.1:8001/api/v1/namespaces/$ns/finalize
done
kill -9 $pid
```
