---
title: Afficher et créer des requêtes pour Azure Logic Apps dans des journaux Azure Monitor
description: Afficher et créer des requêtes dans des journaux Azure Monitor pour Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: divswa, logicappspm
ms.topic: article
ms.date: 01/30/2020
ms.openlocfilehash: d1f575d9d588ccd99523c6b6d5d2ceba6369dc3d
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92309912"
---
# <a name="view-and-create-queries-for-monitoring-and-tracking-in-azure-monitor-logs-for-azure-logic-apps"></a>Afficher et créer des requêtes de surveillance et de suivi dans des journaux Azure Monitor pour Azure Logic Apps

Vous pouvez afficher les requêtes sous-jacentes qui produisent les résultats des [journaux Azure Monitor](../azure-monitor/log-query/log-query-overview.md) et créer des requêtes qui filtrent les résultats en fonction de vos critères spécifiques. Par exemple, vous pouvez rechercher des messages sur la base d’un numéro de contrôle d’échange spécifique. Les requêtes utilisent le [langage de requête Kusto](/azure/data-explorer/kusto/query/) que vous pouvez modifier si vous voulez voir des résultats différents. Pour plus d’informations, voir [Requêtes de journal Azure Monitor](/azure/data-explorer/kusto/query/).

## <a name="prerequisites"></a>Conditions préalables requises

* Un espace de travail Log Analytics. Si vous n’avez pas d’espace de travail Log Analytics, découvrez [comment créer un espace de travail Log Analytics](../azure-monitor/learn/quick-create-workspace.md).

* Application logique configurée avec la journalisation Azure Monitor, qui envoie ces informations à un espace de travail Log Analytics. Découvrez [comment configurer les journaux Azure Monitor pour votre application logique](../logic-apps/monitor-logic-apps.md).

* Si vous utilisez un compte d’intégration, assurez-vous que vous avez configuré celui-ci avec la journalisation Azure Monitor pour envoyer ces informations à un espace de travail Log Analytics. Découvrez comment [configurer la journalisation Azure Monitor pour votre compte d’intégration](../logic-apps/monitor-b2b-messages-log-analytics.md).

## <a name="view-queries-behind-results"></a>Afficher les requêtes à l’origine des résultats

Pour afficher ou modifier la requête qui produit les résultats dans le résumé de votre espace de travail, procédez comme suit :

1. Dans une page de résultats, en bas, sélectionnez **Afficher Tout**.

   ![Voir tous les résultats](./media/create-monitoring-tracking-queries/logic-app-see-all.png)

   La page Journaux s’ouvre et affiche la requête ayant généré la page de résultats précédente.

   ![Page Journaux – Affichage de requête](./media/create-monitoring-tracking-queries/view-query-behind-results.png)

1. Dans la page **Journaux**, vous pouvez sélectionner les options suivantes :

   * Pour afficher les résultats de la requête sous la forme d’une table, dans l’éditeur de requête, sélectionnez **Table**.

   * Pour changer la requête, mettez à jour la chaîne de requête, puis sélectionnez **Exécuter** pour voir les résultats dans le tableau.

## <a name="create-your-own-query"></a>Créer votre propre requête

Pour rechercher ou filtrer des résultats en fonction de propriétés ou de valeurs spécifiques, vous pouvez créer votre propre requête en partant d’une requête vide ou utiliser une requête existante. Pour plus d’informations, voir [Bien démarrer avec les requêtes de journal dans Azure Monitor](../azure-monitor/log-query/get-started-queries.md).

1. Sur le [portail Azure](https://portal.azure.com), recherchez et ouvrez votre espace de travail Log Analytics.

1. Dans le menu de votre espace de travail, sous **Général**, sélectionnez **Journaux**.

1. Partez d’une requête vide ou de toute requête existante disponible.

   * Pour vérifier si une requête existante est disponible, dans la barre d’outils de requête, sélectionnez **Exemples de requêtes** > **Historique**, qui affiche des requêtes précédentes, ou sélectionnez **Explorateur de requêtes** qui affiche des requêtes prédéfinies.

     Par exemple, la solution Logic Apps B2B fournit les requêtes prédéfinies suivantes :

     ![Démarrer avec des requêtes prédéfinies de solution, « Logic Apps B2B »](./media/create-monitoring-tracking-queries/b2b-prebuilt-queries.png)

   * Pour partir d’une requête vide, dans l’éditeur de requête, commencez à taper le [langage de requête Kusto](/azure/data-explorer/kusto/query/) pour votre requête.

     ![Commencer par une requête vide](./media/create-monitoring-tracking-queries/create-query-from-blank.png)

## <a name="next-steps"></a>Étapes suivantes

* [Schémas de suivi AS2](../logic-apps/logic-apps-track-integration-account-as2-tracking-schemas.md)
* [Schémas de suivi X12](../logic-apps/logic-apps-track-integration-account-x12-tracking-schema.md)
* [Schémas de suivi personnalisé](../logic-apps/logic-apps-track-integration-account-custom-tracking-schema.md)