---
title: Asco Limited - Traffic Analysis
date: 2023-05-15 12:00:00 -0700
categories: [traffic analysis]
tags: [blueteam,wireshark,pcap,networking] # TAG names should always be lowercase
---

The site I will be using to get PCAPS for the practice analysis is [Malware Traffic Analysis](https://www.malware-traffic-analysis.net/) . If you want to follow along with the specific PCAP I will be using you can find it [here](https://www.malware-traffic-analysis.net/2021/02/08/index.html)

Today I will be sharing another Threat Hunting/Traffic Analysis post and some of the methodology I have been building in helping me find IOCs.


## TASK: 
---
• Write an incident report based on the pcap and alerts. 
• The incident report should contain the following: 
	• Executive Summary 
	• Details (of the infected Windows host) 
	• Indicators of Compromise (IOCs) 

## Steps Taken
---
For this capture, I started off how I normally would, trying to get a full overview of the capture in Wireshark.
To help me with this, I created a few coloring rules to spot specific types of traffic that could be considered malicious or odd.

- TCP Handshake : ```tcp.flags.syn == 1```

- IP Lookup : ```dns.qry.name contains "ip"```
- Client Hello : ```tls.handshake.type == 1```
- Filtered GET/POST Requests : ```(http.request.method == "GET" ) && !(http.request.uri contains "/msdownload/update/") || (http.request.method == "POST" )```
- Host Discovery : ```(icmp.type == 3 && icmp.code == 2) || (icmp.type == 8 || icmp.type == 0) || (tcp.dstport == 7) || (udp.dstport == 7) || arp.dst.hw_mac == 00:00:00:00:00:00```
- Nmap Scans : ```(tcp.flags.syn == 1 && tcp.flags.ack == 0 && tcp.window_size<=1024) || (tcp.flags.syn == 1 && tcp.flags.ack == 0 && tcp.window_size>1024) || (tcp.flags == 0) || (tcp.flags == 0x001) || (tcp.flags.fin == 1 && tcp.flags.push == 1&&tcp.flags.urg == 1) || (icmp.type == 3 && icmp.code == 3)```
- Network Attacks : ```(arp.duplicate-address-detected || arp.duplicate-address-frame) || (icmp && data.len > 48) || (dtp || vlan.too_many_tags) || (tcp.analysis.lost_segment || tcp.analysis.retransmission)```gi

These rules should help with your task. Right away, I could see the external IP lookup and started exploring it further.
![[/assets/images/pcap-images/ascolimited/ws-ip-lookup.png]]

I started seeing strange GET/POST requests that were shown alongside an ARP scan looking for hosts.
GET/POST requests
![[/assets/images/pcap-images/ascolimited/ws-get-post.png]]
ARP Scan
![[/assets/images/pcap-images/ascolimited/ws-arp-scan.png]]

I exported the downloaded files and ran them through VirusTotal and AlienVault. While some files did not trigger an alert, others did. As a result, any files downloaded from the same source became suspicious. Here are the following hashes from the exported files.
![[/assets/images/pcap-images/ascolimited/file-hashes.png]]

After completing the manual analysis, I like to go through the process again using IDS/IPS solutions and create rules to detect the identified threats or check for pre-existing rules to understand how they are detected.

Using Brim, a data management tool similar to Splunk, with its default Suricata alerts, it instantly detected and alerted me to the malicious traffic.

Hancitor traffic
![[/assets/images/pcap-images/ascolimited/hanicator.png]]

Flicker Stealer traffic
![[/assets/images/pcap-images/ascolimited/flicker-stealer.png]]

Cobalt Strike traffic
![[/assets/images/pcap-images/ascolimited/cobalt-strike.png]]

After further research, I identified the matches between the traffic and malware. I proceeded to fingerprint and match them to the MITRE ATT&CK framework.

Here is my completed report on the subject.

## Executive Summary
---
At approximately 16:00 UTC on 2021-02-08, a Windows host operated by the user Bill Cook showed signs of infection from three types of malware: Hancitor, Cobalt Strike, and Flicker Stealer.


## Details 
---
1. Details of the infected host:
	1. MAC: 00:12:79:41:c2:aa
	2. IP Address: 10.2.8.101 
	3. Hostname: DESKTOP-MGVG60Z
	4. Account Name: bill.cook
	5. Domain Controller: AscoLimited-DC.ascolimited.com


## IOCs
---
According to [VirusTotal](https://virustotal.com) and [AlienVault](https://otx.alienvault.com/)

Hancitor traffic
1. 213.5.229.12 - port 80 - satursed.com - POST request for forum.php containing host information - 2021-02-08 16:00:10
2. 5.124.85.55 - port 80 - tonmatdoanminh.com - GET request for uninviting.php gzip file - 2021-02-08 15:59:12

Flicker stealer traffic
1. 8.208.10.147 - port 80 - roanokemortgages.com - GET requests for /0801.bin, /0801s.bin, /6lhjgfdghj.exe - 2021-02-08 16:00:12
2.  185.100.65.29 - port 80 - strange traffic being sent back and forth - 2021-02-08 16:00:17

Cobalt strike traffic
1. 198.211.10.238 - port 8080 - 198.211.10.238:8080 - GET requests for /6Aov, /ca - 2021-02-08 16:00:12
2. 198.211.10.238 - port 443 - HTTPS traffic - 2021-02-08 16:00:13
3. 198.211.10.238 - port 8080 - 198.211.10.238:8080 - Post system info to submit.php?id=3275377518 - 2021-02-08 16:01:49

Other indicators
1. 54.235.147.252 - port 80 - api.ipify.org - External IP lookup - 2021-02-08 16:00:06
2. 10.2.8.101 - ARP Scan for host discovery starting with ICMP traffic to DNS server at 10.2.8.2 and gateway at 10.2.8.1 - 2021-02-08 16:01:31


## Notes
---
Given alerts from excersize:
![[/assets/images/pcap-images/ascolimited/2021-02-08-traffic-analysis-exercise-alerts.jpg]]

#### VirusTotal Details
Cobalt strike:
![[/assets/images/pcap-images/ascolimited/6Aov_virustotal.png]]

Flicker Stealer:
![[/assets/images/pcap-images/ascolimited/6lhjgfdghj-virustotal.png]]

Hancitor:
![[/assets/images/pcap-images/ascolimited/vt-hancitor.png]]

#### Generated alerts from Brim
Hancitor traffic
![[/assets/images/pcap-images/ascolimited/hanicator.png]]

Flicker Stealer traffic
![[/assets/images/pcap-images/ascolimited/flicker-stealer.png]]

Cobalt Strike traffic
![[/assets/images/pcap-images/ascolimited/cobalt-strike.png]]

## Conclusion
---
Recommendations:

1. Quarantine infected machines: The infected machines should be removed from the network to prevent further spread of the malware.
2. Conduct a forensic analysis: A forensic analysis should be conducted on the infected machines to determine the extent of the infection and identify any other potential security risks.
3. Update antivirus software: Ensure that antivirus software is up-to-date and all machines on the network are scanned for malware.
4. Update Firewall rules to blacklist traffic to C2 and malicious IPs
5. Educate users: Educate users on how to identify and avoid potential malware infection vectors such as phishing emails, click jacking, and suspicious attachments.

Conclusion:

The presence of Hancitor, Cobalt Strike, and Flicker Stealer on the network is a significant security threat. Immediate action should be taken to prevent further spread and mitigate any damage caused by the malware. By following the recommendations outlined in this report, the organization can effectively mitigate the risk of future malware infections and strengthen its overall security posture.