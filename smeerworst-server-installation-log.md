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

(The time service is needed to sync the servers time for use with a trading bot and domotica)

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

Now only this service is actively listening on a port:

```
sudo netstat -tulpan

Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      3289/sshd: /usr/sbi
tcp        0      0 192.168.1.5:22          192.168.1.204:51225     ESTABLISHED 1526/sshd: bas [pri
tcp        0      0 192.168.1.5:22          192.168.1.204:51966     ESTABLISHED 3214/sshd: bas [pri
tcp        0    272 192.168.1.5:22          192.168.1.204:52186     ESTABLISHED 3388/sshd: bas [pri
tcp        0      0 192.168.1.5:22          192.168.1.204:51877     ESTABLISHED 3063/sshd: bas [pri
tcp6       0      0 :::22                   :::*                    LISTEN      3289/sshd: /usr/sbi
udp        0      0 192.168.1.5:123         0.0.0.0:*                           3683/ntpd
udp        0      0 127.0.0.1:123           0.0.0.0:*                           3683/ntpd
udp        0      0 0.0.0.0:123             0.0.0.0:*                           3683/ntpd
udp6       0      0 fe80::e2fc:197e:c76:123 :::*                                3683/ntpd
udp6       0      0 ::1:123                 :::*                                3683/ntpd
udp6       0      0 :::123                  :::*                                3683/ntpd

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

# SSH server configuration

To make SSH more secure, the following configuration should be used:

Create backup of original ssh config

```
sudo cp /etc/ssh/sshd_config{,.bak}
sudo nano /etc/ssh/sshd_config
```

Then change the config so that it looks line this:

```
Include /etc/ssh/sshd_config.d/*.conf
SyslogFacility AUTH
LogLevel INFO
LoginGraceTime 120
PermitRootLogin no
StrictModes yes
X11Forwarding no
X11DisplayOffset 10
ChallengeResponseAuthentication no
UsePAM yes
AcceptEnv LANG LC_*
Subsystem       sftp    /usr/lib/openssh/sftp-server
PrintMotd no
```

Restart server with:

``sudo systemctl restart ssh``

# Synchronise with NTP

To make the server synchronise with a time server, do the following:

```
sudo cp /etc/systemd/timesyncd.conf{,.org}
sudo nano /etc/systemd/timesyncd.conf
```

Change the config to the following:

```
[Time]
NTP=0.nl.pool.ntp.org  
FallbackNTP=0.europe.pool.ntp.org 1.europe.pool.ntp.org
```

Check the timedatectl

```
timedatectl status

               Local time: Wed 2022-07-20 10:52:23 CEST
           Universal time: Wed 2022-07-20 08:52:23 UTC
                 RTC time: Wed 2022-07-20 08:52:23
                Time zone: Europe/Amsterdam (CEST, +0200)
System clock synchronized: yes
              NTP service: n/a
          RTC in local TZ: no
```

Check the date and time

```
date

Wed 20 Jul 2022 10:53:12 AM CEST
```

# Install unattended updates

To keep the server up-to-date with the latest stable packages and security fixes, use the unattended upgrades package.

```
sudo apt install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

# Install Docker and Docker compose

> This is were the fun begins!

This information comes from my [Docker commands sheet](https://github.com/Willemstijn/Linux/blob/main/Docker%20commands.md).

```
sudo apt-get update
sudo apt-get upgrade
```

Ten install Docker with

```
sudo apt install docker*


Reading package lists... Done
Building dependency tree
Reading state information... Done
Note, selecting 'docker-compose' for glob 'docker*'
Note, selecting 'docker' for glob 'docker*'
Note, selecting 'docker.io-doc' for glob 'docker*'
Note, selecting 'docker2aci' for glob 'docker*'
Note, selecting 'docker-registry' for glob 'docker*'
Note, selecting 'docker-doc' for glob 'docker*'
Note, selecting 'docker-ce' for glob 'docker*'
Note, selecting 'docker.io' for glob 'docker*'
Note, selecting 'docker-doc' instead of 'docker.io-doc'
The following additional packages will be installed:
  bridge-utils containerd git git-man liberror-perl pigz python3-attr python3-cached-property python3-distutils python3-docker python3-dockerpty python3-docopt python3-importlib-metadata
  python3-jsonschema python3-lib2to3 python3-more-itertools python3-pyrsistent python3-setuptools python3-texttable python3-websocket python3-zipp runc ubuntu-fan wmdocker
Suggested packages:
  aufs-tools cgroupfs-mount | cgroup-lite debootstrap rinse zfs-fuse | zfsutils git-daemon-run | git-daemon-sysvinit git-doc git-el git-email git-gui gitk gitweb git-cvs git-mediawiki
  git-svn python-attr-doc python-jsonschema-doc python-setuptools-doc
The following NEW packages will be installed:
  bridge-utils containerd docker docker-compose docker-doc docker-registry docker.io docker2aci git git-man liberror-perl pigz python3-attr python3-cached-property python3-distutils
  python3-docker python3-dockerpty python3-docopt python3-importlib-metadata python3-jsonschema python3-lib2to3 python3-more-itertools python3-pyrsistent python3-setuptools
  python3-texttable python3-websocket python3-zipp runc ubuntu-fan wmdocker
0 upgraded, 30 newly installed, 0 to remove and 0 not upgraded.
Need to get 83,6 MB of archives.
After this operation, 404 MB of additional disk space will be used.
Do you want to continue? [Y/n]
```

Now check the Docker version with:

```
docker --version

Docker version 20.10.12, build 20.10.12-0ubuntu2~20.04.1
```
