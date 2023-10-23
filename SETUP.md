# Introduction

The Rhubarb Pi can be run as a headless server or with a display with Openbox as a window manager and Chromium used in kiosk mode to display React oe Electron apps for various applications.

An image is set up and tested on one uSD card and then can be cloned for deployment to other Raspberry Pi's.

# Provision OS and Install Docker

[Based on this tutorial](https://dev.to/elalemanyo/how-to-install-docker-and-docker-compose-on-raspberry-pi-1mo)

1. Provision a uSD card with Raspberry Pi OS Lite (32-bit) based on Debian Buster. The Raspberry Pi Imager app is the easiest way to do this.

&nbsp;

2. Power up RPi with uSD installed and ssh into the RPi to avoid the need for a keyboard.

    It seems Raspberry Pi OS may no longer support ssh by default. If this is the case, then enable it through raspi-config using a keyboard.

    ```
    $ sudo raspi-config
    ```

    From the main menu navigate to Interfacing Options to enable it:
    
    ```
    Interfacing Options
    --> SSH
        --> Yes
            --> OK
                --> Finish
    ```

&nbsp;

3. Configure Startup settings for autologin. Done as a convenience for devlopment. Revisit later.

    ```
    $ sudo raspi-config
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

&nbsp;

4. Update and Upgrade to ensure latest versions of software are loaded.

    ```
    $ sudo apt-get update
    $ sudo apt-get upgrade
    $ sudo reboot
    ```

&nbsp;

5. Install Docker

    ```
    $ curl -sSL https://get.docker.com | sh
    ```

&nbsp;

6. Install Docker Compose

    ```
    $ sudo apt-get install libffi-dev libssl-dev
    $ sudo apt install python3-dev
    $ sudo apt-get install -y python3 python3-pip
    $ sudo pip3 install docker-compose
    ```

&nbsp;

7. Enable the Docker System Service to run on reboot

    ```
    $ sudo systemctl enable docker
    ```

&nbsp;

8. Set up the Docker-Compose file

    See DOCKER-COMPOSE-SERVICE.md

&nbsp;

9. Optionally set up the kiosk (recommended)

    This step sets up the Rhubarb Pi to display a React app on a monitor connected to the HDMI ports. See KIOSK-SETUP.md.

&nbsp;

10. Optionally set up WiFi

    ```
    $ sudo raspi-config
    ```

    From the main menu navigate to Wireless LAN to configure it:
    
    ```
    System Options 
    --> Wireless LAN
    ```

    You will see a list of countries, select the US.

    You will then be asked to enter the wireless SSID (network name)

    Finally, you will be asked to enter the password.

    Select Finish.

    Inspect the list of networks and confirm wlan0 is now configured and note the IP address.