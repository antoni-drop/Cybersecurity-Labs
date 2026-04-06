# 🎣 Phishing Analysis: The Greenholt Phish & Snapped Phishing Line

### In this file there are two different CTF analyses. I have paired them together as they were a part of the "Phishing Analysis" module in the TryHackMe SOC L1 analyst pathway.

## The Scenario 

A sales executive at Greenholt PLC reported a suspicious email that appeared from a known customer. Although the senders name was familiar, the email triggered several high risk indicators that has been escalated to the SOC for further investigation.

The main goal is to find and analyse key artifacts investigating the key message source using tools to assess the potential maliciousness of the email.

After opening the .eml file using Mozilla Thunderbird. This is the email that was sent: 

<img width="861" height="888" alt="Screenshot 2026-04-06 170829" src="https://github.com/user-attachments/assets/0203bbe5-d70c-44e0-98b9-16a6b72795af" />

---

## 🚩Initial Red Flags

Starting the analysis of the initial "Red Flags" of the email I noticed these things:

- The "Good Day" opening to the email is very generic for someone that knows you well.
- I noted the **"Transfer Reference Number"** in the email, which is a high pressure tactic designed to make the victim click the link urgently to "verify" a transaction they weren't expecting.
- I also noted the email's display name, sender's email address and what email address will receive a reply to the email.

## 🔍Analysing the message source

By using the shortcut **Ctrl + U**, I opened the message source to confirm my suspicions. I started off by looking for the originating IP address of the email.

<img width="661" height="31" alt="Screenshot 2026-04-06 172346" src="https://github.com/user-attachments/assets/a8ee2c29-212e-4244-99a0-be72c4e6f6cb" />

By looking for the bottom **"Received"** header, I found the originating IP address. Which in this case was **192.119.71.157**.

Initially I went to the terminal to do a whois lookup but then I realised that it wouldn't work as the outbound network traffic is restriced on the VM so I had to use an OSINT tool.

<img width="402" height="44" alt="Screenshot 2026-04-06 172918" src="https://github.com/user-attachments/assets/a5a07732-519b-4871-9ea7-4b50772c7bc5" />

Using the https://ipinfo.io/ website, I copy and pasted the IP to get all the details on it and it gave me the owner's name of the originating IP.

<img width="824" height="435" alt="Screenshot 2026-04-06 173727" src="https://github.com/user-attachments/assets/8e7438f2-6d3a-4a6b-9dfe-f1d4fb1953b2" />

The reason why this is a major red flag is because the IP the email was sent from was from a budget VPS (Virtual Private Server) provider rather than through a dedicated enterprise mail server (like Microsoft 365).

---
## Verifying Sender Authenticity

My next step was to do an **SPF** (Sender Policy Framework) check on the Return-Path domain which I identified in the headers as:

<img width="265" height="18" alt="Screenshot 2026-04-06 175143" src="https://github.com/user-attachments/assets/1a57eaa8-00c9-4992-a10b-f8058ef72c85" />

I used the https://dmarcian.com/spf-survey/ tool to look up the SPF record of this domain:

<img width="491" height="47" alt="Screenshot 2026-04-06 175425" src="https://github.com/user-attachments/assets/c93170e9-9c58-40ce-96d9-3ee7f71e9e20" />

Looking up the SPF record I confirmed whether the originating IP was an authorised sender. Since this IP was not listed in the domain’s SPF record, it confirmed that the email was sent from an unauthorised server.

I checked the DMARC record of the same domain using the https://dmarcian.com/dmarc-inspector/ tool and looked for the Policy (p=).s I noticed it was set a quarantine which explains why the email wasn't blocked. If it was set to reject the email most likely wouldn't have made it to the victim's inbox. 

<img width="317" height="43" alt="Screenshot 2026-04-06 175643" src="https://github.com/user-attachments/assets/94742a43-65b6-49ab-a541-040933e5a8e2" />

I turned my attention to the file attached to the email. I saved the file ready for analysis.

<img width="389" height="26" alt="image" src="https://github.com/user-attachments/assets/ff569eff-1d45-4c51-b616-81051a78e6da" />

I opened the terminal and got the SHA256 hash of the file and scanned it in https://www.virustotal.com/gui/home/upload which flagged it as malicious. 

<img width="680" height="170" alt="image" src="https://github.com/user-attachments/assets/321af844-8894-45fb-b297-a3adc48d4298" />

The file was a 400.26 KB RAR archive which contained a **Trojan**.






























---
[⬅️ Back to Labs](../README.md)
