---
title: Concepts relatifs au service Météo (préversion) de Microsoft Azure Maps
description: Découvrez les concepts qui s’appliquent au service Météo (préversion) de Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 09/10/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 657cb6a86122d267b86e82b6f02eb58e968cddd3
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96904735"
---
# <a name="weather-services-preview-in-azure-maps"></a>Service Météo (préversion) d’Azure Maps

> [!IMPORTANT]
> Le service Météo d’Azure Maps est disponible en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Cet article présente les concepts qui s’appliquent au [service Météo](/rest/api/maps/weather) d’Azure Maps. Nous vous recommandons de consulter cet article avant de commencer à utiliser les API météo.

## <a name="unit-types"></a>Types d’unités

Certaines API du service Météo (préversion) permettent à l’utilisateur de spécifier si les données doivent être retournées en unités métriques ou en unités impériales. Les réponses renvoyées pour ces API comprennent unitType et une valeur numérique qui peut être utilisée pour les conversions d’unités. Pour interpréter ces valeurs, consultez le tableau ci-dessous.

|unitType|Description         |
|--------|--------------------|
|0       |pieds                |
|1       |pouces              |
|2       |milles               |
|3       |millimètre          |
|4       |centimètre          |
|5       |mètre               |
|6       |kilomètre           |
|7       |kilometersPerHour   |
|8       |nœuds               |
|9       |milesPerHour        |
|10      |metersPerSecond     |
|11      |hectoPascals        |
|12      |inchesOfMercury     |
|13      |kiloPascals         |
|14      |millibars           |
|15      |millimetersOfMercury|
|16      |poundsPerSquareInch |
|17      |celsius             |
|18      |fahrenheit          |
|19      |kelvin              |
|20      |pour cent             |
|21      |float               |
|22      |entier             |


## <a name="weather-icons"></a>Icônes météo

Certaines API du service Météo retournent `iconCode` dans la réponse. `iconCode` est une valeur numérique utilisée pour définir l’icône. Ne créez pas de lien direct avec ces images à partir de vos applications, les URL peuvent changer et changeront.

| Numéro d’icône |Icône| jour | Nuit | Texte |
|-------------|:----:|-----|-------|------|
| 1           | :::image type="icon" source="./media/weather-services-concepts/sunny-i.png"::: | Oui |  Non    | Ensoleillé|
| 2           | :::image type="icon" source="./media/weather-services-concepts/mostly-sunny.png"::: | Oui |  Non    | Principalement ensoleillé|
| 3           | :::image type="icon" source="./media/weather-services-concepts/partly-sunny.png"::: | Oui |  Non    | Partiellement ensoleillé|
| 4           | :::image type="icon" source="./media/weather-services-concepts/intermittent-clouds.png"::: | Oui |  Non    | Nuages intermittents|
| 5           | :::image type="icon" source="./media/weather-services-concepts/hazy-sunshine.png"::: | Oui |  Non    | Rayonnement diffus |
| 6           | :::image type="icon" source="./media/weather-services-concepts/mostly-cloudy.png"::: | Oui |  Non    | Principalement nuageux|
| 7           | :::image type="icon" source="./media/weather-services-concepts/cloudy-i.png"::: | Oui |  Oui   | Nuageux |
| 8           | :::image type="icon" source="./media/weather-services-concepts/dreary-overcast.png"::: | Oui |  Oui   | Maussade (couvert)|
| 11           | :::image type="icon" source="./media/weather-services-concepts/fog-i.png"::: | Oui |  Oui   | Brouillard|
| 12           | :::image type="icon" source="./media/weather-services-concepts/showers-i.png"::: | Oui |  Oui   | Averses|
| 13           | :::image type="icon" source="./media/weather-services-concepts/mostly-cloudy-showers.png"::: | Oui |  Non    | Principalement nuageux avec des averses|
| 14           | :::image type="icon" source="./media/weather-services-concepts/partly-sunny-showers.png"::: | Oui |  Non    | Partiellement ensoleillé avec des averses|
| 15           | :::image type="icon" source="./media/weather-services-concepts/tstorms-i.png"::: | Oui |  Oui   | Orages|
| 16           | :::image type="icon" source="./media/weather-services-concepts/mostly-cloudy-tstorms.png"::: | Oui |  Non    | Principalement nuageux avec des orages|
| 17           | :::image type="icon" source="./media/weather-services-concepts/partly-sunny-tstorms.png"::: | Oui |  Non    | Partiellement ensoleillé avec des orages|
| 18           | :::image type="icon" source="./media/weather-services-concepts/rain-i.png"::: | Oui |  Oui   | Pluie|
| 19           | :::image type="icon" source="./media/weather-services-concepts/flurries-i.png"::: | Oui |  Oui   | Averses de neige|
| 20           | :::image type="icon" source="./media/weather-services-concepts/mostly-cloudy-flurries.png"::: | Oui |  Non    | Principalement nuageux avec des averses de neige|
| 21           | :::image type="icon" source="./media/weather-services-concepts/partly-sunny-flurries.png"::: | Oui |  Non    | Partiellement ensoleillé avec des averses de neige|
| 22           | :::image type="icon" source="./media/weather-services-concepts/snow-i.png"::: | Oui |  Oui   | Neige|
| 23           | :::image type="icon" source="./media/weather-services-concepts/mostly-cloudy-snow.png"::: | Oui |  Non    | Principalement nuageux avec de la neige|     
| 24           | :::image type="icon" source="./media/weather-services-concepts/ice-i.png"::: | Oui |  Oui   | Verglas |
| 25           | :::image type="icon" source="./media/weather-services-concepts/sleet-i.png"::: | Oui |  Oui   | Giboulée|
| 26           | :::image type="icon" source="./media/weather-services-concepts/freezing-rain.png"::: | Oui |  Oui   | Pluie verglaçante|
| 29           | :::image type="icon" source="./media/weather-services-concepts/rain-snow.png"::: | Oui |  Oui   | Pluie et neige|
| 30           | :::image type="icon" source="./media/weather-services-concepts/hot-i.png"::: | Oui |  Oui   | À chaud|
| 31           | :::image type="icon" source="./media/weather-services-concepts/cold-i.png"::: | Oui |  Oui   | Froid|
| 32           | :::image type="icon" source="./media/weather-services-concepts/windy-i.png"::: | Oui |  Oui   | Venteux|
| 33           | :::image type="icon" source="./media/weather-services-concepts/clear-night.png"::: | Non  |  Oui   | Désactiver|
| 34           | :::image type="icon" source="./media/weather-services-concepts/mostly-clear-night.png"::: | Non  |  Oui   | Ciel plutôt dégagé|
| 35           | :::image type="icon" source="./media/weather-services-concepts/partly-cloudy-night.png"::: | Non  |  Oui   | Partiellement nuageux|
| 36           | :::image type="icon" source="./media/weather-services-concepts/intermittent-clouds-Night.png"::: | Non  |  Oui   | Nuages intermittents|
| 37           | :::image type="icon" source="./media/weather-services-concepts/hazymoon-light.png"::: | Non  |  Oui   | Clair de lune brumeux|
| 38           | :::image type="icon" source="./media/weather-services-concepts/mostly-cloudy-night.png"::: | Non  |  Oui   | Principalement nuageux|
| 39           | :::image type="icon" source="./media/weather-services-concepts/partly-cloudy-showers-night.png"::: | Non  |  Oui   | Partiellement nuageux avec des averses|
| 40           | :::image type="icon" source="./media/weather-services-concepts/mostly-cloudy-showers-night.png"::: | Non  |  Oui   | Principalement nuageux avec des averses|
| 41           | :::image type="icon" source="./media/weather-services-concepts/partly-cloudy-tstorms-night.png"::: | Non  |  Oui   | Partiellement nuageux avec des orages|
| 42           | :::image type="icon" source="./media/weather-services-concepts/mostly-cloudy-tstorms-night.png"::: | Non  |  Oui   | Principalement nuageux avec des orages|
| 43           | :::image type="icon" source="./media/weather-services-concepts/mostly-cloudy-flurries-night.png"::: | Non  |  Oui   | Principalement nuageux avec des averses de neige|
| 44           | :::image type="icon" source="./media/weather-services-concepts/mostly-cloudy-snow.png"::: | Non  |  Oui   | Principalement nuageux avec de la neige|


