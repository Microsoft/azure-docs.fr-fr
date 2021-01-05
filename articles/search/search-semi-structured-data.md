---
title: 'Tutoriel : Indexer des données semi-structurées dans des objets blob JSON'
titleSuffix: Azure Cognitive Search
description: Apprenez à indexer et à rechercher des objets blob Azure JSON semi-structurés à l’aide des API REST de la recherche cognitive Azure et Postman.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 09/25/2020
ms.openlocfilehash: 7c88aea6aff942cdcf5cbc022df8f07cfe0d4cce
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94701277"
---
# <a name="tutorial-index-json-blobs-from-azure-storage-using-rest"></a>Tutoriel : Indexer des objets blob JSON à partir de Stockage Azure avec REST

La recherche cognitive Azure peut indexer des tableaux et documents JSON dans le stockage d’objets blob Azure à l’aide d’un [indexeur](search-indexer-overview.md) qui sait comment lire des données semi-structurées. Les données semi-structurées contiennent des balises ou des marquages qui séparent le contenu au sein des données. Elles séparent les données non structurées, qui doivent être indexées entièrement, des données formellement structurées qui respectent un modèle de données (tel qu’un schéma de base de données relationnelle), qui peuvent être indexées par champ.

Ce tutoriel utilise Postman et les [API REST de Recherche](/rest/api/searchservice/) pour effectuer les tâches suivantes :

> [!div class="checklist"]
> * Configurer une source de données de recherche cognitive Azure pour un conteneur d’objets blob Azure
> * Créer un index de recherche cognitive Azure où stocker le contenu pouvant faire l’objet d’une recherche
> * Configurer et exécuter un indexeur pour lire le conteneur et extraire le contenu de recherche à partir du stockage d’objets blob Azure
> * Effectuer une recherche dans l’index que vous venez de créer

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

