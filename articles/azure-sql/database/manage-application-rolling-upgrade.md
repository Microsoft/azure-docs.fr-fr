---
title: Mises à niveau d’application propagées
description: Découvrez comment utiliser la géoréplication Azure SQL Database pour prendre en charge les mises à niveau propagées de votre application cloud.
services: sql-database
ms.service: sql-database
ms.subservice: high-availability
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: anosov1960
ms.author: sashan
ms.reviewer: mathoma, sstein
ms.date: 02/13/2019
ms.openlocfilehash: 659a8a3b38a79cc9dcc97f6f1e9c4395426ef7a8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91450266"
---
# <a name="manage-rolling-upgrades-of-cloud-applications-by-using-sql-database-active-geo-replication"></a>Gérer les mises à niveau propagées des applications cloud à l’aide de la géoréplication active Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Découvrez comment utiliser la [géoréplication active](auto-failover-group-overview.md) dans Azure SQL Database pour autoriser les mises à niveau propagées de votre application cloud. Les mises à niveau étant des opérations qui entraînent une interruption de service, elles doivent être intégrées à votre conception et à votre planification de continuité des activités. Dans cet article, nous allons examiner deux méthodes différentes permettant d’orchestrer le processus de mise à niveau propagée, avant de présenter les avantages et inconvénients de chaque option. Dans le cadre de cet article, nous allons utiliser une application sous la forme d’un site web utilisant une base de données unique comme couche de données. Notre objectif est de mettre à niveau la version 1 (V1) de l’application vers la version 2 (V2) sans que cela ait une incidence significative sur l’expérience des utilisateurs.

Lors de l’évaluation des options de mise à niveau, tenez compte des facteurs suivants :

* Impact sur la disponibilité de l’application pendant les mises à niveau, par exemple la durée pendant laquelle les fonctions de l’application risquent d’être limitées ou détériorées
* Possibilité de restauration en cas d’échec de la mise à niveau
* Vulnérabilité de l’application dans l’éventualité où un sinistre se produirait indépendamment pendant la mise à niveau
* Coût total du processus, notamment les coûts supplémentaires de redondance de base de données et les coûts incrémentiels des composants temporaires utilisés par le processus de mise à niveau

## <a name="upgrade-applications-that-rely-on-database-backups-for-disaster-recovery"></a>Mettre à niveau des applications dont la reprise d’activité après sinistre repose sur des sauvegardes de base de données

Si votre application s’appuie sur des sauvegardes automatiques de la base de données et utilise la géorestauration pour la reprise d’activité, elle est déployée dans une seule région Azure. Afin de minimiser l’interruption pour l’utilisateur, créez un environnement de préproduction dans cette région avec tous les composants d’application impliqués dans la mise à niveau. Le premier schéma illustre l’environnement d’exploitation avant la mise à niveau. Le point de terminaison `contoso.azurewebsites.net` représente un environnement de production de l’application web. Pour pouvoir annuler la mise à niveau, vous devez créer un environnement de préproduction avec une copie entièrement synchronisée de la base de données. Suivez ces étapes pour créer un environnement de préproduction pour la mise à niveau :

1. Créez une base de données secondaire dans la même région Azure. Surveillez la base de données secondaire afin de déterminer si le processus d’amorçage est terminé (1).
2. Créez un nouvel environnement pour votre application web et nommez-le « Staging ». Il sera inscrit dans Azure DNS avec l’URL `contoso-staging.azurewebsites.net` (2).

> [!NOTE]
> Ces étapes de préparation n’affecteront pas l’environnement de production, qui peut fonctionner en mode d’accès complet.

![Diagramme montrant la configuration de la géoréplication de SQL Database pour la récupération d’urgence cloud.](./media/manage-application-rolling-upgrade/option1-1.png)

Une fois les étapes de préparation terminées, l’application est prête pour la mise à niveau. Le schéma suivant illustre les étapes impliquées dans le processus de mise à niveau :

