---
title: Modèle de série chronologique – Azure Time Series Insights Gen2 | Microsoft Docs
description: Découvrez le modèle de série chronologique dans Azure Time Series Insights Gen2.
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 10/01/2020
ms.custom: seodec18
ms.openlocfilehash: 0c201ce984a216a5cc62e221c0433f83a7eeabae
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95021760"
---
# <a name="time-series-model-in-azure-time-series-insights-gen2"></a>Modèle de série chronologique dans Azure Time Series Insights Gen2

Cet article décrit le modèle de série chronologique, les capacités et la façon de commencer à créer et à mettre à jour vos propres modèles dans l’environnement Azure Time Series Insights Gen2.

> [!TIP]
>
> * Pour obtenir un exemple de modèle de série chronologique en direct, accédez à l’environnement de  [démonstration du parc éolien de Contoso](https://insights.timeseries.azure.com/preview/samples).
> * Découvrez [comment utiliser le modèle de série chronologique](./how-to-edit-your-model.md) à l’aide de l’Explorateur Azure Time Series Insights.

## <a name="summary"></a>Résumé

En règle générale, les données collectées à partir d’appareils IoT manquent d’informations contextuelles, ce qui empêche de rechercher et d’analyser rapidement les capteurs. Le principal objectif du modèle de série chronologique est de simplifier la recherche et l’analyse des données IoT ou de série chronologique. Vous pouvez atteindre cet objectif grâce à la collecte, la maintenance et l’enrichissement des données d’une série chronologique afin d’obtenir des jeux de données prêts à l’emploi pour l’analytique.

## <a name="scenario-contosos-new-smart-oven"></a>Scénario : Nouveau four intelligent de Contoso

**Prenons le scénario fictif d’un four intelligent Contoso.** Dans ce scénario, supposons que chaque four intelligent Contoso possède cinq capteurs de température, un pour chacun des quatre brûleurs supérieurs et un pour le four lui-même. Jusqu’à récemment, chaque capteur de température Contoso envoyait, stockait et visualisait ses données individuellement. Pour la surveillance des appareils ménagers, Contoso s’appuyait sur des graphiques de base, un pour chaque capteur individuel.

Bien que Contoso ait été satisfait de sa solution initiale de données et de visualisation, plusieurs limitations sont apparues :

* Les clients souhaitaient savoir quelle serait la température de l’ensemble du four lorsque la plupart des brûleurs supérieurs étaient allumés. Contoso a eu plus de difficulté à analyser et à présenter une réponse unifiée sur les conditions de l’ensemble du four.
* Les ingénieurs de Contoso voulaient vérifier que les brûleurs supérieurs fonctionnant simultanément n’entraîneraient pas une consommation électrique inefficace. Il a été difficile d’établir quel capteur de température était associé à quel capteur de tension et de déterminer comment les localiser dans le magasin.
* L’équipe d’assurance qualité de Contoso souhaitait auditer et comparer l’historique entre deux versions de capteurs. Il était difficile de déterminer quelles données appartenaient à quelle version de capteur.

Sans la possibilité de structurer, d’organiser et de définir le modèle global de série chronologique du four intelligent, chaque capteur de température maintenait des points de données disloqués, isolés et moins informatifs. Le fait de transformer ces points de données en aperçu exploitable était plus difficile puisque chaque jeu de données vivait indépendamment des autres.

Ces limites ont révélé l’importance d’outils intelligents d’agrégation et de visualisation des données pour accompagner le nouveau four de Contoso :

* La visualisation des données s’avère utile lorsque vous êtes en mesure d’associer et de combiner des données dans un affichage pratique. Un exemple est l’illustration de capteurs de tension et de température.
* La gestion de données multidimensionnelles pour plusieurs entités avec des fonctionnalités de comparaison, de zoom et d’intervalle de temps peut être difficile à réaliser.

**Le modèle de série chronologique offre une solution pratique** pour bon nombre des scénarios rencontrés dans cet exemple fictif :

[![Exemple de graphique de four intelligent de modèle de série chronologique](media/v2-update-tsm/time-series-model-smart-oven.png)](media/v2-update-tsm/time-series-model-smart-oven.png#lightbox)

* Le modèle de série chronologique joue un rôle essentiel dans les requêtes et la navigation, car il contextualise les données en permettant d’établir des comparaisons dans des intervalles de temps et entre les types de capteurs et d’appareils. (**A**)
* Les données sont davantage contextualisées parce que les données conservées dans le modèle de série chronologique préservent les calculs de requête de série chronologique en tant que variables et les réutilisent au moment de la requête.
* Le modèle de série chronologique organise et agrège les données pour améliorer les fonctionnalités de visualisation et de gestion. (**B**)

### <a name="key-capabilities"></a>Fonctionnalités clés

Pour faciliter la gestion de la contextualisation des séries chronologiques, Azure Time Series Insights Gen2 propose un modèle de série chronologique doté des capacités suivantes. Vous pouvez ainsi :

* créer et gérer des calculs ou des formules en tirant parti des fonctions scalaires, agréger des opérations, etc. ;
* définir des relations parent-enfant pour activer la navigation, la recherche et la référence ;
* définir les propriétés associées aux instances, définies comme *champs d’instance*, et s’en servir pour créer des hiérarchies.

### <a name="components"></a>Components

Un modèle de série chronologique comprend trois composants de base :

* [Instances de modèle de série chronologique](#time-series-model-instances)
* [Hiérarchies de modèle de série de temps](#time-series-model-hierarchies)
* [Types de modèles de série chronologique](#time-series-model-types)

Ces composants sont combinés pour spécifier un modèle de série chronologique et organiser vos données.

[![Graphique de vue d’ensemble du modèle de la série chronologique](media/v2-update-tsm/time-series-model-overview.png)](media/v2-update-tsm/time-series-model-overview.png#lightbox)

Un modèle de série chronologique peut être créé et géré par le biais de l’[Explorateur Azure Time Series Insights](). Les paramètres de modèle de série chronologique peuvent être gérés via l’[API Model Settings](/rest/api/time-series-insights/reference-model-apis).

## <a name="time-series-model-instances"></a>Instances de modèle de série chronologique

Les *instances* de modèle de série chronologique sont des représentations virtuelles de la série chronologique elle-même.

Dans la plupart des cas, les instances sont identifiées de manière unique par **deviceId** ou **assetId**, propriétés qui sont enregistrées en tant qu’ID de série chronologique.

Les instances sont associées à des informations descriptives appelées *propriétés d’instance*, telles qu’un ID de série chronologique, un type, un nom, une description, des hiérarchies et des champs d’instance. Au minimum, les propriétés d’instance incluent les informations de la hiérarchie.

Les *champs d’instance* sont un ensemble d’informations descriptives qui peuvent inclure des valeurs pour les niveaux hiérarchiques, ainsi que pour le fabricant, l’opérateur, etc.

Une fois qu’une source d’événement est configurée pour l’environnement Azure Time Series Insights Gen2, les instances sont automatiquement découvertes et créées dans un modèle de série chronologique. Elles peuvent être créées ou mises à jour via l’Explorateur Azure Time Series Insights à l’aide de requêtes de modèle de série chronologique.

La [démonstration du parc éolien de Contoso](https://insights.timeseries.azure.com/preview/samples) fournit plusieurs exemples d’instances en direct.

[![Exemple d’instance de modèle de série chronologique](media/v2-update-tsm/time-series-model-instance.png)](media/v2-update-tsm/time-series-model-instance.png#lightbox)

### <a name="instance-properties"></a>Propriétés de l’instance

Les instances sont définies par **timeSeriesId**, **typeId**, **name**, **description**, **hierarchyIds** et **instanceFields**. Chaque instance est mappée à un seul *type* et à une ou plusieurs *hiérarchies*.

| Propriété | Description |
| --- | ---|
| timeSeriesId | ID unique de la série chronologique à laquelle l’instance est associée. Dans la plupart des cas, les instances sont identifiées de manière unique par une propriété comme deviceId ou assetId. Dans certains cas, un ID composite plus spécifique combinant jusqu’à 3 propriétés peut être utilisé. |
| typeId | ID de chaîne unique respectant la casse du type de modèle de série chronologique auquel l’instance est associée. Par défaut, toutes les nouvelles instances découvertes sont associées à un type par défaut.
| name | La propriété **name** est facultative et sensible à la casse. Si **name** n’est pas disponible, la valeur par défaut est **timeSeriesId**. Si un nom est fourni, **timeSeriesId** est toujours disponible dans le [puits](./concepts-ux-panels.md#4-time-series-well). |
| description | Texte de description de l’instance. |
| hierarchyIds | Définit les hiérarchies auxquelles l’instance appartient. |
| instanceFields | Propriétés d’une instance et toutes les données statiques qui définissent une instance. Ils définissent les valeurs des propriétés d’une hiérarchie ou non et prennent en charge l’indexation pour effectuer des opérations de recherche. |

> [!NOTE]
> Les hiérarchies sont créées à l’aide de champs d’instance. D’autres **instanceFields** peuvent être ajoutés pour d’autres définitions de propriété d’instance.

Les instances ont la représentation JSON suivante :

```JSON
{
  "timeSeriesId": ["PU2"],
  "typeId": "545314a5-7166-4b90-abb9-fd93966fa39b",
  "hierarchyIds": ["95f0a8d1-a3ef-4549-b4b3-f138856b3a12"],
  "description": "Pump #2",
  "instanceFields": {
    "Location": "Redmond",
    "Fleet": "Fleet 5",
    "Unit": "Pump Unit 3",
    "Manufacturer": "Contoso",
    "ScalePres": "0.54",
    "scaleTemp": "0.54"
  }
}
```

> [!TIP]
> Pour une prise en charge des opérations CRUD (create, read, update et delete) de l’API d’instances, lisez l’article [Interrogation des données](./concepts-query-overview.md#time-series-model-query-tsm-q-apis) et la [documentation REST de l’API d’instances](/rest/api/time-series-insights/reference-model-apis#instances-api).

## <a name="time-series-model-hierarchies"></a>Hiérarchies de modèle de série de temps

Les *hiérarchies* de modèle de série chronologique classent les instances en spécifiant les noms de propriétés et leurs relations.

Vous pouvez configurer plusieurs hiérarchies dans un environnement Azure Time Series Insights Gen2 donné. Une instance de modèle de série chronologique peut être mappée à une ou plusieurs hiérarchies (relation plusieurs à plusieurs).

La [démonstration du parc éolien Contoso](https://insights.timeseries.azure.com/preview/samples) affiche une hiérarchie d’instances et de types standard.

[![Exemple de hiérarchie de modèle de série chronologique](media/v2-update-tsm/time-series-model-hierarchies.png)](media/v2-update-tsm/time-series-model-hierarchies.png#lightbox)

### <a name="hierarchy-definition"></a>Définition des hiérarchies

Les hiérarchies sont définies par **id**, **name** et **source** de hiérarchie.

| Propriété | Description |
| ---| ---|
| id | Identificateur unique de la hiérarchie, qui est utilisé, par exemple, lorsque vous définissez une instance. |
| name | Chaîne utilisée afin de fournir un nom pour la hiérarchie. |
| source | Spécifie la hiérarchie ou le chemin d’accès de l’organisation qui se présente sous la forme d’un ordre descendant parent-enfant de la hiérarchie que les utilisateurs souhaitent créer. Les propriétés parent-enfant sont mappées aux champs d’instance. |

Les hiérarchies sont représentées au format JSON comme suit :

```JSON
{
  "hierarchies": [
    {
      "id": "6e292e54-9a26-4be1-9034-607d71492707",
      "name": "Location",
      "source": {
        "instanceFieldNames": [
          "state",
          "city"
        ]
      }
    },
    {
      "id": "a28fd14c-6b98-4ab5-9301-3840f142d30e",
      "name": "ManufactureDate",
      "source": {
        "instanceFieldNames": [
          "year",
          "month"
        ]
      }
    }
  ]
}
```

Dans l’exemple JSON précédent :

* `Location` définit une hiérarchie avec une `states` parent et un `cities` enfant. Chaque `location` peut avoir plusieurs `states`, lesquelles peuvent avoir à leur tour plusieurs `cities`.
* `ManufactureDate` définit une hiérarchie avec une `year` parent et un `month` enfant. Chaque `ManufactureDate` peut avoir plusieurs `years`, lesquelles peuvent avoir à leur tour plusieurs `months`.

> [!TIP]
> Pour une prise en charge des opérations CRUD (create, read, update et delete) de l’API de hiérarchie, lisez l’article [Interrogation des données](concepts-query-overview.md#time-series-model-query-tsm-q-apis) et la [documentation REST de l’API de hiérarchie](/rest/api/time-series-insights/reference-model-apis#hierarchies-api).

### <a name="hierarchy-example"></a>Exemple de hiérarchie

Prenons un exemple dans lequel la hiérarchie **H1** a `building`, `floor` et `room` dans le cadre de sa définition **instanceFieldNames** :

```JSON
{
  "id": "aaaaaa-bbbbb-ccccc-ddddd-111111",
  "name": "H1",
  "source": {
    "instanceFieldNames": [
      "building",
      "floor",
      "room"
    ]
  }
}
```

Étant donné les champs d’instance utilisés dans la définition précédente et plusieurs séries chronologiques, les attributs et les valeurs de la hiérarchie apparaissent comme indiqué dans le tableau suivant :

| ID Time Series | Champs d'instance |
| --- | --- |
| ID1 | "building" = "1000", "floor" = "10", "room" = "55"  |
| ID2 | "building" = "1000", "room" = "55" |
| ID3 | "floor" = "10" |
| ID4 | "building" = "1000", "floor" = "10"  |
| ID5 | Aucune des valeurs « building », « floor » ou « room » n’est définie. |

Les séries chronologiques **ID1** et **ID4** sont affichées dans la hiérarchie **H1** dans l’[Explorateur Azure Time Series Insights](./concepts-ux-panels.md), car elles ont des paramètres *building*, *floor* et *room* entièrement définis et correctement ordonnés.

Les autres sont classés sous *Instances sans parent*, car elles ne sont pas conformes à la hiérarchie de données spécifiée.

## <a name="time-series-model-types"></a>Types de modèles de série chronologique

Les *types* de modèles de série chronologique vous aident à définir des variables ou des formules permettant d’effectuer des calculs. Les types sont associés à une instance spécifique.

Un type peut avoir une ou plusieurs variables. Par exemple, une instance de modèle de série chronologique peut être de type *Capteur de température*, qui se compose des variables *avg temperature* (température moyenne), *min temperature* (température minimale) et *max temperature* (température maximale).

La [démonstration du parc éolien Contoso](https://insights.timeseries.azure.com/preview/samples) permet de visualiser plusieurs types de modèles de série chronologique associés à leurs instances respectives.

[![Exemple de type de modèle de série chronologique](media/v2-update-tsm/time-series-model-types.png)](media/v2-update-tsm/time-series-model-types.png#lightbox)

> [!TIP]
> Pour une prise en charge des opérations CRUD (create, read, update et delete) de l’API de types, lisez l’article [Interrogation des données](concepts-query-overview.md#time-series-model-query-tsm-q-apis) et la [documentation REST de l’API de types](/rest/api/time-series-insights/reference-model-apis#types-api).

### <a name="type-properties"></a>Propriétés type

Les types de modèles de série chronologique sont définis par **id**, **name**, **description** et **variables**.

| Propriété | Description |
| ---| ---|
| id | ID de chaîne unique respectant la casse du type. |
| name | Chaîne utilisée afin de fournir un nom pour le type. |
| description | Description sous forme de chaîne pour le type. |
| variables | Spécifient les variables associées au type. |

Les types sont conformes à l’exemple JSON suivant :

```JSON
{
  "types": [
    {
      "id": "1be09af9-f089-4d6b-9f0b-48018b5f7393",
      "name": "DefaultType",
      "description": "Default type",
      "variables": {
        "EventCount": {
          "kind": "aggregate",
          "value": null,
          "filter": null,
          "aggregation": {
            "tsx": "count()"
          }
        },
        "Interpolated Speed": {
          "kind": "numeric",
          "value": {
              "tsx": "$event['Speed-Sensor'].Double"
          },
          "filter": null,
          "interpolation": {
              "kind": "step",
              "boundary": {
                  "span": "P1D"
              }
          },
          "aggregation": {
              "tsx": "right($value)"
          }
        }
      }
    }
  ]
}
```

Les types de modèles de série chronologique peuvent avoir de nombreuses variables qui spécifient des règles de formule et de calcul sur les événements. En savoir plus sur la [façon de définir des variables de modèle de série chronologique](./concepts-variables.md)

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur la modification du modèle à l’aide d’API, consultez la documentation de référence du [modèle de série chronologique](/rest/api/time-series-insights/reference-model-apis).

* Explorez les formules et les calculs que vous pouvez créer à l’aide de [variables de modèle de série chronologique](./concepts-variables.md).

* En savoir plus sur l’[interrogation des données](concepts-query-overview.md) dans Azure Time Series Insights Gen2.