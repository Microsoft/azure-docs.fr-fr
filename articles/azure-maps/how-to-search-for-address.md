---
title: Rechercher un emplacement à l’aide des services Search d’Azure Maps
description: Apprenez-en plus sur le service Search Azure Maps. Découvrez comment utiliser cet ensemble d’API pour le géocodage, le géocodage inversé, les recherches approximatives et les recherches de rues inverses.
author: anastasia-ms
ms.author: v-stharr
ms.date: 10/05/2020
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 00ddb53276c052d538d658f2c40384e86cf72aee
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92910982"
---
# <a name="search-for-a-location-using-azure-maps-search-services"></a>Rechercher un emplacement à l’aide des services Search d’Azure Maps

Le [service de recherche](/rest/api/maps/search) d’Azure Maps est un ensemble d’API RESTful conçu pour aider les développeurs à rechercher des adresses, des lieux et des listes d’entreprises par nom, catégorie et autre information géographique. En plus de prendre en charge le géocodage traditionnel, les services peuvent aussi géocoder en sens inverse les adresses et intersections en fonction des latitudes et longitudes. Les valeurs de latitude et de longitude renvoyées par la recherche peuvent être utilisées comme paramètres dans d’autres services d’Azure Maps, tels que les services [Itinéraire](/rest/api/maps/route) et [Météo](/rest/api/maps/weather).


Dans cet article, vous allez apprendre à :

