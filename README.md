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

sudo apt install -y
zabbix-server-mysql
zabbix-frontend-php
zabbix-nginx-conf
zabbix-sql-scripts
zabbix-agent2
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

New-NetFirewallRule

DisplayName "Zabbix Agent 10050"

Direction Inbound

Protocol TCP

LocalPort 10050

Action Allow

RemoteAddress ADRESSE_SERVEUR_ZABBIX

Dans ce laboratoire, Zabbix Agent 2 version 7.0.32 a été installé sur Windows Server 2025. Le service a été démarré et la communication avec le serveur a été vérifiée.

<img width="1028" height="194" alt="02-01-installation-agent2-windows-server png" src="https://github.com/user-attachments/assets/31c180c9-bb40-48fc-9591-0a90e4ef6a63" />

4.3 Déclaration de l'hôte Windows dans Zabbix

Après l'installation de l'agent, l'hôte doit être déclaré dans l'interface Web.

Dans le menu Collecte de données, puis Hôtes, il faut créer un nouvel hôte et renseigner son nom technique.

Les informations utilisées dans le laboratoire sont :

Nom technique : SRV-DC01.

Nom visible : Windows Server 2025 - DC01.

Groupe : Virtual machines.

Modèle : Windows by Zabbix agent.

Interface : adresse du serveur Windows et port 10050.

Une fois l'hôte créé, Zabbix commence à collecter les informations prévues par le modèle associé.

La disponibilité de l'agent est vérifiable dans la liste des hôtes. Une icône ZBX verte indique que la communication avec l'agent est opérationnelle.

<img width="1110" height="621" alt="02-02-configuration-hote-windows-server-2025 png" src="https://github.com/user-attachments/assets/184a1662-f18f-4451-aad7-5505addbdb69" />

<img width="1737" height="721" alt="02-03-hotes-supervises-dans-zabbix png" src="https://github.com/user-attachments/assets/57c6bf8f-160a-443c-a73b-821716686a23" />

4.4 Installation de l'agent sur Ubuntu

Le serveur Zabbix utilise également Zabbix Agent 2 pour surveiller son propre fonctionnement.

Le fichier de configuration principal est :

sudo nano /etc/zabbix/zabbix_agent2.conf

Les paramètres essentiels sont :

Server=127.0.0.1,ADRESSE_SERVEUR_ZABBIX

ServerActive=127.0.0.1

Hostname=zbx-srv01

Après modification :

sudo systemctl restart zabbix-agent2

sudo systemctl status zabbix-agent2

La communication peut être testée avec l'utilitaire zabbix_get :

zabbix_get -s ADRESSE_AGENT -p 10050 -k agent.ping

Une réponse égale à 1 confirme que l'agent répond correctement à cette vérification.

Dans le laboratoire, le serveur Ubuntu est supervisé avec le modèle Linux by Zabbix agent.

5. Administration des hôtes et organisation de la supervision

5.1 Groupes d'hôtes

Les groupes d'hôtes permettent d'organiser les systèmes selon leur fonction ou leur environnement.

Dans une infrastructure d'entreprise, il est préférable de séparer les serveurs Linux, les serveurs Windows, les équipements réseau et les systèmes de supervision.

Cette organisation facilite la gestion des permissions, des modèles, des actions et des rapports.

Dans le laboratoire, les groupes utilisés comprennent notamment :

Linux servers.

Virtual machines.

Zabbix servers.

La création de groupes supplémentaires doit suivre l'évolution réelle de l'infrastructure.

5.2 Modèles de supervision

Un modèle regroupe les éléments de collecte, les déclencheurs et les graphiques associés à un type de système.

Il permet d'appliquer une configuration cohérente à plusieurs hôtes sans devoir recréer chaque élément individuellement.

Le laboratoire utilise les modèles standards suivants :

Linux by Zabbix agent

Windows by Zabbix agent

Ces modèles fournissent les éléments nécessaires à la surveillance des ressources principales des systèmes.

Pour aller plus loin, l'administrateur peut créer un modèle personnalisé regroupant les paramètres propres à son organisation. Il est recommandé de cloner un modèle officiel avant de le modifier.


5.3 Utilisateurs, rôles et permissions

Dans une installation destinée à plusieurs administrateurs, les droits d'accès doivent être organisés selon les responsabilités.

Le rôle détermine les opérations autorisées dans l'interface. Les permissions sur les groupes d'hôtes déterminent les données que l'utilisateur peut consulter ou modifier.

