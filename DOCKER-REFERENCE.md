# Introduction

This documents just has some notes and useful tips on creating and deploying docker images.

## Building Docker Images

My development for the Blueberry Pi is done predominantly on a Macbook but the target is a Raspberry Pi. This requires the buildx cli plugin that now is included in the Docker install by default. 

Confirm the architecture of the target before running the build. Any one of the following commands can be used to determine this.

```
uname -m
cat /proc/cpuinfo
lscpu
```

The Raspberry Pi (4) should report armv7l.

Build the image with the following command. Docker Desktop might need to be open and I think I had to create a build kit container to run this but I didn't record how. ToDo: work out if Docker desktop has to be open and a build kit created for buildx.

```
docker buildx build --platform=linux/arm/v7 -t doodles67/<image name>:<version> --push .
```

This command with the --push option will push the image to Docker Hub. Omitting <version> will default to "latest".

## Deploy the image to the Raspberry Pi

```
sudo docker pull doodles67/<image name>:<version>
```

Verify image is loaded

```
sudo docker images
```

## Running the Image on the Raspberry Pi

To manually run the image, enter the following command:

```
sudo docker run -d -p <host:container port forward> -t doodles67/<image name>:<version>
```

This creates a container that can be stopped and restarted.

To automatically run the image, add it to the system docker-compose.yml file. See DOCKER-COMPOSE-SERVICE.md.

## Update the Image (Manually)

The docker-compose service is set up to use the latest image but won't pull the latest unless forced to do so.

First, stop and remove all containers built off of the old image.

```
sudo docker ps
sudo docker stop <container id or name>
sudo docker rm <container id or name>
```

Next, pull the latest image.

```
sudo docker image rm -f <image id>
sudo docker pull doodles67/<image name>:<version>
```

The -f flag is only required if you don't remove any containers created from the image.

Reboot the Raspberry Pi and a new container should be started on the new image.

## Update the Image (automated)

ToDo: figure out how to push updates to deployed devices.