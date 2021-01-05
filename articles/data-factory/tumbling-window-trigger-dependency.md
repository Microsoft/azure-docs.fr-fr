---
title: Créer des dépendances de déclencheur de fenêtre bascule
description: Découvrez comment créer une dépendance envers un déclencheur de fenêtre bascule dans Azure Data Factory.
services: data-factory
ms.author: chez
author: chez-charlie
manager: weetok
ms.service: data-factory
ms.workload: data-services
ms.devlang: na
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 09/03/2020
ms.openlocfilehash: 221f8bd7908613812a728d420a68b747051e095b
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97508796"
---
# <a name="create-a-tumbling-window-trigger-dependency"></a>Créer une dépendance de déclencheur de fenêtre bascule
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article explique étape par étape comment créer une dépendance envers un déclencheur de fenêtre bascule. Pour des informations générales sur les déclencheurs de fenêtre bascule, voir [Guide pratique pour créer un déclencheur de fenêtre bascule](how-to-create-tumbling-window-trigger.md).

Pour créer une chaîne de dépendance et faire en sorte qu’un déclencheur ne soit exécuté qu’après l’exécution d’un autre déclencheur dans la fabrique de données, utilisez cette fonctionnalité avancée de création d’une dépendance envers une fenêtre bascule.

Pour une démonstration de la création de pipelines dépendants dans votre fabrique de données Azure à l’aide d’un déclencheur de fenêtre bascule, regardez la vidéo suivante :

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Create-dependent-pipelines-in-your-Azure-Data-Factory/player]

## <a name="create-a-dependency-in-the-data-factory-ui"></a>Créer une dépendance dans l’interface utilisateur de Data Factory

Pour créer une dépendance envers un déclencheur, sélectionnez **Déclencher > Avancé > Nouveau**, puis choisissez un déclencheur présentant un décalage et une taille adaptés. Sélectionnez **Terminer** et publiez les modifications de la fabrique de données pour que les dépendances prennent effet.

![Création de dépendances](media/tumbling-window-trigger-dependency/tumbling-window-dependency-01.png "Création de dépendances")

## <a name="tumbling-window-dependency-properties"></a>Propriétés de la dépendance envers une fenêtre bascule

Un déclencheur de fenêtre bascule avec une dépendance présente les propriétés suivantes :

```json
{
    "name": "MyTriggerName",
    "properties": {
        "type": "TumblingWindowTrigger",
        "runtimeState": <<Started/Stopped/Disabled - readonly>>,
        "typeProperties": {
            "frequency": <<Minute/Hour>>,
            "interval": <<int>>,
            "startTime": <<datetime>>,
            "endTime": <<datetime – optional>>,
            "delay": <<timespan – optional>>,
            "maxConcurrency": <<int>> (required, max allowed: 50),
            "retryPolicy": {
                "count": <<int - optional, default: 0>>,
                "intervalInSeconds": <<int>>,
            },
            "dependsOn": [
                {
                    "type": "TumblingWindowTriggerDependencyReference",
                    "size": <<timespan – optional>>,
                    "offset": <<timespan – optional>>,
                    "referenceTrigger": {
                        "referenceName": "MyTumblingWindowDependency1",
                        "type": "TriggerReference"
                    }
                },
                {
                    "type": "SelfDependencyTumblingWindowTriggerReference",
                    "size": <<timespan – optional>>,
                    "offset": <<timespan>>
                }
            ]
        }
    }
}
```

Le tableau suivant donne la liste des attributs nécessaires pour définir une dépendance envers une fenêtre bascule.

| **Nom de la propriété** | **Description**  | **Type** | **Obligatoire** |
|---|---|---|---|
| type  | Tous les déclencheurs de fenêtre bascule existants s’affichent dans cette liste déroulante. Choisissez le déclencheur sur lequel portera la dépendance.  | TumblingWindowTriggerDependencyReference ou SelfDependencyTumblingWindowTriggerReference | Oui |
| offset | Décalage du déclencheur de dépendance. Indiquez une valeur au format TimeSpan ; sont autorisés les décalages positifs et négatifs. Cette propriété est obligatoire si le déclencheur dépend de lui-même, facultative dans tous les autres cas. L’autodépendance doit toujours correspondre à un décalage négatif. Si aucune valeur n’est spécifiée, la fenêtre est la même que le déclencheur lui-même. | Timespan<br/>(hh:mm:ss) | Autodépendance : Oui<br/>Autre : Non |
| taille | Taille de la fenêtre bascule de dépendance. Fournissez une valeur TimeSpan positive. Cette propriété est facultative. | Timespan<br/>(hh:mm:ss) | Non  |

> [!NOTE]
> Un déclencheur de fenêtre bascule peut dépendre d’un maximum de cinq autres déclencheurs.

## <a name="tumbling-window-self-dependency-properties"></a>Propriétés d’autodépendance d’une fenêtre bascule

Dans les scénarios où le déclencheur ne doit pas passer à la fenêtre suivante tant que la fenêtre précédente n’est pas terminée, créez une autodépendance. Un déclencheur d’autodépendance qui dépend de la réussite de ses exécutions antérieures au cours de l’heure précédente présente les propriétés indiquées dans le code suivant.

