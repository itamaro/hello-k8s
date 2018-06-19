# Hello K8s Deployment

Deployment overview: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

Creating a new deployment with `kubectl`:

```sh
kubectl run --restart=Always --image=gcr.io/kuar-demo/kuard-amd64:1 kuard
```

I prefer using YAML's, but too lazy to author my own (who isn't?),
so let's use `kubectl` to generate a YAML:

```sh
kubectl run --restart=Always --image=gcr.io/kuar-demo/kuard-amd64:1 --port 8080 kuard -o yaml --dry-run
```

Now I can take the output and copy it to `my-deployment.yaml`, changing things, deleting unnecessary parts:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    foo: bar
  name: kuard
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
      - image: gcr.io/kuar-demo/kuard-amd64:1
        name: kuard
        ports:
        - containerPort: 8080
          name: http
        livenessProbe:
          httpGet:
            path: /healthy
            port: http
        readinessProbe:
          httpGet:
            path: /ready
            port: http
        resources:
          limits:
            cpu: 500m
            memory: 1.5G
```

Now create the deployment using `kubectl`:

```sh
kubectl apply -f my-deployment.yaml --record
kubectl get pods -w
kubectl describe deployment kuard
kubectl get deployment kuard -o yaml
```

Experiment with terminating the node the pod landed on and see what happens:

```sh
export POD_NAME="$( kubectl get pods --selector app=hello -o jsonpath='{.items[0].metadata.name}' )"
export POD_NODE="$( kubectl get pod $POD_NAME -o jsonpath='{.spec.nodeName}' )"
gcloud compute instances delete $POD_NODE
kubectl get pods -w -o wide
```

Notice how as the pod terminates on the deleted node, the ReplicaSet (created by the Deployment) immediately starts a new pod on another node.

Now let's rollout a new image version:

```sh
kubectl set image deployment/kuard kuard=gcr.io/kuar-demo/kuard-amd64:2
kubectl get pods -w
kubectl rollout status deployment/kuard
kubectl rollout history deployment/kuard
```

To undo the last rollout (AKA rollback):

```sh
kubectl rollout undo deployment/kuard
kubectl get pods -w
```

To scale the deployment to 3 replicas:

```sh
kubectl scale deployment/kuard --replicas 3
kubectl get pods -o wide -w
```

Scaling does not trigger a "rollout", so running another `undo` at this point will leave us with 3 replicas of version 2 (because we're undoing the undo).

## Fun stuff

Since the deployment has resource limits (1.5G RAM), we can make it allocate 1.5+epsilon and see what happens.

Since the deployment has readiness & liveness probes, we can make these fail for a while and see what happens.

Best used along with `kubectl get pods -w` and the occasional `kubectl describe pod <..>`.
