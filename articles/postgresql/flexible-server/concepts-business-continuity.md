---
title: Vue d’ensemble de la continuité d’activité dans Azure Database pour PostgreSQL – Serveur flexible
description: Découvrez les concepts de continuité d’activité dans Azure Database pour PostgreSQL – Serveur flexible
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: 341d3c6f0a2d84c2b7b725959db9f7e5dd80d3ff
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92542962"
---
# <a name="overview-of-business-continuity-with-azure-database-for-postgresql---flexible-server"></a>Vue d’ensemble de la continuité d’activité dans Azure Database pour PostgreSQL – Serveur flexible

> [!IMPORTANT]
> Azure Database pour PostgreSQL – Serveur flexible est en préversion

La **continuité de l’activité** dans Azure Database pour PostgreSQL fait référence aux mécanismes, stratégies et procédures qui permettent à votre entreprise de continuer à fonctionner en cas d’interruption affectant notamment son infrastructure informatique. Dans la plupart des cas, un serveur flexible gère les événements perturbateurs qui peuvent se produire dans l’environnement cloud, et préserve l’exécution de vos applications et processus métier. Toutefois, certains événements ne peuvent pas être gérés automatiquement, par exemple :

- Un utilisateur supprime ou met à jour accidentellement une ligne dans une table.
- Un séisme occasionne une coupure de courant et désactive temporairement un centre de donnes sur une zone de disponibilité.
- Mise à jour corrective de base de données requise pour corriger un bogue ou un problème de sécurité.

Le serveur flexible fournit des fonctionnalités qui protègent les données et atténuent les temps d’arrêt de vos bases de données stratégiques en cas de temps d’arrêt planifiés et non planifiés. S’appuyant sur l’infrastructure Azure qui offre déjà une résilience et une disponibilité robustes, un serveur flexible intègre des fonctionnalités de continuité d’activité qui offrent une protection supplémentaire contre les pannes, qui répondent aux exigences de temps de récupération et réduisent les risques de perte de données. À mesure que vous architecturez vos applications, vous devez prendre en compte la tolérance aux temps d’arrêt, qui est l’objectif de temps de récupération (RTO), et l’exposition à la perte de données, qui est l’objectif de point de récupération (RPO). Par exemple, votre base de données stratégique impose des exigences de temps d’activité sensiblement plus strictes qu’une base de données de test.  

> [!IMPORTANT]
> Aucun contrat de niveau de service (SLA) en lien avec la durée de bon fonctionnement % n’est pas proposé pendant la période de préversion. 

Le tableau ci-dessous illustre les fonctionnalités qu’offre un serveur flexible.


| **Fonctionnalité** | **Description** | **Considérations** |
| ---------- | ----------- | ------------ |
| **Sauvegardes automatiques** | Un serveur flexible effectue automatiquement des sauvegardes quotidiennes de vos fichiers de base de données et sauvegarde en permanence les journaux des transactions. Les sauvegardes peuvent être conservées de 7 à 35 jours. Vous pouvez restaurer votre serveur de base de données à n’importe quel point dans le temps au cours de la période de rétention de votre sauvegarde. Le RTO dépend de la taille des données à restaurer et la durée d’exécution de la récupération du journal. Il peut aller de quelques minutes à 12 heures. Pour plus d’informations, consultez [Concepts – Sauvegarde et restauration](./concepts-backup-restore.md). |Les données de sauvegarde restent dans la région. |
| **Haute disponibilité redondante interzone** | Un serveur flexible peut être déployé avec une configuration de haute disponibilité redondante interzone, où les serveurs primaire et de secours sont déployés dans deux zones de disponibilité différentes au sein d’une même région. Cette configuration de haute disponibilité protège vos bases de données contre les défaillances au niveau de la zone, et permet de réduire les temps d’arrêt d’applications pendant les temps d’arrêt planifiés et non planifiés. Les données du serveur primaire sont répliquées vers le serveur réplica de secours en mode synchrone. En cas d’interruption du serveur primaire, le serveur est automatiquement basculé vers le réplica de secours. Dans la plupart des cas, le RTO devrait être compris entre 60 et 120 secondes. Le RPO est supposé être égal à zéro (aucune perte de données). Pour plus d’informations, consultez [Concepts – Haute disponibilité](./concepts-high-availability.md). | Pris en charge dans les niveaux de calcul à usage général et à mémoire optimisée. Disponible uniquement dans les régions où plusieurs zones sont disponibles. |
| **Disques managés Premium** | Les fichiers de base de données sont stockés dans un stockage managé Premium durable et fiable. Cela assure une redondance des données avec trois copies du réplica stockées dans une zone de disponibilité avec des fonctionnalités de récupération de données automatiques. Pour plus d’informations, consultez la [Documentation sur la fonctionnalité Disques managés](../../virtual-machines/managed-disks-overview.md). | Données stockées dans une zone de disponibilité. |
| **Sauvegarde redondante interzone** | Les sauvegardes d’un serveur flexible sont stockées automatiquement et en toute sécurité dans un stockage redondant interzone au sein d’une région. Au cours d’une défaillance au niveau de la zone où votre serveur est approvisionné, si votre serveur n’est pas configuré avec une redondance interzone, vous pouvez toujours restaurer votre base de données à l’aide du dernier point de restauration dans une autre zone. Pour plus d’informations, consultez [Concepts – Sauvegarde et restauration](./concepts-backup-restore.md).| Applicable uniquement dans les régions où plusieurs zones sont disponibles.|


