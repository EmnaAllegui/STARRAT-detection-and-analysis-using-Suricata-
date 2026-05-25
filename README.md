# STARRAT-detection-and-analysis-using-Suricata-


## Objective

Analyze the provided PCAP file and prepare a professional incident report based on the observed malicious network activity.

# Environment

- Ubuntu server 24.04 installed on virtual machine.
- Kali 25.3 operating system is used to manage the server remotely.

---

# Tools

- Suricata 8.0.3 release installed on the ubuntu server.
- Wireshark 

---

# LAN Segmentation

The given PCAP represents a traffic captured in the following Active Directory environment:

- Network Range: 172.16.1.0/24(IP range from 172.16.1.0 to 172.16.1.255)
- Domain Name: wiresharkworkshop.online
- Domain Controller:172.16.1.4 — WIRESHARK-WS-DC
- Default Gateway: 172.16.1.1
- Broadcast Address: 172.16.1.255

---


# 1.Executive Summary

At 02:40 AM on 7 July 2024, an internal domain-joined host with IP 172.16.1.66, part of wiresharkworkshop.online Active Directory domain, was infected with STRRAT malware. This malware enables an attacker to remotely control the host and perform reconnaissance, including retrieving the host’s external IP. Then the infected host connected to a remote Command and Control(c2) server.

---

# 2.PCAP analysis

The PCAP file was analyzed using Suricata:

<img width="944" height="81" alt="image" src="https://github.com/user-attachments/assets/353228f3-6240-42c6-833c-b881900e1b12" />

➡️ Suricata generates 4 logs :

- Eve.json: contains detailed logs with all suspicious activity.
- Fast.log: contains a quick alerts for detected threat.
- Suricata.log: contains general log activity.
- Stats.log: shows statistics log at a fixed interval every 8 second.

For our lab, the analysis focuses on eve.json and fast.log as they provide revelant information for detecting and validating malicious network activity .

<img width="945" height="380" alt="image" src="https://github.com/user-attachments/assets/1c5540c5-c3ff-435b-9a50-8853683d8c63" />


➡️The analysis identified command and control activity associated with STRRAT malware, classified as critical severity.

---

## 2.1.DNS analysis

Inspecting the Wireshark DNS filter, several queries to well-known domains were observed including:

- Microsoft.com
- Bing.com
- msn.com

➡️This behavior is legitimate inside Active directory environment, as Windows system communicates with Microsoft services for updates, verifying connectivity and background services

Other DNS requests were identified including:

- repo1.maven.org
- github.com
- objects.githubusercontent.com

➡️These domains are legitimate as they are used to store software repository, source code and applications dependencies particularly within java-based environments.*

<img width="983" height="512" alt="image" src="https://github.com/user-attachments/assets/d837d5bc-e357-4b4e-9a57-f3cdd8c824ff" />

---

## 2.2. HTTP Analysis

By filtering the Wireshark HTTP traffic, several HTTP requests to external IP addresses were observed:

- Request to connecttest.txt of msftconnecttest.com. This activity is legitimate, as it is used by windows to verify internet connectivity.

- Request to /json of ip-api.com, which was used to geolocate public IP host’s address. This activity is malicious, performed by STRRAT malware to retrieve external host’s IP.

<img width="983" height="479" alt="image" src="https://github.com/user-attachments/assets/b7c139e9-0e4d-4b3d-9615-69e203d1bd0c" />


---

## 2.3. Analyzing outbound Communications

The analysis of complete TCP stream using Wireshark reveals a clear indicator of STRRAT malware activity.

<img width="983" height="396" alt="image" src="https://github.com/user-attachments/assets/ea8c0367-ecd0-49b8-9462-37bc635cd745" />

<img width="983" height="692" alt="image" src="https://github.com/user-attachments/assets/5cd63916-a861-47d9-94d7-30897fce55ce" />

---

# 3.Victim Details

## 3.1. Host information from NBNS (NetBios Name Server) traffic

We can use NBNS traffic to identify IP adresses , Mac addresses and hostnames for computers running on Microsoft Windows or Apple hosts running on Mac Os.Applying NBNS filter I identified the infected host’s has:

- IP address: 172.16.1.66
- MAC address: 00:1e:64:ec:f3:08
- hostname: DESKTOP-SKBR25F

<img width="983" height="514" alt="image" src="https://github.com/user-attachments/assets/4bebee75-e196-4d2f-96d6-22f82bacd145" />


---

## 3.2. Windows user account name from Kerberos traffic

For windows hosts in Active directory environment, we can find user account names from Kerberos traffic.

By applying Kerberos.CNameString filter, we identified that the Windows user account for the compromised host is ccollier. To differentiate between a hostname and a username we examine the end of CNameString: if it ends with $ it’s a hostname otherwise it is a username account.

<img width="983" height="516" alt="image" src="https://github.com/user-attachments/assets/978332ff-cf95-4cc8-829e-aff378b94117" />

---

# 4.Indicator of compromise (IOCs)

After analyzing PCAP traffic and Suricata eve.json. The following are major indicators of compromise:

- 172.16.1.66  
Compromised internal host infected with STRRAT malware

- 141.98.10.79  
External Command and Controle C2 server used by STRRAT for remote communication and confirmed suspicious by VirusTotal.

<img width="945" height="410" alt="image" src="https://github.com/user-attachments/assets/512ab8af-ddef-4b59-9212-6f8c0dfe15e8" />

- 208.95.112.1  
Public IP address of the infected host, returned by ip.api.com request. VirusTotal indicates that this IP was associated with phishing activity.

<img width="945" height="382" alt="image" src="https://github.com/user-attachments/assets/8ba13616-6d4f-438e-be57-e360b8e306d7" />

- 12132  
Port used by C2C server to receive commands and maintain host control

- ip-api.com  
External domain accessed to perform reconnaissance, confirmed by VirusTotal as suspicious

<img width="945" height="400" alt="image" src="https://github.com/user-attachments/assets/d8510244-0003-4027-8699-ae98e6edcee7" />

---

# Conclusion

After analyzing the network traffic, I identified a malicious activity with a critical impact.

The presence of STRRAT malware and Command-and-Control C2 communications provides clear evidence that the DESKTOP-SKBR25F machine was compromised.

I therefore immediately recommand to :

- Disconnect and isolate the compromised host from the network
- Disable ccollier account
- Reset ccollier credentials
- Perform a full foreinsic analysis for the infected host
