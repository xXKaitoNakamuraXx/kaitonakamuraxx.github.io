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
First thing I like to do is check if any files have been downloaded to see if anything sticks out to start. Luckily here we see some files:
- insiderushings.com - port 8088 - Receipt-9650354.xls?evagk=2MyeEdhGPszYX 
- buyer-remindment.com - port 8088 - file6.bin
![](/assets/images/pcap-images/dualrunning/files.png)

pulling these files down from the PCAP I hash them using md5sum and sha256sum, and test them against  [VirusTotal](https://virustotal.com) and [Talos Inteligence](https://talosintelligence.com/) just as a quick scan to see if they are malicious.



# Conclusion
---

# Report
---