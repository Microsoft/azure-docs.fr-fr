---
title: Prise en charge du type de données long dans Azure Time Series Insights Gen2 | Microsoft Docs
description: Prise en charge du type de données long dans Azure Time Series Insights Gen2.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.workload: big-data
ms.topic: conceptual
ms.date: 12/07/2020
ms.custom: dpalled
ms.openlocfilehash: 0f7ac9844bfe7eac7094d3b7fdf653e07f236599
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96780822"
---
# <a name="adding-support-for-long-data-type-in-azure-time-series-insights-gen2"></a>Ajout de la prise en charge du type de données long dans Azure Time Series Insights Gen2

L’ajout de la prise en charge du type de données long affecte la façon dont nous stockons et indexons des données numériques uniquement dans des environnements Azure Time Series Insights Gen2. Si vous avez un environnement Gen1, vous pouvez ignorer ces modifications.

Depuis le 29 juin ou 30 juin 2020, selon votre région, vos données sont indexées en tant que **Longues** et **Doubles**.  Si vous avez des questions ou des préoccupations concernant cette modification, soumettez un ticket de support par le biais du portail Azure et mentionnez cette communication.

Si vous êtes concerné par l’un des cas suivants, apportez les modifications recommandées :

- **Cas n° 1** : vous utilisez actuellement des variables de modèle de série chronologique et n’envoyez que des types de données intégrales dans vos données de télémétrie.
- **Cas n° 2** : vous utilisez actuellement des variables de modèle de série chronologique et envoyez des types de données intégrales et non intégrales dans vos données de télémétrie.
- **Cas n° 3** : vous utilisez des variables catégorielles pour mapper des valeurs entières à des catégories.
- **Cas n° 4** : vous utilisez le Kit de développement logiciel (SDK) JavaScript pour créer une application frontale personnalisée.
- **Cas N° 5** : vous vous approchez de la limite de 1 000 noms de propriété dans le magasin Warm et envoyez des données intégrales et non intégrales. Vous pouvez afficher le nombre de propriétés sous la forme d’une métrique dans le [portail Azure](https://portal.azure.com/).

Si l’un des cas s’applique à vous, apportez des modifications à votre modèle. Mettez à jour l’expression de série chronologique (TSX) dans votre définition de variable en apportant les modifications recommandées. Mettez à jour :

- Explorateur Azure Time Series Insights
- tout client personnalisé qui utilise nos API.

En fonction de votre solution IoT et de vos contraintes, vous risquez de ne pas avoir de visibilité des données envoyées à votre environnement Azure Time Series Insights Gen2. Si vous ignorez si vos données sont uniquement intégrales ou à la fois intégrales et non intégrales, plusieurs options s’offrent à vous.

- Vous pouvez attendre que la fonctionnalité soit mise en production. Explorez ensuite vos événements bruts dans l’interface utilisateur de l’Explorateur pour comprendre les propriétés qui sont enregistrées dans deux colonnes distinctes.
- Vous pouvez apporter par anticipation les modifications recommandées à toutes les étiquettes numériques.
- Vous pouvez acheminer temporairement un sous-ensemble d’événements vers le stockage pour mieux comprendre et explorer votre schéma.

Pour stocker des événements, activez la [capture d’événements](../event-hubs/event-hubs-capture-overview.md) pour Azure Event Hub ou le [routage](../iot-hub/iot-hub-devguide-messages-d2c.md#azure-storage-as-a-routing-endpoint) à partir de votre IoT Hub vers le Stockage Blob Azure.

Les données peuvent également être observées par le biais de l’[explorateur Event Hub](https://marketplace.visualstudio.com/items?itemName=Summer.azure-event-hub-explorer) ou à l’aide de l’[hôte du processeur d’événements](../event-hubs/event-hubs-dotnet-standard-getstarted-send.md#receive-events).

Si vous utilisez IoT Hub, accédez à [Lire des messages appareil-à-cloud à partir du point de terminaison intégré](../iot-hub/iot-hub-devguide-messages-read-builtin.md) pour savoir comment accéder au point de terminaison intégré.

> [!NOTE]
> Vous pouvez rencontrer une interruption si vous n’apportez pas les modifications recommandées. Par exemple, les variables de Time Series Insights affectées accessibles via les API de requête ou de l’Explorateur Time Series Insights retournent la valeur **null** (autrement dit, n’affiche pas de données dans l’Explorateur).

## <a name="recommended-changes"></a>Modifications recommandées

### <a name="case-1-using-time-series-model-variables-and-sending-only-integral-data-types-in-telemetry-data"></a>Cas n° 1 : utilisation de variables de modèle de série chronologique et envoi uniquement de types de données intégrales dans les données de télémétrie.

Les modifications recommandées pour le cas n° 1 sont les mêmes que pour le cas n° 2. Suivez les instructions du cas n° 2.

### <a name="case-2-using-time-series-model-variables-and-sending-both-integral-and-nonintegral-types-in-telemetry-data"></a>Cas n° 2 : Utilisation de variables de modèle de série chronologique et envoi de types de données de données intégrales et non intégrales dans les données de télémétrie.

Si vous envoyez actuellement des données de télémétrie entières, vos données sont divisées en deux colonnes :

- **propertyValue_double**
- **propertyValue_long**

Vos données entière sont écrites dans **propertyValue_long**. Les données numériques ingérées précédemment (et ultérieurement) dans **propertyValue_double** ne sont pas copiées.

Si vous souhaitez interroger les données figurant dans ces deux colonnes pour la propriété **propertyValue**, vous devez utiliser la fonction scalaire **coalesce()** dans votre TSX. La fonction accepte les arguments du même **DataType** et retourne la première valeur non null de la liste d’arguments. Pour plus d’informations, consultez [Concepts d’accès aux données Azure Time Series Insights Gen2](/rest/api/time-series-insights/reference-time-series-expression-syntax#other-functions).

#### <a name="variable-definition-in-tsx---numeric"></a>Définition de variable dans TSX – Numérique

*Définition précédente de la variable :*

[![Screenshot shows the Add a new variable dialog box for the PropertyValue Variable, numeric.](media/time-series-insights-long-data-type/var-def-previous.png)](media/time-series-insights-long-data-type/var-def-previous.png#lightbox)

*Nouvelle définition de la variable :*

[![Screenshot shows the Add a new variable dialog box for the PropertyValue Variable with a custom value, numeric.](media/time-series-insights-long-data-type/var-def.png)](media/time-series-insights-long-data-type/var-def.png#lightbox)

Vous pouvez également utiliser **coalesce($event.propertyValue.Double, toDouble($event.propertyValue.Long))** comme [expression de série chronologique](/rest/api/time-series-insights/reference-time-series-expression-syntax) personnalisée.

#### <a name="inline-variable-definition-using-tsx-query-apis---numeric"></a>Définition de variable Inline à l’aide d’API de requête TSX – Numérique

*Définition précédente de la variable :*

```JSON
"PropertyValueVariable": {

    "kind": "numeric",

    "value": {

        "tsx": "$event.propertyValue.Double"

    },

    "filter": null,

    "aggregation": {

        "tsx": "avg($value)"
    }
}
```

*Nouvelle définition de la variable :*

```JSON
"PropertyValueVariable ": {

    "kind": "numeric",

    "value": {

        "tsx": "coalesce($event.propertyValue.Long, toLong($event.propertyValue.Double))"

    },

    "filter": null,

    "aggregation": {

        "tsx": "avg($value)"
    }
}
```

Vous pouvez également utiliser **coalesce($event.propertyValue.Double, toDouble($event.propertyValue.Long))** comme [expression de série chronologique](/rest/api/time-series-insights/reference-time-series-expression-syntax) personnalisée.

> [!NOTE]
> Nous vous recommandons de mettre à jour ces variables dans tous les emplacements où elles pourraient être utilisées. Ces emplacements incluent un modèle de série chronologique, des requêtes enregistrées et les requêtes de connecteur Power BI.

### <a name="case-3-using-categorical-variables-to-map-integer-values-to-categories"></a>Cas n° 3 : Utilisation de variables catégorielles pour mapper des valeurs entières à des catégories.

Si vous utilisez actuellement des variables de catégorie qui mappent des valeurs entières à des catégories, vous utilisez probablement la fonction **toLong** pour convertir des données de type **Double** en type **Long**. Tout comme les cas 1 et 2, vous devez fusionner les colonnes **DataType** **Double** et **Long**.

#### <a name="variable-definition-in-time-series-explorer---categorical"></a>Définition de variable dans l’Explorateur de série chronologique – Catégoriel

*Définition précédente de la variable :*

[![Screenshot shows the Add a new variable dialog box for the PropertyValue Variable, categorical.](media/time-series-insights-long-data-type/var-def-cat-previous.png)](media/time-series-insights-long-data-type/var-def-cat-previous.png#lightbox)

*Nouvelle définition de la variable :*

[![Screenshot shows the Add a new variable dialog box for the PropertyValue Variable with a custom value, categorical.](media/time-series-insights-long-data-type/var-def-cat.png)](media/time-series-insights-long-data-type/var-def-cat.png#lightbox)

Vous pouvez également utiliser **coalesce($event.propertyValue.Double, toDouble($event.propertyValue.Long))** comme [expression de série chronologique](/rest/api/time-series-insights/preview#time-series-expression-and-syntax) personnalisée.

Les variables de catégorie requièrent toujours une valeur de type entier. Le **DataType** de tous les arguments dans **coalesce()** doit être de type **Long** dans l’[expression de série chronologique](/rest/api/time-series-insights/reference-time-series-expression-syntax) personnalisée.

#### <a name="inline-variable-definition-using-tsx-query-apis---categorical"></a>Définition de variable incluse à l’aide d’API de requête TSX – Catégorielle

*Définition précédente de la variable :*

```JSON
"PropertyValueVariable_Long": {

    "kind": "categorical",

    "value": {

        "tsx": "tolong($event.propertyValue.Double)"

    },

    "categories": [

    {
        "label": "Good",

        "values": [0, 1, 2 ]

    },

    {

        "label": "Bad",

        "values": [ 3, 4 ]

    } ],

    "defaultCategory": {

        "label": "Unknown"

    }
}
```

*Nouvelle définition de la variable :*

```JSON
"PropertyValueVariable_Long": {

    "kind": "categorical",

    "value": {

        "tsx": "coalesce($event.propertyValue.Long, tolong($event.propertyValue.Double))"

    },

    "categories": [

    {
        "label": "Good",

        "values": [0, 1, 2 ]

    },

    {

        "label": "Bad",

        "values": [ 3, 4 ]

    } ],

    "defaultCategory": {

        "label": "Unknown"

    }
}
```

Les variables de catégorie requièrent toujours une valeur de type entier. Le **DataType** de tous les arguments dans **coalesce()** doit être de type **Long** dans l’[expression de série chronologique](/rest/api/time-series-insights/reference-time-series-expression-syntax) personnalisée.

> [!NOTE]
> Nous vous recommandons de mettre à jour ces variables dans tous les emplacements où elles pourraient être utilisées. Ces emplacements incluent un modèle de série chronologique, des requêtes enregistrées et les requêtes de connecteur Power BI.

### <a name="case-4-using-the-javascript-sdk-to-build-a-custom-front-end-application"></a>Cas 4 : utilisation du kit de développement logiciel (SDK) JavaScript pour créer une application frontale personnalisée.

Si vous êtes concerné par les cas 1 à 3 ci-dessus et si vous créez des applications personnalisées, vous devez mettre à jour vos requêtes de manière à utiliser la fonction **coalesce()** , comme illustré dans les exemples précédents.

### <a name="case-5-nearing-warm-store-1000-property-limit"></a>Case 5 : Limite de 1 000 propriétés du magasin Warm presque atteinte

Si vous êtes un utilisateur du magasin Warm avec un grand nombre de propriétés et pensez que cette modification pousserait votre environnement au-delà de la limite de 1 000 noms de propriétés du magasin Warm, soumettez un ticket de support via le portail Azure en mentionnant cette communication.

## <a name="next-steps"></a>Étapes suivantes

- Consultez la liste complète des [types de données pris en charge](concepts-supported-data-types.md).
