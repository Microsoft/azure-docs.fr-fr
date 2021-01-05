---
title: Effectuer des recherches dans les données Azure Cosmos DB
titleSuffix: Azure Cognitive Search
description: Importez des données à partir d’Azure Cosmos DB dans un index pouvant faire l’objet d’une recherche dans Recherche cognitive Azure. Les indexeurs automatisent l’ingestion des données pour les sources de données sélectionnées telles qu’Azure Cosmos DB.
author: mgottein
manager: nitinme
ms.author: magottei
ms.devlang: rest-api
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 07/11/2020
ms.openlocfilehash: be7c6ec9dbc577143e6c7219580f42c876f536bc
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96499966"
---
# <a name="how-to-index-cosmos-db-data-using-an-indexer-in-azure-cognitive-search"></a>Guide pratique pour indexer des données Cosmos DB avec un indexeur dans Recherche cognitive Azure 

> [!IMPORTANT] 
> L’API SQL est mise à la disposition générale.
> La prise en charge de l’API MongoDB, de l’API Gremlin et de l’API Cassandra est actuellement en préversion publique. Les fonctionnalités en préversion sont fournies sans contrat de niveau de service et ne sont pas recommandées pour les charges de travail de production. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Vous pouvez demander l’accès aux préversions en remplissant [ce formulaire](https://aka.ms/azure-cognitive-search/indexer-preview). 
> [Les préversions de l’API REST](search-api-preview.md) offrent ces fonctionnalités. La prise en charge du portail est actuellement limitée, et il n’existe pas de prise en charge du kit SDK .NET.

> [!WARNING]
> Seuls les collections Cosmos DB dotées d’une [stratégie d’indexation](../cosmos-db/index-policy.md) configurée sur [Cohérent](../cosmos-db/index-policy.md#indexing-mode) sont prises en charge par Recherche cognitive Azure. L’indexation des collections dotées d’une stratégie d’indexation différée n’est pas recommandée et peut se traduire par des données manquantes. Les collections dont l’indexation est désactivée ne sont pas prises en charge.

Cet article vous montre comment configurer l’[indexeur](search-indexer-overview.md) Azure Cosmos DB pour extraire le contenu et les rendre détectables dans Recherche cognitive Azure. Ce flux de travail crée un index Recherche cognitive Azure et le charge avec le texte existant extrait d’Azure Cosmos DB. 

Étant donné que la terminologie peut prêter à confusion, il est important de souligner que l’[indexation Azure Cosmos DB](../cosmos-db/index-overview.md) et l’[indexation Recherche cognitive Azure](search-what-is-an-index.md) sont des opérations distinctes, propres à chaque service. Avant de commencer l’indexation Recherche cognitive Azure, votre base de données Azure Cosmos DB doit déjà exister et contenir des données.

L’indexeur Cosmos DB dans Recherche cognitive Azure peut analyser les [éléments d’Azure Cosmos DB](../cosmos-db/account-databases-containers-items.md#azure-cosmos-items) accessibles via différents protocoles. 

+ Pour l’[API SQL](../cosmos-db/sql-query-getting-started.md), qui est en disponibilité générale, vous pouvez utiliser le [portail](#cosmos-indexer-portal), l’[API REST](/rest/api/searchservice/indexer-operations) ou le [Kit de développement logiciel (SDK) .NET](/dotnet/api/azure.search.documents.indexes.models.searchindexer) pour créer la source de données et l’indexeur.

+ Pour l’[API MongoDB (préversion)](../cosmos-db/mongodb-introduction.md), vous pouvez utiliser le [portail](#cosmos-indexer-portal) ou l’[API REST version 2020-06-30-Preview](search-api-preview.md) pour créer la source de données et l’indexeur.

+ Pour l’[API Cassandra (préversion)](../cosmos-db/cassandra-introduction.md) et l’[API Gremlin (préversion)](../cosmos-db/graph-introduction.md), vous pouvez uniquement utiliser l’[API REST version 2020-06-30-Preview](search-api-preview.md) pour créer la source de données et l’indexeur.


> [!Note]
> Vous pouvez voter sur User Voice pour l’[API Table](https://feedback.azure.com/forums/263029-azure-search/suggestions/32759746-azure-search-should-be-able-to-index-cosmos-db-tab) si vous souhaitez la voir prise en charge dans Recherche cognitive Azure.
>

<a name="cosmos-indexer-portal"></a>

## <a name="use-the-portal"></a>Utiliser le portail

> [!Note]
> Le portail prend actuellement en charge l’API SQL et l’API MongoDB (préversion).

La méthode la plus simple pour l’indexation d’éléments Azure Cosmos DB consiste à utiliser un Assistant dans le [portail Azure](https://portal.azure.com/). En analysant les données et en lisant les métadonnées dans le conteneur, l’Assistant [**Importation de données**](search-import-data-portal.md) peut créer un index par défaut, mapper des champs sources aux champs d’index cibles et charger l’index en une seule opération. Selon la taille et la complexité de la source de données, vous pouvez obtenir un index de recherche en texte intégral opérationnel en quelques minutes.

Nous vous recommandons d’utiliser la même région ou le même emplacement pour Recherche cognitive Azure et Azure Cosmos DB pour bénéficier d’une latence plus faible et éviter des frais de bande passante.

### <a name="1---prepare-source-data"></a>1 - Préparez les données sources

Vous devez disposer d’un compte Cosmos DB, d’une base de données Azure Cosmos DB mappée à l’API SQL, à l’API MongoDB (préversion) ou à l’API Gremlin (préversion), et d’un contenu dans la base de données.

Assurez-vous que votre base de données Cosmos DB contient des données. L’[Assistant Importation de données](search-import-data-portal.md) lit les métadonnées et effectue un échantillonnage des données pour déduire un schéma d’index, mais il charge également des données à partir de Cosmos DB. Si les données sont manquantes, l’Assistant s’arrête avec l’erreur « Erreur lors de la détection du schéma d’index à partir de la source de données : Impossible de générer un index prototype, car la source de données 'emptycollection' n’a retourné aucune donnée ».

### <a name="2---start-import-data-wizard"></a>2 - Démarrez l’Assistant Importation de données

Vous pouvez [démarrer l’Assistant](search-import-data-portal.md) à partir de la barre de commandes dans la page du service Recherche cognitive Azure ou, si vous vous connectez à l’API SQL de Cosmos DB, vous pouvez cliquer sur **Ajouter Recherche cognitive Azure** dans la section **Paramètres** du volet de navigation gauche de votre compte Cosmos DB.

   ![Commande Importer des données dans le portail](./media/search-import-data-portal/import-data-cmd2.png "Démarrer l’Assistant Importer des données")

### <a name="3---set-the-data-source"></a>3 - Définissez la source de données

Dans la page **Source de données**, la source doit être **Cosmos DB**, avec les spécifications suivantes :

+ Le **Nom** correspond au nom de l’objet source de données. Une fois créé, vous pouvez le choisir pour d’autres charges de travail.

+ Le **compte Cosmos DB** doit être dans l'un des formats suivants :
    1. La chaîne de connexion primaire ou secondaire de Cosmos DB au format suivant : `AccountEndpoint=https://<Cosmos DB account name>.documents.azure.com;AccountKey=<Cosmos DB auth key>;`.
        + Pour les **collections MongoDB** des versions 3.2 et 3.6, utilisez le format suivant pour le compte Cosmos DB dans le portail Azure : `AccountEndpoint=https://<Cosmos DB account name>.documents.azure.com;AccountKey=<Cosmos DB auth key>;ApiKind=MongoDb`
        + Pour les **graphes Gremlin et les tables Cassandra**, inscrivez-vous à la [préversion de l’indexeur contrôlé](https://aka.ms/azure-cognitive-search/indexer-preview) pour accéder à la préversion et aux informations sur la façon de mettre en forme les informations d’identification.
    1.  Une chaîne de connexion d'identité managée au format suivant qui n'inclut pas de clé de compte : `ResourceId=/subscriptions/<your subscription ID>/resourceGroups/<your resource group name>/providers/Microsoft.DocumentDB/databaseAccounts/<your cosmos db account name>/;(ApiKind=[api-kind];)`. Pour utiliser ce format de chaîne de connexion, reportez-vous à [Configurer une connexion d'indexeur à une base de données Cosmos DB à l'aide d'une identité managée](search-howto-managed-identities-cosmos-db.md) et suivez les instructions fournies.

+ La **Base de données** correspond à une base de données existante à partir du compte. 

+ La **Collection** correspond à un conteneur de documents. Des documents doivent exister pour que l’importation réussisse. 

+ Une **Requête** peut être vide si vous souhaitez tous les documents, sinon vous pouvez saisir une requête qui sélectionne un sous-ensemble du document. Une **requête** est disponible seulement pour l’API SQL.

   ![Définition de la source de données de Cosmos DB](media/search-howto-index-cosmosdb/cosmosdb-datasource.png "Définition de la source de données de Cosmos DB")

### <a name="4---skip-the-enrich-content-page-in-the-wizard"></a>4 – Ignorer la page « Enrichir le contenu » de l’Assistant

L’ajout de compétences cognitives (ou enrichissement) n’est pas une condition d’importation. Si vous n’avez pas besoin d’[ajouter un enrichissement de l’IA](cognitive-search-concept-intro.md) à votre pipeline d’indexation, ignorez cette étape.

Pour ignorer cette étape, cliquez sur les boutons bleus au bas de la page pour « Suivant » et « Ignorer ».

### <a name="5---set-index-attributes"></a>5 - Définissez les attributs de l’index

Dans la page **Index**, vous devez voir une liste de champs avec un type de données et une série de cases à cocher permettant de définir les attributs de l’index. L’Assistant peut générer une liste de champs basée sur les métadonnées et en échantillonnant les données sources. 

Vous pouvez sélectionner des attributs en bloc en cliquant sur la case à cocher en haut de la colonne d’attribut. Choisissez **Récupérable** et **Possibilité de recherche** pour chaque champ qui doit être retourné vers une application cliente et soumis à un traitement de recherche de texte intégral. Vous remarquerez que les entiers ne peuvent pas être recherchés en texte intégral ou partiel (les nombres sont évalués textuellement et sont généralement utiles dans les filtres).

Pour plus d’informations, passez en revue la description des [attributs d’index](/rest/api/searchservice/create-index#bkmk_indexAttrib) et des [analyseurs de langage](/rest/api/searchservice/language-support). 

Prenez un moment pour passer en revue vos sélections. Une fois que vous exécutez l’Assistant, des structures de données physiques sont créées : vous ne pourrez donc plus modifier ces champs sans supprimer et recréer tous les objets.

   ![Définition de l’index Cosmos DB](media/search-howto-index-cosmosdb/cosmosdb-index-schema.png "Définition de l’index Cosmos DB")

### <a name="6---create-indexer"></a>6 - Créez un indexeur

Une fois que tout est spécifié, l’Assistant crée trois objets distincts dans votre service de recherche. Un objet source de données et un objet index sont enregistrés comme ressources nommées dans votre service Recherche cognitive Azure. La dernière étape crée un objet indexeur. Le fait de nommer l’indexeur lui permet d’exister comme ressource autonome, que vous pouvez planifier et gérer indépendamment de l’objet index et de l’objet source de données, créés dans la même séquence de l’Assistant.

Si vous n’êtes pas familiarisé avec les indexeurs, en voici une définition : un *indexeur* est une ressource dans Recherche cognitive Azure qui analyse une source de données externe et son contenu avec possibilité de recherche. La sortie de l’Assistant **Importation de données** est un indexeur qui analyse votre source de données Cosmos DB, extrait le contenu avec possibilité de recherche et l’importe dans un index sur Recherche cognitive Azure.

La capture d’écran suivante montre la configuration de l’indexeur par défaut. Vous pouvez basculer sur **Une seule fois** si vous souhaitez exécuter l’indexeur une seule fois. Cliquez sur **Envoyer** pour exécuter l’Assistant et créer tous les objets. L’indexation commence immédiatement.

   ![Définition de l’indexeur de Cosmos DB](media/search-howto-index-cosmosdb/cosmosdb-indexer.png "Définition de l’indexeur de Cosmos DB")

Vous pouvez surveiller l’importation des données dans les pages du portail. Des notifications de l’avancement indiquent l’état de l’indexation et le nombre de documents chargés. 

Quand l’indexation est terminée, vous pouvez utiliser l’[Explorateur de recherche](search-explorer.md) pour interroger votre index.

> [!NOTE]
> Si vous ne voyez pas les données attendues, vous devrez peut-être définir d’autres attributs sur d’autres champs. Supprimez l’index et l’indexeur que vous venez de créer et réexécutez l’Assistant, en modifiant vos sélections pour les attributs d’index à l’étape 5. 

<a name="cosmosdb-indexer-rest"></a>

## <a name="use-rest-apis"></a>Utiliser les API REST

Vous pouvez utiliser l’API REST pour indexer des données Azure Cosmos DB en suivant un flux de travail en trois parties commun à tous les indexeurs dans Recherche cognitive Azure : créer une source de données, créer un index, créer un indexeur. L’extraction de données dans Cosmos DB se produit quand vous envoyez la demande de création d’un indexeur. Lorsque cette requête est terminée, vous disposez d’un index pouvant être interrogé. 

> [!NOTE]
> Pour indexer des données à partir de l’API Gremlin ou de l’API Cassandra de Cosmos DB, vous devez d’abord demander l’accès aux préversions contrôlées en remplissant [ce formulaire](https://aka.ms/azure-cognitive-search/indexer-preview). Une fois votre demande traitée, vous recevez des instructions sur l’utilisation de l’[API REST version 2020-06-30-Preview](search-api-preview.md) pour créer la source de données.

Plus haut dans cet article, il est mentionné que l’[indexation d’Azure Cosmos DB](../cosmos-db/index-overview.md) et l’[indexation de Recherche cognitive Azure ](search-what-is-an-index.md) sont des opérations distinctes. Pour l’indexation Cosmos DB, par défaut, tous les documents sont indexés automatiquement, sauf avec l’API Cassandra. Si vous désactivez l’indexation automatique, les documents sont accessibles seulement via leurs liens réflexifs ou des requêtes avec l’ID de document. L’indexation Recherche cognitive Azure nécessite l’activation de l’indexation automatique Cosmos DB dans la collection qui sera indexée par Recherche cognitive Azure. Lors de l’inscription à la préversion de l’indexeur de l’API Cassandra de Cosmos DB, vous recevez des instructions sur la configuration de l’indexation Cosmos DB.

> [!WARNING]
> Azure Cosmos DB est la nouvelle génération de DocumentDB. Précédemment, avec l’API version **2017-11-11**, vous pouviez utiliser la syntaxe `documentdb`. Autrement dit, vous pouviez spécifier votre type de source de données en tant que `cosmosdb` ou `documentdb`. À compter de la version d’API **2019-05-06**, les APIS Recherche cognitive Azure et le portail prennent uniquement en charge la syntaxe `cosmosdb` comme indiqué dans cet article. Cela signifie que le type de source de données doit être `cosmosdb` si vous souhaitez vous connecter à un point de terminaison Cosmos DB.

### <a name="1---assemble-inputs-for-the-request"></a>1 - Assembler des entrées pour la requête

Pour chaque requête, vous devez fournir le nom du service et la clé d’administration pour Recherche cognitive Azure (dans l’en-tête POST), ainsi que le nom du compte de stockage et la clé pour le stockage d’objets Blob. Vous pouvez utiliser [Postman ou Visual Studio Code](search-get-started-rest.md) pour envoyer des requêtes HTTP à Recherche cognitive Azure.

Copiez les quatre valeurs suivantes dans le bloc-notes pour pouvoir les coller dans une requête :

+ Nom du service Recherche cognitive Azure
+ Clé d’administration de Recherche cognitive Azure
+ Chaîne de connexion Cosmos DB

Vous pouvez trouver ces valeurs dans le portail :

1. Dans les pages du portail pour Recherche cognitive Azure, copiez l’URL du service de recherche dans la page Vue d’ensemble.

2. Dans le volet de navigation gauche, cliquez sur **Clés**, puis copiez la clé primaire ou secondaire (elles sont équivalentes).

3. Basculez vers les pages du portail pour votre compte de stockage Cosmos. Dans le volet de navigation gauche, sous **Paramètres**, sélectionnez **Clés**. Cette page fournit un URI, deux ensembles de chaînes de connexion, et deux ensembles de clés. Copiez une des chaînes de connexion dans le bloc-notes.

### <a name="2---create-a-data-source"></a>2 - Création d'une source de données

Une **source de données** spécifie les données à indexer, les informations d’identification et les stratégies pour identifier les modifications des données (par exemple, les documents modifiés ou supprimés dans votre collection). La source de données est définie en tant que ressource indépendante de manière à pouvoir être utilisée par plusieurs indexeurs.

Pour créer une source de données, formulez une requête POST :

```http

    POST https://[service name].search.windows.net/datasources?api-version=2020-06-30
    Content-Type: application/json
    api-key: [Search service admin key]

    {
        "name": "mycosmosdbdatasource",
        "type": "cosmosdb",
        "credentials": {
            "connectionString": "AccountEndpoint=https://myCosmosDbEndpoint.documents.azure.com;AccountKey=myCosmosDbAuthKey;Database=myCosmosDbDatabaseId"
        },
        "container": { "name": "myCollection", "query": null },
        "dataChangeDetectionPolicy": {
            "@odata.type": "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
            "highWaterMarkColumnName": "_ts"
        }
    }
```

Le corps de la requête contient la définition de la source de données, qui doit inclure les champs suivants :

| Champ   | Description |
|---------|-------------|
| **name** | Obligatoire. Choisissez un nom pour représenter votre objet source de données. |
|**type**| Obligatoire. Doit être `cosmosdb`. |
|**credentials** | Obligatoire. Doit suivre le format de chaîne de connexion Cosmos DB ou un format de chaîne de connexion d'identité managée.<br/><br/>Pour les **collections SQL**, les chaînes de connexion peuvent suivre l'un des formats ci-dessous : <li>`AccountEndpoint=https://<Cosmos DB account name>.documents.azure.com;AccountKey=<Cosmos DB auth key>;Database=<Cosmos DB database id>`<li>Une chaîne de connexion d'identité managée au format suivant qui n'inclut pas de clé de compte : `ResourceId=/subscriptions/<your subscription ID>/resourceGroups/<your resource group name>/providers/Microsoft.DocumentDB/databaseAccounts/<your cosmos db account name>/;`. Pour utiliser ce format de chaîne de connexion, reportez-vous à [Configurer une connexion d'indexeur à une base de données Cosmos DB à l'aide d'une identité managée](search-howto-managed-identities-cosmos-db.md) et suivez les instructions fournies.<br/><br/>Pour les versions 3.2 et 3.6 des **collections MongoDB**, utilisez l'un des formats suivants pour la chaîne de connexion : <li>`AccountEndpoint=https://<Cosmos DB account name>.documents.azure.com;AccountKey=<Cosmos DB auth key>;Database=<Cosmos DB database id>;ApiKind=MongoDb`<li>Une chaîne de connexion d'identité managée au format suivant qui n'inclut pas de clé de compte : `ResourceId=/subscriptions/<your subscription ID>/resourceGroups/<your resource group name>/providers/Microsoft.DocumentDB/databaseAccounts/<your cosmos db account name>/;ApiKind=MongoDb;`. Pour utiliser ce format de chaîne de connexion, reportez-vous à [Configurer une connexion d'indexeur à une base de données Cosmos DB à l'aide d'une identité managée](search-howto-managed-identities-cosmos-db.md) et suivez les instructions fournies.<br/><br/>Pour les **graphes Gremlin et les tables Cassandra**, inscrivez-vous à la [préversion de l’indexeur contrôlé](https://aka.ms/azure-cognitive-search/indexer-preview) pour accéder à la préversion et aux informations sur la façon de mettre en forme les informations d’identification.<br/><br/>Évitez les numéros de port dans l’URL du point de terminaison. Si vous incluez le numéro de port, Recherche cognitive Azure ne peut pas indexer votre base de données Azure Cosmos DB.|
| **container** | Contient les éléments suivants : <br/>**nom** : Obligatoire. Spécifiez l’ID de la collection de bases de données à indexer.<br/>**query** : facultatif. Vous pouvez spécifier une requête pour obtenir un schéma plat à partir d’un document JSON arbitraire de manière à ce qu’Azure Search puisse procéder à l’indexation.<br/>Pour l’API MongoDB, l’API Gremlin et l’API Cassandra, les requêtes ne sont pas prises en charge. |
| **dataChangeDetectionPolicy** | Recommandé. Consultez la section [Indexation des documents modifiés](#DataChangeDetectionPolicy).|
|**dataDeletionDetectionPolicy** | facultatif. Consultez la section [Indexation des documents supprimés](#DataDeletionDetectionPolicy).|

### <a name="using-queries-to-shape-indexed-data"></a>Utilisation de requêtes pour formater les données indexées
Vous pouvez spécifier une requête SQL pour aplatir les propriétés ou les tableaux imbriqués, projeter des propriétés JSON et filtrer les données à indexer. 

> [!WARNING]
> Les requêtes personnalisées ne sont pas prises en charge pour l’**API MongoDB**, l’**API Gremlin** et l’**API Cassandra** : le paramètre `container.query` doit être défini sur null ou être omis. Si vous avez besoin d’utiliser une requête personnalisée, indiquez-le nous sur [UserVoice](https://feedback.azure.com/forums/263029-azure-search).

Exemple de document :

```http
    {
        "userId": 10001,
        "contact": {
            "firstName": "andy",
            "lastName": "hoh"
        },
        "company": "microsoft",
        "tags": ["azure", "cosmosdb", "search"]
    }
```

Requête de filtre :

```sql
SELECT * FROM c WHERE c.company = "microsoft" and c._ts >= @HighWaterMark ORDER BY c._ts
```

Requête d’aplatissage :

```sql
SELECT c.id, c.userId, c.contact.firstName, c.contact.lastName, c.company, c._ts FROM c WHERE c._ts >= @HighWaterMark ORDER BY c._ts
```

Requête de projection :

```sql
SELECT VALUE { "id":c.id, "Name":c.contact.firstName, "Company":c.company, "_ts":c._ts } FROM c WHERE c._ts >= @HighWaterMark ORDER BY c._ts
```

Requête d’aplatissage de tableau :

```sql
SELECT c.id, c.userId, tag, c._ts FROM c JOIN tag IN c.tags WHERE c._ts >= @HighWaterMark ORDER BY c._ts
```

### <a name="3---create-a-target-search-index"></a>3 - Créer un index de recherche cible 

[Créez un index Recherche cognitive Azure cible](/rest/api/searchservice/create-index) si vous n’en avez pas. L'exemple suivant crée un index avec un champ ID et un champ Description :

```http
    POST https://[service name].search.windows.net/indexes?api-version=2020-06-30
    Content-Type: application/json
    api-key: [Search service admin key]

    {
       "name": "mysearchindex",
       "fields": [{
         "name": "id",
         "type": "Edm.String",
         "key": true,
         "searchable": false
       }, {
         "name": "description",
         "type": "Edm.String",
         "filterable": false,
         "sortable": false,
         "facetable": false,
         "suggestions": true
       }]
     }
```

Assurez-vous que le schéma de votre index cible est compatible avec le schéma des documents JSON source ou la sortie de votre projection de requête personnalisée.

> [!NOTE]
> Pour les collections partitionnées, la clé de document par défaut est la propriété `_rid` d’Azure Cosmos DB, que Recherche cognitive Azure renomme automatiquement en `rid`, car les noms de champ ne peuvent pas commencer par un trait de soulignement. De même, les valeurs `_rid` d’Azure Cosmos DB contiennent des caractères non valides dans les clés de Recherche cognitive Azure. Par conséquent, les valeurs `_rid` sont codées en Base64.
> 
> Pour les collections MongoDB, Recherche cognitive Azure renomme automatiquement la propriété `_id``id`.  

### <a name="mapping-between-json-data-types-and-azure-cognitive-search-data-types"></a>Mappage entre les types de données JSON et les types de données Azure Search
| Type de données JSON | Types de champs d’index cible compatibles |
| --- | --- |
| Bool |Edm.Boolean, Edm.String |
| Nombres qui ressemblent à des nombres entiers |Edm.Int32, Edm.Int64, Edm.String |
| Nombres qui ressemblent à des nombres avec points flottants |Edm.Double, Edm.String |
| String |Edm.String |
| Tableaux de types primitifs, par exemple ["a", "b", "c"] |Collection(Edm.String) |
| Chaînes qui ressemblent à des dates |Edm.DateTimeOffset, Edm.String |
| Objets GeoJSON, par exemple { "type": "Point", "coordinates": [long, lat] } |Edm.GeographyPoint |
| Autres objets JSON |N/A |

### <a name="4---configure-and-run-the-indexer"></a>4 - Configurer et exécuter l’indexeur

Une fois l'index et la source de données créés, vous êtes prêt à créer l’indexeur :

```http
    POST https://[service name].search.windows.net/indexers?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
      "name" : "mycosmosdbindexer",
      "dataSourceName" : "mycosmosdbdatasource",
      "targetIndexName" : "mysearchindex",
      "schedule" : { "interval" : "PT2H" }
    }
```

Cet indexeur s’exécute toutes les deux heures (intervalle de planification défini sur « PT2H »). Pour exécuter un indexeur toutes les 30 minutes, définissez l’intervalle sur « PT30M ». Le plus court intervalle pris en charge est de 5 minutes. La planification est facultative : en cas d’omission, un indexeur ne s’exécute qu’une seule fois lorsqu’il est créé. Toutefois, vous pouvez à tout moment exécuter un indexeur à la demande.   

Pour plus d’informations sur l’API Créer un indexeur, consultez [Créer un indexeur](/rest/api/searchservice/create-indexer).

Pour plus d’informations sur la définition des planifications de l’indexeur, consultez [Guide pratique pour planifier les indexeurs pour Recherche cognitive Azure](search-howto-schedule-indexers.md).

## <a name="use-net"></a>Utiliser .NET

Le kit de développement logiciel (SDK) .NET mis à la disposition générale offre une parité complète avec l’API REST mise à la disposition générale. Nous vous recommandons de consulter la section précédente de l’API REST pour découvrir les concepts, les workflows et les exigences. Vous pouvez alors vous référer à la documentation de référence des API .NET suivante pour implémenter un indexeur JSON dans du code managé.

+ [azure.search.documents.indexes.models.searchindexerdatasourceconnection](/dotnet/api/azure.search.documents.indexes.models.searchindexerdatasourceconnection)
+ [azure.search.documents.indexes.models.searchindexerdatasourcetype](/dotnet/api/azure.search.documents.indexes.models.searchindexerdatasourcetype)
+ [azure.search.documents.indexes.models.searchindex](/dotnet/api/azure.search.documents.indexes.models.searchindex)
+ [azure.search.documents.indexes.models.searchindexer](/dotnet/api/azure.search.documents.indexes.models.searchindexer)

<a name="DataChangeDetectionPolicy"></a>

## <a name="indexing-changed-documents"></a>Indexation des documents modifiés

L'objectif d'une stratégie de détection des changements de données est d'identifier efficacement les données modifiées. La seule stratégie actuellement prise en charge est la [`HighWaterMarkChangeDetectionPolicy`](/dotnet/api/azure.search.documents.indexes.models.highwatermarkchangedetectionpolicy) qui utilise la propriété `_ts` (timestamp) fournie par Azure Cosmos DB, définie ainsi :

```http
    {
        "@odata.type" : "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
        "highWaterMarkColumnName" : "_ts"
    }
```

Cette stratégie est vivement recommandée pour garantir de bonnes performances pour l’indexeur. 

Si vous utilisez une requête personnalisée, assurez-vous que la propriété `_ts` est projetée par la requête.

<a name="IncrementalProgress"></a>

### <a name="incremental-progress-and-custom-queries"></a>Progression incrémentielle et requêtes personnalisées

Dans le cas où l’exécution de l’indexeur est interrompue par des défaillances passagères ou un dépassement du délai d’exécution, la progression incrémentielle pendant l’indexation permet à l’indexeur de reprendre là il en était lors de sa dernière exécution, plutôt que d’avoir à tout réindexer depuis le début. Ceci est particulièrement important lors de l’indexation de grandes collections. 

Pour activer la progression incrémentielle lors de l’utilisation d’une requête personnalisée, assurez-vous que votre requête classe les résultats par la colonne `_ts`. Ceci permet de créer des points de contrôle périodiques dont Azure Search se sert pour proposer la progression incrémentielle en cas d’erreurs.   

Dans certains cas, il se peut qu’Azure Search ne déduise pas que la requête est ordonnée par `_ts`, même si elle contient une clause `ORDER BY [collection alias]._ts`. Vous pouvez indiquer à Azure Search que les résultats sont triés à l’aide de la propriété de configuration `assumeOrderByHighWaterMarkColumn`. Pour ce faire, créez ou mettez à jour l’indexeur comme suit : 

```http
    {
     ... other indexer definition properties
     "parameters" : {
            "configuration" : { "assumeOrderByHighWaterMarkColumn" : true } }
    } 
```

<a name="DataDeletionDetectionPolicy"></a>

## <a name="indexing-deleted-documents"></a>Indexation des documents supprimés

Lorsque des lignes sont supprimées de la collection, vous devez normalement supprimer ces lignes de l'index de recherche. L'objectif d'une stratégie de détection des suppressions de données est d'identifier efficacement les données supprimées. La seule stratégie actuellement prise en charge est la stratégie `Soft Delete` (où la suppression est signalée par un indicateur quelconque), spécifiée comme suit :

```http
    {
        "@odata.type" : "#Microsoft.Azure.Search.SoftDeleteColumnDeletionDetectionPolicy",
        "softDeleteColumnName" : "the property that specifies whether a document was deleted",
        "softDeleteMarkerValue" : "the value that identifies a document as deleted"
    }
```

Si vous utilisez une requête personnalisée, assurez-vous que la propriété référencée par `softDeleteColumnName` est projetée par la requête.

L'exemple suivant crée une source de données avec des conseils pour une stratégie de suppression en douceur :

```http
    POST https://[service name].search.windows.net/datasources?api-version=2020-06-30
    Content-Type: application/json
    api-key: [Search service admin key]

    {
        "name": "mycosmosdbdatasource",
        "type": "cosmosdb",
        "credentials": {
            "connectionString": "AccountEndpoint=https://myCosmosDbEndpoint.documents.azure.com;AccountKey=myCosmosDbAuthKey;Database=myCosmosDbDatabaseId"
        },
        "container": { "name": "myCosmosDbCollectionId" },
        "dataChangeDetectionPolicy": {
            "@odata.type": "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
            "highWaterMarkColumnName": "_ts"
        },
        "dataDeletionDetectionPolicy": {
            "@odata.type": "#Microsoft.Azure.Search.SoftDeleteColumnDeletionDetectionPolicy",
            "softDeleteColumnName": "isDeleted",
            "softDeleteMarkerValue": "true"
        }
    }
```

## <a name="next-steps"></a><a name="NextSteps"></a>Étapes suivantes

Félicitations ! Vous avez appris à intégrer Azure Cosmos DB avec Recherche cognitive Azure à l’aide d’un indexeur.

* Pour en savoir plus sur Azure Cosmos DB, consultez la [page du service Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/).
* Pour en savoir plus sur la Recherche cognitive Azure, consultez la [page du service Recherche](https://azure.microsoft.com/services/search/).