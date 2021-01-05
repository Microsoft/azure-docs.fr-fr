---
title: Voir Déploiements (préversion) avec Azure Monitor pour conteneurs | Microsoft Docs
description: Cet article décrit la vue en temps réel de Kubernetes Deployments, sans l’utilisation de kubectl, dans Azure Monitor pour conteneurs.
ms.topic: conceptual
ms.date: 10/15/2019
ms.custom: references_regions
ms.openlocfilehash: 2f1eac82ce67818c7bf86ce3ca8924155d8ee2aa
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85337992"
---
# <a name="how-to-view-deployments-preview-in-real-time"></a>Guide pratique pour voir Déploiements (préversion) en temps réel

Avec Azure Monitor pour conteneurs, la vue de la fonctionnalité Déploiements (préversion) émule l’accès direct aux objets Kubernetes Deployment en temps réel par l’exposition des commandes `kubeclt get deployments` et `kubectl describe deployment {your deployment}`.

>[!NOTE]
>Les clusters AKS activés en tant que [clusters privés](https://azure.microsoft.com/updates/aks-private-cluster/) ne sont pas pris en charge par cette fonctionnalité. Cette fonctionnalité repose sur l’accès direct à l’API Kubernetes via un serveur proxy depuis votre navigateur. L’activation de la sécurité réseau pour bloquer l’API Kubernetes depuis ce proxy bloque ce trafic.

Pour plus d’informations, consultez la documentation de Kubernetes sur les [Déploiements](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

## <a name="how-it-works"></a>Fonctionnement

Comme la fonctionnalité Live Data (préversion) accède directement à l’API Kubernetes, des informations supplémentaires sur le modèle d’authentification sont mises à disposition [ici](https://kubernetes.io/docs/concepts/overview/kubernetes-api/).

La fonctionnalité Déploiements (préversion) exécute une seule fois (actualisable) une charge sur le point de terminaison des déploiements `/apis/apps/v1/deployments`. Elle vous permet de sélectionner un déploiement donné et de charger les détails de la description de ce déploiement particulier sur le point de terminaison de déploiement `/apis/apps/v1/namespaces/${nameSpace}/deployments/${deploymentName}`.

Si vous sélectionnez **Actualiser** en haut à gauche de la page, la liste des déploiements est actualisée. La réexécution de la commande `kubectl` est ainsi simulée.

>[!IMPORTANT]
>Aucune donnée n’est stockée de façon permanente pendant l’exécution de cette fonctionnalité. Toutes les informations capturées lors de la session sont supprimées lorsque vous la quittez, ou que vous fermez votre navigateur.

>[!NOTE]
>Vous ne pouvez pas épingler de données Live Data (Préversion) depuis la console sur un tableau de bord Azure.

## <a name="deployments-describe"></a>Description des déploiements

Pour afficher les détails de la description d’un déploiement, ce qui équivaut à `kubectl describe deployment`, effectuez les étapes suivantes.

1. Dans le portail Azure, accédez au groupe de ressources du cluster AKS et sélectionnez votre ressource AKS.

2. Dans le tableau de bord du cluster AKS, sous **Supervision** à gauche, choisissez **Insights**.

3. Sélectionnez l’onglet **Déploiements (préversion)** .

    ![Vue de Déploiements dans le portail Azure](./media/container-insights-livedata-deployments/deployment-view.png)

La vue révèle la liste de tous les déploiements en cours d’exécution, ainsi que l’espace de noms et d’autres informations détaillées en émulant l’exécution de la commande `kubectl get deployments –all-namespaces`. Vous pouvez trier les résultats en sélectionnant l’une des colonnes.

![Détails du volet des propriétés des déploiements](./media/container-insights-livedata-deployments/deployment-properties-pane-details.png)

Lorsque vous sélectionnez un déploiement dans la liste, un volet de propriétés s’affiche automatiquement sur le côté droit de la page. Il affiche des informations relatives au déploiement sélectionné, que vous pourriez afficher si vous exécutiez la commande `kubectl describe deployment {deploymentName}`. Vous avez peut-être remarqué que certains détails manquent dans les informations de description. En particulier, le **Modèle** est absent. La sélection de l’onglet **Brut** vous permet d’accéder aux détails non analysés de la description.

![Détails à l’état brut du volet des propriétés des déploiements](./media/container-insights-livedata-deployments/deployment-properties-pane-raw.png)

Lorsque vous consultez les détails du déploiement, vous pouvez voir les événements et les journaux de conteneurs en temps réel. Sélectionnez **Montrer la console active** pour que le volet de la console Live Data (préversion) apparaisse sous la grille des données des déploiements, dans laquelle vous pouvez voir les données des journaux en direct dans un flux continu. Si l’indicateur d’état de récupération affiche une coche verte tout à droite du volet, cela signifie que des données sont récupérables et que leur envoi en streaming vers votre console commence.

Vous pouvez également filtrer par événements au niveau du cluster ou de l’espace de noms. Pour en savoir plus sur la consultation des données en temps réel dans la console, consultez [Voir Live Data (préversion) avec Azure Monitor pour conteneurs](container-insights-livedata-overview.md).

![Données en direct dans la vue Déploiements de la console](./media/container-insights-livedata-deployments/deployments-console-view-events.png)

## <a name="next-steps"></a>Étapes suivantes

- Pour continuer à découvrir comment utiliser Azure Monitor et surveiller les autres aspects de votre cluster AKS, consultez [Connaître l’état d’Azure Kubernetes Service](container-insights-analyze.md).

- Consultez [Exemples de requêtes de journal](container-insights-log-search.md#search-logs-to-analyze-data) pour voir des requêtes et des exemples prédéfinis permettant de créer des alertes, des visualisations, ou d’effectuer une analyse plus poussée de vos clusters.
