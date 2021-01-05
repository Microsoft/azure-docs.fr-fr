---
title: Glossaire Azure Maps | Microsoft Docs
description: Glossaire comprenant des termes couramment utilisés dans Azure Maps, les services basés sur l’emplacement et GIS.
author: rbrundritt
ms.author: richbrun
ms.date: 09/18/2018
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.openlocfilehash: 05071be4fc1705787a649d331613e8d9399f983f
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92897175"
---
# <a name="glossary"></a>Glossaire

La liste suivante décrit les mots courants utilisés avec les services Azure Maps.

## <a name="a"></a>Un

<a name="address-validation"></a> **Validation d’adresse**  : Processus de vérification de l’existence d’une adresse.

<a name="advanced-routing"></a> **Routage avancé**  : Collection de services qui effectuent des opérations avancées à l’aide de données de routage, telles que le calcul de distances accessibles (isochrones), les matrices de distance et les demandes d’itinéraires par lot.

<a name="aerial-imagery"></a> **Imagerie aérienne**  : Voir [Imagerie satellite](#satellite-imagery). 

<a name="along-a-route-search"></a> **Recherche sur la route**  : Requête spatiale recherchant des données à une durée ou distance de détour spécifiées par rapport à un itinéraire.

<a name="altitude"></a> **Altitude**  : Hauteur ou élévation verticale d’un point au-dessus d’une surface de référence. Les mesures d’altitude sont basées sur une donnée de référence donnée, telle que le niveau moyen de la mer. Voir également Élévation.

<a name="ambiguous"></a> **Ambigu**  : État d’incertitude dans la classification des données qui existe quand un objet peut être assigné de façon justifiée à deux valeurs ou plus d’un attribut donné. Par exemple, le géocodage de « CA » retourne deux résultats ambigus : « Canada » et « Californie ». « CA » est un pays/une région et un code d’État, respectivement pour « Canada » et « Californie ». 

<a name="annotation"></a> **Annotation**  : Texte ou graphique affichés sur la carte pour fournir des informations à l’utilisateur. Une annotation peut identifier ou décrire une entité de carte, fournir des informations générales sur une zone de la carte ou sur la carte elle-même.

<a name="antimeridian"></a> **Antiméridien**  : Également appelée 180 <sup>e</sup> méridien. Il s’agit du point d’intersection entre -180 degrés et 180 degrés de longitude. Ce qui constitue l’opposé du méridien origine sur le globe.

<a name="application-programming-interface-api"></a> **Interface de programmation d’applications (API)**  : Spécification qui permet aux développeurs de créer des applications.

<a name="api-key"></a> **Clé API**  : Consultez [Authentification par clé partagée](#shared-key-authentication).

<a name="area-of-interest-aoi"></a> **Zone d’intérêt**  : Étendue utilisée afin de définir un domaine stratégique pour la production d’une carte ou d’une base de données.

<a name="asset-tracking"></a> **Suivi des ressources**  : Processus de suivi de l’emplacement d’une ressource, telle qu’une personne, un véhicule ou tout autre objet.

<a name="asynchronous-request"></a> **Requête asynchrone**  : Requête HTTP qui ouvre une connexion et adresse une requête au serveur qui retourne un identificateur pour la requête asynchrone, puis ferme la connexion. Le serveur continue de traiter la requête et l’utilisateur peut en vérifier l’état à l’aide de l’identificateur. Lorsque le traitement de la requête est terminé, l’utilisateur peut télécharger la réponse. Ce type de requête est utilisé pour les processus durables.

<a name="autocomplete"></a> **Saisie semi-automatique**  : Fonctionnalité d'une application qui anticipe la fin d'un mot que l'utilisateur est en train de taper. 

<a name="autosuggest"></a> **Suggestion automatique**  : Fonctionnalité d'une application qui anticipe de manière logique ce que l'utilisateur est en train de taper.

<a name="azure-location-based-services-lbs"></a> **Azure Location Based Services (LBS)**  : Ancien nom d’Azure Maps à l’époque de sa préversion.

<a name="azure-active-directory"></a> **Azure Active Directory (Azure AD)**  : Azure AD est le service Microsoft basé sur le cloud qui gère les identités et les accès. L’intégration Azure AD d’Azure Maps est actuellement disponible en préversion pour toutes les API Azure Maps. Azure AD prend en charge le contrôle d’accès en fonction du rôle Azure (RBAC Azure) pour permettre un accès fin aux ressources Azure Maps. Pour en savoir plus sur l’intégration Azure AD d’Azure Maps, consultez [Azure Maps et Azure AD](azure-maps-authentication.md) et [Gérer l’authentification dans Azure Maps](how-to-manage-authentication.md).

<a name="azure-maps-key"></a> **Clé Azure Maps**  : Consultez [Authentification par clé partagée](#shared-key-authentication).

## <a name="b"></a>B

<a name="base-map"></a> **Carte de base**  : Partie d’une application de carte qui affiche des informations de référence en arrière-plan comme les routes, les points de repère et les frontières politiques.

<a name="batch-request"></a> **Requête de lot**  : Processus de combinaison de plusieurs requêtes en une seule requête.

<a name="bearing"></a> **Repère**  : Direction horizontale d’un point par rapport à un autre point. Elle est exprimée par un angle relatif vers le nord, de 0 degré à 360 degrés dans le sens des aiguilles d’une montre. 

<a name="boundary"></a> **Délimitation**  : Une ligne ou un polygone séparant les entités politiques adjacentes, comme les pays/régions, les provinces et les propriétés. Une délimitation est une ligne qui peut suivre ou non des caractéristiques physiques telles que des rivières, des montagnes ou des murs.

<a name="bounds"></a> **Limites**  : Voir [Rectangle englobant](#bounding-box).

<a name="bounding-box"></a> **Rectangle englobant**  : Ensemble de coordonnées utilisé pour représenter une zone rectangulaire sur la carte. 

## <a name="c"></a>C

<a name="cadastre"></a> **Cadastre**  : Enregistrement de terrains et de propriétés officiels. Voir aussi [Parcelle](#parcel).

<a name="camera"></a> **Appareil photo**  : Dans le contexte d’un contrôle de carte interactive, un appareil photo définit le champ de vision de la carte. La fenêtre d’affichage de l’appareil photo est déterminée en fonction de plusieurs paramètres de carte : centre, niveau de zoom, tonalité, repère. 

<a name="centroid"></a> **Centroïde**  : Centre géométrique d’une caractéristique. Le centroïde d’une ligne est le point central, tandis que le centroïde d’un polygone représente le centre de sa zone.

<a name="choropleth-map"></a> **Carte choroplèthe**  : Carte thématique dans laquelle des zones sont formées proportionnellement à la mesure d’une variable statistique. Cette variable statistique est affichée sur la carte. Par exemple, la coloration de la frontière d’un état américain selon sa population relative comparée à tous les autres états.

<a name="concave-hull"></a> **Enveloppe concave**  : Forme qui représente une géométrie concave possible englobant toutes les formes dans le jeu de données spécifié. La forme ainsi générée pourrait se décrire par un ensemble de données entouré d’une bande plastique que l’on fait chauffer, ce qui crée de grandes étendues entre les points et vers d’autres points de données.

<a name="consumption-model"></a> **Modèle de consommation**  : Informations qui définissent le taux de consommation en carburant ou électricité d’un véhicule. Voir aussi la [documentation sur le modèle de consommation](consumption-model.md).

<a name="control"></a> **Contrôle**  : Composant autonome ou réutilisable consistant en une interface graphique utilisateur qui définit un ensemble de comportements pour l’interface. Par exemple, un contrôle de carte est généralement la portion de l’interface utilisateur qui charge une carte interactive.

<a name="convex-hull"></a> **Enveloppe convexe**  : Forme qui représente une géométrie convexe minimale englobant toutes les formes dans le jeu de données spécifié. La forme ainsi générée pourrait se décrire par un ensemble de données entouré d’une bande élastique.

<a name="coordinate"></a> **Coordonnées**  : Valeurs de longitude et latitude utilisées pour représenter un emplacement sur la carte.

<a name="coordinate-system"></a> **Système de coordonnées**  : Cadre de référence utilisé pour définir les positions des points dans l’espace en deux ou trois dimensions.

<a name="country-code"></a> **Code pays**  : Un identificateur unique d’un pays/d’une région selon la norme ISO. ISO2 est un code à deux caractères pour un pays/une région (par exemple, US), et ISO3 est un code à trois caractères (par exemple, USA).

<a name="country-subdivision"></a> **Sous-division de pays**  : Une sous-division de premier niveau d’un pays/d’une région, couramment appelée état ou province.

<a name="country-secondary-subdivision"></a> **Sous-division secondaire de pays**  : Une sous-division de deuxième niveau d’un pays/d’une région, couramment appelée comté.

<a name="country-tertiary-subdivision"></a> **Sous-division tertiaire de pays**  : Une sous-division de troisième niveau d’un pays/d’une région ; en général, cette zone est appelée circonscription.

<a name="cross-street"></a> **Croisement**  : Point où deux rues ou plus se croisent.

<a name="cylindrical-projection"></a> **Projection cylindrique**  : Projection qui transforme les points d’un sphéroïde ou d’une sphère sur une tangente ou un cylindre sécant. Le cylindre est découpé de haut en bas et aplati en un plan.

## <a name="d"></a>D

<a name="datum"></a> **Donnée**  : Caractéristiques de référence d’un système de mesure, système de positions de coordonnées sur une surface (donnée horizontale) ou de hauteurs au-dessus ou en dessous d’une surface (donnée verticale).

<a name="dbf-file"></a> **Fichier DBF**  : Format de fichier de base de données utilisé avec des fichiers de forme (SHP).

<a name="degree-minutes-seconds-dms"></a> **Degré minutes secondes (DMS)**  : Unité de mesure décrivant la latitude et la longitude. Un degré correspond à 1/360<sup>e</sup> d’un cercle. Un degré est en plus divisé en 60 minutes, et une minute divisée en 60 secondes.

<a name="delaunay-triangulation"></a> **Triangulation de Delaunay**  : Technique pour créer un maillage de triangles contigus, sans chevauchement, à partir d’un jeu de données de points. Chaque cercle entourant un triangle ne contient aucun point du jeu de données à l’intérieur.

<a name="demographics"></a> **Données démographiques**  : Caractéristiques statistiques (telles que l’âge, le taux de natalité et le revenu) d’une population humaine.

<a name="destination"></a> **Destination**  : Point ou lieu d’arrivée vers lequel une personne voyage.

<a name="digital-elevation-model-dem"></a> **Modèle d’élévation numérique (DEM)**  : Jeu de données de valeurs d’élévation associées à une surface, capturées sur une zone dans des intervalles réguliers avec une donnée commune. Les modèles d’élévation numérique sont en général utilisés pour représenter le relief d’un terrain.

<a name="dijkstra's-algorithm"></a> **Algorithme de Dijkstra**  : Algorithme qui examine la connectivité d’un réseau pour trouver le chemin le plus court entre deux points.

<a name="distance-matrix"></a> **Matrice des distances**  : Matrice qui contient des informations sur la durée et la distance de trajet entre un ensemble de départs et de destinations. 

## <a name="e"></a>E

<a name="elevation"></a> **Élévation**  : Distance verticale d’un point ou d’un objet au-dessus ou en dessous d’une surface ou donnée de référence. En règle générale, la surface de référence correspond au niveau moyen de la mer. L’élévation fait en général référence à la hauteur verticale de la terre.

<a name="envelope"></a> **Enveloppe**  : Voir [Rectangle englobant](#bounding-box).

<a name="extended-postal-code"></a> **Code postal étendu**  : Code postal qui peut inclure des informations supplémentaires. Par exemple, aux États-Unis, les codes postaux comportent cinq chiffres. Toutefois, un code postal étendu, appelé zip+4, comprend quatre chiffres supplémentaires. Ces chiffres supplémentaires servent à identifier un segment géographique au sein de la zone de livraison à cinq chiffres, comme le quartier d’une ville, un ensemble d’appartements ou une boîte de bureau de poste. La connaissance du segment géographique facilite le tri et la livraison du courrier.

<a name="extent"></a> **Étendue**  : Voir [Rectangle englobant](#bounding-box).

## <a name="f"></a>F

<a name="federated-authentication"></a> **Authentification fédérée**  : Méthode d’authentification qui permet l’utilisation d’un mécanisme d’ouverture de session/authentification unique dans plusieurs applications web et mobiles. 

<a name="feature"></a> **Caractéristique**  : Objet qui combine une géométrie et des informations de métadonnées supplémentaires. 

<a name="feature-collection"></a> **Collection de caractéristiques**  : Collection d’objets de caractéristiques.

<a name="find-along-route"></a> **Trouver sur l’itinéraire**  : Requête spatiale recherchant des données qui se trouvent à une durée ou distance de détour spécifiées par rapport à un itinéraire.

<a name="find-nearby"></a> **Trouver à proximité**  : Requête spatiale qui effectue une recherche dans une zone située à une distance linéaire fixe (à vol d’oiseau) à partir d’un point.

<a name="fleet-management"></a> **Gestion de flotte**  : Gestion des véhicules commerciaux tels que voitures, camions, bateaux et avions. La gestion de flotte peut comprendre un éventail de fonctions, telles que le financement de véhicule, l’entretien, la télématique (suivi et diagnostics), ainsi que la gestion du conducteur, de la vitesse, du carburant, de l’intégrité et de la sécurité. La gestion de flotte est un processus utilisé par les entreprises dont l’activité repose sur le transport. Les entreprises souhaitent limiter les risques et réduire les coûts de personnel et de transport, tout en garantissant le respect des normes gouvernementales en vigueur.

<a name="free-flow-speed"></a> **Vitesse de libre circulation**  : Vitesse de libre circulation prévue dans des conditions idéales. Il s’agit généralement de la limitation de vitesse.

<a name="free-form-address"></a> **Adresse de forme libre**  : Adresse complète représentée par une seule ligne de texte.

<a name="fuzzy-search"></a> **Recherche approximative**  : Recherche qui accepte une chaîne de texte pouvant être une adresse ou un point d’intérêt. 

## <a name="g"></a>G

<a name="geocode"></a> **Géocode**  : Adresse ou emplacement convertis en coordonnée pouvant être utilisée pour afficher cet emplacement sur une carte. 

<a name="geocoding"></a> **Géocodage**  : Également appelé géocodage avant. Il s’agit du processus de conversion des données d’une adresse de localisation en coordonnées. 

<a name="geodesic-path"></a> **Chemin géodésique**  : Chemin le plus court entre deux points sur une surface courbe. Lors du rendu sur Azure Maps, ce chemin apparaît comme ligne courbe à cause de la projection de Mercator.

<a name="geofence"></a> **Limite géographique**  : Région géographique définie pouvant être utilisée pour déclencher des événements quand un appareil y entre ou en sort.

<a name="geojson"></a> **GeoJSON**  : Format de fichier JSON couramment utilisé pour stocker des données vectorielles géographiques comme des points, des lignes et des polygones. **Remarque**  : Azure Maps utilise une version étendue de GeoJSON comme [décrit ici](extend-geojson.md).

<a name="geometry"></a> **Géométrie**  : Représente un objet spatial, comme un point, une ligne ou un polygone.

<a name="geometrycollection"></a> **Collection de géométries**  : Collection d’objets de géométrie.

<a name="geopol"></a> **GeoPol**  : Fait référence à des données géopolitiques sensibles, comme des frontières et noms de lieux contestés.

<a name="georeference"></a> **Référence géographique**  : Processus d’alignement des données ou imageries géographiques sur un système de coordonnées connu. Ce processus comprendre des déplacements, des rotations, des mises à l’échelle ou des inclinaisons de données.

<a name="georss"></a> **GeoRSS**  : Extension XML pour l’ajout de données spatiales aux flux RSS.

<a name="gis"></a> **GIS**  : Acronyme de Geographic Information System (Système d’information géographique). Terme courant utilisé pour décrire le domaine de cartographie.

<a name="gml"></a> **GML**  : Également appelé Geography Markup Language. Extension de fichier XML pour le stockage de données spatiales.

<a name="gps"></a> **GPS**  : Également appelé Global Positioning System. Il s’agit d’un système de satellites utilisé pour déterminer la position d’un appareil sur la Terre. Les satellites en orbite transmettent des signaux permettant à un récepteur GPS situé n’importe où sur terre de calculer sa position par trilatération.

<a name="gpx"></a> **GPX**  : Également connu sous le nom de format d’échange GPS. Il s’agit d’un format de fichier XML couramment créé à partir d’appareils GPS.  

<a name="great-circle-distance"></a> **Distance orthodromique**  : Distance la plus courte entre deux points sur la surface d’une sphère.

<a name="greenwich-mean-time-gmt"></a> **Heure moyenne de Greenwich (GMT)**  : Heure au méridien d’origine, indiquée à l’Observatoire royal de Greenwich, en Angleterre.

<a name="guid"></a> **GUID**  : Identificateur global unique. Une chaîne utilisée pour identifier uniquement une interface, une classe, une bibliothèque de types, une catégorie de composant ou un enregistrement.

## <a name="h"></a>H

<a name="haversine-formula"></a> **Formule de Haversine**  : Équation couramment utilisée pour calculer la distance orthodromique entre deux points sur une sphère.

<a name="hd-maps"></a> **Cartes HD**  : Également appelées cartes haute définition. Elles comprennent des informations fiables sur le réseau de circulation, comme les marquages au sol, la signalisation et les feux clignotants, nécessaires à une conduite autonome.

<a name="heading"></a> **Direction**  : Direction dans laquelle pointe un élément, ou à laquelle un élément fait face. Voir aussi [Repère](#heading).

<a name="heatmap"></a> **Carte thermique**  : Visualisation des données dans laquelle une gamme de couleurs représente la densité des points dans une zone particulière. Voir aussi Carte thématique.

<a name="hybrid-imagery"></a> **Imagerie hybride**  : Imagerie aérienne ou satellite avec des données et étiquettes de route superposées.

## <a name="i"></a>I

<a name="iana"></a> **IANA**  : Acronyme d’Internet Assigned Numbers Authority. Un groupe à but non lucratif qui supervise la gestion de l’espace d’adressage IP d’Internet.

<a name="isochrone"></a> **Isochrone**  : Un isochrone définit la zone dans laquelle une personne peut voyager dans un délai spécifié et pour un mode de transport particulier, dans n’importe quelle direction, à partir d’un emplacement donné. Voir aussi [Distance accessible](#reachable-range).

<a name="isodistance"></a> **Isodistance**  : À partir d’un emplacement donné, un isochrone définit la zone dans laquelle une personne peut voyager dans un délai spécifié et pour un mode de transport particulier, dans n’importe quelle direction. Voir aussi [Distance accessible](#reachable-range).

## <a name="k"></a>K

<a name="kml"></a> **KML**  : Acronyme de Keyhole Markup Language. Il s’agit d’un format de fichier XML commun pour le stockage de données vectorielles géographiques comme des points, des lignes et des polygones. 

## <a name="l"></a>L

<a name="landsat"></a> **Landsat**  : Satellites multi-spectraux en orbite autour de la Terre, développés par la NASA, qui collectent des images terrestres. Ces images sont utilisées dans plusieurs domaines comme l’agriculture, la sylviculture et la cartographie.

<a name="latitude"></a> **Latitude**  : Distance angulaire mesurée en degrés à partir de l’Équateur dans une direction nord ou sud.

<a name="level-of-detail"></a> **Niveau de détail**  : Voir Niveau de zoom.

<a name="lidar"></a> **Lidar**  : Acronyme de Light Detection And Ranging. Détection et localisation par la lumière. Une technique de détection à distance qui utilise des lasers pour mesurer des distances sur des surfaces réfléchissantes.

<a name="linear-interpolation"></a>**Interpolation linéaire**  : Estimation d’une valeur inconnue à l’aide de la distance linéaire entre des valeurs connues.

<a name="linestring"></a> **LineString**  : Géométrie utilisée pour représenter une ligne. Aussi appelée polyligne. 

<a name="localization"></a> **Localisation**  : Prise en charge de différentes langues et cultures.

<a name="logistics"></a> **Logistique**  : Processus de déplacement des personnes, des véhicules, des fournitures ou des ressources de façon coordonnée.

<a name="longitude"></a> **Longitude**  : Distance angulaire mesurée en degrés à partir du méridien d’origine dans une direction est ou ouest.

## <a name="m"></a>M

<a name="map-tile"></a> **Mosaïque de la carte**  : Image rectangulaire qui représente une partition d’un canevas de carte. Pour plus d’informations, consultez la [documentation Niveaux de zoom et grille mosaïque](zoom-levels-and-tile-grid.md).

<a name="marker"></a> **Marqueur**  : Également appelé épingle ou punaise. Il s’agit d’une icône qui représente un emplacement sur une carte.

<a name="mercator-projection"></a> **Projection Mercator**  : Projection cartographique cylindrique devenue la norme en termes de projection cartographique en mer en raison de sa capacité à représenter des lignes de cap régulières, aussi appelées lignes de rhumb, comme des segments droits qui conservent l’angle avec les méridiens. Toutes les projections cartographiques plates déforment les formes et les tailles d’une carte lorsqu’elles sont comparées à la surface de la Terre. La projection Mercator exagère les zones éloignées de l’équateur, de telle façon que des zones plus petites apparaissent grandes sur la carte au fur et à mesure que vous vous rapprochez des pôles. 

<a name="multilinestring"></a> **MultiLineString**  : Géométrie qui représente une collection d’objets LineString. 

<a name="multipoint"></a> **MultiPoint**  : Géométrie qui représente une collection d’objets Point.

<a name="multipolygon"></a> **MultiPolygon**  : Géométrie qui représente une collection d’objets Polygon. Par exemple, pour afficher la délimitation d’Hawaï, chaque île serait contournée par un polygone. Ainsi, la délimitation d’Hawaï serait MultiPolygon.

<a name="municipality"></a> **Municipalité**  : Ville ou localité. 

<a name="municipality-subdivision"></a> **Sous-division de municipalité**  : Sous-division d’une municipalité, comme un quartier ou un « centre ville ».

## <a name="n"></a>N

<a name="navigation-bar"></a> **Barre de navigation**  : Ensemble des contrôles d’une carte utilisés pour ajuster le niveau de zoom, le tangage, la rotation et le basculement de la couche de base.

<a name="nearby-search"></a> **Recherche à proximité**  : Requête spatiale qui effectue une recherche dans une zone située à une distance linéaire fixe (à vol d’oiseau) à partir d’un point.

<a name="neutral-ground-truth"></a> **Véritable terrain neutre**  : Carte qui affiche des étiquettes dans la langue officielle de la région qu’elle représente et dans les scripts locaux, le cas échéant.

## <a name="o"></a>O

<a name="origin"></a> **Origine**  : Point ou emplacement de départ auquel se trouve un utilisateur.

## <a name="p"></a>P

<a name="panning"></a> **Panoramique**  : Processus de déplacement de la carte dans n’importe quelle direction tout en conservant un niveau de zoom constant.

<a name="parcel"></a> **Parcelle**  : Délimitation d’un terrain vague ou d’une propriété.

<a name="pitch"></a> **Tangage**  : Inclinaison de la carte par rapport à l’axe vertical où 0 correspond au bas de la carte.

<a name="point"></a> **Point**  : Géométrie qui représente une position unique sur la carte. 

<a name="points-of-interest-poi"></a> **Points d’intérêt (POI)**  : Entreprise, monument ou lieu d’intérêt.

<a name="polygon"></a> **Polygone**  : Géométrie pleine qui représente une zone sur une carte. 

<a name="polyline"></a> **Polyligne**  : Géométrie utilisée pour représenter une ligne. Également appelée LineString. 

<a name="position"></a> **Position**  : Longitude, latitude et altitude (coordonnées x, y, z) d’un point.

<a name="post-code"></a> **Postal**  : Voir [Code postal](#postal-code).

<a name="postal-code"></a> **Code postal**  : Une série de lettres et/ou de chiffres, dans un format spécifique. Le code postal est utilisé par le service postal d’un pays/d’une région pour subdiviser des zones géographiques, afin de simplifier la livraison du courrier.

<a name="primary-key"></a> **Clé primaire**  : La première des deux clés d’abonnements fournies pour l’authentification par clé partagée Azure Maps. Consultez [Authentification par clé partagée](#shared-key-authentication).

<a name="prime-meridian"></a> **Méridien d’origine**  : Ligne de longitude qui représente 0 degré de longitude. En règle générale, les valeurs de longitude diminuent en se déplaçant vers l’ouest jusqu’à 180 degrés, et augmentent en direction de l’est jusqu’à -180 degrés. 

<a name="prj"></a> **PRJ**  : Fichier texte qui accompagne souvent un fichier de forme qui contient des informations sur le système de coordonnées projetées dans lequel se trouve le jeu de données.

<a name="projection"></a> **Projection**  : Système de coordonnées projetées selon une projection cartographique comme Mercator transverse, Albers et Robinson. Elles permettent la projection de cartes de la surface sphérique de la Terre sur un plan de coordonnées Cartésiennes en deux dimensions. Les systèmes de coordonnées projetées sont parfois appelés projections cartographiques.

## <a name="q"></a>Q

<a name="quadkey"></a> **Quadkey**  : Index d’adresse base-4 pour une vignette au sein d’un système de mosaïque Quadtree. Pour plus d’informations, consultez la documentation [Niveaux de zoom et grille mosaïque](zoom-levels-and-tile-grid.md).

<a name="quadtree"></a> **Quadtree**  : Structure de données dans laquelle chaque nœud possède exactement quatre enfants. Le système de mosaïque utilisé dans Azure Maps emploie une structure Quadtree. Ainsi, Quand un utilisateur zoome sur un niveau, chaque mosaïque de la carte se divise en quatre mosaïques secondaires.  Pour plus d’informations, consultez la documentation [Niveaux de zoom et grille mosaïque](zoom-levels-and-tile-grid.md).

<a name="queries-per-second-qps"></a> **Requêtes par seconde**  : Nombre de requêtes ou demandes qui peuvent être soumises à un service ou une plateforme en l’espace d’une seconde. 

## <a name="r"></a>R

<a name="radial-search"></a> **Recherche radiale**  : Requête spatiale qui effectue une recherche dans une zone située à une distance linéaire fixe (à vol d’oiseau) à partir d’un point. 

<a name="raster-data"></a> **Données Raster**  : Matrice de cellules (ou pixels) organisées en lignes et en colonnes (ou en grille) où chaque cellule contient une valeur représentant une information, comme la température. Les données raster incluent des photographies aériennes numériques, des images satellites, des images numériques et des cartes scannées.

<a name="raster-layer"></a> **Couche Raster**  : Couche de mosaïques qui se compose d’images Raster.

<a name="reachable-range"></a> **Distance accessible**  : Une distance accessible définit la zone dans laquelle une personne peut voyager dans un délai spécifié et pour un mode de transport particulier, dans n’importe quelle direction, à partir d’un emplacement donné. Voir aussi [Isochrone](#isochrone) et [Isodistance](#isodistance).

<a name="remote-sensing"></a> **Détection à distance**  : Processus de collecte et d’interprétation des données de capteur à distance.

<a name="rest-service"></a> **Service REST**  : Acronyme de Representational State Transfer. Un service REST est un service web basé sur l’URL qui s’appuie sur la technologie web de base pour communiquer. Les méthodes les plus courantes sont les requêtes HTTP GET et POST. Ces types de services ont tendance à être plus rapides et petits que les services traditionnels SOAP.

<a name="reverse-geocode"></a> **Géocode inversé**  : Processus consistant à prendre une coordonnée et à déterminer l’adresse qu’elle indique sur une carte.

<a name="reproject"></a> **Reprojection**  : Voir [Transformation](#transformation).

<a name="rest-service"></a> **Service REST**  : Acronyme de Representational State Transfer. Une architecture pour l’échange d’informations entre pairs dans un environnement distribué et décentralisé. REST permet aux programmes sur différents ordinateurs de communiquer indépendamment d’un système d’exploitation ou d’une plateforme. Un service peut envoyer une requête HTTP (Hypertext Transfer Protocol) à une URL (Uniform Resource Locator) et récupérer des données.

<a name="route"></a> **Route**  : Chemin entre deux lieux ou plus, qui peut également inclure des informations supplémentaires comme des instructions sur les points de cheminement qui jalonnent l’itinéraire.

<a name="requests-per-second-rps"></a> **Demandes par seconde**  : Voir [Requêtes par seconde](#queries-per-second-qps). 

<a name="rss"></a> **RSS**  : Acronyme de Really Simple Syndication, Resource Description Framework (RDF), Rich Site Summary, selon la source. Un format XML structuré et simple pour le partage de contenus sur différents sites web. Les documents RSS incluent des éléments de métadonnées clés tels que l’auteur, la date, le titre, une courte description et un lien hypertexte. Ces informations permettent à l’utilisateur (ou à un service de publication RSS) de décider des composants qui méritent d’être approfondis.

## <a name="s"></a>S

<a name="satellite-imagery"></a> **Imagerie satellite**  : Images de la Terre prises par des avions et satellites depuis le ciel ou l’espace.

<a name="secondary-key"></a> **Clé secondaire**  : La seconde des deux clés d’abonnements fournies pour l’authentification par clé partagée Azure Maps. Consultez [Authentification par clé partagée](#shared-key-authentication).

<a name="shapefile-shp"></a> **Fichier de forme (SHP)**  : Également appelé fichier de forme ESRI. Il s’agit d’un format de stockage de données vectorielles contenant l’emplacement, la forme et les attributs de caractéristiques géographiques. Un fichier de forme est stocké dans un ensemble de fichiers connexes.

<a name="shared-key-authentication"></a> **Authentification par clé partagée**  : L’authentification par clé partagée s’appuie sur la transmission des clés générées pour les comptes Azure Maps lors de chaque requête envoyée à Azure Maps. Ces clés sont souvent appelées clés d’abonnement. Il est recommandé de regénérer les clés régulièrement pour des raisons de sécurité. Deux clés sont fournies, ce qui signifie que vous pouvez maintenir les connexions à l’aide d’une clé pendant que vous regénérez l’autre clé. Lorsque vous regénérez vos clés, vous devez mettre à jour toutes les applications qui accèdent à ce compte pour qu’elles utilisent les nouvelles clés. Pour en savoir plus sur l’authentification Azure Maps, consultez [Azure Maps et Azure AD](azure-maps-authentication.md) et [Gérer l’authentification dans Azure Maps](how-to-manage-authentication.md).

<a name="software-development-kit-sdk"></a> **Kit de développement logiciel (SDK)**  : Collection de documents, exemples de code et exemples d’application pour aider un développeur à utiliser une API afin de créer des applications.

<a name="spherical-mercator-projection"></a> **Projection de Mercator sphérique**  : Voir [Web Mercator](#web-mercator). 

<a name="spatial-query"></a> **Requête spatiale**  : Requête adressée à un service qui effectue une opération spatiale. Par exemple, une recherche radiale ou une recherche sur la route.

<a name="spatial-reference"></a> **Référence spatiale**  : Système de coordonnées local, régional ou mondial utilisé pour localiser précisément des entités géographiques. Elle définit le système de coordonnées utilisé pour transmettre les coordonnées de la carte indiquant un lieu dans le monde réel. Les références spatiales garantissent l’intégration des données spatiales de différentes couches, ou sources, en vue d’une analyse et d’un affichage précis. Azure Maps utilise le système de référence de coordonnées [EPSG:3857](https://epsg.io/3857) et WGS 84 pour entrer des données géométriques.

<a name="sql-spatial"></a> **SQL spatial**  : Fait référence à la fonctionnalité spatiale intégrée à SQL Azure et SQL Server 2008 et versions ultérieures. Cette fonctionnalité spatiale est également disponible en tant que bibliothèque .NET qui peut être utilisée indépendamment de SQL Server. Pour plus d’informations, consultez la [Documentation sur les données spatiales (SQL Server)](/sql/relational-databases/spatial/spatial-data-sql-server).

<a name="subscription-key"></a> **Clé d’abonnement**  : Consultez [Authentification par clé partagée](#shared-key-authentication).

<a name="synchronous-request"></a> **Requête synchrone**  : Une requête HTTP ouvre une connexion et attend une réponse. Les navigateurs limitent le nombre de requêtes HTTP simultanées effectuées sur une page. Si plusieurs requêtes synchrones durables sont effectuées en même temps, cette limite peut être atteinte. Les requêtes peuvent être retardées jusqu’à la fin d’une des autres requêtes.

## <a name="t"></a>T

<a name="telematics"></a> **Télématique**  : Envoi, réception et stockage des informations par le biais d’appareils de télécommunication conjointement à un contrôle sur les objets distants. 

<a name="temporal-data"></a> **Données temporelles**  : Données qui font spécifiquement référence à des heures ou des dates. Les données temporelles peuvent faire référence à des événements distincts, comme les éclairs; les objets mouvants, comme les trains; ou des observations répétées, comme le compteur des capteurs de trafic.

<a name="terrain"></a> **Terrain**  : Zone ayant une caractéristique particulière, par exemple un caractère sablonneux ou montagneux.

<a name="thematic-maps"></a> **Cartes thématiques**  : Carte simple qui a pour but d’illustrer un thème d’une zone géographique. Un scénario courant pour ce type de carte consistant à colorer les régions administratives telles que des pays/régions, en fonction de certaines métriques de données.

<a name="tile-layer"></a> **Couche de mosaïques**  : Couche affichée en assemblant des mosaïques (sections rectangulaires) en une couche continue. Les vignettes peuvent être des images raster ou de vecteur. Les couches de mosaïques Raster sont généralement rendues à l’avance et sont stockées sous forme d’images sur un serveur. Les couches de mosaïques Raster peuvent utiliser un grand espace de stockage. Les couches de mosaïques vectorielles sont affichées presque en temps réel dans l’application cliente. Ainsi, les exigences de stockage côté serveur sont moindres pour les couches de mosaïques vectorielles.

<a name="time-zone"></a> **Fuseau horaire**  : Région du globe qui observe une heure uniforme à des fins commerciales, sociales et juridiques. Les fuseaux horaires ont tendance à respecter les délimitations des pays/régions et de leurs sous-divisions.

<a name="transaction"></a> **Transaction**  : Azure Maps utilise un modèle de licence transactionnel où :

- une transaction est créée toutes les 15 cartes ou vignettes de trafic demandées.
- une transaction est créée à chaque appel d’API vers l’un des services dans Azure Maps. La recherche et le routage sont des exemples de service Azure Maps.

<a name="transformation"></a> **Transformation**  : Processus de conversion des données entre différents systèmes de coordonnées géographiques. Par exemple, vous pouvez disposer de données capturées au Royaume-Uni et basées sur le système de coordonnées géographiques OSGB 1936. Azure Maps utilise le système de référence de coordonnées [EPSG:3857](https://epsg.io/3857), variante de WGS 84. Ainsi, pour afficher les données correctement, il faudra transformer les coordonnées d’un système à un autre.

<a name="traveling-salesmen-problem-tsp"></a> **Problème du voyageur de commerce**  :  Problème d’optimisation de trajet décrit par Hamilton dans lequel un voyageur de commerce doit trouver le moyen le plus efficace de passer par une série d’endroits avant de retourner à son point de départ.  

<a name="trilateration"></a> **Trilatération**  : Processus de détermination de la position d’un point sur la surface de la Terre, en tenant compte de deux autres points, en mesurant les distances entre ces trois points.

<a name="turn-by-turn-navigation"></a> **Navigation étape par étape**  : Application qui fournit des instructions d’itinéraire pour chaque étape lorsque les utilisateurs approchent de l’étape suivante.

## <a name="v"></a>V

<a name="vector-data"></a> **Données vectorielles**  : Données basées sur des coordonnées qui sont représentées sous forme de points, lignes ou polygones.

<a name="vector-tile"></a> **Vignette de vecteur**  : Spécification de données ouverte pour stocker des données vectorielles géospatiales avec le même système de mosaïques pour contrôler la carte. Voir aussi [Couche de mosaïques](#tile-layer).

<a name="vehicle-routing-problem-vrp"></a> **Problème de tournées de véhicules**  : Classe de problèmes, dans laquelle un ensemble d’itinéraires ordonnés définis pour une flotte de véhicules est calculé en tenant compte d’un ensemble de contraintes. Ces contraintes peuvent inclure des fenêtres de délai de livraison, plusieurs capacités d’acheminement et des contraintes de durée de voyage.

<a name="voronoi-diagram"></a> **Diagramme de Voronoi**  : Division de l’espace en zones ou cellules qui entourent un ensemble d’objets géométriques, généralement des caractéristiques de points. Ces cellules, ou polygones, doivent respecter les critères des triangles Delaunay. Tous les emplacements au sein d’une zone sont plus proches de l’objet elle entoure que de n’importe quel autre objet dans l’ensemble. Les diagrammes Voronoi sont souvent utilisés pour délimiter des zones d’influence autour de caractéristiques géographiques. 

## <a name="w"></a>W

<a name="waypoint"></a> **Point de cheminement**  : Emplacement géographique spécifié, défini par une longitude et une latitude, utilisé à des fins de navigation. Souvent utilisé pour représenter un point par lequel une personne définit un itinéraire.

<a name="waypoint-optimization"></a> **Optimisation du point de cheminement**  : Processus de réorganisation d’un ensemble de points de cheminement afin de réduire la durée ou la distance du trajet nécessaires pour passer par tous les points définis. En fonction de la complexité de l’optimisation, cette optimisation est souvent appelée [Problème voyages commerciaux](#traveling-salesmen-problem-tsp) ou [Problème d’acheminement du véhicule](#vehicle-routing-problem-vrp).

<a name="web-map-service-wms"></a> **Web Map Service (WMS)**  : WMS est une norme OGC (Open Geographic Consortium) qui définit les services de cartographie basée sur l’image. Les services WMS fournissent des images de carte de zones spécifiques au sein d’une carte sur demande. Les images incluent une symbologie prérendue et peuvent être rendues dans l’un des nombreux styles nommés si définis par le service.

<a name="web-mercator"></a> **Web Mercator**  : Également appelé projection de Mercator sphérique. Il s’agit d’une légère variante de la projection Mercator, utilisée principalement dans les programmes de cartographie basés sur le web. Elle utilise les mêmes formules que la projection de Mercator standard et est utilisée pour des cartes à petite échelle. Toutefois, le Web Mercator utilise les formules sphériques à toutes les échelles, mais les cartes Mercator à grande échelle utilisent normalement la forme ellipsoïde de la projection. La différence est imperceptible à l’échelle mondiale, mais elle entraîne une légère déviation sur les cartes locales par rapport aux vraies cartes Mercator ellipsoïdes, à la même échelle.

<a name="wgs84"></a> **WGS84**  : Ensemble de constantes utilisées pour faire correspondre des coordonnées spatiales à des lieux à la surface de la carte. La donnée WGS84 est la norme utilisée par la plupart des fournisseurs de cartes en ligne et appareils GPS. Azure Maps utilise le système de référence de coordonnées [EPSG:3857](https://epsg.io/3857), variante de WGS 84.

## <a name="z"></a>Z

<a name="z-coordinate"></a> **Coordonnée Z**  : Voir [Altitude](#altitude). 

<a name="zip-code"></a> **Code ZIP**  : Voir [Code postal](#postal-code).

<a name="Zoom level"></a> **Niveau de zoom**  : Spécifie le niveau de détail et la partie visible de la carte. Avec un zoom jusqu’au niveau 0, la carte du monde est souvent complètement visible. Toutefois, elle n’affiche que des informations limitées comme le nom des pays/régions et les frontières ainsi que les noms des océans. Avec un zoom plus proche du niveau 17, la carte affiche une zone de quelques rues d’une ville avec des informations routières détaillées. Dans Azure Maps, le niveau de zoom le plus élevé est 22. Pour plus d’informations, consultez la [documentation Niveaux de zoom et grille mosaïque](zoom-levels-and-tile-grid.md).