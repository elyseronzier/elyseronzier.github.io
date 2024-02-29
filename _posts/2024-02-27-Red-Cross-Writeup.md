---
layout: single
title:  "Red Cross Writeup"
classes: wide
#header:
  #overlay_image: /assets/images/RedCross.jpg
  #og_image: /assets/images/RedCross.jpg
  #caption: "Photo credit: [**Hackthebox**](https://app.hackthebox.com/machines/162)"
categories:
  - Writeup
tags:
  - HTB
  - Linux
  - Medium
  - SQL Injection
  - OS Command Injection
  - Cross Site Scripting (XSS)
  - Information Disclosure

---

*Disclaimer: A few of these writeups are ones that I've had locally stored for a while, or done a long time ago. So in the interest of time, these will be more sparse and less detailed.*
*The more recent posts have better and more in depth analysis.*

---

**Enumeration/Recon**

First steps: run Nmap against the target IP. Once there is confirmation of a website, start running gobuster/dirbuster.

<figure class="half">
    <a href="/assets/images/red-cross-nmap.png"><img src="/assets/images/red-cross-nmap.png"></a>
    <a href="/assets/images/red-cross-login.png"><img src="/assets/images/red-cross-login.png"></a>
    <figcaption>Screenshots from the Red Cross Machine.</figcaption>
</figure>

* Add intra.redcross.htb to my host files.
* The site redirects to a https intranet website login portal. 
* There is a contact form, let's play around with some injections. 

<figure class="half">
    <a href="/assets/images/red-cross-alert.png"><img src="/assets/images/red-cross-alert.png"></a>
    <a href="/assets/images/red-cross-injection.png"><img src="/assets/images/red-cross-injection.png"></a>
    <figcaption>Screenshots from the Red Cross Machine.</figcaption>
</figure>

*The form gets successfully sent, maybe we can try getting a shell through this method? We can try a cookie stealer. 


**Foothold/Exploitation**


<figure class="half">
    <a href="/assets/images/red-cross-reverse-cookie.png"><img src="/assets/images/red-cross-reverse-cookie.png"></a>
    <a href="/assets/images/red-cross-reverse-cookie-injection.png"><img src="/assets/images/red-cross-reverse-cookie-injection.png"></a>
    <figcaption>Screenshots from the Red Cross Machine.</figcaption>
</figure>

