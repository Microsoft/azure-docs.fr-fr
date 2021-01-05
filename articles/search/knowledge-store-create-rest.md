---
title: Créer une base de connaissances avec REST
titleSuffix: Azure Cognitive Search
description: Utilisez l’API REST et Postman pour créer une base de connaissances Recherche cognitive Azure pour conserver les enrichissements d’un pipeline d’enrichissement par IA.
author: HeidiSteen
manager: nitinme
ms.author: heidist
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 11/18/2020
ms.openlocfilehash: 6af9b8f97d622ae10cfdbcaa8ca50abb42ec7332
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94889051"
---
# <a name="create-a-knowledge-store-using-rest-and-postman"></a>Créer une base de connaissances à l’aide de REST et Postman

Une base de connaissances contient la sortie d’un pipeline d’enrichissement Recherche cognitive Azure pour une analyse ultérieure ou tout autre traitement en aval. Un pipeline enrichi par IA accepte les fichiers image ou les fichiers texte non structurés, les index en utilisant Recherche cognitive Azure, applique des enrichissements par IA provenant de Cognitive Services (par exemple l’analyse d’images et le traitement en langage naturel), puis enregistre les résultats dans une base de connaissances dans Stockage Azure. Vous pouvez utiliser des outils comme Power BI ou l’Explorateur Stockage sur le portail Azure pour explorer la base de connaissances.

Dans cet article, vous allez utiliser l’interface de l’API REST pour ingérer, indexer et appliquer des enrichissements par IA à un ensemble d’avis sur des hôtels. Les avis sur les hôtels sont importés dans le Stockage Blob Azure. Les résultats sont enregistrés sous la forme d’une base de connaissances dans le stockage Table Azure.

Une fois la base de connaissances créée, vous pouvez apprendre à y accéder à l’aide de l’[Explorateur Stockage](knowledge-store-view-storage-explorer.md) ou de [Power BI](knowledge-store-connect-power-bi.md).

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

