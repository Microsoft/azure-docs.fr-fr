---
title: Rechercher des images similaires à celles issues de recherches précédentes en utilisant des jetons d’insights d’images et l’API Recherche visuelle Bing
titleSuffix: Azure Cognitive Services
description: Utilisez la bibliothèque de client Recherche visuelle Bing pour obtenir les URL d’images de recherches précédentes.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-visual-search
ms.topic: tutorial
ms.date: 03/31/2020
ms.author: aahi
ms.custom: devx-track-csharp
ms.openlocfilehash: a9005082148803423ac20b4f18dd974bb5b43b80
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96486859"
---
# <a name="tutorial-find-similar-images-from-previous-searches-using-an-image-insights-token"></a>Tutoriel : Rechercher des images similaires dans des recherches précédentes à l’aide d’un jeton d’insights d’image

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Contrat Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

La bibliothèque de client Recherche visuelle Bing vous permet de rechercher des images en ligne à partir de recherches précédentes qui retournent un `ImageInsightsToken`. Cette application obtient un `ImageInsightsToken` et utilise le jeton dans une recherche ultérieure. Elle envoie alors le `ImageInsightsToken` à Bing, puis retourne des résultats qui incluent les URL de recherche Bing et les URL des images similaires disponibles en ligne.

Le code source complet de ce didacticiel est disponible sur [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/Tutorials/Bing-Visual-Search/BingVisualSearchInsightsTokens.cs) avec une gestion des erreurs et des annotations supplémentaires.

## <a name="prerequisites"></a>Prérequis