* Demander des coordonnées de latitude et de longitude pour une adresse (emplacement de l’adresse géocodée) à l’aide de l’[API de recherche d’adresse]( https://docs.microsoft.com/rest/api/maps/search/getsearchaddress).
* Rechercher une adresse ou un point d’intérêt (POI) à l’aide de l’[API de recherche approximative](/rest/api/maps/search/getsearchfuzzy).
* Créer une [recherche d’adresse inversée](/rest/api/maps/search/getsearchaddressreverse) pour traduire les coordonnées de l’emplacement en adresse postale.
* Traduisez l’emplacement des coordonnées en une intersection compréhensible en utilisant de l’[API Search Address Reverse Cross Street](/rest/api/maps/search/getsearchaddressreversecrossstreet).  Le plus souvent, cela est nécessaire dans les applications de suivi qui reçoivent le flux GPS d’un appareil ou d’une ressource et qui souhaitent connaître l’emplacement auquel correspondent les coordonnées.

## <a name="prerequisites"></a>Prérequis

1. [Créer un compte Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account)
2. [Obtenir une clé d’abonnement principale](quick-demo-map-app.md#get-the-primary-key-for-your-account), également appelée clé primaire ou clé d’abonnement.

Ce tutoriel utilise l’application [Postman](https://www.postman.com/), mais vous pouvez choisir un autre environnement de développement d’API.

## <a name="request-latitude-and-longitude-for-an-address-geocoding"></a>Demander la latitude et la longitude d’une adresse (géocodage)

Dans cet exemple, nous utilisons l’[API Get Search Address](/rest/api/maps/search/getsearchaddress) d’Azure Maps pour convertir une adresse en coordonnées de latitude et de longitude. Ce processus est également appelé *géocodage*. En plus de retourner les coordonnées, la réponse retourne également des propriétés détaillées de l’adresse telles que la rue, le code postal, la commune et les informations relatives au pays ou à la région.

>[!TIP]
>Si vous avez un ensemble d’adresses à géocoder, vous pouvez utiliser l’[API Post Search Address Batch](/rest/api/maps/search/postsearchaddressbatch) pour envoyer un lot de requêtes en un seul appel d’API.

1. Ouvrez l’application Postman. En haut de l'application Postman, sélectionnez **New** (Nouveau). Dans la fenêtre **Create New** (Créer nouveau), sélectionnez **Collection**.  Nommez la collection puis sélectionnez le bouton **Create** (Créer). Vous utiliserez ce regroupement pour le reste des exemples de ce document.

2. Pour créer la demande, sélectionnez **New** à nouveau. Dans la fenêtre **Create New** (Créer nouveau), sélectionnez **Request** (Demande). Entrez un **Request name** (Nom de demande) pour la demande. Sélectionnez la collection que vous avez créée à l’étape précédente, puis sélectionnez **Enregistrer**.

3. Sélectionnez la méthode HTTP **GET** sous l’onglet Générateur, puis entrez l’URL suivante. Dans cette requête, nous recherchons une adresse spécifique : `400 Braod St, Seattle, WA 98109`. Pour cette requête et d’autres requêtes mentionnées dans cet article, remplacez `{Azure-Maps-Primary-Subscription-key}` par votre clé d’abonnement principale.

    ```http
    https://atlas.microsoft.com/search/address/json?&subscription-key={Azure-Maps-Primary-Subscription-key}&api-version=1.0&language=en-US&query=400 Broad St, Seattle, WA 98109
    ```

4. Cliquez sur le bouton bleu **Envoyer**. Le corps de la réponse contient les données d’un emplacement unique.

5. À présent, nous allons rechercher une adresse qui a plusieurs emplacements possibles. Dans la section **Params** , remplacez la clé `query` par `400 Broad, Seattle`. Cliquez sur le bouton bleu **Envoyer**.

    :::image type="content" source="./media/how-to-search-for-address/search-address.png" alt-text="Rechercher une adresse":::

6. Ensuite, essayez de définir la clé `query` sur `400 Broa`.

7. Cliquez sur le bouton **Envoyer**. Vous pouvez maintenant voir que la réponse comprend des réponses réparties dans plusieurs pays. Pour adapter des résultats en fonction de la zone pertinente pour vos utilisateurs, ajoutez toujours un maximum de détails sur l’emplacement dans la requête.

## <a name="using-fuzzy-search-api"></a>Utilisation de l’API Recherche approximative

L’[API de recherche approximative](/rest/api/maps/search/getsearchfuzzy) d’Azure Maps prend en charge les recherches standard à ligne unique et à forme libre. Nous vous recommandons d’utiliser l’API de recherche approximative d’Azure Maps lorsque vous ne connaissez pas votre type d’entrée utilisateur pour une demande de recherche.  L’entrée de requête peut être une adresse complète ou partielle. Il peut également s’agir d’un jeton de point d’intérêt (POI), comme un nom de POI, une catégorie de POI ou un nom de marque. En outre, pour améliorer la pertinence des résultats de votre recherche, les résultats de la requête peuvent être limités par un emplacement et un rayon de coordonnées ou par la définition d’un cadre de délimitation.

>[!TIP]
>La plupart des requêtes de recherche utilisent par défaut le paramétrage « maxFuzzyLevel=1 » pour optimiser les performances et réduire les résultats inhabituels. Vous pouvez ajuster les niveaux d’approximation à l’aide des paramètres `maxFuzzyLevel` ou `minFuzzyLevel`. Pour plus d’informations sur `maxFuzzyLevel` et pour obtenir la liste complète de tous les paramètres facultatifs, consultez [Paramètres d’URI de recherche approximative](/rest/api/maps/search/getsearchfuzzy#uri-parameters).

### <a name="search-for-an-address-using-fuzzy-search"></a>Rechercher une adresse à l’aide de Fuzzy Search

Dans cet exemple, nous allons utiliser la recherche approximative pour rechercher `pizza` dans le monde entier.  Ensuite, nous allons vous montrer comment effectuer une recherche sur le territoire d’un pays spécifique. Enfin, nous allons vous montrer comment utiliser un emplacement et un rayon de coordonnées pour étendre la recherche sur une zone spécifique et limiter le nombre de résultats renvoyés.

>[!IMPORTANT]
>Pour adapter les résultats géographiques en fonction de la zone pertinente pour vos utilisateurs, ajoutez toujours un maximum de détails sur l’emplacement. Pour plus d’informations, consultez [Meilleures pratiques pour la recherche](how-to-use-best-practices-for-search.md#geobiased-search-results).

1. Ouvrez l’application Postman, cliquez sur **New** (Nouveau), puis sélectionnez **Request** (Requête). Entrez un **Request name** (Nom de demande) pour la demande. Sélectionnez la collection que vous avez créée dans la section précédente ou créez-en une, puis sélectionnez **Save** (Enregistrer).

2. Sélectionnez la méthode HTTP **GET** sous l’onglet Générateur, puis entrez l’URL suivante. Pour cette requête et d’autres requêtes mentionnées dans cet article, remplacez `{Azure-Maps-Primary-Subscription-key}` par votre clé d’abonnement principale.

    ```http
   https://atlas.microsoft.com/search/fuzzy/json?&api-version=1.0&subscription-key={Azure-Maps-Primary-Subscription-key}&language=en-US&query=pizza
    ```

    >[!NOTE]
    >L’attribut _json_ dans le chemin de l’URL détermine le format de la réponse. Cet article utilise json pour faciliter l’utilisation et la lisibilité. Pour trouver d’autres formats de réponse pris en charge, consultez la définition de paramètre `format` dans la [documentation de référence sur les paramètres d’URI](/rest/api/maps/search/getsearchfuzzy#uri-parameters).

3. Cliquez sur **Send** (Envoyer), puis examinez le corps de la réponse.

    La chaîne de requête ambiguë « pizza » a retourné 10 [résultats de point d’intérêt](/rest/api/maps/search/getsearchpoi#searchpoiresponse) (POI) dans les catégories « pizza » et « restaurant ». Chaque résultat comprend des détails tels que l’adresse postale, les valeurs de latitude et longitude, le port d’affichage et les points d’entrée pour l’emplacement. Les résultats peuvent désormais varier pour cette requête et ne sont pas liés à un emplacement de référence.
  
    À l’étape suivante, nous allons utiliser le paramètre `countrySet` pour spécifier uniquement les pays/régions pour lesquels votre application a besoin d’une couverture. Pour obtenir la liste complète des pays/régions pris en charge, consultez la [couverture de recherche](./geocoding-coverage.md).

4. Le comportement par défaut consiste à faire porter la recherche sur le monde entier, avec le risque de renvoyer des résultats superflus. Ensuite, nous chercherons le mot pizza uniquement aux États-Unis. Ajoutez la clé `countrySet` à la section **Params** et attribuez-lui la valeur `US`. La définition de la clé `countrySet` sur `US` liera les résultats aux États-Unis.

    :::image type="content" source="./media/how-to-search-for-address/search-fuzzy-country.png" alt-text=" Rechercher le mot pizza aux États-Unis ":::

    Les résultats sont désormais délimités par le code de pays ; en l’occurrence, la requête retourne des pizzerias aux États-Unis.

5. Pour obtenir une recherche encore plus ciblée, vous pouvez effectuer une recherche dans l’étendue d’une paire de coordonnées latitude/longitude. Dans cet exemple, nous allons utiliser la latitude/longitude de la Space Needle de Seattle. Comme nous voulons uniquement retourner les résultats qui se trouvent dans un rayon de 400 mètres, nous allons ajouter le paramètre `radius`. Nous allons également ajouter le paramètre `limit` pour limiter les résultats aux cinq pizzerias les plus proches.

    Dans la section **Params** , ajoutez les paires clé-valeur suivantes :

     | Clé | Valeur |
    |-----|------------|
    | lat | 47.620525 |
    | lon | -122.349274 |
    | radius | 400 |
    | limit | 5|

6. Cliquez sur **Envoyer**. La réponse comprend des résultats pour les pizzerias situées à proximité de la Space Needle de Seattle.

## <a name="search-for-a-street-address-using-reverse-address-search"></a>Rechercher une adresse postale à l’aide d’une recherche d’adresse inverse

L’[API Get Search Address Reverse]( https://docs.microsoft.com/rest/api/maps/search/getsearchaddressreverse) d’Azure Maps traduit les coordonnées en adresses lisibles. Cette API est souvent utilisée pour les applications qui utilisent des flux GPS et qui souhaitent découvrir les adresses de points de coordonnées spécifiques.

>[!IMPORTANT]
>Pour adapter les résultats géographiques en fonction de la zone pertinente pour vos utilisateurs, ajoutez toujours un maximum de détails sur l’emplacement. Pour plus d’informations, consultez [Meilleures pratiques pour la recherche](how-to-use-best-practices-for-search.md#geobiased-search-results).

>[!TIP]
>Si vous avez un ensemble de coordonnées dont vous voulez inverser le géocodage pour obtenir l’adresse correspondante, vous pouvez utiliser l’[API Post Search Address Reverse Batch](/rest/api/maps/search/postsearchaddressreversebatch) pour envoyer un lot de requêtes dans un seul appel d’API.

Dans cet exemple, nous allons effectuer des recherches inversées à l’aide de quelques-uns des paramètres facultatifs disponibles. Pour obtenir la liste complète des paramètres facultatifs, consultez [Paramètres de recherche inversée](/rest/api/maps/search/getsearchaddressreverse#uri-parameters).

1. Dans l’application Postman, cliquez sur **New** (Nouveau), puis sélectionnez **Request** (Requête). Entrez un **Request name** (Nom de demande) pour la demande. Sélectionnez la collection que vous avez créée dans la première section ou créez-en une, puis sélectionnez **Save** (Enregistrer).

2. Sélectionnez la méthode HTTP **GET** sous l’onglet Générateur, puis entrez l’URL suivante. Pour cette requête et d’autres requêtes mentionnées dans cet article, remplacez `{Azure-Maps-Primary-Subscription-key}` par votre clé d’abonnement principale. La requête doit ressembler à l’URL suivante :

    ```http
    https://atlas.microsoft.com/search/address/reverse/json?api-version=1.0&subscription-key={Azure-Maps-Primary-Subscription-key}&language=en-US&query=47.591180,-122.332700&number=1
    ```

3. Cliquez sur **Send** (Envoyer), puis examinez le corps de la réponse. Vous devez voir un résultat de requête. La réponse inclut des informations d’adresse clés sur Safeco Field.
  
4. À présent, nous allons ajouter les paires clé-valeur suivantes à la section **Params**  :

    | Clé | Valeur | Retours
    |-----|------------|------|
    | nombre | 1 |La réponse peut inclure le côté de la rue (gauche/droite) et également une position décalée pour le numéro.|
    | returnSpeedLimit | true | Retourne la limite de vitesse à l’adresse.|
    | returnRoadUse | true | Retourne les types d’utilisation de la route à l’adresse. Pour tous les types d’utilisation routière possibles, consultez [Types d’utilisation routière](/rest/api/maps/search/getsearchaddressreverse#uri-parameters).|
    | returnMatchType | true| Retourne le type de correspondance. Pour toutes les valeurs possibles, consultez [Résultats de la recherche d’adresse inversée](/rest/api/maps/search/getsearchaddressreverse#searchaddressreverseresult).

   :::image type="content" source="./media/how-to-search-for-address/search-reverse.png" alt-text="Recherche inversée.":::

5. Cliquez sur **Send** (Envoyer), puis examinez le corps de la réponse.

6. Ensuite, nous allons ajouter la clé `entityType` et définir sa valeur sur `Municipality`. La clé `entityType` remplace la clé `returnMatchType` de l’étape précédente. Nous devrons également supprimer `returnSpeedLimit` et `returnRoadUse`, puisque nous demandons des informations sur la commune.  Pour tous les types d’entités possibles, consultez [Types d’entités](/rest/api/maps/search/getsearchaddressreverse#entitytype).

    :::image type="content" source="./media/how-to-search-for-address/search-reverse-entity-type.png" alt-text="Recherche d’entityType inversée.":::

7. Cliquez sur **Envoyer**. Comparez les résultats à ceux retournés à l’étape 5.  Étant donné que le type d’entité demandé est désormais `municipality`, la réponse n’inclut pas d’informations sur l’adresse. En outre, la valeur `geometryId` retournée peut être utilisée pour demander un polygone de délimitation par le biais de l’[API Get Search Polygon](/rest/api/maps/search/getsearchpolygon) d’Azure Maps.

>[!TIP]
>Pour obtenir plus d’informations sur ces paramètres, ainsi que pour en connaître d’autres, consultez la section [Paramètres de recherche inversée](/rest/api/maps/search/getsearchaddressreverse#uri-parameters).

## <a name="search-for-cross-street-using-reverse-address-cross-street-search"></a>Rechercher une intersection à l’aide d’une recherche d’intersection d’adresse inverse

Dans cet exemple, nous allons rechercher une intersection en nous basant sur les coordonnées d’une adresse.

1. Dans l’application Postman, cliquez sur **New** (Nouveau), puis sélectionnez **Request** (Requête). Entrez un **Request name** (Nom de demande) pour la demande. Sélectionnez la collection que vous avez créée dans la première section ou créez-en une, puis sélectionnez **Save** (Enregistrer).

2. Sélectionnez la méthode HTTP **GET** sous l’onglet Générateur, puis entrez l’URL suivante. Pour cette requête et d’autres requêtes mentionnées dans cet article, remplacez `{Azure-Maps-Primary-Subscription-key}` par votre clé d’abonnement principale. La requête doit ressembler à l’URL suivante :
  
    ```http
   https://atlas.microsoft.com/search/address/reverse/crossstreet/json?&api-version=1.0&subscription-key={Azure-Maps-Primary-Subscription-key}&language=en-US&query=47.591180,-122.332700
    ```

    :::image type="content" source="./media/how-to-search-for-address/search-address-cross.png" alt-text="Rechercher une intersection.":::
  
3. Cliquez sur **Send** (Envoyer), puis examinez le corps de la réponse. Vous remarquerez que la réponse contient une valeur `crossStreet` pour `Occidental Avenue South`.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [API REST du service de recherche d’Azure Maps](/rest/api/maps/search)

> [!div class="nextstepaction"]
> [Meilleures pratiques d’utilisation du service de recherche d’Azure Maps](how-to-use-best-practices-for-search.md)