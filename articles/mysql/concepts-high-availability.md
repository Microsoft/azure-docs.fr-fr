---
title: Haute disponibilité - Azure Database pour MySQL
description: Cet article fournit des informations sur la haute disponibilité dans Azure Database pour MySQL
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: conceptual
ms.date: 7/7/2020
ms.openlocfilehash: b301946ce818559510b4e401b1f0aaf7c235d5a3
ms.sourcegitcommit: 80034a1819072f45c1772940953fef06d92fefc8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93242294"
---
# <a name="high-availability-in-azure-database-for-mysql"></a>Haute disponibilité dans Azure Database pour MySQL
Le service Azure Database pour MySQL offre un niveau élevé de disponibilité de [99,99 %](https://azure.microsoft.com/support/legal/sla/mysql) de temps d’activité, financièrement garanti par un contrat de niveau de service (SLA). Azure Database pour MySQL offre une haute disponibilité pendant des événements planifiés tels qu’une opération de calcul de mise à l’échelle demandée par l’utilisateur, ainsi que pendant des événements non planifiés tels que des défaillances de matériel, de logiciels ou de réseau sous-jacents. Azure Database pour MySQL peut rapidement récupérer de la plupart des circonstances critiques, ce qui garantit pratiquement une absence totale de temps d’arrêt.

Azure Database pour MySQL convient pour l’exécution de bases de données stratégiques nécessitant une durée de bon fonctionnement longue. Reposant sur l’architecture Azure, le service intègre des fonctionnalités de haute disponibilité, de redondance et de résilience pour réduire les temps d’arrêt de base de données dus à des interruptions planifiées et non planifiées, sans que vous ayez à configurer des composants supplémentaires. 

## <a name="components-in-azure-database-for-mysql"></a>Composants dans Azure Database pour MySQL

| **Composant** | **Description**|
| ------------ | ----------- |
| <b>Serveur de base de données MySQL | Azure Database pour MySQL offre des fonctionnalités de sécurité, d’isolement, de protection des ressources et de redémarrage rapide pour les serveurs de base de données. Ces fonctionnalités facilitent des opérations telles que la mise à l’échelle et la récupération du serveur de base de données après une panne en quelques secondes. <br/> Les modifications de données dans le serveur de base de données se produisent généralement dans le contexte d’une transaction de base de données. Toutes les modifications de base de données sont enregistrées de façon synchrone sous la forme de journaux WAL (write-ahead log), ib_log, sur le Stockage Azure attaché au serveur de base de données. Au cours de processus de [point de contrôle](https://dev.mysql.com/doc/refman/5.7/en/innodb-checkpoints.html) de base de données, les pages de données de la mémoire du serveur de base de données sont également vidées dans le stockage. |
| <b>Stockage à distance | Tous les fichiers de données physiques et de journaux MySQL sont stockés sur le service Stockage Azure qui est architecturé pour stocker trois copies des données à l’intérieur d’une région afin de garantir la redondance, la disponibilité et la fiabilité des données. La couche de stockage est également indépendante du serveur de base de données. Elle peut être détachée d’un serveur de base de données défaillant et être rattachée à un nouveau serveur de base de données dans un délai de quelques secondes. En outre, le service Stockage Azure surveille constamment les erreurs de stockage. Toute altération de bloc détectée est automatiquement corrigée par l’instanciation d’une nouvelle copie du stockage. |
| <b>Passerelle | La passerelle agit comme un proxy de base de données, achemine toutes les connexions clientes vers le serveur de base de données. |

## <a name="planned-downtime-mitigation"></a>Réduction des temps d’arrêt planifiés
Azure Database pour MySQL est architecturé pour offrir une haute disponibilité pendant les opérations planifiées nécessitant un temps d’arrêt. 

:::image type="content" source="./media/concepts-high-availability/elastic-scaling-mysql-server.png" alt-text="vue de la mise à l’échelle élastique dans Azure MySQL":::

Voici quelques scénarios de maintenance planifiée :

| **Scénario** | **Description**|
| ------------ | ----------- |
| <b>Mise à l’échelle du calcul | Quand l’utilisateur effectue une opération de mise à l’échelle du calcul, un nouveau serveur de base de données est approvisionné à l’aide de la configuration de calcul mise à l’échelle. Sur l’ancien serveur de base de données, les points de contrôle actifs sont autorisés à cesser d’opérer, les connexions clientes sont vidées, toutes les transactions non validées sont annulées, puis le serveur est arrêté. Le stockage est ensuite détaché de l’ancien serveur de base de données et attaché au nouveau. Quand l’application cliente retente la connexion ou tente d’établir une nouvelle connexion, la passerelle dirige la demande de connexion vers le nouveau serveur de base de données.|
| <b>Augmentation du stockage | L’augmentation du stockage est une opération en ligne qui n’interrompt pas le serveur de base de données.|
| <b>Déploiement de nouveaux logiciels (Azure) | Le déploiement de nouvelles fonctionnalités ou la corrections de bogues se produisent automatiquement dans le cadre de la maintenance planifiée du service. Pour plus d’informations, consultez la [documentation](concepts-monitoring.md#planned-maintenance-notification) et visitez votre [portail](https://aka.ms/servicehealthpm).|
| <b>Mises à niveau de version mineure | Azure Database pour MySQL opère automatiquement la mise à niveau des serveurs de base de données vers la version mineure déterminée par Azure. Cela se produit dans le cadre de la maintenance planifiée du service. L’opération entraîne un bref temps d’arrêt de quelques secondes, et le serveur de base de données est automatiquement redémarré avec la nouvelle version mineure. Pour plus d’informations, consultez la [documentation](concepts-monitoring.md#planned-maintenance-notification) et visitez votre [portail](https://aka.ms/servicehealthpm).|


##  <a name="unplanned-downtime-mitigation"></a>Réduction des temps d’arrêt non planifiés

Des temps d’arrêt non planifiés peuvent se produire suite à des défaillances imprévues telles qu’un échec matériel sous-jacent, des problèmes de mise en réseau et des bogues logiciels. Si le serveur de base de données tombe en panne de façon inattendue, un nouveau serveur de base de données est automatiquement approvisionné en quelques secondes. Le stockage étendu est automatiquement attaché au nouveau serveur de base de données. Le moteur MySQL effectue l’opération de récupération à l’aide des fichiers WAL et de base de données, puis ouvre le serveur de base de données pour permettre aux clients de se connecter. Les transactions non validées sont perdues et doivent être retentées par l’application. Bien qu’il ne soit pas possible d’éviter un temps d’arrêt non planifié, le service Azure Database pour MySQL réduit les temps d’arrêt en effectuant automatiquement des opérations de récupération au niveau du serveur de base de données et des couches de stockage, sans intervention humaine. 


:::image type="content" source="./media/concepts-high-availability/availability-for-mysql-server.png" alt-text="affichage de la haute disponibilité dans Azure MySQL":::

### <a name="unplanned-downtime-failure-scenarios-and-service-recovery"></a>Temps d’arrêt non planifié : scénarios d’échec et récupération du service
Voici quelques scénarios d’échec et comment le service Azure Database pour MySQL récupère automatiquement :

| **Scénario** | **Récupération automatique** |
| ---------- | ---------- |
| <B>Échec du serveur de base de données | En cas d’arrêt du serveur de base de données en raison d’une erreur matérielle sous-jacente, les connexions actives sont abandonnées et toutes les transactions en cours interrompues. Un nouveau serveur de base de données est déployé automatiquement, et le stockage de données étendu est attaché au nouveau serveur de base de données. Une fois la récupération de la base de données terminée, les clients peuvent se connecter au nouveau serveur de base de données via la passerelle. <br /> <br /> Des applications utilisant les bases de données MySQL doivent être créées de manière à détecter et à retenter les connexions abandonnées et les transactions ayant échoué.  Quand l’application effectue une nouvelle tentative, la passerelle redirige en toute transparence la connexion vers le serveur de base de données nouvellement créé. |
| <B>Échec de stockage | Les applications ne détectent aucun impact des problèmes liés au stockage, tels qu’une défaillance de disque ou une corruption de bloc physique. Les données étant stockées dans 3 copies, la copie des données est servie par le stockage survivant. Les altérations de bloc sont corrigées automatiquement. En cas de perte d’une copie des données, une nouvelle est automatiquement créée. |

Voici quelques scénarios d’échec qui nécessitent une action de l’utilisateur pour la récupération :

| **Scénario** | **Plan de récupération** |
| ---------- | ---------- |
| <b> Défaillance de région | Une défaillance de région est un événement rare. Toutefois, si vous avez besoin d’une protection contre une défaillance de région, vous pouvez configurer un ou plusieurs réplicas en lecture dans d’autres régions à des fins de récupération d’urgence (pour plus d’informations, consultez [cet article](howto-read-replicas-portal.md) sur la création et la gestion des réplicas en lecture). En cas de défaillance au niveau d’une région, vous pouvez promouvoir manuellement le réplica en lecture configuré sur l’autre région comme serveur de base de données de production. |
| <b> Erreurs logiques/de l’utilisateur | La récupération d’erreurs de l’utilisateur, telles qu’une suppression accidentelle de tables ou une mise à jour incorrecte de données, implique l’exécution d’une [récupération jusqu’à une date et heure](concepts-backup.md) (PITR), en restaurant et récupérant les données jusqu’au moment où l’erreur s’est produite.<br> <br>  Si vous ne souhaitez restaurer qu’un sous-ensemble des bases de données ou des tables en particulier plutôt que toutes les bases de données du serveur de base de données, vous pouvez restaurer celui-ci dans une nouvelle instance, exporter la ou les tables avec [mysqldump](concepts-migrate-dump-restore.md), puis utiliser [restore](concepts-migrate-dump-restore.md#restore-your-mysql-database-using-command-line-or-mysql-workbench) pour les restaurer dans votre base de données. |



## <a name="summary"></a>Résumé

Le service Azure Database pour MySQL offre une fonctionnalité de redémarrage rapide des serveurs de base de données, un stockage redondant et un routage efficace à partir de la passerelle. Pour renforcer la protection des données, vous pouvez configurer des sauvegardes à géo-répliquer et déployer un ou plusieurs réplicas en lecture dans d’autres régions. Avec ses fonctionnalités de haute disponibilité intégrées, Azure Database pour MySQL protège vos bases de données contre les pannes les plus courantes, et offre un [contrat de niveau de service de 99,99 % de temps d’activité](https://azure.microsoft.com/support/legal/sla/mysql) adossé à une garantie financière. Toutes ces fonctionnalités de disponibilité et de fiabilité permettent à Azure d’être la plateforme idéale pour l’exécution de vos applications stratégiques.

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur les [Régions Azure](../availability-zones/az-overview.md)
- En savoir plus sur le [traitement des erreurs de connectivité transitoires](concepts-connectivity.md)
- Apprendre à [répliquer vos données avec des réplicas en lecture](howto-read-replicas-portal.md)