# SENUMPRO-ZABBIX-MASTERCLASS

Projet de supervision et d'administration avec Zabbix 7.0

Plateforme : Ubuntu Server 24.04 LTS
Solution : Zabbix 7.0.32
Base de données : MariaDB 10.11
Serveur Web : Nginx

Environnement : VMware – laboratoire virtualisé
Agents : Zabbix Agent 2 pour Linux et Windows Server 2025
Déploiement : Installation native, sans Docker

1.Présentation du projet

1.1 Contexte

Dans une infrastructure informatique, la disponibilité des serveurs, des services et des équipements réseau est essentielle au bon fonctionnement des activités. Une interruption de service, une saturation du stockage ou une dégradation des performances peut avoir des conséquences importantes pour les utilisateurs et pour l'organisation.

La supervision permet de suivre l'état de l'infrastructure, de collecter des indicateurs techniques et de détecter les anomalies avant qu'elles ne deviennent des incidents majeurs.

Ce projet consiste à mettre en place une plateforme de supervision avec Zabbix 7.0 sur Ubuntu Server 24.04 LTS. Le déploiement est effectué directement sur le système d'exploitation, sans utilisation de Docker.

Le laboratoire permet de superviser un serveur Linux et un serveur Windows, de collecter leurs données, d'observer leurs performances, de découvrir les équipements présents sur le réseau et de suivre les événements générés par Zabbix.

L'objectif n'est pas uniquement d'installer un logiciel de supervision. Il s'agit également de comprendre son fonctionnement, d'administrer les hôtes, d'interpréter les données collectées et de suivre les incidents jusqu'à leur résolution.

  1.2 Objectifs techniques

Le projet poursuit plusieurs objectifs :

Installer et configurer un serveur Zabbix sur Ubuntu Server.

Déployer une base de données MariaDB et un serveur Web Nginx.

Installer et configurer Zabbix Agent 2 sur Linux et Windows.

Intégrer les machines dans la console de supervision.

Mettre en place une règle de découverte réseau.

Collecter les données relatives au processeur, à la mémoire, au stockage et au système d'exploitation.

Consulter les graphiques et les tableaux de bord.

Analyser les problèmes détectés par Zabbix.

Vérifier les événements à partir des systèmes concernés.

Comprendre les mécanismes d'administration, de maintenance et de sauvegarde de la plateforme.

1.3 Périmètre du laboratoire

Le laboratoire s'appuie sur les équipements virtuels suivants :

Serveur de supervision

Le serveur Zabbix est installé sur Ubuntu Server 24.04.5 LTS. Il héberge le service Zabbix Server, la base MariaDB, l'interface Web et l'agent local.

Son nom système est zbx-srv01.

Serveur Windows

Une machine Windows Server 2025 est intégrée à la plateforme. Elle utilise Zabbix Agent 2 pour transmettre les informations relatives au système et à ses ressources.

Son nom système est SRV-DC01.

Infrastructure réseau

Une règle de découverte permet d'identifier les équipements présents sur la plage réseau du laboratoire. Les contrôles utilisent notamment ICMP et le service Agent Zabbix.

Les adresses réseau ne sont pas publiées dans cette documentation.

1.4 Résultat attendu

À la fin du projet, l'administrateur doit être capable d'utiliser Zabbix pour surveiller plusieurs systèmes, consulter leurs données, identifier des événements et effectuer un premier diagnostic technique.

Le laboratoire constitue également une base de travail pour approfondir la supervision SNMP, les notifications, les modèles personnalisés, les SLA et la supervision distribuée.

2.Architecture logicielle et environnement

2.1 Composants installés

Le serveur de supervision repose sur plusieurs composants qui travaillent ensemble.

Zabbix Server assure la collecte, le traitement des données, l'évaluation des déclencheurs et la gestion des événements.

MariaDB stocke la configuration de Zabbix, les informations relatives aux hôtes, les éléments de collecte, les événements et l'historique des métriques.

Nginx assure la publication de l'interface Web.

PHP-FPM permet l'exécution du code PHP utilisé par l'interface Zabbix.

Zabbix Agent 2 collecte les informations du système sur lequel il est installé et communique avec le serveur de supervision.

L'ensemble est installé directement sur Ubuntu. Aucun conteneur Docker n'est utilisé.

2.2 Informations de l'environnement

Le serveur principal utilise les caractéristiques suivantes :

Système : Ubuntu Server 24.04.5 LTS.

Architecture : x86_64.

Noyau Linux : 6.8.0-147-generic.

Nom de machine : zbx-srv01.

Virtualisation : VMware.

Zabbix Server : 7.0.32.

MariaDB : 10.11.14.

Nginx : 1.24.0.

PHP : 8.3.6.

Fuseau horaire : America/Toronto.

Les versions correspondent aux informations relevées dans le laboratoire au moment de la documentation. Elles peuvent différer lors d'une nouvelle installation ou d'une mise à jour.

2.3 Vérification du système

Avant d'installer une plateforme de supervision, il est nécessaire de vérifier les caractéristiques du système et les ressources disponibles.

Les commandes suivantes permettent de récupérer les informations essentielles :

hostnamectl
lsb_release -a
uname -r
ip -br address
free -h
df -h
uptime
Elles permettent notamment de vérifier le nom du serveur, la version d'Ubuntu, le noyau, les interfaces réseau, la mémoire disponible, l'espace disque et la durée de fonctionnement.
<img width="1470" height="521" alt="01-01-versions-ubuntu-zabbix-mariadb-nginx-php png" src="https://github.com/user-attachments/assets/06f55ed4-2fa9-4d3f-8cc6-76982f8c568e" />








