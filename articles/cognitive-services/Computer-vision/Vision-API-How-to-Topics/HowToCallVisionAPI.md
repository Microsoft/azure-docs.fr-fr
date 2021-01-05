---
title: Appeler l’API Vision par ordinateur
titleSuffix: Azure Cognitive Services
description: Découvrez comment appeler l’API Vision par ordinateur avec l’API REST dans Azure Cognitive Services.
services: cognitive-services
author: KellyDF
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: sample
ms.date: 09/09/2019
ms.author: kefre
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: e5e404e4cc0e4131ba7ade5ec8ec6115e26268a2
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91965894"
---
# <a name="call-the-computer-vision-api"></a>Appeler l’API Vision par ordinateur

Cet article explique comment appeler l’API Vision par ordinateur avec l’API REST. Les exemples sont écrits en C# avec la bibliothèque de client d’API Vision par ordinateur et en tant qu’appels HTTP POST ou GET. Cet article se concentre sur les points suivants :

- Obtention d’étiquettes, d’une description et de catégories
- Obtention d’informations spécifique à un domaine, ou « celebrities » (célébrités)

Les exemples de cet article illustrent les fonctionnalités suivantes :

* Analyse d’une image pour retourner un tableau d’étiquettes et une description
* Analyse d’une image avec un modèle spécifique à un domaine (ici, le modèle « celebrities ») pour retourner le résultat correspondant au format JSON

Les fonctionnalités offrent les options suivantes :

