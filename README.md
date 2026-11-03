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
