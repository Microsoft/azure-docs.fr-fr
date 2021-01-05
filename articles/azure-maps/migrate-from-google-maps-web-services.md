---
title: Tutoriel – Migrer des services web à partir de Google Maps | Microsoft Azure Maps
description: Didacticiel sur la migration de services web de Google Maps vers Microsoft Azure Maps
author: rbrundritt
ms.author: richbrun
ms.date: 08/19/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: ''
ms.openlocfilehash: 813cb567ab3edddd6fb37cee050dc5e38ee4289f
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96904888"
---
# <a name="tutorial---migrate-web-service-from-google-maps"></a>Tutoriel – Migrer un service web à partir de Google Maps

Azure Maps et Google Maps donnent tous deux accès à des API spatiales via des services web REST. Les interfaces API de ces plateformes offrent des fonctionnalités similaires, mais elles utilisent chacune des conventions de nommage et des objets de réponse différents.

Dans ce didacticiel, vous apprendrez à :

> * Transférer et inverser le géocodage
> * Rechercher des points d’intérêt
> * Calculer des itinéraires et des directions
> * Récupérer une image de carte
> * Calculer une matrice des distances
> * Obtenir les détails d’un fuseau horaire

Vous allez également apprendre : 

> [!div class="checklist"]
> * Quel service REST Azure Maps utiliser lors de la migration à partir d’un service web Google Maps
> * Des conseils pour tirer le meilleur parti des services Azure Maps
> * Des insights dans d’autres services Azure Maps associés

Le tableau suivant présente les API du service Azure Maps, qui offrent des fonctionnalités similaires à celles des API du service Google Maps listées.

