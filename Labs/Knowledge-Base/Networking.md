# 🔍 Networking

### This file documents my understanding of how data moves, different protocols, and traffic behaviour. This is the notes I refer to when it comes to networking.

---

## 🌐 Networking Framework : TCP/IP

While I know that the **OSI model** is the technical standard, I put my focus on the **TCP/IP** model. For me, this model is easier to remember and it's more practical in modern networking.

| TCP/IP Layer | Key Protocols | My Analytical Focus |
| :--- | :--- | :--- |
| **Application** | HTTP, DNS, SMB, SMTP | Where the "payload" is (malicious scripts, SQL injection, malicious links). |
| **Transport** | TCP, UDP | How the data flows, flags (SYN/ACK), and port behavior. |
| **Internet** | IP (IPv4/v6), ICMP | Source/destination reputation and routing anomalies. |
| **Network Access** | Ethernet, ARP | Connecting IPs to MAC addresses. |

---

### De-encapsulation

As encapsulation is what the computer does to build a message, I use de-encapsulation to understand the intent:

**Frame** (Network Access) -> **Packet** (Internet) -> **Segment** (Transport) -> **Data** (Application)

---

## 🤝 TCP 3-Way Handshake

Transmission Control Protocol must establish a reliable connection before any data can be sent. To analyse this, I look at the pattern of the connection:

*A standard handshake*

- **SYN (Synchronise)** Connection has been requested by the client.
- **SYN/ACK (Synchronise/Acknowledge)** Server acknowledges the request and sends its own synchronisation request.
- **ACK (Acknowledge)** Client confirms and the connection has been made.

*Other flags I keep a note of*

- **RST (Reset)** The receiver immediately kills the connection. This usually means a port is closed or a firewall is stopping the connection.
- **FIN (Finish)** A "goodbye" when the connection is done. This is normal.
- **PSH (Push)** Tells the receiver to process the data immediately. Common in SSH or telnet traffic.
- **URG (Urgent)** Marks the data within a packet as a priority.

---

### 🚨 Anomalies

- **Stealth scan (SYN/RST)** : "Knock and run" tactic. The client starts a connection but kills it once the server responds with **SYN/ACK**. The client is trying to identify open ports without getting logged. This is what an **nmap stealth scan** (-sS) would use.
- **Null scanning (No flags)** : TCP packet with no flags set, it is designed to bypass firewall and packet filters. If a port is closed it must respond with an **RST** packet, if it is open it will discard the packet. As this only works on Linux systems and not Windows, it is also used for **OS Fingerprinting**.
- **Xmas scan (FIN/PSH/URG)** : This is a 100% indicator that this is a scan that is "illegal" since it never happens in legitimate traffic. This is used for **OS Fingerprinting**.
- **Contradiction scan (SYN/FIN)** : This is a clear sign of a probe that attackers use to check if a firewall is properly configured or if they can bypass an **IDS (Intrusion Detection System)** that can't handle "garbage data". This works as it overwhelms the IDS making it let any traffic through as it needs to keep the network from crashing.

---

### 🚪 Common Ports & Services

| Port | Service | My Analytical Focus |
| :--- | :--- | :--- |
| **21** | **FTP** | Look for clear-text credential sniffing or unauthorized file exfiltration. |
| **22** | **SSH** | Monitor for high-frequency connection attempts (**Brute Force**). |
| **53** | **DNS** | Check for unusually long queries or high volume (potential **DNS Tunneling**). |
| **80 / 443** | **HTTP/S** | Inspect the payload for **SQL Injection**, **XSS**, or malicious web shells. |
| **139 / 445** | **SMB** | Watch for internal file-share scanning—a key indicator of **Ransomware** spreading. |
| **3389** | **RDP** | Identify unauthorised remote sessions or attempts to bypass GUI authentication. |
















---
[⬅️ Back to Labs](../README.md)
