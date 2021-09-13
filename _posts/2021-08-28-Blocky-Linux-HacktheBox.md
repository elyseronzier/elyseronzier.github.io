---
title: "Blocky - Linux HacktheBox Writeup"
categories:
  - Writeup
tags:
  - Linux
  - HackTheBox
classes:
  - wide
---

Today's box is Blocky. A linux box rated easy on HacktheBox. 
This box focused on enumeration of the directories that allow us to find the .jar files. These files contained the credentials needed to have SSH access. Finally, the privilege escalation was simply creating a root session via our user and the sudo su command. 
Here are my notes/my writeup for the box:

**ENUMERATION**

Using nmap, we get the following results: 
```
$ sudo nmap -sS -sC -sV -Pn 10.10.10.37

Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-13 16:38 BST
Nmap scan report for 10.10.10.37
Host is up (0.077s latency).
Not shown: 996 filtered ports

PORT     STATE  SERVICE VERSION
21/tcp   open   ftp     ProFTPD 1.3.5a
22/tcp   open   ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d6:2b:99:b4:d5:e7:53:ce:2b:fc:b5:d7:9d:79:fb:a2 (RSA)
|   256 5d:7f:38:95:70:c9:be:ac:67:a0:1e:86:e7:97:84:03 (ECDSA)
|_  256 09:d5:c2:04:95:1a:90:ef:87:56:25:97:df:83:70:67 (ED25519)
80/tcp   open   http    Apache httpd 2.4.18 ((Ubuntu))
|_http-generator: WordPress 4.8
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: BlockyCraft &#8211; Under Construction!
8192/tcp closed sophos

Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
``` 
We see three open TCP ports, running FTP(21), SSH(22) and  HTTP(80) respectively. We also have a closed TCP port running on port 8192.
Looking at their version might provide us with potential vulnerabilities. 

Next, we gather Dirbuster/Gobuster information as there is a website which may contain interesting directories. 
Using Dirbuster, we get the following results: 

  