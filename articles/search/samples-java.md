---
title: Exemples relatifs à Java
titleSuffix: Azure Cognitive Search
description: Recherchez des exemples de code Java de démonstration pour Recherche cognitive Azure, qui utilisent le Kit de développement logiciel (SDK) Azure .NET pour Java.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/17/2020
ms.openlocfilehash: 10dff18f7b9db7273fcd6ec92bcca5970bb83b08
ms.sourcegitcommit: df66dff4e34a0b7780cba503bb141d6b72335a96
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96510367"
---
# <a name="java-code-samples-for-azure-cognitive-search"></a>Exemples de code Java pour Recherche cognitive Azure

Découvrez les exemples de code Java qui montrent les fonctionnalités de Recherche cognitive Azure. Les dépôts principaux sont les suivants :

| Référentiel | Description |
|------------|-------------|
| [azure-sdk-for-java/tree/master/sdk/search/azure-search-documents/src/samples](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/search/azure-search-documents/src/samples) | Exemples produits par l’équipe du SDK Azure qui sont fournis avec la bibliothèque de client Azure.Search.Documents dans le SDK. Vous pouvez également consulter des [tests unitaires](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/search/azure-search-documents/src/test) pour la bibliothèque de client pour voir comment différentes API sont appelées. |
| [Azure-Samples/azure-search-java-samples](https://github.com/Azure-Samples/azure-search-java-samples) | Exemples de code qui accompagnent les articles de guide pratique. **Les exemples de ce référentiel n’ont pas encore été mis à jour pour une utilisation avec le Kit de développement logiciel (SDK) Azure pour Java**. Actuellement, ces exemples appellent les API REST en code Java.|

> [!Tip]
> Essayez le [navigateur d’exemples](/samples/browse/?languages=java&products=azure-cognitive-search) pour rechercher des exemples de code Microsoft dans GitHub, filtrés par produit, par service et par langage.

## <a name="java-sdk-samples"></a>Exemples du kit SDK Java

Le Kit de développement logiciel (SDK) Azure pour Java inclut de nombreux exemples et une [page de prise en main](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/search/azure-search-documents/src/samples) qui couvre l’installation du package. La page répertorie également un vaste éventail d’exemples. Plusieurs des opérations les plus courantes sont répertoriées ci-dessous pour plus de commodité.

| Exemples | Description |
|---------|-------------|
| [Création d’index de recherche](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/search/azure-search-documents/src/samples/java/com/azure/search/documents/indexes/CreateIndexExample.java) | Montre comment créer des [index de recherche](search-what-is-an-index.md). |
| [Création de synonyme](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/search/azure-search-documents/src/samples/java/com/azure/search/documents/SynonymMapsCreateExample.java) | Montre comment créer des [cartes de synonymes](search-synonyms.md).  |
| [Créer un indexeur de recherche](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/search/azure-search-documents/src/samples/java/com/azure/search/documents/indexes/CreateIndexerExample.java) | Montre comment créer des [indexeurs](search-indexer-overview.md). |
| [Création de source de données d’indexeur de recherche](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/search/azure-search-documents/src/samples/java/com/azure/search/documents/indexes/DataSourceExample.java) | Montre comment créer des sources de données d’indexeur requises pour l’indexation basée sur un indexeur de [sources de données Azure prises en charge](search-indexer-overview.md#supported-data-sources). |
| [Création d’ensemble de compétences](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/search/azure-search-documents/src/samples/java/com/azure/search/documents/indexes/CreateSkillsetExample.java) |  Montre comment créer des [ensembles de compétences](cognitive-search-working-with-skillsets.md) qui sont des indexeurs attachés, et qui effectuent un enrichissement basé sur l’intelligence artificielle pendant l’indexation. |
| [Chargement de documents](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/search/azure-search-documents/src/samples/java/com/azure/search/documents/IndexContentManagementExample.java) | Montre comment charger ou fusionner des documents dans un index dans le cadre d’une opération d’[importation de données](search-what-is-data-import.md). |
| [Syntaxe de requête](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/search/azure-search-documents/src/samples/java/com/azure/search/documents/SearchAsyncWithFullyTypedDocumentsExample.java) | Montre comment configurer une [requête de base](search-query-overview.md). |

## <a name="documentation-samples"></a>Exemples de la documentation

Un article est associé aux exemples suivants dans la [documentation sur Recherche cognitive Azure](./index.yml).

| Exemples | Description | 
|---------|-------------|
| [démarrage rapide](https://github.com/Azure-Samples/azure-search-java-samples/tree/master/quickstart) | Code source pour [Démarrage rapide : Créer un index de recherche en Java](search-get-started-java.md). Cet exemple appelle les API REST. |
| [search-java-indexer-demo](https://github.com/Azure-Samples/azure-search-java-samples/tree/master/search-java-indexer-demo) | Montre un indexeur Azure Cosmos DB en Java. Cet exemple appelle les API REST. |