> [!TIP]
> Nous vous recommandons l’[application de bureau Postman](https://www.getpostman.com/) pour cet article. Le [code source](https://github.com/Azure-Samples/azure-search-postman-samples/tree/master/knowledge-store) de cet article comprend une collection Postman contenant toutes les requêtes. 

## <a name="create-services-and-load-data"></a>Créer des services et charger des données

Ce guide de démarrage rapide utilise la Recherche cognitive Azure, le Stockage Blob Azure et [Azure Cognitive Services](https://azure.microsoft.com/services/cognitive-services/) pour l’IA. 

En raison de la taille réduite de la charge de travail, Cognitive Services est utilisé en arrière-plan pour traiter gratuitement jusqu’à 20 transactions par jour. Parce que le jeu de données est vraiment petit, vous pouvez ignorer la création ou l’attachement d’une ressource Cognitive Services.

1. [Téléchargez le fichier HotelReviews_Free.csv](https://knowledgestoredemo.blob.core.windows.net/hotel-reviews/HotelReviews_Free.csv?sp=r&st=2019-11-04T01:23:53Z&se=2025-11-04T16:00:00Z&spr=https&sv=2019-02-02&sr=b&sig=siQgWOnI%2FDamhwOgxmj11qwBqqtKMaztQKFNqWx00AY%3D). Ce fichier CSV contient des données d’avis d’hôtel (issues de Kaggle.com). Il rassemble 19 commentaires de clients relatifs à un seul hôtel. 

1. [Créez un compte de stockage Azure](../storage/common/storage-account-create.md?tabs=azure-portal) ou [recherchez un compte existant](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Storage%2storageAccounts/) dans votre abonnement actuel. Vous utilisez le stockage Azure pour le contenu brut à importer, mais aussi pour la base de connaissances qui est le résultat final.

   Choisissez le type de compte **StorageV2 (usage général v2)** .

1. Ouvrez les pages des services Blob et créez un conteneur nommé *hotel-reviews*.

1. Cliquez sur **Télécharger**.

    ![Charger les données](media/knowledge-store-create-portal/upload-command-bar.png "Charger les avis sur les hôtels")

1. Sélectionnez le fichier **HotelReviews-Free.csv** que vous avez téléchargé à la première étape.

    ![Créer le conteneur d’objets Blob Azure](media/knowledge-store-create-portal/hotel-reviews-blob-container.png "Créer le conteneur d’objets Blob Azure")

1. Vous avez presque terminé avec cette ressource, mais avant de quitter ces pages, ouvrez la page **Clés d’accès** à partir du lien correspondant dans le volet de navigation de gauche. Obtenez une chaîne de connexion pour récupérer les données du Stockage Blob. Une chaîne de connexion ressemble à l’exemple suivant : `DefaultEndpointsProtocol=https;AccountName=<YOUR-ACCOUNT-NAME>;AccountKey=<YOUR-ACCOUNT-KEY>;EndpointSuffix=core.windows.net`

1. Toujours dans le portail, basculez vers Recherche cognitive Azure. [Créez un service](search-create-service-portal.md) ou [recherchez un service existant](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices). Vous pouvez utiliser un service gratuit pour cet exercice.

## <a name="configure-postman"></a>Configurer Postman

Installez et configurez Postman.

### <a name="download-and-install-postman"></a>Télécharger et installer Postman

1. Téléchargez le [code source de la collection Postman](https://github.com/Azure-Samples/azure-search-postman-samples/blob/master/knowledge-store/KnowledgeStore.postman_collection.json).
1. Sélectionnez **Fichier** > **Importer** pour importer le code source dans Postman.
1. Sélectionnez l’onglet **Collections**, puis le bouton **...** (points de suspension).
1. Sélectionnez **Modifier**. 
   
   ![Application Postman montrant la navigation](media/knowledge-store-create-rest/postman-edit-menu.png "Accéder au menu Edition dans Postman")
1. Dans la boîte de dialogue **Edit**, sélectionnez l’onglet **Variables**. 

Sous l’onglet **Variables**, vous pouvez ajouter les valeurs que Postman récupère chaque fois qu’il trouve une variable spécifique entre double accolades. Par exemple, Postman remplace le symbole `{{admin-key}}` par la valeur actuelle que vous avez définie pour `admin-key`. Postman effectue la substitution dans les URL, les en-têtes, le corps de la requête, etc. 

Pour obtenir la valeur de `admin-key`, accédez au service Recherche cognitive Azure et sélectionnez l’onglet **Clés**. Remplacez les valeurs de `search-service-name` et `storage-account-name` par celles que vous avez choisies dans [Créer des services](#create-services-and-load-data). Définissez `storage-connection-string` en utilisant la valeur figurant sous l’onglet **Clés d’accès** du compte de stockage. Partout ailleurs, vous pouvez conserver les valeurs par défaut.

![Onglet Variables de l’application Postman](media/knowledge-store-create-rest/postman-variables-window.png "Fenêtre des variables de Postman")


| Variable    | Comment les obtenir |
|-------------|-----------------|
| `admin-key` | Dans la page **Clés** du service Recherche cognitive Azure.  |
| `api-version` | Laisser **2020-06-30**. |
| `datasource-name` | Laisser **hotel-reviews-ds**. | 
| `indexer-name` | Laisser **hotel-reviews-ixr**. | 
| `index-name` | Laisser **hotel-reviews-ix**. | 
| `search-service-name` | Nom du service Recherche cognitive Azure. L’URL est `https://{{search-service-name}}.search.windows.net`. | 
| `skillset-name` | Laisser **hotel-reviews-ss**. | 
| `storage-account-name` | Nom du compte de stockage Azure. | 
| `storage-connection-string` | Dans le compte de stockage, sous l’onglet **Clés d’accès**, sélectionnez **key1** > **Chaîne de connexion**. | 
| `storage-container-name` | Laisser **hotel-reviews**. | 

### <a name="review-the-request-collection-in-postman"></a>Passer en revue la collection de requêtes dans Postman

Quand vous créez une base de connaissances, vous devez émettre quatre requêtes HTTP : 

- **Une requête PUT pour créer l’index** : Cet index contient les données utilisées et retournées par Recherche cognitive Azure.
- **Une requête POST pour créer la source de données** : Cette source de données connecte le comportement du service Recherche cognitive Azure au compte de stockage des données et de la base de connaissances. 
- **Une requête PUT pour créer l’ensemble de compétences** : l’ensemble de compétences spécifie les enrichissements appliqués à vos données et à la structure de la base de connaissances.
- **Une requête PUT pour créer l’indexeur** : L’exécution de l’indexeur lit les données, applique l’ensemble de compétences et stocke les résultats. Vous devez exécuter cette requête en dernier.

Le [code source](https://github.com/Azure-Samples/azure-search-postman-samples/blob/master/knowledge-store/KnowledgeStore.postman_collection.json) contient une collection Postman avec ces quatre requêtes. Pour émettre les requêtes, dans Postman, sélectionnez l’onglet correspondant à la requête. Ajoutez ensuite les en-têtes de requête `api-key` et `Content-Type`. Affectez à `api-key` la valeur `{{admin-key}}`. Affectez à `Content-type` la valeur `application/json`. 

![Capture d’écran présentant l’interface de Postman pour les en-têtes](media/knowledge-store-create-rest/postman-headers-ui.png)

> [!Note]
> Vous devez définir les en-têtes `api-key` et `Content-type` dans toutes vos requêtes. Si Postman reconnaît une variable, celle-ci se apparaît en orange, comme pour `{{admin-key}}` dans la capture d’écran précédente. Si la variable est mal orthographiée, elle apparaît en rouge.
>

## <a name="create-an-azure-cognitive-search-index"></a>Création d’un index Recherche cognitive Azure

Créez un index Recherche cognitive Azure pour représenter les données sur lesquelles vous souhaitez effectuer des recherches et des filtrages, et appliquer des améliorations. Créez l’index en émettant une requête PUT vers `https://{{search-service-name}}.search.windows.net/indexes/{{index-name}}?api-version={{api-version}}`. Postman remplace les symboles placés entre deux accolades (par exemple, `{{search-service-name}}`, `{{index-name}}` et `{{api-version}}`) par les valeurs que vous avez définies dans [Configurer Postman](#configure-postman). Si vous utilisez un autre outil pour émettre vos commandes REST, vous devez vous-même remplacer ces variables.

Définissez la structure de votre index Recherche cognitive Azure dans le corps de la requête. Dans Postman, après avoir défini les en-têtes `api-key` et `Content-type`, accédez au volet **Body** (Corps) de la requête. Le code JSON suivant doit s’afficher. Si ce n’est pas le cas, sélectionnez **Raw** > **JSON (application/json)** , puis collez le code suivant dans le corps :

```JSON
{
    "name": "{{index-name}}",
    "fields": [
        { "name": "address", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "categories", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "city", "type": "Edm.String", "filterable": false, "sortable": false, "facetable": false },
        { "name": "country", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "latitude", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "longitude", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "name", "type": "Edm.String", "filterable": false, "sortable": false, "facetable": false },
        { "name": "postalCode", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "province", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "reviews_date", "type": "Edm.DateTimeOffset", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "reviews_dateAdded", "type": "Edm.DateTimeOffset", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "reviews_rating", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "reviews_text", "type": "Edm.String", "filterable": false,  "sortable": false, "facetable": false },
        { "name": "reviews_title", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "reviews_username", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "AzureSearch_DocumentKey", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false, "key": true },
        { "name": "metadata_storage_content_type", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "metadata_storage_size", "type": "Edm.Int64", "searchable": false, "filterable": false, "sortable": false, "facetable": false},
        { "name": "metadata_storage_last_modified", "type": "Edm.DateTimeOffset", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "metadata_storage_name", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "metadata_storage_path", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "Sentiment", "type": "Collection(Edm.Double)", "searchable": false, "filterable": true, "retrievable": true, "sortable": false, "facetable": true },
        { "name": "Language", "type": "Edm.String", "filterable": true, "sortable": false, "facetable": true },
        { "name": "Keyphrases", "type": "Collection(Edm.String)", "filterable": true, "sortable": false, "facetable": true }
    ]
}

```

Cette définition d’index est une combinaison de données que vous souhaitez présenter à l’utilisateur (nom de l’hôtel, contenu de l’avis, date), de métadonnées de recherche et de données d’amélioration de l’IA (sentiment, phrases clés et langue).

Sélectionnez **Send** (Envoyer) pour émettre la requête PUT. L’état `201 - Created` doit alors s’afficher. Si vous un autre état s’affiche, dans le volet **Body**, recherchez une réponse JSON qui contient un message d’erreur. 

## <a name="create-the-datasource"></a>Créer la source de données

Ensuite, connectez Recherche cognitive Azure aux données relatives aux hôtels que vous avez stockées dans le stockage Blob. Pour créer la source de données, envoyez une requête POST à `https://{{search-service-name}}.search.windows.net/datasources?api-version={{api-version}}`. Vous devez définir les en-têtes `api-key` et `Content-Type` comme indiqué précédemment. 

Dans Postman, accédez à la requête **Create Datasource** (Créer une source de données), puis au volet **Body** (Corps). Le code suivant doit s’afficher :

```json
{
  "name" : "{{datasource-name}}",
  "description" : "Demo files to demonstrate knowledge store capabilities.",
  "type" : "azureblob",
  "credentials" : { "connectionString" : "{{storage-connection-string}}" },
  "container" : { "name" : "{{storage-container-name}}" }
}
```

Sélectionnez **Send** pour émettre la requête POST. 

## <a name="create-the-skillset"></a>Créer l’ensemble de compétences 

L’étape suivante consiste à spécifier l’ensemble de compétences, qui indique à la fois les améliorations à appliquer et la base de connaissances où sont stockés les résultats. Dans Postman, sélectionnez l’onglet **Create the Skillset** (Créer l’ensemble de compétences). Cette requête envoie un PUT à `https://{{search-service-name}}.search.windows.net/skillsets/{{skillset-name}}?api-version={{api-version}}`. Définissez les en-têtes `api-key` et `Content-type` comme vous l’avez fait précédemment. 

Il existe deux grands objets de niveau supérieur : `skills` et `knowledgeStore`. Chaque objet contenu dans l’objet `skills` est un service d’enrichissement. Chaque service d’enrichissement comporte `inputs` et `outputs`. `LanguageDetectionSkill` présente une sortie `targetName` qui correspond à `Language`. La valeur de ce nœud est utilisée par la plupart des autres compétences comme entrée. La source est `document/Language`. La possibilité d’utiliser la sortie d’un nœud comme entrée d’un autre nœud est encore plus évidente dans `ShaperSkill`, qui spécifie les flux de données dans les tables de la base de connaissances.

L’objet `knowledge_store` se connecte au compte de stockage via la variable Postman`{{storage-connection-string}}`. `knowledge_store` contient un ensemble de mappages entre le document amélioré et les tables et colonnes de la base de connaissances. 

Pour générer l’ensemble de compétences, sélectionnez le bouton **Send** dans Postman pour effectuer un PUT de la requête :

```json
{
    "name": "{{skillset-name}}",
    "description": "Skillset to detect language, extract key phrases, and detect sentiment",
    "skills": [ 
        {
            "@odata.type": "#Microsoft.Skills.Text.SplitSkill", 
            "context": "/document/reviews_text", "textSplitMode": "pages", "maximumPageLength": 5000,
            "inputs": [ 
                { "name": "text", "source": "/document/reviews_text" },
                { "name": "languageCode", "source": "/document/Language" }
            ],
            "outputs": [
                { "name": "textItems", "targetName": "pages" }
            ]
        },
        {
            "@odata.type": "#Microsoft.Skills.Text.SentimentSkill",
            "context": "/document/reviews_text/pages/*",
            "inputs": [
                { "name": "text", "source": "/document/reviews_text/pages/*" },
                { "name": "languageCode", "source": "/document/Language" }
            ],
            "outputs": [
                { "name": "score", "targetName": "Sentiment" }
            ]
        },
        {
            "@odata.type": "#Microsoft.Skills.Text.LanguageDetectionSkill",
            "context": "/document",
            "inputs": [
                { "name": "text", "source": "/document/reviews_text" }
            ],
            "outputs": [
                { "name": "languageCode", "targetName": "Language" }
            ]
        },
        {
            "@odata.type": "#Microsoft.Skills.Text.KeyPhraseExtractionSkill",
            "context": "/document/reviews_text/pages/*",
            "inputs": [
                { "name": "text",  "source": "/document/reviews_text/pages/*" },
                { "name": "languageCode",  "source": "/document/Language" }
            ],
            "outputs": [
                { "name": "keyPhrases" , "targetName": "Keyphrases" }
            ]
        },
        {
            "@odata.type": "#Microsoft.Skills.Util.ShaperSkill",
            "context": "/document",
            "inputs": [
                { "name": "name",  "source": "/document/name" },
                { "name": "reviews_date",  "source": "/document/reviews_date" },
                { "name": "reviews_rating",  "source": "/document/reviews_rating" },
                { "name": "reviews_text",  "source": "/document/reviews_text" },
                { "name": "reviews_title",  "source": "/document/reviews_title" },
                { "name": "AzureSearch_DocumentKey",  "source": "/document/AzureSearch_DocumentKey" },
                { 
                    "name": "pages",
                    "sourceContext": "/document/reviews_text/pages/*",
                    "inputs": [
                        { "name": "SentimentScore", "source": "/document/reviews_text/pages/*/Sentiment" },
                        { "name": "LanguageCode", "source": "/document/Language" },
                        { "name": "Page", "source": "/document/reviews_text/pages/*" },
                        { 
                            "name": "keyphrase", "sourceContext": "/document/reviews_text/pages/*/Keyphrases/*",
                            "inputs": [
                                { "name": "Keyphrases", "source": "/document/reviews_text/pages/*/Keyphrases/*" }
                            ]
                        }
                    ]
                }
            ],
            "outputs": [
                { "name": "output" , "targetName": "tableprojection" }
            ]
        }
    ],
    "knowledgeStore": {
        "storageConnectionString": "{{storage-connection-string}}",
        "projections": [
            {
                "tables": [
                    { "tableName": "hotelReviewsDocument", "generatedKeyName": "Documentid", "source": "/document/tableprojection" },
                    { "tableName": "hotelReviewsPages", "generatedKeyName": "Pagesid", "source": "/document/tableprojection/pages/*" },
                    { "tableName": "hotelReviewsKeyPhrases", "generatedKeyName": "KeyPhrasesid", "source": "/document/tableprojection/pages/*/keyphrase/*" },
                    { "tableName": "hotelReviewsSentiment", "generatedKeyName": "Sentimentid", "source": "/document/tableprojection/pages/*/sentiment/*" }
                ],
                "objects": []
            },
            {
                "tables": [
                    { 
                        "tableName": "hotelReviewsInlineDocument", "generatedKeyName": "Documentid", "sourceContext": "/document",
                        "inputs": [
                            { "name": "name", "source": "/document/name"},
                            { "name": "reviews_date", "source": "/document/reviews_date"},
                            { "name": "reviews_rating", "source": "/document/reviews_rating"},
                            { "name": "reviews_text", "source": "/document/reviews_text"},
                            { "name": "reviews_title", "source": "/document/reviews_title"},
                            { "name": "AzureSearch_DocumentKey", "source": "/document/AzureSearch_DocumentKey" }
                        ]
                    },
                    { 
                        "tableName": "hotelReviewsInlinePages", "generatedKeyName": "Pagesid", "sourceContext": "/document/reviews_text/pages/*",
                        "inputs": [
                            { "name": "SentimentScore", "source": "/document/reviews_text/pages/*/Sentiment"},
                            { "name": "LanguageCode", "source": "/document/Language"},
                            { "name": "Page", "source": "/document/reviews_text/pages/*" }
                        ]
                    },
                    { 
                        "tableName": "hotelReviewsInlineKeyPhrases", "generatedKeyName": "kpidv2", "sourceContext": "/document/reviews_text/pages/*/Keyphrases/*",
                        "inputs": [
                            { "name": "Keyphrases", "source": "/document/reviews_text/pages/*/Keyphrases/*" }
                        ]
                    }
                ],
                "objects": []
            }
        ]
    }
}
```

## <a name="create-the-indexer"></a>Créer l’indexeur

La dernière étape consiste à créer l’indexeur. L’indexeur lit les données et active l’ensemble de compétences. Dans Postman, sélectionnez la requête **Create Indexer** (Créer l’indexeur), puis examinez le corps. La définition de l’indexeur fait référence à plusieurs autres ressources que vous avez déjà créées : la source de données, l’index et l’ensemble de compétences. 

L’objet `parameters/configuration` contrôle la manière dont l’indexeur ingère les données. Dans ce cas, les données d’entrée se trouvent dans un même document qui comporte une ligne d’en-tête et des valeurs séparées par des virgules. La clé du document est un identificateur unique du document. Avant l’encodage, la clé du document est l’URL du document source. Enfin, les valeurs de sortie de l’ensemble de l’ensemble de compétences, comme le code de langue, le sentiment et les phrases clés, sont mappées à leurs emplacements dans le document. Bien qu’il existe une seule valeur pour `Language`, `Sentiment` est appliqué à chaque élément du tableau de `pages`. `Keyphrases` est un tableau qui est aussi appliqué à chaque élément du tableau `pages`.

Une fois que vous avez défini les en-têtes `api-key` et `Content-type` et vérifié que le corps de la requête est similaire au code source suivant, sélectionnez **Send** dans Postman. Postman envoie une requête PUT à `https://{{search-service-name}}.search.windows.net/indexers/{{indexer-name}}?api-version={{api-version}}`. Recherche cognitive Azure crée et exécute l’indexeur. 

```json
{
    "name": "{{indexer-name}}",
    "dataSourceName": "{{datasource-name}}",
    "skillsetName": "{{skillset-name}}",
    "targetIndexName": "{{index-name}}",
    "parameters": {
        "configuration": {
            "dataToExtract": "contentAndMetadata",
            "parsingMode": "delimitedText",
            "firstLineContainsHeaders": true,
            "delimitedTextDelimiter": ","
        }
    },
    "fieldMappings": [
        {
            "sourceFieldName": "AzureSearch_DocumentKey",
            "targetFieldName": "AzureSearch_DocumentKey",
            "mappingFunction": { "name": "base64Encode" }
        }
    ],
    "outputFieldMappings": [
        { "sourceFieldName": "/document/reviews_text/pages/*/Keyphrases/*", "targetFieldName": "Keyphrases" },
        { "sourceFieldName": "/document/Language", "targetFieldName": "Language" },
        { "sourceFieldName": "/document/reviews_text/pages/*/Sentiment", "targetFieldName": "Sentiment" }
    ]
}
```

## <a name="run-the-indexer"></a>Exécuter l’indexeur 

Dans le portail Azure, accédez à la page **Vue d’ensemble** du service Recherche cognitive Azure. Sélectionnez l’onglet **Indexeurs**, puis **hotels-reviews-ixr**. Si l’indexeur n’a pas déjà été exécuté, sélectionnez **Exécuter**. La tâche d’indexation peut déclencher des avertissements liés à la reconnaissance de la langue. Les données incluent des avis rédigés dans des langues qui ne sont pas encore prises en charge par les compétences cognitives. 

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez enrichi vos données à l’aide de Cognitive Services et que vous avez projeté les résultats dans une base de connaissances, vous pouvez utiliser l’Explorateur Stockage ou Power BI pour explorer votre jeu de données enrichi.

Pour savoir comment explorer cette base de connaissances à l’aide de l’Explorateur Stockage, consultez cette procédure pas à pas :

> [!div class="nextstepaction"]
> [Voir avec l’Explorateur Stockage](knowledge-store-view-storage-explorer.md)

Pour savoir comment connecter cette base de connaissances à Power BI, consultez cette procédure pas à pas :

> [!div class="nextstepaction"]
> [Connexion avec Power BI](knowledge-store-connect-power-bi.md)

Si vous souhaitez répéter cet exercice ou essayer une autre procédure pas à pas d’enrichissement par IA, supprimez l’indexeur **hotel-reviews-idxr**. La suppression de l’indexeur remet le compteur de transactions quotidiennes gratuites à zéro.