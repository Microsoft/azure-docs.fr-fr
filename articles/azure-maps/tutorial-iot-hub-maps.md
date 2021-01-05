---
title: 'Tutoriel : Implémenter l’analytique spatiale IoT | Microsoft Azure Maps'
description: Tutoriel sur l’intégration d’IoT Hub aux API du service Microsoft Azure Maps
author: anastasia-ms
ms.author: v-stharr
ms.date: 09/01/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc
ms.openlocfilehash: d475c76d338270cb9b8e34ea8563cdfd8fdf5122
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97563056"
---
# <a name="tutorial-implement-iot-spatial-analytics-by-using-azure-maps"></a>Tutoriel : Implémenter l’analytique spatiale IoT avec Azure Maps

Dans un scénario IoT, il est courant de capturer et de suivre les événements pertinents qui se produisent dans l’espace et le temps. Les applications de gestion de flotte, de gestion des biens, de mobilité et de ville intelligente constituent des exemples. Ce tutoriel décrit une solution qui effectue le suivi des déplacements d’une voiture de location à l’aide des API Azure Maps.

Dans ce tutoriel vous allez :

> [!div class="checklist"]
> * Créer un compte Stockage Azure pour journaliser les données de suivi de véhicule.
> * Charger une limite géographique sur le service Data d’Azure Maps (préversion) à l’aide de l’API de chargement de données.
> * Créer un hub dans Azure IoT Hub et inscrire un appareil.
> * Créer une fonction dans Azure Functions, en implémentant une logique métier basée sur l’analytique spatiale Azure Maps.
> * Vous abonner aux événements de télémétrie de l’appareil IoT à partir de la fonction Azure par le biais d’Azure Event Grid.
> * Filtrer les événements de télémétrie à l’aide du routage des messages IoT Hub.

## <a name="prerequisites"></a>Prérequis

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. [Créez un compte Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account).

