# Hello K8s Service

Service overview: https://kubernetes.io/docs/concepts/services-networking/service/

Exposing (e.g. creating a Service) an existing deployment using `kubectl`:

```sh
kubectl expose deployment kuard
```

I prefer using YAML's, but too lazy to author my own (who isn't?),
so let's use `kubectl` to generate a YAML:

```sh
kubectl expose deployment kuard --type LoadBalancer --port 80 --target-port 8080 --dry-run -o=yaml
```

Now I can take the output and copy it to `my-service.yaml`, changing things, deleting unnecessary parts:

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    woot: omg
  name: kuard
spec:
  type: LoadBalancer
  selector:
    app: hello
  ports:
  - port: 80
    protocol: TCP
    targetPort: http
```

Now create the service using `kubectl`:

```sh
kubectl apply -f my-service.yaml --record
kubectl get svc -w
kubectl describe svc kuard  # note the list of endpoints (scale the deployment to 0, 1, 5 and see this list respond)
kubectl get svc kuard -o yaml
```

Wait for the pending IP to become available (a cloud load balancer is being provisioned)

Obtain just the IP:

```sh
export SVC_IP="$( kubectl get svc kuard -o jsonpath='{.status.loadBalancer.ingress[0].ip}' )"
```

Go ahead and play with the exposed application!
