---
title: Afficher des informations à partir d’une coordonnée sur une carte | Microsoft Azure Maps
description: Découvrez comment afficher des informations sur une adresse sur la carte quand un utilisateur sélectionne une coordonnée.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-js
ms.openlocfilehash: 31e4004379340912051204786da592fe33a5bd63
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92890749"
---
# <a name="get-information-from-a-coordinate"></a>Obtenir des informations à partir d’une coordonnée

Cet article explique comment effectuer une recherche d’adresses inversée qui affiche l’adresse d’un emplacement d’une fenêtre contextuelle sur lequel l’utilisateur a cliqué.

Deux méthodes permettent d’effectuer une recherche d’adresses inversée. La première consiste à interroger [l’API Azure Maps Reverse Address Search](/rest/api/maps/search/getsearchaddressreverse) via un module de service. La seconde consiste à utiliser l’[API Fetch](https://fetch.spec.whatwg.org/) pour demander à l’[API Azure Maps Reverse Address Search](/rest/api/maps/search/getsearchaddressreverse) de rechercher une adresse. Les deux méthodes sont examinées ci-dessous.

## <a name="make-a-reverse-search-request-via-service-module"></a>Demande de recherche inversée via un module de service

<iframe height='500' scrolling='no' title='Obtenir des informations à partir d’une coordonnée (module de Service)' src='//codepen.io/azuremaps/embed/ejEYMZ/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Voir la page <a href='https://codepen.io/azuremaps/pen/ejEYMZ/'>Obtenir des informations à partir d’une coordonnée</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

Dans le code ci-dessus, le premier bloc construit un objet de carte et définit le mécanisme d’authentification pour utiliser le jeton d’accès. Vous pouvez consulter la section [Créer une carte](./map-create.md) pour obtenir des instructions.

Le deuxième bloc de code crée un `TokenCredential` pour authentifier les requêtes HTTP auprès d’Azure Maps avec le jeton d’accès. Il transfère ensuite le `TokenCredential` à `atlas.service.MapsURL.newPipeline()` et crée une instance [Pipeline](/javascript/api/azure-maps-rest/atlas.service.pipeline). `searchURL` représente une URL vers les opérations de [recherche](/rest/api/maps/search) d’Azure Maps.

Le troisième bloc de code change le style du curseur de la souris en pointeur et crée un objet [fenêtre contextuelle](/javascript/api/azure-maps-control/atlas.popup#open). Pour plus d’instructions, consultez la section relative à l’[ajout d’une fenêtre contextuelle sur la carte](./map-add-popup.md).

Le quatrième bloc de code ajoute un [détecteur d’événements](/javascript/api/azure-maps-control/atlas.map#events) clic de souris. Lorsqu'il est déclenché, il crée une requête de recherche avec les coordonnées du point activé. Il utilise ensuite la méthode [getSearchAddressReverse](/javascript/api/azure-maps-rest/atlas.service.searchurl#searchaddressreverse-aborter--geojson-position--searchaddressreverseoptions-) pour interroger l’[API Get Search Address Reverse](/rest/api/maps/search/getsearchaddressreverse) et obtenir l’adresse des coordonnées. Une collection d’entités GeoJSON est alors extraite de la réponse à l’aide de la méthode `geojson.getFeatures()`.

Le cinquième bloc de code définit le contenu de la fenêtre contextuelle HTML pour afficher l’adresse de réponse correspondant à la position des coordonnées sur lesquelles vous avez cliqué.

Les modifications du curseur, de l’objet popup et de l’événement de clic sont toutes créées dans le [détecteur d’événements de chargement](/javascript/api/azure-maps-control/atlas.map#events) de la carte. Cette structure de code garantit le chargement complet de la carte avant de récupérer les informations de coordonnées.

## <a name="make-a-reverse-search-request-via-fetch-api"></a>Requête de recherche inversée via l'API de récupération

Cliquez sur la carte pour effectuer une requête de géocode inversé pour cette position à l'aide de récupérer (fetch).

<iframe height='500' scrolling='no' title='Obtenir des informations à partir d’une coordonnée' src='//codepen.io/azuremaps/embed/ddXzoB/?height=516&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez la page <a href='https://codepen.io/azuremaps/pen/ddXzoB/'>Get information from a coordinate</a> (Obtenir des informations à partir d’une coordonnée) d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

Dans le code ci-dessus, le premier bloc de code construit un objet de carte et définit le mécanisme d’authentification pour utiliser le jeton d’accès. Vous pouvez consulter la section [Créer une carte](./map-create.md) pour obtenir des instructions.

Le deuxième bloc de code modifie en pointeur le style du curseur de la souris. Il instancie un objet de [fenêtre contextuelle](/javascript/api/azure-maps-control/atlas.popup#open). Pour plus d’instructions, consultez la section relative à l’[ajout d’une fenêtre contextuelle sur la carte](./map-add-popup.md).

Le troisième bloc de code ajoute un détecteur d’événements pour les clics de souris. À la suite d’un clic de souris, il utilise l’[API Fetch](https://fetch.spec.whatwg.org/) pour interroger l’[API Azure Maps Reverse Address Search](/rest/api/maps/search/getsearchaddressreverse) et obtenir l’adresse des coordonnées activées. Pour une réponse correcte, il collecte l’adresse de l’emplacement sur lequel l’utilisateur a cliqué. Il définit le contenu et la position de la fenêtre contextuelle à l’aide de la fonction [setOptions](/javascript/api/azure-maps-control/atlas.popup#setoptions-popupoptions-) de la classe de fenêtre contextuelle.

Les modifications du curseur, de l’objet popup et de l’événement de clic sont toutes créées dans le [détecteur d’événements de chargement](/javascript/api/azure-maps-control/atlas.map#events) de la carte. Cette structure de code garantit le chargement complet de la carte avant de récupérer les informations de coordonnées.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Meilleures pratiques pour l'utilisation du service de recherche](how-to-use-best-practices-for-search.md)

En savoir plus sur les classes et les méthodes utilisées dans cet article :

> [!div class="nextstepaction"]
> [Map](/javascript/api/azure-maps-control/atlas.map)

> [!div class="nextstepaction"]
> [Popup](/javascript/api/azure-maps-control/atlas.popup)

Pour voir des exemples de codes complets, consultez les articles suivants :

> [!div class="nextstepaction"]
> [Afficher des directions de A à B](./map-route.md)

> [!div class="nextstepaction"]
> [Afficher le trafic](./map-show-traffic.md)