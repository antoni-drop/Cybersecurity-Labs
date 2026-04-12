# 🚩 Incident Response: Summit & Eviction

### In this file there are two CTF incident response analyses from the SOC L1 Analyst pathway on TryHackMe. I have documented how I tracked adversary movement through compromised networks and how I mapped them to the Cyber Kill Chain. As I go through them, I will also explain the different layers of the Pyramid Of Pain and how much of a nuisance it is for the attacker to work around.

# Lab 1 : Summit  🏔️

<img width="299" height="169" alt="image" src="https://github.com/user-attachments/assets/06f90c30-2e2c-4538-8387-9c04e067276d" />

> A diagram of the **Pyramid of Pain**

## Scenario

PicoSecure is conducting a threat simulation and detection engineering management to improve their malware detection capabilities following a high volume of recent incident response activities. 

An external penetration tester is working in this activity in a **purple team** scenario. The tester will be attempting to execute malware samples as I will be configuring PicoSecure's security tools to detect and prevent them from running. The attack will get more sophisticated as I introduce stronger security measures moving up the **Pyramid Of Pain**.

### Level 1 & 2 (Hashes and IPs)

The exercise started off nice and simple. The first email started off with a quick introduction and an attached file to the email:

<img width="299" height="69" alt="image" src="https://github.com/user-attachments/assets/c1dac1f9-5882-44ba-a2f4-a0d0da975dcf" />

Using the PicoSecure **SIEM** console, I scanned the file using the **"Malware Sandbox"** tool and reviewed the generated report.

<img width="424" height="65" alt="image" src="https://github.com/user-attachments/assets/98892135-3221-4c4a-9cb6-253b67d8c57f" />

After locating the hash, I added it to the Hash Block list: 

Blocking the hash is a necessary immediate response however it sits at the bottom of the **Pyramid Of Pain**. This is because the attacker can alter a single line of code in the malware which will change the hash completely making the block useless. For an attacker, it is **trivial** to bypass. However, since hashes are all unique to a file they are the highest confidence indicators of indentifying the threat.

In a real world scenario, **fuzzy hashing** is used as a line of defence because it identifies similar file structures and shared code patterns. Categorising malware into **"families"** making them easier to detect even though their hash has changed.

---

Moving up the pyramid, **IP addresses** are next. As the attacker probably modified the malware sample giving it a different hash. I analysed the SIEM logs and spotted a new connection attempt to a remote server. Extracting the IP, I created a new firewall rule in the **Firewall Rule Manager** in the SIEM:

<img width="760" height="336" alt="image" src="https://github.com/user-attachments/assets/f7fbbee8-8550-4964-95f7-e4ad257b7827" />

>  **NOTE** : "Egress" means traffic coming out.

Since the malware is on the local machine, I need to block any traffic coming out to the attackers **C2 (Command and Control)** server.

Blocking IPs is also an **easy** bypass for an attacker as all they have to do is change their IP through a VPN or use a different proxy and the malware can re-establish its connection.

In this case, the tester signed up to a cloud service provider giving them a large pool of IP addresses to use as proxies.

> **NOTE** : Proxies are the "middlemen" for internet traffic. Requests go through them rather than straight to the website or server on your behalf. Attackers use this as this keeps them hidden.

---

### Level 3 (Domain Names)

In the third sample, I found the suspicious domain the attacker was using with a backdoor so I went to the **DNS Rule Manager** and created a new rule in the SIEM and implemented a block:

<img width="759" height="397" alt="image" src="https://github.com/user-attachments/assets/ea460656-471f-4f6d-ba06-f4796e4885d8" />

Blocking a domain is classed as **"simple"** to change for an attacker on the pyramid of pain. 

Unlike a free IP change using a VPN, you have to buy a new domain but many registrars offer domains for very cheap so the financial pain is minimal for the attacker. A sophisticated attacker would more than likely have more than one domain to work with. However if the attacker has one domain, it does slow them down as getting a new domain isn't as quick and changing your IP because of **DNS Propogation**. This creates a window of opportunity for the defence team giving critical time to isolate and remediate infected systems.

> **NOTE** : **DNS Propogation** is the **"lag time"** that servers around the world register that your website domain has changed. The internet basically has to update its **"phone book"**.

---

### Level 4 (Host Artifacts)

Sample 4 was about finding artifacts the malware left on the system. This is the evidence left on the victims machine:

<img width="983" height="283" alt="image" src="https://github.com/user-attachments/assets/38aaa9b1-e144-4ab9-9f10-1be03ae3acd5" />

The malware attempted to modify the Windows registry keys to disable **Windows Defender Real Time Monitoring** which is more than likely a defence evasion **Host Artifact**.

