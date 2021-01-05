---
title: Utiliser le module E/S spatiales Azure Maps | Microsoft Azure Maps
description: Découvrez comment utiliser le module E/S spatiales fourni par le kit de développement logiciel (SDK) web Azure Maps. Ce module propose de puissantes fonctionnalités pour permettre aux développeurs d’intégrer facilement les données spatiales au kit de développement logiciel (SDK) web Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 02/28/2020
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: devx-track-js
ms.openlocfilehash: cd64c80acceb1542c080fc45efbce59f287d448a
ms.sourcegitcommit: 66b0caafd915544f1c658c131eaf4695daba74c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97680694"
---
# <a name="how-to-use-the-azure-maps-spatial-io-module"></a>Utiliser le module E/S spatiales Azure Maps

Le kit de développement logiciel (SDK) web Azure Maps fournit le module **E/S spatiales**, qui intègre les données spatiales au kit de développement logiciel (SDK) web Azure Maps à l’aide de JavaScript ou TypeScript. Les puissantes fonctionnalités de ce module permettent aux développeurs d’effectuer les opérations suivantes :

- [Lire et écrire des fichiers de données spatiales courants](spatial-io-read-write-spatial-data.md). Les formats de fichiers pris en charge sont les suivants : Fichiers KML, KMZ, GPX, GeoRSS, GML, GeoJSON et CSV contenant des colonnes avec des informations spatiales. Prend également en charge le texte connu (WKT).
- [Connectez-vous aux services Open Geospatial Consortium (OGC) et intégrez-les au kit de développement logiciel (SDK) web Azure Maps. Superposez les services WMS (Web Map Services) et WMTS (Web Map Tile Services) en tant que couches sur la carte](spatial-io-add-ogc-map-layer.md).
- [Interrogez les données dans un service WFS (Web Feature Service)](spatial-io-connect-wfs-service.md).
- [Superposez les jeux de données complexes contenant des informations de style et affichez-les automatiquement avec un minimum de code](spatial-io-add-simple-data-layer.md).
- [Tirez parti des classes de lecture et d’écriture des fichiers XML et délimités](spatial-io-core-operations.md).

Dans ce guide, nous allons apprendre à intégrer et à utiliser le module E/S spatiales dans une application web.

Cette vidéo fournit une vue d’ensemble du module E/S spatiales dans le Kit de développement logiciel (SDK) web Azure Maps.

</br>

