---
title: Rechercher sur des blobs de texte brut
titleSuffix: Azure Cognitive Search
description: Configurez un indexeur de recherche pour extraire du texte brut de blobs Azure à des fins de recherche en texte intégral dans le service Recherche cognitive Azure.
manager: nitinme
author: mgottein
ms.author: magottei
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 09/25/2020
ms.openlocfilehash: 417bdacc3ce8b619d5ec9618e6060ac071882471
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91533923"
---
# <a name="how-to-index-plain-text-blobs-in-azure-cognitive-search"></a>Comment indexer des objets en texte brut dans le service Recherche cognitive Azure

Lors de l’utilisation d’un [indexeur de blobs](search-howto-indexing-azure-blob-storage.md) pour extraire du texte pouvant faire l’objet d’une recherche en texte intégral, vous pouvez appeler différents modes d’analyse pour obtenir de meilleurs résultats d’indexation. Par défaut, l’indexeur analyse les blobs de texte délimité comme un bloc de texte unique. Cependant, si tous vos blobs contiennent du texte brut dans le même encodage, vous pouvez améliorer considérablement les performances d’indexation à l’aide du **mode d’analyse de texte**.

## <a name="set-up-plain-text-indexing"></a>Configurer l’indexation de texte brut

Pour indexer des blobs de texte brut, créez ou mettez à jour une définition d’indexeur avec la propriété de configuration `parsingMode` pour `text` sur une demande [Créer un indexeur](/rest/api/searchservice/create-indexer) :

```http
    PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
      ... other parts of indexer definition
      "parameters" : { "configuration" : { "parsingMode" : "text" } }
    }
```

Par défaut, le `UTF-8` encodage est possible. Pour spécifier un encodage différent, utilisez la `encoding` propriété de configuration : 

```http
    {
      ... other parts of indexer definition
      "parameters" : { "configuration" : { "parsingMode" : "text", "encoding" : "windows-1252" } }
    }
```

## <a name="request-example"></a>Exemple de requête

Les modes d’analyse sont spécifiés dans la définition de l’indexeur.

```http
    POST https://[service name].search.windows.net/indexers?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
      "name" : "my-plaintext-indexer",
      "dataSourceName" : "my-blob-datasource",
      "targetIndexName" : "my-target-index",
      "parameters" : { "configuration" : { "parsingMode" : "delimitedText", "delimitedTextHeaders" : "id,datePublished,tags" } }
    }
```

## <a name="help-us-make-azure-cognitive-search-better"></a>Aidez-nous à améliorer Recherche cognitive Azure

Si vous avez des suggestions de fonctionnalités ou des idées d’amélioration, faites-le-nous savoir [UserVoice](https://feedback.azure.com/forums/263029-azure-search/). Si vous avez besoin d’aide pour utiliser la fonctionnalité existante, publiez votre question sur [Stack Overflow](https://stackoverflow.microsoft.com/questions/tagged/18870).

## <a name="next-steps"></a>Étapes suivantes

* [Indexeurs dans Recherche cognitive Azure](search-indexer-overview.md)
* [Comment configurer un indexeur de blobs](search-howto-indexing-azure-blob-storage.md)
* [Vue d’ensemble de l’indexation de blobs](search-blob-storage-integration.md)