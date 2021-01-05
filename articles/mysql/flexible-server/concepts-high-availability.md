---
title: Vue d’ensemble de la haute disponibilité redondante interzone avec le serveur flexible Azure Database pour MySQL
description: Découvrir les concepts de la haute disponibilité redondante interzone avec le serveur flexible Azure Database pour MySQL
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: conceptual
ms.date: 09/21/2020
ms.openlocfilehash: cd7be998c49a710ee7652cf18c35bed743606ffd
ms.sourcegitcommit: 80034a1819072f45c1772940953fef06d92fefc8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93241182"
---
# <a name="high-availability-concepts-in-azure-database-for-mysql-flexible-server-preview"></a>Concepts de haute disponibilité dans le serveur flexible Azure Database pour MySQL (préversion)

> [!IMPORTANT] 
> Azure Database pour MySQL - Serveur flexible est actuellement en préversion publique.

Le serveur flexible Azure Database pour MySQL (préversion) permet de configurer la haute disponibilité avec basculement automatique à l’aide de l’option de haute disponibilité **redondante interzone**. Déployé dans une configuration redondante interzone, le serveur flexible provisionne et gère automatiquement un réplica de secours dans une autre zone de disponibilité. En utilisant la réplication au niveau du stockage, les données sont **répliquées de manière synchrone** sur le serveur de secours dans la zone secondaire, afin d’éviter toute perte de données après un basculement. Le basculement est entièrement transparent depuis l’application cliente et ne nécessite aucune action de l’utilisateur. Le serveur de secours n’est pas disponible pour les opérations de lecture ou d’écriture, il s’agit en fait d’un serveur passif qui permet de basculer rapidement. Les temps de basculement sont généralement compris entre 60 et 120 secondes.

La configuration de la haute disponibilité redondante interzone permet le basculement automatique lors d’événements planifiés, tels que les opérations de mise à l’échelle du calcul initiées par l’utilisateur, et d’événements non planifiés, comme les pannes matérielles et logicielles sous-jacentes, les problèmes réseau et même les pannes de zone de disponibilité.

:::image type="content" source="./media/concepts-high-availability/1-flexible-server-overview-zone-redundant-ha.png" alt-text="Affichage de la haute disponibilité redondante interzone":::

## <a name="zone-redundancy-architecture"></a>Architecture de la redondance interzone

Le serveur principal est déployé dans la région et dans une zone de disponibilité spécifique. Quand la haute disponibilité est choisie, un serveur réplica de secours doté de la même configuration que celle du serveur principal est déployé automatiquement, avec le niveau de calcul, la taille de calcul, la taille de stockage et la configuration réseau. Les données des journaux sont répliquées de façon synchrone sur le réplica de secours afin de garantir aucune perte de données en cas de panne. Les sauvegardes automatiques, qu’il s’agisse d’instantanés ou de sauvegardes des fichiers journaux, sont effectuées depuis le serveur de base de données primaire. 

L’intégrité de la haute disponibilité est supervisée et signalée en permanence dans la page Vue d’ensemble.

Les différents états de réplication sont listés ci-dessous :

| **État** | **Description** |
| :----- | :------ |
| <b>NotEnabled | La haute disponibilité redondante interzone n’est pas activée |
| <b>CreatingStandby | Création d’un serveur de secours en cours |
| <b>ReplicatingData | Une fois créé, le réplica de secours rattrape le serveur principal. |
| <b>FailingOver | Le serveur de base de données est en cours de basculement, du serveur principal vers le réplica de secours. |
| <b>Healthy | La haute disponibilité redondante interzone présente un état stable et sain. |
| <b>RemovingStandby | Selon l’action de l’utilisateur, le réplica de secours est en cours de suppression.| 

## <a name="advantages"></a>Avantages

Voici quelques avantages à utiliser la fonctionnalité de haute disponibilité avec redondance interzone : 

