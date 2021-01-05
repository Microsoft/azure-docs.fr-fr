---
title: 'Tutoriel : Rogner une image avec le SDK Recherche visuelle Bing'
description: Utilisez le SDK Recherche visuelle Bing pour obtenir des informations à partir d’une zone spécifique d’une image.
services: cognitive-services
titleSuffix: Azure Cognitive Services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-visual-search
ms.topic: tutorial
ms.date: 03/31/2019
ms.author: aahi
ms.custom: devx-track-csharp
ms.openlocfilehash: 0ad73d7086a2066517c24a8b10a7f54503535942
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96490293"
---
# <a name="tutorial-crop-an-image-with-the-bing-visual-search-sdk-for-c"></a>Tutoriel : Rogner une image avec le SDK Recherche visuelle Bing pour C#

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Contrat Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Le SDK Recherche visuelle Bing vous permet de rogner une image avant de rechercher des images en ligne similaires. Cette application rogne une seule personne à partir d’une image contenant plusieurs personnes, puis retourne les résultats de recherche contenant des images similaires disponibles en ligne.

Le code source complet de cette application est disponible sur [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/Tutorials/Bing-Visual-Search/BingVisualSearchCropImage.cs) avec une gestion des erreurs et des annotations supplémentaires.

Le didacticiel illustre les actions suivantes :

> [!div class="checklist"]
> * Envoyer une requête à l’aide du SDK Recherche visuelle Bing
> * Rogner une zone d’image pour effectuer une recherche avec Recherche visuelle Bing
> * Recevoir et gérer la réponse
> * Rechercher les URL d’éléments d’action dans la réponse

## <a name="prerequisites"></a>Prérequis

