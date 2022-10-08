---
title: Nellis AFB InfoSec CTF
date: 2022-04-29 12:00:00 -0700
categories: [ctf]
tags: [network,snmp,web,hacking,linux] # TAG names should always be lowercase
---
# Nellis AFB InfoSec CTF 20220429
## "Attack & Respond"
---

#### Write-up writen by B4ndw1d7h.

------------------------------------------------

- #### First Task
	
	You will have two hours to break into 
	servers of a fictitious organization.

- #### Second Task
	
	You will have one and a half hours of 
	responding to a successful data breach 
	by examining packet captures, memory 
	dumps and hard drive images to extract 
	evidence, then answer questions about 
	the breach.

- #### Conclusion

	First to complete all assignments 
	will be the winner.

- #### Difficulty Level
	
	Intermediate skill level.

- #### Materials Needed
	
	1. Laptop
	2. Cyber Range environment will be 
	   provided by Infosec.

------------------------------------------------

- #### Prep
	
	Before the CTF I made sure to brush up 
	on these techniques and tools.

	1. Metasploit-Framework
		- Navigation and use
	2. Nmap
		- Scan types and 
		  vulnerability discovery
	3. Nikto
		- Web Enumeration
	4. GoBuster
		- Web Enumeration
	5. John the Ripper
		- Use
	6. Burpsuite
		- Use
	7. Autopsy
		- Use
	8. Searchspoit
		- Use
	9. Linux Privlage Escilation
		- Techniques and exploit 
		  discovery
	10. Windows Privlage Escilation
		- Techniques and exploit
		  discovery
	11. Linux System Enumeration
		- Manual system enumeration 
		  and automated enumeration
		  via linpeas.sh
	12. Windows System Enumeration
		- Manual system enumeration
		  and automated enumeration
		  via winpeas.exe
	13. Network Discovery
		- Nmap scripts and scans
	14. Exploitation
		- Metasploit exploitation 
		  and delivery methods
	15. Resource Gathering
		- Writeups, cheatsheets, 
		  and papers of different 
		  CTFs and CVE discoveries.
	16. TryHackMe
		- For general practice

--------------------------------------------------

- #### CTF Time!!!
###### Note
- All IP addresses, URLs, and wordlists used chaged for every user so they are intentialy left blank
		
#### How it went down:
- You had to log into a portal on infosec website and sign up for a 7 day free trial download their modified kali box start the ctf challenge 1

#### I unfortunatly didnt get this info until the ctf had started already :( 

## Round 1
---
#### 1st challenge
---
##### Find authoritative dns on a website:
- this was to see if we knew basic dns recon
- Solution: 
	I used 
	```bash
	dig +short NS <website url>
	```
	to pull up the dns records of the site given to us
	the flag was the soa of the website we where given
#### 2nd challenge 
---
##### Find domain owner:
- again similar to the first only the owner
- Solution:
	I used 
	```bash
	dig <website url> ANY
	```
	to pull up the owners recors of the same site as challenge 1
		the flag was the domain of the websites owner

#### 3rd challenge
---
##### Cracking snmp community string:
- This was new for me. I never heard of onesixtyone before, so I
	  had to google how this one worked. Luckly they had a small course, kind of like tryhackme, on infosec's site that helped walk you through similar senarios.
- Solution:
	I used 
	```bash
	onesixtyone <target ip> -c <location of wordlist> 
	```
	to find the default community string that was used. 
	Then entered the string as the flag.

#### 4th challenge
---
##### SNMP cracking/ find what type of ftp server is running on the box:
- Same as the first. This was new to me, but infosec also had a small course on how this worked as well for me to understand how this worked.
- Solution:
	I used 
	```bash
	snmpwalk -v 2c -c <community string> <target ip> | more 
	```
	to find the process running the ftp server (Filezilla) on the box. Filezilla was the flag.


## Round 2
---
#### 1st challenge 
---
##### Investigating port services:
-  I felt confident with this section having preped with nmap prior, so this was rather simple for me to grasp. They said there was something on port 12345 and to investigate what it is
- Solution:
	I used	
	```bash
	 nmap -p 12345 <website url> 
	```
	to find netbus on port 12345 and determined it is used as 
	a backdoor for a trojan.
	The flag was backdoor.

#### 2nd challenge
---
##### Service identification:
- Again simple nmap scanning to find the version of ftp running
- Solution:
	I found ftp version and number using nmap's -sV option durring a scan
	vsftpd 3.0.2 was the flag

#### 3rd challenge
---
##### Getting first password:
- Here they wanted us to login to the ftp server using the username=admin
- Solution:
	I used metasploit to bruteforce the password to the ftp server using
	auxiliary/scanner/ftp/ftp_login and used the snmp password list that
	comes with metasploit
	password and flag was admin123

#### 4th challenge 
---
##### Getting second password:
- I did not know how to find it and still do not understand how to find the flag. However the competetor next to me found it in a script localy on the system, in a script before he got the 3rd flag XD. He then asked me if I wanted to make a trade deal to show him how I solved the 3rd challenge for the flag here. XD Teamwork makes the dream work!
- Solution:
	It was in a script called "backup" deep on box


---
## Round 3
---
#### 1st challenge:
---
##### Web exploitation:
- They said to skip this section sadly due to server malfunctions :(

## Round 4
#### 1st challenge
---
##### User privlage:
- We needed to find a root exicutible file on the system
- Solution:
	I used sudo -l to see what the user could run as root and found a script
	in /usr/bin/scripts/ that ran as root when executed.
	The flag was in the script. 
	The script just ran other scripts as root.

## 2nd challenge
---
##### Find a root owned file that the script exicutes:
- The previous script exicutes multiple random files however one of them we could write to.
- Solution:
	The flag was the name of the file

## 3rd final challenge
---
##### What is the root password:
- After finding the script that root executes with no password needed I simply edited it to give me the root hash from etc/shadow. 
- Solution: 
	I then used john the ripper to crack the hash. After cracking the hash I enterd the password for the flag

---

# And that was all!!
Unfortunatly I only came in second due to not being the first to answer all in the first round and tying on round 2, but it was a great learning experience! Definatly will check out infosec's courses on new material to learn.

We could not do the second portion of the CTF due to time running out however they gave us a pdf to go learn it and do on our own time. The representative unfortunatly said we cannot distribute the info on the second portion.
