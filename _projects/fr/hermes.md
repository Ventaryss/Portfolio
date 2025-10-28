---
layout: default
title: "HERMES - Monitoring Système"
lang: fr
permalink: /fr/projects/hermes/
---

<!-- Hero Section -->
<section class="py-20 bg-gradient-to-br from-cyan-500/10 to-blue-500/10">
    <div class="max-w-7xl mx-auto px-4">
        <div class="text-center mb-12">
            <div class="inline-block p-6 bg-gradient-to-br from-cyan-500 to-blue-600 rounded-3xl shadow-2xl mb-6">
                <i class="fas fa-chart-line text-white text-7xl"></i>
            </div>
            <h1 class="text-5xl md:text-6xl font-bold mb-4 text-cyan-500">HERMES</h1>
            <p class="text-2xl text-gray-600 dark:text-gray-400 mb-4">
                Highly Efficient Real-time Monitoring and Event System
            </p>
            <span class="inline-block px-6 py-2 bg-cyan-500 text-white rounded-full text-lg font-semibold">
                <i class="fas fa-server mr-2"></i>Système de Monitoring Professionnel
            </span>
        </div>
    </div>
</section>

<!-- Description -->
<section class="py-20">
    <div class="max-w-5xl mx-auto px-4">
        <div class="bg-white dark:bg-dark-navbar rounded-2xl shadow-2xl p-8 md:p-12">
            <h2 class="text-3xl font-bold mb-6 text-cyan-500">
                <i class="fas fa-info-circle mr-3"></i>À propos du projet
            </h2>
            <p class="text-lg text-gray-700 dark:text-gray-300 mb-6">
                <strong>HERMES</strong> est une plateforme complète de monitoring et d'observabilité conçue pour superviser l'ensemble de votre infrastructure IT en temps réel. Basé sur une stack moderne (Grafana, Prometheus, Loki, InfluxDB), HERMES offre une visibilité totale sur vos systèmes.
            </p>
            <p class="text-lg text-gray-700 dark:text-gray-300 mb-6">
                Le système centralise la collecte de métriques, logs et événements depuis n'importe quelle source (serveurs, conteneurs, applications, équipements réseau) et propose des dashboards interactifs pour visualiser et analyser les données en temps réel.
            </p>
            <div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 rounded">
                <p class="text-cyan-800 dark:text-cyan-200">
                    <i class="fas fa-rocket mr-2"></i>
                    <strong>Open Source :</strong> HERMES est disponible sur GitHub et peut être déployé sur n'importe quelle infrastructure (bare metal, VM, cloud).
                </p>
            </div>
        </div>
    </div>
</section>

<!-- Fonctionnalités -->
<section class="py-20 bg-gray-100 dark:bg-dark-navbar">
    <div class="max-w-7xl mx-auto px-4">
        <h2 class="text-4xl font-bold text-center mb-16">
            <i class="fas fa-star text-cyan-500 mr-3"></i>Fonctionnalités Principales
        </h2>

        <div class="grid md:grid-cols-3 gap-8">
            <!-- Monitoring -->
            <div class="bg-white dark:bg-dark-main rounded-xl shadow-lg p-8 text-center">
                <div class="w-20 h-20 mx-auto bg-cyan-500 rounded-full flex items-center justify-center mb-4">
                    <i class="fas fa-tachometer-alt text-white text-3xl"></i>
                </div>
                <h3 class="text-2xl font-bold mb-4 text-cyan-500">Monitoring Temps Réel</h3>
                <p class="text-gray-600 dark:text-gray-400">
                    Surveillance continue des métriques système (CPU, RAM, disque, réseau) avec alertes automatiques
                </p>
            </div>

            <!-- Logs -->
            <div class="bg-white dark:bg-dark-main rounded-xl shadow-lg p-8 text-center">
                <div class="w-20 h-20 mx-auto bg-blue-500 rounded-full flex items-center justify-center mb-4">
                    <i class="fas fa-file-alt text-white text-3xl"></i>
                </div>
                <h3 class="text-2xl font-bold mb-4 text-blue-500">Centralisation Logs</h3>
                <p class="text-gray-600 dark:text-gray-400">
                    Agrégation et recherche dans tous les logs applicatifs et système via Loki et Fluentd
                </p>
            </div>

            <!-- Dashboards -->
            <div class="bg-white dark:bg-dark-main rounded-xl shadow-lg p-8 text-center">
                <div class="w-20 h-20 mx-auto bg-indigo-500 rounded-full flex items-center justify-center mb-4">
                    <i class="fas fa-chart-bar text-white text-3xl"></i>
                </div>
                <h3 class="text-2xl font-bold mb-4 text-indigo-500">Dashboards Interactifs</h3>
                <p class="text-gray-600 dark:text-gray-400">
                    Visualisation avancée avec Grafana : graphiques, alertes, annotations, variables dynamiques
                </p>
            </div>
        </div>
    </div>