> [!VIDEO https://channel9.msdn.com/Shows/Internet-of-Things-Show/Easily-integrate-spatial-data-into-the-Azure-Maps/player?format=ny]

> [!WARNING]
> Utilisez uniquement des données et des services qui proviennent d’une source de confiance, en particulier si vous la référencez à partir d’un autre domaine. Le module d’E/S spatiales prend des mesures pour réduire les risques, mais l’approche la plus sûre n’autorise pas les données dangereuses dans votre application en premier lieu. 

## <a name="prerequisites"></a>Prérequis

Avant d'utiliser le module E/S spatiales, vous devez [créer un compte Azure Maps](./quick-demo-map-app.md#create-an-azure-maps-account) et [obtenir la clé primaire d’abonnement pour votre compte](./quick-demo-map-app.md#get-the-primary-key-for-your-account).

## <a name="installing-the-spatial-io-module"></a>Installation du module E/S spatiales

Vous pouvez charger le module E/S spatiales Azure Maps à l’aide de l'une des deux options suivantes :

* Azure CDN hébergé globalement pour le module E/S spatiales Azure Maps. Pour cette option, vous ajoutez une référence au code JavaScript dans l’élément `<head>` du fichier HTML.

    ```html
    <script src="https://atlas.microsoft.com/sdk/javascript/spatial/0/atlas-spatial.js"></script>
    ```

* Le code source pour [azure-maps-spatial-io](https://www.npmjs.com/package/azure-maps-spatial-io) peut être chargé localement, puis hébergé avec votre application. Ce package inclut aussi des définitions de TypeScript. Pour cette option, utilisez la commande suivante afin d'installer le package :

    ```sh
    npm install azure-maps-spatial-io
    ```

    Ajoutez ensuite une référence au code JavaScript dans l’élément `<head>` du document HTML :

    ```html
    <script src="node_modules/azure-maps-spatial-io/dist/atlas-spatial.min.js"></script>
    ```

## <a name="using-the-spatial-io-module"></a>Utilisation du module E/S spatiales

1. Créez un fichier HTML.

2. Chargez le kit de développement logiciel (SDK) web Azure Maps et initialisez le contrôle de carte. Pour plus d’informations, consultez le guide [Contrôle de carte Azure Maps](./how-to-use-map-control.md). Au terme de cette étape, votre fichier HTML devrait se présenter comme suit :

    ```html
    <!DOCTYPE html>
    <html>

    <head>
        <title></title>

        <meta charset="utf-8">

        <!-- Ensures that IE and Edge uses the latest version and doesn't emulate an older version -->
        <meta http-equiv="x-ua-compatible" content="IE=Edge">

        <!-- Ensures the web page looks good on all screen sizes. -->
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

        <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
        <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
        <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.js"></script>

        <script type='text/javascript'>

            var map;

            function GetMap() {
                //Initialize a map instance.
                map = new atlas.Map('myMap', {
                    view: 'Auto',

                    //Add your Azure Maps subscription key to the map SDK. Get an Azure Maps key at https://azure.com/maps
                    authOptions: {
                        authType: 'subscriptionKey',
                        subscriptionKey: '<Your Azure Maps Key>'
                    }
                });

                //Wait until the map resources are ready.
                map.events.add('ready', function() {

                    // Write your code here to make sure it runs once the map resources are ready

                });
            }
        </script>
    </head>

    <body onload="GetMap()">
        <div id="myMap"></div>
    </body>

    </html>
    ```

2. Chargez le module E/S spatiales Azure Maps. Dans le cadre de cet exercice, utilisez le CDN pour le module E/S spatiales Azure Maps. Ajoutez la référence ci-dessous à l’élément `<head>` de votre fichier HTML :

    ```html
    <script src="https://atlas.microsoft.com/sdk/javascript/spatial/0/atlas-spatial.js"></script>
    ```

3. Initialisez un `datasource` et ajoutez la source de données à la carte. Initialisez un `layer` et ajoutez la source de données à la couche. Affichez ensuite la source de données et la couche. Avant de faire défiler vers le haut pour voir le code complet à l’étape suivante, réfléchissez aux meilleurs emplacements où placer les extraits de code de source de données et de couche. Pour rappel, avant de manipuler la carte par programmation, nous devons attendre que la ressource de carte soit prête.

    ```javascript
    var datasource, layer;
    ```

    and

    ```javascript
    //Create a data source and add it to the map.
    datasource = new atlas.source.DataSource();
    map.sources.add(datasource);
    
    //Add a simple data layer for rendering the data.
    layer = new atlas.layer.SimpleDataLayer(datasource);
    map.layers.add(layer);
    ```

4. Dans l'ensemble, votre code HTML devrait ressembler au code suivant. Cet exemple montre comment lire un fichier XML à partir d’une URL. Ensuite, chargez et affichez les données de fonctionnalité du fichier sur la carte. 

    ```html
    <!DOCTYPE html>
    <html>

    <head>
        <title>Spatial IO Module Example</title>

        <meta charset="utf-8">

        <!-- Ensures that IE and Edge uses the latest version and doesn't emulate an older version -->
        <meta http-equiv="x-ua-compatible" content="IE=Edge">

        <!-- Ensures the web page looks good on all screen sizes. -->
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

        <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
        <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
        <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.js"></script>

        <!-- Add reference to the Azure Maps Spatial IO module. -->
        <script src="https://atlas.microsoft.com/sdk/javascript/spatial/0/atlas-spatial.js"></script>

        <script type='text/javascript'>
            var map, datasource, layer;

            function GetMap() {
                //Initialize a map instance.
                map = new atlas.Map('myMap', {
                    view: 'Auto',

                    //Add your Azure Maps subscription key to the map SDK. Get an Azure Maps key at https://azure.com/maps
                    authOptions: {
                        authType: 'subscriptionKey',
                        subscriptionKey: '<Your Azure Maps Key>'
                    }
                });

                //Wait until the map resources are ready.
                map.events.add('ready', function() {

                    //Create a data source and add it to the map.
                    datasource = new atlas.source.DataSource();
                    map.sources.add(datasource);

                    //Add a simple data layer for rendering the data.
                    layer = new atlas.layer.SimpleDataLayer(datasource);
                    map.layers.add(layer);

                    //Read an XML file from a URL or pass in a raw XML string.
                    atlas.io.read('superCoolKmlFile.xml').then(r => {
                        if (r) {
                            //Add the feature data to the data source.
                            datasource.add(r);

                            //If bounding box information is known for data, set the map view to it.
                            if (r.bbox) {
                                map.setCamera({
                                    bounds: r.bbox,
                                    padding: 50
                                });
                            }
                        }
                    });
                });
            }
        </script>
    </head>

    <body onload="GetMap()">
        <div id="myMap"></div>
    </body>

    </html>
    ```

5. N'oubliez pas de remplacer `<Your Azure Maps Key>` par votre clé primaire. Lorsque vous ouvrez votre fichier HTML, vous verrez des résultats semblables à l’image suivante :

    <center>

    ![Exemple de données spatiales](./media/how-to-use-spatial-io-module/spatial-data-example.png)

    </center>

## <a name="next-steps"></a>Étapes suivantes

La fonctionnalité présentée ici n’est qu’une des nombreuses fonctionnalités disponibles dans le module E/S spatiales. Lisez les guides ci-dessous pour apprendre à utiliser d’autres fonctionnalités dans le module E/S spatiales :

> [!div class="nextstepaction"]
> [Ajouter une couche de données simple](spatial-io-add-simple-data-layer.md)

> [!div class="nextstepaction"]
> [Lire et écrire des données spatiales](spatial-io-read-write-spatial-data.md)

> [!div class="nextstepaction"]
> [Ajouter une couche OGC](spatial-io-add-ogc-map-layer.md)

> [!div class="nextstepaction"]
> [Se connecter à un service WFS](spatial-io-connect-wfs-service.md)

> [!div class="nextstepaction"]
> [Exploiter les opérations de base](spatial-io-core-operations.md)

> [!div class="nextstepaction"]
> [Détails sur les formats de données pris en charge](spatial-io-supported-data-format-details.md)

Reportez-vous à la documentation E/S spatiales Azure Maps :

> [!div class="nextstepaction"]
> [Package E/S spatiales Azure Maps](/javascript/api/azure-maps-spatial-io/)