Une organisation simple peut prévoir un administrateur de la plateforme, un technicien de support en lecture seule et un responsable disposant d'une vue sur les indicateurs de disponibilité.

La procédure consiste à créer les groupes d'utilisateurs, définir les rôles, attribuer les permissions sur les groupes d'hôtes et vérifier les accès en ouvrant une session avec chaque profil.

Cette configuration est une procédure recommandée pour une extension multiutilisateur du laboratoire ; elle n'est pas présentée ici comme une réalisation validée par capture.

5.4 Sécurisation des communications

En production, les communications entre les agents et le serveur doivent être protégées.

Zabbix permet notamment d'utiliser une clé prépartagée, appelée PSK, ou des certificats.

Pour une configuration PSK, il faut générer une clé aléatoire, la déployer sur l'agent et renseigner la même identité et la même clé dans la configuration de l'hôte Zabbix.

Les paramètres principaux sont :

TLSConnect=psk

TLSAccept=psk

TLSPSKIdentity=IDENTITE_PSK

TLSPSKFile=CHEMIN_VERS_LA_CLE

La clé doit être protégée par des permissions strictes. Elle ne doit jamais être ajoutée au dépôt GitHub.

Le chiffrement PSK est une extension recommandée pour ce laboratoire. Les captures actuelles ne démontrent pas son activation.

6.Découverte automatique du réseau

6.1 Principe de la découverte réseau

La découverte réseau permet à Zabbix d'identifier les équipements présents sur une plage d'adresses IP.

Le serveur effectue différents contrôles selon les paramètres définis dans la règle : disponibilité ICMP, présence d'un agent Zabbix ou réponse d'un service particulier.

Cette fonction facilite la reconnaissance d'un réseau et permet de repérer les équipements qui pourraient nécessiter une supervision.

Il faut cependant distinguer la découverte d'un équipement de son intégration complète. Un équipement détecté n'est pas automatiquement un hôte supervisé avec un ensemble d'éléments et de déclencheurs.

6.2 Création de la règle de découverte

Dans le menu Collecte de données, puis Découverte, une règle peut être créée avec un nom explicite.

Dans le laboratoire, la règle porte le nom :

DISCOVERY-LAN

La plage d'adresses correspond au réseau local du laboratoire. Elle est volontairement masquée dans la documentation publique.

L'intervalle de vérification est défini à cinq minutes.

Les contrôles configurés comprennent :

Ping ICMP.

Vérification de la présence d'un agent Zabbix.

La règle doit être activée pour que les contrôles soient exécutés périodiquement.

<img width="1919" height="631" alt="03-01-creation-regle-decouverte-reseau png" src="https://github.com/user-attachments/assets/ac3dd349-1423-4123-a2ef-361155ae994b" />

6.3 Analyse des résultats

Après l'activation de la règle, Zabbix présente les équipements détectés et les résultats des contrôles effectués.

Dans le laboratoire, neuf équipements ont été identifiés par la règle de découverte.

Parmi les résultats figurent le serveur Zabbix Ubuntu, le serveur Windows et plusieurs autres équipements du réseau.

Les résultats permettent de distinguer les machines dont l'agent répond de celles qui sont uniquement identifiées par ICMP.

Cette distinction est importante : une réponse au ping confirme une certaine disponibilité réseau, mais ne garantit pas qu'un agent de supervision est installé ou que les métriques système sont accessibles.

<img width="1912" height="875" alt="03-02-resultats-decouverte-equipements-reseau png" src="https://github.com/user-attachments/assets/622e6e40-239a-44bd-9594-1943bc1eaf61" />

6.4 Exploitation de la découverte

Dans un environnement plus avancé, une règle de découverte peut être associée à des actions.

Ces actions peuvent notamment permettre de créer automatiquement un hôte, de l'affecter à un groupe et de lui associer un modèle.

Avant d'activer une automatisation, il faut définir des critères précis afin d'éviter l'intégration d'équipements inconnus ou non autorisés.

La découverte réseau du laboratoire a été utilisée comme démonstration de reconnaissance des équipements. L'auto-enregistrement automatique des nouveaux hôtes constitue une évolution possible, mais n'est pas déclaré comme déployé dans ce projet.

7. Collecte et analyse des données

7.1 Fonctionnement des éléments de collecte

Dans Zabbix, un élément de collecte, appelé item, correspond à une donnée mesurée sur un hôte.

Il possède une clé, un type de données, une méthode de collecte, un intervalle de mise à jour et une durée de conservation.

