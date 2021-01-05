---
title: Vue d’ensemble de Microsoft Azure Maps
description: Découvrez plus d’informations sur les services et les fonctionnalités de Microsoft Azure Maps, et comment les utiliser dans vos applications.
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: overview
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc, references_regions
ms.openlocfilehash: b53af0fda8bb24bf03d0e02f61ce7d195ae69503
ms.sourcegitcommit: 66b0caafd915544f1c658c131eaf4695daba74c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97680821"
---
# <a name="what-is-azure-maps"></a>Qu’est-ce qu’Azure Maps ?

Azure Maps est une collection de services géospatiaux et de kits de développement logiciel (SDK) qui utilisent des données cartographiques actualisées pour fournir un contexte géographique précis à des applications web et mobiles. Azure Maps fournit les services suivants :

* API REST pour assurer le rendu de cartes vectorielles et raster dans plusieurs styles et une imagerie satellitaire.
* Services Creator (préversion) pour créer et afficher des cartes basées sur des données de carte d’intérieur privées.
* Services Search pour localiser les adresses, les lieux et les points d’intérêt dans le monde entier.
* Diverses options de routage : point à point, multipoint, optimisation multipoint, isochrone, véhicule électrique, véhicule commercial, trafic influencé et routage par matrice.
* Vue du flux de trafic et vue des incidents pour les applications qui ont besoin d’informations de trafic en temps réel.
* Services Mobility (préversion) pour demander des informations de transit public, planifier des itinéraires en fusionnant différents modes de voyage et arrivées en temps réel.
* Services Time Zone et Geolocation (préversion).
* Service Elevation (préversion) avec un modèle d’élévation numérique
* Services de geofencing et stockage des données cartographiques, avec les informations d’emplacement hébergées dans Azure.
* Intelligence géographique via l’analytique géospatiale.

Par ailleurs, les services Azure Maps sont disponibles via le Kit de développement logiciel (SDK) web ou l’Android SDK. Ces outils aident les développeurs à développer et à mettre à l’échelle rapidement des solutions intégrant des informations de localisation dans des solutions Azure.