</section>

<!-- Stack Technique -->
<section class="py-20">
    <div class="max-w-7xl mx-auto px-4">
        <h2 class="text-4xl font-bold text-center mb-16">
            <i class="fas fa-layer-group text-cyan-500 mr-3"></i>Stack Technique
        </h2>

        <div class="grid md:grid-cols-2 gap-8">
            <div class="bg-white dark:bg-dark-navbar rounded-xl shadow-lg p-8">
                <h3 class="text-2xl font-bold mb-4 text-cyan-500">
                    <i class="fas fa-chart-area mr-2"></i>Grafana
                </h3>
                <p class="text-gray-700 dark:text-gray-300 mb-4">
                    Plateforme de visualisation et d'analyse de métriques avec dashboards personnalisables.
                </p>
                <ul class="space-y-2 text-gray-600 dark:text-gray-400">
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Dashboards préconçus (Proxmox, pfSense, Node Exporter)</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Alertes multi-canaux (email, Slack, webhook)</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Variables et templating avancés</li>
                </ul>
            </div>

            <div class="bg-white dark:bg-dark-navbar rounded-xl shadow-lg p-8">
                <h3 class="text-2xl font-bold mb-4 text-blue-500">
                    <i class="fas fa-database mr-2"></i>Prometheus
                </h3>
                <p class="text-gray-700 dark:text-gray-300 mb-4">
                    Base de données time-series optimisée pour le stockage et requêtage de métriques.
                </p>
                <ul class="space-y-2 text-gray-600 dark:text-gray-400">
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Collecte automatique des métriques (pull model)</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>PromQL pour requêtes avancées</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Service Discovery (Kubernetes, Consul)</li>
                </ul>
            </div>

            <div class="bg-white dark:bg-dark-navbar rounded-xl shadow-lg p-8">
                <h3 class="text-2xl font-bold mb-4 text-indigo-500">
                    <i class="fas fa-scroll mr-2"></i>Loki & Promtail
                </h3>
                <p class="text-gray-700 dark:text-gray-300 mb-4">
                    Système de logging inspiré de Prometheus, optimisé pour l'indexation et la recherche.
                </p>
                <ul class="space-y-2 text-gray-600 dark:text-gray-400">
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Agrégation logs multi-sources</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Labels pour filtrage performant</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Corrélation logs/métriques</li>
                </ul>
            </div>

            <div class="bg-white dark:bg-dark-navbar rounded-xl shadow-lg p-8">
                <h3 class="text-2xl font-bold mb-4 text-cyan-600">
                    <i class="fas fa-hdd mr-2"></i>InfluxDB
                </h3>
                <p class="text-gray-700 dark:text-gray-300 mb-4">
                    Base de données time-series haute performance pour IoT et données à forte fréquence.
                </p>
                <ul class="space-y-2 text-gray-600 dark:text-gray-400">
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Compression optimisée des données</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Rétention automatique (RP)</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>API REST et Flux query language</li>
                </ul>
            </div>
        </div>
    </div>
</section>

