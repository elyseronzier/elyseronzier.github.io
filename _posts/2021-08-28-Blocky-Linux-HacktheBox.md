---
layout: single
title: "Blocky - Linux HacktheBox Writeup"
categories:
  - Writeup
tags:
  - Easy
  - Linux
  - HackTheBox
classes:
  - wide
---

Today's box is Blocky. A linux box rated easy on HacktheBox. 
This box focused on enumeration of the directories that allow us to find the .jar files. These files contained the credentials needed to have SSH access. Finally, the privilege escalation was simply creating a root session via our user and the *sudo su* command. 
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
```
/.htpasswd 
/.htpasswd.cgi 
/.htpasswd.pl 
/.htpasswd.txt 
/.htpasswd.sh 
/.htaccess 
/.htaccess.txt 
/.htaccess.sh 
/.htaccess.cgi 
/.htaccess.pl 
/.hta 
/.hta.txt 
/.hta.sh 
/.hta.cgi 
/.hta.pl 
/index.php 
/javascript 
/license.txt 
/phpmyadmin 
/plugins 
/server-status 
/wiki 
/wp-admin 
/wp-content
/wp-includes
```
Of interest, we see the standard http directories such as *./htpasswd* and *./htaccess*.
We also see */index.php* which brings us to the wordpress page, the website.
There is a login page at */phpmyadmin*. 
And */plugins* contains some .jar files which we can grab to have a look at later. 
Finally, there is a */wiki page*, but it doesn't look to have anything of interest at the moment. However, it says that a new core plugin is being made which will store ''*playtime and other information*''. This could be related to the .jar files we have found. They could contain credentials.   

**FOOTHOLD**

The first thing we can do with all the information we have gathered, is have a look at the .jar files to see if they contain anything useful. 
To do this, we need to unzip the files we have and decode them. Because it's a .jar extension, we can use the JD-GUI tool to do this.
This provides us with a nice UI and IDE to decode the .jar files we want to view.  
We can now read the contents of the BlockyCore.jar file because that's the one of interest since the /wiki directory points us to it. 

In it we find credentials!
``` 
sqluser = root
sqlpass = ****************
```
So now let's try the different ports we can login to.
We try FTP, it doesn't succeed.
We use them on the login page but again, this doesn't work.
Same thing with the SSH address.

Then I realised that maybe the user-name wasn't root, but the user that was also root. So I decided to check the name of the web blog writer. I try Ryan as this is the name found in the .jar files. This doesn't work.
Finally, I realise Ryan's alias is *Notch* on the website. So i use Notch and the password and successfully enter through the SSH port.   

I can now easily grab the user.txt flag!

**PRIVILEGE ESCALATION**

First thing to do in in the shell, is to try the *sudo -l* command to check our current privileges. 
We can use the same password and the result tells us that the user has the permission to run all commands on Blocky. 

This means we can just go ahead and open our root session using the following command: 

*sudo -i* 
*sudo su* also works fine. 

And a root session has been opened.
We can now grab the *root.txt* flag in the root directory.

And that's the box!   