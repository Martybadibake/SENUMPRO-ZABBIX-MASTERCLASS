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

<img width="1383" height="627" alt="01-02-informations-systeme-et-ressources-ubuntu png" src="https://github.com/user-attachments/assets/5312b5fe-9fc9-4b15-8cef-1734b818e95f" />

2.4 Vérification des versions logicielles

Les versions des principaux composants peuvent être contrôlées avec les commandes suivantes :

zabbix_server --version

mariadb --version

nginx -v

php -v

Cette vérification permet de confirmer que les composants attendus sont installés et de faciliter le diagnostic lorsqu'une incompatibilité de version est suspectée.

<img width="1470" height="521" alt="01-01-versions-ubuntu-zabbix-mariadb-nginx-php png" src="https://github.com/user-attachments/assets/3a64b6d3-8cb5-412f-89b9-09183ca6bffe" />

3.Installation et configuration du serveur Zabbix

3.1 Préparation d'Ubuntu Server

La première étape consiste à préparer le système d'exploitation.

Il faut commencer par mettre à jour les dépôts et les paquets :

sudo apt update

sudo apt upgrade -y

Le nom de la machine doit être défini avant de poursuivre :

sudo hostnamectl set-hostname zbx-srv01

Le fuseau horaire doit également être configuré pour assurer la cohérence des horodatages des événements et des graphiques.

sudo timedatectl set-timezone America/Toronto
timedatectl status

Une synchronisation correcte de l'heure est importante dans une infrastructure de supervision. Un décalage horaire peut compliquer l'analyse des événements et la corrélation des journaux.


3.2 Installation des dépendances

Les outils nécessaires à l'administration du système sont installés avant le déploiement :

sudo apt install -y curl wget ca-certificates chrony

La synchronisation de l'heure peut être vérifiée avec :

timedatectl status

L'administrateur doit s'assurer que l'horloge du système est synchronisée.

3.3 Installation du dépôt officiel Zabbix

Pour installer Zabbix, il est préférable d'utiliser les paquets du dépôt officiel correspondant à la version et à la distribution choisies.

La procédure générale consiste à télécharger le paquet de configuration du dépôt, à l'installer, puis à actualiser la liste des paquets.

Exemple pour Ubuntu 24.04 et Zabbix 7.0 :

wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest+ubuntu24.04_all.deb

sudo dpkg -i zabbix-release_latest+ubuntu24.04_all.deb

sudo apt update

N.B La version du dépôt doit être vérifiée avant toute installation afin d'éviter de mélanger plusieurs branches de Zabbix.

3.4 Installation des composants

Le laboratoire utilise MariaDB comme moteur de base de données, Nginx comme serveur Web et PHP-FPM pour l'interface.

Les composants Zabbix nécessaires sont installés à l'aide du gestionnaire de paquets :

sudo apt install -y mariadb-server

sudo apt install -y \
zabbix-server-mysql \
zabbix-frontend-php \
zabbix-nginx-conf \
zabbix-sql-scripts \
zabbix-agent2 \
php8.3-mysql

Cette installation fournit les services nécessaires au fonctionnement de la plateforme.

3.5 Préparation de la base MariaDB

Zabbix utilise une base de données pour conserver sa configuration et les informations collectées.

La base doit être créée avec un encodage compatible avec les exigences de Zabbix. Un compte dédié doit être utilisé pour les connexions du serveur de supervision.

Exemple de procédure pour une nouvelle installation :

CREATE DATABASE zabbix

CHARACTER SET utf8mb4

COLLATE utf8mb4_bin;

CREATE USER 'zabbix'@'localhost'

IDENTIFIED BY 'REMPLACER_PAR_UN_MOT_DE_PASSE_FORT';

GRANT ALL PRIVILEGES ON zabbix.*

TO 'zabbix'@'localhost';

FLUSH PRIVILEGES;

Le schéma initial de Zabbix est ensuite importé à partir du fichier SQL fourni par le paquet officiel :

zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz |

sudo mysql --default-character-set=utf8mb4 -uzabbix -p zabbix

Cette procédure concerne une base neuve. Elle ne doit pas être exécutée une seconde fois sur une base déjà initialisée.

Dans un environnement existant, il faut d'abord vérifier l'état de la base et consulter les journaux avant toute opération de modification.


3.6 Configuration du serveur Zabbix

Le fichier principal de configuration du serveur est :

sudo nano /etc/zabbix/zabbix_server.conf

Les paramètres de connexion à la base doivent correspondre aux informations réellement créées dans MariaDB.

DBHost=localhost

DBName=zabbix

DBUser=zabbix

DBPassword=REMPLACER_PAR_LE_MOT_DE_PASSE_CONFIGURE

Le fichier contient des informations sensibles et doit conserver des permissions adaptées.

3.7 Configuration de Nginx et PHP-FPM

L'interface Web de Zabbix est publiée par Nginx.

Le fichier de configuration utilisé est notamment :

sudo nano /etc/zabbix/nginx.conf

