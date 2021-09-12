---
title: "Legacy - Windows HacktheBox Writeup"
categories:
  - Writeup
tags:
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
