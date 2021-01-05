---
title: 'Tutoriel : Trouver une route vers une position | Microsoft Azure Maps'
description: Tutoriel expliquant comment trouver une route vers un point d’intérêt. Découvrez comment définir des coordonnées d’adresse et interroger le service Route d’Azure Maps pour obtenir des itinéraires vers ce point.
author: anastasia-ms
ms.author: v-stharr
ms.date: 09/01/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: mvc, devx-track-js
ms.openlocfilehash: 0004a250173ce6707462b852016d205782479717
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92896678"
---
# <a name="tutorial-how-to-display-route-directions-using-azure-maps-route-service-and-map-control"></a>Tutoriel : Comment afficher les directions de route à l’aide du service Route Azure Maps et du contrôle de carte

Ce tutoriel vous montre comment utiliser [l’API de service Route](/rest/api/maps/route) Azure Maps et [contrôle de route](./how-to-use-map-control.md) pour afficher les directions de route du point de départ au point d’arrivée. Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer et afficher le contrôle de carte sur une page web. 
> * Définir le rendu d’affichage de l’itinéraire en définissant des [c ouches de symboles](map-add-pin.md) et des [couches de lignes](map-add-line-layer.md).
> * Créer des objets GeoJSON et les ajouter à la carte pour représenter les points de départ et d’arrivée.
> * Obtenir des directions de route à partir de points de départ et d’arrivée à l’aide de l’[API Obtenir un itinéraire](/rest/api/maps/route/getroutedirections).

