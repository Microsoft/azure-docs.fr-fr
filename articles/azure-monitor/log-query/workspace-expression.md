---
title: Expression workspace() dans la requête de journal Azure Monitor | Microsoft Docs
description: L’expression workspace est utilisée dans une requête de journal Azure Monitor dans le but de récupérer des données à partir d’un espace de travail spécifique du même groupe de ressources, d’un autre groupe de ressources ou d’un autre abonnement.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 09/10/2018
ms.openlocfilehash: 255888acf5da6149b6a964b23ed038b99715481c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96014038"
---
# <a name="workspace-expression-in-azure-monitor-log-query"></a>Expression workspace() dans la requête de journal Azure Monitor

L’expression `workspace` est utilisée dans une requête Azure Monitor dans le but de récupérer des données à partir d’un espace de travail spécifique du même groupe de ressources, d’un autre groupe de ressources ou d’un autre abonnement. Elle est particulièrement utile pour inclure des données de journal dans une requête Application Insights et pour interroger des données de plusieurs espaces de travail dans une requête de journal.


## <a name="syntax"></a>Syntaxe

`workspace(`*Identificateur*`)`

## <a name="arguments"></a>Arguments

- L’*identificateur* permet d’identifier l’espace de travail à l’aide de l’un des formats du tableau ci-dessous.

| Identificateur | Description | Exemple
|:---|:---|:---|
| Nom de la ressource | Nom lisible de l’espace de travail (également appelé « nom du composant ») | workspace("contosoretail") |
| Nom qualifié | Nom complet de l’espace de travail au format : « nom_abonnement/groupe_ressources/nom_composant » | workspace('Contoso/ContosoResource/ContosoWorkspace') |
| id | GUID de l’espace de travail | workspace("b438b3f6-912a-46d5-9db1-b42069242ab4") |
| ID de la ressource Azure | Identificateur de la ressource Azure | workspace("/subscriptions/e4227-645-44e-9c67-3b84b5982/resourcegroups/ContosoAzureHQ/providers/Microsoft.OperationalInsights/workspaces/contosoretail") |


## <a name="notes"></a>Notes

* Vous devez disposer d’un accès en lecture à l’espace de travail.
* L’expression `app` associée vous permet d’effectuer des requêtes sur les applications Application Insights.

## <a name="examples"></a>Exemples

```Kusto
workspace("contosoretail").Update | count
```
```Kusto
workspace("b438b4f6-912a-46d5-9cb1-b44069212ab4").Update | count
```
```Kusto
workspace("/subscriptions/e427267-5645-4c4e-9c67-3b84b59a6982/resourcegroups/ContosoAzureHQ/providers/Microsoft.OperationalInsights/workspaces/contosoretail").Event | count
```
```Kusto
union 
(workspace("myworkspace").Heartbeat | where Computer contains "Con"),
(app("myapplication").requests | where cloud_RoleInstance contains "Con")
| count  
```
```Kusto
union 
(workspace("myworkspace").Heartbeat), (app("myapplication").requests)
| where TimeGenerated between(todatetime("2018-02-08 15:00:00") .. todatetime("2018-12-08 15:05:00"))
```

## <a name="next-steps"></a>Étapes suivantes

- Consultez l’article relatif à l’[expression app](app-expression.md) pour en savoir plus sur la façon de faire référence à une application Application Insights.
- Découvrez-en plus sur le stockage des [données Azure Monitor](log-query-overview.md).
- Accédez à la documentation complète sur le [langage de requête Kusto](/azure/kusto/query/).
