# Introduction

The docker-compose-service will run as a system service on reboot. All app containers that run on the Blueberry Pi, will be lauched from this service.

# Set Up the Docker-Compose service

1. Create the docker-composer-app.service file.

    ```
    # /etc/systemd/system/docker-compose-app.service

    [Unit]
    Description=Docker Compose Application Service
    Requires=docker.service
    After=docker.service

    [Service]
    Type=oneshot
    RemainAfterExit=yes
    WorkingDirectory=/srv/docker
    ExecStart=/usr/local/bin/docker-compose up -d
    ExecStop=/usr/local/bin/docker-compose down
    TimeoutStartSec=0

    [Install]
    WantedBy=multi-user.target
    ```

2. Create the docker-compose.yaml file.

    Create a docker folder in the /srv folder

    ```
    sudo mkdir /srv/docker
    cd /srv/docker
    ```

    Create the docker-compose.yaml file as follows:

    ```
    services:
        test-node-app:
            image: doodles67/docker-node-app-rpi:<version>
            container_name: test-node-app
            restart: always
            ports:
                - 8080:8081
        zwave-app:
            image: doodles67/zwave-app-rpi:0.1.1
            container_name: zwave-app
            restart: always
            devices: 
                - /dev/ttyUSB0:/dev/ttyUSB0
            ports:
                - 3001:3001
    ```

    The name of the service determines the name of the container. When starting a container from docker-compose.yaml, Docker will prefix the name with docker_ and append _1. The example above would create a container by the name of docker_test-node-app_1. This can be referenced by other scripts. For example, a script that expose a serial port to the container.

3. Enable the docker-compose-app service

    ```
    sudo systemctl enable docker-compose-app
    ```