# 🔗 Moniker Link: Outlook Bypass (CVE-2024-21413)

[![CVE-2024-21413](https://img.shields.io/badge/CVE-2024--21413-critical?style=flat-square&color=red)](https://nvd.nist.gov/vuln/detail/CVE-2024-21413)
[![Category](https://img.shields.io/badge/Category-Credential%20Theft-blue?style=flat-square)](#)

### This my analysis on how a bypass in Microsoft Outlook's "Protected View" allows attackers to force NTLM authentication to a remote server.

During this lab I learned how a single character (an exclamation point) turns a simple link into an automated **"ID thief"**. By exploiting a bypass that tricks Outlook's usual security checks through an exclamation point embedded in an email link **"!"**. I managed to bypass **"protected view"** completely. This let me force a user's computer to hand over its NTLM hashes without the user ever knowing. Doing this myself and seeing the hash pop up instantly made me realise why it was rated as a **9.8 Critical Vulnerability**.

### So what actually is a Moniker?

As microsoft states: **"A moniker in COM is not only a way to identify an object—a moniker is also implemented as an object. This object provides services allowing a component to obtain a pointer to the object identified by the moniker. This process is referred to as binding."**  

> **Note:** **COM** stands for **Component Object Model**. It is the language that Microsoft uses to let other software components talk to each other.

In my own words, a **Moniker** is a **"Smart-Link"**. As a normal link just points to a file, a Moniker Link points to that file as well as carrying instructions on how that file should be opened or **"Bound"**. Binding is essentially the handshake. The computer isn't just looking at the link but actually opens a live connection to that file.
In relation to this exploit the **"!"** symbol triggers the **"binding process"**. This causes Outlook's usual security process to skip completely letting me force a connection with the user and jump to connecting them to my malicious server. 


## So how is it done? 

The goal in this exploit is to bypass Outlook's "Protected View" which will look something like this:

<img width="429" height="272" alt="Screenshot 2026-04-02 192038" src="https://github.com/user-attachments/assets/088d6790-da77-4bc7-90b1-2299ab25df2f" />


>**NOTE:** Screenshots used in this writeup are from the THM room that I learned this exploit from.


Protected View is a "read only sandbox mode". It acts as a security boundary that opens suspicious files in a restricted environment limiting the files access to the rest of the system. Preventing malicious links, scripts and macros from running unless the user trusts the document.

The vulnerability exploits how Outlook processes unconventional URLs. Outlook could safely handle standard web traffic using HTTP/S but it could get tricked by using an exclamation mark within the URL using the **file://** protocol in the hyperlink.

By making your own custom URL using the **file://** protocol, you can tell Outlook to attempt to look at a file on a remote network share which forces the system to automatically send the user's NTLM password hashes to the attackers server. This triggers a silent **SMB (Server Message Block)** authentication - a protocol used for network file sharing - that leaks the user's login information without any of their interaction.




<img width="865" height="54" alt="Screenshot 2026-04-06 091152" src="https://github.com/user-attachments/assets/5609e76e-2677-46ed-9407-fef7c2545644" />



The first URL triggers the "Protected View" and blocks the attempt.



<img width="861" height="57" alt="Screenshot 2026-04-06 091203" src="https://github.com/user-attachments/assets/b1d7d52b-bdb2-4eb4-8b32-0b7cc0b36626" />


This time, the **"!"** is embedded within the URL bypassing "Protected View". The share does not have to exist on the remote device as an authentication attempt will proceed regardless. The vulnerability isn't about successfully opening a file, it's about establishing a connection with the attacker's server. This results in the user's Windows netNTLMv2 hash being sent to the attacker.

---

*In the THM room a PoC with a Python script is provided by the user CMNatic for this exploit which I have provided a link to at the bottom of this page.*

Normally for this exploit you would need your own SMTP server but it has been provided for me as a part of this lab.

Starting off, I used the **Responder** tool to create an SMB listener using the **-I ens5** interface.



<img width="509" height="105" alt="Screenshot 2026-04-06 104243" src="https://github.com/user-attachments/assets/8d7a4392-0d6b-46fe-84da-50982b5f1ecf" />

I then set up the vulnerable machine provided in the THM room.

<img width="1021" height="698" alt="Screenshot 2026-04-06 104713" src="https://github.com/user-attachments/assets/662c0e66-fb16-49cb-82cf-93fccf209023" />

---


I created a new file called **exploit.py** using **nano** (**nano exploit.py**) then copy and pasted the Python script into it. After the script was pasted I needed to change the Moniker Link line and put in the attacker IP address into the placeholder as shown below.

<img width="552" height="16" alt="Screenshot 2026-04-06 134056" src="https://github.com/user-attachments/assets/eea200da-e0cf-414d-bffe-335a3542fb55" />

The mailserver IP also needed changed (to the victims IP) so I replaced that too.

<img width="368" height="15" alt="Screenshot 2026-04-06 134953" src="https://github.com/user-attachments/assets/b1e8bceb-a3a8-4309-b755-5dc375994c9a" />

---

The next step was to execute the python script. In this case, the password was **attacker**.

<img width="398" height="72" alt="Screenshot 2026-04-06 135314" src="https://github.com/user-attachments/assets/fe696a82-ab4a-4240-803d-03765d6daf84" />

Switching over to the vulnerable machine, this is the email that was sent:

<img width="832" height="240" alt="Screenshot 2026-04-06 135712" src="https://github.com/user-attachments/assets/4eb96ae2-f852-4e55-8f0d-f4bca93cce3a" />

After clicking the "click me" I went back to my attacker machine and checked my responder.

<img width="953" height="127" alt="Screenshot 2026-04-06 140004" src="https://github.com/user-attachments/assets/f11ff7b8-d0e4-4a89-bc64-0c4f06cda367" />

The NTLMv2 Hash has been captured!














https://github.com/CMNatic/CVE-2024-21413

---
[⬅️ Back to Labs](../README.md)