* N’importe quelle édition de [Visual Studio 2019](https://www.visualstudio.com/downloads/).
* Si vous utilisez Linux/MacOS, cette application peut être exécutée à l’aide de [Mono](https://www.mono-project.com/).
* Package [NuGet Custom Search](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Search.CustomSearch/1.2.0) installé.
    - Dans l’Explorateur de solutions de Visual Studio, cliquez avec le bouton droit sur votre projet et sélectionnez **Gérer les packages NuGet** dans le menu. Installez le package `Microsoft.Azure.CognitiveServices.Search.CustomSearch`. L’installation du package NuGet Custom Search installe aussi les assemblys suivants :
        - Microsoft.Rest.ClientRuntime
        - Microsoft.Rest.ClientRuntime.Azure
        - Newtonsoft.Json

[!INCLUDE [cognitive-services-bing-image-search-signup-requirements](../../../includes/cognitive-services-bing-visual-search-signup-requirements.md)]

## <a name="specify-the-image-crop-area"></a>Spécifier la zone d'image à rogner

Cette application rogne une zone de cette image de l’Équipe des hauts dirigeants de Microsoft. Cette zone rognée est définie à l’aide des coordonnées des coins supérieur gauche et inférieur droit, représentées sous forme de pourcentage de la totalité de l’image :  

![Équipe des hauts dirigeants de Microsoft](./media/MS_SrLeaders.jpg)

Cette image est rognée par la création d’un objet `ImageInfo` à partir de la zone de rognage et par le chargement de l’objet `ImageInfo` dans un `VisualSearchRequest`. L’objet `ImageInfo` inclut également l’URL de l’image :

```csharp
CropArea CropArea = new CropArea(top: (float)0.01, bottom: (float)0.30, left: (float)0.01, right: (float)0.20);
string imageURL = "https://docs.microsoft.com/azure/cognitive-services/bing-visual-search/media/ms_srleaders.jpg";
ImageInfo imageInfo = new ImageInfo(cropArea: CropArea, url: imageURL);

VisualSearchRequest visualSearchRequest = new VisualSearchRequest(imageInfo: imageInfo);
```

## <a name="search-for-images-similar-to-the-crop-area"></a>Rechercher des images similaires à la zone de rognage

La variable `VisualSearchRequest` contient des informations sur la zone de rognage de l’image et son URL. La méthode `VisualSearchMethodAsync()` obtient les résultats :

```csharp
Console.WriteLine("\r\nSending visual search request with knowledgeRequest that contains URL and crop area");
var visualSearchResults = client.Images.VisualSearchMethodAsync(knowledgeRequest: visualSearchRequest).Result;

```

## <a name="get-the-url-data-from-imagemoduleaction"></a>Obtenir les données d’URL à partir de `ImageModuleAction`

Les résultats de Recherche visuelle Bing sont des objets `ImageTag`. Chaque balise contient une liste d’objets `ImageAction`. Chaque `ImageAction` contient un champ `Data`, qui est une liste de valeurs variant en fonction du type d’action.

Vous pouvez imprimer les différents types avec le code suivant :

```csharp
Console.WriteLine("\r\n" + "ActionType: " + i.ActionType + " -> WebSearchUrl: " + i.WebSearchUrl);
```

L’application complète retourne :

|ActionType  |URL  |
|---------|---------|
|PagesIncluding WebSearchURL     |         |
|MoreSizes WebSearchURL     |         |  
|VisualSearch WebSearchURL    |         |
|ImageById WebSearchURL     |         |  
|RelatedSearches WebSearchURL     |         |
|Entity -> WebSearchUrl     | https\://www.bing.com/cr?IG=E40D0E1A13404994ACB073504BC937A4&CID=03DCF882D7386A442137F49BD6596BEF&rd=1&h=BvvDoRtmZ35Xc_UZE4lZx6_eg7FHgcCkigU1D98NHQo&v=1&r=https%3a%2f%2f www.bing.com%2fsearch%3fq%3dSatya%2bNadella&p=DevEx,5380.1        |
|TopicResults -> WebSearchUrl    |  https\://www.bing.com/cr?IG=E40D0E1A13404994ACB073504BC937A4&CID=03DCF882D7386A442137F49BD6596BEF&rd=1&h=3QGtxPb3W9LemuHRxAlW4CW7XN4sPkUYCUynxAqI9zQ&v=1&r=https%3a%2f%2f www.bing.com%2fdiscover%2fnadella%2bsatya&p=DevEx,5382.1        |
|ImageResults -> WebSearchUrl    |  https\://www.bing.com/cr?IG=E40D0E1A13404994ACB073504BC937A4&CID=03DCF882D7386A442137F49BD6596BEF&rd=1&h=l-WNHO89Kkw69AmIGe2MhlUp6MxR6YsJszgOuM5sVLs&v=1&r=https%3a%2f%2f www.bing.com%2fimages%2fsearch%3fq%3dSatya%2bNadella&p=DevEx,5384.1        |

Comme indiqué ci-dessus, l’ActionType `Entity` contient une requête de recherche Bing qui retourne des informations sur une personne, un endroit ou une chose reconnaissable. Les types `TopicResults` et `ImageResults` contiennent des requêtes pour les images associées. Les URL dans la liste sont liées aux résultats de recherche Bing.

## <a name="get-urls-for-pagesincluding-actiontype-images"></a>Obtenir les URL pour les images `PagesIncluding` `ActionType`

L’obtention des URL des images réelles nécessite un cast qui lit un `ActionType` comme `ImageModuleAction`, qui contient un élément `Data` avec une liste de valeurs. Chaque valeur est l’URL d’une image. Le code suivant convertit le type d’action `PagesIncluding` en `ImageModuleAction` et lit les valeurs :

```csharp
    if (i.ActionType == "PagesIncluding")
    {
        foreach(ImageObject o in (i as ImageModuleAction).Data.Value)
        {
            Console.WriteLine("ContentURL: " + o.ContentUrl);
        }
    }
```

## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Créer une application web monopage de recherche visuelle](tutorial-bing-visual-search-single-page-app.md)

## <a name="see-also"></a>Voir aussi
> [Qu’est-ce que l’API Recherche visuelle Bing ?](./overview.md)