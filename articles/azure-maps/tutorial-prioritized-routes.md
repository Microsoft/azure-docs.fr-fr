---
title: 'Tutoriel : Trouver plusieurs routes en fonction du mode de transport | Microsoft Azure Maps'
description: Didacticiel sur l’utilisation d’Azure Maps afin de rechercher des itinéraires pour des modes spécifiques de déplacement vers des points d'intérêt. Apprenez à afficher plusieurs itinéraires sur des cartes.
author: anastasia-ms
ms.author: v-stharr
ms.date: 09/10/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: mvc, devx-track-js
ms.openlocfilehash: 125ca501dbad74263f32632db44eebd097c3b0a1
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92896699"
---
# <a name="tutorial-find-and-display-routes-for-different-modes-of-travel-using-azure-maps"></a>Tutoriel : Rechercher et afficher des itinéraires pour différents modes de déplacement avec Azure Maps

Ce tutoriel montre comment utiliser le [service Route](/rest/api/maps/route) et le [contrôle de carte](./how-to-use-map-control.md) d’Azure Maps pour afficher des directions pour des véhicules privés et des véhicules commerciaux (camions) avec le type de chargement `USHazmatClass2`. Nous verrons également comment visualiser les données de trafic en temps réel sur une carte. Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer et afficher le contrôle de carte sur une page web.
> * Afficher des données de trafic en temps réel sur une carte.
> * Demander et afficher des itinéraires pour véhicules privés et commerciaux sur une carte.

## <a name="prerequisites"></a>Prérequis

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. [Créez un compte Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account).

