---
title: Sources de données des classeurs Azure Monitor | Microsoft Docs
description: Créez des rapports complexes en toute simplicité grâce à des classeurs Azure Monitor paramétrables prédéfinis et personnalisés créés à partir de plusieurs sources de données.
services: azure-monitor
documentationcenter: ''
manager: carmonm
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 06/29/2020
ms.openlocfilehash: d41629dd9a56272af89a06cb55e9bd88b604baee
ms.sourcegitcommit: dd45ae4fc54f8267cda2ddf4a92ccd123464d411
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92927904"
---
# <a name="azure-monitor-workbooks-data-sources"></a>Sources de données des classeurs Azure Monitor

Les classeurs sont compatibles avec de nombreuses sources de données. Cet article présente les sources de données actuellement disponibles pour les classeurs Azure Monitor.

## <a name="logs"></a>Journaux d’activité

Les classeurs permettent d’interroger les journaux provenant des sources suivantes :

* journaux Azure Monitor (ressources Application Insights et espaces de travail Log Analytics) ;
* données centrées sur les ressources (journaux d’activité).

Les auteurs de classeurs peuvent utiliser des requêtes KQL qui transforment les données de ressource sous-jacentes pour sélectionner un jeu de résultats visualisable sous forme de texte, de graphiques ou de grilles.

![Capture d’écran de l’interface de rapport des journaux des classeurs](./media/workbooks-overview/logs.png)

Les auteurs de classeurs peuvent interroger plusieurs ressources, ce qui donne une expérience de création de rapports riche et entièrement unifiée.

## <a name="metrics"></a>Mesures

Les ressources Azure émettent des [métriques](data-platform-metrics.md) accessibles dans les classeurs, au moyen d’un contrôle spécialisé permettant de spécifier les ressources cibles, les métriques souhaitées et leur agrégation. Ces données peuvent ensuite être tracées dans des graphiques ou des grilles.

![Capture d’écran de graphiques de métriques de classeur concernant l’utilisation du processeur](./media/workbooks-overview/metrics-graph.png)

![Capture d’écran de l’interface de métriques du classeur](./media/workbooks-overview/metrics.png)

## <a name="azure-resource-graph"></a>Azure Resource Graph

Les classeurs prennent en charge l’interrogation des ressources et de leurs métadonnées avec Azure Resource Graph (ARG). Cette fonctionnalité sert principalement à créer des étendues de requêtes personnalisées pour les rapports. L’étendue de ressource est exprimée par un sous-ensemble KQL pris en charge par ARG, ce qui est souvent suffisant pour les cas d’usage courants.

Pour qu’un contrôle de requête utilise cette source de données, utilisez la liste déroulante Type de requête pour choisir Azure Resource Graph et sélectionnez les abonnements à cibler. Utilisez le contrôle de requête pour ajouter le sous-ensemble KQL ARG qui sélectionne un sous-ensemble de ressources intéressant.

![Capture d’écran de la requête KQL Azure Resource Graph](./media/workbooks-overview/azure-resource-graph.png)

## <a name="azure-resource-manager"></a>Azure Resource Manager

Le classeur prend en charge les opérations REST Azure Resource Manager. Cela permet d’interroger le point de terminaison management.azure.com sans avoir à fournir votre propre jeton d’en-tête d’autorisation.

Pour qu’un contrôle de requête utilise cette source de données, utilisez la liste déroulante Source de données pour choisir Azure Resource Manager. Fournissez les paramètres appropriés, tels que la méthode http, le chemin d’accès URL, les en-têtes, les paramètres d’URL et/ou le corps.

> [!NOTE]
> Les types d'opérations `GET`, `POST` et `HEAD` sont les seuls actuellement pris en charge.

## <a name="azure-data-explorer"></a>Explorateur de données Azure

Les classeurs prennent maintenant en charge l’interrogation de clusters [Azure Data Explorer](/azure/data-explorer/) avec le puissant langage de requête [Kusto](/azure/kusto/query/index).   

![Capture d’écran de la fenêtre de requête Kusto](./media/workbooks-overview/data-explorer.png)

## <a name="workload-health"></a>Intégrité de la charge de travail

Azure Monitor possède des fonctionnalités proactives de monitorage de la disponibilité et des performances des systèmes d’exploitation invités Windows ou Linux. Azure Monitor modélise les composants clés et leurs relations, les critères de mesure de leur intégrité et les composants qui donnent lieu à des alertes en cas de détection d’un état défectueux. Les classeurs permettent aux utilisateurs d’utiliser ces informations pour créer des rapports interactifs enrichis.

