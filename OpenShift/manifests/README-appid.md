## Multi-tenant Kubeflow on OpenShift with IBM Cloud AppID

This guide is based on [KfServing on OpenShift](./README-kfserving.md) with multi-tenancy feature enabled by IBM Cloud AppID.

### Prerequisites

1. Follow the [Prepare OpenShift cluster environment](./README.md#prepare-openshift-cluster-environment) to set up the cluster environment.
2. FQDN of OpenShift Route of istio ingress gateway.
3. Provisioning an AppID instance from IBM Cloud. It can start with the Lite plan, but will need the Graduated tier once you need more than 1000 authentication events per month.
4. Create an application with type reguarwebapp under the provioned AppID instance. Make sure the caope contains email and retrieve the following configuration parameters from your AppID. They will be used to configure the OIDC auth service:
    * clientId
    * secret
    * oAuthServerUrl

### Configuration

1. Create the namespace `istio-system` if not exist:
```SHELL
kubectl create namespace istio-system
```
2. Create a secret prior to kubeflow deployment by filling parameters accordingly:
```SHELL
kubectl create secret generic appid-application-configuration -n istio-system \
  --from-literal=clientId=<clientId> \
  --from-literal=secret=<secret> \
  --from-literal=oAuthServerUrl=<oAuthServerUrl> \
  --from-literal=oidcRedirectUrl=https://istio-ingressgateway-istio-system.<ingressSubdomain>/login/oidc
```
* `<oAuthServerUrl>` - fill in the value of `oAuthServerUrl`
* `<clientId>` - fill in the value of `clientId`
* `<secret>` - fill in the value of `secret`
* `<ingressSubdomain>` - fill in the value of _Ingress Subdomain_ out of cluster
details by running command `ibmcloud ks cluster get -c <your-cluster-name>` where replace `<your-cluster-name>` with your OpenShift cluster name.

### Deploy Kubeflow with KfServing

Choose [kfctl_openshift_tekton_kfserving_appid.v1.1.0.yaml](./kfctl_openshift_tekton_kfserving_appid.v1.1.0.yaml) to deploy the required components for multi-tenant Kubeflow with Tekton backend.

```shell
export KFDEF_DIR=<path_to_kfdef>
mkdir -p ${KFDEF_DIR}
cd ${KFDEF_DIR}
wget https://raw.githubusercontent.com/IBM/KubeflowDojo/master/OpenShift/manifests/kfctl_openshift_tekton_kfserving_appid.v1.1.0.yaml
```

If you choose to leverage the pre-installed OpenShift Pipelines as the Tekton backend, please comment out these lines from the above configuration file.

```yaml
  - kustomizeConfig:
      repoRef:
        name: manifests
        path: stacks/openshift/components/tektoncd
    name: kubeflow-apps
```

Run following command to deploy Kubeflow:

```shell
kfctl apply -V -f kfctl_openshift_tekton_kfserving_appid.v1.1.0.yaml
```

### Secure istio ingress gateway with HTTPS

Notice that it uses HTTPS for the value of `oidcRedirectUrl` during configuration, which
requires additional steps after deploying Kubeflow:
1. enable [TLS passthrough](https://docs.openshift.com/enterprise/3.0/architecture/core_concepts/routes.html#passthrough-termination) mode for the route.
2. expose kubeflow dashboard over HTTPS by following steps of [this section](https://www.kubeflow.org/docs/ibm/deploy/authentication/#exposing-the-kubeflow-dashboard-with-dns-and-tls-termination).