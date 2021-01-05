---
title: Obtenir les changements des ressources
description: Découvrez comment savoir quand une ressource a été modifiée, obtenir une liste des propriétés modifiées et évaluer les différences.
ms.date: 10/14/2020
ms.topic: how-to
ms.openlocfilehash: 70213caeaf71e1adc5a11ec0e9cbadfea032dca4
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92203464"
---
# <a name="get-resource-changes"></a>Obtenir les changements des ressources

Les ressources se transforment au fil de l’utilisation quotidienne, de la reconfiguration et même du redéploiement.
Ces modifications peuvent provenir d’une personne ou d’un processus automatisé. La plupart des modifications sont normales, mais ce n’est parfois pas le cas. Avec les 14 derniers jours d’historique des modifications, Azure Resource Graph vous permet de :

- Savoir à quel moment des changements ont été détectés dans une propriété Azure Resource Manager
- Voir le détail des modifications apportées aux propriétés d’une ressource
- Afficher la comparaison complète de la ressource avant et après la modification détectée

La détection des modifications et les détails sont utiles dans les scénarios suivants :

- Au cours de la gestion des incidents pour comprendre les modifications _potentiellement_ associées. Interrogez les événements de modification dans une fenêtre de temps spécifique et évaluez les détails des modifications.
- Dans le maintien d’une base de données de gestion de la configuration (CMBD) à jour. Au lieu de l’actualisation de toutes les ressources et de leurs jeux de propriétés complets selon une fréquence planifiée, obtenez uniquement les informations relatives à ce qui a changé.
- Pour la compréhension des autres propriétés modifiées lorsqu’une ressource a changé d’état de conformité. L’évaluation de ces propriétés supplémentaires peut fournir des insights sur les autres propriétés susceptibles de nécessiter une gestion par le biais d’une définition Azure Policy.

