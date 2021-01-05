---
title: Vue d’ensemble de la continuité d’activité – Serveur flexible Azure Database pour MySQL
description: Découvrir les concepts de la continuité d’activité avec le serveur flexible Azure Database pour MySQL
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: conceptual
ms.date: 09/21/2020
ms.openlocfilehash: c29e952e22aaccf31c10de8f6e16d240b4660a23
ms.sourcegitcommit: 80034a1819072f45c1772940953fef06d92fefc8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93240713"
---
# <a name="overview-of-business-continuity-with-azure-database-for-mysql---flexible-server-preview"></a>Vue d’ensemble de la continuité d’activité avec Azure Database pour MySQL - Serveur flexible (préversion)

> [!IMPORTANT]
> Azure Database pour MySQL - Serveur flexible est actuellement en préversion publique.

Le serveur flexible Azure Database pour MySQL offre des fonctionnalités de continuité d’activité qui protègent vos bases de données en cas d’interruption planifiée et non planifiée. Des fonctionnalités, telles que les sauvegardes automatisées et la haute disponibilité offrent plusieurs niveaux de protection contre les pannes, avec différents temps de récupération et risques de perte de données. Lorsque vous concevez des applications avec l’idée de les protéger contre les défaillances, vous devez tenir compte de l’objectif de délai de récupération (RTO) et de l’objectif de point de récupération (RPO) pour chaque application. L’objectif de délai de récupération est la tolérance aux temps d’arrêt, tandis que l’objectif de point de récupération est la tolérance à la perte de données après une disruption du service de base de données.

Le tableau ci-dessous illustre les fonctionnalités qu’offre le serveur flexible.

| **Fonctionnalité** | **Description** | **Restrictions** |
| ---------- | ----------- | ------------ |
| **Sauvegarde et récupération** | Le serveur flexible effectue automatiquement des sauvegardes quotidiennes de vos fichiers de base de données et sauvegarde en permanence les journaux des transactions. Les sauvegardes peuvent être conservées sur une période comprise entre 1 et 35 jours. Vous pouvez restaurer votre serveur de base de données à n’importe quel point dans le temps au cours de la période de conservation de votre sauvegarde. Le temps de récupération dépend du volume des données à restaurer et du temps d’exécution de la récupération des journaux. Pour plus d’informations, consultez [Concepts – Sauvegarde et restauration](./concepts-backup-restore.md). |Les données de sauvegarde sont conservées dans la région |
| **Sauvegarde localement redondante** | Les sauvegardes d’un serveur flexible sont stockées de façon automatique et sécurisée dans un stockage localement redondant, au sein d’une région et dans la même zone de disponibilité. Les sauvegardes localement redondantes répliquent les fichiers de données de la sauvegarde serveur trois fois dans un même endroit physique au sein de la région primaire. Le stockage des sauvegardes localement redondantes offre une durabilité des objets d’au moins 99,999999999 % (11 « neuf ») sur une année donnée. Pour plus d’informations, consultez [Concepts – Sauvegarde et restauration](./concepts-backup-restore.md).| Applicable dans toutes les régions |
| **Haute disponibilité redondante interzone** | Le serveur flexible peut être déployé en mode haute disponibilité, ce qui a pour effet de déployer le serveur principal et le serveur de secours dans deux zones de disponibilité différentes au sein d’une même région. C’est une protection contre les défaillances au niveau de la zone qui permet également de réduire les temps d’arrêt des applications pendant les interruptions planifiées et non planifiées. Les données du serveur principal sont répliquées de façon synchrone sur le réplica de secours. Lors d’un événement de temps d’arrêt, le serveur de base de données est automatiquement basculé vers le réplica de secours. Pour plus d’informations, consultez [Concepts – Haute disponibilité](./concepts-high-availability.md). | Prise en charge dans les niveaux de calcul à usage général et à mémoire optimisée. Disponible uniquement dans les régions où plusieurs zones sont disponibles.|
| **Partages de fichiers Premium** | Les fichiers de base de données sont stockés dans des partages de fichiers Azure Premium très robustes et fiables qui assurent la redondance des données au moyen de trois copies du réplica, stockées dans une zone de disponibilité à l’aide de fonctionnalités automatiques de récupération de données. Pour plus d’informations, consultez les [Partages de fichiers Premium](../../storage/files/storage-how-to-create-premium-fileshare.md). | Données stockées dans une zone de disponibilité |

