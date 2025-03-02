---
title: 02 - Service Scanning
date: 2025-03-01T13:29
tags:
  - nmap
  - ftp
  - smb
  - smbclient
  - snmpwalk
  - onesixtyone
---

---


##  Cheatsheet
> **Quick commands, syntax, or one-liners you use most often for this specific topic.**

| Command                                       | Description                                                                                                                       | Tag                |
| --------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- | ------------------ |
| `nmap -sC -sV -p- <target>`                   | Performs a full port scan on the target, using default scripts (`-sC`) and service/version detection (`-sV`) on all 65,535 ports. | Recon, Enumeration |
| `nmap --script <script_name> -p<port> <host>` | Runs a specific NSE script (e.g., `citrix-enum-apps.nse`) against the specified port(s) on the host.                              | Recon, Script      |
| `nmap -sV --script=banner -p<port> <host>`    | Banner-grabbing scan: attempts to reveal the service version by fetching the banner.                                              | Recon, Enumeration |
| `ftp <target>`                                | Connects via FTP. May allow anonymous login or reveal useful directories (e.g., `pub`).                                           | Enumeration, File  |
| `smbclient -N -L \\\\<target>`                | Lists SMB shares on a target, suppressing password prompt (`-N`).                                                                 | Enumeration, SMB   |
| `smbclient \\\\<target>\\<sharename>`         | Connects to a specific SMB share. Use `-U <user>` and then provide a password to authenticate.                                    | Enumeration, SMB   |
| `snmpwalk -v 2c -c <community> <host>`        | Enumerates SNMP information using a known community string (e.g., `public`).                                                      | Enumeration, SNMP  |
| `onesixtyone -c dict.txt 10.129.42.254`       | Brute forces the community string names using a dictionary file of common community strings such as the dict.txt file included in the GitHub repo for the tool. | Enumeration, SNMP |                                                                                                                                 |                    |


---

## Overview

High-level summary of what this note is about:

- **Purpose**: To discover active services, identify potential vulnerabilities, and gather information for further exploitation.  
- **Common Ports/Protocols**:
  - **FTP (21)**: Commonly used for file transfers. May allow anonymous access.
  - **SSH (22)**: Secure shell; indicates a *nix system, but can also be configured on Windows.
  - **HTTP (80)**: Web services; may reveal information through web page headers or banners.
  - **SMB (139/445)**: Windows and Samba shares; can leak sensitive data if misconfigured.
  - **SNMP (161)**: Network device stats gathering; default community strings are often “public” or “private.”
- **Typical Use Cases**: Network recon, vulnerability assessment, identifying OS versions, and mapping out potential attack vectors.

---

## Key Steps / Workflow

1. **Step 1: Basic Port Scan**  
   - Run a simple `nmap <target>` to quickly identify the top 1,000 open ports and services.  

2. **Step 2: Service & Version Enumeration**  
   - Use `nmap -sC -sV -p- <target>` to scan all ports, detect service versions, and run default scripts for deeper insight.  

3. **Step 3: Targeted Enumeration**  
   - Investigate specific services using specialized tools:
     - **FTP**: Anonymous login checks, directory listing.  
     - **SMB**: Share enumeration with `smbclient`; look for accessible files or misconfigurations.  
     - **SNMP**: Attempt common community strings with `snmpwalk` or brute force with `onesixtyone`.  
   - Use Nmap scripts or manual checks (e.g., netcat) to grab banners and confirm versions.

---

## Tools & Usage

- **Nmap**[^2]: Primary tool for port scanning and service detection.
  - `nmap -sC -sV -p- <target>` for a comprehensive scan.
  - `nmap --script smb-os-discovery.nse -p445 <target>` for enumerating SMB/OS details.
  - `nmap -sV --script=banner -p21 <host>` for banner grabbing on FTP, etc.

- **Netcat (nc)**:
  - `nc -nv <host> <port>` to manually connect and retrieve service banners.

- **FTP Client**:
  - `ftp <target>`  
  - Check for anonymous login (use `anonymous` as the username, often empty password).

- **SMBClient**:
  - `smbclient -N -L \\<target>` (list shares anonymously)
  - `smbclient -U <user> \\<target>\<share>` (connect to a share with credentials)

- **SNMP**:
  - `snmpwalk -v 2c -c public <host> 1.3.6.1.2.1.1.5.0` to test for default community strings.
  - `onesixtyone -c dict.txt <target>` to brute force common community strings.


---

## Detailed Notes / Explanations

- **Port & Service Basics**:  
  IP addresses identify hosts, while port numbers identify specific services. Common services have default ports; unusual port assignments could be a sign of obscured/misconfigured applications.

- **TCP vs. UDP**:  
  By default, Nmap uses TCP scans; UDP scans require additional flags (`-sU`) and can be slower or more prone to false positives.

- **OS Detection**:  
  Sometimes version info from services (like SSH banners) suggests the target OS. For instance, `OpenSSH 8.2p1 Ubuntu <...>` often points to Ubuntu Focal Fossa 20.04.

- **Banner Grabbing**:  
  Fetches the initial text shown by a service. Nmap’s NSE scripts or Netcat can do this manually. Banners often reveal service versions.

- **FTP Enumeration**:
  - Anonymous access might expose directories (like `pub`) and sensitive files (e.g., `login.txt` with credentials).

- **SMB Enumeration**:
  - Look for shares. Some may allow read or write access. Credentials can unlock more data. Samba on Linux or SMB on Windows can leak OS details.

- **SNMP Enumeration**:
  - Default community strings (e.g., `public`, `private`) are common. Tools like `snmpwalk` or `onesixtyone`[^1]can reveal system info or running processes.

---

## Gotchas / Pitfalls

1. **Filtered Ports**: Firewalls may show ports as filtered, hiding services from normal scans.
2. **False Service Mappings**: The default Nmap service name might not match the actual running service unless version detection confirms it.
3. **Time Constraints**: Full scans (`-p-`) plus scripts (`-sC`, `-sV`) can be slow. Plan accordingly.

---

## Command Metadata

bash_command:: nmap -sC -sV -p- <target>
bash_note:: Perform a thorough port scan with default scripts and version detection on all ports.


## References

[^1]: https://github.com/trailofbits/onesixtyone
[^2]: https://nmap.org/docs.html