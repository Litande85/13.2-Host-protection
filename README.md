# Домашнее задание к занятию 13.2. «Защита хоста» - `Елена Махота`

- [Ответ к Заданию 1](#1)
- [Ответ к Заданию 2](#2)
- [Ответ к Заданию 3*](#3)


------

### Задание 1

1. Установите **eCryptfs**.
2. Добавьте пользователя cryptouser.
3. Зашифруйте домашний каталог пользователя с помощью eCryptfs.


*В качестве ответа  пришлите снимки экрана домашнего каталога пользователя с исходными и зашифрованными данными.*  

### *<a name = "1"> Ответ к Заданию 1</a>*

**1. Установка `eCryptfs`**

```bash
sudo apt install ecryptfs-utils  lsof
# проверяем включен ли модуль
sudo modprobe ecryptfs
```

**2.Добавление пользователя**

```bash
user@makhota-vm10:~$ sudo adduser cryptouser
Adding user `cryptouser' ...
Adding new group `cryptouser' (1001) ...
Adding new user `cryptouser' (1001) with group `cryptouser' ...
Creating home directory `/home/cryptouser' ...
Copying files from `/etc/skel' ...
New password: 
Retype new password: 
passwd: password updated successfully
# добавляем cryptouser в группу sudo
sudo usermod -aG sudo cryptouser
```

**3. Шифрование домашнего каталога пользователя с помощью eCryptfs.**

```bash
user@makhota-vm10:~$ sudo ecryptfs-migrate-home -u cryptouser
INFO:  Checking disk space, this may take a few moments.  Please be patient.
INFO:  Checking for open files in /home/cryptouser
Enter your login passphrase [cryptouser]: 

************************************************************************
YOU SHOULD RECORD YOUR MOUNT PASSPHRASE AND STORE IT IN A SAFE LOCATION.
  ecryptfs-unwrap-passphrase ~/.ecryptfs/wrapped-passphrase
THIS WILL BE REQUIRED IF YOU NEED TO RECOVER YOUR DATA AT A LATER TIME.
************************************************************************


Done configuring.

chown: cannot access '/dev/shm/.ecryptfs-cryptouser': No such file or directory
INFO:  Encrypted home has been set up, encrypting files now...this may take a while.
sending incremental file list
./
.bash_logout
            220 100%    0.00kB/s    0:00:00 (xfr#1, to-chk=2/4)
.bashrc
          3,526 100%    3.36MB/s    0:00:00 (xfr#2, to-chk=1/4)
.profile
            807 100%  788.09kB/s    0:00:00 (xfr#3, to-chk=0/4)
Could not unlink the key(s) from your keying. Please use `keyctl unlink` if you wish to remove the key(s). Proceeding with umount.

========================================================================
Some Important Notes!

 1. The file encryption appears to have completed successfully, however,
    cryptouser MUST LOGIN IMMEDIATELY, _BEFORE_THE_NEXT_REBOOT_,
    TO COMPLETE THE MIGRATION!!!

 2. If cryptouser can log in and read and write their files, then the migration is complete,
    and you should remove /home/cryptouser.dGKkjQU4.
    Otherwise, restore /home/cryptouser.dGKkjQU4 back to /home/cryptouser.

 3. cryptouser should also run 'ecryptfs-unwrap-passphrase' and record
    their randomly generated mount passphrase as soon as possible.

 4. To ensure the integrity of all encrypted data on this system, you
    should also encrypt swap space with 'ecryptfs-setup-swap'.
========================================================================

user@makhota-vm10:~$ sudo ls /home/
cryptouser  cryptouser.dGKkjQU4  user
user@makhota-vm10:~$ su cryptouser
Password: 
cryptouser@makhota-vm10:/home/user$ cd ~
cryptouser@makhota-vm10:~$ ecryptfs-unwrap-passphrase
Passphrase: 
0f577815e54fe57a76520b7a2f44c6d9
cryptouser@makhota-vm10:~$ sudo ecryptfs-setup-swap

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for cryptouser: 
INFO: You do not currently have any swap space defined.

You can create a swap file by doing:
 $ sudo dd if=/dev/zero of=/swapfile count=8121824
 $ sudo mkswap /swapfile
 $ sudo swapon /swapfile

And then re-run /usr/bin/ecryptfs-setup-swap

cryptouser@makhota-vm10:~$ sudo rm -Rf /home/cryptouser.dGKkjQU4/
cryptouser@makhota-vm10:~$ ls /home/
cryptouser  user
cryptouser@makhota-vm10:~$ touch 123
cryptouser@makhota-vm10:~$ ls
123
cryptouser@makhota-vm10:~$ touch 1234
cryptouser@makhota-vm10:~$ ls
123  1234
cryptouser@makhota-vm10:~$ su user
Password: 
user@makhota-vm10:/home/cryptouser$ sudo reboot
user@makhota-vm10:~$ sudo ls /home/cryptouser/
Access-Your-Private-Data.desktop  README.txt
user@makhota-vm10:~$ sudo cat /home/cryptouser/Access-Your-Private-Data.desktop
[Desktop Entry]
_Name=Access Your Private Data
_GenericName=Access Your Private Data
Exec=/usr/bin/ecryptfs-mount-private
Terminal=true
Type=Application
Categories=System;Security;
X-Ubuntu-Gettext-Domain=ecryptfs-utils
user@makhota-vm10:~$ sudo cat /home/cryptouser/README.txt
THIS DIRECTORY HAS BEEN UNMOUNTED TO PROTECT YOUR DATA.

From the graphical desktop, click on:
 "Access Your Private Data"

or

From the command line, run:
 ecryptfs-mount-private
user@makhota-vm10:~$ sudo su
root@makhota-vm10:/home/user# ls /home/cryptouser/
Access-Your-Private-Data.desktop  README.txt
root@makhota-vm10:/home/user# cd /home/cryptouser/
root@makhota-vm10:/home/cryptouser# ls
Access-Your-Private-Data.desktop  README.txt
root@makhota-vm10:/home/cryptouser# 
root@makhota-vm10:/home/cryptouser# exit
exit
user@makhota-vm10:~$ su cryptouser
Password: 
cryptouser@makhota-vm10:/home/user$ cd ~
cryptouser@makhota-vm10:~$ ls /home/cryptouser/
123  1234
```

![cryptouser](img/img%202023-03-25%20235009.png)

---

### Задание 2

1. Установите поддержку **LUKS**.
2. Создайте небольшой раздел, например, 100 Мб.
3. Зашифруйте созданный раздел с помощью LUKS.

*В качестве ответа пришлите снимки экрана с поэтапным выполнением задания.*

### *<a name = "2"> Ответ к Заданию 2</a>*

**1. Установка `LUKS`**

```bash
sudo apt install cryptsetup 
# Проверка установки:
cryptsetup --version
```

![install](img/img%202023-03-26%20173735.png)

**2. Создание раздела 100 Мб**

```bash
┌──(kali㉿makhota-kali)-[~]
└─$ lsblk 
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      8:0    0 80.1G  0 disk 
└─sda1   8:1    0 80.1G  0 part /
sdb      8:16   0    1G  0 disk 
sr0     11:0    1 1024M  0 rom  
                                                          
┌──(kali㉿makhota-kali)-[~]
└─$ sudo fdisk /dev/sdb

Welcome to fdisk (util-linux 2.38.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS (MBR) disklabel with disk identifier 0x9b7262a7.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): 

Using default response p.
Partition number (1-4, default 1): 
First sector (2048-2097151, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-2097151, default 2097151): +100M

Created a new partition 1 of type 'Linux' and of size 100 MiB.

Command (m for help): i
Selected partition 1
         Device: /dev/sdb1
          Start: 2048
            End: 206847
        Sectors: 204800
      Cylinders: 13
           Size: 100M
             Id: 83
           Type: Linux
    Start-C/H/S: 0/32/33
      End-C/H/S: 12/223/19

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

                                                                                                                                                                      
┌──(kali㉿makhota-kali)-[~]
└─$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      8:0    0 80.1G  0 disk 
└─sda1   8:1    0 80.1G  0 part /
sdb      8:16   0    1G  0 disk 
└─sdb1   8:17   0  100M  0 part 
sr0     11:0    1 1024M  0 rom  

```

**3. Шифрование созданного раздела с помощью LUKS**

Подготовка раздела (luksFormat):
```bash
sudo cryptsetup -y -v --type luks2 luksFormat /dev/sdb1
# YES пишем капсом!
```

![luksFormat](img/img%202023-03-26%20182536.png)

Монтирование раздела:

```bash
sudo cryptsetup luksOpen /dev/sdb1 disk
ls /dev/mapper/disk
```

![luksOpen](img/img%202023-03-26%20182908.png)

Форматирование раздела:

```bash
sudo dd if=/dev/zero of=/dev/mapper/disk
sudo mkfs.ext4 /dev/mapper/disk 
```

![dd](img/img%202023-03-26%20183726.png)

![mkfs](img/img%202023-03-26%20183756.png)

Монтирование «открытого» раздела:
```bash
mkdir .secret
sudo mount /dev/mapper/disk .secret/
```
![mount](img/img%202023-03-26%20184428.png)

Завершение работы:
```bash
sudo umount .secret
sudo cryptsetup luksClose disk
```

![luksClose](img/img%202023-03-26%20185604.png)

![status](img/img%202023-03-26%20190203.png)

---
## Дополнительные задания (со звёздочкой*)

Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале

### Задание 3 *

1. Установите **apparmor**.
2. Повторите эксперимент, указанный в лекции.
3. Отключите (удалите) apparmor.


*В качестве ответа пришлите снимки экрана с поэтапным выполнением задания.*

### *<a name = "3"> Ответ к Заданию 3 </a>*

```bash
┌──(kali㉿makhota-kali)-[~]
└─$ sudo apt install apparmor-profiles apparmor-utils apparmor-profiles-extra
...

┌──(kali㉿makhota-kali)-[~]
└─$ sudo apparmor_status
apparmor module is loaded.
29 profiles are loaded.
7 profiles are in enforce mode.
   /usr/bin/pidgin
   /usr/bin/pidgin//sanitized_helper
   /usr/bin/totem
   /usr/bin/totem-audio-preview
   /usr/bin/totem-video-thumbnailer
   /usr/bin/totem//sanitized_helper
   apt-cacher-ng
22 profiles are in complain mode.
   /usr/bin/irssi
   avahi-daemon
   dnsmasq
   dnsmasq//libvirt_leaseshelper
   identd
   klogd
   mdnsd
   nmbd
   nscd
   php-fpm
   ping
   samba-bgqd
   samba-dcerpcd
   samba-rpcd
   samba-rpcd-classic
   samba-rpcd-spoolss
   smbd
   smbldap-useradd
   smbldap-useradd///etc/init.d/nscd
   syslog-ng
   syslogd
   traceroute
0 profiles are in kill mode.
0 profiles are in unconfined mode.
0 processes have profiles defined.
0 processes are in enforce mode.
0 processes are in complain mode.
0 processes are unconfined but have a profile defined.
0 processes are in mixed mode.
0 processes are in kill mode.

                                                                                                                                                                      
┌──(kali㉿makhota-kali)-[~]
└─$  ls /etc/apparmor.d/

abi                    local            samba-rpcd          tunables                  usr.lib.ipsec.charon    usr.sbin.mariadbd         usr.sbin.traceroute
abstractions           lsb_release      samba-rpcd-classic  usr.bin.irssi             usr.lib.ipsec.stroke    usr.sbin.mdnsd
apache2.d              nvidia_modprobe  samba-rpcd-spoolss  usr.bin.man               usr.sbin.apt-cacher-ng  usr.sbin.nmbd
bin.ping               php-fpm          sbin.dhclient       usr.bin.pidgin            usr.sbin.avahi-daemon   usr.sbin.nscd
disable                samba            sbin.klogd          usr.bin.tcpdump           usr.sbin.dnsmasq        usr.sbin.ntpd
force-complain         samba-bgqd       sbin.syslogd        usr.bin.totem             usr.sbin.haveged        usr.sbin.smbd
lightdm-guest-session  samba-dcerpcd    sbin.syslog-ng      usr.bin.totem-previewers  usr.sbin.identd         usr.sbin.smbldap-useradd
                                                                                                                                                                      
┌──(kali㉿makhota-kali)-[~]
└─$ sudo cp /usr/bin/man /usr/bin/man1

┌──(kali㉿makhota-kali)-[~]
└─$ ls /usr/bin | grep '^man' 
man
man1
mandb
manpath
man-recode
                                                                                                                                                                      
┌──(kali㉿makhota-kali)-[~]
└─$ sudo cp /bin/ping /usr/bin/man

                                                                                                                                                                      
┌──(kali㉿makhota-kali)-[~]
└─$ sudo man 127.0.0.1
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.033 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.059 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.044 ms
^C
--- 127.0.0.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2047ms
rtt min/avg/max/mdev = 0.033/0.045/0.059/0.010 ms
                                                                                                                                                                      
┌──(kali㉿makhota-kali)-[~]
└─$ sudo aa-enforce man
Setting /usr/bin/man to enforce mode.
                                                                                                                                                                      
┌──(kali㉿makhota-kali)-[~]
└─$ sudo man 127.0.0.1
man: socktype: SOCK_RAW
man: socket: Permission denied
                                                                                                                                                                      
┌──(kali㉿makhota-kali)-[~]
└─$ cat /etc/apparmor.d/bin.ping

# ------------------------------------------------------------------
#
#    Copyright (C) 2002-2009 Novell/SUSE
#    Copyright (C) 2010 Canonical Ltd.
#
#    This program is free software; you can redistribute it and/or
#    modify it under the terms of version 2 of the GNU General Public
#    License published by the Free Software Foundation.
#
# ------------------------------------------------------------------

abi <abi/3.0>,

include <tunables/global>
profile ping /{usr/,}bin/{,iputils-}ping flags=(complain) {
  include <abstractions/base>
  include <abstractions/consoles>
  include <abstractions/nameservice>

  capability net_raw,
  capability setuid,
  network inet raw,
  network inet6 raw,

  /{,usr/}bin/{,iputils-}ping mixr,
  /etc/modules.conf r,

  # Site-specific additions and overrides. See local/README for details.
  include if exists <local/bin.ping>
}
                                                                                                                                                                      
┌──(kali㉿makhota-kali)-[~]
└─$ sudo service apparmor stop

┌──(kali㉿makhota-kali)-[~]
└─$ sudo man 127.0.0.1          
man: socktype: SOCK_RAW
man: socket: Permission denied
                                                                                                                                                                      
┌──(kali㉿makhota-kali)-[~]
└─$ sudo service apparmor teardown
Usage: /etc/init.d/apparmor {start|stop|restart|reload|force-reload|status}
                                                                                                                                                                    
┌──(kali㉿makhota-kali)-[~]
└─$ sudo cp /usr/bin/man1 /usr/bin/man

┌──(kali㉿makhota-kali)-[~]
└─$ sudo service apparmor status
○ apparmor.service - Load AppArmor profiles
     Loaded: loaded (/lib/systemd/system/apparmor.service; disabled; preset: disabled)
     Active: inactive (dead)
       Docs: man:apparmor(7)
             https://gitlab.com/apparmor/apparmor/wikis/home/
```


