# Deploy Kubernetes Applications on Google Cloud

Objectives:
- Create a Docker image and store the Dockerfile.
- Test the created Doocker image
- Push the Docker image into the Artifact Registry
- Use the image to crate and expose a deployment in Kubernetes

## Create a Docker image and store the Dockerfile

### Install the marking scripts you will use to help check your progress
```
source <(gsutil cat gs://cloud-training/gsp318/marking/setup_marking_v2.sh)
```

### Clone the `valikyrie-app` source code repository
```
gcloud source repos clone valkyrie-app
```

### Create a `valkyrie-app/Dockfile` with the configuration below:
```
cat > Dockerfile <<EOF
FROM golang:1.10
WORKDIR /go/src/app
COPY source .
RUN go install -v
ENTRYPOINT ["app","-single=true","-port=8080"]
EOF
```