> [!NOTE]
> Si votre pipeline déclenché compte sur la sortie des pipelines dans les fenêtres déclenchées précédemment, nous vous recommandons d’utiliser uniquement l’autodépendance du déclencheur de la fenêtre bascule. Pour limiter les exécutions de déclencheurs parallèles, définissez le nombre maximal de déclencheurs simultanés.

```json
{
    "name": "DemoSelfDependency",
    "properties": {
        "runtimeState": "Started",
        "pipeline": {
            "pipelineReference": {
                "referenceName": "Demo",
                "type": "PipelineReference"
            }
        },
        "type": "TumblingWindowTrigger",
        "typeProperties": {
            "frequency": "Hour",
            "interval": 1,
            "startTime": "2018-10-04T00:00:00Z",
            "delay": "00:01:00",
            "maxConcurrency": 50,
            "retryPolicy": {
                "intervalInSeconds": 30
            },
            "dependsOn": [
                {
                    "type": "SelfDependencyTumblingWindowTriggerReference",
                    "size": "01:00:00",
                    "offset": "-01:00:00"
                }
            ]
        }
    }
}
```
## <a name="usage-scenarios-and-examples"></a>Scénarios et exemples d’utilisation

Voici les illustrations des scénarios et de l’utilisation des propriétés de dépendance envers une fenêtre bascule.

### <a name="dependency-offset"></a>Décalage de la dépendance

![Exemple de décalage](media/tumbling-window-trigger-dependency/tumbling-window-dependency-02.png "Exemple de décalage")

### <a name="dependency-size"></a>Taille de la dépendance

![Exemple de taille](media/tumbling-window-trigger-dependency/tumbling-window-dependency-03.png "Exemple de taille")

### <a name="self-dependency"></a>Autodépendance

![Autodépendance](media/tumbling-window-trigger-dependency/tumbling-window-dependency-04.png "Autodépendance")

### <a name="dependency-on-another-tumbling-window-trigger"></a>Dépendance envers un autre déclencheur de fenêtre bascule

Une tâche de traitement quotidien des données de télémétrie dépend d’une autre tâche quotidienne d’agrégation de la sortie des sept derniers jours et génère des flux de sept jours sur fenêtre glissante :

![Exemple de dépendance](media/tumbling-window-trigger-dependency/tumbling-window-dependency-05.png "Exemple de dépendance")

### <a name="dependency-on-itself"></a>Dépendance envers soi-même

Voici une tâche quotidienne sans écart dans les flux de sortie :

![Exemple d’autodépendance](media/tumbling-window-trigger-dependency/tumbling-window-dependency-06.png "Exemple d’autodépendance")

## <a name="monitor-dependencies"></a>Effectuer le monitoring des dépendances

Vous pouvez effectuer le monitoring de la chaîne de dépendance et des fenêtres correspondantes sur la page de monitoring des exécutions du déclencheur. Accédez à **Monitoring > Exécutions du déclencheur**. Si un déclencheur de fenêtre bascule a des dépendances, le nom du déclencheur porte un lien hypertexte vers la vue d’analyse des dépendances.  

![Superviser les exécutions de déclencheur](media/tumbling-window-trigger-dependency/tumbling-window-dependency-07.png "Surveiller les exécutions du déclencheur : affichage des dépendances des entiers à la fenêtre bascule")

Cliquez sur le nom du déclencheur pour afficher les dépendances du déclencheur. Le panneau de droite affiche des informations détaillées sur l’exécution du déclencheur, telles que RunID, l’heure de la fenêtre, l’état, etc.

![Monitor dependencies list view](media/tumbling-window-trigger-dependency/tumbling-window-dependency-08.png "Surveiller les dépendances Vue de la liste")

Vous pouvez voir l’état des dépendances et les fenêtres pour chaque déclencheur dépendant. Si l’un des déclencheurs de dépendances échoue, vous devez le réexécuter correctement pour que le déclencheur dépendant s’exécute.

Un déclencheur de fenêtre bascule attend des dépendances pendant _sept jours_ avant d’arriver à expiration. Après sept jours, l’exécution du déclencheur échoue.

Pour un rendu plus visuel de la planification des dépendances du déclencheur, sélectionnez l’affichage de Gantt.

![Monitor dependencies gantt chart](media/tumbling-window-trigger-dependency/tumbling-window-dependency-09.png "Surveiller les dépendances Vue du diagramme de Gantt")

Les zones transparentes montrent les fenêtres de dépendance pour chaque déclencheur dépendant du flux, tandis que les zones colorées ci-dessus montrent les différentes exécutions de fenêtre. Voici quelques conseils pour interpréter la vue du diagramme de Gantt :

* La zone transparente s’affiche en bleu lorsque les fenêtres dépendantes sont en attente ou en cours d’exécution.
* Une fois que toutes les fenêtres ont été correctement exécutées pour un déclencheur dépendant, la zone transparente devient verte.
* La zone transparente s’affiche en rouge lorsque certaines fenêtres dépendantes échouent. Rechercher une zone rouge pour identifier l’exécution de la fenêtre en échec

Pour réexécuter une fenêtre dans la vie du diagramme de Gantt, sélectionnez la zone colorée pour la fenêtre et un panneau d’action s’affiche avec les détails et les options de réexécution.

## <a name="next-steps"></a>Étapes suivantes

* Consulter le [Guide pratique pour créer un déclencheur de fenêtre bascule](how-to-create-tumbling-window-trigger.md)
