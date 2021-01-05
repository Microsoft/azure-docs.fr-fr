---
title: Visualisation des données à partir d’Azure Monitor | Microsoft Docs
description: Récapitulatif des méthodes disponibles pour visualiser les métriques et les données de journal stockées dans Azure Monitor.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/17/2020
ms.openlocfilehash: f9f95a5e85ec4eebf82cca707476750595331d73
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95522666"
---
# <a name="visualizing-data-from-azure-monitor"></a>Visualisation des données à partir d’Azure Monitor
Cet article récapitule les méthodes disponibles pour visualiser les métriques et les données de journal stockées dans Azure Monitor.

Les visualisations telles que les diagrammes ou graphiques peuvent vous aider à analyser vos données de supervision pour explorer des problèmes en profondeur et identifier des modèles. En fonction de l’outil que vous utilisez, vous pouvez également avoir la possibilité de partager les visualisations avec d’autres utilisateurs à l’intérieur et en dehors de votre organisation.

## <a name="workbooks"></a>Workbooks
Les [classeurs](./platform/workbooks-overview.md) sont des documents interactifs qui fournissent des insights détaillés sur vos données, et favorisent l’investigation et la collaboration au sein de l’équipe. Parmi les exemples typiques d’utilisation des classeurs citons les guides de résolution des problèmes et les bilans d’incidents.

![Schéma illustrant des captures d’écran de plusieurs pages d’un classeur, notamment Analyse des consultations de page, Utilisation et Temps passé sur la page.](media/visualizations/workbook.png)

### <a name="advantages"></a>Avantages
- Prend en charge les métriques et les journaux d’activité.
- Prend en charge des paramètres qui permettent de créer des rapports interactifs où la sélection d’un élément dans une table met dynamiquement à jour les visualisations et graphiques associés.
- Flux de type document.
- Possibilité de définir des classeurs personnels ou partagés.
- Expérience de création simple et conviviale.
- Les modèles prennent en charge la galerie de modèles publique basée sur GitHub.

### <a name="limitations"></a>Limites
- Aucune actualisation automatique.
- Les classeurs présentant une disposition moins dense que les tableaux de bord, ils n’offrent pas la même vision panoramique. Conviennent davantage pour fournir des insights plus approfondis.


## <a name="azure-dashboards"></a>Tableaux de bord Azure
Les [tableaux de bord Azure](../azure-portal/azure-portal-dashboards.md) constituent la principale technologie de création de tableaux de bord pour Azure. Ils sont particulièrement efficaces pour fournir un panorama unique de vos infrastructure et services Azure, ce qui vous permet d’identifier rapidement les problèmes importants.

![Capture d’écran montrant un exemple de tableau de bord Azure avec des informations personnalisables.](media/visualizations/dashboard.png)

Voici une procédure vidéo expliquant la création de tableaux de bord.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4AslH]

### <a name="advantages"></a>Avantages
- Intégration approfondie à Azure. Les visualisations peuvent être épinglées aux tableaux de bord à partir de plusieurs pages Azure, notamment Metrics Explorer, Log Analytics et Application Insights.
- Prend en charge les métriques et les journaux d’activité.
- Combinez des données provenant de plusieurs sources, notamment une sortie [Metrics Explorer](platform/metrics-charts.md), des [requêtes de journal](log-query/log-query-overview.md) ainsi que des [cartes](app/app-map.md) et des données de disponibilité dans Application Insights.
- Possibilité de définir des tableaux de bord personnels ou partagés. Intégration avec le [contrôle d’accès Azure en fonction du rôle (Azure RBAC)](../role-based-access-control/overview.md).
- Actualisation automatique. L’actualisation des métriques a lieu selon un intervalle de temps de cinq minutes au minimum. Les journaux sont actualisés toutes les heures, avec une option d’actualisation manuelle à la demande en cliquant sur l’icône d’actualisation d’une visualisation donnée ou en actualisant le tableau de bord complet.
- Les tableaux de bord de métriques sont paramétrés avec un horodatage et des paramètres personnalisés.
- Options de disposition flexibles.
- Mode plein écran.


### <a name="limitations"></a>Limites
- Contrôle limité des visualisations de journal sans prise en charge des tables de données. Le nombre total de séries de données est limité à 10, les nouvelles séries de données étant regroupées sous un _autre_ compartiment.
- Aucune prise en charge de paramètre personnalisé pour les graphiques de journal.
- Les graphiques de journal sont limités aux 30 derniers jours.
- Les graphiques de journal peuvent uniquement être épinglés aux tableaux de bord partagés.
- Aucune interactivité avec les données de tableau de bord.
- Exploration contextuelle limitée.


