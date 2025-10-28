---
layout: default
title: "HERMES - System Monitoring"
lang: en
permalink: /en/projects/hermes/
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
                <i class="fas fa-server mr-2"></i>Professional Monitoring System
            </span>
        </div>
    </div>
</section>

<!-- Description -->
<section class="py-20">
    <div class="max-w-5xl mx-auto px-4">
        <div class="bg-white dark:bg-dark-navbar rounded-2xl shadow-2xl p-8 md:p-12">
            <h2 class="text-3xl font-bold mb-6 text-cyan-500">
                <i class="fas fa-info-circle mr-3"></i>About the Project
            </h2>
            <p class="text-lg text-gray-700 dark:text-gray-300 mb-6">
                <strong>HERMES</strong> is a complete monitoring and observability platform designed to supervise your entire IT infrastructure in real-time. Built on a modern stack (Grafana, Prometheus, Loki, InfluxDB), HERMES provides total visibility into your systems.
            </p>
            <p class="text-lg text-gray-700 dark:text-gray-300 mb-6">
                The system centralizes the collection of metrics, logs, and events from any source (servers, containers, applications, network equipment) and offers interactive dashboards to visualize and analyze data in real-time.
            </p>
            <div class="bg-cyan-100 dark:bg-cyan-900/20 border-l-4 border-cyan-500 p-4 rounded">
                <p class="text-cyan-800 dark:text-cyan-200">
                    <i class="fas fa-rocket mr-2"></i>
                    <strong>Open Source:</strong> HERMES is available on GitHub and can be deployed on any infrastructure (bare metal, VM, cloud).
                </p>
            </div>
        </div>
    </div>
</section>

<!-- Features -->
<section class="py-20 bg-gray-100 dark:bg-dark-navbar">
    <div class="max-w-7xl mx-auto px-4">
        <h2 class="text-4xl font-bold text-center mb-16">
            <i class="fas fa-star text-cyan-500 mr-3"></i>Key Features
        </h2>

        <div class="grid md:grid-cols-3 gap-8">
            <!-- Monitoring -->
            <div class="bg-white dark:bg-dark-main rounded-xl shadow-lg p-8 text-center">
                <div class="w-20 h-20 mx-auto bg-cyan-500 rounded-full flex items-center justify-center mb-4">
                    <i class="fas fa-tachometer-alt text-white text-3xl"></i>
                </div>
                <h3 class="text-2xl font-bold mb-4 text-cyan-500">Real-time Monitoring</h3>
                <p class="text-gray-600 dark:text-gray-400">
                    Continuous monitoring of system metrics (CPU, RAM, disk, network) with automatic alerts
                </p>
            </div>

            <!-- Logs -->
            <div class="bg-white dark:bg-dark-main rounded-xl shadow-lg p-8 text-center">
                <div class="w-20 h-20 mx-auto bg-blue-500 rounded-full flex items-center justify-center mb-4">
                    <i class="fas fa-file-alt text-white text-3xl"></i>
                </div>
                <h3 class="text-2xl font-bold mb-4 text-blue-500">Log Centralization</h3>
                <p class="text-gray-600 dark:text-gray-400">
                    Aggregation and search across all application and system logs via Loki and Fluentd
                </p>
            </div>

            <!-- Dashboards -->
            <div class="bg-white dark:bg-dark-main rounded-xl shadow-lg p-8 text-center">
                <div class="w-20 h-20 mx-auto bg-indigo-500 rounded-full flex items-center justify-center mb-4">
                    <i class="fas fa-chart-bar text-white text-3xl"></i>
                </div>
                <h3 class="text-2xl font-bold mb-4 text-indigo-500">Interactive Dashboards</h3>
                <p class="text-gray-600 dark:text-gray-400">
                    Advanced visualization with Grafana: graphs, alerts, annotations, dynamic variables
                </p>
            </div>
        </div>
    </div>
</section>

