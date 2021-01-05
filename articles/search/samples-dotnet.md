---
title: Exemples relatifs à .NET
titleSuffix: Azure Cognitive Search
description: Trouvez des exemples de code C# de démonstration de Recherche cognitive Azure qui utilisent les bibliothèques de client .NET.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/17/2020
ms.openlocfilehash: ab6408621616a4be62631391456f73e90fced752
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498997"
---
# <a name="net-c-code-samples-for-azure-cognitive-search"></a>Exemples de code (C#) .NET pour Recherche cognitive Azure

Découvrez les exemples de code C# qui montrent les fonctionnalités de Recherche cognitive Azure. Les dépôts principaux sont les suivants :

| Référentiel | Description |
|------------|-------------|
| [azure-sdk-for-net/sdk/search/Azure.Search.Documents/samples/](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/search/Azure.Search.Documents/samples) | Exemples produits par l’équipe du SDK Azure qui sont fournis avec la bibliothèque de client Azure.Search.Documents dans le SDK. Vous pouvez également consulter des [tests unitaires](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/search/Azure.Search.Documents/tests) pour la bibliothèque de client pour voir comment différentes API sont appelées. |
| [Azure-Samples/azure-search-dotnet-samples](https://github.com/Azure-Samples/azure-search-dotnet-samples) | Exemples qui accompagnent les guides pratiques dans la documentation, notamment [Guide pratique pour utiliser la bibliothèque de client .NET](search-howto-dotnet-sdk.md).|
| [Azure-Samples/search-dotnet-getting-started](https://github.com/Azure-Samples/search-dotnet-getting-started) | Exemples qui accompagnent les guides de démarrage rapide et les tutoriels dans la documentation.|

> [!Tip]
> Essayez le [navigateur d’exemples](/samples/browse/?languages=csharp&products=azure-cognitive-search) pour rechercher des exemples de code Microsoft dans GitHub, filtrés par produit, par service et par langage.

## <a name="net-sdk-samples"></a>Exemples du kit SDK .NET

Le SDK Azure pour .NET comprend de nombreux exemples et un [fichier Lisez-moi](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/samples/README.md) qui décrit chaque exemple. Cette liste est fournie ci-dessous à titre indicatif.

| Exemples | Description |
|---------|-------------|
| [« Hello World », synchrone](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/samples/Sample01a_HelloWorld.md) | Montre comment créer un client, s’authentifier et gérer les erreurs en utilisant des méthodes synchrones.|
| [« Hello World », asynchrone](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/samples/Sample01b_HelloWorldAsync.md) | Montre comment créer un client, s’authentifier et gérer les erreurs en utilisant des méthodes asynchrones.  |
| [Opérations au niveau du service](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/samples/Sample02_Service.md) | Montre comment créer des index, des indexeurs, des sources de données, des ensembles de compétences et des mappages de synonymes. Cet exemple montre également comment obtenir des statistiques sur le service et comment interroger un index.  |
| [Opérations d’index](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/samples/Sample03_Index.md) | Montre comment effectuer une action sur un index existant, en l’occurrence obtenir le nombre de documents stockés dans l’index.  |
| [FieldBuilderIgnore](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/samples/Sample04_FieldBuilderIgnore.md) | Montre une technique permettant de travailler avec des types de données non pris en charge.  |
| [Indexation de documents (modèle push)](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/samples/Sample05_IndexingDocuments.md) | Indexation de modèle « push », où vous envoyez une charge utile JSON à un index sur un service.   |
| [Exemple de clé de chiffrement](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/samples/Sample06_EncryptedIndex.md) | Montre comment utiliser une clé de chiffrement gérée par le client pour ajouter une couche supplémentaire de protection sur du contenu sensible.  |

## <a name="documentation-samples"></a>Exemples de la documentation

Un article est associé aux exemples suivants dans la [documentation Recherche cognitive Azure](./index.yml).

| Exemples | Description |
|---------|-------------|
| [démarrage rapide](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/quickstart) | Code source pour [Démarrage rapide : Créer un index de recherche](search-get-started-dotnet.md).  |
| [DotNetHowTo](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowTo)  | [Guide pratique pour utiliser la bibliothèque de client .NET](search-howto-dotnet-sdk.md) |
| [DotNetHowToSynonyms](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowToSynonyms)  | Les listes de synonymes sont utilisées pour l’extension des requêtes, fournissant des termes externes à un index, qui peuvent être mis en correspondance. Cet exemple est inclus dans [Exemple : Ajouter des synonymes en C#](search-synonyms-tutorial-sdk.md). |
| [DotNetToIndexers](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowToIndexers) | Code source derrière des extraits de code liés à l’indexeur dans différents articles. Cet exemple montre comment configurer un indexeur qui a une planification, des mappages de champs et des paramètres.  |
| [DotNetHowToEncryptionUsingCMK](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowToEncryptionUsingCMK)  | Code source pour [Guide pratique pour configurer des clés gérées par le client pour le chiffrement des données](search-security-manage-encryption-keys.md) |
| [Créer votre première application en C#](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v11) |  Code source pour [Tutoriel : Créer votre première application de recherche](tutorial-csharp-create-first-app.md). Bien que la plupart des exemples soient des applications console, cet exemple MVC utilise une page web pour montrer l’exemple d’index Hotels avec la recherche de base, la pagination, l’autocomplétion, et les requêtes, facettes et filtres suggérés. |
| [multiple-data-sources](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/multiple-data-sources)  | Code source pour [Tutoriel : Indexer à partir de plusieurs sources de données](tutorial-multiple-data-sources.md). |
|  [optimize-data-indexing](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/optimize-data-indexing) | Code source pour [Tutoriel : Optimiser l’indexation avec l’API Push](tutorial-optimize-indexing-push-api.md).  |
| [tutorial-ai-enrichment](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/tutorial-ai-enrichment)  | Code source pour [Tutoriel : Contenu permettant la recherche, généré par IA à partir de blobs Azure avec le SDK .NET](cognitive-search-tutorial-blob-dotnet.md).  |

## <a name="standalone-samples-and-solutions"></a>Exemples et solutions de bout en bout

| Exemples | Description |
|---------|-------------|
| [azure-search-power-skills](https://github.com/Azure-Samples/azure-search-power-skills)  | Code source pour des compétences personnalisées consommables que vous pouvez incorporer dans vos propres solutions.  |
| [Accélérateur de solution d’exploration des connaissances](/samples/azure-samples/azure-search-knowledge-mining/azure-search-knowledge-mining/) | Inclut des modèles, des fichiers de support et des rapports analytiques pour vous aider à prototyper une solution d’exploration des connaissances de bout en bout.  |
| [Dépôt de l’application de recherche sur la Covid-19](https://github.com/liamca/covid19search) | Dépôt de code source pour l’[application de recherche sur la Covid-19](https://covid19search.azurewebsites.net/) basée sur Recherche cognitive |
| [JFK](https://github.com/Microsoft/AzureSearch_JFK_Files) | En savoir plus sur la [solution JFK](https://www.microsoft.com/ai/ai-lab-jfk-files). |