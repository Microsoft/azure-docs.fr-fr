---
title: Migration vers les nouvelles versions de l’API Azure Time Series Insights Gen2 | Microsoft Docs
description: Comment mettre à jour les environnements Azure Time Series Insights Gen2 pour utiliser les nouvelles versions mises à la disposition générale.
ms.service: time-series-insights
services: time-series-insights
author: shreyasharmamsft
ms.author: shresha
manager: dpalled
ms.workload: big-data
ms.topic: conceptual
ms.date: 10/01/2020
ms.custom: shresha
ms.openlocfilehash: 3aedbef079ba62f42ea79afdcd9995d7ee23d9fa
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95020723"
---
# <a name="migrating-to-new-azure-time-series-insights-gen2-api-versions"></a>Migration vers les nouvelles versions d’API d’Azure Time Series Insights Gen2

## <a name="overview"></a>Vue d’ensemble

Si vous avez créé un environnement Azure Time Series Insights Gen2 lors de la préversion publique (avant le 16 juillet 2020), mettez à jour votre environnement TSI pour utiliser les nouvelles versions mises à la disposition générale des API en suivant la procédure décrite dans cet article. Cette modification n’affecte pas les utilisateurs qui exécutent la version Gen1 d’Azure Time Series Insights.

> [!IMPORTANT]
> Les mises à jour décrites dans cet article permettent UNIQUEMENT de mettre à niveau les versions d’API utilisées par votre environnement TSI. Cette modification n’est pas liée aux nouvelles [règles d’aplanissement et d’échappement JSON](./concepts-json-flattening-escaping-rules.md) introduites pour les environnements Gen2.

La nouvelle version de l’API est `2020-07-31` et utilise une [syntaxe d’expression de série chronologique](/rest/api/time-series-insights/reference-time-series-expression-syntax) mise à jour.

Les utilisateurs doivent migrer de leur environnement les [variables du modèle de série chronologique](./concepts-variables.md), les requêtes enregistrées, les requêtes Power BI et les outils personnalisés effectuant des appels aux points de terminaison de l’API. Si vous avez des questions ou des préoccupations concernant ce processus de migration, soumettez un ticket de support par le biais du portail Azure et mentionnez ce document.

> [!IMPORTANT]
> La préversion de l’API `2018-11-01-preview` sera toujours prise en charge jusqu’au 31 octobre 2020. Effectuez toutes les étapes applicables de cette migration au préalable afin d’éviter toute interruption du service.

## <a name="migrate-time-series-model-and-saved-queries"></a>Migrer un modèle de série chronologique et des requêtes enregistrées

Pour aider les utilisateurs à migrer leurs [variables de modèle de série chronologique](./concepts-variables.md) et les requêtes enregistrées, un outil intégré est disponible via l’[Explorateur Azure Time Series Insights](https://insights.timeseries.azure.com). Accédez à l’environnement que vous souhaitez migrer et suivez les étapes ci-dessous. **Vous pouvez effectuer la migration partiellement et revenir à la finalisation ultérieurement. Toutefois, aucune des mises à jour ne peut être annulée.**

> [!NOTE]
> Vous devez être contributeur de l’environnement pour effectuer des mises à jour du modèle de série chronologique et des requêtes enregistrées. Si vous n’êtes pas un contributeur, vous ne pourrez migrer que vos requêtes personnelles enregistrées. Consultez les [stratégies d’accès de l’environnement](./concepts-access-policies.md) et votre niveau d’accès avant de continuer.

1. L’Explorateur vous invite à mettre à jour la syntaxe utilisée par vos variables de modèle de série chronologique et les requêtes enregistrées.

    [![Invite](media/api-migration/ux-prompt.png)](media/v2-update-overview/overview-one.png#lightbox)

    Si vous fermez accidentellement la notification, elle se trouve dans le panneau de notification.

1. Cliquez sur **Afficher les mises à jour** pour ouvrir l’outil de migration.

1. Cliquez sur **Télécharger les types**. Étant donné que la migration remplacera vos types actuels pour modifier la syntaxe des variables, vous devez enregistrer une copie de vos types actuels. L’outil vous avertira quand les types auront été téléchargés.

    [![Télécharger des types](media/api-migration/ux-migration-tool.png)](media/v2-update-overview/overview-one.png#lightbox)

1. Cliquez sur **Mettre à jour les variables**. L’outil vous avertit quand des variables ont été mises à jour.

    > [!IMPORTANT]
    > Si vous mettez à jour vos types, les applications personnalisées qui utilisent une ancienne version de l’API (`2018-11-01-preview`) devront utiliser la nouvelle version de l’API (`2020-07-31`) pour continuer à fonctionner. Si vous n’êtes pas sûr de la version de l’API que vous utilisez, contactez votre administrateur avant de mettre à jour. Vous pouvez fermer l’outil de migration et revenir ultérieurement à ces étapes. En savoir plus sur les [Comment migrer des applications personnalisées](#migrate-custom-applications).

    [![Mettre à jour les variables](media/api-migration/ux-migration-tool-downloaded-types.png)](media/v2-update-overview/overview-one.png#lightbox)

1. Cliquez sur **Mettre à jour les requêtes enregistrées**. L’outil vous avertit quand les requêtes enregistrées sont mises à jour.

    [![Mettre à jour les requêtes enregistrées](media/api-migration/ux-migration-tool-updated-variables.png)](media/v2-update-overview/overview-one.png#lightbox)

1. Cliquez sur **Done**.

    [![Migration terminée](media/api-migration/ux-migration-tool-updated-saved-queries.png)](media/v2-update-overview/overview-one.png#lightbox)

Passez en revue votre environnement mis à jour en créant des graphiques d’une partie des variables nouvellement créées et des requêtes enregistrées. Si vous constatez un comportement inattendu lors de la représentation sous forme de graphiques, veuillez nous envoyer vos commentaires à l’aide de l’outil de commentaires de l’Explorateur.

## <a name="migrate-power-bi-queries"></a>Migrer des requêtes Power BI

Si vous avez généré des requêtes à l’aide du connecteur Power BI, elles effectuent des appels à Azure Time Series Insights à l’aide de la préversion de l’API et l’ancienne syntaxe d’expression de série chronologique. Ces requêtes continueront à récupérer correctement des données jusqu’à ce que l’API en préversion soit déconseillée.

Pour mettre à jour les requêtes afin d’utiliser la nouvelle version de l’API et la nouvelle syntaxe d’expression de série chronologique, les requêtes doivent être régénérées à partir de l’Explorateur. Plus d’informations sur la [création de requêtes à l’aide du connecteur Power BI](./how-to-connect-power-bi.md).

> [!NOTE]
> Vous devez utiliser la version de juillet 2020 de Power BI Desktop. Sinon, une [erreur de version de charge utile de requête non valide](./how-to-diagnose-troubleshoot.md#problem-power-bi-connector-shows-unable-to-connect) pourrait s’afficher.

## <a name="migrate-custom-applications"></a>Migrer des applications personnalisées

Si votre application personnalisée effectue des appels aux points de terminaison REST suivants, il suffit de mettre à jour la version de l’API sur `2020-07-31` dans l’URI :

- API Time Series Model
  - API Paramètres du modèle
    - [Get](/rest/api/time-series-insights/dataaccessgen2/modelsettings/get)
    - [Mettre à jour](/rest/api/time-series-insights/dataaccessgen2/modelsettings/update)
  - API d’instance
    - [Toutes les opérations de traitement par lots](/rest/api/time-series-insights/dataaccessgen2/timeseriesinstances/executebatch)
    - [Liste](/rest/api/time-series-insights/dataaccessgen2/timeseriesinstances/list)
    - [action](/rest/api/time-series-insights/dataaccessgen2/timeseriesinstances/search)
    - [Suggérer](/rest/api/time-series-insights/dataaccessgen2/timeseriesinstances/suggest)
  - API de hiérarchie
    - [Toutes les opérations de traitement par lots](/rest/api/time-series-insights/dataaccessgen2/timeserieshierarchies/executebatch)
    - [Liste](/rest/api/time-series-insights/dataaccessgen2/timeserieshierarchies/list)
  - API de types
    - [Opérations de suppression et de récupération](/rest/api/time-series-insights/dataaccessgen2/timeseriestypes/executebatch)
    - [Liste](/rest/api/time-series-insights/dataaccessgen2/timeseriestypes/list)

Pour les points de terminaison REST suivants, vous devez mettre à jour la version de l’API sur `2020-07-31` dans l’URI et vérifier que toutes les occurrences de la propriété `tsx` utilisent la [syntaxe d’expression de série chronologique mise à jour](/rest/api/time-series-insights/reference-time-series-expression-syntax).

- API de types
  - [Opération Put](/rest/api/time-series-insights/dataaccessgen2/timeseriestypes/executebatch#typesbatchput)
- API de requête
  - [GetEvents](/rest/api/time-series-insights/dataaccessgen2/query/execute#getevents)
  - [GetSeries](/rest/api/time-series-insights/dataaccessgen2/query/execute#getseries)
  - [GetAggregateSeries](/rest/api/time-series-insights/dataaccessgen2/query/execute#aggregateseries)

### <a name="examples"></a>Exemples

#### <a name="typesbatchput"></a>TypesBatchPut

Ancien corps de la demande (utilisé par `2018-11-01-preview`) :

```JSON
{
  "put": [
    {
      "id": "c1cb7a33-ed9b-4cf1-9958-f3162fed8ee8",
      "name": "OutdoorTemperatureSensor",
      "description": "This is an outdoor temperature sensor.",
      "variables": {
        "AverageTemperature": {
          "kind": "numeric",
          "value": {
            "tsx": "$event.[Temperature_Celsius].Double"
          },
          "filter": {
            "tsx": "$event.[Mode].String = 'outdoor'"
          },
          "aggregation": {
            "tsx": "avg($value)"
          }
        }
      }
    }
  ]
}
```

Corps de la demande mis à jour (utilisé par `2020-07-31`) :

```JSON
{
  "put": [
    {
      "id": "c1cb7a33-ed9b-4cf1-9958-f3162fed8ee8",
      "name": "OutdoorTemperatureSensor",
      "description": "This is an outdoor temperature sensor.",
      "variables": {
        "AverageTemperature": {
          "kind": "numeric",
          "value": {
            "tsx": "$event['Temperature_Celsius'].Double"
          },
          "filter": {
            "tsx": "$event['Mode'].String = 'outdoor'"
          },
          "aggregation": {
            "tsx": "avg($value)"
          }
        }
      }
    }
  ]
}
```

`filter` peut également être `$event.Mode.String = 'outdoor'`. `value` doit utiliser les crochets pour échapper le caractère spécial (`_`).

#### <a name="getevents"></a>GetEvents

Ancien corps de la demande (utilisé par `2018-11-01-preview`) :

```JSON
{
  "getEvents": {
    "timeSeriesId": [
      "006dfc2d-0324-4937-998c-d16f3b4f1952",
      "T1"
    ],
    "searchSpan": {
      "from": "2016-08-01T00:00:00Z",
      "to": "2016-08-01T00:16:50Z"
    },
    "filter": {
      "tsx": "($event.[Value].Double != null) OR ($event.[Status].String = 'Good')"
    },
    "projectedProperties": [
      {
        "name": "Temperature",
        "type": "Double"
      }
    ]
  }
}
```

Corps de la demande mis à jour (utilisé par `2020-07-31`) :

```JSON
{
  "getEvents": {
    "timeSeriesId": [
      "006dfc2d-0324-4937-998c-d16f3b4f1952",
      "T1"
    ],
    "searchSpan": {
      "from": "2016-08-01T00:00:00Z",
      "to": "2016-08-01T00:16:50Z"
    },
    "filter": {
      "tsx": "($event.Value.Double != null) OR ($event.Status.String = 'Good')"
    },
    "projectedProperties": [
      {
        "name": "Temperature",
        "type": "Double"
      }
    ]
  }
}
```

`filter` peut également être `($event['Value'].Double != null) OR ($event['Status'].String = 'Good')`.

#### <a name="getseries"></a>GetSeries

Ancien corps de la demande (utilisé par `2018-11-01-preview`) :

```JSON
{
  "getSeries": {
    "timeSeriesId": [
      "006dfc2d-0324-4937-998c-d16f3b4f1952"
    ],
    "searchSpan": {
      "from": "2016-08-01T00:00:00Z",
      "to": "2016-08-01T00:16:50Z"
    },
    "inlineVariables": {
      "pressure": {
        "kind": "numeric",
        "value": {
          "tsx": "$event.[Bar-Pressure-Offset]"
        },
        "aggregation": {
          "tsx": "avg($value)"
        }
      }
    },
    "projectedVariables": [
      "pressure"
    ]
  }
}
```

Corps de la demande mis à jour (utilisé par `2020-07-31`) :

```JSON
{
  "getSeries": {
    "timeSeriesId": [
      "006dfc2d-0324-4937-998c-d16f3b4f1952"
    ],
    "searchSpan": {
      "from": "2016-08-01T00:00:00Z",
      "to": "2016-08-01T00:16:50Z"
    },
    "inlineVariables": {
      "pressure": {
        "kind": "numeric",
        "value": {
          "tsx": "$event['Bar-Pressure-Offset']"
        },
        "aggregation": {
          "tsx": "avg($value)"
        }
      }
    },
    "projectedVariables": [
      "pressure"
    ]
  }
}
```

`value` peut également être `$event['Bar-Pressure-Offset'].Double`. Si aucun type de données n’est spécifié, le type de données est toujours supposé être Double. La notation entre crochets doit être utilisée pour échapper le caractère spécial (`-`).

#### <a name="aggregateseries"></a>AggregateSeries

Ancien corps de la demande (utilisé par `2018-11-01-preview`) :

```JSON
{
  "aggregateSeries": {
    "timeSeriesId": [
      "006dfc2d-0324-4937-998c-d16f3b4f1952"
    ],
    "searchSpan": {
      "from": "2016-08-01T00:00:00Z",
      "to": "2016-08-01T00:16:50Z"
    },
    "interval": "PT1M",
    "inlineVariables": {
      "MinTemperature": {
        "kind": "numeric",
        "value": {
          "tsx": "coalesce($event.[Temp].Double, toDouble($event.[Temp].Long))"
        },
        "aggregation": {
          "tsx": "min($value)"
        }
      },
    },
    "projectedVariables": [
      "MinTemperature"
    ]
  }
}
```

Corps de la demande mis à jour (utilisé par `2020-07-31`) :

```JSON
  "aggregateSeries": {
    "timeSeriesId": [
      "006dfc2d-0324-4937-998c-d16f3b4f1952"
    ],
    "searchSpan": {
      "from": "2016-08-01T00:00:00Z",
      "to": "2016-08-01T00:16:50Z"
    },
    "interval": "PT1M",
    "inlineVariables": {
      "MinTemperature": {
        "kind": "numeric",
        "value": {
          "tsx": "coalesce($event.Temp.Double, toDouble($event.Temp.Long))"
        },
        "aggregation": {
          "tsx": "min($value)"
        }
      },
    },
    "projectedVariables": [
      "MinTemperature"
    ]
  }
}
```

`value` peut également être `coalesce($event['Temp'].Double, toDouble($event['Temp'].Long))`.

### <a name="potential-errors"></a>Erreurs potentielles

#### <a name="invalidinput"></a>InvalidInput

Si vous voyez l’erreur suivante, vous utilisez la nouvelle version de l’API (`2020-07-31`), mais la syntaxe TSX n’a pas été mise à jour. Consultez la [syntaxe d’expression de série chronologique](/rest/api/time-series-insights/reference-time-series-expression-syntax) et les exemples de migration ci-dessus. Assurez-vous que toutes les propriétés de `tsx` sont correctement mises à jour avant de renvoyer la requête d’API.

```JSON
{
    "error": {
        "code": "InvalidInput",
        "message": "Unable to parse 'value' time series expression (TSX) in variable 'Temperature'.",
        "target": "projectedVariables.Temperature.value",
        "innerError": {
            "parseErrorDetails": [
                {
                    "pos": [
                        0,
                        5
                    ],
                    "line": 1,
                    "msg": "Unsupported Time Series Expression version TSX01 used instead of TSX00.",
                    "code": "UnsupportedTSXVersionTSX01",
                    "target": "$event"
                }
            ],
            "code": "TsxParseError"
        }
    }
}
```

## <a name="next-steps"></a>Étapes suivantes

- Testez votre environnement via l’[Explorateur Azure Time Series Insights](./concepts-ux-panels.md) ou votre application personnalisée.