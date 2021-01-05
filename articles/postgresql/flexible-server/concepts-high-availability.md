---
title: Vue d’ensemble de la haute disponibilité redondante interzone dans Azure Database pour PostgreSQL – Serveur flexible (préversion)
description: Apprenez-en davantage sur les concepts de haute disponibilité redondante interzone dans Azure Database pour PostgreSQL – Serveur flexible
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: c0d9b6042ae695caa73d926653f237b756bf4971
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94366721"
---
# <a name="high-availability-concepts-in-azure-database-for-postgresql---flexible-server"></a>Concepts de haute disponibilité dans Azure Database pour PostgreSQL – Serveur flexible

> [!IMPORTANT]
> Azure Database pour PostgreSQL – Serveur flexible est en préversion

Azure Database pour PostgreSQL – Serveur flexible offre une configuration de haute disponibilité avec fonctionnalité de basculement automatique utilisant un déploiement de serveur **redondant interzone**. Déployé dans une configuration redondante interzone, le serveur flexible provisionne et gère automatiquement un réplica de secours dans une autre zone de disponibilité. À l’aide d’une réplication en streaming PostgreSQL, les données sont répliquées sur le serveur réplica de secours en mode **synchrone**. 

La configuration redondante interzone active une fonctionnalité de basculement automatique sans perte de données pendant des événements planifiés, tels qu’une opération de mise à l’échelle du calcul lancée par l’utilisateur, ainsi que pendant des événements non planifiés, tels que des erreurs matérielles et logicielles sous-jacentes, des échecs réseau et des défaillances de zone de disponibilité. 

:::image type="content" source="./media/business-continuity/concepts-zone-redundant-high-availability-architecture.png" alt-text="Haute disponibilité redondante interzone"::: 

## <a name="zone-redundant-high-availability-architecture"></a>Architecture de haute disponibilité redondante interzone

Vous pouvez choisir la région et la zone de disponibilité pour déployer votre serveur de base de données primaire. Un serveur réplica de secours est approvisionné dans une zone de disponibilité différente avec la même configuration que le serveur primaire, à savoir le niveau de calcul, la taille de calcul, la taille de stockage et la configuration réseau. Les journaux des transactions sont répliqués en mode synchrone sur le réplica de secours à l’aide d’une réplication en streaming PostgreSQL. Des sauvegardes automatiques sont effectuées régulièrement à partir du serveur de base de données primaire, tandis que les journaux de transactions sont archivés en permanence dans le stockage de sauvegarde à partir du réplica de secours. 

L’intégrité de la configuration de haute disponibilité est surveillée et signalée en continu sur le portail. Les états de haute disponibilité redondante interzone sont répertoriés ci-dessous :

| **État** | **Description** |
| ------- | ------ |
| <b> Initialisation | Dans le processus de création d’un serveur de secours |
| <b> Réplication des données | Une fois le serveur de secours créé, il rattrape le serveur principal. |
| <b> Intègre | La réplication est dans un état stable et intègre. |
| <b> Basculement | Le serveur de base de données est en cours de basculement vers le serveur de secours. |
| <b> Suppression du serveur de secours | Lors du processus de suppression du serveur de secours. | 
| <b> Non activée | La haute disponibilité redondante interzone n’est pas activée.  |

## <a name="steady-state-operations"></a>Opérations à l’état stable

Les applications clientes PostgreSQL sont connectées au serveur primaire à l’aide du nom du serveur BDD. Les lectures d’application sont servies directement à partir du serveur primaire, tandis que les validations et les écritures sont confirmées à l’application uniquement une fois les données conservées tant sur le serveur primaire que sur le réplica de secours. En raison de cette exigence d’aller-retour supplémentaire, les applications peuvent s’attendre à une latence élevée pour les écritures et les validations. Vous pouvez surveiller l’intégrité de la haute disponibilité sur le portail.

:::image type="content" source="./media/business-continuity/concepts-high-availability-steady-state.png" alt-text="Haute disponibilité redondante interzone – État stable"::: 

