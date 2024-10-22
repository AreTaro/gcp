# Introduction to Docker

## Hello World

### Run a container

To run a container with Docker use the following commmand. Hello world container is used in this example it is a Docker "official image" for [hello-world](https://hub.docker.com/_/hello-world).

```
docker run hello-world
```
With this command the Docker daemon search first for the hello-world image, if it didn't find it locally, pulled the image from a public registry called Docker Hub, create a container from the image and ran the container. If run a second time, as the image is in the local registry, the Docker daomon use it to run the container.

### Take a look at the container

```
docker images
```
## Build a Docker image

### Create a folder
```
mkdir test && cd test
```

### Create a Dockerfile:

```
cat > Dockerfile <<EOF
# Use an official Node runtime as the parent image
FROM node:lts

# Set the working directory in the container to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Make the container's port 80 available to the outside world
EXPOSE 80

# Run app.js using node when the container launches
CMD ["node", "app.js"]
EOF
```
### Create a Node application

A **Node application** is a program built using Node.js, a JavaScript runtime environment. It allows to run JavaScript code outside of a web browser.
*Une application Node est un programme construit avec Node.js, un environnement d'exécution JavaScript.*

```
cat > app.js << EOF;
const http = require("http");

const hostname = "0.0.0.0";
const port = 80;

const server = http.createServer((req, res) => {
	res.statusCode = 200;
	res.setHeader("Content-Type", "text/plain");
	res.end("Hello World\n");
});

server.listen(port, hostname, () => {
	console.log("Server running at http://%s:%s/", hostname, port);
});

process.on("SIGINT", function () {
	console.log("Caught interrupt signal and will exit");
	process.exit();
});
EOF
```
### Build an image

The following command must be run from the file containing the Dockerfile. If the path is different to the Dockerfile replace `.` by `[DOCKERFILE_PATH]`.

```
docker build -t node-app:0.1 .
```

## Run a container

To build a container based on the image build previously, execute the following command:

```
docker run -p 4000:80 --name my-app node-app:0.1
```

Open a new terminal to test the server with the following command:
```
curl http://localhost:4000
```

The container will run as long as the terminal is running. Close the terminal where the container is running. To stop and remove the container use the following command:
```
docker stop my-app && docker rm my-app
```

To run the container in background, use the following command:
```
docker run -p 4000:80 --name my-app -d node-app:0.1
```

To list containers, use `docker ps` command.

## Debug

To look at the logs of a container use the following command:
```
docker logs -f [container_id]
```
`-f` let following the log's outputs while the container is running.

Start an interactive start inside the running container, with the following command:
```
docker exec -it [container_id] bash
```
`-it` flags let you interact with the container and keep stdin open.

To exit the bash session, use the command `exit`.

## Push the image in Google Artifact Registry

The format is `<regional-repository>-docker.pkg.dev/my-project/my-repo/my-image`

An Artifact Registry Repository is required in GCP. Create it with the console or the following command line:
```
gcloud artifacts repositories create my-repository 
    --repository-format=docker \
    --location="REGION" \
    --description="Docker repository"
```

### Set up authentification to Docker repository
Before push or pull images, authenticate requests to Artifact Registry with the following command:
```
gcloud auth configure-docker "REGION" -docker.pkg.dev
```

Run the command to tag `node-app:0.2`
```
docker build -t "REGION" -docker.pkg.dev\"PROJECT_ID"\my-repository\node-app:0.2 .
```

Push the image to Artifact Registry with the following command:
```
docker push "REGION" -docker.pkg.dev\"PROJECT_ID"\my-repository\node-app:0.2
```
After this step, the image should be visible in the Artifact Registry repository in the GCP platform.


### Test the image

#### Stop and remove all containers:
```
docker stop $(docker ps -q) && docker rm $(docker ps -aq)
```
`-q` (or `--quiet`) displays only containers IDs for running containers. `-a` stands for `--all` 

#### Remove all Docker images
```
docker rmi "REGION"-docker.pkg.dev/"PROJECT_ID"/my-repository/node-app:0.2
docker rmi node:lts
docker rmi -f $(docker images -aq) # remove remaining images
docker images
```

#### Pull the image and run it
```
docker run -p 4000:80 -d "REGION"-docker.pkg.dev/"PROJECT_ID"/my-repository/node-app:0.2
```

Test the server with the command `curl http://localhost:4000`
