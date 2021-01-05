---
title: Restaurer une base de données dans une application SaaS multilocataire
description: Apprenez à restaurer une instance d'Azure SQL Database à locataire unique après une suppression accidentelle de données
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 12/04/2018
ms.openlocfilehash: 88496a39b0186cefb7c64e227530b5d73e693094
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92780476"
---
# <a name="restore-a-single-tenant-with-a-database-per-tenant-saas-application"></a>Restaurer un seul locataire avec une application SaaS de base de données par locataire
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Le modèle de base de données par locataire facilite la restauration d’un seul locataire sur un point antérieur dans le temps sans affecter d’autres locataires.

Dans ce didacticiel, vous allez découvrir deux modèles de récupération des données :

> [!div class="checklist"]
> * Restaurer une base de données dans une base de données parallèle (côte à côte).
> * Restaurer une base de données sur place en remplaçant la base de données existante.

| Modèle | Description |
|:--|:--|
| Restaurer dans une base de données parallèle | Ce modèle peut être utilisé pour des tâches telles que la révision, l’audit et la conformité, afin d’autoriser un locataire à inspecter ses données à partir d’un point antérieur. La base de données actuelle du locataire reste inchangée et en ligne. |
| Restaurer sur place | Ce modèle est généralement utilisé pour récupérer un locataire à un point antérieur dans le temps, après la suppression ou la corruption accidentelle des données. La base de données d’origine est mise hors connexion et remplacée par la base de données restaurée. |
|||

Pour suivre ce didacticiel, vérifiez que les prérequis suivants sont remplis :

* L’application SaaS Wingtip est déployée. Pour procéder à un déploiement en moins de cinq minutes, consultez [Déployer et explorer l’application SaaS Wingtip](./saas-dbpertenant-get-started-deploy.md).
* Azure PowerShell est installé. Pour plus d’informations, consultez [Prise en main d’Azure PowerShell](/powershell/azure/get-started-azureps).

## <a name="introduction-to-the-saas-tenant-restore-patterns"></a>Présentation des modèles SaaS de restauration d’un locataire

Il existe deux modèles simples pour restaurer les données d’un locataire individuel. Étant donné que les bases de données des clients sont isolées les unes des autres, la restauration d’un client n’a aucun impact sur les données des autres clients. La fonctionnalité de limite de restauration dans le temps (PITR) d’Azure SQL Database est utilisée dans les deux modèles. PITR crée toujours une nouvelle base de données.

* **Restaurer en parallèle** : dans le premier modèle, une nouvelle base de données parallèle est créée à côté de la base de données actuelle du locataire. Le locataire reçoit l’accès en lecture seule à la base de données restaurée. Les données restaurées peuvent être consultées et potentiellement utilisées pour remplacer les valeurs des données actuelles. Il revient au concepteur d’application de choisir comment le locataire accède à la base de données restaurée et les options de récupération fournies. Autoriser simplement le locataire à consulter ses données à un point antérieur dans le temps peut suffire dans certains scénarios.

* **Restaurer sur place** : le second modèle est utile si des données ont été perdues ou corrompues et si le locataire souhaite restaurer à un point antérieur dans le temps. Le locataire est mis hors connexion pendant la restauration de la base de données. La base de données d’origine est supprimée et la base de données restaurée est renommée. La chaîne de sauvegarde de la base de données d’origine reste accessible après la suppression, vous pouvez ainsi restaurer la base de données à un point antérieur dans le temps, si nécessaire.

Si la base de données utilise la [géoréplication active](active-geo-replication-overview.md) et la restauration en parallèle, nous vous recommandons de copier les données requises de la copie restaurée vers la base de données d’origine. Si vous remplacez la base de données d’origine par la base de données restaurée, vous devez reconfigurer et resynchroniser la géoréplication.

## <a name="get-the-wingtip-tickets-saas-database-per-tenant-application-scripts"></a>Obtenir les scripts de l’application de base de données par locataire SaaS Wingtip Tickets

