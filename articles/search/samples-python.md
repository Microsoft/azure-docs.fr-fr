---
title: Exemples Python
titleSuffix: Azure Cognitive Search
description: Recherchez des exemples de code Python de démonstration pour Recherche cognitive Azure, qui utilisent le Kit de développement logiciel (SDK) Azure .NET pour Python ou REST.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/17/2020
ms.openlocfilehash: 6e0f3d318cc462b03151d5a4935ae318df46e2c5
ms.sourcegitcommit: df66dff4e34a0b7780cba503bb141d6b72335a96
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96510554"
---
# <a name="python-code-samples-for-azure-cognitive-search"></a>Exemples de code Python pour Recherche cognitive Azure

Découvrez les exemples de code Python qui montrent les fonctionnalités de Recherche cognitive Azure. Les dépôts principaux sont les suivants :

| Référentiel | Description |
|------------|-------------|
| [azure-sdk-for-python/tree/master/sdk/search/azure-search-documents/samples/](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/search/azure-search-documents/samples) | Exemples produits par l’équipe du SDK Azure qui sont fournis avec la bibliothèque de client Azure.Search.Documents dans le SDK. Vous pouvez également consulter des [tests unitaires](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/search/azure-search-documents/tests) pour la bibliothèque de client pour voir comment différentes API sont appelées. |
| [Azure-Samples/azure-search-python-samples](https://github.com/Azure-Samples/azure-search-python-samples) | Exemples de code qui accompagnent les articles de guide pratique, sont [Démarrage rapide : Créer un index de recherche en Python](search-get-started-python.md).|

> [!Tip]
> Essayez le [navigateur d’exemples](/samples/browse/?languages=python&products=azure-cognitive-search) pour rechercher des exemples de code Microsoft dans GitHub, filtrés par produit, par service et par langage.

## <a name="python-sdk-samples"></a>Exemples du kit SDK Python

Le Kit de développement logiciel (SDK) Azure pour Python inclut de nombreux exemples et une [page de prise en main](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/search/azure-search-documents/samples) qui couvre les prérequis et l’installation du package. La page contient également des liens vers les exemples suivants, répertoriés ici par souci de commodité.

| Exemples | Description |
|---------|-------------|
| [Authentifier](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/search/azure-search-documents/samples/sample_authentication.py) | Montre comment configurer un client et s’authentifier auprès du service. | 
| [Opérations de création, de lecture, de mise à jour et de suppression d’index](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/search/azure-search-documents/samples/sample_index_crud_operations.py) | Montre comment créer, mettre à jour, extraire, répertorier et supprimer des [index de recherche](search-what-is-an-index.md). |
| [Opérations de création, de lecture, de mise à jour et de suppression d’un indexeur](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/search/azure-search-documents/samples/sample_indexers_operations.py) | Montre comment créer, mettre à jour, obtenir, répertorier, réinitialiser et supprimer des [indexeurs](search-indexer-overview.md). |
| [Rechercher des sources de données d’indexeur](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/search/azure-search-documents/samples/sample_indexer_datasource_skillset.py) | Montre comment créer, mettre à jour, obtenir, répertorier et supprimer des sources de données d’indexeur, nécessaires pour l’indexation basée sur un indexeur de [sources de données Azure prises en charge](search-indexer-overview.md#supported-data-sources). |
| [Synonymes](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/search/azure-search-documents/samples/sample_synonym_map_operations.py) | Montre comment créer, mettre à jour, obtenir, répertorier et supprimer des [cartes de synonymes](search-synonyms.md).  |
| [Chargement de documents](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/search/azure-search-documents/samples/sample_crud_operations.py) | Montre comment charger ou fusionner des documents dans un index dans le cadre d’une opération d’[importation de données](search-what-is-data-import.md). |
| [Requête simple](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/search/azure-search-documents/samples/sample_simple_query.py) | Montre comment configurer une [requête de base](search-query-overview.md). |
| [Requête de filtre](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/search/azure-search-documents/samples/sample_filter_query.py) | Montre comment configurer une [expression de filtre](search-filters.md). |
| [Requête de facette](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/search/azure-search-documents/samples/sample_facet_query.py) | Montre comment utiliser des [facettes](search-filters-facets.md). |

## <a name="documentation-samples"></a>Exemples de la documentation

Un article est associé aux exemples suivants dans la [documentation sur Recherche cognitive Azure](./index.yml).

| Exemples | Description | 
|---------|-------------|
| [démarrage rapide](https://github.com/Azure-Samples/azure-search-python-samples/tree/master/Quickstart) | Code source pour [Démarrage rapide : Créer un index de recherche en Python](search-get-started-python.md).  |
| [tutorial-ai-enrichment](https://github.com/Azure-Samples/azure-search-python-samples/tree/master/Tutorial-AI-Enrichment)  | Code source pour [Tutoriel : Utiliser Python et l’IA pour générer du contenu pouvant faire l’objet de recherches à partir de blobs Azure](cognitive-search-tutorial-blob-python.md).  |
| [AzureML-Custom-Skill](https://github.com/Azure-Samples/azure-search-python-samples/tree/master/AzureML-Custom-Skill)  | Code source pour [Exemple : Créer une compétence personnalisée avec Python](cognitive-search-custom-skill-python.md).  |