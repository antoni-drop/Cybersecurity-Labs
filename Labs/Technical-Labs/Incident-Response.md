# 🚩 Incident Response: Summit & Eviction

### In this file there are two CTF incident response analyses from the SOC L1 Analyst pathway on TryHackMe. I have documented how I tracked adversary movement through compromised networks and how I mapped them to the Cyber Kill Chain. As I go through them, I will also explain the different layers of the Pyramid Of Pain and how much of a nuisance it is for the attacker to work around.

# Lab 1 : Summit  🏔️

## Scenario

PicoSecure is conducting a threat simulation and detection engineering management to improve their malware detection capabilities following a high volume of recent incident response activities. 

An external penetration tester is working in this activity in a **purple team** scenario. The tester will be attempting to execute malware samples as I will be configuring PicoSecure's security tools to detect and prevent them from running. The attack will get more sophisticated as I introduce stronger security measures moving up the **Pyramid Of Pain**.

### Level 1 & 2 (Hashes and IPs)

The exercise started off nice and simple. The first email started off with a quick introduction and an attached file to the email:

<img width="299" height="69" alt="image" src="https://github.com/user-attachments/assets/c1dac1f9-5882-44ba-a2f4-a0d0da975dcf" />

Using the PicoSecure **SIEM** console, I scanned the file using the **"Malware Sandbox"** tool and reviewed the generated report.

<img width="424" height="65" alt="image" src="https://github.com/user-attachments/assets/98892135-3221-4c4a-9cb6-253b67d8c57f" />

After locating the hash, I added it to the Hash Block list: 

Blocking the hash is a necessary immediate response however it sits at the bottom of the **Pyramid Of Pain**. This is because the attacker can alter a single line of code in the malware which will change the hash completely making the block useless. For an attacker, it is trivial to bypass. However, since hashes are all unique to a file they are the highest confidence indicators of indentifying the threat.

In a real world scenario, **fuzzy hashing** is used as a line of defence because it identifies similar file structures and shared code patterns. Categorising malware into **"families"** making them easier to detect even though their hash has changed.

---

Moving up the pyramid, **IP addresses** are next. As the attacker probably modified the malware sample giving it a different hash. I analysed the SIEM logs and spotted a new connection attempt to a remote server. Extracting the IP, I created a new firewall rule in the **Firewall Rule Manager** in the SIEM:

<img width="760" height="336" alt="image" src="https://github.com/user-attachments/assets/f7fbbee8-8550-4964-95f7-e4ad257b7827" />

>  **NOTE** : "Egress" means traffic coming out.

Since the malware is on the local machine, I need to block any traffic coming out to the attackers **C2 (Command and Control)** server.

Blocking IPs is also a trivial bypass for an attacker as all they have to do is change their IP through a VPN or use a different proxy and the malware can re-establish its connection.

In this case, the tester signed up to a cloud service provider giving them a large pool of IP addresses to use as proxies.

> **NOTE** : Proxies are the "middlemen" for internet traffic. Requests go through them rather than straight to the website or server on your behalf. Attackers use this as this keeps them hidden.
---

### Level 3 (Domain Names)

In the third sample, I found the suspicious domain the attacker was using with a backdoor so I went to the **DNS Rule Manager** and created a new rule in the SIEM and implemented a block:

<img width="759" height="397" alt="image" src="https://github.com/user-attachments/assets/ea460656-471f-4f6d-ba06-f4796e4885d8" />

Blocking a domain is classed as **"simple"** to change for an attacker on the pyramid of pain. 

Unlike a free IP change using a VPN, you have to buy a new domain but many registrars offer domains for very cheap so the financial pain is minimal for the attacker. A sophisticated attacker would more than likely have more than one domain to work with. However if the attacker has one domain, it does slow them down as getting a new domain isn't as quick and changing your IP because of **DNS Propogation**. This creates a window of opportunity for the defense team giving critical time to isolate and remediate infected systems.

> **NOTE** : **DNS Propogation** is the **"lag time"** that servers around the world register that your website domain has changed. The internet basically has to update its **"phone book"**.

---

### Level 4 (Host Artifacts)

Sample 4 was about finding artifacts the malware left on the system. This is the evidence left on the victims machine:

<img width="983" height="283" alt="image" src="https://github.com/user-attachments/assets/38aaa9b1-e144-4ab9-9f10-1be03ae3acd5" />

The malware attempted to modify the windows registry keys to disable **Windows Defender Real Time Monitoring** which is more than likely a defense evasion **Host Artifact**.

By using the **Sigma Rule Builder** in the SIEM I created a new sigma rule in the system to stop the malwares attempt of disabling real time monitoring:

<img width="658" height="260" alt="image" src="https://github.com/user-attachments/assets/3bfbae40-beda-48c6-9a83-d1b5dbfcc970" />

This is a long term defence as this isn't just blocking an IP or a domain, its detecting a specific behaviour and stopping it. This is classed as **"annoying"** for the attacker to deal with as it is defending against the malwares code itself rather than how it reaches the system. Breaking how the attacker communicates with the host making it time consuming and expensive for the attacker to change.

---

### Level 5 (Network Artifacts)

<img width="816" height="751" alt="image" src="https://github.com/user-attachments/assets/702e1c0f-dc77-4490-a00e-56e096555e1a" />








---
[⬅️ Back to Labs](../README.md)
