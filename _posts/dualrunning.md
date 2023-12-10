---
title: Dual Running - Traffic Analysis
date: 2023-03-29 12:00:00 -0700
categories: [traffic analysis]
tags: [blueteam,wireshark,pcap,networking] # TAG names should always be lowercase
---

[Malware Traffic Analysis](https://www.malware-traffic-analysis.net/)



# TASK:
---
- Write an incident report based on the pcap and alerts.
- The incident report should contain the following:
	- Executive Summary
	- Details (of the infected Windows host)
	- Indicators of Compromise (IOCs)


---

# Intro
---
Welcome back! Still trying to work out a good writing flow to keep things entertaining and readable.(Not an English major if you can't tell) So thank you so much for sticking with it!
Today I will be going over another [Malware Traffic Analysis](https://www.malware-traffic-analysis.net/) PCAP from my backlog. I don't do spoilers so go pull down your traces from [here](https://www.malware-traffic-analysis.net/2021/07/14/index.html)and follow along!

# Investigation time!
---
with this excersise we are given an image and a text file containing allerts that have poped up on our siem or dashboard. given this we can start looking in to the potential intrusion.

the very first thing we see is that there was an HTTP request to a non-standard port. going into wireshark we can filter for all the traffic going to and from that port with the following
- `tcp.port == 8088`
from here we can look at all the connections made using this port such as the following:
- GET /wp-content/Receipt-9650354.xls?evagk=2MyeEdhGPszYX HTTP/1.1\r\n - insiderushings.com - port 8088 
- GET /templates/file6.bin HTTP/1.1\r\n - buyer-remindment.com - port 8088
we see in the alerts and in wireshark that the first file downloaded was an ms-excel file containing VBA code. this alone is not bad however the files content flags many signatures indicating this could be malicious. following this another file was downloaded showing signs of being a strain of the Dridex malware.

if you are not familiar with Dridex malware feel free to read [this](). if you want the TL;DR it primarily targets financial services such as banks and payment. it will 

# Conclusion
---

# Report
---