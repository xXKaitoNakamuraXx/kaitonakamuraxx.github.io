---
title: THM Red Team Fundamentals Overview 
date: 2022-09-30 12:00:00 -0700
categories: [tryhackme,info]
tags: [writeup,hacking,red-team] # TAG names should always be lowercase
---

# Red Team Fundamentals Overview
---
Hello! For this post I will be giving an overview of the content provided in the first section of the new Red Team pathway - Red Team Fundamentals. If you want the TL;DR, It was very informative and if you don't know anything about the red team you will learn a lot! 

Now to kick it off with the first room Red Team Fundamentals.


## Red Team Fundamentals
---
My thoughts and writeups on new Red Team pathway from [TryHackMe](https://tryhackme.com/paths) Red Team fundamentals. Here we will be discussing the information associated with the fundamentals of breaking in (legally).

So what is a Red Teamer and what do they do?

A Red Team is the internal or external group that will perform penetration test on a network to ensure compliance, physical security, and bring to light the holes in the network the blue team might not have seen.

Objectives from the room are to explain in broad detail the basics of what the Red Team does.

The info provided in this room was very informative. The steps, roles, and overall structure of how a red team operates was great to get insight on and what would be required if you where to enter the field.



## Red Team Engagements 
---
Here we will be going more in depth on Red Team engagements touched on in the last section.

This room outlined the types of engagements, rules, plans, and documentation of said engagements.

Here we learned about what a "Scope" is and the difference between the 2 main types of engagements. The main takeaway was the engagements for me. From going through the Cyber Defense pathway i learned how the blue team  can build their security based on the TTP's and knowing this the red team can test if the counter measures are viable within the given rules put in place by the company or customer. Breaking down the planing faze helped me understand the "upgrade" from CTF to enterprise tests. outlining the documentation that will be provided to the customer after the pentest and the variations there of



## Red Team Threat Intel
---
Here we will go over the Threat Intel room.
Coming from a military background this room was very familiar to me. we often have exercises that consist of simulated adversary chemical attacks, so the cyber implementation of learning the attacks of  the adversary and building plans and tactics around them using frameworks like MITRE ATT&CK for intel gathering along with logs is a lot of fun.


## Intro To C2
---
This was my favorite of the 5. While I knew what C2 did, I did not know all the in depth info that was provided. Now i will say that on Task 6 i was having a bit of trouble. As much as I wanted to learn how to use Armitage I could not figure out how to set it up on my machine nor use the provided one in the Attackbox. the error i consistently received forums said that it was deprecated and that could be the reason it would not load or install properly. I ultimately decided to go ahead and just use what i know best and go old school CLI and just use metasploit. if you would like to see what i did to get around it please continue if you don't want spoilers continue to the next section.

the first thing i did as it stated in the task was an nmap scan of the IP to see what services where open.
![[nmap.png]]
i see that smb is available and that the os is win7 and the machines name was ted-pc. while reading it mentioned eternal blue, and having exploited this vulnerability before in the blue room ctf.

once exploited i dumped the user hashes to answer the questions by using
command

![[meterpreter.png]]

and for the user flags we can spawn a shell with meterpreter as system and get both located in their respective desktop directories

![[root-flag.png]]

![[user-flag.png]]

# Conclusion
---
And so concludes my thoughts on the start of this pathway. I highly recommend anyone who has an interest in Red Teaming or Cyber Security in general take a look at this path and other rooms on [TryHackMe](https://tryhackme.com)!