Vous pouvez vous inscrire pour un [compte Azure Maps](https://azure.microsoft.com/services/azure-maps/) gratuit et commencer à développer.

La vidéo suivante explique Azure Maps plus en détail :

</br>

> [!VIDEO https://channel9.msdn.com/Shows/Internet-of-Things-Show/Azure-Maps/player?format=ny]

## <a name="map-controls"></a>Contrôles de carte

### <a name="web-sdk"></a>Kit de développement logiciel (SDK) web

Le SDK web Azure Maps vous permet de personnaliser des cartes interactives avec du contenu et des images qui vous sont propres. Vous pouvez utiliser cette carte interactive pour vos applications web ou mobiles. Le contrôle de carte utilise WebGL : vous pouvez donc afficher les jeux de données volumineux avec un haut niveau de performance. Vous pouvez développer avec le Kit de développement logiciel (SDK) en utilisant JavaScript ou TypeScript.

:::image type="content" source="./media/about-azure-maps/intro_web_map_control.png" alt-text="Exemple de carte de la modification de population créée à l’aide du Kit de développement logiciel (SDK) web Azure Maps":::

### <a name="android-sdk"></a>Kit de développement logiciel Android

Utilisez le SDK Android d’Azure Maps pour créer des applications cartographiques mobiles.

:::image type="content" source="./media/about-azure-maps/android_sdk.png" border="false" alt-text="Exemples de cartes sur un appareil mobile":::

## <a name="services-in-azure-maps"></a>Services d’Azure Maps

Azure Maps se compose des services suivants, qui peuvent fournir un contexte géographique à vos applications Azure.

### <a name="data-service-preview"></a>Service Data (préversion)

Les données sont indispensables pour les cartes. Le service Data permet de charger et stocker des données géospatiales pour une utilisation avec des opérations spatiales ou la composition d’images.  Le fait de pouvoir rapprocher les données client du service Azure Maps permettra de réduire la latence, d’améliorer la productivité et de créer de nouveaux scénarios dans vos applications. Pour plus d’informations sur ce service, consultez la [Documentation du service Data](/rest/api/maps/data).

### <a name="geolocation-service-preview"></a>Service Geolocation (préversion)

Le service Geolocation permet de prévisualiser le code de pays/région à deux lettres récupéré pour une adresse IP. Ce service peut vous aider à améliorer l’expérience utilisateur en personnalisant le contenu des applications en fonction de la localisation géographique.

Pour plus d’informations, consultez la [Documentation du service Geolocation](/rest/api/maps/geolocation).

### <a name="mobility-services-preview"></a>Services Mobility (préversion) 

Les services Mobility d’Azure Maps améliorent le temps de développement pour les applications présentant des fonctionnalités de transport public, comme le calcul d’itinéraires de transport et la recherche d’arrêts à proximité. Les utilisateurs peuvent extraire des informations détaillées sur les arrêts, les lignes et les horaires des transports publics. Le service Mobility permet aussi aux utilisateurs de récupérer les géométries d’arrêts et de lignes, les alertes pour les arrêts, les lignes, les zones de service ainsi que les arrivées des transports publics et les alertes de service en temps réel. De plus, les services Mobility offrent des fonctionnalités de calcul d’itinéraires avec des options de planification de trajet multimodal. La planification de trajet multimodal permet de combiner marche, vélo et transports publics. Les utilisateurs peuvent aussi accéder à des itinéraires pas à pas multimodaux détaillés.

Pour en savoir plus sur le service, consultez la [Documentation des services Mobility](/rest/api/maps/mobility).

### <a name="render-service"></a>Render Service

Le [service Render V2 (préversion)](/rest/api/maps/renderv2) introduit une nouvelle version de l’[API Get Map Tile V2](/rest/api/maps/renderv2/getmaptilepreview). L’API Get Map Tile V2 permet aux clients de demander à Azure Maps des vignettes routières, des vignettes météo ou des vignettes de carte créées à l’aide du Créateur Azure Maps. Il est recommandé d’utiliser la nouvelle API Get Map Tile V2.  

:::image type="content" source="./media/about-azure-maps/intro_map.png" border="false" alt-text="Exemple de carte provenant du service Render V2":::

Pour plus d’informations, consultez la [Documentation du service Render V2](/rest/api/maps/renderv2).

Pour en savoir plus sur le service Render V1 en disponibilité générale, consultez la [Documentation du service Render V1](/rest/api/maps/render).  

### <a name="route-service"></a>Service d’itinéraire

Les services de routage permettent de calculer les heures d’arrivée estimées pour chaque itinéraire demandé. Les API Route tiennent compte de facteurs tels que des informations de trafic en temps réel et des données de trafic historiques comme les vitesses routières typiques correspondant au jour et à l’heure demandés. Les API retournent les itinéraires les plus courts ou les plus rapides disponibles vers plusieurs destinations à la fois à la suite ou dans un ordre optimisé, en fonction de l’heure ou de la distance. Le service permet aux développeurs de calculer des directions pour plusieurs modes de déplacement, à savoir en voiture, en camion, à vélo, à pied et en véhicule électrique. Le service prend aussi en compte des entrées comme l’heure de départ, les restrictions de poids ou le transport de matières dangereuses.

:::image type="content" source="./media/about-azure-maps/intro_route.png" border="false" alt-text="Exemple de carte provenant de Route Service":::

Le service Route offre des fonctionnalités avancées, telles que :

* Traitement par lots de plusieurs demandes d’itinéraire.
* Matrices de durées et de distances des trajets entre un ensemble d’origines et de destinations.
* Recherche des itinéraires ou des distances que les utilisateurs peuvent parcourir en fonction d’exigences horaires ou de carburant.

Pour plus d’informations sur les fonctionnalités de routage, consultez la [Documentation du service Route](/rest/api/maps/route).

### <a name="search-service"></a>Service de recherche

Search Service aide les développeurs à rechercher des adresses, des lieux, des listes d’entreprises par nom ou par catégorie, et d’autres informations d’ordre géographique. Les services peuvent aussi effectuer un [géocodage inverse](https://en.wikipedia.org/wiki/Reverse_geocoding) d’adresses et d’intersections basé sur des latitudes et longitudes.

:::image type="content" source="./media/about-azure-maps/intro_search.png" border="false" alt-text="Exemple d’une recherche sur une carte":::

Search Service fournit également des fonctionnalités avancées comme les suivantes :

* Rechercher le long d’une route.
* Rechercher à l’intérieur d’une zone plus large.
* Traiter par lot un groupe de demandes de recherche.
* Rechercher des bornes de recharge de véhicule électrique et des données de point d’intérêt par nom de marque.

Pour plus d’informations sur les fonctionnalités de recherche, consultez la [Documentation du service Search](/rest/api/maps/search).

### <a name="spatial-service"></a>Service spatial

Le service Spatial analyse rapidement les informations d’emplacement afin que de tenir les clients au courant d’événements qui se produisent dans le temps et dans l’espace. Il permet une analyse en quasi temps réel et la modélisation prédictive des événements.

Le service permet aux clients d’améliorer leurs informations de localisation, avec une bibliothèque de calculs mathématiques géospatiaux courants. Les calculs courants incluent le point le plus proche, la distance orthodromique et les zones tampons. Pour plus d’informations sur le service et ses différentes fonctionnalités, consultez la [Documentation du service Spatial](/rest/api/maps/spatial).

### <a name="timezone-service"></a>Service de fuseau horaire

Le service Time Zone vous permet d’interroger des informations actuelles, historiques et futures sur les fuseaux horaires. Vous pouvez utiliser en entrée des combinaisons de latitude et de longitude, ou un [ID IANA](https://www.iana.org/). Le service Time Zone vous permet également d’effectuer les opérations suivantes :

* Convertir les ID de fuseau horaire Microsoft Windows en fuseaux horaires IANA.
* Récupérer un décalage de fuseau horaire au format UTC.
* Obtenir l’heure actuelle dans un fuseau horaire déterminé.

Voici un exemple de réponse JSON typique à une requête adressée au service Time Zone :

```JSON
{
  "Version": "2020a",
  "ReferenceUtcTimestamp": "2020-07-31T19:15:14.4570053Z",
  "TimeZones": [
    {
      "Id": "America/Los_Angeles",
      "Names": {
        "ISO6391LanguageCode": "en",
        "Generic": "Pacific Time",
        "Standard": "Pacific Standard Time",
        "Daylight": "Pacific Daylight Time"
      },
      "ReferenceTime": {
        "Tag": "PDT",
        "StandardOffset": "-08:00:00",
        "DaylightSavings": "01:00:00",
        "WallTime": "2020-07-31T12:15:14.4570053-07:00",
        "PosixTzValidYear": 2020,
        "PosixTz": "PST+8PDT,M3.2.0,M11.1.0"
      }
    }
  ]
}
```

Pour plus d’informations sur ce service, consultez la [Documentation du service Time Zone](/rest/api/maps/timezone).

### <a name="traffic-service"></a>Traffic Service

Traffic Service est une suite de services web conçue que les développeurs peuvent utiliser pour créer des applications web et mobiles nécessitant des informations sur le trafic. Ce service fournit deux types de données :

* Flux de trafic : Vitesses et durées de déplacement observées en temps réel pour toutes les routes clés du réseau.
* Incidents de trafic : Vue actualisée des embouteillages et des incidents sur le réseau routier.

![Exemple de carte avec des informations sur le trafic](media/about-azure-maps/intro_traffic.png)

Pour plus d’informations, consultez la [Documentation du service Traffic](/rest/api/maps/traffic).

### <a name="weather-services-preview"></a>Services Météo (préversion) 

Les services Météo offrent des API dont les développeurs peuvent se servir pour récupérer des informations météorologiques pour un lieu particulier. Ces informations contiennent des détails tels que la date et l’heure de l’observation, une brève description des conditions météorologiques, une icône météo, des indicateurs de précipitation, ainsi que des données concernant la température et la vitesse du vent. Des détails supplémentaires tels que la température RealFeel™ et l’indice UV sont également retournés.

Les développeurs peuvent utiliser l’[API d’obtention de données météorologiques](/rest/api/maps/weather/getweatheralongroutepreview) pour récupérer des informations météorologiques concernant un itinéraire particulier. En outre, le service prend en charge la génération de notifications météorologiques pour des points de route affectés par des aléas climatiques tels que des inondations ou de fortes pluies.

L’[API Get Map Tile V2](/rest/api/maps/renderv2/getmaptilepreview) vous permet de demander des vignettes de satellite et de radar passées, actuelles et futures.

![Exemple de carte avec des vignettes de radar météo en temps réel](media/about-azure-maps/intro_weather.png)

### <a name="maps-creator-service-preview"></a>Service Creator d’Azure Maps (préversion) 

Le Créateur Azure Maps est une suite de services web que les développeurs peuvent utiliser pour créer des applications avec des fonctionnalités cartographiques basées sur des données de carte d’intérieur.

Le Créateur Azure Maps fournit trois services de base :

* [Service Jeu de données](/rest/api/maps/dataset). Utilisez le service DataSet pour créer un jeu de données à partir de données d’un package de dessin converti. Pour plus d’informations sur les exigences du package de dessin, consultez Exigences du package de dessin.

* [Service de conversion](/rest/api/maps/dataset). Le service de conversion permet de convertir un fichier de conception DWG en données de package de dessin pour des cartes intérieures.

* [Service Tileset](/rest/api/maps/tileset). Utilisez le service Tileset pour créer une représentation vectorielle d’un jeu de données. Les applications peuvent utiliser un tileset pour présenter une vue basée sur des vignettes du jeu de données.

* [Service d’état de la fonctionnalité](/rest/api/maps/featurestate). Le service d’état de la fonctionnalité permet de prendre en charge l’application de style de carte dynamique. L’application de style de carte dynamique permet aux applications de refléter des événements en temps réel sur des espaces fournis par des systèmes IoT.

* [Service WFS](/rest/api/maps/featurestate). Le service WFS vous permet d’interroger vos données de carte intérieure. Le service WFS respecte les normes de l’[API Open Geospatial Consortium](http://docs.opengeospatial.org/is/17-069r3/17-069r3.html) pour l’interrogation d’un jeu de données.

### <a name="elevation-service-preview"></a>Service Elevation (préversion)

Le service Elevation d’Azure Maps est un service web dont les développeurs peuvent se servir pour récupérer les données d’élévation de n’importe quel lieu sur la surface de la Terre.

Le service Elevation vous permet de récupérer les données d’élévation dans deux formats :

* **Format raster GeoTIFF**. Utilisez l’[API Render V2 – Get Map Tile](/rest/api/maps/renderv2) pour récupérer les données d’élévation au format vignette.

* **Format GeoJSON**. Utilisez les [API Elevation](/rest/api/maps/elevation) pour demander des données d’élévation échantillonnées présentes sur des tracés, dans un cadre englobant défini ou à des coordonnées précises. 

:::image type="content" source="./media/about-azure-maps/elevation.png" alt-text="Exemple de carte avec des données d’élévation":::


## <a name="programming-model"></a>Modèle de programmation

Azure Maps est conçu pour la mobilité et peut vous permettre de développer des applications multiplateformes. Il utilise un modèle de programmation indépendant du langage et il prend en charge les sorties au format JSON via des [API REST](/rest/api/maps/).

De plus, Azure Maps offre un [contrôle de carte JavaScript](/javascript/api/azure-maps-control) pratique avec un modèle de programmation simple. Le développement est simple et rapide pour les applications web et mobiles.





## <a name="power-bi-visual"></a>Visuel Power BI

Le visuel Azure Maps pour Power BI fournit un ensemble complet de visualisations de données pour les données spatiales sur une carte. On estime que plus de 80 % des données d’entreprise possèdent un contexte d’emplacement. Le visuel Azure Maps offre une solution sans code permettant d’obtenir des insights sur la façon dont ce contexte de localisation est associé aux données de votre entreprise et les influence.

:::image type="content" source="./media/about-azure-maps/intro-power-bi.png" border="false" alt-text="Power BI Desktop avec le visuel Azure Maps affichant des données d’entreprise":::

Pour plus d’informations, consultez la documentation [Bien démarrer avec le visuel Azure Maps pour Power BI](power-bi-visual-getting-started.md).

## <a name="usage"></a>Usage

Pour accéder aux services Azure Maps, accédez au [portail Azure](https://portal.azure.com) et créez un compte Azure Maps.

Azure Maps utilise un schéma d’authentification basé sur une clé. Lorsque vous créez votre compte, deux clés sont générées. Pour vous authentifier auprès des services Azure Maps, vous pouvez utiliser celle de votre choix.

Remarque : Azure Maps partage les demandes d’adresse/de lieu fournies par le client (« demandes ») avec la société tierce TomTom pour la fonctionnalité de carte. Les demandes ne sont liées à aucun client ou utilisateur final quand elles sont partagées avec TomTom et ne peuvent pas être utilisées pour identifier des individus. Les services Mobility et Météo, qui incluent l’intégration avec Moovit et AccuWeather, sont actuellement en [PRÉVERSION](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Microsoft est en train d’ajouter TomTom, Moovit et AccuWeather à la liste des sous-traitants de services en ligne.

## <a name="supported-regions"></a>Régions prises en charge

Les services Azure Maps sont actuellement disponibles, sauf dans les pays/régions suivants :

* Chine
* Corée du Sud

Vérifiez que la localisation de votre adresse IP actuelle se trouve dans un pays/une région pris en charge.

## <a name="next-steps"></a>Étapes suivantes

Essayez un exemple d’application qui montre Azure Maps :

[Démarrage rapide : Créer une application web](quick-demo-map-app.md)

Restez informé sur Azure Maps :

[Blog Azure Maps](https://azure.microsoft.com/blog/topics/azure-maps/)