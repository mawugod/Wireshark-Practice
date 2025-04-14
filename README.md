# 🔐 SOC Analyst Report – Wireshark PCAP Investigation

## 🏢 Organization: BulbaTech Innovations

**Incident Trigger:**  
An alert was raised due to abnormal traffic patterns and repeated outbound queries from an external-facing LAN endpoint (`172.16.1.16`).

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
  IP address `172.16.1.16` had ongoing communication with external entities. This host, identified as external-facing on the LAN, was involved in persistent and repetitive queries — suggesting it may be compromised or targeted.
  <picture>![image](https://github.com/user-attachments/assets/cdb9dc04-6e61-4257-9c8c-495ec20d9e66)</picture>
  <picture>![image](https://github.com/user-attachments/assets/dcfc72fe-592a-4253-a1d0-a4fdeb472443)</picture>



---

## 🌐 HTTP Request Analysis

Navigating to: Statistics > HTTP > Requests  




The following was observed:
- A suspicious **HTTP GET request** was made to IP `162.252.172.54`.
- **DNS resolution was bypassed** — direct IP access suggests evasion tactics.
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

---

## 📁 Object Extraction and Threat Confirmation

- Using `File > Export Objects > HTTP`, the payload was successfully extracted.
- Upon inspection, the object was identified as a `.dll` (Dynamic Link Library).
- The **SHA-256 hash** of the file was computed.
- Cross-checking with **VirusTotal** confirmed the file was **malicious**, classified as a **Trojan**.

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

