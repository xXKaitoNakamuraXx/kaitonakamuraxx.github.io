---
title: THM Vulnversity Writeup
date: 2022-05-20 12:00:00 -0700
categories: [tryhackme,ctf]
tags: [thm,ctf,writeup,hacking] # TAG names should always be lowercase
---
# Vulnversity write-up
---

### Task
**Recon**
	use nmap to solve 
```bash
nmap -sV -n $IP
```

**Directory traversal**
	use gobuster to solve
```bash
gobuster dir -u http://$IP:3333 -w /path/to/wordlist
```

**Compromise web-server**
	use burpsuite to solve
 - **Steps**
	- go to proxy tab
	- click open browser
	- navigate to http://$IP:3333/internal/
	- turn intercept on
	- try to upload a [[SecLists/Web-Shells/laudanum-0.8/php/php-reverse-shell.php|php-reverse-shell]]
	- in the request change the file extetion to .phtml
	- forward the request
	- in a terminal set up a listener
	```bash
nc -lnvp 1234
	```
	- now navigate to http://$IP:3333/internal/uploads/php-reverse-shell.phtml
	- go to your terminal and see you have a shell
	- now stabilize the shell with python
	```python
python -c 'import pty; pty.spawn("/bin/bash")'
	```
	- export a terminal
	```bash
export TERM=xterm
	```
	- now view the home directory to get an answer
	- cat out the flag.txt for the next answer

**Privlage Escalation**
	this one was simple but i'm an idiot
- **Steps**
	- enumerate the system 
	- when checking SUIDs find them with this
	```bash
find / -perm -4000 2>/dev/null
	```
	- you'll see systemctl is usefull
	- check out [GTFOBins](https://gtfobins.github.io) 
	- change the "id > /tmp/output" to a reverse shell or just have it cat the next flag
	- **Don't make it one line, it will be a formatting error for systemctl !!**
	- go to /root/ and read root.txt for the final flag