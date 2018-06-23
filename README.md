# Hello Kubernetes

A few basic Kubernetes examples, based on a GKE cluster. Intended for teaching purposes.

Ref: https://cloud.google.com/kubernetes-engine/docs/quickstart

## Create GKE Cluster

Using Google Cloud Shell:

```sh
export PROJECT_ID="<GCE Project ID>"
export ZONE="us-central1-a"
export CLUSTER="my-cluster"
gcloud config set project $PROJECT_ID
gcloud config set compute/zone $ZONE
gcloud container clusters create $CLUSTER --num-nodes=3 --machine-type n1-standard-2 --cluster-version=1.9
# kubectl is configured to talk with the newly created cluster
kubectl get nodes
kubectl get pods --all-namespaces
```

## Cleanup GKE Cluster

```sh
gcloud container clusters delete $CLUSTER
```