1. Définissez la base de données primaire sur le mode lecture seule (3). Ce mode garantit que l’environnement de production de l’application web (V1) reste en lecture seule au cours de la mise à niveau, ce qui évite toute divergence des données entre les instances de base de données V1 et V2.
2. Déconnectez la base de données secondaire à l’aide du mode d’arrêt planifié (4). Cette action crée une copie indépendante entièrement synchronisée de la base de données primaire. Cette base de données est alors mise à niveau.
3. Configurez la base de données secondaire en mode lecture-écriture et exécutez le script de mise à niveau (5).

![Diagramme montrant la configuration de la géoréplication de SQL Database pour la récupération d’urgence cloud qui exécute le script de mise à niveau.](./media/manage-application-rolling-upgrade/option1-2.png)

Si la mise à niveau se termine correctement, vous êtes maintenant prêt à faire basculer les utilisateurs vers la copie mise à niveau de l’application, qui devient un environnement de production. Ce basculement implique quelques étapes supplémentaires, comme l’illustre le schéma suivant :

1. Effectuez une opération de permutation entre les environnements de production et préproduction de l’application web (6). Cette opération permute les URL des deux environnements. `contoso.azurewebsites.net` pointe maintenant vers la version V2 du site web et de la base de données (environnement de production). 
2. Si vous n’avez plus besoin de la version V1, qui est devenue une copie de préproduction après la permutation, vous pouvez mettre l’environnement de préproduction hors service (7).

![Configuration de la géoréplication SQL Database pour la reprise d’activité cloud.](./media/manage-application-rolling-upgrade/option1-3.png)

Si la mise à niveau échoue, par exemple en raison d’une erreur dans le script de mise à niveau, considérez l’environnement de préproduction comme compromis. Pour restaurer l’application telle qu’elle se trouvait avant la mise à niveau, restaurez l’accès complet à l’application dans l’environnement de production. Le schéma suivant montre les étapes de restauration :

1. Définition de la copie de base de données en mode lecture-écriture (8). Cette action restaure toutes les fonctionnalités V1 de la copie de production.
2. Effectuez l’analyse de la cause racine et mettez l’environnement de préproduction hors service (9).

À ce stade, l’application est entièrement fonctionnelle et vous pouvez répéter les étapes de mise à niveau.

> [!NOTE]
> La restauration ne nécessite pas de modifications DNS, car vous n’aviez pas encore effectué d’opération de permutation.

![Diagramme montrant la configuration de la géoréplication de SQL Database pour la récupération d’urgence cloud avec l’environnement intermédiaire désaffecté.](./media/manage-application-rolling-upgrade/option1-4.png)

Le principal avantage de cette option est qu’elle vous permet de mettre à niveau une application dans une seule région en effectuant une série d’étapes simples. Le coût de la mise à niveau est relativement faible. 

L’inconvénient est que, si une défaillance irrémédiable se produit pendant la mise à niveau, vous devez redéployer l’application dans une autre région et restaurer la base de données à partir de la sauvegarde à l’aide de la géorestauration pour pouvoir rétablir l’application telle qu’elle se trouvait avant la mise à niveau. Ce processus entraîne des interruptions de service importantes.

## <a name="upgrade-applications-that-rely-on-database-geo-replication-for-disaster-recovery"></a>Mettre à niveau des applications dont la reprise d’activité repose sur la géoréplication de la base de données

Si votre application utilise la géoréplication active ou des groupes de basculement automatique pour la continuité de l’activité, elle est déployée dans au moins deux régions différentes. Il existe une base de données active primaire dans une région primaire, et une base de données secondaire en lecture seule dans une région de sauvegarde. Outre les facteurs mentionnés au début de cet article, le processus de mise à niveau doit également garantir que :

* L’application demeure constamment à l’abri des sinistres pendant le processus de mise à niveau.
* Les composants géoredondants de l’application sont mis à niveau parallèlement aux composants actifs.

Pour atteindre ces objectifs, en plus d’utiliser les environnements Web Apps, vous tirerez parti d’Azure Traffic Manager en faisant appel à un profil de basculement avec un point de terminaison actif et un point de terminaison de sauvegarde. Le schéma suivant illustre l’environnement d’exploitation avant la mise à niveau. Les sites web `contoso-1.azurewebsites.net` et `contoso-dr.azurewebsites.net` représentent un environnement de production de l’application avec redondance géographique complète. L’environnement de production comprend les composants suivants :

