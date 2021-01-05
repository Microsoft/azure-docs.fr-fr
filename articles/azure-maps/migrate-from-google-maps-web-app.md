---
title: Tutoriel – Migrer une application web de Google Maps vers Microsoft Azure Maps
description: Tutoriel expliquant comment migrer une application web de Google Maps vers Microsoft Azure Maps.
author: rbrundritt
ms.author: richbrun
ms.date: 12/07/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: devx-track-js
ms.openlocfilehash: 4dee8de8f42b78ecdab9d9e15bb277d58fa8ba70
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96905058"
---
# <a name="tutorial---migrate-a-web-app-from-google-maps"></a>Tutoriel : Migrer une application web à partir de Google Maps

La plupart des applications web qui utilisent Google Maps utilisent le kit SDK JavaScript Google Maps v3. Le Kit de développement logiciel (SDK) web Azure Maps est le kit de développement logiciel (SDK) Azure approprié vers lequel migrer. Le SDK web Azure Maps vous permet de personnaliser des cartes interactives avec du contenu et des images qui vous sont propres. Vous pouvez exécuter votre application sur des applications web ou mobiles. Ce contrôle utilise WebGL, ce qui vous permet d’afficher d’importants jeux de données avec des performances élevées. Développez avec le kit de développement logiciel (SDK) à l’aide de JavaScript ou de TypeScript. Dans ce didacticiel, vous apprendrez à :

> [!div class="checklist"]
> * Charger une carte
> * Localiser une carte
> * Ajouter des marqueurs, des polylignes et des polygones
> * Afficher des informations dans une fenêtre contextuelle ou d’informations
> * Charger et afficher des données KML et GeoJSON
> * Regrouper des marqueurs
> * Superposer une couche de mosaïques
> * Afficher les données du trafic
> * Ajouter une superposition de sol

Vous allez également apprendre : 

> [!div class="checklist"]
> * À effectuer des tâches de cartographie courantes à l’aide du SDK web Azure Maps
> * Des bonnes pratiques pour améliorer les performances et l’expérience utilisateur
> * Des conseils pour que votre application exploite des fonctionnalités plus avancées disponibles dans Azure Maps

Si vous migrez une application web existante, vérifiez si elle utilise une bibliothèque de contrôle de carte open source. Voici des exemples of bibliothèque de contrôle de carte open source : Cesium, Leaflet et OpenLayers. Vous pouvez quand même effectuer la migration si votre application utilise une bibliothèque de contrôle de carte open source et que vous ne souhaitez pas utiliser le SDK web Azure Maps. Dans ce cas, connectez votre application aux services de mosaïques Azure Maps ([mosaïques routières](/rest/api/maps/render/getmaptile) \| [mosaïques satellites](/rest/api/maps/render/getmapimagerytile)). Les points suivants indiquent en détail comment utiliser Azure Maps dans certaines bibliothèques de contrôle de carte open source couramment utilisées.