Les éléments peuvent provenir d'un agent, d'une vérification simple, d'une interface SNMP, d'une requête HTTP ou d'autres méthodes prises en charge.

Les données sont ensuite stockées dans la base afin d'être utilisées par les graphiques, les déclencheurs et les rapports.

7.2 Consultation des dernières données

La rubrique Dernières données permet de consulter les valeurs collectées pour un hôte.

L'administrateur peut filtrer les résultats selon l'hôte, le groupe, le nom de l'élément ou les étiquettes.

Cette fonctionnalité est utile lorsqu'un problème est signalé : elle permet de vérifier si les données remontent correctement et d'observer les dernières valeurs disponibles.

Dans le laboratoire, les dernières données du serveur Ubuntu ont été consultées pour vérifier les éléments associés au système d'exploitation et à l'inventaire des paquets.

<img width="1919" height="966" alt="04-01-filtrage-des-dernieres-donnees png" src="https://github.com/user-attachments/assets/3d0ef2e4-ee3f-438e-bb26-0ad558f6db5c" />

7.3 Configuration d'un élément de collecte

Un élément peut être configuré directement sur un hôte ou hérité d'un modèle.

Sa configuration doit préciser le nom, la clé, le type de collecte, le type d'information et l'intervalle de mise à jour.

Dans le laboratoire, l'élément Number of installed packages utilise la clé :

system.sw.packages.get

Il permet de récupérer le nombre de paquets logiciels installés sur le serveur Linux.

La capture montre également les paramètres de conservation de l'historique et des tendances.

Une attention particulière doit être accordée aux intervalles personnalisés. Une collecte très fréquente n'est pas nécessairement utile pour une donnée qui évolue rarement. Elle peut augmenter inutilement la charge du serveur et le volume de la base de données.

<img width="1199" height="840" alt="04-02-configuration-element-collecte-paquets png" src="https://github.com/user-attachments/assets/e7e622f7-7a9a-4c24-83bf-e4e5d7723bbd" />

7.4 Validation d'un élément

Après la création ou la modification d'un élément, il est nécessaire de vérifier que la requête fonctionne.

Zabbix propose une fonction de test qui permet de vérifier la réponse de l'élément selon son type et sa configuration.

Dans le laboratoire, une requête de test a été exécutée avec succès pour l'élément relatif au nombre de paquets installés.

La validation de la requête confirme que le test a été envoyé avec succès. Il faut également vérifier que la valeur apparaît dans les dernières données et que l'élément reste pris en charge.

<img width="1155" height="845" alt="04-03-validation-requete-element-zabbix png" src="https://github.com/user-attachments/assets/5434e1ca-2fd6-4d71-97f0-f7827d938977" />

7.5 Surveillance de l'espace disque

L'espace disque constitue un indicateur essentiel pour prévenir les interruptions de service.

Une partition saturée peut empêcher l'écriture de journaux, perturber les applications et provoquer des dysfonctionnements du système.

Le modèle Linux fournit des éléments relatifs à la capacité totale, à l'espace utilisé et à l'espace disponible des systèmes de fichiers.

Les données peuvent être consultées sous forme de valeurs et de graphiques.

Dans le laboratoire, les graphiques de l'espace disque et de la partition de démarrage ont été consultés.

L'analyse doit porter sur l'évolution de l'espace disponible, et non uniquement sur une valeur ponctuelle.

<img width="1910" height="957" alt="04-04-surveillance-espace-disque-linux png" src="https://github.com/user-attachments/assets/13a772d0-8a8f-4194-9cc9-e8e6c7b1c049" />

7.6 Surveillance du processeur et de la mémoire

La surveillance des ressources permet de détecter les situations de surcharge.

Pour le processeur, les principaux indicateurs comprennent l'utilisation CPU et la charge système.

Pour la mémoire, il faut observer la mémoire totale, la mémoire utilisée, la mémoire disponible et l'utilisation du swap.

Dans le laboratoire, les tableaux de bord du serveur Ubuntu présentent notamment les graphiques de charge système, d'utilisation CPU, de mémoire et de swap.

Ces informations permettent d'établir un état de référence du serveur et de comparer son comportement dans le temps.

Une utilisation CPU élevée et persistante peut indiquer une surcharge applicative. Une mémoire disponible faible peut nécessiter une analyse des processus et de la consommation mémoire.

<img width="1919" height="990" alt="04-05-surveillance-cpu-memoire-et-charge-linux png" src="https://github.com/user-attachments/assets/28dde53d-754e-428f-bb79-797525d2cbdd" />

