---
title: Demander des données d’élévation à l’aide du service Elevation d’Azure Maps (préversion)
description: Découvrez comment demander des données d’élévation à l’aide du service Elevation d’Azure Maps (préversion).
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc
ms.openlocfilehash: 76232a917e8856a06645fabc0ab4716195c5c0e1
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97094197"
---
# <a name="request-elevation-data-using-the-azure-maps-elevation-service-preview"></a>Demander des données d’élévation à l’aide du service Elevation d’Azure Maps (préversion)

> [!IMPORTANT]
> Le service Elevation d’Azure Maps est actuellement en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Le [service Elevation](https://docs.microsoft.com/rest/api/maps/elevation) d’Azure Maps fournit des API permettant d’interroger les données d’élévation où qu’elles se trouvent sur la planète. Vous pouvez demander des données d’élévation échantillonnées présentes sur des tracés, dans un cadre englobant défini ou à des coordonnées précises. En outre, vous pouvez utiliser l’[API Render V2 - Get Map Tile](https://docs.microsoft.com/rest/api/maps/renderv2) pour récupérer les données d’élévation au format vignette. Les vignettes sont fournies au format raster GeoTIFF. Cet article explique comment utiliser le service Elevation d’Azure Maps et l’API Get Map Tile pour demander des données d’élévation. Les données d’élévation peuvent être demandées au format GeoJSON ou GeoTiff.

## <a name="prerequisites"></a>Prérequis

1. [Créer un compte Azure Maps avec le niveau tarifaire S1](quick-demo-map-app.md#create-an-azure-maps-account)
2. [Obtenir une clé d’abonnement principale](quick-demo-map-app.md#get-the-primary-key-for-your-account), également appelée clé primaire ou clé d’abonnement.

Pour plus d’informations sur l’authentification dans Azure Maps, consultez [Gérer l’authentification dans Azure Maps](how-to-manage-authentication.md).

Cet article utilise l’application [Postman](https://www.postman.com/), mais vous pouvez choisir un autre environnement de développement d’API.

## <a name="request-elevation-data-in-raster-tiled-format"></a>Demander des données d’élévation au format de vignette raster

Pour demander les données d’élévation au format de vignette raster, utilisez l’[API Render V2 - Get Map Tile](https://docs.microsoft.com/rest/api/maps/renderv2). Si la vignette est disponible, l’API la retournera au format GeoTIFF. Dans le cas contraire, l’API retourne 0. Toutes les vignettes DEM raster utilisent le mode Terre géoïde (niveau de la mer). Dans cet exemple, nous allons demander des données d’élévation pour l’Everest.

>[!TIP]
>Pour récupérer une vignette sur la carte du monde, vous devez trouver la vignette avec le niveau de zoom approprié. Notez également que WorldDEM couvre l’intégralité de la surface terrestre, mais pas les océans.  Pour plus d’informations, consultez [Niveaux de zoom et grille mosaïque](zoom-levels-and-tile-grid.md).

1. Ouvrez l’application Postman. En haut de l'application Postman, sélectionnez **New** (Nouveau). Dans la fenêtre **Create New** (Créer nouveau), sélectionnez **Collection**.  Nommez la collection puis sélectionnez le bouton **Create** (Créer).

2. Pour créer la demande, sélectionnez **New** à nouveau. Dans la fenêtre **Create New** (Créer nouveau), sélectionnez **Request** (Demande). Entrez un **Request name** (Nom de demande) pour la demande. Sélectionnez la collection que vous avez créée à l’étape précédente, puis sélectionnez **Enregistrer**.

3. Sélectionnez la méthode HTTP **GET** sous l’onglet Générateur, puis entrez l’URL suivante pour demander une vignette raster. Pour cette requête et d’autres requêtes mentionnées dans cet article, remplacez `{Azure-Maps-Primary-Subscription-key}` par votre clé d’abonnement principale.

    ```http
    https://atlas.microsoft.com/map/tile?subscription-key={Azure-Maps-Primary-Subscription-key}&api-version=2.0&tilesetId=microsoft.dem&zoom=13&x=6074&y=3432
    ```

4. Cliquez sur le bouton **Envoyer**. Vous devez recevoir la vignette raster contenant les données d’élévation au format GeoTIFF. Chaque pixel des données brutes de vignette raster est de type `float`. La valeur de chaque pixel représente la hauteur d’élévation en mètres.

## <a name="request-elevation-data-in-geojson-format"></a>Demander des données d’élévation au format GeoJSON

Utilisez les API du service Elevation (préversion) pour demander des données d’élévation au format GeoJSON. Cette section présente chacune de ces trois API :

* [Get Data for Points](/rest/api/maps/elevation/getdataforpoints)
* [Post Data for Points](/rest/api/maps/elevation/postdataforpoints)
* [Get Data for Polyline](https://docs.microsoft.com/rest/api/maps/elevation/getdataforpolyline)
* [Post Data for Polyline](https://docs.microsoft.com/rest/api/maps/elevation/postdataforpolyline)
* [Get Data for Bounding Box](https://docs.microsoft.com/rest/api/maps/elevation/getdataforboundingbox)

>[!IMPORTANT]
> Si aucune donnée ne peut être retournée, toutes les API retourneront `0`.

### <a name="request-elevation-data-for-points"></a>Demander des données d’élévation pour des points

Dans cet exemple, nous allons utiliser l’[API Get Data for Points](/rest/api/maps/elevation/getdataforpoints) afin de demander des données d’élévation au niveau de l’Everest et du Chamlang. Ensuite, nous allons utiliser l’[API Post Data for Points](/rest/api/maps/elevation/postdataforpoints) pour demander des données d’élévation à l’aide des deux mêmes points. Les latitudes et les longitudes de l’URL sont censées être exprimées en degrés décimaux WGS84 (système géodésique mondial).

 >[!IMPORTANT]
 >Étant donné que le nombre de caractères d’une URL est limité à 2 048, il n’est pas possible de passer plus de 100 coordonnées sous la forme d’une chaîne délimitée par un pipeline dans une requête GET d’URL. Si vous prévoyez de passer plus de 100 coordonnées sous la forme d’une chaîne délimitée par un pipeline, utilisez POST Data For Points.

1. Pour créer la demande, sélectionnez **New** à nouveau. Dans la fenêtre **Create New** (Créer nouveau), sélectionnez **Request** (Demande). Entrez un **Request name** (Nom de demande) pour la demande. Sélectionnez la collection que vous avez créée à l’étape précédente, puis sélectionnez **Enregistrer**.

2. Sélectionnez la méthode HTTP **GET** sous l’onglet Générateur, puis entrez l’URL suivante. Pour cette requête et d’autres requêtes mentionnées dans cet article, remplacez `{Azure-Maps-Primary-Subscription-key}` par votre clé d’abonnement principale.

    ```http
    https://atlas.microsoft.com/elevation/point/json?subscription-key={Azure-Maps-Primary-Subscription-key}&api-version=1.0&points=-73.998672,40.714728|150.644,-34.397
    ```

3. Cliquez sur le bouton **Envoyer**.  Vous recevrez la réponse JSON suivante :

    ```json
    {
    "data": [
        {
            "coordinate": {
                "latitude": 40.714728,
                "longitude": -73.998672
            },
            "elevationInMeter": 12.142355447638208
        },
        {
            "coordinate": {
                "latitude": -34.397,
                "longitude": 150.644
            },
            "elevationInMeter": 384.47041445517846
        }
    ]
    }
    ```

4. Ensuite, nous allons appeler l’[API Post Data for Points](/rest/api/maps/elevation/postdataforpoints) afin d’obtenir des données d’élévation pour les deux mêmes points. Sélectionnez la méthode HTTP **POST** sous l’onglet du générateur, puis entrez l’URL suivante. Pour cette requête et d’autres requêtes mentionnées dans cet article, remplacez `{Azure-Maps-Primary-Subscription-key}` par votre clé d’abonnement principale.

    ```http
    https://atlas.microsoft.com/elevation/point/json?subscription-key={Azure-Maps-Primary-Subscription-key}&api-version=1.0
    ```

5. Dans les **En-têtes** de la requête **POST**, définissez `Content-Type` sur `application/json`. Dans le **Corps**, fournissez les points de coordonnées ci-dessous. Quand vous avez terminé, cliquez sur **Envoyer**.

     ```json
    [
        {
            "lon": -73.998672,
            "lat": 40.714728
        },
        {
            "lon": 150.644,
            "lat": -34.397
        }
    ]
    ```

### <a name="request-elevation-data-samples-along-a-polyline"></a>Demander des échantillons de données d’élévation situées le long d’une polyligne

Dans cet exemple, nous allons utiliser l’API [Get Data for Polyline](https://docs.microsoft.com/rest/api/maps/elevation/getdataforpolyline) pour demander cinq échantillons uniformément espacés de données d’élévation situées le long d’une droite, entre les coordonnées de l’Everest et du Chamlang. Les deux coordonnées doivent être définies au format Long/Lat. Si vous ne spécifiez pas de valeur pour le paramètre `samples`, le nombre d’échantillons est défini par défaut sur 10. Le nombre maximal d’échantillons est de 2 000.

Ensuite, nous allons utiliser Get Data for Polyline pour demander trois échantillons uniformément espacés de données d’élévation situées le long d’un tracé. Nous allons définir le lieu précis des exemples en passant trois paires de coordonnées Long/Lat.

Enfin, nous allons utiliser l’[API Post Data For Polyline](https://docs.microsoft.com/rest/api/maps/elevation/postdataforpolyline) pour demander des données d’élévation au niveau des trois mêmes échantillons uniformément espacés.

Les latitudes et les longitudes de l’URL sont censées être exprimées en degrés décimaux WGS84 (système géodésique mondial).

 >[!IMPORTANT]
 >Étant donné que le nombre de caractères d’une URL est limité à 2 048, il n’est pas possible de passer plus de 100 coordonnées sous la forme d’une chaîne délimitée par un pipeline dans une requête GET d’URL. Si vous prévoyez de passer plus de 100 coordonnées sous la forme d’une chaîne délimitée par un pipeline, utilisez POST Data For Points.

1. Sélectionnez **Nouveau**. Dans la fenêtre **Create New** (Créer nouveau), sélectionnez **Request** (Demande). Entrez un **Nom de demande**, puis sélectionnez une collection. Cliquez sur **Enregistrer**.

2. Sélectionnez la méthode HTTP **GET** sous l’onglet Générateur, puis entrez l’URL suivante. Pour cette requête et d’autres requêtes mentionnées dans cet article, remplacez `{Azure-Maps-Primary-Subscription-key}` par votre clé d’abonnement principale.

   ```http
    https://atlas.microsoft.com/elevation/line/json?api-version=1.0&subscription-key={Azure-Maps-Primary-Subscription-key}&lines=-73.998672,40.714728|150.644,-34.397&samples=5
    ```

3. Cliquez sur le bouton **Envoyer**.  Vous recevrez la réponse JSON suivante :

    ```JSON
    {
        "data": [
            {
                "coordinate": {
                    "latitude": 40.714728,
                    "longitude": -73.998672
                },
                "elevationInMeter": 12.14236
            },
            {
                "coordinate": {
                    "latitude": 21.936796000000001,
                    "longitude": -17.838003999999998
                },
                "elevationInMeter": 0.0
            },
            {
                "coordinate": {
                    "latitude": 3.1588640000000012,
                    "longitude": 38.322664000000003
                },
                "elevationInMeter": 598.66943
            },
            {
                "coordinate": {
                    "latitude": -15.619067999999999,
                    "longitude": 94.483332000000019
                },
                "elevationInMeter": 0.0
            },
            {
                "coordinate": {
                    "latitude": -34.397,
                    "longitude": 150.644
                },
                "elevationInMeter": 384.47041
            }
        ]
    }
    ```

4. À présent, nous allons demander trois échantillons de données d’élévation situées le long d’un tracé, entre les coordonnées de l’Everest, du Chamlang et du Jannu. Dans la section **Params**, copiez le tableau de coordonnées suivant pour la valeur de clé de requête `lines`.

    ```html
        86.9797222, 27.775|86.9252778, 27.9880556 | 88.0444444, 27.6822222
    ```

5. Remplacez la valeur de clé de requête `samples` par la valeur `3`.  L’image ci-dessous montre les nouvelles valeurs.

     :::image type="content" source="./media/how-to-request-elevation-data/get-elevation-samples.png" alt-text="Récupérez trois échantillons de données d’élévation.":::

6. Cliquez sur le bouton bleu **Envoyer**. Vous recevrez la réponse JSON suivante :

    ```json
    {
        "data": [
            {
                "coordinate": {
                    "latitude": 27.775,
                    "longitude": 86.9797222
                },
                "elevationInMeter": 7116.0348851572589
            },
            {
                "coordinate": {
                    "latitude": 27.737403546316028,
                    "longitude": 87.411180791156454
                },
                "elevationInMeter": 1798.6945512521534
            },
            {
                "coordinate": {
                    "latitude": 27.682222199999998,
                    "longitude": 88.0444444
                },
                "elevationInMeter": 7016.9372013588072
            }
        ]
    }
    ```

7. Ensuite, nous allons appeler l’[API Post Data For Polyline](https://docs.microsoft.com/rest/api/maps/elevation/postdataforpolyline) afin d’obtenir des données d’élévation pour les trois mêmes points. Sélectionnez la méthode HTTP **POST** sous l’onglet du générateur, puis entrez l’URL suivante. Pour cette requête et d’autres requêtes mentionnées dans cet article, remplacez `{Azure-Maps-Primary-Subscription-key}` par votre clé d’abonnement principale.

    ```http
    https://atlas.microsoft.com/elevation/line/json?api-version=1.0&subscription-key={Azure-Maps-Primary-Subscription-key}&samples=5
    ```

8. Dans les **En-têtes** de la requête **POST**, définissez `Content-Type` sur `application/json`. Dans le **Corps**, fournissez les points de coordonnées ci-dessous. Quand vous avez terminé, cliquez sur **Envoyer**.

     ```json
    [
        {
            "lon": 86.9797222,
            "lat": 27.775
        },
        {
            "lon": 86.9252778,
            "lat": 27.9880556
        },
        {
            "lon": 88.0444444,
            "lat": 27.6822222
        }
    ]
    ```

### <a name="request-elevation-data-by-bounding-box"></a>Demander des données d’élévation par cadre englobant

À présent, nous allons utiliser l’API [Get Data for Bounding Box](https://docs.microsoft.com/rest/api/maps/elevation/getdataforboundingbox) pour demander des données d’élévation à proximité du Mont Rainier dans l’État de Washington. Les données d’élévation sont retournées à des emplacements uniformément espacés à l’intérieur d’un cadre englobant. Le cadre englobant défini par (2) jeux de coordonnées Lat/Long (latitude sud, longitude ouest | latitude nord, longitude est) est divisé en lignes et en colonnes. Les bords du cadre englobant comptent pour deux (2) lignes et deux (2) colonnes. Les élévations sont retournées pour les vertex de grille créés au niveau des intersections des lignes et des colonnes. Jusqu’à 2 000 élévations peuvent être retournées dans une même requête.

Dans cet exemple, nous allons spécifier rows=3 et columns=6. 18 valeurs d’élévation sont retournées dans la réponse. Dans le diagramme suivant, les valeurs d’élévation sont classées en partant de l’angle sud-ouest, puis de l’ouest vers l’est, et du sud vers le nord.  Les points d’élévation sont numérotés dans l’ordre dans lequel ils sont retournés.

:::image type="content" source="./media/how-to-request-elevation-data/bounding-box.png" border="false" alt-text="Coordonnées du cadre englobant aux angles nord-est et sud-est.":::

1. Sélectionnez **Nouveau**. Dans la fenêtre **Create New** (Créer nouveau), sélectionnez **Request** (Demande). Entrez un **Nom de demande**, puis sélectionnez une collection. Cliquez sur **Enregistrer**.

2. Sélectionnez la méthode HTTP **GET** sous l’onglet Générateur, puis entrez l’URL suivante. Pour cette requête et d’autres requêtes mentionnées dans cet article, remplacez `{Azure-Maps-Primary-Subscription-key}` par votre clé d’abonnement principale.

    ```http
    https://atlas.microsoft.com/elevation/lattice/json?subscription-key={Azure-Maps-Primary-Subscription-key}&api-version=1.0&bounds=-121.66853362143818, 46.84646479863713,-121.65853362143818, 46.85646479863713&rows=2&columns=3
    ```

3. Cliquez sur le bouton bleu **Envoyer**. 18 exemples de données d’élévation, un pour chaque vertex de la grille, sont retournés dans la réponse.

    ```json
    {
    "data": [
        {
            "coordinate": {
                "latitude": 46.846464798637129,
                "longitude": -121.66853362143819
            },
            "elevationInMeter": 2298.6581875651746
        },
        {
            "coordinate": {
                "latitude": 46.846464798637129,
                "longitude": -121.66653362143819
            },
            "elevationInMeter": 2306.3980756609963
        },
        {
            "coordinate": {
                "latitude": 46.846464798637129,
                "longitude": -121.66453362143818
            },
            "elevationInMeter": 2279.3385479564113
        },
        {
            "coordinate": {
                "latitude": 46.846464798637129,
                "longitude": -121.66253362143819
            },
            "elevationInMeter": 2233.1549264690366
        },
        {
            "coordinate": {
                "latitude": 46.846464798637129,
                "longitude": -121.66053362143818
            },
            "elevationInMeter": 2196.4485923541492
        },
        {
            "coordinate": {
                "latitude": 46.846464798637129,
                "longitude": -121.65853362143818
            },
            "elevationInMeter": 2133.1756767157253
        },
        {
            "coordinate": {
                "latitude": 46.849798131970459,
                "longitude": -121.66853362143819
            },
            "elevationInMeter": 2345.3227848228803
        },
        {
            "coordinate": {
                "latitude": 46.849798131970459,
                "longitude": -121.66653362143819
            },
            "elevationInMeter": 2292.2449195443587
        },
        {
            "coordinate": {
                "latitude": 46.849798131970459,
                "longitude": -121.66453362143818
            },
            "elevationInMeter": 2270.5867788258074
        },
        {
            "coordinate": {
                "latitude": 46.849798131970459,
                "longitude": -121.66253362143819
            },
            "elevationInMeter": 2296.8311427390604
        },
        {
            "coordinate": {
                "latitude": 46.849798131970459,
                "longitude": -121.66053362143818
            },
            "elevationInMeter": 2266.0729430891065
        },
        {
            "coordinate": {
                "latitude": 46.849798131970459,
                "longitude": -121.65853362143818
            },
            "elevationInMeter": 2242.216346631234
        },
        {
            "coordinate": {
                "latitude": 46.8531314653038,
                "longitude": -121.66853362143819
            },
            "elevationInMeter": 2378.460838833359
        },
        {
            "coordinate": {
                "latitude": 46.8531314653038,
                "longitude": -121.66653362143819
            },
            "elevationInMeter": 2327.6761137260387
        },
        {
            "coordinate": {
                "latitude": 46.8531314653038,
                "longitude": -121.66453362143818
            },
            "elevationInMeter": 2208.3782743402949
        },
        {
            "coordinate": {
                "latitude": 46.8531314653038,
                "longitude": -121.66253362143819
            },
            "elevationInMeter": 2106.9526472760981
        },
        {
            "coordinate": {
                "latitude": 46.8531314653038,
                "longitude": -121.66053362143818
            },
            "elevationInMeter": 2054.3270174034078
        },
        {
            "coordinate": {
                "latitude": 46.8531314653038,
                "longitude": -121.65853362143818
            },
            "elevationInMeter": 2030.6438331110671
        },
        {
            "coordinate": {
                "latitude": 46.856464798637127,
                "longitude": -121.66853362143819
            },
            "elevationInMeter": 2318.753153399402
        },
        {
            "coordinate": {
                "latitude": 46.856464798637127,
                "longitude": -121.66653362143819
            },
            "elevationInMeter": 2253.88875188271
        },
        {
            "coordinate": {
                "latitude": 46.856464798637127,
                "longitude": -121.66453362143818
            },
            "elevationInMeter": 2136.6145845357587
        },
        {
            "coordinate": {
                "latitude": 46.856464798637127,
                "longitude": -121.66253362143819
            },
            "elevationInMeter": 2073.6734467948486
        },
        {
            "coordinate": {
                "latitude": 46.856464798637127,
                "longitude": -121.66053362143818
            },
            "elevationInMeter": 2042.994055784251
        },
        {
            "coordinate": {
                "latitude": 46.856464798637127,
                "longitude": -121.65853362143818
            },
            "elevationInMeter": 1988.3631481900356
        }
    ]
    }
    ```

## <a name="samples-use-elevation-service-preview-apis-in-azure-maps-control"></a>Exemples : Utiliser les API du service Elevation (préversion) dans Azure Maps Control

### <a name="get-elevation-data-by-coordinate-position"></a>Obtenir les données d’élévation par position de coordonnée

L’exemple de page web suivant vous montre comment utiliser le contrôle de carte pour afficher les données d’élévation à un point de coordonnées précis. Lorsque l’utilisateur fait glisser le marqueur, la carte affiche les données d’élévation dans une fenêtre contextuelle.

<br/>

<iframe height="500" style="width:100%;" scrolling="no" title="Obtenir l’élévation à une position précise" src="https://codepen.io/azuremaps/embed/c840b510e113ba7cb32809591d5f96a2?height=500&theme-id=default&default-tab=js,result&editable=true" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
Consultez <a href='https://codepen.io/azuremaps/pen/c840b510e113ba7cb32809591d5f96a2'>Get elevation at position by Azure Maps</a> (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

### <a name="get-elevation-data-by-bounding-box"></a>Obtenir des données d’élévation par cadre englobant

L’exemple de page web suivant vous montre comment utiliser le contrôle de carte pour afficher les données d’élévation contenues dans un cadre englobant. L’utilisateur définit le cadre englobant en cliquant sur l’icône `square` située dans l’angle supérieur gauche, et en dessinant le carré sur la carte à l’emplacement souhaité. Le contrôle de carte restitue ensuite les données d’élévation en respectant les couleurs spécifiées dans la clé qui se trouve en haut à droite.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Élévations par cadre englobant" src="https://codepen.io/azuremaps/embed/619c888c70089c3350a3e95d499f3e48?height=500&theme-id=default&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
Consultez <a href='https://codepen.io/azuremaps/pen/619c888c70089c3350a3e95d499f3e48'>Elevations by bounding box by Azure Maps</a> (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

### <a name="get-elevation-data-by-polyline-path"></a>Obtenir des données d’élévation par tracé polyligne

L’exemple de page web suivant vous montre comment utiliser le contrôle de carte pour afficher les données d’élévation situées le long d’un tracé. L’utilisateur définit le tracé en cliquant sur l’icône `PolyLine` située dans l’angle supérieur gauche, et en dessinant la polyligne sur la carte. Le contrôle de carte restitue ensuite les données d’élévation en respectant les couleurs spécifiées dans la clé qui se trouve en haut à droite.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Dégradé du tracé d’élévation" src="https://codepen.io/azuremaps/embed/7bee08e5cb13d05cb0a11636b60f14ca?height=500&theme-id=default&default-tab=js,result&editable=true" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
Consultez <a href='https://codepen.io/azuremaps/pen/7bee08e5cb13d05cb0a11636b60f14ca'>Elevation path gradient by Azure Maps</a> (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>


## <a name="next-steps"></a>Étapes suivantes

Pour aller plus loin dans l’exploration des API d’élévation Azure Maps (préversion), consultez :

> [!div class="nextstepaction"]
> [Elevation (préversion) - Get Data For Points](/rest/api/maps/elevation/getdataforpoints)

> [!div class="nextstepaction"]
> [Elevation (préversion) - Get Data for Bounding Box](https://docs.microsoft.com/rest/api/maps/elevation/getdataforboundingbox)

> [!div class="nextstepaction"]
> [Elevation (préversion) - Get Data for Polyline](https://docs.microsoft.com/rest/api/maps/elevation/getdataforpolyline)

> [!div class="nextstepaction"]
> [Render V2 – Get Map Tile](https://docs.microsoft.com/rest/api/maps/renderv2)

Pour obtenir la liste complète des API REST Azure Maps, consultez :

> [!div class="nextstepaction"]
> [API REST Azure Maps](https://docs.microsoft.com/rest/api/maps/)