<!-- Tech Stack -->
<section class="py-20">
    <div class="max-w-7xl mx-auto px-4">
        <h2 class="text-4xl font-bold text-center mb-16">
            <i class="fas fa-layer-group text-cyan-500 mr-3"></i>Tech Stack
        </h2>

        <div class="grid md:grid-cols-2 gap-8">
            <div class="bg-white dark:bg-dark-navbar rounded-xl shadow-lg p-8">
                <h3 class="text-2xl font-bold mb-4 text-cyan-500">
                    <i class="fas fa-chart-area mr-2"></i>Grafana
                </h3>
                <p class="text-gray-700 dark:text-gray-300 mb-4">
                    Visualization and analytics platform with customizable dashboards.
                </p>
                <ul class="space-y-2 text-gray-600 dark:text-gray-400">
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Pre-built dashboards (Proxmox, pfSense, Node Exporter)</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Multi-channel alerts (email, Slack, webhook)</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Advanced variables and templating</li>
                </ul>
            </div>

            <div class="bg-white dark:bg-dark-navbar rounded-xl shadow-lg p-8">
                <h3 class="text-2xl font-bold mb-4 text-blue-500">
                    <i class="fas fa-database mr-2"></i>Prometheus
                </h3>
                <p class="text-gray-700 dark:text-gray-300 mb-4">
                    Time-series database optimized for metrics storage and querying.
                </p>
                <ul class="space-y-2 text-gray-600 dark:text-gray-400">
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Automatic metrics collection (pull model)</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>PromQL for advanced queries</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Service Discovery (Kubernetes, Consul)</li>
                </ul>
            </div>

            <div class="bg-white dark:bg-dark-navbar rounded-xl shadow-lg p-8">
                <h3 class="text-2xl font-bold mb-4 text-indigo-500">
                    <i class="fas fa-scroll mr-2"></i>Loki & Promtail
                </h3>
                <p class="text-gray-700 dark:text-gray-300 mb-4">
                    Prometheus-inspired logging system, optimized for indexing and search.
                </p>
                <ul class="space-y-2 text-gray-600 dark:text-gray-400">
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Multi-source log aggregation</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Labels for efficient filtering</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Log/metrics correlation</li>
                </ul>
            </div>

            <div class="bg-white dark:bg-dark-navbar rounded-xl shadow-lg p-8">
                <h3 class="text-2xl font-bold mb-4 text-cyan-600">
                    <i class="fas fa-hdd mr-2"></i>InfluxDB
                </h3>
                <p class="text-gray-700 dark:text-gray-300 mb-4">
                    High-performance time-series database for IoT and high-frequency data.
                </p>
                <ul class="space-y-2 text-gray-600 dark:text-gray-400">
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Optimized data compression</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Automatic retention policies (RP)</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>REST API and Flux query language</li>
                </ul>
            </div>
        </div>
    </div>
</section>

<!-- Architecture -->
<section class="py-20 bg-gray-100 dark:bg-dark-navbar">
    <div class="max-w-7xl mx-auto px-4">
        <h2 class="text-4xl font-bold text-center mb-16">
            <i class="fas fa-project-diagram text-cyan-500 mr-3"></i>Architecture & Deployment
        </h2>

        <div class="grid md:grid-cols-2 gap-8">
            <div class="bg-white dark:bg-dark-main rounded-xl shadow-lg p-8">
                <h3 class="text-2xl font-bold mb-4 text-cyan-500">
                    <i class="fab fa-docker mr-2"></i>Docker Containerization
                </h3>
                <p class="text-gray-700 dark:text-gray-300 mb-4">
                    Complete stack deployed via Docker Compose for simple and portable installation.
                </p>
                <ul class="space-y-2 text-gray-600 dark:text-gray-400">
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Service isolation</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Zero-downtime updates</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Data persistence (volumes)</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Private inter-service network</li>
                </ul>
            </div>

            <div class="bg-white dark:bg-dark-main rounded-xl shadow-lg p-8">
                <h3 class="text-2xl font-bold mb-4 text-blue-500">
                    <i class="fas fa-terminal mr-2"></i>Installation Scripts
                </h3>
                <p class="text-gray-700 dark:text-gray-300 mb-4">
                    Ultra-graphical Bash scripts with spinners, progress bars, and multi-OS support.
                </p>
                <ul class="space-y-2 text-gray-600 dark:text-gray-400">
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Support for Debian, Ubuntu, CentOS, RHEL</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Automatic OS detection</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Assisted configuration</li>
                    <li><i class="fas fa-check text-green-500 mr-2"></i>Prerequisites validation</li>
                </ul>
            </div>
        </div>
    </div>
</section>

