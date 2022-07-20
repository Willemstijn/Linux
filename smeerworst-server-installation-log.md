# Intro

This log contains all commands and information that I gathered to install and configure my Linux Mint 20.3 based "Domotica & more" server.

# Initial installation

Download and install the latest version of Linux Mint XFCE edition (lightweight) on the server from the [Linux Mint website](https://linuxmint.com/edition.php?id=294) [(iso here)](https://mirrors.layeronline.com/linuxmint/stable/20.3/linuxmint-20.3-xfce-64bit.iso). 

# Update

After installation, update the system from the terminal with the following commands:

```
sudo apt update
sudo apt upgrade
```

# Network address

## First method

By default, the server has a DHCP enabled ethernet interface. Since this is a server, it should be static so that I can always reach it on it's default IP address. 

You can configure static IP addresses via the command-line interface (CLI). To do so, execute the following command:

``nmtui``

The following step is to change the “IPv4 CONFIGURATION” setting from automatic to manual and enter the essential info to make it work. if you want to verify that the updated network parameters have been applied, you can run the following command in the terminal.

``ip a``

## Second method

You can also set static IP addresses by making changes to the network configuration file. To do so, open the file with your preferred editor:

``sudo nano etc/network/interfaces``

We’re using a nano editor in the above command, and after loading this file, you’ll need to enter a few lines specified below and then save the file.

```
# interfaces(5) file used by ifup(8) and ifdown(8)
# Include files from /etc/network/interfaces.d:
source-directory /etc/network/interfaces.d

auto eno1
iface eno1 static
address: 192.168.1.5
netmask: 255.255.255.0
gateway: 192.168.1.1
dns-nameservers 208.67.222.222 208.67.220.220
```
Then restart networking

``sudo service network-manager restart``

Check the IP address with:

``ip a``

or

``ifconfig``

# Starting the server in terminal mode by default

Normally Linux Mint starts in X windows. But since this will be a headless server, it is not necessary. Therefore, after boot it will start in terminal mode (which also will save resources and is safer). 

To see the current default target,

``sudo systemctl get-default``

To make linux Mint log in into the command line, use

``sudo systemctl set-default multi-user.target``

To return to default booting into X, use

``sudo systemctl set-default graphical.target``

# Disabling unneeded services

This is a security item. There is no need for some service since I do not use them. Also it will prevent additional possbible security leaks.