-   Le réplica de secours sera déployé dans une configuration de machine virtuelle à l’identique de celle du serveur principal, par exemple les vCores, le stockage, les paramètres réseau (réseau virtuel, pare-feu), etc.
-   Possibilité de supprimer le réplica de secours en désactivant la haute disponibilité.
-   Les sauvegardes automatiques se font par instantanés ; elles s’effectuent depuis le serveur de base de données primaire et sont stockées dans un stockage redondant interzone.
-   Si un événement de basculement se produit, un nouveau réplica de secours est provisionné dans la zone de disponibilité principale d’origine.
-   Les clients se connectent toujours au serveur de base de données primaire.
-   En cas de plantage d’une base de données ou de défaillance d’un nœud, le redémarrage sera tenté en premier sur le même nœud. En cas d’échec, le basculement automatique est déclenché.
-   Possibilité de redémarrer le serveur pour intégrer toute modification de paramètre du serveur statique.

## <a name="steady-state-operations"></a>Opérations à l’état stable

Les applications sont connectées au serveur principal à l’aide du nom du serveur de base de données. Les informations du réplica de secours ne sont pas exposées à un accès direct. Les validations et les écritures ne sont reconnues dans l’application qu’une fois les fichiers journaux conservés sur le disque du serveur principal et sur le réplica de secours de manière synchrone. En raison de cet aller-retour obligatoire supplémentaire, les applications peuvent s’attendre à une latence élevée pour les écritures et les validations. Vous pouvez superviser l’intégrité de la haute disponibilité sur le portail.

## <a name="failover-process"></a>Processus de basculement 
Afin de préserver la continuité des activités, vous devez disposer d’un processus de basculement pour les événements planifiés et non planifiés. 

### <a name="planned-events"></a>Événements planifiés

Les événements de temps d’arrêt planifiés comprennent les activités planifiées par Azure (mises à jour logicielles périodiques, mises à niveau de versions mineures) ou lancées par les clients, comme les opérations de mise à l’échelle du calcul et du stockage. Toutes ces modifications sont d’abord appliquées au réplica de secours. Pendant ce temps, les applications continuent d’accéder au serveur principal. Une fois le réplica de secours mis à jour, les connexions au serveur principal sont purgées et un basculement est déclenché : le réplica de secours est activé en tant que serveur principal prenant le même nom de serveur de base de données par l’effet de la mise à jour de l’enregistrement DNS. Les connexions clientes sont déconnectées ; elles doivent se reconnecter et peuvent ensuite reprendre leurs opérations. Un nouveau serveur de secours est établi dans la même zone que l’ancien serveur principal. Le basculement global doit se situer entre 60 et 120 s. 

>[!NOTE]
> En cas d’opération de mise à l’échelle du calcul, nous mettons à l’échelle le serveur réplica secondaire, puis le serveur principal. Aucun basculement n’est impliqué.

### <a name="failover-process---unplanned-events"></a>Processus de basculement – événements non planifiés
Les interruptions de service non planifiées comprennent les bogues logiciels, ou les erreurs d’infrastructure telles que les défaillances de calcul, de réseau, de stockage, ou encore les pannes de courant ; elles ont un impact sur la disponibilité de la base de données. En cas d’indisponibilité de la base de données, la réplication sur le réplica de secours est interrompue, et le réplica de secours est activé pour devenir la base de données primaire. Le système DNS est mis à jour et les clients se reconnectent ensuite au serveur de base de données pour reprendre leurs opérations. Le temps de basculement global doit prendre entre 60 et 120 s. Toutefois, en fonction de l’activité sur le serveur de base de données primaire au moment du basculement, par exemple si des transactions sont volumineuses et que le temps de récupération est long, le basculement peut prendre plus de temps.

## <a name="schedule-maintenance-window"></a>Fenêtre Planifier la maintenance 

Les serveurs flexibles offrent une fonctionnalité de planification de la maintenance, où vous pouvez choisir un créneau de 30 minutes sur un jour de votre choix, au cours duquel la maintenance Azure s’effectue, comme une mise à jour de correctifs ou une mise à niveau de versions mineures se produirait. Pour vos serveurs flexibles configurés avec une haute disponibilité, ces activités de maintenance sont d’abord effectuées sur le réplica de secours. 

