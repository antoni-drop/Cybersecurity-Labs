# 🔗 Moniker Link: Outlook Bypass (CVE-2024-21413)

[![CVE-2024-21413](https://img.shields.io/badge/CVE-2024--21413-critical?style=flat-square&color=red)](https://nvd.nist.gov/vuln/detail/CVE-2024-21413)
[![Category](https://img.shields.io/badge/Category-Credential%20Theft-blue?style=flat-square)](#)

### This my analysis on how a bypass in Microsoft Outlook's "Protected View" allows attackers to force NTLM authentication to a remote server.

During this lab I learned how a single character (an exclamation point) turns a simple link into an automated **"ID thief"**. By exploiting a bypass that tricks Outlook's usual security checks through an exclamation point embedded in an email link (**"!"**). I managed to bypass **"protected view"** completely. This let me force a user's computer to hand over its NTLM hashes without the user ever knowing. Doing this myself and seeing the hash pop up instantly made me realise why it was rated as a **9.8 Critical Vulnerability**.

### So what actually is a Moniker?

As microsoft states: **"A moniker in COM is not only a way to identify an object—a moniker is also implemented as an object. This object provides services allowing a component to obtain a pointer to the object identified by the moniker. This process is referred to as binding."**  

> **Note:** **COM** stands for **Component Object Model**. It is the language that Microsoft uses to let other software components talk to each other.

In my own words, a **Moniker** is a **"Smart-Link"**. As a normal link just points to a file, a moniker link points to that file as well as carrying instructions on how that file should be opened or **"Bound"**. Binding is essentially the handshake. The computer isn't just looking at the link but actually opens a live connection to that file.
In relation to this exploit the **"!"** symbol triggers the **"binding process"**. This causes Outlook's usual security process to skip completely letting me force a connection with the user and jump to connecting them to my malicious server. 

Below is how I managed this: 























---
[⬅️ Back to Labs](../README.md)