## <a name="power-bi"></a>Power BI
[Power BI](https://powerbi.microsoft.com/documentation/powerbi-service-get-started/) est particulièrement utile pour créer des rapports et des tableaux de bord centrés sur l’entreprise, ainsi que des rapports d’analyse de tendances d’indicateur de performance clé à long terme. Vous pouvez [importer les résultats d’une requête Log](platform/powerbi.md) dans un jeu de données Power BI afin de bénéficier de ses fonctionnalités, comme la combinaison de données provenant de différentes sources et le partage de rapports sur le web et sur des appareils mobiles.

![Power BI](media/visualizations/power-bi.png)

### <a name="advantages"></a>Avantages
- Visualisations riches.
- Interactivité extensive, notamment par le biais du zoom avant et du filtrage croisé.
- Partage aisé à travers votre organisation.
- Intégration aux autres données issues de plusieurs sources de données.
- Meilleures performances avec les résultats mis en cache dans un cube.


### <a name="limitations"></a>Limites
- Prend en charge les journaux d’activité, mais pas les métriques.
- Aucune intégration à Azure. Ne peut pas gérer les tableaux de bord et les modèles par le biais d’Azure Resource Manager.
- Les résultats des requêtes doivent être importés dans le modèle Power BI à configurer. L’actualisation et la taille des résultats sont soumises à des limites.
- L’actualisation des données est limitée à huit exécutions par jour.


## <a name="grafana"></a>Grafana
[Grafana](https://grafana.com/) est une plateforme ouverte qui convient parfaitement dans les tableaux de bord opérationnels. Elle est particulièrement utile pour détecter, isoler et trier les incidents opérationnels. Vous pouvez ajouter le [plug-in de source de données Azure Monitor pour Grafana](platform/grafana-plugin.md) à votre abonnement Azure afin qu’il visualise vos données de métriques Azure.

![Capture d’écran montrant des visualisations Grafana.](media/visualizations/grafana.png)

### <a name="advantages"></a>Avantages
- Visualisations riches.
- Écosystème de sources de données riche.
- Interactivité des données, notamment par le biais du zoom avant.
- Prend en charge les paramètres.

### <a name="limitations"></a>Limites
- Aucune intégration à Azure. Ne peut pas gérer les tableaux de bord et les modèles par le biais d’Azure Resource Manager.
- Coût lié à la prise en charge d’une infrastructure Grafana supplémentaire ou coûts supplémentaires liés à Grafana Cloud.


## <a name="build-your-own-custom-application"></a>Créer votre propre application personnalisée
Vous pouvez accéder aux données dans les journaux et aux métriques dans Azure Monitor via leur API à l’aide de n’importe quel client REST, et créer ainsi vos propres applications et sites web personnalisés.

### <a name="advantages"></a>Avantages
- Flexibilité complète de l’interface utilisateur, de la visualisation, de l’interactivité et des fonctionnalités.
- Possibilité de combiner les métriques et les données de journal avec d’autres sources de données.

### <a name="disadvantages"></a>Inconvénients
- Effort d’ingénierie significatif requis.


## <a name="azure-monitor-views"></a>Affichages Azure Monitor

> [!IMPORTANT]
> Les vues sont en cours de dépréciation. Pour obtenir des conseils sur la conversion des vues en classeurs, consultez le [Guide de transition : du concepteur de vues Azure Monitor aux classeurs](platform/view-designer-conversion-overview.md).

Les [affichages dans Azure Monitor](platform/view-designer.md) vous permettent de créer des visualisations personnalisées avec les données de journal stockées. Les [solutions de supervision](insights/solutions.md) s’en servent pour présenter les données qu’elles collectent.


![Capture d’écran montrant une vignette Solution de supervision de conteneur et la vue détaillée Azure Monitor qui s’ouvre lorsque vous la sélectionnez.](media/visualizations/view.png)

### <a name="advantages"></a>Avantages
- Visualisations riches des données de journal.
- Possibilité d’exporter et d’importer des affichages pour les transférer vers d’autres groupes de ressources et abonnements.
- Intégration au modèle de gestion Azure Monitor avec des espaces de travail et des solutions de supervision.
- [Filtres](platform/view-designer-filters.md) pour les paramètres personnalisés.
- Interactif ; prend en charge l’exploration sur plusieurs niveaux (d’un affichage à l’autre)

### <a name="limitations"></a>Limites
- Prend en charge les journaux d’activité, mais pas les métriques.
- Aucun affichage personnel. Disponible pour tous les utilisateurs ayant accès à l’espace de travail.
- Aucune actualisation automatique.
- Options de disposition limitées.
- Pas prise en charge de l’interrogation de plusieurs espaces de travail ou applications Application Insights.
- La taille de la réponse des requêtes est limitée à 8 Mo, tandis que le temps d’exécution des requêtes est limité à 110 secondes.

## <a name="next-steps"></a>Étapes suivantes
- Découvrez les [données collectées par Azure Monitor](platform/data-platform.md).
- Découvrez les [tableaux de bord Azure](../azure-portal/azure-portal-dashboards.md).
- Découvrez les [affichages dans Azure Monitor](platform/view-designer.md).
- Découvrez les [classeurs](./platform/workbooks-overview.md).
- Découvrez [l’importation de données de journal dans Power BI](./platform/powerbi.md).
- Découvrez le [plug-in de source de données Azure Monitor pour Grafana](./platform/grafana-plugin.md).

