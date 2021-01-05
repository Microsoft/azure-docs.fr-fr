---
title: 'Tutoriel : Extraire les détails d’une image avec l’API REST et C# - Recherche d’images Bing'
titleSuffix: Azure Cognitive Services
description: Utilisez ce tutoriel pour créer une application C# qui extrait les détails de l’image avec l’API Recherche d’images Bing.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-image-search
ms.topic: tutorial
ms.date: 12/06/2019
ms.author: aahi
ms.custom: devx-track-js
ms.openlocfilehash: cef39b940f0d61fc60ea10156acfd781289157de
ms.sourcegitcommit: 1cf157f9a57850739adef72219e79d76ed89e264
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94591969"
---
# <a name="tutorial-extract-image-details-using-the-bing-image-search-api-and-c"></a>Tutoriel : Extraire les détails de l’image avec l’API Recherche d’images Bing et C#

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Contrat Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Plusieurs [points de terminaison](./image-search-endpoint.md) sont disponibles via l’API Recherche d’images Bing. Le point de terminaison `/details` accepte une requête POST avec une image, et peut retourner divers détails relatifs à l’image. Cette application C# envoie une image à l’aide de cette API et affiche les détails retournés par Bing, qui sont des objets JSON, par exemple les éléments suivants :

![[Résultats JSON]](media/cognitive-services-bing-images-api/jsonResult.jpg)

Ce didacticiel explique comment :

> [!div class="checklist"]
> * Utiliser le point de terminaison Recherche d’images `/details` dans une requête `POST`
> * Spécifier des en-têtes pour la requête
> * Utiliser les paramètres d’URL pour spécifier les résultats
> * Télécharger les données de l’image et envoyer la requête `POST`
> * Imprimer les résultats JSON vers la console

Le code source de cet exemple est disponible sur [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/Tutorials/BingGetSimilarImages.cs).

## <a name="prerequisites"></a>Prérequis

* N’importe quelle édition de [Visual Studio 2017 ou ultérieure](https://visualstudio.microsoft.com/downloads/).

[!INCLUDE [cognitive-services-bing-image-search-signup-requirements](../../../includes/cognitive-services-bing-image-search-signup-requirements.md)]

## <a name="construct-an-image-details-search-request"></a>Construire une requête de recherche de détails d’image

Voici le point de terminaison `/details`, qui accepte les requêtes POST avec des données d’image dans le corps de la requête. Vous pouvez utiliser le point de terminaison global ci-dessous, ou le point de terminaison de [sous-domaine personnalisé](../../cognitive-services/cognitive-services-custom-subdomains.md) affiché dans le portail Azure pour votre ressource.
```
https://api.cognitive.microsoft.com/bing/v7.0/images/details
```

Lors de la construction de l’URL de requête de recherche, le paramètre `modules` suit le point de terminaison ci-dessus et spécifie les types de détails que contiennent les résultats :

* `modules=All`
* `modules=RecognizedEntities` (personnes ou lieux visibles dans l’image)

Spécifiez `modules=All` dans la requête POST pour obtenir du texte JSON qui inclut les éléments suivants :

* `bestRepresentativeQuery` - une requête Bing qui renvoie des images similaire à l’image chargée
* `detectedObjects` : objets se trouvant dans l’image
* `image` : métadonnées de l’image
* `imageInsightsToken` : un jeton pour une requête GET ultérieure qui obtient `RecognizedEntities` (personnes ou endroits visibles dans l’image) à partir de l’image.
* `imageTags` : étiquettes pour l’image
* `pagesIncluding` - pages Web où l’image est intégrée
* `relatedSearches` - recherches basées sur les détails de l’image.
* `visuallySimilarImages` - images similaires sur le web.

Spécifiez `modules=RecognizedEntities` dans la requête POST pour obtenir uniquement `imageInsightsToken`, qui peut être utilisé dans une requête GET ultérieure pour identifier les personnes ou les lieux figurant dans l’image.

## <a name="create-a-webclient-object-and-set-headers-for-the-api-request"></a>Créer un objet WebClient et définir des en-têtes pour la requête d’API

Créez un objet `WebClient` et définissez les en-têtes. Pour effectuer des requêtes vers l’API Recherche Bing, vous avez besoin d’une clé `Ocp-Apim-Subscription-Key`. La requête `POST` permettant de charger une image doit également spécifier `ContentType: multipart/form-data`.

```javascript
WebClient client = new WebClient();
client.Headers["Ocp-Apim-Subscription-Key"] = accessKey;
client.Headers["ContentType"] = "multipart/form-data";
```

## <a name="upload-the-image-and-display-the-results"></a>Charger l’image et afficher les résultats

La méthode `UpLoadFile()` de la classe `WebClient` met en forme les données de la requête `POST`, notamment la mise en forme de `RequestStream` et l’appel de `HttpWebRequest`.

Appelez `WebClient.UpLoadFile()` avec le point de terminaison `/details` et le fichier image à télécharger. Utilisez la réponse JSON pour initialiser une instance de la structure `SearchResult` et stocker la réponse.

```javascript        
const string uriBase = "https://api.cognitive.microsoft.com/bing/v7.0/images/details";
// The image to upload. Replace with your file and path.
const string imageFile = "your-image.jpg";
byte[] resp = client.UploadFile(uriBase + "?modules=All", imageFile);
var json = System.Text.Encoding.Default.GetString(resp);
// Create result object for return
var searchResult = new SearchResult()
{
    jsonResult = json,
    relevantHeaders = new Dictionary<String, String>()
};
```
Cette réponse JSON peut ensuite être affichée sur la console.

## <a name="use-an-image-insights-token-in-a-request"></a>Utiliser un jeton d’insights sur l’image dans une requête

Pour utiliser le `ImageInsightsToken` retourné avec les résultats d’un `POST`, vous pouvez l’ajouter à une requête `GET`. Par exemple :

```
https://api.cognitive.microsoft.com/bing/v7.0/images/details?InsightsToken="bcid_A2C4BB81AA2C9EF8E049C5933C546449*ccid_osS7gaos*mid_BF7CC4FC4A882A3C3D56E644685BFF7B8BACEAF2
```

Si des personnes ou des lieux peuvent être identifiés dans l’image, cette requête renverra des informations à leur sujet.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Afficher des images et des options de recherche dans une application web monopage ](tutorial-bing-image-search-single-page-app.md)

## <a name="see-also"></a>Voir aussi

* [Informations de référence sur l’API Recherche d’images Bing](//docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference)