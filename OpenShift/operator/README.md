## Deploy Kubeflow with Kubeflow Operator

Follow instructions on this [link](https://github.com/operator-framework/community-operators/blob/master/docs/testing-operators.md).

### Prereq:

* make sure you are in python3 env with pip

* install `operator-courier`
  
  ```shell
  pip install operator-courier --upgrade
  ```

* have an account and access to quay.io

  To use quay.io you will need to create free account in https://quay.io/repository/ and login into it.

### Register the operator

1. Preparation

  ```shell
  # into your working directory
  cd $WORKBENCH

  mkdir test-kubeflow-operator
  cd test-kubeflow-operator

  # clone several operator-framework repos
  git clone https://github.com/operator-framework/operator-marketplace.git
  git clone https://github.com/operator-framework/operator-courier.git
  git clone https://github.com/operator-framework/operator-lifecycle-manager.git

  # copy over the kubeflow olm-catalog directory
  git clone https://github.com/IBM/KubeflowDojo.git
  cp -r KubeflowDojo/OpenShift/operator/kubeflow .

  # retrieve your quay.io access token
  ./operator-courier/scripts/get-quay-token
  export QUAY_TOKEN="basic yyyyy"
  ```

2. Verification

```shell
# verify operator package following the OLM standard
operator-courier verify --ui_validate_io kubeflow
```

3. Registration

Replace `<quay_username>` with your quay.io account id.

```shell
# register the operator to OLM by pushing to quay.io
export OPERATOR_DIR=kubeflow
export QUAY_NAMESPACE=<quay_username>
export PACKAGE_NAME=kubeflow
export PACKAGE_VERSION=1.0.0
export TOKEN=$QUAY_TOKEN
operator-courier push "$OPERATOR_DIR" "$QUAY_NAMESPACE" "$PACKAGE_NAME" "$PACKAGE_VERSION" "$TOKEN"
```

You should now be able to see the `kubeflow` in your quay.io account's `Applications` tab. If it is private, make it public.

### Create the OperatorSource

```shell
cat <<EOF > os.yaml
apiVersion: operators.coreos.com/v1
kind: OperatorSource
metadata:
  name: kubeflow-operators
  namespace: openshift-marketplace
spec:
  type: appregistry
  endpoint: https://quay.io/cnr
  registryNamespace: <quay_username>
  displayName: "Kubeflow Operators"
  publisher: "Kubeflow"
EOF
oc apply -f os.yaml
```

### Deploy the Kubeflow

From your OCP web console, you should be able to find it from the `Operators | OperatorHub` tab. Follow the [instruction](https://github.com/operator-framework/community-operators/blob/master/docs/testing-operators.md#testing-operator-deployment-on-openshift) to install and create a deployment.

While creating the `Kubeflow` instance, you can choose the built-in CR example to deploy Kubeflow with Kubeflow Pipelines. On the other hand, if you want to deploy Kubeflow with Tekton Pipelines, use this [manifest](../manifests/kfctl_tekton_openshift_minimal.v1.1.0.yaml) as the KfDef CR.

**IMPORTANT* make sure to create kubeflow instance in `kubeflow` namespace.

To access dashboad look for istio ingress route:

```shell
oc -n istio-system get route
NAME                   HOST/PORT                                                                                                         PATH   SERVICES               PORT    TERMINATION   WILDCARD
istio-ingressgateway   istio-ingressgateway-istio-system.o86-111a9c29...-0000.us-south.containers.appdomain.cloud          istio-ingressgateway   http2                 None
```

And open http://istio-ingressgateway-istio-system.o86-111a9c29...-0000.us-south.containers.appdomain.cloud.

### Cleanup

You can also delete the deployment and the operator through the web console.

### Misc

If you have modified the operator code and rebuild the image, you should update the image in this [line](https://github.com/adrian555/community-operators/blob/openshift/community-operators/kubeflow/1.0.0/kubeflow.v1.0.0.clusterserviceversion.yaml#L562).