1. Les clients se connectent au serveur flexible et effectuent des opérations d’écriture.
2. Les modifications sont répliquées sur le site de secours.
3. Le serveur primaire reçoit un accusé de réception.
4. Les écritures/validations font l’objet d’un accusé de réception.

## <a name="failover-process---planned-downtimes"></a>Processus de basculement – Temps d’arrêt planifiés

Les événements de temps d’arrêt planifiés incluent des mises à jour logicielles périodiques planifiées Azure et des mises à niveau de version mineure. Configurées en mode haute disponibilité, ces opérations sont d’abord appliquées au réplica de secours, tandis que les applications continuent d’accéder au serveur primaire. Une fois le réplica de secours mis à jour, les connexions au serveur primaire sont purgées et un basculement est déclenché, qui a pour effet d’activer le serveur réplica de secours en tant que serveur principal portant le même nom de serveur de base de données. Les applications clientes doivent se reconnecter avec le même nom de serveur de base de données au nouveau serveur primaire, et peuvent reprendre leurs opérations. Un nouveau serveur de secours est établi dans la même zone que l’ancien serveur primaire. 

Pour les autres opérations initiées par l’utilisateur, telles que la mise à l’échelle du calcul ou du stockage, les modifications sont appliquées d’abord au serveur réplica de secours, puis au serveur primaire. Actuellement, les connexions ne sont pas basculées vers le serveur réplica de secours, et entraînent donc un temps d’arrêt pendant que l’opération est effectuée sur le serveur primaire.

### <a name="reducing-planned-downtime-with-managed-maintenance-window"></a>Réduction des temps d’arrêt planifiés avec une fenêtre de maintenance gérée

 Vous pouvez planifier des activités de maintenance lancées par Azure en choisissant une fenêtre de 30 minutes dans le jour de votre choix, où les activités sur les bases de données devraient être faibles. Les tâches de maintenance Azure, telles que les mises à jour correctives ou les mises à niveau de version mineure, seront effectuées dans cette fenêtre.  Pour des serveurs flexibles configurés avec une haute disponibilité, ces activités de maintenance sont effectuées sur le serveur réplica de secours, puis activées. Les applications se reconnectent ensuite au nouveau serveur primaire et reprennent leurs opérations quand un nouveau serveur secours est approvisionné.

## <a name="failover-process---unplanned-downtimes"></a>Processus de basculement – Temps d’arrêt non planifiés

Les interruptions non planifiées incluent des bogues logiciels ou des défaillances de composants d’infrastructure ayant un impact sur la disponibilité de la base de données. En cas d’indisponibilité du serveur détectée par le système de surveillance, la réplication vers le serveur réplica de secours est interrompue, et le serveur réplica de secours est activé pour en faire le serveur de base de données primaire. Les clients peuvent se reconnecter au serveur de base de données à l’aide de la même chaîne de connexion et reprendre leurs opérations. Le basculement global est censé prendre entre 60 et 120 secondes. Toutefois, en fonction de l’activité sur le serveur de base de données primaire au moment du basculement, par exemple, si les transactions et temps de récupération sont conséquents, le basculement peut prendre plus de temps.

:::image type="content" source="./media/business-continuity/concepts-high-availability-failover-state.png" alt-text="Haute disponibilité redondante interzone – Basculement"::: 

1. Le serveur de base de données primaire est à l’arrêt et les clients perdent la connexion à la base de données. 
2. Le serveur de secours est activé pour devenir le nouveau serveur primaire. Le client se connecte au nouveau serveur primaire en utilisant la même chaîne de connexion. Le fait que l’application cliente se trouve dans la même zone que le serveur de base de données primaire réduit la latence et améliore les performances.
3. Le serveur de secours est établi dans la même zone que l’ancien serveur primaire, et la réplication en streaming est lancée. 
4. Une fois la réplication à l’état stable, les validations et les écritures de l’application cliente font l’objet d’un accusé de réception quand les données sont conservées sur les deux sites.

## <a name="point-in-time-restore"></a>Restauration dans le temps 

