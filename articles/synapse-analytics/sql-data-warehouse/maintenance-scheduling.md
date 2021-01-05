---
title: Planifications de maintenance pour le pool SQL Synapse
description: La planification de maintenance permet aux clients de planifier les événements de maintenance planifiée nécessaires à Azure Synapse Analytics pour déployer de nouveaux correctifs, de nouvelles mises à niveau et de nouvelles fonctionnalités.
services: synapse-analytics
author: antvgski
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 02/02/2019
ms.author: anvang
ms.reviewer: jrasnick
ms.openlocfilehash: 1c5bc4400e99fb1c24e321e623aaee523b9c7383
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85210981"
---
# <a name="use-maintenance-schedules-to-manage-service-updates-and-maintenance"></a>Utiliser les planifications de maintenance pour gérer les mises à jour et la maintenance des services

La fonctionnalité de planification de maintenance intègre les notifications de maintenance planifiée de Service Health, l’analyse de vérification de Resource Health et le service de planification de la maintenance pour le pool SQL Synapse (entrepôt de données) dans Azure Synapse Analytics.

Vous devez utiliser la planification de maintenance afin de choisir la bonne fenêtre pour recevoir les nouvelles fonctionnalités, les mises à niveau et les correctifs. Vous devrez choisir une fenêtre de maintenance principale et secondaire au cours d’une période de sept jours. Chaque fenêtre doit se trouver dans des plages de jours distinctes.

Par exemple, vous pouvez planifier une fenêtre principale du samedi 22:00 au dimanche 01:00, et une fenêtre secondaire le mercredi de 19:00 à 22:00. Si la maintenance ne peut pas être effectuée pendant la fenêtre de maintenance principale, une nouvelle tentative a lieu pendant la fenêtre secondaire. À l’occasion, la maintenance du service peut se produire à la fois pendant les fenêtres principale et secondaire. Pour garantir l’exécution rapide de toutes les opérations de maintenance, les niveaux d’entrepôt de données DW400c et inférieurs peuvent effectuer la maintenance en dehors d’une fenêtre de maintenance désignée.

Pendant le déploiement, une planification de maintenance définie par le système est appliquée à toutes les instances d’entrepôt de données récemment créées. La planification peut être modifiée une fois le déploiement terminé.

Bien qu’une fenêtre de maintenance puisse être comprise entre trois et huit heures, cela ne signifie pas que l’entrepôt de données est hors connexion pendant toute la durée. La maintenance peut se produire à tout moment dans cette fenêtre. En outre, vous devez vous attendre à une déconnexion unique pendant cette période d’environ 5 à 6 minutes, lorsque le service déploie du nouveau code dans votre entrepôt de données. DW400c et les niveaux inférieurs peuvent rencontrer plusieurs brèves pertes de connectivité à différents moments de la fenêtre de maintenance. Quand la maintenance démarre, toutes les sessions actives sont annulées et les transactions non validées sont restaurées. Pour réduire le temps d’arrêt des instances, vérifiez qu’aucune transaction de longue durée n’est en cours dans votre entrepôt de données avant le début de la période de maintenance choisie.

Toutes les opérations de maintenance doivent se terminer dans les fenêtres de maintenance spécifiées, sauf si nous sommes obligés de déployer une mise à jour urgente. Si votre entrepôt de données est suspendu pendant une maintenance planifiée, il sera mis à jour pendant l’opération de reprise. Vous êtes averti une fois la maintenance de votre entrepôt de données terminée.

## <a name="alerts-and-monitoring"></a>Alertes et supervision

L’intégration des notifications Azure Service Health et du moniteur de vérifications Azure Resource Health permet aux clients d’être informés des activités de maintenance imminentes. Cette automatisation tire parti d’Azure Monitor. Vous décidez comment vous souhaitez être informé des événements de maintenance imminente. Vous pouvez également choisir les flux automatisés qui vous aideront à gérer les temps d’arrêt et à minimiser l’impact sur les opérations.

Une notification 24 heures à l’avance doit précéder tous les événements de maintenance qui ne concernent pas les niveaux DWC400c et inférieurs.

> [!NOTE]
> Dans le cas où nous sommes amenés à déployer une mise à jour critique dans le temps, les temps de notification avancés peuvent être considérablement réduits.

Si vous avez reçu une notification pour une prochaine maintenance mais que la maintenance ne peut pas être effectuée pendant la période annoncée, vous recevrez une notification d’annulation. La maintenance reprendra lors de la prochaine période de maintenance planifiée.

Tous les événements de maintenance actifs sont affichés dans la section **Service Health - Maintenance planifiée**. L’historique d’Azure Service Health contient un enregistrement complet des événements passés. Vous pouvez surveiller la maintenance via le tableau de bord du portail de vérification Azure Service Health pendant un événement actif.

