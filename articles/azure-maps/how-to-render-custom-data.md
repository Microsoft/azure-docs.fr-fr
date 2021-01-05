---
title: Afficher des données personnalisées sur une carte raster | Microsoft Azure Maps
description: Découvrez comment ajouter des clics-infos, des étiquettes et des formes géométriques à une carte raster. Pour ce faire, découvrez comment utiliser le service d’images statiques dans Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: mvc
ms.openlocfilehash: 5c70835c11bafb3fd06645ba51099b33d1eb6149
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96906078"
---
# <a name="render-custom-data-on-a-raster-map"></a>Afficher des données personnalisées sur une carte raster

Cet article explique comment utiliser le [service d’image statique](/rest/api/maps/render/getmapimage), avec la fonctionnalité de composition d’image, afin de permettre les superpositions sur une carte raster. La composition d’image inclut la possibilité d’obtenir une vignette raster dotée de données supplémentaires, telles que des repères personnalisés, des étiquettes et des superpositions géométriques.

Pour afficher une superposition géométrique, des étiquettes et des repères personnalisés, vous pouvez utiliser l’application Postman. Vous pouvez utiliser les [API du service de données](/rest/api/maps/data) d’Azure Maps pour stocker et afficher les superpositions.

> [!Tip]
> Il est souvent plus économique d’utiliser le kit SDK d’Azure Maps Web pour afficher une carte simple sur une page Web que d’utiliser le service d’images statiques. Le kit SDK Web utilise des vignettes de carte et, sauf si l’utilisateur effectue un panoramique et un zoom sur la carte, il ne génère souvent qu’une fraction de transaction par charge de carte. Notez que le kit SDK Azure Maps Web dispose d’options pour désactiver le panoramique et le zoom. En outre, le kit de développement logiciel (SDK) web Azure Maps offre beaucoup plus d’options de visualisation de données qu’un service web de cartes statiques.  

## <a name="prerequisites"></a>Prérequis

### <a name="create-an-azure-maps-account"></a>Créer un compte Azure Maps