## <a name="radar-and-satellite-imagery-color-scale"></a>Mise à l’échelle de couleurs des images radar et par satellite

Via l’[API Get Map Tile v2](/rest/api/maps/renderv2/getmaptilepreview), les utilisateurs peuvent demander les images radar et infra-rouges par satellite les plus récentes. Consultez le guide ci-dessous pour interpréter les couleurs utilisées pour les vignettes radar et satellite.

### <a name="radar-images"></a>Images radar

Le tableau ci-dessous fournit des conseils pour interpréter les images radar et créer une légende de carte pour les données de mosaïques radar.

| Code couleur hexadécimal | Échantillon de couleur | Conditions météorologiques |
|----------------|--------------|-------------------|
| #93c701        | ![Couleur pour pluie-légère.](./media/weather-services-concepts/color-93c701.png) | Pluie-Faible |
| #ffd701        | ![Couleur pour pluie-modérée.](./media/weather-services-concepts/color-ffd701.png) | Pluie-Modérée |
| #f05514        | ![Couleur pour pluie-forte.](./media/weather-services-concepts/color-f05514.png) | Pluie-Intense |
| #dc250e        | ![Couleur pour pluie-intense.](./media/weather-services-concepts/color-dc250e.png) | Pluie-Torrentielle |
| #9ec8f2        | ![Couleur pour neige-légère.](./media/weather-services-concepts/color-9ec8f2.png) | Neige-Faible |
| #2a8fdb        | ![Couleur pour neige-modérée.](./media/weather-services-concepts/color-2a8fdb.png) | Neige-Modérée |
| #144bed        | ![Couleur pour neige-forte.](./media/weather-services-concepts/color-144bed.png) | Neige-Intense |
| #020096        | ![Couleur pour neige-intense.](./media/weather-services-concepts/color-020096.png) | Neige-Extrême |
| #e6a5c8        | ![Couleur pour glace-légère.](./media/weather-services-concepts/color-e6a5c8.png) | Verglas-Faible |
| #d24fa0        | ![Couleur pour glace-modérée.](./media/weather-services-concepts/color-d24fa0.png) | Verglas-Modéré |
| #b71691        | ![Couleur pour glace-forte.](./media/weather-services-concepts/color-b71691.png) | Verglas-Intense |
| #7a1570        | ![Couleur pour glace-intense.](./media/weather-services-concepts/color-7a1570.png) | Verglas-Extrême |
| #c196e6        | ![Couleur pour mélange-léger.](./media/weather-services-concepts/color-c196e6.png) | Conditions mixtes-Légères |
| #ae6ee6        | ![Couleur pour mélange-modéré.](./media/weather-services-concepts/color-ae6ee6.png) | Conditions mixtes-Modérées |
| #8a32d7        | ![Couleur pour mélange-fort.](./media/weather-services-concepts/color-8a32d7.png) | Conditions mixtes-Intenses |
| #6500ba        | ![Couleur pour mélange-intense.](./media/weather-services-concepts/color-6500ba.png) | Conditions mixtes-Extrêmes |