8.Déclencheurs et gestion des incidents

8.1 Principe des déclencheurs

Un déclencheur, ou trigger, est une expression logique évaluée à partir des données collectées.

Lorsqu'une condition est remplie, Zabbix génère un événement et peut ouvrir un problème.

La sévérité permet de distinguer les événements selon leur importance : information, avertissement, moyen, haut ou désastre.

Les déclencheurs doivent être configurés avec des seuils cohérents et des périodes d'observation adaptées.

Un seuil trop sensible peut générer des alertes inutiles. À l'inverse, un seuil trop permissif peut retarder la détection d'un incident.

8.2 Exemples de déclencheurs

Pour surveiller le processeur, un seuil peut être défini à partir d'une moyenne sur plusieurs minutes.

Exemple conceptuel :

avg(/NOM_HOTE/system.cpu.util,5m)>90

Pour surveiller l'espace libre d'un système de fichiers :

last(/NOM_HOTE/vfs.fs.size[/,pfree])<10

Pour détecter l'absence de données :

nodata(/NOM_HOTE/agent.ping,5m)=1

Ces expressions sont des exemples à adapter aux noms d'hôtes, aux éléments disponibles et à la version utilisée. Elles ne sont pas présentées comme des déclencheurs personnalisés effectivement déployés dans le laboratoire.

8.3 Détection des événements Windows

La plateforme a permis d'identifier des événements concernant deux services Windows :

InventorySvc

AppXSvc

Les événements signalaient que les services concernés n'étaient pas en cours d'exécution alors que leur configuration de démarrage était automatique ou automatique différée.

Zabbix a associé ces événements à une sévérité moyenne.

Cette situation constitue un exemple de surveillance des services Windows à partir du modèle de supervision associé à l'hôte.

<img width="1915" height="755" alt="05-01-detection-problemes-windows-et-linux png" src="https://github.com/user-attachments/assets/6f19dce4-b9e6-44fb-b4a0-7cfac7f7d516" />

8.4 Analyse des événements

Lorsqu'un problème apparaît dans Zabbix, l'administrateur doit examiner le nom de l'hôte, le déclencheur, la sévérité, l'heure de détection et la durée de l'événement.

Il faut ensuite déterminer si le problème est lié à une panne réelle, à une configuration particulière du système ou à une condition temporaire.

Dans ce laboratoire, les événements Windows ont été examinés depuis la rubrique Surveillance, puis Problèmes.

L'analyse a permis d'identifier les services concernés et de poursuivre le diagnostic directement sur le serveur Windows.

<img width="1656" height="614" alt="05-02-analyse-des-evenements-zabbix png" src="https://github.com/user-attachments/assets/1b8f0eae-19b1-4a6e-b717-9ea1c937fbee" />

8.5 Vérification sur le serveur Windows

Une alerte de supervision doit être confrontée à l'état réel du système.

Dans le cas de AppXSvc, la console Services Windows permet de consulter le nom du service, sa description, son état et son type de démarrage.

Cette vérification permet de confirmer si le service est actuellement démarré et de mieux comprendre l'événement détecté.

Il ne faut pas arrêter ou modifier un service Windows uniquement parce qu'il apparaît dans Zabbix. Certains services fonctionnent à la demande ou selon des mécanismes de déclenchement spécifiques.

Toute intervention doit être précédée d'une analyse de son rôle et de son impact.

<img width="1616" height="853" alt="05-03-verification-service-windows-appxsvc png" src="https://github.com/user-attachments/assets/3e10cda2-c2c8-4435-94d5-fe65b68b85c6" />

8.6 Suivi de la résolution

Après la vérification, l'administrateur retourne dans Zabbix pour suivre l'évolution du problème.

Un événement peut passer de l'état PROBLÈME à l'état RÉSOLU lorsque la condition du déclencheur n'est plus remplie ou lorsque la récupération est enregistrée.

Dans le laboratoire, les événements relatifs aux services Windows sont ensuite apparus comme résolus.

Les captures documentent le suivi de cette évolution. Elles ne permettent pas, à elles seules, d'attribuer la résolution à une intervention précise.
<img width="1916" height="894" alt="05-04-suivi-resolution-des-incidents png" src="https://github.com/user-attachments/assets/1e9c6cab-7740-4cc8-9c5d-23b3143f60c6" />
<img width="1892" height="957" alt="05-05-confirmation-resolution-evenements png" src="https://github.com/user-attachments/assets/988b4da5-19da-49b4-8ec2-727a72f1b1b2" />

