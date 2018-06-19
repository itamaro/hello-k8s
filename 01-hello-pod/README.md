# Hello K8s Pod

Pod overview: https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/

Creating a new pod with `kubectl`:

```sh
kubectl run --restart=Never --image=gcr.io/kuar-demo/kuard-amd64:1 kuard
```

I prefer using YAML's, but too lazy to author my own (who isn't?),
so let's use `kubectl` to generate a YAML:

```sh
kubectl run --restart=Never --image=gcr.io/kuar-demo/kuard-amd64:1 --port 8080 kuard -o yaml --dry-run
```

Now I can take the output and copy it to `my-pod.yaml`, changing things, deleting unnecessary parts:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  containers:
  - name: kuard
    image: gcr.io/kuar-demo/kuard-amd64:1
    imagePullPolicy: IfNotPresent
    ports:
    - containerPort: 8080
  restartPolicy: Always
```

Now create the pod using `kubectl`:

```sh
kubectl create -f my-pod.yaml
kubectl get pods -w
kubectl describe pod kuard
kubectl get pod kuard -o yaml
```

Experiment with terminating the node the pod landed on and see what happens:

```sh
export POD_NODE="$( kubectl get pod kuard -o jsonpath='{.spec.nodeName}' )"
gcloud compute instances stop $POD_NODE
kubectl get pods -w
```

Note the pod disappears completely (even though we specified "restart" to be "Always" - why is that?).
