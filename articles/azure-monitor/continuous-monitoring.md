---
title: Supervision continue avec Azure Monitor | Microsoft Docs
description: Décrit les étapes spécifiques de l’utilisation d’Azure Monitor pour activer la supervision continue dans vos workflows.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 10/12/2018
ms.openlocfilehash: 1892afde42ccbcf1c28b7d6a757875772ee3ac5d
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96905602"
---
# <a name="continuous-monitoring-with-azure-monitor"></a>Supervision continue avec Azure Monitor

La supervision continue désigne le processus et la technologie nécessaires à l’intégration de la supervision dans chaque phase des cycles de vie de vos opérations informatiques et DevOps. Elle permet de garantir en permanence l’intégrité, les performances et la fiabilité de votre application et de votre infrastructure durant le passage du développement à la production. La supervision continue prend appui sur les concepts d’intégration continue et de déploiement continu (CI/CD) qui vous aident à développer et à livrer des logiciels de façon plus rapide et plus fiable pour fournir une valeur continue à vos utilisateurs.

[Azure Monitor](overview.md) constitue la solution de supervision unifiée Azure qui fournit l’observabilité de la pile complète entre les applications et l’infrastructure, dans le cloud et localement. Il fonctionne parfaitement avec [Visual Studio et Visual Studio Code](https://visualstudio.microsoft.com/) lors des phases de développement et de test, et s’intègre à [Azure DevOps](/azure/devops/user-guide/index) pour la gestion des mises en production et la gestion des éléments de travail pendant le déploiement et les opérations. Il s’intègre même aux outils ITSM et SIEM de votre choix, pour permettre le suivi des problèmes et des incidents au sein de vos processus informatiques existants.

Cet article décrit les étapes spécifiques de l’utilisation d’Azure Monitor pour activer la supervision continue dans vos workflows. Il inclut des liens vers d’autres documents qui contiennent des détails sur l’implémentation des différentes fonctionnalités.


## <a name="enable-monitoring-for-all-your-applications"></a>Activer la supervision pour toutes vos applications
Afin d’obtenir l’observabilité sur l’ensemble de votre environnement, vous devez activer la supervision sur tous vos services et applications web. Vous pourrez ainsi facilement visualiser les transactions et connexions de bout en bout de tous les composants.

- [Azure DevOps Projects](../devops-project/overview.md) vous offre une expérience simplifiée avec votre code et dépôt Git existants, mais vous pouvez choisir d’utiliser un des exemples d’application pour créer un pipeline d’intégration continue (CI) et de déploiement continu (CD) vers Azure.
- La [supervision continue dans votre pipeline de mise en production DevOps](./app/continuous-monitoring.md) vous permet de réguler ou de restaurer votre déploiement en fonction des données de supervision.
- [Status Monitor](./app/monitor-performance-live-website-now.md) vous permet d’instrumenter une application .NET dynamique sur Windows avec Azure Application Insights, sans avoir besoin de modifier ou de redéployer votre code.
- Si vous avez accès au code de votre application, activez la supervision complète avec [Application Insights](./app/app-insights-overview.md) en installant le SDK Azure Monitor Application Insights pour [.NET](./app/asp-net.md), [.NET Core](./app/asp-net-core.md), [Java](./app/java-get-started.md), [Node.js](./learn/nodejs-quick-start.md) ou [autres langages de programmation](./app/platforms.md). Vous pouvez alors spécifier des événements, métriques ou vues de page personnalisés qui correspondent à votre application et à votre activité.



## <a name="enable-monitoring-for-your-entire-infrastructure"></a>Activer la supervision pour toute votre infrastructure
Les applications ne sont fiables que si leur infrastructure sous-jacente l’est. L’activation de la supervision sur l’intégralité de votre infrastructure vous permet d’obtenir l’observabilité totale et de faciliter la découverte d’une cause racine potentielle lors d’un échec. Azure Monitor vous permet de suivre l’intégrité et les performances de votre infrastructure hybride dans son intégralité, y compris de ressources telles que les machines virtuelles, les conteneurs, le stockage et le réseau.

- Vous obtenez automatiquement les [métriques de plateforme, les journaux d’activité et les journaux de diagnostic](platform/data-sources.md) de la plupart de vos ressources Azure, sans aucune configuration.
- Activez une supervision plus approfondie pour les machines virtuelles avec [Azure Monitor pour machines virtuelles](insights/vminsights-overview.md).
-  Activez une supervision plus approfondie pour les clusters AKS avec [Azure Monitor pour conteneurs](insights/container-insights-overview.md).
- Ajoutez des [solutions de supervision](./monitor-reference.md) pour différents services et applications de votre environnement.


L’[infrastructure en tant que code](/azure/devops/learn/what-is-infrastructure-as-code) est la gestion de l’infrastructure dans un modèle descriptif, en utilisant la même gestion de versions que celle utilisée par les équipes DevOps pour le code source. Cette possibilité ajoute fiabilité et scalabilité à votre environnement, tout en vous permettant de tirer parti de processus similaires à ceux qui habituellement gèrent vos applications.

-  Utilisez des [modèles Resource Manager](./samples/resource-manager-workspace.md) pour activer la supervision et configurer des alertes sur un large éventail de ressources.
- Utilisez [Azure Policy](../governance/policy/overview.md) pour appliquer différentes règles sur vos ressources. Vous avez ainsi l’assurance que vos ressources demeurent en conformité avec les standards et contrats de niveau de service de votre entreprise. 


##  <a name="combine-resources-in-azure-resource-groups"></a>Combiner des ressources dans des groupes de ressources Azure
Aujourd’hui, une application classique sur Azure comprend plusieurs ressources, telles que des machines virtuelles et App Services, ou des microservices hébergés sur des services cloud, des clusters AKS ou Service Fabric. Ces applications utilisent fréquemment des dépendances, comme Event Hubs, le stockage, SQL et Service Bus.

- Combinez les ressources dans des groupes de ressources Azure pour obtenir une visibilité totale sur toutes les ressources qui composent vos différentes applications. [Azure Monitor pour les groupes de ressources](./insights/resource-group-insights.md) fournit un moyen simple d’effectuer le suivi de l’intégrité et des performances de votre application de pile complète toute entière, et vous permet d’explorer en détail les composants respectifs pour tout débogage ou toute investigation.

## <a name="ensure-quality-through-continuous-deployment"></a>Garantir la qualité par l’intermédiaire du déploiement continu
L’intégration continue / le déploiement continu vous donne la possibilité d’intégrer et de déployer automatiquement les modifications de code à votre application en fonction des résultats de tests automatisés. Cette solution simplifie le processus de déploiement et garantit la qualité de toutes les modifications avant leur départ en production.


- Utilisez [Azure Pipelines](/azure/devops/pipelines) pour implémenter un déploiement continu et automatiser votre processus en entier, de la validation de code à la production, selon vos tests CI/CD.
- Utilisez [Quality Gates](/azure/devops/pipelines/release/approvals/gates) pour intégrer la supervision à votre prédéploiement ou post-déploiement. Vous êtes ainsi assuré de respecter les métriques d’intégrité/de performances clés (KPI) tandis que vos applications passent du développement en production, et qu’aucune différence dans la mise à l’échelle ou l’environnement d’infrastructure n’a d’impact négatif sur vos KPI.
- [Tenez à jour des instances de supervision distinctes](./app/separate-resources.md) entre vos différents environnements de déploiement, tels que le développement, les tests, le contrôle de validité et la production. Les données collectées sont ainsi pertinentes sur l’ensemble de l’infrastructure et des applications associées. Si vous devez mettre en corrélation des données entre différents environnements, vous pouvez utiliser des [graphiques multi-ressources dans Metrics Explorer](./platform/metrics-charts.md) ou créer des [requêtes inter-ressources dans Azure Monitor](log-query/cross-workspace-query.md).


## <a name="create-actionable-alerts-with-actions"></a>Créer des alertes actionnables avec des actions
Un aspect essentiel de la supervision est la notification des administrateurs à un stade précoce des problèmes actuels et prédits. 

- Créez des [alertes dans Azure Monitor](./platform/alerts-overview.md) en fonction des journaux d’activité et des métriques pour identifier les états d’échec prédictibles. Votre objectif doit être de rendre toutes les alertes actionnables, ce qui implique qu’elles représentent des conditions critiques réelles et qu’elles cherchent à réduire les faux positifs. Utilisez des [seuils dynamiques](platform/alerts-dynamic-thresholds.md) pour calculer automatiquement les bases de référence sur les données de métriques plutôt que de définir vos propres seuils statiques. 
- Définissez des actions pour que les alertes utilisent le moyen le plus efficace d’informer vos administrateurs. Les [actions disponibles pour les notifications](platform/action-groups.md#create-an-action-group-by-using-the-azure-portal) sont les SMS, e-mails, notifications Push ou appels vocaux.
- Utilisez des actions plus élaborées pour vous [connecter à votre outil ITSM](platform/itsmc-overview.md) ou à d’autres systèmes de gestion d’alertes au moyen de [webhooks](platform/activity-log-alerts-webhook.md).
- Corrigez les situations identifiées dans les alertes, aussi bien avec les [runbooks Azure Automation](../automation/automation-webhooks.md) qu’avec [Logic Apps](/connectors/custom-connectors/create-webhook-trigger) qui peut être lancé à partir d’une alerte avec des webhooks. 
- Utilisez la [mise à l’échelle automatique](./learn/tutorial-autoscale-performance-schedule.md) pour augmenter et diminuer dynamiquement vos ressources de calcul en fonction des métriques collectés.

## <a name="prepare-dashboards-and-workbooks"></a>Préparer des tableaux de bord et des classeurs
Le fait de veiller à ce que votre développement et vos opérations aient accès à la même télémétrie et aux mêmes outils leur permet d’afficher des modèles dans tout votre environnement, et de réduire les temps de détection moyen (MTTD) et temps de restauration moyen (MTTR).

- Préparez des [tableaux de bord personnalisés](./learn/tutorial-app-dashboards.md) en fonction des journaux d’activité et métriques communs des différents rôles de votre organisation. Les tableaux de bord peuvent combiner des données provenant de toutes les ressources Azure.
- Préparez des [classeurs](./platform/workbooks-overview.md) pour garantir le partage des connaissances entre le développement et les opérations. Ils peuvent être organisés sous forme de rapports dynamiques contenant des graphiques de métrique et des requêtes de journal, ou même sous forme de guides de résolution des problèmes, conçus par des développeurs aidant le support technique ou les opérations à régler des problèmes de base.

## <a name="continuously-optimize"></a>Optimiser en continu
 La supervision est un des aspects fondamentaux de la philosophie très répandue Construire-Mesurer-Apprendre ; elle recommande le suivi en continu des KPI et des métriques de comportement d’utilisateur, pour ensuite s’efforcer de les optimiser par la planification d’itérations. Azure Monitor vous permet de collecter des journaux d’activité et des métriques en rapport avec votre activité, et d’ajouter de nouveaux points de données dans le déploiement suivant, en fonction des besoins.

- Utilisez les outils dans Application Insights pour [suivre le comportement et l’engagement des utilisateurs finaux](./learn/tutorial-users.md).
- Utilisez [Analyse d’impact](./app/usage-impact.md) pour vous aider à classer par ordre de priorité les zones sur lesquelles se concentrer pour mener à d’importants KPI.


## <a name="next-steps"></a>Étapes suivantes

- Découvrir les différents composants d’[Azure Monitor](overview.md).
- [Ajouter la supervision continue](./app/continuous-monitoring.md) à votre pipeline de mise en production.