* L’environnement de production de l’application web `contoso-1.azurewebsites.net` dans la région primaire (1)
* La base de données primaire dans la région primaire (2)
* Une instance de secours de l’application web dans la région de sauvegarde (3)
* La base de données secondaire géorépliquée dans la région de sauvegarde (4)
* Un profil de performance Traffic Manager avec un point de terminaison en ligne nommé `contoso-1.azurewebsites.net` et un point de terminaison hors connexion nommé `contoso-dr.azurewebsites.net`

Pour permettre l’annulation de la mise à niveau, vous devez créer un environnement de préproduction avec une copie entièrement synchronisée de l’application. Pour avoir la garantie que l’application sera capable de récupérer rapidement en cas de défaillance irrémédiable pendant le processus de mise à niveau, l’environnement de préproduction doit également être géoredondant. Les étapes suivantes sont nécessaires pour créer un environnement intermédiaire pour la mise à niveau :

1. Déployez un environnement de préproduction de l’application web dans la région primaire (6).
2. Créez une base de données secondaire dans la même Azure primaire (7). Configurez l’environnement de préproduction de l’application web pour vous y connecter. 
3. Créez une autre base de données secondaire géoredondante dans la région de sauvegarde en répliquant la base de données secondaire dans la région primaire. (Cette méthode porte le nom de *géoréplication chaînée*.) (8)
4. Déployez un environnement de préproduction de l’instance d’application web dans la région de sauvegarde (9) et configurez-le pour connecter la base de données secondaire géoredondante créée à l’étape (8).

> [!NOTE]
> Ces étapes de préparation n’affecteront pas l’application dans l’environnement de production. Elle restera totalement fonctionnelle en mode lecture-écriture.

![Diagramme montrant la configuration de la géoréplication de SQL Database pour la récupération d’urgence cloud avec une copie entièrement synchronisée de l’application.](./media/manage-application-rolling-upgrade/option2-1.png)

Une fois les étapes de préparation terminées, l’environnement de préproduction est prêt pour la mise à niveau. Le schéma suivant illustre ces étapes de la mise à niveau :

1. Configurez la base de données primaire dans l’environnement de production en mode lecture seule (10). Ce mode garantit que la base de données de production (V1) ne changera pas pendant la mise à niveau, évitant ainsi toute divergence des données entre les instances de base de données V1 et V2.

```sql
-- Set the production database to read-only mode
ALTER DATABASE <Prod_DB>
SET (ALLOW_CONNECTIONS = NO)
```

2. Arrêtez la géoréplication en déconnectant la base de données secondaire (11). Cette action crée une copie indépendante mais entièrement synchronisée de la base de données de production. Cette base de données est alors mise à niveau. L’exemple suivant utilise Transact-SQL mais [PowerShell](/powershell/module/az.sql/remove-azsqldatabasesecondary?view=azps-1.5.0&preserve-view=true) est également disponible. 

```sql
-- Disconnect the secondary, terminating geo-replication
ALTER DATABASE <Prod_DB>
REMOVE SECONDARY ON SERVER <Partner-Server>
```

3. Exécutez le script de mise à niveau sur `contoso-1-staging.azurewebsites.net`, `contoso-dr-staging.azurewebsites.net` et la base de données primaire de préproduction (12). Les modifications apportées à la base de données sont répliquées automatiquement sur la secondaire de préproduction.

![Diagramme montrant la configuration de la géoréplication de SQL Database pour la récupération d’urgence cloud avec les modifications de base de données répliquées pour l’environnement intermédiaire.](./media/manage-application-rolling-upgrade/option2-2.png)

Si la mise à niveau s’est correctement déroulée, vous êtes maintenant prêt à basculer les utilisateurs sur la version V2 de l’application. Le schéma suivant illustre les étapes impliquées dans ce processus :

