---
title: Rechercher dans le contenu du Stockage Blob Azure
titleSuffix: Azure Cognitive Search
description: Apprenez-en davantage sur l’extraction de texte de blobs Azure et la recherche en texte intégral dans un index de Recherche cognitive Azure.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 09/23/2020
ms.openlocfilehash: f61bf635cc61a2153a7bb016ef4b4711d7ba7391
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91355293"
---
# <a name="search-over-azure-blob-storage-content"></a>Rechercher dans le contenu du Stockage Blob Azure

La recherche dans les différents types de contenu enregistrés dans le Stockage Blob Azure peut constituer un problème difficile à résoudre. Cet article examine le flux de travail de base pour l’extraction de contenu et de métadonnées de blobs et leur envoi à un index de recherche dans le service Recherche cognitive Azure. L’index obtenu peut être interrogé à l’aide d’une recherche en texte intégral.

> [!NOTE]
> Vous êtes déjà familiarisé avec le flux de travail et la composition ? Votre prochaine étape est [Comment configurer un indexeur de blobs](search-howto-indexing-azure-blob-storage.md).

## <a name="what-it-means-to-add-full-text-search-to-blob-data"></a>Implications de l’ajout de la recherche en texte intégral aux données blob

Le service Recherche cognitive Azure est un service de recherche qui prend en charge les charges de travail d’indexation et de requête sur des index définis par l’utilisateur qui contiennent votre contenu pouvant faire l’objet d’une recherche à distance hébergé dans le cloud. Colocaliser votre contenu pouvant faire l’objet d’une recherche avec le moteur de requête est une nécessité pour offrir aux utilisateurs les performances qu’ils attendent eu égard au délai d’affichage des résultats des requêtes de recherche.

Le service Recherche cognitive s’intègre avec le service Stockage Blog Azure au niveau de la couche d’indexation. Le contenu de votre blob est ainsi importé sous forme de documents de recherche indexés dans des *index inversés* et d’autres structures de requête qui prennent en charge les requêtes de texte en forme libre et les expressions de filtre. Le contenu de votre blob étant indexé dans un index de recherche, vous pouvez utiliser la panoplie complète des fonctionnalités de requête du service Recherche cognitive Azure pour trouver des informations dans le contenu de votre blob.

Les entrées sont vos objets blob, dans un même conteneur, dans Stockage Blob Azure. Les objets blob peuvent correspondre à pratiquement tout type de données texte. Si vos blobs contiennent des images, vous pouvez ajouter l’[enrichissement par IA à l’indexation d’objets blob](search-blob-ai-integration.md) pour créer et extraire du texte des images.

La sortie est toujours un index Recherche cognitive Azure, utilisé pour la recherche, l’extraction et l’exploration rapides de texte dans les applications clientes. Au milieu se trouve l’architecture proprement dite du pipeline d’indexation. Le pipeline est basé sur la fonctionnalité d’*indexeur*, décrite plus loin dans cet article.

Une fois créé et rempli, l’index existe indépendamment de votre conteneur d’objets blob. Vous pouvez néanmoins réexécuter les opérations d’indexation de façon à actualiser l’index sur la base des documents modifiés. Les informations d’horodatage des différents objets blob servent à détecter les modifications. Vous pouvez opter pour une exécution planifiée ou une indexation à la demande en guise de mécanisme d’actualisation.

## <a name="required-resources"></a>Ressources nécessaires

Vous avez besoin des services Recherche cognitive Azure et Stockage Blob Azure. Dans Stockage Blob, vous avez besoin d’un conteneur qui fournit le contenu source.

Vous pouvez commencer directement dans votre page du portail des comptes Stockage. Dans la page de navigation de gauche, sous **Service Blob**, cliquez sur **Ajouter Recherche cognitive Azure** pour créer un nouveau service ou en sélectionner un existant. 

Une fois que vous avez ajouté Recherche cognitive Azure à votre compte de stockage, vous pouvez suivre le processus standard d’indexation de données blob. Nous vous recommandons d’utiliser l’Assistant **Importation des données** de Recherche cognitive Azure pour une mise en route simple ou d’appeler les API REST à l’aide d’un outil comme Postman. Ce tutoriel vous explique pas à pas comment appeler l’API REST dans Postman : [Indexer et rechercher des données semi-structurées (objets blob JSON) dans la Recherche cognitive Azure](search-semi-structured-data.md). 

## <a name="use-a-blob-indexer"></a>Utiliser un indexeur d’objets blob

Un *indexeur* est un sous-service qui reconnaît les sources de données dans Recherche cognitive. Avec sa logique interne, il échantillonne les données, lit les données de métadonnées, extrait les données et les sérialise dans des documents JSON à partir de formats natifs en vue d’une importation ultérieure. 

