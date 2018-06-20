# Hello Helm

Refs:

- https://docs.helm.sh/using_helm/#quickstart-guide
- https://docs.helm.sh/using_helm/#installing-helm
- https://docs.helm.sh/using_helm/#role-based-access-control

Get the Helm binary:

```sh
mkdir -p ~/bin
curl https://storage.googleapis.com/kubernetes-helm/helm-v2.9.1-linux-amd64.tar.gz | tar xvz
mv linux-amd64/helm ~/bin
```

Configure RBAC for Tiller (using cluster-admin role):

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```

```sh
kubectl create -f rbac-config.yaml
```

Install Helm on the cluster:

```sh
helm init --service-account tiller
helm update
```

Install an example chart (https://github.com/kubernetes/charts/tree/master/stable/wordpress):

```sh
helm install --name my-wordpress stable/wordpress
```

Connect to WordPress:

```sh
kubectl get svc --namespace default -w my-wordpress-wordpress
export SERVICE_IP=$(kubectl get svc --namespace default my-wordpress-wordpress -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo http://$SERVICE_IP/admin
echo Username: user
echo Password: $(kubectl get secret --namespace default my-wordpress-wordpress -o jsonpath="{.data.wordpress-password}" | base64 --decode)
```

Some Helming:

```sh
helm list
helm status my-wordpress
```

Cleanup:

```sh
helm delete my-wordpress
```
