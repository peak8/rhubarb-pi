# Introduction

This document just has some notes and useful tips on creating and deploying docker images to the Rhubarb Pi.

&nbsp;

# Creating Docker files

All deployed containers should be based on a reference image from the [Docker Hub image repository](https://hub.docker.com/search?image_filter=official&q=).

Reference images I have used:

- [node:20.10-buster-slim](https://hub.docker.com/layers/library/node/20.10-buster-slim/images/sha256-8f4d2a2533fe5319e819f7af78e27563e2c1307b7550c549ad7dfbf310fe3a1b?context=explore)
- [python:3.11-slim-buster](https://hub.docker.com/layers/library/python/3.11-slim-buster/images/sha256-b94af75d4ff65c50bf1b2119bca6d0ba707037bacd0cb75314801a6953c03241?context=explore)

Each application should test reference images and document results, starting with the smallest image.

Each app should expose port 6769 at a minimum, even if only to provide a welcome message or info message. A docker-compose file will map system ports to 6769.

&nbsp;

# Building Docker Images

My development for the Rhubarb Pi is done predominantly on a Macbook but the target is a Raspberry Pi. This requires the buildx cli plugin that now is included in the Docker install by default. 

Confirm the architecture of the target before running the build. Any one of the following commands can be used to determine this.

```
uname -m
cat /proc/cpuinfo
lscpu
```

The Raspberry Pi (4) should report armv7l.

Build the image with the following command. Docker Desktop might need to be open and I think I had to create a build kit container to run this but I didn't record how. ToDo: work out if Docker desktop has to be open and a build kit created for buildx.

```
docker buildx build --no-cache --platform=linux/arm/v7 -t <dockerhub namespace>/<image name>:<version> --push -f <dockerfile name> .
```

This command with the --push option will push the image to Docker Hub. Omitting <version> will default to "latest".

**NOTE** I ran into issues with the docker cache trying to build. Sometimes app changes would be omitted or if I tried to build two different images within the same terminal window then both images would be identical. Adding --no-cache forces a complete build that adds time but ensures everything is up to date.

&nbsp;

# Deploy the image to the Raspberry Pi

If the image is already running on the rPI then do the following steps first:

```
sudo docker ps -a
sudo docker rm -f <container name>
sudo docker images
sudo docker image rm -f <image id>
```

Now pull the desired image to the device:

```
sudo docker pull <dockerhub namespace>/<image name>:<version>
```

Verify image is loaded

```
sudo docker images
```

&nbsp;

# Running the Image on the Raspberry Pi

To manually run the image, enter the following command:

```
sudo docker run -d -p <host:container port forward> --name <name> -t <dockerhub namespace>/<image name>:<version>
```

This creates a container that can be stopped and restarted. To see console messages, omit the -d flag. Omitting the --name flag will result in a random name being assigned.

See "Accessing tty Devices" for run command that exposes a serial USB device within the container.

To automatically run the image, add it to the system docker-compose.yml file. See DOCKER-COMPOSE-SERVICE.md.

&nbsp;

# Inspect the Container File System

```
sudo docker exec -t -i <container name> /bin/bash
```

&nbsp;

# Update the Image (Manually)

The docker-compose service is set up to use the latest image but won't pull the latest unless forced to do so.

First, stop and remove all containers built off of the old image.

```
sudo docker ps
sudo docker stop <container id or name>
sudo docker rm <container id or name>
```

Then update the app version in the docker-compose.yaml. 

Next, pull the latest image.

```
sudo docker image rm -f <image id>
sudo docker pull <dockerhub namespace>/<image name>:<version>
```

The -f flag is only required if you don't remove any containers created from the image.

Reboot the Raspberry Pi and a new container should be started on the new image.

&nbsp;

# Update the Image (automated)

ToDo: figure out how to push updates to deployed devices.

&nbsp;

# Accessing tty Devices

For an app that requires access to a serial tty device, the --device option must be added to the run command as follows:

```
sudo docker run -p <host port:container port> --name <name> --device /dev/ttyUSBx -t <dockerhub namespace>/<image name>:<version>
```

For the moment I am hard coding the port number in the app expecting that a serial device plugged into it will match.

See DOCKER-COMPOSE-SERVICE.md for adding to the docker-compose file.

ToDo: add flexibility in both the app and the way port numbers are mapped to the docker container so that the port number is not hard coded. Will likely require the procedure below:

&nbsp;

## A Procedure to have the Rhubarb Pi map a port to a container

The following procedure is a safe way to give access to ttyUSBx devices from within the Docker container without granting --priveleged access, which creates a security vulnerability.

This procedure was successful in allowing the container to see the serial port but the mapping doesn't occur until after the app makes its first attempt to talk to the serial device. The first attempt fails to find the port and destroys the device driver. When it tries again a minute later, it fails to open the port due to permissions. This issue has not been resolved. Also, I am still hard coding the port number so it is incomplete. 

First find the cgroup properties of the device.

```
ls -l /dev/ | grep ttyUSB
crw-rw---- 1 root dialout 188, 0 Feb  2 05:29 /dev/ttyUSB0
```

In the example above, a Zwave controller hub is plugged in to a USB port and looks like a serial USB device represented by the general dialout group 188. This number is used to set a cgroup rule in the Docker Run command as shown below:

```
sudo docker run -d -p <host:container port forward> --device-cgroup-rule='c 188:* rmw' -t doodles67/<image name>:<version>
```

Next, create a custom rule for a script that will run every time a USB device is plugged in or removed. In a GNU/Linux system, while devices low level support is handled at the kernel level, the management of events related to them is managed in userspace by udev, and more precisely by the udevd daemon. Custom rules are stored in /etc/udev/rules.d/

```
sudo nano /etc/udev/rules.d/99-docker-tty.rules
```

```
ACTION=="add", SUBSYSTEM=="tty", RUN+="/usr/local/bin/docker_tty.sh 'added' '%E{DEVNAME}' '%M' '%m'"
ACTION=="remove", SUBSYSTEM=="tty", RUN+="/usr/local/bin/docker_tty.sh 'removed' '%E{DEVNAME}' '%M' '%m'"
```

"%M" returns the major cgroup (188 in the case of USB to Serial devices). "%m" seems to return a port number. 

Apply the rules:

```
sudo udevadm control --reload
```

Now create the script to respond to plug events. This works to add or remove a serial device on plug events but doesn't do anything when the device is already plugged in. And even when the device is added to the container, the app still reports the serial port being closed. ToDo: fix this.

```
sudo nano /usr/local/bin/docker_tty.sh
```

```
#!/usr/bin/env bash  
                                                           
echo "Usb event: $1 $2 $3 $4" >> /tmp/docker_tty.log        
if [ ! -z "$(docker ps -qf name=<container name>)" ]                                     
then                                                                            
if [ "$1" == "added" ]                                                          
    then                                                                        
        docker exec -u 0 <container name> mknod $2 c $3 $4                               
        docker exec -u 0 <container name> chmod -R 777 $2                                
        echo "Adding $2 to docker" >> /tmp/docker_tty.log                
    else                                                                        
        docker exec -u 0 <container_name> rm $2                                          
        echo "Removing $2 from docker" >> /tmp/docker_tty.log            
    fi                                                                          
fi
```

And make it executable:

```
sudo chmod +x /usr/local/bin/docker_tty.sh
```

This script creates or deletes the tty device in the running docker container. To inspect if the device can be seen within the container, run the following command:

```
sudo docker exec -it <container name> bash
```

This will open a shell into the container where you can then list the contents of the /dev/ folder.