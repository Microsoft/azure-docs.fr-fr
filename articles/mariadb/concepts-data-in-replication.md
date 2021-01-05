---
title: Réplication des données entrantes - Azure Database for MariaDB
description: Découvrez l’utilisation de la réplication de données entrantes à synchroniser entre un serveur externe et le service Azure Database for MariaDB.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: conceptual
ms.date: 3/18/2020
ms.openlocfilehash: 331e064bcf11af31a778cb8dd06c463712421b7c
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94533427"
---
# <a name="replicate-data-into-azure-database-for-mariadb"></a>Répliquer des données dans Azure Database for MariaDB

La réplication des données entrantes permet de synchroniser les données à partir d’un serveur MariaDB qui s’exécute en local, dans des machines virtuelles ou des services de base de données hébergés par d’autres fournisseurs cloud dans le service Azure Database for MariaDB. La réplication des données entrantes est basée sur la réplication selon la position du fichier journal binaire (binlog) native à MariaDB. Pour découvrir plus en détail la réplication binlog, consultez la [vue d’ensemble de la réplication binlog](https://mariadb.com/kb/en/library/replication-overview/).

## <a name="when-to-use-data-in-replication"></a>Quand utiliser la réplication des données entrantes
Voici les principaux scénarios à prendre en compte avant d’utiliser la réplication des données entrantes :

- **Synchronisation de données hybride :** avec la réplication des données entrantes, vous pouvez maintenir des données synchronisées entre vos serveurs locaux et Azure Database for MariaDB. Cette synchronisation est utile pour créer des applications hybrides. Cette méthode est intéressante si vous disposez d’un serveur de base de données local mais souhaitez déplacer les données vers une région proche des utilisateurs finaux.
- **Synchronisation de plusieurs cloud :** pour les solutions cloud complexes, utilisez la réplication des données entrantes pour synchroniser les données entre Azure Database for MariaDB et différents fournisseurs de cloud, notamment les machines virtuelles et les services de base de données hébergés dans ces clouds.

## <a name="limitations-and-considerations"></a>Limitations et considérations

### <a name="data-not-replicated"></a>Données non répliquées
La [*base de données système mysql*](https://mariadb.com/kb/en/library/the-mysql-database-tables/) sur le serveur source n’est pas répliquée. Les modifications apportées aux comptes et aux autorisations sur le serveur source ne le sont pas non plus. Si vous créez un compte sur le serveur source et que ce compte a besoin d’accéder au serveur réplica, créez manuellement le même compte du côté du serveur réplica. Pour une présentation des tables figurant dans la base de données système, consultez la [documentation MariaDB](https://mariadb.com/kb/en/library/the-mysql-database-tables/).

### <a name="requirements"></a>Spécifications
- Le serveur source doit posséder au moins la version 10.2 de MariaDB.
- La version du serveur source et celle du serveur réplica doivent être identiques. Par exemple, les deux doivent être MariaDB version 10.2.
- Chaque table doit avoir une clé primaire.
- Le serveur source doit de préférence utiliser le moteur InnoDB.
- L’utilisateur doit disposer des autorisations nécessaires pour configurer la journalisation binaire et créer de nouveaux utilisateurs sur le serveur source.
- Si SSL est activé sur le serveur source, vérifiez que le certificat d’autorité de certification SSL fourni pour le domaine a été inclus dans la procédure stockée `mariadb.az_replication_change_master`. Consultez les [exemples](howto-data-in-replication.md#link-the-source-and-replica-servers-to-start-data-in-replication) suivants et le paramètre `master_ssl_ca`.
- Vérifiez que l’adresse IP du serveur source a été ajoutée aux règles de pare-feu du serveur réplica Azure Database for MariaDB. Mettez à jour les règles de pare-feu à l’aide du [portail Azure](howto-manage-firewall-portal.md) ou d’[Azure CLI](howto-manage-firewall-cli.md).
- Vérifiez que la machine qui héberge le serveur source autorise à la fois le trafic entrant et le trafic sortant sur le port 3306.
- Vérifiez que le serveur source dispose d’une **adresse IP publique** et que le système DNS est accessible publiquement ou comporte un nom de domaine complet (FQDN).

### <a name="other"></a>Autres
- La réplication des données entrantes est prise en charge uniquement dans les niveaux tarifaires Usage général et Mémoire optimisée.

## <a name="next-steps"></a>Étapes suivantes
- Découvrez comment [configurer la réplication des données entrantes](howto-data-in-replication.md).
