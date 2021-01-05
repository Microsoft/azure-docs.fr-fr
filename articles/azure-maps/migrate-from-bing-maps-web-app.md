---
title: 'Tutoriel : Migrer une application web à partir de Bing Cartes | Microsoft Azure Maps'
description: Tutoriel sur la migration d’une application web de Bing Cartes vers Microsoft Azure Maps.
author: rbrundritt
ms.author: richbrun
ms.date: 9/10/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: devx-track-js
ms.openlocfilehash: 6037deb484ca966ab3a54cc60b0d53ac8299d500
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97589999"
---
# <a name="tutorial---migrate-a-web-app-from-bing-maps"></a>Tutoriel – Migrer une application web à partir de Bing Cartes

Les applications web qui utilisent Bing Cartes utilisent souvent le kit SDK JavaScript Bing Cartes V8. Le Kit de développement logiciel (SDK) web Azure Maps est le kit de développement logiciel (SDK) Azure approprié vers lequel migrer. Le Kit de développement logiciel (SDK) web Azure Maps vous permet de personnaliser des cartes interactives avec du contenu et des images qui vous sont propres pour les afficher dans vos applications web ou mobiles. Ce contrôle utilise WebGL, ce qui vous permet d’afficher d’importants jeux de données avec des performances élevées. Développez avec le kit de développement logiciel (SDK) à l’aide de JavaScript ou de TypeScript.

Si vous migrez une application Web existante, vérifiez si elle utilise une bibliothèque de contrôle de carte Open source telle que Cesium, Leaflet et OpenLayers. Si c’est le cas et que vous préférez continuer à utiliser cette bibliothèque, vous pouvez la connecter aux services de mosaïques Azure Maps ([mosaïques routières](/rest/api/maps/render/getmaptile) \| [mosaïques satellites](/rest/api/maps/render/getmapimagerytile)). Les liens ci-dessous fournissent des détails sur l’utilisation d’Azure Maps dans certaines bibliothèques de contrôle de carte open source couramment utilisées.