3. [Obtenir une clé d’abonnement principale](quick-demo-map-app.md#get-the-primary-key-for-your-account), également appelée clé primaire ou clé d’abonnement. Pour plus d’informations, consultez [Gérer l’authentification dans Azure Maps](how-to-manage-authentication.md).

4. [Créez un groupe de ressources](../azure-resource-manager/management/manage-resource-groups-portal.md#create-resource-groups). Dans ce tutoriel, nous allons nommer notre groupe de ressources *ContosoRental*, mais vous pouvez choisir le nom de votre choix.

5. Téléchargez le [projet C# rentalCarSimulation](https://github.com/Azure-Samples/iothub-to-azure-maps-geofencing/tree/master/src/rentalCarSimulation).

Ce tutoriel utilise l’application [Postman](https://www.postman.com/), mais vous pouvez choisir un autre environnement de développement d’API.

## <a name="use-case-rental-car-tracking"></a>Cas d’usage : Suivi de voiture de location

Imaginez qu’une société de location de voitures souhaite journaliser les informations sur la localisation, les distances parcourues et l’état de fonctionnement de ses voitures de location. Elle souhaite par ailleurs stocker ces informations à chaque fois qu’une voiture quitte la région géographique autorisée.

Les voitures de location sont équipées d’appareils IoT qui envoient régulièrement des données de télémétrie à IoT Hub. La télémétrie comprend la localisation actuelle et indique si le moteur de la voiture tourne. Le schéma d’emplacement des appareils est conforme au [schéma Plug-and-Play des données géospatiales](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v1-preview/schemas/geospatial.md) IoT. Le schéma de télémétrie des appareils de la voiture de location est semblable au code JSON suivant :

```JSON
{
    "data": {
        "properties": {
            "Engine": "ON"
        },
        "systemProperties": {
            "iothub-content-type": "application/json",
            "iothub-content-encoding": "utf-8",
            "iothub-connection-device-id": "ContosoRentalDevice",
            "iothub-connection-auth-method": "{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
            "iothub-connection-auth-generation-id": "636959817064335548",
            "iothub-enqueuedtime": "2019-06-18T00:17:20.608Z",
            "iothub-message-source": "Telemetry"
        },
        "body": {
            "location": {
                "type": "Point",
                "coordinates": [ -77.025988698005662, 38.9015330523316 ]
            }
        }
    }
}
```

Dans ce tutoriel, vous n’allez suivre qu’un seul véhicule. Après avoir configuré les services Azure, vous devez télécharger le [projet C# rentalCarSimulation](https://github.com/Azure-Samples/iothub-to-azure-maps-geofencing/tree/master/src/rentalCarSimulation) pour exécuter le simulateur de véhicule. L’ensemble du processus, de l’événement à l’exécution de la fonction, est résumé dans les étapes suivantes :

1. L’appareil embarqué envoie des données de télémétrie à IoT Hub.

2. Si le moteur de la voiture est allumé, le hub publie les données de télémétrie sur Event Grid.

3. Une fonction Azure est déclenchée en raison de son abonnement aux événements de télémétrie de l’appareil.

4. La fonction journalise les coordonnées de la localisation de l’appareil du véhicule, l’heure de l’événement et l’ID de l’appareil. Elle utilise ensuite l’[API d’obtention de limite géographique spatiale](/rest/api/maps/spatial/getgeofence) pour déterminer si la voiture est allée à l’extérieur de la limite géographique. Si c’est le cas, la fonction stocke les données de localisation reçues de l’événement dans un conteneur d’objets blob. Elle interroge également la [Recherche d’adresse inversée](/rest/api/maps/search/getsearchaddressreverse) pour convertir les coordonnées en adresse postale et stocker celle-ci avec les autres données de localisation de l’appareil.

Le diagramme suivant montre une vue d’ensemble du système.

   :::image type="content" source="./media/tutorial-iot-hub-maps/system-diagram.png" border="false" alt-text="Diagramme de vue d’ensemble du système.":::

La figure ci-dessous met en évidence la zone de limite géographique en bleu. L’itinéraire du véhicule de location est indiqué par une ligne verte.

   :::image type="content" source="./media/tutorial-iot-hub-maps/geofence-route.png" border="false" alt-text="Figure présentant l’itinéraire à travers la limite géographique.":::

## <a name="create-an-azure-storage-account"></a>Créer un compte de stockage Azure

Pour stocker les données de suivi des infractions de la voiture, créez un [compte de stockage universel v2](../storage/common/storage-account-overview.md#general-purpose-v2-accounts) dans votre groupe de ressources. Si vous n’avez pas créé de groupe de ressources, suivez les instructions de la section [Créer un groupe de ressources](../azure-resource-manager/management/manage-resource-groups-portal.md#create-resource-groups). Dans ce tutoriel, vous allez nommer votre groupe de ressources *ContosoRental*.

Pour créer un compte de stockage, suivez les instructions de la section [Créer un compte de stockage](../storage/common/storage-account-create.md?tabs=azure-portal). Dans ce tutoriel, nommez le compte de stockage *contosorentalstorage* (en règle générale, vous pouvez le nommer comme vous le souhaitez).

Une fois le compte de stockage créé, vous devez créer un conteneur pour stocker les données de journalisation.

1. Accédez au compte de stockage que vous venez de créer. Dans la section **Essentials**, sélectionnez le lien **Conteneurs**.

    :::image type="content" source="./media/tutorial-iot-hub-maps/containers.png" alt-text="Capture d’écran des conteneurs pour le stockage d’objets blob.":::

2. Dans le coin supérieur gauche, sélectionnez **+ Conteneur**. Un panneau s’affiche sur le côté droit du navigateur. Nommez votre conteneur *contoso-rental-logs* et sélectionnez **Créer**.

     :::image type="content" source="./media/tutorial-iot-hub-maps/container-new.png" alt-text="Capture d’écran de la création d’un conteneur d’objets blob":::

3. Accédez au volet **Clés d’accès** de votre compte de stockage et copiez le **nom du compte de stockage** et la valeur **Clé** dans la section **key1**. Vous aurez besoin de ces deux valeurs dans la section « Créer une fonction Azure et ajouter un abonnement Event Grid ».

    :::image type="content" source="./media/tutorial-iot-hub-maps/access-keys.png" alt-text="Capture d’écran de la copie du nom du compte de stockage et de la clé.":::

## <a name="upload-a-geofence"></a>Charger une limite géographique

Ensuite, utilisez l’[application Postman](https://www.getpostman.com) pour [charger la limite géographique](./geofence-geojson.md) dans Azure Maps. La limite géographique définit la zone géographique autorisée pour notre véhicule de location. Vous utiliserez la limite géographique dans votre fonction Azure pour déterminer si une voiture est allée en dehors de la limite géographique.

Effectuez les étapes suivantes pour charger la limite géographique à l’aide de l’API de chargement de données Azure Maps : 

1. Ouvrez l’application Postman et sélectionnez **New**. Dans la fenêtre **Create New** (Créer nouveau), sélectionnez **Collection**. Nommez la collection, puis sélectionnez **Create**.

2. Pour créer la demande, sélectionnez **New** à nouveau. Dans la fenêtre **Create New**, sélectionnez **Request**, puis entrez le nom de la requête. Sélectionnez la collection que vous avez créée à l’étape précédente, puis sélectionnez **Enregistrer**.

3. Sélectionnez la méthode HTTP **POST** sous l’onglet du générateur, puis entrez l’URL suivante pour charger la limite géographique dans l’API de chargement de données. Remplacez `{subscription-key}` par la clé de votre abonnement principal.

    ```HTTP
    https://atlas.microsoft.com/mapData/upload?subscription-key={subscription-key}&api-version=1.0&dataFormat=geojson
    ```

    Dans le chemin d’URL, la valeur `geojson` définie pour le paramètre `dataFormat` représente le format des données en cours de chargement.

4. Sélectionnez **Body** > **raw** comme format d’entrée, puis choisissez **JSON** dans la liste déroulante. [Ouvrez le fichier de données JSON](https://raw.githubusercontent.com/Azure-Samples/iothub-to-azure-maps-geofencing/master/src/Data/geofence.json?token=AKD25BYJYKDJBJ55PT62N4C5LRNN4) et copiez le code JSON dans la section du corps. Sélectionnez **Envoyer**.

5. Sélectionnez **Send** et attendez que la requête soit traitée. Une fois la requête terminée, accédez à l’onglet **En-têtes** de la réponse. Copiez la valeur de la **Emplacement**, qui est l’URL `status URL`.

    ```http
    https://atlas.microsoft.com/mapData/operations/<operationId>?api-version=1.0
    ```

6. Pour vérifier l’état de l’appel d’API, créez une requête HTTP **GET** sur `status URL`. Vous devez ajouter votre clé d’abonnement principale à l’URL pour l’authentification. La requête **GET** doit ressembler à l’URL suivante :

   ```HTTP
   https://atlas.microsoft.com/mapData/<operationId>/status?api-version=1.0&subscription-key={subscription-key}
   ```
   
7. Lorsque la requête HTTP **GET** se termine avec succès, elle retourne un `resourceLocation`. Le `resourceLocation` contient également la valeur unique `udid` pour les données chargées. Copiez ce `udid` pour une utilisation ultérieure dans ce tutoriel.

      ```json
      {
          "status": "Succeeded",
          "resourceLocation": "https://atlas.microsoft.com/mapData/metadata/{udid}?api-version=1.0"
      }
      ```

## <a name="create-an-iot-hub"></a>Créer un hub IoT

IoT Hub permet une communication bidirectionnelle sécurisée et fiable entre une application IoT et les appareils qu’elle gère. Pour ce tutoriel, vous souhaitez obtenir des informations à partir de l’appareil embarqué dans le véhicule afin de déterminer la localisation de la voiture de location. Dans cette section, vous allez créer un hub IoT dans le groupe de ressources *ContosoRental*. Ce hub est chargé de la publication des événements de télémétrie de l’appareil.

> [!NOTE]
> La capacité à publier des événements de télémétrie de l’appareil sur Event Grid est actuellement en préversion. Cette fonctionnalité est disponible dans toutes les régions, à l’exception des suivantes : USA Est, USA Ouest, Europe Ouest, Azure Government, Azure China 21Vianet et Azure Allemagne.

Pour créer un hub IoT dans le groupe de ressources *ContosoRental*, effectuez les étapes décrites dans [Créer un hub IoT](https://docs.microsoft.com/azure/iot-hub/quickstart-send-telemetry-dotnet#create-an-iot-hub).

## <a name="register-a-device-in-your-iot-hub"></a>Inscrire votre appareil dans votre hub IoT

Les appareils ne peuvent pas se connecter à IoT Hub, sauf s’ils sont inscrits dans le registre des identités IoT Hub. Ici, vous allez créer un appareil portant le nom *InVehicleDevice*. Pour créer et inscrire l’appareil dans votre hub IoT, suivez les étapes décrites dans [Inscrire un nouvel appareil dans le hub IoT](https://docs.microsoft.com/azure/iot-hub/iot-hub-create-through-portal#register-a-new-device-in-the-iot-hub). N’oubliez pas de copier la chaîne de connexion principale de votre appareil. Vous en aurez besoin ultérieurement.

## <a name="create-a-function-and-add-an-event-grid-subscription"></a>Créer une fonction et ajouter un abonnement Event Grid

Azure Functions est un service de calcul serverless qui vous permet d’exécuter de petites portions de code (« fonctions »), sans qu’il soit nécessaire de provisionner ou de gérer explicitement l’infrastructure de calcul. Pour en savoir plus, consultez [Azure Functions](https://docs.microsoft.com/azure/azure-functions/functions-overview).

Une fonction est déclenchée par un événement spécifique. Ici, vous allez créer une fonction déclenchée par un déclencheur Event Grid. Créez la relation entre le déclencheur et la fonction en créant un abonnement à un événement pour les événements de télémétrie de l’appareil IoT Hub. Lorsqu’un événement de télémétrie de l’appareil se produit, votre fonction est appelée comme point de terminaison et reçoit les données pertinentes pour l’appareil que vous avez précédemment inscrit dans IoT Hub.

Voici le [code de script C# que votre fonction contiendra](https://github.com/Azure-Samples/iothub-to-azure-maps-geofencing/blob/master/src/Azure%20Function/run.csx).

À présent, configurez votre fonction Azure.

1. Dans le tableau de bord du portail Azure, sélectionnez **Créer une ressource**. Saisissez **Function App** dans la zone de texte de recherche. Sélectionnez **Application de fonction** > **Créer**.

1. Dans la page de création **Function App**, nommez votre application de fonction. Sous **Groupe de ressources**, sélectionnez **ContosoRental** dans la liste déroulante. Sélectionnez **.NET Core** comme **pile d’exécution**. En bas de la page, sélectionnez **Suivant  : Hébergement >** .

    :::image type="content" source="./media/tutorial-iot-hub-maps/rental-app.png" alt-text="Capture d’écran de la création d’une application de fonction.":::

1. Pour **Compte de stockage**, sélectionnez le compte de stockage que vous avez créé dans [Créer un compte de stockage Azure](#create-an-azure-storage-account). Sélectionnez **Revoir + créer**.

1. Passez en revue les détails de l’application de fonction, puis sélectionnez **Créer**.

1. Une fois l’application créée, vous allez y ajouter une fonction. Accédez à l’application de fonction. Sélectionnez le volet **Fonctions**. En haut de la page, sélectionnez **+ Ajouter**. Le panneau de modèle de fonction s’affiche. Faites défiler le panneau, puis sélectionnez **Déclencheur Azure Event Grid**.

     >[!IMPORTANT]
    > Les modèles **Azure Event Hub Trigger** (Déclencheur Azure Event Hub) et **Azure Event Grid Trigger** (Déclencheur Azure Event Grid) ont des noms similaires. Veillez à sélectionner le modèle **Déclencheur Azure Event Grid**.

    :::image type="content" source="./media/tutorial-iot-hub-maps/function-create.png" alt-text="Capture d’écran de création de fonction.":::

1. Donnez un nom à la fonction. Dans ce tutoriel, vous allez utiliser le nom *GetGeoFunction*, mais en général vous pouvez utiliser le nom de votre choix. Sélectionnez **Créer une fonction**.

1. Dans le menu de gauche, sélectionnez le volet **Code + test**. Copiez et collez le [Script C#](https://github.com/Azure-Samples/iothub-to-azure-maps-geofencing/blob/master/src/Azure%20Function/run.csx) dans la fenêtre de code.

     :::image type="content" source="./media/tutorial-iot-hub-maps/function-code.png" alt-text="Copie/Capture d’écran du code de collage dans la fenêtre de fonction.":::

1. Dans le code C#, remplacez les paramètres suivants :
    * Remplacez **SUBSCRIPTION_KEY** par la clé principale d’abonnement de votre compte Azure Maps.
    * Remplacez **UDID** par `udid` de la limite géographique que vous avez chargée dans [Charger une limite géographique](#upload-a-geofence).
    * La fonction `CreateBlobAsync` du script crée un objet blob par événement dans le compte de stockage de données. Remplacez les valeurs **ACCESS_KEY**, **ACCOUNT_NAME** et **STORAGE_CONTAINER_NAME** par la clé d’accès de votre compte de stockage, le nom du compte et le conteneur de stockage de données. Ces valeurs ont été générées lors de la création de votre compte de stockage dans la section [Créer un compte de stockage Azure](#create-an-azure-storage-account).

1. Dans le menu de gauche, sélectionnez le volet **Intégration**. Sélectionnez **Déclencheur Event Grid** dans le diagramme. Tapez un nom pour le déclencheur, *eventGridEvent*, puis sélectionnez **Créer un abonnement Event Grid**.

     :::image type="content" source="./media/tutorial-iot-hub-maps/function-integration.png" alt-text="Capture d’écran de l’ajout d’un abonnement aux événements.":::

1. Renseignez les détails de l’abonnement. Donnez un nom à l’abonnement à l’événement. Pour **Schéma d’événement**, sélectionnez **Schéma Event Grid**. Pour **Types de rubrique**, sélectionnez **Comptes Azure IoT Hub**. Pour **Groupe de ressources**, sélectionnez le groupe de ressources que vous avez créé au début de ce tutoriel. Pour **Ressource**, sélectionnez le hub IoT que vous avez créé dans « Créer un hub IoT Azure ». Pour **Filtrer sur les types d’événements**, sélectionnez **Télémétrie de l’appareil**.

   Une fois ces options choisies, le **Type de rubrique** devient **IoT Hub**. Pour **Nom de rubrique système**, vous pouvez utiliser le même nom que celui de votre ressource. Pour finir, dans la section **Détails du point de terminaison**, sélectionnez **Sélectionner un point de terminaison**. Acceptez tous les paramètres, puis sélectionnez **Confirmer la sélection**.

    :::image type="content" source="./media/tutorial-iot-hub-maps/function-create-event-subscription.png" alt-text="Capture d’écran de la création d’un abonnement aux événements.":::

1. Passez vos paramètres en revue. Assurez-vous que le point de terminaison spécifie la fonction que vous avez créée au début de cette section. Sélectionnez **Create** (Créer).

    :::image type="content" source="./media/tutorial-iot-hub-maps/function-create-event-subscription-confirm.png" alt-text="Capture d’écran de la confirmation de création d’un abonnement aux événements.":::

1. Vous êtes à présent de retour dans le panneau **Modifier le déclencheur**. Sélectionnez **Enregistrer**.

## <a name="filter-events-by-using-iot-hub-message-routing"></a>Filtrer les événements à l’aide du routage des messages IoT Hub

Lorsque vous ajoutez un abonnement Event Grid à la fonction Azure, un itinéraire de messagerie est créé automatiquement dans le hub IoT spécifié. Le routage des messages vous permet d’acheminer différents types de données vers différents points de terminaison. Vous pouvez par exemple router les messages de télémétrie de l’appareil, les événements du cycle de vie de l’appareil et les événements de changement du jumeau d’appareil. Pour plus d’informations, consultez [Routage des messages IoT Hub](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-messages-d2c).

:::image type="content" source="./media/tutorial-iot-hub-maps/hub-route.png" alt-text="Capture d’écran du routage des messages dans IoT Hub.":::

Dans l’exemple de scénario, vous ne voulez recevoir des messages que lorsque la voiture de location se déplace. Créez une requête de routage pour filtrer les événements dont la propriété `Engine` est égale à **« ON »** . Pour créer une requête de routage, sélectionnez la route **RouteToEventGrid** et remplacez la **requête de routage** par **« Engine=’ON’ »** . Ensuite, sélectionnez **Enregistrer**. Désormais, le hub IoT publie uniquement les données de télémétrie d’appareil où le moteur est en marche.

:::image type="content" source="./media/tutorial-iot-hub-maps/hub-filter.png" alt-text="Capture d’écran du filtrage de routage des messages.":::

>[!TIP]
>Il existe plusieurs façons d’interroger des messages appareil-à-cloud IoT. Pour en savoir plus sur la syntaxe de routage des messages, consultez [Routage des messages IoT Hub](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-routing-query-syntax).

## <a name="send-telemetry-data-to-iot-hub"></a>Envoyer des données de télémétrie à IoT Hub

Quand votre fonction Azure s’exécute, vous pouvez maintenant envoyer les données de télémétrie au hub IoT, qui les routera vers Event Grid. Utilisez une application C# pour simuler les données de localisation d’un appareil embarqué d’une voiture de location. Pour exécuter l’application, votre ordinateur de développement doit disposer du SDK .NET Core 2.1.0 ou version ultérieure. Effectuez ces étapes pour envoyer les données de télémétrie simulées au hub IoT :

1. Si ce n’est déjà fait, téléchargez le projet C# [rentalCarSimulation](https://github.com/Azure-Samples/iothub-to-azure-maps-geofencing/tree/master/src/rentalCarSimulation).

2. Ouvrez le fichier `simulatedCar.cs` dans l’éditeur de texte de votre choix, et remplacez la valeur de `connectionString` par celle que vous avez enregistrée lors de l’inscription de l’appareil. Enregistrez les modifications apportées au fichier.

3. Vérifiez que .NET Core est installé sur votre ordinateur. Dans la fenêtre de votre terminal local, accédez au dossier racine du projet C#, puis exécutez la commande suivante afin d’installer les packages nécessaires pour l’application d’appareil simulé :

    ```cmd/sh
    dotnet restore
    ```

4. Dans le même terminal, exécutez la commande suivante pour générer et exécuter l’application de simulation de voiture de location :

    ```cmd/sh
    dotnet run
    ```


  Votre terminal local doit ressembler à celui présenté ci-dessous.

:::image type="content" source="./media/tutorial-iot-hub-maps/terminal.png" alt-text="Capture d’écran de la sortie de terminal.":::

Si vous ouvrez le conteneur de stockage blob maintenant, vous pouvez voir quatre objets blob correspondant aux emplacements où le véhicule se trouvait hors de la limite géographique.

:::image type="content" source="./media/tutorial-iot-hub-maps/blob.png" alt-text="Capture d’écran de l’affichage des objets blob dans le conteneur.":::

La carte suivante montre quatre points de localisation du véhicule en dehors de la limite géographique. Chaque emplacement a été journalisé à intervalles réguliers.

:::image type="content" source="./media/tutorial-iot-hub-maps/violation-map.png" alt-text="Capture d’écran de la carte des violations.":::

## <a name="explore-azure-maps-and-iot"></a>Explorer Azure Maps et IoT

Pour explorer les API Azure Maps utilisées dans ce tutoriel, consultez :

* [Get Search Address Reverse](/rest/api/maps/search/getsearchaddressreverse)
* [Get Geofence](/rest/api/maps/spatial/getgeofence) (Obtenir la limite géographique)

Pour obtenir la liste complète des API REST Azure Maps, consultez :

* [API REST Azure Maps](/rest/api/maps/spatial/getgeofence)

* [IoT Plug and Play](../iot-pnp/index.yml)

Pour obtenir la liste des appareils qui sont certifiés Azure pour IoT, visitez :

* [Appareils certifiés Azure](https://catalog.azureiotsolutions.com/)

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la façon d’envoyer des données de télémétrie appareil-à-cloud, et vice versa, consultez :


> [!div class="nextstepaction"]
> [Envoyer des données de télémétrie depuis un appareil](../iot-hub/quickstart-send-telemetry-dotnet.md)
