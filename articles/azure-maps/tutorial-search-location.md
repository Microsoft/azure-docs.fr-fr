---
title: 'Tutoriel : Rechercher des lieux à proximité sur une carte | Microsoft Azure Maps'
description: Tutoriel sur la recherche de points d’intérêt sur une carte. Découvrez comment utiliser le Kit de développement logiciel (SDK) web Azure Maps pour ajouter des fonctionnalités de recherche et des zones contextuelles interactives à une carte.
author: anastasia-ms
ms.author: v-stharr
ms.date: 1/15/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: mvc, devx-track-js
ms.openlocfilehash: 8cc7c1e0b776574ec7908557108e2cda49bb2a11
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92896658"
---
# <a name="tutorial-search-nearby-points-of-interest-using-azure-maps"></a>Tutoriel : Rechercher des points d’intérêt de proximité à l’aide d’Azure Maps

Ce didacticiel montre comment configurer un compte avec Azure Maps, puis utiliser les API Maps pour rechercher un point d’intérêt. Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer un compte Azure Maps
> * Récupérer la clé primaire de votre compte Maps
> * Créer une page web à l’aide de l’API Map Control
> * Utiliser le service Azure Maps Search pour rechercher un point d’intérêt de proximité

## <a name="prerequisites"></a>Prérequis

<a id="createaccount"></a>
<a id="getkey"></a>