Dans Stockage Azure, les objets blob sont indexés à l’aide de l’[indexeur de stockage d’objets blob de Recherche cognitive Azure](search-howto-indexing-azure-blob-storage.md). Vous pouvez appeler cet indexeur à partir de l’Assistant **Importation des données**, d’une API REST ou du kit SDK .NET. Dans le code, vous pouvez utiliser cet indexeur en définissant le type et en fournissant des informations de connexion qui incluent un compte Stockage Azure associé à un conteneur d’objets blob. Vous pouvez créer un sous-ensemble de vos objets blob en créant un répertoire virtuel, que vous pouvez ensuite transmettre comme paramètre, ou en filtrant sur une extension de type de fichier.

Un indexeur effectue le « craquage de document » en ouvrant un objet blob pour en inspecter le contenu. Une fois connecté à la source de données, il s’agit de la première étape du pipeline. Pour les données blob, c’est à ce stade que les fichiers PDF, les documents Office et d’autres types de contenu sont détectés. Le craquage de document avec extraction de texte n’est pas facturé. Si vos objets blob contiennent des images, celles-ci sont ignorées si vous n’[ajoutez pas l’enrichissement par IA](search-blob-ai-integration.md). L’indexation standard s’applique uniquement au contenu texte.

L’indexeur d’objets blob est assorti de paramètres de configuration et prend en charge le suivi des modifications si les données sous-jacentes fournissent suffisamment d’informations. Vous trouverez des informations supplémentaires sur les fonctionnalités de base dans [Indexeur de stockage d’objets blob de Recherche cognitive Azure](search-howto-indexing-azure-blob-storage.md).

### <a name="supported-content-types"></a>Types de contenu pris en charge

En exécutant un indexeur d’objets blob sur un conteneur, vous pouvez extraire du texte et des métadonnées à partir des types de contenu suivants avec une seule requête :

[!INCLUDE [search-blob-data-sources](../../includes/search-blob-data-sources.md)]

### <a name="indexing-blob-metadata"></a>Indexation de métadonnées blob

Pour faciliter le tri dans les objets blob constitués de tout type de contenu, un scénario courant consiste à indexer les métadonnées personnalisées et les propriétés système pour chaque objet blob. De cette façon, les informations de tous les objets blob sont indexées indépendamment du type de document et stockées dans un index de votre service de recherche. Le nouvel index vous permet alors d’effectuer un tri, un filtrage et une facette dans l’ensemble du contenu du stockage Blob.

> [!NOTE]
> Les balises d’index d’objet blob sont indexées en mode natif par le service de stockage d’objets blob et exposées pour l’interrogation. Si les attributs clé/valeur de vos objets blob nécessitent des fonctionnalités d’indexation et de filtrage, les balises d’index d’objet blob doivent être exploitées à la place des métadonnées.
>
> Pour en savoir plus sur un index d’objets blob, consultez [Gérer et rechercher des données sur le Stockage Blob Azure avec un index d’objets blob](../storage/blobs/storage-manage-find-blobs.md).

### <a name="indexing-json-blobs"></a>Indexation d’objets JSON

Les indexeurs peuvent être configurés pour extraire le contenu structuré des objets blob qui contiennent des objets JSON. Un indexeur peut lire les objets blob JSON et analyser le contenu structuré dans les champs adaptés d’un document de recherche. Les indexeurs peuvent également extraire les objets blob contenant des objets JSON et mapper chaque élément avec un document de recherche différent. Vous pouvez définir un mode d’analyse pour affecter le type d’objet JSON créé par l’indexeur.

## <a name="search-blob-content-in-a-search-index"></a>Rechercher du contenu d’objet blob dans un index de recherche 

La sortie d’un indexeur est un index de recherche, qui permet une exploration interactive à l’aide de requêtes de texte libre et filtrées dans une application cliente. Pour une exploration et une vérification initiales de contenu, nous vous recommandons de commencer avec l’[Explorateur de recherche](search-explorer.md) sur le portail, qui vous permet d’examiner la structure du document. L’Explorateur de recherche vous permet d’utiliser une [syntaxe de requête simple](query-simple-syntax.md), une [syntaxe de requête complète](query-lucene-syntax.md) et une [syntaxe d’expression de filtre](query-odata-filter-orderby-syntax.md).

Une solution plus permanente consiste à regrouper les entrées de requête et à présenter la réponse sous forme de résultats de recherche dans une application cliente. Le tutoriel C# suivant explique comment créer une application de recherche : [Créer votre première application dans Recherche cognitive Azure](tutorial-csharp-create-first-app.md).

## <a name="next-steps"></a>Étapes suivantes

+ [Charger, télécharger et répertorier des blobs à l’aide du portail Azure (Stockage blob Azure)](../storage/blobs/storage-quickstart-blobs-portal.md)
+ [Configurer un indexeur blob (Recherche cognitive Azure)](search-howto-indexing-azure-blob-storage.md)