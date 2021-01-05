---
title: Effectuer des recherches sur les objets blob CSV
titleSuffix: Azure Cognitive Search
description: Extrayez et importez un fichier CSV à partir de Stockage Blob Azure en utilisant le mode d’analyse delimitedText.
manager: nitinme
author: mgottein
ms.author: magottei
ms.devlang: rest-api
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 07/11/2020
ms.openlocfilehash: f9c01e8e31e78c277a7a3ec1e5d8d0c32b58f8bc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91403651"
---
# <a name="how-to-index-csv-blobs-using-delimitedtext-parsing-mode-and-blob-indexers-in-azure-cognitive-search"></a>Comment indexer des objets blob CSV en utilisant le mode d’analyse delimitedText et des indexeurs d’objets blob dans Recherche cognitive Azure

Par défaut, l’[indexeur d’objets blob Recherche cognitive Azure](search-howto-indexing-azure-blob-storage.md) analyse les objets blob de texte délimité comme un bloc de texte unique. Toutefois, avec des objets blob contenant des données CSV, vous souhaitez généralement traiter chaque ligne dans l’objet blob comme un document distinct. Par exemple, vous pouvez analyser le texte délimité suivant dans deux documents contenant chacun les champs « id », « datePublished » et « tags » : 

```text
id, datePublished, tags
1, 2016-01-12, "azure-search,azure,cloud"
2, 2016-07-07, "cloud,mobile"
```

Cet article explique comment analyser les objets blob CSV avec un indexeur d’objets blob Recherche cognitive Azure en définissant le mode d’analyse `delimitedText`. 

> [!NOTE]
> Suivez les recommandations de configuration de l’indexeur dans [Indexation un-à-plusieurs](search-howto-index-one-to-many-blobs.md) pour générer plusieurs documents de recherche à partir d’un objet Blob Azure.

## <a name="setting-up-csv-indexing"></a>Configuration de l’indexation CSV
Pour indexer des objets blob CSV, créez ou mettez à jour une définition d’indexeur avec le mode d’analyse `delimitedText` sur une demande [Créer un indexeur](/rest/api/searchservice/create-indexer) :

```http
    {
      "name" : "my-csv-indexer",
      ... other indexer properties
      "parameters" : { "configuration" : { "parsingMode" : "delimitedText", "firstLineContainsHeaders" : true } }
    }
```

`firstLineContainsHeaders` Indique que la première ligne (non vide) de chaque objet blob contient des en-têtes.
Si les objets blob ne contiennent pas de ligne d’en-tête initiale, les en-têtes doivent être spécifiés dans la configuration de l’indexeur : 

```http
"parameters" : { "configuration" : { "parsingMode" : "delimitedText", "delimitedTextHeaders" : "id,datePublished,tags" } } 
```

Vous pouvez personnaliser le caractère délimiteur à l’aide du paramètre de configuration `delimitedTextDelimiter`. Par exemple :

```http
"parameters" : { "configuration" : { "parsingMode" : "delimitedText", "delimitedTextDelimiter" : "|" } }
```

> [!NOTE]
> Actuellement, seul le format UTF-8 est pris en charge. Si vous devez prendre en charge d’autres encodages, faites-le-nous savoir sur [UserVoice](https://feedback.azure.com/forums/263029-azure-search).

> [!IMPORTANT]
> Lorsque vous utilisez le mode d’analyse de texte délimité, Recherche cognitive Azure suppose que tous les objets blob dans votre source de données seront de type CSV. Si vous devez prendre en charge une combinaison d’objets blob CSV et autres dans la même source de données, faites-le nous savoir sur [UserVoice](https://feedback.azure.com/forums/263029-azure-search).
> 
> 

## <a name="request-examples"></a>Exemples de requête
En résumé, voici des exemples complets de charges utiles. 

Source de données : 

```http
    POST https://[service name].search.windows.net/datasources?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
        "name" : "my-blob-datasource",
        "type" : "azureblob",
        "credentials" : { "connectionString" : "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>;" },
        "container" : { "name" : "my-container", "query" : "<optional, my-folder>" }
    }   
```

Indexeur :

```http
    POST https://[service name].search.windows.net/indexers?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
      "name" : "my-csv-indexer",
      "dataSourceName" : "my-blob-datasource",
      "targetIndexName" : "my-target-index",
      "parameters" : { "configuration" : { "parsingMode" : "delimitedText", "delimitedTextHeaders" : "id,datePublished,tags" } }
    }
```

## <a name="help-us-make-azure-cognitive-search-better"></a>Aidez-nous à améliorer Recherche cognitive Azure
Si vous avez des suggestions de fonctionnalités ou des idées d’amélioration, faites-le-nous savoir [UserVoice](https://feedback.azure.com/forums/263029-azure-search/). Si vous avez besoin d’aide pour utiliser la fonctionnalité existante, publiez votre question sur [Stack Overflow](https://stackoverflow.microsoft.com/questions/tagged/18870).