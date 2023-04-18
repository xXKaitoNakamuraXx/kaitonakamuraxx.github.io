---
title: Angry Poutine - Traffic Analysis
date: 2023-03-29 12:00:00 -0700
categories: [traffic analysis]
tags: [blueteam,wireshark,pcap,networking] # TAG names should always be lowercase
---

Welcome to the start of my Traffic Analysis training! I'm wanting to do this not only because its new to me and I find it interesting, but also to develop hands-on experience and practice the concepts and procedures for being an analyst. 

The site I will be using to get PCAPS for the practice analysis is [Malware Traffic Analysis](https://www.malware-traffic-analysis.net/) . If you want to follow along with the specific PCAP I will be using you can find it [here](https://www.malware-traffic-analysis.net/2021/09/10/index.html)

Now onto the task!!

---
Side note:
Unfortunately the task objectives are right above the answers so just don't peek down at them ;P
For reference I've written them down here.


# Task
---
• Write an incident report based on the PCAP and alerts. 
• The incident report should contain the following: 
	• Executive Summary 
	• Details (of the infected Windows host) 
	• Indicators of Compromise (IOCs) 


# Steps Taken
---

Given the tasks I now open the PCAP in Wireshark. 

![[assets/images/pcap-images/angry-poutine/initial-wireshark.png]]

1. First thing i did is go to the statistics tab > conversations, to get a view of relevant traffic.
	I start looking at the info in the TCP tab, and see many IPs that are using port 80. I decide to start their due to traffic nowadays normally being sent over port 443 using SSL/TLS.
	
![[assets/images/pcap-images/angry-poutine/convo-screen.png]]

2. I sort the traffic out using the amount of packets, and begin on the one with the most traffic. Taking a glance looking at the lists for anything that would stick out like GET or POST requests I saw something odd that narrowed my search down to this IP: 
	- 194.62.42.206
	What caught my eye was a GET request to a site: 
	
```url
http://simpsonsavingss.com/bmdff/BhoHsCtZ/MLdmpfjaX/5uFG3Dz7yt/date1?BNLv65=pAAS
```

3. Copying the URL I put the URL into [Virustotal](https://www.virustotal.com/gui/url/9461958e723e10c73e2f423c591f3ad04a35b4acbee6f1b48da68c1ee8aada31/details) and got hits for it being malicious.
	We now know that it is fairly certain that this URL is malicious in some form or fashion.

4. With this information I go to the main Wireshark page and click View > Time Display Format > UTC Time of Day. To change the time format to UTC ( just to make it more comprehensible for others.) and note the time and date of the event. 
	- Date: Fri, 10 Sep 2021 23:17:14 GMT

5. I then start filtering for the hosts information using the IP address of the victim, found while looking at the packet that made the get request, and filters for the following: 
- kerberos.CNameString for username and host name information (could have used nbns traffic to find the hostname as well)

![[assets/images/pcap-images/angry-poutine/kerberos-cnamestring.png]]

- useragent string in the GET request for the host OS and MAC(Windows NT 10.0 aka Windows 10)

![[assets/images/pcap-images/angry-poutine/useragent.png]]

6. Once I gathered all the host's info I extracted the file by going to File > Export Objects > HTTP ..., selected the file associated with the URL found previously and grabbed an md5sum of it using the commandline .

```bash
md5sum 'date1?BNLv65=pAAS'

eed363fc4af7a9070d69340592dcab7c78db4f90710357de29e3b624aa957cf8
```

7. I then went back to Virustotal and checked the file hash and found the following information.
	[File Info from Virustotal.com](https://www.virustotal.com/gui/file/eed363fc4af7a9070d69340592dcab7c78db4f90710357de29e3b624aa957cf8/detection)

	File type
	- Win32 DLL
	
	Potential IP Traffic on our victim system for C2 communication according to virus total known IPs
	-   167.172.37.9:443 (TCP)
	-   94.158.245.52:443 (TCP)
	
	Name of Malware:
	- trojan.kryplod/bazar

8. Looking back at the TCP tab under conversations I see that 2 IPs mentioned in the virus total report are in fact talking back and forth in the PCAP.

![[assets/images/pcap-images/angry-poutine/mal-ips.png]]

This is pretty much all for this analysis. Here is how the following report might look.
(Never did this before so I had [ChatGPT](https://chat.openai.com/chat) draft a template for me to fill out LOL. Amazing resource BTW ;P)



---
---
# Bazar C2 Infection
## Date: 2023-03-30

### Overview:
---
The purpose of this report is to outline the findings of a malware analysis and provide recommendations for mitigation. The malware was discovered during a PCAP analysis and was identified as a security threat.


### Findings
---

#### Executive Summary
---
On Fri, 10 Sep 2021 23:17:14 GMT, a Windows 10 system used by Hobart Gunnarsson was infected with BazarC2 malware

Details:
1. Malware Type: Malicious DLL Trojan
2. Malware Name: Bazar
3. IP Address of Infected Machine(s): 10.9.10.102
4. Infection Vector: HTTP GET Request


#### Details (of the infected Windows host) 
---
- MAC: 00:4f:49:b1:e8:c3
- Host name: DESKTOP-KKITB6Q
- Host IP: 10.9.10.102
- Host OS: Windows 10
-  Username: hobart.gunnarsson


#### Indicators of Compromise (IOCs) 
---
194.62.42.206:80 - simpsonsavingss.com - downloaded /bmdff/BhoHsCtZ/MLdmpfjaX/5uFG3Dz7yt/date1?BNLv65=pAAS

Virustotal output for the malicious URL [here](https://www.virustotal.com/gui/url/9461958e723e10c73e2f423c591f3ad04a35b4acbee6f1b48da68c1ee8aada31/detection)

Active C2 communications over 
- 167.172.37.9:443 - HTTPS Traffic
- 94.158.245.52:443 - HTTPS Traffic

Virustotal output for the downloaded malicious file [here](https://www.virustotal.com/gui/file/eed363fc4af7a9070d69340592dcab7c78db4f90710357de29e3b624aa957cf8/detection)


### Notes
---

The malware was analyzed using manual techniques with Wireshark. The analysis revealed that the malware was designed to maintain persistence and elevate privileges to establish secure C2 connections. The malware communicated with a C2 server located at:
- 167.172.37.9:443 
- 94.158.245.52:443
Sending encrypted data over the network.

![[assets/images/pcap-images/angry-poutine/virustotal-site.png]]

![[assets/images/pcap-images/angry-poutine/virustotal-app-hash.png]]

![[assets/images/pcap-images/angry-poutine/mal-ips.png]]

Recommendations:

1. Quarantine infected machines: The infected machines should be removed from the network to prevent further spread of the malware.
2. Conduct a forensic analysis: A forensic analysis should be conducted on the infected machines to determine the extent of the infection and identify any other potential security risks.
3. Update antivirus software: Ensure that antivirus software is up-to-date and all machines on the network are scanned for malware.
4. Update Firewall rules to blacklist traffic to C2 IPs
5. Educate users: Educate users on how to identify and avoid potential malware infection vectors such as phishing emails, click jacking, and suspicious attachments.

Conclusion:

The presence of Bazar on the network is a significant security threat. Immediate action should be taken to prevent further spread and mitigate any damage caused by the malware. By following the recommendations outlined in this report, the organization can effectively mitigate the risk of future malware infections and strengthen its overall security posture.

---

And so concludes my first part in a series of Traffic Analysis Posts. Hope you enjoyed!