Vous pouvez obtenir le code source complet de l’exemple [ici](https://github.com/Azure-Samples/AzureMapsCodeSamples/blob/master/AzureMapsCodeSamples/Tutorials/route.html). Vous pouvez trouver un exemple en direct [ici](https://azuremapscodesamples.azurewebsites.net/?sample=Route%20to%20a%20destination).

## <a name="prerequisites"></a>Prérequis

1. [Créer un compte Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account)
2. [Obtenir une clé d’abonnement principale](quick-demo-map-app.md#get-the-primary-key-for-your-account), également appelée clé primaire ou clé d’abonnement.

<a id="getcoordinates"></a>

## <a name="create-and-display-the-map-control"></a>Créer et afficher le contrôle de carte

Les étapes suivantes vous montrent comment créer et afficher le contrôle de carte dans une page web.

1. Sur votre ordinateur local, créez un fichier et nommez-le **MapRoute.html** .
2. Copiez/collez le balisage HTML suivant dans le fichier.

    ```HTML
    <!DOCTYPE html>
    <html>
    <head>
        <title>Map Route</title>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

        <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
        <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css">
        <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>

        <!-- Add a reference to the Azure Maps Services Module JavaScript file. -->
        <script src="https://atlas.microsoft.com/sdk/javascript/service/2/atlas-service.min.js"></script>

        <script>
            var map, datasource, client;

            function GetMap() {
                //Add Map Control JavaScript code here.
            }
        </script>
        <style>
            html,
            body {
                width: 100%;
                height: 100%;
                padding: 0;
                margin: 0;
            }

            #myMap {
                width: 100%;
                height: 100%;
            }
        </style>
    </head>
    <body onload="GetMap()">
        <div id="myMap"></div>
    </body>
    </html>
    ```

    L’en-tête HTML inclut les fichiers de ressources CSS et JavaScript hébergés par la bibliothèque Azure Map Control. L’événement `onload` du corps appelle la fonction `GetMap`. À l’étape suivante, nous allons ajouter le code d’initialisation du contrôle de carte.

3. Ajoutez le code JavaScript suivant à la fonction `GetMap`. Remplacez la chaîne `<Your Azure Maps Key>` par la clé primaire copiée à partir de votre compte Maps.

    ```JavaScript
   //Instantiate a map object
   var map = new atlas.Map("myMap", {
        //Add your Azure Maps subscription key to the map SDK. Get an Azure Maps key at https://azure.com/maps
        authOptions: {
           authType: 'subscriptionKey',
           subscriptionKey: '<Your Azure Maps Key>'
        }
   });
   ```

4. Enregistrez le fichier et ouvrez-le dans votre navigateur. Un modèle s’affiche.

     :::image type="content" source="./media/tutorial-route-location/basic-map.png" alt-text="Rendu de la carte de base du contrôle de carte":::

## <a name="define-route-display-rendering"></a>Définir le rendu de l’affichage de l’itinéraire

Dans ce tutoriel, nous allons afficher l’itinéraire à l’aide d’une couche de lignes. Les points de départ et d’arrivée seront affichés à l’aide d’un calque de symboles. Pour plus d’informations sur l’ajout de couches de lignes, consultez [Ajouter une couche de lignes à une carte](map-add-line-layer.md). Pour en savoir plus sur les couches de symboles, consultez [Ajouter une couche de symboles à une carte](map-add-pin.md).

1. Ajoutez le code JavaScript suivant à la fonction `GetMap`. Ce code implémente le gestionnaire d’événements du contrôle de carte `ready`. Le reste du code de ce tutoriel sera placé dans le gestionnaire d’événements `ready`.

    ```JavaScript
    //Wait until the map resources are ready.
    map.events.add('ready', function() {

        //Create a data source and add it to the map.
        datasource = new atlas.source.DataSource();
        map.sources.add(datasource);

        //Add a layer for rendering the route lines and have it render under the map labels.
        map.layers.add(new atlas.layer.LineLayer(datasource, null, {
            strokeColor: '#2272B9',
            strokeWidth: 5,
            lineJoin: 'round',
            lineCap: 'round'
        }), 'labels');

        //Add a layer for rendering point data.
        map.layers.add(new atlas.layer.SymbolLayer(datasource, null, {
            iconOptions: {
                image: ['get', 'icon'],
                allowOverlap: true
           },
            textOptions: {
                textField: ['get', 'title'],
                offset: [0, 1.2]
            },
            filter: ['any', ['==', ['geometry-type'], 'Point'], ['==', ['geometry-type'], 'MultiPoint']] //Only render Point or MultiPoints in this layer.
        }));
    });
    ```

    Dans le gestionnaire d’événements `ready` du contrôle de carte, une source de données est créée pour stocker l’itinéraire du point de départ au point d’arrivée. Pour définir le rendu de la ligne d’itinéraire, une couche de lignes est créée et jointe à la source de données.  Pour vous assurer que la ligne d’itinéraire ne couvre pas les étiquettes de route, nous avons transmis un deuxième paramètre avec la valeur de `'labels'`.

    Ensuite, une couche de symbole est créée et jointe à la source de données. Ce calque spécifie comment les points de départ et d’arrivée sont affichés. Des expressions ont été ajoutées pour récupérer les informations d’image d’icône et d’étiquette de texte des propriétés à partir de chaque objet de point. Pour en savoir plus sur les expressions, consultez [Expressions de style basé sur les données](data-driven-style-expressions-web-sdk.md).

2. Définissez Microsoft comme point de départ, et une station-service de Seattle comme point de destination.  Dans le gestionnaire d’événements du contrôle de carte `ready`, ajoutez le code suivant.

    ```JavaScript
    //Create the GeoJSON objects which represent the start and end points of the route.
    var startPoint = new atlas.data.Feature(new atlas.data.Point([-122.130137, 47.644702]), {
        title: "Redmond",
        icon: "pin-blue"
    });

    var endPoint = new atlas.data.Feature(new atlas.data.Point([-122.3352, 47.61397]), {
        title: "Seattle",
        icon: "pin-round-blue"
    });

    //Add the data to the data source.
    datasource.add([startPoint, endPoint]);

    map.setCamera({
        bounds: atlas.data.BoundingBox.fromData([startPoint, endPoint]),
        padding: 80
    });
    ```

    Ce code crée deux [objets point GeoJSON](https://en.wikipedia.org/wiki/GeoJSON) pour représenter des points de départ et d’arrivée qui sont ensuite ajoutés à la source de données. 

    Le dernier bloc de code définit la vue de l’appareil photo sur la base de la latitude et de la longitude des points de départ et d’arrivée. Les points de départ et d’arrivée sont ajoutés à la source de données. Le rectangle englobant des points de départ et d’arrivée est calculé à l’aide de la fonction `atlas.data.BoundingBox.fromData`. Ce rectangle englobant est utilisé pour définir la vue de caméra de la carte sur l’itinéraire entier à l’aide de la fonction `map.setCamera`. Une marge intérieure est ajoutée pour compenser les dimensions en pixels des icônes de symbole. Pour plus d’informations sur la propriété setCamera du contrôle de carte setCamera, consultez la propriété [setCamera(CameraOptions | CameraBoundsOptions & AnimationOptions)](/javascript/api/azure-maps-control/atlas.map?view=azure-maps-typescript-latest#setcamera-cameraoptions---cameraboundsoptions---animationoptions-&preserve-view=false).

3. Enregistrez **MapRoute.html** et actualisez votre navigateur. La carte est maintenant centrée sur Seattle. La broche bleue en forme de larme marque le point de départ. La broche bleue ronde marque le point d’arrivée.

    :::image type="content" source="./media/tutorial-route-location/map-pins.png" alt-text="Rendu de la carte de base du contrôle de carte":::

<a id="getroute"></a>

## <a name="get-route-directions"></a>Get Route Directions (Obtenir des directions)

Cette section montre comment utiliser l’API Obtenir des directions Azure Maps pour obtenir des directions d’un point à un autre et l’heure d’arrivée estimée.

>[!TIP]
>Le service Route Azure Maps fournit des API afin de planifier des itinéraires basés sur différents types d’itinéraires, par exemple *le plus rapide* , *le plus court* , le plus *économique* ou le plus *intéressant* en fonction de la distance, des conditions de circulation et du mode de transport utilisé. Il permet également aux utilisateurs de planifier des itinéraires futurs en fonction de l’historique des conditions de circulation. Les utilisateurs peuvent voir la prédiction des durées d’itinéraire pour un moment donné. Pour plus d’informations, consultez l’[API Obtenir les itinéraires](/rest/api/maps/route/getroutedirections).

1. Dans la fonction `GetMap`, à l’intérieur du gestionnaire d’événements `ready` du contrôle, ajoutez le code suivant au code JavaScript.

    ```JavaScript
    // Use SubscriptionKeyCredential with a subscription key
    var subscriptionKeyCredential = new atlas.service.SubscriptionKeyCredential(atlas.getSubscriptionKey());

    // Use subscriptionKeyCredential to create a pipeline
    var pipeline = atlas.service.MapsURL.newPipeline(subscriptionKeyCredential);

    // Construct the RouteURL object
    var routeURL = new atlas.service.RouteURL(pipeline);
    ```

   `SubscriptionKeyCredential` crée un `SubscriptionKeyCredentialPolicy` pour authentifier les requêtes HTTP auprès d’Azure Maps avec la clé d’abonnement. `atlas.service.MapsURL.newPipeline()` utilise la stratégie `SubscriptionKeyCredential` et crée une instance de [pipeline](/javascript/api/azure-maps-rest/atlas.service.pipeline). `routeURL` représente une URL des opérations sur l’[itinéraire](/rest/api/maps/route) d’Azure Maps.

2. Après avoir configuré les informations d’identification et l’URL, ajoutez le code suivant dans le gestionnaire d’événements `ready` du contrôle. Ce code construit l’itinéraire du point de départ au point d’arrivée. L’URL `routeURL` demande à l’API de service Route Azure Maps de calculer des itinéraires. Une collection de fonctionnalités GeoJSON de la réponse est alors extraite à l’aide de la méthode `geojson.getFeatures()` et ajoutée à la source de données.

    ```JavaScript
    //Start and end point input to the routeURL
    var coordinates= [[startPoint.geometry.coordinates[0], startPoint.geometry.coordinates[1]], [endPoint.geometry.coordinates[0], endPoint.geometry.coordinates[1]]];

    //Make a search route request
    routeURL.calculateRouteDirections(atlas.service.Aborter.timeout(10000), coordinates).then((directions) => {
        //Get data features from response
        var data = directions.geojson.getFeatures();
        datasource.add(data);
    });
    ```

3. Enregistrez le fichier **MapRoute.html** et actualisez votre navigateur web. La carte doit maintenant afficher l’itinéraire du point de départ au point d’arrivée.

     :::image type="content" source="./media/tutorial-route-location/map-route.png" alt-text="Rendu de la carte de base du contrôle de carte":::

Vous pouvez obtenir le code source complet de l’exemple [ici](https://github.com/Azure-Samples/AzureMapsCodeSamples/blob/master/AzureMapsCodeSamples/Tutorials/route.html). Vous pouvez trouver un exemple en direct [ici](https://azuremapscodesamples.azurewebsites.net/?sample=Route%20to%20a%20destination).

## <a name="next-steps"></a>Étapes suivantes

Le didacticiel suivant vous montre comment créer une requête d’itinéraire avec des restrictions comme le mode de déplacement ou le type de chargement. Vous pouvez ensuite afficher plusieurs itinéraires sur la même carte.

> [!div class="nextstepaction"]
> [Rechercher des itinéraires pour différents modes de déplacements](./tutorial-prioritized-routes.md)