---
title: Vulnhub Infosec-OSCP
date: 2022-05-16 12:00:00 -0700
categories: [ctf]
tags: [hacking,writeup,web,linux] # TAG names should always be lowercase
---

# Infosec OSCP Box
---
This box was pretty simple to solve. The only issue I ran into was Privesc but that is do to my inexperience.

First things first I start with an nmap scan to see what ports are open, along with gobuster just in-case to start the enumeration.
```bash
nmap -sC -sV -oN nmap/initial $IP

gobuster dir -u http://$IP -w wordlist.txt -x .cgi,.txt,.php,.html,.py,.sh,.jpg,.png -t 200
```
During the enumeration I see a robots.txt and see /secret.txt.

I go to it and find a base64 encoded string.
I decode it and get an openssh private key.
```bash
base64 -d encoded.txt > decode.txt
```
i then use the key to access the box with the user "oscp" that is called out in the main page of the website.

once in i immediately upload linpeas for my enumeration.
```bash
python3 -m http.server
```
I see that the SUID bit is set on /usr/bin/bash. 
That's my way in.
I then go to GTFObins and see all I need to do is use -p and I'm root(kinda embarrassing lol).

I then navigate to the /root dir and cat out the flag.
