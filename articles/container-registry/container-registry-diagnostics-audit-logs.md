---
title: Collecter et analyser les journaux de ressources
description: Enregistrez et analysez les événements des journaux des ressources pour Azure Container Registry tels que l’authentification, l’envoi (push) d’images et le tirage (pull) d’images.
ms.topic: article
ms.date: 06/01/2020
ms.openlocfilehash: 8b05d34e6c50fea3760e30d28f59e55d8c5f211a
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96348583"
---
# <a name="azure-container-registry-logs-for-diagnostic-evaluation-and-auditing"></a>Journaux d’Azure Container Registry pour l’évaluation et l’audit de diagnostics

Cet article explique comment collecter des données de journal pour un registre de conteneurs Azure à l’aide des fonctionnalités d’[Azure Monitor](../azure-monitor/overview.md). Azure Monitor collecte les [journaux de ressources](../azure-monitor/platform/platform-logs-overview.md) (anciennement appelés *journaux de diagnostic*) pour les événements pilotés par l’utilisateur dans votre registre. Collectez et utilisez ces données pour répondre à des besoins tels que :

* Auditer les événements d’authentification du registre pour garantir la sécurité et la conformité 

* Fournir une trace d’activité complète sur les artefacts du registre, tels que les événements d’envoi (push) et de tirage (pull), pour vous permettre de diagnostiquer les problèmes opérationnels avec votre registre 

La collecte de données de journal de ressources à l’aide d’Azure Monitor peut entraîner des coûts supplémentaires. Consultez [Tarification Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/). 

## <a name="repository-events"></a>Événements de dépôt

Les événements suivants au niveau du référentiel pour les images et autres artefacts sont actuellement journalisés :

* **Envoi (push)**
* **Tirage (pull)**
* **Suppression de balise**
* **Suppression** (y compris les événements de suppression de référentiel)
* **Vidage de balise** et **Vider de manifeste**

> [!NOTE]
> Les événements de vidage sont journalisés uniquement si une [stratégie de rétention](container-registry-retention-policy.md) du registre est configurée.

## <a name="registry-resource-logs"></a>Journaux de ressources de registre

Les journaux de ressource contiennent des informations émises par des ressources Azure qui décrivent leur fonctionnement interne. Pour un registre de conteneurs Azure, les journaux contiennent des événements d’authentification et de niveau de référentiel stockés dans les tableaux suivants. 

