---
title: Mises à jour de la terminologie Azure Monitor | Microsoft Docs
description: Décrit les modifications de terminologie récentes apportées aux services de surveillance Azure.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 10/08/2019
ms.openlocfilehash: 819ef7e61045c0ed4205eeaf0094fec706b0b9da
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86498608"
---
# <a name="azure-monitor-naming-and-terminology-changes"></a>Modification des noms et de la terminologie d’Azure Monitor
Des modifications significatives ont récemment été apportées à Azure Monitor, différents services étant en cours de consolidation afin de simplifier la surveillance pour les clients Azure. Cet article décrit les dernières évolutions des noms et de la terminologie de la documentation d’Azure Monitor.

## <a name="october-2019---diagnostic-log-to-resource-log"></a>Octobre 2019 – Journal de diagnostic renommé en Journal des ressources
Les « journaux de diagnostic » ont été renommés en « journaux de ressources » pour mieux refléter ce qui est réellement collecté. Le terme « paramètres de diagnostic » est conservé.  

## <a name="february-2019---log-analytics-terminology"></a>Février 2019 – Terminologie de Log Analytics
Après la consolidation de différents services sous Azure Monitor, nous passons à la vitesse supérieure en modifiant la terminologie dans notre documentation de façon à mieux décrire le service Azure Monitor et ses différents composants. 

### <a name="log-analytics"></a>Log Analytics
Les données de journal d’activité Azure Monitor sont toujours stockées dans un espace de travail Log Analytics et collectées et analysées par le même service Log Analytics, mais nous sommes en train de remplacer le terme _Log Analytics_ par _journaux d’activité Azure Monitor_ en de nombreux endroits. Ce terme reflète mieux son rôle dans Azure Monitor et renforce la cohérence des [métriques dans Azure Monitor](platform/data-platform-metrics.md).

Le terme _analytique des journaux d’activité_ s’applique maintenant principalement à la page du Portail Azure servant à écrire et exécuter des requêtes et à analyser les données de journal d’activité. C’est l’équivalent fonctionnel de [Metrics Explorer](platform/metrics-charts.md), la page du Portail Azure utilisée pour analyser les données des métriques.

### <a name="log-analytics-workspaces"></a>Espaces de travail Log Analytics
Les [espaces de travail](platform/manage-access.md) qui contiennent des données de journal dans Azure Monitor sont toujours appelés espaces de travail Log Analytics. Le menu **Log Analytics** du Portail Azure a été renommé **Espaces de travail Log Analytics** ; c’est ici que l’on [crée des espaces de travail](learn/quick-create-workspace.md) et que l’on configure des sources de données. Analysez vos journaux d’activité et d’autres données de monitoring dans **Azure Monitor** et configurez votre espace de travail dans les **espaces de travail Log Analytics**.

### <a name="management-solutions"></a>Liste des solutions de gestion
Les [solutions de gestion](insights/solutions.md) ont été renommées _Solutions de monitoring_, ce qui décrit mieux leur fonction.


## <a name="august-2018---consolidation-of-monitoring-services-into-azure-monitor"></a>Août 2018 – Consolidation des services de monitoring dans Azure Monitor
Log Analytics et Application Insights ont été consolidés dans Azure Monitor afin de fournir une expérience intégrée unique pour la surveillance des ressources Azure et des environnements hybrides. Aucune fonctionnalité n’a été supprimée dans ces services, et les utilisateurs peuvent effectuer les mêmes scénarios qu’auparavant sans perte ni compromis dans les fonctionnalités.

La documentation de chacun de ces services a été fusionnée dans un même ensemble de contenu pour Azure Monitor. Cela permet d’aider le lecteur à trouver tout le contenu d’un scénario de surveillance spécifique à un même emplacement au lieu de consulter plusieurs ensembles de contenu. L’intégration du contenu se poursuivra en parallèle de l’évolution du service consolidé.

D’autres fonctionnalités autrefois considérées comme faisant partie de Log Analytics, comme les agents et les vues, ont également été repositionnées en tant que fonctionnalités d’Azure Monitor. Ces fonctionnalités n’ont pas changé, à part quelques améliorations potentielles de leur expérience dans le portail Azure.


## <a name="april-2018---retirement-of-operations-management-suite-brand"></a>Avril 2018 – Retrait de la marque Operations Management Suite
Operations Management Suite (OMS) était un regroupement des services de gestion Azure suivants pour la gestion des licences :

- Application Insights
- Azure Automation
- Sauvegarde Azure
- Log Analytics
- Site Recovery

[Une nouvelle tarification a été introduite pour ces services](https://azure.microsoft.com/blog/introducing-a-new-way-to-purchase-azure-monitoring-services/), et le regroupement OMS n’est plus disponible pour les nouveaux clients. Aucun des services qui faisaient partie d’OMS n’a changé, à l’exception de la consolidation dans Azure Monitor décrite ci-dessus. 




## <a name="next-steps"></a>Étapes suivantes

- Consultez la [vue d’ensemble d’Azure Monitor](overview.md), qui décrit ses différents composants et fonctionnalités.
- En savoir plus sur la [transition du portail OMS](./platform/oms-portal-transition.md).