1. Connectez-vous au [portail Azure](https://portal.azure.com). Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
2. [Créer un compte Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account)
3. [Obtenir une clé d’abonnement principale](quick-demo-map-app.md#get-the-primary-key-for-your-account), également appelée clé primaire ou clé d’abonnement. Pour plus d’informations sur l’authentification dans Azure Maps, voir [Gérer l’authentification dans Azure Maps](how-to-manage-authentication.md).

<a id="createmap"></a>

## <a name="create-a-new-map"></a>Créer une carte

L’API Map Control est une bibliothèque de client pratique. Elle vous permet d’intégrer facilement des cartes dans votre application web. Elle masque la complexité des appels de service REST bruts et améliore votre productivité grâce à des composants personnalisables. Les étapes suivantes vous indiquent comment créer une page HTML statique intégrée avec l’API Map Control.

1. Sur votre ordinateur local, créez un fichier et nommez-le **MapSearch.html** .
2. Ajoutez les composants HTML suivants au fichier :

   ```HTML
    <!DOCTYPE html>
    <html>
    <head>
        <title>Map Search</title>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

        <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
        <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css">
        <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>

        <!-- Add a reference to the Azure Maps Services Module JavaScript file. -->
        <script src="https://atlas.microsoft.com/sdk/javascript/service/2/atlas-service.min.js"></script>

        <script>
        function GetMap(){
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

   Notez que l’en-tête HTML inclut les fichiers de ressources CSS et JavaScript hébergés par la bibliothèque Azure Map Control. Notez l’événement `onload` sur le corps de la page, qui appelle la fonction `GetMap` lorsque le corps de la page est chargé. La fonction `GetMap` est destinée à contenir le code JavaScript inline pour accéder aux API Azure Maps.

3. Ajoutez le code JavaScript suivant à la fonction `GetMap` du fichier HTML. Remplacez la chaîne `<Your Azure Maps Key>` par la clé primaire copiée à partir de votre compte Maps.

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

   Ce segment lance l’API Map Control pour votre clé de compte Azure Maps. `atlas` est l’espace de noms qui contient l’API et les composants visuels associés. `atlas.Map` fournit le contrôle d’une carte web visuelle et interactive.

4. Enregistrez vos modifications du fichier, puis ouvrez la page HTML dans un navigateur. La carte illustrée est la plus basique que vous pouvez créer en appelant `atlas.Map` à l’aide de votre clé de compte.

   ![Afficher la carte](./media/tutorial-search-location/basic-map.png)

5. Dans la fonction `GetMap`, après l’initialisation de la carte, ajoutez le code JavaScript suivant.

    ```JavaScript
    //Wait until the map resources are ready.
    map.events.add('ready', function() {

        //Create a data source and add it to the map.
        datasource = new atlas.source.DataSource();
        map.sources.add(datasource);

        //Add a layer for rendering point data.
        var resultLayer = new atlas.layer.SymbolLayer(datasource, null, {
            iconOptions: {
                image: 'pin-round-darkblue',
                anchor: 'center',
                allowOverlap: true
            },
            textOptions: {
                anchor: "top"
            }
        });

        map.layers.add(resultLayer);
    });
    ```

   Dans ce segment de code, un événement `ready` est ajouté à la carte. Il est déclenché une fois que les ressources de la carte ont été chargées et que la carte est accessible. Dans le gestionnaire d’événements `ready` de la carte, une source de données est créée pour stocker des données de résultat. Une couche de symbole est créée et jointe à la source de données. Cette couche spécifie comment les données de résultat dans la source de données doivent être restituées. En l’occurrence, le résultat est restitué avec une icône d’épingle bleu foncé, centrée sur la coordonnée des résultats, et le chevauchement d’autres icônes est autorisé. La couche des résultats est ajoutée aux couches de la carte.

<a id="usesearch"></a>

## <a name="add-search-capabilities"></a>Ajouter les fonctionnalités de recherche

Cette section montre comment utiliser l’[API de recherche](/rest/api/maps/search) de Maps pour rechercher un point d’intérêt sur votre carte. Il s’agit d’une API RESTful destinée aux développeurs souhaitant mettre en place des fonctionnalités de recherche d’adresses, de points d’intérêt et d’autres informations d’ordre géographique. Search Service affecte une latitude et une longitude à une adresse spécifiée. Le **module de service** expliqué ci-dessous peut être utilisé pour rechercher un emplacement à l’aide de l’API Maps Search.

### <a name="service-module"></a>Module de service

1. Dans le gestionnaire d’événements `ready` de carte, construisez l’URL du service de recherche en ajoutant le code JavaScript suivant.

    ```JavaScript
   // Use SubscriptionKeyCredential with a subscription key
   var subscriptionKeyCredential = new atlas.service.SubscriptionKeyCredential(atlas.getSubscriptionKey());

   // Use subscriptionKeyCredential to create a pipeline
   var pipeline = atlas.service.MapsURL.newPipeline(subscriptionKeyCredential);

   // Construct the SearchURL object
   var searchURL = new atlas.service.SearchURL(pipeline); 
   ```

   `SubscriptionKeyCredential` crée un `SubscriptionKeyCredentialPolicy` pour authentifier les requêtes HTTP auprès d’Azure Maps avec la clé d’abonnement. `atlas.service.MapsURL.newPipeline()` utilise la stratégie `SubscriptionKeyCredential` et crée une instance de [pipeline](/javascript/api/azure-maps-rest/atlas.service.pipeline). `searchURL` représente une URL vers les opérations de [recherche](/rest/api/maps/search) d’Azure Maps.

2. Ajoutez ensuite le bloc de script suivant pour générer la requête de recherche. Il utilise le service Fuzzy Search, qui est l’API de recherche de base de Search Service. Le service Fuzzy Search gère la plupart des entrées partielles comme des adresses, des lieux et des points d’intérêt (POI). Ce code recherche les stations-service situées à proximité dans le rayon spécifié par la latitude et la longitude fournies. Une collection de fonctionnalités GeoJSON déterminée par la réponse est ensuite extraite à l’aide de la méthode `geojson.getFeatures()` et ajoutée à la source de données. Les données sont alors automatiquement restituées sur la carte par le biais de la couche de symboles. La dernière partie du script définit la vue de caméra de la carte en utilisant le rectangle englobant des résultats à l’aide de la propriété [setCamera](/javascript/api/azure-maps-control/atlas.map#setcamera-cameraoptions---cameraboundsoptions---animationoptions-) de la carte.

    ```JavaScript
    var query =  'gasoline-station';
    var radius = 9000;
    var lat = 47.64452336193245;
    var lon = -122.13687658309935;

    searchURL.searchPOI(atlas.service.Aborter.timeout(10000), query, {
        limit: 10,
        lat: lat,
        lon: lon,
        radius: radius
    }).then((results) => {

        // Extract GeoJSON feature collection from the response and add it to the datasource
        var data = results.geojson.getFeatures();
        datasource.add(data);

        // set camera to bounds to show the results
        map.setCamera({
            bounds: data.bbox,
            zoom: 10
        });
    });
    ```

3. Enregistrez le fichier **MapSearch.html** , puis actualisez votre navigateur. Vous devriez voir que la carte est centrée sur Seattle, avec des épingles de couleur bleu pour les stations-service dans la zone.

   ![Afficher la carte avec les résultats de la recherche](./media/tutorial-search-location/pins-map.png)

4. Vous pouvez voir les données brutes rendues par la carte en entrant la requête HTTP suivante dans votre navigateur. Remplacez \<Your Azure Maps Key\> par votre clé primaire.

   ```http
   https://atlas.microsoft.com/search/poi/json?api-version=1.0&query=gasoline%20station&subscription-key=<subscription-key>&lat=47.6292&lon=-122.2337&radius=100000
   ```

À ce stade, la page MapSearch peut afficher les emplacements des points d’intérêt renvoyés d’une requête de recherche approximative. Ajoutons quelques fonctionnalités interactives et des informations supplémentaires sur les emplacements.

## <a name="add-interactive-data"></a>Ajouter des données interactives

La carte développée jusqu’à présent produit des résultats de recherche comportant uniquement les données de longitude/latitude. Toutefois, le JSON brut retourné par le service de recherche Maps contient des informations supplémentaires sur chaque station-service, notamment le nom et l’adresse postale. Vous pouvez intégrer ces données dans la carte à l’aide de zones contextuelles interactives.

1. Ajoutez les lignes de code suivantes dans le gestionnaire d’événements `ready` de la carte après le code pour interroger le service de recherche approximative. Ce code crée une instance de fenêtre contextuelle et il ajoute un événement mouseover à la couche de symbole.

    ```JavaScript
   //Create a popup but leave it closed so we can update it and display it later.
    popup = new atlas.Popup();

    //Add a mouse over event to the result layer and display a popup when this event fires.
    map.events.add('mouseover', resultLayer, showPopup);
    ```

    L’API `*atlas.Popup` fournit une fenêtre d’informations ancrée à la position exigée sur la carte. 

2. Ajoutez le code suivant dans la fonction `GetMap` pour afficher les informations de résultat de l’événement mouseover dans la fenêtre contextuelle.

    ```JavaScript
    function showPopup(e) {
        //Get the properties and coordinates of the first shape that the event occurred on.

        var p = e.shapes[0].getProperties();
        var position = e.shapes[0].getCoordinates();

        //Create HTML from properties of the selected result.
        var html = `
          <div style="padding:5px">
            <div><b>${p.poi.name}</b></div>
            <div>${p.address.freeformAddress}</div>
            <div>${position[1]}, ${position[0]}</div>
          </div>`;

        //Update the content and position of the popup.
        popup.setPopupOptions({
            content: html,
            position: position
        });

        //Open the popup.
        popup.open(map);
    }
    ```

3. Enregistrez le fichier et actualisez votre navigateur. Désormais, la carte dans le navigateur affiche des fenêtres contextuelles de données lorsque vous survolez les repères de recherche.

    ![Azure Map Control et Search Service](./media/tutorial-search-location/popup-map.png)

Pour voir le code complet de ce tutoriel, cliquez [ici](https://github.com/Azure-Samples/AzureMapsCodeSamples/blob/master/AzureMapsCodeSamples/Tutorials/search.html). Pour voir l’exemple en direct, cliquez [ici](https://azuremapscodesamples.azurewebsites.net/?sample=Search%20for%20points%20of%20interest)

## <a name="next-steps"></a>Étapes suivantes

Le prochain didacticiel vous explique comment afficher un itinéraire entre deux emplacements.

> [!div class="nextstepaction"]
> [Itinéraire vers une destination](./tutorial-route-location.md)