---
title: Procédures stockées de gestion - Azure Database pour MySQL
description: Découvrez les procédures stockées d’Azure Database pour MySQL qui vous aideront à configurer la réplication des données entrantes, à définir le fuseau horaire et à supprimer des requêtes.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 3/18/2020
ms.openlocfilehash: 7a1aa061bb8c8be3a676e0e5bb690b2a9749b6c8
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94536130"
---
# <a name="azure-database-for-mysql-management-stored-procedures"></a>Procédures stockées de gestion Azure Database pour MySQL

Des procédures stockées sont disponibles sur les serveurs Azure Database pour MySQL afin de faciliter la gestion de votre serveur MySQL. Cela comprend la gestion des connexions de votre serveur, des requêtes et de la configuration de la réplication des données entrantes.  

## <a name="data-in-replication-stored-procedures"></a>Procédures stockées de réplication des données entrantes

La réplication des données entrantes permet de synchroniser les données à partir d’un serveur MySQL qui s’exécute en local, dans des machines virtuelles ou des services de base de données hébergés par d’autres fournisseurs cloud dans le service Azure Database pour MySQL.

Les procédures stockées suivantes sont utilisées pour définir ou supprimer la réplication des données entrantes entre un serveur source et un réplica.

|**Nom de la procédure stockée**|**Paramètres d’entrée**|**Paramètres de sortie**|**Remarque sur l’utilisation**|
|-----|-----|-----|-----|
|*mysql.az_replication_change_master*|master_host<br/>master_user<br/>master_password<br/>master_port<br/>master_log_file<br/>master_log_pos<br/>master_ssl_ca|N/A|Pour transférer des données en mode SSL, passez le contexte du certificat d’autorité de certification dans le paramètre master_ssl_ca. </br><br>Pour transférer des données sans SSL, transmettez une chaîne vide dans le paramètre master_ssl_ca.|
|*mysql.az_replication _start*|N/A|N/A|Lance la réplication.|
|*mysql.az_replication _stop*|N/A|N/A|Arrête la réplication.|
|*mysql.az_replication _remove_master*|N/A|N/A|Supprime la relation de réplication entre le serveur source et le réplica.|
|*mysql.az_replication_skip_counter*|N/A|N/A|Ignore une erreur de réplication.|

Pour configurer la réplication des données entrantes entre un serveur source et un réplica dans Azure Database pour MySQL, consultez [Guide pratique pour configurer la réplication des données entrantes](howto-data-in-replication.md).

## <a name="other-stored-procedures"></a>Autres procédures stockées

Les procédures stockées suivantes sont disponibles dans Azure Database pour MySQL pour gérer votre serveur.

|**Nom de la procédure stockée**|**Paramètres d’entrée**|**Paramètres de sortie**|**Remarque sur l’utilisation**|
|-----|-----|-----|-----|
|*mysql.az_kill*|processlist_id|N/A|Équivaut à la commande [`KILL CONNECTION`](https://dev.mysql.com/doc/refman/8.0/en/kill.html). Met fin à la connexion associée au processlist_id fourni après avoir mis fin à l’exécution de toute instruction exécutée actuellement par la connexion.|
|*mysql.az_kill_query*|processlist_id|N/A|Équivaut à la commande [`KILL QUERY`](https://dev.mysql.com/doc/refman/8.0/en/kill.html). Met fin à l’instruction exécutée actuellement par la connexion. Laisse la connexion active.|
|*mysql.az_load_timezone*|N/A|N/A|Charge les tables de fuseau horaire pour permettre d’affecter des valeurs nommées au paramètre `time_zone` (par exemple « US/Pacific »).|

## <a name="next-steps"></a>Étapes suivantes
- Découvrez comment configurer la [réplication des données entrantes](howto-data-in-replication.md).
- Découvrez comment utiliser les [tables de fuseau horaire](howto-server-parameters.md#working-with-the-time-zone-parameter).
