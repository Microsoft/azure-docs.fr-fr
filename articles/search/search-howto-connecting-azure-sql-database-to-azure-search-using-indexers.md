---
title: Effectuer des recherches sur les données Azure SQL
titleSuffix: Azure Cognitive Search
description: Importez des données dans Azure SQL Database ou SQL Managed Instance à l’aide d’indexeurs pour la recherche en texte intégral dans Recherche cognitive Azure. Cet article traite des connexions, de la configuration des indexeurs et de l’ingestion des données.
manager: nitinme
author: mgottein
ms.author: magottei
ms.devlang: rest-api
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 07/12/2020
ms.openlocfilehash: 04e4801c26b0ac8ef91af0b028d9dc2bb9a3cd1c
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94358624"
---
# <a name="connect-to-and-index-azure-sql-content-using-an-azure-cognitive-search-indexer"></a>Se connecter à du contenu Azure SQL et l’indexer à l’aide d’un indexeur Recherche cognitive Azure

Avant d’interroger un [index Recherche cognitive Azure](search-what-is-an-index.md), vous devez le remplir avec vos données. Si les données se trouvent dans Azure SQL Database ou SQL Managed Instance, un **indexeur Recherche cognitive Azure pour Azure SQL Database** (ou **indexeur Azure SQL** ) peut automatiser le processus d’indexation. En d’autres termes, vous avez moins de code à écrire et la maintenance de l’infrastructure est moins lourde.

Cet article décrit l’utilisation des [indexeurs](search-indexer-overview.md), mais aussi les fonctionnalités propres à Azure SQL Database ou SQL Managed Instance (par exemple, le suivi intégré des modifications). 

