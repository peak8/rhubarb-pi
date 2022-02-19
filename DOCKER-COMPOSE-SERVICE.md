# Introduction

The docker-compose-service will run as a system service on reboot. All app containers that run on the Blueberry Pi, will be lauched from this service.

## Container Networks

The docker-compose file sets up networks that containers use to communicate with one another. These networks are set up with static addresses. Address ranges to be use by private networks are:

```
Class A: 10.0.0.0 to 10.255.255.255
Class B: 172.16.0.0 to 172.31.255.255
Class C: 192.168.0.0 to 192.168.255.255
```

I have chosen 172.16.10.0/24, 172.16.20.0/24, and 172.16.30.0/24 for the first app containers deployed on the Blueberry Pi. Future app containers will continue to increment by 10.

The Blueberry Pi host system maps ports to apps as follows:

```
6701 - Kiosk app
6702 - Test Node app
6703 - Zwave app

Each app exposes port 6769 at a minimum. This port is arbitrarily chosen. It is unassigned with ICANN and it is the years of release for my two favorite Beatles albums, Abbey Road and Sgt. Peppers.

The host system

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
        kiosk-app:
            image: doodles67/kiosk-app:<version>
            container_name: kiosk-app
            restart: always
            networks: 
                blueberrypi:
                    ipv4_address: 172.16.10.2
            ports:
                - 6701:6769
        test-node-app:
            image: doodles67/docker-node-app-rpi:<version>
            container_name: test-node-app
            restart: always
            networks: 
                blueberrypi:
                    ipv4_address: 172.16.10.3
                testnode:
                    ipv4_address: 172.16.20.3
            ports:
                - 6702:6769
        zwave-app:
            image: doodles67/zwave-app-rpi:<version>
            container_name: zwave-app
            restart: always
            devices: 
                - /dev/ttyUSB0:/dev/ttyUSB0
            networks:
                blueberrypi:
                    ipv4_address: 172.16.10.4
                zwave:
                    ipv4_address: 172.16.30.4
            ports:
                - 6703:6769
    
    networks:
        blueberrypi:
            driver: bridge
            enable_ipv6: false
            ipam:
                driver: default
                config:
                    - subnet: 172.16.10.0/24
                      gateway: 172.16.10.1
        testnode:
            driver: bridge
            enable_ipv6: false
            ipam:
                driver: default
                config:
                    - subnet: 172.16.20.0/24
                      gateway: 172.16.20.1
        zwave:
            driver: bridge
            enable_ipv6: false
            ipam:
                driver: default
                config:
                    - subnet: 172.16.30.0/24
                      gateway: 172.16.30.1
    ```

    The blueberrypi network is for the main server application (pointed to by the Chromium Kiosk) and allows it to reach other apps for data to be rendered on the Kiosk screen. The other defined networks (zwave, etc.) are sub-networks dedicated to each app allowing private databases, microservices, etc. for each.

3. Enable the docker-compose-app service

    ```
    sudo systemctl enable docker-compose-app
    ```

    Now, every time the Blueberry Pi reboots the containers are torn down and recreated.

# Miscellaneous Comments

To test network setup you can make a call into a container apps api from a terminal as follows (zwave example):

```
curl 172.16.10.3:3001/admin/status001/admin/status
```

To inspect properties of a network:

```
sudo docker network inspect docker_blueberrypi
```

