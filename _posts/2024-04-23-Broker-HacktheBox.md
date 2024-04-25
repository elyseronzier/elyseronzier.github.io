---
layout: single
title: "Broker - HacktheBox Writeup"
categories:
  - Writeup
tags:
  - Easy
  - HackTheBox
  - Retired Box
classes:
  - wide
---


**Enumeration/Recon**


First steps: run Nmap against the target IP. If there is confirmation of a website, start running gobuster/dirbuster.

<figure class="half">
    <a href="/assets/images/broker/nmap1.png"><img src="/assets/images/broker/nmap1.png"></a>
    <a href="/assets/images/broker/nmap2.png"><img src="/assets/images/broker/nmap2.png"></a>
</figure>

Lots of fun ports here. 
Quickly checked out port 80 which was asking for login creds. Randomly tried 'admin:admin' and it worked!

<figure class="half">
    <a href="/assets/images/broker/login.png"><img src="/assets/images/broker/login.png"></a>
    <a href="/assets/images/broker/ezcreds.png"><img src="/assets/images/broker/ezcreds.png"></a>
</figure>

I know I shouldn't but I'm going to let myself be distracted by this...oh well!

So immediately, let's do some version control. 

<figure class="half">
    <a href="/assets/images/broker/version.png"><img src="/assets/images/broker/version.png"></a>
    <a href="/assets/images/broker/cve.png"><img src="/assets/images/broker/cve.png"></a>
</figure>

Oh boy...

This is in fact a vulnerable version of activemq. Let's try out this exploit.

Side note: I spent way too long fiddling around with the original golang script before I gave up and found a python version which ended up working much better. :)

<figure class="half">
    <a href="/assets/images/broker/exploit.png"><img src="/assets/images/broker/exploit.png"></a>
    <a href="/assets/images/broker/gitexploit.png"><img src="/assets/images/broker/gitexploit.png"></a>
    <figcaption>Metasploit module version vs the actual git exploit I found</figcaption>
</figure>

After modifying some of the information in the files for the exploit and running a listener paired with a python webserver...we successfully get a shell back. 
Which get's TTY'd immediately.

<figure class="half">
    <a href="/assets/images/broker/pocxml.png"><img src="/assets/images/broker/pocxml.png"></a>
    <a href="/assets/images/broker/shell1.png"><img src="/assets/images/broker/shell1.png"></a>
</figure>

<figure class="half">
    <a href="/assets/images/broker/shell2.png"><img src="/assets/images/broker/shell2.png"></a>
    <a href="/assets/images/broker/whoami.png"><img src="/assets/images/broker/whoami.png"></a>
</figure>

Right. Let's look into why this worked.

* CVE-2023-46604
* CVSS v2 score of 10
* Deserialisation vulnerability
* Flaw in Openwire protocol
* RCE

"OpenWire is a binary protocol that has been specifically designed for working with message-oriented middleware. It serves as the native wire format of ActiveMQ, which is a widely used open-source messaging and integration platform."


{% capture fig_img %}
![Foo]({{ "/assets/images/broker/patchdiff.png" | relative_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
</figure>


"Based on the patch difference, we can see that the validateIsThrowable method has been included in the BaseDataStreamMarshall class. When the marshaller fails to validate the class type of a Throwable (an object that represents exceptions and errors in Java), it can accidentally create and execute instances of any class. This can result in RCE vulnerabilities, allowing an attacker to execute arbitrary code on the server or application."

The existing exploit we're using in this writeup is essentially leveraging the unpatch ProcessBuilder method where no validation is being made, in order to execute code. 

The payload looks like a serialised object and sends it through the ActiveMQ port. The method won't verify anything and the commands are executed. 

---------------------

Now that we have a foothold, let's look for the user flag. 
It is in our current user's home directory, as per usual. 
The first thing to do here is to check our user's privileges on the machine. 
Sudo -l is a classic command which shows us anything the user can perform as a sudoer.

<figure class="half">
    <a href="/assets/images/broker/flagtxt.png"><img src="/assets/images/broker/flagtxt.png"></a>
    <a href="/assets/images/broker/sudo-l.png"><img src="/assets/images/broker/sudo-l.png"></a>
</figure>

It's a great start, seeing as our user can run nginx without any passwords, as root. 
Let's find a way to leverage this to privesc to root and grab the flag. 

Here's where I accidentally rabbit hole'd into an exploit that ultimately didn't work because I missread what my user could do. It happens...

Although the exploit was a fun read and deffo something that is good to know for future reference. 

Here are some screenshots:

<figure class="half">
    <a href="/assets/images/broker/nginxprivesc1.png"><img src="/assets/images/broker/nginxprivesc1.png"></a>
    <a href="/assets/images/broker/ngixprivesc2.png"><img src="/assets/images/broker/ngixprivesc2.png"></a>
</figure>


{% capture fig_img %}
![Foo]({{ "/assets/images/broker/nginxprivesc3.png" | relative_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
</figure>

Eventually, after some more googling, I came across this blog on a relativaly similar situation. 
Time to try it out. 

<figure class="half">
    <a href="/assets/images/broker/lookingforhack.png"><img src="/assets/images/broker/lookingforhack.png"></a>
    <a href="/assets/images/broker/nginxlocalrootexploit.png"><img src="/assets/images/broker/nginxlocalrootexploit.png"></a>
</figure>

So within this exploit, some things needed to be changed. Firstly, let's cURL the root flag. 
Then we need to change the sudo command to what our user is allowed to do. 

<figure class="half">
    <a href="/assets/images/broker/rootexploit.png"><img src="/assets/images/broker/rootexploit.png"></a>
    <a href="/assets/images/broker/roottxt.png"><img src="/assets/images/broker/roottxt.png"></a>
</figure>

Happy Hacking!