Cet article montre comment rassembler ces informations au moyen du kit SDK Resource Graph. Pour obtenir ces informations dans le Portail Azure, voir [Historique des changements](../../policy/how-to/determine-non-compliance.md#change-history) d’Azure Policy ou [Historique des changements](../../../azure-monitor/platform/activity-log.md#view-the-activity-log) d’Azure Activity Log. Pour plus de détails sur les changements apportés à vos applications de la couche d’infrastructure jusqu’au déploiement des applications, consultez [Utiliser l’analyse des changements d’application (préversion)](../../../azure-monitor/app/change-analysis.md) dans Azure Monitor.

> [!NOTE]
> Les détails des modifications dans Resource Graph concernent les propriétés Resource Manager. Pour le suivi des modifications à l’intérieur d’une machine virtuelle, voir [Suivi des modifications](../../../automation/change-tracking/overview.md) d’Azure Automation ou [Guest Configuration for VMs (Configuration invitée pour les machines virtuelles)](../../policy/concepts/guest-configuration.md) d’Azure Policy.

> [!IMPORTANT]
> L’historique des changements dans Azure Resource Graph est en préversion publique.

## <a name="find-detected-change-events-and-view-change-details"></a>Rechercher les événements de modification détectés et afficher le détail des modifications

La première étape pour voir ce qui a changé sur une ressource consiste à rechercher les événements de modification associés à cette ressource au sein d’une fenêtre de temps donnée. Chaque événement de modification comprend également des détails sur les modifications qui ont été apportées à la ressource. Cette étape s’effectue au moyen du point de terminaison REST **resourceChanges**.

Le point de terminaison **resourceChanges** accepte les paramètres suivants dans le corps de la requête :

- **resourceId** \[requis\] : ressource Azure sur laquelle rechercher des modifications.
- **interval** \[requis\] : propriété avec dates de _début_ et de _fin_ pour le contrôle d’un événement de modification à l’aide de **Zulu fuseau horaire (Z)** .
- **fetchPropertyChanges** (facultatif) : propriété booléenne qui définit si l’objet de réponse comprend des modifications de propriété.

Exemple de corps de requête :

```json
{
    "resourceId": "/subscriptions/{subscriptionId}/resourceGroups/MyResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount",
    "interval": {
        "start": "2019-09-28T00:00:00.000Z",
        "end": "2019-09-29T00:00:00.000Z"
    },
    "fetchPropertyChanges": true
}
```

Avec le corps de la demande ci-dessus, l’URI d’API REST pour **resourceChanges** est :

```http
POST https://management.azure.com/providers/Microsoft.ResourceGraph/resourceChanges?api-version=2018-09-01-preview
```

La réponse se présente ainsi :

```json
{
    "changes": [
        {
            "changeId": "{\"beforeId\":\"3262e382-9f73-4866-a2e9-9d9dbee6a796\",\"beforeTime\":\"2019-09-28T00:45:35.012Z\",\"afterId\":\"6178968e-981e-4dac-ac37-340ee73eb577\",\"afterTime\":\"2019-09-28T00:52:53.371Z\"}",
            "beforeSnapshot": {
                "snapshotId": "3262e382-9f73-4866-a2e9-9d9dbee6a796",
                "timestamp": "2019-09-28T00:45:35.012Z"
            },
            "afterSnapshot": {
                "snapshotId": "6178968e-981e-4dac-ac37-340ee73eb577",
                "timestamp": "2019-09-28T00:52:53.371Z"
            },
            "changeType": "Create"
        },
        {
            "changeId": "{\"beforeId\":\"a00f5dac-86a1-4d86-a1c5-a9f7c8147b7c\",\"beforeTime\":\"2019-09-28T00:43:38.366Z\",\"afterId\":\"3262e382-9f73-4866-a2e9-9d9dbee6a796\",\"afterTime\":\"2019-09-28T00:45:35.012Z\"}",
            "beforeSnapshot": {
                "snapshotId": "a00f5dac-86a1-4d86-a1c5-a9f7c8147b7c",
                "timestamp": "2019-09-28T00:43:38.366Z"
            },
            "afterSnapshot": {
                "snapshotId": "3262e382-9f73-4866-a2e9-9d9dbee6a796",
                "timestamp": "2019-09-28T00:45:35.012Z"
            },
            "changeType": "Delete"
        },
        {
            "changeId": "{\"beforeId\":\"b37a90d1-7ebf-41cd-8766-eb95e7ee4f1c\",\"beforeTime\":\"2019-09-28T00:43:15.518Z\",\"afterId\":\"a00f5dac-86a1-4d86-a1c5-a9f7c8147b7c\",\"afterTime\":\"2019-09-28T00:43:38.366Z\"}",
            "beforeSnapshot": {
                "snapshotId": "b37a90d1-7ebf-41cd-8766-eb95e7ee4f1c",
                "timestamp": "2019-09-28T00:43:15.518Z"
            },
            "afterSnapshot": {
                "snapshotId": "a00f5dac-86a1-4d86-a1c5-a9f7c8147b7c",
                "timestamp": "2019-09-28T00:43:38.366Z"
            },
            "propertyChanges": [
                {
                    "propertyName": "tags.org",
                    "afterValue": "compute",
                    "changeCategory": "User",
                    "changeType": "Insert"
                },
                {
                    "propertyName": "tags.team",
                    "afterValue": "ARG",
                    "changeCategory": "User",
                    "changeType": "Insert"
                }
            ],
            "changeType": "Update"
        },
        {
            "changeId": "{\"beforeId\":\"19d12ab1-6ac6-4cd7-a2fe-d453a8e5b268\",\"beforeTime\":\"2019-09-28T00:42:46.839Z\",\"afterId\":\"b37a90d1-7ebf-41cd-8766-eb95e7ee4f1c\",\"afterTime\":\"2019-09-28T00:43:15.518Z\"}",
            "beforeSnapshot": {
                "snapshotId": "19d12ab1-6ac6-4cd7-a2fe-d453a8e5b268",
                "timestamp": "2019-09-28T00:42:46.839Z"
            },
            "afterSnapshot": {
                "snapshotId": "b37a90d1-7ebf-41cd-8766-eb95e7ee4f1c",
                "timestamp": "2019-09-28T00:43:15.518Z"
            },
            "propertyChanges": [{
                "propertyName": "tags.cgtest",
                "afterValue": "hello",
                "changeCategory": "User",
                "changeType": "Insert"
            }],
            "changeType": "Update"
        }
    ]
}
```

Chaque événement de modification détecté pour **resourceId** a les propriétés suivantes :

- **changeId** : cette valeur concerne uniquement cette ressource. Bien que la chaîne **changeId** puisse parfois contenir d’autres propriétés, elle est uniquement garantie pour son unicité.
- **beforeSnapshot** : contient les propriétés **snapshotId** et **timestamp** de l’instantané de ressource qui a été créé avant qu’une modification n’ait été détectée.
- **afterSnapshot** : contient les propriétés **snapshotId** et **timestamp** de l’instantané de ressource qui a été créé après la détection de la modification.
- **changeType** : décrit le type de la modification détectée pour l’ensemble de l’enregistrement de modification, entre **beforeSnapshot** et **afterSnapshot**. Les valeurs sont les suivantes : _Create_, _Update_ et _Delete_. Le tableau de propriétés **propertyChanges** est inclus uniquement quand **changeType** a la valeur _Update_.
- **propertyChanges** : ce tableau de propriétés détaille toutes les propriétés de ressource qui ont été mises à jour entre **beforeSnapshot** et **afterSnapshot** :
  - **propertyName** : nom de la propriété de ressource qui a été modifiée.
  - **changeCategory** : décrit l’effet de la modification. Les valeurs sont les suivantes : _System_ et _User_.
  - **changeType** : décrit le type de la modification détectée pour chaque propriété de ressource.
    Les valeurs sont les suivantes : _Insert_, _Update_, _Remove_.
  - **beforeValue** : valeur de la propriété de ressource dans **beforeSnapshot**. Ne s’affiche pas quand **changeType** a la valeur _Insert_.
  - **afterValue** : valeur de la propriété de ressource dans **afterSnapshot**. Ne s’affiche pas quand **changeType** a la valeur _Remove_.

## <a name="compare-resource-changes"></a>Comparer les modifications des ressources

Avec le **changeId** du point de terminaison **resourceChanges**, le point de terminaison REST **resourceChangeDetails** est utilisé pour obtenir les instantanés pris avant et après la modification de la ressource.

Le point de terminaison **resourceChangeDetails** requiert deux paramètres dans le corps de la demande :

- **resourceId** : ressource Azure pour laquelle comparer les modifications.
- **changeId** : événement de modification unique pour le **resourceId** collecté à partir de **resourceChanges**.

Exemple de corps de requête :

```json
{
    "resourceId": "/subscriptions/{subscriptionId}/resourceGroups/MyResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount",
    "changeId": "{\"beforeId\":\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\",\"beforeTime\":'2019-05-09T00:00:00.000Z\",\"afterId\":\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\",\"afterTime\":'2019-05-10T00:00:00.000Z\"}"
}
```

Avec le corps de la demande ci-dessus, l’URI d’API REST pour **resourceChangeDetails** est :

```http
POST https://management.azure.com/providers/Microsoft.ResourceGraph/resourceChangeDetails?api-version=2018-09-01-preview
```

La réponse se présente ainsi :

```json
{
    "changeId": "{\"beforeId\":\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\",\"beforeTime\":'2019-05-09T00:00:00.000Z\",\"afterId\":\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\",\"beforeTime\":'2019-05-10T00:00:00.000Z\"}",
    "beforeSnapshot": {
        "timestamp": "2019-03-29T01:32:05.993Z",
        "content": {
            "sku": {
                "name": "Standard_LRS",
                "tier": "Standard"
            },
            "kind": "Storage",
            "id": "/subscriptions/{subscriptionId}/resourceGroups/MyResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount",
            "name": "mystorageaccount",
            "type": "Microsoft.Storage/storageAccounts",
            "location": "westus",
            "tags": {},
            "properties": {
                "networkAcls": {
                    "bypass": "AzureServices",
                    "virtualNetworkRules": [],
                    "ipRules": [],
                    "defaultAction": "Allow"
                },
                "supportsHttpsTrafficOnly": false,
                "encryption": {
                    "services": {
                        "file": {
                            "enabled": true,
                            "lastEnabledTime": "2018-07-27T18:37:21.8333895Z"
                        },
                        "blob": {
                            "enabled": true,
                            "lastEnabledTime": "2018-07-27T18:37:21.8333895Z"
                        }
                    },
                    "keySource": "Microsoft.Storage"
                },
                "provisioningState": "Succeeded",
                "creationTime": "2018-07-27T18:37:21.7708872Z",
                "primaryEndpoints": {
                    "blob": "https://mystorageaccount.blob.core.windows.net/",
                    "queue": "https://mystorageaccount.queue.core.windows.net/",
                    "table": "https://mystorageaccount.table.core.windows.net/",
                    "file": "https://mystorageaccount.file.core.windows.net/"
                },
                "primaryLocation": "westus",
                "statusOfPrimary": "available"
            }
        }
    },
    "afterSnapshot": {
        "timestamp": "2019-03-29T01:54:24.42Z",
        "content": {
            "sku": {
                "name": "Standard_LRS",
                "tier": "Standard"
            },
            "kind": "Storage",
            "id": "/subscriptions/{subscriptionId}/resourceGroups/MyResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount",
            "name": "mystorageaccount",
            "type": "Microsoft.Storage/storageAccounts",
            "location": "westus",
            "tags": {},
            "properties": {
                "networkAcls": {
                    "bypass": "AzureServices",
                    "virtualNetworkRules": [],
                    "ipRules": [],
                    "defaultAction": "Allow"
                },
                "supportsHttpsTrafficOnly": true,
                "encryption": {
                    "services": {
                        "file": {
                            "enabled": true,
                            "lastEnabledTime": "2018-07-27T18:37:21.8333895Z"
                        },
                        "blob": {
                            "enabled": true,
                            "lastEnabledTime": "2018-07-27T18:37:21.8333895Z"
                        }
                    },
                    "keySource": "Microsoft.Storage"
                },
                "provisioningState": "Succeeded",
                "creationTime": "2018-07-27T18:37:21.7708872Z",
                "primaryEndpoints": {
                    "blob": "https://mystorageaccount.blob.core.windows.net/",
                    "queue": "https://mystorageaccount.queue.core.windows.net/",
                    "table": "https://mystorageaccount.table.core.windows.net/",
                    "file": "https://mystorageaccount.file.core.windows.net/"
                },
                "primaryLocation": "westus",
                "statusOfPrimary": "available"
            }
        }
    }
}
```

**beforeSnapshot** et **afterSnapshot** donnent chacun l’heure à laquelle la capture instantanée a été effectuée et les propriétés à ce moment-là. La modification a eu lieu à un moment donné entre ces captures instantanées. L’exemple précédent montre que la propriété modifiée est **supportsHttpsTrafficOnly**.

Pour comparer les résultats, utilisez la propriété **changes** de **resourceChanges** ou évaluez la partie **content** de chaque instantané dans **resourceChangeDetails** pour déterminer les différences. Si vous comparez les instantanés, le **timestamp** s’affiche toujours comme une différence, même si cela est attendu.

## <a name="next-steps"></a>Étapes suivantes

- Examinez le langage utilisé dans les [requêtes de démarrage](../samples/starter.md).
- Examinez les utilisations avancées dans les [Requêtes avancées](../samples/advanced.md).
- Découvrez plus en détails comment [explorer des ressources](../concepts/explore-resources.md).
- Pour obtenir des conseils sur l’utilisation des requêtes à une fréquence élevée, voir [Instructions pour les requêtes limitées](../concepts/guidance-for-throttled-requests.md).
