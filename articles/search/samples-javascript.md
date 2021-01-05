---
title: Exemples JavaScript
titleSuffix: Azure Cognitive Search
description: Recherchez des exemples de code JavaScript de démonstration pour Recherche cognitive Azure, qui utilisent le Kit de développement logiciel (SDK) Azure .NET pour JavaScript.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/17/2020
ms.openlocfilehash: 09a9ca2ad1b1f1e5578ecc5d35a85c81b32a5b1a
ms.sourcegitcommit: df66dff4e34a0b7780cba503bb141d6b72335a96
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96511744"
---
# <a name="javascript-code-samples-for-azure-cognitive-search"></a>Exemples de code JavaScript pour Recherche cognitive Azure

Découvrez les exemples de code JavaScript qui montrent les fonctions et la fonctionnalité de Recherche cognitive Azure. Les dépôts principaux sont les suivants :

| Référentiel | Description |
|------------|-------------|
| [azure-sdk-for-js/tree/master/sdk/search/search-documents](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents) | Exemples produits par l’équipe du SDK Azure qui sont fournis avec la bibliothèque de client Azure.Search.Documents dans le SDK. Vous pouvez également consulter des [tests unitaires](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/test) pour la bibliothèque de client pour voir comment différentes API sont appelées. |
| [Azure-Samples/azure-search-javascript-samples](https://github.com/Azure-Samples/azure-search-javascript-samples) | Exemples de code qui accompagnent les articles de guide pratique, sont [Démarrage rapide : Créer un index de recherche en JavaScript](search-get-started-javascript.md).|

> [!Tip]
> Essayez le [navigateur d’exemples](/samples/browse/?languages=javascript&products=azure-cognitive-search) pour rechercher des exemples de code Microsoft dans GitHub, filtrés par produit, par service et par langage.

## <a name="javascript-sdk-samples"></a>Exemples du Kit de développement logiciel (SDK) JavaScript

Le Kit de développement logiciel (SDK) Azure pour Java inclut de nombreux exemples et une [page de prise en main](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/search/azure-search-documents/README.md#getting-started) qui couvre l’installation du package, la configuration du client et la résolution des problèmes. La page décrit également les exemples de catégories suivants, répertoriés ici par souci de commodité.

| Exemples | Description |
|---------|-------------|
| [index](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/javascript/src/indexes) | Montre comment créer, mettre à jour, extraire, répertorier et supprimer des [index de recherche](search-what-is-an-index.md). Cet exemple de catégorie comprend également un exemple de statistique de service. |
| [dataSourceConnections (pour les indexeurs)](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/javascript/src/dataSourceConnections) | Montre comment créer, mettre à jour, obtenir, répertorier et supprimer des sources de données d’indexeur, nécessaires pour l’indexation basée sur un indexeur de [sources de données Azure prises en charge](search-indexer-overview.md#supported-data-sources). |
| [indexeurs](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/javascript/src/indexers) |  Montre comment créer, mettre à jour, obtenir, répertorier, réinitialiser et supprimer des [indexeurs](search-indexer-overview.md).|
| [skillSet](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/javascript/src/skillSets) |   Montre comment créer, mettre à jour, obtenir, répertorier et supprimer des [ensembles de compétences](cognitive-search-working-with-skillsets.md) qui sont des indexeurs attachés, et qui effectuent un enrichissement basé sur l’intelligence artificielle pendant l’indexation. |
| [synonymMaps](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/javascript/src/synonymMaps) | Montre comment créer, mettre à jour, obtenir, répertorier et supprimer des [cartes de synonymes](search-synonyms.md).  |
| [Requêtes](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/search/search-documents/samples/javascript/src/readonlyQuery.js) | Montre l’exécution d’une requête sur un index public en lecture seule hébergé par Microsoft.  |

## <a name="typescript-samples"></a>Exemples TypeScript

Le Kit de développement logiciel (SDK) fournit également des exemples de TypeScript répertoriés ici par commodité.

| Exemples | Description |
|---------|-------------|
| [index](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/typescript/src/indexes) | Montre comment créer, mettre à jour, extraire, répertorier et supprimer des [index de recherche](search-what-is-an-index.md). Cet exemple de catégorie comprend également un exemple de statistique de service. |
| [dataSourceConnections (pour les indexeurs)](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/typescript/src/dataSourceConnections) | Montre comment créer, mettre à jour, obtenir, répertorier et supprimer des sources de données d’indexeur, nécessaires pour l’indexation basée sur un indexeur de [sources de données Azure prises en charge](search-indexer-overview.md#supported-data-sources). |
| [indexeurs](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/typescript/src/indexers) |  Montre comment créer, mettre à jour, obtenir, répertorier, réinitialiser et supprimer des [indexeurs](search-indexer-overview.md).|
| [skillSet](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/typescript/src/skillSets) |   Montre comment créer, mettre à jour, obtenir, répertorier et supprimer des [ensembles de compétences](cognitive-search-working-with-skillsets.md) qui sont des indexeurs attachés, et qui effectuent un enrichissement basé sur l’intelligence artificielle pendant l’indexation. |
| [synonymMaps](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/typescript/src/synonymMaps) | Montre comment créer, mettre à jour, obtenir, répertorier et supprimer des [cartes de synonymes](search-synonyms.md).  |
| [Requêtes](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/search/search-documents/samples/typescript/src/readonlyQuery.ts) | Montre l’exécution d’une requête sur un index public en lecture seule hébergé par Microsoft.  |

## <a name="documentation-samples"></a>Exemples de la documentation

Un article est associé aux exemples suivants dans la [documentation sur Recherche cognitive Azure](./index.yml).

| Exemples | Description | 
|---------|-------------|
| [démarrage rapide](https://github.com/Azure-Samples/azure-search-javascript-samples/tree/master/quickstart/v11) | Code source pour [Démarrage rapide : Créer un index de recherche en JavaScript](search-get-started-javascript.md).  |

## <a name="standalone-samples"></a>Exemples autonomes

| Exemples | Description |
|---------|-------------|
| [azure-search-react-template](https://github.com/dereklegenzoff/azure-search-react-template) | Modèle React pour Recherche cognitive Azure (github.com) |