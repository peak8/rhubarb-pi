# Introduction

The Rhubarb Pi is intended to be used with a display with Openbox as the window manager.  Chromium is used in kiosk mode to display React apps for various applications.

An image is set up and tested on one uSD card and then can be cloned for deployment to other Raspberry Pi's.

# Provision OS and Install Docker

[Based on this tutorial](https://dev.to/elalemanyo/how-to-install-docker-and-docker-compose-on-raspberry-pi-1mo)

1. Provision a uSD card with Raspberry Pi OS Lite (32-bit) based on Debian Buster. The Raspberry Pi Imager app is the easiest way to do this.

2. Power up RPi with uSD installed and ssh into the RPi to avoid the need for a keyboard.

    It seems Raspberry Pi OS may no longer support ssh by default. If this is the case, then enable it through raspi-config using a keyboard.

    ```
    sudo raspi-config
    ```

    From the main menu navigate to Interfacing Options to enable it:
    
    ```
    Interfacing Options
    --> SSH
        --> Yes
            --> OK
                --> Finish
    ```

3. Configure Startup settings for autologin. Done as a convenience for devlopment. Revisit later.

    ```
    sudo raspi-config
    ```

    From the main menu navigate to Console Autologin to enable it on reboot:
    
    ```
    System Options 
    --> Boot / Auto Login 
        --> Console Autologin  
    ```

    Return to the main menu and disable overscan:
    
    ```
    Display Options 
    --> Underscan 
        --> No 
            --> OK
    ```

    Navigate to the Finish option and enter. When asked to reboot select yes.

4. Update and Upgrade to ensure latest versions of software are loaded.

    ```
    sudo apt-get update
    sudo apt-get upgrade
    sudo reboot
    ```

5. Install Docker

    ```
    curl -sSL https://get.docker.com | sh
    ```

6. Install Docker Compose

    ```
    sudo apt-get install libffi-dev libssl-dev
    sudo apt install python3-dev
    sudo apt-get install -y python3 python3-pip
    sudo pip3 install docker-compose
    ```

7. Enable the Docker System Service to run on reboot

    ```
    sudo systemctl enable docker
    ```

8. Install Nodejs and NPM 

    Note: doing "sudo apt install nodejs" will result in an old version (10.24) that can't be updated. Likely due to Raspbian package manager? The following commands needed to be used instead.

    ```
    curl -sL https://deb.nodesource.com/setup_17.x | sudo -E bash -
    sudo apt install -y nodejs
    ```

    After installing Node and npm, I realized that the container image would have Node installed, so I uninstalled as follows:

    ```
    sudo apt purge nodejs
    sudo apt autoremove
    ```

9. Set up the Docker-Compose file

    See DOCKER-COMPOSE-SERVICE.md

10. Optionally set up the kiosk

    This step sets up the Rhubarb Pi to display a React app on a monitor connected to the HDMI ports. See KIOSK-SETUP.md.