By using the **Sigma Rule Builder** in the SIEM I created a new sigma rule in the system to stop the malwares attempt of disabling real time monitoring:

<img width="658" height="260" alt="image" src="https://github.com/user-attachments/assets/3bfbae40-beda-48c6-9a83-d1b5dbfcc970" />

This is a long term defence as this isn't just blocking an IP or a domain, its detecting a specific behaviour and stopping it. This is classed as **"annoying"** for the attacker to deal with as it is defending against the malwares code itself rather than how it reaches the system. Breaking how the attacker communicates with the host making it time consuming and expensive for the attacker to change.

---

### Level 5 (Network Artifacts)

This time, the next sample given was a network traffic log file. This contained the behavioural fingerprints of how the malware communicated with the system:

<img width="816" height="751" alt="image" src="https://github.com/user-attachments/assets/702e1c0f-dc77-4490-a00e-56e096555e1a" />

Looking at this file, I noticed this:

- Consistent packet size of 97 bytes.
- Fixed frequency of 30 minutes.

This pattern shows that the malware reconnects every 30 minutes to ask if the attackers **Command and Control** server has any new instructions. This is because if the malware was connected all the time it would be a lot easier to detect. Instead it blends in with the normal background noise.

<img width="640" height="247" alt="image" src="https://github.com/user-attachments/assets/e3d9c10c-a852-48f4-a885-3ef8d2c3e2c2" />

To combat this, I created a new rule in the sigma rule builder and blocked all connections with a 97 byte file size every 30 minutes (1800 seconds).

This is classed as **"annoying"** on the pyramid of pain. This is because the attacker will have to change the communication protocol in the malware which will take some time to test and reconfigure for an attack.

---

### Level 6 (TTPs)

The highest level is **Tactics, Techniques and Procedures**. This is the highest on the pyramid classed as **"tough"** as this isn't anything to do with the tools being used but the behavioural patterns and methodologies of the attacker.

For the final sample I was given a command log where I was able to identify the attackers playbook:

<img width="449" height="246" alt="image" src="https://github.com/user-attachments/assets/22282bce-e31b-45e7-8a57-819260417267" />

> **NOTE**: I copy and pasted the logs into AI to give me an analysis of the commands as at the time I didn't completely understand every single log in the file. 

I noticed the attacker was piping outputs to a hidden file named **exfiltr8.log** in the **%temp%** folder preparing it for theft.

The attacker was using the **dir** command to enumarate the file system to map out drives (C: and D:) to look for sensitive data in **documents** and **settings** or the **program files**. Using the **>>** operator to gather all the output into a single staging file. A staging file is like the attackers "shopping bag". The attacker isn't sending a bunch of suspicious requests but instead gathering all the data into a single file.
The attacker was also using other commands:

- **ver** and **systeminfo** to identify the system version to find specific exploits for privelege escalation
- **ipconfig /all** to map the network interface and identify where the machine sits in the local network.
- **netstat -ano** to check for active connections so they can find their next target.
- **net start** to list services that are running so they can identify secuirty software that they need to bypass or disable.

Under the **MITRE ATT&CK** tactics, the commands would be categorised like this:

- System Information Discovery: **ver, systeminfo**
- System Network Configuration Discovery: **ipconfig /all**
- System Network Connections Discovery: **netstat -ano**
- System Service Discovery: **net start**

I created a new sigma rule in the SIEM that blocks any output into the attackers **%temp%** temporary directory as blocing the commands would cause issues for the whole system:

<img width="646" height="146" alt="Screenshot 2026-04-12 104735" src="https://github.com/user-attachments/assets/8dd5ed76-5d93-43c2-aeb2-2abe309ea2b3" />

By blocking the attackers **data staging** process, I prevented the attacker from reaching the **Exfiltration** phase of their attack.

### The Summit CTF is complete!

<img width="296" height="92" alt="Screenshot 2026-04-12 104813" src="https://github.com/user-attachments/assets/d5c312f6-666d-405e-8495-8cc462ef1028" />

---

# Lab 2 : Eviction

This lab is focused on the **Cyber Kill Chain** mapping threat intelligence to the **MITRE ATT&CK** framework. The link to the **ATT&CK Navigator** is provided below:

https://static-labs.tryhackme.cloud/sites/eviction/

> Methodology : I also used AI assisted analysis here to help me map the complex TTP's to the adversary group as I understand that using AI is common in the SOC workflow. The lab itself was just finding the different techniques listed in the framework to answer questions but i've done my best to try understand the whole process and the "why" behind it to make it a bit more of an advanced writeup.

## Scenario

