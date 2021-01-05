---
title: Configurer un indexeur d’objets blob
titleSuffix: Azure Cognitive Search
description: Configurez un indexeur de blobs Azure pour automatiser l’indexation du contenu de blobs pour les opérations de recherche en texte intégral dans le service Recherche cognitive Azure.
manager: nitinme
author: MarkHeff
ms.author: maheff
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 09/23/2020
ms.openlocfilehash: e3419711c9a7358914f85574f6dbd5af29def1cf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91403606"
---
# <a name="how-to-configure-a-blob-indexer-in-azure-cognitive-search"></a>Comment configurer un indexeur de blobs dans Recherche cognitive Azure

Cet article explique comment utiliser la Recherche cognitive Azure pour indexer des documents texte (tels que des fichiers PDF, des documents Microsoft Office et plusieurs autres formats courants) stockés dans le Stockage Blob Azure. Tout d’abord, il présente les concepts de base de la définition et de la configuration d’un indexeur d’objets blob. Ensuite, il offre une exploration plus approfondie des comportements et des scénarios que vous êtes susceptible de rencontrer.

<a name="SupportedFormats"></a>

## <a name="supported-formats"></a>Formats pris en charge

L’indexeur d’objets blob peut extraire du texte à partir des formats de document suivants :

[!INCLUDE [search-blob-data-sources](../../includes/search-blob-data-sources.md)]

## <a name="set-up-blob-indexing"></a>Configurer l’indexation de blobs

Vous pouvez configurer un indexeur de Stockage Blob Azure avec les outils suivants :

