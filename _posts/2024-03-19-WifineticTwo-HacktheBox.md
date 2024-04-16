---
layout: single
title: "WifineticTwo - HacktheBox Writeup"
categories:
  - Writeup
tags:
  - Medium
  - HackTheBox
  - Ranked/Active Box
classes:
  - wide
---


**Enumeration/Recon**


First steps: run Nmap against the target IP. Once there is confirmation of a website, start running gobuster/dirbuster.

We get a very verbose Nmap output, which is always fun. 

{% capture fig_img %}
![Foo]({{ "/assets/images/wifinetic2/nmap.png" | relative_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
</figure>

2 ports stand out here: 

* port 22 - SSH
* port 8080 - HTTP

Visiting the website, we are faced with a login page for something called OpenPLC. 

<figure class="half">
    <a href="/assets/images/wifinetic2/login.png"><img src="/assets/images/wifinetic2/login.png"></a>
    <a href="/assets/images/wifinetic2/researchgate.png"><img src="/assets/images/wifinetic2/researchgate.png"></a>
</figure>

> OpenPLC is an open-source Programmable Logic Controller that is based on an easy to use software. It is the first fully functional standardized open source PLC, both in software and in hardware.

Did a quick search of latest vulns and even reading through a paper discussing the security of the software. There are definitely mentions of a few, nothing that jumps to my face. 
Next I looked up the default creds for openPLC and tried them out. 
Lo and behold, we are in. 

<figure class="half">
    <a href="/assets/images/wifinetic2/defaults.png"><img src="/assets/images/wifinetic2/defaults.png"></a>
    <a href="/assets/images/wifinetic2/defaultcreds.png"><img src="/assets/images/wifinetic2/defaultcreds.png"></a>
</figure>

Always try the simplest answer first folks. 
Default OpenPLC creds:

Username: openplc
Password: openplc

This dashboard area allows us to interact in many ways. 

* Adding programs
* Adding users
* Adding devices
* Changing hardware and monitoring options

<figure class="half">
    <a href="/assets/images/wifinetic2/uploadprog.png"><img src="/assets/images/wifinetic2/uploadprog.png"></a>
    <a href="/assets/images/wifinetic2/currentusers.png"><img src="/assets/images/wifinetic2/currentusers.png"></a>
</figure>


Let's first add ourselves as a user. Then let's quickly make sure these default creds and this new user can't just connect via SSH. 


<figure class="half">
    <a href="/assets/images/wifinetic2/addmeli.png"><img src="/assets/images/wifinetic2/addmeli.png"></a>
    <a href="/assets/images/wifinetic2/sshtry.png"><img src="/assets/images/wifinetic2/sshtry.png"></a>
</figure>

----

Right so after some searching, reading and exploring for a while now. 
There's this really interesting paper called 'Good Night, and Good Luck: A Control Logic
Injection Attack on OpenPLC' which investigates the security of the controller. 

And another document talking about using a reverse shell using ladder logic code. 
The latter seems more in tune with what I want to try right now.

----

Okay, so after some playing around, writing my own ladder program is a little complicated without the help of actual hardware to mimic the boolians.

So instead I've gone back to a exploit I found earlier which I couldn't get to work at the time. This exploit is perfect for this scenario, I just need to debug it...

<figure class="half">
    <a href="/assets/images/wifinetic2/exploitfile.png"><img src="/assets/images/wifinetic2/exploitfile.png"></a>
    <a href="/assets/images/wifinetic2/error.png"><img src="/assets/images/wifinetic2/error.png"></a>
</figure>

So Occam's razor. The exploit was pointing to the wrong program. I kept trying to write a new file when I could overwrite the pre-existing one in this portal. 

We now have a lovely reverse shell. Let's make it a TTY and move on to find the user flag (hopefully).

<figure class="half">
    <a href="/assets/images/wifinetic2/reverseshell.png"><img src="/assets/images/wifinetic2/reverseshell.png"></a>
    <a href="/assets/images/wifinetic2/userflag.png"><img src="/assets/images/wifinetic2/userflag.png"></a>
</figure>

Ah. Well that's funky, we are already root so I ended up finding the user flag in the root directory. But the real funky part is being root.
This makes me think we are going to need to laterally move to another machine or backend? I'm guessing we are in some sort of container?

<figure class="half">
    <a href="/assets/images/wifinetic2/ipa.png"><img src="/assets/images/wifinetic2/ipa.png"></a>
    <a href="/assets/images/wifinetic2/hacktrickswifi.png"><img src="/assets/images/wifinetic2/hacktrickswifi.png"></a>
</figure>

* eth0 is your standard LAN interface which usually has the target IP.
* lo is the standard localhost interface 127.0.0.1 
* mon interfaces are monitor mode interfaces used for sniffing and monitoring traffic on a WIFI network. This is why some tools used for wifi pentesting require you to use mon interfaces. 
* wlan interfaces are used for interfacing with wireless networks. 

<figure class="half">
    <a href="/assets/images/wifinetic2/hacktricks2.png"><img src="/assets/images/wifinetic2/hacktricks2.png"></a>
    <a href="/assets/images/wifinetic2/hacktricks3.png"><img src="/assets/images/wifinetic2/hacktricks3.png"></a>
</figure>

After some looking around, and also knowing the name of this box. I get a sense that wifihacking is the game. So lots of hacktricks reading helps here as I'm not too versed in this. 
Okay so let's try a classic: the pixiedust attack. 

Lots of trouble getting a script on the target...I decided to look for a different version of the tool. 

<figure class="half">
    <a href="/assets/images/wifinetic2/pythonpixie.png"><img src="/assets/images/wifinetic2/pythonpixie.png"></a>
    <a href="/assets/images/wifinetic2/oneshotpy.png"><img src="/assets/images/wifinetic2/oneshotpy.png"></a>
</figure>

Success, we've got the script on the target. Now let's run it, and hopefully get ourselves some creds/passcodes.

{% capture fig_img %}
![Foo]({{ "/assets/images/wifinetic2/pixiedustattack.png" | relative_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
</figure>

Excellent!

<figure class="half">
    <a href="/assets/images/wifinetic2/learningwificonnect.png"><img src="/assets/images/wifinetic2/learningwificonnect.png"></a>
    <a href="/assets/images/wifinetic2/wifistuff.png"><img src="/assets/images/wifinetic2/wifistuff.png"></a>
</figure>

Wireless settings are typically stored in /etc/wpa_supplicant.conf, and when we check said directory there are no config files. 
So not only do we have the service active but with no assigned IP, we also have no config files. 
This makes me think, we can be the ones to set it up with our new found pin and passphrase. 

<figure class="half">
    <a href="/assets/images/wifinetic2/passphrasesetting.png"><img src="/assets/images/wifinetic2/passphrasesetting.png"></a>
    <a href="/assets/images/wifinetic2/runningdaemon.png"><img src="/assets/images/wifinetic2/runningdaemon.png"></a>
</figure>

Following these instructions about setting up a new wLAN online, I manage to manually assign an IP address and a netmask to the wLAN0 interface.

<figure class="half">
    <a href="/assets/images/wifinetic2/settingip.png"><img src="/assets/images/wifinetic2/settingip.png"></a>
    <a href="/assets/images/wifinetic2/sshroot.png"><img src="/assets/images/wifinetic2/sshroot.png"></a>
</figure>

{% capture fig_img %}
![Foo]({{ "/assets/images/wifinetic2/rootflag.png" | relative_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
</figure>

When we try to SSH to the new IP as root it doesn't work, so I try a different IP within the range and boom! We ssh and find ourselves a nice root flag. 

Happy Hacking!
