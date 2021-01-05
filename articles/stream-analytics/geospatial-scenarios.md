---
title: Geofencing et agrégation géospatiale avec Azure Stream Analytics
description: Cet article explique comment utiliser Azure Stream Analytics pour le geofencing et l’agrégation géospatiale.
author: mamccrea
ms.author: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 04/02/2019
ms.openlocfilehash: 90d8c70ac0a34470e8121d2234fd6b898cfed1ab
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93130593"
---
# <a name="geofencing-and-geospatial-aggregation-scenarios-with-azure-stream-analytics"></a>Scénarios de geofencing et d’agrégation géospatiale avec Azure Stream Analytics

Avec ses fonctions géospatiales intégrées, Azure Stream Analytics vous permet de créer des applications destinées à des scénarios de gestion de flottes, de covoiturage, de voitures connectées, de suivi de ressources, etc.

## <a name="geofencing"></a>Geofencing

Azure Stream Analytics prend en charge les calculs de geofencing en temps réel et à faible latence dans le cloud et sur le runtime IoT Edge.

### <a name="geofencing-scenario"></a>Scénarios de geofencing

Une entreprise de fabrication a besoin d’assurer le suivi des ressources dans ses bâtiments. Elle a équipé chaque appareil d’un système GPS et souhaite recevoir une notification si un appareil quitte une zone déterminée.

Parmi les données de référence utilisées dans cet exemple figurent des informations de limites géographiques (ou « geofence ») pour les bâtiments et les appareils autorisés dans chaque bâtiment. Rappelez-vous que les données de référence peuvent être statiques ou à évolution lente. Les données de référence utilisées dans ce scénario sont de type statique. Un flux de données émet en continu l’ID des appareils et leur position du moment.

### <a name="define-geofences-in-reference-data"></a>Définir des limites géographiques dans les données de référence

Il est possible de définir des limites géographiques (« geofence ») à l’aide d’un objet GeoJSON. Pour les tâches effectuées avec la version de compatibilité 1.2 et supérieure, les limites géographiques peuvent aussi être définies en utilisant WKT (Well Known Text) comme `NVARCHAR(MAX)`. WKT est un standard de l’Open Geospatial Consortium (OGC) qui sert à représenter les données spatiales dans un format texte.

Les fonctions géospatiales intégrées peuvent utiliser des limites géographiques définies pour déterminer si un élément se trouve à l’intérieur ou à l’extérieur des limites géographiques d’un polygone particulier.

Le tableau suivant est un exemple de données de référence de limites géographiques qui pourraient être stockées dans le stockage Blob Azure ou une table Azure SQL. Chaque site est représenté par un polygone géospatial, et chaque appareil est associé à un ID de site autorisé.

|SiteID|SiteName|Geofence|AllowedDeviceID|
|------|--------|--------|---------------|
|1|« Redmond Building 41 »|« POLYGON((-122.1337357922017 47.63782998329432,-122.13373042778369 47.637634793257305,-122.13346757130023 47.637642022530954,-122.13348902897235 47.637508280806806,-122.13361777500506 47.637508280806806,-122.13361241058703 47.63732393354484,-122.13265754417773 47.63730947490855,-122.13266290859576 47.637519124743164,-122.13302232460376 47.637515510097955,-122.13301696018573 47.63764925180358,-122.13272728161212 47.63764925180358,-122.13274873928424 47.63784082716388,-122.13373579220172 47.63782998329432)) »|« B »|
|2|« Redmond Building 40 »|« POLYGON((-122.1336154507967 47.6366745947009,-122.13361008637867 47.636483015064535,-122.13349206918201 47.636479400347675,-122.13349743360004 47.63636372927573,-122.13372810357532 47.63636372927573,-122.13373346799335 47.63617576323771,-122.13263912671528 47.63616491902258,-122.13264985555134 47.63635649982525,-122.13304682248554 47.636367344000604,-122.13305218690357 47.63650831807564,-122.13276250832996 47.636497473929516,-122.13277323716602 47.63668543881025,-122.1336154507967 47.6366745947009)) »|« A »|
|3|« Redmond Building 22 »|« POLYGON((-122.13611660248233 47.63758544698554,-122.13635263687564 47.6374083293018,-122.13622389084293 47.63733603619712,-122.13622389084293 47.63717699101473,-122.13581619507266 47.63692757827657,-122.13559625393344 47.637046862778135,-122.13569281345798 47.637144458985965,-122.13570890671207 47.637314348246214,-122.13611660248233 47.63758544698554)) »|« C »|