3. [Obtenir une clé d’abonnement principale](quick-demo-map-app.md#get-the-primary-key-for-your-account), également appelée clé primaire ou clé d’abonnement. Pour plus d’informations sur l’authentification dans Azure Maps, voir [Gérer l’authentification dans Azure Maps](how-to-manage-authentication.md).

Vous pouvez obtenir le code source complet de l’exemple [ici](https://github.com/Azure-Samples/AzureMapsCodeSamples/blob/master/AzureMapsCodeSamples/Tutorials/truckRoute.html). Vous pouvez trouver un exemple en direct [ici](https://azuremapscodesamples.azurewebsites.net/?sample=Multiple%20routes%20by%20mode%20of%20travel).

## <a name="create-a-new-web-page-using-the-map-control-api"></a>Créer une page web à l’aide de l’API Map Control

Les étapes suivantes vous montrent comment créer et afficher le contrôle de carte dans une page web.

1. Sur votre ordinateur local, créez un fichier et nommez-le **MapTruckRoute.html** .
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

    :::image type="content" source="./media/tutorial-prioritized-routes/basic-map.png" alt-text="Rendu de la carte de base du contrôle de carte":::

## <a name="render-real-time-traffic-data-on-a-map"></a>Afficher des données de trafic en temps réel sur une carte.

1. Ajoutez le code JavaScript suivant à la fonction `GetMap`. Ce code implémente le gestionnaire d’événements du contrôle de carte `ready`. Le reste du code de ce tutoriel sera placé dans le gestionnaire d’événements `ready`.

    ```javascript
    map.events.add("ready", function() {
        // Add Traffic Flow to the Map
        map.setTraffic({
            flow: "relative"
        });
    });
    ```

    Dans le gestionnaire d’événements `ready` de la carte, le paramètre de circulation sur la carte est défini sur `relative`, c’est-à-dire la vitesse de la route correspondant à un trafic fluide. Pour plus d’options de trafic, consultez [TrafficOptions, interface](/javascript/api/azure-maps-control/atlas.trafficoptions?preserve-view=false&view=azure-maps-typescript-latest).

2. Enregistrez le fichier **MapTruckRoute.html** et actualisez la page dans votre navigateur. Si vous effectuez un zoom sur une ville, par exemple Los Angeles, vous verrez que les rues sont affichées avec les données de circulation actuelles.

    :::image type="content" source="./media/tutorial-prioritized-routes/traffic-map.png" alt-text="Rendu de la carte de base du contrôle de carte":::

<a id="queryroutes"></a>

## <a name="define-route-display-rendering"></a>Définir le rendu de l’affichage de l’itinéraire

Dans ce didacticiel, deux itinéraires sont calculés et affichés sur la carte. Le premier itinéraire est calculé pour un véhicule privé (voiture). Le deuxième itinéraire est calculé pour un véhicule commercial (camion) afin de montrer la différence entre les résultats. Lorsque la carte est rendue, elle affiche une icône de symbole pour les points de départ et de fin de l’itinéraire, et des formes géométriques de ligne d’itinéraire avec différentes couleurs pour chaque tracé d’itinéraire. Pour plus d’informations sur l’ajout de couches de lignes, consultez [Ajouter une couche de lignes à une carte](map-add-line-layer.md). Pour en savoir plus sur les couches de symboles, consultez [Ajouter une couche de symboles à une carte](map-add-pin.md).

1. Dans le gestionnaire d’événements du contrôle de carte `ready`, ajoutez le code suivant.

    ```JavaScript

    //Create a data source and add it to the map.
    datasource = new atlas.source.DataSource();
    map.sources.add(datasource);

    //Add a layer for rendering the route lines and have it render under the map labels.
    map.layers.add(new atlas.layer.LineLayer(datasource, null, {
        strokeColor: ['get', 'strokeColor'],
        strokeWidth: ['get', 'strokeWidth'],
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

    ```


    Dans le gestionnaire d’événements `ready` du contrôle de carte, une source de données est créée pour stocker l’itinéraire du départ jusqu’à l’arrivée. Des [expressions](data-driven-style-expressions-web-sdk.md) sont utilisées pour récupérer la largeur et la couleur de ligne à partir des propriétés de la caractéristique de ligne d’itinéraire. Pour vous assurer que la ligne d’itinéraire ne couvre pas les étiquettes de route, nous avons transmis un deuxième paramètre avec la valeur de `'labels'`.

    Ensuite, une couche de symbole est créée et jointe à la source de données. Ce calque spécifie comment les points de départ et d’arrivée sont affichés. Des expressions ont été ajoutées pour récupérer les informations d’image d’icône et d’étiquette de texte des propriétés à partir de chaque objet de point. Pour en savoir plus sur les expressions, consultez [Expressions de style basé sur les données](data-driven-style-expressions-web-sdk.md).

2. Définissez le point de départ sur une entreprise fictive de Seattle appelée Fabrikam, et le point d’arrivée sur un bâtiment Microsoft.  Dans le gestionnaire d’événements du contrôle de carte `ready`, ajoutez le code suivant.


    ```JavaScript
    //Create the GeoJSON objects which represent the start and end point of the route.
    var startPoint = new atlas.data.Feature(new atlas.data.Point([-122.356099, 47.580045]), {
        title: 'Fabrikam, Inc.',
        icon: 'pin-blue'
    });

    var endPoint = new atlas.data.Feature(new atlas.data.Point([-122.201164, 47.616940]), {
        title: 'Microsoft - Lincoln Square',
        icon: 'pin-round-blue'
    });

    //Add the data to the data source.
    datasource.add([startPoint, endPoint]);

    //Fit the map window to the bounding box defined by the start and end positions.
    map.setCamera({
        bounds: atlas.data.BoundingBox.fromData([startPoint, endPoint]),
        padding: 100
    });

    ```

    Ce code crée deux [objets point GeoJSON](https://en.wikipedia.org/wiki/GeoJSON) pour représenter des points de départ et d’arrivée qui sont ensuite ajoutés à la source de données.

    Le dernier bloc de code définit la vue de l’appareil photo sur la base de la latitude et de la longitude des points de départ et d’arrivée. Les points de départ et d’arrivée sont ajoutés à la source de données. Le rectangle englobant des points de départ et d’arrivée est calculé à l’aide de la fonction `atlas.data.BoundingBox.fromData`. Ce rectangle englobant est utilisé pour définir la vue de caméra de la carte sur l’itinéraire entier à l’aide de la fonction `map.setCamera`. Une marge intérieure est ajoutée pour compenser les dimensions en pixels des icônes de symbole. Pour plus d’informations sur la propriété setCamera du contrôle de carte setCamera, consultez la propriété [setCamera(CameraOptions | CameraBoundsOptions & AnimationOptions)](/javascript/api/azure-maps-control/atlas.map?view=azure-maps-typescript-latest#setcamera-cameraoptions---cameraboundsoptions---animationoptions-&preserve-view=false).

3. Enregistrez **TruckRoute.html** et actualisez votre navigateur. La carte est maintenant centrée sur Seattle. La broche bleue en forme de larme marque le point de départ. La broche bleue ronde marque le point d’arrivée.

   :::image type="content" source="./media/tutorial-prioritized-routes/pins-map.png" alt-text="Rendu de la carte de base du contrôle de carte":::

<a id="multipleroutes"></a>

## <a name="request-and-display-private-and-commercial-vehicle-routes-on-a-map"></a>Demander et afficher des itinéraires pour véhicules privés et commerciaux sur une carte.

Cette section montre comment utiliser le service Route Azure Maps pour trouver des directions d’un point à un autre, en fonction du mode de transport. Nous allons utiliser deux modes de transport : camion et voiture.

>[!TIP]
>Le service Route fournit des API afin de planifier l’itinéraire le plus *rapide* , le plus *court* , le plus *économique* ou le plus *intéressant* en fonction de la distance, des conditions de circulation et du mode de transport utilisé. Il permet également aux utilisateurs de planifier de futurs itinéraires en fonction de l’historique des conditions de circulation. Les utilisateurs peuvent voir la prédiction des durées d’itinéraire pour un moment donné. Pour plus d’informations, consultez l’[API Obtenir les itinéraires](/rest/api/maps/route/getroutedirections).

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

2. Après avoir configuré les informations d’identification et l’URL, ajoutez le code JavaScript suivant pour construire un itinéraire pour camion du point de départ au point d’arrivée. Cet itinéraire est créé et affiché pour un camion transportant un chargement classé `USHazmatClass2`.

    ```JavaScript
    //Start and end point input to the routeURL
    var coordinates= [[startPoint.geometry.coordinates[0], startPoint.geometry.coordinates[1]], [endPoint.geometry.coordinates[0], endPoint.geometry.coordinates[1]]];

    //Make a search route request for a truck vehicle type
    routeURL.calculateRouteDirections(atlas.service.Aborter.timeout(10000), coordinates,{
        travelMode: 'truck',
        vehicleWidth: 2,
        vehicleHeight: 2,
        vehicleLength: 5,
        vehicleLoadType: 'USHazmatClass2'
    }).then((directions) => {
        //Get data features from response
        var data = directions.geojson.getFeatures();

        //Get the route line and add some style properties to it.  
        var routeLine = data.features[0];
        routeLine.properties.strokeColor = '#2272B9';
        routeLine.properties.strokeWidth = 9;

        //Add the route line to the data source. We want this to render below the car route which will likely be added to the data source faster, so insert it at index 0.
        datasource.add(routeLine, 0);
    });
    ```

    Le code ci-dessus interroge le service Route Azure Maps par le biais de l’[API Obtenir des directions Azure Maps](/javascript/api/azure-maps-rest/atlas.service.routeurl#calculateroutedirections-aborter--geojson-position----calculateroutedirectionsoptions-). La ligne d’itinéraire est ensuite extraite de la collection de fonctionnalités GeoJSON à partir de la réponse extraite à l’aide de la méthode `geojson.getFeatures()`. Pour finir, la ligne d’itinéraire est ajoutée à la source de données. Nous l’ajoutons à l’index 0 afin de nous assurer que l’itinéraire pour camion est affiché avant toute autre ligne dans la source de données, car le calcul de l’itinéraire pour camion sera souvent plus lent que le calcul d’un itinéraire pour voiture. Si la ligne d’itinéraire de camion est ajoutée à la source de données après l’itinéraire de voiture, elle s’affiche au-dessus d’elle. Deux propriétés sont ajoutées à la ligne d’itinéraire pour camion : un trait de couleur bleue et une épaisseur de trait de neuf pixels.

    >[!TIP]
    > Pour afficher toutes les options et valeurs possibles pour l’API Obtenir des directions Azure Maps, consultez les [paramètres d’URI pour la publication de directions](/rest/api/maps/route/postroutedirections#uri-parameters).

3. Ajoutez maintenant le code JavaScript suivant afin de construire un itinéraire pour une voiture.

    ```JavaScript
    routeURL.calculateRouteDirections(atlas.service.Aborter.timeout(10000), coordinates).then((directions) => {

        //Get data features from response
        var data = directions.geojson.getFeatures();

        //Get the route line and add some style properties to it.  
        var routeLine = data.features[0];
        routeLine.properties.strokeColor = '#B76DAB';
        routeLine.properties.strokeWidth = 5;

        //Add the route line to the data source. This will add the car route after the truck route.  
        datasource.add(routeLine);
    });
    ```

    Le code ci-dessus interroge le service d’itinéraire Azure Maps par le biais de la méthode d’[API Obtenir des directions Azure Maps](/javascript/api/azure-maps-rest/atlas.service.routeurl#calculateroutedirections-aborter--geojson-position----calculateroutedirectionsoptions-). La ligne d’itinéraire est ensuite extraite de la collection de fonctionnalités GeoJSON à partir de la réponse extraite à l’aide de la méthode `geojson.getFeatures()`. Pour finir, la ligne d’itinéraire est ajoutée à la source de données. Deux propriétés sont ajoutées à la ligne d’itinéraire pour camion : un trait de couleur violette et une épaisseur de trait de cinq pixels.

4. Enregistrez le fichier **TruckRoute.html** et actualisez votre navigateur web. La carte doit maintenant afficher les itinéraires pour camion et pour voiture.

    :::image type="content" source="./media/tutorial-prioritized-routes/prioritized-routes.png" alt-text="Rendu de la carte de base du contrôle de carte":::

    L’itinéraire pour camion est affiché à l’aide d’une ligne bleue épaisse. L’itinéraire pour voiture est affichée à l’aide d’une fine ligne violette. L’itinéraire pour voiture passe au-dessus du Lac Washington via l’I-90, qui traverse des tunnels installés sous des zones résidentielles. Les tunnels étant proches de zones résidentielles, tout transport de déchets dangereux est interdit. L’itinéraire pour camion, pour lequel est défini le type de chargement `USHazmatClass2`, utilise une voie de circulation différente.

Vous pouvez obtenir le code source complet de l’exemple [ici](https://github.com/Azure-Samples/AzureMapsCodeSamples/blob/master/AzureMapsCodeSamples/Tutorials/truckRoute.html). Vous pouvez trouver un exemple en direct [ici](https://azuremapscodesamples.azurewebsites.net/?sample=Multiple%20routes%20by%20mode%20of%20travel).

Vous pouvez également [Utiliser des expressions de style basées sur les données](data-driven-style-expressions-web-sdk.md)



## <a name="next-steps"></a>Étapes suivantes

Le tutoriel suivant montre comment créer un localisateur de magasin simple à l’aide d’Azure Maps.

> [!div class="nextstepaction"]
> [Créer un localisateur de magasin à l’aide d’Azure Maps](./tutorial-create-store-locator.md)