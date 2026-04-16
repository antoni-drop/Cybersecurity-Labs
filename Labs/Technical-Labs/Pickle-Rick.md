# 🥒 Pickle Rick CTF: Full-Chain Exploitation

### This writeup documents my thought process on my successful exploitation of a web server in order to get the "secret ingredients" required to make a potion. The goal was to gain root access to achieve full system compromise.

## 🔍 Enumeration 

To begin the enumeration phase, I did an **nmap** scan to check for open ports and running server versions:


<img width="672" height="25" alt="image" src="https://github.com/user-attachments/assets/2440d8b3-b67d-48ec-a14e-22aa528dd518" />

For the command I used the following flags: 

- **-sV** to determine what software and version the target IP is running.
- **-sC** to run default **nmap** scripts that check for common vulnerabilities.
- **-oA** to save all the results into three different file types (normal, grepable, XML) to make it easier to search through data later if needed. It is also good for future-proofing my work if I accidentally close the terminal.

>[!NOTE]
>I understand that for this CTF, saving the output may not have been necessary but I know it is good practice to do so.

<img width="740" height="111" alt="image" src="https://github.com/user-attachments/assets/3b4b5dca-e474-4fcf-8056-36ecdada91c0" />

My focus was **port 80** as it is hosting a web service and it is my entry point for potential exploitation. **Port 22** is **ssh** so it would be difficult to do anything with it as it is built to be secure making it harder to exploit. The version results confirm this.

---

## 🚪 Directory Brute-Forcing

Moving onto the web server, I did not find anything significant on the page itself so I viewed the page source and found this written in HTML code: 

<img width="296" height="76" alt="image" src="https://github.com/user-attachments/assets/43885af7-4a94-44dc-93e3-f2dec1a800cc" />

Noting this down, my next step was to search for hidden directories. Seeing username credentials instantly made me think to find a hidden login page and other directories that may have more hidden information I could use.

My method of doing this was using the **gobuster** tool:


<img width="1408" height="27" alt="image" src="https://github.com/user-attachments/assets/327b4a36-e0ee-4abf-9055-6fe850c651d1" />


The flags I used are:

- **dir** to tell gobuster to use directory/file enumeration mode.
- **-u** to specify the target URL to be scanned.
- **-w** to specify the wordlist to be used in the scan. I used **directory-list-2.3-medium.txt** as it contains common directory names used in web environments.
- **-x** tells the tool to check for each word in the list as a folder and a specific file type.

>[!NOTE]
>I specified **php** as an extension as this website runs on a Linux server and php is the language it uses. I also chose **txt** because there may be hidden information in those files and **html** because it allows the discovery of other pages.

These were the results: 

<img width="431" height="189" alt="image" src="https://github.com/user-attachments/assets/fd068864-b3a9-458c-8dc1-c0edf4fa1cb3" />