1. Effectuez une opération de permutation entre les environnements de production et de préproduction de l’application web dans la région primaire (13) et dans la région de sauvegarde (14). La version V2 de l’application est maintenant un environnement de production avec une copie redondante dans la région de sauvegarde.
2. Si vous n’avez plus besoin de la version V1 de l’application (15 et 16), vous pouvez mettre l’environnement de préproduction hors service.

![Diagramme montrant la configuration de la géoréplication de SQL Database pour la récupération d’urgence cloud avec désaffectation facultative de l’environnement intermédiaire.](./media/manage-application-rolling-upgrade/option2-3.png)

Si la mise à niveau échoue, par exemple en raison d’une erreur dans le script de mise à niveau, considérez l’environnement de préproduction comme étant dans un état incohérent. Pour restaurer l’application telle qu’elle se trouvait avant la mise à niveau, rétablissez l’utilisation de la version V1 de l’application dans l’environnement de production. Les étapes nécessaires sont indiquées sur le schéma suivant :

1. Configurez la copie de la base de données primaire dans l’environnement de production en mode lecture-écriture (17). Cette action restaure toutes les fonctionnalités V1 dans l’environnement de production.
2. Effectuez l’analyse de la cause racine et réparez ou retirez l’environnement de préproduction (18 et 19).

À ce stade, l’application est entièrement fonctionnelle et vous pouvez répéter les étapes de mise à niveau.

> [!NOTE]
> La restauration ne nécessite pas de modifications DNS, car vous n’avez pas effectué d’opération de permutation.

![Diagramme montrant la configuration de la géoréplication de SQL Database pour la récupération d’urgence cloud avec le processus de mise à niveau annulé.](./media/manage-application-rolling-upgrade/option2-4.png)

Le principal avantage de cette option est qu’elle vous permet de mettre à niveau l’application et sa copie géoredondante en parallèle sans compromettre votre continuité de l’activité lors de la mise à niveau.

L’inconvénient est qu’elle implique une double redondance de chaque composant de l’application, ce qui augmente le coût total de l’opération. Elle implique également un flux de travail plus complexe.

## <a name="summary"></a>Résumé

Les deux méthodes de mise à niveau décrites dans cet article présentent certaines différences en termes de complexité et de coût, mais les deux visent à réduire la durée pendant laquelle l’utilisateur est limité aux opérations en lecture seule. Cette durée dépend directement de la durée du script de mise à niveau. Elle ne dépend pas de la taille de la base de données, du niveau de service que vous avez choisi, de la configuration du site web ou d’autres facteurs que vous ne pouvez pas facilement contrôler. Toutes les étapes de préparation sont dissociées de la procédure de mise à niveau et n’ont aucun impact sur l’application de production. L’efficacité du script de mise à niveau est essentielle pour déterminer l’expérience utilisateur au cours des mises à niveau. La meilleure façon d’améliorer cette expérience consiste donc à concentrer vos efforts sur la création d’un script de mise à niveau aussi efficace que possible.

## <a name="next-steps"></a>Étapes suivantes

* Pour une vue d’ensemble de la continuité des activités et des scénarios, consultez [Vue d’ensemble de la continuité des activités](business-continuity-high-availability-disaster-recover-hadr-overview.md).
* Pour en savoir plus sur la géoréplication active Azure SQL Database, consultez [Créer des bases de données secondaires accessibles en lecture à l’aide de la géoréplication active](active-geo-replication-overview.md).
* Pour en savoir plus sur les groupes de basculement automatique Azure SQL Database, consultez [Utiliser les groupes de basculement automatique pour permettre le basculement transparent et coordonné de plusieurs bases de données](auto-failover-group-overview.md).
* Pour en savoir plus sur les environnements de préproduction dans Azure App Service, consultez [Configurer des environnements de préproduction dans Azure App Service](../../app-service/deploy-staging-slots.md).
* Pour en savoir plus les profils Azure Traffic Manager, consultez [Gestion d’un profil Azure Traffic Manager](../../traffic-manager/traffic-manager-manage-profiles.md).
