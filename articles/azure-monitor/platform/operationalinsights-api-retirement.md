---
title: Mise hors service de l’API Azure Monitor
description: Décrit la mise hors service des versions antérieures de l’API de fournisseur de ressources OperationalInsights.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 10/29/2020
ms.openlocfilehash: e2b12d7a2206ab369328563af438c6ef1ea39327
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96184108"
---
# <a name="operationalinsights-api-version-retirement"></a>Mise hors service de la version de l’API OperationalInsights
Microsoft envoie une notification au moins 12 mois avant la mise hors service d’une API pour faciliter la transition vers une version plus récente/prise en charge. Nous avons publié une nouvelle version (2020-08-01) des API de fournisseur de ressources **OperationalInsights** et nous mettront hors service toutes les versions antérieures des API le 29 février 2024.

Nous vous encourageons à utiliser la version 2020-08-01 dès à présent pour tirer parti des nouvelles fonctionnalités, telles que les [clusters dédiés](../log-query/logs-dedicated-clusters.md), les [clés gérées par le client](./customer-managed-keys.md), la [liaison privée](./private-link-security.md) et l’[exportation de données](./logs-data-export.md). En outre, les nouvelles fonctionnalités et optimisations sont ajoutées uniquement à l’API actuelle.

Après le 29 février 2024, Azure Monitor ne prendra plus en charge les versions d’API antérieures à la version 2020-08-01. Si vous préférez ne pas effectuer la mise à niveau, les requêtes envoyées depuis des versions antérieures continueront d’être traitées par le service Azure Monitor jusqu’au 29 février 2024.

## <a name="migration-steps"></a>Étapes de la migration
Selon la méthode de configuration que vous utilisez, vous devez mettre à jour la nouvelle version dans des requêtes **REST** et des **modèles Resource Manager**. Suivez les exemples ci-dessous pour mettre à jour la version de l’API :

1. Les requêtes de l’API REST utilisent la version de l’API dans l’URL de la demande. Remplacez cette version par la version la plus récente (2020-08-01), comme indiqué dans l’exemple suivant.

    ```rest
    https://management.azure.com/subscriptions/{subscriptionId}/resourcegroups/{resourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{workspaceName}?api-version=2020-08-01
    ```

2. Les modèles Resource Manager utilisent la version de l’API dans la propriété **apiVersion** de la ressource. Remplacez cette version par la version la plus récente (2020-08-01), comme indiqué dans l’exemple suivant.


    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2019-08-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "workspaceName": {
                "type": "string",
                "metadata": {
                "description": "Name of the workspace."
                }
            },
            "resources": [
            {
                "type": "Microsoft.OperationalInsights/workspaces",
                "name": "[parameters('workspaceName')]",
                "apiVersion": "2020-08-01",
                "location": "westus",
                "properties": {
                    "sku": {
                        "name": "pergb2018"
                    },
                    "retentionInDays": 30,
                    "features": {
                        "searchVersion": 1,
                        "legacy": 0,
                        "enableLogAccessUsingOnlyResourcePermissions": true
                    }
                }
            }
        ]
    }
    }
    ```


## <a name="next-steps"></a>Étapes suivantes

- Consultez la [référence pour l’API d’espace de travail OperationalInsights](/rest/api/loganalytics/workspaces).