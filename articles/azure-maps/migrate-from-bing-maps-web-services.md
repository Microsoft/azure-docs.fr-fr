---
title: 'Tutoriel : Migrer des services web à partir de Bing Cartes | Microsoft Azure Maps'
description: Tutoriel sur la migration des services web de Bing Cartes vers Microsoft Azure Maps.
author: rbrundritt
ms.author: richbrun
ms.date: 12/07/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: ''
ms.openlocfilehash: d257c66de8fb62fb57c573d91966f3e7d8d1b123
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96904956"
---
# <a name="tutorial---migrate-web-service-from-bing-maps"></a>Tutoriel - Migrer des services web à partir de Bing Cartes

Azure Maps et Bing Cartes donnent tous deux accès à des API spatiales par le biais de services web REST. Les interfaces API de ces plateformes offrent des fonctionnalités similaires mais utilisent des conventions d’affectation de noms et des objets de réponse différents.

Le tableau suivant présente les API du service Azure Maps qui offrent des fonctionnalités similaires à celles des API du service Bing Cartes listées.

| API du service Bing Cartes                 | API du service Azure Maps      |
|---------------------------------------|-----------------------------|
| Suggestion automatique                           | [action](/rest/api/maps/search)     |
| Directions (y compris camion)          | [Obtenir des directions](/rest/api/maps/route/getroutedirections)                          |
| Matrice des distances                       | [Matrice d’itinéraire](/rest/api/maps/route/postroutematrixpreview)                          |
| Imagerie – Carte statique                  | [Render](/rest/api/maps/render/getmapimage)                                   |
| Isochrones                            | [Zone d’itinéraire](/rest/api/maps/route/getrouterange)                                    |
| Local Insights                        | [Recherche](/rest/api/maps/search) + [Zone d’itinéraire](/rest/api/maps/route/getrouterange)    |
| Recherche locale                          | [action](/rest/api/maps/search)     |
| Reconnaissance des localisations (POI)           | [action](/rest/api/maps/search)     |
| Localisations (géocodage direct/inversé) | [action](/rest/api/maps/search)                                               |
| Snap to Roads                          | [Publier des directions](/rest/api/maps/route/postroutedirections)                         |
| Services de données spatiales           | [Recherche](/rest/api/maps/search) + [Itinéraires](/rest/api/maps/route) + autres services Azure |
| Time Zone (Fuseau horaire)                             | [Fuseau horaire](/rest/api/maps/timezone)  |
| Incidents de trafic                     | [Détails des incidents de trafic](/rest/api/maps/traffic/gettrafficincidentdetail)                     |
| Elevation                             | [Élévation (préversion)](/rest/api/maps/elevation)

Actuellement, les API de service suivantes ne sont pas disponibles dans Azure Maps :

-   Itinéraires optimisés : planifié. L’API Itinéraire Azure Maps prend en charge l’optimisation pour les commerciaux en déplacement avec un seul véhicule
-   Métadonnées d’imagerie : principalement utilisées pour obtenir des URL de mosaïques dans Bing Cartes. Azure Maps a un service autonome pour accéder directement aux mosaïques

Azure Maps offre plusieurs autres services web REST qui peuvent être utiles :

-   [Azure Maps Creator (préversion)](./creator-indoor-maps.md) : créez un jumeau numérique privé et personnalisé de bâtiments et d’espaces.
-   [opérations spatiales](/rest/api/maps/spatial) : déplacez vers un service les calculs et les opérations spatiales complexes, par exemple le geofencing.
-   [Mosaïques](/rest/api/maps/render/getmaptile) : accédez aux mosaïques d’imagerie et de route à partir d’Azure Maps en tant que mosaïques vectorielles et raster.
-   [Itinéraires par lot](/rest/api/maps/route/postroutedirectionsbatchpreview) : permet d’effectuer jusqu’à 1000 requêtes d’itinéraires dans un lot unique sur une période donnée. Les itinéraires sont calculés en parallèle sur le serveur pour un traitement plus rapide.
-   [Circulation](/rest/api/maps/traffic) : accédez aux données de circulation en temps réel à la fois sous la forme de mosaïques raster et vectorielles.
-   [API de géolocalisation (préversion)](/rest/api/maps/geolocation/getiptolocationpreview) : permet d’obtenir la localisation d’une adresse IP.
-   [Services météo](/rest/api/maps/weather) : permet d’accéder aux données météo en temps réel et aux prévisions.

Veillez également à consulter les guides de bonnes pratiques suivants :

-   [Bonnes pratiques pour la recherche](./how-to-use-best-practices-for-search.md)
-   [Bonne pratiques en matière de routage](./how-to-use-best-practices-for-routing.md)

## <a name="geocoding-addresses"></a>Géocodage d’adresses

Le géocodage est le processus de conversion d’une adresse (par exemple `"1 Microsoft way, Redmond, WA"`) en une coordonnée (par exemple longitude : -122.1298, latitude : 47.64005). Les coordonnées sont alors souvent utilisées pour positionner une punaise sur une carte ou centrer une carte.

Azure Maps propose plusieurs méthodes pour le géocodage des adresses :

-   [Géocodage d’adresses de forme libre](/rest/api/maps/search/getsearchaddress) : spécifiez une chaîne d’adresse unique (comme `"1 Microsoft way, Redmond, WA"`) et traitez immédiatement la requête. Ce service est recommandé si vous avez besoin de géocoder rapidement des adresses individuelles.
-   [Géocodage d’adresses structurées](/rest/api/maps/search/getsearchaddressstructured) : Précisez les parties d’une même adresse, par exemple le nom de la rue, la ville, le pays et le code postal, et traitez aussitôt la requête. Ce service est recommandé si vous avez besoin de géocoder rapidement des adresses individuelles, et que les données sont déjà analysées en leurs parties d’adresse individuelles.
-   [Géocodage des adresses par lots](/rest/api/maps/search/postsearchaddressbatchpreview) : Créez une requête contenant jusqu'à 10 000 adresses, puis traitez-les sur une certaine période. Toutes les adresses seront géocodées en parallèle sur le serveur et, une fois l’opération terminée, l’ensemble complet des résultats pourra être téléchargé. Ce service est recommandé pour le géocodage de grands jeux de données.
-   [Recherche approximative](/rest/api/maps/search/getsearchfuzzy) : Cette API combine le géocodage des adresses et la recherche de points d’intérêt. Cette API prend en compte une chaîne de forme libre qui peut être une adresse, un lieu, un point de repère, un point d'intérêt ou une catégorie de point d'intérêt, puis traite immédiatement la requête. Cette API est recommandée pour les applications où les utilisateurs peuvent rechercher des adresses ou des points d'intérêt à partir d'une même zone de texte.
-   [Recherche approximative par lot](/rest/api/maps/search/postsearchfuzzybatchpreview)  : Créez une requête contenant jusqu'à 10 000 adresses, lieux, points de repère ou centres d'intérêt, puis traitez-les sur une certaine période. Toutes les données seront traitées en parallèle sur le serveur et, une fois l’opération terminée, l’ensemble complet des résultats pourra être téléchargé.

Les tableaux suivants référencent de manière croisée les paramètres de l’API Bing Cartes et les paramètres d’API comparables dans Azure Maps pour le géocodage d’adresses structurées et de forme libre.

**Localisation par adresse (adresse structurée)**

