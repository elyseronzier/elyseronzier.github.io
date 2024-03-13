---
layout: single
title: "Legacy - Windows HacktheBox Writeup"
categories:
  - Writeup
tags:
  - Easy
  - Windows
  - HackTheBox
classes:
  - wide
---

Today's writeup will focus on the Legacy windows box. This box is rated easy on HacktheBox. It's a great box for beginners and really reinforces the strength of initial enumeration. 

**ENUMERATION**

Running Nmap gives us the following results for ports:

```
PORT     STATE  SERVICE       VERSION

139/tcp  open   netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open   microsoft-ds  Windows XP microsoft-ds
3389/tcp closed ms-wbt-server

Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp
```
The open ports of interest here are two tcp ports. 
Nmap also gives us scripts results, and what immediately stands out is that smb protocol is used. This, in tandem with the windows XP usage and a bit of researching shows up as an exploitable vulnerability. 
In fact, this exploit is known as [MS08-067](https://www.rapid7.com/db/modules/exploit/windows/smb/ms08_067_netapi/). 
According to the website the exploit works because of a parsing flaw in the 'NETAPI32.dll' path canonicalization code. This is an executable file that has the Windows NET API machine code which is used by other apps to access the windows network. 

This is available as a metasploit module so we can go set it up in our terminal. 

*(Note to self: You can also use metasploit with its abundance of auxiliary modules to find version of running services and and protocols. Worth a shot, doesn't always work.)*

**FOOTHOLD**

We see our exploit called *exploit/windows/smb/ms08_067_netapi*.
Modifying the options for the exploit, we can then run it and successfully get a shell.

First thing we do when in a meterpreter shell is run the following commands: 
```
> getuid
> sysinfo
```
What we find out is that we are NT AUTHORITY\SYSTEM on the shell, this is the equivalent of being root and having root privileges on Linux. This likely means we'll be able to access the directory and files needed to find the flags. 
The second command allows to find out a few things about our shell, namely if the architecture of the OS matches that of the shell.

I found moving around in the Windows shell a bit awkward at first since I had gotten used to exploit Linux boxes. But very quickly, you can get the hang of things. 

So, moving around the shell, we finally find the user flag in C:\Documents and Settings\john\Desktop\user.txt
and the root flag in
C:\Documents and Settings\Administrator\Desktop\root.txt

**Success!**