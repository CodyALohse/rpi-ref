# RPi Setup

1. Flash latest Raspbian Lite to SD card using Etcher app.
1. Enable SSH by creating a file called `ssh` on the SD card.
1. Enable Wifi auto connect by creating a new file called `wpa_supplicant.conf` on the SD card. Add the following to the wpa_supplicant.conf:

    ```bash
    ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
    network={
        ssid="YOUR_SSID"
        psk="YOUR_WIFI_PASSWORD"
        key_mgmt=WPA-PSK
    }
    ```

1. Insert SD card into RPi, apply power
1. At the prompt enter the RPi default creds

    ```bash
    Username: pi
    Password: raspberry
    ```

1. Now configure the RPi settings by issuing the following command:

    ```bash
    sudo raspi-config
    ```

1. Change default password
1. Change Network Options -> hostname: desired_schema
1. Update localisation settings:
    * Localisation Options -> Change locale: en_US.UTF-8 UTF-8
    * Next screen choose en_US.UTF-8
1. Localisation Options -> Change Timezone: US -> Central
1. Localisation Options -> Change Keyboard: Generic 104-key PC
1. Next select Other
1. Next select English (US)
1. For Key to function as AltGr: choose default for keyboard
1. Compose key: choose no compose key
1. Localisation Options -> Change Wi-fi Country: US
1. Since we are using the Lite version of Rasbian and don’t have a GUI we can re-allocate some of the GPU memory back to the system
    * Advanced Options -> Memory Split: Change from 64 to 16MB
1. From the main menu, choose Finish, and reboot the system

## Apply Raspbian Updates

1. First make sure we are connected to the internet by issuing the following:

```bash
ping -c 5 google.com
```

1. Update apt with latest 

```bash
sudo apt update
```

1. Then do a full system update

```bash
sudo apt full-upgrade -y
```

## Hostname resolution

In order to connect by hostname instead of ip address we need to install samba

```bash
sudo apt install -y samba
```

Once samba is installed you should be able to ping the Rpi from another machine on the network using the hostname.	

## BCM Library Setup for GPIO access

1. Install the BCM library to provide access to the GPIO pins <https://www.airspayce.com/mikem/bcm2835/>

    ```bash
    tar zxvf bcm2835-1.xx.tar.gz
    cd bcm2835-1.xx
    ./configure
    make
    sudo make check
    sudo make install
    ```

## Dotnet Core Setup - Self Contained

<https://github.com/dotnet/core/blob/master/samples/RaspberryPiInstructions.md>

1. Install dotnet core dependencies on Rpi

    ```bash
    sudo apt-get update
    sudo apt-get install curl libunwind8 gettext apt-transport-https
    ```

1. On the build machine run the following to build for linux-arm

    ```bash
    dotnet publish -r linux-arm
    ```

1. Copy the contents of the publish folder to the rpi <https://github.com/unosquare/sshdeploy>
1. Update permissions to run the app

    ```bash
    chmod +x "dotnet-dll-name"
    ```

1. Run the app:

    ```bash
    Dotnet "dotnet-dll-name".dll
    ```

### Publish via sshdeploy

1. Navigate to your project folder where the csproj file resides. Run the following:

```bash
dotnet-sshdeploy push netcoreapp2.2 -t /THE/PATH/TO/TARGET/DIRECTORY -h DEVICE_IP_ADDRESS -u THE_DEVICE_USERNAME -w THE_DEVICE_PASSWORD
```

**Example** : dotnet-sshdeploy push netcoreapp2.2  -t "/home/pi/dotnet/testapp2" -h 192.168.0.xxx -u pi -w *******

## Expose Web App

Configure NGINX for Rpi: <https://www.thomaslevesque.com/2018/04/17/hosting-an-asp-net-core-2-application-on-a-raspberry-pi/>

1. Install nginx

    ```bash
    sudo apt-get install nginx
    ```

1. Start nginx

    ```bash
    sudo service nginx start
    ```

## Debugging

### Debugging C++ app

Debugging from Visual Studio 2017

1. Install the Visual Studio for C++ on Linux plugin
1. Create  a new project select C++ for Linux.
1. Click Build and a connection popup will display
    * Host: RPi Ip
    * Port: 22
    * Rpi username and password
    * **Note** : You may need to enable SSH on the raspberry pi. Also make sure the following are installed by running

        ```bash
        sudo apt-get install openssh-server g++ gdb gdbserver
        ```

### Debugging Dotnet Core app

1. Install the VS Debugger on the RPi

    ```bash
    curl -sSL https://aka.ms/getvsdbgsh | bash /dev/stdin -r linux-arm -v latest -l ~/vsdbg
    ```

1. Start the Web App on the RPi
1. In VS go to Debug -> Attach To Process
    * Set the connection type to SSH
    * In the Connection Target enter the userName@host (pi@ipAddress)
    * Click Find, a new dialog should open where the device password can be entered.
    * In the list of Available Processes select the process named dotnet (if using self contained app then the process will be named the same as the app).
    * A new dialog will popup asking which type of code you want to debug. 
    * Select the Managed (Dot net core for Unix).
    * **NOTE** : If the client code has changed and has not been loaded onto the RPi, that needs to be done first (See Deploying Section).

1. Now you should be able to set breakpoints in VS and interact with the app on the RPI.

## Docker

### Install Docker:

```bash
curl -sSL https://get.docker.com | sh
```

### Install Docker Compose

```bash
sudo apt-get install libffi-dev libssl-dev
sudo apt-get install -y python python-pip
sudo apt-get remove python-configparser
sudo pip install docker-compose
```

#### Setup auto start on restart

```bash
sudo systemctl enable docker
```


# Misc

Raspberry Pi Info:

```bash
more /proc/cpuinfo
```

### Nano ref

Ctrl + ^ - mark, then arrow to end of text to cut
Ctrl + k to kuts the text
Ctrl + u to paste

### Mount Windows Shared

<https://rasspberrypi.wordpress.com/2012/09/04/mounting-and-automounting-windows-shares-on-raspberry-pi/>

1. Create windows directory
1. Create new user if necessary, didn’t have any luck with guest
1. Share directory with user
1. On pi create new dir in /mnt

    ```bash
    sudo mount -t cifs -o username=yourusername,password=yourpassword //WindowsPC/share1 /mnt/mountfoldername
    ```

## SCREEN

Install

```bash
sudo apt install screen
```

List screens

```bash
Screen -ls
```

Reattach

```bash
Screen -r
```

#### Once using screen

Create a new screen

```bash
Cntr + a c
```

Switch between screens

```bash
Cntr + a n
```

Detach

```bash
Cntr + a d
```
	
## Killing a process

```bash
Ps -A | grep valueToSearch
Kill -9 PID
```