<!-- Use Cases -->
<section class="py-20">
    <div class="max-w-7xl mx-auto px-4">
        <h2 class="text-4xl font-bold text-center mb-16">
            <i class="fas fa-briefcase text-cyan-500 mr-3"></i>Use Cases
        </h2>

        <div class="grid md:grid-cols-3 gap-8">
            <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg">
                <div class="text-4xl text-cyan-500 mb-4">
                    <i class="fas fa-building"></i>
                </div>
                <h4 class="font-bold text-xl mb-3">Enterprise Infrastructure</h4>
                <p class="text-gray-600 dark:text-gray-400">
                    Supervision of datacenters, Kubernetes clusters, load balancers, databases
                </p>
            </div>

            <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg">
                <div class="text-4xl text-blue-500 mb-4">
                    <i class="fas fa-cloud"></i>
                </div>
                <h4 class="font-bold text-xl mb-3">Cloud & Hybrid</h4>
                <p class="text-gray-600 dark:text-gray-400">
                    Multi-cloud monitoring (AWS, Azure, GCP) and hybrid on-premise/cloud architectures
                </p>
            </div>

            <div class="bg-white dark:bg-dark-navbar p-6 rounded-xl shadow-lg">
                <div class="text-4xl text-indigo-500 mb-4">
                    <i class="fas fa-microchip"></i>
                </div>
                <h4 class="font-bold text-xl mb-3">IoT & Edge Computing</h4>
                <p class="text-gray-600 dark:text-gray-400">
                    Metrics collection from IoT sensors, edge devices, industrial equipment
                </p>
            </div>
        </div>
    </div>
</section>

<!-- Skills -->
<section class="py-20 bg-gray-100 dark:bg-dark-navbar">
    <div class="max-w-7xl mx-auto px-4">
        <h2 class="text-4xl font-bold text-center mb-16">
            <i class="fas fa-award text-cyan-500 mr-3"></i>Skills Developed
        </h2>

        <div class="grid md:grid-cols-4 gap-6">
            <div class="bg-white dark:bg-dark-main p-6 rounded-xl shadow-lg text-center">
                <i class="fas fa-server text-4xl text-cyan-500 mb-3"></i>
                <h4 class="font-bold mb-2">DevOps</h4>
                <p class="text-sm text-gray-600 dark:text-gray-400">CI/CD, IaC, containerization</p>
            </div>
            <div class="bg-white dark:bg-dark-main p-6 rounded-xl shadow-lg text-center">
                <i class="fas fa-chart-line text-4xl text-blue-500 mb-3"></i>
                <h4 class="font-bold mb-2">Observability</h4>
                <p class="text-sm text-gray-600 dark:text-gray-400">Metrics, logs, traces (MELT)</p>
            </div>
            <div class="bg-white dark:bg-dark-main p-6 rounded-xl shadow-lg text-center">
                <i class="fab fa-linux text-4xl text-indigo-500 mb-3"></i>
                <h4 class="font-bold mb-2">System Administration</h4>
                <p class="text-sm text-gray-600 dark:text-gray-400">Linux, networking, Bash scripting</p>
            </div>
            <div class="bg-white dark:bg-dark-main p-6 rounded-xl shadow-lg text-center">
                <i class="fas fa-database text-4xl text-cyan-600 mb-3"></i>
                <h4 class="font-bold mb-2">Time-Series DB</h4>
                <p class="text-sm text-gray-600 dark:text-gray-400">PromQL, Flux, optimization</p>
            </div>
        </div>
    </div>
</section>

<!-- Additional Technologies -->
<section class="py-20">
    <div class="max-w-7xl mx-auto px-4">
        <h2 class="text-4xl font-bold text-center mb-12">
            <i class="fas fa-tools text-cyan-500 mr-3"></i>Technologies & Tools
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

<!-- Links -->
<section class="py-20 bg-gray-100 dark:bg-dark-navbar">
    <div class="max-w-5xl mx-auto px-4 text-center">
        <h2 class="text-3xl font-bold mb-8">
            <i class="fas fa-link text-cyan-500 mr-3"></i>Project Links
        </h2>
        <div class="flex flex-wrap justify-center gap-4">
            <a href="https://github.com/Ventaryss/HERMES" target="_blank" class="inline-block bg-gray-800 hover:bg-gray-900 text-white font-bold py-3 px-8 rounded-lg shadow-lg transition duration-300">
                <i class="fab fa-github mr-2"></i>Source Code on GitHub
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
        <a href="{{ site.baseurl }}/en/projects/" class="inline-block bg-cyan-500 hover:bg-cyan-600 text-white font-bold py-3 px-8 rounded-lg shadow-lg transition duration-300">
            <i class="fas fa-arrow-left mr-2"></i>Back to Projects
        </a>
    </div>
</section>