<!-- Architecture -->
<section class="py-20 bg-gray-100 dark:bg-dark-navbar">
    <div class="max-w-7xl mx-auto px-4">
        <h2 class="text-4xl font-bold text-center mb-16">
            <i class="fas fa-project-diagram text-cyan-500 mr-3"></i>Architecture & Déploiement
        </h2>

        <div class="grid md:grid-cols-2 gap-8">
            <div class="bg-white dark:bg-dark-main rounded-xl shadow-lg p-8">
                <h3 class="text-2xl font-bold mb-4 text-cyan-500">
                    <i class="fab fa-docker mr-2"></i>Conteneurisation Docker
                </h3>
                <p class="text-gray-700 dark:text-gray-300 mb-4">
                    Stack complète déployée via Docker Compose pour une installation simple et portable.
                </p>
                <ul class="space-y-2 text-gray-600 dark:text-gray-400">
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Isolation des services</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Mise à jour sans downtime</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Persistance des données (volumes)</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Network privé inter-services</li>
                </ul>
            </div>

            <div class="bg-white dark:bg-dark-main rounded-xl shadow-lg p-8">
                <h3 class="text-2xl font-bold mb-4 text-blue-500">
                    <i class="fas fa-terminal mr-2"></i>Scripts d'Installation
                </h3>
                <p class="text-gray-700 dark:text-gray-300 mb-4">
                    Scripts Bash ultra-graphiques avec spinners, barres de progression et support multi-OS.
                </p>
                <ul class="space-y-2 text-gray-600 dark:text-gray-400">
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Support Debian, Ubuntu, CentOS, RHEL</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Détection automatique de l'OS</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Configuration assistée</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Validation des prérequis</li>
                </ul>
            </div>
        </div>
    </div>
</section>

<!-- Cas d'Usage -->
<section class="py-20">
    <div class="max-w-7xl mx-auto px-4">
        <h2 class="text-4xl font-bold text-center mb-16">
            <i class="fas fa-briefcase text-cyan-500 mr-3"></i>Cas d'Usage
        </h2>

        <div class="grid md:grid-cols-3 gap-8">
            <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg">
                <div class="text-4xl text-cyan-500 mb-4 text-center">
                    <i class="fas fa-building"></i>
                </div>
                <h4 class="font-bold text-xl mb-3">Infrastructure d'Entreprise</h4>
                <p class="text-gray-600 dark:text-gray-400">
                    Supervision de datacenters, clusters Kubernetes, load balancers, bases de données
                </p>
            </div>

            <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg">
                <div class="text-4xl text-blue-500 mb-4 text-center">
                    <i class="fas fa-cloud"></i>
                </div>
                <h4 class="font-bold text-xl mb-3">Cloud & Hybride</h4>
                <p class="text-gray-600 dark:text-gray-400">
                    Monitoring multi-cloud (AWS, Azure, GCP) et architectures on-premise/cloud hybrides
                </p>
            </div>

            <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg">
                <div class="text-4xl text-indigo-500 mb-4 text-center">
                    <i class="fas fa-microchip"></i>
                </div>
                <h4 class="font-bold text-xl mb-3">IoT & Edge Computing</h4>
                <p class="text-gray-600 dark:text-gray-400">
                    Collecte de métriques depuis capteurs IoT, edge devices, équipements industriels
                </p>
            </div>
        </div>
    </div>
</section>

<!-- Compétences -->
<section class="py-20 bg-gray-100 dark:bg-dark-navbar">
    <div class="max-w-7xl mx-auto px-4">
        <h2 class="text-4xl font-bold text-center mb-16">
            <i class="fas fa-award text-cyan-500 mr-3"></i>Compétences Développées
        </h2>

        <div class="grid md:grid-cols-4 gap-6">
            <div class="bg-white dark:bg-dark-main p-6 rounded-xl shadow-lg text-center">
                <i class="fas fa-server text-4xl text-cyan-500 mb-3"></i>
                <h4 class="font-bold mb-2">DevOps</h4>
                <p class="text-sm text-gray-600 dark:text-gray-400">CI/CD, IaC, conteneurisation</p>
            </div>
            <div class="bg-white dark:bg-dark-main p-6 rounded-xl shadow-lg text-center">
                <i class="fas fa-chart-line text-4xl text-blue-500 mb-3"></i>
                <h4 class="font-bold mb-2">Observabilité</h4>
                <p class="text-sm text-gray-600 dark:text-gray-400">Métriques, logs, traces (MELT)</p>
            </div>
            <div class="bg-white dark:bg-dark-main p-6 rounded-xl shadow-lg text-center">
                <i class="fab fa-linux text-4xl text-indigo-500 mb-3"></i>
                <h4 class="font-bold mb-2">Administration Système</h4>
                <p class="text-sm text-gray-600 dark:text-gray-400">Linux, réseaux, scripting Bash</p>
            </div>
            <div class="bg-white dark:bg-dark-main p-6 rounded-xl shadow-lg text-center">
                <i class="fas fa-database text-4xl text-cyan-600 mb-3"></i>
                <h4 class="font-bold mb-2">Time-Series DB</h4>
                <p class="text-sm text-gray-600 dark:text-gray-400">PromQL, Flux, optimisation</p>
            </div>
        </div>
    </div>
