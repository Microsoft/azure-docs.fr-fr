---
title: Effectuer des recherches sur les objets blob JSON
titleSuffix: Azure Cognitive Search
description: Analyser les objets blob Azure JSON pour le contenu de texte à l’aide de l’indexeur d’objets blob Recherche cognitive Azure. Les indexeurs automatisent l’ingestion des données pour certaines sources de données, comme le Stockage Blob Azure.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.devlang: rest-api
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 09/25/2020
ms.openlocfilehash: a3c44d667b6baaf16e109dfb88c22c16a1ea2ce1
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94697201"
---
# <a name="how-to-index-json-blobs-using-a-blob-indexer-in-azure-cognitive-search"></a>Guide pratique pour indexer des objets blob JSON avec un indexeur d’objets blob dans Recherche cognitive Azure

Cet article explique comment configurer un [indexeur](search-indexer-overview.md) d’objets Blob Recherche cognitive Azure pour extraire le contenu structuré de documents JSON dans Stockage Blob Azure et pouvoir le rechercher dans Recherche cognitive Azure. Ce flux de travail crée un index Recherche cognitive Azure et le charge avec le texte existant extrait d’objets Blob JSON. 

Vous pouvez utiliser le [portail](#json-indexer-portal), l’[API REST](#json-indexer-rest) ou le [SDK .NET](#json-indexer-dotnet) pour indexer du contenu JSON. Dans toutes les approches, les documents JSON sont généralement situés dans un conteneur d’objets Blob, dans un compte de Stockage Azure. Pour obtenir des conseils sur l’envoi (push) de documents JSON depuis d’autres plateformes qu’Azure, consultez [Importation de données dans Recherche cognitive Azure](search-what-is-data-import.md).

Les objets Blob JSON dans Stockage Blob Azure se composent généralement d’un seul document JSON (le mode d’analyse est `json`) ou d’une collection d’entités. Pour les collections JSON, l’objet blob peut avoir un **tableau** d’éléments JSON bien formés (le mode d’analyse est `jsonArray`). Les objets Blob peuvent également être composés de plusieurs entités JSON individuelles séparées par un saut de ligne (le mode d’analyse est `jsonLines`). Le paramètre **parsingMode** sur la demande détermine les structures de sortie.

> [!NOTE]
> Pour plus d’informations sur l’indexation de plusieurs documents de recherche à partir d’un seul objet blob, consultez [indexation un-à-plusieurs](search-howto-index-one-to-many-blobs.md).

<a name="json-indexer-portal"></a>

## <a name="use-the-portal"></a>Utiliser le portail

La méthode la plus simple pour l’indexation de documents JSON consiste à utiliser un Assistant dans le [portail Azure](https://portal.azure.com/). En analysant les métadonnées dans le conteneur d’objets blob Azure, l’Assistant [**Importation de données**](search-import-data-portal.md) peut créer un index par défaut, mapper des champs sources aux champs d’index cibles et charger l’index en une seule opération. Selon la taille et la complexité de la source de données, vous pouvez obtenir un index de recherche en texte intégral opérationnel en quelques minutes.

Nous vous recommandons d’utiliser la même région ou le même emplacement pour Recherche cognitive Azure et Stockage Azure pour obtenir une latence plus faible et pour éviter les frais de bande passante.

### <a name="1---prepare-source-data"></a>1 - Préparez les données sources

[Connectez-vous au portail Azure](https://portal.azure.com/) et [créez un de conteneur d’objets blob](../storage/blobs/storage-quickstart-blobs-portal.md) pour accueillir vos données. Le niveau d’accès public peut être défini sur l’une de ses valeurs valides.

Vous aurez besoin du nom du compte de stockage, du nom du conteneur et d’une clé d’accès pour récupérer vos données dans l’Assistant **Importation de données**.

### <a name="2---start-import-data-wizard"></a>2 - Démarrez l’Assistant Importation de données

Dans la page Vue d’ensemble de votre service de recherche, vous pouvez [démarrer l’Assistant](search-import-data-portal.md) à partir de la barre de commandes.

   :::image type="content" source="media/search-import-data-portal/import-data-cmd2.png" alt-text="Commande Importer des données dans le portail" border="false":::

### <a name="3---set-the-data-source"></a>3 - Définissez la source de données

Dans la page **Source de données**, la source doit être **Stockage Blob Azure**, avec les spécifications suivantes :

+ **Données à extraire** doit être *Contenu et métadonnées*. Le choix de cette option permet à l’Assistant d’inférer un schéma d’index et de mapper les champs pour l’importation.
   
+ **Mode d’analyse** doit être défini sur *JSON*, *Tableau JSON* ou *Lignes JSON*. 

  *JSON* considère chaque objet blob comme un seul document de recherche, qui apparaît comme élément indépendant dans les résultats de la recherche. 

  *Tableau JSON* s’applique aux objets Blob contenant des données JSON bien formées. Le JSON bien formé correspond à un tableau d’objets, ou à une propriété qui est un tableau d’objets et vous voulez que chaque élément soit considéré comme un document de recherche autonome et indépendant. Si les objets blob sont complexes et que vous ne choisissez pas *Tableau JSON*, l’objet blob tout entier est ingéré sous la forme d’un seul document.

  *Lignes JSON* concerne les objets Blob composés de plusieurs entités JSON séparées par un saut de ligne, où vous voulez que chaque élément soit considéré comme un document de recherche autonome et indépendant. Si les objets Blob sont complexes et que vous ne choisissez pas le mode d’analyse *Lignes JSON*, l’objet Blob tout entier est alors ingéré sous la forme d’un seul document.
   
+ **Conteneur de stockage** doit spécifier votre compte de stockage et votre conteneur, ou une chaîne de connexion dont la résolution aboutit au conteneur. Vous pouvez obtenir des chaînes de connexion sur la page du portail Service Blob.

   :::image type="content" source="media/search-howto-index-json/import-wizard-json-data-source.png" alt-text="Définition de la source de données des objets blob" border="false":::

### <a name="4---skip-the-enrich-content-page-in-the-wizard"></a>4 – Ignorer la page « Enrichir le contenu » de l’Assistant

L’ajout de compétences cognitives (ou enrichissement) n’est pas une condition d’importation. Si vous n’avez pas besoin d’[ajouter un enrichissement de l’IA](cognitive-search-concept-intro.md) à votre pipeline d’indexation, ignorez cette étape.

Pour ignorer cette étape, cliquez sur les boutons bleus au bas de la page pour « Suivant » et « Ignorer ».

### <a name="5---set-index-attributes"></a>5 - Définissez les attributs de l’index

Dans la page **Index**, vous devez voir une liste de champs avec un type de données et une série de cases à cocher permettant de définir les attributs de l’index. L’Assistant peut générer une liste de champs basée sur les métadonnées et en échantillonnant les données sources. 

Vous pouvez sélectionner des attributs en bloc en cliquant sur la case à cocher en haut de la colonne d’attribut. Choisissez **Récupérable** et **Possibilité de recherche** pour chaque champ qui doit être retourné vers une application cliente et soumis à un traitement de recherche de texte intégral. Vous remarquerez que les entiers ne peuvent pas être recherchés en texte intégral ou partiel (les nombres sont évalués textuellement et sont généralement utiles dans les filtres).

Pour plus d’informations, passez en revue la description des [attributs d’index](/rest/api/searchservice/create-index#bkmk_indexAttrib) et des [analyseurs de langage](/rest/api/searchservice/language-support). 

Prenez un moment pour passer en revue vos sélections. Une fois que vous exécutez l’Assistant, des structures de données physiques sont créées : vous ne pourrez donc plus modifier ces champs sans supprimer et recréer tous les objets.

   :::image type="content" source="media/search-howto-index-json/import-wizard-json-index.png" alt-text="Définition de l’index d’objets blob" border="false":::

### <a name="6---create-indexer"></a>6 - Créez un indexeur

Une fois que tout est spécifié, l’Assistant crée trois objets distincts dans votre service de recherche. Un objet source de données et un objet index sont enregistrés comme ressources nommées dans votre service Recherche cognitive Azure. La dernière étape crée un objet indexeur. Le fait de nommer l’indexeur lui permet d’exister comme ressource autonome, que vous pouvez planifier et gérer indépendamment de l’objet index et de l’objet source de données, créés dans la même séquence de l’Assistant.

Si vous n’êtes pas familiarisé avec les indexeurs, en voici une définition : un *indexeur* est une ressource dans Recherche cognitive Azure qui analyse une source de données externe et son contenu avec possibilité de recherche. La sortie de l’Assistant **Importation de données** est un indexeur qui analyse votre source de données JSON, extrait le contenu avec possibilité de recherche et l’importe dans un index sur Recherche cognitive Azure.

   :::image type="content" source="media/search-howto-index-json/import-wizard-json-indexer.png" alt-text="Définition de l’indexeur d’objets blob" border="false":::

Cliquez sur **OK** pour exécuter l’Assistant et créer tous les objets. L’indexation commence immédiatement.

Vous pouvez surveiller l’importation des données dans les pages du portail. Des notifications de l’avancement indiquent l’état de l’indexation et le nombre de documents chargés. 

Quand l’indexation est terminée, vous pouvez utiliser l’[Explorateur de recherche](search-explorer.md) pour interroger votre index.

> [!NOTE]
> Si vous ne voyez pas les données attendues, vous devrez peut-être définir d’autres attributs sur d’autres champs. Supprimez l’index et l’indexeur que vous venez de créer et réexécutez l’Assistant, en modifiant vos sélections pour les attributs d’index à l’étape 5. 

<a name="json-indexer-rest"></a>

## <a name="use-rest-apis"></a>Utiliser les API REST

Vous pouvez utiliser l’API REST pour indexer des objets Blob JSON en suivant un flux de travail en trois parties commun à tous les indexeurs dans Recherche cognitive Azure : créer une source de données, créer un index, créer un indexeur. L’extraction de données du stockage d’objets Blob se produit lorsque vous envoyez la requête de création d’un indexeur. Lorsque cette requête est terminée, vous disposez d’un index pouvant être interrogé. 

Vous pouvez consulter l’[exemple de code REST](#rest-example) à la fin de cette section qui montre comment créer les trois objets. Cette section fournit également des détails sur les [modes d’analyse JSON](#parsing-modes), les [objets Blob uniques](#parsing-single-blobs), les [tableaux JSON](#parsing-arrays) et les [tableaux imbriqués](#nested-json-arrays).

Pour l’indexation JSON basée sur le code, utilisez [Postman ou Visual Studio Code](search-get-started-rest.md) et l’API REST pour créer ces objets :

+ [index](/rest/api/searchservice/create-index)
+ [source de données](/rest/api/searchservice/create-data-source)
+ [indexeur](/rest/api/searchservice/create-indexer)

L’ordre des opérations nécessite que vous créiez et appeliez des objets dans cet ordre. Contrairement au flux de travail du portail, une approche de code requiert un index pour accepter les documents JSON envoyés via la requête de **création d’un indexeur**.

Les objets Blob JSON dans Stockage Blob Azure se composent généralement d’un seul document JSON ou d’un tableau JSON. L’indexeur d’objets blob dans Recherche cognitive Azure peut analyser l’une ou l’autre de ces constructions selon la définition du paramètre **parsingMode** sur la requête.

| Document JSON | parsingMode | Description | Disponibilité |
|--------------|-------------|--------------|--------------|
| Un seul par objet blob | `json` | Analyse les objets blob JSON comme un bloc de texte unique. Chaque objet blob JSON devient un document Recherche cognitive Azure unique. | Généralement disponible dans les API [REST](/rest/api/searchservice/indexer-operations) et le Kit de développement logiciel (SDK) [.NET](/dotnet/api/azure.search.documents.indexes.models.searchindexer). |
| Plusieurs par objet blob | `jsonArray` | Analyse un tableau JSON dans l’objet blob, où chaque élément du tableau devient un document Recherche cognitive Azure distinct.  | Généralement disponible dans les API [REST](/rest/api/searchservice/indexer-operations) et le Kit de développement logiciel (SDK) [.NET](/dotnet/api/azure.search.documents.indexes.models.searchindexer). |
| Plusieurs par objet blob | `jsonLines` | Analyse un objet Blob qui contient plusieurs entités JSON (« tableau ») séparées par un saut de ligne, où chaque entité devient un document Recherche cognitive Azure distinct. | Généralement disponible dans les API [REST](/rest/api/searchservice/indexer-operations) et le Kit de développement logiciel (SDK) [.NET](/dotnet/api/azure.search.documents.indexes.models.searchindexer). |

### <a name="1---assemble-inputs-for-the-request"></a>1 - Assembler des entrées pour la requête

Pour chaque requête, vous devez fournir le nom du service et la clé d’administration pour Recherche cognitive Azure (dans l’en-tête POST), ainsi que le nom du compte de stockage et la clé pour le stockage d’objets Blob. Vous pouvez utiliser un [outil de test d’API web](search-get-started-rest.md) pour envoyer des requêtes HTTP à Recherche cognitive Azure.

Copiez les quatre valeurs suivantes dans le bloc-notes pour pouvoir les coller dans une requête :

+ Nom du service Recherche cognitive Azure
+ Clé d’administration de Recherche cognitive Azure
+ Nom du compte de stockage Azure
+ Clé du compte de Stockage Azure

Vous pouvez trouver ces valeurs dans le portail :

1. Dans les pages du portail pour Recherche cognitive Azure, copiez l’URL du service de recherche dans la page Vue d’ensemble.

2. Dans le volet de navigation gauche, cliquez sur **Clés**, puis copiez la clé primaire ou secondaire (elles sont équivalentes).

3. Basculez vers les pages du portail pour votre compte de stockage. Dans le volet de navigation gauche, sous **Paramètres**, sélectionnez **Clés d’accès**. Cette page fournit le nom du compte et la clé. Copiez le nom du compte de stockage et une des clés dans le bloc-notes.

### <a name="2---create-a-data-source"></a>2 - Création d'une source de données

Cette étape consiste à fournir les informations de connexion à la source de données utilisées par l’indexeur. La source de données est un objet nommé dans Recherche cognitive Azure qui rend persistantes les informations de connexion. Le type de source de données, `azureblob`, détermine les comportements d’extraction de données appelés par l’indexeur. 

Remplacez les valeurs valides des espaces réservés de nom du service, clé d’administration, compte de stockage et clé de compte.

```http
    POST https://[service name].search.windows.net/datasources?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key for Azure Cognitive Search]

    {
        "name" : "my-blob-datasource",
        "type" : "azureblob",
        "credentials" : { "connectionString" : "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>;" },
        "container" : { "name" : "my-container", "query" : "optional, my-folder" }
    }   
```

### <a name="3---create-a-target-search-index"></a>3 - Créer un index de recherche cible 

Les indexeurs sont couplés à un schéma d’index. Si vous utilisez l’API (à la place du portail), préparez un index à l’avance pour pouvoir le spécifier sur l’opération de l’indexeur.

L’index stocke le contenu avec possibilité de recherche dans Recherche cognitive Azure. Pour créer un index, fournissez un schéma qui spécifie les champs d’un document, les attributs et d’autres constructions qui façonnent l’expérience de recherche. Si vous créez un index qui a les mêmes noms de champs et les mêmes types de données que la source, l’indexeur met en correspondance les champs sources et de destination, ce qui vous évite de devoir mapper explicitement les champs.

L’exemple suivant montre une demande [Créer un index](/rest/api/searchservice/create-index). L’index aura un champ `content` avec possibilité de recherche pour stocker le texte extrait d’objets blob :   

```http
    POST https://[service name].search.windows.net/indexes?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key for Azure Cognitive Search]

    {
          "name" : "my-target-index",
          "fields": [
            { "name": "id", "type": "Edm.String", "key": true, "searchable": false },
            { "name": "content", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false }
          ]
    }
```


### <a name="4---configure-and-run-the-indexer"></a>4 - Configurer et exécuter l’indexeur

Comme c’est le cas pour l’index et la source de données, un indexeur est également objet nommé que vous créez et réutilisez sur un service Recherche cognitive Azure. Une requête complète pour créer un indexeur peut se présenter comme suit :

```http
    POST https://[service name].search.windows.net/indexers?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key for Azure Cognitive Search]

    {
      "name" : "my-json-indexer",
      "dataSourceName" : "my-blob-datasource",
      "targetIndexName" : "my-target-index",
      "schedule" : { "interval" : "PT2H" },
      "parameters" : { "configuration" : { "parsingMode" : "json" } }
    }
```

La configuration de l’indexeur se trouve dans le corps de la requête. Elle nécessite une source de données et un index cible vide qui existe déjà dans Recherche cognitive Azure. 

La planification et les paramètres sont facultatifs. Si vous les omettez, l’indexeur s’exécute immédiatement en mode d’analyse `json`.

Cet indexeur particulier n’inclut pas les mappages de champs. Dans la définition de l’indexeur, vous pouvez ignorer les **mappages de champs** si les propriétés du document JSON source correspond aux champs de votre index de recherche cible. 


### <a name="rest-example"></a>Exemple REST

Cette section est un récapitulatif de toutes les requêtes utilisées pour la création d’objets. Pour une présentation des composants, consultez les sections précédentes de cet article.

### <a name="data-source-request"></a>Requête de source de données

Tous les indexeurs nécessitent un objet de source de données qui fournit des informations de connexion aux données existantes. 

```http
    POST https://[service name].search.windows.net/datasources?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key for Azure Cognitive Search]

    {
        "name" : "my-blob-datasource",
        "type" : "azureblob",
        "credentials" : { "connectionString" : "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>;" },
        "container" : { "name" : "my-container", "query" : "optional, my-folder" }
    }  
```

### <a name="index-request"></a>Requête d’index

Tous les indexeurs nécessitent un index cible qui reçoit les données. Le corps de la requête définit le schéma d’index, composé de champs, attribué pour prendre en charge les comportements souhaités dans un index pouvant faire l’objet d’une recherche. Cet index doit être vide lorsque vous exécutez l’indexeur. 

```http
    POST https://[service name].search.windows.net/indexes?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key for Azure Cognitive Search]

    {
          "name" : "my-target-index",
          "fields": [
            { "name": "id", "type": "Edm.String", "key": true, "searchable": false },
            { "name": "content", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false }
          ]
    }
```

### <a name="indexer-request"></a>Requête d’indexeur

Cette requête montre un indexeur complètement spécifié. Il inclut des mappages de champs, qui ont été omis dans les exemples précédents. Rappelez-vous que « schedule », « parameters » et « fieldMappings » sont facultatifs tant qu’une valeur par défaut est disponible. L’omission de « schedule » entraîne une exécution immédiate de l’indexeur. Si « parsingMode » est omis, l’index utilise la valeur « json » par défaut.

La création de l’indexeur sur Recherche cognitive Azure déclenche l’importation des données. Elle s’exécute immédiatement, puis selon une planification si vous en avez fourni une.

```http
    POST https://[service name].search.windows.net/indexers?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key for Azure Cognitive Search]

    {
      "name" : "my-json-indexer",
      "dataSourceName" : "my-blob-datasource",
      "targetIndexName" : "my-target-index",
      "schedule" : { "interval" : "PT2H" },
      "parameters" : { "configuration" : { "parsingMode" : "json" } },
      "fieldMappings" : [
        { "sourceFieldName" : "/article/text", "targetFieldName" : "text" },
        { "sourceFieldName" : "/article/datePublished", "targetFieldName" : "date" },
        { "sourceFieldName" : "/article/tags", "targetFieldName" : "tags" }
        ]
    }
```

<a name="json-indexer-dotnet"></a>

## <a name="use-net-sdk"></a>Utilisation du Kit de développement logiciel (SDK) .NET

Le Kit de développement logiciel (SDK) .NET est totalement identique à l’API REST. Nous vous recommandons de consulter la section précédente de l’API REST pour découvrir les concepts, les workflows et les exigences. Vous pouvez alors vous référer à la documentation de référence des API .NET suivante pour implémenter un indexeur JSON dans du code managé.

+ [azure.search.documents.indexes.models.searchindexerdatasourceconnection](/dotnet/api/azure.search.documents.indexes.models.searchindexerdatasourceconnection)
+ [azure.search.documents.indexes.models.searchindexerdatasourcetype](/dotnet/api/azure.search.documents.indexes.models.searchindexerdatasourcetype) 
+ [azure.search.documents.indexes.models.searchindex](/dotnet/api/azure.search.documents.indexes.models.searchindex) 
+ [azure.search.documents.indexes.models.searchindexer](/dotnet/api/azure.search.documents.indexes.models.searchindexer)

<a name="parsing-modes"></a>

## <a name="parsing-modes"></a>Modes d’analyse

Les objets Blob JSON peuvent prendre plusieurs formules. Le paramètre **parsingMode** de l’indexeur JSON détermine comment le contenu d’objet Blob JSON est analysé et structuré dans un index Recherche cognitive Azure :

| parsingMode | Description |
|-------------|-------------|
| `json`  | Indexez chaque objet Blob comme un seul document. Il s’agit de la valeur par défaut. |
| `jsonArray` | Choisissez ce mode si vos objets Blob sont constitués de tableaux JSON et s’il faut que chaque élément du tableau devienne un document distinct dans Recherche cognitive Azure. |
|`jsonLines` | Choisissez ce mode si vos objets Blob sont constitués de plusieurs entités JSON, qui sont séparées par un saut de ligne, et que chacune doit devenir un document distinct dans Recherche cognitive Azure. |

Vous pouvez considérer un document comme un élément individuel dans les résultats de la recherche. Si vous voulez que chaque élément du tableau apparaisse dans les résultats de la recherche comme élément indépendant, utilisez l’option `jsonArray` ou `jsonLines` le cas échéant.

Dans la définition de l’indexeur, vous pouvez utiliser des [mappages de champs](search-indexer-field-mappings.md) pour choisir les propriétés du document JSON source à utiliser pour remplir votre index de recherche cible. Pour le mode d’analyse `jsonArray`, si le tableau existe en tant que propriété de plus bas niveau, vous pouvez définir une racine de document qui indique l’emplacement du tableau dans l’objet Blob.

> [!IMPORTANT]
> Lorsque vous utilisez `json`, `jsonArray` ou `jsonLines`, Recherche cognitive Azure suppose que tous les objets Blob dans votre source de données contiennent JSON. Si vous devez prendre en charge une combinaison d’objets blob JSON et autres dans la même source de données, faites-le nous savoir sur [notre site UserVoice](https://feedback.azure.com/forums/263029-azure-search).


<a name="parsing-single-blobs"></a>

## <a name="parse-single-json-blobs"></a>Analyser des objets Blob JSON uniques

Par défaut, [l’indexeur d’objets blob Azure Search](search-howto-indexing-azure-blob-storage.md) analyse les objets blob JSON comme un bloc de texte unique. Vous souhaitez généralement conserver la structure de vos documents JSON. En guise d’exemple, prenons le document JSON suivant dans Stockage Blob Azure :

```http
    {
        "article" : {
            "text" : "A hopefully useful article explaining how to parse JSON blobs",
            "datePublished" : "2016-04-13",
            "tags" : [ "search", "storage", "howto" ]    
        }
    }
```

L’indexeur d’objets blob analyse le document JSON en un seul document Recherche cognitive Azure. L’indexeur charge un index en mettant en correspondance les champs « text », « datePublished » et « tags » de la source avec les champs cibles de même nom et de même type.

Comme indiqué, les mappages de champs ne sont pas nécessaires. Étant donné un index avec les champs « text », « datePublished » et « tags », l’indexeur d’objets blob peut déduire le mappage correct sans qu’un mappage de champs ne soit présent dans la requête.

<a name="parsing-arrays"></a>

## <a name="parse-json-arrays"></a>Analyser des tableaux JSON

Vous pouvez également utiliser l’option de tableau JSON. Cette option est utile lorsque les objets Blob contiennent un *tableau d’objets JSON bien formés* et que vous souhaitez que chaque élément devienne un document Recherche cognitive Azure distinct. Prenons par exemple l’objet blob JSON suivant. Vous pouvez remplir l’index Recherche cognitive Azure avec trois documents distincts contenant chacun les champs « id » et « text ».  

```text
    [
        { "id" : "1", "text" : "example 1" },
        { "id" : "2", "text" : "example 2" },
        { "id" : "3", "text" : "example 3" }
    ]
```

Pour un tableau JSON, la définition de l’indexeur doit être similaire à l’exemple suivant. Notez que le paramètre parsingMode spécifie l’analyseur `jsonArray`. Spécifier l’analyseur correct et avoir les bonnes entrées de données sont les deux seules conditions spécifiques aux tableaux pour l’indexation d’objets Blob JSON.

```http
    POST https://[service name].search.windows.net/indexers?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
      "name" : "my-json-indexer",
      "dataSourceName" : "my-blob-datasource",
      "targetIndexName" : "my-target-index",
      "schedule" : { "interval" : "PT2H" },
      "parameters" : { "configuration" : { "parsingMode" : "jsonArray" } }
    }
```

Là encore, notez que les mappages des champs peuvent être omis. En supposant un index avec des champs nommés de façon identique « id » et « text », l’indexeur d’objets blob peut inférer le mappage correct sans une liste de mappages des champs explicites.

<a name="nested-json-arrays"></a>

## <a name="parse-nested-arrays"></a>Analyser des tableaux imbriqués
Pour les tableaux JSON comprenant des éléments imbriqués, vous pouvez spécifier un `documentRoot` pour indiquer une structure à plusieurs niveaux. Par exemple, si vos objets blob ressemblent à ceci :

```http
    {
        "level1" : {
            "level2" : [
                { "id" : "1", "text" : "Use the documentRoot property" },
                { "id" : "2", "text" : "to pluck the array you want to index" },
                { "id" : "3", "text" : "even if it's nested inside the document" }  
            ]
        }
    }
```

Utilisez cette configuration pour indexer le tableau contenu dans la propriété `level2` :

```http
    {
        "name" : "my-json-array-indexer",
        ... other indexer properties
        "parameters" : { "configuration" : { "parsingMode" : "jsonArray", "documentRoot" : "/level1/level2" } }
    }
```

## <a name="parse-blobs-separated-by-newlines"></a>Analyser des objets Blob séparés par des sauts de ligne

Si votre objet Blob contient plusieurs entités JSON séparées par un saut de ligne et que vous souhaitez que chaque élément devienne un document de Recherche cognitive Azure distinct, vous pouvez choisir l’option de lignes JSON. Prenons, par exemple, l’objet Blob JSON suivant. Vous pouvez remplir l’index Recherche cognitive Azure avec trois documents distincts contenant chacun les champs « id » et « text ».

```text
{ "id" : "1", "text" : "example 1" }
{ "id" : "2", "text" : "example 2" }
{ "id" : "3", "text" : "example 3" }
```

Pour les lignes JSON, la définition de l’indexeur doit être similaire à l’exemple suivant. Notez que le paramètre parsingMode spécifie l’analyseur `jsonLines`. 

```http
    POST https://[service name].search.windows.net/indexers?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
      "name" : "my-json-indexer",
      "dataSourceName" : "my-blob-datasource",
      "targetIndexName" : "my-target-index",
      "schedule" : { "interval" : "PT2H" },
      "parameters" : { "configuration" : { "parsingMode" : "jsonLines" } }
    }
```

Là encore, notez que les mappages des champs peuvent être omis, comme dans le mode d’analyse `jsonArray`.

## <a name="add-field-mappings"></a>Ajouter des mappages de champs

Quand les champs sources et cibles ne sont pas parfaitement alignés, vous pouvez définir une section de mappage de champs dans le corps de la requête pour des associations champ à champ explicites.

Recherche cognitive Azure ne peut actuellement pas indexer des documents JSON arbitraires directement, car il ne prend en charge que les types de données primitifs, les tableaux de chaînes et les points GeoJSON. Toutefois, vous pouvez utiliser les **mappages de champ** pour sélectionner des parties de votre document JSON et les intégrer dans les champs de niveau supérieur du document de recherche. Pour en savoir plus sur les principes de base des mappages de champs, consultez [Mappages de champs dans les indexeurs Recherche cognitive Azure](search-indexer-field-mappings.md).

Revenons à notre exemple de document JSON :

```http
    {
        "article" : {
            "text" : "A hopefully useful article explaining how to parse JSON blobs",
            "datePublished" : "2016-04-13"
            "tags" : [ "search", "storage", "howto" ]    
        }
    }
```

Prenons un index de recherche avec les champs suivants : `text` de type `Edm.String`, `date` de type `Edm.DateTimeOffset` et `tags` de type `Collection(Edm.String)`. Notez la différence entre « datePublished » dans la source et le champ `date` dans l’index. Pour mapper votre document JSON à la forme souhaitée, utilisez les mappages de champ suivants :

```http
    "fieldMappings" : [
        { "sourceFieldName" : "/article/text", "targetFieldName" : "text" },
        { "sourceFieldName" : "/article/datePublished", "targetFieldName" : "date" },
        { "sourceFieldName" : "/article/tags", "targetFieldName" : "tags" }
      ]
```

Les noms de champ source dans les mappages sont spécifiés selon la notation de [pointeur JSON](https://tools.ietf.org/html/rfc6901) . Vous débutez par une barre oblique pour faire référence à la racine de votre document JSON, puis sélectionnez la propriété souhaitée (au niveau arbitraire de l’imbrication) en utilisant un chemin d’accès séparé par des barres obliques avant.

Vous pouvez également faire référence à des éléments de tableau en utilisant un index de base zéro. Par exemple, pour sélectionner le premier élément du tableau « tags » dans l’exemple ci-dessus, utilisez un mappage de champ similaire au suivant :

```http
    { "sourceFieldName" : "/article/tags/0", "targetFieldName" : "firstTag" }
```

> [!NOTE]
> Si un nom de champ source dans un chemin de mappage de champ fait référence à une propriété qui n’existe pas dans JSON, ce mappage est ignoré sans erreur. Cela nous permet de prendre en charge les documents avec un schéma différent (cas fréquent). Comme il n’y a aucune validation, vous devez veiller à éviter les fautes de frappe dans la spécification du mappage de champ.
>

## <a name="help-us-make-azure-cognitive-search-better"></a>Aidez-nous à améliorer Recherche cognitive Azure
Si vous avez des suggestions de fonctionnalités ou des idées d’amélioration, faites-le-nous savoir [UserVoice](https://feedback.azure.com/forums/263029-azure-search/). Si vous avez besoin d’aide pour utiliser la fonctionnalité existante, publiez votre question sur [Stack Overflow](https://stackoverflow.microsoft.com/questions/tagged/18870).

## <a name="see-also"></a>Voir également

+ [Indexeurs dans Recherche cognitive Azure](search-indexer-overview.md)
+ [Indexation de Stockage Blob Azure avec Recherche cognitive Azure](search-howto-index-json-blobs.md)
+ [Indexation d’objets blob CSV avec l’indexeur d’objets blob de Recherche cognitive Azure](search-howto-index-csv-blobs.md)
+ [Tutoriel : Rechercher des données semi-structurées à partir de Stockage Blob Azure](search-semi-structured-data.md)