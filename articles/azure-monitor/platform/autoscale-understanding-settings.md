---
title: Comprendre les paramètres de mise à l’échelle automatique dans Azure Monitor
description: Il s’agit d’une description détaillée des paramètres de mise à l’échelle automatique et de leur fonctionnement. S’applique aux machines virtuelles, aux services cloud et aux applications web
ms.topic: conceptual
ms.date: 12/18/2017
ms.subservice: autoscale
ms.openlocfilehash: a914f6d71c013acea8dfde0f6578985bc009bb26
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97605238"
---
# <a name="understand-autoscale-settings"></a>Comprendre les paramètres de mise à l’échelle automatique
Les paramètres de mise à l’échelle automatique permettent de s’assurer qu’un nombre approprié de ressources s’exécute pour gérer la charge fluctuante de votre application. Vous pouvez configurer les paramètres de mise à l’échelle automatique de sorte qu’ils soient déclenchés en fonction de mesures indiquant la charge ou les performances, ou un déclenchement à une date et une heure planifiées. Cet article examine de manière détaillée l’anatomie d’un paramètre de mise à l’échelle automatique. L’article commence par le schéma et les propriétés d’un paramètre, puis examine les différents types de profil qui peuvent être configurés. Enfin, il explique comment la fonctionnalité de mise à l’échelle automatique dans Azure évalue chaque profil à exécuter à un moment donné.

## <a name="autoscale-setting-schema"></a>Schéma du paramètre de mise à l'échelle automatique
Pour illustrer le schéma du paramètre de mise à l’échelle automatique, le paramètre de mise à l’échelle automatique suivant est utilisé. Il est important de noter que ce paramètre de mise à l’échelle automatique comporte les éléments suivants :
- Un profil. 
- Deux règles de métrique dans ce profil : une pour le scale-out et l’autre pour le scale-in.
  - La règle d’augmentation de la taille des instances est déclenchée lorsque la mesure de pourcentage moyen d’utilisation du processeur du groupe de machines virtuelles identiques est supérieure à 85 % pendant les 10 dernières minutes.
  - La règle de diminution de la taille des instances est déclenchée lorsque la moyenne du groupe de machines virtuelles identiques est inférieure à 60 % pendant la dernière minute.

