# 🥒 Pickle Rick CTF: Full-Chain Exploitation

### This writeup documents my thought process on my successful exploitation of a web server in order to get the "secret ingredients" required to make a potion. The goal was to gain root access to achieve full system compromise.

## Enumeration 

To begin the enumeration phase, I did an **nmap** scan to check for open ports and running server versions:


<img width="672" height="25" alt="image" src="https://github.com/user-attachments/assets/2440d8b3-b67d-48ec-a14e-22aa528dd518" />

For the command I used the following flags: 

- **-sV** to determine what software and version the target IP is running.
- **-sC** to run default **nmap** scripts that check for common vulnerabilities.
- **-oA** to save all the results into three different file types (normal, grepable, XML) to make it easier to search through data later if needs be. It is also good for future-proofing my work if I accidentally close the terminal.

>[!NOTE]
>I understand that for this CTF, saving the output may not have been necessary but I know it is good practice to do so.

<img width="740" height="111" alt="image" src="https://github.com/user-attachments/assets/3b4b5dca-e474-4fcf-8056-36ecdada91c0" />

My focus was **port 80** as it is hosting a web service and it is my entry point for potential exploitation. **Port 22** is **ssh** so it would be difficult to do anything with it as it is built to be secure making it harder to exploit. The version results also confirm this.

---

## Directory Brute-Forcing

Moving onto the web server, I didn't find anything significant on the page itself so I viewed the page source and found this written in HTML code: 

<img width="296" height="76" alt="image" src="https://github.com/user-attachments/assets/43885af7-4a94-44dc-93e3-f2dec1a800cc" />

Noting this down, my next step was to search for hidden directories. Seeing username credentials instantly made me think to find a hidden login page and other directories that may have more hidden information I could use.

My method of doing this was using the **gobuster** tool:

<img width="1282" height="22" alt="image" src="https://github.com/user-attachments/assets/93637010-b4c6-4151-9e3f-2559ae93104b" />

The flags I used are:

- **dir** to tell gobuster to use directory/file enumeration mode.
- **-u** to specify the target URL to be scanned.
- **-w** to specify the wordlist to be used in the scan. I used **directory-list-2.3-medium.txt** as it contains common directory names used in web environments.
- **-x** tells the tool to check for each word in the list as a folder and a specific file type.

>[!NOTE]
>I specified **php** as an extension as this website runs on a Linux server and php is the language it uses. I also chose **txt** because there may be hidden information in those files and **html** because it allows the discovery of other pages.

These were the results: 

<img width="431" height="189" alt="image" src="https://github.com/user-attachments/assets/fd068864-b3a9-458c-8dc1-c0edf4fa1cb3" />
































---
[⬅️ Back to Labs](../README.md)
