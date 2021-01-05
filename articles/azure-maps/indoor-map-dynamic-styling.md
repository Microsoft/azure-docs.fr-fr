---
title: Implémenter des styles dynamiques pour les cartes d’intérieur Azure Maps Creator (préversion)
description: Découvrez comment implémenter des styles dynamiques pour les cartes d’intérieur Creator (préversion).
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 266dc5d62f6224495075546528ad71d806d415ac
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96903443"
---
# <a name="implement-dynamic-styling-for-creator-preview-indoor-maps"></a>Implémenter des styles dynamiques pour les cartes d’intérieur Creator (préversion)

> [!IMPORTANT]
> Les services Azure Maps Creator sont disponibles en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Le [service État de caractéristique](/rest/api/maps/featurestate) du Créateur Azure Maps permet d’appliquer des styles basés sur les propriétés dynamiques des caractéristiques de données cartographiques intérieures.  Par exemple, vous pouvez afficher des salles de réunion d’un bâtiment dans une couleur spécifique reflétant leur état d’occupation. Cet article explique comment afficher de façon dynamique des caractéristiques de carte d’intérieur avec le [service État de caractéristique](/rest/api/maps/featurestate) et le [module web Intérieur](how-to-use-indoor-module.md).

## <a name="prerequisites"></a>Prérequis