-   Cesium : contrôle carte 3D pour le Web. [Documentation](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Raster%20Tiles%20in%20Cesium%20JS) \| [de l’exemple de code](https://cesiumjs.org/)
-   Leaflet : contrôle de carte 2D léger pour le Web. [Documentation](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Azure%20Maps%20Raster%20Tiles%20in%20Leaflet%20JS) \| [de l’exemple de code](https://leafletjs.com/)
-   OpenLayers : contrôle de carte 2D pour le Web qui prend en charge les projections. [Documentation](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Raster%20Tiles%20in%20OpenLayers) \| [de l’exemple de code](https://openlayers.org/)

Si vous développez à l’aide d’un framework JavaScript, l’un des projets open source suivants peut être utile :

- [ng-azure-maps](https://github.com/arnaudleclerc/ng-azure-maps) : wrapper Angular 10 autour d’Azure Maps.
- [AzureMapsControl.Components](https://github.com/arnaudleclerc/AzureMapsControl.Components) : composant Blazor Azure Maps.
- [Composant react Azure Maps](https://github.com/WiredSolutions/react-azure-maps) : wrapper react pour le contrôle Azure Maps.
- [Vue Azure Maps](https://github.com/rickyruiz/vue-azure-maps) : composant Azure Maps pour l’application Vue.

## <a name="key-features-support"></a>Prise en charge des fonctionnalités clés

Le tableau suivant liste les principales fonctionnalités de l’API du SDK JavaScript Bing Cartes v8 et la prise en charge d’une API similaire dans le SDK web Azure Maps.

| Fonctionnalité Bing Cartes        | Prise en charge du kit de développement logiciel (SDK) Web Azure Maps                                                             |
|--------------------------|:--------------------------------------------------------------------------------------:|
| Punaises                 | ✓                                                                                      |
| Clustering de punaises       | ✓                                                                                      |
| Polylignes et polygones     | ✓                                                                                      |
| Superposition de sol          | ✓                                                                                      |
| Cartes thermiques                | ✓                                                                                      |
| Couches de mosaïques              | ✓                                                                                      |
| Couche KML                | ✓                                                                                      |
| Couche de contour            | [Exemples](https://azuremapscodesamples.azurewebsites.net/?search=contour)              |
| Couche de compartimentage des données       | [Exemples](https://azuremapscodesamples.azurewebsites.net/?search=data%20binning)       |
| Couche de mosaïque animée      | Incluse dans le [module d’animation](https://github.com/Azure-Samples/azure-maps-animations) open source Azure Maps |
| Outils de dessin            | ✓                                                                                      |
| Service de geocoder         | ✓                                                                                      |
| Service de directions       | ✓                                                                                      |
| Service Matrice des distances  | ✓                                                                                      |
| Service de données spatiales     | N/A                                                                                    |
| Imagerie aérienne/satellite | ✓                                                                                      |
| Imagerie Vue aérienne         | Prévu                                                                                |
| Image Streetside       | Prévu                                                                                |
| Prise en charge de GeoJSON          | ✓                                                                                      |
| Prise en charge de GeoXML           | ✓                                                                                      |
| Prise en charge de Well-Known Text  | ✓                                                                                      |
| Styles de carte personnalisés        | Partial                                                                                |

Azure Maps a également de nombreux [modules open- source supplémentaires pour le SDK web](open-source-projects.md#open-web-sdk-modules) qui étendent ses fonctionnalités.

## <a name="notable-differences-in-the-web-sdks"></a>Différences notables dans les kits de développement logiciel (SDK) Web

Voici quelques-unes des principales différences entre les SDK web Bing Cartes et Azure Maps à connaître :

-   En plus de fournir un point de terminaison hébergé pour accéder au kit de développement logiciel (SDK) Web d’Azure Maps, un package NPM est également disponible pour incorporer le kit de développement logiciel (SDK) Web dans les applications, si vous préférez. Pour plus d’informations, consultez cette [documentation](./how-to-use-map-control.md). Ce package inclut aussi des définitions de TypeScript.
-   Bing Cartes fournit deux branches hébergées de son kit SDK : Release et Experimental. La branche Experimental peut recevoir plusieurs mises à jour par jour pendant le nouveau développement. Azure Maps héberge uniquement une branche Release ; toutefois, des fonctionnalités expérimentales sont créées en tant que modules personnalisés dans le projet d’exemples de code open source Azure Maps. Avant, Bing Cartes avait également une branche figée qui était mise à jour moins fréquemment, réduisant ainsi le risque de changements cassants dus à une mise en production. Dans Azure Maps, vous pouvez utiliser le module NPM et pointer vers n’importe quelle version mineure précédente.

> [!TIP]
> Azure Maps publie des versions minifiées et déminifiées du kit SDK. Il vous suffit de supprimer `.min` des noms de fichiers. La version déminifiée est utile lors du débogage de problème, mais veillez à utiliser la version minifiée en production afin de tirer parti de la plus petite taille de fichier.

-   Après avoir créé une instance de la classe Map dans Azure Maps, votre code doit attendre que les mappages `ready` ou les événements `load` se déclenchent avant d’interagir avec le mappage. Ces événements permettent de s’assurer que toutes les ressources de carte ont été chargées et sont prêtes à être sollicitées.
-   Les deux plateformes utilisent un système de mosaïque similaire pour les cartes de base, mais les mosaïques dans Bing Cartes ont une dimension de 256 pixels tandis que celles d’Azure Maps ont une dimension de 512 pixels. Par conséquent, pour obtenir la même vue cartographique que Bing Cartes dans Azure Maps, le niveau de zoom utilisé dans Bing Cartes doit être soustrait d’une unité dans Azure Maps.
-   Dans Bing Cartes, les coordonnées se présentent sous la forme `latitude, longitude`, tandis qu’Azure Maps utilise `longitude, latitude`. Ce format est conforme à la norme `[x, y]` qui est respectée par la plupart des plateformes GIS.

-   Les formes du kit de développement logiciel (SDK) Web Azure Maps sont basées sur le schéma GeoJSON. Les classes d’assistance sont exposées par le biais de l’espace de noms [atlas.data](/javascript/api/azure-maps-control/atlas.data). Il existe également la classe [atlas.Shape](/javascript/api/azure-maps-control/atlas.shape) qui peut être utilisée pour encapsuler des objets GeoJSON et simplifier leur mise à jour et leur gestion, de manière pouvant être liée aux données.
-   Les coordonnées dans Azure Maps sont définies comme des objets Position qui peuvent être spécifiés sous forme d’un simple tableau de nombres au format `[longitude, latitude]` ou `new atlas.data.Position(longitude, latitude)`.

> [!TIP]
> La classe Position a une fonction d’assistance statique pour l’importation de coordonnées qui sont au format `latitude, longitude`. La fonction [atlas.data.Position.fromLatLng](/javascript/api/azure-maps-control/atlas.data.position) peut souvent remplacer la fonction `new Microsoft.Maps.Location` dans le code Bing Cartes.

-   Au lieu de spécifier des informations de style pour chaque forme ajoutée à la carte, Azure Maps sépare les styles des données. Les données sont stockées dans des sources de données et sont connectées aux couches de rendu que le code Azure Maps utilise pour afficher les données. Cette approche offre un gain de performances amélioré. De plus, de nombreux calques prennent en charge le style basé sur les données dans lequel la logique métier peut être ajoutée aux options de style de calque qui changeront la façon dont les formes individuelles sont rendues au sein d'un calque en fonction des propriétés définies dans la forme.
-   Azure Maps fournit un ensemble de fonctions mathématiques spatiales utiles dans l’espace de noms `atlas.math`, mais elles diffèrent de celles du module mathématique spatial Bing Cartes. La principale différence réside dans le fait qu’Azure Maps ne fournit pas de fonctions intégrées pour les opérations binaires telles que l’union et l’intersection. Toutefois, Azure Maps étant basé sur GeoJSON, qui est une norme ouverte, de nombreuses bibliothèques open source sont disponibles. Une option très populaire qui fonctionne bien avec Azure Maps et fournit une multitude de fonctionnalités mathématiques spatiales est [turf js](http://turfjs.org/).

Consultez également le [Glossaire Azure Maps](./glossary.md) pour obtenir une liste détaillée de la terminologie associée à Azure Maps.

## <a name="web-sdk-side-by-side-examples"></a>Exemples côte à côte du kit de développement logiciel (SDK) Web

Vous trouverez ci-dessous une collection d’exemples de code pour chaque plateforme, qui couvrent les cas d’usage courants pour vous aider à migrer votre application web du kit SDK JavaScript Bing Cartes v8 vers le kit SDK web Azure Maps. Des exemples de code liés aux applications Web sont fournis en JavaScript. Toutefois, Azure Maps fournit également des définitions TypeScript sous la forme d’une option supplémentaire via un module [NPM](./how-to-use-map-control.md).

**Rubriques**

- [Charger une carte](#load-a-map)
- [Localisation de la carte](#localizing-the-map)
- [Définition de la vue cartographique](#setting-the-map-view)
- [Ajout d’une punaise](#adding-a-pushpin)
- [Ajout d’une punaise personnalisée](#adding-a-custom-pushpin)
- [Ajout d’une polyligne](#adding-a-polyline)
- [Ajout d’un polygone](#adding-a-polygon)
- [Afficher une zone d’informations](#display-an-infobox)
- [Clustering de punaises](#pushpin-clustering)
- [Ajouter une carte thermique](#add-a-heat-map)
- [Superposer une couche de mosaïques](#overlay-a-tile-layer)
- [Afficher les données du trafic](#show-traffic-data)
- [Ajouter une superposition de sol](#add-a-ground-overlay)
- [Ajouter des données KML à la carte](#add-kml-data-to-the-map)
- [Ajouter des outils de dessin](#add-drawing-tools)


### <a name="load-a-map"></a>Charger une carte

Le chargement d’une carte dans les deux SDK suit le même ensemble d’étapes :

-   Ajout d'une référence au Kit de développement logiciel (SDK) de la carte.
-   Ajoutez une balise `div` au corps de la page, qui fera office d’espace réservé pour la carte.
-   Créez une fonction JavaScript qui est appelée lorsque la page a été chargée.
-   Créez une instance de la classe de carte respective.

**Quelques différences clés**

-   Bing Cartes demande la spécification d’une clé de compte dans la référence de script de l’API ou en tant qu’option de carte. Les informations d’authentification pour Azure Maps sont spécifiées en tant qu’options de la classe de carte. Il peut s’agir d’une clé d’abonnement ou d’une information Azure Active Directory.
-   Bing Cartes prend une fonction de rappel dans la référence de script de l’API qui est utilisée pour appeler une fonction d’initialisation afin de charger la carte. Avec Azure Maps, l’événement OnLoad de la page doit être utilisé.
-   Quand vous utilisez un ID pour référencer l’élément `div` dans lequel la carte sera affichée, Bing Cartes utilise un sélecteur HTML (c’est-à-dire `#myMap`), tandis qu’Azure Maps utilise uniquement la valeur d’ID (c’est-à-dire `myMap`).
-   Les coordonnées dans Azure Maps sont définies comme des objets Position qui peuvent être spécifiés sous forme d’un simple tableau de nombres au format `[longitude, latitude]`.
-   Le niveau de zoom dans Azure Maps est inférieur d’un niveau à celui de l’exemple Bing Cartes en raison de la différence de taille du système de mosaïques entre les plateformes.
-   Par défaut, Azure Maps n’ajoute aucun contrôle de navigation au canevas de carte, comme des boutons de zoom et des boutons de style de carte. Il existe cependant des commandes pour ajouter un sélecteur de style de carte, des boutons de zoom, une boussole ou une commande de rotation et un réglage de vitesse.
-   Un gestionnaire d’événements est ajouté dans Azure Maps pour surveiller l’événement `ready` de l’instance de la carte. Cela se déclenche lorsque la carte a terminé le chargement du contexte WebGL et de toutes les ressources nécessaires. Vous pouvez ajouter n’importe quel code de chargement dans ce gestionnaire d’événements.

Les exemples ci-dessous montrent comment charger une carte de base de manière à ce qu’elle soit centrée sur les coordonnées de New York à des coordonnées (longitude : -73,985, latitude : 40,747) et est au niveau de zoom 12 dans Bing Cartes.

**Avant : Bing Cartes**

Le code suivant est un exemple d’affichage d’une carte Bing centrée et zoomée sur un emplacement.

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
            map = new Microsoft.Maps.Map('#myMap', {
                credentials: '<Your Bing Maps Key>',
          center: new Microsoft.Maps.Location(40.747, -73.985),
                zoom: 12
            });
        }
    </script>

    <!-- Bing Maps Script Reference -->
    <script src="https://www.bing.com/api/maps/mapcontrol?callback=initMap" async defer></script>
</head>
<body>
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```

L’exécution de ce code dans un navigateur affichera une carte ressemblant à l’image suivante :

<center>

![Carte Bing Cartes](media/migrate-bing-maps-web-app/bing-maps-load-map.jpg)</center>

**Après : Azure Maps**

Le code suivant montre comment charger une carte avec la même vue dans Azure Maps, ainsi qu’un contrôle de style cartographique et des boutons de zoom.

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

                //Add your Azure Maps key to the map SDK. Get an Azure Maps key at https://azure.com/maps. NOTE: The primary key should be used as the key.
                authOptions: {
                    authType: 'subscriptionKey',
                    subscriptionKey: '<Your Azure Maps Key>'
                }
            });

            //Wait until the map resources are ready.
            map.events.add('ready', function () {
                //Add zoom and map style controls to top right of map.
                map.controls.add([
                    new atlas.control.StyleControl(),
                    new atlas.control.ZoomControl()
                ], {
                    position: 'top-right'
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

<center>

![Carte Azure Maps](media/migrate-bing-maps-web-app/azure-maps-load-map.jpg)</center>

Vous trouverez des informations détaillées sur la façon de configurer et d’utiliser le contrôle de carte Azure Maps dans une application Web [ici](./how-to-use-map-control.md).

> [!TIP]
> Azure Maps publie des versions minifiées et déminifiées du kit SDK. Supprimez `.min` des noms de fichiers. La version déminifiée est utile lors du débogage de problème, mais veillez à utiliser la version minifiée en production afin de tirer parti de la plus petite taille de fichier.

**Ressources supplémentaires**

-   Azure Maps fournit également des contrôles de navigation pour faire pivoter et mettre en forme la vue cartographique comme indiqué [ici](./map-add-controls.md).

### <a name="localizing-the-map"></a>Localisation de la carte

Si votre audience est répartie dans plusieurs pays ou parle différentes langues, la localisation est importante.

**Avant : Bing Cartes**

Pour localiser les cartes Bing, la langue et la région sont spécifiées à l’aide des paramètres `setLang` et `UR` ajoutés à la référence de balise `<script>` à l’API. Certaines fonctionnalités de Bing Cartes sont disponibles uniquement sur certains marchés ; le marché de l’utilisateur est spécifié à l’aide du paramètre `setMkt`.

```html
<script type="text/javascript" src="https://www.bing.com/api/maps/mapcontrol?callback=initMap&setLang=[language_code]&setMkt=[market]&UR=[region_code]" async defer></script>
```

Voici un exemple Bing Cartes avec la langue définie sur « fr-FR ».

<center>

![Carte Bing Cartes localisée](media/migrate-bing-maps-web-app/bing-maps-localized-map.jpg)</center>

**Après : Azure Maps**

Azure Maps fournit uniquement des options pour définir la langue et la vue régionale de la carte. Aucun paramètre de marché n’est utilisé pour limiter les fonctionnalités. Il existe deux méthodes pour définir la langue et la vue régionale de la carte. La première option consiste à ajouter ces informations à l’espace de noms `atlas` global, ce qui a pour effet que toutes les instances de contrôle de carte dans votre application ont ces paramètres par défaut. Le code suivant définit la langue sur le français (« fr-FR ») et la vue régionale sur `"auto"` :

```javascript
atlas.setLanguage('fr-FR');
atlas.setView('auto');
```

La deuxième option consiste à transmettre ces informations dans les options de mappage lors du chargement de la carte, comme suit :

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
> Avec Azure Maps il est possible de charger plusieurs instances de carte sur la même page avec des paramètres de langue et de région différents. En outre, il est également possible de mettre à jour ces paramètres dans la carte après son chargement. Vous trouverez une liste détaillée des langues prises en charge dans Azure Maps [ici](./supported-languages.md).

Voici un exemple de Azure Maps avec la langue définie sur « fr » et la région de l’utilisateur définie sur « fr-FR ».

<center>

![Carte Azure Maps localisée](media/migrate-bing-maps-web-app/bing-maps-localized-map.jpg)</center>

### <a name="setting-the-map-view"></a>Définition de la vue cartographique

Les cartes dynamiques dans Bing et Azure Maps peuvent être déplacées par programmation vers de nouvelles positions géographiques en appelant les fonctions appropriées dans JavaScript. Les exemples ci-dessous montrent comment faire en sorte que la carte affiche une image aérienne satellite, centrer la carte sur un emplacement avec des coordonnées (longitude : -111,0225, latitude : 35,0272) et définir le niveau de zoom sur 15 dans Bing Cartes.

> [!NOTE]
> Bing Cartes utilise des mosaïques de 256 pixels, tandis qu’Azure Maps utilise une mosaïque de 512 pixels. Cela réduit le nombre de requêtes réseau nécessaires à Azure Maps pour charger la même zone réactive que Bing Cartes. Toutefois, en raison de la façon dont les pyramides de mosaïques fonctionnent dans les contrôles de carte et de la plus grande taille des mosaïques dans Azure Maps, vous devez soustraire le niveau de zoom utilisé dans Bing Cartes d’une unité lors de l’utilisation d’Azure Maps pour obtenir cette même zone visualisable sous forme de carte dans Bing Cartes.

**Avant : Bing Cartes**

Le contrôle de carte Bing Cartes peut être déplacé par programmation à l’aide de la fonction `setView`, qui vous permet de spécifier le centre de la carte et un niveau de zoom.

```javascript
map.setView({
    mapTypeId: Microsoft.Maps.MapTypeId.aerial,
    center: new Microsoft.Maps.Location(35.0272, -111.0225),
    zoom: 15
});
```

<center>

![Bing Cartes - Définir la vue cartographique](media/migrate-bing-maps-web-app/bing-maps-set-map-view.jpg)</center>

**Après : Azure Maps**

Dans Azure Maps, la position de la carte peut être modifiée par programmation à l’aide de la fonction `setCamera` de la carte, et le style de la carte peut être modifié à l’aide de la fonction `setStyle`. Notez que les coordonnées dans Azure Maps sont au format « longitude, latitude » et que la valeur de niveau de zoom est soustraite d’une unité.

```javascript
map.setCamera({
    center: [-111.0225, 35.0272],
    zoom: 14
});

map.setStyle({
    style: 'satellite_with_roads'
});
```

<center>

![Azure Maps - Définir la vue cartographique](media/migrate-bing-maps-web-app/azure-maps-set-map-view.jpg)</center>

**Ressources supplémentaires**

-   [Choisir un style de carte](./choose-map-style.md)
-   [Styles de carte pris en charge](./supported-map-styles.md)

### <a name="adding-a-pushpin"></a>Ajout d’une punaise

Dans Azure Maps il existe plusieurs manières d’afficher des données de point sur la carte.

-   Marqueurs HTML : effectue le rendu des points à l’aide d’éléments DOM traditionnels. Les marqueurs HTML prennent en charge le glissement.
-   Calque de symbole : effectue le rendu des points avec une icône et/ou un texte dans le contexte WebGL.
-   Calque de bulles : affiche les données de point sous forme de cercles sur la carte. Les rayons des cercles peuvent être mis à l’échelle en fonction des propriétés des données.

Les couches de symboles et de bulles sont rendues dans le contexte WebGL et peuvent afficher de très grands ensembles de points sur la carte. Ces calques requièrent que les données soient stockées dans une source de données. Les sources de données et les calques de rendu doivent être ajoutées à la carte après le déclenchement de l’événement `ready`. Les marqueurs HTML sont affichées sous forme d’éléments DOM dans la page et n’utilisent pas de source de données. Plus le nombre d’éléments DOM d’une page est élevé, plus la page devient rapide. En cas de rendu de plus de quelques centaines de points sur une carte, il est recommandé d’utiliser à la place l’un des calques de rendu.

Les exemples ci-dessous montrent comment ajouter un marqueur à la carte (longitude : -0,2, latitude : 51,5) avec le nombre 10 superposé sous la forme d’une étiquette.

**Avant : Bing Cartes**

Avec Bing Cartes, les marqueurs sont ajoutés à la carte à l’aide de la classe `Microsoft.Maps.Pushpin`*. Les punaises sont ensuite ajoutées à la carte à l’aide de l’une des deux fonctions disponibles.

La première fonction consiste à créer une couche, à insérer la punaise, puis à ajouter la couche à la propriété `layers` de la carte.

```javascript
var pushpin = new Microsoft.Maps.Pushpin(new Microsoft.Maps.Location(51.5, -0.2), {
    text: '10'
});

var layer = new Microsoft.Maps.Layer();
layer.add(pushpin);
map.layers.insert(layer);
```

La seconde consiste à l’ajouter à l’aide de la propriété `entities` de la carte. Cette fonction est marquée comme dépréciée dans la documentation de Bing Cartes V8, mais elle reste partiellement fonctionnelle pour les scénarios de base.

```javascript
var pushpin = new Microsoft.Maps.Pushpin(new Microsoft.Maps.Location(51.5, -0.2), {
    text: '10'
});

map.entities.add(pushpin);
```

<center>

![Bing Cartes - Ajouter une punaise](media/migrate-bing-maps-web-app/bing-maps-add-pushpin.jpg)</center>

**Après : Azure Maps à l’aide de marqueurs HTML**

Dans Azure Maps, vous pouvez utiliser des marqueurs HTML pour afficher facilement un point sur la carte. Leur utilisation est recommandée pour les applications qui doivent simplement afficher un petit nombre de points sur la carte. Pour utiliser un marqueur HTML, créez une instance de la classe `atlas.HtmlMarker`, définissez les options de texte et de position, puis ajoutez le marqueur à la carte à l’aide de la fonction `map.markers.add`.

```javascript
//Create a HTML marker and add it to the map.
map.markers.add(new atlas.HtmlMarker({
    text: '10',
    position: [-0.2, 51.5]
}));
```

<center>

![Azure Maps - Ajouter un marqueur](media/migrate-bing-maps-web-app/azure-maps-add-pushpin.jpg)</center>

**Après : Azure Maps à l’aide d’un calque de symbole**

Lorsque vous utilisez un calque de symbole, les données doivent être ajoutées à une source de données et à la source de données associée au calque. En outre, la source de données et la couche doivent être ajoutées à la carte après le déclenchement de l’événement `ready`. Pour restituer une valeur de texte unique au-dessus d’un symbole, les informations de texte doivent être stockées en tant que propriété du point de données et de la propriété référencée dans l’option `textField` du calque. Il s’agit d’un peu plus de travail que l’utilisation des marqueurs HTML, mais offre de nombreux avantages en termes de performances.

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

<center>

![Azure Maps - Ajouter un calque de symboles](media/migrate-bing-maps-web-app/azure-maps-add-pushpin.jpg)</center>

**Ressources supplémentaires**

-   [Créer une source de données](./create-data-source-web-sdk.md)
-   [Ajouter une calque de symbole](./map-add-pin.md)
-   [Ajouter un calque de bulles](./map-add-bubble-layer.md)
-   [Données de point de cluster](./clustering-point-data-web-sdk.md)
-   [Ajouter des marqueurs HTML](./map-add-custom-html.md)
-   [Utiliser des expressions de style basées sur les données](./data-driven-style-expressions-web-sdk.md)
-   [Options de l’icône de calque de symbole](/javascript/api/azure-maps-control/atlas.iconoptions)
-   [Option de texte du calque de symbole](/javascript/api/azure-maps-control/atlas.textoptions)
-   [Classe de marqueur HTML](/javascript/api/azure-maps-control/atlas.htmlmarker)
-   [Options du marqueur HTML](/javascript/api/azure-maps-control/atlas.htmlmarkeroptions)

### <a name="adding-a-custom-pushpin"></a>Ajout d’une punaise personnalisée

Les images personnalisées peuvent être utilisées pour représenter des points sur une carte. L’image suivante est utilisée dans les exemples ci-dessous. Elle utilise une image personnalisée pour afficher un point sur la carte à la position (latitude : 51,5, longitude : -0,2) et décale la position du marqueur afin que le point de l’icône de la punaise s’aligne avec la position correcte sur la carte.

| ![Azure Maps - Ajouter une punaise](media/migrate-bing-maps-web-app/yellow-pushpin.png)|
|:-----------------------------------------------------------------------:|
| yellow-pushpin.png                                                        |


**Avant : Bing Cartes**

Dans Bing Cartes, un marqueur personnalisé est créé en passant une URL à une image dans les options `icon` d’une punaise. L’option `anchor` est utilisée pour aligner le point de l’image de punaise avec la coordonnée sur la carte. Valeur d’ancrage dans Bing Cartes par rapport à l’angle supérieur gauche de l’image.

```javascript
var pushpin = new Microsoft.Maps.Pushpin(new Microsoft.Maps.Location(51.5, -0.2), {
    icon: 'ylw-pushpin.png',
    anchor: new Microsoft.Maps.Point(5, 30)
});

var layer = new Microsoft.Maps.Layer();
layer.add(pushpin);
map.layers.insert(layer);
```

<center>

![Bing Cartes - Ajouter une punaise personnalisée](media/migrate-bing-maps-web-app/bing-maps-add-custom-pushpin.jpg)</center>

**Après : Azure Maps à l’aide de marqueurs HTML**

Pour personnaliser un marqueur HTML dans Azure Maps un `string` ou `HTMLElement` HTML peut être passé dans l’option `htmlContent` du marqueur. Dans Azure Maps, une option `anchor` est utilisée pour spécifier la position relative du marqueur par rapport à la coordonnée de position à l’aide d’un des neuf points de référence définis : « center », « top », « bottom », « left », « right », « top-left », « top-right », « bottom-left », « bottom-right ». Le contenu est ancré et défini sur « bottom » par défaut, ce qui correspond à la partie « en bas au centre » du contenu HTML. Pour faciliter la migration du code à partir de Bing Cartes, définissez l’ancre sur « top-left », puis utilisez l’option `offset` avec le même décalage que celui utilisé dans Bing Cartes. Les décalages dans Azure Maps se déplacent dans la direction opposée de Bing Cartes. Par conséquent, multipliez-les par moins un.

> [!TIP]
> Ajoutez `pointer-events:none` en tant que style au contenu HTML pour désactiver le comportement de glissement par défaut dans MS Edge, qui affichera une icône indésirable.

```html
map.markers.add(new atlas.HtmlMarker({
    htmlContent: '<img src="ylw-pushpin.png" style="pointer-events: none;" />',
    anchor: 'top-left',
    pixelOffset: [-5, -30],
    position: [-0.2, 51.5]
}));
```

<center>

![Azure Maps - Ajouter un marqueur personnalisé](media/migrate-bing-maps-web-app/azure-maps-add-custom-marker.jpg)</center>

**Après : Azure Maps à l’aide d’un calque de symbole**

Dans Azure Maps, les calques de symboles prennent également en charge les images personnalisées, mais l’image doit d’abord être chargée dans les ressources de carte et recevoir un ID unique. Le calque de symbole peut ensuite référencer cet ID. Le symbole peut être décalé pour s’aligner sur le point correct sur l’image à l’aide de l’icône `offset` option. Dans Azure Maps, une option `anchor` est utilisée pour spécifier la position relative du symbole par rapport à la coordonnée de position à l’aide d’un des neuf points de référence définis : « center », « top », « bottom », « left », « right », « top-left », « top-right », « bottom-left », « bottom-right ». Le contenu est ancré et défini sur « bottom » par défaut, ce qui correspond à la partie « en bas au centre » du contenu HTML. Pour faciliter la migration du code à partir de Bing Cartes, définissez l’ancre sur « top-left », puis utilisez l’option `offset` avec le même décalage que celui utilisé dans Bing Cartes. Les décalages dans Azure Maps se déplacent dans la direction opposée de Bing Cartes. Par conséquent, multipliez-les par moins un.

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
                map.imageSprite.add('my-yellow-pin', 'ylw-pushpin.png').then(function () {

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

<center>

![Bing Cartes - Ajouter un calque de symboles personnalisé](media/migrate-bing-maps-web-app/azure-maps-add-custom-symbol-layer.jpg)</center>

> [!TIP]
> Pour créer un rendu personnalisé avancé des points, utilisez plusieurs calques de rendu ensemble. Par exemple, si vous souhaitez avoir plusieurs punaises qui ont la même icône sur des cercles de couleurs différentes, au lieu de créer un groupe d'images pour chaque couleur, superposez un calque de symboles au-dessus d'un calque de bulles et demandez-leur de référencer la même source de données. Cela sera beaucoup plus efficace que de créer un nouveau groupe d’images et de faire en sorte que la carte maintienne une série d’images différentes.

**Ressources supplémentaires**

-   [Créer une source de données](./create-data-source-web-sdk.md)
-   [Ajouter une calque de symbole](./map-add-pin.md)
-   [Ajouter des marqueurs HTML](./map-add-custom-html.md)
-   [Utiliser des expressions de style basées sur les données](./data-driven-style-expressions-web-sdk.md)
-   [Options de l’icône de calque de symbole](/javascript/api/azure-maps-control/atlas.iconoptions)
-   [Option de texte du calque de symbole](/javascript/api/azure-maps-control/atlas.textoptions)
-   [Classe de marqueur HTML](/javascript/api/azure-maps-control/atlas.htmlmarker)
-   [Options du marqueur HTML](/javascript/api/azure-maps-control/atlas.htmlmarkeroptions)

### <a name="adding-a-polyline"></a>Ajout d’une polyligne

Les polylignes sont utilisées pour représenter une ligne ou un tracé sur la carte. Les exemples ci-dessous montrent comment créer une polyligne en pointillés sur la carte.

**Avant : Bing Cartes**

Dans Bing Cartes, la classe Polyline prend un tableau de positions et un ensemble d’options.

```javascript
//Get the center of the map.
var center = map.getCenter();

//Create the polyline.
var polyline = new Microsoft.Maps.Polyline([
        center,
        new Microsoft.Maps.Location(center.latitude - 0.5, center.longitude - 1),
        new Microsoft.Maps.Location(center.latitude - 0.5, center.longitude + 1)
    ], {
        strokeColor: 'red',
        strokeThickness: 4,
        strokeDashArray: [3, 3]
    });

//Add the polyline to the map using a layer.
var layer = new Microsoft.Maps.Layer();
layer.add(polyline);
map.layers.insert(layer);
```

<center>

![Polyligne Bing Cartes](media/migrate-bing-maps-web-app/bing-maps-line.jpg)</center>

**Après : Azure Maps**

Dans Azure Maps, il est fait référence aux polylignes par les termes géospatiaux plus courants d’objets `LineString` ou `MultiLineString`. Ces objets peuvent être ajoutés à une source de données et restitués à l’aide d’une couche de lignes. Les options de couleur du trait, de largeur et de tableau de tirets sont quasiment identiques entre les plateformes.

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

<center>

![Lignes Azure Maps](media/migrate-bing-maps-web-app/azure-maps-line.jpg)</center>

**Ressources supplémentaires**

-   [Ajouter des lignes à la carte](./map-add-line-layer.md)
-   [Options du calque de ligne](/javascript/api/azure-maps-control/atlas.linelayeroptions)
-   [Utiliser des expressions de style basées sur les données](./data-driven-style-expressions-web-sdk.md)

### <a name="adding-a-polygon"></a>Ajout d’un polygone

Les polygones sont utilisés pour représenter une zone sur la carte. Azure Maps et Bing Cartes offrent une prise en charge très similaire des polygones. Les exemples ci-dessous montrent comment créer un polygone qui forme un triangle en fonction de la coordonnée centrale de la carte.

**Avant : Bing Cartes**

Dans Bing Cartes, la classe Polygon prend un tableau de coordonnées ou d’anneaux de coordonnées et un ensemble d’options.

```javascript
//Get the center of the map.
var center = map.getCenter();

//Create the polygon.
var polygon = new Microsoft.Maps.Polygon([
        center,
        new Microsoft.Maps.Location(center.latitude - 0.5, center.longitude - 1),
        new Microsoft.Maps.Location(center.latitude - 0.5, center.longitude + 1),
        center
    ], {
        fillColor: 'rgba(0, 255, 0, 0.5)',
        strokeColor: 'red',
        strokeThickness: 2
    });

//Add the polygon to the map using a layer.
var layer = new Microsoft.Maps.Layer();
layer.add(polygon);
map.layers.insert(layer);
```

<center>

![Polygone Bing Cartes](media/migrate-bing-maps-web-app/azure-maps-polygon.jpg)</center>

**Après : Azure Maps**

Dans Azure Maps, les objets Polygone et Multipolygone peuvent être ajoutés à une source de données et restitués sur la carte à l’aide de couches. La zone d’un polygone peut être affichée dans une couche de polygones. Le contour d’un polygone peut être affiché à l’aide d’un calque de lignes.

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

<center>

![Polygone Azure Maps](media/migrate-bing-maps-web-app/azure-maps-polygon.jpg)</center>

**Ressources supplémentaires**

-   [Ajouter un polygone à la carte](./map-add-shape.md#use-a-polygon-layer)
-   [Ajouter un cercle à la carte](./map-add-shape.md#add-a-circle-to-the-map)
-   [Options du calque de polygones](/javascript/api/azure-maps-control/atlas.polygonlayeroptions)
-   [Options du calque de ligne](/javascript/api/azure-maps-control/atlas.linelayeroptions)
-   [Utiliser des expressions de style basées sur les données](./data-driven-style-expressions-web-sdk.md)

### <a name="display-an-infobox"></a>Afficher une zone d’informations

Des informations supplémentaires pour une entité peuvent être affichées sur la carte en tant que classe `Microsoft.Maps.Infobox` dans Bing Cartes ; dans Azure Maps, cela peut être obtenu à l’aide de la classe `atlas.Popup`. Les exemples ci-dessous ajoutent une punaise ou un marqueur à la carte et, lorsque vous cliquez dessus, affichent une zone d’informations/fenêtre contextuelle.

**Avant : Bing Cartes**

Avec Bing Cartes, une zone d’informations est créée à l’aide du constructeur `Microsoft.Maps.Infobox`.

```javascript
//Add a pushpin where we want to display an infobox.
var pushpin = new Microsoft.Maps.Pushpin(new Microsoft.Maps.Location(47.6, -122.33));

//Add the pushpin to the map using a layer.
var layer = new Microsoft.Maps.Layer();
layer.add(pushpin);
map.layers.insert(layer);

//Create an infobox and bind it to the map.
var infobox = new Microsoft.Maps.Infobox(new Microsoft.Maps.Location(47.6, -122.33), {
    description: '<div style="padding:5px"><b>Hello World!</b></div>',
    visible: false
});
infobox.setMap(map);

//Add a click event to the pushpin to open the infobox.
Microsoft.Maps.Events.addHandler(pushpin, 'click', function () {
    infobox.setOptions({ visible: true });
});
```

<center>

![Zone d’informations Bing Cartes](media/migrate-bing-maps-web-app/bing-maps-infobox.jpg)</center>

**Après : Azure Maps**

Dans Azure Maps, une fenêtre contextuelle peut être utilisée pour afficher des informations supplémentaires sur une position. Un objet `string` ou `HTMLElement` HTML peut être passé dans l’option `content` de la fenêtre contextuelle. Les fenêtres contextuelles peuvent être affichées indépendamment de toute forme si vous le souhaitez. par conséquent, vous devez spécifier une valeur `position`. Pour afficher une fenêtre contextuelle, appelez la fonction `open` et transmettez la carte sur laquelle la fenêtre contextuelle doit être affichée.

```javascript
//Add a marker to the map that to display a popup for.
var marker = new atlas.HtmlMarker({
    position: [-122.33, 47.6]
});

//Add the marker to the map.
map.markers.add(marker);

//Create a popup.
var popup = new atlas.Popup({
    content: '<div style="padding:10px"><b>Hello World!</b></div>',
    position: [-122.33, 47.6],
    pixelOffset: [0, -35]
});

//Add a click event to the marker to open the popup.
map.events.add('click', marker, function () {
    //Open the popup
    popup.open(map);
});
```

<center>

![Fenêtre contextuelle Azure Maps](media/migrate-bing-maps-web-app/azure-maps-popup.jpg)</center>

> [!NOTE]
> Pour effectuer la même opération avec un calque de symboles, de bulles, de lignes ou de polygones, transmettez le calque au code d’événement de la carte plutôt qu’à un marqueur.

**Ressources supplémentaires**

-   [Ajouter une fenêtre contextuelle](./map-add-popup.md)
-   [Fenêtre contextuelle avec contenu multimédia](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Popup%20with%20Media%20Content)
-   [Fenêtres contextuelles sur les formes](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Popups%20on%20Shapes)
-   [Réutilisation d’une fenêtre contextuelle avec plusieurs épingles](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Reusing%20Popup%20with%20Multiple%20Pins)
-   [Classe de fenêtre contextuelle](/javascript/api/azure-maps-control/atlas.popup)
-   [Options de la fenêtre contextuelle](/javascript/api/azure-maps-control/atlas.popupoptions)

### <a name="pushpin-clustering"></a>Clustering de punaises

Lorsque vous examinez plusieurs points de données sur la carte, les points se chevauchent et la carte semble surchargée, ce qui nuit à la visibilité et à l’utilisation. Le clustering de données point peut être utilisé pour améliorer cette expérience utilisateur et améliorer les performances. Le clustering de point de données est le processus permettant de combiner des données de point proches les unes des autres et de les représenter sur une carte en tant qu’un point de données en cluster unique. Lorsque l’utilisateur effectue un zoom avant sur la carte, les clusters se décomposent pour afficher les points de données individuels qui les composent.

Les exemples ci-dessous chargent un flux GeoJSON de données de séisme de la semaine dernière et l’ajoutent à la carte. Les clusters sont rendus sous forme de cercles mis à l’échelle et colorés en fonction du nombre de points qu’ils contiennent.

> [!NOTE]
> Il existe plusieurs algorithmes différents utilisés pour le clustering de punaises. Bing Cartes utilise une fonction simple basée sur une grille, tandis qu’Azure Maps utilise des méthodes de clustering basées sur des points plus avancées et visuellement plus attrayantes.

**Avant : Bing Cartes**

Dans Bing Cartes, les données GeoJSON peuvent être chargées à l’aide du module GeoJSON. Les punaises peuvent être mises en cluster en chargeant le module de clustering et en utilisant la couche de clustering qu’il contient.

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
            map = new Microsoft.Maps.Map(document.getElementById('myMap'), {
                credentials: '<Your Bing Maps Key>',
                center: new Microsoft.Maps.Location(20, -160),
                zoom: 2
            });

            //Load the GeoJSON and Clustering modules.
            Microsoft.Maps.loadModule(['Microsoft.Maps.GeoJson', 'Microsoft.Maps.Clustering'], function () {

                //Load the GeoJSON data from a URL.
                Microsoft.Maps.GeoJson.readFromUrl(earthquakeFeed, function (pins) {

                    //Create a ClusterLayer with options and add it to the map.
                    clusterLayer = new Microsoft.Maps.ClusterLayer(pins, {
                        clusteredPinCallback: createCustomClusteredPin,
                        gridSize: 100
                    });

                    map.layers.insert(clusterLayer);
                });
            });
        }

        //A function that defines how clustered pins are rendered.
        function createCustomClusteredPin(cluster) {
            //Get the number of pushpins in the cluster
            var clusterSize = cluster.containedPushpins.length;

            var radius = 20;    //Default radius to 20 pixels.
            var fillColor = 'lime';   //Default to lime green.

            if (clusterSize >= 750) {
                radius = 40;   //If point_count >= 750, radius is 40 pixels.
                fillColor = 'red';    //If the point_count >= 750, color is red.
            } else if (clusterSize >= 100) {
                radius = 30;    //If point_count >= 100, radius is 30 pixels.
                fillColor = 'yellow';    //If the point_count >= 100, color is yellow.
            }

            //Create an SVG string of a circle with the specified radius and color.
            var svg = ['<svg xmlns="http://www.w3.org/2000/svg" width="', (radius * 2), '" height="', (radius * 2), '">',
                '<circle cx="', radius, '" cy="', radius, '" r="', radius, '" fill="', fillColor, '"/>',
                '<text x="50%" y="50%" dominant-baseline="middle" text-anchor="middle" style="font-size:12px;font-family:arial;fill:black;" >{text}</text>',
                '</svg>'];

            //Customize the clustered pushpin using the generated SVG and anchor on its center.
            cluster.setOptions({
                icon: svg.join(''),
                anchor: new Microsoft.Maps.Point(radius, radius),
                textOffset: new Microsoft.Maps.Point(0, radius - 8) //Subtract 8 to compensate for height of text.
            });
        }
    </script>

    <!-- Bing Maps Script Reference -->
    <script src="https://www.bing.com/api/maps/mapcontrol?callback=initMap" async defer></script>
</head>
<body>
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```

<center>

![Clustering Bing Cartes](media/migrate-bing-maps-web-app/bing-maps-clustering.jpg)</center>

**Après : Azure Maps**

Dans Azure Maps, les données sont ajoutées et gérées par une source de données. Les calques se connectent aux sources de données et affichent les données qu’elles contiennent. La classe `DataSource` dans Azure Maps fournit plusieurs options de clustering.

-   `cluster` : Indique à la source de données les données de point de cluster. 
-   `clusterRadius` : Rayon, en pixels, des points de cluster.
-   `clusterMaxZoom` : niveau de zoom maximal où le clustering se produit. Si vous effectuez un zoom plus grand que celui-ci, tous les points sont rendus sous forme de symboles.
-   `clusterProperties` : Définit des propriétés personnalisées qui sont calculées à l’aide d’expressions sur tous les points de chaque cluster et ajoutées aux propriétés de chaque point de cluster.

Lorsque le clustering est activé, la source de données envoie des points de données en cluster et non cluster aux calques pour le rendu. La source de données est en charge de la mise en cluster de centaines de milliers de points de données. Un point de données en cluster possède les propriétés suivantes :

| Nom de la propriété               | Type    | Description                                    |
|-----------------------------|---------|------------------------------------------------|
| `cluster`                   | boolean | Indique si la fonctionnalité représente un cluster.     |
| `cluster_id`                | string  | ID unique pour le cluster qui peut être utilisé avec les fonctions `getClusterExpansionZoom`, `getClusterChildren` et `getClusterLeaves` des classes `DataSource`. |
| `point_count`               | nombre  | Le nombre de points que contient le cluster.     |
| `point_count_abbreviated`   | string  | Une chaîne qui abrège la valeur `point_count` si elle est trop longue. (par exemple, 4 000 devient 4K) |

La classe `DataSource` possède la fonction d’assistance suivante pour accéder à des informations supplémentaires sur un cluster à l’aide de `cluster_id`.

| Fonction       | Type de retour        | Description     |
|----------------|--------------------|-----------------|
| `getClusterChildren(clusterId: number)`                              | `Promise<Feature<Geometry, any> | Shape>` | Récupère les enfants du cluster donné sur le niveau de zoom suivant. Ces enfants peuvent être une combinaison de formes et sous-clusters. Les sous-clusters seront des fonctionnalités dont les propriétés correspondent à propriétés de cluster. |
| `getClusterExpansionZoom(clusterId: number)`                         | `Promise<number>`                            | Calcule un niveau de zoom à partir duquel le cluster commence à se développer ou à se décomposer.    |
| `getClusterLeaves(clusterId: number, limit: number, offset: number)` | `Promise<Feature<Geometry, any> | Shape>` | Récupère tous les points dans un cluster. Définissez le paramètre `limit` de manière à retourner un sous-ensemble des points, et utilisez `offset` pour parcourir les points.    |

Lors du rendu des données en cluster sur la carte, il est souvent plus facile d’utiliser au moins deux calques. L’exemple ci-dessous utilise trois calques : un calque de bulles pour dessiner des cercles de couleur mis à l’échelle en fonction de la taille des clusters, un calque de symboles pour afficher la taille de cluster en tant que texte, et un deuxième calque de symboles pour le rendu des points non cluster. Il existe de nombreuses autres façons de restituer des données en cluster dans Azure Maps mises en surbrillance dans la documentation [Données de points de cluster](./clustering-point-data-web-sdk.md).

Les données GeoJSON peuvent être importées directement dans Azure Maps à l’aide de la fonction `importDataFromUrl` sur la classe `DataSource`.

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

                //Add your Azure Maps key to the map SDK. Get an Azure Maps key at https://azure.com/maps. NOTE: The primary key should be used as the key.
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
                        filter: ['has', 'point_count'] //Only rendered data points that have a point_count property, which clusters do.
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

<center>

![Clusters Azure Maps](media/migrate-bing-maps-web-app/azure-maps-clustering.jpg)</center>

**Ressources supplémentaires**

-   [Ajouter une calque de symbole](./map-add-pin.md)
-   [Ajouter un calque de bulles](./map-add-bubble-layer.md)
-   [Données de point de cluster](./clustering-point-data-web-sdk.md)
-   [Utiliser des expressions de style basées sur les données](./data-driven-style-expressions-web-sdk.md)

### <a name="add-a-heat-map"></a>Ajouter une carte thermique

Les cartes thermiques, également appelées « cartes de densité de points », sont un type de visualisation de données utilisé pour représenter la densité de données à l’aide d’une palette de couleurs. Souvent utilisées pour afficher les « points chauds » des données sur une carte, elles sont idéales pour restituer de grands jeux de données de points.

Les exemples ci-dessous chargent un flux GeoJSON de tous les séismes détectés par l’Institut d’études géologiques des États-Unis (USGS) au cours du mois précédent, et les affichent sous la forme d’une carte thermique.

**Avant : Bing Cartes**

Dans Bing Cartes, pour créer une carte thermique, chargez le module de carte thermique. De même, le module GeoJSON est chargé pour ajouter la prise en charge des données GeoJSON.

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
            map = new Microsoft.Maps.Map(document.getElementById('myMap'), {
                credentials: '<Your Bing Maps Key>',
                center: new Microsoft.Maps.Location(20, -160),
                zoom: 2,
                mapTypeId: Microsoft.Maps.MapTypeId.aerial
            });

            //Load the GeoJSON and HeatMap modules.
            Microsoft.Maps.loadModule(['Microsoft.Maps.GeoJson', 'Microsoft.Maps.HeatMap'], function () {

                //Load the GeoJSON data from a URL.
                Microsoft.Maps.GeoJson.readFromUrl(earthquakeFeed, function (shapes) {

                    //Create a heat map and add it to the map.
                    var heatMap = new Microsoft.Maps.HeatMapLayer(shapes, {
                        opacity: 0.65,
                        radius: 10
                    });
                    map.layers.insert(heatMap);
                });
            });
        }
    </script>

    <!-- Bing Maps Script Reference -->
    <script src="https://www.bing.com/api/maps/mapcontrol?callback=initMap" async defer></script>
</head>
<body>
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```

<center>

![Bing Cartes - Carte thermique](media/migrate-bing-maps-web-app/bing-maps-heatmap.jpg)</center>

**Après : Azure Maps**

Dans Azure Maps, chargez les données GeoJSON dans une source de données et connectez la source de données à une couche de carte thermique. Les données GeoJSON peuvent être importées directement dans Azure Maps à l’aide de la fonction `importDataFromUrl` sur la classe `DataSource`.

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
                style: 'satellite_with_roads',

                //Add your Azure Maps key to the map SDK. Get an Azure Maps key at https://azure.com/maps. NOTE: The primary key should be used as the key.
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

<center>

![Azure Maps - Carte thermique](media/migrate-bing-maps-web-app/azure-maps-heatmap.jpg)</center>

**Ressources supplémentaires**

-   [Ajouter une couche de carte thermique](./map-add-heat-map-layer.md)
-   [Classe de couche de carte thermique](/javascript/api/azure-maps-control/atlas.layer.heatmaplayer)
-   [Options de la couche de carte thermique](/javascript/api/azure-maps-control/atlas.heatmaplayeroptions)
-   [Utiliser des expressions de style basées sur les données](./data-driven-style-expressions-web-sdk.md)

### <a name="overlay-a-tile-layer"></a>Superposer une couche de mosaïques

Les couches de mosaïques vous permettent de superposer des images de grande taille qui ont été divisées en images en mosaïque plus petites qui s’alignent sur le système de mosaïque de cartes. Il s’agit d’un moyen courant de superposer des images de grande taille ou des jeux de données très volumineux.

Les exemples ci-dessous superposent une couche de mosaïques radar météo issue de l’Iowa Environmental Mesonet de l’Iowa State University qui utilise un schéma de nommage de mosaïque de zoom X, Y.

**Avant : Bing Cartes**

Dans Bing Cartes, les couches de mosaïques peuvent être créées à l’aide de la classe `Microsoft.Maps.TileLayer`.

```javascript
var weatherTileLayer = new Microsoft.Maps.TileLayer({
    mercator: new Microsoft.Maps.TileSource({
        uriConstructor: 'https://mesonet.agron.iastate.edu/cache/tile.py/1.0.0/nexrad-n0q-900913/{zoom}/{x}/{y}.png'
    })
});
map.layers.insert(weatherTileLayer);
```

<center>

![Bing Cartes - Carte thermique pondérée](media/migrate-bing-maps-web-app/bing-maps-weighted-heatmap.jpg)</center>

**Après : Azure Maps**

Dans Azure Maps, une couche de mosaïques peut être ajoutée à la carte à peu près de la même façon que n’importe quelle autre couche. URL mise en forme qui a des espaces réservés de zoom x, y ; `{x}`, `{y}`, `{z}` sont utilisés respectivement pour indiquer à la couche où accéder aux vignettes. Les couches de tuiles Azure Maps prennent également en charge les espaces réservés `{quadkey}`, `{bbox-epsg-3857}` et `{subdomain}`.

> [!TIP]
> Dans Azure Maps, les couches peuvent facilement être rendues sous d’autres couches, y compris les couches de la carte de base. Il est souvent souhaitable de restituer des couches de mosaïques sous les étiquettes de carte afin qu’elles soient faciles à lire. La fonction `map.layers.add` accepte un deuxième paramètre qui est l’ID d’une deuxième couche sous laquelle insérer la nouvelle couche. Pour insérer une couche de mosaïques sous les étiquettes de carte, vous pouvez utiliser le code suivant :
> 
> `map.layers.add(myTileLayer, "labels");`

```javascript
//Create a tile layer and add it to the map below the label layer.
map.layers.add(new atlas.layer.TileLayer({
    tileUrl: 'https://mesonet.agron.iastate.edu/cache/tile.py/1.0.0/nexrad-n0q-900913/{z}/{x}/{y}.png',
    opacity: 0.8,
    tileSize: 256
}), 'labels');
```

<center>

![Azure Maps - Carte thermique pondérée](media/migrate-bing-maps-web-app/azure-maps-weighted-heatmap.jpg)</center>

> [!TIP]
> Les demandes de mosaïque peuvent être capturées à l’aide de l’option `transformRequest` de la carte. Cela vous permettra de modifier ou d’ajouter des en-têtes à la demande, si vous le souhaitez.

**Ressources supplémentaires**

-   [Ajouter des couches de vignettes](./map-add-tile-layer.md)
-   [Classe de couche de mosaïque](/javascript/api/azure-maps-control/atlas.layer.tilelayer)
-   [Options de la couche de mosaïques](/javascript/api/azure-maps-control/atlas.tilelayeroptions)

### <a name="show-traffic-data"></a>Afficher les données du trafic

Les données de trafic peuvent être superposées à la fois sur des cartes Bing et Azure.

**Avant : Bing Cartes**

Dans Bing Cartes, les données de trafic peuvent être superposées à la carte à l’aide du module de trafic.

```javascript
Microsoft.Maps.loadModule('Microsoft.Maps.Traffic', function () {
    var manager = new Microsoft.Maps.Traffic.TrafficManager(map);
    manager.show();
});
```

<center>

![Bing Cartes - Trafic](media/migrate-bing-maps-web-app/bing-maps-traffic.jpg)</center>

**Après : Azure Maps**

Azure Maps offre plusieurs options différentes pour l’affichage du trafic. Les incidents de trafic, tels que les fermetures de route et les accidents, peuvent être affichés sous forme d’icônes sur la carte. Des routes codées en couleurs représentant le flux de trafic peuvent être superposées sur la carte ; les couleurs peuvent être modifiées pour tenir compte de la limite de vitesse indiquée, d’un retard prévu normal ou d’un retard absolu. Les données d’incident dans Azure Maps sont mises à jour toutes les minutes et les données de transit toutes les deux minutes.

```javascript
map.setTraffic({
    incidents: true,
    flow: 'relative'
});
```

<center>

![Trafic Azure Maps](media/migrate-bing-maps-web-app/azure-maps-traffic.jpg)</center>

Si vous cliquez sur l’une des icônes de trafic dans Azure Maps, des informations supplémentaires s’affichent dans une fenêtre contextuelle.

<center>

![Azure Maps - Fenêtre contextuelle de trafic](media/migrate-bing-maps-web-app/azure-maps-traffic-popup.jpg)</center>

**Ressources supplémentaires**

-   [Afficher le trafic sur la carte](./map-show-traffic.md)
-   [Options de superposition du trafic](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Traffic%20Overlay%20Options)
-   [Gestion du trafic](https://azuremapscodesamples.azurewebsites.net/?sample=Traffic%20controls)

### <a name="add-a-ground-overlay"></a>Ajouter une superposition de sol

Les cartes Bing et Azure prennent en charge la superposition des images géoréférencées sur la carte ; ainsi, elles se déplacent et sont mises à l’échelle à mesure que vous effectuez un panoramique et un zoom sur la carte. Dans Bing Cartes, ces superpositions sont appelées « superpositions au sol », tandis que dans Azure Maps elles sont appelées « calques d’images ». Ils sont très utiles pour créer des plans d’étage, superposer d’anciennes cartes ou images à partir d’un drone.

**Avant : Bing Cartes**

Lors de la création d’une superposition au sol dans Bing Cartes, vous devez spécifier l’URL de l’image à superposer et un cadre englobant auquel lier l’image sur la carte. Cet exemple superpose une image de carte de [Newark New Jersey en 1922](https://www.lib.utexas.edu/maps/historical/newark_nj_1922.jpg) sur la carte.

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
            map = new Microsoft.Maps.Map(document.getElementById('myMap'), {
                credentials: '<Your Bing Maps Key>',
                center: new Microsoft.Maps.Location(40.740, -74.18),
                zoom: 12
            });

            var overlay = new Microsoft.Maps.GroundOverlay({
                //Create a LocationRect from the edges of the bounding box; north, west, south, east.
                bounds: Microsoft.Maps.LocationRect.fromEdges(40.773941, -74.22655, 40.712216, -74.12544),
                imageUrl: 'newark_nj_1922.jpg'
            });
            map.layers.insert(overlay);
        }
    </script>

    <!-- Bing Maps Script Reference -->
    <script src="https://www.bing.com/api/maps/mapcontrol?callback=initMap" async defer></script>
</head>
<body>
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```

L’exécution de ce code dans un navigateur affichera une carte ressemblant à l’image suivante :

<center>

![Bing Cartes - Superposition au sol](media/migrate-bing-maps-web-app/bing-maps-ground-overlay.jpg)</center>

**Après : Azure Maps**

Dans Azure Maps, les images géoréférencées peuvent être superposées à l’aide de la classe `atlas.layer.ImageLayer`. Cette classe requiert une URL vers une image et un ensemble de coordonnées pour les quatre coins de l’image. L’image doit être hébergée soit sur le même domaine, soit avec CORs activé.

> [!TIP]
> Si vous avez uniquement des informations relatives au nord, au sud, à l’est, à l’ouest et à la rotation, et non les coordonnées de chaque angle de l’image, vous pouvez utiliser la fonction statique [atlas.layer.ImageLayer.getCoordinatesFromEdges](/javascript/api/azure-maps-control/atlas.layer.imagelayer#getcoordinatesfromedges-number--number--number--number--number-).

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

                //Add your Azure Maps key to the map SDK. Get an Azure Maps key at https://azure.com/maps. NOTE: The primary key should be used as the key.
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

<center>

![Azure Maps - Calque d’image](media/migrate-bing-maps-web-app/azure-maps-ground-overlay.jpg)</center>

**Ressources supplémentaires**

-   [Superposer une image](./map-add-image-layer.md)
-   [Classe de couche image](/javascript/api/azure-maps-control/atlas.layer.imagelayer)

### <a name="add-kml-data-to-the-map"></a>Ajouter des données KML à la carte

Les cartes Azure et Bing peuvent importer et restituer des données KML, KMZ, GeoRSS, GeoJSON et Well-Known Text (WKT) sur la carte. Azure Maps prend également en charge les fichiers GPX, GML, les fichiers CSV de données spatiales, et les services WMS (Web-Mapping Service), WMTS (Web-Mapping Tile Service) et WFS (Web Feature Service).

**Avant : Bing Cartes**

L’exécution de ce code dans un navigateur affichera une carte ressemblant à l’image suivante :

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
            map = new Microsoft.Maps.Map('#myMap', {
                credentials: '<Your Bing Maps Key>',
                center: new Microsoft.Maps.Location(40.747, -73.985),
                zoom: 12
            });

            Microsoft.Maps.loadModule('Microsoft.Maps.GeoXml', function () {
                var callback = function (dataset) {
                    if (dataset.shapes) {
                        var l = new Microsoft.Maps.Layer();
                        l.add(dataset.shapes);
                        map.layers.insert(l);
                    }
                    if (dataset.layers) {
                        for (var i = 0, len = dataset.layers.length; i < len; i++) {
                            map.layers.insert(dataset.layers[i]);
                        }
                    }
                };
                Microsoft.Maps.GeoXml.readFromUrl('myKMLFile.kml', { error: function (msg) { alert(msg); } }, callback);
            });                
        }
    </script>

    <!-- Bing Maps Script Reference -->
    <script src="https://www.bing.com/api/maps/mapcontrol?callback=initMap" async defer></script>
</head>
<body>
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```

<center>

![Bing Cartes - kml](media/migrate-bing-maps-web-app/bing-maps-kml.jpg)</center>

**Après : Azure Maps**

Dans Azure Maps, le format GeoJSON est le format de données principal utilisé dans le SDK web. Vous pouvez facilement intégrer d’autres formats de données spatiales à l’aide du [module d’E/S de données spatiales](/javascript/api/azure-maps-spatial-io/). Ce module présente des fonctions de lecture et d’écriture de données spatiales, et inclut un calque de données simple restituant facilement les données à partir de l’un de ces formats de données spatiales. Pour lire les données d’un fichier de données spatiales, transmettez une URL ou des données brutes sous la forme d’une chaîne ou d’un objet blob dans la fonction `atlas.io.read`. Ceci retournera toutes les données analysées du fichier, qui pourront alors être ajoutées à la carte. Le format KML est un peu plus complexe que la plupart des autres formats de données spatiales. En effet, il inclut beaucoup plus d’informations de style. La classe `SpatialDataLayer` prend en charge le rendu d’une majorité de ces styles. Cependant, les images d’icônes doivent être chargées dans la carte avant les données de caractéristiques, et les calques de relief doivent être ajoutés séparément à la carte sous forme de couches. Quand vous chargez des données par le biais d’une URL, elles doivent être hébergées sur un point de terminaison CORS, ou un service proxy doit être transmis comme option dans la fonction read.

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

    <!-- Add reference to the Azure Maps Spatial IO module. -->
    <script src="https://atlas.microsoft.com/sdk/javascript/spatial/0/atlas-spatial.js"></script>

    <script type='text/javascript'>
        var map, datasource, layer;

        function initMap() {
            //Initialize a map instance.
            map = new atlas.Map('myMap', {
                view: 'Auto',

                //Add your Azure Maps key to the map SDK. Get an Azure Maps key at https://azure.com/maps. NOTE: The primary key should be used as the key.
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
                atlas.io.read('myKMLFile.kml').then(async r => {
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

<center>

![Azure Maps - kml](media/migrate-bing-maps-web-app/azure-maps-kml.jpg)</center>

**Ressources supplémentaires**

-   [Fonction atlas.io.read](/javascript/api/azure-maps-spatial-io/atlas.io#read-string---arraybuffer---blob--spatialdatareadoptions-)
-   [SimpleDataLayer](/javascript/api/azure-maps-spatial-io/atlas.layer.simpledatalayer)
-   [SimpleDataLayerOptions](/javascript/api/azure-maps-spatial-io/atlas.simpledatalayeroptions)

### <a name="add-drawing-tools"></a>Ajouter des outils de dessin

Bing et Azure Maps fournissent tous deux un module qui ajoute la possibilité pour l’utilisateur de dessiner et de modifier des formes sur la carte à l’aide de la souris ou d’un autre périphérique d’entrée. Tous deux prennent en charge le dessin de punaise, de ligne et de polygones. Azure Maps fournit également des options pour dessiner des cercles et des rectangles.

**Avant : Bing Cartes**

Dans Bing Cartes, le module `DrawingTools` est chargé à l’aide de la fonction `Microsoft.Maps.loadModule`. Une fois le chargement effectué, une instance de la classe DrawingTools peut être créée et la fonction `showDrawingManager` est appelée afin d’ajouter une barre d’outils à la carte.

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <script type='text/javascript'>
    var map, drawingManager;

    function initMap() {
        map = new Microsoft.Maps.Map('#myMap', {
            credentials: '<Your Bing Maps Key>'
        });

        //Load the DrawingTools module
        Microsoft.Maps.loadModule('Microsoft.Maps.DrawingTools', function () {
            //Create an instance of the DrawingTools class and bind it to the map.
            var tools = new Microsoft.Maps.DrawingTools(map);

            //Show the drawing toolbar and enable editting on the map.
            tools.showDrawingManager(function (manager) {
                //Store a reference to the drawing manager as it will be useful later.
                drawingManager = manager;
            });
        });
    }
    </script>

    <!-- Bing Maps Script Reference -->
    <script src="https://www.bing.com/api/maps/mapcontrol?callback=initMap" async defer></script>
</head>
<body>
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>

```

<center>

![Bing Cartes - Outils de dessin](media/migrate-bing-maps-web-app/bing-maps-drawing-tools.jpg)</center>

**Après : Azure Maps**

Dans Azure Maps, le module d’outils de dessin doit être chargé en chargeant le code JavaScript, et les fichiers CSS doivent être référencés dans l’application. Une fois la carte chargée, une instance de la classe `DrawingManager` peut être créée et une instance `DrawingToolbar` jointe.

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

    <!-- Add references to the Azure Maps Map Drawing Tools JavaScript and CSS files. -->
    <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/drawing/0/atlas-drawing.min.css" type="text/css" />
    <script src="https://atlas.microsoft.com/sdk/javascript/drawing/0/atlas-drawing.min.js"></script>
    
    <script type='text/javascript'>
        var map, drawingManager;

        function initMap() {
            //Initialize a map instance.
            map = new atlas.Map('myMap', {
                view: 'Auto',
                
                //Add your Azure Maps key to the map SDK. Get an Azure Maps key at https://azure.com/maps. NOTE: The primary key should be used as the key.
                authOptions: {
                    authType: 'subscriptionKey',
                    subscriptionKey: '<Your Azure Maps Key>'
                }
            });

            //Wait until the map resources are ready.
            map.events.add('ready', function () {

                //Create an instance of the drawing manager and display the drawing toolbar.
                drawingManager = new atlas.drawing.DrawingManager(map, {
                    toolbar: new atlas.control.DrawingToolbar({ position: 'top-left' })
                });
            });
        }
    </script>
</head>
<body onload="initMap()">
    <div id="myMap" style="position:relative;width:600px;height:400px;"></div>
</body>
</html>
```

<center>

![Azure Maps - Outils de dessin](media/migrate-bing-maps-web-app/azure-maps-drawing-tools.jpg)</center>

> [!TIP]
> Dans les couches Azure Maps, les outils de dessin offrent aux utilisateurs plusieurs moyens de dessiner des formes. Par exemple, lorsqu’il dessine un polygone, l’utilisateur peut cliquer pour ajouter chaque point, ou maintenir le bouton gauche de la souris enfoncé et faire glisser la souris pour dessiner un tracé. Cela peut être modifié à l’aide de l’option `interactionType` de `DrawingManager`.

**Ressources supplémentaires**

-   [Documentation](./set-drawing-options.md)
-   [Exemples de code](https://azuremapscodesamples.azurewebsites.net/#Drawing-Tools-Module)

## <a name="next-steps"></a>Étapes suivantes

Jetez un coup d’œil aux [modules du kit SDK web Azure Maps open source](open-source-projects.md#open-web-sdk-modules). Ces modules fournissent une multitude de fonctionnalités supplémentaires et sont entièrement personnalisables.

Passez en revue les exemples de code relatifs à la migration d’autres fonctionnalités Bing Cartes :

**Visualisations de données**

> [!div class="nextstepaction"]
> [Couche de contour](https://azuremapscodesamples.azurewebsites.net/?search=contour)

> [!div class="nextstepaction"]
> [Compartimentage de données](https://azuremapscodesamples.azurewebsites.net/?search=data%20binning)

**Services**

> [!div class="nextstepaction"]
> [Utiliser le module de service Azure Maps](./how-to-use-services-module.md)

> [!div class="nextstepaction"]
> [Rechercher des points d’intérêt](./map-search-location.md)

> [!div class="nextstepaction"]
> [Obtenir des informations à partir d’une coordonnée (géocode inversé)](./map-get-information-from-coordinate.md)

> [!div class="nextstepaction"]
> [Afficher des directions de A à B](./map-route.md)

> [!div class="nextstepaction"]
> [Rechercher la suggestion automatique avec JQuery UI](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Search%20Autosuggest%20and%20JQuery%20UI)

En savoir plus sur le SDK web Azure Maps.

> [!div class="nextstepaction"]
> [Comment utiliser le contrôle de carte](how-to-use-map-control.md)

> [!div class="nextstepaction"]
> [Comment utiliser le module de services](how-to-use-services-module.md)

> [!div class="nextstepaction"]
> [Comment utiliser le module Outils de dessin](set-drawing-options.md)

> [!div class="nextstepaction"]
> [Exemples de code](/samples/browse/?products=azure-maps)

> [!div class="nextstepaction"]
> [Documentation de référence sur l’API de service du kit SDK web Azure Maps](/javascript/api/azure-maps-control/)