---
title: Importation et ingestion des données dans les index de recherche
titleSuffix: Azure Cognitive Search
description: Remplissez et chargez des données dans un index dans Recherche cognitive Azure à partir de sources de données externes.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/05/2020
ms.openlocfilehash: b57d55e91918ba612ad42acd5e6059ae0dbd0090
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93422448"
---
# <a name="data-import-overview---azure-cognitive-search"></a>Vue d’ensemble de l’importation des données - Recherche cognitive Azure

Dans Recherche cognitive Azure, les requêtes s’exécutent sur le contenu chargé et enregistré dans un [index de recherche](search-what-is-an-index.md). Cet article examine les deux méthodes de base pour remplir un index : *envoyer* (push) les données dans l’index par programme ou pointer un [indexeur Recherche cognitive Azure](search-indexer-overview.md) vers une source de données prise en charge pour *extraire* les données.

Ces deux approches ont pour objectif de charger des données depuis une source de données externe vers un index Recherche cognitive Azure. Recherche cognitive Azure vous permet de créer un index vide, mais ce dernier ne pourra être interrogé qu’après envoi (push) ou extraction des données.

> [!NOTE]
> Si l’[enrichissement en IA](cognitive-search-concept-intro.md) est une condition de la solution, vous devez utiliser le modèle d’extraction (indexeurs) pour charger un index. Le traitement externe est pris en charge uniquement par le biais de compétences liées à un indexeur.

## <a name="pushing-data-to-an-index"></a>Envoyer des données à un index

Le modèle d’envoi (push), utilisé pour envoyer vos données à Recherche cognitive Azure par programme, constitue l’approche la plus flexible. Tout d’abord, il n’y a pas de restrictions sur le type de source de données. Tout jeu de données composé de documents JSON peut être envoyé (push) à un index Recherche cognitive Azure, en supposant que chaque document dans le jeu de données possède des champs mappés sur des champs définis dans votre schéma d’index. En second lieu, il n’y a aucune restriction sur la fréquence d’exécution. Vous pouvez transmettre des modifications à un index aussi souvent que vous le souhaitez. Pour les applications ayant des exigences à très faible latence (par exemple, si vous devez synchroniser les opérations de recherche avec les bases de données d’inventaire dynamiques), le modèle d’émission est la seule option.

Cette approche est plus flexible que le modèle d’extraction, car vous pouvez charger des documents individuellement ou par lots (jusqu’à 1 000 par lot ou 16 Mo, quelle que soit la limite atteinte en premier). Le modèle d’envoi (push) vous permet également de charger des documents dans Recherche cognitive Azure indépendamment de l’emplacement des données.

### <a name="how-to-push-data-to-an-azure-cognitive-search-index"></a>Comment envoyer (push) des données à un index Recherche cognitive Azure

Vous pouvez utiliser les API suivantes pour charger un ou plusieurs documents dans un index :

+ [Ajout, mise à jour ou suppression de documents (API REST)](/rest/api/searchservice/AddUpdate-or-Delete-Documents)
+ [Classe IndexDocumentsAction](/dotnet/api/azure.search.documents.models.indexdocumentsaction) ou [classe IndexDocumentsBatch](/dotnet/api/azure.search.documents.models.indexdocumentsbatch) 

Il n’existe actuellement aucune prise en charge de l’outil de diffusion de données via le portail.

