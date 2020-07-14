# Install OLM and OLM Console on native Kubernetes clusters

## Install OLM

Instructions are taken from [Operator Hub](https://operatorhub.io)

```shell
curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/0.15.1/install.sh | bash -s 0.15.1
```

## Install OLM Console

```shell
kubectl apply -f [https://](https://raw.githubusercontent.com/adrian555/KubeflowDojo/master/manifests/olm-console.yaml)
```

## Open OLM Console

```shell
export OLM_CONSOLE_PORT=$(kubectl get svc -n olm|grep olm-console|awk '{print $5}'|cut -d':' -f2|cut -d'/' -f1)
export CLUSTER_IP=$(kubectl get node -o wide|grep Ready|awk '{print $7; exit}')

# On MacOS
open http://$CLUSTER_IP:$OLM_CONSOLE_PORT
```

Access OLM Console through `http://$CLUSTER_IP:OLM_CONSOLE_PORT`.
