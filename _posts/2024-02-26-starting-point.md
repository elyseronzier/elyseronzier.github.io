---
layout: single
title:  "How to Hello World..?"
classes: wide

---

# How to Hello World..?


It's been a while since I've explored the very easy boxes on HTB, so you can imagine my surprise when I came across the new 'Starting Point' space.

This new area is dedicated to bringing a complete noob through all the basics they'll need before trying harder machines. 
HackTheBox have clearly made a lot of effort in making this new space intuitive and beginner friendly. With very easy boxes now accompanied by step-by-step guided questions and a dedicated pdf walkthrough, hacking couldn't be any more accessible.

**Starting Point is divided into 3 main modules:**

{% capture fig_img %}
![Foo]({{ "/assets/images/starting-point-tiers.jpg" | relative_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>Screenshot from Hackthebox.</figcaption>
</figure>

Tier 0 - 
* Learn how to connect FTP, SMB, Telnet, Rsync and RDP anonymously.
* Learn how to use Nmap to identify open ports.
* Learn how to connect to a MongoDB server.

Tier 1 - 
* Learn basic web exploitation techniques such as SQL injection, Server Side Template Injection, Remote File Inclusion and how to use Web/Reverse Shells.
* Use the services showcased in the previous module for exploitation.
* Learn how to login to Jenkins and upload a Groovy Shell Script.
* Learn how to upload files to an S3 Bucket.

Tier 2 -
* Learn how to exploit XXE, IDOR, Log4j and perform cookie manipulation.
* Learn how to exploit binary path hijacking and sudo permissions for privilege escalation.
* Learn the basics of Brute Forcing.
* Learn how to exploit LXD for privileged filesystem access.
* Learn how to exploit insecure functions like "stcmp()" in PHP.