{% capture fig_img %}
![Foo]({{ "/assets/images/red-cross-cookie.png" | relative_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>Screenshot from the Red Cross Machine.</figcaption>
</figure>

{% capture fig_img %}
![Foo]({{ "/assets/images/red-cross-admin-session.png" | relative_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>Screenshot from the Red Cross Machine.</figcaption>
</figure>

* Let's keep this admin cookie session and manually change the domain to admin. Makes sense nomenclature wise. 
* Using inspect element isn;t quite working like I would like it to work. Let's manually change it in the URL up top. 

<figure class="half">
    <a href="/assets/images/red-cross-admin-url.png"><img src="/assets/images/red-cross-admin-url.png"></a>
    <a href="/assets/images/red-cross-admin-panel.png"><img src="/assets/images/red-cross-admin-panel.png"></a>
    <figcaption>Screenshots from the Red Cross Machine.</figcaption>
</figure>

* After adding admin.redcross to our hosts files, we've successfully hit the admin panel.

<figure class="half">
    <a href="/assets/images/red-cross-user-manage.png"><img src="/assets/images/red-cross-user-manage.png"></a>
    <a href="/assets/images/red-cross-network-access.png"><img src="/assets/images/red-cross-network-access.png"></a>
    <figcaption>Screenshots from the Red Cross Machine.</figcaption>
</figure>

* After exploring the panel's two options, let's play with user management.
* Adding a test user, gives us some new credentials. 

<figure class="half">
    <a href="/assets/images/red-cross-test-user-creds.png"><img src="/assets/images/red-cross-test-user-creds.png"></a>
    <a href="/assets/images/red-cross-test-user.png"><img src="/assets/images/red-cross-test-user.png"></a>
    <figcaption>Screenshots from the Red Cross Machine.</figcaption>
</figure>

* With our new user, can we use SSH? Spoiler alert: yes.

<figure class="half">
    <a href="/assets/images/red-cross-test-ssh.png"><img src="/assets/images/red-cross-test-ssh.png"></a>
    <a href="/assets/images/red-cross-test-id.png"><img src="/assets/images/red-cross-test-id.png"></a>
    <figcaption>Screenshots from the Red Cross Machine.</figcaption>
</figure>

* Let's do some exploring

<figure class="half">
    <a href="/assets/images/red-cross-test-home.png"><img src="/assets/images/red-cross-test-home.png"></a>
    <a href="/assets/images/red-cross-test-pen.png"><img src="/assets/images/red-cross-test-pen.png"></a>
    <figcaption>Screenshots from the Red Cross Machine.</figcaption>
</figure>

<figure class="half">
    <a href="/assets/images/red-cross-iptctl.png"><img src="/assets/images/red-cross-iptctl.png"></a>
    <a href="/assets/images/red-cross-iptctl-file.png"><img src="/assets/images/red-cross-iptctl-file.png"></a>
    <figcaption>Screenshots from the Red Cross Machine.</figcaption>
</figure>

Okay, so up until now, I've discovered another user (Penelope) on the box. Although I can't do much with them as of right now. 
I've seen the IPTCTL file that I can read even as the test user. 
So I'm going to try to do some IP whitelisting since we have the network management area on the admin portal. Because I need more visibility and want to understand the responses better, I'm going to use BurpSuite. 

<figure class="half">
    <a href="/assets/images/red-cross-burp.png"><img src="/assets/images/red-cross-burp.png"></a>
    <a href="/assets/images/red-cross-burp-shell.png"><img src="/assets/images/red-cross-burp-shell.png"></a>
    <figcaption>Screenshots from the Red Cross Machine.</figcaption>
</figure>

By intercepting a request within Burp, I am able to slot in a nice little reverse shell. 
*(NB: I used cyberchef to url encode, but later learn you can directly encode using Ctrl-U in Burp)*

Let's upgrade to a better shell using a nifty python command. Once that's done, we can immediately start exploring again. 

* Read action.php and discovered some database credentials. Let's figure out how to interact with this db then. 
* Documentation time! (Just going to show a BUNCH of screenshots to show you some of my brain thoughts)

<figure class="half">
    <a href="/assets/images/red-cross-db-creds.png"><img src="/assets/images/red-cross-db-creds.png"></a>
    <a href="/assets/images/red-cross-db-docs.png"><img src="/assets/images/red-cross-db-docs.png"></a>
    <figcaption>Screenshots from the Red Cross Machine.</figcaption>
</figure>

<figure class="half">
    <a href="/assets/images/red-cross-pgconnect.png"><img src="/assets/images/red-cross-pgconnect.png"></a>
    <a href="/assets/images/red-cross-pgconnect-2.png"><img src="/assets/images/red-cross-pgconnect-2.png"></a>
    <figcaption>Screenshots from the Red Cross Machine.</figcaption>
</figure>

<figure class="half">
    <a href="/assets/images/red-cross-pgconnect-3.png"><img src="/assets/images/red-cross-pgconnect-3.png"></a>
    <a href="/assets/images/red-cross-db-connect.png"><img src="/assets/images/red-cross-db-connect.png"></a>
    <figcaption>Screenshots from the Red Cross Machine.</figcaption>
</figure>

* Successfully connected to the database! Let's list out users

{% capture fig_img %}
![Foo]({{ "/assets/images/red-cross-db-users.png" | relative_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>Screenshot from the Red Cross Machine.</figcaption>
</figure>

* 'postgres' is the superuser whereas our current user is basically trash in comparison.
* Can we see passwords?

<figure class="half">
    <a href="/assets/images/red-cross-db-etcshad.png"><img src="/assets/images/red-cross-db-etcshad.png"></a>
    <a href="/assets/images/red-cross-db-etcpass.png"><img src="/assets/images/red-cross-db-etcpass.png"></a>
    <figcaption>Screenshots from the Red Cross Machine.</figcaption>
</figure>


<figure class="half">
    <a href="/assets/images/red-cross-db-etcpass2.png"><img src="/assets/images/red-cross-db-etcpass2.png"></a>
    <a href="/assets/images/red-cross-db-etcpass3.png"><img src="/assets/images/red-cross-db-etcpass3.png"></a>
    <figcaption>Screenshots from the Red Cross Machine.</figcaption>
</figure>

* Hmm, after some tinkering, safe to assume there is no table called pg_passwd.
* Note to self and anyone else: when you can’t find the most useful info on something out there, someone has probs made a repo about it.

<figure class="half">
    <a href="/assets/images/red-cross-psql-repo.png"><img src="/assets/images/red-cross-psql-repo.png"></a>
    <a href="/assets/images/red-cross-psql-repo2.png"><img src="/assets/images/red-cross-psql-repo2.png"></a>
    <figcaption>Screenshots from the Red Cross Machine.</figcaption>
</figure>

<figure class="half">
    <a href="/assets/images/red-cross-db-enum.png"><img src="/assets/images/red-cross-db-enum.png"></a>
    <a href="/assets/images/red-cross-db-enum2.png"><img src="/assets/images/red-cross-db-enum2.png"></a>
    <figcaption>Screenshots from the Red Cross Machine.</figcaption>
</figure>

* pg_passwd was actually passwd_table
* Next steps should be trying to add a user to the table, or inserting a test user, maybe tinkering with Penelope...
* Queue a bunch of screenshots of me trying *stuff*.

<figure class="half">
    <a href="/assets/images/wolverine.png"><img src="/assets/images/wolverine.png"></a>
    <a href="/assets/images/wolverine2.png"><img src="/assets/images/wolverine2.png"></a>
    <figcaption>Screenshots from the Red Cross Machine.</figcaption>
</figure>

* Alright then so we need a password. I guess it has to be encrypted in the same hash type. 
* Soooo linux MD5 I believe, also known as MD5-crypt back in the day. Which means there has to be an easy way of creating one. 

<figure class="half">
    <a href="/assets/images/cryptlib.png"><img src="/assets/images/cryptlib.png"></a>
    <a href="/assets/images/cryptlib2.png"><img src="/assets/images/cryptlib2.png"></a>
    <figcaption>Screenshots from the Red Cross Machine.</figcaption>
</figure>

* So in this example the password is "a" I believe . Let's try that

* Okay so we have a password value 
* We will choose 0 for both GID and UID because those are for root. Therefore we should choose /root for the homedir. 

* As for gecos... what is gecos? There aren't any values for it in the table so I'm guessing it isn't required. Good, let's ignore it.
* But Let's just quickly find out what it is though. 

{% capture fig_img %}
![Foo]({{ "/assets/images/gecoswiki.png" | relative_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>Screenshot from the Red Cross Machine.</figcaption>
</figure>

* This checks out considering how old this db seems to be.

<figure class="half">
    <a href="/assets/images/insertuser.png"><img src="/assets/images/insertuser.png"></a>
    <a href="/assets/images/insertuser2.png"><img src="/assets/images/insertuser2.png"></a>
    <figcaption>Screenshots from the Red Cross Machine.</figcaption>
</figure>

*broke my connections a few times... whatever, it happens.*

{% capture fig_img %}
![Foo]({{ "/assets/images/noroot.png" | relative_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>Screenshot from the Red Cross Machine.</figcaption>
</figure>

* So I guess I can't add any root users. Although I'm not surprised, I am still disappointed.
* Can we add a normal one? Like the same gid and uid as penelope I guess. Cause she was the only other one that was with www-data.
* Let's try those values then. We have all the values accept the homedir, because there isn't one. Do I just invent one? I assume not…
* I'll steal penelope's one I guess. Oh yeah because then if we end up getting her permissions we can also read the user flag... hopefully.

{% capture fig_img %}
![Foo]({{ "/assets/images/newuser.png" | relative_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>Screenshot from the Red Cross Machine.</figcaption>
</figure>

* Let's try to SSH as our new user. Then it'll be explore time. 

<figure class="half">
    <a href="/assets/images/sshaccess.png"><img src="/assets/images/sshaccess.png"></a>
    <a href="/assets/images/sshaccess2.png"><img src="/assets/images/sshaccess2.png"></a>
    <figcaption>Screenshots from the Red Cross Machine.</figcaption>
</figure>

* User flag? SUCCESS
* Time to escalate. We know we can't create a root user but since our user can't run SUDO commands, maybe we can make a user that can?

<figure class="half">
    <a href="/assets/images/sudouser.png"><img src="/assets/images/sudouser.png"></a>
    <a href="/assets/images/sudouser2.png"><img src="/assets/images/sudouser2.png"></a>
    <figcaption>Screenshots from the Red Cross Machine.</figcaption>
</figure>

And just like that, we have managed to grab the root flag from this machine.
Happy Hacking!