### <a name="maintenance-schedule-availability"></a>Disponibilité de la planification de maintenance

Même si la planification de la maintenance n’est pas encore disponible dans votre région, vous pouvez l’afficher et la modifier à tout moment. Quand la planification de la maintenance sera disponible dans votre région, la planification identifiée deviendra immédiatement active dans votre pool SQL Synapse.

## <a name="view-a-maintenance-schedule"></a>Afficher une planification de maintenance

Par défaut, toutes les instances d’entrepôt de données récemment créées ont deux fenêtres de maintenance (une principale et une secondaire) de huit heures, appliquées pendant le déploiement. Comme indiqué ci-dessus, vous pouvez modifier les fenêtres dès que le déploiement est terminé. Aucune maintenance ne peut avoir lieu en dehors des fenêtres de maintenance définies, sans notification préalable.

Pour voir la planification de maintenance qui a été appliquée à votre pool SQL Synapse, effectuez les étapes suivantes :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Sélectionnez le pool SQL Synapse que vous voulez voir.
3. Le pool SQL Synapse sélectionné s’ouvre dans le panneau Vue d’ensemble. La planification de maintenance appliquée à l’entrepôt de données s’affiche sous **Planification de la maintenance**.

![Panneau Vue d’ensemble](./media/maintenance-scheduling/clear-overview-blade.PNG)

## <a name="change-a-maintenance-schedule"></a>Changer une planification de maintenance

Il est possible de mettre à jour ou de modifier une planification de maintenance à tout moment. Si l’instance sélectionnée se trouve dans un cycle de maintenance actif, les paramètres sont enregistrés. Ils deviennent actifs au cours de la prochaine période de maintenance identifiée. [Explorez](../../service-health/resource-health-overview.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) la supervision de votre entrepôt de données pendant un événement de maintenance actif.

## <a name="identifying-the-primary-and-secondary-windows"></a>Identification des fenêtres principale et secondaire

Les fenêtres principale et secondaire doivent avoir des plages de jour distinctes. Par exemple, une fenêtre principale allant du mardi au jeudi, et une fenêtre secondaire allant du samedi au dimanche.

Pour changer la planification de maintenance de votre pool SQL Synapse, effectuez les étapes suivantes :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Sélectionnez le pool SQL Synapse que vous voulez mettre à jour. La page s’ouvre dans le panneau Vue d’ensemble.
Ouvrez la page de paramètres de planification de maintenance en sélectionnant le lien **Résumé de la planification de maintenance** du panneau Vue d’ensemble. Ou sélectionnez l’option **Planification de la maintenance** dans le menu de ressource à gauche.

    ![Options du panneau Vue d’ensemble](./media/maintenance-scheduling/maintenance-change-option.png)

3. Identifiez la plage de jours par défaut pour votre fenêtre de maintenance principale à l’aide des options en haut de la page. Cette sélection détermine si la fenêtre principale a lieu un jour de semaine ou pendant le week-end. Votre sélection met à jour les valeurs des listes déroulantes.
Dans la préversion, il est possible que certaines régions ne prennent pas en charge l’ensemble complet des options **Jour** disponibles.

   ![Panneau Paramètres de maintenance](./media/maintenance-scheduling/maintenance-settings-page.png)

4. Choisissez vos fenêtres de maintenance principale et secondaire par défaut à l’aide des zones de liste déroulante :
   - **Jour** : jour par défaut où doit avoir lieu la maintenance pendant la fenêtre sélectionnée.
   - **Heure de début** : heure par défaut à laquelle doit commencer la fenêtre de maintenance.
   - **Fenêtre de temps** : durée par défaut de la fenêtre de temps.

   La zone **Résumé de la planification** en bas du panneau est mise à jour avec les valeurs que vous avez sélectionnées.
  
5. Sélectionnez **Enregistrer**. Un message s’affiche pour confirmer que votre nouvelle planification est maintenant active.

   Si vous enregistrez une planification dans une région qui ne prend pas en charge la planification de maintenance, le message suivant s’affiche. Vos paramètres sont enregistrés et deviennent actifs dès que la fonctionnalité est disponible dans la région que vous avez sélectionnée.

   ![Message concernant la disponibilité dans la région](./media/maintenance-scheduling/maintenance-not-active-toast.png)

## <a name="next-steps"></a>Étapes suivantes

- [En savoir plus](../../azure-monitor/platform/alerts-metric.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) sur la création, l’affichage et la gestion des alertes à l’aide d’Azure Monitor.
- [En savoir plus](../..//azure-monitor/platform/alerts-log-webhook.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) sur les actions webhook pour les règles d’alerte de journal.
- [En savoir plus](../..//azure-monitor/platform/action-groups.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) sur la création et la gestion de groupes d’actions.
- [En savoir plus](../../service-health/service-health-overview.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) sur Azure Service Health.