Des serveurs flexibles configurés avec une haute disponibilité répliquent les données en temps réel sur le serveur de secours afin de tenir celui-ci à jour. Toute erreur d’utilisateur sur le serveur primaire, par exemple, une suppression accidentelle de table ou une mise à jour de données incorrecte, est répliquées fidèlement sur le serveur réplica de secours. Par conséquent, vous ne pouvez pas utiliser le serveur de secours pour récupérer de telles erreurs logiques. Pour récupérer de telles erreurs, vous devez effectuer une restauration à un instant dans le passé à partir de sauvegardes.  La fonctionnalité de restauration à un instant dans le passé du serveur flexible vous permet d’opérer une récupération jusqu`à une date et heure antérieures à la survenance de l’erreur. Pour les bases de données configurées avec une haute disponibilité, un nouveau serveur de base de données est restauré en tant que serveur flexible dans une zone unique avec un nom fourni par l’utilisateur. Vous pouvez ensuite exporter l’objet à partir du serveur de base de données, puis l’importer sur votre serveur de base de données de production. De même, si vous souhaitez cloner votre serveur de base de données à des fins de test et de développement, ou si vous souhaitez effectuer une restauration pour toute autre raison, vous pouvez opérer des récupérations jusqu`à une date et heure.

## <a name="zone-redundant-high-availability---features"></a>Haute disponibilité redondante interzone – Fonctionnalités

-   Le réplica de secours sera déployé dans une configuration de machine virtuelle exacte identique à celle du serveur primaire, incluant les vCores, le stockage, les paramètres réseau (VNET, pare-feu), etc.

-   Possibilité d’ajouter une haute disponibilité pour un serveur de base de données existant.

-   Possibilité de supprimer le réplica de secours en désactivant la haute disponibilité.

-   Possibilité de choisir votre zone de disponibilité pour votre serveur de base de données primaire.

-   Possibilité d’arrêter, de démarrer et de redémarrer les serveurs de base de données principal et de secours.

-   Des sauvegardes automatiques sont effectuées à partir du serveur de base de données primaire, et stockées dans un stockage redondant interzone.

-   Les clients se connectent toujours au serveur de base de données primaire.

-   Possibilité de redémarrer le serveur pour intégrer toute modification de paramètre du serveur statique.
  
-   Les activités de maintenance périodiques telles que les mises à niveau de version mineure se déroulent sur le serveur de secours, puis le service est basculé pour réduire le temps d’arrêt.  

## <a name="zone-redundant-high-availability---limitations"></a>Haute disponibilité redondante interzone – Limitations

-   La haute disponibilité n’est pas prise en charge avec le niveau de calcul par rafale.
-   La haute disponibilité est prise en charge uniquement dans les régions où plusieurs zones sont disponibles.
-   En raison de la réplication synchrone vers une autre zone de disponibilité, les applications peuvent rencontrer une latence d’écriture et de validation élevée.

-   Un réplica de secours ne peut pas être utilisé pour des requêtes en lecture.

-   En fonction de la charge de travail et de l’activité sur le serveur principal, le processus de basculement peut prendre plus de 120 secondes.

-   Le redémarrage du serveur de base de données primaire redémarre également le réplica de secours. 

-   La configuration de réplicas en lecture supplémentaires n’est pas prise en charge.

-   La configuration des tâches de gestion lancées par le client ne peut pas être planifiée pendant la fenêtre de maintenance gérée.

-   Des événements planifiés, tels que la mise à l’échelle du calcul et du stockage, se produisent d’abord sur le serveur de secours, puis sur le serveur primaire. Le serveur n’a pas basculé pour ces opérations planifiées. 

-  Si le décodage logique ou la réplication logique sont configurés avec un serveur flexible configuré pour la haute disponibilité, en cas de basculement vers le serveur de secours, les emplacements de réplication logique ne sont pas copiés sur le serveur de secours.  

## <a name="next-steps"></a>Étapes suivantes

-   Découvrir la [continuité de l’activité](./concepts-business-continuity.md)
-   Découvrez comment [gérer la haute disponibilité](./how-to-manage-high-availability-portal.md)
-   Découvrir [la sauvegarde et la récupération](./concepts-backup-restore.md)