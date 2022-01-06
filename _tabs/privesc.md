---
title: PrivEsc
icon: fas fa-info-circle
order: 4
---

The below is a modification from [here](https://github.com/Ignitetechnologies/Privilege-Escalation) !


Table of Contents
=================

* [Abusing Sudo Rights](#sudo)
* [SUID Bit](#suid)
* [Writable /etc/passwd file](#etc)
* [Python Pickle](#pickle)
* [Path Variable](#path)
* [Docker Escape](#docker)
* [Writable files or script](#wfos)
* [Environment Variable](#ev)

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
 
<a name="suid"></a>
##  SUID Bit [⤴](#table-of-contents)

|No.| Machine Name                 |SUID Bit| Reference | Method |
|-------|------------------------------|-------|-------|--------------------------------------------------------|
|1|[ShellDredd #1 Hannah](https://www.vulnhub.com/entry/onsystem-shelldredd-1-hannah,545/)| cpulimit|[Reference](https://am-a-circle.github.io/posts/ONSYSTEM-HANNAH/)  | cpulimit -l 50 -f cp /bin/bash /tmp/bash <br> cpulimit -l 50 -f chmod +s /tmp/bash <br>  /tmp/bash -p|
|2|[hackmyvm : connection](https://hackmyvm.eu/machines/machine.php?vm=Connection)| gdb |[Reference](https://am-a-circle.github.io/posts/connection/)  | gdb -nx -ex 'python import os; os.execl("/bin/sh", "sh", "-p")' -ex quit |



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

| No | Machine Name|
|----|-----------|
|1.| [icarus](https://tamanduaflag.github.io/posts/hack-my-vm-icarus/)| LD_PRELOAD |
    |