- **Status 200 (OK)** the request was successful and I have full permission to view the content.
- **Status 302 (Found/Redirect)** the source exists but it has been temporarily moved to another URL (for example, an admin page that exists but you don't have the login so it redirects you to the login page.) 
- **Status 403 (Forbidden)** the resource exists but the server is blocking my access.

## 🚩 Exploitation and Post-Exploitation Enumeration

I chose to look inside the **/robots.txt** file first as this is a high priority file to inspect as it often reveals sensitive directories and information the web developer is trying to hide. In the file I found a single word:

<img width="134" height="41" alt="image" src="https://github.com/user-attachments/assets/b1a0f9c6-5bfc-4b5f-8afd-99ae9e6f30ec" />

Obviously, this is a Rick and Morty reference and my first thought was that this could be the password that belongs to the username that I got from the page source. I decided to visit **/login.php** and I found the login page: 

<img width="379" height="547" alt="image" src="https://github.com/user-attachments/assets/efc0a831-c17c-4cb0-a1d0-226adff306df" />

I tried the username and password and my intuition was correct. After logging in I was redirected to a "Rick Portal" containing a command panel and different page elements: 

<img width="672" height="267" alt="image" src="https://github.com/user-attachments/assets/83e78dda-8a6c-43d5-a72f-ecf161f6ea8e" />

The only page element I had access to was the command panel. When I tried to access the other elements I was given this:

<img width="623" height="370" alt="image" src="https://github.com/user-attachments/assets/4184439c-e73a-4b79-b4c2-ac8998d789f5" />

This shows that the developer has put security measures in place so my next option from here was to use the command panel to try find more information. After using the **ls** command I found a list of different files:

<img width="236" height="170" alt="image" src="https://github.com/user-attachments/assets/304f5dbb-b343-4389-b9c9-6009896a31b8" />

I tried viewing the files using the **cat** command but I came across an obstacle as it was disabled which shows there must be a command blacklist:

<img width="461" height="194" alt="image" src="https://github.com/user-attachments/assets/97a7ae5f-11f5-45a7-a3d3-e550485df32d" />

This was the part of the CTF where I was stuck for a while as I couldn't find a way to view the files until I found the **less** command. 

<img width="230" height="31" alt="image" src="https://github.com/user-attachments/assets/b03ae6c4-6d2e-4032-9b92-aa5bca0df92a" />

Since this command wasn't blacklisted, I managed to view the file and find my first flag: 

<img width="152" height="36" alt="image" src="https://github.com/user-attachments/assets/a8ed6dd8-36cf-4eed-a9e4-89a2b6931e46" />

Moving on I viewed the clue.txt file:

<img width="440" height="41" alt="image" src="https://github.com/user-attachments/assets/284f7b1e-b193-4452-951e-0adee85138f9" />

I checked who I am on the system using the **whoami** command and used the **sudo -l** command to list my current privileges and noticed that there is no password set to use sudo:

<img width="931" height="113" alt="image" src="https://github.com/user-attachments/assets/69590f90-f452-4b53-bad9-77c7cdc5c04b" />

Checking the root directory using **ls /** I navigated to the **/home** directory to check what users were in the system: 

<img width="81" height="53" alt="image" src="https://github.com/user-attachments/assets/c44b8738-e45e-4bf2-b6ee-604fe1c45720" />

Using the **ls /home/rick** command I found a file named **"second ingredients"**. To view the contents of the file I used **less /home/rick/"second ingredients"** and got my second flag:

<img width="116" height="39" alt="image" src="https://github.com/user-attachments/assets/499581ba-03ab-40e8-bb70-9de7747b10a3" />

>[!NOTE]
>I was stuck here for 15 minutes figuring out why I couldn't view the **"second ingredients"** file. Until I realised to view it I needed to use quotation marks as there is a space between the words.

To get my final flag I checked the root directory, to view it I needed to use the **sudo ls /root** command as I needed super user privileges this time however this wasn't an issue as there was no password set to use sudo:

<img width="86" height="56" alt="image" src="https://github.com/user-attachments/assets/45a026aa-fe29-47df-809b-9edfaa0f085c" />

Using the command **sudo less /root/3rd.txt** I got my third and final flag!

<img width="239" height="39" alt="image" src="https://github.com/user-attachments/assets/311bf451-d3a6-4c18-8c80-ec2f2f0167e8" />

## Conclusion

This CTF was my first exercise on completing a full attack lifecycle where I was able to eventually gain root access: 

- **Leaked information** : Viewing the page source to find hidden credentials and the robots.txt file to find sensitive information. It shows how one piece of information could lead to a total breach of a web application.
- **Bypassing Blacklists** : Finding an alternative command to view files when **cat** was blocked. This made me think outside the box and stick to the method rather than looking for a new one.
- **Privilege Escalation** : Exploiting the misconfiguration of **sudo** privileges to be able to view the information I needed in the command panel. With the sudoers file not being configured properly, it allowed me to gain root access over the whole system.
- **Principle of Least Privilege** : Doing this writeup made me realise why this is so important, if the users were given minimal access then there would have been a lot less options to exploit the web server.











---
[⬅️ Back to Labs](../README.md)