Pour qu’un contrôle de requête utilise cette source de données, utilisez la liste déroulante **Type de requête** pour choisir Intégrité des charges de travail et sélectionnez l’abonnement, le groupe de ressources ou les ressources de machine virtuelle à cibler. Utilisez les listes déroulantes de filtrage de l’intégrité pour sélectionner un sous-ensemble d’incidents d’intégrité intéressant pour vos besoins d’analytique.

![Capture d’écran de la requête Alertes](./media/workbooks-overview/workload-health.png)

## <a name="azure-resource-health"></a>Azure Resource Health

Les classeurs permettent de récupérer l’intégrité des ressources Azure et de la combiner avec d’autres sources de données pour créer des rapports d’intégrité enrichis et interactifs.

Pour qu’un contrôle de requête utilise cette source de données, utilisez la liste déroulante **Type de requête** pour choisir Intégrité Azure et sélectionnez les ressources à cibler. Utilisez les listes déroulantes de filtrage de l’intégrité pour sélectionner un sous-ensemble de problèmes de ressources intéressant pour vos besoins d’analytique.

![Capture d’écran de la requête d’alertes qui permet d’afficher les listes de filtres d’intégrité](./media/workbooks-overview/resource-health.png)

## <a name="json"></a>JSON

Le fournisseur JSON vous permet de créer un résultat de requête à partir d’un contenu JSON statique. Il est le plus souvent utilisé dans Paramètres pour créer des paramètres déroulants de valeurs statiques. Des tableaux ou objets JSON simples seront automatiquement convertis en lignes et colonnes de grille.  Pour des comportements plus spécifiques, vous pouvez utiliser l’onglet Résultats et les paramètres JSONPATH afin de configurer les colonnes.

## <a name="alerts-preview"></a>Alertes (préversion)

> [!NOTE]
> La méthode suggérée pour interroger les informations Azure Alerte consiste à utiliser la source de données [Azure Resource Graph](#azure-resource-graph), en interrogeant le tableau `AlertsManagementResources`.
>
> Pour obtenir des exemples, consultez la [référence de table Azure Resource Graph](../../governance/resource-graph/reference/supported-tables-resources.md)ou le [modèle Alerts](https://github.com/microsoft/Application-Insights-Workbooks/blob/master/Workbooks/Azure%20Resources/Alerts/Alerts.workbook).
>
> La source de données Alerts reste disponible pendant un certain temps, tandis que les auteurs passent à l’utilisation de ARG. L’utilisation de cette source de données dans les modèles est déconseillée. 

Les classeurs permettent aux utilisateurs de visualiser les alertes actives liées à leurs ressources. Limitations : la source de données Alerts requiert un accès en lecture à l’abonnement pour interroger les ressources et peut ne pas afficher les types d’alertes plus récents. 

Pour qu’un contrôle de requête utilise cette source de données, utilisez la liste déroulante _Source de données_ pour choisir _Alertes (préversion)_ et sélectionnez les abonnements, groupes de ressources ou ressources à cibler. Utilisez les listes déroulantes de filtrage des alertes pour sélectionner un sous-ensemble d’alertes intéressant pour vos besoins d’analytique.

## <a name="custom-endpoint"></a>Point de terminaison personnalisé

Les classeurs prennent en charge l’obtention de données à partir de n’importe quelle source externe. Si vos données résident en dehors d’Azure, vous pouvez les placer dans des classeurs à l’aide de ce type de source de données.

Pour qu’un contrôle de requête utilise cette source de données, utilisez la liste déroulante _Source de données_ pour choisir _Point de terminaison personnalisé_. Fournissez les paramètres appropriés, tels que `Http method`, `url`, `headers`, `url parameters` et/ou `body`. Assurez-vous que votre source de données prenne en charge [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS). sinon, la requête échouera.

Pour éviter d’effectuer automatiquement des appels à des hôtes non approuvés lors de l’utilisation de modèles, l’utilisateur doit marquer les hôtes utilisés comme approuvés. Pour ce faire, vous pouvez soit cliquer sur le bouton _Ajouter en tant qu’approuvé_ , soit l’ajouter en tant qu’hôte approuvé dans les Paramètres du classeur. Ces paramètres sont enregistrés dans les navigateurs qui prennent en charge IndexDb avec les traitements Web, plus d’informations [ici](https://caniuse.com/#feat=indexeddb).

> [!NOTE]
> N’écrivez aucun secret dans les champs (`headers`, `parameters`, `body`, `url`), car ils seront visibles par tous les utilisateurs du Classeur.

## <a name="next-steps"></a>Étapes suivantes

* [Commencez](./workbooks-overview.md#visualizations) à en apprendre davantage sur les nombreuses options pour les visualisations enrichies des classeurs.
* [Contrôlez](workbooks-access-control.md) et partagez l’accès à vos ressources de classeur.
* [Conseils d’optimisation des requêtes de Log Analytics](../log-query/query-optimization.md)