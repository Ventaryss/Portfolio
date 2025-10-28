---
layout: default
title: "DDF - Drone de Fer"
lang: fr
permalink: /fr/projects/ddf/
---

<!-- Hero Section -->
<section class="py-20 bg-gradient-to-br from-red-500/10 to-orange-500/10">
    <div class="max-w-7xl mx-auto px-4">
        <div class="text-center mb-12">
            <div class="inline-block p-6 bg-gradient-to-br from-red-500 to-orange-600 rounded-3xl shadow-2xl mb-6">
                <i class="fas fa-helicopter text-white text-7xl"></i>
            </div>
            <h1 class="text-5xl md:text-6xl font-bold mb-4 text-red-500">DDF</h1>
            <p class="text-2xl text-gray-600 dark:text-gray-400 mb-4">
                Drone de Fer v2.0 - Détection IA en Temps Réel
            </p>
            <span class="inline-block px-6 py-2 bg-amber-500 text-white rounded-full text-lg font-semibold">
                <i class="fas fa-lock mr-2"></i>Projet Confidentiel ESNA
            </span>
        </div>
    </div>
</section>

<!-- Description -->
<section class="py-20">
    <div class="max-w-5xl mx-auto px-4">
        <div class="bg-white dark:bg-dark-navbar rounded-2xl shadow-2xl p-8 md:p-12">
            <h2 class="text-3xl font-bold mb-6 text-red-500">
                <i class="fas fa-info-circle mr-3"></i>À propos du projet
            </h2>
            <p class="text-lg text-gray-700 dark:text-gray-300 mb-6">
                <strong>DDF (Drone de Fer)</strong> est un projet de fin d'études de 2 ans à l'<strong>ESNA Bretagne</strong>, consistant à développer un système de détection anti-drones par intelligence artificielle inspiré du célèbre <strong>Dôme de Fer</strong> israélien.
            </p>
            <p class="text-lg text-gray-700 dark:text-gray-300 mb-6">
                Le système utilise <strong>YOLOv11</strong> pour la détection en temps réel, <strong>Django 5.1</strong> comme backend, et une <strong>API REST complète</strong> pour l'intégration avec des systèmes de surveillance. Architecture optimisée avec lazy loading et threading pour des performances maximales (+40% FPS).
            </p>
            <div class="bg-amber-100 dark:bg-amber-900/20 border-l-4 border-amber-500 p-4 rounded">
                <p class="text-amber-800 dark:text-amber-200">
                    <i class="fas fa-exclamation-triangle mr-2"></i>
                    <strong>Note :</strong> Ce projet étant confidentiel dans le cadre de l'ESNA, les détails techniques et les résultats précis ne peuvent être divulgués publiquement.
                </p>
            </div>
        </div>
    </div>
</section>

<!-- Objectifs -->
<section class="py-20 bg-gray-100 dark:bg-dark-navbar">
    <div class="max-w-7xl mx-auto px-4">
        <h2 class="text-4xl font-bold text-center mb-16">
            <i class="fas fa-bullseye text-red-500 mr-3"></i>Objectifs du Projet
        </h2>

        <div class="grid md:grid-cols-3 gap-8">
            <!-- Détection -->
            <div class="bg-white dark:bg-dark-main rounded-xl shadow-lg p-8 text-center">
                <div class="w-20 h-20 mx-auto bg-red-500 rounded-full flex items-center justify-center mb-4">
                    <i class="fas fa-satellite-dish text-white text-3xl"></i>
                </div>
                <h3 class="text-2xl font-bold mb-4 text-red-500">Détection</h3>
                <p class="text-gray-600 dark:text-gray-400">
                    Identification automatique des drones dans un périmètre défini grâce à des capteurs avancés
                </p>
            </div>

            <!-- Tracking -->
            <div class="bg-white dark:bg-dark-main rounded-xl shadow-lg p-8 text-center">
                <div class="w-20 h-20 mx-auto bg-orange-500 rounded-full flex items-center justify-center mb-4">
                    <i class="fas fa-crosshairs text-white text-3xl"></i>
                </div>
                <h3 class="text-2xl font-bold mb-4 text-orange-500">Tracking</h3>
                <p class="text-gray-600 dark:text-gray-400">
                    Suivi en temps réel de la trajectoire et prédiction des mouvements du drone cible
                </p>
            </div>

            <!-- Neutralisation -->
            <div class="bg-white dark:bg-dark-main rounded-xl shadow-lg p-8 text-center">
                <div class="w-20 h-20 mx-auto bg-amber-500 rounded-full flex items-center justify-center mb-4">
                    <i class="fas fa-shield-alt text-white text-3xl"></i>
                </div>
                <h3 class="text-2xl font-bold mb-4 text-amber-500">Neutralisation</h3>
                <p class="text-gray-600 dark:text-gray-400">
                    Mise en œuvre de contre-mesures appropriées pour neutraliser la menace de manière contrôlée
                </p>
            </div>
        </div>
    </div>
