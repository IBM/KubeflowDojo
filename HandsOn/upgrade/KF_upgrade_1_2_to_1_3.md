# Kubeflow upgrade 1.2 to 1.3 (multi-user, AppID, IKS, K8s v1.19)
_Raw Technical Writeup_

*Credits to Tommy Li and Mofi Rahman!*

Prerequisites:
- bash
- kubectl, logged in to cluster
- old KF 1.2 installation
- [KF 1.3 kfdef](https://github.com/kubeflow/manifests/blob/v1.3-branch/distributions/kfdef/kfctl_ibm_multi_user.v1.3.0.yaml)
- kfctl 1.2.0
- [kill-kube-ns](https://github.com/ctron/kill-kube-ns/blob/master/kill-kube-ns)

---

```bash
ORIGINAL_MYSQL_VOL=<original mysql volume name>
ORIGINAL_MINIO_VOL=<original minio volume name>
STORAGE_CLASSNAME=<e.g. ibmc-block-gold>
alias k=kubectl
```

## 1. Change retainPolicy of ORIGINAL_MYSQL_VOL and ORIGINAL_MINIO_VOL to Retain
We want to keep the existing volumes containing KFP data.
```bash
k patch pv ${ORIGINAL_MYSQL_VOL} -p '[
    {
        "op": "replace",
        "path": "/spec/persistentVolumeReclaimPolicy",
        "value": "Retain"
    }
]'
k patch pv ${ORIGINAL_MINIO_VOL} -p '[
    {
        "op": "replace",
        "path": "/spec/persistentVolumeReclaimPolicy",
        "value": "Retain"
    }
]'
```

## 2. Delete old KF 1.2 installation
```bash
cd ..
git clone git@github.com:kubeflow/manifests.git
cd manifests/
git checkout v1.2-branch
cd ..
mkdir kf-upgrade
cd kf-upgrade/
cat "
apiVersion: kfdef.apps.kubeflow.org/v1
kind: KfDef
metadata:
  annotations:
    kfctl.kubeflow.io/force-delete: "false"
  clusterName: <cluster name here>
  creationTimestamp: null
  name: kfp
  namespace: kubeflow
spec:
  applications:
  - kustomizeConfig:
      repoRef:
        name: manifests
        path: stacks/ibm/application/istio-1-3-1-stack
    name: istio-stack
  - kustomizeConfig:
      repoRef:
        name: manifests
        path: stacks/ibm/application/cluster-local-gateway-1-3-1
    name: cluster-local-gateway
  - kustomizeConfig:
      repoRef:
        name: manifests
        path: istio/istio/base
    name: istio
  - kustomizeConfig:
      repoRef:
        name: manifests
        path: application/v3
    name: application
  - kustomizeConfig:
      repoRef:
        name: manifests
        path: stacks/ibm/application/bootstrap
    name: bootstrap
  - kustomizeConfig:
      repoRef:
        name: manifests
        path: stacks/ibm/application/cert-manager-crds
    name: cert-manager-crds
  - kustomizeConfig:
      repoRef:
        name: manifests
        path: stacks/ibm/application/cert-manager-kube-system-resources
    name: cert-manager-kube-system-resources
  - kustomizeConfig:
      repoRef:
        name: manifests
        path: stacks/ibm/application/cert-manager
    name: cert-manager
  - kustomizeConfig:
      repoRef:
        name: manifests
        path: stacks/ibm/application/oidc-authservice-appid
    name: oidc-authservice
  - kustomizeConfig:
      repoRef:
        name: manifests
        path: metacontroller/base
    name: metacontroller
  - kustomizeConfig:
      repoRef:
        name: manifests
        path: tektoncd/tektoncd-install/base
    name: tektoncd-install
  - kustomizeConfig:
      repoRef:
        name: manifests
        path: tektoncd/tektoncd-dashboard/base
    name: tektoncd-dashboard
  - kustomizeConfig:
      repoRef:
        name: manifests
        path: stacks/ibm/application/admission-webhook
    name: kubeflow-apps
  - kustomizeConfig:
      repoRef:
        name: manifests
        path: stacks/ibm/application/katib
    name: katib
  - kustomizeConfig:
      repoRef:
        name: manifests
        path: stacks/ibm/application/notebooks
    name: notebooks
  - kustomizeConfig:
      repoRef:
        name: manifests
        path: stacks/ibm/application/pytorch-job
    name: pytorch-job
  - kustomizeConfig:
      repoRef:
        name: manifests
        path: stacks/ibm/application/tf-job
    name: tf-job
  - kustomizeConfig:
      repoRef:
        name: manifests
        path: knative/installs/generic
    name: knative
  - kustomizeConfig:
      repoRef:
        name: manifests
        path: kfserving/installs/generic
    name: kfserving
  - kustomizeConfig:
      repoRef:
        name: manifests
        path: stacks/ibm/application/spartakus
    name: spartakus
  repos:
  - name: manifests
    uri: https://github.com/kubeflow/manifests/archive/v1.2.0.tar.gz
  version: v1.2-branch
status:
  reposCache:
  - localPath: '".cache/manifests/manifests-1.2.0"'
    name: manifests
" > kfctl.yaml
# WARN: kfctl removes the entire kubeflow namespace in the end
kfctl delete -V -f kfctl.yaml 
chmod +x ../kf-upgrade/kill-kube-ns 
../kf-upgrade/kill-kube-ns istio-system
kfctl delete -V -f kfctl.yaml 
```

## 3. Install KF 1.3
```bash
cd ..
mkdir kfdir
cd kfdir/
kfctl apply -V -f kfctl_ibm_multi_user.v1.3.0.yaml 
```

## 4. Re-bind original volumes
```bash
cd ../kf-upgrade/
# Reset PV claimref
k patch pv ${ORIGINAL_MYSQL_VOL} -p '{"spec":{"claimRef": null}}'
k patch pv ${ORIGINAL_MINIO_VOL} -p '{"spec":{"claimRef": null}}'
# Fill PV storage class in metadata.annotations
k patch pv ${ORIGINAL_MYSQL_VOL} -p "[
    {
        \"op\": \"replace\",
        \"path\": \"/metadata/annotations/volume.beta.kubernetes.io~1storage-class\",
        \"value\": \"${STORAGE_CLASSNAME}\"
    }
]"
k patch pv ${ORIGINAL_MINIO_VOL} -p "[
    {
        \"op\": \"replace\",
        \"path\": \"/metadata/annotations/volume.beta.kubernetes.io~1storage-class\",
        \"value\": \"${STORAGE_CLASSNAME}\"
    }
]"
# Re-apply PVC for KFP mysql storage
k -n kubeflow delete pvc mysql-pv-claim
k apply -f - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    ibm.io/provisioning-status: complete
    volume.beta.kubernetes.io/storage-provisioner: <storage type, e.g.: ibm.io/ibmc-block>
  finalizers:
    - kubernetes.io/pvc-protection
  labels:
    application-crd-id: kubeflow-pipelines
    region: <region here>
    zone: <zone here>
  name: mysql-pv-claim
  namespace: kubeflow
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  storageClassName: ${STORAGE_CLASSNAME}
  volumeMode: Filesystem
  volumeName: <original mysql vol name>
EOF
k -n kubeflow delete pvc minio-pvc
# Re-apply minio PVC for KFP artifacts
k -n kubeflow delete pvc minio-pvc
k apply -f - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    ibm.io/provisioning-status: complete
    volume.beta.kubernetes.io/storage-provisioner: <storage type, e.g.: ibm.io/ibmc-block>
  finalizers:
    - kubernetes.io/pvc-protection
  labels:
    application-crd-id: kubeflow-pipelines
    region: <region here>
    zone: <zone here>
  name: minio-pvc
  namespace: kubeflow
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  storageClassName: ${STORAGE_CLASSNAME}
  volumeMode: Filesystem
  volumeName: <original minio vol name>
EOF
kubectl rollout restart deploy mysql -n kubeflow
kubectl rollout restart deploy minio -n kubeflow
kubectl rollout restart deploy ml-pipeline -n kubeflow
```

## 5. Add missing Namespace column in MySQL DB
As far as I can tell, this is the only schema change in KF 1.3 compared to 1.2
```bash
k -n kubeflow exec -it deploy/mysql -- mysql <<EOF
USE mlpipeline;
LOCK TABLES `pipelines` WRITE;
ALTER TABLE pipelines ADD COLUMN Namespace VARCHAR(63) DEFAULT '';
UNLOCK TABLES;
EOF
kubectl rollout restart deploy mysql -n kubeflow
```

## 6. Follow instructions for AppID config
Based on https://www.kubeflow.org/docs/distributions/ibm/deploy/install-kubeflow-on-iks/#multi-user-auth-enabled.
```bash
export OIDC_PROVIDER=https://<region name>.appid.cloud.ibm.com/oauth/v4/...
export OIDC_AUTH_URL=https://<region name>.appid.cloud.ibm.com/oauth/v4/.../authorization
# replace below with your own cluster URL
export REDIRECT_URL=https://<cluster name>.<region, e.g. us-south>.containers.appdomain.cloud/login/oidc
export PATCH=$(printf '{"data": {"OIDC_AUTH_URL": "%s", "OIDC_PROVIDER": "%s", "REDIRECT_URL": "%s"}}' $OIDC_AUTH_URL $OIDC_PROVIDER $REDIRECT_URL)
# didn't work: kubectl -n istio-system patch cm oidc-authservice-parameters -p=${PATCH}
# manually modified cm:
k -n istio-system edit cm oidc-authservice-parameters
# patch secret
kubectl patch secret -n istio-system oidc-authservice-client -p='{"stringData": {"CLIENT_ID": "<AppID client id here>", "CLIENT_SECRET": "<AppID client secret here>"}}'
kubectl rollout restart statefulset authservice -n istio-system
```

## 6. Set up HTTPS
Based on https://www.kubeflow.org/docs/distributions/ibm/deploy/install-kubeflow-on-iks/#multi-user-auth-enabled.
```bash
k -n istio-system edit cm appid-application-configuration
ibmcloud ks nlb-dns ls --cluster <cluster name>
export INGRESS_GATEWAY_SECRET=<secret name>
# remove existing IP
ibmcloud ks nlb-dns rm classic --cluster <cluster name> --ip <old istio IP> --nlb-host <hostname>
kubectl patch svc istio-ingressgateway -n istio-system -p '{"spec":{"type":"LoadBalancer"}}'
# add new IP
export INGRESS_GATEWAY_IP=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
ibmcloud ks nlb-dns add -c <cluster name> --ip ${INGRESS_GATEWAY_IP} --nlb-host <hostname>

kubectl get secret $INGRESS_GATEWAY_SECRET -n istio-system -o yaml > istio-ingressgateway-certs.yaml
kubectl apply -f istio-ingressgateway-certs.yaml -n istio-system
kubectl rollout restart deploy istio-ingressgateway -n istio-system
rm istio-ingressgateway-certs.yaml

k apply -f kubeflow-gateway.yaml 
k -n istio-system edit cm appid-application-configuration

kubectl rollout restart deploy istio-ingressgateway -n istio-system
# force delete and restart authservice
k -n istio-system delete pod authservice-0
kubectl rollout restart statefulset authservice -n istio-system
```