Pour une présentation de chaque méthode, consultez [Guide de démarrage rapide : Créer un index Recherche cognitive Azure à l’aide de PowerShell](./search-get-started-powershell.md) ou [Guide de démarrage rapide C# : Créer un index Recherche cognitive Azure à l’aide du Kit de développement logiciel (SDK) .NET](search-get-started-dotnet.md).

<a name="indexing-actions"></a>

### <a name="indexing-actions-upload-merge-mergeorupload-delete"></a>Actions d’indexation : upload, merge, mergeOrUpload, supprimer

Vous pouvez contrôler le type d’action d’indexation par document, en spécifiant si le document doit être chargé en intégralité, fusionné avec du contenu de document existant ou supprimé.

Dans l’API REST, émettez des requêtes HTTP POST avec un corps de requête JSON à l’URL de point de terminaison de votre index Recherche cognitive Azure. Chaque objet JSON du tableau « valeur » contient la clé du document et spécifie qu’une action d’indexation doit ajouter, mettre à jour ou supprimer le contenu d’un document. Pour un exemple de code, consultez [Charger des documents](search-get-started-dotnet.md#load-documents).

Dans le kit de développement logiciel (SDK) .NET, empaquetez vos données dans un objet `IndexBatch`. Un objet `IndexBatch` encapsule une collection d’objets `IndexAction`, chacun d’entre eux contenant un document et une propriété qui indique à Recherche cognitive Azure les actions à effectuer sur ce document. Pour obtenir un exemple de code, consultez le [guide de démarrage rapide C#](search-get-started-dotnet.md).


| @search.action | Description | Champs requis pour chaque document | Notes |
| -------------- | ----------- | ---------------------------------- | ----- |
| `upload` |Une action `upload` est similaire à celle d’un « upsert », où le document est inséré s’il est nouveau et mis à jour/remplacé s’il existe déjà. |une clé, ainsi que tout autre champ que vous souhaitez définir |Lors de la mise à jour ou du remplacement d’un document existant, un champ qui n’est pas spécifié dans la requête sera défini sur la valeur `null`, y compris lorsque le champ a été précédemment défini sur une valeur non null. |
| `merge` |Met à jour un document existant avec les champs spécifiés. Si le document n’existe pas dans l’index, la fusion échoue. |une clé, ainsi que tout autre champ que vous souhaitez définir |N'importe quel champ que vous spécifiez dans une fusion remplace le champ existant dans le document. Dans le kit de développement logiciel (SDK), cela inclut les champs de type `DataType.Collection(DataType.String)`. Dans l'API REST, cela inclut les champs de type `Collection(Edm.String)`. Par exemple, si le document contient un champ `tags` avec la valeur `["budget"]` et que vous exécutez une fusion avec la valeur `["economy", "pool"]` pour le champ `tags`, la valeur finale du champ `tags` sera `["economy", "pool"]`, et non `["budget", "economy", "pool"]`. |
| `mergeOrUpload` |Cette action est similaire à celle d’une action `merge` s’il existe déjà dans l’index un document comportant la clé spécifiée. Dans le cas contraire, elle exécutera une action `upload` avec un nouveau document. |une clé, ainsi que tout autre champ que vous souhaitez définir |- |
| `delete` |Cette action supprime de l’index le document spécifié. |clé uniquement |Tous les champs que vous spécifiez en dehors du champ de clé sont ignorés. Si vous souhaitez supprimer un champ individuel dans un document, utilisez plutôt `merge` et définissez simplement le champ de manière explicite sur la valeur null. |

### <a name="formulate-your-query"></a>Formuler votre requête

Deux méthodes permettent d’effectuer une [recherche dans un index à l’aide de l’API REST](/rest/api/searchservice/Search-Documents). L’une consiste à émettre une requête HTTP POST, dans laquelle vos paramètres de requête sont définis dans un objet JSON contenu dans le corps de la requête. L’autre consiste à émettre une requête HTTP GET, dans laquelle vos paramètres de requête seront définis à l’intérieur de l’URL de requête. Notez que les limites en matière de taille des paramètres de requête sont [plus souples](/rest/api/searchservice/Search-Documents) pour la méthode POST que pour la méthode GET. Pour cette raison, nous vous recommandons d’utiliser POST, à moins que la situation justifie l’utilisation de GET.

Pour POST et GET, vous devez fournir votre *nom de service* , *nom d’index* et une *version de l’API* dans l’URL de la demande. 

Pour la méthode GET, vous renseignez les paramètres de requête au niveau de la *chaîne de requête* à la fin de l’URL. Voici le format URL à utiliser :

```http
    https://[service name].search.windows.net/indexes/[index name]/docs?[query string]&api-version=2019-05-06
```

La méthode POST suit un format identique, mais seule `api-version` dans les paramètres de chaîne de requête.

## <a name="pulling-data-into-an-index"></a>Extraction de données dans un index

Le modèle d’extraction analyse une source de données prise en charge et charge automatiquement les données dans votre index. Dans Recherche cognitive Azure, cette fonctionnalité est implémentée via des *indexeurs* , actuellement disponibles pour ces plateformes :

+ [Stockage Blob](search-howto-indexing-azure-blob-storage.md)
+ [Stockage Table](search-howto-indexing-azure-tables.md)
+ [Azure Cosmos DB](search-howto-index-cosmosdb.md)
+ [Azure SQL Database, SQL Managed Instance et SQL Server sur des machines virtuelles Azure](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md)

Les indexeurs connectent un index à une source de données (généralement une table, une vue ou une structure équivalente) et mappent les champs source aux champs équivalents de l’index. Pendant l’exécution, l’ensemble de lignes est automatiquement transformé en JSON et chargé dans l’index spécifié. Tous les indexeurs prennent en charge la planification de sorte que vous puissiez spécifier la fréquence à laquelle les données sont à actualiser. La plupart des indexeurs fournissent le suivi des modifications si la source de données le prend en charge. En suivant les modifications et les suppressions effectuées dans les documents existants, et en reconnaissant les nouveaux documents, les indexeurs suppriment la nécessité de gérer activement les données de votre index.

### <a name="how-to-pull-data-into-an-azure-cognitive-search-index"></a>Comment extraire des données dans un index Recherche cognitive Azure

La fonctionnalité de l’indexeur est exposée dans le [Portail Azure](search-import-data-portal.md), ainsi que dans [l’API REST](/rest/api/searchservice/Indexer-operations) et le [Kit de développement logiciel (SDK) .NET](/dotnet/api/azure.search.documents.indexes.searchindexerclient).

Grâce au portail, Recherche cognitive Azure peut générer un schéma d’index par défaut pour vous en lisant les métadonnées du jeu de données source. Vous pouvez modifier l’index généré jusqu’à ce que l’index soit traité, après quoi les seules modifications de schéma autorisées sont celles qui ne nécessitent pas la réindexation. Si les modifications que vous souhaitez apporter impactent directement le schéma, vous devez reconstruire l’index. 

## <a name="verify-data-import-with-search-explorer"></a>Vérifier l’importation de données avec l’Explorateur de recherche

Un moyen rapide d’effectuer une vérification préliminaire sur le téléchargement de document consiste à utiliser **l’Explorateur de recherche** dans le portail. L’Explorateur vous permet d’interroger un index sans avoir à écrire du code. L’expérience de recherche est basée sur les paramètres par défaut, tels que la [syntaxe simple](/rest/api/searchservice/simple-query-syntax-in-azure-search) et le [paramètre de requête searchMode](/rest/api/searchservice/search-documents). Les résultats sont retournés au format JSON afin que vous puissiez inspecter le document dans son intégralité.

> [!TIP]
> De nombreux [exemples de code Recherche cognitive Azure](https://github.com/Azure-Samples/?utf8=%E2%9C%93&query=search) incluent des jeux de données incorporés ou disponibles rapidement, offrant ainsi une prise en main simplifiée. Le portail fournit également un exemple d’indexeur et de source de données composée d’un petit jeu de données immobilières (nommé « realestate-us-sample »). Lorsque vous exécutez l’indexeur préconfiguré sur l’exemple de source de données, un index est créé et chargé avec des documents qui peuvent ensuite être interrogés dans l’Explorateur de recherche ou par le code que vous écrivez.

## <a name="see-also"></a>Voir aussi

+ [Présentation de l’indexeur](search-indexer-overview.md)
+ [Procédure pas à pas dans le portail : créer, charger, interroger un index](search-get-started-portal.md)