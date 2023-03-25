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
sudo apt install ecryptfs-utils cryptsetup lsof
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
# повышаем права
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



---
## Дополнительные задания (со звёздочкой*)

Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале

### Задание 3 *

1. Установите **apparmor**.
2. Повторите эксперимент, указанный в лекции.
3. Отключите (удалите) apparmor.


*В качестве ответа пришлите снимки экрана с поэтапным выполнением задания.*

### *<a name = "3"> Ответ к Заданию 3</a>*