### <a name="generate-alerts-with-geofence"></a>Générer des alertes avec les limites géographiques

Les appareils peuvent émettre leur ID et leur emplacement toutes les minutes via un flux nommé `DeviceStreamInput`. Le tableau suivant est un flux d’entrée.

|DeviceID|GeoPosition|
|--------|-----------|
|« A »|« POINT(-122.13292341559497 47.636318374032726) »|
|« B »|« POINT(-122.13338475554553 47.63743531308874) »|
|« C »|« POINT(-122.13354001095752 47.63627622505007) »|

Vous pouvez écrire une requête qui établit une jonction entre le flux d’appareils et les données de référence de limites géographiques et qui génère une alerte chaque fois qu’un appareil est à l’extérieur d’un bâtiment autorisé.

```SQL
SELECT DeviceStreamInput.DeviceID, SiteReferenceInput.SiteID, SiteReferenceInput.SiteName 
INTO Output
FROM DeviceStreamInput 
JOIN SiteReferenceInput
ON st_within(DeviceStreamInput.GeoPosition, SiteReferenceInput.Geofence) = 0
WHERE DeviceStreamInput.DeviceID = SiteReferenceInput.AllowedDeviceID
```

L’image suivante représente les limites géographiques. Vous pouvez voir où se trouvent les appareils en fonction de l’entrée de données de flux.

![Limites géographiques des bâtiments](./media/geospatial-scenarios/building-geofences.png)

L’appareil « C » se trouve à l’intérieur du bâtiment numéro 2, ce qui n’est pas autorisé d’après les données de référence. Cet appareil devrait se trouver à l’intérieur du bâtiment numéro 3. L’exécution de cette tâche génère une alerte pour cette violation spécifique.

### <a name="site-with-multiple-allowed-devices"></a>Site avec plusieurs appareils autorisés

