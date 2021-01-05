---
title: Gérer les requêtes de chasse et de diffusion en continu dans Azure Sentinel à l’aide de l’API REST | Microsoft Docs
description: Cet article explique comment les fonctionnalités de chasse Azure Sentinel vous permettent de tirer parti de l’API REST de Log Analytics pour gérer les requêtes de chasse et de diffusion en continu.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/06/2020
ms.author: yelevin
ms.openlocfilehash: cd52f22004bf72f3328d1e6a0d1ec988c2406317
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94660846"
---
# <a name="manage-hunting-and-livestream-queries-in-azure-sentinel-using-rest-api"></a>Gérer les requêtes de chasse et de diffusion en continu dans Azure Sentinel à l’aide de l’API REST

Azure Sentinel, construit en partie sur Azure Monitor Log Analytics, vous permet d’utiliser l’API REST de Log Analytics pour gérer les requêtes de chasse et de diffusion en continu. Ce document explique les procédures de création et de gestion des requêtes de chasse à l’aide de l’API REST.  Les requêtes créées de cette façon sont affichées dans l’interface utilisateur d’Azure Sentinel.

Consultez la référence de l’API REST définitive pour plus de détails sur les [API de recherche enregistrées](/rest/api/loganalytics/savedsearches).

## <a name="api-examples"></a>Exemples d'API

Dans les exemples suivants, remplacez ces espaces réservés par l’élément de remplacement prescrit dans le tableau suivant :

| Espace réservé | Remplacer par |
|-|-|
| **{subscriptionId}** | nom de l’abonnement auquel vous appliquez la requête de chasse ou de diffusion en continu. |
| **{resourceGroupName}** | nom du groupe de ressources auquel vous appliquez la requête de chasse ou de diffusion en continu. |
| **{savedSearchId}** | ID unique (GUID) pour chaque requête de chasse. |
| **{WorkspaceName}** | nom de l’espace de travail Log Analytics qui est la cible de la requête. |
| **{DisplayName}** | nom d'affichage de votre choix pour la requête. |
| **{Description}** | description de la requête de chasse ou de diffusion en continu. |
| **{Tactics}** | tactique MITRE ATT&CK pertinente qui s’applique à la requête. |
| **{Query}** | expression de requête pour votre requête. |
|  

### <a name="example-1"></a>Exemple 1

Cet exemple explique la procédure de création ou de mise à jour d’une requête de chasse pour un espace de travail Azure Sentinel donné.  Pour une requête de diffusion en continu, remplacez *« Catégorie » : « requêtes de chasse »* par *« Catégorie » : « requêtes de diffusion en continu »* dans le **corps de la demande** : 

#### <a name="request-header"></a>En-tête de requête

```http
PUT https://management.azure.com/subscriptions/{subscriptionId} _
    /resourcegroups/{resourceGroupName} _
    /providers/Microsoft.OperationalInsights/workspaces/{workspaceName} _
    /savedSearches/{savedSearchId}?api-version=2020-03-01-preview
```

#### <a name="request-body"></a>Corps de la demande

```json
{
"properties": {
    “Category”: “Hunting Queries”,
    "DisplayName": "HuntingRule02",
    "Query": "SecurityEvent | where EventID == \"4688\" | where CommandLine contains \"-noni -ep bypass $\"",
    “Tags”: [
        { 
        “Name”: “Description”,
        “Value”: “Test Hunting Query”
        },
        { 
        “Name”: “Tactics”,
        “Value”: “Execution, Discovery”
        }
        ]        
    }
}
```

### <a name="example-2"></a>Exemple 2

Cet exemple explique la procédure de suppression d’une requête de chasse ou de diffusion en continu pour un espace de travail Azure Sentinel donné :

```http
DELETE https://management.azure.com/subscriptions/{subscriptionId} _
       /resourcegroups/{resourceGroupName} _
       /providers/Microsoft.OperationalInsights/workspaces/{workspaceName} _
       /savedSearches/{savedSearchId}?api-version=2020-03-01-preview
```

### <a name="example-3"></a>Exemple 3

Cet exemple explique la procédure de récupération d’une requête de chasse ou de diffusion en continu pour un espace de travail donné :

```http
GET https://management.azure.com/subscriptions/{subscriptionId} _
    /resourcegroups/{resourceGroupName} _
    /providers/Microsoft.OperationalInsights/workspaces/{workspaceName} _
    /savedSearches/{savedSearchId}?api-version=2020-03-01-preview
```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à gérer les requêtes de chasse et de diffusion en continu dans Azure Sentinel à l’aide de l’API Log Analytics. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :

- [Hunt for threats with in Azure Sentinel Preview](hunting.md) (Rechercher des menaces dans Azure Sentinel en préversion)
- [Use notebooks to run automated hunting campaigns](notebooks.md) (Utiliser des notebooks pour exécuter des campagnes de repérage automatisées)