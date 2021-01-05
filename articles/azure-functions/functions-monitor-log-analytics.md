---
title: Monitorage d’Azure Functions avec Azure Monitor Logs
description: Découvrez comment utiliser Azure Monitor Logs avec Azure Functions pour surveiller les exécutions de fonctions.
author: craigshoemaker
ms.topic: conceptual
ms.date: 04/15/2020
ms.author: cshoe
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: 582dafba40012e9ff9c59bc09adb1a0831e999f5
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94491220"
---
# <a name="monitoring-azure-functions-with-azure-monitor-logs"></a>Monitorage d’Azure Functions avec Azure Monitor Logs

Azure Functions s’intègre à [Azure Monitor Logs](../azure-monitor/platform/data-platform-logs.md) pour effectuer le monitorage des fonctions. Cet article explique comment configurer Azure Functions de façon à envoyer des journaux générés par le système et par l’utilisateur à Azure Monitor Logs.

Azure Monitor Logs offre la possibilité de consolider les journaux de différentes ressources dans le même espace de travail, où ils peuvent être analysés avec des [requêtes](../azure-monitor/log-query/log-query-overview.md) pour récupérer, consolider et analyser rapidement les données collectées.  Vous pouvez créer et tester des requêtes à l’aide de [Log Analytics](../azure-monitor/log-query/log-query-overview.md) dans le Portail Azure, avant d’analyser directement les données à l’aide de ces outils ou d’enregistrer les requêtes pour les utiliser pour les [visualisations](../azure-monitor/visualizations.md) ou les [règles d’alerte](../azure-monitor/platform/alerts-overview.md).

Azure Monitor utilise une version du [langage de requête Kusto](/azure/kusto/query/) utilisé par Azure Data Explorer qui est adapté aux requêtes simples dans les journaux, mais inclut également des fonctionnalités avancées telles que les agrégations, les jointures et les analyses intelligentes. Il existe [plusieurs leçons](../azure-monitor/log-query/get-started-queries.md) pour vous aider à apprendre le langage de requête.

> [!NOTE]
> L’intégration avec Azure Monitor Logs est actuellement en préversion publique pour les applications de fonction v2 et v3 qui s’exécutent sur des plans d’hébergement Windows Consumption, Premium et Dedicated.

## <a name="setting-up"></a>Configuration

1. Dans la section de **surveillance** de votre application de fonction dans le [Portail Azure](https://portal.azure.com), sélectionnez **Paramètres de diagnostic**, puis sélectionnez **Ajouter un paramètre de diagnostic**.

   :::image type="content" source="media/functions-monitor-log-analytics/diagnostic-settings-add.png" alt-text="Sélectionner Paramètres de diagnostic":::

1. Dans la page des **paramètres de diagnostic**, sous **Détails de la catégorie** et **Journal**, choisissez **FunctionAppLogs**.

   Le tableau **FunctionAppLogs** contient les journaux souhaités.

1. Sous **Détails de la destination**, choisissez **Envoyer à Log Analytics**, puis sélectionnez votre **espace de travail Log Analytics**. 

1. Entrez un **nom de paramètres de diagnostic**, puis sélectionnez **Enregistrer**.

   :::image type="content" source="media/functions-monitor-log-analytics/choose-table.png" alt-text="Ajouter un paramètre de diagnostic":::

## <a name="user-generated-logs"></a>Journaux générés par l’utilisateur

Pour générer des journaux personnalisés, utilisez l’instruction de journalisation propre à votre langage. Voici des exemples d’extraits de code :


# <a name="c"></a>[C#](#tab/csharp)

```csharp
log.LogInformation("My app logs here.");
```

# <a name="java"></a>[Java](#tab/java)

```java
context.getLogger().info("My app logs here.");
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
context.log('My app logs here.');
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
Write-Host "My app logs here."
```

# <a name="python"></a>[Python](#tab/python)

```python
logging.info('My app logs here.')
```

---

## <a name="querying-the-logs"></a>Interroger les journaux

Pour interroger les journaux générés :
 
1. Dans votre application de fonction, **Paramètres de diagnostic**. 

1. Dans la liste **Paramètres de diagnostic**, sélectionnez l’espace de travail Log Analytics que vous avez configuré pour lui envoyer les journaux de fonction. 

1. Sur la page **Espaces de travail Log Analytics**, sélectionnez **Journaux**.

   Azure Functions écrit tous les journaux dans la table **FunctionAppLogs** sous **LogManagement**. 

   :::image type="content" source="media/functions-monitor-log-analytics/querying.png" alt-text="Fenêtre de requête dans l’espace de travail Log Analytics":::

Voici quelques exemples de requêtes :

### <a name="all-logs"></a>Tous les journaux d’activité

```

FunctionAppLogs
| order by TimeGenerated desc

```

### <a name="specific-function-logs"></a>Journaux d’une fonction spécifique

```

FunctionAppLogs
| where FunctionName == "<Function name>" 

```

### <a name="exceptions"></a>Exceptions

```

FunctionAppLogs
| where ExceptionDetails != ""  
| order by TimeGenerated asc

```

## <a name="next-steps"></a>Étapes suivantes

- Consultez la [vue d’ensemble d’Azure Functions](functions-overview.md)
- En savoir plus sur les [Journaux Azure Monitor](../azure-monitor/platform/data-platform-logs.md).
- Découvrez plus en détails le [langage de requête](../azure-monitor/log-query/get-started-queries.md).
