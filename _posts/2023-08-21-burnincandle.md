---
title: Burn in Candle - Traffic Analysis
date: 2023-08-21 12:00:00 -0700
categories:
  - traffic analysis
tags:
  - blueteam
  - wireshark
  - pcap
  - networking
---
## Where have I been?
---
Apologies for the long silence. Lots of personal life stuff hit and had to take a break from my posts (even tho their aren't many). Any who I haven't just stopped my learning! When I had some free time I started learning more on Ansible and docker, did the google cyber security cert, and have been teaching fellow DoD members some networking and stuff. Any who enough with the excuses lets get to it!

## Topic
___
The site I will be using to get PCAPs for the practice analysis is [Malware Traffic Analysis](https://www.malware-traffic-analysis.net/) . If you want to follow along with the specific PCAP I will be using you can find it [here](https://www.malware-traffic-analysis.net/2022/03/21/index3.html), and for my coloring rules you can either take them from my previous post here, or from my Github.

Today I will be sharing another Threat Hunting/Traffic Analysis post and some of the methodology I have been building in helping me find IOCs.

### TASK
---
- Write an incident report based on traffic from the pcap. 

## Steps Taken
---
So, here's what I found: There was a strange GET request to 188.166.154.118 on port 80, heading to the domain oceriesfornot.top. What stood out was that it used the ".top" domain, and it only had a cookie with system info.

I did some more digging and ran the domain and IP through VirusTotal. Turns out, both were marked as suspicious. Digging deeper, I found the downloaded file was Icedid malware, a sneaky remote access Trojan. I read up on it on [this site](https://www.blackberry.com/us/en/solutions/endpoint-security/ransomware-protection/icedid), and it's one of those second-stage threats. The search for the dropper isn't over yet.

Before jumping back in, I noted down details about the infected system for future reference:

Host info 
IP: 10.0.19.14 
MAC: 00:60:52:b7:33:0f 
Device Name: DESKTOP-5QS3D5D ?
User: patrick.zimmerman

Going back to the PCAP, after the previous action, a new connection popped up via HTTPS to IP "157.245.142.66" linked to antnosience.com.

Turns out, this IP didn't pass VirusTotal's test either. Plus, another domain, otectagain.top, was flagged.

Skimming through more packets, I saw a connection to IP "160.153.32.99," which is connected to the domain suncoastpinball.com.

The next part gets messy: More action from the previous IP, along with a lot of messy stuff from suncoastpinball.com. Think duplicate acknowledgments, out-of-order stuff, and reset packets. What's interesting is that the infected system asked the domain controller for an endpoint map and wanted to use LSARPC. Basically, it's like asking for permission for remote actions. This hints at the potential for NTLM Relay Attacks on the Active Directory (AD) Certificate Services.

At 13:59:40 PDT, something significant happened: The infected system successfully logged onto the SMB server hosted by our domain controller at 10.0.19.9.

Then it started hunting for backup file versions on the server, probably to grab data. At the same time, it went on a domain expedition to figure out the network.

As time went on, encrypted chats started between the infected system and the bad guys at 157.245.142.66 and 160.153.32.99 – they were up to some data sneaking.

The story kept going, with more back-and-forths between the infected system and the domain controller, plus calls to those known bad IPs and ongoing SMB connections.

At 16:55:55 PDT, the host started looking into the domain controller's policies through SMB traffic.

Hold on, there's a new connection! This time, it's targeting "seaskysafe.com" (IP: "91.193.16.181"). VirusTotal said it's a no-go zone, tagged as spyware, malware, and part of a bot network.

There's more: The IP "157.245.142.66" changed domains to otectagain.top, and 91.193.16.181 switched to "dilimoretast.com."

With all these domain changes, I took a closer look at the DNS requests and responses. Three domains caught my attention: "filebin.net" (185.47.40.36) seems to be a source or storage for infections; "situla.bitbit.net" (87.238.33.8 and 87.238.33.7) might be a proxy; and "bupdater.com" (23.227.198.203) is a sign of Cobalt Strike mischief.

Interestingly, "filebin.net" and "situla.bitbit.net" appeared during SMB chats, hinting at data leaving the scene.

Finally, towards the end, the calls went to "bupdater.com" for some sneaky persistence.

I kept digging through this PCAP a few more times just to figure out what tricks this malware has up its sleeve on the network side. I also did some extra research and went deeper into understanding how it could sneak past the AD Certificate Service and use SMB to grab config info and other credentials from the DC. It's pretty cool stuff, and I'm definitely planning to give some of these AD attacks a shot in my lab!

So, there you have it – Thanks for reading! See you in the next one!

## Report
---
### Executive Summary
---
At approximately 13:58:11 PDT on Mar 21, 2022, a Windows host operated by the user Patrick Zimmerman showed signs of infection from IcedID malware with data being stolen via File sharing sites and persistence with Cobalt Strike.


### Details 
---

1. Host info
	1. IP: 10.0.19.14
	2. MAC: 00:60:52:b7:33:0f
	3. Device Name: DESKTOP-5QS3D5D
	4. User: patrick.zimmerman
	5. Domain Controller: BURNINCANDLE DC.burnincandle.com
	6. DC IP: 10.0.19.9


### IOCs
---
According to [VirusTotal](https://virustotal.com) and [Talos Inteligence](https://talosintelligence.com/)

icedid malware
1. 188.166.154.118 - port 80 - oceriesforno t.top - GET /
2. 157.245.142.66 - port 443 - antnosience.com / otectagain.top - HTTPS Traffic
3. 160.153.32.99 - port 443 - suncoastpinball.com - HTTPS Traffic
4. 91.193.16.181 - port 443 - seaskysafe.com / dilimoretast.com - HTTPS Traffic

suspicious file sharing sites
1. 185.47.40.36 - filebin.net - infection source/data storage
2. 87.238.33.8 - 87.238.33.7 - situla.bitbit.net - proxy

C2 traffic
1. 23.227.198.203 - bupdater.com -  malicious site/IOC for Cobalt Strike

## Notes
---
Icedid malware infection.
![](/assets/images/pcap-images/burnincandle/initial-infection.png)
![](/assets/images/pcap-images/burnincandle/vt-initial.png)
![](/assets/images/pcap-images/burnincandle/vt-1.png)
![](/assets/images/pcap-images/burnincandle/vt-2.png)
![](/assets/images/pcap-images/burnincandle/vt-3.png)

SMB logon and data gathering attempts
![](/assets/images/pcap-images/burnincandle/malicious-smb-traffic.png)

file exfiltration
![](/assets/images/pcap-images/burnincandle/filebin-traffic.png)
![](/assets/images/pcap-images/burnincandle/situla-traffic.png)
![](/assets/images/pcap-images/burnincandle/vt-file-share.png)

Cobalt Strike connection
![](/assets/images/pcap-images/burnincandle/cobalt-strike.png)
![](/assets/images/pcap-images/burnincandle/vt-cobalt.png)


### Conclusion
---
Recommendations:

1. Quarantine infected machines: The infected machines should be removed from the network to prevent further spread of the malware.
2. Conduct a forensic analysis: A forensic analysis should be conducted on the infected machines to determine the extent of the infection and identify any other potential security risks.
3. Update antivirus software: Ensure that antivirus software is up-to-date and all machines on the network are scanned for malware.
4. Update Firewall rules to blacklist traffic to C2 and malicious IPs
5. Educate users: Educate users on how to identify and avoid potential malware infection vectors such as phishing emails, click jacking, and suspicious attachments.

Conclusion:

The presence of IcedID and Cobalt Strike on the network is a significant security threat. Immediate action should be taken to prevent further spread and mitigate any damage caused by the malware. By following the recommendations outlined in this report, the organization can effectively mitigate the risk of future malware infections and strengthen its overall security posture.