</section>

<!-- Technologies Additionnelles -->
<section class="py-20">
    <div class="max-w-7xl mx-auto px-4">
        <h2 class="text-4xl font-bold text-center mb-12">
            <i class="fas fa-tools text-cyan-500 mr-3"></i>Technologies & Outils
        </h2>

        <div class="flex flex-wrap justify-center gap-4">
            <span class="px-6 py-3 bg-cyan-500 text-white rounded-full font-semibold">
                <i class="fab fa-docker mr-2"></i>Docker
            </span>
            <span class="px-6 py-3 bg-blue-500 text-white rounded-full font-semibold">
                <i class="fas fa-chart-area mr-2"></i>Grafana
            </span>
            <span class="px-6 py-3 bg-indigo-500 text-white rounded-full font-semibold">
                <i class="fas fa-fire mr-2"></i>Prometheus
            </span>
            <span class="px-6 py-3 bg-purple-500 text-white rounded-full font-semibold">
                <i class="fas fa-scroll mr-2"></i>Loki
            </span>
            <span class="px-6 py-3 bg-cyan-600 text-white rounded-full font-semibold">
                <i class="fas fa-database mr-2"></i>InfluxDB
            </span>
            <span class="px-6 py-3 bg-gray-700 text-white rounded-full font-semibold">
                <i class="fas fa-stream mr-2"></i>Fluentd
            </span>
            <span class="px-6 py-3 bg-orange-500 text-white rounded-full font-semibold">
                <i class="fas fa-file-alt mr-2"></i>Rsyslog
            </span>
            <span class="px-6 py-3 bg-green-500 text-white rounded-full font-semibold">
                <i class="fab fa-linux mr-2"></i>Linux
            </span>
            <span class="px-6 py-3 bg-yellow-500 text-white rounded-full font-semibold">
                <i class="fas fa-terminal mr-2"></i>Bash
            </span>
        </div>
    </div>
</section>

<!-- Liens -->
<section class="py-20 bg-gray-100 dark:bg-dark-navbar">
    <div class="max-w-5xl mx-auto px-4 text-center">
        <h2 class="text-3xl font-bold mb-8">
            <i class="fas fa-link text-cyan-500 mr-3"></i>Liens du Projet
        </h2>
        <div class="flex flex-wrap justify-center gap-4">
            <a href="https://github.com/Ventaryss/HERMES" target="_blank" class="inline-block bg-gray-800 hover:bg-gray-900 text-white font-bold py-3 px-8 rounded-lg shadow-lg transition duration-300">
                <i class="fab fa-github mr-2"></i>Code Source GitHub
            </a>
            <a href="https://github.com/Ventaryss/HERMES/blob/main/README.md" target="_blank" class="inline-block bg-cyan-500 hover:bg-cyan-600 text-white font-bold py-3 px-8 rounded-lg shadow-lg transition duration-300">
                <i class="fas fa-book mr-2"></i>Documentation
            </a>
        </div>
    </div>
</section>

<!-- Navigation -->
<section class="py-12">
    <div class="max-w-5xl mx-auto px-4 text-center">
        <a href="{{ site.baseurl }}/fr/projects/" class="inline-block bg-cyan-500 hover:bg-cyan-600 text-white font-bold py-3 px-8 rounded-lg shadow-lg transition duration-300">
            <i class="fas fa-arrow-left mr-2"></i>Retour aux projets
        </a>
    </div>
</section>