</section>

<!-- Technologies -->
<section class="py-20">
    <div class="max-w-7xl mx-auto px-4">
        <h2 class="text-4xl font-bold text-center mb-16">
            <i class="fas fa-microchip text-red-500 mr-3"></i>Domaines Techniques Explorés
        </h2>

        <div class="grid md:grid-cols-2 gap-8">
            <div class="bg-white dark:bg-dark-navbar rounded-xl shadow-lg p-8">
                <h3 class="text-2xl font-bold mb-4 text-red-500">
                    <i class="fas fa-camera mr-2"></i>Intelligence Artificielle
                </h3>
                <p class="text-gray-700 dark:text-gray-300 mb-4">
                    Détection par IA avec YOLOv11 (Ultralytics 8.3.34) et PyTorch 2.5.1 pour une précision maximale.
                </p>
                <ul class="space-y-2 text-gray-600 dark:text-gray-400">
                    <li><i class="fas fa-check text-green-500 mr-2"></i>YOLOv11 - Détection ultra-rapide</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Lazy loading (-87% temps chargement)</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Threading optimisé (+40% FPS)</li>
                </ul>
            </div>

            <div class="bg-white dark:bg-dark-navbar rounded-xl shadow-lg p-8">
                <h3 class="text-2xl font-bold mb-4 text-orange-500">
                    <i class="fab fa-python mr-2"></i>Backend Django
                </h3>
                <p class="text-gray-700 dark:text-gray-300 mb-4">
                    Backend complet avec Django 5.1.3 et Django REST Framework 3.15.2 pour une gestion professionnelle.
                </p>
                <ul class="space-y-2 text-gray-600 dark:text-gray-400">
                    <li><i class="fas fa-check text-green-500 mr-2"></i>API REST complète (15+ endpoints)</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Authentification par token</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Modèle Detection (historique)</li>
                </ul>
            </div>

            <div class="bg-white dark:bg-dark-navbar rounded-xl shadow-lg p-8">
                <h3 class="text-2xl font-bold mb-4 text-amber-500">
                    <i class="fas fa-server mr-2"></i>Architecture Web
                </h3>
                <p class="text-gray-700 dark:text-gray-300 mb-4">
                    Interface web complète avec live streaming vidéo et dashboard de surveillance en temps réel.
                </p>
                <ul class="space-y-2 text-gray-600 dark:text-gray-400">
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Live feed caméra temps réel</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Panel admin Django</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Dashboard capteurs</li>
                </ul>
            </div>

            <div class="bg-white dark:bg-dark-navbar rounded-xl shadow-lg p-8">
                <h3 class="text-2xl font-bold mb-4 text-yellow-500">
                    <i class="fas fa-cogs mr-2"></i>DevOps & Déploiement
                </h3>
                <p class="text-gray-700 dark:text-gray-300 mb-4">
                    Script d'installation automatisé multi-OS avec détection et configuration intelligente.
                </p>
                <ul class="space-y-2 text-gray-600 dark:text-gray-400">
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Support multi-OS (Debian, Ubuntu, RHEL)</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Configuration .env sécurisée</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Logs structurés avec rotation</li>
                </ul>
            </div>
        </div>
    </div>
