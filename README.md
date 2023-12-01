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

1. Provision a Raspberry Pi device as detailed in the SETUP.md file. Image name and version details will vary. You have now made Rhubarb Pi :-)

2. Set up a registry at Docker Hub for the dockerized applications to be deployed on the Rhubarb Pi.

3. Develop a core-server application. At a minimum, this application exposes port 6769 to the rhubarbpi network defined in the docker-compose-service file. This port is mapped to system port 6701 allowing for external communication with the app.

4. Optionally, develop a front-end application that exposes port 3000, mapped to system port 6702. In addition to being accessible on port 6702, the front-end application will also be displayed on a screen connected to the HDMI port (if the optional kiosk setup is performed in SETUP.md).

5. Optionally, develop other applications to support the core-server and/or front-end applications as needed. These applications can connect to the rhubarbpi network or custom networks defined in the docker-compose file and will typically expose custom ports that are not mapped to any system port.

6. Create docker files to package each app for deployment. See the DOCKER-REFERENCE.md file for guidance.

7. Build and deploy the application images to Docker Hub and ensure the image name name and version matches what is specified in the docker-compose file. See DOCKER-REFERENCE.md