| Paramètre d’API Bing Cartes              | Paramètre d’API Azure Maps comparable                 |
|--------------------------------------|-----------------------------------------------------|
| `addressLine`                      | `streetNumber`, `streetName` ou `crossStreet` |
| `adminDistrict`                    | `countrySubdivision`                              |
| `countryRegion`                    | `country` et `countryCode`                     |
| `locality`                         | `municipality` ou `municipalitySubdivision`     |
| `postalCode`                       | `postalCode`                                      |
| `maxResults` (`maxRes`)          | `limit`                                           |
| `includeNeighborhood` (`inclnb`) | N/A : toujours retourné par Azure Maps si disponible.   |
| `include` (`incl`)               | N/A : le code ISO2 du pays est toujours retourné par Azure Maps. |
| `key`                              | `subscription-key` : voir également la documentation [Authentification avec Azure Maps](./azure-maps-authentication.md). |
| `culture` (`c`)                  | `language` : voir la documentation [Langues prises en charge](./supported-languages.md). |
| `userRegion` (`ur`)              | `view` : voir la documentation sur les [vues prises en charge](./supported-languages.md#azure-maps-supported-views). |

Azure Maps prend également en charge :

-   `countrySecondarySubdivision` : comté, districts
-   `countryTertiarySubdivision` : zones nommées ; boroughs, canton, communes
-   `ofs` : parcourir les résultats en combinaison avec le paramètre `maxResults`.

**Localisation par requête (chaîne d’adresse de forme libre)**

| Paramètre d’API Bing Cartes              | Paramètre d’API Azure Maps comparable      |
|--------------------------------------|------------------------------------------|
| `query`                            | `query`                                |
| `maxResults` (`maxRes`)          | `limit`                                |
| `includeNeighborhood` (`inclnb`) | N/A : toujours retourné par Azure Maps si disponible.  |
| `include` (`incl`)               | N/A : le code ISO2 du pays est toujours retourné par Azure Maps.  |
| `key`                              | `subscription-key` : voir également la documentation [Authentification avec Azure Maps](./azure-maps-authentication.md). |
| `culture` (`c`)                  | `language` : voir la documentation [Langues prises en charge](./supported-languages.md).  |
| `userRegion` (`ur`)              | `view` : voir la documentation sur les [vues prises en charge](./supported-languages.md#azure-maps-supported-views). |

Azure Maps prend également en charge :

-   `typeahead` : indique si la requête sera interprétée en tant qu’entrée partielle et si la recherche passera en mode prédictif (suggestion automatique/autocomplétion).
-   `countrySet` : liste séparée par des virgules de codes de pays ISO2 dans lesquels limiter la recherche.
-   `lat`/`lon`, `topLeft`/`btmRight`, `radius` : spécifiez la localisation et la zone de l’utilisateur pour que les résultats soient plus localement pertinents.
-   `ofs` : parcourir les résultats en combinaison avec le paramètre `maxResults`.

Vous trouverez [ici](./how-to-search-for-address.md) un exemple d’utilisation du service de recherche. Veillez à consulter la documentation [Bonnes pratiques pour la recherche](./how-to-use-best-practices-for-search.md).

## <a name="reverse-geocode-a-coordinate-find-a-location-by-point"></a>Inverser le géocode d’une coordonnée (Rechercher une localisation par point)

Le géocodage inverse est le processus de conversion de coordonnées géographiques (par exemple longitude : -122.1298, la latitude : 47.64005) en son adresse approximative (par exemple `"1 Microsoft way, Redmond, WA"`).

Azure Maps propose plusieurs méthodes de géocodage inverse :

-   [Géocodeur inverse d’adresse](/rest/api/maps/search/getsearchaddressreverse) : Spécifiez une seule coordonnée géographique pour obtenir son adresse approximative et traitez aussitôt la requête.
-   [Géocodeur inverse d’intersection](/rest/api/maps/search/getsearchaddressreversecrossstreet) : Spécifiez une seule coordonnée géographique pour obtenir des informations sur les intersections proches (par exemple, 1re et rue principale) et traitez aussitôt la requête.
-   [Géocodeur inverse d’adresses par lots](/rest/api/maps/search/postsearchaddressreversebatchpreview) : Créez une requête contenant jusqu'à 10 000 coordonnées, puis traitez-les sur une certaine période. Toutes les données seront traitées en parallèle sur le serveur et, une fois l’opération terminée, l’ensemble complet des résultats pourra être téléchargé.

Le tableau suivant référence de manière croisée les paramètres de l’API Bing Cartes et les paramètres d’API comparables dans Azure Maps.

| Paramètre d’API Bing Cartes              | Paramètre d’API Azure Maps comparable       |
|--------------------------------------|-------------------------------------------|
| `point`                              | `query`                                   |
| `includeEntityTypes`                 | `entityType` : voir le tableau de comparaison des types d’entités ci-dessous.    |
| `includeNeighborhood` (`inclnb`)     | N/A : toujours retourné par Azure Maps si disponible.         |
| `include` (`incl`)                   | N/A : le code ISO2 du pays est toujours retourné par Azure Maps.    |
| `key`                                | `subscription-key` : voir également la documentation [Authentification avec Azure Maps](./azure-maps-authentication.md). |
| `culture` (`c`)                      | `language` : voir la documentation [Langues prises en charge](./supported-languages.md).   |
| `userRegion` (`ur`)                  | `view` : voir la documentation sur les [vues prises en charge](./supported-languages.md#azure-maps-supported-views). |

Veillez à consulter la documentation [Bonnes pratiques pour la recherche](./how-to-use-best-practices-for-search.md).

L’API de géocodage inverse d’Azure Maps offre quelques fonctionnalités supplémentaires non disponibles dans Bing Cartes et dont l’intégration pourrait être utile lors de la migration de votre application :

-   Récupérer les données de limitations de vitesse.
-   Récupérer des informations sur la nature des routes : route locale, artère, accès limité, rampe, etc.
-   Côté de la rue de la coordonnée spécifiée.

**Tableau de comparaison des types d’entités**

Le tableau suivant référencent les valeurs de types d’entités Bing Cartes avec les noms de propriétés équivalents dans Azure Maps.

| Type d’entité Bing Cartes | Type d’entité Azure Maps comparable               | Description                                |
|-----------------------|-------------------------------------------------|--------------------------------------------|
| `Address`             |                                                 | *Adresse*                                  |
| `Neighborhood`        | `Neighbourhood`                                 | *Voisinage*                             |
| `PopulatedPlace`      | `Municipality` ou `MunicipalitySubdivision`     | *Cité*, *Ville ou sous-division* ou *Super ville*     |
| `Postcode1`           | `PostalCodeArea`                                | *Code postal*                |
| `AdminDivision1`      | `CountrySubdivision`                            | *État* ou *Province*                      |
| `AdminDivision2`      | `CountrySecondarySubdivison`                    | *Comté* ou *Districts*                    |
| `CountryRegion`       | `Country`                                       | *Nom du pays*                             |
|                       | `CountryTertiarySubdivision`                    | *Boroughs*, *Cantons*, *Communes*          |

## <a name="get-location-suggestions-autosuggest"></a>Obtenir des suggestions de localisation (suggestion automatique)

Plusieurs des API de recherche Azure Maps prennent en charge le mode prédictif, qui peut être utilisé pour les scénarios de suggestion automatique. L’API Azure Maps de [recherche approximative](/rest/api/maps/search/getsearchfuzzy) est celle qui s’apparente le plus à l’API de suggestion automatique de Bing Cartes. Les API suivantes prennent également en charge le mode prédictif ; ajoutez `&typeahead=true` à la requête.

-   [Géocodage d’adresses de forme libre](/rest/api/maps/search/getsearchaddress) : spécifiez une chaîne d’adresse unique (comme `"1 Microsoft way, Redmond, WA"`) et traitez immédiatement la requête. Ce service est recommandé si vous avez besoin de géocoder rapidement des adresses individuelles.
-   [Recherche approximative](/rest/api/maps/search/getsearchfuzzy) : Cette API combine le géocodage des adresses et la recherche de points d’intérêt. Cette API prend en compte une chaîne de forme libre qui peut être une adresse, un lieu, un point de repère, un point d'intérêt ou une catégorie de point d'intérêt, puis traite immédiatement la requête. Cette API est recommandée pour les applications où les utilisateurs peuvent rechercher des adresses ou des points d'intérêt à partir d'une même zone de texte.
-   [Recherche de points d'intérêt](/rest/api/maps/search/getsearchpoi) : Rechercher des points d’intérêt par nom. Par exemple : `"starbucks"`.
-   [Recherche de catégories de points d'intérêt](/rest/api/maps/search/getsearchpoicategory) : Rechercher des points d’intérêt par catégorie. Par exemple : « restaurant ».

## <a name="calculate-routes-and-directions"></a>Calculer des itinéraires et des directions

Azure Maps permet de calculer des itinéraires et des directions. Azure Maps offre de nombreuses fonctionnalités similaires à celles du service d’itinéraire Bing Cartes, par exemple :

-   Heures d’arrivée et de départ
-   Itinéraires en temps réel et selon les prédictions de circulation
-   Différents modes de transport : voiture, marche, camion
-   Optimisation de l’ordre des points de repère (commerciaux itinérants)

> [!NOTE]
> Pour Azure Maps, tous les points de cheminement doivent être des coordonnées. Les adresses devront d’abord être géocodées.

Le service d’itinéraire Azure Maps fournit les API suivantes pour le calcul des itinéraires :

-   [Calculer l’itinéraire](/rest/api/maps/route/getroutedirections) : Calculez un itinéraire et traitez aussitôt la requête. Cette API prend en charge les requêtes GET et POST. Les requêtes POST sont recommandées lors de la spécification d'un grand nombre de points de cheminement ou lors de l'utilisation de nombreuses options de routage pour s'assurer que la requête URL ne devienne pas trop longue et n’entraîne des problèmes.
-   [Itinéraire par lots](/rest/api/maps/route/postroutedirectionsbatchpreview) : Créez une requête contenant jusqu'à 1 000 requêtes d’itinéraire, puis traitez-les sur une certaine période. Toutes les données seront traitées en parallèle sur le serveur et, une fois l’opération terminée, l’ensemble complet des résultats pourra être téléchargé.
-   [Services Mobility (préversion) ](/rest/api/maps/mobility) : Calculez des itinéraires et des directions en utilisant les transports en commun.

Le tableau suivant référence de manière croisée les paramètres de l’API Bing Cartes et les paramètres d’API comparables dans Azure Maps.

| Paramètre d’API Bing Cartes                                    | Paramètre d’API Azure Maps comparable               |
|------------------------------------------------------------|---------------------------------------------------|
| `avoid`                                                    | `avoid`                                           |
| `dateTime` (`dt`)                                          | `departAt` ou `arriveAt`                          |
| `distanceBeforeFirstTurn` (`dbft`)                         | N/A                                               |
| `distanceUnit` (`du`)                                      | N/A : Azure Maps utilise uniquement le système métrique.     |
| `heading` (`hd`)                                           | `vehicleHeading`                                  |
| `maxSolutions` (`maxSolns`)                                | `maxAlternatives`, `alternativeType`, `minDeviationDistance` et `minDeviationTime`  |
| `optimize` (`optwz`)                                       | `routeType` et `traffic`                         |
| `optimizeWaypoints` (`optWp`)                              | `computeBestOrder`                                |
| `routeAttributes` (`ra`)                                   | `instructionsType`                                |
| `routePathOutput` (`rpo`)                                  | `routeRepresentation`                             |
| `timeType` (`tt`)                                          | `departAt` ou `arriveAt`                          |
| `tolerances` (`tl`)                                        | N/A                                               |
| `travelMode`                                               | `travelMode`                                      |
| `waypoint.n` (`wp.n`) ou `viaWaypoint.n` (`vwp.n`)         | `query` – coordonnées au format `lat0,lon0:lat1,lon1….`   |
| `key`                                                      | `subscription-key` : voir également la documentation [Authentification avec Azure Maps](./azure-maps-authentication.md). |
| `culture` (`c`)                                            | `language` : voir la documentation [Langues prises en charge](./supported-languages.md). |
| `userRegion` (`ur`)                                        | `view` : voir la documentation sur les [vues prises en charge](./supported-languages.md#azure-maps-supported-views). |

L’API d’itinéraire Azure Maps prend également en charge les itinéraires des camions au sein de la même API. Le tableau suivant référence de manière croisée les paramètres d’itinéraires des camions Bing Cartes et les paramètres d’API comparables dans Azure Maps.

| Paramètre d’API Bing Cartes                  | Paramètre d’API Azure Maps comparable        |
|------------------------------------------|--------------------------------------------|
| `dimensionUnit` (`dims`)                 | N/A : dimensions en mètres uniquement prises en charge. |
| `weightUnit` (`wu`)                      | N/A : poids en kilogrammes uniquement pris en charge. |
| `vehicleHeight` (`height`)               | `vehicleHeight`                            |
| `vehicleWidth` (`width`)                 | `vehicleWidth`                             |
| `vehicleLength` (`vl`)                   | `vehicleLength`                            |
| `vehicleWeight` (`weight`)               | `vehicleWeight`                            |
| `vehicleAxles` (`axles`)                 | `vehicleAxelWeight`                        |
| `vehicleTrailers` (`vt`)                 | **N/A**                                    |
| `vehicleSemi` (`semi`)                   | `vehicleCommercial`                        |
| `vehicleMaxGradient` (`vmg`)             | **N/A**                                    |
| `vehicleMinTurnRadius` (`vmtr`)          | **N/A**                                    |
| `vehicleAvoidCrossWind` (`vacw`)         | **N/A**                                    |
| `vehicleAvoidGroundingRisk` (`vagr`)     | **N/A**                                    |
| `vehicleHazardousMaterials` (`vhm`)      | `vehicleLoadType`                          |
| `vehicleHazardousPermits` (`vhp`)        | `vehicleLoadType`                          |

> [!TIP]
> Par défaut, l'API de routage Azure Maps ne renvoie qu'un résumé (distance et temps) et les coordonnées de l’itinéraire. Utilisez le paramètre `instructionsType` pour obtenir des instructions de navigation étape par étape. Le paramètre `routeRepresentation` peut être utilisé pour filtrer le résumé et l’itinéraire.

Veillez également à consulter la documentation sur les [bonnes pratiques en matière d’itinéraire](./how-to-use-best-practices-for-routing.md).

L’API d’itinéraire d’Azure Maps offre de nombreuses fonctionnalités supplémentaires non disponibles dans Bing Cartes et dont l’intégration pourrait être utile lors de la migration de votre application :

-   Prise en charge du type d'itinéraire : le plus court, le plus rapide, vers plusieurs destinations simultanément, et le plus économique en carburant.
-   Prise en charge de modes de déplacement supplémentaires : vélo, bus, moto, taxi, camion et camionnette.
-   Prise en charge de 150 points de cheminement.
-   Calculez plusieurs temps de parcours dans une même requête ; trafic historique, trafic en direct, aucun trafic.
-   Autres types de routes à éviter : itinéraires de covoiturage, routes non goudronnées, routes déjà utilisées.
-   Routage basé sur les caractéristiques du moteur. Calculez les itinéraires pour les véhicules à combustion ou électriques en fonction de la réserve de carburant ou d’électricité, et des caractéristiques du moteur.
-   Spécifiez la vitesse maximale du véhicule.

## <a name="snap-coordinates-to-road"></a>Ancrer des coordonnées sur la route

Il existe plusieurs façons d’ancrer des coordonnées sur les routes dans Azure Maps.

-   Utilisez l’API Itinéraires pour ancrer les coordonnées sur un itinéraire logique dans le réseau routier.
-   Utilisez le kit SDK web Azure Maps pour ancrer des coordonnées individuelles sur la route la plus proche dans les mosaïques vectorielles.
-   Utilisez les mosaïques vectorielles Azure Maps directement pour ancrer des coordonnées individuelles.

**Utilisation de l’API Obtenir des directions pour ancrer des coordonnées**

Azure Maps peut ancrer des coordonnées sur des routes à l’aide de l’API [Obtenir des directions](/rest/api/maps/route/postroutedirections). Vous pouvez utiliser ce service pour reconstruire un itinéraire logique entre un ensemble de coordonnées ; il s’apparente à l’API Snap to Road de Bing Cartes.

Il existe deux manières différentes d’utiliser l’API Obtenir des directions pour ancrer des coordonnées sur des routes.

-   S’il y a 150 coordonnées ou moins, elles peuvent être transmises en tant que points de repère dans l’API Obtenir des directions. À l’aide de cette approche, vous pouvez récupérer deux différents types de données ancrées. Les directions d’itinéraire contiendront les points de repère ancrés, tandis que le tracé d’itinéraire aura un ensemble interpolé de coordonnées remplissant le tracé complet entre les coordonnées.
-   S’il y a plus de 150 coordonnées, vous pouvez utiliser l’API Publier des directions. Les coordonnées de début et de fin doivent être transmises dans le paramètre de requête, mais toutes les coordonnées peuvent être transmises dans le paramètre `supportingPoints` dans le corps de la requête POST et mises en forme en tant que collection de points de géométrie GeoJSON. Les seules données ancrées disponibles à l’aide de cette approche seront le tracé d’itinéraire qui est un ensemble interpolé de coordonnées remplissant le tracé complet entre les coordonnées. [Voici un exemple](https://azuremapscodesamples.azurewebsites.net/?sample=Snap%20points%20to%20logical%20route%20path) de cette approche utilisant le module de services dans le kit SDK web Azure Maps.

Le tableau suivant référence de manière croisée les paramètres de l’API Bing Cartes et les paramètres d’API comparables dans Azure Maps.

| Paramètre d’API Bing Cartes    | Paramètre d’API Azure Maps comparable                                 |
|----------------------------|---------------------------------------------------------------------|
| `points`                   | `supportingPoints` : passez ces points dans le corps de la requête POST  |
| `interpolate`              | N/A                                                                 |
| `includeSpeedLimit`        | N/A                                                                 |
| `includeTruckSpeedLimit`   | N/A                                                                 |
| `speedUnit`                | N/A                                                                 |
| `travelMode`               | `travelMode`                                                        |
| `key`                      | `subscription-key` : voir également la documentation [Authentification avec Azure Maps](./azure-maps-authentication.md). |
| `culture` (`c`)            | `language` : voir la documentation [Langues prises en charge](./supported-languages.md).   |
| `userRegion` (`ur`)        | `view` : voir la documentation sur les [vues prises en charge](./supported-languages.md#azure-maps-supported-views).   |

L’API d’itinéraire Azure Maps prend également en charge le paramètre d’itinéraire de camion dans la même API afin de garantir le calcul de tracés itinéraires logiques. Le tableau suivant référence de manière croisée les paramètres d’itinéraires des camions Bing Cartes et les paramètres d’API comparables dans Azure Maps.

| Paramètre d’API Bing Cartes                 | Paramètre d’API Azure Maps comparable        |
|-----------------------------------------|--------------------------------------------|
| `dimensionUnit` (`dims`)                | N/A : dimensions en mètres uniquement prises en charge. |
| `weightUnit` (`wu`)                     | N/A : poids en kilogrammes uniquement pris en charge. |
| `vehicleHeight` (`height`)              | `vehicleHeight`                            |
| `vehicleWidth` (`width`)                | `vehicleWidth`                             |
| `vehicleLength` (`vl`)                  | `vehicleLength`                            |
| `vehicleWeight` (`weight`)              | `vehicleWeight`                            |
| `vehicleAxles` (`axles`)                | `vehicleAxelWeight`                        |
| `vehicleTrailers` (`vt`)                | **N/A**                                    |
| `vehicleSemi` (`semi`)                  | `vehicleCommercial`                        |
| `vehicleMaxGradient` (`vmg`)            | **N/A**                                    |
| `vehicleMinTurnRadius` (`vmtr`)         | **N/A**                                    |
| `vehicleAvoidCrossWind` (`vacw`)        | **N/A**                                    |
| `vehicleAvoidGroundingRisk` (`vagr`)    | **N/A**                                    |
| `vehicleHazardousMaterials` (`vhm`)     | `vehicleLoadType`                          |
| `vehicleHazardousPermits` (`vhp`)       | `vehicleLoadType`                          |

Étant donné que cette approche utilise l’API Obtenir des directions, l’ensemble complet d’options de ce service peut être utilisé pour personnaliser la logique utilisée pour ancrer les coordonnées sur les routes. Par exemple, si vous spécifiez une heure de départ, les données de trafic historiques seront prises en compte.

L’API Obtenir des directions Azure Maps ne retourne pas les données de limite de vitesse à l’heure actuelle. Toutefois, elles peuvent être récupérées à l’aide de l’API de géocodage inverse Azure Maps.

**Utilisation du kit SDK web pour ancrer des coordonnées**

Le kit SDK web Azure Maps utilise des mosaïques vectorielles pour afficher les cartes. Ces mosaïques vectorielles contiennent les informations de géométrie de route brutes, qui peuvent être utilisées pour calculer la route la plus proche d’une coordonnée afin d’effectuer un ancrage simple de coordonnées individuelles. Cela est utile lorsque vous souhaitez que les coordonnées soient visibles sur les routes et que vous utilisez déjà le kit SDK web Azure Maps pour visualiser les données.

Toutefois, cette approche effectue uniquement un ancrage sur les segments de routes qui sont chargées dans la vue cartographique. En cas de zoom arrière au niveau du pays, il se peut qu’il n’y ait pas de données de route ; l’ancrage ne pourra donc pas être effectué. Toutefois, à ce niveau de zoom, un pixel unique peut représenter la surface de plusieurs blocs urbains, si bien que l’ancrage n’est pas nécessaire. Pour remédier à ce problème, la logique d’ancrage peut être appliquée à chaque fois que le déplacement de la carte est terminé. [Voici un exemple de code](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Basic%20snap%20to%20road%20logic) qui illustre cela.

**Utilisation des mosaïques vectorielles Azure Maps directement pour ancrer des coordonnées**

Les mosaïques vectorielles Azure Maps contiennent les données de géométrie de route brutes, qui peuvent être utilisées pour calculer le point le plus proche d’une coordonnée sur une route afin d’effectuer un ancrage de base de coordonnées individuelles. Tous les segments de route sont affichés dans les secteurs au niveau de zoom 15 ; vous devez donc récupérer des mosaïques à partir de là. Vous pouvez ensuite utiliser les [mathématiques de pyramide de mosaïques quadtree](./zoom-levels-and-tile-grid.md) afin de déterminer que des mosaïques sont nécessaires et de les convertir en géométries. À partir de là, vous pouvez utiliser une bibliothèque mathématique spatiale, par exemple [turf js](http://turfjs.org/) ou [NetTopologySuite](https://github.com/NetTopologySuite/NetTopologySuite), pour calculer les segments de ligne les plus proches.

## <a name="retrieve-a-map-image-static-map"></a>Récupérer une image de carte (carte statique)

Azure Maps fournit une API pour le rendu des images de carte statiques avec données superposées. L’API de [rendu des images de carte](/rest/api/maps/render/getmapimagerytile) d’Azure Maps est comparable à l’API de carte statique de Bing Cartes.

> [!NOTE]
> Azure Maps a besoin que le centre, toutes les punaises et les localisations de tracés soient des coordonnées au format `longitude,latitude`, tandis que Bing Cartes utilise le format `latitude,longitude`.</p>
<p>Les adresses devront d’abord être géocodées.

Le tableau suivant référence de manière croisée les paramètres de l’API Bing Cartes et les paramètres d’API comparables dans Azure Maps.

| Paramètre d’API Bing Cartes  | Paramètre d’API Azure Maps comparable            |
|--------------------------|------------------------------------------------|
| `centerPoint`            | `center`                                       |
| `format`                 | `format` : spécifié comme faisant partie du chemin d'accès à l’URL. Actuellement, seul le format PNG est pris en charge.  |
| `heading`                | N/A : Streetside non pris en charge.                |
| `imagerySet`             | `layer` et `style` : voir la documentation [Styles de carte pris en charge](./supported-map-styles.md).   |
| `mapArea` (`ma`)         | `bbox`                                         |
| `mapLayer` (`ml`)        | N/A                                            |
| `mapSize` (`ms`)         | `width` et `height` : peut avoir une taille maximale de 8 192 x 8 192. |
| `declutterPins` (`dcl`)  | N/A                                            |
| `dpi`                    | N/A                                            |
| `drawCurve`              | `path`                                         |
| `mapMetadata`            | N/A                                            |
| `pitch`                  | N/A : Streetside non pris en charge.                |
| `pushpin` (`pp`)         | `pins`                                         |
| `zoomLevel`              | `zoom`                                         |
| `query`                  | N/A : le centre ou cadre englobant doit être utilisé.     |
| `highlightEntity` (`he`) | N/A                                            |
| `style`                  | N/A                                            |
| paramètres d’itinéraire         | N/A                                            |
| `key`                    | `subscription-key` : voir également la documentation [Authentification avec Azure Maps](./azure-maps-authentication.md). |
| `culture` (`c`)          | `language` : voir la documentation [Langues prises en charge](./supported-languages.md).   |
| `userRegion` (`ur`)      | `view` : voir la documentation sur les [vues prises en charge](./supported-languages.md#azure-maps-supported-views). |

> [!NOTE]
> Azure Maps utilise un système de mosaïques dont la taille est le double de celle des mosaïques utilisées dans Bing Cartes. Ainsi, la valeur du niveau de zoom dans Azure Maps apparaîtra un niveau de zoom plus proche dans Azure Maps par rapport à Bing Cartes. Pour compenser cette différence, réduisez d’une unité le niveau de zoom dans les requêtes que vous migrez.

Pour plus d’informations, consultez le [Guide pratique de l’API de rendu des images de carte](./how-to-render-custom-data.md).

En plus de pouvoir générer une image de carte statique, le service de rendu Azure Maps permet également d’accéder directement aux mosaïques au format raster (PNG) et vectoriel :

-   [Mosaïque](/rest/api/maps/render/getmaptile) : récupérez les mosaïques raster (PNG) et vectorielles pour les cartes de base (routes, frontières, fond).
-   [Mosaïque d’imagerie de carte](/rest/api/maps/render/getmapimagerytile) : récupérez les mosaïques d’images aériennes et satellites.

### <a name="pushpin-url-parameter-format-comparison"></a>Comparaison du format du paramètre URL des punaises

**Avant : Bing Cartes**

Dans Bing Cartes, des punaises peuvent être ajoutées à une image de carte statique en utilisant le paramètre `pushpin` dans l’URL. Le paramètre `pushpin` prend une localisation au format `latitude,longitude`, un style d’icône et une étiquette de texte (jusqu’à trois caractères), comme indiqué ci-dessous :

> `&pushpin=latitude,longitude;iconStyle;label`

Des punaises supplémentaires peuvent être ajoutées en ajoutant des paramètres `pushpin` supplémentaires à l’URL avec un ensemble de valeurs différent. Les styles d’icône de punaise sont limités à l’un des styles prédéfinis disponibles dans l’API Bing Cartes.

Par exemple, dans Bing Cartes, une punaise rouge avec l’étiquette « AB » peut être ajoutée à la carte aux coordonnées (longitude : -110, latitude : 45) avec le paramètre d’URL suivant :

> `&pushpin=45,-110;7;AB`

<center>

![Punaise de carte statique Bing Cartes](media/migrate-bing-maps-web-service/bing-maps-static-map-pin.jpg)</center>

**Après : Azure Maps**

Dans Azure Maps, des punaises peuvent également être ajoutées à une image de carte statique en spécifiant le paramètre `pins` dans l’URL. Les punaises dans Azure Maps sont définies en spécifiant un style d’icône et une liste de localisations qui utilisent ce style d’icône. Ces informations sont ensuite transmises au paramètre `pins`, qui peut être spécifié plusieurs fois pour prendre en charge des punaises avec des styles différents.

> `&pins=iconType|pinStyles||pinLocation1|pinLocation2|...`

Des styles supplémentaires peuvent être utilisés en ajoutant des paramètres `pins` supplémentaires à l'URL avec un style et un jeu d’emplacements différents.

En ce qui concerne les localisations de punaises, Azure Maps exige que les coordonnées soient au format `longitude latitude`, tandis que Bing Cartes utilise le format `latitude,longitude`. Notez également qu’**un espace, et non une virgule**, sépare la longitude et la latitude dans Azure Maps.

La valeur `iconType` spécifie le type d’épingle à créer et peut avoir les valeurs suivantes :

-   `default` : icône d’épingle par défaut.
-   `none` : aucune icône n’apparaît, seules les étiquettes seront affichées.
-   `custom` : spécifie une icône personnalisée à utiliser. Une URL pointant vers l'image de l'icône peut être ajoutée à la fin du paramètre `pins`, après les informations d’emplacement de l’épingle.
-   `{udid}` : identifiant unique de données (UDID) pour une icône stockée dans la plateforme de stockage de données Azure Maps.

Les styles d’épingles dans Azure Maps sont ajoutés au format `optionNameValue`, avec plusieurs styles séparés par un caractère barre verticale (`|`), par exemple `iconType|optionName1Value1|optionName2Value2`. Notez que les noms et les valeurs des options ne sont pas séparés. Les noms d’options de style suivants peuvent être utilisés pour appliquer un style à des punaises dans Azure Maps :

-   `al` : spécifie l’opacité (alpha) des punaises. Peut être un nombre compris entre 0 et 1.
-   `an` : spécifie l’ancrage de l’épingle. Les valeurs des pixels x et y sont spécifiées au format `x y`.
-   `co` : couleur de l’épingle. Doit être une couleur hexadécimale 24 bits : `000000` à `FFFFFF`.
-   `la` : spécifie l’ancrage de l’étiquette. Les valeurs des pixels x et y sont spécifiées au format `x y`.
-   `lc` : couleur de l’étiquette. Doit être une couleur hexadécimale 24 bits : `000000` à `FFFFFF`.
-   `ls` : taille de l’étiquette en pixels. Peut être un nombre supérieur à 0.
-   `ro` : valeur en degrés de la rotation de l’icône. Peut être un nombre compris entre -360 et 360.
-   `sc` : valeur de mise à l'échelle de l’icône de l’épingle. Peut être un nombre supérieur à 0.

Des valeurs d’étiquette sont spécifiées pour chaque localisation de punaise, au lieu d’utiliser une seule valeur d’étiquette qui s’applique à toutes les punaises de la liste des localisations. La valeur de l’étiquette peut être une chaîne de plusieurs caractères et être entourée de guillemets simples pour éviter de la confondre avec une valeur de style ou d'emplacement.

Par exemple, dans Azure Maps, une icône rouge (`FF0000`) par défaut avec l'étiquette « Space Needle » positionnée sous (15 50) l'icône aux coordonnées (longitude : -122.349300, latitude : 47.620180) peut être ajoutée avec le paramètre URL suivant :

> `&pins=default|coFF0000|la15 50||'Space Needle'-122.349300 47.620180`

<center>

![Punaise de carte statique Azure Maps](media/migrate-bing-maps-web-service/azure-maps-static-map-pin.jpg)</center>

L’exemple suivant ajoute trois épingles avec les valeurs d'étiquette '1', '2' et '3' :

> `&pins=default||'1'-122 45|'2'-119.5 43.2|'3'-121.67 47.12`

<center>

![Punaises multiples de carte statique Azure Maps](media/migrate-bing-maps-web-service/azure-maps-static-map-multiple-pins.jpg)</center>

### <a name="draw-curve-url-parameter-format-comparison"></a>Comparaison du format du paramètre URL de dessin de courbe

**Avant : Bing Cartes**

Dans Bing Cartes, des lignes et des polygones peuvent être ajoutés à une image de carte statique en utilisant le paramètre `drawCurve` dans l’URL. Le paramètre `drawCurve` prend un type de forme, un type de style et une liste de localisations à afficher sur la carte, comme indiqué ci-dessous :

> `&drawCurve=shapeType,styleType,location1,location2...`

Des styles supplémentaires peuvent être utilisés en ajoutant des paramètres `drawCurve` supplémentaires à l'URL avec un style et un jeu d’emplacements différents.

Les localisations dans Google Maps sont spécifiées au format `latitude1,longitude1_latitude2,longitude2_…`. Elles peuvent également être encodées.

Les types de formes dans Bing Cartes incluent les lignes, les polygones, les cercles et les courbes. Les types de styles incluent la couleur de ligne, l’épaisseur de ligne, la couleur de contour, la couleur de remplissage, l’épaisseur du contour et le rayon circulaire.

Par exemple, dans Bing Cartes, une ligne bleue avec une opacité de 50 % et une épaisseur de quatre pixels peut être ajoutée à la carte entre les coordonnées (longitude : -110, latitude : 45 et longitude : -100, latitude : 50) avec le paramètre d’URL suivant :

`&drawCurve=l,FF000088,4;45,-110_50,-100`

<center>

![Ligne de carte statique Bing Cartes](media/migrate-bing-maps-web-service/bing-maps-static-map-line.jpg)</center>

**Après : Azure Maps**

Dans Azure Maps, des lignes et des polygones peuvent également être ajoutés à une image de carte statique en spécifiant le paramètre *path* dans l’URL. Comme dans Bing Cartes, un style et une liste de localisations peuvent être spécifiés dans ce paramètre, et le paramètre *path* peut être spécifié plusieurs fois pour afficher plusieurs cercles, lignes et polygones de différents styles.

> `&path=pathStyles||pathLocation1|pathLocation2|...`

En ce qui concerne les localisations de tracés, Azure Maps exige que les coordonnées soient au format `longitude latitude`, tandis que Bing Cartes utilise le format `latitude,longitude`. Notez également qu’**un espace, et non une virgule**, sépare la longitude et la latitude dans Azure Maps. Actuellement, Azure Maps ne prend pas en charge les tracés encodés. De plus grands jeux de données peuvent être téléchargés lorsqu'un GeoJSON remplit l'API de stockage de données Azure Maps, comme documenté [ici](./how-to-render-custom-data.md#get-data-from-azure-maps-data-storage).

Les styles de chemins d'accès dans Azure Maps sont ajoutés au format `optionNameValue`, avec plusieurs styles séparés par un caractère barre verticale (`|`), par exemple `optionName1Value1|optionName2Value2`. Notez que les noms et les valeurs des options ne sont pas séparés. Les noms d’options de style suivants peuvent être utilisés pour appliquer un style à des chemins d'accès dans Azure Maps :

-   `fa` : opacité de la couleur de remplissage (alpha) utilisée lors du rendu des polygones. Peut être un nombre compris entre 0 et 1.
-   `fc` : couleur de remplissage utilisée pour afficher l’aire d’un polygone.
-   `la` : opacité de la couleur des lignes (alpha) utilisée lors du rendu des lignes et du contour des polygones. Peut être un nombre compris entre 0 et 1.
-   `lc` : couleur de ligne utilisée pour rendre les lignes et le contour des polygones.
-   `lw` : largeur de la ligne en pixels.
-   `ra` : spécifie un rayon de cercle en mètres.

Par exemple, dans Azure Maps, une ligne bleue avec une opacité de 50 % et une épaisseur de quatre pixels peut être ajoutée à la carte entre les coordonnées (longitude : -110, latitude : 45 et longitude : -100, latitude : 50) avec le paramètre d’URL suivant :

> `&path=lc0000FF|la.5|lw4||-110 45|-100 50`

<center>

![Ligne de carte statique Azure Maps](media/migrate-bing-maps-web-service/azure-maps-static-map-line.jpg)</center>

## <a name="calculate-a-distance-matrix"></a>Calculer une matrice des distances

Azure Maps fournit une API pour le calcul des durées de déplacement et des distances entre un jeu d'emplacements sous forme de matrice des distances. L’API de matrice des distances Azure Maps est comparable à l’API de matrice des distances Bing Cartes :

-   [Matrice d’itinéraire](/rest/api/maps/route/postroutematrixpreview) : Calcule de manière asynchrone les durées de déplacement et les distances pour un jeu d'origines et de destinations. Jusqu'à 700 cellules par requête sont prises en charge (le nombre d'origines multiplié par le nombre de destinations). En gardant cette contrainte à l’esprit, voici des exemples de dimensions matricielles possibles : `700x1`, `50x10`, `10x10`, `28x25`, `10x70`.

> [!NOTE]
> Une requête à l'API de matrice des distances ne peut être faite qu'en utilisant une requête POST avec les informations d'origine et de destination dans le corps de la requête.</p>
<p>De plus, Azure Maps exige que toutes les origines et destinations soient des coordonnées. Les adresses devront d’abord être géocodées.

Le tableau suivant référence de manière croisée les paramètres de l’API Bing Cartes et les paramètres d’API comparables dans Azure Maps.

| Paramètre d’API Bing Cartes | Paramètre d’API Azure Maps comparable                         |
|-------------------------|-------------------------------------------------------------|
| `origins`               | `origins` : spécifiez GeoJSON dans le corps de la requête POST.    |
| `destinations`          | `destination` : spécifiez GeoJSON dans le corps de la requête POST. |
| `endTime`               | `arriveAt`                                                  |
| `startTime`             | `departAt`                                                  |
| `travelMode`            | `travelMode`                                                |
| `resolution`            | N/A                                                         |
| `distanceUnit`          | N/A : toutes les distances sont en mètres.                              |
| `timeUnit`              | N/A : toutes les heures sont en secondes.                                 |
| `key`                   | `subscription-key` : voir également la documentation [Authentification avec Azure Maps](./azure-maps-authentication.md). |
| `culture` (`c`)         | `language` : voir la documentation [Langues prises en charge](./supported-languages.md).  |
| `userRegion` (`ur`)     | `view` : voir la documentation sur les [vues prises en charge](./supported-languages.md#azure-maps-supported-views).     |

> [!TIP]
> Toutes les options de routage avancées disponibles dans l'API de routage Azure Maps (routage de camions, caractéristiques des moteurs, éléments à éviter...) sont prises en charge dans l'API de matrice des distances Azure Maps.

## <a name="calculate-an-isochrone"></a>Calculer un isochrone

Azure Maps fournit une API pour calculer un isochrone, un polygone couvrant une zone qui peut être parcourue dans n’importe quelle direction à partir d’un point d’origine selon un intervalle de temps ou un montant de frais/carburant spécifié. L’API Zone d’itinéraire Azure Maps est comparable à l’API d’isochrone dans Bing Cartes.

-   Zone d’[itinéraire](/rest/api/maps/route/getrouterange) ** : calculer un polygone couvrant une zone qui peut être parcourue dans n’importe quelle direction à partir d’un point d’origine selon un intervalle de temps, une distance ou un montant de frais/carburant spécifié disponible.

> [!NOTE]
> Azure Maps exige que l’origine de la requête soit une coordonnée. Les adresses devront d’abord être géocodées.</p>
<p>De plus, Bing Cartes peut calculer des isochrones en fonction de l’heure ou de la distance, tandis qu’Azure Maps peut calculer des isochrones en fonction de l’heure, de la distance ou du montant de frais/carburant disponible.

Le tableau suivant référence de manière croisée les paramètres de l’API Bing Cartes et les paramètres d’API comparables dans Azure Maps.

| Paramètre d’API Bing Cartes      | Paramètre d’API Azure Maps comparable            |
|------------------------------|------------------------------------------------|
| `waypoint` (`wp`)            | `query`                                        |
| `dateTime` (`dt`)            | `departAt`                                     |
| `maxTime`                    | `timeBudgetInSec`                              |
| `timeUnit` (`tu`)            | N/A : toutes les heures sont en secondes.                    |
| `travelMode` (`mode`)        | `travelMode`                                   |
| `maxDistance` (`maxDis`)     | `distanceBudgetInMeters`                       |
| `distanceUnit` (`du`)        | N/A : toutes les distances sont en mètres.                 |
| `optimize` (`optmz`)         | `routeType`                                    |
| `key`                        | `subscription-key` : voir également la documentation [Authentification avec Azure Maps](./azure-maps-authentication.md). |
| `culture` (`c`)              | `language` : voir la documentation [Langues prises en charge](./supported-languages.md).  |
| `userRegion` (`ur`)          | `view` : voir la documentation sur les [vues prises en charge](./supported-languages.md#azure-maps-supported-views). |

> [!TIP]
> Toutes les options d’itinéraire avancées disponibles dans l’API d’itinéraire Azure Maps (routage de camions, caractéristiques des moteurs, éléments à éviter...) sont prises en charge dans l’API d’isochrone Azure Maps.

## <a name="search-for-points-of-interest"></a>Rechercher des points d’intérêt

Vous pouvez rechercher des données de point d’intérêt dans Bing Cartes à l’aide des API suivantes :

-   **Recherche locale :** recherche les points d’intérêt proches (recherche radiale), par nom ou par type d’entité (catégorie). Les API de [recherche de points d’intérêt](/rest/api/maps/search/getsearchpoi) et de [recherche de catégorie de points d’intérêt](/rest/api/maps/search/getsearchpoicategory) Azure Maps sont celles qui s’apparentent le plus à cette API.
-   **Reconnaissance des localisations** : recherche les points d'intérêt se trouvant à une certaine distance d'un endroit. L’API de [recherche de proximité](/rest/api/maps/search/getsearchnearby) Azure Maps est celle qui s’apparente le plus à cette API.
-   **Local Insights** : recherche les points d’intérêt qui se trouvent dans le temps de conduite ou la distance maximal(e) spécifié(e) par rapport à une coordonnée spécifique. Cela est possible avec Azure Maps en calculant d’abord un isochrone, puis en le transmettant dans l’API de [recherche dans une géométrie](/rest/api/maps/search/postsearchinsidegeometry).

Azure Maps fournit plusieurs API de recherche de points d'intérêt :

-   [Recherche de points d'intérêt](/rest/api/maps/search/getsearchpoi) : Rechercher des points d’intérêt par nom. Par exemple : `"starbucks"`.
-   [Recherche de catégories de points d'intérêt](/rest/api/maps/search/getsearchpoicategory) : Rechercher des points d’intérêt par catégorie. Par exemple : « restaurant ».
-   [Recherche à proximité](/rest/api/maps/search/getsearchnearby) : Recherche des points d'intérêt se trouvant à une certaine distance d'un endroit.
-   [Recherche approximative](/rest/api/maps/search/getsearchfuzzy) : Cette API combine le géocodage des adresses et la recherche de points d’intérêt. Cette API prend en compte une chaîne de forme libre qui peut être une adresse, un lieu, un point de repère, un point d'intérêt ou une catégorie de point d'intérêt, puis traite immédiatement la requête. Cette API est recommandée pour les applications où les utilisateurs peuvent rechercher des adresses ou des points d'intérêt à partir d'une même zone de texte.
-   [Rechercher dans une géométrie](/rest/api/maps/search/postsearchinsidegeometry) : recherche de points d'intérêt dans une géométrie spécifiée (polygone).
-   [Recherche le long d’un itinéraire](/rest/api/maps/search/postsearchalongroute) : Recherche de points d'intérêt qui se trouvent le long d'un itinéraire spécifié.
-   [Recherche approximative par lot](/rest/api/maps/search/postsearchfuzzybatchpreview)  : Créez une requête contenant jusqu'à 10 000 adresses, lieux, points de repère ou centres d'intérêt, puis traitez-les sur une certaine période. Toutes les données seront traitées en parallèle sur le serveur et, une fois l’opération terminée, l’ensemble complet des résultats pourra être téléchargé.

Veillez à consulter la documentation [Bonnes pratiques pour la recherche](./how-to-use-best-practices-for-search.md).

## <a name="get-traffic-incidents"></a>Obtenir les incidents de trafic

Azure Maps fournit plusieurs API pour la récupération des données de trafic. Deux types de données de trafic sont disponibles :

-   **Données de circulation** : elles fournissent des métriques concernant la circulation sur des sections de route. Elles sont souvent utilisées pour coder les routes par couleur. Ces données sont mises à jour toutes les deux minutes.
-   **Données d’incident** : fournit des données sur la construction, les fermetures de route, les accidents et autres incidents susceptibles d’affecter le trafic. Ces données sont mises à jour toutes les minutes.

Bing Cartes fournit des données de circulation et d’incident dans ses contrôles de carte interactifs, et rend également les données d’incident disponibles en tant que service.

Les données de trafic sont également intégrées dans les contrôles de carte interactifs Azure Maps. Azure Maps fournit également les API de services de trafic suivantes :

-   [Segments de circulation](/rest/api/maps/traffic/gettrafficflowsegment) : fournit des informations sur les vitesses et les temps de trajet du fragment routier le plus proche des coordonnées données.
-   [Mosaïques de circulation](/rest/api/maps/traffic/gettrafficflowtile) : fournit des mosaïques raster et vectorielles contenant des données de circulation. Elles peuvent être utilisées avec les contrôles Azure Maps ou dans des contrôles de carte tiers, tels que Leaflet. Les mosaïques vectorielles peuvent également être utilisées pour l’analyse avancée des données.
-   [Détails des incidents de trafic](/rest/api/maps/traffic/gettrafficincidentdetail) : fournit des détails sur les incidents de trafic qui se trouvent dans les limites d’un cadre englobant, d’un niveau de zoom et d’un modèle de trafic.
-   [Mosaïques d’incident de trafic](/rest/api/maps/traffic/gettrafficincidenttile) : fournit des mosaïques raster et vectorielles contenant des données sur les incidents de trafic.
-   [Fenêtre d’affichage des incidents de trafic](/rest/api/maps/traffic/gettrafficincidentviewport) : récupère les informations juridiques et techniques pour la fenêtre d’affichage décrite dans la requête, telles que l’ID du modèle de trafic.

Le tableau suivant référence de manière croisée les paramètres de l’API de trafic Bing Cartes et les paramètres d’API de détails des incidents de trafic comparables dans Azure Maps.

| Paramètre d’API Bing Cartes  | Paramètre d’API Azure Maps comparable   |
|--------------------------|---------------------------------------|
| `mapArea`                | `boundingBox` et `boundingZoom`      |
| `includeLocationCodes`   | N/A                                   |
| `severity` (`s`)         | N/A : toutes les données sont retournées               |
| `type` (`t`)             | N/A : toutes les données sont retournées               |
| `key`                    | `subscription-key` : voir également la documentation [Authentification avec Azure Maps](./azure-maps-authentication.md). |
| `culture` (`c`)          | `language` : voir la documentation [Langues prises en charge](./supported-languages.md). |
| `userRegion` (`ur`)      | `view` : voir la documentation sur les [vues prises en charge](./supported-languages.md#azure-maps-supported-views). |

## <a name="get-a-time-zone"></a>Obtenir un fuseau horaire

Azure Maps fournit une API pour obtenir le fuseau horaire dans lequel se trouve une coordonnée. L’API de fuseau horaire d’Azure Maps est comparable à l’API de fuseau horaire de Bing Cartes :

-   [Fuseau horaire par coordonnée](/rest/api/maps/timezone/gettimezonebycoordinates) : Spécifiez une coordonnée et obtenez les détails du fuseau horaire dans lequel elle se trouve.

Le tableau suivant référence de manière croisée les paramètres de l’API Bing Cartes et les paramètres d’API comparables dans Azure Maps.

| Paramètre d’API Bing Cartes | Paramètre d’API Azure Maps comparable          |
|-------------------------|----------------------------------------------|
| `point`                 | `query`                                      |
| `query`                 | N/A : les localisations doivent d’abord être géocodées.      |
| `dateTime`              | `timeStamp`                                  |
| `includeDstRules`       | N/A : toujours inclus dans la réponse par Azure Maps. |
| `key`                   | `subscription-key` : voir également la documentation [Authentification avec Azure Maps](./azure-maps-authentication.md). |
| `culture` (`c`)         | `language` : voir la documentation [Langues prises en charge](./supported-languages.md).  |
| `userRegion` (`ur`)     | `view` : voir la documentation sur les [vues prises en charge](./supported-languages.md#azure-maps-supported-views).  |

Par ailleurs, la plateforme Azure Maps fournit également un certain nombre d’API de fuseau horaire supplémentaires pour faciliter les conversions avec les noms de fuseaux horaires et les ID :

-   [Fuseau horaire par ID](/rest/api/maps/timezone/gettimezonebyid) : Renvoie les informations de fuseau horaire actuelles, historiques et futures pour l'ID de fuseau horaire IANA spécifié.
-   [Fuseau horaire avec Enum IANA](/rest/api/maps/timezone/gettimezoneenumiana) : Renvoie une liste complète des ID de fuseau horaire IANA. Les mises à jour du service IANA sont reflétées dans le système dans un délai de 24 heures. 
-   [Fuseau horaire avec Enum Windows](/rest/api/maps/timezone/gettimezoneenumwindows) : Renvoie une liste complète des ID de fuseau horaire Windows.
-   [Fuseau horaire avec version IANA](/rest/api/maps/timezone/gettimezoneianaversion) : Renvoie le numéro de la version actuelle IANA utilisée par Azure Maps. 
-   [Fuseau horaire Windows à IANA](/rest/api/maps/timezone/gettimezonewindowstoiana) : Renvoie un ID IANA correspondant, avec un ID de fuseau horaire Windows valide. Plusieurs ID IANA peuvent être renvoyés pour un seul ID Windows.

## <a name="spatial-data-services-sds"></a>Services de données spatiales

Les services de données spatiales dans Bing Cartes offrent trois fonctionnalités clés :

-   Géocodage par lot : traitez un grand lot de géocodes d’adresses en une seule requête.
-   Récupérer les données de frontières administratives : utilisez une coordonnée et obtenez une frontière d’intersection pour un type d’entité spécifié.
-   Héberger et interroger des données métier spatiales : chargez une table de données 2D simple et accédez-y à l’aide de quelques requêtes spatiales simples.

### <a name="batch-geocode-data"></a>Données de géocodes par lot

Le géocodage par lot est le processus qui consiste à traiter un grand nombre d’adresses ou de lieux, à les transmettre en une seule requête à un service, à faire en sorte qu’elles soient géocodées en parallèle et que les résultats soient retournés dans une réponse unique.

Bing Cartes permet de transmettre jusqu’à 200 000 adresses dans une requête unique de géocodage par lot. Cette requête est envoyée dans une file d’attente et est généralement exécutée sur une période de temps, entre quelques minutes et quelques heures, en fonction de la taille du jeu de données et de la charge sur le service. Chaque adresse de la requête a généré une transaction.

Azure Maps dispose d’un service de géocodage par lot, mais il permet de transmettre jusqu’à 10 000 adresses en une requête unique et sont traitement prend de quelques secondes à quelques minutes, en fonction de la taille du jeu de données et de la charge sur le service. Chaque adresse de la requête a généré une transaction. Dans Azure Maps, le service de géocodage par lot est uniquement disponible au niveau S1.

Une autre option pour géocoder un grand nombre d’adresses avec Azure Maps consiste à effectuer des requêtes parallèles aux API de recherche standard. Ces services acceptent une seule adresse par requête, mais peuvent être utilisés avec le niveau S0 qui fournit également des limites d’utilisation gratuites. Le niveau S0 autorise jusqu’à 50 requêtes par seconde à la plateforme Azure Maps à partir d’un compte unique. Ainsi, si votre traitement ne dépasse pas cette limite, il est possible de géocoder jusqu’à 180 000 adresses par heure. Le niveau S1 n’a pas de limite documentée quant au nombre de requêtes par seconde qui peuvent être effectuées à partir d’un compte. Par conséquent, beaucoup plus de données peuvent être traitées plus rapidement lors de l’utilisation de ce niveau tarifaire. Toutefois, l’utilisation du service de géocodage par lot aide à réduire la quantité totale de données transférées et réduit considérablement le trafic réseau.

-   [Géocodage d’adresses de forme libre](/rest/api/maps/search/getsearchaddress) : spécifiez une chaîne d’adresse unique (comme `"1 Microsoft way, Redmond, WA"`) et traitez immédiatement la requête. Ce service est recommandé si vous avez besoin de géocoder rapidement des adresses individuelles.
-   [Géocodage d’adresses structurées](/rest/api/maps/search/getsearchaddressstructured) : Précisez les parties d’une même adresse, par exemple le nom de la rue, la ville, le pays et le code postal, et traitez aussitôt la requête. Ce service est recommandé si vous avez besoin de géocoder rapidement des adresses individuelles, et que les données sont déjà analysées en leurs parties d’adresse individuelles.
-   [Géocodage des adresses par lots](/rest/api/maps/search/postsearchaddressbatchpreview) : Créez une requête contenant jusqu'à 10 000 adresses, puis traitez-les sur une certaine période. Toutes les adresses seront géocodées en parallèle sur le serveur et, une fois l’opération terminée, l’ensemble complet des résultats pourra être téléchargé. Ce service est recommandé pour le géocodage de grands jeux de données.
-   [Recherche approximative](/rest/api/maps/search/getsearchfuzzy) : Cette API combine le géocodage des adresses et la recherche de points d’intérêt. Cette API prend en compte une chaîne de forme libre qui peut être une adresse, un lieu, un point de repère, un point d'intérêt ou une catégorie de point d'intérêt, puis traite immédiatement la requête. Cette API est recommandée pour les applications où les utilisateurs peuvent rechercher des adresses ou des points d'intérêt à partir d'une même zone de texte.
-   [Recherche approximative par lot](/rest/api/maps/search/postsearchfuzzybatchpreview)  : Créez une requête contenant jusqu'à 10 000 adresses, lieux, points de repère ou centres d'intérêt, puis traitez-les sur une certaine période. Toutes les données seront traitées en parallèle sur le serveur et, une fois l’opération terminée, l’ensemble complet des résultats pourra être téléchargé.

### <a name="get-administrative-boundary-data"></a>Obtenir des données de frontière administrative

Dans Bing Cartes, les frontières administratives des pays, états, comtés, villes et codes postaux sont accessibles par le biais de l’API de géodonnées. Cette API prend une coordonnée ou une requête à géocoder. Si une requête est transmise, elle est géocodée et les coordonnées du premier résultat sont utilisées. Cette API prend les coordonnées et récupère la frontière du type d’entité spécifié qui croise la coordonnée. Notez que cette API ne retourne pas nécessairement la frontière pour la requête transmise. Si une requête pour `"Seattle, WA"` est transmise, mais que la valeur de type d’entité est définie sur la région du pays, la frontière des USA est retournée.

Azure Maps permet également d’accéder aux frontières administratives (pays, états, comtés, villes et codes postaux). Pour récupérer une frontière, vous devez interroger l’une des API de recherche pour la frontière de votre choix (par exemple, `Seattle, WA`). Si le résultat de la recherche est associé à une frontière, un ID de géométrie est fourni dans la réponse du résultat. L’API de polygone de recherche peut ensuite être utilisée pour récupérer les frontières exactes d’un ou plusieurs ID de géométrie. Cela diffère quelque peu de Bing Cartes, car Azure Maps retourne la frontière de ce qui a été recherché, tandis que Bing Cartes retourne une frontière pour un type d’entité spécifié à une coordonnée spécifiée. En outre, les données de frontières retournées par Azure Maps sont au format GeoJSON.

Pour récapituler :

1.  Transmettez une requête pour la frontière que vous souhaitez recevoir dans l’une des API de recherche suivantes.
    -   [Géocodage d’adresses de forme libre](/rest/api/maps/search/getsearchaddress)
    -   [Géocodage d’adresses structurées](/rest/api/maps/search/getsearchaddressstructured)
    -   [Géocodage d’adresses par lots](/rest/api/maps/search/postsearchaddressbatchpreview)
    -   [Recherche partielle](/rest/api/maps/search/getsearchfuzzy)
    -   [Recherche approximative par lot](/rest/api/maps/search/postsearchfuzzybatchpreview)
2.  Si le ou les résultats souhaités ont un ou plusieurs ID géométriques, transmettez-les dans l’[API de polygone de recherche](/rest/api/maps/search/getsearchpolygon).

### <a name="host-and-query-spatial-business-data"></a>Héberger et interroger des données métier spatiales

Les services de données spatiales de Bing Cartes fournissent une solution de stockage de données spatiales simple pour l’hébergement de données métier et leur exposition en tant que service REST spatial. Ce service fournit quatre requêtes principales : rechercher par propriété, rechercher à proximité, rechercher dans le cadre englobant et rechercher avec un mile d’itinéraire. De nombreuses entreprises qui utilisent ce service ont déjà leurs données métier stockées dans une base de données quelque part, et chargent un petit sous-ensemble de ce service pour alimenter des applications telles que des localisateurs de magasins. Étant donné que l’authentification basée sur les clés fournit une sécurité de base, il est recommandé d’utiliser ce service uniquement avec des données publiques.

La plupart des données de localisation d’entreprise ont pour origine une base de données. Par conséquent, il est recommandé d’utiliser des solutions de stockage Azure existantes, telles qu’Azure SQL ou Azure PostgreSQL (avec le plug-in PostGIS). Ces deux solutions de stockage prennent en charge les données spatiales et fournissent un ensemble complet de fonctionnalités d’interrogation spatiale. Une fois que vos données se trouvent dans une solution de stockage appropriée, vous pouvez les intégrer à votre application en créant un service web personnalisé ou à l’aide d’un framework comme ASP.NET ou Entity Framework. Cette approche offre davantage de fonctionnalités d’interrogation ainsi que des options de sécurité beaucoup plus poussées.

Azure Cosmos DB fournit également un ensemble limité de capacités spatiales qui, en fonction de votre scénario, peuvent suffire.

Voici quelques ressources utiles concernant l’hébergement et l’interrogation des données spatiales dans Azure.

-   [Présentation des types de données spatiales Azure SQL](/sql/relational-databases/spatial/spatial-data-types-overview)
-   [Azure SQL – Interroger des données spatiales au sujet du plus proche voisin](/sql/relational-databases/spatial/query-spatial-data-for-nearest-neighbor)
-   [Présentation des fonctionnalités géospatiales Azure Cosmos DB](../cosmos-db/sql-query-geospatial-intro.md)

## <a name="client-libraries"></a>Bibliothèques clientes

Azure Maps fournit des bibliothèques de client pour les langages de programmation suivants :

-   JavaScript, TypeScript, Node.js : [documentation](./how-to-use-services-module.md) \| [Package NPM](https://www.npmjs.com/package/azure-maps-rest)

Bibliothèques de client open source pour d’autres langages de programmation :

-   .NET Standard 2.0 : [projet GitHub](https://github.com/perfahlen/AzureMapsRestServices) \| [package NuGet](https://www.nuget.org/packages/AzureMapsRestToolkit/)

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les services REST Azure Maps.

> [!div class="nextstepaction"]
> [Meilleures pratiques pour l'utilisation du service de recherche](how-to-use-best-practices-for-search.md)

> [!div class="nextstepaction"]
> [Bonnes pratiques en matière d’utilisation du service de routage](how-to-use-best-practices-for-search.md)

> [!div class="nextstepaction"]
> [Comment utiliser le module de services (SDK web)](how-to-use-best-practices-for-routing.md)

> [!div class="nextstepaction"]
> [Documentation de référence sur l’API de service REST Azure Maps](/rest/api/maps/)

> [!div class="nextstepaction"]
> [Exemples de code](/samples/browse/?products=azure-maps)
