---
title: Envoyer des événements à un environnement – Azure Time Series Insights | Microsoft Docs
description: Apprenez à configurer un Event Hub, à exécuter un exemple d’application et à envoyer des événements à votre environnement Azure Time Series Insights.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.devlang: csharp
ms.workload: big-data
ms.topic: conceptual
ms.date: 09/30/2020
ms.custom: seodec18
ms.openlocfilehash: 9a9115b5400cc6d6c1ecc5740af796d831f5dee3
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95023256"
---
# <a name="send-events-to-an-azure-time-series-insights-gen1-environment-by-using-an-event-hub"></a>Envoyer des événements à un environnement Azure Time Series Insights Gen1 Insights à l’aide d’un hub d’événements

> [!CAUTION]
> Il s’agit d’un article Gen1.

Ce tutoriel explique comment créer et configurer un hub d’événements dans Azure Event Hubs. Il traite également de l’exécution d’un exemple d’application pour envoyer (push) des événements dans Azure Time Series Insights à partir d’Event Hubs. Si vous disposez d’un concentrateur d’événements existant qui a déjà des événements au format JSON, ignorez ce didacticiel et affichez votre environnement dans [Azure Time Series Insights](./tutorials-set-up-tsi-environment.md).

## <a name="configure-an-event-hub"></a>Configurer un concentrateur d’événements

