# Introduction

Blueberry Pi is a platform for hosting dockerized applications on a customized Raspberry Pi running a minimal Linux OS. In addition to the capabilities available on a Raspberry Pi (dual 4k HDMI, ethernet, wifi, USB, etc.), the Blueberry Pi platform also has support for:

- Zwave compatible USB controllers (i.e. Nortek)

### References

| File | Description |
| - | - |
| SETUP.md | Notes and procedures for setting up the OS. |
| DOCKER-COMPOSE-SERVICE.md | Instructions for creating a docker compose service file |

# General Notes on Setup and Design

Here‘s the basic process of developing and deploying an app with docker:

1. Install Docker on the Raspberry Pi

2. Set up a registry at Docker Hub

    Docker hub is the github of docker images. This is where I will deploy images.

3. Initiate a Docker build of the app to create the Docker Image.

    If building on the target directly, then the docker "build" command is sufficient. If developing on a different architecture like a Macbook, then you must build using the buildx cli plugin that now is included in the Docker install by default. Confirm the architecture of the target before running the build.

    ```
    docker buildx build --platform=linux/arm/v7 -t doodles67/<image name> --push .
    ```

    Docker Desktop must be open even though the command is run in a terminal?

    This will push the image to Docker Hub where it can be pulled onto the Raspberry Pi.

4. Deploy the image to the Raspberry Pi

    ```
    sudo docker pull doodles67/<image name>
    ```

    Verify image is loaded

    ```
    sudo docker images
    ```

5. To manually run the service, enter the following command:

    ```
    sudo docker run -d -p 8080:8081 -t doodles67/<image name>
    ```