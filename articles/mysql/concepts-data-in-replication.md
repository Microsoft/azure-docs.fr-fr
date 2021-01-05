---
title: Réplication des données entrantes - Azure Database pour MySQL
description: Découvrez comment utiliser la réplication de données entrantes à synchroniser entre un serveur externe et le service Azure Database pour MySQL.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 8/7/2020
ms.openlocfilehash: 99beddba470f73d6eadb448dfe1b77453ce6426d
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95996217"
---
# <a name="replicate-data-into-azure-database-for-mysql"></a>Répliquer des données dans Azure Database pour MySQL

La Réplication des données entrantes vous permet de synchroniser les données d'un serveur MySQL externe avec le service Azure Database pour MySQL. Le serveur externe peut être hébergé localement, dans des machines virtuelles, ou il peut s'agir d'un service de base de données hébergé par d'autres fournisseurs de services cloud. La réplication des données est basée sur une réplication selon la position du fichier journal binaire (binlog) native à MySQL. Pour en savoir plus sur la réplication binlog, consultez la [vue d’ensemble de la réplication binlog MySQL](https://dev.mysql.com/doc/refman/5.7/en/binlog-replication-configuration-overview.html). 

## <a name="when-to-use-data-in-replication"></a>Quand utiliser la réplication des données entrantes
Voici les principaux scénarios à prendre en compte avant d’utiliser la réplication des données entrantes :

- **Synchronisation de données hybride :** avec la réplication des données entrantes, vous pouvez maintenir la synchronisation des données entre vos serveurs locaux et Azure Database pour MySQL. Cette synchronisation est utile pour créer des applications hybrides. Cette méthode est intéressante si vous disposez d'un serveur de base de données local mais souhaitez déplacer les données vers une région proche des utilisateurs finaux.
- **Synchronisation de plusieurs clouds :** pour les solutions cloud complexes, utilisez la réplication des données entrantes pour synchroniser les données entre Azure Database pour MySQL et différents fournisseurs de cloud, notamment les machines virtuelles et les services de base de données hébergés dans ces clouds.
 
Pour les scénarios de migration, utilisez [Azure Database Migration Service](https://azure.microsoft.com/services/database-migration/) (DMS).

## <a name="limitations-and-considerations"></a>Limitations et considérations

### <a name="data-not-replicated"></a>Données non répliquées
La [*base de données système mysql*](https://dev.mysql.com/doc/refman/5.7/en/system-schema.html) sur le serveur source n’est pas répliquée. Les modifications apportées aux comptes et aux autorisations sur le serveur source ne le sont pas non plus. Si vous créez un compte sur le serveur source et que ce compte a besoin d’accéder au serveur réplica, créez manuellement le même compte du côté du serveur réplica. Pour une présentation des tables figurant dans la base de données système, consultez le [manuel MySQL](https://dev.mysql.com/doc/refman/5.7/en/system-schema.html).

### <a name="filtering"></a>Filtrage
Pour ignorer la réplication des tables à partir de votre serveur source (hébergé localement, dans des machines virtuelles ou un service de base de données hébergé par d’autres fournisseurs cloud), le paramètre `replicate_wild_ignore_table` est pris en charge. Si vous le souhaitez, mettez à jour ce paramètre sur le serveur de réplication hébergé dans Azure à l’aide du [portail Azure](howto-server-parameters.md) ou [d’Azure CLI](howto-configure-server-parameters-using-cli.md).

Consultez la [documentation MySQL](https://dev.mysql.com/doc/refman/8.0/en/replication-options-replica.html#option_mysqld_replicate-wild-ignore-table) pour en savoir plus sur ce paramètre.

### <a name="requirements"></a>Spécifications
- La version du serveur source doit être au moins MySQL version 5.6. 
- La version du serveur source et celle du serveur réplica doivent être identiques. Par exemple, ce doit être MySQL version 5.6 ou MySQL version 5.7.
- Chaque table doit avoir une clé primaire.
- Le serveur source doit utiliser le moteur MySQL InnoDB.
- L’utilisateur doit disposer des autorisations nécessaires pour configurer la journalisation binaire et créer de nouveaux utilisateurs sur le serveur source.
- Si SSL est activé sur le serveur source, vérifiez que le certificat d’autorité de certification SSL fourni pour le domaine a été inclus dans la procédure stockée `mysql.az_replication_change_master`. Consultez les [exemples](./howto-data-in-replication.md#link-source-and-replica-servers-to-start-data-in-replication) suivants et le paramètre `master_ssl_ca`.
- Vérifiez que l’adresse IP du serveur source a été ajoutée aux règles de pare-feu du serveur réplica Azure Database pour MySQL. Mettez à jour les règles de pare-feu à l’aide du [portail Azure](./howto-manage-firewall-using-portal.md) ou d’[Azure CLI](./howto-manage-firewall-using-cli.md).
- Vérifiez que la machine qui héberge le serveur source autorise à la fois le trafic entrant et le trafic sortant sur le port 3306.
- Vérifiez que le serveur source dispose d’une **adresse IP publique** et que le système DNS est accessible publiquement ou comporte un nom de domaine complet (FQDN).

### <a name="other"></a>Autres
- La réplication des données entrantes est prise en charge uniquement dans les niveaux tarifaires Usage général et Mémoire optimisée.
- Les identificateurs de transaction globaux (GTID) ne sont pas pris en charge.

## <a name="next-steps"></a>Étapes suivantes
- Découvrez comment [configurer la réplication des données entrantes](howto-data-in-replication.md)
- En savoir plus sur la [réplication dans Azure avec des réplicas en lecture](concepts-read-replicas.md)
- Découvrez comment [migrer des données avec un temps d’arrêt minimal à l’aide de DMS](howto-migrate-online.md).