E-Corp is a company that manufactures rare earth metals for government and non-government clients. Sunny, an SOC analyst in the company received an intelligence report that informs her about an attacker group **APT28** might be trying to attack organisations similar to E-Corp. For this task I need to act on this intelligence identifying the groups "digital fingerprints" using the **MITRE ATT&CK** framework to see if they have been successful in breaching E-Corp's manufacturing data.

> **NOTE** : in a real world scenario this would be called a **Threat-Informed Defence** where company's look for what current adversary groups are up to before an attack takes place. Making their company a less attractive target.

---
## ⛓️ The Cyber Kill Chain

This is a framework used to identify and prevent cyberattacks. The idea is that an attacker must go through every stage to reach their goal. There are 7 different stages:

- **Reconnaissance**: Researching and identifying targets.

- **Weaponization**: Creating a malicious payload (e.g, a virus attached to a document).

- **Delivery**: Sending the payload to the victim (e.g, via spearphishing emails).

- **Exploitation**: The code executes on the victim's system, often through a software vulnerability.

- **Installation**: The malware installs a permanent "backdoor" to maintain access (Persistence).

- **Command & Control (C2)**: The infected host opens a channel to the attacker's server to receive instructions.

- **Actions on Objectives**: The final goal, such as data exfiltration (theft) or encryption.



## APT28 Adversary Playbook

Based on the **ATT&CK NAVIGATOR** analysis for APT28, I mapped the likely path they would use to go through E-Corp's network so protective measures are in place before waiting for an alert.

1 - **Reconnaissance and initial access**

- **Spearphishing Link (T1566.002)** : The adversary group used a **Spearphishing Link** to gain a foothold in the network and perform reconnaissance.
- **Email Accounts (T1586.002)** : Developing their resources, the attacker compromises email accounts to create a level of trust while sending these links. 

2 - **Execution and Persistence**

- **Malicious File (T1204.002) & Malicious Link (T1204.001)** : These two methods are what makes the attack change from a "threat" into an "incident" if a user is tricked by these compromised emails to click one of these links.
- **PowerShell (T1059.001) & Windows Command Shell (T1059.003)** : APT's use this as PowerShell and CMD are already installed on every Windows machine. This is because rather than bringing in their own tools which an antivirus might catch, they use the computers to do their dirty work on their own system to gain access. This is known as "living off the land".
- **Registry Run Keys / Startup Folder (T1547.001)** : This is how the attacker made sure they don't get kicked out. By changing the Windows registry run keys they can force the malware to start up automatically whenever the computer is turned on or a user logs in. Monitoring these keys should be a top priority as this is a permanent "backdoor" within the system.

3 - **Defence Evasion and Discovery**

- **Rundll32 (T1218.011)** : This system binary is a normal part of Windows so the APT is using it to run their malicious code while making it look like it is a standard background process. It keeps the code disguised and trusted in the system.
- **Network Sniffing (T1040)** : The APT used the **Responder** tool within the system capturing usernames and hashed passwords that allowed access to legitimate credentials. The attacker is "eavesdropping" on the network traffic within the system.

4 - **Lateral Movement and Exfiltration**

- **SMB/Windows Admin Shares (T1021.002)** : The APT is moving "sideways" from computer to computer. They are using **[Net]** (NetNTLM Hashes) and administrator credentials to map network drives. This is done to gain access to shared folders and servers where the company's sensitive data is stored.
- **SharePoint (T1213.002)** : The primary target was to steal intellectual property from E-corp's information repositories as Microsoft Sharepoint was the company's "digital vault".
- **External Proxy & Multi-hop Proxy** (T1090.002 & T1090.003) : The APT uses "middle man" computers to bounce the data back without getting caught which disguises the traffic making it look like a person browsing through the web.

---

While **MITRE ATT&CK** provides the specific "moves" the attacker makes, I have organized this playbook using the **Cyber Kill Chain** to show the timeline of the attack. By mapping these together, I can identify where we have the best chance to "break the chain."

- Phases 1-2 (Access & Execution): Where the "entry" can be stopped.

- Phases 3-4 (Persistence & Evasion): Where you catch them "hiding."

- Phases 5-7 (Lateral Movement & Exfiltration): Identifying the "theft".

By spotting the "digital fingerprints" the APT is using E-Corp can identify the adversary groups TTP's. Going from looking for an alert to knowing how these groups function. This makes them easier to spot and ensures the system is defended against their tactics.

### Conclusion

Through completing these two CTFs I've managed to demonstrate how the **Pyramid Of Pain** and the **Cyber Kill Chain** works. Using the **MITRE ATT&CK** framework to map adversary movement while using the **Cyber Kill Chain** to be able to see what phase of the attack the APT was in.


---
[⬅️ Back to Labs](../README.md)