> [!NOTE]
> Un paramètre peut avoir plusieurs profils. Pour plus d’informations, consultez la section [Profils](#autoscale-profiles). Un profil peut également avoir plusieurs règles de montée et descente en puissance définies. Pour voir comment elles sont évaluées, consultez la section [Évaluation](#autoscale-evaluation).

```JSON
{
  "id": "/subscriptions/s1/resourceGroups/rg1/providers/microsoft.insights/autoscalesettings/setting1",
  "name": "setting1",
  "type": "Microsoft.Insights/autoscaleSettings",
  "location": "East US",
  "properties": {
    "enabled": true,
    "targetResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.Compute/virtualMachineScaleSets/vmss1",
    "profiles": [
      {
        "name": "mainProfile",
        "capacity": {
          "minimum": "1",
          "maximum": "4",
          "default": "1"
        },
        "rules": [
          {
            "metricTrigger": {
              "metricName": "Percentage CPU",
              "metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.Compute/virtualMachineScaleSets/vmss1",
              "timeGrain": "PT1M",
              "statistic": "Average",
              "timeWindow": "PT10M",
              "timeAggregation": "Average",
              "operator": "GreaterThan",
              "threshold": 85
            },
            "scaleAction": {
              "direction": "Increase",
              "type": "ChangeCount",
              "value": "1",
              "cooldown": "PT5M"
            }
          },
          {
            "metricTrigger": {
              "metricName": "Percentage CPU",
              "metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.Compute/virtualMachineScaleSets/vmss1",
              "timeGrain": "PT1M",
              "statistic": "Average",
              "timeWindow": "PT10M",
              "timeAggregation": "Average",
              "operator": "LessThan",
              "threshold": 60
            },
            "scaleAction": {
              "direction": "Decrease",
              "type": "ChangeCount",
              "value": "1",
              "cooldown": "PT5M"
            }
          }
        ]
      }
    ]
  }
}
```

| Section | Nom de l'élément | Description |
| --- | --- | --- |
| Paramètre | id | ID de ressource du paramètre de mise à l’échelle automatique. Les paramètres de mise à l’échelle automatique sont une ressource Azure Resource Manager. |
| Paramètre | name | Nom du paramètre de mise à l'échelle automatique. |
| Paramètre | location | Emplacement du paramètre de mise à l’échelle automatique. Cet emplacement peut être différent de celui de la ressource en cours de mise à l’échelle. |
| properties | targetResourceUri | ID de la ressource mise à l’échelle. Vous ne pouvez avoir qu’un seul paramètre de mise à l’échelle automatique par ressource. |
| properties | profiles | Un paramètre de mise à l’échelle automatique se compose d’un ou plusieurs profils. Chaque fois que le moteur de mise à l’échelle automatique s’exécute, un profil est exécuté. |
| profile | name | Nom du profil. Vous pouvez choisir n’importe quel nom vous permettant d’identifier le profil. |
| profile | capacity.maximum | Capacité maximale autorisée. Elle garantit que la mise à l’échelle automatique, lors de l’exécution de ce profil, ne met pas votre ressource à l’échelle au-dessus de ce nombre. |
| profile | capacity.minimum | Capacité minimale autorisée. Elle garantit que la mise à l’échelle automatique, lors de l’exécution de ce profil, ne met pas votre ressource à l’échelle au-dessous de ce nombre. |
| profile | capacity.default | S’il existe un problème de lecture des mesures de ressource (dans ce cas l’UC de « vmss1 ») et que la capacité actuelle est inférieure à la capacité par défaut, la mise à l’échelle automatique sera modifiée sur la valeur par défaut. Cela permet de garantir la disponibilité de la ressource. Si la capacité actuelle est déjà supérieure à la capacité par défaut, la mise à l’échelle automatique n’effectue pas de scale-in. |
| profile | rules | La mise à l’échelle s’ajuste automatiquement entre les capacités maximales et minimales en utilisant les règles du profil. Un profil peut contenir plusieurs règles. Généralement, deux règles sont utilisées : une pour déterminer quand effectuer un scale-out, et une autre pour déterminer quand effectuer un scale-in. |
| rule | metricTrigger | Définit la condition de mesure de la règle. |
| metricTrigger | metricName | Nom de la mesure. |
| metricTrigger |  metricResourceUri | ID de la ressource qui a généré la mesure. Dans la plupart des cas, il est identique à la ressource mise à l’échelle. Dans certains cas, il peut être différent. Par exemple, vous pouvez faire évoluer un groupe de machines virtuelles identiques en fonction du nombre de messages figurant dans la file d’attente de stockage. |
| metricTrigger | timeGrain | Durée d’échantillonnage de la mesure. Par exemple, **TimeGrain = "PT1M"** signifie que les mesures seront agrégées chaque minute à l'aide de la méthode d'agrégation spécifiée dans l’élément « statistic ». |
| metricTrigger | statistic | Méthode d’agrégation au cours de la période timeGrain. Par exemple, **statistic = “Average”** et **timeGrain = “PT1M”** signifient que les mesures sont agrégées chaque minute en prenant la valeur moyenne. Cette propriété détermine la façon dont la mesure est échantillonnée. |
| metricTrigger | timeWindow | Durée nécessaire à l’examen des mesures. Par exemple, **timeWindow = “PT10M”** signifie qu’à chaque exécution d’une mise à l’échelle automatique, les mesures sont interrogées sur les 10 dernières minutes. La fenêtre de temps permet la normalisation de vos mesures et évite de réagir aux pics temporaires. |
| metricTrigger | timeAggregation | Méthode d’agrégation utilisée pour agréger les mesures échantillonnées. Par exemple, **TimeAggregation = “Average”** doit agréger les mesures échantillonnées en prenant la moyenne. Dans le cas précédent, choisissez les dix échantillons de 1 minute et calculez leur moyenne. |
| rule | scaleAction | Action à entreprendre lors du déclenchement du paramètre metricTrigger de la règle. |
| scaleAction | direction | « Increase » pour scale-out, ou « Decrease » pour scale-in.|
| scaleAction | value | Indique dans quelle mesure augmenter ou diminuer la capacité de la ressource. |
| scaleAction | cooldown | Temps d’attente entre deux opérations de mise à l’échelle. Par exemple, si **cooldown = “PT10M”** , aucune tentative de mise à l’échelle automatique ne peut survenir dans les 10 minutes. Le cooldown permet la stabilisation des mesures après l’ajout ou la suppression d’instances. |

## <a name="autoscale-profiles"></a>Profils de mise à l’échelle automatique

Il existe trois types de profils de mise à l’échelle automatique :

- **Profil régulier :** profil le plus courant. Si vous n’avez pas besoin de mettre à l’échelle votre ressource en fonction du jour de la semaine ou d’un jour donné, vous pouvez utiliser un profil régulier. Ce profil peut ensuite être configuré à l’aide de règles de mesures qui déterminent quand effectuer un scale-out et quand effectuer un scale-in. Un seul profil régulier doit être défini.

    L’exemple de profil utilisé précédemment dans cet article est un exemple de profil régulier. Notez qu’il est également possible de définir un profil à mettre à l’échelle sur un certain nombre d’instances statiques pour votre ressource.

- **Profil de date fixe :** ce profil est réservé aux cas particuliers. Par exemple, supposons qu’un événement important est prévu le 26 décembre 2017 (PST). Vous souhaitez que les capacités minimales et maximales de votre ressource soient différentes ce jour-là, mais en conservant les mêmes métriques. Dans ce cas, vous devez ajouter un profil de date fixe à la liste des profils de votre paramètre. Le profil est configuré de manière à s’exécuter uniquement le jour de l’événement. Tous les autres jours, la mise à l’échelle automatique utilise le profil régulier.

    ```json
    "profiles": [
        {
            "name": " regularProfile",
            "capacity": {
                ...
            },
            "rules": [
                {
                ...
                },
                {
                ...
                }
            ]
        },
        {
            "name": "eventProfile",
            "capacity": {
            ...
            },
            "rules": [
                {
                ...
                }, 
                {
                ...
                }
            ],
            "fixedDate": {
                "timeZone": "Pacific Standard Time",
                "start": "2017-12-26T00:00:00",
                "end": "2017-12-26T23:59:00"
            }
        }
    ]
    ```
    
- **Profil de périodicité :** ce type de profil permet de s’assurer que ce profil est toujours utilisé un jour spécifique de la semaine. Les profils de périodicité affichent uniquement une heure de début. Ils s’exécutent jusqu'à ce que le profil de périodicité suivant ou le profil de date fixe est configuré pour démarrer. Un paramètre de mise à l’échelle automatique doté d’un seul profil de périodicité exécute ce profil, même s’il existe un profil régulier défini dans le même paramètre. Les deux exemples suivants illustrent l’utilisation de ce profil :

    **Exemple 1 : jours de la semaine et week-ends**
    
    Supposons que les week-ends, vous souhaitez disposer d’une capacité maximale de 4. Les jours de la semaine, étant donné que vous prévoyez plus de charge, cette capacité maximale doit être de 10. Dans ce cas, le paramètre contiendrait deux profils de périodicité, un à exécuter les week-ends et l’autre les jours de la semaine.
    Le paramètre ressemble à ceci :

    ``` JSON
    "profiles": [
    {
    "name": "weekdayProfile",
    "capacity": {
        ...
    },
    "rules": [{
        ...
    }],
    "recurrence": {
        "frequency": "Week",
        "schedule": {
            "timeZone": "Pacific Standard Time",
            "days": [
                "Monday"
            ],
            "hours": [
                0
            ],
            "minutes": [
                0
            ]
        }
    }}
    },
    {
    "name": "weekendProfile",
    "capacity": {
        ...
    },
    "rules": [{
        ...
    }]
    "recurrence": {
        "frequency": "Week",
        "schedule": {
            "timeZone": "Pacific Standard Time",
            "days": [
                "Saturday"
            ],
            "hours": [
                0
            ],
            "minutes": [
                0
            ]
        }
    }
    }]
    ```

    Le paramètre précédent montre que chaque profil de périodicité comporte une planification. Cette planification détermine quand le profil sera exécuté. Le profil s’interrompt lorsqu’il est temps d’exécuter un autre profil.

    Par exemple, dans le paramètre précédent, « weekdayProfile » est défini pour démarrer le lundi à 12h00. Cela signifie que ce profil commence à s’exécuter le lundi à 12h00. Il s’exécute jusqu'au samedi 12h00, moment auquel « weekendProfile » doit démarrer.

    **Exemple 2 : heures d’ouverture**
    
    Supposons que vous souhaitez avoir un seuil de métrique pendant les heures d’ouverture (9h00 à 17h00), et un autre seuil le reste du temps. Le paramètre pourrait ressemble à ceci :
    
    ``` JSON
    "profiles": [
    {
    "name": "businessHoursProfile",
    "capacity": {
        ...
    },
    "rules": [{
        ...
    }],
    "recurrence": {
        "frequency": "Week",
        "schedule": {
            "timeZone": "Pacific Standard Time",
            "days": [
                "Monday", “Tuesday”, “Wednesday”, “Thursday”, “Friday”
            ],
            "hours": [
                9
            ],
            "minutes": [
                0
            ]
        }
    }
    },
    {
    "name": "nonBusinessHoursProfile",
    "capacity": {
        ...
    },
    "rules": [{
        ...
    }]
    "recurrence": {
        "frequency": "Week",
        "schedule": {
            "timeZone": "Pacific Standard Time",
            "days": [
                "Monday", “Tuesday”, “Wednesday”, “Thursday”, “Friday”
            ],
            "hours": [
                17
            ],
            "minutes": [
                0
            ]
        }
    }
    }]
    ```
    
    Le paramètre précédent montre que l’option « businessHoursProfile » commence le lundi à 9h00 et se poursuit jusqu’à 17h00, moment où « nonBusinessHoursProfile » commence à s’exécuter. « nonBusinessHoursProfile » s’exécute jusqu'au mardi 9h00, puis « businessHoursProfile » reprend le relais. Ce processus se répète jusqu'au vendredi à 17h00. À ce stade, « nonBusinessHoursProfile » s’exécute jusqu'au lundi à 9h00.
    
> [!Note]
> L’interface utilisateur de mise à l’échelle automatique dans le portail Azure applique les heures de fin pour les profils de périodicité et commence à exécuter le profil par défaut du paramètre de mise à l’échelle entre les profils de périodicité.
    
## <a name="autoscale-evaluation"></a>Évaluation de mise à l'échelle automatique
Dans la mesure où les paramètres de mise à l’échelle automatique peuvent comporter plusieurs profils et que chaque profil peut comporter plusieurs règles de mesure, il est important de comprendre comment un tel paramètre est évalué. Chaque fois que le travail de mise à l’échelle automatique s’exécute, il commence par choisir le profil applicable. La mise à l’échelle automatique du profil évalue ensuite les valeurs minimales et maximales et les règles de mesure du profil et décide si une action de mise à l’échelle est nécessaire.

### <a name="which-profile-will-autoscale-pick"></a>Quel profil est choisi par la mise à l’échelle automatique ?

La mise à l’échelle automatique utilise la séquence suivante pour choisir le profil :
1. Elle recherche d’abord les profils de date fixe configurés pour s’exécuter maintenant. Une fois le profil trouvé, la mise à l’échelle l’exécute. Si plusieurs profils de date fixe sont supposés s’exécuter, la mise à l’échelle automatique sélectionne le premier.
2. En l’absence de profil de date fixe, la mise à l’échelle recherche un profil de périodicité. Si un profil de périodicité est trouvé, elle l’exécute.
3. En l’absence de profils de date fixe et de périodicité, la mise à l’échelle automatique exécute le profil régulier.

### <a name="how-does-autoscale-evaluate-multiple-rules"></a>Comment la mise à l’échelle automatique évalue plusieurs règles ?

Dès que la mise à l’échelle automatique a déterminé le profil censé s’exécuter, elle évalue toutes les règles d’augmentation de la taille dans le profil (règles dans lesquelles **direction = “Increase”** ).

Si une ou plusieurs règles d’augmentation de la taille des instances sont déclenchées, la mise à l’échelle automatique calcule la nouvelle capacité déterminée par **scaleAction** de chacune de ces règles. Elle augmente ensuite la taille des instances de manière à atteindre les capacités maximales pour garantir la disponibilité du service.

Par exemple, choisissons un groupe de machines virtuelles avec une capacité actuelle de 10. Il existe deux règles de montée en puissance : une qui augmente la capacité de 10 %, et une autre qui augmente la capacité de 3 unités. La première règle entraînerait une nouvelle capacité de 11, et la deuxième règle une capacité de 13. Pour garantir la disponibilité du service, la mise à l’échelle automatique choisit l’action qui résulte dans la capacité maximale, par conséquent, la deuxième règle.

Si aucune règle d’augmentation de la taille des instances n’est déclenchée, la mise à l’échelle automatique évalue toutes les règles de diminution de la taille des instances (règles dans lesquelles **direction = “Decrease”** ). La mise à l’échelle automatique n’accepte une action de diminution de la taille des instances que si toutes les règles correspondantes sont déclenchées.

La mise à l’échelle automatique calcule la nouvelle capacité déterminée par **scaleAction** de chacune de ces règles. Elle choisit ensuite l’action de mise à l’échelle offrant le maximum de capacité pour garantir la disponibilité du service.

Par exemple, choisissons un groupe de machines virtuelles avec une capacité actuelle de 10. Il existe deux règles de descente en puissance : une qui diminue la capacité de 50 %, et une autre qui diminue la capacité de 3 unités. La première règle entraînerait une nouvelle capacité de 5, et la deuxième règle une capacité de 7. Pour garantir la disponibilité du service, la mise à l’échelle automatique choisit l’action qui résulte dans la capacité maximale, par conséquent, la deuxième règle.

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur la mise à l’échelle automatique, consultez les ressources suivantes :

* [Vue d’ensemble de la mise à l’échelle automatique](./autoscale-overview.md)
* [Mesures courantes pour la mise à l’échelle automatique dans Azure Monitor](./autoscale-common-metrics.md)
* [Meilleures pratiques pour la mise à l’échelle automatique d’Azure Insights](./autoscale-best-practices.md)
* [Utilisation d’actions de mise à l’échelle automatique pour envoyer des notifications d’alerte webhook et par courrier électronique](./autoscale-webhook-email.md)
* [Paramètres de mise à l’échelle automatique](/rest/api/monitor/autoscalesettings)