> [!IMPORTANT]
> Aucun temps d’activité ni aucun contrat de niveau de service RTO et RPO ne sont proposés pendant la préversion. Détails fournis dans cette page uniquement à titre d’information et à des fins de planification.

## <a name="planned-downtime-mitigation"></a>Réduction des temps d’arrêt planifiés

Voici quelques scénarios de maintenance planifiée qui subissent un temps d’arrêt :

| **Scénario** | **Processus**|
| :------------ | :----------- |
| **Mise à l’échelle du calcul (Utilisateur)**| Quand vous effectuez une opération de mise à l’échelle du calcul, un nouveau serveur flexible est provisionné en utilisant la configuration du calcul mis à l’échelle. Sur le serveur de base de données existant, les points de contrôle actifs sont autorisés à se produire, les connexions clientes sont purgées, toutes les transactions non validées sont annulées, puis le serveur est arrêté. Le stockage est ensuite attaché au nouveau serveur et la base de données démarrée, ce qui entraîne une récupération si nécessaire avant d’accepter les connexions clientes. |
| **Déploiement de nouveaux logiciels (Azure)** | Le déploiement de nouvelles fonctionnalités ou les corrections de bogues se produisent automatiquement dans le cadre de la maintenance planifiée du service, et vous pouvez planifier le moment auquel ces activités se produisent. Pour plus d’informations, consultez la [documentation](https://aka.ms/servicehealthpm) et votre [portail](https://aka.ms/servicehealthpm). |
| **Mises à niveau de version mineure (Azure)** | Azure Database pour MySQL opère automatiquement la mise à niveau des serveurs de base de données vers la version mineure déterminée par Azure. Cela se produit dans le cadre de la maintenance planifiée du service. L’opération entraîne un bref temps d’arrêt de quelques secondes, et le serveur de base de données est automatiquement redémarré avec la nouvelle version mineure. Pour plus d’informations, consultez la [documentation](../concepts-monitoring.md#planned-maintenance-notification) et votre [portail](https://aka.ms/servicehealthpm).|

Lorsque le serveur flexible est configuré avec la **haute disponibilité redondante interzone** , il effectue d’abord des opérations sur le serveur de secours et ensuite sur le serveur principal, sans basculement. Pour plus d’informations, consultez [Concepts – Haute disponibilité](./concepts-high-availability.md).

## <a name="unplanned-downtime-mitigation"></a>Réduction des temps d’arrêt non planifiés

Des temps d’arrêt non planifiés peuvent se produire suite à des défaillances imprévues, telles qu’une panne matérielle sous-jacente, des problèmes réseau et des bogues logiciels. Si le serveur de base de données tombe en panne de façon inattendue et qu’il est configuré avec une haute disponibilité [HA], le réplica de secours est activé. Si ce n’est pas le cas, un nouveau serveur de base de données est automatiquement provisionné. Alors qu’il n’est pas possible d’éviter un temps d’arrêt non planifié, le serveur flexible réduit ce temps d’arrêt en effectuant automatiquement des opérations de récupération au niveau du serveur de base de données et des couches de stockage, sans intervention humaine.

### <a name="unplanned-downtime-failure-scenarios-and-service-recovery"></a>Temps d’arrêt non planifié : scénarios d’échec et récupération du service

Voici quelques scénarios d’échec non planifiés et le processus de récupération :

| **Scénario** | **Processus de récupération [sans haute disponibilité]** | **Processus de récupération [haute disponibilité]** |
| :---------- | ---------- | ------- |
| **Panne du serveur de base de données** | En cas d’arrêt du serveur de base de données en raison d’une erreur matérielle sous-jacente, les connexions actives sont abandonnées et toutes les transactions en cours interrompues. Azure tente de redémarrer le serveur de base de données. En cas de réussite, la récupération de la base de données est effectuée. Si le redémarrage échoue, une tentative de redémarrage du serveur de base de données est effectuée sur un autre nœud physique.  <br /> <br /> Le temps de récupération (RTO) dépend de différents facteurs, dont l’activité au moment de l’erreur, telle qu’une transaction de grande ampleur, et le volume de la récupération à effectuer pendant le processus de démarrage du serveur de base de données. <br /> <br /> Des applications utilisant les bases de données MySQL doivent être créées de manière à détecter et à retenter les connexions abandonnées et les transactions ayant échoué.  Quand l’application effectue une nouvelle tentative, les connexions sont dirigées vers le serveur de base de données nouvellement créé. | Si la panne du serveur de base de données est détectée, le serveur de base de données de secours est activé, ce qui réduit le temps d’arrêt. Pour plus d’informations, consultez la page [Concepts de haute disponibilité](concepts-high-availability.md). L’objectif de délai de récupération doit être de 60 à 120 s, avec un objectif de point de récupération = 0 |
| **Échec de stockage** | Les applications ne détectent aucun impact des problèmes liés au stockage, tels qu’une défaillance de disque ou une corruption de bloc physique. Les données étant stockées dans 3 copies, la copie des données est servie par le stockage survivant. Les altérations de bloc sont corrigées automatiquement. En cas de perte d’une copie des données, une nouvelle est automatiquement créée. | Pour les erreurs non récupérables, le serveur flexible est basculé sur le réplica de secours afin de réduire le temps d’arrêt. Pour plus d’informations, consultez la page [Concepts de haute disponibilité](./concepts-high-availability.md). |
| **Erreurs logiques/de l’utilisateur** | La récupération d’erreurs de l’utilisateur, telles qu’une suppression accidentelle de tables ou une mise à jour incorrecte de données, implique l’exécution d’une [récupération jusqu’à une date et heure](concepts-backup-restore.md) (PITR), en restaurant et récupérant les données jusqu’au moment où l’erreur s’est produite.<br> <br>  Si vous ne souhaitez restaurer qu’un sous-ensemble de bases de données ou de tables spécifiques plutôt que toutes les bases de données du serveur de base de données, vous pouvez restaurer celui-ci dans une nouvelle instance, exporter les tables via l’utilitaire [pg_dump](https://www.postgresql.org/docs/current/app-pgdump.html), puis vous servir de l’utilitaire [pg_restore](https://www.postgresql.org/docs/current/app-pgrestore.html) pour restaurer ces tables dans votre base de données. | Ces erreurs d’utilisateur ne sont pas protégées par la haute disponibilité en raison du fait que toutes les opérations des utilisateurs sont également répliquées sur le serveur de secours. |
| **Défaillance de zone de disponibilité** | Bien qu’il s’agisse d’un événement rare, si vous souhaitez procéder à une récupération après une défaillance au niveau d’une zone, vous pouvez lancer la récupération à un instant dans le passé à l’aide de la sauvegarde et tout en choisissant un point de restauration personnalisé pour accéder aux données les plus récentes. Un nouveau serveur flexible sera déployé dans une autre zone. Le temps que prend la restauration dépend de la sauvegarde précédente et du nombre des journaux des transactions à récupérer. | Le serveur flexible effectue le basculement automatique vers le site de secours. Pour plus d’informations, consultez la page [Concepts de haute disponibilité](./concepts-high-availability.md). |
| **Panne de région** | Les fonctionnalités de géorestauration et de réplica inter-région ne sont pas encore prises en charge dans la préversion. | |

> [!IMPORTANT]
> Il n’est **pas** possible de restaurer des serveurs supprimés. Si vous supprimez le serveur, toutes les bases de données qui appartiennent au serveur sont également supprimées, sans pouvoir être restaurées. Utilisez le [verrouillage des ressources Azure](../../azure-resource-manager/management/lock-resources.md) pour éviter la suppression accidentelle de votre serveur.

## <a name="next-steps"></a>Étapes suivantes

- Découvrir la [haute disponibilité avec redondance interzone](./concepts-high-availability.md)
- Découvrir [la sauvegarde et la récupération](./concepts-backup-restore.md)