+ [Stockage Azure](../storage/common/storage-account-create.md)
+ [Application de bureau Postman](https://www.getpostman.com/)
+ [Créer](search-create-service-portal.md) ou [rechercher un service de recherche existant](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) 

> [!Note]
> Vous pouvez utiliser le service gratuit pour ce tutoriel. Avec un service de recherche gratuit, vous êtes limité à trois index, trois indexeurs et trois sources de données. Ce didacticiel crée une occurrence de chaque élément. Avant de commencer, veillez à disposer de l’espace suffisant sur votre service pour accepter les nouvelles ressources.

## <a name="download-files"></a>Télécharger les fichiers

[Clinical-trials-json.zip](https://github.com/Azure-Samples/storage-blob-integration-with-cdn-search-hdi/raw/master/clinical-trials-json.zip) contient les données utilisées dans ce tutoriel. Téléchargez et décompressez ce fichier dans son propre dossier. Les données proviennent de [clinicaltrials.gov](https://clinicaltrials.gov/ct2/results), converties au format JSON pour ce tutoriel.

## <a name="1---create-services"></a>1 - Créer les services

Ce tutoriel utilise Recherche cognitive Azure pour l’indexation et les requêtes, et Stockage Blob Azure pour fournir les données. 

Si possible, créez les deux services dans la même région et le même groupe de ressources pour des raisons de proximité et de facilité de gestion. Dans la pratique, votre compte de stockage Azure peut être dans une région quelconque.

### <a name="start-with-azure-storage"></a>Démarrer avec le stockage Azure

1. [Connectez-vous au portail Azure](https://portal.azure.com/) et cliquez sur **+ Créer une ressource**.

1. Recherchez un *compte de stockage* et sélectionnez l’offre de compte de stockage Microsoft.

   :::image type="content" source="media/cognitive-search-tutorial-blob/storage-account.png" alt-text="Créer un compte de stockage" border="false":::

1. Sous l’onglet Bases, les éléments suivants sont obligatoires. Acceptez les valeurs par défaut pour tout le reste.

   + **Groupe de ressources**. Sélectionnez un groupe existant ou créez-en un, mais utilisez le même groupe pour tous les services afin de pouvoir les gérer collectivement.

   + **Nom du compte de stockage**. Si vous pensez que vous pouvez avoir plusieurs ressources du même type, utilisez le nom pour lever l’ambiguïté par type et par région, par exemple *blobstoragewestus*. 

   + **Emplacement**. Si possible, choisissez le même emplacement que celui utilisé pour Recherche cognitive Azure et Cognitive Services. Un emplacement unique annule les frais liés à la bande passante.

   + **Type de compte**. Choisissez la valeur par défaut, *StorageV2 (v2 universel)* .

1. Cliquez sur **Vérifier + créer** pour créer le service.

1. Une fois qu’il est créé, cliquez sur **Accéder à la ressource** pour ouvrir la page Vue d’ensemble.

1. Cliquez sur le service **Objets blob**.

1. [Créez un conteneur d’objets blob](../storage/blobs/storage-quickstart-blobs-portal.md) pour contenir des exemples de données. Vous pouvez définir le niveau d’accès public sur l’une de ses valeurs valides.

1. Une fois le conteneur créé, ouvrez-le et sélectionnez **Charger** dans la barre de commandes.

   :::image type="content" source="media/search-semi-structured-data/upload-command-bar.png" alt-text="Charger dans la barre de commandes" border="false":::

1. Accédez au dossier contenant les exemples de fichiers. Sélectionnez-les tous, puis cliquez sur **Charger**.

   :::image type="content" source="media/search-semi-structured-data/clinicalupload.png" alt-text="Charger des fichiers" border="false":::

Une fois le chargement terminé, les fichiers doivent apparaître dans leur propre sous-dossier au sein du conteneur de données.

### <a name="azure-cognitive-search"></a>Recherche cognitive Azure

La ressource suivante est Recherche cognitive Azure, que vous pouvez [créer dans le portail](search-create-service-portal.md). Vous pouvez utiliser le niveau gratuit pour effectuer cette procédure pas à pas. 

Comme avec le stockage Blob Azure, prenez un moment pour collecter la clé d’accès. Par ailleurs, lorsque vous commencez à structurer les demandes, vous devez fournir le point de terminaison et la clé API d’administration utilisés pour authentifier chaque demande.

### <a name="get-a-key-and-url"></a>Obtenir une clé et une URL

Les appels REST requièrent l’URL du service et une clé d’accès et ce, sur chaque demande. Un service de recherche est créé avec les deux. Ainsi, si vous avez ajouté la Recherche cognitive Azure à votre abonnement, effectuez ce qui suit pour obtenir les informations nécessaires :

1. [Connectez-vous au portail Azure](https://portal.azure.com/), puis dans la page **Vue d’ensemble** du service de recherche, récupérez l’URL. Voici un exemple de point de terminaison : `https://mydemo.search.windows.net`.

1. Dans **Paramètres** > **Clés**, obtenez une clé d’administration pour avoir des droits d’accès complets sur le service. Il existe deux clés d’administration interchangeables, fournies pour assurer la continuité de l’activité au cas où vous deviez en remplacer une. Vous pouvez utiliser la clé primaire ou secondaire sur les demandes d’ajout, de modification et de suppression d’objets.

:::image type="content" source="media/search-get-started-rest/get-url-key.png" alt-text="Obtenir un point de terminaison et une clé d’accès HTTP" border="false":::

Toutes les demandes nécessitent une clé API sur chaque demande envoyée à votre service. L’utilisation d’une clé valide permet d’établir, en fonction de chaque demande, une relation de confiance entre l’application qui envoie la demande et le service qui en assure le traitement.

## <a name="2---set-up-postman"></a>2 - Configurer Postman

Démarrez Postman et paramétrez une requête HTTP. Si vous ne connaissez pas bien cet outil, consultez [Explorer les API REST de la Recherche cognitive Azure](search-get-started-rest.md).

Les méthodes de requête utilisées pour chaque appel dans ce tutoriel sont **POST** et **GET**. Vous allez faire trois appels d’API à votre service de recherche pour créer une source de données, un index et un indexeur. La source de données inclut un pointeur vers votre compte de stockage et vos données JSON. Votre service de recherche établit la connexion lors du chargement des données.

Dans les en-têtes, définissez « Content-type » sur `application/json` et `api-key` sur la clé API d’administration de votre service Recherche cognitive Azure. Une fois que vous avez défini les en-têtes, vous pouvez les utiliser pour chaque demande de cet exercice.

  :::image type="content" source="media/search-get-started-rest/postman-url.png" alt-text="URL et en-tête de requête Postman" border="false":::

Les URI doivent spécifier un élément « api-version » et chaque appel doit retourner une réponse **201 Créé**. L'api-version généralement disponible pour utiliser les tableaux JSON est `2020-06-30`.

## <a name="3---create-a-data-source"></a>3 - Créer une source de données

L’[API Create Data Source](/rest/api/searchservice/create-data-source) crée un objet Recherche cognitive Azure qui spécifie les données à indexer.

1. Définissez le point de terminaison de cet appel sur `https://[service name].search.windows.net/datasources?api-version=2020-06-30`. Remplacez `[service name]` par le nom de votre service de recherche. 

1. Copiez le code JSON suivant dans le corps de la requête.

    ```json
    {
        "name" : "clinical-trials-json-ds",
        "type" : "azureblob",
        "credentials" : { "connectionString" : "DefaultEndpointsProtocol=https;AccountName=[storage account name];AccountKey=[storage account key];" },
        "container" : { "name" : "[blob container name]"}
    }
    ```

1. Remplacez la chaîne de connexion par une chaîne valide pour votre compte.

1. Remplacez « [blob container name] » par le conteneur que vous avez créé pour l’exemple de données. 

1. Envoyez la demande. La réponse doit ressembler à ce qui suit :

    ```json
    {
        "@odata.context": "https://exampleurl.search.windows.net/$metadata#datasources/$entity",
        "@odata.etag": "\"0x8D505FBC3856C9E\"",
        "name": "clinical-trials-json-ds",
        "description": null,
        "type": "azureblob",
        "subtype": null,
        "credentials": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=[mystorageaccounthere];AccountKey=[[myaccountkeyhere]]];"
        },
        "container": {
            "name": "[mycontainernamehere]",
            "query": null
        },
        "dataChangeDetectionPolicy": null,
        "dataDeletionDetectionPolicy": null
    }
    ```

## <a name="4---create-an-index"></a>4 - Créer un index
    
Le deuxième appel est celui de l’[API de création d’index](/rest/api/searchservice/create-index), qui permet de créer un index de recherche cognitive Azure pour stocker toutes les données pouvant faire l’objet de recherches. Un index spécifie tous les paramètres et leurs attributs.

1. Définissez le point de terminaison de cet appel sur `https://[service name].search.windows.net/indexes?api-version=2020-06-30`. Remplacez `[service name]` par le nom de votre service de recherche.

1. Copiez le code JSON suivant dans le corps de la requête.

    ```json
    {
      "name": "clinical-trials-json-index",  
      "fields": [
      {"name": "FileName", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": false, "filterable": false, "sortable": true},
      {"name": "Description", "type": "Edm.String", "searchable": true, "retrievable": false, "facetable": false, "filterable": false, "sortable": false},
      {"name": "MinimumAge", "type": "Edm.Int32", "searchable": false, "retrievable": true, "facetable": true, "filterable": true, "sortable": true},
      {"name": "Title", "type": "Edm.String", "searchable": true, "retrievable": true, "facetable": false, "filterable": true, "sortable": true},
      {"name": "URL", "type": "Edm.String", "searchable": false, "retrievable": false, "facetable": false, "filterable": false, "sortable": false},
      {"name": "MyURL", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": false, "filterable": false, "sortable": false},
      {"name": "Gender", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": true, "filterable": true, "sortable": false},
      {"name": "MaximumAge", "type": "Edm.Int32", "searchable": false, "retrievable": true, "facetable": true, "filterable": true, "sortable": true},
      {"name": "Summary", "type": "Edm.String", "searchable": true, "retrievable": true, "facetable": false, "filterable": false, "sortable": false},
      {"name": "NCTID", "type": "Edm.String", "key": true, "searchable": true, "retrievable": true, "facetable": false, "filterable": true, "sortable": true},
      {"name": "Phase", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": true, "filterable": true, "sortable": false},
      {"name": "Date", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": false, "filterable": false, "sortable": true},
      {"name": "OverallStatus", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": true, "filterable": true, "sortable": false},
      {"name": "OrgStudyId", "type": "Edm.String", "searchable": true, "retrievable": true, "facetable": false, "filterable": true, "sortable": false},
      {"name": "HealthyVolunteers", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": true, "filterable": true, "sortable": false},
      {"name": "Keywords", "type": "Collection(Edm.String)", "searchable": true, "retrievable": true, "facetable": true, "filterable": false, "sortable": false},
      {"name": "metadata_storage_last_modified", "type":"Edm.DateTimeOffset", "searchable": false, "retrievable": true, "filterable": true, "sortable": false},
      {"name": "metadata_storage_size", "type":"Edm.String", "searchable": false, "retrievable": true, "filterable": true, "sortable": false},
      {"name": "metadata_content_type", "type":"Edm.String", "searchable": true, "retrievable": true, "filterable": true, "sortable": false}
      ]
    }
   ```

1. Envoyez la demande. La réponse doit ressembler à ce qui suit :

    ```json
    {
        "@odata.context": "https://exampleurl.search.windows.net/$metadata#indexes/$entity",
        "@odata.etag": "\"0x8D505FC00EDD5FA\"",
        "name": "clinical-trials-json-index",
        "fields": [
            {
                "name": "FileName",
                "type": "Edm.String",
                "searchable": false,
                "filterable": false,
                "retrievable": true,
                "sortable": true,
                "facetable": false,
                "key": false,
                "indexAnalyzer": null,
                "searchAnalyzer": null,
                "analyzer": null,
                "synonymMaps": []
            },
            {
                "name": "Description",
                "type": "Edm.String",
                "searchable": true,
                "filterable": false,
                "retrievable": false,
                "sortable": false,
                "facetable": false,
                "key": false,
                "indexAnalyzer": null,
                "searchAnalyzer": null,
                "analyzer": null,
                "synonymMaps": []
            },
            ...
          }
    ```

## <a name="5---create-and-run-an-indexer"></a>5 - Créer et exécuter un indexeur

Un indexeur se connecte à la source de données, importe les données dans l’index de recherche cible, et peut fournir une planification afin d’automatiser l’actualisation des données. L’API REST est celle qui consiste à [créer un indexeur](/rest/api/searchservice/create-indexer).

1. Définissez l’URI pour cet appel sur `https://[service name].search.windows.net/indexers?api-version=2020-06-30`. Remplacez `[service name]` par le nom de votre service de recherche.

1. Copiez le code JSON suivant dans le corps de la requête.

    ```json
    {
      "name" : "clinical-trials-json-indexer",
      "dataSourceName" : "clinical-trials-json-ds",
      "targetIndexName" : "clinical-trials-json-index",
      "parameters" : { "configuration" : { "parsingMode" : "jsonArray" } }
    }
    ```

1. Envoyez la demande. La requête est traitée immédiatement. Une fois la réponse reçue, vous aurez un index de recherche en texte intégral consultable. La réponse doit ressembler à ce qui suit :

    ```json
    {
        "@odata.context": "https://exampleurl.search.windows.net/$metadata#indexers/$entity",
        "@odata.etag": "\"0x8D505FDE143D164\"",
        "name": "clinical-trials-json-indexer",
        "description": null,
        "dataSourceName": "clinical-trials-json-ds",
        "targetIndexName": "clinical-trials-json-index",
        "schedule": null,
        "parameters": {
            "batchSize": null,
            "maxFailedItems": null,
            "maxFailedItemsPerBatch": null,
            "base64EncodeKeys": null,
            "configuration": {
                "parsingMode": "jsonArray"
            }
        },
        "fieldMappings": [],
        "enrichers": [],
        "disabled": null
    }
    ```

## <a name="6---search-your-json-files"></a>6- Rechercher dans vos fichiers JSON

Vous pouvez démarrer la recherche dès que le premier document est chargé.

1. Remplacez le verbe par **GET**.

1. Définissez l’URI pour cet appel sur `https://[service name].search.windows.net/indexes/clinical-trials-json-index/docs?search=*&api-version=2020-06-30&$count=true`. Remplacez `[service name]` par le nom de votre service de recherche.

1. Envoyez la demande. Il s’agit d’une requête de recherche en texte intégral non spécifiée qui retourne tous les champs marqués comme récupérables dans l’index, ainsi qu’un nombre de documents. La réponse doit ressembler à ce qui suit :

    ```json
    {
        "@odata.context": "https://exampleurl.search.windows.net/indexes('clinical-trials-json-index')/$metadata#docs(*)",
        "@odata.count": 100,
        "value": [
            {
                "@search.score": 1.0,
                "FileName": "NCT00000102.txt",
                "MinimumAge": 14,
                "Title": "Congenital Adrenal Hyperplasia: Calcium Channels as Therapeutic Targets",
                "MyURL": "https://azure.storagedemos.com/clinical-trials/NCT00000102.txt",
                "Gender": "Both",
                "MaximumAge": 35,
                "Summary": "This study will test the ability of extended release nifedipine (Procardia XL), a blood pressure medication, to permit a decrease in the dose of glucocorticoid medication children take to treat congenital adrenal hyperplasia (CAH).",
                "NCTID": "NCT00000102",
                "Phase": "Phase 1/Phase 2",
                "Date": "ClinicalTrials.gov processed this data on October 25, 2016",
                "OverallStatus": "Completed",
                "OrgStudyId": "NCRR-M01RR01070-0506",
                "HealthyVolunteers": "No",
                "Keywords": [],
                "metadata_storage_last_modified": "2019-04-09T18:16:24Z",
                "metadata_storage_size": "33060",
                "metadata_content_type": null
            },
            . . . 
    ```

1. Ajoutez le paramètre de requête `$select` pour limiter les résultats à moins de champs : `https://[service name].search.windows.net/indexes/clinical-trials-json-index/docs?search=*&$select=Gender,metadata_storage_size&api-version=2020-06-30&$count=true`.  Pour cette requête, 100 documents correspondent, mais par défaut, Recherche cognitive Azure retourne seulement 50 dans les résultats.

   :::image type="content" source="media/search-semi-structured-data/lastquery.png" alt-text="Requête paramétrable" border="false":::

1. `$filter=MinimumAge ge 30 and MaximumAge lt 75` est un exemple de requête plus complexe qui retourne seulement les résultats pour lesquels le paramètre MinimumAge est supérieur ou égal à 30 et le paramètre MaximumAge inférieur à 75. Remplacez l’expression `$select` par l’expression `$filter`.

   :::image type="content" source="media/search-semi-structured-data/metadatashort.png" alt-text="Recherche de données semi-structurées" border="false":::

Vous pouvez aussi utiliser des opérateurs logiques (AND, OR, NOT) et des opérateurs de comparaison (eq, ne, gt, lt, ge, le). Les comparaisons de chaînes sont sensibles à la casse. Pour plus d’informations et d’exemples, consultez [Créer une requête simple](search-query-simple-examples.md).

> [!NOTE]
> Le paramètre `$filter` fonctionne uniquement avec des métadonnées qui ont été marquées comme « filtrables » lors de la création de l’index.

## <a name="reset-and-rerun"></a>Réinitialiser et réexécuter

Dans les premières étapes expérimentales du développement, l’approche la plus pratique pour les itérations de conception consiste à supprimer les objets de Recherche cognitive Azure et à autoriser votre code à les reconstruire. Les noms des ressources sont uniques. La suppression d’un objet vous permet de le recréer en utilisant le même nom.

Vous pouvez utiliser le portail pour supprimer les index, les indexeurs et les sources de données. Vous pouvez aussi utiliser **DELETE** et fournir des URL vers chaque objet. La commande suivante supprime un indexeur.

```http
DELETE https://[YOUR-SERVICE-NAME].search.windows.net/indexers/clinical-trials-json-indexer?api-version=2020-06-30
```

Le code d’état 204 est retourné lorsque la suppression réussit.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous travaillez dans votre propre abonnement, il est judicieux à la fin d’un projet de supprimer les ressources dont vous n’avez plus besoin. Les ressources laissées en cours d’exécution peuvent vous coûter de l’argent. Vous pouvez supprimer les ressources une par une, ou choisir de supprimer le groupe de ressources afin de supprimer l’ensemble des ressources.

Vous pouvez rechercher et gérer les ressources dans le portail à l’aide des liens Toutes les ressources ou Groupes de ressources situés dans le volet de navigation de gauche.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous êtes familiarisé avec les principes fondamentaux de l’indexation de Stockage Blob Azure, examinons de plus près la configuration de l’indexeur pour les objets blob JSON dans Stockage Azure.

> [!div class="nextstepaction"]
> [Configurer l’indexation d’objets blob JSON](search-howto-index-json-blobs.md)