Si un site autorise plusieurs appareils, un tableau d’ID d’appareil peut être défini dans `AllowedDeviceID` et une fonction définie par l’utilisateur peut être utilisée dans la clause `WHERE` pour vérifier si un ID d’appareil du flux correspond à un ID d’appareil de cette liste. Pour plus d’informations, consultez le tutoriel [Fonctions Javascript définies par l’utilisateur](stream-analytics-javascript-user-defined-functions.md) pour les tâches cloud et le tutoriel [Fonctions C# définies par l’utilisateur](stream-analytics-edge-csharp-udf.md) pour les tâches Edge.

## <a name="geospatial-aggregation"></a>Agrégation géospatiale

Azure Stream Analytics prend en charge l’agrégation géospatiale en temps réel et à faible latence dans le cloud et sur le runtime IoT Edge.

### <a name="geospatial-aggregation-scenario"></a>Scénario d’agrégation géospatiale

Une société de taxis souhaite créer une application en temps réel pour guider leurs chauffeurs dans la recherche de courses dans les zones des villes où la demande est la plus forte.

La société stocke les régions logiques des villes comme données de référence. Chaque région est définie par les attributs RegionID, RegionName et Geofence.

### <a name="define-the-geofences"></a>Définir les limites géographiques

Le tableau suivant est un exemple de données de référence de limites géographiques qui pourraient être stockées dans le stockage Blob Azure ou une table Azure SQL. Toutes les régions sont représentées par un polygone géographique, qui sert à la mise en corrélation avec les demandes en provenance des données de streaming.

Ces polygones servent uniquement de référence et ne visent pas à séparer la ville de façon logique ou physique.

|RegionID|RegionName|Geofence|
|--------|----------|--------|
|1|« SoHo »|« POLYGON((-74.00279525078275 40.72833625216264,-74.00547745979765 40.721929158663244,-74.00125029839018 40.71893680218994,-73.9957785919998 40.72521409075776,-73.9972377137039 40.72557184584898,-74.00279525078275 40.72833625216264)) »|
|2|« Chinatown »|« POLYGON((-73.99712367114876 40.71281582267133,-73.9901070123658 40.71336881907936,-73.99023575839851 40.71452359088633,-73.98976368961189 40.71554823078944,-73.99551434573982 40.717337246783735,-73.99480624255989 40.718491949759304,-73.99652285632942 40.719109951574,-73.99776740131233 40.7168005470334,-73.99903340396736 40.71727219249899,-74.00193018970344 40.71938642421256,-74.00409741458748 40.71688186545551,-74.00051398334358 40.71517415773184,-74.0004281526551 40.714377212470005,-73.99849696216438 40.713450141693166,-73.99748845157478 40.71405192594819,-73.99712367114876 40.71281582267133)) »|
|3|« Tribeca »|« POLYGON((-74.01091641815208 40.72583120006787,-74.01338405044578 40.71436586362705,-74.01370591552757 40.713617702123415,-74.00862044723533 40.711308107057235,-74.00194711120628 40.7194238654018,-74.01091641815208 40.72583120006787)) »|

### <a name="aggregate-data-over-a-window-of-time"></a>Agréger des données sur une fenêtre de temps

Le tableau suivant contient des données de streaming de « courses ».

|UserID|FromLocation|ToLocation|TripRequestedTime|
|------|------------|----------|-----------------|
|« A »|« POINT(-74.00726861389182 40.71610611981975) »|« POINT(-73.98615095917779 40.703107386025835) »|« 2019-03-12T07:00:00Z »|
|« B »|« POINT(-74.00249841021645 40.723827238895666) »|« POINT(-74.01160699942085 40.71378884930115) »|« 2019-03-12T07:01:00Z »|
|« C »|« POINT(-73.99680120565864 40.716439898624024) »|« POINT(-73.98289663412544 40.72582343969828) »|« 2019-03-12T07:02:00Z »|
|« D »|« POINT(-74.00741090068288 40.71615626755086) »|« POINT(-73.97999843120539 40.73477895807408) »|« 2019-03-12T07:03:00Z »|

La requête suivante établit une jonction entre le flux d’appareils et les données de référence de limites géographiques et calcule le nombre de demandes par région sur une fenêtre de temps de 15 minutes toutes les minutes.

```SQL
SELECT count(*) as NumberOfRequests, RegionsRefDataInput.RegionName 
FROM UserRequestStreamDataInput
JOIN RegionsRefDataInput 
ON st_within(UserRequestStreamDataInput.FromLocation, RegionsRefDataInput.Geofence) = 1
GROUP BY RegionsRefDataInput.RegionName, hoppingwindow(minute, 1, 15)
```

Cette requête génère un nombre de demandes toutes les minutes pour les 15 dernières minutes et pour chaque région de la ville. Ces informations peuvent être facilement affichées dans le tableau de bord Power BI ou diffusées à tous les chauffeurs par SMS via une intégration avec des services comme Azure Functions.

L’image ci-dessous illustre la sortie de la requête dans le tableau de bord Power BI. 

![Sortie du résultat dans le tableau de bord Power BI](./media/geospatial-scenarios/power-bi-output.png)


## <a name="next-steps"></a>Étapes suivantes

* [Présentation des fonctions géospatiales Stream Analytics](stream-analytics-geospatial-functions.md)
* [Fonctions géospatiales (Azure Stream Analytics)](/stream-analytics-query/geospatial-functions)