8.7 Analyse d'un changement sur le système Linux

Un autre événement a été observé sur le serveur Ubuntu :

Linux: Number of installed packages has been changed

Cet événement indique que le nombre de paquets installés a changé depuis la dernière collecte de référence.

Ce type de détection peut être utile pour suivre les modifications logicielles apportées à un serveur.

Il ne signifie pas nécessairement qu'une panne est survenue. Il peut être déclenché après une installation, une suppression ou une mise à jour de paquets.

Dans une exploitation réelle, il convient de corréler cet événement avec les journaux du gestionnaire de paquets et les opérations de maintenance.

Exemples de commandes de vérification :

grep -i " install " /var/log/dpkg.log
grep -i " upgrade " /var/log/dpkg.log
grep -i " remove " /var/log/dpkg.log

Cet événement a été observé dans le laboratoire et doit être présenté comme un changement détecté, non comme une panne résolue.

9.Inventaire des hôtes

9.1 Objectif de l'inventaire

L'inventaire permet de centraliser certaines informations relatives aux machines supervisées.

Selon les éléments configurés et les données disponibles, il peut contenir le système d'exploitation, le matériel, le numéro de série, le nom de l'hôte et des informations de localisation.

Il peut être alimenté manuellement ou automatiquement.

9.2 Consultation de l'inventaire

Dans Zabbix, l'inventaire peut être consulté depuis le menu Inventaire.

Il est possible de filtrer les résultats selon les hôtes, les groupes et les champs disponibles.

Dans le laboratoire, la consultation de l'inventaire permet notamment de retrouver les informations relatives au serveur Ubuntu et à Windows Server.

La capture démontre l'accès à la rubrique d'inventaire et l'affichage des informations disponibles. Elle ne démontre pas que tous les champs matériels sont automatiquement renseignés.
<img width="1907" height="451" alt="06-01-inventaire-et-filtrage-des-hotes png" src="https://github.com/user-attachments/assets/50f3198e-6fa4-4dde-903b-c0628b4552d6" />

9.3 Automatisation de l'inventaire

Pour aller plus loin, il est possible d'activer le mode d'inventaire automatique sur les hôtes et d'associer certains éléments de collecte aux champs correspondants.

L'administrateur doit vérifier que les données remontées sont fiables et adaptées au champ d'inventaire concerné.

Cette automatisation facilite le suivi du parc informatique et réduit la saisie manuelle.

10. Notifications et actions

10.1 Fonctionnement des notifications

Zabbix peut envoyer des notifications lorsqu'un déclencheur génère un problème.

Le mécanisme repose sur plusieurs composants : les événements, les actions, les opérations, les types de médias et les informations de contact des utilisateurs.

Une notification efficace doit fournir les informations nécessaires à la prise en charge de l'incident : nom de l'hôte, problème, sévérité, date et heure.

10.2 Configuration du courriel

Pour utiliser les notifications par courriel, il faut configurer un type de média SMTP dans l'administration de Zabbix.

La configuration nécessite notamment :

Le serveur SMTP.

Le port de connexion.

Le type de sécurité.

Le compte d'authentification.

Le mot de passe ou mécanisme d'authentification autorisé.

L'adresse d'expédition.

Il faut ensuite associer une adresse électronique à l'utilisateur concerné et tester le type de média.

Les identifiants SMTP doivent être protégés et ne doivent jamais être publiés dans un dépôt de code.

10.3 Création d'une action

Une action permet de définir les conditions qui provoquent l'envoi d'une notification.

Par exemple, une organisation peut décider de transmettre les problèmes de sévérité haute concernant les serveurs Windows à l'équipe d'administration.

Les opérations peuvent inclure l'envoi d'un message, une notification de récupération ou une escalade vers un autre groupe de support.

La procédure consiste à définir les conditions, les destinataires, le canal de notification et le contenu du message.

La configuration SMTP, les webhooks et les notifications automatiques ne sont pas démontrés par les captures actuelles. Ils constituent une étape d'approfondissement.

11. Modèles personnalisés et découverte de bas niveau

11.1 Pourquoi utiliser des modèles ?

Les modèles permettent de standardiser la supervision.

Plutôt que de créer manuellement les mêmes éléments sur chaque serveur, l'administrateur regroupe les éléments, les déclencheurs et les graphiques dans un modèle réutilisable.

Lorsqu'un hôte est associé au modèle, il hérite de sa configuration.

Cette approche facilite la maintenance et la cohérence de la supervision.

11.2 Création d'un modèle personnalisé

