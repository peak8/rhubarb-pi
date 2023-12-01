# Introduction

The docker-compose-service will run as a system service on reboot. All app containers that run on the Rhubarb Pi, will be lauched from this service.

## Container Networks

The docker-compose file sets up networks that containers used to communicate with one another. These networks are set up with static addresses. Address ranges to be used by private networks are:

```
Class A: 10.0.0.0 to 10.255.255.255
Class B: 172.16.0.0 to 172.31.255.255
Class C: 192.168.0.0 to 192.168.255.255
```

I have chosen 172.16.10.0/24 for the main Rhubarb Pi network. The front end and core server will both talk on this network. Other applications installed in the future may require an isolated network (such as a database). New networks will increment the network id by 10 (172.16.20, 172.16.30, etc.).

Each app exposes port 6769 at a minimum. This port is arbitrarily chosen. It is unassigned with ICANN and it is the years of release for my two favorite Beatles albums, Abbey Road and Sgt. Peppers.

The Rhubarb Pi host system maps ports to app container ports as follows:

- 6701 - core server app
- 6702 - Kiosk app port

The core server app maps its internal port 6769 to system port 6701 and the kiosk app maps its internal port 6769 to system port 6702. Future services can be accessed by the core server app over the "rhubarbpi" Docker bridge network. To access the other services, the core server app exposes end points that can be forwarded to the other services. Alternatively, new services may be mapped to a system port.

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

    &nbsp;

2. Create the docker-compose.yaml file.

    Create a docker folder in the /srv folder

    ```
    sudo mkdir /srv/docker
    cd /srv/docker
    ```

    Create the docker-compose.yaml file. The minimum configuration requires a core-server app and a rhubarbpi network as shown below:

    ```
    services:
        rhubarb-pi-core-server:
            image: <docker-hub-path>:<version>
            container_name: core-server
            restart: always
            networks:
                - rhubarbpi
            ports:
                - 6701:6769
    networks:
        rhubarbpi:
            driver: bridge
            enable_ipv6: false
            ipam:
                driver: default
                config:
                    - subnet: 172.16.10.0/24
                      gateway: 172.16.10.1
    ```

    Optionally, you can add a kiosk service as shown below:

    ```
        rhubarb-pi-frontend:
            image: <docker-hub-path>:<version>
            container_name: frontend
            restart: always
            networks:
                - rhubarbpi
            ports:
                - 6702:3000
            //environment:
            //    WAIT_HOSTS: core-server:6769
    ```

    Additional services may be added as the application requires. Details for these additional applications shall be specified in the core-server app repository deployed on the system.

&nbsp;

3. Disable the default "bridge" network by adding the following statement in the /etc/docker/daemon.json file

    ```
    {
        "iptables": false,
        "bridge": "none"
    }
    ```

    Create the file if it doesn't already exist

&nbsp;

4. Enable the docker-compose-app service

    ```
    sudo systemctl enable docker-compose-app
    ```

    Now, every time the Rhubarb Pi reboots the containers are torn down and recreated (? or stopped and restarted?).