## <a name="planned-downtime-events"></a>Événements de temps d’arrêt planifiés
Voici quelques scénarios de maintenance planifiée. Ces événements occasionnent généralement quelques minutes de temps d’arrêt, sans perte de données.

| **Scénario** | **Processus**|
| ------------------- | ----------- | 
| <b>Mise à l’échelle du calcul (initiée par l’utilisateur)| Pendant l’opération de mise à l’échelle du calcul, les points de contrôle actifs sont autorisés à cesser d’opérer, les connexions clientes sont purgées, toutes les transactions non validées sont annulées, le stockage est détaché puis le serveur est arrêté. Un nouveau serveur flexible portant le même nom de serveur de base de données est approvisionné avec la configuration de calcul mise à l’échelle. Le stockage est ensuite attaché au nouveau serveur et la base de données démarrée, ce qui entraîne une récupération si nécessaire avant d’accepter les connexions clientes. |
| <b>Mise à l’échelle du stockage (initiée par l’utilisateur) | Quand une mise à l’échelle de opération de stockage est lancée, les points de contrôle actifs sont autorisés à cesser d’opérer, les connexions clientes sont purgées, toutes les transactions non validées sont annulées, puis le serveur est arrêté. Le stockage est mis à l’échelle à la taille souhaitée, puis attaché au nouveau serveur. Une récupération est effectuée si nécessaire avant d’accepter les connexions client. Notez qu’une réduction d’échelle de la taille de stockage n’est pas prise en charge. |
| <b>Déploiement de nouveaux logiciels (initiés par Azure) | Le déploiement de nouvelles fonctionnalités ou les corrections de bogues se produisent automatiquement dans le cadre de la maintenance planifiée du service, et vous pouvez planifier le moment auquel ces activités se produisent. Pour plus d’informations, consultez votre [portail](https://aka.ms/servicehealthpm). | 
| <b>Mises à niveau de version mineure (initiées par Azure) | Le service Azure Database pour PostgreSQL opère automatiquement la mise à niveau des serveurs de base de données vers la version mineure déterminée par Azure. Cela se produit dans le cadre de la maintenance planifiée du service. Le serveur de base de données est redémarré automatiquement avec la nouvelle version mineure. Pour plus d’informations, consultez la [documentation](../concepts-monitoring.md#planned-maintenance-notification). Vous pouvez également consulter votre [portail](https://aka.ms/servicehealthpm).| 

 Lorsque le serveur flexible est configuré avec une **haute disponibilité redondante interzone** , ils commence par effectuer les opérations de mise à l’échelle et de maintenance sur le serveur de secours. Pour plus d’informations, consultez [Concepts – Haute disponibilité](./concepts-high-availability.md).

##  <a name="unplanned-downtime-mitigation"></a>Réduction des temps d’arrêt non planifiés

Des temps d’arrêt non planifiés peuvent se produire suite à des interruptions imprévues, telles qu’une panne matérielle sous-jacente, des problèmes de mise en réseau et des bogues logiciels. Si le serveur de base de données configuré avec une haute disponibilité tombe en panne de façon inattendue, le serveur réplica de secours est activé et les clients peuvent reprendre leurs opérations. S’il n’est pas configuré avec une haute disponibilité (HA), si la tentative de redémarrage échoue, un nouveau serveur de base de données est automatiquement approvisionné. Bien qu’il ne soit pas possible d’éviter les temps d’arrêt non planifiés, un serveur flexible permet de réduire les temps d’arrêt en effectuant automatiquement des opérations de récupération sans nécessiter d’intervention humaine. 
   
### <a name="unplanned-downtime-failure-scenarios-and-service-recovery"></a>Temps d’arrêt non planifié : scénarios d’échec et récupération du service
Voici quelques scénarios d’échec non planifiés et le processus de récupération. 

| **Scénario** | **Processus de récupération [sans haute disponibilité]** | **Processus de récupération [haute disponibilité]** |
| ---------- | ---------- | ------- |
| <B>Échec du serveur de base de données | Si le serveur de base de données est à l’arrêt, Azure tente de le redémarrer. En cas d’échec, le serveur de base de données est redémarré sur un autre nœud physique.  <br /> <br /> Le temps de récupération (RTO) dépend de différents facteurs, dont l’activité au moment de la panne, telle qu’une transaction d’ampleur conséquence, et le volume de la récupération à effectuer pendant le processus de démarrage du serveur de base de données. <br /> <br /> Des applications utilisant les bases de données PostgreSQL doivent être créées de manière à détecter et à retenter les connexions abandonnées et les transactions ayant échoué. | Si la défaillance du serveur de base de données est détectée, le serveur est basculé sur le serveur de secours, ce qui réduit le temps d’arrêt. Pour plus d’informations, consultez la [page sur les concepts de haute disponibilité](./concepts-high-availability.md). Le RTO devrait être compris entre 60 et 120 secondes, sans perte de données. |
| <B>Échec de stockage | Les applications ne détectent aucun impact des problèmes liés au stockage, tels qu’une défaillance de disque ou une corruption de bloc physique. Les données étant stockées dans trois copies, leur copie est servie par le stockage survivant. Le bloc de données endommagé est réparé automatiquement, et une nouvelle copie des données est automatiquement créée. | Pour toute erreur rare et non récupérable, telle que l’inaccessibilité du stockage entier, le serveur flexible est basculé vers le serveur réplica de secours pour réduire le temps d’arrêt. Pour plus d’informations, consultez la [page sur les concepts de haute disponibilité](./concepts-high-availability.md). |
| <b> Erreurs logiques/de l’utilisateur | Pour récupérer d’erreurs d’utilisateur, telles que des tables supprimées accidentellement ou des données incorrectement mises à jour, vous devez effectuer une [Récupération jusqu’à une date et heure](../concepts-backup.md) (PITR). Lors de l’exécution de l’opération de restauration, vous spécifiez le point de restauration personnalisé, qui est l’heure juste avant la survenance de l’erreur.<br> <br>  Si vous ne souhaitez restaurer qu’un sous-ensemble de bases de données ou de tables spécifiques plutôt que toutes les bases de données du serveur de base de données, vous pouvez restaurer celui-ci dans une nouvelle instance, exporter les tables via l’utilitaire [pg_dump](https://www.postgresql.org/docs/11/app-pgdump.html), puis vous servir de l’utilitaire [pg_restore](https://www.postgresql.org/docs/11/app-pgrestore.html) pour restaurer ces tables dans votre base de données. | Ces erreurs utilisateur ne sont pas protégées avec une haute disponibilité, car toutes les modifications sont répliquées de façon synchrone sur le serveur réplica de secours. Pour récupérer de telles erreurs, vous devez effectuer une récupération jusqu`à une date et heure. |
| <b> Défaillance de zone de disponibilité | Pour récupérer d’une défaillance au niveau d’une zone, vous pouvez effectuer une restauration à un instant dans le passé à l’aide de la sauvegarde, en choisissant un point de restauration personnalisé avec la dernière heure pour restaurer les données les plus récentes. Un nouveau serveur flexible sera déployé dans une autre zone non affectée. Le temps que prend la restauration dépend de la sauvegarde précédente et du volume des journaux des transactions à récupérer. | Un serveur flexible est automatiquement basculé vers le serveur de secours dans un délai de 60 à 120 secondes sans perte de données. Pour plus d’informations, consultez la [page sur les concepts de haute disponibilité](./concepts-high-availability.md). | 
| <b> Défaillance de région | Les fonctionnalités de réplica en lecture inter-régions et de géo-restauration des sauvegardes ne sont pas encore prises en charge dans la préversion. | |


> [!IMPORTANT]
> Il n’est **pas** possible de restaurer des serveurs supprimés. Si vous supprimez le serveur, toutes les bases de données qui appartiennent au serveur sont également supprimées, sans pouvoir être restaurées. Utilisez le [verrouillage des ressources Azure](../../azure-resource-manager/management/lock-resources.md) pour éviter la suppression accidentelle de votre serveur.


## <a name="next-steps"></a>Étapes suivantes

-   Découvrir la [haute disponibilité avec redondance interzone](./concepts-high-availability.md)
-   Découvrir [la sauvegarde et la récupération](./concepts-backup-restore.md)