Une procédure possible consiste à créer un modèle nommé :

SENUMPRO - Serveur de base

Le modèle peut regrouper les éléments relatifs au processeur, à la mémoire, au stockage, aux interfaces réseau et à la disponibilité de l'agent.

Des macros peuvent être utilisées pour rendre les seuils configurables.

Par exemple :

{$CPU.UTIL.MAX}=90
{$DISK.FREE.MIN}=10

Les valeurs doivent être adaptées à la politique de supervision de l'organisation.

Après création, le modèle doit être testé sur un hôte avant d'être déployé à plus grande échelle.

11.3 Découverte de bas niveau (LLD)

La découverte de bas niveau permet à Zabbix de détecter automatiquement des ressources internes à un système.

Elle est notamment utilisée pour identifier les systèmes de fichiers et les interfaces réseau.

Une règle LLD peut générer des prototypes d'éléments, de déclencheurs et de graphiques.

Par exemple, la découverte des systèmes de fichiers permet de créer automatiquement des éléments pour les volumes détectés.

Cette fonctionnalité évite de devoir configurer manuellement chaque disque sur chaque hôte.

La création d'un modèle personnalisé et de règles LLD est une procédure d'extension. Elle n'est pas présentée comme une réalisation validée dans le laboratoire actuel.

12. Tableaux de bord et visualisation

12.1 Utilité des tableaux de bord

Les tableaux de bord permettent de regrouper les informations importantes sur un même écran.

Ils peuvent être adaptés aux besoins de l'équipe technique, du support informatique ou de la direction.

Un administrateur peut y afficher les problèmes actifs, les performances des serveurs, les graphiques de ressources et l'état général des hôtes.

12.2 Tableaux de bord du serveur Ubuntu

Dans le laboratoire, les tableaux de bord de l'hôte Zabbix Server ont été consultés.

La vue Filesystems présente l'utilisation des systèmes de fichiers et l'espace disponible.

La vue System performance présente notamment la charge système, l'utilisation du processeur, la mémoire et le swap.

Ces vues permettent d'observer les ressources du serveur sur une période définie et d'analyser leur évolution.

<img width="1910" height="957" alt="04-04-surveillance-espace-disque-linux png" src="https://github.com/user-attachments/assets/1ff62a48-af1e-4f5d-ba11-da8ba63404cc" />

<img width="1919" height="990" alt="04-05-surveillance-cpu-memoire-et-charge-linux png" src="https://github.com/user-attachments/assets/2b7dc275-c7a5-4330-a6a2-df23cf9c0c1d" />

12.3 Organisation recommandée

Dans une infrastructure plus importante, il est utile de créer plusieurs tableaux de bord.

Une vue destinée au support peut présenter les problèmes actifs, les hôtes indisponibles et les principaux indicateurs de performance.

Une vue destinée à la direction peut présenter les indicateurs de disponibilité, les tendances et les incidents majeurs.

Le principe est de privilégier une lecture rapide et de ne pas surcharger les tableaux de bord avec des informations secondaires.

13. Supervision des équipements réseau avec SNMP

13.1 Présentation de SNMP

SNMP est un protocole utilisé pour superviser les équipements réseau et certains équipements spécialisés.

Il permet de récupérer des informations sur les interfaces, le trafic, l'état du matériel, la mémoire et d'autres indicateurs selon les capacités de l'équipement.

Contrairement à un agent installé sur un système d'exploitation, SNMP permet généralement de superviser un équipement réseau sans installer de logiciel supplémentaire sur celui-ci.

13.2 Versions SNMP

SNMPv1 et SNMPv2c utilisent une communauté pour contrôler l'accès. Cette méthode ne fournit pas un niveau de protection suffisant pour un réseau de production exposé.

SNMPv3 apporte des mécanismes d'authentification et de chiffrement. Il est à privilégier lorsque l'équipement le permet.

Les informations d'authentification ne doivent jamais être publiées dans les captures ou les fichiers de configuration partagés.

13.3 Procédure de configuration

La première étape consiste à activer SNMP sur l'équipement concerné et à définir les paramètres d'accès.

Il faut ensuite vérifier que le serveur Zabbix peut joindre l'équipement sur UDP 161.

Des outils comme snmpwalk et snmpget permettent de tester la communication et de consulter les OID disponibles.

Exemple de vérification SNMPv3 :

snmpwalk -v3 -l authPriv
-u UTILISATEUR_SNMP
-a SHA -A MOT_DE_PASSE_AUTH
-x AES -X MOT_DE_PASSE_PRIV
ADRESSE_EQUIPEMENT sysDescr