- Cesium : contrôle carte 3D pour le Web. [Documentation](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Raster%20Tiles%20in%20Cesium%20JS) \| [de l’exemple de code](https://cesiumjs.org/)
- Leaflet : contrôle de carte 2D léger pour le Web. [Documentation](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Azure%20Maps%20Raster%20Tiles%20in%20Leaflet%20JS) \| [de l’exemple de code](https://leafletjs.com/)
- OpenLayers : contrôle de carte 2D pour le Web qui prend en charge les projections. [Documentation](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Raster%20Tiles%20in%20OpenLayers) \| [de l’exemple de code](https://openlayers.org/)

Si vous développez à l’aide d’un framework JavaScript, l’un des projets open source suivants peut être utile :

- [ng-azure-maps](https://github.com/arnaudleclerc/ng-azure-maps) : wrapper Angular 10 autour d’Azure Maps.
- [AzureMapsControl.Components](https://github.com/arnaudleclerc/AzureMapsControl.Components) : composant Blazor Azure Maps.
- [Composant react Azure Maps](https://github.com/WiredSolutions/react-azure-maps) : wrapper react pour le contrôle Azure Maps.
- [Vue Azure Maps](https://github.com/rickyruiz/vue-azure-maps) : composant Azure Maps pour l’application Vue.

## <a name="prerequisites"></a>Prérequis 

1. Connectez-vous au [portail Azure](https://portal.azure.com). Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
2. [Créer un compte Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account)
3. [Obtenir une clé d’abonnement principale](quick-demo-map-app.md#get-the-primary-key-for-your-account), également appelée clé primaire ou clé d’abonnement. Pour plus d’informations sur l’authentification dans Azure Maps, voir [Gérer l’authentification dans Azure Maps](how-to-manage-authentication.md).

## <a name="key-features-support"></a>Prise en charge des fonctionnalités clés

Le tableau liste les principales fonctionnalités d’API du SDK JavaScript Google Maps v3 et la fonctionnalité d’API prise en charge dans le SDK web Azure Maps.

| Fonctionnalité Google Maps     | Prise en charge du kit de développement logiciel (SDK) Web Azure Maps |
|-------------------------|:--------------------------:|
| Marqueurs                 | ✓                          |
| Clustering de marqueurs       | ✓                          |
| Polylignes et polygones    | ✓                          |
| Couches de données             | ✓                          |
| Superposition de sol         | ✓                          |
| Cartes thermiques               | ✓                          |
| Couches de mosaïques             | ✓                          |
| Couche KML               | ✓                          |
| Outils de dessin           | ✓                          |
| Service de geocoder        | ✓                          |
| Service de directions      | ✓                          |
| Service Matrice des distances | ✓                          |
| Service d’élévation       | ✓                          |

## <a name="notable-differences-in-the-web-sdks"></a>Différences notables dans les kits de développement logiciel (SDK) Web

Voici quelques-unes des principales différences entre les SDK web Google Maps et Azure Maps à connaître :

- En plus de fournir un point de terminaison hébergé pour accéder au SDK web Azure Maps, un package NPM est disponible Incorporez le SDK web dans les applications. Pour plus d’informations, consultez [cette documentation](how-to-use-map-control.md). Ce package inclut aussi des définitions de TypeScript.
- Vous devez d’abord créer une instance de la classe Map dans Azure Maps. Attendez que l’événement `ready` ou `load` de carte se déclenche avant d’interagir par programmation avec la carte. Cet ordre permet de garantir que toutes les ressources de carte ont été chargées et qu’elles sont accessibles.
- Les deux plateformes utilisent un système de mosaïque similaire pour les cartes de base. La dimension des mosaïques dans Google Maps est de 256 pixels, alors qu’elle est de 512 pixels dans Azure Maps. Pour obtenir dans Azure Maps la même vue cartographique que dans Google Maps, diminuez le niveau de zoom Google Maps d’une unité dans Azure Maps.
- Dans Google Maps, les coordonnées se présentent sous la forme `latitude,longitude`, tandis qu’Azure Maps utilise `longitude,latitude`. Le format Azure Maps est conforme à la norme `[x, y]`, qui est suivie par la plupart des plateformes GIS.
- Les formes du kit de développement logiciel (SDK) Web Azure Maps sont basées sur le schéma GeoJSON. Les classes d’assistance sont exposées par le biais de l’espace de noms [*atlas.data*](/javascript/api/azure-maps-control/atlas.data). Il y a également la classe [*atlas.Shape*](/javascript/api/azure-maps-control/atlas.shape). Utilisez cette classe pour wrapper les objets GeoJSON, afin de faciliter leur mise à jour et leur maintenance d’une manière pouvant être liée aux données.
- Les coordonnées dans Azure Maps sont définies comme des objets Position. Une coordonnée est spécifiée sous la forme d’un tableau de nombres au format `[longitude,latitude]`, ou alors elle est spécifiée à l’aide d’un nouveau atlas.data.Position(longitude, latitude).
    > [!TIP]
    > La classe Position a une méthode d’assistance statique pour l’importation de coordonnées qui sont au format « Latitude, longitude ». La méthode [atlas.data.Position.fromLatLng](/javascript/api/azure-maps-control/atlas.data.position) peut souvent être remplacée par la méthode `new google.maps.LatLng` dans le code Google Maps.
- Au lieu de spécifier des informations de style pour chaque forme ajoutée à la carte, Azure Maps sépare les styles des données. Les données sont stockées dans une source de données et sont connectées aux calques de rendu. Le code Azure Maps utilise les sources de données pour restituer les données. Cette approche offre un gain de performances amélioré. De plus, de nombreux calques prennent en charge le style basé sur les données dans lequel la logique métier peut être ajoutée aux options de style de calque. Cette prise en charge change la façon dont les formes individuelles sont restituées dans un calque en fonction des propriétés définies dans la forme.

## <a name="web-sdk-side-by-side-examples"></a>Exemples côte à côte du kit de développement logiciel (SDK) Web

Cette collection contient des exemples de code pour chaque plateforme, chacun couvrant un cas d’usage courant. Elle est destinée à vous aider à migrer votre application web du SDK JavaScript Google Maps v3 vers le SDK web Azure Maps. Des exemples de code liés aux applications web sont fournis en JavaScript. Toutefois, Azure Maps fournit également des définitions TypeScript sous la forme d’une option supplémentaire disponible par le biais d’un module [NPM](how-to-use-map-control.md).

**Rubriques**

- [Charger une carte](#load-a-map)
- [Localisation de la carte](#localizing-the-map)
- [Définition de la vue cartographique](#setting-the-map-view)
- [Ajout d’un marqueur](#adding-a-marker)
- [Ajout d’un marqueur personnalisé](#adding-a-custom-marker)
- [Ajout d’une polyligne](#adding-a-polyline)
- [Ajout d’un polygone](#adding-a-polygon)
- [Afficher une fenêtre d’info](#display-an-info-window)
- [Importer un fichier GeoJSON](#import-a-geojson-file)- 
- [Clustering de marqueurs](#marker-clustering)
- [Ajouter une carte thermique](#add-a-heat-map)
- [Superposer une couche de mosaïques](#overlay-a-tile-layer)
- [Afficher les données du trafic](#show-traffic-data)
- [Ajouter une superposition de sol](#add-a-ground-overlay)
- [Ajouter des données KML à la carte](#add-kml-data-to-the-map)

### <a name="load-a-map"></a>Charger une carte

Les deux SDK nécessitent les mêmes étapes pour charger une carte :

- Ajout d'une référence au Kit de développement logiciel (SDK) de la carte.
- Ajoutez une balise `div` au corps de la page, qui fera office d’espace réservé pour la carte.
- Créez une fonction JavaScript qui est appelée lorsque la page a été chargée.
- Créez une instance de la classe de carte respective.

**Quelques différences clés**

- Google Maps requiert la spécification d’une clé de compte dans la référence de script de l’API. Les informations d’authentification pour Azure Maps sont spécifiées en tant qu’options de la classe de carte. Ces informations d’identification peuvent être une clé d’abonnement ou des informations Azure Active Directory.
- Google Maps accepte une fonction de rappel dans la référence de script de l’API, qui est utilisée pour appeler une fonction d’initialisation afin de charger la carte. Avec Azure Maps, l’événement OnLoad de la page doit être utilisé.
- Lors du référencement de l’élément `div` dans lequel la carte est rendue, la classe `Map` dans Azure Maps nécessite uniquement la valeur `id`, tandis que Google Maps requiert un objet `HTMLElement`.
- Les coordonnées dans Azure Maps sont définies comme des objets Position qui peuvent être spécifiés comme un simple tableau de nombres au format `[longitude, latitude]`.
- Le niveau de zoom dans Azure Maps est inférieur d’un niveau à celui de l’exemple Google Maps. Cette différence est due à la différence de taille du système de mosaïques entre les deux plateformes.
- Azure Maps n’ajoute aucun contrôle de navigation au canevas de carte. Par conséquent, par défaut, une carte n’a pas de boutons de zoom et de boutons de style de carte. Il existe cependant des options de commandes pour ajouter un sélecteur de style de carte, des boutons de zoom, une boussole ou une commande de rotation et un réglage de vitesse.
- Un gestionnaire d’événements est ajouté dans Azure Maps pour surveiller l’événement `ready` de l’instance de la carte. Cet événement se déclenche quand la carte a terminé le chargement du contexte WebGL et de toutes les ressources nécessaires. Une fois le chargement de la carte terminé, ajoutez tout code que vous voulez exécuter à ce gestionnaire d’événements.

Les exemples de base ci-dessous utilisent Google Maps pour charger une carte centrée sur New York aux coordonnées suivantes : longitude -73.985, latitude 40.747. La carte est au niveau de zoom 12.

#### <a name="before-google-maps"></a>Avant : Google Maps

Afficher une carte Google centrée et zoomée sur un endroit.

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <script type='text/javascript'>
        var map;

        function initMap() {
            map = new google.maps.Map(document.getElementById('myMap'), {
                center: new google.maps.LatLng(40.747, -73.985),
                zoom: 12
            });
        }
    </script>

    <!-- Google Maps Script Reference -->
    <script src="https://maps.googleapis.com/maps/api/js?callback=initMap&key=[Your Google Maps Key]" async defer></script>
</head>
<body>
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```

L’exécution de ce code dans un navigateur affichera une carte ressemblant à l’image suivante :

![Google Maps simple](media/migrate-google-maps-web-app/simple-google-map.png)

#### <a name="after-azure-maps"></a>Après : Azure Maps

Charger une carte avec la même vue dans Azure Maps, ainsi qu’un contrôle de style cartographique et des boutons de zoom.

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
    <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
    <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>

    <script type='text/javascript'>
        var map;

        function initMap() {
            map = new atlas.Map('myMap', {
                center: [-73.985, 40.747],  //Format coordinates as longitude, latitude.
                zoom: 11,   //Subtract the zoom level by one.

                //Specify authentication information when loading the map.
                authOptions: {
                    authType: 'subscriptionKey',
                    subscriptionKey: '<Your Azure Maps Key>'
                }
            });

            //Wait until the map resources are ready.
            map.events.add('ready', function () {
                //Add zoom controls to bottom right of map.
                map.controls.add(new atlas.control.ZoomControl(), {
                    position: 'bottom-right'
                });

                //Add map style control in top left corner of map.
                map.controls.add(new atlas.control.StyleControl(), {
                    position: 'top-left'
                });
            });
        }
    </script>
</head>
<body onload="initMap()">
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```

L’exécution de ce code dans un navigateur affichera une carte ressemblant à l’image suivante :

![Azure Maps simple](media/migrate-google-maps-web-app/simple-azure-maps.png)

Pour obtenir des informations détaillées sur la façon de configurer et d’utiliser le contrôle de carte Azure Maps dans une application web, cliquez [ici](how-to-use-map-control.md).

> [!NOTE]
> Contrairement à Google Maps, Azure Maps n’a pas besoin de niveau de zoom et de centre initial pour charger la carte. Si ces informations ne sont pas fournies lors du chargement de la carte, Azure Maps tente de déterminer la ville de l’utilisateur. Il centrera et effectuera un zoom sur la carte à cet endroit.

**Ressources supplémentaires :**

- Azure Maps fournit également des contrôles de navigation pour faire pivoter et mettre en forme la vue cartographique comme indiqué [ici](map-add-controls.md).

### <a name="localizing-the-map"></a>Localisation de la carte

Si votre audience est répartie dans plusieurs pays/régions ou parle différentes langues, la localisation est importante.

#### <a name="before-google-maps"></a>Avant : Google Maps

Pour localiser les paramètres Google Maps, ajoutez des paramètres de langue et de région.

```html
<script type="text/javascript" src=" https://maps.googleapis.com/maps/api/js?callback=initMap&key=[api_key]& language=[language_code]&region=[region_code]" async defer></script>
```

Voici un exemple de Google Maps avec la langue définie sur « fr-FR ».

![Localisation de Google Maps](media/migrate-google-maps-web-app/google-maps-localization.png)

#### <a name="after-azure-maps"></a>Après : Azure Maps

Azure Maps propose deux méthodes pour définir la langue et l’affichage régional de la carte. La première option consiste à ajouter ces informations à l’espace de noms global *Atlas*. Toutes les instances de contrôle de carte dans votre application utiliseront alors ces paramètres par défaut. Le code suivant définit la langue sur le français (« fr-FR ») et l’affichage régional sur « auto » :

```javascript
atlas.setLanguage('fr-FR');
atlas.setView('auto');
```

La deuxième option consiste à transmettre ces informations dans les options de carte lors du chargement de la carte. Comme ceci :

```javascript
map = new atlas.Map('myMap', {
    language: 'fr-FR',
    view: 'auto',

    authOptions: {
        authType: 'subscriptionKey',
        subscriptionKey: '<Your Azure Maps Key>'
    }
});
```

> [!NOTE]
> Avec Azure Maps, il est possible de charger plusieurs instances de carte sur la même page avec des paramètres de langue et de région différents. Il est également possible de mettre à jour ces paramètres dans la carte après son chargement. 

Vous trouverez une liste détaillée des langues prises en charge dans Azure Maps [ici](supported-languages.md).

Voici un exemple de Azure Maps avec la langue définie sur « fr » et la région de l’utilisateur définie sur « fr-FR ».

![Localisation de Google Maps](media/migrate-google-maps-web-app/azure-maps-localization.png)

### <a name="setting-the-map-view"></a>Définition de la vue cartographique

Les cartes dynamiques dans Azure et Google Maps peuvent être déplacées par programmation vers de nouveaux emplacements géographiques. Pour ce faire, appelez les fonctions appropriées en JavaScript. Les exemples montrent comment faire en sorte que la carte affiche une image aérienne satellite, centrer la carte sur un endroit et définir le niveau de zoom sur 15 dans Google Maps. Les coordonnées d’emplacement suivantes sont utilisées : longitude : -111,0225 et latitude : 35.0272.

> [!NOTE]
> Google Maps utilise des mosaïques de 256 pixels, tandis qu’Azure Maps utilise une mosaïque de 512 pixels. Ainsi, Azure Maps nécessite moins de requêtes réseau pour charger la même zone réactive que Google Maps. En raison du mode de fonctionnement des pyramides de mosaïques dans les contrôles de carte, vous devez diminuer le niveau de zoom utilisé dans Google Maps d’une unité lors de l’utilisation d’Azure Maps. Cette opération arithmétique permet de s’assurer que les mosaïques plus grandes dans Azure Maps affichent la même zone réactive que dans Google Maps.

#### <a name="before-google-maps"></a>Avant : Google Maps

Déplacez le contrôle de carte Google Maps à l’aide de la méthode `setOptions`. Cette méthode vous permet de spécifier le centre de la carte et un niveau de zoom.

```javascript
map.setOptions({
    mapTypeId: google.maps.MapTypeId.SATELLITE,
    center: new google.maps.LatLng(35.0272, -111.0225),
    zoom: 15
});
```

![Vue définie de Google Maps](media/migrate-google-maps-web-app/google-maps-set-view.png)

#### <a name="after-azure-maps"></a>Après : Azure Maps

Dans Azure Maps, changez la position de la carte à l’aide de la méthode `setCamera` et changez le style de la carte à l’aide de la méthode `setStyle`. Les coordonnées dans Azure Maps sont au format « longitude, latitude » et la valeur de niveau de zoom est soustraite d’une unité.

```javascript
map.setCamera({
    center: [-111.0225, 35.0272],
    zoom: 14
});

map.setStyle({
    style: 'satellite'
});
```

![Vue définie d’Azure Maps](media/migrate-google-maps-web-app/azure-maps-set-view.jpeg)

**Ressources supplémentaires :**

- [Choisir un style de carte](choose-map-style.md)
- [Styles de carte pris en charge](supported-map-styles.md)

### <a name="adding-a-marker"></a>Ajout d’un marqueur

Dans Azure Maps il existe plusieurs façons de faire en sorte que les données de point soient rendues sur la carte :

- **Marqueurs HTML** : effectue le rendu des points à l’aide d’éléments DOM traditionnels. Les marqueurs HTML prennent en charge le glissement.
- **Calque de symbole** : effectue le rendu des points avec une icône ou un texte dans le contexte WebGL.
- **Calque de bulles** : affiche les données de point sous forme de cercles sur la carte. Les rayons des cercles peuvent être mis à l’échelle en fonction des propriétés des données.

Effectuez le rendu des calques de symboles et de bulles dans le contexte WebGL. Les deux calques peuvent afficher de grands ensembles de points sur la carte. Ces calques requièrent que les données soient stockées dans une source de données. Les sources de données et les calques de rendu doivent être ajoutées à la carte après le déclenchement de l’événement `ready`. Les marqueurs HTML sont rendus sous la forme d’éléments DOM dans la page, et n’utilisent pas de source de données. Plus le nombre d’éléments DOM d’une page est élevé, plus la page devient rapide. En cas de rendu de plus de quelques centaines de points sur une carte, nous vous recommandons d’utiliser à la place l’un des calques de rendu.

Nous allons ajouter un marqueur à la carte, avec le chiffre 10 superposé comme étiquette. Utilisez longitude : -0.2 et latitude : 51.5.

#### <a name="before-google-maps"></a>Avant : Google Maps

Avec Google Maps, ajoutez des marqueurs à la carte à l’aide de la classe `google.maps.Marker` et spécifiez la carte comme l’une des options.

```javascript
//Create a marker and add it to the map.
var marker = new google.maps.Marker({
    position: new google.maps.LatLng(51.5, -0.2),
    label: '10',
    map: map
});
```

![Marqueur Google Maps](media/migrate-google-maps-web-app/google-maps-marker.png)

**Après : Azure Maps à l’aide de marqueurs HTML**

Dans Azure Maps, utilisez des marqueurs HTML pour afficher un point sur la carte. Ils sont recommandés pour les applications qui ne doivent afficher qu’un petit nombre de points sur la carte. Pour utiliser un marqueur HTML, créez une instance de la classe `atlas.HtmlMarker`. Définissez les options de texte et de position, puis ajoutez le marqueur à la carte à l’aide de la méthode `map.markers.add`.

```javascript
//Create a HTML marker and add it to the map.
map.markers.add(new atlas.HtmlMarker({
    text: '10',
    position: [-0.2, 51.5]
}));
```

![Marqueur HTML Azure Maps](media/migrate-google-maps-web-app/azure-maps-html-marker.png)

**Après : Azure Maps à l’aide d’un calque de symbole**

Pour un calque de symbole, ajoutez les données à une source de données. Attachez la source de données au calque. En outre, la source de données et la couche doivent être ajoutées à la carte après le déclenchement de l’événement `ready`. Pour afficher une valeur de texte unique au-dessus d’un symbole, les informations de texte doivent être stockées en tant que propriété du point de données. La propriété doit être référencée dans l’option `textField` du calque. Cette approche représente un peu plus de travail que l’utilisation des marqueurs HTML, mais elle offre de meilleures performances.

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
    <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
    <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>

    <script type='text/javascript'>
        var map, datasource;

        function initMap() {
            map = new atlas.Map('myMap', {
                center: [-0.2, 51.5],
                zoom: 9,
                
                authOptions: {
                    authType: 'subscriptionKey',
                    subscriptionKey: '<Your Azure Maps Key>'
                }
            });

            //Wait until the map resources are ready.
            map.events.add('ready', function () {

                //Create a data source and add it to the map.
                datasource = new atlas.source.DataSource();
                map.sources.add(datasource);

                //Create a point feature, add a property to store a label for it, and add it to the data source.
                datasource.add(new atlas.data.Feature(new atlas.data.Point([-0.2, 51.5]), {
                    label: '10'
                }));

                //Add a layer for rendering point data as symbols.
                map.layers.add(new atlas.layer.SymbolLayer(datasource, null, {
                    textOptions: {
                        //Use the label property to populate the text for the symbols.
                        textField: ['get', 'label'],
                        color: 'white',
                        offset: [0, -1]
                    }
                }));
            });
        }
    </script>
</head>
<body onload="initMap()">
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```

![Calque de symbole Azure Maps](media/migrate-google-maps-web-app/azure-maps-symbol-layer.png)

**Ressources supplémentaires :**

- [Créer une source de données](create-data-source-web-sdk.md)
- [Ajouter une calque de symbole](map-add-pin.md)
- [Ajouter un calque de bulles](map-add-bubble-layer.md)
- [Données de point de cluster](clustering-point-data-web-sdk.md)
- [Ajouter des marqueurs HTML](map-add-custom-html.md)
- [Utiliser des expressions de style basées sur les données](data-driven-style-expressions-web-sdk.md)
- [Options de l’icône de calque de symbole](/javascript/api/azure-maps-control/atlas.iconoptions)
- [Option de texte du calque de symbole](/javascript/api/azure-maps-control/atlas.textoptions)
- [Classe de marqueur HTML](/javascript/api/azure-maps-control/atlas.htmlmarker)
- [Options du marqueur HTML](/javascript/api/azure-maps-control/atlas.htmlmarkeroptions)

### <a name="adding-a-custom-marker"></a>Ajout d’un marqueur personnalisé

Vous pouvez utiliser des images personnalisées pour représenter des points sur une carte. La carte ci-dessous utilise une image personnalisée pour afficher un point sur la carte. Le point est affiché à la latitude : 51.5 et à la longitude : -0.2. L’ancre décale la position du marqueur afin que le point de l’icône de punaise s’aligne sur la position correcte sur la carte.

<center>

![image de punaise jaune](media/migrate-google-maps-web-app/yellow-pushpin.png)<br/>
yellow-pushpin.png</center>


#### <a name="before-google-maps"></a>Avant : Google Maps

Créez un marqueur personnalisé en spécifiant un objet `Icon` qui contient l’`url` de l’image. Spécifiez un point `anchor` pour aligner le point de l’image de punaise avec la coordonnée sur la carte. La valeur d’ancrage dans Google Maps est relative à l’angle supérieur gauche de l’image.

```javascript
var marker = new google.maps.Marker({
    position: new google.maps.LatLng(51.5, -0.2),
    icon: {
        url: 'https://azuremapscodesamples.azurewebsites.net/Common/images/icons/ylw-pushpin.png',
        anchor: new google.maps.Point(5, 30)
    },
    map: map
});
```


![Marqueur personnalisé Google Maps](media/migrate-google-maps-web-app/google-maps-custom-marker.png)

**Après : Azure Maps à l’aide de marqueurs HTML**

Pour personnaliser un marqueur HTML, transmettez un `string` ou `HTMLElement` HTML à l’option `htmlContent` du marqueur. Utilisez l’option `anchor` pour spécifier la position relative du marqueur par rapport aux coordonnées de position. Assignez l’un des neuf points de référence définis à l’option `anchor`. Ces points définis sont : « center », « top », « bottom », « left », « right », « top-left », « top-right », « bottom-left », « bottom-right ». Le contenu est ancré par défaut au centre en bas du contenu HTML. Pour faciliter la migration du code à partir de Google Maps, définissez le `anchor` sur « top-left », puis utilisez l’option `pixelOffset` avec le même décalage que celui utilisé dans Google Maps. Les décalages dans Azure Maps se déplacent dans la direction opposée à celle des décalages dans Google Maps. Vous devez donc multiplier les décalages par moins un.

> [!TIP]
> Ajoutez `pointer-events:none` en tant que style au contenu HTML pour désactiver le comportement de glissement par défaut dans Microsoft Edge, qui affichera une icône indésirable.

```javascript
map.markers.add(new atlas.HtmlMarker({
    htmlContent: '<img src="https://azuremapscodesamples.azurewebsites.net/Common/images/icons/ylw-pushpin.png" style="pointer-events: none;" />',
    anchor: 'top-left',
    pixelOffset: [-5, -30],
    position: [-0.2, 51.5]
}));
```

![Marqueur HTML personnalisé Azure Maps](media/migrate-google-maps-web-app/azure-maps-custom-html-marker.png)

**Après : Azure Maps à l’aide d’un calque de symbole**

Les calques de symboles dans Azure Maps prennent également en charge les images personnalisées. Tout d’abord, chargez l’image dans les ressources de la carte et attribuez-lui un ID unique. Référencez l’image dans le calque de symbole. Utilisez l’option `offset` pour aligner l’image sur le point correct sur la carte. Utilisez l’option `anchor` pour spécifier la position relative du symbole par rapport aux coordonnées de position. Utilisez l’un des neuf points de référence définis. Ces points sont : « center », « top », « bottom », « left », « right », « top-left », « top-right », « bottom-left », « bottom-right ». Le contenu est ancré par défaut au centre en bas du contenu HTML. Pour faciliter la migration du code à partir de Google Maps, définissez le `anchor` sur « top-left », puis utilisez l’option `offset` avec le même décalage que celui utilisé dans Google Maps. Les décalages dans Azure Maps se déplacent dans la direction opposée à celle des décalages dans Google Maps. Vous devez donc multiplier les décalages par moins un.

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
    <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
    <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>

    <script type='text/javascript'>
        var map, datasource;

        function initMap() {
            map = new atlas.Map('myMap', {
                center: [-0.2, 51.5],
                zoom: 9,
                authOptions: {
                    authType: 'subscriptionKey',
                    subscriptionKey: '<Your Azure Maps Key>'
                }
            });

            //Wait until the map resources are ready.
            map.events.add('ready', function () {

                //Load the custom image icon into the map resources.
                map.imageSprite.add('my-yellow-pin', 'https://azuremapscodesamples.azurewebsites.net/Common/images/icons/ylw-pushpin.png').then(function () {

                    //Create a data source and add it to the map.
                    datasource = new atlas.source.DataSource();
                    map.sources.add(datasource);

                    //Create a point and add it to the data source.
                    datasource.add(new atlas.data.Point([-0.2, 51.5]));

                    //Add a layer for rendering point data as symbols.
                    map.layers.add(new atlas.layer.SymbolLayer(datasource, null, {
                        iconOptions: {
                            //Set the image option to the id of the custom icon that was loaded into the map resources.
                            image: 'my-yellow-pin',
                            anchor: 'top-left',
                            offset: [-5, -30]
                        }
                    }));
                });
            });
        }
    </script>
</head>
<body onload="initMap()">
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```

![Calque de symbole d'icône personnalisée Azure Maps](media/migrate-google-maps-web-app/azure-maps-custom-icon-symbol-layer.png)</

> [!TIP]
> Pour afficher des points personnalisés avancés, utilisez plusieurs calques de rendu ensemble. Supposons, par exemple, que vous souhaitiez avoir plusieurs punaises qui ont la même icône sur des cercles de couleurs différentes. Au lieu de créer une série d’images pour chaque superposition de couleurs, ajoutez un calque de symbole au-dessus d’un calque de bulles. Faites en sorte que les punaises référencent la même source de données. Cette approche est plus efficace que de créer une série d’images et de maintenir une série d’images différentes.

**Ressources supplémentaires :**

- [Créer une source de données](create-data-source-web-sdk.md)
- [Ajouter une calque de symbole](map-add-pin.md)
- [Ajouter des marqueurs HTML](map-add-custom-html.md)
- [Utiliser des expressions de style basées sur les données](data-driven-style-expressions-web-sdk.md)
- [Options de l’icône de calque de symbole](/javascript/api/azure-maps-control/atlas.iconoptions)
- [Option de texte du calque de symbole](/javascript/api/azure-maps-control/atlas.textoptions)
- [Classe de marqueur HTML](/javascript/api/azure-maps-control/atlas.htmlmarker)
- [Options du marqueur HTML](/javascript/api/azure-maps-control/atlas.htmlmarkeroptions)

### <a name="adding-a-polyline"></a>Ajout d’une polyligne

Utilisez des polylignes pour représenter une ligne ou un tracé sur la carte. Nous allons créer une polyligne en pointillés sur la carte.

#### <a name="before-google-maps"></a>Avant : Google Maps

La classe Polyline accepte un ensemble d’options. Transmettez un tableau de coordonnées dans l’option `path` de la polyligne.

```javascript
//Get the center of the map.
var center = map.getCenter();

//Define a symbol using SVG path notation, with an opacity of 1.
var lineSymbol = {
    path: 'M 0,-1 0,1',
    strokeOpacity: 1,
    scale: 4
};

//Create the polyline.
var line = new google.maps.Polyline({
    path: [
        center,
        new google.maps.LatLng(center.lat() - 0.5, center.lng() - 1),
        new google.maps.LatLng(center.lat() - 0.5, center.lng() + 1)
    ],
    strokeColor: 'red',
    strokeOpacity: 0,
    strokeWeight: 4,
    icons: [{
        icon: lineSymbol,
        offset: '0',
        repeat: '20px'
    }]
});

//Add the polyline to the map.
line.setMap(map);
```

![Polyligne Google Maps](media/migrate-google-maps-web-app/google-maps-polyline.png)

#### <a name="after-azure-maps"></a>Après : Azure Maps

Les polylignes sont appelées objets `LineString` ou `MultiLineString`. Ces objets peuvent être ajoutés à une source de données et restitués à l’aide d’une couche de lignes. Ajoutez `LineString` à une source de données, puis ajoutez la source de données à un `LineLayer` pour l’afficher.

```javascript
//Get the center of the map.
var center = map.getCamera().center;

//Create a data source and add it to the map.
var datasource = new atlas.source.DataSource();
map.sources.add(datasource);

//Create a line and add it to the data source.
datasource.add(new atlas.data.LineString([
    center,
    [center[0] - 1, center[1] - 0.5],
    [center[0] + 1, center[1] - 0.5]
]));

//Add a layer for rendering line data.
map.layers.add(new atlas.layer.LineLayer(datasource, null, {
    strokeColor: 'red',
    strokeWidth: 4,
    strokeDashArray: [3, 3]
}));
```
![Polyligne Azure Maps](media/migrate-google-maps-web-app/azure-maps-polyline.png)

**Ressources supplémentaires :**

- [Ajouter des lignes à la carte](map-add-line-layer.md)
- [Options du calque de ligne](/javascript/api/azure-maps-control/atlas.linelayeroptions)
- [Utiliser des expressions de style basées sur les données](data-driven-style-expressions-web-sdk.md)

### <a name="adding-a-polygon"></a>Ajout d’un polygone

Azure Maps et Google Maps offrent une prise en charge similaire des polygones. Les polygones sont utilisés pour représenter une zone sur la carte. Les exemples suivants montrent comment créer un polygone qui forme un triangle en fonction de la coordonnée centrale de la carte.

#### <a name="before-google-maps"></a>Avant : Google Maps

La classe Polygon accepte un ensemble d’options. Transmettez un tableau de coordonnées à l’option `paths` du polygone.

```javascript
//Get the center of the map.
var center = map.getCenter();

//Create a polygon.
var polygon = new google.maps.Polygon({
    paths: [
        center,
        new google.maps.LatLng(center.lat() - 0.5, center.lng() - 1),
        new google.maps.LatLng(center.lat() - 0.5, center.lng() + 1),
        center
    ],
    strokeColor: 'red',
    strokeWeight: 2,
    fillColor: 'rgba(0, 255, 0, 0.5)'
});

//Add the polygon to the map
polygon.setMap(map);
```

![Polygone Google Maps](media/migrate-google-maps-web-app/google-maps-polygon.png)

#### <a name="after-azure-maps"></a>Après : Azure Maps

Ajoutez un objet `Polygon` ou `MultiPolygon` à une source de données. Affichez l’objet sur la carte à l’aide de calques. Affichez la zone d’un polygone à l’aide d’une couche de polygones. Ensuite, affichez le contour d’un polygone à l’aide d’une couche de lignes.

```javascript
//Get the center of the map.
var center = map.getCamera().center;

//Create a data source and add it to the map.
datasource = new atlas.source.DataSource();
map.sources.add(datasource);

//Create a polygon and add it to the data source.
datasource.add(new atlas.data.Polygon([
    center,
    [center[0] - 1, center[1] - 0.5],
    [center[0] + 1, center[1] - 0.5],
    center
]));

//Add a polygon layer for rendering the polygon area.
map.layers.add(new atlas.layer.PolygonLayer(datasource, null, {
    fillColor: 'rgba(0, 255, 0, 0.5)'
}));

//Add a line layer for rendering the polygon outline.
map.layers.add(new atlas.layer.LineLayer(datasource, null, {
    strokeColor: 'red',
    strokeWidth: 2
}));
```
![Polygone Azure Maps](media/migrate-google-maps-web-app/azure-maps-polygon.png)

**Ressources supplémentaires :**

- [Ajouter un polygone à la carte](map-add-shape.md)
- [Ajouter un cercle à la carte](map-add-shape.md#add-a-circle-to-the-map)
- [Options du calque de polygones](/javascript/api/azure-maps-control/atlas.polygonlayeroptions)
- [Options du calque de ligne](/javascript/api/azure-maps-control/atlas.linelayeroptions)
- [Utiliser des expressions de style basées sur les données](data-driven-style-expressions-web-sdk.md)

### <a name="display-an-info-window"></a>Afficher une fenêtre d’info

Des informations supplémentaires pour une entité peuvent être affichées sur la carte en tant que classe `google.maps.InfoWindow` dans Google Maps. Dans Azure Maps, cette fonctionnalité peut être obtenue à l’aide de la classe `atlas.Popup`. Les exemples suivants ajoutent un marqueur à la carte. Quand vous cliquez sur le marqueur, une fenêtre d’informations ou fenêtre contextuelle s’affiche.

#### <a name="before-google-maps"></a>Avant : Google Maps

Instanciez une fenêtre d’informations à l’aide du constructeur `google.maps.InfoWindow`.

```javascript
//Add a marker in which to display an infowindow for.
var marker = new google.maps.Marker({
    position: new google.maps.LatLng(47.6, -122.33),
    map: map
});

//Create an infowindow.
var infowindow = new google.maps.InfoWindow({
    content: '<div style="padding:5px"><b>Hello World!</b></div>'
});

//Add a click event to the marker to open the infowindow.
marker.addListener('click', function () {
    infowindow.open(map, marker);
});
```
![Fenêtre contextuelle Google Maps](media/migrate-google-maps-web-app/google-maps-popup.png)

#### <a name="after-azure-maps"></a>Après : Azure Maps

Nous allons utiliser popup pour afficher des informations supplémentaires sur l’endroit. Transmettez un objet `string` ou `HTMLElement` HTML à l’option `content` de la fenêtre contextuelle. Si vous le souhaitez, les fenêtres contextuelles peuvent être affichées indépendamment de toute forme. Par conséquent, elles nécessitent la spécification d’une valeur `position`. Spécifiez la valeur `position`. Pour afficher une fenêtre contextuelle, appelez la méthode `open` et transmettez le `map` dans lequel la fenêtre contextuelle doit être affichée.

```javascript
//Add a marker to the map in which to display a popup for.
var marker = new atlas.HtmlMarker({
    position: [-122.33, 47.6]
});

//Add the marker to the map.
map.markers.add(marker);

//Create a popup.
var popup = new atlas.Popup({
    content: '<div style="padding:5px"><b>Hello World!</b></div>',
    position: [-122.33, 47.6],
    pixelOffset: [0, -35]
});

//Add a click event to the marker to open the popup.
map.events.add('click', marker, function () {
    //Open the popup
    popup.open(map);
});
```
![Fenêtre contextuelle Azure Maps](media/migrate-google-maps-web-app/azure-maps-popup.png)

> [!NOTE]
> Vous pouvez effectuer la même opération avec un calque de symbole ou de bulles, ou avec une couche de lignes ou de polygones, en transmettant le calque ou la couche choisi(e) au code d’événement de la carte plutôt qu’à un marqueur.

**Ressources supplémentaires :**

- [Ajouter une fenêtre contextuelle](map-add-popup.md)
- [Fenêtre contextuelle avec contenu multimédia](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Popup%20with%20Media%20Content)
- [Fenêtres contextuelles sur les formes](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Popups%20on%20Shapes)
- [Réutilisation d’une fenêtre contextuelle avec plusieurs épingles](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Reusing%20Popup%20with%20Multiple%20Pins)
- [Classe de fenêtre contextuelle](/javascript/api/azure-maps-control/atlas.popup)
- [Options de la fenêtre contextuelle](/javascript/api/azure-maps-control/atlas.popupoptions)

### <a name="import-a-geojson-file"></a>Importer un fichier GeoJSON

Google Maps prend en charge le chargement et le stylisation dynamique des données GeoJSON via la classe `google.maps.Data`. La fonctionnalité de cette classe s'aligne beaucoup plus sur le style basé sur les données d'Azure Maps. Toutefois, il existe une différence essentielle. Avec Google Maps, vous spécifiez une fonction de rappel et la logique métier pour appliquer un style à chaque fonctionnalité traitée individuellement dans le thread d’interface utilisateur. En revanche, dans Azure Maps, les calques prennent en charge la spécification d’expressions pilotées par les données sous forme d’options de style. Ces expressions sont traitées au moment du rendu sur un thread distinct. L’approche Azure Maps améliore les performances de rendu. Cet avantage est observé quand des jeux de données plus volumineux doivent être restitués rapidement.

Les exemples suivants chargent un flux GeoJSON de tous les séismes au cours des sept derniers jours à partir des groupes universels de sécurité (USG). Les données de séismes sont affichées sous forme de cercles mis à l’échelle sur la carte. La couleur et l’échelle de chaque cercle sont basées sur l’ampleur de chaque tremblement de terre, qui est stockée dans la propriété `"mag"` de chaque fonctionnalité du jeu de données. Si la magnitude est supérieure ou égale à 5, le cercle est rouge. S’il est supérieur ou égal à 3 mais inférieur à 5, le cercle est orange. S’il est inférieur à 3, le cercle est vert. Le rayon de chaque cercle est l’exponentiel de la grandeur multiplié par 0,1.

#### <a name="before-google-maps"></a>Avant : Google Maps

Spécifiez une seule fonction de rappel dans la méthode `map.data.setStyle`. À l’intérieur de la fonction de rappel, appliquez la logique métier à chaque fonctionnalité. Chargez le flux GeoJSON avec la méthode `map.data.loadGeoJson`.

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <script type='text/javascript'>
        var map;
        var earthquakeFeed = 'https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_week.geojson';

        function initMap() {
            map = new google.maps.Map(document.getElementById('myMap'), {
                center: new google.maps.LatLng(20, -160),
                zoom: 2
            });

            //Define a callback to style each feature.
            map.data.setStyle(function (feature) {

                //Extract the 'mag' property from the feature.
                var mag = parseFloat(feature.getProperty('mag'));

                //Set the color value to 'green' by default.
                var color = 'green';

                //If the mag value is greater than 5, set the color to 'red'.
                if (mag >= 5) {
                    color = 'red';
                }
                //If the mag value is greater than 3, set the color to 'orange'.
                else if (mag >= 3) {
                    color = 'orange';
                }

                return /** @type {google.maps.Data.StyleOptions} */({
                    icon: {
                        path: google.maps.SymbolPath.CIRCLE,

                        //Scale the radius based on an exponential of the 'mag' value.
                        scale: Math.exp(mag) * 0.1,
                        fillColor: color,
                        fillOpacity: 0.75,
                        strokeWeight: 2,
                        strokeColor: 'white'
                    }
                });
            });

            //Load the data feed.
            map.data.loadGeoJson(earthquakeFeed);
        }
    </script>

    <!-- Google Maps Script Reference -->
    <script src="https://maps.googleapis.com/maps/api/js?callback=initMap&key=[Your Google Maps Key]" async defer></script>
</head>
<body>
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```

![GeoJSON de Google Maps](media/migrate-google-maps-web-app/google-maps-geojson.png)

#### <a name="after-azure-maps"></a>Après : Azure Maps

GeoJSON est le type de données de base dans Azure Maps. Importez-le dans une source de données à l’aide de la méthode `datasource.importFromUrl`. Utilisez un calque de bulles. Le calque de bulles fournit des fonctionnalités pour le rendu des cercles mis à l’échelle en fonction des propriétés des fonctionnalités d’une source de données. Au lieu d’avoir une fonction de rappel, la logique métier est convertie en expression et transmise dans les options de style. Les expressions définissent le fonctionnement de la logique métier. Elles peuvent être passées dans un autre thread et évaluées par rapport aux données des fonctionnalités. Plusieurs sources de données et calques peuvent être ajoutés à Azure Maps, chacun avec une logique métier différente. Cette fonctionnalité permet de restituer plusieurs jeux de données sur la carte de différentes façons.

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
    <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
    <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>

    <script type='text/javascript'>
        var map;
        var earthquakeFeed = 'https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_week.geojson';

        function initMap() {
            //Initialize a map instance.
            map = new atlas.Map('myMap', {
                center: [-160, 20],
                zoom: 1,

                //Add your Azure Maps subscription key to the map SDK. Get an Azure Maps key at https://azure.com/maps
                authOptions: {
                    authType: 'subscriptionKey',
                    subscriptionKey: '<Your Azure Maps Key>'
                }
            });

            //Wait until the map resources are ready.
            map.events.add('ready', function () {

                //Create a data source and add it to the map.
                datasource = new atlas.source.DataSource();
                map.sources.add(datasource);

                //Load the earthquake data.
                datasource.importDataFromUrl(earthquakeFeed);

                //Create a layer to render the data points as scaled circles.
                map.layers.add(new atlas.layer.BubbleLayer(datasource, null, {
                    //Make the circles semi-transparent.
                    opacity: 0.75,

                    color: [
                        'case',

                        //If the mag value is greater than 5, return 'red'.
                        ['>=', ['get', 'mag'], 5],
                        'red',

                        //If the mag value is greater than 3, return 'orange'.
                        ['>=', ['get', 'mag'], 3],
                        'orange',

                        //Return 'green' as a fallback.
                        'green'
                    ],

                    //Scale the radius based on an exponential of the 'mag' value.
                    radius: ['*', ['^', ['e'], ['get', 'mag']], 0.1]
                }));
            });
        }
    </script>
</head>
<body onload="initMap()">
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```



![GeoJSON d’Azure Maps](media/migrate-google-maps-web-app/azure-maps-geojson.png)

**Ressources supplémentaires :**

- [Ajouter une calque de symbole](map-add-pin.md)
- [Ajouter un calque de bulles](map-add-bubble-layer.md)
- [Données de point de cluster](clustering-point-data-web-sdk.md)
- [Utiliser des expressions de style basées sur les données](data-driven-style-expressions-web-sdk.md)

### <a name="marker-clustering"></a>Clustering de marqueurs

Lors de la visualisation de nombreux points de données sur la carte, les points peuvent se chevaucher. Le chevauchement rend la carte plus encombrée, et elle devient difficile à lire et à utiliser. Le clustering de point de données est le processus permettant de combiner des points de données proches les uns des autres et de les représenter sur une carte sous forme de point de données en cluster unique. Lorsque l’utilisateur effectue un zoom avant sur la carte, les clusters se décomposent pour afficher les points de données individuels qui les composent. Affichez les points de données en cluster pour améliorer l’expérience utilisateur et les performances cartographiques.

Dans les exemples suivants, le code charge un flux GeoJSON de données relatives aux séismes de la semaine dernière et l’ajoute à la carte. Les clusters sont rendus sous forme de cercles mis à l’échelle et colorés. L’échelle et la couleur des cercles dépendent du nombre de points qu’ils contiennent.

> [!NOTE]
> Google Maps et Azure Maps utilisent des algorithmes de clustering légèrement différents. Par conséquent, la distribution des points dans les clusters varie parfois.

#### <a name="before-google-maps"></a>Avant : Google Maps

Utilisez la bibliothèque MarkerCluster pour les marqueurs de cluster. Les icônes de cluster sont limitées aux images, qui ont les numéros 1 à 5 comme nom. Elles sont hébergées dans le même répertoire.

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <script type='text/javascript'>
        var map;
        var earthquakeFeed = 'https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_week.geojson';

        function initMap() {
            map = new google.maps.Map(document.getElementById('myMap'), {
                center: new google.maps.LatLng(20, -160),
                zoom: 2
            });

            //Download the GeoJSON data.
            fetch(earthquakeFeed)
                .then(function (response) {
                    return response.json();
                }).then(function (data) {
                    //Loop through the GeoJSON data and create a marker for each data point.
                    var markers = [];

                    for (var i = 0; i < data.features.length; i++) {

                        markers.push(new google.maps.Marker({
                            position: new google.maps.LatLng(data.features[i].geometry.coordinates[1], data.features[i].geometry.coordinates[0])
                        }));
                    }

                    //Create a marker clusterer instance and tell it where to find the cluster icons.
                    var markerCluster = new MarkerClusterer(map, markers,
                        { imagePath: 'https://developers.google.com/maps/documentation/javascript/examples/markerclusterer/m' });
                });
        }
    </script>

    <!-- Load the marker cluster library. -->
    <script src="https://developers.google.com/maps/documentation/javascript/examples/markerclusterer/markerclusterer.js"></script>

    <!-- Google Maps Script Reference -->
    <script src="https://maps.googleapis.com/maps/api/js?callback=initMap&key=[Your Google Maps Key]" async defer></script>
</head>
<body>
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```



![Clusters Google Maps](media/migrate-google-maps-web-app/google-maps-clustering.png)

#### <a name="after-azure-maps"></a>Après : Azure Maps

Ajoutez et gérez des données dans une source de données. Connectez les sources de données et les calques, puis affichez les données. La classe `DataSource` dans Azure Maps fournit plusieurs options de clustering.

- `cluster` : Indique à la source de données les données de point de cluster.
- `clusterRadius` : Rayon, en pixels, des points de cluster.
- `clusterMaxZoom` : Niveau de zoom maximal dans lequel le clustering se produit. Si vous effectuez un zoom avant plus grand que ce niveau, tous les points sont restitués sous forme de symboles.
- `clusterProperties` : Définit des propriétés personnalisées qui sont calculées à l’aide d’expressions sur tous les points de chaque cluster et ajoutées aux propriétés de chaque point de cluster.

Lorsque le clustering est activé, la source de données envoie des points de données en cluster et non cluster aux calques pour le rendu. La source de données est en charge de la mise en cluster de centaines de milliers de points de données. Un point de données en cluster a les propriétés suivantes :

| Nom de la propriété             | Type    | Description   |
|---------------------------|---------|---------------|
| `cluster`                 | boolean | Indique si la fonctionnalité représente un cluster. |
| `cluster_id`              | string  | Un ID unique pour le cluster qui peut être utilisé avec les méthodes `getClusterExpansionZoom`, `getClusterChildren` et `getClusterLeaves` de DataSource. |
| `point_count`             | nombre  | Le nombre de points que contient le cluster.  |
| `point_count_abbreviated` | string  | Une chaîne qui abrège la valeur `point_count` si elle est trop longue. (par exemple, 4 000 devient 4K)  |

La classe `DataSource` possède la fonction d’assistance suivante pour accéder à des informations supplémentaires sur un cluster à l’aide de `cluster_id`.

| Méthode | Type de retour | Description |
|--------|-------------|-------------|
| `getClusterChildren(clusterId: number)` | Promise&lt;Array&lt;Feature&lt;Geometry, any&gt;\| Shape&gt;&gt; | Récupère les enfants du cluster donné sur le niveau de zoom suivant. Ces enfants peuvent être une combinaison de formes et sous-clusters. Les sous-clusters seront des fonctionnalités dont les propriétés correspondent à ClusteredProperties. |
| `getClusterExpansionZoom(clusterId: number)` | Promise&lt;number&gt; | Calcule un niveau de zoom à partir duquel le cluster commence à se développer ou à se décomposer. |
| `getClusterLeaves(clusterId: number, limit: number, offset: number)` | Promise&lt;Array&lt;Feature&lt;Geometry, any&gt;\| Shape&gt;&gt; | Récupère tous les points dans un cluster. Définissez le paramètre `limit` de manière à renvoyer un sous-ensemble des points et utilisez `offset` pour parcourir les points. |

Lors du rendu des données en cluster sur la carte, il est souvent plus préférable d’utiliser au moins deux calques. L’exemple suivant utilise trois calques : un calque de bulles pour tracer des cercles de couleur à l’échelle en fonction de la taille des clusters, un calque de symboles pour afficher la taille de cluster sous forme de texte et un deuxième calque de symboles pour le rendu des points non-cluster. Il existe de nombreuses autres façons de restituer des données en cluster. Pour plus d’informations, consultez la documentation [Données de point de cluster](clustering-point-data-web-sdk.md).

Importez directement des données GeoJSON à l’aide de la fonction `importDataFromUrl` sur la classe `DataSource`, à l’intérieur d’une carte Azure Maps.

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
    <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
    <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>

    <script type='text/javascript'>
        var map, datasource;
        var earthquakeFeed = 'https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_week.geojson';

        function initMap() {
            //Initialize a map instance.
            map = new atlas.Map('myMap', {
                center: [-160, 20],
                zoom: 1,

                //Add your Azure Maps subscription key to the map SDK. Get an Azure Maps key at https://azure.com/maps
                authOptions: {
                    authType: 'subscriptionKey',
                    subscriptionKey: '<Your Azure Maps Key>'
                }
            });

            //Wait until the map resources are ready.
            map.events.add('ready', function () {

                //Create a data source and add it to the map.
                datasource = new atlas.source.DataSource(null, {
                    //Tell the data source to cluster point data.
                    cluster: true
                });
                map.sources.add(datasource);

                //Create layers for rendering clusters, their counts and unclustered points and add the layers to the map.
                map.layers.add([
                    //Create a bubble layer for rendering clustered data points.
                    new atlas.layer.BubbleLayer(datasource, null, {
                        //Scale the size of the clustered bubble based on the number of points inthe cluster.
                        radius: [
                            'step',
                            ['get', 'point_count'],
                            20,         //Default of 20 pixel radius.
                            100, 30,    //If point_count >= 100, radius is 30 pixels.
                            750, 40     //If point_count >= 750, radius is 40 pixels.
                        ],

                        //Change the color of the cluster based on the value on the point_cluster property of the cluster.
                        color: [
                            'step',
                            ['get', 'point_count'],
                            'lime',            //Default to lime green. 
                            100, 'yellow',     //If the point_count >= 100, color is yellow.
                            750, 'red'         //If the point_count >= 750, color is red.
                        ],
                        strokeWidth: 0,
                        filter: ['has', 'point_count'] //Only rendered data points which have a point_count property, which clusters do.
                    }),

                    //Create a symbol layer to render the count of locations in a cluster.
                    new atlas.layer.SymbolLayer(datasource, null, {
                        iconOptions: {
                            image: 'none' //Hide the icon image.
                        },
                        textOptions: {
                            textField: ['get', 'point_count_abbreviated'],
                            offset: [0, 0.4]
                        }
                    }),

                    //Create a layer to render the individual locations.
                    new atlas.layer.SymbolLayer(datasource, null, {
                        filter: ['!', ['has', 'point_count']] //Filter out clustered points from this layer.
                    })
                ]);

                //Retrieve a GeoJSON data set and add it to the data source. 
                datasource.importDataFromUrl(earthquakeFeed);
            });
        }
    </script>
</head>
<body onload="initMap()">
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```



![Clusters Azure Maps](media/migrate-google-maps-web-app/azure-maps-clustering.png)

**Ressources supplémentaires :**

- [Ajouter une calque de symbole](map-add-pin.md)
- [Ajouter un calque de bulles](map-add-bubble-layer.md)
- [Données de point de cluster](clustering-point-data-web-sdk.md)
- [Utiliser des expressions de style basées sur les données](data-driven-style-expressions-web-sdk.md)

### <a name="add-a-heat-map"></a>Ajouter une carte thermique

Les cartes thermiques, également appelées « cartes de densité de points », sont un type de visualisation de données. Elles servent à représenter la densité de données à l’aide d’une palette de couleurs. De plus, elles sont souvent utilisées pour afficher les « points chauds » des données sur une carte. Les cartes thermiques sont idéales pour restituer des jeux de données de points volumineux.

Les exemples suivants chargent un flux GeoJSON de tous les séismes détectés par l’Institut d’études géologiques des États-Unis (USGS) au cours du mois précédent, et les affichent sous la forme d’une carte thermique pondérée. La propriété `"mag"` est utilisée comme poids.

#### <a name="before-google-maps"></a>Avant : Google Maps

Pour créer une carte thermique, chargez la bibliothèque « visualization » en ajoutant `&libraries=visualization` à l’URL du script API. La couche de carte thermique dans Google Maps ne prend pas directement en charge les données GeoJSON. Tout d’abord, téléchargez les données et convertissez-les en un tableau de points de données pondérés :

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <script type='text/javascript'>
        var map;
        var earthquakeFeed = 'https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_month.geojson';

        function initMap() {

            var map = new google.maps.Map(document.getElementById('myMap'), {
                center: new google.maps.LatLng(20, -160),
                zoom: 2,
                mapTypeId: 'satellite'
            });

            //Download the GeoJSON data.
            fetch(url).then(function (response) {
                return response.json();
            }).then(function (res) {
                var points = getDataPoints(res);

                var heatmap = new google.maps.visualization.HeatmapLayer({
                    data: points
                });
                heatmap.setMap(map);
            });
        }

        function getDataPoints(geojson, weightProp) {
            var points = [];

            for (var i = 0; i < geojson.features.length; i++) {
                var f = geojson.features[i];

                if (f.geometry.type === 'Point') {
                    points.push({
                        location: new google.maps.LatLng(f.geometry.coordinates[1], f.geometry.coordinates[0]),
                        weight: f.properties[weightProp]
                    });
                }
            }

            return points;
        } 
    </script>

    <!-- Google Maps Script Reference -->
    <script src="https://maps.googleapis.com/maps/api/js?callback=initMap&key=[Your Google Maps Key]&libraries=visualization" async defer></script>
</head>
<body>
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```



![Carte thermique Google Maps](media/migrate-google-maps-web-app/google-maps-heatmap.png)

#### <a name="after-azure-maps"></a>Après : Azure Maps

Chargez les données GeoJSON dans une source de données et connectez la source de données à une couche de carte thermique. La propriété qui sera utilisée pour le poids peut être passée dans l’option `weight` à l’aide d’une expression. Importez directement les données GeoJSON dans Azure Maps à l’aide de la fonction `importDataFromUrl` sur la classe `DataSource`.

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
    <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
    <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>

    <script type='text/javascript'>
        var map;
        var earthquakeFeed = 'https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_month.geojson';

        function initMap() {
            //Initialize a map instance.
            map = new atlas.Map('myMap', {
                center: [-160, 20],
                zoom: 1,
                style: 'satellite',

                //Add your Azure Maps subscription key to the map SDK. Get an Azure Maps key at https://azure.com/maps
                authOptions: {
                    authType: 'subscriptionKey',
                    subscriptionKey: '<Your Azure Maps Key>'
                }
            });

            //Wait until the map resources are ready.
            map.events.add('ready', function () {

                //Create a data source and add it to the map.
                datasource = new atlas.source.DataSource();
                map.sources.add(datasource);

                //Load the earthquake data.
                datasource.importDataFromUrl(earthquakeFeed);

                //Create a layer to render the data points as a heat map.
                map.layers.add(new atlas.layer.HeatMapLayer(datasource, null, {
                    weight: ['get', 'mag'],
                    intensity: 0.005,
                    opacity: 0.65,
                    radius: 10
                }));
            });
        }
    </script>
</head>
<body onload="initMap()">
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```



![Carte thermique Azure Maps](media/migrate-google-maps-web-app/azure-maps-heatmap.png)

**Ressources supplémentaires :**

- [Ajouter une couche de carte thermique](map-add-heat-map-layer.md)
- [Classe de couche de carte thermique](/javascript/api/azure-maps-control/atlas.layer.heatmaplayer)
- [Options de la couche de carte thermique](/javascript/api/azure-maps-control/atlas.heatmaplayeroptions)
- [Utiliser des expressions de style basées sur les données](data-driven-style-expressions-web-sdk.md)

### <a name="overlay-a-tile-layer"></a>Superposer une couche de mosaïques

Les couches de mosaïques dans Azure Maps sont appelés « superpositions d’images » dans Google Maps. Les couches de mosaïques vous permettent de superposer de grandes images qui ont été divisées en images en mosaïque plus petites qui s’alignent sur le système de mosaïque de cartes. Cette approche est couramment utilisée pour superposer des images de grande taille ou des jeux de données volumineux.

Les exemples suivants superposent une couche de mosaïques radar météo issue de l’Iowa Environmental Mesonet de l’Iowa State University.

#### <a name="before-google-maps"></a>Avant : Google Maps

Dans Google Maps, les couches de mosaïques peuvent être créées à l’aide de la classe `google.maps.ImageMapType`.

```javascript
map.overlayMapTypes.insertAt(0, new google.maps.ImageMapType({
    getTileUrl: function (coord, zoom) {
        return "https://mesonet.agron.iastate.edu/cache/tile.py/1.0.0/nexrad-n0q-900913/" + zoom + "/" + coord.x + "/" + coord.y;
    },
    tileSize: new google.maps.Size(256, 256),
    opacity: 0.8
}));
```



![Couche de mosaïques Google Maps](media/migrate-google-maps-web-app/google-maps-tile-layer.png)

#### <a name="after-azure-maps"></a>Après : Azure Maps

Ajoutez une couche de mosaïques à la carte de la même façon que n’importe quelle autre couche. Utilisez une URL mise en forme qui a des espaces réservés de zoom x, y ; `{x}`, `{y}`, `{z}` pour indiquer à la couche où accéder aux mosaïques. Les couches de mosaïques Azure Maps prennent également en charge les espaces réservés `{quadkey}`, `{bbox-epsg-3857}` et `{subdomain}`.

> [!TIP]
> Dans Azure Maps, les couches peuvent facilement être rendues sous d’autres couches, y compris les couches de la carte de base. Il est souvent souhaitable de restituer des couches de mosaïques sous les étiquettes de carte afin qu’elles soient faciles à lire. La méthode `map.layers.add` prend un deuxième paramètre, qui est l’ID de la couche sous laquelle insérer la nouvelle couche. Pour insérer une couche de mosaïques sous les étiquettes de carte, utilisez ce code : `map.layers.add(myTileLayer, "labels");`

```javascript
//Create a tile layer and add it to the map below the label layer.
map.layers.add(new atlas.layer.TileLayer({
    tileUrl: 'https://mesonet.agron.iastate.edu/cache/tile.py/1.0.0/nexrad-n0q-900913/{z}/{x}/{y}.png',
    opacity: 0.8,
    tileSize: 256
}), 'labels');
```



![Couche de mosaïques Azure Maps](media/migrate-google-maps-web-app/azure-maps-tile-layer.png)

> [!TIP]
> Les demandes de mosaïque peuvent être capturées à l’aide de l’option `transformRequest` de la carte. Cela vous permettra de modifier ou d’ajouter des en-têtes à la demande, si vous le souhaitez.

**Ressources supplémentaires :**

- [Ajouter des couches de vignettes](map-add-tile-layer.md)
- [Classe de couche de mosaïque](/javascript/api/azure-maps-control/atlas.layer.tilelayer)
- [Options de la couche de mosaïques](/javascript/api/azure-maps-control/atlas.tilelayeroptions)

### <a name="show-traffic-data"></a>Afficher les données du trafic

Les données de trafic peuvent être superposées à la fois dans Azure et Google Maps.

#### <a name="before-google-maps"></a>Avant : Google Maps

Superposez les données de trafic sur la carte à l’aide de la couche de trafic.

```javascript
var trafficLayer = new google.maps.TrafficLayer();
trafficLayer.setMap(map);
```



![Trafic Google Maps](media/migrate-google-maps-web-app/google-maps-traffic.png)

#### <a name="after-azure-maps"></a>Après : Azure Maps

Azure Maps offre plusieurs options différentes pour l’affichage du trafic. Affichez les incidents de trafic, tels que les fermetures de route et les accidents, sous forme d’icônes sur la carte. Superposez le flux de trafic et les routes codées en couleur sur la carte. Les couleurs peuvent être modifiées en fonction de la limite de vitesse indiquée ou pour tenir compte d’un retard prévu normal ou d’un retard absolu. Les données d’incident dans Azure Maps sont mises à jour toutes les minutes et les données de circulation le sont toutes les deux minutes.

Assignez les valeurs souhaitées pour les options `setTraffic`.

```javascript
map.setTraffic({
    incidents: true,
    flow: 'relative'
});
```



![Trafic Azure Maps](media/migrate-google-maps-web-app/azure-maps-traffic.png)

Si vous cliquez sur l’une des icônes de trafic dans Azure Maps, des informations supplémentaires s’affichent dans une fenêtre contextuelle.



![Incident de trafic Azure Maps](media/migrate-google-maps-web-app/azure-maps-traffic-incident.png)

**Ressources supplémentaires :**

- [Afficher le trafic sur la carte](map-show-traffic.md)
- [Options de superposition du trafic](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Traffic%20Overlay%20Options)

### <a name="add-a-ground-overlay"></a>Ajouter une superposition de sol

Azure et Google Maps prennent en charge la superposition d’images géoréférencées sur la carte. Les images géoréférencées se déplacent et sont mises à l’échelle à mesure que vous effectuez un panoramique et un zoom sur la carte. Dans Google Maps, ces images géoréférencées sont appelées « calques de relief », alors qu’elles sont appelées « calques d’images » dans Azure Maps. Elles sont très utiles pour créer des plans d’étage, pour superposer d’anciennes cartes, ou pour les images d’un drone.

#### <a name="before-google-maps"></a>Avant : Google Maps

Spécifiez l’URL de l’image que vous souhaitez superposer, et un cadre englobant pour lier l’image sur la carte. Cet exemple superpose une image de carte de [Newark New Jersey en 1922](https://www.lib.utexas.edu/maps/historical/newark_nj_1922.jpg) sur la carte.

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <script type='text/javascript'>
        var map, historicalOverlay;

        function initMap() {
            map = new google.maps.Map(document.getElementById('myMap'), {
                center: new google.maps.LatLng(40.740, -74.18),
                zoom: 12
            });

            var imageBounds = {
                north: 40.773941,
                south: 40.712216,
                east: -74.12544,
                west: -74.22655
            };

            historicalOverlay = new google.maps.GroundOverlay(
                'https://www.lib.utexas.edu/maps/historical/newark_nj_1922.jpg',
                imageBounds);
            historicalOverlay.setMap(map);
        }
    </script>

    <!-- Google Maps Script Reference -->
    <script src="https://maps.googleapis.com/maps/api/js?callback=initMap&key=[Your Google Maps Key]" async defer></script>
</head>
<body>
    <div id="myMap" style="position:relative;width:600px;height:400px;"></div>
</body>
</html>
```

L’exécution de ce code dans un navigateur affichera une carte ressemblant à l’image suivante :

![Superposition d’images Google Maps](media/migrate-google-maps-web-app/google-maps-image-overlay.png)

#### <a name="after-azure-maps"></a>Après : Azure Maps

Utilisez la classe `atlas.layer.ImageLayer` pour superposer des images géoréférencées. Cette classe requiert une URL vers une image et un ensemble de coordonnées pour les quatre coins de l’image. L’image doit être hébergée soit sur le même domaine, soit avec CORs activé.

> [!TIP]
> Si vous avez uniquement des informations relatives au nord, au sud, à l’est, à l’ouest et à la rotation, et que vous n’avez pas de coordonnées pour chaque angle de l’image, vous pouvez utiliser la méthode statique [`atlas.layer.ImageLayer.getCoordinatesFromEdges`](/javascript/api/azure-maps-control/atlas.layer.imagelayer#getcoordinatesfromedges-number--number--number--number--number-).

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
    <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
    <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>

    <script type='text/javascript'>
        var map;

        function initMap() {
            //Initialize a map instance.
            map = new atlas.Map('myMap', {
                center: [-74.18, 40.740],
                zoom: 12,

                //Add your Azure Maps subscription key to the map SDK. Get an Azure Maps key at https://azure.com/maps
                authOptions: {
                    authType: 'subscriptionKey',
                    subscriptionKey: '<Your Azure Maps Key>'
                }
            });

            //Wait until the map resources are ready.
            map.events.add('ready', function () {

                //Create an image layer and add it to the map.
                map.layers.add(new atlas.layer.ImageLayer({
                    url: 'newark_nj_1922.jpg',
                    coordinates: [
                        [-74.22655, 40.773941], //Top Left Corner
                        [-74.12544, 40.773941], //Top Right Corner
                        [-74.12544, 40.712216], //Bottom Right Corner
                        [-74.22655, 40.712216]  //Bottom Left Corner
                    ]
                }));
            });
        }
    </script>
</head>
<body onload="initMap()">
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```



![Superposition d’images Azure Maps](media/migrate-google-maps-web-app/azure-maps-image-overlay.png)

**Ressources supplémentaires :**

- [Superposer une image](map-add-image-layer.md)
- [Classe de couche image](/javascript/api/azure-maps-control/atlas.layer.imagelayer)

### <a name="add-kml-data-to-the-map"></a>Ajouter des données KML à la carte

Azure et Google Maps peuvent importer des données KML, KMZ et GeoRSS et les restituer sur la carte. Azure Maps prend également en charge les fichiers GPX, GML et fichiers CSV de données spatiales, les formats GeoJSON et WKT (Well Known Text) et les services WMS (Web-Mapping Service), WMTS (Web-Mapping Tile Service) et WFS (Web Feature Service). Azure Maps lit les fichiers localement en mémoire et peut, dans la plupart des cas, gérer des fichiers KML beaucoup plus gros. 

#### <a name="before-google-maps"></a>Avant : Google Maps


```javascript
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <script type='text/javascript'>
        var map, historicalOverlay;

        function initMap() {
            map = new google.maps.Map(document.getElementById('myMap'), {
                center: new google.maps.LatLng(0, 0),
                zoom: 1
            });

             var layer = new google.maps.KmlLayer({
              url: 'https://googlearchive.github.io/js-v2-samples/ggeoxml/cta.kml',
              map: map
            });
        }
    </script>

    <!-- Google Maps Script Reference -->
    <script src="https://maps.googleapis.com/maps/api/js?callback=initMap&key=[Your Google Maps Key]" async defer></script>
</head>
<body>
    <div id="myMap" style="position:relative;width:600px;height:400px;"></div>
</body>
</html>
```

L’exécution de ce code dans un navigateur affichera une carte ressemblant à l’image suivante :

![Google Maps - KML](media/migrate-google-maps-web-app/google-maps-kml.png)

#### <a name="after-azure-maps"></a>Après : Azure Maps

Dans Azure Maps, le format GeoJSON est le format de données principal utilisé dans le SDK web. Vous pouvez facilement intégrer d’autres formats de données spatiales à l’aide du [module d’E/S de données spatiales](/javascript/api/azure-maps-spatial-io/). Ce module présente des fonctions de lecture et d’écriture de données spatiales et inclut un calque de données simples restituant facilement les données à partir de l’un de ces formats de données spatiales. Pour lire les données d’un fichier de données spatiales, transmettez une URL ou des données brutes sous la forme d’une chaîne ou d’un objet blob dans la fonction `atlas.io.read`. Ceci retournera toutes les données analysées du fichier, qui pourront alors être ajoutées à la carte. Le format KML est un peu plus complexe que la plupart des autres formats de données spatiales. En effet, il inclut beaucoup plus d’informations de style. La classe `SpatialDataLayer` prend en charge le rendu d’une majorité de ces styles. Cependant, les images d’icônes doivent être chargées dans la carte avant les données de caractéristiques, et les calques de relief doivent être ajoutés séparément à la carte sous forme de couches. Quand vous chargez des données par le biais d’une URL, elles doivent être hébergées sur un point de terminaison CORS, ou un service proxy doit être transmis comme option dans la fonction read. 

```javascript
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
    <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
    <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>

    <!-- Add reference to the Azure Maps Spatial IO module. -->
    <script src="https://atlas.microsoft.com/sdk/javascript/spatial/0/atlas-spatial.js"></script>

    <script type='text/javascript'>
        var map, datasource, layer;

        function initMap() {
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
            map.events.add('ready', function () {
            
                //Create a data source and add it to the map.
                datasource = new atlas.source.DataSource();
                map.sources.add(datasource);

                //Add a simple data layer for rendering the data.
                layer = new atlas.layer.SimpleDataLayer(datasource);
                map.layers.add(layer);

                //Read a KML file from a URL or pass in a raw KML string.
                atlas.io.read('https://googlearchive.github.io/js-v2-samples/ggeoxml/cta.kml').then(async r => {
                    if (r) {

                        //Check to see if there are any icons in the data set that need to be loaded into the map resources.
                        if (r.icons) {
                            //For each icon image, create a promise to add it to the map, then run the promises in parrallel.
                            var imagePromises = [];

                            //The keys are the names of each icon image.
                            var keys = Object.keys(r.icons);

                            if (keys.length !== 0) {
                                keys.forEach(function (key) {
                                    imagePromises.push(map.imageSprite.add(key, r.icons[key]));
                                });

                                await Promise.all(imagePromises);
                            }
                        }

                        //Load all features.
                        if (r.features && r.features.length > 0) {
                            datasource.add(r.features);
                        }

                        //Load all ground overlays.
                        if (r.groundOverlays && r.groundOverlays.length > 0) {
                            map.layers.add(r.groundOverlays);
                        }

                        //If bounding box information is known for data, set the map view to it.
                        if (r.bbox) {
                            map.setCamera({ bounds: r.bbox, padding: 50 });
                        }
                    }
                });
            });
        }
    </script>
</head>
<body onload="initMap()">
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```


![Azure Maps - KML](media/migrate-google-maps-web-app/azure-maps-kml.png)</center>


**Ressources supplémentaires :**

- [Fonction atlas.io.read](/javascript/api/azure-maps-spatial-io/atlas.io#read-string---arraybuffer---blob--spatialdatareadoptions-)
- [SimpleDataLayer](/javascript/api/azure-maps-spatial-io/atlas.layer.simpledatalayer)
- [SimpleDataLayerOptions](/javascript/api/azure-maps-spatial-io/atlas.simpledatalayeroptions)

## <a name="additional-code-samples"></a>Exemples de code supplémentaires

Voici quelques exemples de code supplémentaires liés à la migration de Google Maps :

- [Outils de dessin](map-add-drawing-toolbar.md)
- [Limiter le mappage au panoramique à deux doigts](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Limit%20Map%20to%20Two%20Finger%20Panning)
- [Limiter le zoom de la roulette de défilement](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Limit%20Scroll%20Wheel%20Zoom)
- [Créer un contrôle fullscreen](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Create%20a%20Fullscreen%20Control)

**Services :**

- [Utiliser le module de service Azure Maps](how-to-use-services-module.md)
- [Rechercher des points d’intérêt](map-search-location.md)
- [Obtenir des informations à partir d’une coordonnée (géocode inversé)](map-get-information-from-coordinate.md)
- [Afficher des directions de A à B](map-route.md)
- [Rechercher la suggestion automatique avec JQuery UI](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Search%20Autosuggest%20and%20JQuery%20UI)

## <a name="google-maps-v3-to-azure-maps-web-sdk-class-mapping"></a>Mappage de la classe SDK de Google Maps v3 vers Azure Maps

L’annexe suivante fournit une référence croisée des classes couramment utilisées dans Google Maps v3 et l’équivalent dans le SDK web Azure Maps.

### <a name="core-classes"></a>Classes principales

| Google Maps   | Azure Maps  |
|---------------|-------------|
| `google.maps.Map` | [atlas.Map](/javascript/api/azure-maps-control/atlas.map)  |
| `google.maps.InfoWindow` | [atlas.Popup](/javascript/api/azure-maps-control/atlas.popup)  |
| `google.maps.InfoWindowOptions` | [atlas.PopupOptions](https://docs.microsoft.com/) |
| `google.maps.LatLng`  | [atlas.data.Position](/javascript/api/azure-maps-control/atlas.data.position)  |
| `google.maps.LatLngBounds` | [atlas.data.BoundingBox](/javascript/api/azure-maps-control/atlas.data.boundingbox) |
| `google.maps.MapOptions`  | [atlas.CameraOptions](/javascript/api/azure-maps-control/atlas.cameraoptions)<br/>[atlas.CameraBoundsOptions](/javascript/api/azure-maps-control/atlas.cameraboundsoptions)<br/>[atlas.ServiceOptions](/javascript/api/azure-maps-control/atlas.serviceoptions)<br/>[atlas.StyleOptions](/javascript/api/azure-maps-control/atlas.styleoptions)<br/>[atlas.UserInteractionOptions](/javascript/api/azure-maps-control/atlas.userinteractionoptions) |
| `google.maps.Point`  | [atlas.Pixel](/javascript/api/azure-maps-control/atlas.pixel)   |

## <a name="overlay-classes"></a>Classes de superposition

| Google Maps  | Azure Maps  |
|--------------|-------------|
| `google.maps.Marker` | [atlas.HtmlMarker](/javascript/api/azure-maps-control/atlas.htmlmarker)<br/>[atlas.data.Point](/javascript/api/azure-maps-control/atlas.data.point)  |
| `google.maps.MarkerOptions`  | [atlas.HtmlMarkerOptions](/javascript/api/azure-maps-control/atlas.htmlmarkeroptions)<br/>[atlas.layer.SymbolLayer](/javascript/api/azure-maps-control/atlas.layer.symbollayer)<br/>[atlas.SymbolLayerOptions](/javascript/api/azure-maps-control/atlas.symbollayeroptions)<br/>[atlas.IconOptions](/javascript/api/azure-maps-control/atlas.iconoptions)<br/>[atlas.TextOptions](/javascript/api/azure-maps-control/atlas.textoptions)<br/>[atlas.layer.BubbleLayer](/javascript/api/azure-maps-control/atlas.layer.bubblelayer)<br/>[atlas.BubbleLayerOptions](/javascript/api/azure-maps-control/atlas.bubblelayeroptions) |
| `google.maps.Polygon`  | [atlas.data.Polygon](/javascript/api/azure-maps-control/atlas.data.polygon)               |
| `google.maps.PolygonOptions` |[atlas.layer.PolygonLayer](/javascript/api/azure-maps-control/atlas.layer.polygonlayer)<br/> [atlas.PolygonLayerOptions](/javascript/api/azure-maps-control/atlas.polygonlayeroptions)<br/> [atlas.layer.LineLayer](/javascript/api/azure-maps-control/atlas.layer.linelayer)<br/> [atlas.LineLayerOptions](/javascript/api/azure-maps-control/atlas.linelayeroptions)|
| `google.maps.Polyline` | [atlas.data.LineString](/javascript/api/azure-maps-control/atlas.data.linestring)         |
| `google.maps.PolylineOptions` | [atlas.layer.LineLayer](/javascript/api/azure-maps-control/atlas.layer.linelayer)<br/>[atlas.LineLayerOptions](/javascript/api/azure-maps-control/atlas.linelayeroptions) |
| `google.maps.Circle`  | Consultez [Ajouter un cercle à la carte](map-add-shape.md#add-a-circle-to-the-map)                                     |
| `google.maps.ImageMapType`  | [atlas.TileLayer](/javascript/api/azure-maps-control/atlas.layer.tilelayer)         |
| `google.maps.ImageMapTypeOptions` | [atlas.TileLayerOptions](/javascript/api/azure-maps-control/atlas.tilelayeroptions) |
| `google.maps.GroundOverlay`  | [atlas.layer.ImageLayer](/javascript/api/azure-maps-control/atlas.layer.imagelayer)<br/>[atlas.ImageLayerOptions](/javascript/api/azure-maps-control/atlas.imagelayeroptions) |

## <a name="service-classes"></a>Classes de service

Le SDK web Azure Maps comprend un module de services qui peut être chargé séparément. Ce module wrappe les services REST Azure Maps avec une API web et peut être utilisé dans des applications JavaScript, TypeScript et Node.js.

| Google Maps | Azure Maps  |
|-------------|-------------|
| `google.maps.Geocoder` | [atlas.service.SearchUrl](/javascript/api/azure-maps-rest/atlas.service.searchurl)  |
| `google.maps.GeocoderRequest`  | [atlas.SearchAddressOptions](/javascript/api/azure-maps-rest/atlas.service.searchaddressoptions)<br/>[atlas.SearchAddressRevrseOptions](/javascript/api/azure-maps-rest/atlas.service.searchaddressreverseoptions)<br/>[atlas.SearchAddressReverseCrossStreetOptions](/javascript/api/azure-maps-rest/atlas.service.searchaddressreversecrossstreetoptions)<br/>[atlas.SearchAddressStructuredOptions](/javascript/api/azure-maps-rest/atlas.service.searchaddressstructuredoptions)<br/>[atlas.SearchAlongRouteOptions](/javascript/api/azure-maps-rest/atlas.service.searchalongrouteoptions)<br/>[atlas.SearchFuzzyOptions](/javascript/api/azure-maps-rest/atlas.service.searchfuzzyoptions)<br/>[atlas.SearchInsideGeometryOptions](/javascript/api/azure-maps-rest/atlas.service.searchinsidegeometryoptions)<br/>[atlas.SearchNearbyOptions](/javascript/api/azure-maps-rest/atlas.service.searchnearbyoptions)<br/>[atlas.SearchPOIOptions](/javascript/api/azure-maps-rest/atlas.service.searchpoioptions)<br/>[atlas.SearchPOICategoryOptions](/javascript/api/azure-maps-rest/atlas.service.searchpoicategoryoptions) |
| `google.maps.DirectionsService`  | [atlas.service.RouteUrl](/javascript/api/azure-maps-rest/atlas.service.routeurl)  |
| `google.maps.DirectionsRequest`  | [atlas.CalculateRouteDirectionsOptions](/javascript/api/azure-maps-rest/atlas.service.calculateroutedirectionsoptions) |
| `google.maps.places.PlacesService` | [atlas.service.SearchUrl](/javascript/api/azure-maps-rest/atlas.service.searchurl)  |

## <a name="libraries"></a>Bibliothèques

Les bibliothèques ajoutent des fonctionnalités supplémentaires à la carte. Bon nombre de ces bibliothèques se trouvent dans le SDK principal Azure Maps. Voici quelques-unes des classes équivalentes à utiliser à la place de ces bibliothèques Google Maps

| Google Maps           | Azure Maps   |
|-----------------------|--------------|
| Bibliothèque de dessins       | [Module Outils de dessin](set-drawing-options.md) |
| Bibliothèque Geometry      | [atlas.math](/javascript/api/azure-maps-control/atlas.math)   |
| Bibliothèque de visualisation | [Couche de carte thermique](map-add-heat-map-layer.md) |

## <a name="next-steps"></a>Étapes suivantes

Découvrez-en plus sur le SDK web Azure Maps :

> [!div class="nextstepaction"]
> [Comment utiliser le contrôle de carte](how-to-use-map-control.md)

> [!div class="nextstepaction"]
> [Comment utiliser le module Outils de dessin](set-drawing-options.md)

> [!div class="nextstepaction"]
> [Comment utiliser le module de services](how-to-use-services-module.md)

> [!div class="nextstepaction"]
> [Comment utiliser le module d’E/S spatiales](how-to-use-spatial-io-module.md)