* N’importe quelle édition de [Visual Studio 2019](https://www.visualstudio.com/downloads/).
* Si vous utilisez Linux/MacOS, vous pouvez exécuter cette application en utilisant [Mono](https://www.mono-project.com/).
* Les packages Recherche visuelle NuGet et Recherche d’images.
    - Dans l’Explorateur de solutions de Visual Studio, cliquez avec le bouton droit sur votre projet et sélectionnez **Gérer les packages NuGet** dans le menu. Installez le package `Microsoft.Azure.CognitiveServices.Search.CustomSearch` et le package `Microsoft.Azure.CognitiveServices.Search.ImageSearch`. L’installation des packages NuGet installe également les éléments suivants :
        - Microsoft.Rest.ClientRuntime
        - Microsoft.Rest.ClientRuntime.Azure
        - Newtonsoft.Json


[!INCLUDE [cognitive-services-bing-visual-search-signup-requirements](../../../includes/cognitive-services-bing-visual-search-signup-requirements.md)]

## <a name="get-the-imageinsightstoken-from-the-bing-image-search-client-library"></a>Obtenir le jeton ImageInsightsToken à partir de la bibliothèque de client Recherche d’images Bing

Cette application utilise un `ImageInsightsToken` obtenu par l’intermédiaire de la [bibliothèque de client Recherche d’images Bing](../bing-image-search/quickstarts/client-libraries.md?pivots=programming-language-csharp%253fpivots%253dprogramming-language-csharp). Dans une nouvelle application console C#, créez un client pour appeler l’API en utilisant `ImageSearchClient()`. Utilisez ensuite `SearchAsync()` avec votre requête :

```csharp
var client = new ImageSearchClient(new Microsoft.Azure.CognitiveServices.Search.ImageSearch.ApiKeyServiceClientCredentials(subKey));
var imageResults = client.Images.SearchAsync(query: "canadian rockies").Result;
Console.WriteLine("Search images for query \"canadian rockies\"");
```

Enregistrez le premier résultat de la recherche à l’aide de `imageResults.Value.First()`, puis enregistrez la valeur `ImageInsightsToken` des informations d’image.

```csharp
String insightTok = "None";
if (imageResults.Value.Count > 0)
{
    var firstImageResult = imageResults.Value.First();
    insightTok = firstImageResult.ImageInsightsToken;
}
else
{
    insightTok = "None found";
    Console.WriteLine("Couldn't find image results!");
}
```

Cette valeur `ImageInsightsToken` est envoyée à Recherche visuelle Bing dans une requête.

## <a name="add-the-imageinsightstoken-to-a-visual-search-request"></a>Ajouter le jeton ImageInsightsToken à une requête de Recherche visuelle

Spécifiez la valeur `ImageInsightsToken` dans une requête Recherche visuelle en créant un objet `ImageInfo` à partir de la valeur `ImageInsightsToken` contenue dans les réponses de Recherche visuelle Bing.

```csharp
ImageInfo ImageInfo = new ImageInfo(imageInsightsToken: insightsTok);
```

## <a name="use-bing-visual-search-to-find-images-from-an-imageinsightstoken"></a>Utiliser la Recherche visuelle Bing pour trouver des images à partir d’un jeton ImageInsightsToken

L’objet `VisualSearchRequest` contient des informations sur l’image à rechercher dans `ImageInfo`. La méthode `VisualSearchMethodAsync()` obtient les résultats. Vous n’avez pas besoin de fournir un binaire d’image, car l’image est représentée par le jeton.

```csharp
VisualSearchRequest VisualSearchRequest = new VisualSearchRequest(ImageInfo);

var visualSearchResults = client.Images.VisualSearchMethodAsync(knowledgeRequest: VisualSearchRequest).Result;

```

## <a name="iterate-through-the-visual-search-results"></a>Effectuer une itération dans les résultats de Recherche visuelle Bing

Les résultats de Recherche visuelle sont des objets `ImageTag`. Chaque balise contient une liste d’objets `ImageAction`. Chaque `ImageAction` contient un champ `Data`, qui est une liste de valeurs variant en fonction du type d’action. Vous pouvez itérer à travers les objets `ImageTag` dans `visualSearchResults.Tags`, par exemple, et obtenir la balise `ImageAction` intégrée. L’exemple ci-dessous imprime les détails d’actions `PagesIncluding` :

```csharp
if (visualSearchResults.Tags.Count > 0)
{
    // List of tags
    foreach (ImageTag t in visualSearchResults.Tags)
    {
        foreach (ImageAction i in t.Actions)
        {
            Console.WriteLine("\r\n" + "ActionType: " + i.ActionType + " WebSearchURL: " + i.WebSearchUrl);

            if (i.ActionType == "PagesIncluding")
            {
                foreach (ImageObject o in (i as ImageModuleAction).Data.Value)
                {
                    Console.WriteLine("ContentURL: " + o.ContentUrl);
                }
            }
        }
    }
}
```

### <a name="pagesincluding-actiontypes"></a>ActionTypes PagesIncluding

L’obtention des URL des images réelles à partir de types d’action nécessite un cast qui lit un `ActionType` comme `ImageModuleAction`, qui contient un élément `Data` avec une liste de valeurs. Chaque valeur est l’URL d’une image.  Le code suivant convertit le type d’action `PagesIncluding` en `ImageModuleAction` et lit les valeurs :

```csharp
    if (i.ActionType == "PagesIncluding")
    {
        foreach(ImageObject o in (i as ImageModuleAction).Data.Value)
        {
            Console.WriteLine("ContentURL: " + o.ContentUrl);
        }
    }
```

Pour plus d’informations sur ces types de données, consultez [Images : Recherche visuelle](/rest/api/cognitiveservices/bingvisualsearch/images/visualsearch).

## <a name="returned-urls"></a>URL retournées

L’application complète retourne les URL suivantes :

|ActionType  |URL  |
|---------|---------|
|MoreSizes -> WebSearchUrl     |         |
|VisualSearch -> WebSearchUrl     |         |
|ImageById -> WebSearchUrl    |         |
|RelatedSearches -> WebSearchUrl :    |         |
|DocumentLevelSuggestions -> WebSearchUrl :     |         |
|TopicResults -> WebSearchUrl    | https:\//www.bing.com/cr?IG=3E32CC6CA5934FBBA14ABC3B2E4651F9&CID=1BA795A21EAF6A63175699B71FC36B7C&rd=1&h=BcQifmzdKFyyBusjLxxgO42kzq1Geh7RucVVqvH-900&v=1&r=https%3a%2f%2f www.bing.com%2fdiscover%2fcanadian%2brocky&p=DevEx,5823.1       |
|ImageResults -> WebSearchUrl    |  https:\//www.bing.com/cr?IG=3E32CC6CA5934FBBA14ABC3B2E4651F9&CID=1BA795A21EAF6A63175699B71FC36B7C&rd=1&h=PV9GzMFOI0AHZp2gKeWJ8DcveSDRE3fP2jHDKMpJSU8&v=1&r=https%3a%2f%2f www.bing.com%2fimages%2fsearch%3fq%3doutdoor&p=DevEx,5831.1       |

Comme indiqué ci-dessus, les types `TopicResults` et `ImageResults` contiennent des requêtes pour les images associées. Les URL sont liées aux résultats de recherche Bing.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer une application web monopage de recherche visuelle](tutorial-bing-visual-search-single-page-app.md)