Une fois le test réussi, l'administrateur crée l'hôte dans Zabbix, ajoute une interface SNMP et associe un modèle adapté.

La découverte des interfaces peut ensuite permettre de collecter les compteurs de trafic.

La supervision SNMP n'est pas présentée comme déployée dans le laboratoire documenté. Cette section décrit la procédure à suivre pour une prochaine extension.

14. Maintenance, sauvegarde et exploitation

14.1 Gestion des périodes de maintenance

Les périodes de maintenance permettent de planifier les interventions sans générer inutilement des alertes.

Une maintenance peut être ponctuelle ou récurrente et être associée à un ou plusieurs hôtes.

Il faut définir précisément la période, les systèmes concernés et le comportement souhaité pour la collecte des données.

Avant une intervention, l'administrateur doit vérifier que les hôtes concernés sont correctement associés à la maintenance.

Cette procédure est particulièrement utile lors de l'installation de correctifs, des redémarrages planifiés et des opérations de maintenance matérielle.

14.2 Sauvegarde de Zabbix

La sauvegarde d'une plateforme Zabbix doit prendre en compte la base de données et les fichiers de configuration.

Une sauvegarde de la base seule ne suffit pas toujours à restaurer l'ensemble des personnalisations. Il faut également conserver les configurations du serveur, les fichiers de l'agent, les scripts externes et les modèles personnalisés.

Exemple de sauvegarde de la base MariaDB :

sudo mysqldump
--single-transaction
--routines
--triggers
zabbix | gzip > zabbix-database.sql.gz

Exemple de sauvegarde des fichiers de configuration :

sudo tar czf zabbix-config.tar.gz /etc/zabbix

Les archives doivent être stockées dans un emplacement protégé, séparé du système sauvegardé lorsque cela est possible.

14.3 Vérification d'une sauvegarde

Une sauvegarde n'est réellement utile que si sa restauration est possible.

Il est recommandé de réaliser périodiquement un test de restauration dans un environnement isolé.

La procédure doit vérifier l'intégrité de l'archive, la restauration de la base, la récupération des fichiers de configuration et le démarrage des services.

La restauration ne doit jamais être testée directement sur le serveur de production sans plan de retour arrière.

La sauvegarde planifiée et le test de restauration ne sont pas documentés comme réalisés dans ce laboratoire.

14.4 Mise à jour de la plateforme

Avant toute mise à jour, il faut consulter les notes de version, vérifier les dépendances et sauvegarder la base de données.

La mise à jour doit maintenir la cohérence entre le serveur, l'interface Web, les scripts SQL et les agents.

Exemple de mise à jour des paquets installés :

sudo apt update

sudo apt install --only-upgrade
zabbix-server-mysql
zabbix-frontend-php
zabbix-nginx-conf
zabbix-sql-scripts
zabbix-agent2

Après la mise à jour, il faut vérifier l'état des services et consulter les journaux.

15. Dépannage et méthodes de diagnostic

15.1 Méthode générale

Le dépannage d'une plateforme de supervision doit suivre une démarche structurée.

Il faut commencer par identifier le symptôme, déterminer le composant concerné et vérifier les journaux avant de modifier la configuration.

Une intervention doit être suivie d'un nouveau test afin de confirmer que le problème est réellement résolu.

Cette méthode évite les changements inutiles et permet de conserver une trace des opérations effectuées.

15.2 Vérification des services

Pour consulter l'état des principaux services :

sudo systemctl status zabbix-server
sudo systemctl status zabbix-agent2
sudo systemctl status mariadb
sudo systemctl status nginx
sudo systemctl status php8.3-fpm

Pour suivre les journaux :

sudo tail -f /var/log/zabbix/zabbix_server.log

Pour consulter les événements systemd :

sudo journalctl -u zabbix-server

Ces commandes permettent de repérer les erreurs de démarrage, les problèmes de connexion à la base et certaines difficultés de communication.

15.3 Diagnostic d'un agent inaccessible

Lorsqu'un hôte apparaît indisponible, il faut vérifier le service agent sur la machine concernée, les paramètres Server et Hostname, ainsi que les règles du pare-feu.

Sur Windows :

Get-Service "Zabbix Agent 2"
Test-NetConnection ADRESSE_SERVEUR -Port 10051

Sur Linux :

sudo systemctl status zabbix-agent2
sudo ss -tlnp | grep 10050

Depuis le serveur de supervision :

