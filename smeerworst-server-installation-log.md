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

Or ports can be removed with the following procedure:

```
# sudo ufw status numbered

Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     ALLOW IN    Anywhere
[ 2] 22/tcp (v6)                ALLOW IN    Anywhere (v6)
[ 3] 9000/tcp (v6)              ALLOW IN    Anywhere (v6)

# ufw delete 3

Deleting:
 allow 9000/tcp
Proceed with operation (y|n)? y
Rule deleted (v6)

# sudo ufw status numbered

Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     ALLOW IN    Anywhere
[ 2] 22/tcp (v6)                ALLOW IN    Anywhere (v6)

```

See also my [Server baseling document](https://github.com/Willemstijn/Linux/blob/main/Server-baseline.md#adding-exceptions-for-services)

# Nameserver

After removing the systemd-resolved service, you should check if the file /etc/resolv.conf still exists or is a dead symlink.

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

## network manager issues...

It seems that after each reboot, network manager overwrites ``resolv.conf`` with a faulty file:

```
# Generated by NetworkManager
nameserver 127.0.0.53
```

Why that is so is beyond my comprehension... However I have to do the following to make it work once again:

First open the network manager conf file ``/etc/NetworkManager/NetworkManager.conf``:

```
sudo nano /etc/NetworkManager/NetworkManager.conf
```

And add this to the [main] section:

```
dns=none
rc-manager=unmanaged
```

So that it looks like this:

```
[main]
plugins=ifupdown,keyfile
dns=none
rc-manager=unmanaged

[ifupdown]
managed=false

[device]
wifi.scan-rand-mac-address=no
```

Save, exit and restart network-manager:

```
sudo service network-manager restart
```

Another option might be:

``/etc/resolv.conf`` is symlinked to ``/run/resolvconf/resolv.conf``. NetworkManager doesn't update /etc/resolv.conf directly (only updates /run/resolvconf/resolv.conf). 

So:

- remove symlink (rm /etc/resolv.conf)
- write you own version of /etc/resolv.conf

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

# Mounting addidional disks

Follow the procedure below to add additional disks to the system:

Find out what is attached with ``lsblk``

```
lsblk

NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 223,6G  0 disk
├─sda1   8:1    0   512M  0 part /boot/efi
└─sda2   8:2    0 223,1G  0 part /
sdb      8:16   0   3,7T  0 disk
└─sdb1   8:17   0   3,7T  0 part
```

Find if any of them is mounted with ``findmnt`` (or ``mount -l``)

```
findmnt

TARGET                                SOURCE     FSTYPE     OPTIONS
/                                     /dev/sda2  ext4       rw,relatime,errors=remount-ro
├─/sys                                sysfs      sysfs      rw,nosuid,nodev,noexec,relatime
│ ├─/sys/kernel/security              securityfs securityfs rw,nosuid,nodev,noexec,relatime
│ ├─/sys/fs/cgroup                    tmpfs      tmpfs      ro,nosuid,nodev,noexec,mode=755
│ │ ├─/sys/fs/cgroup/unified          cgroup2    cgroup2    rw,nosuid,nodev,noexec,relatime,nsdelegate
│ │ ├─/sys/fs/cgroup/systemd          cgroup     cgroup     rw,nosuid,nodev,noexec,relatime,xattr,name=systemd
│ │ ├─/sys/fs/cgroup/perf_event       cgroup     cgroup     rw,nosuid,nodev,noexec,relatime,perf_event
│ │ ├─/sys/fs/cgroup/net_cls,net_prio cgroup     cgroup     rw,nosuid,nodev,noexec,relatime,net_cls,net_prio
│ │ ├─/sys/fs/cgroup/cpu,cpuacct      cgroup     cgroup     rw,nosuid,nodev,noexec,relatime,cpu,cpuacct
│ │ ├─/sys/fs/cgroup/cpuset           cgroup     cgroup     rw,nosuid,nodev,noexec,relatime,cpuset
│ │ ├─/sys/fs/cgroup/hugetlb          cgroup     cgroup     rw,nosuid,nodev,noexec,relatime,hugetlb
│ │ ├─/sys/fs/cgroup/rdma             cgroup     cgroup     rw,nosuid,nodev,noexec,relatime,rdma
│ │ ├─/sys/fs/cgroup/freezer          cgroup     cgroup     rw,nosuid,nodev,noexec,relatime,freezer
│ │ ├─/sys/fs/cgroup/pids             cgroup     cgroup     rw,nosuid,nodev,noexec,relatime,pids
│ │ ├─/sys/fs/cgroup/memory           cgroup     cgroup     rw,nosuid,nodev,noexec,relatime,memory
│ │ ├─/sys/fs/cgroup/blkio            cgroup     cgroup     rw,nosuid,nodev,noexec,relatime,blkio
│ │ └─/sys/fs/cgroup/devices          cgroup     cgroup     rw,nosuid,nodev,noexec,relatime,devices
│ ├─/sys/fs/pstore                    pstore     pstore     rw,nosuid,nodev,noexec,relatime
│ ├─/sys/firmware/efi/efivars         efivarfs   efivarfs   rw,nosuid,nodev,noexec,relatime
│ ├─/sys/fs/bpf                       none       bpf        rw,nosuid,nodev,noexec,relatime,mode=700
│ ├─/sys/kernel/debug                 debugfs    debugfs    rw,nosuid,nodev,noexec,relatime
│ ├─/sys/kernel/tracing               tracefs    tracefs    rw,nosuid,nodev,noexec,relatime
│ ├─/sys/fs/fuse/connections          fusectl    fusectl    rw,nosuid,nodev,noexec,relatime
│ └─/sys/kernel/config                configfs   configfs   rw,nosuid,nodev,noexec,relatime
├─/proc                               proc       proc       rw,nosuid,nodev,noexec,relatime
│ └─/proc/sys/fs/binfmt_misc          systemd-1  autofs     rw,relatime,fd=28,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=18576
├─/dev                                udev       devtmpfs   rw,nosuid,noexec,relatime,size=4018108k,nr_inodes=1004527,mode=755
│ ├─/dev/pts                          devpts     devpts     rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000
│ ├─/dev/shm                          tmpfs      tmpfs      rw,nosuid,nodev
│ ├─/dev/mqueue                       mqueue     mqueue     rw,nosuid,nodev,noexec,relatime
│ └─/dev/hugepages                    hugetlbfs  hugetlbfs  rw,relatime,pagesize=2M
├─/run                                tmpfs      tmpfs      rw,nosuid,nodev,noexec,relatime,size=813016k,mode=755
│ ├─/run/lock                         tmpfs      tmpfs      rw,nosuid,nodev,noexec,relatime,size=5120k
│ └─/run/user/1000                    tmpfs      tmpfs      rw,nosuid,nodev,relatime,size=813012k,mode=700,uid=1000,gid=1000
└─/boot/efi                           /dev/sda1  vfat       rw,relatime,fmask=0077,dmask=0077,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro
```

Disk sdb1 is not used here...

Mount the drive to a location on the disk. In my case I use /mnt.

```
sudo mount -t auto /dev/sdb1 /mnt
```

To find out the disk file type, use the following command ``df -Th``

```
df -Th

Filesystem     Type      Size  Used Avail Use% Mounted on
udev           devtmpfs  3,9G     0  3,9G   0% /dev
tmpfs          tmpfs     794M  1,2M  793M   1% /run
/dev/sda2      ext4      219G  9,2G  199G   5% /
tmpfs          tmpfs     3,9G     0  3,9G   0% /dev/shm
tmpfs          tmpfs     5,0M  4,0K  5,0M   1% /run/lock
tmpfs          tmpfs     3,9G     0  3,9G   0% /sys/fs/cgroup
/dev/sda1      vfat      511M  5,3M  506M   2% /boot/efi
tmpfs          tmpfs     794M  4,0K  794M   1% /run/user/1000
/dev/sdb1      ext4      3,6T   28K  3,4T   1% /mnt
```

sdb1 is of type ext4 and almost completely empty, which is fortunate for me!

## Mount disk at boot with fstab

Find out the UUID of the disk you want to mount:

```
sudo blkid -p /dev/sdb1

/dev/sdb1: LABEL="bigdisk" UUID="5d437851-5667-4907-888c-5372028c6342" VERSION="1.0" TYPE="ext4" USAGE="filesystem" PART_ENTRY_SCHEME="gpt" PART_ENTRY_UUID="9a12b6c1-b954-4883-b3ab-527570d21f56" PART_ENTRY_TYPE="0fc63daf-8483-4772-8e79-3d69d8477de4" PART_ENTRY_NUMBER="1" PART_ENTRY_OFFSET="2048" PART_ENTRY_SIZE="7814033408" PART_ENTRY_DISK="8:16"
```

Make a copy of /etc/fstab:

```
sudo cp /etc/fstab{,.org}
```

Then add the following to the file to make the sdb1 disk mount at sysetm boot:

```
# 4 TB harddrive on /dev/sdb1
UUID=5d437851-5667-4907-888c-5372028c6342 /mnt ext4 defaults 0 2
```

So fstab should look like this now:

```
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sda2 during installation
UUID=2c91431f-6e42-46d3-bba1-22e7c013481a /               ext4    errors=remount-ro 0       1
# /boot/efi was on /dev/sda1 during installation
UUID=B419-BD4C  /boot/efi       vfat    umask=0077      0       1
/swapfile                                 none            swap    sw              0       0
# 4 TB harddrive on /dev/sdb1
UUID=5d437851-5667-4907-888c-5372028c6342 /mnt ext4 defaults 0 2
```

Let's do a reboot and see if everything that we did so far also works after the system has booted...

```
 sudo shutdown -r now
```

Now that the system is configured, we can start with installing additional programs...

# Install unattended updates

To keep the server up-to-date with the latest stable packages and security fixes, use the unattended upgrades package.

```
sudo apt install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

# Installing other programs & stuff

> This is were the fun begins!

The following list of programms are added with apt-get:

```
sudo apt install mc  python3-pip python-is-python3
```

# Install Docker and Docker compose

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

## Docker and ufw

Before we continue. There was a problem where I could reach ports of docker containers, while ufw was blocking them all. There also were no indications that thes ports were allowed by ufw. Nonetheless I could still reach these ports (e.g. port 80 of nginx was open before there even was a firewall rule to let this traffic pass).

It turns out that Docker makes changes directly on your iptables, which are not shown with ufw status.

I had to create the file /etc/docker/daemon.json and put the following in:

```
{
    "iptables": false
}
```

I then issued ``sudo service docker stop`` then ``sudo service docker start`` FINALLY docker is simply following the appropriate rules in UFW. 

See also [here](https://askubuntu.com/questions/652556/uncomplicated-firewall-ufw-is-not-blocking-anything-when-using-docker)

But the story continues about that...

## Docker and name resolution

At some point freqtrade was broken. After investigation it seems like that the iptables entry causes docker to be very strict with it's firewall (no outside traffic or DNS querie possible, or whatever

```
freqtrade    | 2022-07-23 05:49:58,051 - freqtrade.exchange.exchange - ERROR - Unable to initialize markets.
freqtrade    | Traceback (most recent call last):
freqtrade    |   File "/home/ftuser/.local/lib/python3.10/site-packages/urllib3/connection.py", line 174, in _new_conn
freqtrade    |     conn = connection.create_connection(
freqtrade    |   File "/home/ftuser/.local/lib/python3.10/site-packages/urllib3/util/connection.py", line 72, in create_connection
freqtrade    |     for res in socket.getaddrinfo(host, port, family, socket.SOCK_STREAM):
freqtrade    |   File "/usr/local/lib/python3.10/socket.py", line 955, in getaddrinfo
freqtrade    |     for res in _socket.getaddrinfo(host, port, family, type, proto, flags):
freqtrade    | socket.gaierror: [Errno -3] Temporary failure in name resolution
freqtrade    |
freqtrade    | During handling of the above exception, another exception occurred:
freqtrade    |
freqtrade    | Traceback (most recent call last):
freqtrade    |   File "/home/ftuser/.local/lib/python3.10/site-packages/urllib3/connectionpool.py", line 703, in urlopen
freqtrade    |     httplib_response = self._make_request(
freqtrade    |   File "/home/ftuser/.local/lib/python3.10/site-packages/urllib3/connectionpool.py", line 386, in _make_request
freqtrade    |     self._validate_conn(conn)
freqtrade    |   File "/home/ftuser/.local/lib/python3.10/site-packages/urllib3/connectionpool.py", line 1040, in _validate_conn
freqtrade    |     conn.connect()
freqtrade    |   File "/home/ftuser/.local/lib/python3.10/site-packages/urllib3/connection.py", line 358, in connect
freqtrade    |     self.sock = conn = self._new_conn()
freqtrade    |   File "/home/ftuser/.local/lib/python3.10/site-packages/urllib3/connection.py", line 186, in _new_conn
freqtrade    |     raise NewConnectionError(
freqtrade    | urllib3.exceptions.NewConnectionError: <urllib3.connection.HTTPSConnection object at 0x7f5d731ccc70>: Failed to establish a new connection: [Errno -3] Temporary failure in name resolution
```

etc. etc.

When I remove the daemon.json file. Everything works like usual...

```
2022-07-23 06:13:58,350 - freqtrade.resolvers.strategy_resolver - INFO - Strategy using max_entry_position_adjustment: -1
2022-07-23 06:13:58,350 - freqtrade.configuration.config_validation - INFO - Validating configuration ...
2022-07-23 06:13:58,354 - freqtrade.exchange.exchange - INFO - Instance is running with dry_run enabled
2022-07-23 06:13:58,354 - freqtrade.exchange.exchange - INFO - Using CCXT 1.89.14
2022-07-23 06:13:58,361 - freqtrade.exchange.exchange - INFO - Using Exchange "Kraken"
2022-07-23 06:14:01,839 - freqtrade.resolvers.exchange_resolver - INFO - Using resolved exchange 'Kraken'...
2022-07-23 06:14:02,049 - freqtrade.wallets - INFO - Wallets synced.
2022-07-23 06:14:02,149 - freqtrade.rpc.rpc_manager - INFO - Enabling rpc.api_server
2022-07-23 06:14:03,030 - freqtrade.rpc.api_server.webserver - INFO - Starting HTTP Server at 0.0.0.0:8080
2022-07-23 06:14:03,030 - freqtrade.rpc.api_server.webserver - WARNING - SECURITY WARNING - Local Rest Server listening to external connections
2022-07-23 06:14:03,030 - freqtrade.rpc.api_server.webserver - WARNING - SECURITY WARNING - This is insecure please set to your loopback,e.g 127.0.0.1 in config.json
2022-07-23 06:14:03,030 - freqtrade.rpc.api_server.webserver - INFO - Starting Local Rest Server.
2022-07-23 06:14:03,042 - uvicorn.error - INFO - Started server process [1]
2022-07-23 06:14:03,042 - uvicorn.error - INFO - Waiting for application startup.
2022-07-23 06:14:03,043 - uvicorn.error - INFO - Application startup complete.
2022-07-23 06:14:03,059 - freqtrade.resolvers.iresolver - INFO - Using resolved pairlist VolumePairList from '/freqtrade/freqtrade/plugins/pairlist/VolumePairList.py'...
2022-07-23 06:14:03,245 - VolumePairList - INFO - Searching 20 pairs: ['BTC/USDT', 'USDC/USDT', 'ETH/USDT', 'ADA/USDT', 'DOT/USDT', 'DAI/USDT', 'EOS/USDT', 'XRP/USDT', 'DOGE/USDT', 'LTC/USDT', 'LINK/USDT', 'BCH/USDT', 'USTC/USDT']
2022-07-23 06:14:03,249 - freqtrade.strategy.hyper - INFO - No params for buy found, using default values.
2022-07-23 06:14:03,249 - freqtrade.strategy.hyper - INFO - Strategy Parameter(default): buy_rsi = 30
2022-07-23 06:14:03,250 - freqtrade.strategy.hyper - INFO - Strategy Parameter(default): exit_short_rsi = 30
2022-07-23 06:14:03,250 - freqtrade.strategy.hyper - INFO - No params for sell found, using default values.
2022-07-23 06:14:03,250 - freqtrade.strategy.hyper - INFO - Strategy Parameter(default): sell_rsi = 70
2022-07-23 06:14:03,251 - freqtrade.strategy.hyper - INFO - Strategy Parameter(default): short_rsi = 70
2022-07-23 06:14:03,251 - freqtrade.strategy.hyper - INFO - No params for protection found, using default values.
2022-07-23 06:14:03,251 - freqtrade.plugins.protectionmanager - INFO - No protection Handlers defined.
2022-07-23 06:14:03,252 - freqtrade.rpc.rpc_manager - INFO - Sending rpc message: {'type': status, 'status': 'running'}
2022-07-23 06:14:03,252 - freqtrade.worker - INFO - Changing state to: RUNNING
2022-07-23 06:14:03,252 - freqtrade.rpc.rpc_manager - INFO - Sending rpc message: {'type': warning, 'status': 'Dry run is enabled. All trades are simulated.'}
2022-07-23 06:14:03,252 - freqtrade.rpc.rpc_manager - INFO - Sending rpc message: {'type': startup, 'status': "*Exchange:* `kraken`\n*Stake per trade:* `100 USDT`\n*Minimum ROI:* `{'60': 0.01, '30': 0.02, '0': 0.04}`\n*Stoploss:* `-0.1`\n*Position adjustment:* `Off`\n*Timeframe:* `5m`\n*Strategy:* `SampleStrategy`"}
2022-07-23 06:14:03,252 - freqtrade.rpc.rpc_manager - INFO - Sending rpc message: {'type': startup, 'status': "Searching for USDT pairs to buy and sell based on [{'VolumePairList': 'VolumePairList - top 20 volume pairs.'}]"}
```

After reading, [Docker advised](https://docs.docker.com/network/iptables/#prevent-docker-from-manipulating-iptables) to not use the iptables option, because:

> Setting iptables to false will more than likely break container networking for the Docker engine.

So I enabled this again and removed the firewall rules from ufw. Letting Docker do the firewall stuff for the containers from now on. 

## Docker containers

All Docker images and containers will be handled by docker compose. The most easy way to handle this is by making one docker expose file and adding all these containers to it, so to manage all running containers from one file. This makes it easier to backup and restore.

Also all other container configurations will be located in one single directory: ``/opt``

This way there is only one complete directory to backup and restore in combination with only one docker-compose.yml file.

The actual docker-compose file I use is managed via [this github repository](https://github.com/Willemstijn/dockerstack).

So far the folowing programs are running by means of a docker container:

- freqtrade
- home assistant
- zwavejs2mqtt
- mosquitto
- nginx
- portainer
- pihole
- jupyterlabs

### Additional container information

All containers are managed with Portainer. All configuration files are located in ``/opt/`` for an easy backup.

It has the following directory structure:

```
tree -L 2 -d /opt/

/opt/
├── BitcoinLottery
│   └── __pycache__
├── CandleHoarder
│   ├── data
│   ├── log
│   ├── mdwiki
│   ├── misc
│   ├── modules
│   ├── notebooks
│   └── __pycache__
├── containerd
│   ├── bin
│   └── lib
├── domotica
│   ├── domoticz
│   ├── hass
│   ├── mqtt
│   └── zwavejs2mqtt
├── ft_userdata
│   └── user_data
├── jupyter
│   └── python_modules
├── nginx
├── notebooks
│   ├── datasets
│   ├── modules
│   ├── other
│   └── tests
├── pihole
│   ├── etc-dnsmasq.d
│   └── etc-pihole
├── portainer
│   └── data
├── scripts
└── www
    └── content
```

#### Home Assistant

To see on which ports the devices RFXCom and ZWave are located, check the ``/dev/serial/by-id/`` directory:

```
ls -alh /dev/serial/by-id/
total 0
drwxr-xr-x 2 root root 80 Jul 25 11:39 .
drwxr-xr-x 4 root root 80 Jul 25 10:33 ..
lrwxrwxrwx 1 root root 13 Jul 25 10:33 usb-0658_0200-if00 -> ../../ttyACM0
lrwxrwxrwx 1 root root 13 Jul 25 11:39 usb-RFXCOM_RFXtrx433XL_DO2ZY2AK-if00-port0 -> ../../ttyUSB0
```

Restart HASS if there seems to be any problem with it...

## Downloading backups of configuration files from a network share

Because my backups are kept on a network share, I have to download them in this server. This can be done by mounting the share to a local folder and then downloading the necessary files...

To mount the external drive/share, use the next command:

```
mkdir /home/bas/Remote

sudo mount.cifs //192.168.1.3/ActiveBackup /home/bas/Remote -o user=[USERNAAM],pass=[PASSWORD],vers=2.0
```

Then just use Midnight Commander to copy files from one location to the other.

# Installing non-Docker programs 

## Bitcoinlottery

See [Bitcoinlottery github repo](https://github.com/Willemstijn/BitcoinLottery).

## CandleHoarder

See [CandleHoarder github repo](https://github.com/Willemstijn/CandleHoarder).

# Ports

The following services and ports are used on the server:

| Outside Port | Inside port (Docker) | Service (container) | Remarks |
|:-:|:-:|:-|:-|
| 80 | 80 | nginx | Proxying webserver, also with Wiki |
| 8123 | 8123 | Home assistant | Domotica service |
| 3000 | 3000 | Zwavejs2Mqtt | Domotica MQTT service |
| 1883 | 1883 | Mosquitto | Domotica MQTT service |
| 9000 | 9000 | Portainer | Docker container management |
| 8080 | 8080 | Freqtrade | Trading bot |
| 53 | 53 | PiHole DNS | PiHole DNS service port |
| 67 | 67 | PiHole DHCP | PiHole DHCP port (not in use) |
| 81 | 80 | PiHole Web | PiHole administration interface |
| 8888 | 8888 | Jupyter | Jupyterlabs notebook service |
|  |  |  |  |
