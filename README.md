# 🔐 SOC Analyst Report – Wireshark PCAP Investigation

## 🏢 Organization: BulbaTech Innovations

**Incident Trigger:**  
An alert was raised due to abnormal traffic patterns and repeated outbound queries from an external-facing LAN endpoint (`172.16.1.16`).  

PCAP URL: https://challenges.malwarecube.com/#/c/7265ec1c-9773-4c7c-9ed4-2ea26e19f346  
PCAP Provided By https://www.malware-traffic-analysis.net/

---

## 🎯 Objective

To analyze the `wireshark_challenge.pcap` network traffic capture using **Wireshark**, identify suspicious or malicious behaviors, and uncover any potential **Indicators of Compromise (IoCs)**.

---

## 📊 Initial Assessment

Initial triage involved navigating Wireshark's `Statistics` menu to review **Protocol Hierarchy**, **Conversations**, and **Endpoints**.

- **Protocol Hierarchy:**  
  Only a minimal set of protocols was active, prompting deeper investigation.
  <picture>![image](https://github.com/user-attachments/assets/3b34befc-e8da-4922-ba0d-0ea8c21a87d0)</picture>


- **Conversations & Endpoints:**  
  IP address `172.16.1.16` had ongoing communication with external entities. This host, identified as external-facing IP on the LAN, was involved in persistent and repetitive queries — suggesting it may be compromised or targeted.
  Detailed insights from the conversation view revealed that internal host `172.16.1.16` was engaged in persistent communication with an external entity...
  <picture>![image](https://github.com/user-attachments/assets/cdb9dc04-6e61-4257-9c8c-495ec20d9e66)</picture>
  <p align="center">
    <strong>Figure 2 – Conversation Screenshot pop-up </strong>
</p>

  <picture>![image](https://github.com/user-attachments/assets/dcfc72fe-592a-4253-a1d0-a4fdeb472443)</picture>  
  <p align="center">
    <strong>Figure 2 – Screenshot of Endpoints pop-up </strong>
</p>

---

## 🌐 HTTP Request Analysis

Navigating to: Statistics > HTTP > Requests  

The following was observed:
- A suspicious **HTTP GET request** was made to IP `162.252.172.54`.
- Direct IP usage is suggestive of an IoC, possibly an evasion tactics.
- The requested URI was obscure and non-standard.

<picture>![image](https://github.com/user-attachments/assets/e98974e2-0579-45e6-a298-eedd6be704e6)</picture>

---

## 📥 Payload Discovery via HTTP Stream

Following the suspicious HTTP stream revealed:

- The response contained the string:  
  `"This program cannot be run in DOS mode."`
- The file began with the **MZ** header — the standard signature of **Windows executable files (.exe / .dll)**.
- MIME type was falsely reported as `image/gif`, a common obfuscation method to bypass filtering.
<picture>![image](https://github.com/user-attachments/assets/35fb0ef1-a1c3-4335-a9ad-0c3493f8ac34)</picture>

> These characteristics strongly indicated the payload was an **executable masquerading as a benign image.**
</picture>![image](https://github.com/user-attachments/assets/87b4c237-6ef5-49fb-aa07-5fcd2141f4dd)</picture>
This is likely a potential payload and further investigation is carried out in the next section.
---

## 📁 Object Extraction and Threat Confirmation

- Using `File > Export Objects > HTTP`, the payload was successfully extracted.
  <picture>![image](https://github.com/user-attachments/assets/ece45a3b-7a0f-4378-80da-f1bb2bd13662)</picture>

- Upon inspection, the object was identified as a `.dll` (Dynamic Link Library).
  <picture>![image](https://github.com/user-attachments/assets/d1d0e81b-1931-487c-8ad5-1df394743467)</picture>

- The **SHA-256 hash** of the file was computed.
- Cross-checking with **VirusTotal** confirmed the file was **malicious**, classified as a **Trojan**.
  <picture>![image](https://github.com/user-attachments/assets/b5d4a399-c809-482c-85d7-e581c783e912)</picture>


---

## 🧩 Indicators of Compromise (IoCs)

| **Artifact**             | **Details**                                |
|--------------------------|---------------------------------------------|
| **Internal Host IP**     | `172.16.1.16`                               |
| **Suspicious External IP** | `162.252.172.54`                          |
| **Protocol**             | HTTP (non-DNS resolved GET request)         |
| **File Type**            | `.dll` with MZ executable signature         |
| **Threat Verdict**       | **Malicious Trojan** (confirmed via VirusTotal) |

---

## 🛡️ Recommendations

- 🔒 **Isolate** host `172.16.1.16` for detailed forensic investigation.
- 🚫 **Block outbound connections** to IP `162.252.172.54`.
- 🔍 **Monitor for future HTTP GET requests** that bypass DNS resolution.
- 🧰 **Update security controls** to detect executables disguised as media (e.g., `image/gif` with `.dll` payloads).
- 📡 **Review DNS and proxy logs** for similar bypass attempts.
- 🧨 **Conduct a threat hunt** across the environment for related IoCs.

---

## ✅ Conclusion

The investigation confirms that a **malicious DLL file** was delivered via a deceptive HTTP request to an internal host. The tactics observed — including direct IP access and content-type obfuscation — are consistent with stealthy malware delivery, potentially indicative of **Command and Control (C2)** activity.

**Immediate containment and further environment-wide threat detection are advised.**

---
| Questions and Answers |
|-----------------------|
|Q1. How many total packets are in the wireshark_challenge.pcap packet capture?|
|Navigate to Statistics --> Capture FIle Properties and it is 39106  
![image](https://github.com/user-attachments/assets/7ad5ff83-7c5b-442d-84dc-3ad587077202)|
|Q2. What was the first domain name queried and resolved in the capture?|
|Type in DNS in the Display filter and hit Enter Key. Scroll up to the top of the query.In the Info section, you will see the first 'Standard query response...'Click on it and go to Domain Names System below, Click on Answers.  
It is webmasterdev.com and IP is 184.168.98.68|
|Q3.Q3 What is the associated IP address of the domain name?|
|184.168.98.68|
|Q4 How many HTTP packets are contained in the capture file?|
|Typed http in the response header, hit the enter key and counted them|
|![image](https://github.com/user-attachments/assets/2fcb413a-329d-45bb-9ab3-b57e7f3a7b29)
|
|Q5 What is the relative path the victim accessed on the web server to request a file for download?|

|![image](https://github.com/user-attachments/assets/0f201508-b035-47b4-b8d7-b5c74d86158d)  
/9GQ5A8/6ctf5JL  
This was found by navigating to Statistics->HTTP->Requests|
|Q6 Based on the response header, what file type or format does the web server claim the downloaded file to be?|
|![image](https://github.com/user-attachments/assets/a2b8b187-c199-4912-bec6-2d5562366140)  
image/gif. In the display filter, I entered http and hit the enter key, Right click and Follow Http Stream|
|Q7 However, what is the actual file signature or magic bytes contained in the file?|
|![image](https://github.com/user-attachments/assets/c99682f1-6212-4491-b98d-2bcb34176acb)  
MZ|
|Q8 What command-line utility or program was used by the victim to download the file?|
|WindowsPowerShell 
![image](https://github.com/user-attachments/assets/b04e1014-abb7-4f54-b140-4148d8234333)
|
|Q9 What is the sha256 hash of the downloaded file?|
|9b8ffdc8ba2b2caa485cca56a82b2dcbd251f65fb30bc88f0ac3da6704e4d3c6  
![image](https://github.com/user-attachments/assets/6b687774-127f-480d-9611-85ff725673ae)|
|Q10 Submit the uncovered hash to VirusTotal. Based on the popular threat label and tags, what type of malware did the endpoint get infected with?|
|pikabot  
![image](https://github.com/user-attachments/assets/aff4ef7d-0486-4ca3-81c9-b8008e25dc65)
|
|Q11 What protocol makes up the majority of UDP packets?|
|DNS|

|Q12 Look at the domain names that were queried within the capture. In defanged format, what is the base domain name that is continually queried?|
|steasteel[.]net  
In the Display filter, I typed in dns and hit the enter key. I then manually observed the frequent domain nammes. Got it and used cybersheff tool to defang it.|

|Q13 Read up on MITRE ATT&CK ID T1071.004. What is the attack technique we re likely seeing in the PCAP file often known as?|
|DNS tunneling|







