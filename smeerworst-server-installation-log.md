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

```
 sudo netstat -tulpan

Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      655/systemd-resolve
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      760/sshd: /usr/sbin
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      676/cupsd
tcp        0     80 192.168.1.5:22          192.168.1.204:50696     ESTABLISHED 1197/sshd: bas [pri
tcp6       0      0 :::22                   :::*                    LISTEN      760/sshd: /usr/sbin
tcp6       0      0 ::1:631                 :::*                    LISTEN      676/cupsd
udp        0      0 127.0.0.53:53           0.0.0.0:*                           655/systemd-resolve
udp        0      0 192.168.1.5:123         0.0.0.0:*                           755/ntpd
udp        0      0 127.0.0.1:123           0.0.0.0:*                           755/ntpd
udp        0      0 0.0.0.0:123             0.0.0.0:*                           755/ntpd
udp        0      0 0.0.0.0:631             0.0.0.0:*                           736/cups-browsed
udp        0      0 0.0.0.0:5353            0.0.0.0:*                           674/avahi-daemon: r
udp        0      0 0.0.0.0:54826           0.0.0.0:*                           674/avahi-daemon: r
udp6       0      0 fe80::e2fc:197e:c76:123 :::*                                755/ntpd
udp6       0      0 ::1:123                 :::*                                755/ntpd
udp6       0      0 :::123                  :::*                                755/ntpd
udp6       0      0 :::5353                 :::*                                674/avahi-daemon: r
udp6       0      0 :::34640                :::*                                674/avahi-daemon: r
```

Another way of seeing which services are enabled by default is:

```
 sudo service --status-all

 [ + ]  acpid
 [ - ]  alsa-utils
 [ - ]  anacron
 [ + ]  apparmor
 [ + ]  avahi-daemon
 [ - ]  bluetooth
 [ - ]  console-setup.sh
 [ + ]  cron
 [ - ]  cryptdisks
 [ - ]  cryptdisks-early
 [ + ]  cups
 [ + ]  cups-browsed
 [ + ]  dbus
 [ - ]  dns-clean
 [ - ]  grub-common
 [ + ]  hddtemp
 [ - ]  hwclock.sh
 [ + ]  irqbalance
 [ + ]  kerneloops
 [ - ]  keyboard-setup.sh
 [ + ]  kmod
 [ - ]  lightdm
 [ + ]  lm-sensors
 [ - ]  lvm2
 [ - ]  lvm2-lvmpolld
 [ - ]  mintsystem
 [ + ]  network-manager
 [ - ]  networking
 [ + ]  ntp
 [ + ]  openvpn
 [ - ]  plymouth
 [ - ]  plymouth-log
 [ - ]  pppd-dns
 [ + ]  procps
 [ - ]  pulseaudio-enable-autospawn
 [ - ]  rsync
 [ + ]  rsyslog
 [ - ]  saned
 [ - ]  speech-dispatcher
 [ + ]  ssh
 [ + ]  udev
 [ + ]  ufw
 [ - ]  uuidd
 [ - ]  x11-common
```

The following service ports should be disables:

* 53
* 631
* 5353
* 123

This can be done byfollowing:

## systemd

```
sudo systemctl status systemd-resolved
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved
sudo systemctl status systemd-resolved
```

## cups

```
sudo systemctl status cups
sudo systemctl stop cups
sudo systemctl disable cups
sudo systemctl status cups
```

and

```
sudo systemctl status cups-browsed
sudo systemctl stop cups-browsed
sudo systemctl disable cups-browsed
sudo systemctl status cups-browsed
```

## avahi-daemon

```
sudo systemctl status avahi-daemon
sudo systemctl stop avahi-daemon
sudo systemctl disable avahi-daemon
sudo systemctl status avahi-daemon
```

## ntp

```
sudo systemctl status ntp
sudo systemctl stop ntp
sudo systemctl disable ntp
sudo systemctl status ntp
```

Now only this service is actively listening on a port:

```
 sudo netstat -tulpan
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      760/sshd: /usr/sbin
tcp        0     64 192.168.1.5:22          192.168.1.204:51225     ESTABLISHED 1526/sshd: bas [pri
tcp6       0      0 :::22                   :::*                    LISTEN      760/sshd: /usr/sbin
```

# Firewall

To install a firewall, do:

``sudo apt install ufw``

Add ssh port to the firewall and enable it

```
sudo ufw allow ssh/tcp
sudo ufw limit ssh/tcp
sudo ufw enable
sudo ufw status
```

Other ports can be added with the following command:

``ufw allow [PORT]/tcp``

See also my [Server baseling document](https://github.com/Willemstijn/Linux/blob/main/Server-baseline.md#adding-exceptions-for-services)

# Nameserver

To install the correct nameserver, do the following:

```
sudo cp /etc/resolv.conf{,.org}
sudo nano /etc/resolv.conf
```

Make the config look like this:

```
#nameserver 127.0.0.53
options edns0 trust-ad
nameserver 208.67.222.222
nameserver 208.67.220.220
```
