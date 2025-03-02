---
title: gobuster
date: 2025-03-01T12:12
tags:
  - gobuster
---

---

##  Cheatsheet
> **Quick commands, syntax, or one-liners you use most often for this specific topic.**

| Command                                                          | Description                                                                                                      | Tag                |
| ---------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- | ------------------ |
| `gobuster dir -u http://example.com -w /path/to/wordlist.txt`    | Directory and file brute force on a website.                                                                     | Recon, Enumeration |
| `gobuster dns -d example.com -w /path/to/wordlist.txt`           | Subdomain brute force against a domain.                                                                          | Recon, Enumeration |
| `gobuster vhost -u http://example.com -w /path/to/wordlist.txt`  | Virtual host enumeration on a target web server.                                                                 | Recon, Discovery   |
| `gobuster s3 -w /path/to/wordlist.txt`                           | Enumerate open Amazon S3 buckets listed in the wordlist.                                                         | Cloud, Enumeration |
| `gobuster gcs -w /path/to/wordlist.txt`                          | Enumerate open Google Cloud Storage buckets listed in the wordlist.                                             | Cloud, Enumeration |
| `gobuster tftp -u 192.168.1.100 -w /path/to/wordlist.txt`        | Enumerate files on a TFTP server using wordlist entries.                                                         | Recon, Enumeration |
| `gobuster dir -u http://example.com -w /path/to/wordlist.txt -x php,txt,html` | Include multiple file extensions in the directory brute force.                                    | Recon, Enumeration |
| `gobuster dir -u http://example.com -w /path/to/wordlist.txt -t 50 -o results.txt` | Run with 50 threads and save output to `results.txt` file.                              | Recon, Enumeration |


## Overview

### Purpose:

Gobuster is a powerful, fast, and flexible brute-force tool for enumerating hidden resources or discovering misconfigured services. It enables pentesters and security professionals to discover files, directories, subdomains, and more by quickly testing large numbers of potential paths or identifiers.


### Typical Use Cases:

- Web Application Testing: Enumerate hidden directories and files to find admin panels, config files, backups, etc.
- Subdomain Enumeration: Identify potential subdomains that might lead to additional attack surfaces.
- Virtual Host Discovery: Find virtual host configurations running on the same server.
- Cloud Storage Enumeration: Check for publicly accessible or misconfigured S3/GCS buckets.
- TFTP Server Enumeration: Discover files available on TFTP servers.


## Key Steps / Workflow

Below is a typical step-by-step approach for using Gobuster in a pentest or security assessment:

1. Gather Requirements
	1. Identify the target (e.g., domain, IP, or URL).
	2. Choose the proper mode (dir, dns, vhost, s3, gcs, or tftp).
	3. Select (or create) a suitable wordlist.
2. Step 2: Configure Gobuster Options
	1. Set the target with -u or -d (for DNS).
	2. Provide the wordlist with -w.
	3. Adjust additional options (threads, extensions, output file, etc.) based on the environment and goals.
3. Step 3: Run and Review Results
	1. Execute Gobuster in the chosen mode.
	2. Monitor for potential false positives (especially in DNS brute-forcing with wildcards).
	3. Analyze discovered paths, subdomains, or buckets.
	4. Log critical findings and determine next steps (e.g., further enumeration, exploitation attempts).
	
## Example Usage

```bash
gobuster dns -d <target-domain> -w /usr/share/SecLists/Discovery/DNS/namelist.txt
```

```bash
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Domain:     inlanefreight.com
[+] Threads:    10
[+] Timeout:    1s
[+] Wordlist:   /usr/share/SecLists/Discovery/DNS/namelist.txt
===============================================================
2020/12/17 23:08:55 Starting gobuster
===============================================================
Found: blog.inlanefreight.com
Found: customer.inlanefreight.com
Found: my.inlanefreight.com
Found: ns1.inlanefreight.com
Found: ns2.inlanefreight.com
Found: ns3.inlanefreight.com
===============================================================
2020/12/17 23:10:34 Finished
===============================================================

```


    Tool 1: tool1 --options
    Tool 2: tool2 --example

## Detailed Notes / Explanations

    Deeper technical details, references to known CVEs, configuration notes, or specific edge cases.

    Concept A: Explanation
    Concept B: Explanation

## Gotchas / Pitfalls

    Common mistakes, edge cases, or tricky scenarios worth remembering.

    Pitfall 1:
    Pitfall 2:

---
## Command Metadata

bash_command:: test123
bash_note:: test111 

---
## References

    Links, external docs, or other notes you want to remember.

    Nmap Documentation
    HackTricks
    [[Another_Obsidian_Note]] (Internal link example)
