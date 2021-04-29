This guide describes how to use the kfctl binary to deploy Kubeflow on IBM Cloud Kubernetes Service (IKS).

## Prerequisites

* Authenticate with IBM Cloud

  Log into IBM Cloud using the [IBM Cloud Dashboard](https://cloud.ibm.com)

* Create and access a OpenShift cluster on [IBM Cloud](https://cloud.ibm.com/kubernetes/clusters?platformType=openshift)

  To deploy Kubeflow on IBM Cloud OpenShift, you need a OpenShift 4.5+ cluster running on IBM Cloud. If you don't have a cluster running you can create one [here](https://cloud.ibm.com/kubernetes/catalog/create?platformType=openshift)

* Once your cluster is ready you can go to `OpenShift web console` from the `Overview` window.  

* In the OpenShift console click on the top right drop down and click on `Copy Login Command` This will give you a command similar to this:

  ```shell
  oc login --token=<some-api-token> --server=<some-server>
  ```
* Paste this command into your terminal and you are ready to go. 

> This guide assumes you have `oc` command line tool already installed. If not install the cli. 


## Understanding the Kubeflow deployment process

The deployment process is controlled by the following commands:

* **build** - (Optional) Creates configuration files defining the various
  resources in your deployment. You only need to run `kfctl build` if you want
  to edit the resources before running `kfctl apply`.
* **apply** - Creates or updates the resources.
* **delete** - Deletes the resources.

### App layout

Your Kubeflow application directory **${KF_DIR}** contains the following files and 
directories:

* **${CONFIG_FILE}** is a YAML file that defines configurations related to your 
  Kubeflow deployment.

  * This file is a copy of the GitHub-based configuration YAML file that
    you used when deploying Kubeflow. For example, {{% config-uri-ibm %}}.
  * When you run `kfctl apply` or `kfctl build`, kfctl creates
    a local version of the configuration file, `${CONFIG_FILE}`,
    which you can further customize if necessary.

* **kustomize** is a directory that contains the kustomize packages for Kubeflow applications.
    * The directory is created when you run `kfctl build` or `kfctl apply`.
    * You can customize the Kubernetes resources (modify the manifests and run `kfctl apply` again).


## Kubeflow installation

**Note**: kfctl is currently available for Linux and macOS users only. If you use Windows, you can install kfctl on Windows Subsystem for Linux (WSL). Refer to the official [instructions](https://docs.microsoft.com/en-us/windows/wsl/install-win10) for setting up WSL.

Run the following commands to set up and deploy Kubeflow:

1. Download the latest kfctl {{% kf-latest-version %}} release from the
  [Kubeflow releases 
  page](https://github.com/kubeflow/kfctl/releases/tag/{{% kf-latest-version %}}).
  
  **Note**: You're strongly recommended to install **kfctl v1.2** or above because kfctl v1.2 addresses several critical bugs that can break the Kubeflow deployment.

2. Extract the archived TAR file:

  ```shell
  tar -xvf kfctl_{{% kf-latest-version %}}_<platform>.tar.gz
  ```
3. Make kfctl binary easier to use (optional). If you don’t add the binary to your path, you must use the full path to the kfctl binary each time you run it.

  ```shell
  export PATH=$PATH:<path to where kfctl was unpacked>
  ```
    
Choose either **single user** or **multi-tenant** section based on your usage.

If you're experiencing issues during the installation because of conflicts on your Kubeflow deployment, you can [uninstall Kubeflow](#uninstall-kubeflow) and install it again.

### Single user

Run the following commands to set up and deploy Kubeflow for a single user without any authentication.

**Note**: By default, Kubeflow deployment on IBM Cloud uses the [Kubeflow pipeline with the Tekton backend](https://github.com/kubeflow/kfp-tekton#kubeflow-pipelines-with-tekton).
If you want to use the Kubeflow pipeline with the Argo backend, modify and uncomment the `argo` and `kfp-argo` applications
inside the `kfctl_ibm.yaml` below and remove the `kfp-tekton`, `tektoncd-install`, and `tektoncd-dashboard` applications. 

```shell
# Set KF_NAME to the name of your Kubeflow deployment. This also becomes the
# name of the directory containing your configuration.
# For example, your deployment name can be 'my-kubeflow' or 'kf-test'.
export KF_NAME=<your choice of name for the Kubeflow deployment>

# Set the path to the base directory where you want to store one or more 
# Kubeflow deployments. For example, /opt/.
# Then set the Kubeflow application directory for this deployment.
export BASE_DIR=<path to a base directory>
export KF_DIR=${BASE_DIR}/${KF_NAME}

# Set the configuration file to use, such as:
export CONFIG_FILE=kfctl_ibm.yaml
export CONFIG_URI="https://raw.githubusercontent.com/kubeflow/manifests/master/kfdef/kfctl_ibm_custom_openshift.yaml"

# Generate Kubeflow:
mkdir -p ${KF_DIR}
cd ${KF_DIR}
curl -L ${CONFIG_URI} > ${CONFIG_FILE}

# Deploy Kubeflow. You can customize the CONFIG_FILE if needed.
kfctl apply -V -f ${CONFIG_FILE}
```

* **${KF_NAME}** - The name of your Kubeflow deployment.
  If you want a custom deployment name, specify that name here.
  For example,  `my-kubeflow` or `kf-test`.
  The value of KF_NAME must consist of lower case alphanumeric characters or
  '-', and must start and end with an alphanumeric character.
  The value of this variable cannot be greater than 25 characters. It must
  contain just a name, not a directory path.
  This value also becomes the name of the directory where your Kubeflow 
  configurations are stored, that is, the Kubeflow application directory. 

* **${KF_DIR}** - The full path to your Kubeflow application directory.

The Kubeflow endpoint is exposed with [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#nodeport) 31380. If you don't have any experience on Kubernetes, you can [expose the Kubeflow endpoint as a LoadBalancer](#expose-the-kubeflow-endpoint-as-loadbalancer) and access the **EXTERNAL_IP**.

### Multi-user, auth-enabled

Run the following steps to deploy Kubeflow with [IBM Cloud AppID](https://cloud.ibm.com/catalog/services/app-id)
as an authentication provider. 

The scenario is a Kubeflow cluster admin configures Kubeflow as a web
application in AppID and manages user authentication with builtin identity
providers (Cloud Directory, SAML, social log-in with Google or Facebook etc.) or
custom providers.

1. Set up environment variables:

    ```shell
    export KF_NAME=<your choice of name for the Kubeflow deployment>

    # Set the path to the base directory where you want to store one or more 
    # Kubeflow deployments. For example, use `/opt/`.
    export BASE_DIR=<path to a base directory>

    # Then set the Kubeflow application directory for this deployment.
    export KF_DIR=${BASE_DIR}/${KF_NAME}
    ```

2. Set up configuration files:

    ```shell
    export CONFIG_FILE=kfctl_ibm_multi_user.yaml
    export CONFIG_URI="https://raw.githubusercontent.com/kubeflow/manifests/master/kfdef/kfctl_ibm_custom_openshift_multiuser.yaml"
    # Generate and deploy Kubeflow:
    mkdir -p ${KF_DIR}
    cd ${KF_DIR}
    curl -L ${CONFIG_URI} > ${CONFIG_FILE}
    ```
    
**Note**: By default, the IBM configuration is using the [Kubeflow pipeline with the Tekton backend](https://github.com/kubeflow/kfp-tekton#kubeflow-pipelines-with-tekton).
    If you want to use the Kubeflow pipeline with the Argo backend, modify and uncomment the `argo` and `kfp-argo-multi-user` applications
    inside the `kfctl_ibm_multi_user.yaml` and remove the `kfp-tekton-multi-user`, `tektoncd-install`, and `tektoncd-dashboard` applications. 
    
3. Deploy Kubeflow:

    ```shell
    kfctl apply -V -f ${CONFIG_FILE}
    ```

4. Wait until the deployment finishes successfully — for example, all pods should be in the `Running` state when you run the command:

    ```shell
    oc  get pod -n kubeflow
    ```

5. Follow the [Creating an App ID service instance on IBM Cloud](https://cloud.ibm.com/catalog/services/app-id) guide for Kubeflow authentication. 
You can also learn [how to use App ID](https://cloud.ibm.com/docs/appid?topic=appid-getting-started) with different authentication methods.

6. Follow the [Registering your app](https://cloud.ibm.com/docs/appid?topic=appid-app#app-register) section of the App ID guide
to create an application with type _regularwebapp_ under the provisioned AppID
instance. Make sure the _scope_ contains _email_. Then retrieve the following
configuration parameters from your AppID:
    * `clientId`
    * `secret`
    * `oAuthServerUrl`
    
7. Register the Kubeflow OIDC redirect page. The Kubeflow OIDC redirect URL is `http://<kubeflow-FQDN>/login/oidc`. 
`<kubeflow-FQDN>` is the endpoint for accessing Kubeflow. By default, the `<kubeflow-FQDN>` on IBM Cloud is `<worker_node_external_ip>:31380`. If you don't have any experience on Kubernetes, you can [expose the Kubeflow endpoint as a LoadBalancer](#expose-the-kubeflow-endpoint-as-loadbalancer) and use the **EXTERNAL_IP** for your `<kubeflow-FQDN>`.

  Then, you need to place the Kubeflow OIDC redirect URL under **Manage Authentication** > **Authentication settings** > **Add web redirect URLs**.
  
8. Create the namespace `istio-system` if it does not exist:

    ```shell
    oc create namespace istio-system
    ```

9. Create a secret prior to Kubeflow deployment by filling parameters from the
step 2 accordingly:

    ```shell
    oc create secret generic appid-application-configuration -n istio-system \
      --from-literal=clientId=<clientId> \
      --from-literal=secret=<secret> \
      --from-literal=oAuthServerUrl=<oAuthServerUrl> \
      --from-literal=oidcRedirectUrl=http://<kubeflow-FQDN>/login/oidc
    ```

    * `<oAuthServerUrl>` - fill in the value of oAuthServerUrl
    * `<clientId>` - fill in the value of clientId
    * `<secret>` - fill in the value of secret
    * `<kubeflow-FQDN>` - fill in the FQDN of Kubeflow, if you don't know yet, just give a dummy one like `localhost`. Then change it after you got one.
    
    **Note**: If any of the parameters changed after the initial Kubeflow deployment, you 
    will need to manually update these parameters in the secret `appid-application-configuration`.
    Then, restart authservice by running the command `oc rollout restart sts authservice -n istio-system`.

### Verify mutli-user installation

Check the pod `authservice-0` is in running state in namespace `istio-system`:

```SHELL
oc get pod authservice-0 -n istio-system
```

## Next steps

To secure the Kubeflow dashboard with HTTPS, follow the steps in [Exposing the Kubeflow dashboard with DNS and TLS termination](../authentication/#exposing-the-kubeflow-dashboard-with-dns-and-tls-termination).
Then, you will have the required DNS name as Kubeflow FQDN to enable the OIDC flow for AppID:


1. Follow the step [Adding redirect URIs](https://cloud.ibm.com/docs/appid?topic=appid-managing-idp#add-redirect-uri)
to fill a URL for AppID to redirect to Kubeflow. The URL should look like `https://<kubeflow-FQDN>/login/oidc`.

2. Update the secret `appid-application-configuration` with the updated Kubeflow FQDN to replace `<kubeflow-FQDN>` in below command:

```SHELL
redirect_url=$(printf https://<kubeflow-FQDN>/login/oidc | base64 -w0) \
 oc patch secret appid-application-configuration -n istio-system \
 -p $(printf '{"data":{"oidcRedirectUrl": "%s"}}' $redirect_url)
```
3. Restart the pod `authservice-0`:

```SHELL
oc rollout restart statefulset authservice -n istio-system
```

Then, visit `https://<kubeflow-FQDN>/`. The page should redirect you to AppID for authentication.

## Additional information

You can find general information about Kubeflow configuration in the guide to [configuring Kubeflow with kfctl and kustomize](/docs/other-guides/kustomize/).

## Troubleshooting

### Expose the Kubeflow endpoint as a LoadBalancer

By default, the Kubeflow deployment on IBM Cloud only exposes the endpoint as [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#nodeport) 31380. If you want to expose the endpoint as a [LoadBalancer](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer), run:

```shell
oc patch svc istio-ingressgateway -n istio-system -p '{"spec": {"type": "LoadBalancer"}}'
```

Then, you can locate the LoadBalancer in the **EXTERNAL_IP** column when you run the following command:

```shell
oc get svc istio-ingressgateway -n istio-system
```

