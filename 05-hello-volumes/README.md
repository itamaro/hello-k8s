# Hello K8s Volumes

Volumes overview: https://kubernetes.io/docs/concepts/storage/volumes/

This demonstrates using volumes in a pod to mount a persistent disk (using PVC assuming volume provisioner), and to project secrets / config maps / downward API into the container filesystem.

Create a Secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
data:
  # echo -n "don't tell anyone..." | base64 -w 0
  username: ZG9uJ3QgdGVsbCBhbnlvbmUuLi4=
```

Create a ConfigMap:

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: myconfigmap
data:
  config: "Hello config world!"
```

Create a PVC:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: standard
```

And finally, create a Pod that uses all of the above:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard
  labels:
    foo: bar
    key: val
spec:
  restartPolicy: Always
  containers:
  - name: kuard
    image: gcr.io/kuar-demo/kuard-amd64:1
    imagePullPolicy: IfNotPresent
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
    volumeMounts:
    - name: cloud-pv
      mountPath: "/my-cloud-disk"
    - name: all-in-one
      mountPath: "/projected-volume"
      readOnly: true
  volumes:
  - name: cloud-pv
    persistentVolumeClaim:
      claimName: mypvc
  - name: all-in-one
    projected:
      sources:
      - secret:
          name: mysecret
          items:
            - key: username
              path: my-group/my-username
      - downwardAPI:
          items:
            - path: "labels"
              fieldRef:
                fieldPath: metadata.labels
            - path: "cpu_limit"
              resourceFieldRef:
                containerName: kuard
                resource: limits.cpu
      - configMap:
          name: myconfigmap
          items:
            - key: config
              path: my-group/my-config
```

Create all:

```sh
kubectl apply -f pod-with-volumes.yaml
```

Port-forward between localhost and the pod:

```sh
kubectl port-forward kuard 8080:8080
```

And browse the filesystem: http://localhost:8080/fs/

Cleanup:

```sh
kubectl delete -f pod-with-volumes.yaml
```