1. [Créer un compte Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account)
2. [Obtenir une clé d’abonnement principale](quick-demo-map-app.md#get-the-primary-key-for-your-account), également appelée clé primaire ou clé d’abonnement.
3. [Créer une ressource Creator (préversion)](how-to-manage-creator.md)
4. Téléchargez l’[exemple de package de dessin](https://github.com/Azure-Samples/am-creator-indoor-data-examples).
5. [Créez une carte d’intérieur](tutorial-creator-indoor-maps.md) pour obtenir un ID `tilesetId` et un ID `statesetId`.
6. Générez une application web en suivant les étapes décrites dans [comment utiliser le module Cartes d’intérieur](how-to-use-indoor-module.md).

Ce tutoriel utilise l’application [Postman](https://www.postman.com/), mais vous pouvez choisir un autre environnement de développement d’API.

## <a name="implement-dynamic-styling"></a>Implémenter l’application de style dynamique

Une fois les prérequis réunis, vous devez disposer d’une application web simple configurée avec votre clé d’abonnement et vos ID `tilesetId` et `statesetId`.

### <a name="select-features"></a>Sélectionner les caractéristiques

Pour implémenter l’application de style dynamique, une caractéristique telle qu’une salle de réunion ou de conférence doit être référencée par sa caractéristique `id`. Vous utiliserez la caractéristique `id` pour mettre à jour la propriété dynamique, ou *état*, de cette caractéristique. Pour afficher les fonctionnalités définies dans un jeu de données, vous pouvez utiliser l’une des méthodes suivantes :

* API WFS (service de caractéristique web). Des jeux de données peuvent être interrogés à l’aide de l’API de service de caractéristique web. L’API de service de caractéristique web suit les caractéristiques de l’API Open Geospatial Consortium. L’API de service de caractéristique web est utile pour interroger des caractéristiques à l’intérieur d’un jeu de données. Par exemple, vous pouvez utiliser le service de caractéristique web pour rechercher toutes les salles de réunion de taille moyenne d’un bâtiment et d’un étage donnés.

* Implémentez du code personnalisé qui permet à un utilisateur de sélectionner des caractéristiques sur une carte à l’aide de votre application Web. Dans cet article, nous allons utiliser cette option.  

Le script suivant implémente l’événement de clic de souris. Le code récupère la caractéristique `id` en fonction du point cliqué. Dans votre application, vous pouvez insérer le code sous votre bloc de code du Gestionnaire d’intérieur. Exécutez votre application et vérifiez la console pour obtenir l’`id` de caractéristique du point cliqué.

```javascript
/* Upon a mouse click, log the feature properties to the browser's console. */
map.events.add("click", function(e){

    var features = map.layers.getRenderedShapes(e.position, "indoor");

    var result = features.reduce(function (ids, feature) {
        if (feature.layer.id == "indoor_unit_office") {
            console.log(feature);
        }
    }, []);
});
```

Le tutoriel [Créer une carte d’intérieur](tutorial-creator-indoor-maps.md) a configuré le stateset de caractéristique pour accepter les mises à jour d’état pour `occupancy`.

Dans la section suivante, nous allons définir l’*état* d’occupation du bureau `UNIT26` sur `true`. tandis le bureau `UNIT27` est défini sur `false`.

### <a name="set-occupancy-status"></a>Définir l’état d’occupation

 Nous allons maintenant mettre à jour l’état des deux bureaux, `UNIT26` et `UNIT27` :

1. Dans l’application Postman, sélectionnez **Nouveau**. Dans la fenêtre **Create New** (Créer nouveau), sélectionnez **Request** (Demande). Entrez un **Nom de demande**, puis sélectionnez une collection. Cliquez sur **Enregistrer**.

2. Utilisez l’[API d’états de mise à jour de caractéristique](/rest/api/maps/featurestate/updatestatespreview) pour mettre à jour l’état. Transmettez l’ID de stateset, et `UNIT26` pour l’une des deux unités. Ajoutez votre clé d’abonnement Azure Maps. Voici l’URL d’une requête **POST** pour mettre à jour l’état :

    ```http
    https://atlas.microsoft.com/featureState/state?api-version=1.0&statesetID={statesetId}&featureID=UNIT26&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

3. Dans les **En-têtes** de la requête **POST**, définissez `Content-Type` sur `application/json`. Dans le **CORPS** de la requête **POST**, écrivez le code JSON suivant avec les mises à jour de caractéristique. La mise à jour n’est enregistrée que si l’horodatage de publication est postérieur à l’horodatage utilisé dans les requêtes de mise à jour d’état de caractéristique précédentes pour le même `ID` de caractéristique. Transmettez le `keyName` « occupé » pour mettre à jour sa valeur.

    ```json
    {
        "states": [
            {
                "keyName": "occupied",
                "value": true,
                "eventTimestamp": "2019-11-14T17:10:20"
            }
        ]
    }
    ```

4. Répétez les étapes 2 et 3 en utilisant `UNIT27` avec le code JSON suivant.

    ``` json
    {
        "states": [
            {
                "keyName": "occupied",
                "value": false,
                "eventTimestamp": "2019-11-14T17:10:20"
            }
        ]
    }
    ```

### <a name="visualize-dynamic-styles-on-a-map"></a>Visualiser des styles dynamiques sur une carte

L’application web que vous avez précédemment ouverte dans un navigateur doit maintenant refléter l’état mis à jour des caractéristiques de carte. `UNIT27`(151) doit apparaître en vert et `UNIT26`(157) doit en rouge.

![Salle libre en vert et salle occupée en rouge](./media/indoor-map-dynamic-styling/room-state.png)

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus, consultez :

> [!div class="nextstepaction"]
> [Creator (préversion) pour cartes d’intérieur](creator-indoor-maps.md)

Consultez les références des API mentionnées dans cet article :

> [!div class="nextstepaction"]
> [Chargement de données](creator-indoor-maps.md#upload-a-drawing-package)

> [!div class="nextstepaction"]
> [Conversion de données](creator-indoor-maps.md#convert-a-drawing-package)

> [!div class="nextstepaction"]
> [Dataset](creator-indoor-maps.md#datasets)

> [!div class="nextstepaction"]
> [Tileset](creator-indoor-maps.md#tilesets)

> [!div class="nextstepaction"]
> [Jeu d’état de caractéristique](creator-indoor-maps.md#feature-statesets)

> [!div class="nextstepaction"]
> [Service de caractéristique web](creator-indoor-maps.md#web-feature-service-api)