Les scripts et le code source de l’application de base de données multilocataire SaaS Wingtip Tickets sont disponibles dans le référentiel GitHub [WingtipTicketsSaaS-DbPerTenant](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant). Pour connaître les étapes de téléchargement et de déblocage des scripts SaaS Wingtip Tickets, consultez les [conseils généraux](saas-tenancy-wingtip-app-guidance-tips.md).

## <a name="before-you-start"></a>Avant de commencer

Lorsqu’une base de données est créée, entre 10 et 15 minutes peuvent être nécessaires avant que la première sauvegarde complète soit disponible pour la restauration. Si vous venez d’installer l’application, vous devrez peut-être attendre quelques minutes avant d’essayer de ce scénario.

## <a name="simulate-a-tenant-accidentally-deleting-data"></a>Simuler la suppression accidentelle des données par le client

Pour illustrer ces scénarios de récupération, commencez par supprimer « accidentellement » un événement dans l’une des bases de données de locataire. 

### <a name="open-the-events-app-to-review-the-current-events"></a>Ouvrir l’application Events pour passer en revue les événements en cours

1. Ouvrez le hub d’événements (http://events.wtp.&lt;user&gt;.trafficmanager.net) et sélectionnez **Contoso Concert Hall** .

   ![Concentrateur d’événements](./media/saas-dbpertenant-restore-single-tenant/events-hub.png)

2. Faites défiler la liste des événements, puis notez le dernier événement de la liste.

   ![Le dernier événement s’affiche](./media/saas-dbpertenant-restore-single-tenant/last-event.png)

### <a name="accidentally-delete-the-last-event"></a>Supprimer « accidentellement » le dernier événement

1. Dans PowerShell ISE, ouvrez ...\\Learning Modules\\Business Continuity and Disaster Recovery\\RestoreTenant\\*Demo-RestoreTenant.ps1* , puis définissez la valeur suivante :

   * **$DemoScenario** = **1** , *Supprimer le dernier événement (sans ventes de tickets)* .
2. Appuyez sur F5 pour exécuter le script et supprimer le dernier événement. Le message de confirmation suivant s’affiche :

   ```Console
   Deleting last unsold event from Contoso Concert Hall ...
   Deleted event 'Seriously Strauss' from Contoso Concert Hall venue.
   ```

3. La page des événements de Contoso s’ouvre. Faites défiler vers le bas et vérifiez que l’événement a disparu. Si l’événement figure toujours dans la liste, sélectionnez **Actualiser** et vérifiez qu’il a disparu.
   ![Dernier événement supprimé](./media/saas-dbpertenant-restore-single-tenant/last-event-deleted.png)

## <a name="restore-a-tenant-database-in-parallel-with-the-production-database"></a>Restaurer une base de données client en parallèle avec la base de données de production

Cet exercice restaure la base de données Contoso Concert Hall à un point dans le temps, avant la suppression de l’événement. Ce scénario suppose que vous souhaitez consulter les données supprimées dans une base de données parallèle.

 Le script *Restore-TenantInParallel.ps1* crée une base de données de locataire parallèle nommée *ContosoConcertHall\_old* , avec une entrée de catalogue parallèle. Ce modèle de restauration convient davantage dans le cas d’une récupération après une perte de données mineure. Vous pouvez également utiliser ce modèle si vous devez vérifier des données à des fins de conformité et d’audit. Il est également recommandé lorsque vous utilisez la [géoréplication active](active-geo-replication-overview.md).

1. Terminez la section [Simuler la suppression accidentelle des données par le client](#simulate-a-tenant-accidentally-deleting-data).
2. Dans PowerShell ISE, ouvrez ...\\Learning Modules\\Business Continuity and Disaster Recovery\\RestoreTenant\\_Demo-RestoreTenant.ps1_ .
3. Définissez **$DemoScenario** = **2** , *Restore tenant in parallel* (Restaurer le locataire en parallèle).
4. Pour exécuter le script, appuyez sur la touche F5.

Le script restaure la base de données de locataire sur un point dans le temps situé avant la suppression de l’événement. La base de données est restaurée vers une nouvelle base de données nommée _ContosoConcertHall\_old_ . Les métadonnées de catalogue qui existent dans cette base de données restaurée sont supprimées, puis la base de données est ajoutée au catalogue à l’aide d’une clé créée à partir du nom *ContosoConcertHall\_old* .

Le script de démonstration ouvre la page des événements pour cette nouvelle base de donnés de locataire dans votre navigateur. À noter dans l’URL : ```http://events.wingtip-dpt.&lt;user&gt;.trafficmanager.net/contosoconcerthall_old``` les données sont affichées d’après la base de données restaurée et *_old* est ajouté au nom.

Faites défiler les événements répertoriés dans le navigateur pour confirmer que l’événement supprimé dans la section précédente a bien été restauré.

Le fait d’exposer le locataire restauré en tant que locataire supplémentaire (avec sa propre application Events) n’est probablement pas ce que vous feriez pour fournir un accès au locataire aux données restaurées. Cela nous permet d’illustrer le modèle de restauration. Vous donnez généralement accès en lecture seule aux anciennes données et conservez cette base de données restaurée pendant une période définie. Dans l’exemple, vous pouvez supprimer l’entrée de locataire restauré lorsque vous avez terminé en exécutant le scénario _Remove restored tenant_ (Supprimer le locataire restauré).

1. Définissez **$DemoScenario** = **4** , *Remove restored tenant* (Supprimer le locataire restauré).
2. Pour exécuter le script, appuyez sur la touche F5.
3. L’entrée *ContosoConcertHall\_old* est maintenant supprimée du catalogue. Fermez la page des événements pour ce locataire dans votre navigateur.

## <a name="restore-a-tenant-in-place-replacing-the-existing-tenant-database"></a>Restaurer un client sur place en remplaçant la base de données client existante

Cet exercice restaure le locataire Contoso Concert Hall à un point dans le temps, avant la suppression de l’événement. Le script *Restore-TenantInPlace* restaure une base de données de locataire sur une nouvelle base de données et supprime l’original. Ce modèle de restauration convient davantage dans le cas d’une récupération après une grave corruption des données, et le locataire peut faire face à une perte importante de données.

1. Dans PowerShell ISE, ouvrez le fichier **Demo-RestoreTenant.ps1** .
2. Définissez **$DemoScenario** = **5** , *Restore tenant in place* (Restaurer le locataire en place).
3. Pour exécuter le script, appuyez sur la touche F5.

Le script restaure la base de données de locataire sur un point dans le temps avant la suppression de l’événement. Il déconnecte tout d’abord le locataire Contoso Concert Hall afin d’empêcher d’autres mises à jour. Ensuite, une base de données parallèle est créée en restaurant à partir du point de restauration. La base de données restaurée est nommée avec un horodatage pour garantir que le nom de la base de données n’entre pas en conflit avec le nom de base de données de locataire existante. Ensuite, l’ancienne base de données client est supprimée, et la base de données restaurée est renommée d’après le nom de la base de données d’origine. Enfin, la connexion de Contoso Concert Hall est rétablie pour permettre l’accès de l’application à la base de données restaurée.

Vous avez correctement restauré la base de données à un point dans le temps, avant la suppression de l’événement. La page **Événements** s’ouvre, vérifiez que le dernier événement a été restauré.

Une fois que vous restaurez la base de données, 10 à 15 minutes supplémentaires sont nécessaires avant que la première sauvegarde complète soit à nouveau disponible pour la restauration.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Restaurer une base de données dans une base de données parallèle (côte à côte).
> * Restaurer une base de données sur place.

Essayez le didacticiel [Gérer le schéma de base de données client](saas-tenancy-schema-management.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Autres didacticiels qui s’appuient sur l’application SaaS Wingtip](saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)
* [Vue d’ensemble de la continuité de l’activité avec la base de données Azure SQL](business-continuity-high-availability-disaster-recover-hadr-overview.md)
* [En savoir plus sur les sauvegardes SQL Database](automated-backups-overview.md)