zabbix_get -s ADRESSE_AGENT -p 10050 -k agent.ping

Le résultat doit être interprété selon le mode de communication configuré.

15.4 Diagnostic de la base de données

Si le serveur Zabbix ne démarre pas correctement, il faut vérifier l'état de MariaDB et les journaux du serveur.

sudo systemctl status mariadb
sudo journalctl -u mariadb
sudo tail -n 50 /var/log/zabbix/zabbix_server.log

Une erreur de connexion peut être liée à un mot de passe incorrect, à une base indisponible ou à une saturation des ressources.

Il ne faut pas réimporter le schéma de la base pour corriger une erreur de connexion. Il faut d'abord identifier la cause réelle.

15.5 Diagnostic des éléments non pris en charge

Lorsqu'un élément passe à l'état non pris en charge, il faut ouvrir sa configuration et consulter le message d'erreur fourni par Zabbix.

Les causes possibles comprennent une clé incorrecte, un droit insuffisant, un agent incompatible ou un type d'information mal configuré.

L'administrateur peut ensuite tester la clé directement avec zabbix_get, lorsque le type d'élément le permet.

15.6 Analyse des événements et confirmation du retour à la normale

Lorsqu'un événement est détecté, il faut vérifier les informations disponibles dans Zabbix, puis confronter le résultat à l'état du système concerné.

Dans le cas d'un service Windows, la console Services permet de vérifier son état réel.

Après le diagnostic, il faut revenir dans Zabbix pour vérifier que le problème a été récupéré et que les dernières données sont de nouveau disponibles.

Le laboratoire présente cette démarche avec les événements Windows relatifs à InventorySvc et AppXSvc.

16. Extensions possibles du laboratoire

La plateforme actuelle constitue une base fonctionnelle pour la supervision de systèmes Linux et Windows.

Plusieurs améliorations peuvent être envisagées pour approfondir les compétences d'administration.

Supervision SNMP : intégrer un switch, un routeur ou un pare-feu et collecter les statistiques des interfaces réseau.

Notifications : configurer SMTP et un webhook afin de transmettre automatiquement les événements aux équipes concernées.

Modèles personnalisés : créer des modèles adaptés aux besoins de l'environnement et les réutiliser sur plusieurs hôtes.

Découverte de bas niveau : automatiser la découverte des disques, des interfaces et des services.

Services et SLA : construire une vue métier de la disponibilité et mesurer les engagements de service.

Proxy Zabbix : déployer un collecteur intermédiaire pour superviser un site distant.

Supervision cloud : intégrer des ressources AWS ou Azure avec des comptes en lecture seule.

Sauvegarde et restauration : automatiser les sauvegardes et valider régulièrement leur restauration.

Ces extensions sont des pistes d'évolution. Elles ne font pas partie des fonctionnalités démontrées dans les captures de la version actuelle du laboratoire.

17. Conclusion

Ce projet a permis de déployer une plateforme de supervision Zabbix 7.0 sur Ubuntu Server 24.04 LTS, avec une installation native reposant sur MariaDB, Nginx et PHP-FPM.

L'intégration de Zabbix Agent 2 sur Linux et Windows Server 2025 a permis de centraliser la collecte des données et de suivre les principales ressources des systèmes.

La découverte réseau a également permis d'identifier plusieurs équipements présents dans le laboratoire. Les données collectées ont été consultées à travers les dernières données, les graphiques et les tableaux de bord.

Enfin, les événements détectés sur Windows Server ont été analysés à partir de Zabbix puis vérifiés directement sur le système concerné. Le suivi des événements jusqu'à leur état résolu a permis de mettre en pratique une démarche de diagnostic et d'exploitation.

Au-delà de l'installation, ce laboratoire constitue une démonstration pratique des principes de supervision : collecte, observation, détection, analyse et suivi des incidents.

Il est conçu comme une base de référence que d'autres administrateurs peuvent reprendre, adapter et enrichir selon leur propre infrastructure.

18. Références techniques

La documentation officielle doit rester la référence principale lors de l'installation, de la configuration et de l'évolution de Zabbix.

Site officiel : https://www.zabbix.com/

Documentation Zabbix 7.0 : https://www.zabbix.com/documentation/7.0/en/manual

Téléchargement Zabbix : https://www.zabbix.com/download

Téléchargement des agents : https://www.zabbix.com/download_agents

Dépôt officiel : https://repo.zabbix.com/

Code source et modèles : https://github.com/zabbix/zabbix

Documentation Ubuntu Server : https://documentation.ubuntu.com/server/