Pour accomplir les procédures de cet article, vous devez d’abord créer un compte Azure Maps et obtenir votre clé de compte Maps. Suivez les instructions mentionnées dans [Créer un compte](quick-demo-map-app.md#create-an-azure-maps-account) pour créer un abonnement de compte Azure Maps, puis effectuez les étapes indiquées dans [Obtenir la clé primaire](quick-demo-map-app.md#get-the-primary-key-for-your-account) afin d’obtenir la clé primaire de votre compte. Pour plus d’informations sur l’authentification dans Azure Maps, voir [Gérer l’authentification dans Azure Maps](./how-to-manage-authentication.md).


## <a name="render-pushpins-with-labels-and-a-custom-image"></a>Afficher des repères avec des étiquettes et une image personnalisée

> [!Note]
> La réalisation de cette procédure nécessite un compte Azure Maps dans le niveau tarifaire S0 ou S1.

Le compte Azure Maps au niveau S0 prend en charge une seule instance du paramètre `pins`. Cela vous permet d’afficher jusqu’à cinq repères, spécifiés dans la requête URL, avec une image personnalisée.

Pour afficher les repères avec les étiquettes et une image personnalisée, effectuez ces étapes :

1. Créez une collection dans laquelle stocker les demandes. Dans l’application Postman, sélectionnez **New** (Nouveau). Dans la fenêtre **Create New** (Créer nouveau), sélectionnez **Collection**. Nommez la collection puis sélectionnez le bouton **Create** (Créer). 

2. Pour créer la demande, sélectionnez **New** à nouveau. Dans la fenêtre **Create New** (Créer nouveau), sélectionnez **Request** (Demande). Entrez un **Request name** (Nom de demande) pour les clics-infos. Sélectionnez la collection que vous avez créée à l’étape précédente comme emplacement dans lequel enregistrer la demande. Sélectionnez ensuite **Enregistrer**.
    
    ![Créer une demande dans Postman](./media/how-to-render-custom-data/postman-new.png)

3. Sélectionnez la méthode HTTP GET sous l’onglet Builder (Générateur), puis entrez l’URL suivante pour créer une requête GET.

    ```HTTP
    https://atlas.microsoft.com/map/static/png?subscription-key={subscription-key}&api-version=1.0&layer=basic&style=main&zoom=12&center=-73.98,%2040.77&pins=custom%7Cla15+50%7Cls12%7Clc003b61%7C%7C%27CentralPark%27-73.9657974+40.781971%7C%7Chttps%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2FAzureMapsCodeSamples%2Fmaster%2FAzureMapsCodeSamples%2FCommon%2Fimages%2Ficons%2Fylw-pushpin.png
    ```
    Voici l’image obtenue :

    ![Un repère personnalisé avec une étiquette](./media/how-to-render-custom-data/render-pins.png)


## <a name="get-data-from-azure-maps-data-storage"></a>Obtenir des données à partir du stockage de données Azure Maps

> [!Note]
> La réalisation de cette procédure nécessite un compte Azure Maps dans le niveau tarifaire S1.

Vous pouvez aussi obtenir les informations sur le chemin et l’emplacement du repère à l’aide de l’[API de chargement des données](/rest/api/maps/data/uploadpreview). Suivez les étapes ci-dessous pour charger les données du chemin et des repères.

1. Dans l’application Postman, ouvrez un nouvel onglet dans la collection que vous avez créée dans la section précédente. Sélectionnez la méthode HTTP POST sous l’onglet Builder (Générateur), puis entrez l’URL suivante pour envoyer une requête POST :

    ```HTTP
    https://atlas.microsoft.com/mapData/upload?subscription-key={subscription-key}&api-version=1.0&dataFormat=geojson
    ```

2. Sur l’onglet **Params**, entrez les paires clé/valeur suivantes, utilisées pour l’URL de la requête POST. Remplacez la valeur `subscription-key` par votre clé d’abonnement Azure Maps.
    
    ![Paramètres clé/valeur dans Postman](./media/how-to-render-custom-data/postman-key-vals.png)

3. Sur l’onglet **Body** (Corps), sélectionnez le format d’entrée Raw (Brut), puis choisissez JSON comme format d’entrée dans la liste déroulante. Fournissez le code JSON en tant que données à charger :
    
    ```JSON
    {
      "type": "FeatureCollection",
      "features": [
        {
          "type": "Feature",
          "properties": {},
          "geometry": {
            "type": "Polygon",
            "coordinates": [
              [
                [
                  -73.98235,
                  40.76799
                ],
                [
                  -73.95785,
                  40.80044
                ],
                [
                  -73.94928,
                  40.7968
                ],
                [
                  -73.97317,
                  40.76437
                ],
                [
                  -73.98235,
                  40.76799
                ]
              ]
            ]
          }
        },
        {
          "type": "Feature",
          "properties": {},
          "geometry": {
            "type": "LineString",
            "coordinates": [
              [
                -73.97624731063843,
                40.76560773817073
              ],
              [
                -73.97914409637451,
                40.766826609362575
              ],
              [
                -73.98513078689575,
                40.7585866048861
              ]
            ]
          }
        }
      ]
    }
    ```

4. Sélectionnez **Send** (Envoyer), puis examinez l’en-tête de la réponse. Quand une requête réussit, l’en-tête d’emplacement contient l’URI d’état, qui sert à la vérification de l’état actuel de la requête de chargement. L’URI d’état se présente comme suit.  

   ```HTTP
   https://atlas.microsoft.com/mapData/{uploadStatusId}/status?api-version=1.0
   ```

5. Copiez votre URI d’état et ajoutez-lui un paramètre de clé d’abonnement. La valeur de ce paramètre doit correspondre à la clé d’abonnement de votre compte Azure Maps. Utilisez la clé d’abonnement de compte que vous avez utilisée pour charger les données. L’URI d’état doit se présenter comme suit :

   ```HTTP
   https://atlas.microsoft.com/mapData/{uploadStatusId}/status?api-version=1.0&subscription-key={Subscription-key}
   ```

6. Pour obtenir l’udId, ouvrez un nouvel onglet dans l’application Postman. Sélectionnez la méthode HTTP GET sous l’onglet Builder (Générateur). Effectuez une requête GET sur l’URI d’état. Si le chargement de données réussit, vous recevez un udId dans le corps de la réponse. Copiez l’udId.

   ```JSON
   {
      "udid" : "{udId}"
   }
   ```

7. Utilisez la valeur `udId` reçue de l’API de chargement des données pour afficher les fonctions sur la carte. Pour ce faire, ouvrez un nouvel onglet dans la collection que vous avez créée dans la section précédente. Sélectionnez la méthode HTTP GET sous l’onglet Builder (Générateur), remplacez {subscription-key} et {udId} par vos valeurs, puis entrez cette URL pour effectuer une requête GET :

    ```HTTP
    https://atlas.microsoft.com/map/static/png?subscription-key={subscription-key}&api-version=1.0&layer=basic&style=main&zoom=12&center=-73.96682739257812%2C40.78119135317995&pins=default|la-35+50|ls12|lc003C62|co9B2F15||'Times Square'-73.98516297340393 40.758781646381024|'Central Park'-73.96682739257812 40.78119135317995&path=lc0000FF|fc0000FF|lw3|la0.80|fa0.30||udid-{udId}
    ```

    Voici l’image de la réponse :

    ![Obtenir des données à partir du stockage de données Azure Maps](./media/how-to-render-custom-data/uploaded-path.png)

## <a name="render-a-polygon-with-color-and-opacity"></a>Afficher un polygone avec une couleur et une opacité

> [!Note]
> La réalisation de cette procédure nécessite un compte Azure Maps dans le niveau tarifaire S1.


Vous pouvez modifier l’apparence d’un polygone en utilisant des modificateurs de style avec le [paramètre path](/rest/api/maps/render/getmapimage#uri-parameters).

1. Dans l’application Postman, ouvrez un nouvel onglet dans la collection que vous avez créée précédemment. Sélectionnez la méthode HTTP GET sous l’onglet Builder (Générateur), puis entrez l’URL suivante pour configurer une requête GET afin d’afficher un polygone de couleur opaque :
    
    ```HTTP
    https://atlas.microsoft.com/map/static/png?api-version=1.0&style=main&layer=basic&sku=S1&zoom=14&height=500&Width=500&center=-74.040701, 40.698666&path=lc0000FF|fc0000FF|lw3|la0.80|fa0.50||-74.03995513916016 40.70090237454063|-74.04082417488098 40.70028420372218|-74.04113531112671 40.70049568385827|-74.04298067092896 40.69899904076542|-74.04271245002747 40.69879568992435|-74.04367804527283 40.6980961582905|-74.04364585876465 40.698055487620714|-74.04368877410889 40.698022951066996|-74.04168248176573 40.696444909137|-74.03901100158691 40.69837271818651|-74.03824925422668 40.69837271818651|-74.03809905052185 40.69903971085914|-74.03771281242369 40.699340668780984|-74.03940796852112 40.70058515602143|-74.03948307037354 40.70052821920425|-74.03995513916016 40.70090237454063
    &subscription-key={subscription-key}
    ```

    Voici l’image de la réponse :

    ![Afficher un polygone opaque](./media/how-to-render-custom-data/opaque-polygon.png)


## <a name="render-a-circle-and-pushpins-with-custom-labels"></a>Afficher un cercle et des repères avec des étiquettes personnalisées

> [!Note]
> La réalisation de cette procédure nécessite un compte Azure Maps dans le niveau tarifaire S1.


Vous pouvez modifier l’apparence des repères en ajoutant des modificateurs de style. Par exemple, pour augmenter ou diminuer la taille des clics-infos et de leurs étiquettes, utilisez le modificateur de « style de l’échelle » `sc`. Ce modificateur accepte une valeur supérieure à zéro. La valeur 1 représente l’échelle standard. Les valeurs supérieures à 1 agrandissent les repères tandis que les valeurs qui lui sont inférieures les diminuent. Pour plus d’informations sur les modificateurs de style, consultez [Paramètres du chemin du service d’image statique](/rest/api/maps/render/getmapimage#uri-parameters).


Suivez ces étapes pour afficher un cercle et des repères avec des étiquettes personnalisées :

1. Dans l’application Postman, ouvrez un nouvel onglet dans la collection que vous avez créée précédemment. Sélectionnez la méthode HTTP GET sous l’onglet Builder (Générateur), puis entrez cette URL pour envoyer une requête GET :

    ```HTTP
    https://atlas.microsoft.com/map/static/png?api-version=1.0&style=main&layer=basic&zoom=14&height=700&Width=700&center=-122.13230609893799,47.64599069048016&path=lcFF0000|lw2|la0.60|ra1000||-122.13230609893799 47.64599069048016&pins=default|la15+50|al0.66|lc003C62|co002D62||'Microsoft Corporate Headquarters'-122.14131832122801  47.64690503939462|'Microsoft Visitor Center'-122.136828 47.642224|'Microsoft Conference Center'-122.12552547454833 47.642940335653996|'Microsoft The Commons'-122.13687658309935  47.64452336193245&subscription-key={subscription-key}
    ```

    Voici l’image de la réponse :

    ![Afficher un cercle avec des repères personnalisés](./media/how-to-render-custom-data/circle-custom-pins.png)

2. Pour modifier la couleur des clics-infos à partir de la dernière étape, changez le modificateur de style « co ». Examinez `pins=default|la15+50|al0.66|lc003C62|co002D62|`, la couleur actuelle est spécifiée comme étant #002D62 dans CSS. Supposons que vous souhaitez la remplacer par #41d42a. Écrivez la nouvelle valeur de la couleur après le spécificateur « co », comme suit : `pins=default|la15+50|al0.66|lc003C62|co41D42A|`. Créez une nouvelle requête GET :

    ```HTTP
    https://atlas.microsoft.com/map/static/png?api-version=1.0&style=main&layer=basic&zoom=14&height=700&Width=700&center=-122.13230609893799,47.64599069048016&path=lcFF0000|lw2|la0.60|ra1000||-122.13230609893799 47.64599069048016&pins=default|la15+50|al0.66|lc003C62|co41D42A||'Microsoft Corporate Headquarters'-122.14131832122801  47.64690503939462|'Microsoft Visitor Center'-122.136828 47.642224|'Microsoft Conference Center'-122.12552547454833 47.642940335653996|'Microsoft The Commons'-122.13687658309935  47.64452336193245&subscription-key={subscription-key}
    ```

    Voici l’image de réponse après avoir modifié les couleurs des repères :

    ![Afficher un cercle avec des clics-infos mis à jour](./media/how-to-render-custom-data/circle-updated-pins.png)

De même, vous pouvez modifier, ajouter et supprimer d’autres modificateurs de style.

## <a name="next-steps"></a>Étapes suivantes


* Explorez la documentation de l’[API Azure Maps Get Map Image](/rest/api/maps/render/getmapimage).
* Pour plus d’informations sur le service Data Azure Maps (préversion), consultez la [documentation du service](/rest/api/maps/data).