| API du service Google Maps | API du service Azure Maps                                                                      |
|-------------------------|---------------------------------------------------------------------------------------------|
| Directions              | [Itinéraire](/rest/api/maps/route)                                     |                         
| Matrice des distances         | [Matrice d’itinéraire](/rest/api/maps/route/postroutematrixpreview)       |                         
| Géocodage               | [action](/rest/api/maps/search)                                   |                         
| Recherche de lieux           | [action](/rest/api/maps/search)                                   |                         
| Autocomplétion des lieux      | [action](/rest/api/maps/search)                                   |                         
| Snap to Roads            | Consultez la section [Calculer des itinéraires et des directions](#calculate-routes-and-directions).            
| Speed Limits            | Consultez la section [Géocodage inverse d’une coordonnée](#reverse-geocode-a-coordinate).                  
| Carte statique              | [Render](/rest/api/maps/render/getmapimage)                       |                         
| Time Zone (Fuseau horaire)               | [Fuseau horaire](/rest/api/maps/timezone)                              |                         
| Elevation               | [Élévation (préversion)](/rest/api/maps/elevation)                   |                         |

Actuellement, les API de service suivantes ne sont pas disponibles dans Azure Maps :

- Géolocalisation
- Places details and photos (détails et photos des lieux) : les numéros de téléphone et URL de site web sont disponibles dans l’API de recherche Azure Maps.
- URL de cartes
- Nearest Roads (routes les plus proches) : cette fonctionnalité est proposée par le SDK web comme indiqué [ici](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Basic%20snap%20to%20road%20logic
) mais n’est pas disponible comme service pour l’instant.
- Affichage des rues statiques

Azure Maps offre plusieurs autres services web REST qui peuvent être utiles :

- [Opérations spatiales](/rest/api/maps/spatial) : déchargez vers un service les calculs et les opérations spatiales complexes, par exemple le geofencing.
- [Trafic](/rest/api/maps/traffic) : accédez en temps réel aux données sur le trafic et les incidents.

## <a name="prerequisites"></a>Prérequis 

1. Connectez-vous au [portail Azure](https://portal.azure.com). Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
2. [Créer un compte Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account)
3. [Obtenir une clé d’abonnement principale](quick-demo-map-app.md#get-the-primary-key-for-your-account), également appelée clé primaire ou clé d’abonnement. Pour plus d’informations sur l’authentification dans Azure Maps, voir [Gérer l’authentification dans Azure Maps](how-to-manage-authentication.md).

## <a name="geocoding-addresses"></a>Géocodage d’adresses

Le géocodage est le processus de conversion d’une adresse en coordonnée. Par exemple, « 1 Microsoft Way, Redmond, WA » est convertie en longitude : -122.1298, latitude : 47.64005. Les coordonnées peuvent ensuite être utilisées pour différents types d’opérations, telles que le positionnement ou le centrage d’un marqueur sur une carte.

Azure Maps propose plusieurs méthodes pour le géocodage des adresses :

- [**Géocodage d’adresses en forme libre**](/rest/api/maps/search/getsearchaddress) : Spécifiez une seule chaîne d’adresse et traitez immédiatement la requête. « 1 Microsoft Way, Redmond, WA » est un exemple de chaîne d’adresse unique. Cette API est recommandée si vous avez besoin de géocoder rapidement des adresses individuelles.
- [**Géocodage d’adresses structurées**](/rest/api/maps/search/getsearchaddressstructured) : Précisez les parties d’une même adresse, par exemple le nom de la rue, la ville, le pays/la région et le code postal, et traitez aussitôt la requête. Cette API est recommandée si vous avez besoin de géocoder rapidement des adresses individuelles, et que les données sont déjà analysées dans leurs parties d’adresse individuelles.
- [**Géocodage des adresses par lots**](/rest/api/maps/search/postsearchaddressbatchpreview) : Créez une requête contenant jusqu'à 10 000 adresses, puis traitez-les sur une certaine période. Toutes les adresses seront géocodées en parallèle sur le serveur et, une fois l’opération terminée, l’ensemble complet des résultats pourra être téléchargé. Ceci est recommandé pour le géocodage de grands groupes de données.
- [**Recherche approximative**](/rest/api/maps/search/getsearchfuzzy) : Cette API combine le géocodage des adresses et la recherche de points d’intérêt. Cette API prend une chaîne de forme libre. Cette chaîne peut être une adresse, un lieu, un repère, un point d’intérêt ou une catégorie de point d’intérêt. Cette API traite la requête en quasi temps réel. Cette API est recommandée pour les applications où les utilisateurs recherchent des adresses ou des points d’intérêt dans la même zone de texte.
- [**Recherche approximative par lot**](/rest/api/maps/search/postsearchfuzzybatchpreview) : Créez une requête contenant jusqu'à 10 000 adresses, lieux, points de repère ou centres d'intérêt, puis traitez-les sur une certaine période. Toutes les données seront traitées en parallèle sur le serveur et, une fois l’opération terminée, l’ensemble complet des résultats pourra être téléchargé.

Le tableau suivant référence de manière croisée les paramètres de l’API Google Maps et les paramètres d’API comparables dans Azure Maps.

| Paramètre d’API Google Maps | Paramètre d’API Azure Maps comparable  |
|---------------------------|--------------------------------------|
| `address`                   | `query`                            |
| `bounds`                    | `topLeft` et `btmRight`           |
| `components`                | `streetNumber`<br/>`streetName`<br/>`crossStreet`<br/>`postalCode`<br/>`municipality` - ville<br/>`municipalitySubdivision` – voisinage, petite / grande ville<br/>`countrySubdivision` - état ou province<br/>`countrySecondarySubdivision` - comté<br/>`countryTertiarySubdivision` - district<br/>`countryCode` - code de pays/région à deux lettres |
| `key`                       | `subscription-key` : voir également la documentation [Authentification avec Azure Maps](azure-maps-authentication.md). |
| `language`                  | `language` : voir la documentation [Langues prises en charge](supported-languages.md).  |
| `region`                    | `countrySet`                       |

Vous trouverez [ici](how-to-search-for-address.md) un exemple d’utilisation du service de recherche. Veillez à consulter les [bonnes pratiques pour la recherche](how-to-use-best-practices-for-search.md).

> [!TIP]
> Les API de géocodage d’adresses en forme libre et de recherche approximative peuvent être utilisées en mode autocomplétion en ajoutant `&typeahead=true` à l’URL de la requête. Cela indiquera au serveur que le texte entré est probablement partiel, et la recherche passera en mode prédictif.

## <a name="reverse-geocode-a-coordinate"></a>Géocodage inverse d’une coordonnée

Le géocodage inverse est le processus de conversion de coordonnées géographiques en une adresse approximative. Les coordonnées avec « longitude : -122.1298, latitude : 47.64005 » sont converties en « 1 Microsoft Way, Redmond, WA ».

Azure Maps propose plusieurs méthodes de géocodage inverse :

- [**Géocodeur inverse d’adresse**](/rest/api/maps/search/getsearchaddressreverse) : Spécifiez une seule coordonnée géographique pour obtenir l’adresse approximative correspondant à cette coordonnée. Traite la requête en quasi temps réel.
- [**Géocodeur inverse d’intersection**](/rest/api/maps/search/getsearchaddressreversecrossstreet) : Spécifiez une seule coordonnée géographique pour obtenir des informations sur les intersections proches et traitez aussitôt la requête. Par exemple, vous pouvez recevoir l’intersection suivante : 1st Ave et Main St.
- [**Géocodeur inverse d’adresses par lots**](/rest/api/maps/search/postsearchaddressreversebatchpreview) : Créez une requête contenant jusqu'à 10 000 coordonnées, puis traitez-les sur une certaine période. Toutes les données sont traitées en parallèle sur le serveur. Une fois la requête terminée, vous pouvez télécharger l’ensemble complet des résultats.

Le tableau suivant référence de manière croisée les paramètres de l’API Google Maps et les paramètres d’API comparables dans Azure Maps.

| Paramètre d’API Google Maps   | Paramètre d’API Azure Maps comparable   |
|-----------------------------|---------------------------------------|
| `key`                       | `subscription-key` : voir également la documentation [Authentification avec Azure Maps](azure-maps-authentication.md). |
| `language`                  | `language` : voir la documentation [Langues prises en charge](supported-languages.md).  |
| `latlng`                    | `query`  |
| `location_type`             | *N/A*     |
| `result_type`               | `entityType`    |

Consulter les [bonnes pratiques pour la recherche](how-to-use-best-practices-for-search.md).

L’API de géocodage inversé Azure Maps offre certaines fonctionnalités supplémentaires qui ne sont pas disponibles dans Google Maps. Il peut être utile d’intégrer ces fonctionnalités à votre application lors de sa migration :

- Récupérer les données de limitations de vitesse
- Récupérer des informations sur la nature des routes : route locale, artère, accès limité, rampe, et ainsi de suite
- Récupérer le côté de la rue duquel se trouve une coordonnée

## <a name="search-for-points-of-interest"></a>Rechercher des points d’intérêt

Les données de points d’intérêt peuvent être recherchées dans Google Maps à l’aide de l’API de recherche de lieux. Cette API offre trois façons différentes de rechercher des points d'intérêt :

- **Trouver un lieu à partir d’un texte :** Recherche d’un point d'intérêt à partir de son nom, de son adresse ou de son numéro de téléphone.
- **Recherche à proximité** : Recherche des points d'intérêt se trouvant à une certaine distance d'un endroit.
- **Recherche de texte :** Recherche de lieux à l’aide d’un texte libre comprenant des informations sur des points d’intérêt et des lieux. Par exemple, « pizza à New York » ou « restaurants près de la rue principale ».

Azure Maps fournit plusieurs API de recherche de points d'intérêt :

- [**Recherche de points d'intérêt**](/rest/api/maps/search/getsearchpoi) : Rechercher des points d’intérêt par nom. Par exemple, « Starbucks ».
- [**Recherche de catégories de points d'intérêt**](/rest/api/maps/search/getsearchpoicategory) : Rechercher des points d’intérêt par catégorie. Par exemple, « restaurant ».
- [**Recherche à proximité**](/rest/api/maps/search/getsearchnearby) : Recherche des points d'intérêt se trouvant à une certaine distance d'un endroit.
- [**Recherche approximative**](/rest/api/maps/search/getsearchfuzzy) : Cette API combine le géocodage des adresses et la recherche de points d’intérêt. Cette API prend une chaîne de forme libre qui peut être une adresse, un lieu, un point de repère, un point d’intérêt ou une catégorie de point d’intérêt. Elle traite la requête en quasi temps réel. Cette API est recommandée pour les applications où les utilisateurs recherchent des adresses ou des points d’intérêt dans la même zone de texte.
- [**Rechercher dans la géométrie**](/rest/api/maps/search/postsearchinsidegeometry) : Recherche de points d’intérêt dans une géométrie spécifiée. Par exemple, recherchez un point d’intérêt au sein d’un polygone.
- [**Recherche le long d’un itinéraire**](/rest/api/maps/search/postsearchalongroute) : Recherche de points d'intérêt qui se trouvent le long d'un itinéraire spécifié.
- [**Recherche approximative par lot**](/rest/api/maps/search/postsearchfuzzybatchpreview) : Créez une requête contenant jusqu’à 10 000 adresses, lieux, points de repère ou points d’intérêt. Traitez ensuite la requête sur une certaine période. Toutes les données sont traitées en parallèle sur le serveur. Une fois le traitement de la requête terminé, vous pouvez télécharger l’ensemble complet des résultats.

Actuellement, Azure Maps ne propose pas d’API comparable à l’API de recherche de texte de Google Maps.

> [!TIP]
> Les API de recherche de points d’intérêt, de catégorie de points d’intérêt et de recherche approximative peuvent être utilisées en mode autocomplétion en ajoutant `&typeahead=true` à l’URL de la requête. Cela indiquera au serveur que le texte entré est probablement partiel. L’API effectuera la recherche en mode prédictif.

Consultez la documentation [Bonnes pratiques pour la recherche](how-to-use-best-practices-for-search.md).

### <a name="find-place-from-text"></a>Trouver un lieu à partir d’un texte

Utilisez les API Azure Maps [Recherche de points d’intérêt](/rest/api/maps/search/getsearchpoi) et [Recherche approximative](/rest/api/maps/search/getsearchfuzzy) pour rechercher des points d’intérêt par nom ou par adresse.

Le tableau suivant référence de manière croisée les paramètres de l’API Google Maps et les paramètres d’API Azure Maps comparables.

| Paramètre d’API Google Maps | Paramètre d’API Azure Maps comparable |
|---------------------------|-------------------------------------|
| `fields`                  | *N/A*                               |
| `input`                   | `query`                             |
| `inputtype`               | *N/A*                               |
| `key`                     | `subscription-key` : voir également la documentation [Authentification avec Azure Maps](azure-maps-authentication.md). |
| `language`                | `language` : voir la documentation [Langues prises en charge](supported-languages.md).  |
| `locationbias`            | `lat`, `lon` et `radius`<br/>`topLeft` et `btmRight`<br/>`countrySet`  |

### <a name="nearby-search"></a>Recherche à proximité

Utilisez l’API [Recherche à proximité](/rest/api/maps/search/getsearchnearby) pour récupérer des points d’intérêt à proximité dans Azure Maps.

Le tableau suivant montre les paramètres de l’API Google Maps et les paramètres d’API Azure Maps comparables.

| Paramètre d’API Google Maps | Paramètre d’API Azure Maps comparable  |
|---------------------------|--------------------------------------|
| `key`                       | `subscription-key` : voir également la documentation [Authentification avec Azure Maps](azure-maps-authentication.md). |
| `keyword`                   | `categorySet` et `brandSet`        |
| `language`                  | `language` : voir la documentation [Langues prises en charge](supported-languages.md).  |
| `location`                  | `lat` et `lon`                     |
| `maxprice`                  | *N/A*                               |
| `minprice`                  | *N/A*                               |
| `name`                      | `categorySet` et `brandSet`        |
| `opennow`                   | *N/A*                               |
| `pagetoken`                 | `ofs` et `limit`                   |
| `radius`                    | `radius`                            |
| `rankby`                    | *N/A*                               |
| `type`                      | `categorySet –` Voir la documentation [Catégories de recherche prise en charge](supported-search-categories.md).   |

## <a name="calculate-routes-and-directions"></a>Calculer des itinéraires et des directions

Calculez des itinéraires et des directions à l’aide d’Azure Maps. Azure Maps offre de nombreuses fonctionnalités similaires à celles du service de routage Google Maps, par exemple :

- Heures d’arrivée et de départ
- Itinéraires en temps réel et selon les prédictions de circulation
- Différents modes de transport, tels que la voiture, la marche ou le vélo

> [!NOTE]
> Pour Azure Maps, tous les points de cheminement doivent être des coordonnées. Les adresses doivent d’abord être géocodées.

Le service de routage Azure Maps fournit les API suivantes pour le calcul des itinéraires :

- [**Calculer l’itinéraire**](/rest/api/maps/route/getroutedirections) : Calculez un itinéraire et traitez aussitôt la requête. Cette API prend en charge les requêtes GET et POST. Les requêtes POST sont recommandées lors de la spécification d’un grand nombre de points de cheminement ou lors de l’utilisation de nombreuses options de routage pour s’assurer que la requête URL ne devienne pas trop longue et n’entraîne des problèmes. L’opération POST Route Direction dans Azure Maps permet d’utiliser des milliers de [points de référence](/rest/api/maps/route/postroutedirections#supportingpoints) et de recréer un itinéraire logique entre eux (« alignement sur la route »). 
- [**Itinéraire par lots**](/rest/api/maps/route/postroutedirectionsbatchpreview) : Créez une requête contenant jusqu'à 1 000 requêtes d’itinéraire, puis traitez-les sur une certaine période. Toutes les données seront traitées en parallèle sur le serveur et, une fois l’opération terminée, l’ensemble complet des résultats pourra être téléchargé.
- [**Services Mobility (préversion) **](/rest/api/maps/mobility) : Calculez des itinéraires et des directions en utilisant les transports en commun.

Le tableau suivant référence de manière croisée les paramètres de l’API Google Maps et les paramètres d’API comparables dans Azure Maps.

| Paramètre d’API Google Maps    | Paramètre d’API Azure Maps comparable  |
|------------------------------|--------------------------------------|
| `alternatives`                 | `maxAlternatives`                  |
| `arrival_time`                | `arriveAt`                          |
| `avoid`                        | `avoid`                            |
| `departure_time`              | `departAt`                          |
| `destination`                  | `query` – coordonnées au format `"lat0,lon0:lat1,lon1…."`  |
| `key`                          | `subscription-key` : voir également la documentation [Authentification avec Azure Maps](azure-maps-authentication.md). |
| `language`                     | `language` : voir la documentation [Langues prises en charge](supported-languages.md).   |
| `mode`                         | `travelMode`                       |
| `optimize`                     | `computeBestOrder`                 |
| `origin`                       | `query`                            |
| `region`                       | *N/A* : cette fonctionnalité est liée au géocodage. Utilisez le paramètre *countrySet* lorsque vous utilisez l'API de géocodage d’Azure Maps.  |
| `traffic_model`               | *N/A* : permet uniquement de spécifier si les données de trafic doivent être utilisées avec le paramètre *traffic*. |
| `transit_mode`                | Voir la [documentation sur les services Mobility (préversion)](/rest/api/maps/mobility) |
| `transit_routing_preference` | Voir la [documentation sur les services Mobility (préversion)](/rest/api/maps/mobility) |
| `units`                        | *N/A* : Azure Maps utilise uniquement le système métrique.  |
| `waypoints`                    | `query`                            |

> [!TIP]
> Par défaut, l’API d’itinéraire Azure Maps retourne uniquement un résumé. Elle retourne la distance, la durée et les coordonnées de l’itinéraire. Utilisez le paramètre `instructionsType` pour obtenir des instructions de navigation étape par étape. Utilisez le paramètre `routeRepresentation` pour filtrer le résumé et l’itinéraire.

L’API d’itinéraire Azure Maps offre certaines fonctionnalités supplémentaires qui ne sont pas disponibles dans Google Maps. Pensez à ces fonctionnalités lors de la migration de votre application ; elles pourraient vous être utiles.

- Prise en charge du type d'itinéraire : le plus court, le plus rapide, vers plusieurs destinations simultanément, et le plus économique en carburant.
- Prise en charge de modes de déplacement supplémentaires : bus, moto, taxi, camion et camionnette.
- Prise en charge de 150 points de cheminement.
- Calculez plusieurs temps de parcours dans une même requête ; trafic historique, trafic en direct, aucun trafic.
- Autres types de routes à éviter : itinéraires de covoiturage, routes non goudronnées, routes déjà utilisées.
- Spécifiez des zones personnalisées à éviter.
- Limiter l’élévation de l’itinéraire.
- Itinéraire basé sur les spécifications du moteur. Calculez les itinéraires pour les véhicules à combustion ou électriques en fonction des caractéristiques du moteur et de la réserve de carburant ou d’électricité.
- Prise en charge des paramètres d’itinéraire pour véhicules utilitaires, tels que les dimensions du véhicule, le poids, le nombre d’axes et le type de cargaison.
- Spécifiez la vitesse maximale du véhicule.

De plus, le service Route dans Azure Maps prend en charge le [calcul des plages d’itinéraires](/rest/api/maps/route/getrouterange). Le calcul des plages d’itinéraires est également appelé isochrones. Cela implique la génération d’un polygone couvrant une zone qui peut être parcourue dans n’importe quelle direction à partir d’un point d’origine, tout ceci dans un laps de temps ou une quantité de carburant spécifié(e).

Consultez la documentation sur les [bonnes pratiques en matière de routage](how-to-use-best-practices-for-routing.md).

## <a name="retrieve-a-map-image"></a>Récupérer une image de carte

Azure Maps fournit une API pour le rendu des images de carte statiques avec données superposées. L’API de [rendu des images de carte](/rest/api/maps/render/getmapimagerytile) dans Azure Maps est comparable à l’API de carte statique de Google Maps.

> [!NOTE]
> Avec Azure Maps, le centre ainsi que toutes les localisations des marqueurs et des tracés doivent être des coordonnées au format « longitude, latitude », alors que Google Maps utilise le format « latitude,longitude ». Les adresses devront d’abord être géocodées.

Le tableau suivant référence de manière croisée les paramètres de l’API Google Maps et les paramètres d’API comparables dans Azure Maps.

| Paramètre d’API Google Maps | Paramètre d’API Azure Maps comparable  |
|---------------------------|--------------------------------------|
| `center`                    | `center`                           |
| `format`                    | `format` : spécifié comme faisant partie du chemin d'accès à l’URL. Actuellement, seul le format PNG est pris en charge. |
| `key`                       | `subscription-key` : voir également la documentation [Authentification avec Azure Maps](azure-maps-authentication.md). |
| `language`                  | `language` : voir la documentation [Langues prises en charge](supported-languages.md).  |
| `maptype`                   | `layer` et `style` : voir la documentation [Styles de carte pris en charge](supported-map-styles.md). |
| `markers`                   | `pins`                             |
| `path`                      | `path`                             |
| `region`                    | *N/A* : cette fonctionnalité est liée au géocodage. Utilisez le paramètre `countrySet` lorsque vous utilisez l'API de géocodage d’Azure Maps.  |
| `scale`                     | *N/A*                              |
| `size`                      | `width` et `height` : peut avoir une taille maximale de 8 192 x 8 192. |
| `style`                     | *N/A*                              |
| `visible`                   | *N/A*                              |
| `zoom`                      | `zoom`                             |

> [!NOTE]
> Dans le système de mosaïques Azure Maps, la taille des mosaïques est deux fois supérieure à celle des mosaïques de carte utilisées dans Google Maps. Ainsi, la valeur du niveau de zoom dans Azure Maps apparaîtra un niveau de zoom plus proche dans Azure Maps par rapport à Google Maps. Pour compenser cette différence, réduisez d’une unité le niveau de zoom dans les requêtes que vous migrez.

Pour plus d’informations, consultez le [Guide pratique de l'API de rendu des images de carte](how-to-render-custom-data.md).

En plus de pouvoir générer une image de carte statique, le service de rendu Azure Maps permet d’accéder directement aux mosaïques de la carte au format raster (PNG) et vectoriel :

- [**Mosaïque de la carte**](/rest/api/maps/render/getmaptile) : Récupérez les mosaïques raster (PNG) et vectorielles pour les cartes de base (routes, limites, fond).
- [**Mosaïque d’imagerie de carte**](/rest/api/maps/render/getmapimagerytile) : Récupérez les mosaïques d'images aériennes et satellites.

> [!TIP]
> De nombreuses applications Google Maps sont passées, il y a quelques années, d’une expérience cartographique interactive à des images cartographiques statiques, ceci en vue de réduire les coûts. Dans Azure Maps, il est généralement plus rentable d’utiliser le contrôle de carte interactif dans le SDK web. Le contrôle de carte interactif facture en fonction du nombre de chargements de mosaïques. Les mosaïques cartographiques dans Azure Maps sont volumineuses. Souvent, il suffit de quelques mosaïques pour recréer la même vue cartographique qu’une carte statique. Les mosaïques cartographiques sont automatiquement mises en cache par le navigateur. Ainsi, le contrôle de carte interactif ne génère souvent qu’une fraction d’une transaction lors de la reproduction d’une vue de carte statique. Les fonctions de panoramique et de zoom chargeront davantage de mosaïques, mais des options de contrôle de carte vous permettent de désactiver ce comportement. Le contrôle de carte interactif offre également beaucoup plus d’options de visualisation que les services de carte statique.

### <a name="marker-url-parameter-format-comparison"></a>Comparaison du format des paramètres URL des marqueurs

**Avant : Google Maps**

Ajoutez des marqueurs à l’aide du paramètre `markers` dans l’URL. Le paramètre `markers` prend en compte un style et une liste d’emplacements à afficher sur la carte avec ce style, comme indiqué ci-dessous :

```
&markers=markerStyles|markerLocation1|markerLocation2|...
```

Pour ajouter des styles supplémentaires, utilisez les paramètres `markers` de l’URL avec un style et un jeu de localisations différents.

Spécifiez les localisations des marqueurs au format « latitude,longitude ».

Ajoutez des styles de marqueurs au format `optionName:value`, en séparant les différents styles avec une barre verticale (\|), par exemple « optionName1:value1\|optionName2:value2 ». Notez que les noms et les valeurs des options sont séparés par deux points ( :). Utilisez les noms d’options de style suivants pour appliquer un style à des marqueurs dans Google Maps :

- `color` : couleur de l’icône du marqueur par défaut. Peut être une couleur hexadécimale 24 bits (`0xrrggbb`) ou une des valeurs suivantes ; `black`, `brown`, `green`, `purple`, `yellow`, `blue`, `gray`, `orange`, `red`, `white`.
- `label` : caractère alphanumérique majuscule unique à afficher en haut de l’icône.
- `size` : taille du marqueur. Peut être `tiny`, `mid` ou `small`.

Utilisez les noms d’options de style suivants pour les icônes personnalisées dans Google Maps :

- `anchor` : spécifie comment aligner l’image de l’icône à la coordonnée. Peut être une valeur de pixel (x,y) ou l’une des valeurs suivantes ; `top`, `bottom`, `left`, `right`, `center`, `topleft`, `topright`, `bottomleft` ou `bottomright`.
- `icon` : URL pointant vers l’image de l’icône.

Par exemple, ajoutons un marqueur rouge de taille moyenne à la carte, aux coordonnées longitude : -110, latitude : 45 :

```
&markers=color:red|size:mid|45,-110
```


![Marqueur Google Maps](media/migrate-google-maps-web-services/google-maps-marker.png)

**Après : Azure Maps**

Ajoutez des marqueurs à une image de carte statique en spécifiant le paramètre `pins` dans l’URL. Comme avec Google Maps, spécifiez un style et une liste de localisations dans le paramètre. Le paramètre `pins` peut être spécifié plusieurs fois pour prendre en charge des marqueurs avec des styles différents.

```
&pins=iconType|pinStyles||pinLocation1|pinLocation2|...
```

Pour utiliser des styles supplémentaires, ajoutez des paramètres `pins` supplémentaires à l’URL avec un style et un jeu de localisations différents.

Dans Azure Maps, la localisation de l’épingle doit être au format « longitude latitude ». Google Maps utilise le format « latitude,longitude ». Un espace, et non une virgule, sépare la longitude de la latitude au format Azure Maps.

`iconType` spécifie le type d’épingle à créer. Il peut avoir les valeurs suivantes :

- `default` : icône d’épingle par défaut.
- `none` : aucune icône n’apparaît, seules les étiquettes seront affichées.
- `custom` : spécifie une icône personnalisée à utiliser. Une URL pointant vers l'image de l'icône peut être ajoutée à la fin du paramètre `pins`, après les informations d’emplacement de l’épingle.
- `{udid}` : identifiant unique de données (UDID) pour une icône stockée dans la plateforme de stockage de données Azure Maps.

Ajoutez des styles d’épingle avec le format `optionNameValue`. Séparez les différents styles à l’aide de barres verticales (\|). Par exemple : `iconType|optionName1Value1|optionName2Value2`. Les noms et les valeurs des options ne sont pas séparés. Utilisez les noms d’options de style suivants pour appliquer un style aux marqueurs :

- `al` : spécifie l’opacité (alpha) du marqueur. Choisissez un nombre compris entre 0 et 1.
- `an` : spécifie l’ancrage de l’épingle. Spécifiez les valeurs de pixel x et y au format « x y ».
- `co` : couleur de l’épingle. Spécifiez une couleur hexadécimale 24 bits : de `000000` à `FFFFFF`.
- `la` : spécifie l’ancrage de l’étiquette. Spécifiez les valeurs de pixel x et y au format « x y ».
- `lc` : couleur de l’étiquette. Spécifiez une couleur hexadécimale 24 bits : de `000000` à `FFFFFF`.
- `ls` : taille de l’étiquette en pixels. Choisissez un nombre supérieur à 0.
- `ro` : valeur en degrés de la rotation de l’icône. Choisissez un nombre compris entre -360 et 360.
- `sc` : valeur de mise à l'échelle de l’icône de l’épingle. Choisissez un nombre supérieur à 0.

Spécifiez les valeurs d’étiquette pour chaque localisation d’épingle. Cette approche est plus efficace que l’application d’une valeur d’étiquette unique à tous les marqueurs dans la liste des localisations. La valeur de l’étiquette peut être une chaîne de plusieurs caractères. Placez la chaîne entre des guillemets simples pour vous assurer qu’elle ne soit pas confondue avec une valeur de style ou de localisation.

Ajoutons une icône par défaut rouge (`FF0000`), avec l’étiquette « Space Needle », positionnée en dessous (15 50). L’icône est à la longitude : -122.349300, latitude : 47.620180 :

```
&pins=default|coFF0000|la15 50||'Space Needle' -122.349300 47.620180
```

![Marqueur Azure Maps](media/migrate-google-maps-web-services/azure-maps-marker.png)

Ajoutons trois épingles avec les valeurs d’étiquette « 1 », « 2 » et « 3 » :

```
&pins=default||'1'-122 45|'2'-119.5 43.2|'3'-121.67 47.12
```

![Marqueurs multiples Azure Maps](media/migrate-google-maps-web-services/azure-maps-multiple-markers.png)

### <a name="path-url-parameter-format-comparison"></a>Comparaison du format du paramètre URL des chemins d'accès

**Avant : Google Maps**

Ajoutez des lignes et un polygone à une image de carte statique à l’aide du paramètre `path` dans l’URL. Le paramètre `path` prend un style et une liste de localisations à afficher sur la carte, comme indiqué ci-dessous :

```
&path=pathStyles|pathLocation1|pathLocation2|...
```

Utilisez des styles supplémentaires en ajoutant des paramètres `path` supplémentaires à l’URL avec un style et un jeu de localisations différents.

Les localisations de tracés sont spécifiés au format `latitude1,longitude1|latitude2,longitude2|…`. Les chemins d’accès peuvent être encodés ou contenir des adresses de points.

Ajoutez des styles de tracés au format `optionName:value`, et séparez les différents styles à l’aide de barres verticales (\|). Séparez également les noms et les valeurs des options à l’aide d’un signe deux-points (:). Comme ceci : `optionName1:value1|optionName2:value2`. Les noms d’options de style suivants peuvent être utilisés pour appliquer un style à des chemins d'accès dans Google Maps :

- `color` : couleur du contour du chemin d'accès ou du polygone. Peut être une couleur hexadécimale 24 bits (`0xrrggbb`), une couleur hexadécimale 32 bits (`0xrrggbbbaa`) ou l’une des valeurs suivantes : noir, brun, vert, violet, jaune, bleu, gris, orange, rouge, blanc.
- `fillColor` : couleur de remplissage de la zone du chemin d'accès avec (polygone). Peut être une couleur hexadécimale 24 bits (`0xrrggbb`), une couleur hexadécimale 32 bits (`0xrrggbbbaa`) ou l’une des valeurs suivantes : noir, brun, vert, violet, jaune, bleu, gris, orange, rouge, blanc.
- `geodesic` : indique si le chemin d’accès doit être une ligne qui suit la courbure de la Terre.
- `weight` : épaisseur de la ligne du chemin d'accès en pixels.

Ajoutez une opacité de ligne rouge et une épaisseur de pixel à la carte entre les coordonnées, dans le paramètre d’URL. Dans l’exemple ci-dessous, la ligne a une opacité de 50 % et une épaisseur de quatre pixels. Les coordonnées sont longitude : -110, latitude : 45 et longitude : -100, latitude : 50.

```
&path=color:0xFF000088|weight:4|45,-110|50,-100
```

![Polyligne Google Maps](media/migrate-google-maps-web-services/google-maps-polyline.png)

**Après : Azure Maps**

Ajoutez des lignes et des polygones à une image de carte statique en spécifiant le paramètre `path` dans l’URL. Comme avec Google Maps, spécifiez un style et une liste de localisations dans ce paramètre. Spécifiez le paramètre `path` plusieurs fois pour afficher plusieurs cercles, lignes et polygones avec des styles différents.

```
&path=pathStyles||pathLocation1|pathLocation2|...
```

En ce qui concerne les localisations de tracés, Azure Maps demande que les coordonnées soient au format « longitude latitude ». Google Maps utilise le format « latitude,longitude ». Un espace, et non une virgule, sépare la longitude de la latitude au format Azure Maps. Azure Maps ne prend pas en charge les adresses ou les tracés encodés pour des points. Chargez les plus grands jeux de données sous forme de fichier GeoJSON dans l’API de stockage de données Azure Maps, comme documenté [ici](how-to-render-custom-data.md#get-data-from-azure-maps-data-storage).

Ajoutez des styles de tracés au format `optionNameValue`. Séparez les différents styles à l’aide de barres verticales (\|), comme ceci : `optionName1Value1|optionName2Value2`. Les noms et les valeurs des options ne sont pas séparés. Utilisez les noms d’options de style suivants pour appliquer un style à des tracés dans Azure Maps :

- `fa` : opacité de la couleur de remplissage (alpha) utilisée lors du rendu des polygones. Choisissez un nombre compris entre 0 et 1.
- `fc` : couleur de remplissage utilisée pour rendre l’aire d'un polygone.
- `la` : opacité de la couleur des lignes (alpha) utilisée lors du rendu des lignes et du contour des polygones. Choisissez un nombre compris entre 0 et 1.
- `lc` : couleur de ligne utilisée pour rendre les lignes et le contour des polygones.
- `lw` : largeur de la ligne en pixels.
- `ra` : spécifie un rayon de cercle en mètres.

Ajoutez une opacité de ligne rouge et une épaisseur de pixel entre les coordonnées, dans le paramètre d’URL. Dans l’exemple ci-dessous, la ligne a une opacité de 50 % et une épaisseur de quatre pixels. Les coordonnées ont les valeurs suivantes : longitude : -110, latitude : 45 et longitude : -100, latitude : 50.

```
&path=lcFF0000|la.5|lw4||-110 45|-100 50
```

![Polyligne Azure Maps](media/migrate-google-maps-web-services/azure-maps-polyline.png)

## <a name="calculate-a-distance-matrix"></a>Calculer une matrice des distances

Azure Maps fournit l’API de matrice des distances. Utilisez cette API pour calculer les durées et les distances des trajets entre un ensemble de localisations, avec une matrice des distances. Elle est comparable à l’API de matrice des distances dans Google Maps.

- [**Matrice d’itinéraire**](/rest/api/maps/route/postroutematrixpreview) : Calcule de manière asynchrone les durées de déplacement et les distances pour un jeu d'origines et de destinations. Prend en charge jusqu’à 700 cellules par requête. Il s’agit du nombre d’origines multiplié par le nombre de destinations. En gardant cette contrainte à l'esprit, voici des exemples de dimensions matricielles possibles : 700x1, 50x10, 10x10, 28x25, 10x70.

> [!NOTE]
> Une requête à l'API de matrice des distances ne peut être faite qu'en utilisant une requête POST avec les informations d'origine et de destination dans le corps de la requête. De plus, Azure Maps exige que toutes les origines et destinations soient des coordonnées. Les adresses devront d’abord être géocodées.

Le tableau suivant référence de manière croisée les paramètres de l’API Google Maps et les paramètres d’API comparables dans Azure Maps.

| Paramètre d’API Google Maps      | Paramètre d’API Azure Maps comparable  |
|--------------------------------|--------------------------------------|
| `arrivial_time`                | `arriveAt`                           |
| `avoid`                        | `avoid`                              |
| `depature_time`                | `departAt`                           |
| `destinations`                 | `destination` : spécifiez GeoJSON dans le corps de la requête POST. |
| `key`                          | `subscription-key` : voir également la documentation [Authentification avec Azure Maps](azure-maps-authentication.md). |
| `language`                     | `language` : voir la documentation [Langues prises en charge](supported-languages.md).  |
| `mode`                         | `travelMode`                         |
| `origins`                      | `origins` : spécifiez GeoJSON dans le corps de la requête POST.  |
| `region`                       | *N/A* : cette fonctionnalité est liée au géocodage. Utilisez le paramètre `countrySet` lorsque vous utilisez l'API de géocodage d’Azure Maps. |
| `traffic_model`                | *N/A* : permet uniquement de spécifier si les données de trafic doivent être utilisées avec le paramètre `traffic`. |
| `transit_mode`                 | *N/A* : les matrices des distances basées sur le transit ne sont pas prises en charge actuellement.  |
| `transit_routing_preference`   | *N/A* : les matrices des distances basées sur le transit ne sont pas prises en charge actuellement.  |
| `units`                        | *N/A* : Azure Maps utilise uniquement le système métrique. |

> [!TIP]
> Toutes les options d’itinéraire avancées disponibles dans l’API de route Azure Maps sont prises en charge dans l’API de matrice des distances Azure Maps. Les options d’itinéraire avancées comprennent notamment les itinéraires pour camions et les caractéristiques du moteur.

Consultez la documentation sur les [bonnes pratiques en matière de routage](how-to-use-best-practices-for-routing.md).

## <a name="get-a-time-zone"></a>Obtenir un fuseau horaire

Azure Maps fournit une API pour récupérer le fuseau horaire d’une coordonnée. L'API de fuseau horaire d’Azure Maps est comparable à l'API de fuseau horaire de Google Maps :

- [**Fuseau horaire par coordonnée**](/rest/api/maps/timezone/gettimezonebycoordinates) : Spécifiez une coordonnée et recevez les détails du fuseau horaire de la coordonnée.

Le tableau suivant référence de manière croisée les paramètres de l’API Google Maps et les paramètres d’API comparables dans Azure Maps.

| Paramètre d’API Google Maps | Paramètre d’API Azure Maps comparable   |
|---------------------------|---------------------------------------|
| `key`                       | `subscription-key` : voir également la documentation [Authentification avec Azure Maps](azure-maps-authentication.md).       |
| `language`                  | `language` : voir la documentation [Langues prises en charge](supported-languages.md).    |
| `location`                  | `query`             |
| `timestamp`                 | `timeStamp`         |

En plus de cette API, Azure Maps fournit un certain nombre d’API de fuseau horaire. Ces API convertissent l’heure en fonction des noms ou des ID du fuseau horaire :

- [**Fuseau horaire par ID**](/rest/api/maps/timezone/gettimezonebyid) : Renvoie les informations de fuseau horaire actuelles, historiques et futures pour l'ID de fuseau horaire IANA spécifié.
- [**Fuseau horaire avec Enum IANA**](/rest/api/maps/timezone/gettimezoneenumiana) : Renvoie une liste complète des ID de fuseau horaire IANA. Les mises à jour du service IANA sont reflétées dans le système dans un délai de 24 heures.
- [**Fuseau horaire avec Enum Windows**](/rest/api/maps/timezone/gettimezoneenumwindows) : Renvoie une liste complète des ID de fuseau horaire Windows.
- [**Fuseau horaire avec version IANA**](/rest/api/maps/timezone/gettimezoneianaversion) : Renvoie le numéro de la version actuelle IANA utilisée par Azure Maps.
- [**Fuseau horaire Windows à IANA**](/rest/api/maps/timezone/gettimezonewindowstoiana) : Renvoie un ID IANA correspondant, avec un ID de fuseau horaire Windows valide. Plusieurs ID IANA peuvent être renvoyés pour un seul ID Windows.

## <a name="client-libraries"></a>Bibliothèques clientes

Azure Maps fournit des bibliothèques clientes pour les langages de programmation suivants :

- JavaScript, TypeScript, Node.js : [documentation](how-to-use-services-module.md) \| [Package NPM](https://www.npmjs.com/package/azure-maps-rest)

Ces bibliothèques de client open source concernent d’autres langages de programmation :

- .NET Standard 2.0 : [projet GitHub](https://github.com/perfahlen/AzureMapsRestServices) \| [package NuGet](https://www.nuget.org/packages/AzureMapsRestToolkit/)

## <a name="next-steps"></a>Étapes suivantes

Découvrez-en plus sur les services REST Azure Maps :

> [!div class="nextstepaction"]
> [Bonnes pratiques pour la recherche](how-to-use-best-practices-for-search.md)

> [!div class="nextstepaction"]
> [Rechercher une adresse](how-to-search-for-address.md)

> [!div class="nextstepaction"]
> [Bonne pratiques en matière de routage](how-to-use-best-practices-for-routing.md)

> [!div class="nextstepaction"]
> [Documentation de référence sur l’API de service REST Azure Maps](/rest/api/maps/)

> [!div class="nextstepaction"]
> [Exemples de code](/samples/browse/?products=azure-maps)

> [!div class="nextstepaction"]
> [Comment utiliser le module de services (SDK web)](how-to-use-best-practices-for-routing.md)