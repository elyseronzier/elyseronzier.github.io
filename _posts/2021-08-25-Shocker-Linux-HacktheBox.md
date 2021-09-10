---
title: "Shocker - Linux HacktheBox Writeup"
categories:
  - Blog
  - Writeup
tags:
  - link
  - Post Formats
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