## <a name="point-in-time-restore"></a>Restauration dans le temps 
Comme le serveur flexible est configuré en haute disponibilité et qu’il réplique les données de façon synchrone, le serveur de secours est à jour par rapport au serveur principal. Toute erreur utilisateur sur le serveur principal (par exemple, la suppression accidentelle d’une table ou des mises à jour incorrectes) est répliquée fidèlement sur le serveur de secours. Par conséquent, vous ne pouvez pas utiliser le serveur de secours pour récupérer à partir de ces erreurs logiques. Pour récupérer à partir de ces erreurs logiques, vous devez procéder à une restauration à un instant dans le passé, juste avant que l’erreur ne se produise. Lorsque, à l’aide de la fonctionnalité de restauration à un instant dans le passé du serveur flexible, vous restaurez la base de données configurée avec une haute disponibilité, un nouveau serveur de base de données est restauré en tant que nouveau serveur flexible, avec le nom fourni par l’utilisateur. Vous pouvez ensuite exporter l’objet à partir du serveur de base de données, puis l’importer sur votre serveur de base de données de production. De même, si vous souhaitez cloner votre serveur de base de données à des fins de test et de développement, ou que vous voulez effectuer une restauration à d’autres fins, vous pouvez choisir le point de restauration le plus récent ou un point de restauration personnalisé. L’opération de restauration crée un serveur flexible dans une seule zone.

## <a name="mitigate-downtime"></a>Réduire les temps d’arrêt 
Lorsque vous n’utilisez pas la fonctionnalité de haute disponibilité redondante interzone, vous devez malgré tout encore être en mesure de réduire les temps d’arrêt de votre application. La planification d’interruptions de service (par exemple, les mises à jour correctives prévues, les mises à niveau de version mineure ou les opérations initiées par l’utilisateur) peut être effectuée dans le cadre de fenêtres de maintenance planifiée. Les interruptions de service planifiées, telles que les mises à jour correctives et mises à niveau de version mineure prévues, ou les opérations lancées par l’utilisateur comme la mise à l’échelle du calcul, subissent un temps d’arrêt. Pour limiter l’impact des tâches de maintenance lancées par Azure sur une application, vous pouvez choisir de planifier ces tâches au jour de la semaine et à l’heure qui ont le moins d’impact sur l’application. 

Au cours d’événements d’interruption non planifiés, tels qu’un plantage de base de données ou une panne du serveur, le serveur concerné est redémarré dans la même zone. Bien que le cas soit rare, si la zone entière est affectée, vous pouvez restaurer la base de données sur une autre zone de la région. 

## <a name="things-to-know-with-zone-redundancy"></a>Informations à connaître sur la redondance de zone 

Voici quelques réflexions à prendre en compte lorsque vous utilisez la haute disponibilité redondante interzone : 

-   La haute disponibilité peut **uniquement** être définie lors de la création du serveur flexible.
-   La haute disponibilité n’est pas prise en charge au niveau du calcul expansible.
-   En raison de la réplication synchrone vers une autre zone de disponibilité, le serveur de base de données primaire peut rencontrer une latence d’écriture et de validation élevée.
-   Un réplica de secours ne peut pas être utilisé pour des requêtes en lecture seule.
-   En fonction de l’activité sur le serveur principal au moment du basculement, l’exécution du basculement peut prendre entre 60 et 120 secondes, voire plus.
-   Un redémarrage du serveur de base de données primaire pour récupérer les modifications de paramètres statiques a également pour effet de redémarrer le serveur réplica de secours.
-   La configuration de réplicas en lecture pour les serveurs à haute disponibilité redondante interzone n’est pas prise en charge.
-   La configuration des tâches de gestion lancées par le client ne peut pas être automatisée pendant la fenêtre de maintenance gérée.
-   Les événements planifiés, tels que la mise à l’échelle du calcul et les mises à niveau de version mineure, se produisent en même temps sur le serveur principal et le réplica de secours. 


## <a name="next-steps"></a>Étapes suivantes

-   Découvrir la [continuité de l’activité](./concepts-business-continuity.md)
-   Découvrir la [haute disponibilité avec redondance interzone](./concepts-high-availability.md)
-   Découvrir [la sauvegarde et la récupération](./concepts-backup-restore.md)
