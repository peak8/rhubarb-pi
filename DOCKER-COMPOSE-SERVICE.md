# Introduction

The docker-compose-service will run as a system service on reboot. All app containers that run on the Rhubarb Pi, will be lauched from this service.

## Container Networks

The docker-compose file sets up networks that containers use to communicate with one another. These networks are set up with static addresses. Address ranges to be use by private networks are:

```
Class A: 10.0.0.0 to 10.255.255.255
Class B: 172.16.0.0 to 172.31.255.255
Class C: 192.168.0.0 to 192.168.255.255
```

I have chosen 172.16.10.0/24, 172.16.20.0/24, and 172.16.30.0/24 for the first app containers deployed on the Rhubarb Pi. Future app containers will continue to increment by 10.

Each app exposes port 6769 at a minimum. This port is arbitrarily chosen. It is unassigned with ICANN and it is the years of release for my two favorite Beatles albums, Abbey Road and Sgt. Peppers.

The Rhubarb Pi host system maps ports to app container ports as follows:

- 6701 - core server app

The core server app maps its internal port 6769 to system port 6701 for debugging purposes. The other services are accessed by the core server app over the "rhubarbpi" Docker bridge network. To debug the other services, the core server app exposes end points that can be forwarded to the other services.

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
        rhubarb-pi-core-server:
            image: doodles67/rhubarb-pi-core-server:<version>
            container_name: core-server
            restart: always
            networks:
                - rhubarbpi
            ports:
                - 6701:6769
        test-node-app-rpi:
            image: doodles67/docker-node-app-rpi:<version>
            container_name: test-node-app
            restart: always
            networks:
                - rhubarbpi
                - testnode
        zwave-app-rpi:
            image: doodles67/zwave-app-rpi:<version>
            container_name: zwave-app
            restart: always
            devices:
                - /dev/ttyUSB0:/dev/ttyUSB0
            networks:
                - rhubarbpi
                - zwave
        rhubarb-pi-frontend:
            image: doodles67/rhubarb-pi-client-poc:<version>
            container_name: frontend
            restart: always
            //networks:
            //    - rhubarbpi
            ports:
                - 6702:80
            //environment:
            //    WAIT_HOSTS: core-server:6769
    networks:
        rhubarbpi:
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
    
    The other defined networks (zwave, etc.) are sub-networks dedicated to each app allowing private databases, microservices, etc. for each.

3. Disable the default "bridge" network by adding the following statement in the /etc/docker/daemon.json file

    ```
    {
        "iptables": false,
        "bridge": "none"
    }
    ```

    Create the file if it doesn't already exist

4. Enable the docker-compose-app service

    ```
    sudo systemctl enable docker-compose-app
    ```

    Now, every time the Rhubarb Pi reboots the containers are torn down and recreated (? or stopped and restarted?).
