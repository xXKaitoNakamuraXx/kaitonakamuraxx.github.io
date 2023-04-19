---
title: Binary Reverse Engineering (Cracking inhouse software)
date: 2023-02-09 12:00:00 -0700
categories: [realworld]
tags: [hacking,windows,binary] # TAG names should always be lowercase
---

## Where have i been?
---
First apologies for the 4 month hiatus. Family things you know. Any who on to the main course!

# Binary Reverse Engineering
---


In this post, I'll be going over the process I used to reverse engineer a program I was given at work. Unfortunately I can only show select screenshots from the process due to confidentiality purposes.

The program was an old in-house build from over a decade ago, running on a Windows 7 PC in a corner. The person who had made it abandoned it after locking it, out of spite as a last "Screw you" to their supervisor.

The issue was that a patch was pushed to the program, requiring a registration key upon execution and not allowing users to log in. For the past decade, the shop had been reduced to using Excel and a notepad for logging and monitoring the in-house job. When I came in, I was told about this program that would automate the entire job and track everything.

I offered to take a look at the program and see if I could do anything about it. I had no experience whatsoever in reverse engineering or messing with assembly or coding in C. It was a full-on "unknown unknown" for me. Between family life, work, and studying, I poked around and tried different things with the program. Here was my general workflow over a year on what I did.

First, I pulled a copy of the program from work. Then, I ran strings on the program using different encoding methods to see if anything would come out. I saw no hard-coded registration key tho I did find a few helpful strings.

![](/assets/images/batt-program/batt-strings.png)
And
![](/assets/images/batt-program/batt-strings-login-creds.png)

Seeing these I figured that their is a way to bypass the check somehow for the key and the credentials for the Admin user was "12345".

Armed with this, I proceeded in finding out how this would work. I heard about Ghidra and how it is used to reverse engineer programs. I instantly tried to run this against it, but I was completely lost and had no idea what I was doing.

Over time, I fiddled around and explored the different functions and what all I could do. I learned how to filter the strings and look through functions, etc. However, I had never coded in C other than an Arduino script for a few robotics projects, and I had never looked at or messed with assembly in my life.

After mindlessly looking through the assembly and functions that Ghidra was able to parse out, I remembered from the strings command earlier that I saw some things that were describing a registration key, and completing the registration. I thought to myself, "I found it!!" and then had no idea where to go from there.

![](/assets/images/batt-program/ghidra-success-calls.png)

When hovering over the calls it showed me the function associated with it.

![](/assets/images/batt-program/ghidra-popup.png)

Clicking it it brought me to that section of the code and i began to look around and try and make sense of it.

After watching a few YouTube videos, I learned that there was a way to edit the command to have it execute different things. Again, I had no idea how to write assembly or really read it. I just used context clues as to what things might mean and from prior experience with software development, web exploitation and using null bytes, I figured this could work here.

I tried it on multiple different calls, exporting the program each time, attempting to run it and see if I had done it. After the 8th or 20th attempt at changing the calls to "PUSH 0x0," which I thought was a null input, I found the correct part in memory that does the validation and successfully bypassed the registration key check, enabling the program to finally be used.

I had to change this :

![](/assets/images/batt-program/ghidra-code-before.png)

To this :

![](/assets/images/batt-program/ghidra-code-after.png)

I know the better solution would be to rewrite a new management software. However, as the system is fully offline and there is no viable threat, it will be fine. Plus, this gave me a whole new skill to learn more about, along with massive respect from higher-ups.

This concludes my overview of cracking some old in-house software.

EDIT: After a few days of them trying to use it. They found out that it was to outdated and buggy in how it worked. I ultimately got recruited to rewrite similar software from scratch. Write-up coming soon on my process!



