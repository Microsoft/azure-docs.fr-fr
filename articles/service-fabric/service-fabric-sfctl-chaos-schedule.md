---
title: Interface CLI Azure Service Fabric - sfctl chaos schedule
description: Apprenez-en plus sur sfctl, l’interface de ligne de commande d’Azure Service Fabric. Inclut la liste des commandes liées à la planification de chaos.
author: jeffj6123
ms.topic: reference
ms.date: 1/16/2020
ms.author: jejarry
ms.openlocfilehash: dd911574cd37c18e3043cfc2482bd1fcd5f3857b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86260845"
---
# <a name="sfctl-chaos-schedule"></a>sfctl chaos schedule
Permet d’obtenir et de définir la planification de chaos.

## <a name="commands"></a>Commandes

|Commande|Description|
| --- | --- |
| get | Obtenez la planification de chaos définissant quand et comment exécuter Chaos. |
| set | Définissez la planification utilisée par Chaos. |

## <a name="sfctl-chaos-schedule-get"></a>sfctl chaos schedule get
Obtenez la planification de chaos définissant quand et comment exécuter Chaos.

Obtient la version de la planification de chaos en cours d’utilisation et la planification de chaos qui définit quand et comment exécuter Chaos.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-chaos-schedule-set"></a>sfctl chaos schedule set
Définissez la planification utilisée par Chaos.

Chaos va planifier automatiquement les exécutions en fonction de la planification de chaos. La planification du chaos sera mise à jour si la version fournie correspond à la version sur le serveur. Lors de la mise à jour de la planification du chaos, la version est incrémentée de 1 sur le serveur. La version sur le serveur sera renvoyée à 0 après avoir atteint un nombre élevé. Si Chaos est en cours d’exécution lorsque cet appel est effectué, l’appel va échouer.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --chaos-parameters-dictionary | Liste encodée JSON représentant un mappage des noms de chaîne pour ChaosParameters à utiliser par Jobs. |
| --expiry-date-utc | La date et l’heure d’arrêt à l’aide de la planification pour planifier Chaos.  Valeur par défaut\: 9999-12-31T23\:59\:59.999Z. |
| --jobs | Liste encodée JSON de ChaosScheduleJobs représentant quand exécuter Chaos et avec quels paramètres. |
| --start-date-utc | La date et l’heure de démarrage à l’aide de la planification pour planifier Chaos.  Valeur par défaut\: 1601-01-01T00\:00\:00.000Z. |
| --timeout -t | Valeur par défaut \: 60. |
| --version | Le numéro de version de la planification. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

### <a name="examples"></a>Exemples

La commande suivante définit une planification (en supposant que la planification actuelle a la version 0) qui démarre le 01-01-2016 et expire le 01-01-2038, et qui exécute Chaos 24 heures/24 et 7 jours/7.
Chaos va être planifié sur le cluster pour ces dates.
```
sfctl chaos schedule set --version 0 --start-date-utc "2016-01-01T00:00:00.000Z" --expiry-date-utc "2038-01-01T00:00:00.000Z"
    --chaos-parameters-dictionary
    [
    {
        "Key":"adhoc",
        "Value":{
            "MaxConcurrentFaults":3,
            "EnableMoveReplicaFaults":true,
            "ChaosTargetFilter":{
                "NodeTypeInclusionList":[
                "N0010Ref",
                "N0020Ref",
                "N0030Ref",
                "N0040Ref",
                "N0050Ref"
                ]
            },
            "MaxClusterStabilizationTimeoutInSeconds":60,
            "WaitTimeBetweenIterationsInSeconds":15,
            "WaitTimeBetweenFaultsInSeconds":30,
            "TimeToRunInSeconds":"600",
            "Context":{
                "Map":{
                "test":"value"
                }
            },
            "ClusterHealthPolicy":{
                "MaxPercentUnhealthyNodes":0,
                "ConsiderWarningAsError":true,
                "MaxPercentUnhealthyApplications":0
            }
        }
    }
    ]
    --jobs
    [
    {
        "ChaosParameters":"adhoc",
        "Days":{
            "Sunday":true,
            "Monday":true,
            "Tuesday":true,
            "Wednesday":true,
            "Thursday":true,
            "Friday":true,
            "Saturday":true
        },
        "Times":[
            {
                "StartTime":{
                "Hour":0,
                "Minute":0
                },
                "EndTime":{
                "Hour":23,
                "Minute":59
                }
            }
        ]
    }
    ]
```



## <a name="next-steps"></a>Étapes suivantes
- [Configurez](service-fabric-cli.md) l’interface de ligne de commande Service Fabric.
- Découvrez comment utiliser l’interface de ligne de commande (CLI) Service Fabric à l’aide d’[exemples de scripts](./scripts/sfctl-upgrade-application.md).
