# Introduction

The Blueberry Pi can have a Kiosk mode, where a React app is displayed on a monitor connected to one of the HDMI ports. This requires a number of packages be installed for managing the graphics. This document details the procedure for setting up the environment using Openbox as a Window Manager.

The procedures are based on the following tutorials and assumes Raspberry Pi OS Lite (32-bit) based on Debian Buster is the base OS:

- [pimylifeup](https://pimylifeup.com/raspberry-pi-kiosk/)
- [desertbot](https://desertbot.io/blog/raspberry-pi-touchscreen-kiosk-setup)

# Instructions

1. Install Minimum UI Components

    Blueberry Pi requires a Chromium browser and a minimal set of UI components to support it.

    ```
    sudo apt-get install --no-install-recommends xserver-xorg x11-xserver-utils xinit openbox
    ```

    ToDo: add obconf, obmenu, and other addons described in the openbox wiki?

2. Install Chromium

    ```
    sudo apt-get install --no-install-recommends chromium-browser
    ```

3. Edit Openbox autostart script

    Openbox window manager will launch the Chromium browser. There are two scripts in the /etc/xdg/openbox folder that need some customization. The environment script will setup any environment variables, etc. and the autostart script will setup and launch my app.

    Open the autostart script

    ```
    sudo nano /etc/xdg/openbox/autostart
    ```

    Add commands to turn off power management, screen blanking, and screen saving

    ```
    xset -dpms            # turn off display power management system  
    xset s noblank        # turn off screen blanking  
    xset s off            # turn off screen saver  
    ```

    Suppress error messages when Chromium crashes

    ```
    # Remove exit errors from the config files that could trigger a warning  
    sed -i 's/"exited_cleanly":false/"exited_cleanly":true/' ~/.config/chromium/'Local State'
    sed -i 's/"exited_cleanly":false/"exited_cleanly":true/; s/"exit_type":"[^"]\+"/"exit_type":"Normal"/' ~/.config/chromium/Default/Preferences
    ```

    Run the Chromium browser in kiosk mode. Pass in an environment variable ($KIOSK_URL) that contains the URL of the Web app to launch.

    ```
    # Run Chromium in kiosk mode
    chromium-browser  --noerrdialogs --disable-infobars --kiosk $KIOSK_URL
    ```

    Set the check for update interval to 30 days (what does this do?)

    ```
    --check-for-update-interval=2592000
    ```

    Save the autostart script

    Note: the [Archlinux wiki site for Openbox](https://wiki.archlinux.org/title/openbox) mentions I must create a local Openbox profile by copying the four scripts/config files. What todo?

4. Point Openbox at the desired app running on the Blueberry Pi

    Open the environment script

    ```
    sudo nano /etc/xdg/openbox/environment
    ```

    Add the KIOSK_URL to the file

    ```
    export KIOSK_URL=http://172.16.10.2:6769/app
    ```

    172.16.10.2 is the static IP address on the docker blueberrypi network which is assigned to the main kiosk app. See DOCKER-COMPOSE-SERVICE.md for more details.