En plus d’Azure SQL Database et de SQL Managed Instance, Recherche cognitive Azure fournit des indexeurs pour [Azure Cosmos DB](search-howto-index-cosmosdb.md), le [stockage blob Azure](search-howto-indexing-azure-blob-storage.md) et le [stockage de table Azure](search-howto-indexing-azure-tables.md). Pour obtenir de l’aide concernant d’autres sources de données, indiquez vos souhaits sur le [forum Recherche cognitive Azure](https://feedback.azure.com/forums/263029-azure-search/).

## <a name="indexers-and-data-sources"></a>Indexeurs et sources de données

Une **source de données** spécifie les données à indexer, les informations d’identification pour accéder aux données, et les stratégies qui identifient efficacement les modifications apportées aux données (comme les lignes nouvelles, modifiées ou supprimées). Elle est définie en tant que ressource indépendante utilisable par plusieurs indexeurs.

Un **indexeur** est une ressource qui connecte une source de données unique à un index de recherche cible. Un indexeur est utilisé pour :

* effectuer une copie unique des données pour remplir un index ;
* Synchroniser un index avec les modifications apportées à la source de données selon une planification donnée.
* S’exécuter à la demande afin de mettre à jour un index en fonction des besoins.

Un indexeur unique peut utiliser une seule table ou une seule vue, mais vous pouvez créer plusieurs indexeurs pour remplir plusieurs index de recherche. Pour plus d’informations sur ces concepts, consultez [Opérations d’indexeur : workflow classique](/rest/api/searchservice/Indexer-operations#typical-workflow).

Vous pouvez installer et configurer un indexeur SQL Azure avec les outils suivants :

* Assistant Importation de données sur le [Portail Azure](https://portal.azure.com)
* [Kit de développement logiciel (SDK) .NET](/dotnet/api/azure.search.documents.indexes.models.searchindexer) de Recherche cognitive Azure
* [API REST](/rest/api/searchservice/indexer-operations) de Recherche cognitive Azure

Dans cet article, nous allons utiliser l’API REST pour créer des **indexeurs** et des **sources de données**.

## <a name="when-to-use-azure-sql-indexer"></a>Quand utiliser l’indexeur Azure SQL
Selon plusieurs facteurs relatifs à vos données, l'utilisation de l'indexeur Azure SQL peut être ou ne pas être appropriée. Si vos données répondent aux conditions suivantes, vous pouvez utiliser l’indexeur Azure SQL.

| Critères | Détails |
|----------|---------|
| Les données proviennent d’une seule table ou d’une seule vue | Si les données sont disséminées entre plusieurs tables, vous pouvez créer une vue unique des données. Toutefois, si vous utilisez une vue, vous ne pourrez plus utiliser la fonction intégrée de détection des modifications de SQL Server pour actualiser un index avec des modifications incrémentielles. Pour plus d’informations, consultez la section [Capture des lignes modifiées et supprimées](#CaptureChangedRows) ci-dessous. |
| Les types de données sont compatibles | Mais certains types SQL ne sont pas pris en charge dans les index Recherche cognitive Azure. Pour obtenir une liste, consultez [Mappage des types de données](#TypeMapping). |
| La synchronisation de données en temps réel n’est pas requise | Un indexeur peut réindexer votre table toutes les cinq minutes au maximum. Si vos données changent fréquemment et si les modifications doivent être intégrées dans l’index en quelques secondes ou quelques minutes, nous vous recommandons d’utiliser l’[API REST](/rest/api/searchservice/AddUpdate-or-Delete-Documents) ou le [SDK .NET](./search-get-started-dotnet.md) pour émettre directement les lignes mises à jour. |
| Une indexation incrémentielle est possible | Si vous avez un jeu de données important et si vous comptez exécuter l’indexeur selon une planification, Recherche cognitive Azure doit être en mesure d’identifier efficacement les lignes nouvelles, modifiées ou supprimées. L’indexation non incrémentielle n’est autorisée que si vous effectuez une indexation à la demande (non planifiée) ou une indexation de moins de 100 000 lignes. Pour plus d’informations, consultez la section [Capture des lignes modifiées et supprimées](#CaptureChangedRows) ci-dessous. |

> [!NOTE] 
> Recherche cognitive Azure ne prend en charge que l’authentification SQL Server. Si vous avez besoin de prise en charge pour l’authentification du mot de passe Azure Active Directory, veuillez voter pour cette [suggestion UserVoice](https://feedback.azure.com/forums/263029-azure-search/suggestions/33595465-support-azure-active-directory-password-authentica).

## <a name="create-an-azure-sql-indexer"></a>Créer un indexeur Azure SQL

1. Créez la source de données :

   ```
    POST https://myservice.search.windows.net/datasources?api-version=2020-06-30
    Content-Type: application/json
    api-key: admin-key

    {
        "name" : "myazuresqldatasource",
        "type" : "azuresql",
        "credentials" : { "connectionString" : "Server=tcp:<your server>.database.windows.net,1433;Database=<your database>;User ID=<your user name>;Password=<your password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30;" },
        "container" : { "name" : "name of the table or view that you want to index" }
    }
   ```

   La chaîne de connexion peut suivre l’un des formats suivants :
    1. Vous pouvez obtenir la chaîne de connexion auprès du [portail Azure](https://portal.azure.com). Utilisez l’option `ADO.NET connection string`.
    1. Une chaîne de connexion d’identité managée qui n’inclut pas de clé de compte, au format suivant : `Initial Catalog|Database=<your database name>;ResourceId=/subscriptions/<your subscription ID>/resourceGroups/<your resource group name>/providers/Microsoft.Sql/servers/<your SQL Server name>/;Connection Timeout=connection timeout length;`. Pour utiliser cette chaîne de connexion, suivez les instructions fournies dans [Configurer une connexion d’indexeur à une base de données Azure SQL Database à l’aide d’une identité managée](search-howto-managed-identities-sql.md).

2. Créez l’index Recherche cognitive Azure cible si vous n’en avez pas encore. Pour créer un index, utilisez le [portail](https://portal.azure.com) ou l’[API Créer un index](/rest/api/searchservice/Create-Index). Vérifiez que le schéma de votre index cible est compatible avec le schéma de la table source. Consultez [Mappage entre les types de données SQL et les types de données de Recherche cognitive Azure](#TypeMapping).

3. Créez l’indexeur en lui attribuant un nom et en référençant les sources de données sources et cibles :

   ```
    POST https://myservice.search.windows.net/indexers?api-version=2020-06-30
    Content-Type: application/json
    api-key: admin-key

    {
        "name" : "myindexer",
        "dataSourceName" : "myazuresqldatasource",
        "targetIndexName" : "target index name"
    }
   ```

Un indexeur créé de cette façon n’a pas de planification. Il s’exécute automatiquement une fois créé. Vous pouvez le réexécuter à tout moment à l'aide d’une requête **run indexer** :

```
    POST https://myservice.search.windows.net/indexers/myindexer/run?api-version=2020-06-30
    api-key: admin-key
```

Vous pouvez personnaliser différents aspects du comportement des indexeurs, notamment la taille du lot et le nombre de documents pouvant être ignorés avant que l’exécution d’un indexeur n’échoue. Pour plus d’informations, consultez [Créer une API d’indexeur](/rest/api/searchservice/Create-Indexer).

Il se peut que vous deviez autoriser des services Azure pour vous connecter à votre base de données. Pour plus d’informations sur la marche à suivre, consultez la section [Connexion à partir de Azure](../azure-sql/database/firewall-configure.md) .

Pour surveiller l’état et l’historique d’exécution de l'indexeur (nombre d’éléments indexés, échecs, etc.), utilisez une requête **indexer status** :

```
    GET https://myservice.search.windows.net/indexers/myindexer/status?api-version=2020-06-30
    api-key: admin-key
```

La réponse doit être semblable à ce qui suit :

```
    {
        "\@odata.context":"https://myservice.search.windows.net/$metadata#Microsoft.Azure.Search.V2015_02_28.IndexerExecutionInfo",
        "status":"running",
        "lastResult": {
            "status":"success",
            "errorMessage":null,
            "startTime":"2015-02-21T00:23:24.957Z",
            "endTime":"2015-02-21T00:36:47.752Z",
            "errors":[],
            "itemsProcessed":1599501,
            "itemsFailed":0,
            "initialTrackingState":null,
            "finalTrackingState":null
        },
        "executionHistory":
        [
            {
                "status":"success",
                "errorMessage":null,
                "startTime":"2015-02-21T00:23:24.957Z",
                "endTime":"2015-02-21T00:36:47.752Z",
                "errors":[],
                "itemsProcessed":1599501,
                "itemsFailed":0,
                "initialTrackingState":null,
                "finalTrackingState":null
            },
            ... earlier history items
        ]
    }
```

L'historique d'exécution contient jusqu’à 50 exécutions les plus récentes, classées par ordre antichronologique (la dernière exécution apparaît en premier dans la réponse).
Vous trouverez des informations supplémentaires sur la réponse dans [Obtenir l’état de l’indexeur](/rest/api/searchservice/get-indexer-status)

## <a name="run-indexers-on-a-schedule"></a>Exécuter des indexeurs selon une planification
Vous pouvez également configurer l'indexeur pour qu’il s’exécute à intervalles périodiques. Pour ce faire, ajoutez la propriété **schedule** lors de la création ou de la mise à jour de l’indexeur. L'exemple ci-dessous montre une requête PUT mettant à jour l'indexeur :

```
    PUT https://myservice.search.windows.net/indexers/myindexer?api-version=2020-06-30
    Content-Type: application/json
    api-key: admin-key

    {
        "dataSourceName" : "myazuresqldatasource",
        "targetIndexName" : "target index name",
        "schedule" : { "interval" : "PT10M", "startTime" : "2015-01-01T00:00:00Z" }
    }
```

Le paramètre **interval** est obligatoire. Il correspond à la durée entre le début de deux exécutions consécutives de l’indexeur. L'intervalle minimal autorisé est de 5 minutes, l'intervalle maximal autorisé est d'une journée. Il doit être formaté en tant que valeur « dayTimeDuration » XSD (un sous-ensemble limité d'une valeur de [durée ISO 8601](https://www.w3.org/TR/xmlschema11-2/#dayTimeDuration) ). Le modèle est le suivant : `P(nD)(T(nH)(nM))`. Exemples : `PT15M` toutes les 15 minutes, `PT2H` toutes les deux heures.

Pour plus d’informations sur la définition des planifications de l’indexeur, consultez [Comment planifier des indexeurs pour la Recherche cognitive Azure](search-howto-schedule-indexers.md).

<a name="CaptureChangedRows"></a>

## <a name="capture-new-changed-and-deleted-rows"></a>Capturer des lignes nouvelles, modifiées et supprimées

La Recherche cognitive Azure utilise **l’indexation incrémentielle** pour éviter d’avoir à réindexer toute la table ou toute la vue à chaque exécution d’un indexeur. Recherche cognitive Azure fournit deux stratégies de détection des modifications pour la prise en charge de l’indexation incrémentielle. 

### <a name="sql-integrated-change-tracking-policy"></a>Stratégie de suivi intégré des modifications SQL
Si votre base de données SQL prend en charge le [suivi des modifications](/sql/relational-databases/track-changes/about-change-tracking-sql-server), nous recommandons d'utiliser la **stratégie de suivi intégré des modifications SQL**. Il s’agit de la stratégie la plus efficace. De plus, elle permet à la Recherche cognitive Azure d’identifier les lignes supprimées, sans avoir à ajouter une colonne « suppression réversible » explicite à votre table.

#### <a name="requirements"></a>Spécifications 

+ Configuration requise pour la version de base de données :
  * SQL Server 2012 SP3 et versions ultérieures, si vous utilisez SQL Server sur des machines virtuelles Azure.
  * Azure SQL Database ou SQL Managed Instance.
+ Tables uniquement (aucune vue). 
+ Dans la base de données, [activez le suivi](/sql/relational-databases/track-changes/enable-and-disable-change-tracking-sql-server) de la table. 
+ Aucune clé primaire composite (clé primaire contenant plusieurs colonnes) dans la table.  

#### <a name="usage"></a>Usage

Pour utiliser cette stratégie, créez ou mettez à jour votre source de données comme suit :

```
    {
        "name" : "myazuresqldatasource",
        "type" : "azuresql",
        "credentials" : { "connectionString" : "connection string" },
        "container" : { "name" : "table or view name" },
        "dataChangeDetectionPolicy" : {
           "@odata.type" : "#Microsoft.Azure.Search.SqlIntegratedChangeTrackingPolicy"
      }
    }
```

Si vous utilisez le suivi intégré des modifications SQL, ne spécifiez pas une stratégie de détection des lignes supprimées. Elle intègre la prise en charge de l'identification des lignes supprimées. Toutefois, pour les suppressions détectées automatiquement, la clé de document de votre index de recherche doit être identique à la clé primaire de la table SQL. 

> [!NOTE]  
> Lorsque vous utilisez [TRUNCATE TABLE](/sql/t-sql/statements/truncate-table-transact-sql) pour supprimer un grand nombre de lignes dans une table SQL, l'indexeur doit être [reset](/rest/api/searchservice/reset-indexer) pour réinitialiser l'état de suivi des modifications et récupérer les suppressions de lignes.

<a name="HighWaterMarkPolicy"></a>

### <a name="high-water-mark-change-detection-policy"></a>Stratégie de détection des modifications de limite supérieure

Cette stratégie de détection des modifications s’appuie sur une colonne « Limite supérieure » qui capture la version ou l’heure pour laquelle une ligne a été mise à jour. Si vous utilisez une vue, vous devez vous servir d’une stratégie de limite supérieure. La colonne de limite supérieure doit remplir les conditions suivantes.

#### <a name="requirements"></a>Spécifications 

* Toutes les insertions spécifient une valeur pour la colonne.
* Toutes les mises à jour d'un élément modifient également la valeur de la colonne.
* La valeur de cette colonne augmente à chaque insertion ou mise à jour.
* Les requêtes utilisant les clauses WHERE et ORDER BY suivantes peuvent être exécutées efficacement : `WHERE [High Water Mark Column] > [Current High Water Mark Value] ORDER BY [High Water Mark Column]`

> [!IMPORTANT] 
> Nous vous recommandons d’utiliser le type de données [rowversion](/sql/t-sql/data-types/rowversion-transact-sql) pour la colonne dédiée à la limite supérieure. Si un autre type de données est utilisé, il n’est pas garanti que le suivi des modifications capture toutes les modifications en présence de transactions qui s’exécutent en même temps qu’une requête de l’indexeur. Lorsque vous utilisez **rowversion** dans une configuration avec des réplicas en lecture seule, vous devez pointer l’indexeur sur le réplica principal. Seul un réplica principal peut être utilisé dans les scénarios de synchronisation de données.

#### <a name="usage"></a>Usage

Pour utiliser une stratégie de limite supérieure, créez ou mettez à jour votre source de données comme suit :

```
    {
        "name" : "myazuresqldatasource",
        "type" : "azuresql",
        "credentials" : { "connectionString" : "connection string" },
        "container" : { "name" : "table or view name" },
        "dataChangeDetectionPolicy" : {
           "@odata.type" : "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
           "highWaterMarkColumnName" : "[a rowversion or last_updated column name]"
      }
    }
```

> [!WARNING]
> Si la table source n’a pas d’index dans la colonne de limite supérieure, les requêtes utilisées par l’indexeur SQL risquent d’expirer. En particulier, la clause `ORDER BY [High Water Mark Column]` a besoin d’un index pour pouvoir s’exécuter efficacement lorsque la table contient de nombreuses lignes.
>
>

<a name="convertHighWaterMarkToRowVersion"></a>

##### <a name="converthighwatermarktorowversion"></a>convertHighWaterMarkToRowVersion

Si vous utilisez un type de données [rowversion](/sql/t-sql/data-types/rowversion-transact-sql) pour la colonne de limite supérieure, envisagez d’utiliser le paramètre de configuration de l’indexeur `convertHighWaterMarkToRowVersion`. `convertHighWaterMarkToRowVersion` effectue deux opérations :

* Utilisez le type de données rowversion pour la colonne de limite supérieure dans la requête sql de l’indexeur. L’utilisation du type de données correct améliore le niveau de performance de requête de l’indexeur.
* Soustrayez 1 de la valeur rowversion avant l’exécution de la requête de l’indexeur. Les affichages comportant de 1 à plusieurs jointures peuvent contenir des lignes avec des valeurs rowversion en double. Soustraire 1 garantit que la requête de l’indexeur n’ignore pas ces lignes.

Pour activer cette fonctionnalité, créez ou mettez à jour l’indexeur avec la configuration suivante :

```
    {
      ... other indexer definition properties
     "parameters" : {
            "configuration" : { "convertHighWaterMarkToRowVersion" : true } }
    }
```

<a name="queryTimeout"></a>

##### <a name="querytimeout"></a>queryTimeout

Si vous rencontrez des erreurs de temporisation, vous pouvez utiliser le paramètre de configuration d’indexeur `queryTimeout` pour donner une valeur plus élevée que les 5 minutes par défaut au délai d’expiration de la requête. Par exemple, pour fixer un délai d’expiration de 10 minutes, créez ou mettez à jour l’indexeur avec la configuration suivante :

```
    {
      ... other indexer definition properties
     "parameters" : {
            "configuration" : { "queryTimeout" : "00:10:00" } }
    }
```

<a name="disableOrderByHighWaterMarkColumn"></a>

##### <a name="disableorderbyhighwatermarkcolumn"></a>disableOrderByHighWaterMarkColumn

Vous pouvez également désactiver la clause `ORDER BY [High Water Mark Column]`. Toutefois, cette action est déconseillée car, si l’exécution de l’indexeur est interrompue par une erreur, l’indexeur doit traiter à nouveau toutes les lignes quand son exécution reprend, même s’il avait déjà traité la quasi-totalité des lignes au moment de l’interruption. Pour désactiver la clause `ORDER BY`, utilisez le paramètre `disableOrderByHighWaterMarkColumn` dans la définition de l’indexeur :  

```
    {
     ... other indexer definition properties
     "parameters" : {
            "configuration" : { "disableOrderByHighWaterMarkColumn" : true } }
    }
```

### <a name="soft-delete-column-deletion-detection-policy"></a>Stratégie de détection des colonnes à suppression réversible
Lorsque des lignes sont supprimées de la table source, vous devez également supprimer ces lignes de l'index de recherche. Si vous utilisez la stratégie de suivi intégré des modifications SQL, cette opération est prise en charge à votre place. Mais la stratégie de suivi des modifications de limite supérieure ne vous est d’aucune aide pour les lignes supprimées. Que faire, alors ?

Si des lignes sont physiquement supprimées de la table, Recherche cognitive Azure n’a aucun moyen de déduire la présence d’enregistrements qui n’existent plus.  Toutefois, vous pouvez utiliser la technique de la « suppression réversible » pour supprimer des lignes logiquement sans les supprimer de la table. Ajoutez une colonne à votre table ou votre vue et marquez les lignes comme supprimées à l’aide de cette colonne.

Lorsque vous utilisez la technique de suppression réversible, vous pouvez spécifier cette stratégie réversible comme suit lors de la création ou de la mise à jour de la source de données :

```
    {
        …,
        "dataDeletionDetectionPolicy" : {
           "@odata.type" : "#Microsoft.Azure.Search.SoftDeleteColumnDeletionDetectionPolicy",
           "softDeleteColumnName" : "[a column name]",
           "softDeleteMarkerValue" : "[the value that indicates that a row is deleted]"
        }
    }
```

**softDeleteMarkerValue** doit être une chaîne. Utilisez la représentation au format chaîne de votre valeur. Par exemple, si vous avez une colonne d’entiers dans laquelle les lignes supprimées sont marquées avec la valeur 1, utilisez `"1"`. Si vous avez une colonne BIT dans laquelle les lignes supprimées sont marquées avec la valeur booléenne True, utilisez le littéral de chaîne `True` ou `true`, la casse ne comptant pas.

<a name="TypeMapping"></a>

## <a name="mapping-between-sql-and-azure-cognitive-search-data-types"></a>Mappage entre les types de données SQL et Recherche cognitive Azure
| Type de données SQL | Types de champs d'index cible autorisés | Notes |
| --- | --- | --- |
| bit |Edm.Boolean, Edm.String | |
| int, smallint, tinyint |Edm.Int32, Edm.Int64, Edm.String | |
| bigint |Edm.Int64, Edm.String | |
| real, float |Edm.Double, Edm.String | |
| smallmoney, money decimal numeric |Edm.String |Recherche cognitive Azure ne prend pas en charge la conversion de types décimaux en Edm.Double, car elle entraîne une perte de précision |
| char, nchar, varchar, nvarchar |Edm.String<br/>Collection(Edm.String) |Une chaîne SQL peut être utilisée pour remplir un champ Collection(Edm.String) si la chaîne représente un tableau JSON de chaînes : `["red", "white", "blue"]` |
| smalldatetime, datetime, datetime2, date, datetimeoffset |Edm.DateTimeOffset, Edm.String | |
| uniqueidentifer |Edm.String | |
| Geography |Edm.GeographyPoint |Seules les instances Geography de type POINT avec SRID 4326 (valeur par défaut) sont prises en charge |
| rowversion |N/A |Les colonnes de version de ligne ne peuvent pas être stockées dans l'index de recherche, mais peuvent être utilisées pour le suivi des modifications |
| time, timespan, binary, varbinary, image, xml, geometry, types CLR |N/A |Non pris en charge |

## <a name="configuration-settings"></a>Paramètres de configuration
L’indexeur SQL expose plusieurs paramètres de configuration :

| Paramètre | Type de données | Objectif | Valeur par défaut |
| --- | --- | --- | --- |
| queryTimeout |string |Définit le délai d’expiration de l’exécution de la requête SQL |5 minutes ("00:05:00") |
| disableOrderByHighWaterMarkColumn |bool |Indique que la requête SQL utilisée par la stratégie de limite supérieure doit omettre la clause ORDER BY. Consultez [Stratégie de limite supérieure](#HighWaterMarkPolicy) |false |

Ces paramètres sont utilisés dans l’objet `parameters.configuration`, dans la définition de l’indexeur. Par exemple, pour fixer un délai d’expiration de la requête de 10 minutes, créez ou mettez à jour l’indexeur avec la configuration suivante :

```
    {
      ... other indexer definition properties
     "parameters" : {
            "configuration" : { "queryTimeout" : "00:10:00" } }
    }
```

## <a name="faq"></a>Questions fréquentes (FAQ)

**Q : Puis-je utiliser l’indexeur SQL Azure avec des bases de données SQL exécutées sur des machines virtuelles IaaS dans Azure ?**

Oui. Toutefois, vous devez autoriser votre service de recherche à se connecter à votre base de données. Pour plus d’informations, consultez l’article [Configurer une connexion d’un indexeur de Recherche cognitive Azure à SQL Server sur une machine virtuelle Azure](search-howto-connecting-azure-sql-iaas-to-azure-search-using-indexers.md).

**Q : Puis-je utiliser l’indexeur Azure SQL avec des bases de données SQL exécutées localement ?**

Pas directement. La connexion directe n’est pas prise en charge, ni recommandée, car elle vous oblige à ouvrir vos bases de données au trafic Internet. Les clients ont réussi à l’aide de technologies de pont telles qu’Azure Data Factory. Pour plus d’informations, consultez [Envoyer des données à un index Recherche cognitive Azure à l’aide d’Azure Data Factory](../data-factory/v1/data-factory-azure-search-connector.md).

**Q : Puis-je utiliser l’indexeur Azure SQL avec des bases de données autres que SQL Server exécutées en IaaS sur Azure ?**

Non. Ce cas de figure n’est pas pris en charge, car nous n’avons pas testé l’indexeur avec des bases de données autres que SQL Server.  

**Q : Puis-je créer plusieurs indexeurs qui s’exécutent selon une planification ?**

Oui. Cependant, seul un indexeur peut s'exécuter sur un nœud à la fois. Si vous avez besoin d’exécuter plusieurs indexeurs simultanément, envisagez d’ajouter d’autres unités de recherche à votre service de recherche.

**Q : L’exécution d’un indexeur affecte-t-elle la charge de travail de mes requêtes ?**

Oui. L’indexeur s'exécute sur un des nœuds de votre service de recherche, et les ressources de ce nœud sont partagées entre l'indexation et le traitement du trafic de requêtes d’une part, et d’autres requêtes d’API d’autre part. Si vous exécutez des charges de travail intensives d’indexation et de requête et si vous rencontrez un taux élevé d’erreurs 503 ou une augmentation des délais de réponse, [redimensionnez votre service de recherche](search-capacity-planning.md).

**Q : Puis-je utiliser un réplica secondaire dans un [cluster de basculement](../azure-sql/database/auto-failover-group-overview.md) comme source de données ?**

Cela dépend. Pour l’indexation intégrale d’une table ou d’une vue, vous pouvez utiliser un réplica secondaire. 

Pour l’indexation incrémentielle, Recherche cognitive Azure prend en charge deux stratégies de détection des modifications : le suivi des modifications intégré SQL et la limite supérieure.

Sur les réplicas en lecture seule, SQL Database ne prend pas en charge le suivi des modifications intégré. Par conséquent, vous devez utiliser la stratégie de limite supérieure. 

Nous vous recommandons d’utiliser le type de données rowversion pour la colonne dédiée à la limite supérieure. Toutefois, l’utilisation de rowversion repose sur la fonction `MIN_ACTIVE_ROWVERSION`, qui n’est pas prise en charge sur les réplicas en lecture seule. Par conséquent, vous devez pointer l’indexeur sur un réplica principal si vous utilisez rowversion.

Si vous essayez d’utiliser rowversion sur un réplica en lecture seule, l’erreur suivante s’affiche : 

« L’utilisation d’une colonne rowversion pour le suivi des modifications n’est pas prise en charge sur les réplicas de disponibilité secondaires (en lecture seule). Veuillez mettre à jour la source de données et spécifier une connexion au réplica de disponibilité principal. La propriété de « capacité de mise à jour » de la base de données actuelle est « READ_ONLY ».

**Q : Puis-je utiliser une colonne autre que rowversion pour le suivi des modifications de la limite supérieure ?**

Cela n’est pas recommandé. Seule la colonne **rowversion** permet une synchronisation fiable des données. Toutefois, en fonction de votre logique d’application, cette opération peut être sécurisée si :

+ Vous pouvez vous assurer que pendant l’exécution de l’indexeur, aucune transaction n’est en attente sur la table en cours d’indexation (par exemple, toutes les mises à jour de la table s’effectuent de manière planifiée par lot, et la planification de l’indexeur Recherche cognitive Azure est définie de manière à éviter tout chevauchement avec la planification de la mise à jour de la table).  

+ Vous procédez régulièrement à une réindexation complète pour sélectionner toutes les lignes manquantes.