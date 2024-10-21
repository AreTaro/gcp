# Google Kubernetes Engine: Qwik Start

Google Kubernetes Engine (GKE) custers are powered by [Kubernetes](https://kubernetes.io/) open source cluster management. Kubernetes provides mechanisms through which you interact with your cluster. Cluster management features include:
- Load balancing
- Node pools
- Automatic scalling
- Automatic upgrades
- Node auto-repair
- Logging and Monitoring

## Create a cluster

```bash
gcloud container clusters create --machine-type=e2-medium --zone=[ZONE_NAME] lab-cluster
```
## Authenticate with the cluster
Authentication credentials are needed to interact with the cluster

```bash
gcloud container clusters get-credentials lab-cluster
```
`--zone=[ZONE_NAME]` can be tagged.


## Deploy an application to the cluster

```bash
kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
```
`--image=[IMAGE_LOCATION]/[NAME]:[VERSION]` 

## Expose application to traffic

```bash
kubectl expose deployment hello-server --type=LoadBalancer --port 8080
```

## Inspect deployed service
```bash
kubectl get service
```

## Access to the application
`http://[EXTERNAL-IP]:[PORT]`

## Delete the cluster
```bash
gcloud container clusters delete lab-cluster --zone=[ZONE_NAME] --quiet
```
`--zone=[ZONE_NAME] --quiet` is optional
