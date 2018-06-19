# Hello K8s Pod

Job overview: https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/

Example job:

```sh
kubectl create -f my-job.yaml
kubectl get jobs -w
kubectl describe job pi
```

Get pods that belong to the job:

```sh
kubectl get pods --selector=job-name=pi --output=jsonpath={.items..metadata.name}
```

View the result (output) of the job using these pods:

```sh
kubectl logs $( kubectl get pods --selector=job-name=pi --output=jsonpath={.items..metadata.name} )
```