- **Option 1** : Analyse délimitée - Analyser seulement un modèle spécifié
- **Option 2** : Analyse élargie - Analyser pour fournir des détails supplémentaires en utilisant une [taxonomie de 86 catégories](../Category-Taxonomy.md)

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure - [En créer un gratuitement](https://azure.microsoft.com/free/cognitive-services/)
* Une fois que vous avez votre abonnement Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision"  title="créez une ressource Vision par ordinateur"  target="_blank">créer une ressource Vision par ordinateur <span class="docon docon-navigate-external x-hidden-focus"></span></a> dans le Portail Azure pour obtenir votre clé et votre point de terminaison. Une fois le déploiement effectué, cliquez sur **Accéder à la ressource**.
    * Vous aurez besoin de la clé et du point de terminaison de la ressource que vous créez pour connecter votre application au service Vision par ordinateur. Vous collerez votre clé et votre point de terminaison dans le code ci-dessous plus loin dans le guide de démarrage rapide.
    * Vous pouvez utiliser le niveau tarifaire Gratuit (`F0`) pour tester le service, puis passer par la suite à un niveau payant pour la production.
* Une URL d’image ou un chemin vers l’image stockée localement
* Méthodes d’entrée prises en charge : image raw binaire sous forme de flux application/octet ou une URL d’image
* Formats de fichier d’image pris en charge : JPEG, PNG, GIF et BMP
* Taille du fichier image : 4 Mo ou moins
* Dimensions de l’image : 50 &times; 50 pixels ou plus
  
## <a name="authorize-the-api-call"></a>Autoriser l’appel d’API

Chaque appel de l’API Vision par ordinateur nécessite une clé d’abonnement. Cette clé peut être passée via un paramètre de chaîne de requête ou spécifiée dans l’en-tête de la demande.

Vous pouvez passer la clé d’abonnement de l’une des façons suivantes :

* Passez-la via une chaîne de requête, comme dans cet exemple de l’API Vision par ordinateur :

  ```
  https://westus.api.cognitive.microsoft.com/vision/v2.1/analyze?visualFeatures=Description,Tags&subscription-key=<Your subscription key>
  ```

* Spécifiez-la dans l’en-tête de requête HTTP :

  ```
  ocp-apim-subscription-key: <Your subscription key>
  ```

* Quand vous utilisez la bibliothèque de client, passez la clé via le constructeur de ComputerVisionClient et spécifiez la région dans une propriété du client :

    ```
    var visionClient = new ComputerVisionClient(new ApiKeyServiceClientCredentials("Your subscriptionKey"))
    {
        Endpoint = "https://westus.api.cognitive.microsoft.com"
    }
    ```

## <a name="upload-an-image-to-the-computer-vision-api-service"></a>Charger une image sur le service API Vision par ordinateur

La façon la plus simple d’effectuer l’appel de l’API Vision par ordinateur est de charger une image directement pour retourner des étiquettes, une description et des « célébrités ». Pour cela, vous envoyez une demande de « POST » avec l’image binaire dans le corps HTTP ainsi que les données lues à partir de l’image. La méthode de chargement est la même pour tous les appels de l’API Vision par ordinateur. La seule différence concerne les paramètres de requête que vous spécifiez. 

Pour une image spécifiée, obtenez des étiquettes et une description en utilisant une des options suivantes :

### <a name="option-1-get-a-list-of-tags-and-a-description"></a>Option 1 : Obtenir une liste d’étiquettes et une description

```
POST https://westus.api.cognitive.microsoft.com/vision/v2.1/analyze?visualFeatures=Description,Tags&subscription-key=<Your subscription key>
```

```csharp
using System.IO;
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision;
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision.Models;

ImageAnalysis imageAnalysis;
var features = new VisualFeatureTypes[] { VisualFeatureTypes.Tags, VisualFeatureTypes.Description };

using (var fs = new FileStream(@"C:\Vision\Sample.jpg", FileMode.Open))
{
  imageAnalysis = await visionClient.AnalyzeImageInStreamAsync(fs, features);
}
```

### <a name="option-2-get-a-list-of-tags-only-or-a-description-only"></a>Option n°2 : Obtenir seulement une liste d’étiquettes ou seulement une description

Pour les étiquettes seulement, exécutez :

```
POST https://westus.api.cognitive.microsoft.com/vision/v2.1/tag?subscription-key=<Your subscription key>
var tagResults = await visionClient.TagImageAsync("http://contoso.com/example.jpg");
```

Pour une description seulement, exécutez :

```
POST https://westus.api.cognitive.microsoft.com/vision/v2.1/describe?subscription-key=<Your subscription key>
using (var fs = new FileStream(@"C:\Vision\Sample.jpg", FileMode.Open))
{
  imageDescription = await visionClient.DescribeImageInStreamAsync(fs);
}
```

## <a name="get-domain-specific-analysis-celebrities"></a>Obtenir une analyse spécifique au domaine (célébrités)

### <a name="option-1-scoped-analysis---analyze-only-a-specified-model"></a>Option 1 : Analyse délimitée - Analyser seulement un modèle spécifié
```
POST https://westus.api.cognitive.microsoft.com/vision/v2.1/models/celebrities/analyze
var celebritiesResult = await visionClient.AnalyzeImageInDomainAsync(url, "celebrities");
```

Avec cette option, les autres paramètres de requête {visualFeatures, details} ne sont pas valides. Pour afficher la liste des modèles pris en charge, utilisez la commande suivante :

```
GET https://westus.api.cognitive.microsoft.com/vision/v2.1/models 
var models = await visionClient.ListModelsAsync();
```

### <a name="option-2-enhanced-analysis---analyze-to-provide-additional-details-by-using-86-categories-taxonomy"></a>Option n°2 : Analyse élargie - Analyser pour fournir des détails supplémentaires en utilisant une taxonomie de 86 catégories

Dans les applications où vous voulez obtenir une analyse d’image générique en plus des détails issus d’un ou plusieurs modèles spécifiques à un domaine, étendez l’API v1 avec le paramètre de requête « models ».

```
POST https://westus.api.cognitive.microsoft.com/vision/v2.1/analyze?details=celebrities
```

Quand vous appelez cette méthode, vous appelez d’abord le classifieur [86-category](../Category-Taxonomy.md) (86 catégories). Si aucune des catégories ne correspond à celle d’un modèle connu ou correspondant, une deuxième passe d’appels du classifieur est effectuée. Par exemple, si "details=all" ou "details" inclut "celebrities", vous appelez le modèle « celebrities » après avoir appelé le classifieur de 86 catégories. Le résultat inclut la catégorie person. Contrairement à l’option 1, cette méthode augmente la latence pour les utilisateurs qui sont intéressés par les célébrités.

Dans ce cas, tous les paramètres de requête v1 se comportent de la même façon. Si vous ne spécifiez pas visualFeatures=categories, il est activé implicitement.

## <a name="retrieve-and-understand-the-json-output-for-analysis"></a>Récupérer et comprendre la sortie JSON pour l’analyse

Voici un exemple :

```json
{  
  "tags":[  
    {  
      "name":"outdoor",
      "score":0.976
    },
    {  
      "name":"bird",
      "score":0.95
    }
  ],
  "description":{  
    "tags":[  
      "outdoor",
      "bird"
    ],
    "captions":[  
      {  
        "text":"partridge in a pear tree",
        "confidence":0.96
      }
    ]
  }
}
```

Champ | Type | Contenu
------|------|------|
Balises  | `object` | Objet de plus haut niveau pour un tableau d’étiquettes.
tags[].Name | `string`    | Mot clé du classifieur d’étiquettes.
tags[].Score    | `number`    | Score de confiance, compris entre 0 et 1.
description     | `object`    | Objet de plus haut niveau pour une description.
description.tags[] |    `string`    | Liste d’étiquettes.  En cas confiance insuffisante pour pouvoir produire une légende, les étiquettes peuvent être les seules informations disponibles pour l’appelant.
description.captions[].text    | `string`    | Expression décrivant l’image.
description.captions[].confidence    | `number`    | Score de confiance pour l’expression.

## <a name="retrieve-and-understand-the-json-output-of-domain-specific-models"></a>Récupérer et comprendre la sortie JSON des modèles spécifiques au domaine

### <a name="option-1-scoped-analysis---analyze-only-a-specified-model"></a>Option 1 : Analyse délimitée - Analyser seulement un modèle spécifié

La sortie est un tableau d’étiquettes, comme illustré dans l’exemple suivant :

```json
{  
  "result":[  
    {  
      "name":"golden retriever",
      "score":0.98
    },
    {  
      "name":"Labrador retriever",
      "score":0.78
    }
  ]
}
```

### <a name="option-2-enhanced-analysis---analyze-to-provide-additional-details-by-using-the-86-categories-taxonomy"></a>Option n°2 : Analyse élargie - Analyser pour fournir des détails supplémentaires en utilisant la taxonomie « 86-categories » (86 catégories)

Pour les modèles spécifiques au domaine qui utilisent l’option 2 (analyse élargie), le type de retour des catégories est étendu, comme illustré dans l’exemple suivant :

```json
{  
  "requestId":"87e44580-925a-49c8-b661-d1c54d1b83b5",
  "metadata":{  
    "width":640,
    "height":430,
    "format":"Jpeg"
  },
  "result":{  
    "celebrities":[  
      {  
        "name":"Richard Nixon",
        "faceRectangle":{  
          "left":107,
          "top":98,
          "width":165,
          "height":165
        },
        "confidence":0.9999827
      }
    ]
  }
}
```

Le champ « categories » est une liste d’une ou plusieurs catégories parmi les [86 catégories](../Category-Taxonomy.md) de la taxonomie d’origine. Les catégories qui se terminent par un trait de soulignement correspondent à cette catégorie et à ses enfants (par exemple, « people_ » et « people_group » pour le modèle « celebrities »).

Champ    | Type    | Contenu
------|------|------|
categories | `object`    | Objet de plus haut niveau.
categories[].name     | `string`    | Nom de la liste de la taxonomie de 86 catégories.
categories[].score    | `number`    | Score de confiance, compris entre 0 et 1.
categories[].detail     | `object?`      | (Facultatif) Objet de détail.

Si plusieurs catégories correspondent (par exemple, le classifieur de 86 catégories retourne un score à la fois pour « people_ » et pour « people_young » quand model=celebrities), les détails sont attachés à la correspondance du niveau le plus général (dans cet exemple, « people_ »).

## <a name="error-responses"></a>Réponses d’erreur

Ces erreurs sont identiques à celles de vision.analyze, avec en plus l’erreur NotSupportedModel (HTTP 400), qui peut être retournée dans les deux scénarios Option 1 et Option 2. Dans Option 2 (analyse élargie), si un des modèles spécifiés dans les détails n’est pas reconnu, l’API retourne une erreur NotSupportedModel, même si un ou plusieurs des modèles sont valides. Pour connaître les modèles pris en charge, vous pouvez appeler listModels.

## <a name="next-steps"></a>Étapes suivantes

Pour utiliser l’API REST, consultez les [Informations de référence sur l’API Vision par ordinateur](https://westus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-1-ga/operations/56f91f2e778daf14a499f21b).
