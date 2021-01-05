---
title: Surveiller l’état, afficher l’historique et configurer les alertes
description: Détecter des problèmes d’applications logiques en vérifiant leur état d’exécution, en examinant leur historique du déclencheur et en activant des alertes dans Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: divswa, logicappspm
ms.topic: article
ms.date: 05/04/2020
ms.openlocfilehash: 356353da639ab97a1a4e5483abf56050f5a236f8
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92676061"
---
# <a name="monitor-run-status-review-trigger-history-and-set-up-alerts-for-azure-logic-apps"></a>Surveiller l’état d’exécution, examiner l’historique du déclencheur et configurer des alertes pour Azure Logic Apps

Après avoir [créé et exécuté une application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md), vous pouvez vérifier son état d’exécution, son [historique des exécutions](#review-runs-history), son [historique du déclencheur](#review-trigger-history) et ses performances. Pour obtenir des notifications concernant des échecs ou d’autres problèmes éventuels, configurez des [alertes](#add-azure-alerts). Par exemple, vous pouvez créer une alerte qui détecte quand plus de cinq exécutions échouent en une heure.

Pour une surveillance des événements en temps réel et un débogage enrichi, configurez une journalisation des diagnostics pour votre application logique à l’aide de [journaux Azure Monitor](../azure-monitor/overview.md). Ce service Azure vous aide à superviser vos environnements cloud et locaux afin pouvoir maintenir plus facilement leur disponibilité et leurs performances. Vous pouvez ensuite rechercher et consulter des événements, tels que des événements de déclencheur, des événements d’exécution et des événements d’action. En stockant ces informations dans des [journaux Azure Monitor](../azure-monitor/platform/data-platform-logs.md), vous pouvez créer des [requêtes de journal](../azure-monitor/log-query/log-query-overview.md) qui vous aident à rechercher et à analyser ces informations. Vous pouvez également utiliser ces données de diagnostic avec d’autres services Azure, tels que Stockage Azure et Azure Event Hubs. Pour plus d’informations, voir [Superviser les applications logiques à l’aide d’Azure Monitor](../logic-apps/monitor-logic-apps-log-analytics.md).

> [!NOTE]
> Si vos applications logiques s’exécutent dans un [environnement de service d’intégration](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) créé pour utiliser un [point de terminaison d’accès interne](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md#endpoint-access), vous pouvez afficher les entrées et sorties de l’historique des exécutions de l’application logique, ainsi qu’y accéder, *uniquement depuis l’intérieur de votre réseau virtuel*. Vérifiez que vous disposez d’une connectivité réseau entre les points de terminaison privés et l’ordinateur à partir duquel vous souhaitez accéder à l’historique des exécutions. Par exemple, votre ordinateur client peut exister à l’intérieur du réseau virtuel de l’environnement de service d’intégration ou à l’intérieur d’un réseau virtuel connecté au réseau virtuel de l’environnement de service d’intégration, par exemple, par le biais d’un appairage (peering) ou d’un réseau privé virtuel. Pour plus d’informations, consultez [Accès aux points de terminaison de l’environnement ISE](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md#endpoint-access). 

<a name="review-runs-history"></a>

## <a name="review-runs-history"></a>Examiner l’historique des exécutions

Chaque fois que le déclencheur est activé pour un élément ou un événement, le moteur de Logic Apps crée et exécute une instance de flux de travail distincte pour chaque élément ou événement. Par défaut, chaque instance de flux de travail s’exécute en parallèle afin qu’aucun flux de travail ne doive attendre avant de démarrer une exécution. Vous pouvez examiner le déroulement de cette exécution, y compris l’état de chaque étape du flux de travail, ainsi que les entrées et sorties de celle-ci.

1. Dans le [portail Azure](https://portal.azure.com), recherchez et ouvrez votre application logique dans le concepteur d’applications logiques.

   Pour rechercher votre application logique, dans la zone de recherche Azure principale, entrez `logic apps`, puis sélectionnez **Logic Apps**.

   ![Rechercher et sélectionner le service « Logic Apps »](./media/monitor-logic-apps/find-your-logic-app.png)

   Le portail Azure affiche toutes les applications logiques associées à vos abonnements Azure. Vous pouvez filtrer cette liste en fonction du nom, de l’abonnement, du groupe de ressources, de l’emplacement, etc.

   ![Afficher les applications logiques associées aux abonnements](./media/monitor-logic-apps/logic-apps-list-in-subscription.png)

1. Sélectionnez votre application logique, puis **Vue d’ensemble**.

   Dans le volet Vue d’ensemble, sous **Historique des exécutions** , toutes les exécutions passées, actuelles et en attente de votre application logique s’affichent. Si la liste affiche de nombreuses exécutions et que vous ne trouvez pas l’entrée souhaitée, essayez de filtrer la liste.

   > [!TIP]
   > Si l’état d’exécution ne s’affiche pas, essayez d’actualiser la page de présentation en sélectionnant **Actualiser**. Aucune exécution ne se produit pour un déclencheur ignoré en raison de critères non respectés ou d’une absence de données.

   ![Vue d’ensemble, historique des exécutions et autres informations sur l’application logique](./media/monitor-logic-apps/overview-pane-logic-app-details-run-history.png)

   Voici les états d’exécution possibles :

   | État d’exécution | Description |
   |------------|-------------|
   | **Abandonné** | L’exécution s’est arrêtée ou ne s’est pas terminée en raison de problèmes externes, par exemple, une panne système ou un abonnement Azure expiré. |
   | **Annulé** | L’exécution a été déclenchée et a démarré, mais a reçu une demande d’annulation. |
   | **Échec** | Au moins une action de l’exécution a échoué. Aucune action subséquente n’était configurée dans le flux de travail pour gérer l’échec. |
   | **Exécution** | L’exécution a été déclenchée et est en cours, mais cet état peut également apparaître pour une exécution limitée en raison de [limites d’action](logic-apps-limits-and-config.md) ou du [plan de tarification actuel](https://azure.microsoft.com/pricing/details/logic-apps/). <p><p>**Conseil** : Si vous configurez la [journalisation des diagnostics](monitor-logic-apps-log-analytics.md), vous pouvez obtenir des informations sur tous les événements de limitation qui se produisent. |
   | **Réussi** | L’exécution a réussi. Si une action a échoué, une action subséquente dans le flux de travail a géré cet échec. |
   | **Délai dépassé** | L’exécution a expiré parce qu’elle a dépassé la limite de durée d’exécution qui est contrôlée par le [paramètre **Run history retention in days** (rétention de l’historique d’exécution en jours)](logic-apps-limits-and-config.md#run-duration-retention-limits). La durée d’une exécution est calculée en utilisant l’heure de début et la limite de durée d’exécution à cette heure. <p><p>**Remarque**  : Si la durée de l’exécution dépasse aussi la *limite actuelle de rétention de l’historique d’exécution* , qui est également contrôlée par le [paramètre **Run history retention in days** (Rétention de l’historique d’exécution en jours)](logic-apps-limits-and-config.md#run-duration-retention-limits), l’exécution est effacée de l’historique d’exécution par un travail de nettoyage quotidien. Que l’exécution expire ou se termine, la période de rétention est toujours calculée sur la base de l’heure de début de l’exécution et de la limite de rétention *actuelle*. Par conséquent, si vous réduisez la limite de durée pour une exécution en cours, l’exécution expire. Toutefois, l’exécution reste dans l’historique des exécutions ou est effacée de celui-ci selon que la durée de l’exécution a ou non dépassé la limite de rétention. |
   | **En attente** | L’exécution n’a pas démarré ou est suspendue, par exemple, en raison d’une instance de flux de travail antérieure qui est toujours en cours d’exécution. |
   |||

1. Pour examiner les étapes et d’autres informations d’une exécution spécifique, sous **Historique des exécutions** , sélectionnez cette exécution.

   ![Sélectionner une exécution spécifique à examiner](./media/monitor-logic-apps/select-specific-logic-app-run.png)

   Le volet **exécution de l’application logique** affiche chaque étape de l’exécution sélectionnée, l’état d’exécution de chaque étape et le temps pris par l’exécution de chaque étape, par exemple :

   ![Chaque action dans l’exécution spécifique](./media/monitor-logic-apps/logic-app-run-pane.png)

   Pour afficher ces informations sous forme de liste, dans la barre d’outils **exécution de l’application logique** , sélectionnez **Détails de l’exécution**.

   ![Dans la barre d’outils, sélectionner « Détails de l’exécution »](./media/monitor-logic-apps/select-run-details-on-toolbar.png)

   La vue Détails de l’exécution affiche chaque étape, son état et d’autres informations.

   ![Examiner les détails de chaque étape de l’exécution](./media/monitor-logic-apps/review-logic-app-run-details.png)

   Par exemple, vous pouvez obtenir la propriété **ID de corrélation** de l’exécution, dont vous pouvez avoir besoin lorsque vous utilisez l’ [API REST pour Logic Apps](/rest/api/logic).

1. Pour obtenir plus d’informations sur une étape spécifique, sélectionnez l’une des options suivantes :

   * Dans le volet **Exécution d’application logique** , sélectionnez l’étape pour développer la forme. Vous pouvez désormais afficher des informations telles que des entrées, des sorties et des erreurs qui se sont produites dans cette étape, par exemple :

     ![Dans le volet Exécution d’application logique, afficher l’étape en échec](./media/monitor-logic-apps/specific-step-inputs-outputs-errors.png)

   * Dans le volet **Détails de l’exécution de l’application logique** , sélectionnez l’étape de votre choix.

     ![Dans le volet Détails de l’exécution, afficher l’étape en échec](./media/monitor-logic-apps/select-failed-step-in-failed-run.png)

     Vous pouvez maintenant afficher des informations telles que des entrées et des sorties pour cette étape, par exemple :

   > [!NOTE]
   > Tous les détails et événements d’exécution sont chiffrés dans le service Logic Apps. Il sont déchiffrés uniquement quand un utilisateur demande à consulter ces données. Vous pouvez [masquer des entrées et sorties dans l’historique des exécutions](../logic-apps/logic-apps-securing-a-logic-app.md#obfuscate) ou contrôler l’accès utilisateur à ces informations à l’aide du [contrôle d’accès en fonction du rôle (Azure RBAC)](../role-based-access-control/overview.md).

<a name="review-trigger-history"></a>

## <a name="review-trigger-history"></a>Examiner l’historique du déclencheur

Chaque exécution d’application logique démarre avec un déclencheur. L’historique du déclencheur répertorie toutes les tentatives de déclenchement que votre application logique a faites et contient des informations sur les entrées et sorties pour chaque tentative.

1. Dans le [portail Azure](https://portal.azure.com), recherchez et ouvrez votre application logique dans le concepteur d’applications logiques.

   Pour rechercher votre application logique, dans la zone de recherche Azure principale, entrez `logic apps`, puis sélectionnez **Logic Apps**.

   ![Rechercher et sélectionner le service « Logic Apps »](./media/monitor-logic-apps/find-your-logic-app.png)

   Le portail Azure affiche toutes les applications logiques associées à vos abonnements Azure. Vous pouvez filtrer cette liste en fonction du nom, de l’abonnement, du groupe de ressources, de l’emplacement, etc.

   ![Afficher les applications logiques associées aux abonnements](./media/monitor-logic-apps/logic-apps-list-in-subscription.png)

1. Sélectionnez votre application logique, puis **Vue d’ensemble**.

1. Dans le menu de votre application logique, sélectionnez **Vue d’ensemble**. Dans la section **Résumé** , sous **Évaluation** , sélectionnez **Afficher l’historique du déclencheur**.

   ![Afficher l’historique du déclencheur pour votre application logique](./media/monitor-logic-apps/overview-pane-logic-app-details-trigger-history.png)

   Le volet Historique du déclencheur affiche tous les tentatives de déclencheur que votre application logique a faites. Chaque fois que le déclencheur s’active pour un élément ou un événement, le moteur Logic Apps crée une instance d’application logique distincte qui exécute le flux de travail. Par défaut, chaque instance s’exécute en parallèle afin qu’aucun flux de travail ne doive attendre avant de lancer l’exécution. Par conséquent, si votre application logique se déclenche sur plusieurs éléments en même temps, une entrée de déclencheur avec les mêmes date et heure s’affiche pour chaque élément.

   ![Plusieurs tentatives du déclencheur pour différents éléments](./media/monitor-logic-apps/logic-app-trigger-history.png)

   Les états de tentative de déclenchement possibles sont les suivants :

   | État du déclencheur | Description |
   |----------------|-------------|
   | **Échec** | Une erreur est survenue. Pour consulter les messages d’erreur générés suite à un échec de déclencheur, sélectionnez cette tentative de déclenchement, puis choisissez **Sorties**. Vous pourrez par exemple trouver des entrées qui ne sont pas valides. |
   | **Ignoré** | Le déclencheur a vérifié le point de terminaison, mais n’a pas trouvé de données répondant aux critères spécifiés. |
   | **Réussi** | Le déclencheur a vérifié le point de terminaison et a trouvé des données disponibles. En général, un état **Déclenché** s’affiche à côté de cet état. Dans le cas contraire, la définition du déclencheur peut comporter une condition ou une commande `SplitOn` qui n’a pas été satisfaite. <p><p>Cet état peut provenir d’un déclencheur manuel, d’un déclencheur de périodicité ou d’un déclencheur d’interrogation. Un déclencheur peut s’exécuter correctement, mais l’exécution elle-même risque malgré tout d’échouer si les actions génèrent des erreurs non gérées. |
   |||

   > [!TIP]
   > Vous pouvez revérifier le déclencheur sans attendre la prochaine récurrence. Dans la barre d’outils de la vue d’ensemble, sélectionnez **Déclencheur d’exécution** , puis choisissez le déclencheur, ce qui force une vérification. Vous pouvez également sélectionner **Exécuter** dans la barre d’outils du Concepteur d’applications logiques.

1. Pour afficher des informations sur une tentative de déclencheur spécifique, dans le volet Déclencheur, sélectionnez cet événement du déclencheur. Si la liste affiche plusieurs tentatives de déclencheur et que vous ne trouvez pas l’entrée souhaitée, essayez de filtrer la liste. Si vous ne trouvez pas les données que vous attendez, choisissez **Actualiser** dans la barre d’outils.

   ![Afficher une tentative de déclencheur spécifique](./media/monitor-logic-apps/select-trigger-event-for-review.png)

   Vous pouvez maintenant examiner des informations relatives à l’événement de déclencheur sélectionné, par exemple :

   ![Afficher des informations de déclencheur spécifiques](./media/monitor-logic-apps/view-specific-trigger-details.png)

<a name="add-azure-alerts"></a>

## <a name="set-up-monitoring-alerts"></a>Configurer des alertes de surveillance

Pour recevoir des alertes basées sur des métriques spécifiques ou des dépassements de seuils de votre application logique, configurez des [alertes dans Azure Monitor](../azure-monitor/platform/alerts-overview.md). En savoir plus sur les [métriques dans Azure](../azure-monitor/platform/data-platform.md). Pour configurer des alertes sans utiliser [Azure Monitor](../azure-monitor/log-query/log-query-overview.md), procédez comme suit.

1. Dans le menu de votre application logique, sous **Surveillance** , sélectionnez **Alertes** > **Nouvelle règle d’alerte**.

   ![Ajouter une alerte pour votre application logique](./media/monitor-logic-apps/add-new-alert-rule.png)

1. Dans le volet **Créer une règle** , sous **Ressource** , sélectionnez votre application logique si ce n’est déjà fait. Sous **Condition** , sélectionnez **Ajouter** afin de pouvoir définir la condition qui déclenche l’alerte.

   ![Ajouter une condition pour la règle](./media/monitor-logic-apps/add-condition-for-rule.png)

1. Dans le volet **Configurer la logique de signal** , recherchez et sélectionnez le signal pour lequel vous souhaitez obtenir une alerte. Vous pouvez utiliser la zone de recherche ou, pour trier les signaux par ordre alphabétique, sélectionner l’en-tête de colonne **Nom du signal**.

   Par exemple, si vous souhaitez envoyer une alerte en cas d’échec d’un déclencheur, procédez comme suit :

   1. Dans la colonne **Nom du signal** , recherchez et sélectionnez le signal **Déclenchements ayant échoué**.

      ![Sélectionner un signal pour créer une alerte](./media/monitor-logic-apps/find-and-select-signal.png)

   1. Dans le volet Informations qui s’ouvre pour le signal sélectionné, sous **Logique d’alerte** , configurez votre condition, par exemple :

   1. Pour **Opérateur** , sélectionnez **Supérieur ou égal à**.

   1. Pour **Type d’agrégation** , sélectionnez **Compte**.

   1. Pour **Valeur de seuil** , entrez `1`.

   1. Sous **Aperçu de la condition** , vérifiez que votre condition s’affiche correctement.

   1. Sous **Évaluée sur la base de** , définissez l’intervalle et la fréquence pour l’exécution de la règle d’alerte. Pour **Précision d’agrégation (période)** , sélectionnez la période de regroupement des données. Pour **Fréquence d’évaluation** , sélectionnez la fréquence à laquelle vous souhaitez vérifier la condition.

   1. Quand vous êtes prêt, sélectionnez **Terminé**.

   Voici la condition terminée :

   ![Configurer une condition pour une alerte](./media/monitor-logic-apps/set-up-condition-for-alert.png)

   La page **Créer une règle** affiche maintenant la condition que vous avez créée et le coût d’exécution de cette alerte.

   ![Nouvelle alerte sur la page « Créer une règle »](./media/monitor-logic-apps/finished-alert-condition-cost.png)

1. Spécifiez un nom, une description facultative et un niveau de gravité pour votre alerte. Laissez le paramètre **Activer la règle lors de sa création** activé, ou désactivez-le jusqu’à ce que vous soyez prêt à activer la règle.

1. Lorsque vous avez terminé, sélectionnez **Créer une règle d’alerte**.

> [!TIP]
> Pour exécuter une application logique à partir d’une alerte, vous pouvez inclure le [déclencheur de demande](../connectors/connectors-native-reqres.md) dans votre flux de travail, ce qui vous permet d’effectuer des tâches telles que les exemples suivants :
> 
> * [Publier sur Slack](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-slack-with-logic-app)
> * [Envoyer un texte](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-text-message-with-logic-app)
> * [Ajouter un message à une file d'attente](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-queue-with-logic-app)

## <a name="next-steps"></a>Étapes suivantes

* [Superviser les applications logiques à d’Azure Monitor](../logic-apps/monitor-logic-apps-log-analytics.md)