* [Azure portal](https://ms.portal.azure.com)
* [API REST](/rest/api/searchservice/Indexer-operations) de Recherche cognitive Azure
* [Kit de développement logiciel (SDK) .NET](/dotnet/api/overview/azure/search) de Recherche cognitive Azure

> [!NOTE]
> Certaines fonctionnalités (par exemple, les mappages de champs) ne sont pas encore disponibles dans le portail et doivent être utilisées par l’intermédiaire de programmes.
>

Ici, nous vous présentons le flux à l’aide de l’API REST.

### <a name="step-1-create-a-data-source"></a>Étape 1 : Création d'une source de données

Une source de données spécifie les données à indexer, les informations d’identification nécessaires pour accéder aux données et les stratégies qui identifient efficacement les changements dans les données (telles que des lignes modifiées ou supprimées). Une source de données peut être utilisée par plusieurs indexeurs dans le même service de recherche.

Pour l’indexation des objets blob, la source de données doit avoir les propriétés requises suivantes :

* **name** est le nom unique de la source de données au sein de votre service de recherche.
* **type** doit être `azureblob`.
* **les informations d’identification fournissent la chaîne de connexion de compte de stockage en tant que paramètre `credentials.connectionString`. Pour plus d’informations, consultez [Comment spécifier des informations d’identification](#Credentials) ci-dessous.
* **container** spécifie un conteneur dans votre compte de stockage. Par défaut, tous les objets blob du conteneur sont récupérables. Si vous souhaitez indexer uniquement les objets blob dans un répertoire virtuel particulier, vous pouvez spécifier ce répertoire à l’aide du paramètre facultatif **query**.

Pour créer une source de données :

```http
    POST https://[service name].search.windows.net/datasources?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
        "name" : "blob-datasource",
        "type" : "azureblob",
        "credentials" : { "connectionString" : "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>;" },
        "container" : { "name" : "my-container", "query" : "<optional-virtual-directory-name>" }
    }
```

Pour plus d’informations sur l’API Créer une source de données, consultez [Créer une source de données](/rest/api/searchservice/create-data-source).

<a name="Credentials"></a>

#### <a name="how-to-specify-credentials"></a>Comment spécifier des informations d’identification

Vous pouvez fournir les informations d’identification du conteneur d’objets blob de l’une des manières suivantes :

* **Chaîne de connexion d’identité managée** : `ResourceId=/subscriptions/<your subscription ID>/resourceGroups/<your resource group name>/providers/Microsoft.Storage/storageAccounts/<your storage account name>/;` 

  Cette chaîne de connexion ne nécessite pas de clé de compte, mais vous devez suivre les instructions fournies dans [Configurer une connexion à un compte Stockage Azure à l’aide d’une identité managée](search-howto-managed-identities-storage.md).

* **Chaîne de connexion au compte de stockage avec accès complet** : `DefaultEndpointsProtocol=https;AccountName=<your storage account>;AccountKey=<your account key>`

  Vous pouvez obtenir la chaîne de connexion sur le portail Azure en sélectionnant le panneau du compte de stockage > Paramètres > Clés (pour les comptes de stockage Classic) ou en sélectionnant Paramètres > Clés d’accès (pour les comptes de stockage ARM).

* Chaîne de connexion de la **signature d’accès partagé (SAP) au compte de stockage** : `BlobEndpoint=https://<your account>.blob.core.windows.net/;SharedAccessSignature=?sv=2016-05-31&sig=<the signature>&spr=https&se=<the validity end time>&srt=co&ss=b&sp=rl`

  La SAP doit avoir les autorisations de liste et de lecture sur les conteneurs et les objets (blob en l’occurrence).

* **Signature d’accès partagé du conteneur** : `ContainerSharedAccessUri=https://<your storage account>.blob.core.windows.net/<container name>?sv=2016-05-31&sr=c&sig=<the signature>&se=<the validity end time>&sp=rl`

  La SAP doit avoir les autorisations de liste et lecture sur le conteneur.

Pour plus d’informations sur les signatures d’accès partagé au stockage, consultez [Utilisation des signatures d’accès partagé](../storage/common/storage-sas-overview.md).

> [!NOTE]
> Si vous utilisez des informations d’identification d’une SAP, vous devez mettre à jour les informations d’identification de la source de données régulièrement avec des signatures renouvelées afin d’éviter leur expiration. Si les informations d’identification de la SAP expirent, l’indexeur se bloque et affiche un message d’erreur similaire à `Credentials provided in the connection string are invalid or have expired.`.  

### <a name="step-2-create-an-index"></a>Étape 2 : Création d'un index

L’index spécifie les champs d’un document, les attributs et d’autres constructions qui façonnent l’expérience de recherche.

Voici comment créer un index avec un champ `content` pouvant faire l'objet d'une recherche afin de stocker le texte extrait d'objets blob :

```http
    POST https://[service name].search.windows.net/indexes?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
          "name" : "my-target-index",
          "fields": [
            { "name": "id", "type": "Edm.String", "key": true, "searchable": false },
            { "name": "content", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false }
          ]
    }
```

Pour plus d’informations, consultez [Créer un index (API REST)](/rest/api/searchservice/create-index).

### <a name="step-3-create-an-indexer"></a>Étape 3 : Créer un indexeur

Un indexeur connecte une source de données à un index de recherche cible et fournit une planification afin d’automatiser l’actualisation des données.

Une fois l'index et la source de données créés, vous êtes prêt à créer l’indexeur :

```http
    POST https://[service name].search.windows.net/indexers?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
      "name" : "blob-indexer",
      "dataSourceName" : "blob-datasource",
      "targetIndexName" : "my-target-index",
      "schedule" : { "interval" : "PT2H" }
    }
```

Cet indexeur s’exécutera toutes les deux heures (intervalle de planification défini sur « PT2H »). Pour exécuter un indexeur toutes les 30 minutes, définissez l’intervalle sur « PT30M ». Le plus court intervalle pris en charge est de 5 minutes. La planification est facultative : en cas d’omission, un indexeur ne s’exécute qu’une seule fois lorsqu’il est créé. Toutefois, vous pouvez à tout moment exécuter un indexeur à la demande.   

Pour plus d’informations, consultez [Créer un indexeur (API REST)](/rest/api/searchservice/create-indexer). Pour plus d’informations sur la définition des planifications de l’indexeur, consultez [Comment planifier des indexeurs pour la Recherche cognitive Azure](search-howto-schedule-indexers.md).

<a name="how-azure-search-indexes-blobs"></a>

## <a name="how-blobs-are-indexed"></a>Comment les blobs sont indexés

En fonction de sa [configuration](#PartsOfBlobToIndex), l’indexeur d’objets blob peut indexer uniquement les métadonnées de stockage (une fonctionnalité utile lorsque vous ne vous préoccupez que des métadonnées et n’avez pas besoin d’indexer le contenu des objets blob), le stockage et le contenu des métadonnées, ou les métadonnées et le contenu textuel. Par défaut, l’indexeur extrait les métadonnées et le contenu.

> [!NOTE]
> Par défaut, les objets blob avec contenu structuré tels que JSON ou CSV sont indexés en tant que bloc de texte unique. Si vous souhaitez indexer des objets blob JSON et CSV de manière structurée, consultez [Indexation d’objets blob JSON](search-howto-index-json-blobs.md) et [Indexation d’objets blob CSV](search-howto-index-csv-blobs.md) pour en savoir plus.
>
> Un document composé ou incorporé (tel qu’une archive ZIP ou un document Word avec e-mail Outlook incorporé intégrant des pièces jointes, ou un fichier .MSG avec des pièces jointes) est également indexé en tant que document unique. Par exemple, toutes les images extraites des pièces jointes d’un fichier .MSG seront renvoyées dans le champ normalized_images.

* Le contenu de texte du document est extrait dans un champ de chaîne nommé `content`.

  > [!NOTE]
  > La Recherche cognitive Azure limite la quantité de texte extrait en fonction du niveau tarifaire : 32 000 caractères pour le niveau Gratuit, 64 000 pour le niveau De base, 4 millions pour le niveau Standard, 8 millions pour le niveau Standard S2 et 16 millions pour le niveau Standard S3. Un avertissement est inclus dans la réponse d’état de l’indexeur pour les documents tronqués.  

* Les propriétés de métadonnées spécifiées par l’utilisateur qui sont éventuellement présentes dans l’objet blob sont extraites textuellement. Notez que cela nécessite la définition d’un champ dans l’index portant le même nom que la clé de métadonnées du BLOB. Par exemple, si votre BLOB a une clé de métadonnées de `Sensitivity` avec la valeur `High`, vous devez définir un champ nommé `Sensitivity` dans votre index de recherche et il sera rempli avec la valeur `High`.

* Les propriétés de métadonnées d’objet blob standard sont extraites dans les champs suivants :

  * **metadata\_storage\_name** (Edm.String) : nom de fichier de l’objet blob. Par exemple, si vous disposez de l’objet blob /my-container/my-folder/subfolder/resume.pdf, ce champ présente la valeur `resume.pdf`.

  * **metadata\_storage\_path** (Edm.String) : URI complet de l’objet blob, incluant le compte de stockage. Par exemple : `https://myaccount.blob.core.windows.net/my-container/my-folder/subfolder/resume.pdf`

  * **metadata\_storage\_content\_type** (Edm.String) : type de contenu tel que spécifié par le code que vous avez utilisé pour charger l’objet blob. Par exemple : `application/octet-stream`.

  * **metadata\_storage\_last\_modified** (Edm.DateTimeOffset) : horodateur de la dernière modification de l’objet blob. La Recherche cognitive Azure utilise cet horodateur pour identifier les objets blob modifiés afin d’éviter une réindexation complète après l’indexation initiale.

  * **metadata\_storage\_size** (Edm.Int64) : taille de l’objet blob en octets.

  * **metadata\_storage\_content\_md5** (Edm.String) : code de hachage MD5 du contenu de l’objet blob s’il est disponible.

  * **metadata\_storage\_sas\_token** (Edm.String) : jeton SAS temporaire qui peut être utilisé par des [compétences personnalisées](cognitive-search-custom-skill-interface.md) pour accéder à l’objet blob. Ce jeton ne doit pas être stocké pour une utilisation ultérieure dans la mesure où il peut expirer.

* Les propriétés de métadonnées propres à chaque format de document sont extraites dans les champs répertoriés [ici](#ContentSpecificMetadata).

Vous n’avez pas besoin de définir les champs relatifs à chacune des propriétés ci-dessus dans votre index de recherche. Il vous suffit de capturer les propriétés dont vous devez disposer pour votre application.

> [!NOTE]
> Les noms de champ figurant dans votre index existant diffèrent généralement des noms de champ générés lors de l’extraction de document. Dans ce cas, vous pouvez utiliser les **mappages de champs** pour mapper les noms de propriétés fournis par la Recherche cognitive Azure sur les noms de champs de votre index de recherche. Découvrez ci-dessous une exemple d’utilisation de mappage de champ.
>

<a name="DocumentKeys"></a>

### <a name="defining-document-keys-and-field-mappings"></a>Définition des clés de document et des mappages de champs

Dans la Recherche cognitive Azure, la clé de document identifie un document de manière unique. Chaque index de recherche doit comporter exactement un champ de clé de type Edm.String. Ce champ de clé est nécessaire pour chaque document ajouté à l’index (il constitue en fait le seul champ obligatoire).  

Vous devez déterminer avec soin le champ extrait que vous souhaitez mapper sur le champ de clé de votre index. Les candidats sont les suivants :

* **metadata\_storage\_name** : ce champ pourrait se révéler un choix commode, mais notez que (1) les noms ne sont pas forcément uniques, car vous pouvez disposer d’objets blob portant le même nom dans différents dossiers, et (2) le nom peut contenir des caractères qui ne sont pas valides dans les clés de document, comme des tirets. Vous pouvez gérer les caractères non valides en utilisant la [fonction de mappage de champs](search-indexer-field-mappings.md#base64EncodeFunction) `base64Encode`. Dans ce cas, pensez à encoder les clés de documents lorsque vous les transmettez dans des appels d’API, comme l’API Lookup. (Par exemple, dans .NET, vous pouvez utiliser la [méthode UrlTokenEncode](/dotnet/api/system.web.httpserverutility.urltokenencode) à cet effet).

* **metadata\_storage\_path** : l’utilisation du chemin d’accès complet garantit l’unicité, mais le chemin d’accès contient invariablement des caractères `/` qui ne sont [pas valides dans une clé de document](/rest/api/searchservice/naming-rules).  Comme ci-dessus, vous avez la possibilité d’encoder les clés à l’aide de la [fonction](search-indexer-field-mappings.md#base64EncodeFunction) `base64Encode`.

* Une troisième option consiste à ajouter une propriété de métadonnées personnalisée aux blobs. Toutefois, cette option contraint votre processus de chargement de blob à ajouter cette propriété de métadonnées à tous les blobs. Étant donné que la clé est une propriété obligatoire, tous les objets blob dépourvus de cette propriété ne seront pas indexés.

> [!IMPORTANT]
> En l’absence de mappage explicite pour le champ de clé dans l’index, la Recherche cognitive Azure utilise automatiquement `metadata_storage_path` en guise de clé et encode les valeurs de clés en base 64 (la deuxième option ci-dessus).
>
>

Pour cet exemple, sélectionnons le champ `metadata_storage_name` en tant que clé de document. Supposons également que votre index comporte un champ de clé nommé `key` et un champ `fileSize` pour le stockage de la taille du document. Pour obtenir le résultat souhaité, spécifiez les mappages de champs ci-après lors de la création ou de la mise à jour de votre indexeur :

```http
    "fieldMappings" : [
      { "sourceFieldName" : "metadata_storage_name", "targetFieldName" : "key", "mappingFunction" : { "name" : "base64Encode" } },
      { "sourceFieldName" : "metadata_storage_size", "targetFieldName" : "fileSize" }
    ]
```

Pour regrouper tous ces éléments, utilisez le code ci-après pour ajouter des mappages de champs et activer le codage base 64 des clés pour un indexeur existant :

```http
    PUT https://[service name].search.windows.net/indexers/blob-indexer?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
      "dataSourceName" : " blob-datasource ",
      "targetIndexName" : "my-target-index",
      "schedule" : { "interval" : "PT2H" },
      "fieldMappings" : [
        { "sourceFieldName" : "metadata_storage_name", "targetFieldName" : "key", "mappingFunction" : { "name" : "base64Encode" } },
        { "sourceFieldName" : "metadata_storage_size", "targetFieldName" : "fileSize" }
      ]
    }
```

Pour plus d’informations, consultez [Mappages et transformations de champs](search-indexer-field-mappings.md).

#### <a name="what-if-you-need-to-encode-a-field-to-use-it-as-a-key-but-you-also-want-to-search-it"></a>Que se passe-t-il si vous devez encoder un champ à utiliser comme clé, que vous souhaitez également pouvoir rechercher ?

Il peut arriver que vous deviez utiliser une version encodée d’un champ tel que metadata_storage_path en tant que clé, et que vous ayez également besoin de pouvoir rechercher ce champ (sans encodage). Pour résoudre ce dilemme, vous pouvez mapper ce champ à deux champs, l’un étant utilisé pour la clé, et l’autre à des fins de recherche. Dans l’exemple ci-dessous, le champ *key* (clé) contient le chemin d’accès encodé, tandis que le champ *path* (chemin) n’est pas encodé et sera utilisé en tant que champ pouvant faire l’objet d’une recherche dans l’index.

```http
    PUT https://[service name].search.windows.net/indexers/blob-indexer?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
      "dataSourceName" : " blob-datasource ",
      "targetIndexName" : "my-target-index",
      "schedule" : { "interval" : "PT2H" },
      "fieldMappings" : [
        { "sourceFieldName" : "metadata_storage_path", "targetFieldName" : "key", "mappingFunction" : { "name" : "base64Encode" } },
        { "sourceFieldName" : "metadata_storage_path", "targetFieldName" : "path" }
      ]
    }
```
<a name="WhichBlobsAreIndexed"></a>

## <a name="index-by-file-type"></a>Indexer par type de fichier

Vous pouvez contrôler les objets BLOB qui sont indexés et ignorés.

### <a name="include-blobs-having-specific-file-extensions"></a>Inclure les blobs ayant des extensions de fichier spécifiques

Vous pouvez indexer uniquement les objets blob avec des extensions de nom de fichier que vous spécifiez à l’aide du paramètre de configuration d’indexeur `indexedFileNameExtensions`. La valeur est une chaîne contenant une liste d'extensions de fichier séparées par des virgules (précédées d'un point). Par exemple, pour indexer uniquement les objets blob .PDF et .DOCX, procédez comme suit :

```http
    PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
      ... other parts of indexer definition
      "parameters" : { "configuration" : { "indexedFileNameExtensions" : ".pdf,.docx" } }
    }
```

### <a name="exclude-blobs-having-specific-file-extensions"></a>Exclure les blobs ayant des extensions de fichier spécifiques

Vous pouvez exclure de l’indexation des objets blob avec des extensions de nom de fichier spécifiques à l’aide du paramètre de configuration `excludedFileNameExtensions`. La valeur est une chaîne contenant une liste d'extensions de fichier séparées par des virgules (précédées d'un point). Par exemple, pour indexer tous les objets blob, sauf ceux qui ont les extensions .PNG et .JPEG, procédez comme suit :

```http
    PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
      ... other parts of indexer definition
      "parameters" : { "configuration" : { "excludedFileNameExtensions" : ".png,.jpeg" } }
    }
```

Si les paramètres `indexedFileNameExtensions` et `excludedFileNameExtensions` sont tous deux présents, la Recherche cognitive Azure regarde d’abord `indexedFileNameExtensions`, puis `excludedFileNameExtensions`. Cela signifie que, si la même extension de fichier est présente dans les deux listes, elle sera exclue de l'indexation.

<a name="PartsOfBlobToIndex"></a>

## <a name="index-parts-of-a-blob"></a>Indexer des parties d’un blob

Vous pouvez contrôler les parties des objets blob à indexer à l’aide du paramètre de configuration `dataToExtract`. Il peut avoir les valeurs suivantes :

* `storageMetadata` : spécifie que seuls les propriétés standard [ et les métadonnées ](../storage/blobs/storage-blob-container-properties-metadata.md) spécifiées par l’utilisateur sont indexés.
* `allMetadata` : spécifie que les métadonnées de stockage et les [métadonnées spécifiques du type de contenu](#ContentSpecificMetadata) extraites du contenu des objets blob sont indexés.
* `contentAndMetadata` : spécifie que toutes les métadonnées et tous les contenus textuels extraits de l’objet blob sont indexés. Il s’agit de la valeur par défaut.

Par exemple, pour indexer uniquement les métadonnées de stockage, utilisez :

```http
    PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
      ... other parts of indexer definition
      "parameters" : { "configuration" : { "dataToExtract" : "storageMetadata" } }
    }
```

### <a name="using-blob-metadata-to-control-how-blobs-are-indexed"></a>Utilisation de métadonnées d’objets blob pour contrôler la manière dont les objets blob sont indexés

Les paramètres de configuration décrits ci-dessus s’appliquent à tous les objets blob. Parfois, vous pouvez souhaiter contrôler la manière dont *différents objets blob* sont indexés. Pour cela, vous pouvez ajouter les propriétés et valeurs de métadonnées d’objets blob suivantes :

| Nom de la propriété | Valeur de la propriété | Explication |
| --- | --- | --- |
| AzureSearch_Skip |"true" |Indique à l’indexeur d’objets blob d’ignorer complètement l’objet blob. Ni l’extraction des métadonnées, ni l’extraction de contenu n’est tentée. Cette propriété est utile lorsqu’un objet blob spécifique échoue à plusieurs reprises et interrompt le processus d’indexation. |
| AzureSearch_SkipContent |"true" |Équivaut au réglage `"dataToExtract" : "allMetadata"` décrit [ci-dessus](#PartsOfBlobToIndex) au niveau d’un objet blob donné. |

## <a name="index-from-multiple-sources"></a>Indexer à partir de plusieurs sources

Vous pourriez souhaiter « rassembler » des documents provenant de plusieurs sources dans votre index. Par exemple, vous pourriez souhaiter fusionner des textes de blobs avec d’autres métadonnées stockées dans la base de données Cosmos. Vous pouvez même utiliser le push de l’indexation des API ainsi que plusieurs indexeurs pour générer des documents de recherche à partir de plusieurs parties.

Pour ce faire, tous les indexeurs et les autres composants doivent s’accorder sur la clé de document. Pour plus d’informations à ce sujet, consultez [Indexer plusieurs sources de données Azure](./tutorial-multiple-data-sources.md) ou le billet de blog [Combine documents with other data in Azure Search](https://blog.lytzen.name/2017/01/combine-documents-with-other-data-in.html) (Associer des documents à d’autres données dans Recherche cognitive Azure).

## <a name="index-large-datasets"></a>Indexer des jeux de données volumineux

L’indexation d’objets blob peut être un processus long. Dans le cas où vous avez des millions d’objets blob à indexer, vous pouvez accélérer l’indexation en partitionnant les données et en utilisant plusieurs indexeurs pour traiter les données en parallèle. Par exemple, vous pouvez effectuer la configuration suivante :

* Partitionnez les données dans plusieurs conteneurs d’objets blob ou des dossiers virtuels.

* Configurez plusieurs sources de données de Recherche cognitive Azure, une par conteneur ou dossier. Pour pointer vers un dossier d’objets blob, utilisez le paramètre `query` :

    ```json
    {
        "name" : "blob-datasource",
        "type" : "azureblob",
        "credentials" : { "connectionString" : "<your storage connection string>" },
        "container" : { "name" : "my-container", "query" : "my-folder" }
    }
    ```

* Créez un indexeur correspondant pour chaque source de données. Tous les indexeurs peuvent pointer vers le même index de recherche cible.  

* Une unité de recherche dans votre service peut exécuter un indexeur à tout moment donné. La création de plusieurs indexeurs comme décrit ci-dessus est utile uniquement s’ils s’exécutent en parallèle. Pour exécuter plusieurs indexeurs en parallèle, effectuez un scale-out de votre service de recherche en créant un nombre approprié de partitions et réplicas. Par exemple, si votre service de recherche a 6 unités de recherche (2 partitions x 3 réplicas), 6 indexeurs peuvent s’exécuter simultanément, ce qui augmente le débit d’indexation par six. Pour en savoir plus sur la mise à l’échelle et la planification de capacité, consultez [Ajuster la capacité d’un service Recherche cognitive Azure](search-capacity-planning.md).

<a name="DealingWithErrors"></a>

## <a name="handle-errors"></a>des erreurs

Par défaut, l’indexeur d’objets blob s’arrête dès qu’il rencontre un objet blob avec un type de contenu non pris en charge (par exemple, une image). Vous pouvez évidemment utiliser le paramètre `excludedFileNameExtensions` pour ignorer certains types de contenu. Toutefois, vous devrez peut-être indexer des objets blob sans connaître à l’avance tous les types de contenu possibles. Pour poursuivre l’indexation lorsqu’un type de contenu non pris en charge est détecté, définissez le paramètre de configuration `failOnUnsupportedContentType` sur `false` :

```http
    PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
      ... other parts of indexer definition
      "parameters" : { "configuration" : { "failOnUnsupportedContentType" : false } }
    }
```

Pour certains objets blob, le service Recherche cognitive Azure ne parvient pas à déterminer le type de contenu ou à traiter un document avec un autre type de contenu pris en charge. Pour ignorer ce mode d’échec, définissez le paramètre de configuration `failOnUnprocessableDocument` sur false :

```http
      "parameters" : { "configuration" : { "failOnUnprocessableDocument" : false } }
```

La Recherche cognitive Azure limite la taille des objets blob indexés. Ces limites sont documentées dans [Limites de service de Recherche cognitive Azure](./search-limits-quotas-capacity.md). Par défaut, les objets blob surdimensionnés sont traités comme des erreurs. Toutefois, vous pouvez toujours indexer des métadonnées de stockage d’objets blob surdimensionnés en définissant la valeur du paramètre configuration `indexStorageMetadataOnlyForOversizedDocuments` sur true :

```http
    "parameters" : { "configuration" : { "indexStorageMetadataOnlyForOversizedDocuments" : true } }
```

Vous pouvez également poursuivre l’indexation si des erreurs se produisent à tout moment du traitement, que ce soit durant l’analyse d’objets blob ou l’ajout de documents à un index. Pour ignorer un nombre spécifique d’erreurs, définissez les paramètres de configuration `maxFailedItems` et `maxFailedItemsPerBatch` sur les valeurs souhaitées. Par exemple :

```http
    {
      ... other parts of indexer definition
      "parameters" : { "maxFailedItems" : 10, "maxFailedItemsPerBatch" : 10 }
    }
```

<a name="ContentSpecificMetadata"></a>

## <a name="content-type-specific-metadata-properties"></a>Propriétés de métadonnées propres au type de contenu

Le tableau ci-après récapitule le traitement appliqué pour chaque format de document et décrit les propriétés de métadonnées extraites par la Recherche cognitive Azure.

| Format de document/type de contenu | Métadonnées extraites | Détails du traitement |
| --- | --- | --- |
| HTML (texte/html) |`metadata_content_encoding`<br/>`metadata_content_type`<br/>`metadata_language`<br/>`metadata_description`<br/>`metadata_keywords`<br/>`metadata_title` |Suppression du balisage HTML et extraction du texte |
| PDF (application/pdf) |`metadata_content_type`<br/>`metadata_language`<br/>`metadata_author`<br/>`metadata_title` |Extraction du texte, y compris les documents incorporés (à l’exclusion des images) |
| DOCX (application/vnd.openxmlformats-officedocument.wordprocessingml.document) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_character_count`<br/>`metadata_creation_date`<br/>`metadata_last_modified`<br/>`metadata_page_count`<br/>`metadata_word_count` |Extraction du texte, y compris les documents incorporés |
| DOC (application/msword) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_character_count`<br/>`metadata_creation_date`<br/>`metadata_last_modified`<br/>`metadata_page_count`<br/>`metadata_word_count` |Extraction du texte, y compris les documents incorporés |
| DOCM (application/vnd.ms-word.document.macroenabled.12) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_character_count`<br/>`metadata_creation_date`<br/>`metadata_last_modified`<br/>`metadata_page_count`<br/>`metadata_word_count` |Extraction du texte, y compris les documents incorporés |
| WORD XML (application/vnd.ms-word2006ml) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_character_count`<br/>`metadata_creation_date`<br/>`metadata_last_modified`<br/>`metadata_page_count`<br/>`metadata_word_count` |Suppression du balisage XML et extraction du texte |
| WORD 2003 XML (application/vnd.ms-wordml) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_creation_date` |Suppression du balisage XML et extraction du texte |
| XLSX (application/vnd.openxmlformats-officedocument.spreadsheetml.sheet) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_creation_date`<br/>`metadata_last_modified` |Extraction du texte, y compris les documents incorporés |
| XLS (application/vnd.ms-excel) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_creation_date`<br/>`metadata_last_modified` |Extraction du texte, y compris les documents incorporés |
| XLSM (application/vnd.ms-excel.sheet.macroenabled.12) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_creation_date`<br/>`metadata_last_modified` |Extraction du texte, y compris les documents incorporés |
| PPTX (application/vnd.openxmlformats-officedocument.presentationml.presentation) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_creation_date`<br/>`metadata_last_modified`<br/>`metadata_slide_count`<br/>`metadata_title` |Extraction du texte, y compris les documents incorporés |
| PPT (application/vnd.ms-powerpoint) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_creation_date`<br/>`metadata_last_modified`<br/>`metadata_slide_count`<br/>`metadata_title` |Extraction du texte, y compris les documents incorporés |
| PPTM (application/vnd.ms-powerpoint.presentation.macroenabled.12) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_creation_date`<br/>`metadata_last_modified`<br/>`metadata_slide_count`<br/>`metadata_title` |Extraction du texte, y compris les documents incorporés |
| MSG (application/vnd.ms-outlook) |`metadata_content_type`<br/>`metadata_message_from`<br/>`metadata_message_from_email`<br/>`metadata_message_to`<br/>`metadata_message_to_email`<br/>`metadata_message_cc`<br/>`metadata_message_cc_email`<br/>`metadata_message_bcc`<br/>`metadata_message_bcc_email`<br/>`metadata_creation_date`<br/>`metadata_last_modified`<br/>`metadata_subject` |Extrayez le texte, y compris celui des pièces jointes. `metadata_message_to_email`, `metadata_message_cc_email` et `metadata_message_bcc_email` sont des collections de chaînes ; les autres champs sont des chaînes.|
| ODT (application/vnd.oasis.opendocument.text) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_character_count`<br/>`metadata_creation_date`<br/>`metadata_last_modified`<br/>`metadata_page_count`<br/>`metadata_word_count` |Extraction du texte, y compris les documents incorporés |
| ODS (application/vnd.oasis.opendocument.spreadsheet) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_creation_date`<br/>`metadata_last_modified` |Extraction du texte, y compris les documents incorporés |
| ODP (application/vnd.oasis.opendocument.presentation) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_creation_date`<br/>`metadata_last_modified`<br/>`title` |Extraction du texte, y compris les documents incorporés |
| ZIP (application/zip) |`metadata_content_type` |Extraction du texte de tous les documents figurant dans l’archive |
| GZ (application/gzip) |`metadata_content_type` |Extraction du texte de tous les documents figurant dans l’archive |
| EPUB (application/epub+zip) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_creation_date`<br/>`metadata_title`<br/>`metadata_description`<br/>`metadata_language`<br/>`metadata_keywords`<br/>`metadata_identifier`<br/>`metadata_publisher` |Extraction du texte de tous les documents figurant dans l’archive |
| XML (application/xml) |`metadata_content_type`<br/>`metadata_content_encoding`<br/> |Suppression du balisage XML et extraction du texte |
| JSON (application/json) |`metadata_content_type`<br/>`metadata_content_encoding` |Extraction du texte<br/>REMARQUE :  si vous devez extraire plusieurs champs de document à partir d’un objet blob JSON, consultez [Indexation d’objets blob JSON](search-howto-index-json-blobs.md) pour plus de détails |
| EML (message/rfc822) |`metadata_content_type`<br/>`metadata_message_from`<br/>`metadata_message_to`<br/>`metadata_message_cc`<br/>`metadata_creation_date`<br/>`metadata_subject` |Extraction du texte, y compris les pièces jointes |
| RTF (application/rtf) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_character_count`<br/>`metadata_creation_date`<br/>`metadata_page_count`<br/>`metadata_word_count`<br/> | Extraction du texte|
| Texte brut (text/plain) |`metadata_content_type`<br/>`metadata_content_encoding`<br/> | Extraction du texte|

## <a name="see-also"></a>Voir également

* [Indexeurs dans Recherche cognitive Azure](search-indexer-overview.md)
* [Comprendre les blobs avec l’intelligence artificielle](search-blob-ai-integration.md)
* [Vue d’ensemble de l’indexation de blobs](search-blob-storage-integration.md)