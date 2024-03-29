# Deploy Kubeflow on IBM Cloud

## Access to IBM Cloud workshop cluster

If you already have access to an IBM Cloud Kubernetes cluster, please use it for the following experiment. If not, you will be provided a cluster just for the purpose of this handson.

* Create an `IBMid`

(Optional) If you do not have an `IBMid`, go to [IBM Cloud](https://ibm.biz/Bdqgck) to create an account to access IBM Cloud clusters. For IBM internal developers, you can use your internet id with SSO to access IBM Cloud.

* Install CLI tools to access IBM Kubernetes cluster

```shell
curl -sL https://ibm.biz/idt-installer | bash
```

* Request access to a workshop cluster

(Optional) There are a limited number of workshop clusters for the live dojo session. If you need the access to one of the clusters, open your browser with [`https://ikskubeflow.mybluemix.net`](https://ikskubeflow.mybluemix.net), enter `ikslab` as the lab key and your IBMid email to request a cluster ![request access](../../images/workshop-request.png).

Once the request gets through, follow the `IBM Cloud account` link to view the cluster info ![cluster granted](../../images/workshop-granted.png).

* Access the cluster

```shell
ibmcloud ks cluster config --cluster <cluster_name/id>
```
Replace the `<cluster_name/id>` with the cluster name or id provided to you.

* Login to IBM Cloud

```shell
ibmcloud login -a cloud.ibm.com -r us-south

# if you have a federated id
# ibmcloud login -a cloud.ibm.com -r us-south --sso
```

Note: if you have multiple accounts, choose `1840867 - Advowork`.

## Set up IBM Cloud Block Storage

* Set up the default storage class with Group ID support

```shell
NEW_STORAGE_CLASS=ibmc-file-gold-gid
OLD_STORAGE_CLASS=$(kubectl get sc -o jsonpath='{.items[?(@.metadata.annotations.storageclass\.kubernetes\.io\/is-default-class=="true")].metadata.name}')
kubectl patch storageclass ${NEW_STORAGE_CLASS} -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
kubectl patch storageclass ${OLD_STORAGE_CLASS} -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'
```

<hr>

## Kubeflow deployment using `kustomize` CLI

### Install `kustomize` (version 3.2.0)

`kustomize` can be downloaded from [here](https://github.com/kubernetes-sigs/kustomize/releases/tag/v3.2.0). For MacOS
users, use [darwin_amd64](https://github.com/kubernetes-sigs/kustomize/releases/download/v3.2.0/kustomize_3.2.0_darwin_amd64).
For Linux users, use [linux_amd64](https://github.com/kubernetes-sigs/kustomize/releases/download/v3.2.0/kustomize_3.2.0_linux_amd64)
```shell
wget <URL> -O kustomize
chmod +x kustomize
mv kustomize /usr/local/bin
```

### git clone manifests from IBM/manifests repo

Use the manifests from [IBM/manifests](https://github.com/IBM/manifests) in `v1.5-branch` branch.
git clone the repo and get into the directory:
```shell
git clone https://github.com/IBM/manifests.git -b v1.5-branch
cd manifests
```

### Generate a password for login

In single user deployment, a password is needed for default user: `user@example.com`. Use
[python3](https://www.python.org/downloads/), [passlib](https://pypi.org/project/passlib/),
and [bcrypt](https://pypi.org/project/bcrypt/) to generate a password. Make sure you have these
pypi packages installed in your Python3 environment and use the follow command to generate the password:
```shell
python3 -c 'from passlib.hash import bcrypt; import getpass; print(bcrypt.using(rounds=12, ident="2y").hash(getpass.getpass()))'
```
Type your password and press `<Enter>` after you see `Password:` prompt. Copy the hash code for next step.

### Update env file for the password

Edit `dist/stacks/ibm/application/dex-auth/custom-env.yaml` and fill the relevant field with the hash code from previous step:
```yaml
staticPasswords:
- email: user@example.com
  hash: <enter the generated hash here>
```

If you'd like to change the email address, you have to update the email address in both files:
- `common/user-namespace/base/params.env`
- `dist/stacks/ibm/application/dex-auth/custom-env.yaml`

### Deploy Kubeflow

* Run the deployment with application manifests specified for IBM Cloud

```shell
while ! kustomize build iks-single | kubectl apply -f -; do echo "Retrying to apply resources"; sleep 10; done
```

* Check the deployment

```shell
kubectl get ns
kubectl get pods -n istio-system
kubectl get pods -n kubeflow
```

Wait until all pods and services are up and running in the `kubeflow` namespace than continue to the next steps.

### Access Kubeflow dashboard

  Thereare two approaches to access the dashboard. To access the dashboard with the cluster ip, run following:

  - Retrieve cluster ip

  ```shell
  export CLUSTER_IP=$(kubectl get node -o wide|grep Ready|awk '{print $7; exit}')
  ```

  Now you can access the dashboard through `http://$CLUSTER_IP:30380`.

  Another approach is to use port forwarding as follow:

  - Port forward `istio-ingressgateway`

  ```shell
  kubectl port-forward --namespace istio-system svc/istio-ingressgateway 8080:80 &
  ```

  - Access the dashboard through `localhost:8080`

  You can access the kubeflow central dashboard, created, and run pipelines.

### Delete the Kubeflow deployment

Only do this once you are don't need this Kubeflow deployment.
First of all, delete the user profile/namespace first:
```shell
# clean up user profile/namespace first
kubectl delete profile --all
```
Then wait until the user namespace is removed.
```
# check namespace
kubectl get ns
```

Make sure you are under manifests directory where you git clone the IBM/manifests repo.
Use the follow command to delete kubeflow deployment:
```
kustomize build iks-single | kubectl delete -f -

# manually remove some leftover webhooks
kubectl delete mutatingwebhookconfigurations --all
kubectl delete validatingwebhookconfigurations --all
```

## Use `tekton pipelines`

* Install CLI follow the [instructions](https://github.com/tektoncd/cli#installing-tkn)

```shell
# on MacOS
brew tap tektoncd/tools
brew install tektoncd/tools/tektoncd-cli
```

* Access `tekton dashboard`

```shell
kubectl port-forward -n tekton-pipelines svc/tekton-dashboard 9097:9097&
```

To access, from browser [`http://localhost:9097`](http://localhost:9097).

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

### Install Go tools

* Install `go` following this [link](https://golang.org/dl/)

  After install, do following to set up the environment
  
  ```shell
  mkdir -p $HOME/go/src
  export GOPATH=$HOME/go
  ```

Refer to [Kubeflow on IBM Cloud](https://www.kubeflow.org/docs/ibm/install-kubeflow/) for all details.