La palette de couleurs détaillée pour les mosaïques radar avec des codes de couleur hex et des valeurs dBZ est indiquée ci-dessous. dBZ représente l’intensité de précipitation dans le radar météo. 

| **PLUIE**             | **VERGLAS**              | **NEIGE**              | **CONDITIONS MIXTES**             |
|----------------------|----------------------|-----------------------|-----------------------|
| **dBZ** **(color)**  | **dBZ** **(color)**  | **dBZ** **(color)**   | **dBZ** **(color)**   |
| 1,25       (#93C701) | 1,25       (#E6A5C8) | 1,25       (#9EC8F2)  | 1,25        (#C196E6) |
| 2,5        (#92C201) | 2,5        (#E6A2C6) | 2,5        (#98C5F0)  | 2,5         (#BF92E6) |
| 3,75       (#92BE01) | 3,75       (#E69FC5) | 3,75       (#93C3EF)  | 3,75        (#BD8EE6) |
| 5          (#92BA02) | 5          (#E69DC4) | 5          (#8DC1EE)  | 5           (#BB8BE6) |
| 6,25       (#92B502) | 6,25       (#E69AC2) | 6,25       (#88BFEC)  | 6,25        (#BA87E6) |
| 6,75       (#92B403) | 7,5       (#E697C1)  | 7,5        (#82BDEB)   | 7,5         (#B883E6) |
| 8          (#80AD02) | 8,75       (#E695C0) | 8,75       (#7DBAEA)  | 8,75        (#B680E6) |
| 9,25       (#6FA602) | 10         (#E692BE) | 10         (#77B8E8)  | 10          (#B47CE6) |
| 10,5       (#5EA002) | 11,25      (#E68FBD) | 11,25      (#72B6E7)  | 11,25       (#B378E6) |
| 11,75      (#4D9902) | 12,5       (#E68DBC) | 12,5       (#6CB4E6)  | 12,5        (#B175E6) |
| 12,25      (#479702) | 13,75      (#E68ABA) | 13,75      (#67B2E5)  | 13,75       (#AF71E6) |
| 13,5       (#3D9202) | 15         (#E687B9) | 15         (#61AEE4)  | 15          (#AE6EE6) |
| 14,75      (#338D02) | 16,25      (#E685B8) | 16,25      (#5BABE3)  | 16,25       (#AB6AE4) |
| 16         (#298802)  | 17,5       (#E682B6) | 17,5       (#56A8E2)  | 17,5        (#A967E3) |
| 17,25      (#1F8302) | 18,75      (#E67FB5) | 18,75      (#50A5E1)  | 18,75       (#A764E2) |
| 17,75      (#1B8103) | 20         (#E67DB4) | 20         (#4BA2E0)  | 20          (#A560E1) |
| 19         (#187102) | 21,25      (#E275B0) | 21,25       (#459EDF) | 21,25       (#A35DE0) |
| 20,25      (#166102) | 22,5       (#DF6DAD) | 22,5       (#409BDE)  | 22,5        (#A15ADF) |
| 20,75      (#165B02) | 23,75      (#DC66AA) | 23,75      (#3A98DD)  | 23,75       (#9F56DE) |
| 22         (#135001) | 25         (#D85EA6) | 25         (#3595DC)  | 25          (#9D53DD) |
| 23,25      (#114501) | 26,25      (#D556A3) | 26,25      (#2F92DB)  | 26,25       (#9B50DC) |
| 24,5       (#0F3A01) | 27,5       (#D24FA0) | 27,5       (#2A8FDB)  | 27,5        (#9648DA) |
| 25,75      (#124C01) | 28,75      (#CE479E) | 28,75      (#2581DE)  | 28,75       (#9241D9) |
| 27         (#114401) | 30         (#CB409C) | 30         (#2173E2)  | 30          (#8E39D8) |
| 28,25      (#0F3D01) | 31,25      (#C7399A) | 31,25      (#1C66E5)  | 31,25       (#8A32D7) |
| 28,75      (#0F3A01) | 32,5       (#C43298) | 32,5       (#1858E9)  | 32,5        (#862ED2) |
| 30         (#375401) | 33,75      (#C12B96) | 33,75      (#144BED)  | 33,75       (#832BCE) |
| 31,25      (#5F6E01) | 35         (#BD2494) | 35         (#1348EA)  | 35          (#7F28C9) |
| 32,5       (#878801) | 36,25      (#BA1D92) | 36,25      (#1246E7)  | 36,25       (#7C25C5) |
| 33,75      (#AFA201) | 37,5       (#B71691) | 37,5       (#1144E4)  | 37,5        (#7822C1) |
| 35         (#D7BC01) | 38,75      (#B51690) | 38,75      (#1142E1)  | 38,75       (#751FBC) |
| 36,25      (#FFD701) | 40         (#B3168F) | 40         (#1040DE)  | 40          (#711CB8) |
| 37,5       (#FEB805) | 41,25      (#B1168E) | 41,25      (#0F3EDB)  | 41,25       (#6E19B4) |
| 38,75      (#FCAB06) | 42,5       (#AF168D) | 42,5       (#0F3CD8)  | 42,5        (#6D18B4) |
| 40         (#FA9E07) | 43,75      (#AD168C) | 43,75      (#0E3AD5)  | 43,75       (#6D17B4) |
| 41,25      (#F89209) | 45         (#AB168B) | 45         (#0D38D2)  | 45          (#6D16B4) |
| 42,5       (#F05514) | 46,25      (#A9168A) | 46,25      (#0C36CF)  | 46,25       (#6C15B4) |
| 43,75      (#E74111) | 47,5       (#A81689) | 47,5       (#0C34CC)  | 47,5        (#6C14B5) |
| 45         (#DF2D0F) | 48,75      (#A61688) | 48,75      (#0B32C9)  | 48,75       (#6C13B5) |
| 45,5       (#DC250E) | 50         (#A41687) | 50         (#0A30C6)  | 50          (#6B12B5) |
| 46,75      (#D21C0C) | 51,25      (#A21686) | 51,25      (#0A2EC4)  | 51,25       (#6B11B5) |
| 48         (#C9140A) | 52,5       (#A01685) | 52,5       (#092BC1)  | 52,5        (#6B10B6) |
| 49,25      (#BF0C09) | 53,75      (#9E1684) | 53,75      (#0929BF)  | 53,75       (#6A0FB6) |
| 50         (#BA0808) | 55         (#9C1683) | 55         (#0826BC)  | 55          (#6A0EB6) |
| 56,25      (#6f031b) | 56,25      (#9B1682) | 56,25      (#0824BA)  | 56,25       (#6A0DB6) |
| 57,5       (#9f0143) | 57,5       (#981580) | 57,5       (#0721B7)  | 57,5        (#690CB6) |
| 58,75      (#c10060) | 58,75      (#96157F) | 58,75      (#071FB5)  | 58,75       (#690CB7) |
| 60         (#e70086) | 60         (#94157E) | 60         (#071DB3)  | 60          (#690BB7) |
| 61,25      (#e205a0) | 61,25      (#92157D) | 61,25      (#061AB0)  | 61,25       (#680AB7) |
| 62,5       (#cc09ac) | 62,5       (#90157C) | 62,5       (#0618AE)  | 62,5        (#6809B7) |
| 63,75      (#b50eb7) | 63,75      (#8D157A) | 63,75      (#0515AB)  | 63,75       (#6808B8) |
| 65         (#9315c8) | 65         (#8B1579) | 65         (#0513A9)  | 65          (#6707B8) |
| 66,25      (#8f21cc) | 66,25      (#891578) | 66,25      (#0410A6)  | 66,25       (#6706B8) |
| 67,5       (#983acb) | 67,5       (#871577) | 67,5       (#040EA4)  | 67,5        (#6705B8) |
| 68,75      (#9d49cb) | 68,75      (#851576) | 68,75      (#040CA2)  | 68,75       (#6604B8) |
| 70         (#a661ca) | 70         (#821574) | 70         (#03099F)  | 70          (#6603B9) |
| 71,25      (#ad72c9) | 71,25      (#801573) | 71,25      (#03079D)  | 71,25       (#6602B9) |
| 72,5       (#b78bc6) | 72,5       (#7E1572) | 72,5       (#02049A)  | 72,5        (#6501B9) |
| 73,75      (#bf9bc4) | 73,75      (#7C1571) | 73,75      (#020298)  | 73,75       (#6500B9) |
| 75         (#c9b5c2) | 75         (#7A1570) | 75         (#020096)  | 75          (#6500BA) |



### <a name="satellite-images"></a>Images satellites

Le tableau ci-dessous fournit des conseils pour interpréter les images satellites infrarouges qui affichent les nuages en fonction de leur température et indique comment créer une légende de carte pour ces mosaïques. 

| Code couleur hexadécimal | Échantillon de couleur | Température du nuage |
|----------------|--------------|-------------------|
| #b5b5b5        | ![Vignette de couleur pour #b5b5b5.](./media/weather-services-concepts/color-b5b5b5.png) | Température-Valeur basse | 
| #d24fa0        | ![Vignette de couleur pour #d24fa0.](./media/weather-services-concepts/color-d24fa0.png) |  |
| #8a32d7        | ![Vignette de couleur pour #8a32d7.](./media/weather-services-concepts/color-8a32d7.png) |  |
| #144bed        | ![Vignette de couleur pour #144bed.](./media/weather-services-concepts/color-144bed.png) |  |
| #479702        | ![Vignette de couleur pour #479702.](./media/weather-services-concepts/color-479702.png) |  |
| #72b403        | ![Vignette de couleur pour #72b403.](./media/weather-services-concepts/color-72b403.png) |  |
| #93c701        | ![Vignette de couleur pour #93c701.](./media/weather-services-concepts/color-93c701.png) |  |
| #ffd701        | ![Vignette de couleur pour #ffd701.](./media/weather-services-concepts/color-ffd701.png) |  |
| #f05514        | ![Vignette de couleur pour #f05514.](./media/weather-services-concepts/color-f05514.png) |  |
| #dc250e        | ![Vignette de couleur pour #dc250e.](./media/weather-services-concepts/color-dc250e.png) |  |
| #ba0808        | ![Vignette de couleur pour #ba0808.](./media/weather-services-concepts/color-ba0808.png) |  |
| #1f1f1f        | ![Vignette de couleur pour #1f1f1f.](./media/weather-services-concepts/color-1f1f1f.png) | Température-Valeur élevée |


La palette de couleurs détaillée pour les mosaïques de satellite infrarouges est illustrée ci-dessous.

|**Temp (K)**|**Code couleur hexadécimal**|
|--------|--------------|
|198     |#fe050505     |
|198,43  |#fe120505     |
|198,87  |#fc1f0505     |
|199,3   |#fc2c0606     |
|199,74  |#fa390606     |
|200,17  |#fa460606     |
|200,61  |#f8530606     |
|201,04  |#f8600707     |
|201,48  |#f66c0707     |
|201,91  |#f6790707     |
|202,35  |#f4860707     |
|202,78  |#f4930707     |
|203,22  |#f2a00808     |
|203,65  |#f2ad0808     |
|204,09  |#f0ba0808     |
|204,52  |#f0bd0a09     |
|204,96  |#eec00d09     |
|205,39  |#eec30f0a     |
|205,83  |#ecc5120a     |
|206,26  |#ecc8140b     |
|206,7   |#eacb170b     |
|207,13  |#eace190c     |
|207,57  |#e8d11b0c     |
|208     |#e8d41e0d     |
|208,43  |#e6d6200d     |
|208,87  |#e6d9230e     |
|209,3   |#e4dc250e     |
|209,74  |#e4df2c0f     |
|210,17  |#e2e23310     |
|210,61  |#e2e53a11     |
|211,04  |#e0e73a11     |
|211,48  |#e0ea4712     |
|211,91  |#deed4e13     |
|212,35  |#def05514     |
|212,78  |#dcf15e13     |
|213,22  |#dcf26811     |
|213,65  |#daf37110     |
|214,09  |#daf47a0f     |
|214,52  |#d8f5830d     |
|214,96  |#d8f68d0c     |
|215,39  |#d6f8960b     |
|215,83  |#d6f99f09     |
|216,26  |#d4faa908     |
|216,7   |#d4fbb206     |
|217,13  |#d2fcbb05     |
|217,57  |#d2fdc404     |
|218     |#d0fece02     |
|218,43  |#d0ffd701     |
|218,87  |#cef8d601     |
|219,3   |#cef2d501     |
|219,74  |#ccebd401     |
|220,17  |#cce4d301     |
|220,61  |#caddd201     |
|221,04  |#cad7d101     |
|221,48  |#c8d0d001     |
|221,91  |#c8c9cf01     |
|222,35  |#c6c2ce01     |
|222,78  |#c6bccd01     |
|223,22  |#c4b5cc01     |
|223,65  |#c4aecb01     |
|224,09  |#c2a7ca01     |
|224,52  |#c2a1c901     |
|224,96  |#c09ac801     |
|225,39  |#c093c701     |
|225,83  |#be90c501     |
|226,26  |#be8ec401     |
|226,7   |#bc8bc202     |
|227,13  |#bc88c102     |
|227,57  |#ba85bf02     |
|228     |#ba83be02     |
|228,43  |#b880bc02     |
|228,87  |#b87dba02     |
|229,3   |#b678b703     |
|229,74  |#b675b603     |
|230,17  |#b472b403     |
|230,61  |#b46eb203     |
|231,04  |#b26bb203     |
|231,48  |#b267ad03     |
|231,97  |#b064aa03     |
|232,35  |#b060a803     |
|232,78  |#ae5da603     |
|233,22  |#ae59a302     |
|233,65  |#ac55a102     |
|234,09  |#ac529e02     |
|234,52  |#aa4e9c02     |
|234,96  |#aa4b9902     |
|235,39  |#a8479702     |
|235,83  |#a845940b     |
|236,26  |#a6439115     |
|236,7   |#a6418e1e     |
|237,13  |#a43f8b28     |
|237,57  |#a43d8831     |
|238     |#a23b853a     |
|238,43  |#a2398244     |
|238,87  |#a0377f4d     |
|239,3   |#a0357c57     |
|239,74  |#9e337960     |
|240,17  |#9e317669     |
|240,61  |#9c2f7373     |
|241,04  |#9c2c6f7c     |
|241,48  |#9a2a6c86     |
|241,91  |#9a28698f     |
|242,35  |#98266698     |
|242,78  |#982463a2     |
|243,22  |#962260ab     |
|243,65  |#96205db5     |
|244,09  |#941e5abe     |
|244,52  |#941c57c7     |
|244,85  |#921a54d1     |
|245,39  |#921851da     |
|245,83  |#90164ee4     |
|246,26  |#90144bed     |
|246,7   |#8e2148eb     |
|247,13  |#8e2e45e8     |
|247,57  |#8c3b43e6     |
|248     |#8c4840e3     |
|248,43  |#8a563de1     |
|248,87  |#8a633ade     |
|249,3   |#887038dc     |
|249,74  |#887d35d9     |
|250,17  |#868a32d7     |
|250,61  |#869034d2     |
|251,04  |#849637ce     |
|251,48  |#849c39c9     |
|251,91  |#82a23cc5     |
|252,35  |#82a83ec0     |
|252,78  |#80ae41bc     |
|253,22  |#80b443b7     |
|253,65  |#7eba45b2     |
|253,09  |#7ec048ae     |
|254,52  |#7cc64aa9     |
|254,96  |#7ccc4da5     |
|255,39  |#7ad24fa0     |
|255,83  |#7ad85fac     |
|256,26  |#78dd6eb8     |
|256,7   |#78e37ec4     |
|257,13  |#76e98ed0     |
|257,57  |#76ee9ddb     |
|258     |#74f4ade7     |
|258,43  |#74f9bcf3     |
|258,87  |#72ffccff     |
|259,3   |#71ffffff     |
|259,74  |#71fcfcfc     |
|260,17  |#6ff6f6f6     |
|260,61  |#6ff6f6f6     |
|261,04  |#6df3f3f3     |
|261,48  |#6df3f3f3     |
|261,91  |#6bededed     |
|262,35  |#6bededed     |
|262,78  |#69e7e7e7     |
|263,22  |#69e7e7e7     |
|263,65  |#67e1e1e1     |
|264,09  |#67e1e1e1     |
|264,52  |#65dedede     |
|264,96  |#65dedede     |
|265,39  |#63d8d8d8     |
|265,83  |#63d8d8d8     |
|265,84  |#61d1d1d1     |
|266,26  |#61d1d1d1     |
|267,13  |#5fcecece     |
|267,57  |#5fcecece     |
|268     |#5dc8c8c8     |
|268,43  |#5dc8c8c8     |
|268,87  |#5bc2c2c2     |
|269,3   |#5bc2c2c2     |
|269,74  |#59bcbcbc     |
|270,17  |#59bcbcbc     |
|270,61  |#57b9b9b9     |
|271,04  |#57b9b9b9     |
|271,48  |#55b3b3b3     |
|271,91  |#55b3b3b3     |
|272,35  |#53adadad     |
|272,78  |#53adadad     |
|273,22  |#51aaaaaa     |
|273,65  |#51aaaaaa     |
|274,09  |#4fa4a4a4     |
|274,52  |#4fa4a4a4     |
|274,96  |#4d9e9e9e     |
|275,39  |#4d9e9e9e     |
|275,83  |#4b989898     |
|276,26  |#4b989898     |
|276,7   |#49959595     |
|277,13  |#49959595     |
|277,57  |#478f8f8f     |
|278     |#478f8f8f     |
|278,43  |#45898989     |
|278,87  |#45898989     |
|279,2   |#43868686     |
|279,74  |#43868686     |
|280,17  |#417f7f7f     |
|280,61  |#417f7f7f     |
|281,04  |#3f797979     |
|281,48  |#3f797979     |
|281,91  |#3d737373     |
|282,35  |#3d737373     |
|282,78  |#3b707070     |
|283,22  |#3b707070     |
|283,65  |#396a6a6a     |
|284,09  |#396a6a6a     |
|284,52  |#37646464     |
|284,96  |#37646464     |
|285,39  |#35616161     |
|285,83  |#35616161     |
|286,26  |#335b5b5b     |
|286,7   |#335b5b5b     |
|287,13  |#31555555     |
|287,57  |#31555555     |
|288     |#2f4f4f4f     |
|288,43  |#2f4f4f4f     |
|288,87  |#2d4c4c4c     |
|289,3   |#2d4c4c4c     |
|289,74  |#2b464646     |
|290,17  |#2b464646     |
|290,61  |#29404040     |
|291,04  |#29404040     |
|291,48  |#273d3d3d     |
|291,91  |#273d3d3d     |
|292,35  |#25373737     |
|292,78  |#25373737     |
|293,22  |#23313131     |
|293,65  |#23313131     |
|294,09  |#212a2a2a     |
|294,52  |#212a2a2a     |
|294,96  |#1f272727     |
|295,39  |#1f272727     |
|295,83  |#1d212121     |
|296,26  |#1d212121     |
|296,7   |#1b1b1b1b     |
|297,13  |#1b1b1b1b     |
|297,57  |#19181818     |
|298     |#19181818     |
|298,43  |#17121212     |
|298,87  |#17121212     |
|299,3   |#150c0c0c     |
|299,74  |#150c0c0c     |
|300,17  |#13060606     |
|300,61  |#13060606     |
|301,04  |#11000000     |
|301,48  |#11000000     |
|301,91  |#0f797979     |
|302,35  |#0f797979     |
|302,78  |#0d737373     |
|303,22  |#0d737373     |
|303,65  |#0b6d6d6d     |
|304,09  |#0b6d6d6d     |
|304,52  |#09676767     |
|304,92  |#09676767     |
|305,39  |#07616161     |
|305,83  |#07616161     |
|306,26  |#055b5b5b     |
|306,7   |#055b5b5b     |
|307,13  |#02555555     |
|307,57  |#02555555     |
|308     |#00525252     |
|308     |#00525252     |

## <a name="index-ids-and-index-groups-ids"></a>ID d’index et ID de groupes d’index

L’[API Obtenir des index quotidiens](/rest/api/maps/weather) permet aux utilisateurs de limiter les résultats renvoyés à des types ou groupes d’index spécifiques.

Vous trouverez ci-dessous une table des ID d’index disponibles, leurs noms et un lien vers leurs ensembles de plages. Le tableau ci-dessous répertorie les différents groupes d’index.

  Nom de l'index |  id  | Plage de valeurs
  -------------------------- |---|-----
  Douleur arthritique             |21 | [Bénéfique-À risque extrême](#beneficial-at-extreme-risk)
  Asthme                     |23|  [Bénéfique-À risque extrême](#beneficial-at-extreme-risk)
  Plage et piscine               |10| [Médiocre-Excellent 1](#poor-excellent-1)
  Cyclisme                  |4| [Médiocre-Excellent 1](#poor-excellent-1)
  Rhume                |25|  [Bénéfique-À risque extrême](#beneficial-at-extreme-risk)
  Compostage                 |38| [Médiocre-Excellent 1](#poor-excellent-1)
  Construction               |14| [Médiocre-Excellent 1](#poor-excellent-1)
  COPD                       |44|  [Bénéfique-À risque extrême](#beneficial-at-extreme-risk)
  Promenade avec chien        |43| [Médiocre-Excellent 1](#poor-excellent-1)
  Conduite                    |40|  [Médiocre-Excellent 2](#poor-excellent-2)
  Poussière et phanères              |18| [Faible-Extrême 1](#low-extreme-1)
  Préparation du terrain            |32| [Médiocre-Excellent 1](#poor-excellent-1)
  Pêche                    |13| [Médiocre-Excellent 1](#poor-excellent-1)
  Retards des vols              |-3|  [Très peu probable- Très probable 2](#very-unlikely-very-likely-2)
  Grippe                        |26|  [Bénéfique-À risque extrême](#beneficial-at-extreme-risk)
  Index des voyages en avion        |31| [Excellent-Médiocre](#excellent-poor)
  Économie de carburant               |37| [Médiocre-Excellent 1](#poor-excellent-1)
  Météo pour le golf               |5| [Médiocre-Excellent 1](#poor-excellent-1)
  Pousse du gazon              |33| [Médiocre-Excellent 1](#poor-excellent-1)
  Frisottis                 |42| [Peu probable-Urgence](#unlikely-emergency)
  Bonne santé cardiaque      |16| [Médiocre-Excellent 1](#poor-excellent-1)
  Randonnée                     |3| [Médiocre-Excellent 1](#poor-excellent-1)
  Efficacité énergétique du domicile     |36| [Médiocre-Excellent 1](#poor-excellent-1)
  Chasse                    | 20| [Médiocre-Excellent 1](#poor-excellent-1)
  Activité intérieure            | -2| [Médiocre-Excellent 1](#poor-excellent-1)
  Jogging                    |2| [Médiocre-Excellent 1](#poor-excellent-1)
  Cerf-volant                |9| [Médiocre-Excellent 1](#poor-excellent-1)
  Tonte du gazon                |28| [Médiocre-Excellent 1](#poor-excellent-1)
  Migraine          |27|  [Bénéfique-À risque extrême](#beneficial-at-extreme-risk)
  Bus scolaire du matin         |35| [Médiocre-Excellent 1](#poor-excellent-1)
  Moustiques          |17|[Faible-Extrême 1](#low-extreme-1)
  Activité extérieure           |29| [Médiocre-Excellent 1](#poor-excellent-1)
  Barbecue en plein air           |24| [Médiocre-Excellent 1](#poor-excellent-1)
  Concert en plein air            |8| [Médiocre-Excellent 1](#poor-excellent-1)
  En cours d’exécution                    |1|  [Médiocre-Excellent 1](#poor-excellent-1)
  Tennis                     |6| [Médiocre-Excellent 1](#poor-excellent-1)
  Soif                     |41| [Faible-Extrême 2](#low-extreme-2)
  Voile                    |11| [Médiocre-Excellent 1](#poor-excellent-1)
  Achats                   |39| [Médiocre-Excellent 1](#poor-excellent-1)
  Sinusite             |30|  [Bénéfique-À risque extrême](#beneficial-at-extreme-risk)
  Skateboard              | 7| [Médiocre-Excellent 1](#poor-excellent-1)
  Météo des pistes de ski                | 15| [Médiocre-Excellent 1](#poor-excellent-1)
  Jours de neige                  | 19| [Très peu probable - Très probable](#very-unlikely-very-likely)
  Humidité du sol              | 34| [Médiocre-Excellent 1](#poor-excellent-1)
  Observation des étoiles                 | 12| [Médiocre-Excellent 1](#poor-excellent-1)

Voici la liste des groupes d’index disponibles (indexGroupId) :

  id   | Nom du groupe | Index dans ce groupe |
  -------- | ------------------|------
  1       |Tous | Tous
  2       |Douleurs et maux | Douleur arthritique (21) </br> Migraine (27) </br> Sinusite (30)
  3       |Respiration | Asthme (23) </br> Rhume (25) </br> Prévision de grippe (26)
  4       |Jardinage | Préparation du terrain (32) </br> Tonte du gazon (28) </br> Humidité du sol (34)</br>
  5       |Environnement | Compostage (38) </br> Efficacité énergétique du domicile (36) </br> Économie de carburant (37)
  6       |Plein air | Barbecue en plein air (24) </br> Moustiques (17)
  7       |Plage et activités nautiques | Plage et piscine (10) </br> Pêche (13) </br> Voile (11)
  8       |Sports | Pêche (13) </br> Chasse (20) </br> Activité extérieure (29)</br>
  9       |Agriculture |  Préparation du terrain (32) </br>  Humidité du sol (34)
  10      |Santé | Douleur arthritique (21) </br> Asthme (23) </br> Rhume (25) </br> Poussière et phanères (18) </br> Grippe (26) </br> Bonne santé cardiaque (16) </br> Migraine (27)
  11      |Plein air | Barbecue en plein air (24) </br> Plage et piscine (10) </br> Cyclisme (4) </br> Concert en plein air (8) </br>  Préparation du terrain (32) </br> Pêche (13) </br> Météo pour le golf (5) </br> Randonnée (3) </br> Chasse (20) </br> Jogging (2) </br> Cerf-volant (9) </br> Moustiques (17)</br> Tonte du gazon (28) </br> Activité extérieure (29) </br> Course à pied (1) </br> Voile (11) </br> Skateboard (7) </br> Météo des pistes de ski (15) </br>  Humidité du sol (34)</br> Observation des étoiles (12) </br> Tennis (6)
  12      |Sports | Cyclisme (4) </br> Météo pour le golf (5) </br> Randonnée (3) </br>  Jogging (2) </br> Course à pied (1) </br> Skateboard (7) </br> Météo des pistes de ski (15) </br>Tennis (6)
  13      |Accueil | Efficacité énergétique du domicile (36) </br> Économie de carburant (37) </br> Activité intérieure (-2)

## <a name="daily-index-range-sets"></a>Ensembles de plages d’index quotidiens

L’[API Obtenir des index quotidiens](/rest/api/maps/weather) renvoie la valeur de plage et son nom de catégorie associé pour chaque ID d’index. Les ensembles de plages ne sont pas identiques pour tous les index. Les tableaux ci-dessous montrent les différents ensembles de plages utilisés par les index pris en charge et répertoriés dans [ID d’index et de groupes d’index](#index-ids-and-index-groups-ids). Pour déterminer les index qui utilisent les ensembles de plages, consultez la section [ID d’index et groupes d’index](#index-ids-and-index-groups-ids) de ce document.

### <a name="poor-excellent-1"></a>Médiocre-Excellent 1

  | Nom de catégorie | Plage de début | Plage de fin |
  ----------------|--------------|------------
  Médiocre              |  0 |                2.99
  Moyen              |  3  |               4.99
  Bien              |  5  |              6,99
  Très bien         |  7  |               8.99
  Excellent         |  9  |               10

### <a name="poor-excellent-2"></a>Médiocre-Excellent 2

 | Nom de catégorie | Plage de début | Plage de fin |
  ----------------|--------------|------------
  Médiocre           |0              |  3
  Moyen           |3.01           |  6
  Bien           |6.01           |  7,5
  Très bien      |7.51           |  8.99
  Excellent      |9              |  10

### <a name="excellent-poor"></a>Excellent-Médiocre

 | Nom de catégorie | Plage de début | Plage de fin |
  ----------------|--------------|------------
  Excellent      |     0,00        |    1.00
  Très bien        |   1.01          |  3.00
  Bien             |   3.01          |  5,00
  Moyen             |   5,01          |  7,00
  Médiocre             |   7.01          |  10.00

### <a name="low-extreme-1"></a>Faible-Extrême 1

   | Nom de catégorie | Plage de début | Plage de fin |
  ----------------|--------------|------------
  Faible                |  0         |        1.99
  Modéré           |  2         |        3.99
  Élevé               |  4         |        5,99
  Très élevée          |  6         |        7.99
  Extreme            |  8         |        10

### <a name="low-extreme-2"></a>Faible-Extrême 2

   | Nom de catégorie | Plage de début | Plage de fin |
  ----------------|--------------|------------
  Faible                |  0            |      2.99
  Modéré           |  3            |      4.99
  Élevé               |  5            |      6,99
  Très élevée          |  7            |      8.99
  Extreme            |  9            |      10

### <a name="very-unlikely-very-likely"></a>Très peu probable- Très probable 2

 | Nom de catégorie | Plage de début | Plage de fin |
  ----------------|--------------|------------
  Très peu probable      | 0     |           1.99
  Peu probable           | 2     |           3.99
  Possible           | 4     |           5,99
  Probable             | 6     |           7.99
  Très probablement        | 8     |           10

### <a name="very-unlikely-very-likely-2"></a>Très peu probable- Très probable 2

 | Nom de catégorie | Plage de début | Plage de fin |
  ----------------|--------------|------------
  Très peu probable      |  0,00     |         1.00
  Peu probable           |  1.01     |         3.00
  Possible           |  3.01     |         5,00
  Probable             |  5,01     |         7,00
  Très probablement        |  7.01     |         10.00

### <a name="unlikely-emergency"></a>Peu probable-Urgence

| Nom de catégorie | Plage de début | Plage de fin |
  ----------------|--------------|------------
  Peu probable         |  0     |          2.99
  Espion            |  3     |          4.99
  Conseil         |  5     |          6,99
  Avertissement          |  7     |          8.99
  Urgence        |  9     |          10

### <a name="beneficial-at-extreme-risk"></a>Bénéfique-À risque extrême

| Nom de catégorie | Plage de début | Plage de fin |
  ----------------|--------------|------------
  Bénéfique        |    0        |        1.99
  Neutre           |    2        |        3.99
  À risque           |    4        |        5,99
  À risque élevé      |    6        |        7.99
  À risque extrême   |    8        |        10