</section>

<!-- Contexte -->
<section class="py-20 bg-gray-100 dark:bg-dark-navbar">
    <div class="max-w-5xl mx-auto px-4">
        <h2 class="text-4xl font-bold text-center mb-12">
            <i class="fas fa-graduation-cap text-red-500 mr-3"></i>Cadre du Projet
        </h2>
        
        <div class="bg-white dark:bg-dark-main rounded-xl shadow-lg p-8">
            <div class="grid md:grid-cols-2 gap-8">
                <div>
                    <h3 class="text-xl font-bold mb-4 text-red-500">
                        <i class="fas fa-university mr-2"></i>Formation
                    </h3>
                    <p class="text-gray-700 dark:text-gray-300 mb-3">
                        <strong>École :</strong> ESNA Bretagne
                    </p>
                    <p class="text-gray-700 dark:text-gray-300 mb-3">
                        <strong>Type :</strong> Projet de fin d'études
                    </p>
                    <p class="text-gray-700 dark:text-gray-300">
                        <strong>Durée :</strong> 2 ans
                    </p>
                </div>
                <div>
                    <h3 class="text-xl font-bold mb-4 text-orange-500">
                        <i class="fas fa-users mr-2"></i>Équipe
                    </h3>
                    <p class="text-gray-700 dark:text-gray-300 mb-3">
                        Projet réalisé en équipe pluridisciplinaire
                    </p>
                    <p class="text-gray-700 dark:text-gray-300 mb-3">
                        Collaboration étudiants ingénieurs
                    </p>
                    <p class="text-gray-700 dark:text-gray-300">
                        Encadrement par des experts du domaine
                    </p>
                </div>
            </div>
        </div>
    </div>
</section>

<!-- Compétences Acquises -->
<section class="py-20">
    <div class="max-w-7xl mx-auto px-4">
        <h2 class="text-4xl font-bold text-center mb-16">
            <i class="fas fa-award text-amber-500 mr-3"></i>Compétences Développées
        </h2>

        <div class="grid md:grid-cols-4 gap-6">
            <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg text-center">
                <i class="fas fa-brain text-4xl text-red-500 mb-3"></i>
                <h4 class="font-bold mb-2">IA & ML</h4>
                <p class="text-sm text-gray-600 dark:text-gray-400">Intelligence artificielle appliquée</p>
            </div>
            <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg text-center">
                <i class="fas fa-code text-4xl text-orange-500 mb-3"></i>
                <h4 class="font-bold mb-2">Développement</h4>
                <p class="text-sm text-gray-600 dark:text-gray-400">Python, systèmes temps réel</p>
            </div>
            <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg text-center">
                <i class="fas fa-project-diagram text-4xl text-amber-500 mb-3"></i>
                <h4 class="font-bold mb-2">Gestion de Projet</h4>
                <p class="text-sm text-gray-600 dark:text-gray-400">Conduite de projet longue durée</p>
            </div>
            <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg text-center">
                <i class="fas fa-handshake text-4xl text-yellow-500 mb-3"></i>
                <h4 class="font-bold mb-2">Travail d'Équipe</h4>
                <p class="text-sm text-gray-600 dark:text-gray-400">Collaboration pluridisciplinaire</p>
            </div>
        </div>
    </div>
</section>

<!-- Navigation -->
<section class="py-12">
    <div class="max-w-5xl mx-auto px-4 text-center">
        <a href="{{ site.baseurl }}/fr/projects/" class="inline-block bg-red-500 hover:bg-red-600 text-white font-bold py-3 px-8 rounded-lg shadow-lg transition duration-300">
            <i class="fas fa-arrow-left mr-2"></i>Retour aux projets
        </a>
    </div>
</section>
