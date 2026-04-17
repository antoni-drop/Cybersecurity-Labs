# 🔍 Networking

### This file documents my understanding of how data moves, different protocols, and traffic behaviour. It is the notes I refer to when it comes to networking.

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

### Traffic Anomalies



















---
[⬅️ Back to Labs](../README.md)