* **ContainerRegistryLoginEvents** - Événements et état d’authentification du registre, y compris l’identité entrante et l’adresse IP
* **ContainerRegistryRepositoryEvents** - Opérations telles que « push and pull » (envoyer et tirer) pour les images et autres artefacts dans les référentiels de registre
* **AzureMetrics** - [Métriques du registre des conteneurs](../azure-monitor/platform/metrics-supported.md#microsoftcontainerregistryregistries), tels que le nombre agrégé d’opérations d’envoi (push) et de tirage (pull).

Pour les opérations, les données de journal incluent :
  * L’état de réussite ou d’échec
  * Les horodatages de début et de fin

Outre les journaux de ressources, Azure fournit un [journal d’activité](../azure-monitor/platform/platform-logs-overview.md), un enregistrement unique d’événements de gestion Azure au niveau de l’abonnement, tels que la création ou la suppression d’un registre de conteneurs.

## <a name="enable-collection-of-resource-logs"></a>Permet la collecte des journaux de ressources

La collecte des journaux de ressources pour un registre de conteneurs n’est pas activée par défaut. Activez explicitement les paramètres de diagnostic pour chaque registre à surveiller. Pour les options permettant d’activer les paramètre de diagnostic, consultez [Créer un paramètre de diagnostic pour collecter des journaux et métriques de plateforme dans Azure](../azure-monitor/platform/diagnostic-settings.md).

Par exemple, pour afficher les journaux et les métriques d’un registre de conteneurs quasiment en temps réel dans Azure Monitor, collectez les journaux des ressources dans un espace de travail Log Analytics. Pour activer ce paramètre de diagnostic à l’aide du Portail Azure :

1. Si vous n’avez pas encore d’espace de travail, créez-en un à l’aide du [Portail Azure](../azure-monitor/learn/quick-create-workspace.md). Pour réduire la latence lors de la collecte de données, assurez-vous que l’espace de travail se trouve dans la **même région** que votre registre de conteneurs.
1. Dans le portail, sélectionnez le registre, puis sélectionnez **Surveillance > Paramètres de diagnostic > Ajouter un paramètre de diagnostic**.
1. Entrez un nom pour le paramètre, puis sélectionnez **Envoyer à Log Analytics**.
1. Sélectionnez l’espace de travail pour les journaux de diagnostic du registre.
1. Sélectionnez les données de journal que vous souhaitez collecter, puis cliquez sur **Enregistrer**.

L’illustration suivante montre la création d’un paramètre de diagnostic pour un registre à l’aide du portail.

![Activer les paramètres de diagnostic](media/container-registry-diagnostics-audit-logs/diagnostic-settings.png)

> [!TIP]
> Collectez uniquement les données dont vous avez besoin, en équilibrant les coûts et vos besoins de surveillance. Par exemple, si vous devez uniquement auditer les événements d’authentification, sélectionnez uniquement le journal **ContainerRegistryLoginEvents**. 

## <a name="view-data-in-azure-monitor"></a>Afficher des données dans Azure Monitor

Une fois que vous avez activé la collecte des journaux de diagnostic dans Log Analytics, quelques minutes peuvent être nécessaires pour que les données s’affichent dans Azure Monitor. Pour afficher les données dans le portail, sélectionnez le registre, puis sélectionnez **Surveillance > Journaux**. Sélectionnez l’une des tables qui contient les données du registre. 

Exécutez des requêtes pour afficher les données. Plusieurs exemples de requêtes sont fournis, ou exécutez les vôtres. Par exemple, la requête suivante récupère les 24 heures de données les plus récentes à partir de la table **ContainerRegistryRepositoryEvents** :

```Kusto
ContainerRegistryRepositoryEvents
| where TimeGenerated > ago(1d) 
```

L’image suivante présente un exemple de sortie :

![Interroger des données de journal](media/container-registry-diagnostics-audit-logs/azure-monitor-query.png)

Pour obtenir un didacticiel sur l’utilisation de Log Analytics dans le Portail Azure, consultez [Bien démarrer avec Azure Monitor Log Analytics](../azure-monitor/log-query/log-analytics-tutorial.md), ou essayez l’[Environnement de démonstration](https://portal.loganalytics.io/demo) Log Analytics. 

Pour plus d’informations sur les requêtes de journal, consultez [Vue d’ensemble des requêtes de journal dans Azure Monitor](../azure-monitor/log-query/log-query-overview.md).

## <a name="query-examples"></a>Exemples de requêtes

### <a name="error-events-from-the-last-hour"></a>Événements d’erreur au cours de la dernière heure

```Kusto
union Event, Syslog // Event table stores Windows event records, Syslog stores Linux records
| where TimeGenerated > ago(1h)
| where EventLevelName == "Error" // EventLevelName is used in the Event (Windows) records
    or SeverityLevel== "err" // SeverityLevel is used in Syslog (Linux) records
```

### <a name="100-most-recent-registry-events"></a>Les 100 événements de registre les plus récents

```Kusto
ContainerRegistryRepositoryEvents
| union ContainerRegistryLoginEvents
| top 100 by TimeGenerated
| project TimeGenerated, LoginServer, OperationName, Identity, Repository, DurationMs, Region , ResultType
```

### <a name="identity-of-user-or-object-that-deleted-repository"></a>Identité de l’utilisateur ou de l’objet qui a supprimé le référentiel

```Kusto
ContainerRegistryRepositoryEvents
| where OperationName contains "Delete"
| project LoginServer, OperationName, Repository, Identity, CallerIpAddress
```

### <a name="identity-of-user-or-object-that-deleted-tag"></a>Identité de l’utilisateur ou de l’objet qui a supprimé la balise

```Kusto
ContainerRegistryRepositoryEvents
| where OperationName contains "Untag"
| project LoginServer, OperationName, Repository, Tag, Identity, CallerIpAddress
```

### <a name="repository-level-operation-failures"></a>Échecs d’opérations au niveau du référentiel

```kusto
ContainerRegistryRepositoryEvents 
| where ResultDescription contains "40"
| project TimeGenerated, OperationName, Repository, Tag, ResultDescription
```

### <a name="registry-authentication-failures"></a>Échecs d’authentification du registre

```kusto
ContainerRegistryLoginEvents 
| where ResultDescription != "200"
| project TimeGenerated, Identity, CallerIpAddress, ResultDescription
```


## <a name="additional-log-destinations"></a>Destinations de journal supplémentaires

En plus d’envoyer les journaux à Log Analytics, ou comme alternative, un scénario courant consiste à sélectionner un compte de stockage Azure comme destination du journal. Pour archiver les journaux dans le stockage Azure, créez un compte de stockage avant d’activer l’archivage via les paramètres de diagnostic.

Vous pouvez également diffuser en continu les événements du journal de diagnostic vers un service [Azure Event Hub](../event-hubs/event-hubs-about.md). Le service Event Hubs peut traiter à chaque seconde des millions d’événements que vous pouvez transformer et stocker à l’aide de tout fournisseur d’analyses en temps réel. 

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur l’utilisation de [Log Analytics](../azure-monitor/log-query/log-analytics-tutorial.md) et la création de [requêtes de journal](../azure-monitor/log-query/get-started-queries.md).
* Consultez [vue d’ensemble des journaux de plateforme Azure](../azure-monitor/platform/platform-logs-overview.md) pour en savoir plus sur les journaux de plateforme disponibles dans différentes couches d’Azure.