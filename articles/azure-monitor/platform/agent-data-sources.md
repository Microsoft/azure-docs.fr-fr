---
title: Sources de données d’agent Log Analytics dans Azure Monitor
description: Les sources de données définissent les données de journal qu’Azure Monitor collecte auprès des agents et d’autres sources connectées.  Cet article décrit la façon dont Azure Monitor utilise les sources de données, explique en détail comment les configurer, et fournit un résumé des différentes sources de données disponibles.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 10/21/2020
ms.openlocfilehash: ad5e91a6dcdb61e09a64e61a27f12148ec28168e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000740"
---
# <a name="log-analytics-agent-data-sources-in-azure-monitor"></a>Sources de données d’agent Log Analytics dans Azure Monitor
Les données qu’Azure Monitor collecte à partir des machines virtuelles avec l’[agent Log Analytics](log-analytics-agent.md) sont définies par les sources de données que vous configurez dans l’[espace de travail Log Analytics](data-platform-logs.md).   Chaque source de données crée des enregistrements d'un type particulier, chaque type ayant son propre ensemble de propriétés.

> [!IMPORTANT]
> Cet article traite des sources de données pour l’[agent Log Analytics](log-analytics-agent.md), qui est un des agents utilisés par Azure Monitor. D’autres agents collectent des données différentes et sont configurés différemment. Pour obtenir la liste des agents disponibles et les données qu’ils peuvent collecter, consultez [Vue d’ensemble des agents Azure Monitor](agents-overview.md).

![Collecte de données de journal](media/agent-data-sources/overview.png)

> [!IMPORTANT]
> Les sources de données décrites dans cet article s’appliquent uniquement aux machines virtuelles qui exécutent l’agent Log Analytics. 

## <a name="summary-of-data-sources"></a>Résumé des sources de données
Le tableau suivant répertorie les sources de données des agents actuellement disponibles avec l’agent Log Analytics.  Chacune de ces sources comporte un lien vers un article distinct qui fournit des détails sur cette source de données.   Il fournit également des informations sur leur méthode et leur fréquence de collecte. 


| Source de données | Plateforme | Agent Log Analytics | Agent Operations Manager | Stockage Azure | Operations Manager requis ? | Données de l’agent Operations Manager envoyées via un groupe d’administration | Fréquence de collecte |
| --- | --- | --- | --- | --- | --- | --- | --- |
| [Journaux d’activité personnalisés](data-sources-custom-logs.md) | Windows |&#8226; |  | |  |  | à l'arrivée |
| [Journaux d’activité personnalisés](data-sources-custom-logs.md) | Linux   |&#8226; |  | |  |  | à l'arrivée |
| [Journaux d’activité IIS](data-sources-iis-logs.md) | Windows |&#8226; |&#8226; |&#8226; |  |  |Dépend du paramètre Substitution de fichier journal |
| [Compteurs de performance](data-sources-performance-counters.md) | Windows |&#8226; |&#8226; |  |  |  |comme prévu, minimum de 10 secondes |
| [Compteurs de performance](data-sources-performance-counters.md) | Linux |&#8226; |  |  |  |  |comme prévu, minimum de 10 secondes |
| [Syslog](data-sources-syslog.md) | Linux |&#8226; |  |  |  |  |depuis le stockage Azure : 10 minutes ; à partir de l’agent : à l’arrivée |
| [Journaux d’événements Windows](data-sources-windows-events.md) |Windows |&#8226; |&#8226; |&#8226; |  |&#8226; | à l'arrivée |


## <a name="configuring-data-sources"></a>Configuration des sources de données
Pour configurer des sources de données pour les agents Log Analytics, accédez au menu **Espaces de travail Log Analytics** dans le portail Azure et sélectionnez un espace de travail. Cliquez sur **Paramètres avancés** puis sur **Données**. Sélectionnez la source de données à configurer. Vous pouvez suivre le lien dans le tableau ci-dessus vers la documentation de chaque source de données pour plus d’informations sur leur configuration.

Toutes les configurations sont remises à tous les agents connectés à cet espace de travail.  Nous ne pouvez exclure aucun agent connecté de cette configuration.

![Configurer les événements Windows](media/agent-data-sources/configure-events.png)



## <a name="data-collection"></a>Collecte de données
Les configurations des sources de données sont remises en quelques minutes aux agents directement connectés à Azure Monitor.  Les données spécifiées sont collectées à partir de l’agent et remises directement à Azure Monitor à des intervalles spécifiques pour chaque source de données.  Consultez la documentation de chaque source de données pour obtenir ces informations spécifiques.

Pour les agents System Center Operations Manager d’un groupe d'administration connecté, les configurations de sources de données sont traduites en packs d'administration et remises au groupe d'administration toutes les 5 minutes par défaut.  L’agent télécharge le pack d’administration comme tout autre, et collecte les données spécifiées. Selon la source de données, soit les données sont envoyées à un serveur d’administration qui les transfère à Azure Monitor, soit l’agent les envoie à Azure Monitor sans passer par le serveur de gestion. Pour plus d’informations, consultez [Data collection details for monitoring solutions in Azure](../monitor-reference.md) (Détails sur la collecte de données pour les solutions de supervision dans Azure).  Vous pouvez consulter les détails de la connexion à Operations Manager et à Azure Monitor et de la modification de la fréquence à laquelle la configuration est remise dans la rubrique [Configurer l’intégration avec System Center Operations Manager](om-agents.md).

Si l’agent ne peut pas se connecter à Azure Monitor ou à Operations Manager, il continue à collecter des données qu’il fournira lorsqu’une connexion sera établie.  Les données peuvent être perdues si le volume de données atteint la taille maximale du cache du client ou si l’agent n’est pas en mesure d’établir une connexion dans un délai de 24 heures.

## <a name="log-records"></a>Enregistrement de journaux
Toutes les données de journaux collectées par Azure Monitor sont stockées dans l’espace de travail en tant qu’enregistrements.  Les enregistrements collectés par différentes sources de données auront leur propre jeu de propriétés et seront identifiés par leur propriété **Type** .  Consultez la documentation de chaque source de données et solution pour plus d'informations sur chaque type d'enregistrement.

## <a name="next-steps"></a>Étapes suivantes
* Découvrez les [solutions de supervision](../insights/solutions.md) qui ajoutent des fonctionnalités à Azure Monitor et collectent des données dans l’espace de travail.
* Découvrez les [requêtes de journal](../log-query/log-query-overview.md) pour analyser les données collectées à partir de sources de données et de solutions de supervision.  
* Configurez des [alertes](alerts-overview.md) qui vous informeront de façon proactive des données critiques collectées à partir de sources de données et de solutions de supervision.
