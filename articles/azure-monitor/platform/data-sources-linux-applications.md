---
title: Collecte des performances d’application Linux dans Azure Monitor| Microsoft Docs
description: Cet article fournit des détails sur la configuration de l’agent Log Analytics pour Linux pour la collecte des compteurs de performances pour MySQL et Apache HTTP Server.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 05/04/2017
ms.openlocfilehash: db83e24931ec91449ac8c08bf4ce476e0f527b26
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92461343"
---
# <a name="collect-performance-counters-for-linux-applications-in-azure-monitor"></a>Collecte des compteurs de performances pour les applications Linux dans Azure Monitor 

Cet article fournit des informations détaillées sur la configuration de l'[agent Log Analytics pour Linux](https://github.com/Microsoft/OMS-Agent-for-Linux) afin de relever les compteurs de performances d'applications spécifiques dans Azure Monitor.  Les applications incluses dans cet article sont :  

- [MySQL](#mysql)
- [Apache HTTP Server](#apache-http-server)

## <a name="mysql"></a>MySQL
Si le serveur MySQL ou MariaDB est détecté sur l’ordinateur lorsque l’agent Log Analytics est installé, un fournisseur de surveillance des performances pour le serveur MySQL est automatiquement installé. Ce fournisseur se connecte au serveur MySQL/MariaDB local pour afficher les statistiques de performances. Vous devez configurer les informations d’identification de l’utilisateur MySQL, afin que le fournisseur puisse accéder au serveur MySQL.

### <a name="configure-mysql-credentials"></a>Configuration des informations d’identification de MySQL
Le fournisseur MySQL OMI nécessite un utilisateur MySQL préconfiguré et des bibliothèques clientes MySQL installées pour pouvoir interroger les informations d’intégrité et de performances de l’instance MySQL.  Ces informations d’identification sont stockées dans un fichier d’authentification, lui-même stocké sur l’agent Linux.  Le fichier d’authentification indique l’adresse de liaison et le port qu’écoute l’instance MySQL, ainsi que les informations d’identification à utiliser pour collecter des mesures.  

Pendant l’installation de l’agent Log Analytics pour Linux, le fournisseur OMI MySQL recherche l’adresse de liaison et le port dans les fichiers de configuration my.cnf MySQL (emplacements par défaut), et configure partiellement le fichier d’authentification OMI MySQL.

Le fichier d’authentification MySQL est stocké dans `/var/opt/microsoft/mysql-cimprov/auth/omsagent/mysql-auth`.


### <a name="authentication-file-format"></a>Format du fichier d’authentification
Le format du fichier d’authentification OMI MySQL est le suivant

> [Port]=[Adresse de liaison], [nom d’utilisateur], [Mot de passe codé en base64]  
> (Port) = (Adresse de liaison), (nom d’utilisateur), (mot de passe codé en base64)  
> (Port) = (Adresse de liaison), (nom d’utilisateur), (mot de passe codé en base64)  
> AutoUpdate=[true|false]  

Les entrées du fichier d’authentification sont décrites dans le tableau suivant.

| Propriété | Description |
|:--|:--|
| Port | Représente le port actif sur lequel écoute l’instance MySQL. Le port 0 indique que les propriétés suivantes sont utilisées pour l’instance par défaut. |
| Adresse de liaison| Adresse de liaison MySQL active. |
| username| Utilisateur MySQL utilisé pour surveiller l’instance de serveur MySQL. |
| Mot de passe encodé en base 64| Mot de passe de l’utilisateur surveillant MySQL encodé en Base64. |
| AutoUpdate| Indique si les modifications doivent à nouveau être recherchées dans le fichier my.cnf et si le fichier d’authentification MySQL OMI doit être remplacé lorsque le fournisseur MySQL est mis à niveau. |

### <a name="default-instance"></a>Instance par défaut
Le fichier d’authentification MySQL OMI peut définir un numéro d’instance et de port par défaut pour gérer plusieurs instances de MySQL sur un hôte Linux plus facilement.  L’instance par défaut est définie par une instance avec le port 0. Toutes les instances supplémentaires hériteront des propriétés définies à partir de l’instance par défaut, sauf si elles indiquent des valeurs différentes. Par exemple, si l’instance MySQL qui écoute le port 3308 est ajoutée, l’adresse de liaison, le nom d’utilisateur et le mot de passe encodé en Base64 de l’instance par défaut sont utilisés pour tester et surveiller l’instance qui écoute le port 3308. Si l’instance sur le port 3308 est liée à une autre adresse et utilise les mêmes nom d’utilisateur et mot de passe MySQL, seule l’adresse de liaison est nécessaire. Les autres propriétés seront héritées.

Le tableau suivant contient des exemples de paramètres d’instance 

| Description | Fichier |
|:--|:--|
| Instance par défaut et instance avec le port 3308. | `0=127.0.0.1, myuser, cnBwdA==`<br>`3308=, ,`<br>`AutoUpdate=true` |
| Instance par défaut et instance avec le port 3308 et un autre nom d’utilisateur et un autre mot de passe. | `0=127.0.0.1, myuser, cnBwdA==`<br>`3308=127.0.1.1, myuser2,cGluaGVhZA==`<br>`AutoUpdate=true` |


### <a name="mysql-omi-authentication-file-program"></a>Programme du fichier d’authentification OMI MySQL
Le programme du fichier d’authentification OMI MySQL est inclus dans l’installation du fournisseur OMI MySQL. Il peut être utilisé pour modifier le fichier d’authentification OMI MySQL. Vous trouverez le programme du fichier d’authentification à l’emplacement suivant.

`/opt/microsoft/mysql-cimprov/bin/mycimprovauth`

> [!NOTE]
> Le fichier des informations d’identification doit être accessible en lecture par le compte omsagent. L’exécution de la commande mycimprovauth comme omsgent est recommandée.

Le tableau suivant fournit des détails sur la syntaxe pour utiliser mycimprovauth.

| Opération | Exemple | Description
|:--|:--|:--|
| autoupdate *false ou true* | mycimprovauth autoupdate false | Définit si le fichier d’authentification sera automatiquement mis à jour au redémarrage ou lors de la mise à jour. |
| default *adresse_de_liaison nom_utilisateur mot_de_passe* | mycimprovauth default 127.0.0.1 root pwd | Définit l’instance par défaut dans le fichier d’authentification OMI MySQL.<br>Le champ du mot de passe doit être renseigné en texte brut ; le mot de passe dans le fichier d’authentification MySQL OMI sera encodé en Base64. |
| delete *port par défaut ou numéro_port* | mycimprovauth 3308 | Supprime l’instance indiquée par valeur par défaut ou numéro de port. |
| help | mycimprov help | Imprime une liste de commandes à utiliser. |
| print | mycimprov print | Imprime un fichier d’authentification OMI MySQL facile à lire. |
| update num_port *adresse_de_liaison nom_utilisateur mot_de_passe* | mycimprov update 3307 127.0.0.1 root pwd | Met à jour l’instance indiquée ou ajoute l’instance si elle n’existe pas. |

Les exemples de commande suivants définissent un compte utilisateur par défaut pour le serveur MySQL sur localhost.  Le champ du mot de passe doit être renseigné en texte brut ; le mot de passe dans le fichier d’authentification MySQL OMI sera encodé en Base64

```console
sudo su omsagent -c '/opt/microsoft/mysql-cimprov/bin/mycimprovauth default 127.0.0.1 <username> <password>'
sudo /opt/omi/bin/service_control restart
```

### <a name="database-permissions-required-for-mysql-performance-counters"></a>Autorisations de base de données requises pour les compteurs de performances MySQL
L’utilisateur MySQL requiert l’accès aux requêtes suivantes pour collecter les données de performances du serveur MySQL. 

```sql
SHOW GLOBAL STATUS;
SHOW GLOBAL VARIABLES:
```

L’utilisateur MySQL requiert également un accès SELECT aux tables par défaut suivantes.

- information_schema
- mysql. 

Ces privilèges peuvent être accordés à l’aide des commandes Grant suivantes.

```sql
GRANT SELECT ON information_schema.* TO ‘monuser’@’localhost’;
GRANT SELECT ON mysql.* TO ‘monuser’@’localhost’;
```

> [!NOTE]
> Pour accorder des autorisations à un utilisateur surveillant MySQL, l’utilisateur qui souhaite accorder un privilège doit posséder le privilège « GRANT option », ainsi que le privilège accordé.

### <a name="define-performance-counters"></a>Définition des compteurs de performances

Une fois l’agent Log Analytics pour Linux configuré pour envoyer des données vers Azure Monitor, vous devez configurer les compteurs de performances à collecter.  Utilisez la procédure décrite dans [Sources de données de performances Windows et Linux dans Azure Monitor](data-sources-performance-counters.md) avec les compteurs du tableau suivant.

| Nom d’objet | Nom de compteur |
|:--|:--|
| Base de données MySQL | Disk Space in Bytes |
| Base de données MySQL | Tables |
| MySQL Server | Aborted Connection Pct |
| MySQL Server | Connection Use Pct |
| MySQL Server | Disk Space Use in Bytes |
| MySQL Server | Full Table Scan Pct |
| MySQL Server | InnoDB Buffer Pool Hit Pct |
| MySQL Server | InnoDB Buffer Pool Use Pct |
| MySQL Server | InnoDB Buffer Pool Use Pct |
| MySQL Server | Key Cache Hit Pct |
| MySQL Server | Key Cache Use Pct |
| MySQL Server | Key Cache Write Pct |
| MySQL Server | Query Cache Hit Pct |
| MySQL Server | Query Cache Prunes Pct |
| MySQL Server | Query Cache Use Pct |
| MySQL Server | Table Cache Hit Pct |
| MySQL Server | Table Cache Use Pct |
| MySQL Server | Table Lock Contention Pct |

## <a name="apache-http-server"></a>Apache HTTP Server 
Si Apache HTTP Server est détecté sur l’ordinateur lors de l’installation du groupe omsagent, un fournisseur de surveillance des performances pour Apache HTTP Server est automatiquement installé. Ce fournisseur repose sur un module Apache qui doit être chargé dans Apache HTTP Server afin d’accéder aux données de performances. Le module peut être chargé à l’aide de la commande suivante :

```console
sudo /opt/microsoft/apache-cimprov/bin/apache_config.sh -c
```

Pour décharger le module de surveillance Apache, exécutez la commande suivante :

```console
sudo /opt/microsoft/apache-cimprov/bin/apache_config.sh -u
```

### <a name="define-performance-counters"></a>Définition des compteurs de performances

Une fois l’agent Log Analytics pour Linux configuré pour envoyer des données vers Azure Monitor, vous devez configurer les compteurs de performances à collecter.  Utilisez la procédure décrite dans [Sources de données de performances Windows et Linux dans Azure Monitor](data-sources-performance-counters.md) avec les compteurs du tableau suivant.

| Nom d’objet | Nom de compteur |
|:--|:--|
| Apache HTTP Server | Busy Workers |
| Apache HTTP Server | Idle Workers |
| Apache HTTP Server | Pct Busy Workers |
| Apache HTTP Server | Total Pct CPU |
| Apache Virtual Host | Errors per Minute - Client |
| Apache Virtual Host | Errors per Minute - Server |
| Apache Virtual Host | KB per Request |
| Apache Virtual Host | Requests KB per Second |
| Apache Virtual Host | Requests per Second |



## <a name="next-steps"></a>Étapes suivantes
* [Collecter les compteurs de performances](data-sources-performance-counters.md) à partir d’agents Linux.
* Découvrez les [requêtes dans les journaux](../log-query/log-query-overview.md) pour analyser les données collectées à partir de sources de données et de solutions. 