1. Pour en savoir plus sur la création d'un hub d'événements, consultez la [documentation Event Hubs](../event-hubs/index.yml).
1. Dans la zone de recherche, recherchez **Event Hubs**. Dans la liste retournée, sélectionnez **Event Hubs**.
1. Sélectionnez votre hub d'événements.
1. Lorsque vous créez un hub d’événements, vous créez un espace de noms de hub d’événements. Si vous n’avez pas encore créé de hub d’événements dans l’espace de noms, accédez au menu et, sous **Entités**, créez-en un.  

    [![Liste de hubs d’événements](media/send-events/tsi-connect-event-hub-namespace.png)](media/send-events/tsi-connect-event-hub-namespace.png#lightbox)

1. Une fois votre hub d'événements créé, sélectionnez-le dans la liste.
1. Dans le menu, sous **Entités**, sélectionnez **Event Hubs**.
1. Sélectionnez le nom du hub d'événements pour le configurer.
1. Sous **Vue d’ensemble**, sélectionnez **Groupes de consommateurs**, puis **Groupe de consommateurs**.

    [![Créer un groupe de consommateurs](media/send-events/add-event-hub-consumer-group.png)](media/send-events/add-event-hub-consumer-group.png#lightbox)

1. Veillez à créer un groupe de consommateurs qui sera utilisé exclusivement par votre source d’événement Azure Time Series Insights.

    > [!IMPORTANT]
    > Assurez-vous que ce groupe de consommateurs n’est pas utilisé par un autre service, par exemple, une tâche Azure Stream Analytics ou un autre environnement Azure Time Series Insights. Si le groupe de consommateurs est utilisé par les autres services, les opérations de lecture sont affectées pour cet environnement et pour les autres services. Si vous utilisez le groupe de consommateurs **$Default**, d'autres lecteurs risquent de pouvoir réutiliser votre groupe de consommateurs.

1. Dans le menu, sous **Paramètres**, sélectionnez **Stratégies d’accès partagé**, puis **Ajouter**.

    [![Sélectionnez Stratégies d’accès partagé, puis le bouton Ajouter](media/send-events/add-shared-access-policy.png)](media/send-events/add-shared-access-policy.png#lightbox)

1. Dans le volet **Ajouter une nouvelle stratégie d'accès partagé**, créez un accès partagé intitulé **MySendPolicy**. Vous utilisez cette stratégie d’accès partagé pour envoyer des événements dans les exemples C# plus loin dans cet article.

    [![Dans la zone Nom de la stratégie, entrez MySendPolicy](media/send-events/configure-shared-access-policy-confirm.png)](media/send-events/configure-shared-access-policy-confirm.png#lightbox)

1. Sous **Revendication**, cochez la case **Envoyer**.

## <a name="add-an-azure-time-series-insights-instance"></a>Ajoutez une instance Azure Time Series Insights

Dans Azure Time Series Insights Gen2, vous pouvez ajouter des données contextuelles à la télémétrie entrante à l’aide de Time Series Insights (TSM). Dans TSM, vos balises ou vos signaux sont référencés en tant qu’*instances,* et vous pouvez stocker des données contextuelles dans *champs d’instance.* Les données sont jointes au moment de la requête à l'aide d'un **ID de série chronologique**. L’**ID de série chronologique** de l’exemple de projet d’éoliennes que nous utiliserons plus loin dans cet article est `id`. Pour en savoir plus sur le stockage des données dans les champs d’instance, consultez la présentation [Time Series Model](./concepts-model-overview.md).

### <a name="create-an-azure-time-series-insights-event-source"></a>Créer une source d’événement Azure Time Series Insights

1. Si vous n'avez créé aucune source d'événement, suivez la procédure de [création d'une source d'événement](./how-to-ingest-data-event-hub.md).

1. Définissez une valeur pour `timeSeriesId`. Pour en savoir plus sur l'**ID de série chronologique**, consultez [Modèles de séries chronologiques](./concepts-model-overview.md).

### <a name="push-events-to-windmills-sample"></a>Envoyer (push) des événements vers l’exemple de projet d’éoliennes

1. Sur la barre de recherche, recherchez **Event Hubs**. Dans la liste retournée, sélectionnez **Event Hubs**.

1. Sélectionnez votre instance d’hub d’événements.

1. Accédez à **Stratégies d’accès partagé** > **MySendPolicy**. Copiez la valeur de la **Chaîne de connexion-clé primaire**.

    [![Copier la valeur de la chaîne de connexion de la clé primaire](media/send-events/configure-sample-code-connection-string.png)](media/send-events/configure-sample-code-connection-string.png#lightbox)

1. Atteindre <https://tsiclientsample.azurewebsites.net/windFarmGen.html>. L’URL crée et exécute des simulations d’éoliennes.
1. Dans la zone **Chaîne de connexion du hub d’événements** de la page web, collez la chaîne de connexion que vous avez copiée dans le [champ d’entrée des éoliennes](#push-events-to-windmills-sample).
  
    [![Coller la chaîne de connexion de la clé primaire dans la zone Chaîne de connexion Event Hub](media/send-events/configure-wind-mill-sim.png)](media/send-events/configure-wind-mill-sim.png#lightbox)

1. Sélectionnez **Cliquez pour démarrer**.

    > [!TIP]
    > Le simulateur d’éoliennes crée également un fichier JSON que vous pouvez utiliser comme charge utile avec les [API de requête en disponibilité générale Azure Time Series Insights](/rest/api/time-series-insights/gen1-query).

    > [!NOTE]
    > Le simulateur continuera à envoyer des données jusqu’à ce que l’onglet du navigateur soit fermé.

1. Revenez à votre hub d'événements sur le portail Azure. La page **Aperçu** affiche les nouveaux événements reçus par Event Hub.

    [![Page d’aperçu du hub d’événements affichant les métriques de celui-ci](media/send-events/review-windmill-telemetry.png)](media/send-events/review-windmill-telemetry.png#lightbox)

## <a name="supported-json-shapes"></a>Structures JSON prises en charge

### <a name="example-one"></a>Premier exemple

* **Entrée**: Un objet JSON simple.

    ```JSON
    {
        "id":"device1",
        "timestamp":"2016-01-08T01:08:00Z"
    }
    ```

* **Sortie**: Un événement.

    |id|timestamp|
    |--------|---------------|
    |device1|2016-01-08T01:08:00Z|

### <a name="example-two"></a>Deuxième exemple

* **Entrée**: Un tableau JSON avec deux objets JSON. Chaque objet JSON est converti en événement.

    ```JSON
    [
        {
            "id":"device1",
            "timestamp":"2016-01-08T01:08:00Z"
        },
        {
            "id":"device2",
            "timestamp":"2016-01-17T01:17:00Z"
        }
    ]
    ```

* **Sortie**: Deux événements.

    |id|timestamp|
    |--------|---------------|
    |device1|2016-01-08T01:08:00Z|
    |device2|2016-01-08T01:17:00Z|

### <a name="example-three"></a>Troisième exemple

* **Entrée**: Objet JSON avec tableau JSON imbriqué contenant deux objets JSON.

    ```JSON
    {
        "location":"WestUs",
        "events":[
            {
                "id":"device1",
                "timestamp":"2016-01-08T01:08:00Z"
            },
            {
                "id":"device2",
                "timestamp":"2016-01-17T01:17:00Z"
            }
        ]
    }
    ```

* **Sortie**: Deux événements. La propriété **location** est copiée dans chaque événement.

    |location|events.id|events.timestamp|
    |--------|---------------|----------------------|
    |WestUs|device1|2016-01-08T01:08:00Z|
    |WestUs|device2|2016-01-08T01:17:00Z|

### <a name="example-four"></a>Quatrième exemple

* **Entrée**: Objet JSON avec tableau JSON imbriqué contenant deux objets JSON. Cette entrée montre que les propriétés globales peuvent être représentées par l'objet JSON complexe.

    ```JSON
    {
        "location":"WestUs",
        "manufacturer":{
            "name":"manufacturer1",
            "location":"EastUs"
        },
        "events":[
            {
                "id":"device1",
                "timestamp":"2016-01-08T01:08:00Z",
                "data":{
                    "type":"pressure",
                    "units":"psi",
                    "value":108.09
                }
            },
            {
                "id":"device2",
                "timestamp":"2016-01-17T01:17:00Z",
                "data":{
                    "type":"vibration",
                    "units":"abs G",
                    "value":217.09
                }
            }
        ]
    }
    ```

* **Sortie**: Deux événements.

    |location|manufacturer.name|manufacturer.location|events.id|events.timestamp|events.data.type|events.data.units|events.data.value|
    |---|---|---|---|---|---|---|---|
    |WestUs|manufacturer1|EastUs|device1|2016-01-08T01:08:00Z|pression|psi|108.09|
    |WestUs|manufacturer1|EastUs|device2|2016-01-08T01:17:00Z|vibration|abs G|217.09|

## <a name="next-steps"></a>Étapes suivantes

* [Affichez votre environnement](https://insights.timeseries.azure.com) dans l’Explorateur Time Series Insights.

* En savoir plus sur les [messages de l’appareil IoT Hub](../iot-hub/iot-hub-devguide-messages-construct.md)