# Introduction

Rhubarb Pi is a platform for hosting dockerized applications on a customized Raspberry Pi running a minimal Linux OS. In addition to the capabilities available on a Raspberry Pi (dual 4k HDMI, ethernet, wifi, USB, etc.), the Rhubarb Pi platform also has support for:

- Zwave compatible USB controllers (i.e. Nortek)

### References

| File | Description |
| - | - |
| SETUP.md | Notes and procedures for setting up the OS. |
| DOCKER-COMPOSE-SERVICE.md | Instructions for creating a docker compose service file |
| DOCKER-REFERENCE.md | Instructions and notes on building and deploying docker images|
| KIOSK-SETUP.md | Instructions for setting up the Rhubarb Pi Kiosk |

# General Notes on Setup and Design

Here‘s the basic process of developing and deploying an app with docker:

1. Install Docker and Docker-Compose on the Raspberry Pi

    See SETUP.md.

2. Set up a registry at Docker Hub

    Docker hub is the github of docker images. I have account with doodles67 as the user name. 

3. Initiate a Docker build of the app to create the Docker Image.

    See DOCKER-REFERENCE.md

4. Deploy the image to the Raspberry Pi

    Se DOCKER-REFERENCE.md

5. Add the image to the docker-compose service on the Raspberry Pi.

    See DOCKER-COMPOSE-SERVICE.md