Le nom DNS ou le nom de serveur doit être adapté à l'environnement. Le fuseau horaire doit également être défini dans la configuration PHP-FPM.

sudo nano /etc/zabbix/php-fpm.conf

Paramètre :

php_value[date.timezone] = America/Toronto

Avant de recharger le service, il faut contrôler la syntaxe de la configuration Nginx :

sudo nginx -t

Une configuration incorrecte doit être corrigée avant le redémarrage du service.

3.8 Démarrage des services

Une fois les composants installés et configurés, les services doivent être démarrés et activés au démarrage du système.

sudo systemctl restart mariadb
sudo systemctl restart zabbix-server
sudo systemctl restart zabbix-agent2
sudo systemctl restart nginx
sudo systemctl restart php8.3-fpm

Activation au démarrage :

sudo systemctl enable mariadb
sudo systemctl enable zabbix-server
sudo systemctl enable zabbix-agent2
sudo systemctl enable nginx
sudo systemctl enable php8.3-fpm

Le contrôle de l'état des services permet de confirmer leur fonctionnement :

sudo systemctl status zabbix-server
sudo systemctl status mariadb
sudo systemctl status nginx
sudo systemctl status php8.3-fpm

Dans le laboratoire, les services Zabbix Server, MariaDB et Nginx ont été vérifiés comme actifs.

<img width="1452" height="741" alt="01-03-etat-du-service-zabbix-server png" src="https://github.com/user-attachments/assets/38e60919-71a2-40a8-b58e-edce62d664dd" />

<img width="1460" height="718" alt="01-04-etat-des-services-nginx-et-mariadb png" src="https://github.com/user-attachments/assets/3d17c0f3-9baf-4d72-a6f5-4a84ac0bedd6" />

3.9 Accès à l'interface Web

Après le démarrage des services, l'interface Web est accessible depuis un navigateur à l'adresse du serveur.

L'assistant d'installation permet de vérifier les prérequis PHP, les paramètres de connexion à la base de données et les informations générales du serveur.

Une fois la configuration terminée, l'administrateur accède à la console Zabbix.

Le compte administrateur initial doit être sécurisé immédiatement. Il est recommandé de créer un compte nominatif et de limiter l'utilisation du compte d'administration général.

3.10 Sécurisation de l'accès Web

Pour protéger les communications avec l'interface Web, HTTPS doit être configuré.

Dans un laboratoire privé, un certificat auto-signé peut être utilisé. Il permet de chiffrer les communications, mais les navigateurs affichent généralement un avertissement tant que le certificat n'est pas approuvé.

Dans un environnement de production, il est préférable d'utiliser un certificat émis par une autorité de certification reconnue.

La configuration doit également protéger la clé privée et limiter l'accès réseau à l'interface d'administration.


4.Installation et intégration des agents

4.1 Rôle de Zabbix Agent 2

Zabbix Agent 2 est installé sur les machines que l'on souhaite superviser.

Il collecte les informations relatives au système d'exploitation, au processeur, à la mémoire, aux disques, aux interfaces réseau et à différents services.

L'agent communique avec le serveur Zabbix selon deux modes.

En mode passif, le serveur interroge directement l'agent. Le port TCP 10050 doit être accessible depuis le serveur de supervision.

En mode actif, l'agent contacte le serveur et lui transmet les données. La communication utilise le port TCP 10051 côté serveur.

Le choix du mode dépend de la topologie réseau, des règles de pare-feu et des contraintes de sécurité.

4.2 Installation de l'agent sur Windows Server 2025

L'agent Windows est distribué sous forme de paquet MSI.

La procédure consiste à télécharger la version correspondant à l'architecture du serveur, à exécuter l'installation et à renseigner les paramètres de communication.

Les principaux paramètres sont :

Server=ADRESSE_SERVEUR_ZABBIX

ServerActive=ADRESSE_SERVEUR_ZABBIX

Hostname=SRV-DC01

Le nom Hostname doit correspondre exactement au nom technique de l'hôte déclaré dans l'interface Zabbix.

Le service Windows doit ensuite être vérifié.

Dans PowerShell administrateur :

Get-Service "Zabbix Agent 2"

Si le service est arrêté :

Start-Service "Zabbix Agent 2"

Le pare-feu Windows doit autoriser les connexions entrantes sur le port TCP 10050 lorsque le mode passif est utilisé. Il est recommandé de limiter cette règle à l'adresse du serveur Zabbix.

Exemple :

New-NetFirewallRule `
-DisplayName "Zabbix Agent 10050" `
-Direction Inbound `
-Protocol TCP `
-LocalPort 10050 `
-Action Allow `
-RemoteAddress ADRESSE_SERVEUR_ZABBIX

Dans ce laboratoire, Zabbix Agent 2 version 7.0.32 a été installé sur Windows Server 2025. Le service a été démarré et la communication avec le serveur a été vérifiée.

<img width="1028" height="194" alt="02-01-installation-agent2-windows-server png" src="https://github.com/user-attachments/assets/31c180c9-bb40-48fc-9591-0a90e4ef6a63" />


