---
title: Afficher les résultats de la recherche sur une carte | Microsoft Azure Maps
description: Dans cet article, vous apprendrez à effectuer une demande de recherche à l’aide du Kit de développement logiciel (SDK) web Microsoft Azure Maps et à afficher les résultats sur la carte.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen
ms.openlocfilehash: 647a8fc25f27ef7f441ed7459ecd543d4f35581e
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92895271"
---
# <a name="show-search-results-on-the-map"></a>Afficher les résultats de recherche sur la carte

Cet article explique comment rechercher des lieux d’intérêt et afficher les résultats de la recherche sur la carte.

Vous pouvez rechercher un lieu d’intérêt de deux façons. La première consiste à utiliser un module de service pour effectuer une demande de recherche. La seconde consiste à effectuer une requête de recherche à l'[API de recherche approximative Azure Maps](/rest/api/maps/search/getsearchfuzzy) via l'[API de récupération](https://fetch.spec.whatwg.org/). Les deux méthodes sont décrites ci-dessous.

## <a name="make-a-search-request-via-service-module"></a>Demande de recherche via un module de service

<iframe height='500' scrolling='no' title='Afficher les résultats de la recherche sur une carte (module de service)' src='//codepen.io/azuremaps/embed/zLdYEB/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez <a href='https://codepen.io/azuremaps/pen/zLdYEB/'>Show search results on a map (Service Module)</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

Dans le code ci-dessus, le premier bloc construit un objet de carte et définit le mécanisme d’authentification pour utiliser le jeton d’accès. Vous pouvez consulter la section [Créer une carte](./map-create.md) pour obtenir des instructions.

Le deuxième bloc de code crée un `TokenCredential` pour authentifier les requêtes HTTP auprès d’Azure Maps avec le jeton d’accès. Il transfère ensuite le `TokenCredential` à `atlas.service.MapsURL.newPipeline()` et crée une instance [Pipeline](/javascript/api/azure-maps-rest/atlas.service.pipeline). `searchURL` représente une URL vers les opérations de [recherche](/rest/api/maps/search) d’Azure Maps.

Le troisième bloc de code crée un objet source de données à l’aide de la classe [DataSource](/javascript/api/azure-maps-control/atlas.source.datasource) et y ajoute les résultats de la recherche. Une [couche de symboles](/javascript/api/azure-maps-control/atlas.layer.symbollayer) utilise du texte ou des icônes pour afficher les données basées sur le point, qui sont wrappées dans la [source de données](/javascript/api/azure-maps-control/atlas.source.datasource) en tant que symboles sur la carte.  Une couche de symboles est alors créée. La source de données est ajoutée à la couche de symbole, qui est ensuite ajoutée à la carte.

Le quatrième bloc de code utilise la méthode [SearchFuzzy](/javascript/api/azure-maps-rest/atlas.service.models.searchgetsearchfuzzyoptionalparams) dans le [module de service](how-to-use-services-module.md). Cela vous permet d’effectuer une recherche de texte de forme libre via [l’API REST de recherche approximative](/rest/api/maps/search/getsearchfuzzy) pour rechercher des points d’intérêt. Les requêtes GET à l’API Search Fuzzy peuvent gérer toutes les combinaisons d’entrées approximatives. Une collection de fonctionnalités GeoJSON déterminée par la réponse est ensuite extraite à l’aide de la méthode `geojson.getFeatures()` et ajoutée à la source de données. Les données sont alors automatiquement restituées sur la carte par le biais de la couche de symboles.

Le dernier bloc de code ajuste les limites d’appareil photo pour la carte à l’aide de la propriété [setCamera](/javascript/api/azure-maps-control/atlas.map#setcamera-cameraoptions---cameraboundsoptions---animationoptions-) de la carte.

La demande de recherche, la source de données, la couche de symboles et les limites d’appareil photo se trouvent à l’intérieur du [détecteur d’événements](/javascript/api/azure-maps-control/atlas.map#events) de la carte. Nous voulons nous assurer que les résultats s’affichent après le chargement complet de la carte.


## <a name="make-a-search-request-via-fetch-api"></a>Requête de recherche via l'API de récupération

<iframe height='500' scrolling='no' title='Afficher les résultats de la recherche sur une carte' src='//codepen.io/azuremaps/embed/KQbaeM/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez la page <a href='https://codepen.io/azuremaps/pen/KQbaeM/'>Show search results on a map </a> (Afficher les résultats de recherche sur une carte) d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

Dans le code ci-dessus, le premier bloc de code construit un objet de carte. Il définit le mécanisme d’authentification pour utiliser le jeton d’accès. Vous pouvez consulter la section [Créer une carte](./map-create.md) pour obtenir des instructions.

Le deuxième bloc de code crée une URL pour effectuer une requête de recherche. Il crée également deux tableaux pour stocker les limites et les repères des résultats de la recherche.

Le troisième bloc de code utilise l’[API Fetch](https://fetch.spec.whatwg.org/). L’[API Fetch](https://fetch.spec.whatwg.org/) est utilisée pour demander à [l’API Fuzzy Search d’Azure Maps](/rest/api/maps/search/getsearchfuzzy) de rechercher les points d’intérêt. L’API de recherche floue peut gérer n’importe quelle combinaison d’entrées approximatives. Elle gère et analyse ensuite la réponse de recherche et ajoute les repères qui en résultent au tableau searchPins.

Le quatrième bloc de code crée un objet source de données à l’aide de la classe [DataSource](/javascript/api/azure-maps-control/atlas.source.datasource). Dans le code, nous ajoutons des résultats de recherche à l’objet source. Une [couche de symboles](/javascript/api/azure-maps-control/atlas.layer.symbollayer) utilise du texte ou des icônes pour afficher les données basées sur le point, qui sont wrappées dans la [source de données](/javascript/api/azure-maps-control/atlas.source.datasource) en tant que symboles sur la carte. Une couche de symboles est alors créée. La source de données est ajoutée à la couche de symbole, qui est ensuite ajoutée à la carte.

Le dernier bloc de code crée un objet [BoundingBox](/javascript/api/azure-maps-control/atlas.data.boundingbox). Il utilise le tableau de résultats, puis ajuste les limites d’appareil photo pour la carte à l’aide de l’option [setCamera](/javascript/api/azure-maps-control/atlas.map#setcamera-cameraoptions---cameraboundsoptions---animationoptions-) de la carte. Il affiche ensuite les épingles qui en résultent.

La demande de recherche, la source de données, la couche de symboles et les limites d’appareil photo sont définies au sein du [détecteur d’événements](/javascript/api/azure-maps-control/atlas.map#events) de la carte pour garantir que les résultats sont affichés après le chargement complet de la carte.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Meilleures pratiques pour l'utilisation du service de recherche](how-to-use-best-practices-for-search.md)

En savoir plus sur **Recherche approximative** :

> [!div class="nextstepaction"]
> [API de recherche approximative Azure Maps](/rest/api/maps/search/getsearchfuzzy)

En savoir plus sur les classes et les méthodes utilisées dans cet article :

> [!div class="nextstepaction"]
> [Map](/javascript/api/azure-maps-control/atlas.map)

Pour voir des exemples de codes complets, consultez les articles suivants :

> [!div class="nextstepaction"]
> [Obtenir des informations à partir d’une coordonnée](map-get-information-from-coordinate.md)
<!-- Comment added to suppress false positive warning -->
> [!div class="nextstepaction"]
> [Afficher des directions de A à B](map-route.md)