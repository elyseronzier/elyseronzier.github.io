---
layout: single
title: "Popcorn - HacktheBox Writeup"
categories:
  - Writeup
tags:
  - Medium
  - HackTheBox
classes:
  - wide
---


**Enumeration/Recon**

First steps: run Nmap against the target IP. Once there is confirmation of a website, start running gobuster/dirbuster.

<figure class="half">
    <a href="/assets/images/popcorn/nmap.png"><img src="/assets/images/popcorn/nmap.png"></a>
    <a href="/assets/images/popcorn/gobuster.png"><img src="/assets/images/popcorn/gobuster.png"></a>
    <figcaption>HTB</figcaption>
</figure>

Let's explore these different directories real quick: 

* /index

Okay. Sure. Nothing useful here.

* /test

This directory gives us A LOT of info on the website. Including the ability to upload files. 

{% capture fig_img %}
![Foo]({{ "/assets/images/popcorn/testdirectory.png" | relative_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
</figure>

* /rename

Not that useful for now. I'll have to come back to it when I've done some more enumeration. 

* /torrent

Okay, so there was a lot more to play with here. We seem to have a fully fledge website, where I eventually discovered a login/register page.

{% capture fig_img %}
![Foo]({{ "/assets/images/popcorn/loginphp.png" | relative_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
</figure>


**Foothold/Exploitation**


So I created a 'test' user which I used to log in with. Because we know that files can be supposedly uploaded, I went directly to the upload section. I didn't accept the files I tested...

Do I actually have to upload a tor file? I mean, I guess that would actually make sense, considering the kali tor files I found.
The easiest thing to do, is to do the exact same thing right? So I head over to (Kali)[https://www.kali.org/get-kali/#kali-virtual-machines] and get myself some of the spicy iso files. 


<figure class="half">
    <a href="/assets/images/popcorn/kalitorrent.png"><img src="/assets/images/popcorn/kalitorrent.png"></a>
    <a href="/assets/images/popcorn/kalitor.png"><img src="/assets/images/popcorn/kalitor.png"></a>
    <figcaption>HTB</figcaption>
</figure>

I successfully uploaded the kali iso torrent file. And when I check out the settings of this new upload, I get an option to edit it. This option allows me to add a screenshot.
This is probably the best place to pop in a little reverse shell. 

Let's first test that the feature works. Puppy time!

<figure class="half">
    <a href="/assets/images/popcorn/editkalitor.png"><img src="/assets/images/popcorn/editkalitor.png"></a>
    <a href="/assets/images/popcorn/puppytest.png"><img src="/assets/images/popcorn/puppytest.png"></a>
    <figcaption>HTB</figcaption>
</figure>

Right, so that works. Let's try a simple php reverse shell, since the website is php based. 

<figure class="half">
    <a href="/assets/images/popcorn/reverselisten.png"><img src="/assets/images/popcorn/reverselisten.png"></a>
    <a href="/assets/images/popcorn/oopsfile.png"><img src="/assets/images/popcorn/oopsfile.png"></a>
    <figcaption>HTB</figcaption>
</figure>

Hmm...that didn't work.
That's okay though, so let's try it a little differently. But before we try, we need to look through the OP lens of Burp Suite because there is probably some sort of filtering going that I can't just counter with an extension change. 


<figure class="half">
    <a href="/assets/images/popcorn/burpintercept.png"><img src="/assets/images/popcorn/burpintercept.png"></a>
    <a href="/assets/images/popcorn/invalidburp.png"><img src="/assets/images/popcorn/invalidburp.png"></a>
    <figcaption>HTB</figcaption>
</figure>

Burp shows me a few filtering potentials here. The first obvious one is the extension and then the content-type header maybe. 
So let's manually change the content-type header in burp before forward the entire thingy. 

A quick look up on stackoverflow tells me the header I need is 'image/png', so let's try that!

<figure class="half">
    <a href="/assets/images/popcorn/newheader.png"><img src="/assets/images/popcorn/newheader.png"></a>
    <a href="/assets/images/popcorn/successupload.png"><img src="/assets/images/popcorn/successupload.png"></a>
    <figcaption>HTB</figcaption>
</figure>

It worked! Let's go (hopefully) activate the callback. 

<figure class="half">
    <a href="/assets/images/popcorn/listener.png"><img src="/assets/images/popcorn/listener.png"></a>
    <a href="/assets/images/popcorn/wwwdata.png"><img src="/assets/images/popcorn/wwwdata.png"></a>
    <figcaption>HTB</figcaption>
</figure>

Nice!


**Enumeration/Recon as www-data**


After some upgrading and exploring , we see a george directory.

<figure class="half">
    <a href="/assets/images/popcorn/explore.png"><img src="/assets/images/popcorn/explore.png"></a>
    <a href="/assets/images/popcorn/usertxt.png"><img src="/assets/images/popcorn/usertxt.png"></a>
    <figcaption>HTB</figcaption>
</figure>

Oh, and not only can we access it, but we can also read the user flag! Lovely.

Alright let's find out more about George since we'll most likely try to privesc into him. Let's also look for some hidden files.

<figure class="half">
    <a href="/assets/images/popcorn/whoisg.png"><img src="/assets/images/popcorn/whoisg.png"></a>
    <a href="/assets/images/popcorn/secrets.png"><img src="/assets/images/popcorn/secrets.png"></a>
    <figcaption>HTB</figcaption>
</figure>

Okayyy, so a few things jump to my face. First, 'sudo as admin successful'. Then the MOTD file. I remember both of those being options for privesc. Maybe...

I also see a mysql user in the /etc/passwd file, and some ssh directories. 
Many options.

Okay let's start with the sudo as admin file to see if there are any privesc techniques out there. 
So, no...I was misremembering I guess.


**Foothold/Exploitation to get Root**


For the motd file I do see some exploit. On exploit db too, so that's a good sign. Let's try it out. 

<figure class="half">
    <a href="/assets/images/popcorn/motdexploit.png"><img src="/assets/images/popcorn/motdexploit.png"></a>
    <a href="/assets/images/popcorn/motd2.png"><img src="/assets/images/popcorn/motd2.png"></a>
    <figcaption>HTB</figcaption>
</figure>

I was having issues using the tmp directory on popcorn, so I've moved to /dev/shm. 

-------------------

Wow, I am back and this took me a while with much debugging involved. 

The exploit doesn't seem to work at first glance since half way through the run it asks me for my current temporary www-data password. Which I don't have...
So after lots of googling and exploring the htb forums. I found someone explaining:

<figure class="half">
    <a href="/assets/images/popcorn/result1.png"><img src="/assets/images/popcorn/result1.png"></a>
    <a href="/assets/images/popcorn/result2.png"><img src="/assets/images/popcorn/result2.png"></a>
    <figcaption>HTB</figcaption>
</figure>

<figure class="half">
    <a href="/assets/images/popcorn/oof.png"><img src="/assets/images/popcorn/oof.png"></a>
    <a href="/assets/images/popcorn/difficult.png"><img src="/assets/images/popcorn/difficult.png"></a>
    <figcaption>HTB</figcaption>
</figure>

Nice. And then the exploit worked as described and got ourselves a nice root flag. 

Box pwned!