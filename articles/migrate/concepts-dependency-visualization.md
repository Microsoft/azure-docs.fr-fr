---
title: Analyse des dépendances dans l’outil d’évaluation de serveur Azure Migrate
description: Décrit comment utiliser l’analyse des dépendances pour l’évaluation à l’aide de l’outil d’évaluation de serveur Azure Migrate.
ms.topic: conceptual
author: vineetvikram
ms.author: vivikram
ms.manager: abhemraj
ms.date: 09/15/2020
ms.openlocfilehash: 1f198d47191e7893e74b072ae8fd10546e3a6ee7
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96752207"
---
# <a name="dependency-analysis"></a>Analyse des dépendances

Cet article décrit l’analyse des dépendances dans l’outil Azure Migrate : évaluation de serveur.


L’analyse des dépendances identifie les dépendances entre les machines locales découvertes. Il offre les avantages suivants : 

- Vous pouvez rassembler des machines dans des groupes à des fins d’évaluation, de manière plus précise, avec une plus grande confiance.
- Vous pouvez identifier les machines qui doivent être migrées ensemble. Cela est particulièrement utile quand vous ne savez pas avec certitude quelles machines font partie d’un déploiement d’application que vous souhaitez faire migrer vers Azure.
- Vous pouvez déterminer si les machines sont en cours d’utilisation, ou lesquelles peuvent être désactivées au lieu d’être migrées.
- L’analyse des dépendances permet de s’assurer que rien n’est laissé de côté et d’éviter ainsi les pannes inopinées après la migration.
- [Passez en revue les questions courantes sur l’analyse des dépendances](common-questions-discovery-assessment.md#what-is-dependency-visualization).


## <a name="analysis-types"></a>Types d’analyse

Deux options permettent de déployer l’analyse des dépendances.

**Option** | **Détails** | **Cloud public** | **Azure Government**
----  |---- | ---- 
**Sans agent** | Interroge les données des machines virtuelles VMware à l’aide des API vSphere.<br/><br/> Vous n’avez pas besoin d’installer l’agent sur des machines virtuelles.<br/><br/> Cette option est en préversion pour les machines virtuelles VMware. | Pris en charge. | Pris en charge.
**Analyse basée sur les agents** | Utilise la [solution Service Map](../azure-monitor/insights/service-map.md) dans Azure Monitor pour activer la visualisation et l’analyse des dépendances.<br/><br/> Vous devez installer des agents sur chaque machine locale à analyser. | Prise en charge | Non pris en charge.


## <a name="agentless-analysis"></a>Analyse sans agent

L’analyse des dépendances sans agent fonctionne en capturant les données de connexion TCP des machines pour lesquelles elle est activée. Aucun agent n’est installé sur les machines virtuelles. Les connexions avec les mêmes serveur et processus source, ainsi que le serveur de destination, le processus et le port sont regroupés de façon logique dans une dépendance. Vous pouvez visualiser les données de dépendance capturées dans une vue cartographique ou les exporter au format CSV. Aucun agent n’est installé sur les machines à analyser.

### <a name="dependency-data"></a>Données de dépendance

Une fois la découverte des données de dépendance lancée, l’interrogation commence :

- L’appliance Azure Migrate interroge les données de connexion TCP des machines toutes les cinq minutes pour les rassembler.
- Les données sont collectées à partir de machines virtuelles invitées par le biais de vCenter Server, à l’aide d’API vSphere.
- L’interrogation rassemble ces données :

    - Nom des processus ayant des connexions actives
    - Nom de l’application qui exécute des processus qui ont des connexions actives
    - Port de destination sur les connexions actives

- Les données collectées sont traitées sur l’appliance Azure Migrate pour déduire les informations d’identité et sont envoyées à Azure Migrate toutes les six heures


## <a name="agent-based-analysis"></a>Analyse basée sur les agents

Pour l’analyse basée sur les agents, l’outil d’évaluation de serveur utilise la solution [Service Map](../azure-monitor/insights/service-map.md) dans Azure Monitor. Vous installez l’agent [Microsoft Monitoring Agent/Log Analytics](../azure-monitor/platform/agents-overview.md#log-analytics-agent) et l’[agent de dépendances](../azure-monitor/platform/agents-overview.md#dependency-agent) doit être installé sur chaque machine que vous souhaitez analyser.

### <a name="dependency-data"></a>Données de dépendance

L’analyse basée sur les agents fournit les données suivantes :

- Nom du serveur de l’ordinateur source, processus, nom de l’application.
- Nom du serveur de l’ordinateur de destination, processus, nom de l’application et port.
- Le nombre de connexions, la latence et les informations de transfert de données sont collectés et mis à la disposition des requêtes Log Analytics. 



## <a name="compare-agentless-and-agent-based"></a>Comparaison entre la visualisation sans agent et la visualisation basée sur les agents

Les différences entre la visualisation sans agent et la visualisation basée sur les agents sont résumées dans le tableau.

**Prérequis** | **Sans agent** | **Basé sur un agent**
--- | --- | ---
**Support** | En version préliminaire pour les machines virtuelles VMware uniquement. [Vérifiez](migrate-support-matrix-vmware.md#dependency-analysis-requirements-agentless) quels sont les systèmes d’exploitation pris en charge. | En disponibilité générale.
**Agent** | Aucun agent nécessaire sur les machines à analyser. | Agents requis sur chaque machine locale à analyser.
**Log Analytics** | Non requis. | Azure Migrate utilise la solution [Service Map](../azure-monitor/insights/service-map.md) dans les [journaux Azure Monitor](../azure-monitor/log-query/log-query-overview.md) pour l’analyse des dépendances.<br/><br/> Vous associez un espace de travail Log Analytics à un projet Azure Migrate. L’espace de travail doit résider dans les régions USA Est, Asie Sud-Est ou Europe Ouest. L’espace de travail doit se trouver au sein d’une région dans laquelle [Service Map est pris en charge](../azure-monitor/insights/vminsights-configure-workspace.md#supported-regions).
**Processus** | Capture les données de connexion TCP. Après la découverte, il collecte des données toutes les cinq minutes. | Les agents Service Map installés sur un ordinateur collectent des données sur les processus TCP et sur les connexions entrantes/sortantes pour chaque processus.
**Données** | Nom du serveur de l’ordinateur source, processus, nom de l’application.<br/><br/> Nom du serveur de l’ordinateur de destination, processus, nom de l’application et port. | Nom du serveur de l’ordinateur source, processus, nom de l’application.<br/><br/> Nom du serveur de l’ordinateur de destination, processus, nom de l’application et port.<br/><br/> Le nombre de connexions, la latence et les informations de transfert de données sont collectés et mis à la disposition des requêtes Log Analytics. 
**Visualisation** | La carte des dépendances d’un serveur unique peut être affichée sur une durée allant d’une heure à 30 jours. | Carte des dépendances d’un serveur unique.<br/><br/> Carte des dépendances d’un groupe de serveurs.<br/><br/>  La carte peut être affichée pendant une heure uniquement.<br/><br/> Ajoutez et supprimez des serveurs dans un groupe à partir de la vue cartographique.
Exportation de données | Les données des 30 derniers jours peuvent être téléchargées dans un format CSV. | Les données peuvent être interrogées avec Log Analytics.



## <a name="next-steps"></a>Étapes suivantes

- [Configurez](how-to-create-group-machine-dependencies.md) la visualisation des dépendances basée sur les agents.
- [Essayez](how-to-create-group-machine-dependencies-agentless.md) la visualisation des dépendances sans agent pour les machines virtuelles VMware.
- Passer en revue les [questions courantes](common-questions-discovery-assessment.md#what-is-dependency-visualization) sur la visualisation des dépendances
