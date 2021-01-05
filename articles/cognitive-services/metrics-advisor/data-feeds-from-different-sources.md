---
title: Comment ajouter des flux de données à partir de différentes sources dans Metrics Advisor
titleSuffix: Azure Cognitive Services
description: ajouter des flux de données différents à Metrics Advisor
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: conceptual
ms.date: 10/12/2020
ms.author: mbullwin
ms.openlocfilehash: c4d1d23da5fd9678cc5b9477ddeed0daf4f5ac36
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96348617"
---
# <a name="add-data-feeds-from-different-data-sources-to-metrics-advisor"></a>Ajouter des flux de données à partir de différentes sources dans Metrics Advisor

Utilisez cet article pour rechercher les paramètres et les exigences de connexion des différents types de sources de données à Metrics Advisor. Lisez comment [Intégrer vos données](how-tos/onboard-your-data.md) pour en savoir plus sur les concepts clés de l’utilisation de vos données avec Metrics Advisor. 

## <a name="supported-authentication-types"></a>Types d’authentification pris en charge

| Types d’authentification | Description |
| ---------------------|-------------|
|**De base** | Vous devez être en mesure de fournir des paramètres de base pour accéder aux sources de données. Par exemple, une chaîne de connexion ou une clé. Les administrateurs du flux de données peuvent afficher ces informations d’identification. |
| **AzureManagedIdentity** | [Identités managées](../../active-directory/managed-identities-azure-resources/overview.md) pour les ressources Azure est une fonctionnalité d’Azure Active Directory. Elle fournit aux services Azure une identité managée automatiquement dans Azure AD. Vous pouvez utiliser cette identité pour authentifier tous les services qui prennent en charge l’authentification Azure AD.|
| **AzureSQLConnectionString**| Stockez votre chaîne de connexion AzureSQL en tant qu’**entité d’informations d’identification** dans Metrics Advisor et utilisez-la directement à chaque fois que vous intégrez des données de mesures. Seuls les administrateurs de l’entité d’informations d’identification sont en mesure d’afficher ces informations d’identification, mais elles permettent aux lecteurs autorisés de créer des flux de données sans avoir besoin de connaître les détails des informations d’identification. |
| **DataLakeGen2SharedKey**| Stockez votre clé de compte de lac de données en tant qu’**entité d’informations d’identification** dans Metrics Advisor et utilisez-la directement à chaque fois que vous intégrez des données de mesures. Seuls les administrateurs de l’entité d’informations d’identification sont en mesure d’afficher ces informations d’identification, mais elles permettent aux lecteurs autorisés de créer un flux de données sans avoir besoin de connaître les détails des informations d’identification.|
| **Principal du service**| Stockez votre principal de service en tant qu’**entité d’informations d’identification** dans Metrics Advisor, et utilisez celle-ci directement chaque fois que vous intégrez des données de mesures. Seuls les administrateurs de l’entité d’informations d’identification sont en mesure d’afficher les informations d’identification, mais celles-ci permettent aux lecteurs autorisés de créer un flux de données sans avoir besoin de connaître les détails des informations d’identification.|
| **Principal de service du coffre de clés**|Stockez votre principal de service dans coffre de clés en tant qu’**entité d’informations d’identification** dans Metrics Advisor, et utilisez-le directement à chaque fois que vous intégrez des données de métriques. Seuls les administrateurs d’une **entité d’informations d’identification** sont en mesure d’afficher les informations d’identification, mais celles-ci permettent également aux lecteurs de créer des flux de données sans avoir besoin de connaître les détails des informations d’identification. |

## <a name="data-sources-supported-and-corresponding-authentication-types"></a>Sources de données prises en charge et types d’authentification correspondants


| Sources de données | Types d'authentification |
|-------------| ---------------------|
|[**Azure Application Insights**](#appinsights)|  Basic |
|[**Stockage Blob Azure (JSON)**](#blob) | Basic<br>ManagedIdentity|
|[**Azure Cosmos DB (SQL)**](#cosmosdb) | Basic |
|[**Azure Data Explorer (Kusto)**](#kusto) | Basic<br>ManagedIdentity|
|[**Azure Data Lake Storage Gen2**](#adl) | Basic<br>DataLakeGen2SharedKey<br>Principal du service<br>Principal de service du coffre de clés<br> |
|[**Azure SQL Database / SQL Server**](#sql) | Basic<br>ManagedIdentity<br>Principal du service<br>Principal de service du coffre de clés<br>AzureSQLConnectionString
|[**Stockage Table Azure**](#table) | Basic | 
|[**ElasticSearch**](#es) | Basic |
|[**requête HTTP**](#http) | Basic | 
|[**InfluxDB (InfluxQL)**](#influxdb) | Basic |
|[**MongoDB**](#mongodb) | Basic |
|[**MySQL**](#mysql) | Basic |
|[**PostgreSQL**](#pgsql)| Basic|

Créez une **entité d’informations d’identification** et utilisez-la pour l’authentification auprès de vos sources de données. Les sections suivantes spécifient les paramètres requis par pour une authentification *de base*. 

## <a name="span-idappinsightsazure-application-insightsspan"></a><span id="appinsights">Azure Application Insights</span>

* **ID d’application** : Celui-ci permet d'identifier cette application durant l'utilisation de l'API Application Insights. Pour obtenir l’ID d’application, effectuez les étapes suivantes :

    1. À partir de votre ressource Application Insights, cliquez sur Accès API.

    2. Copiez l’ID d’application généré dans le champ **ID d’application** dans Metrics Advisor. 
    
    Pour plus d’informations, consultez la [documentation Azure Bot Service](/azure/bot-service/bot-service-resources-app-insights-keys#application-id).

* **Clé API** : Les clés API sont utilisées par les applications en dehors du navigateur pour accéder à cette ressource. Pour obtenir la clé API, effectuez les étapes suivantes :

    1. À partir de la ressource Application Insights, cliquez sur Accès API.

    2. Cliquez sur Créer une clé API.

    3. Entrez une brève description, cochez l’option Lire les données de télémétrie, puis cliquez sur le bouton Générer la clé.

    4. Copiez la clé API dans le champ **Clé API** dans Metrics Advisor.

* **Requête** : Les journaux d’activité Azure Application Insights reposent sur Azure Data Explorer ; les requêtes de journal Azure Monitor utilisent une version du même langage de requête Kusto. La [documentation du langage de requête Kusto](/azure/data-explorer/kusto/query/), qui rassemble toutes les informations sur ce langage, doit constituer votre principale référence pour écrire une requête par rapport à Application Insights. 


## <a name="span-idblobazure-blob-storage-jsonspan"></a><span id="blob">Stockage Blob Azure (JSON)</span>

* **Chaîne de connexion** : Pour plus d’informations sur la récupération de cette chaîne, consultez l’article [chaîne de connexion](../../storage/common/storage-configure-connection-string.md#configure-a-connection-string-for-an-azure-storage-account) du Stockage Blob Azure.

* **Conteneur** : Metrics Advisor attend des données de série chronologique stockées en tant que fichiers BLOB (un BLOB par timestamp) sous un seul conteneur. Il s’agit du champ nom du conteneur.

* **Modèle de BLOB** : Il s’agit du modèle des noms de fichiers BLOB. Par exemple : `/%Y/%m/X_%Y-%m-%d-%h-%M.json`. Les paramètres suivants sont pris en charge :
  * `%Y` est l’année au format `yyyy`
  * `%m` est le mois au format `MM`
  * `%d` est le jour au format `dd`
  * `%h` est l’heure au format `HH`
  * `%M` est la minute au format `mm`

* **Version du format JSON** : Définit le schéma de données dans les fichiers JSON. Metrics Advisor prend actuellement en charge deux versions :
  
  * v1 (valeur par défaut)

      Seuls *Nom* et *Valeur* des mesures sont acceptés. Exemple :
    
      ``` JSON
      {"count":11, "revenue":1.23}
      ```

  * v2

      Les *Dimensions* et *timestamp* des mesures sont également acceptés. Exemple :
      
      ``` JSON
      [
        {"date": "2018-01-01T00:00:00Z", "market":"en-us", "count":11, "revenue":1.23},
        {"date": "2018-01-01T00:00:00Z", "market":"zh-cn", "count":22, "revenue":4.56}
      ]
      ```

Un seul timestamp est autorisé par fichier JSON. 

## <a name="span-idcosmosdbazure-cosmos-db-sqlspan"></a><span id="cosmosdb">Azure Cosmos DB (SQL)</span>

* **Chaîne de connexion** : La chaîne de connexion pour accéder à votre compte Azure Cosmos DB. Vous pouvez le trouver dans la ressource Cosmos DB, dans **Clés**. 
* **Base de données** : La base de données à interroger. Vous pouvez la trouver dans la page **Parcourir** sous la section **Conteneurs**.
* **ID de la collection** : ID de collection sur lequel effectuer la requête. Vous pouvez le trouver dans la page **Parcourir** sous la section **Conteneurs**.
* **Requête SQL** : Une requête SQL permettant d’obtenir et de formuler des données dans des données de séries chronologiques multidimensionnelles. Vous pouvez utiliser les variables `@StartTime` et `@EndTime` dans votre requête. Le format utilisé doit être le suivant : `yyyy-MM-dd HH:mm:ss`.

    Exemple de requête :
    
    ``` mssql
    select StartDate, JobStatusId, COUNT(*) AS JobNumber from IngestionJobs WHERE and StartDate = @StartTime
    ```
    
    Exemple de requête pour une tranche de données à partir du 12/12/2019 :
    
    ``` mssql
    select StartDate, JobStatusId, COUNT(*) AS JobNumber from IngestionJobs WHERE and StartDate = '2019-12-12 00:00:00'
    ```

## <a name="span-idkustoazure-data-explorer-kustospan"></a><span id="kusto">Azure Data Explorer (Kusto)</span>

* **Chaîne de connexion** : Metrics Advisor prend en charge l’accès à Azure Data Explorer (Kusto) à l’aide de l’authentification d’application Azure AD. Vous devrez créer et inscrire une application Azure AD, puis l’autoriser à accéder à une base de données Azure Data Explorer. Pour obtenir votre chaîne de connexion, consultez la documentation [Azure Data Explorer](/azure/data-explorer/provision-azure-ad-app).

* **Requête** : Consultez [Langage de la requête Kusto](/azure/data-explorer/kusto/query) pour obtenir et formuler des données dans des données de séries chronologiques multidimensionnelles. Vous pouvez utiliser les variables `@StartTime` et `@EndTime` dans votre requête. Le format utilisé doit être le suivant : `yyyy-MM-dd HH:mm:ss`.

## <a name="span-idadlazure-data-lake-storage-gen2span"></a><span id="adl">Azure Data Lake Storage Gen2</span>

* **Nom du compte** : Nom de votre compte Azure Data Lake Storage Gen2. Vous pouvez le trouver dans votre ressource de compte de Stockage Azure (Azure Data Lake Storage Gen2) dans **Clés d’accès**.

* **Clé de compte** : Spécifiez le nom de votre compte pour accéder à Azure Data Lake Storage Gen2. Vous pouvez le trouver dans votre ressource de compte de Stockage Azure (Azure Data Lake Storage Gen2) dans le paramètre **Clés d’accès**.

* **Nom du système de fichiers** : Metrics Advisor attend vos données de série chronologique stockées en tant que fichiers BLOB (un BLOB par timestamp) sous un seul conteneur. Il s’agit du champ nom du conteneur. Vous pouvez le trouver dans votre instance de compte de Stockage Azure (Azure Data Lake Storage Gen2), puis cliquez sur « conteneurs » dans la section « service BLOB ».

* **Modèle de répertoire** : Il s’agit du modèle de répertoire du fichier BLOB. Par exemple : */%Y/%m/%d*. Les paramètres suivants sont pris en charge :
  * `%Y` est l’année au format `yyyy`
  * `%m` est le mois au format `MM`
  * `%d` est le jour au format `dd`
  * `%h` est l’heure au format `HH`
  * `%M` est la minute au format `mm`

* **Modèle de fichier** : Il s’agit du modèle du fichier BLOB. Par exemple : *X_%Y-%m-%d-%h-%M.json*. Les paramètres suivants sont pris en charge :
  * `%Y` est l’année au format `yyyy`
  * `%m` est le mois au format `MM`
  * `%d` est le jour au format `dd`
  * `%h` est l’heure au format `HH`
  * `%M` est la minute au format `mm`

Metrics Advisor prend actuellement en charge le schéma de données dans les fichiers JSON comme suit. Exemple :

``` JSON
[
  {"date": "2018-01-01T00:00:00Z", "market":"en-us", "count":11, "revenue":1.23},
  {"date": "2018-01-01T00:00:00Z", "market":"zh-cn", "count":22, "revenue":4.56}
]
```
<!--
## <span id="eventhubs">Azure Event Hubs</span>

* **Connection String**: This can be found in 'Shared access policies' in your Event Hubs instance. Also for the 'EntityPath', it could be found by clicking into your Event Hubs instance and clicking at 'Event Hubs' in 'Entities' blade. Items that listed can be input as EntityPath. 

* **Consumer Group**: A [consumer group](../../event-hubs/event-hubs-features.md#consumer-groups) is a view (state, position, or offset) of an entire event hub.
Event Hubs use the latest offset of a consumer group to consume (subscribe from) the data from data source. Therefore a dedicated consumer group should be created for one data feed in your Metrics Advisor instance.

* **Timestamp**: Metrics Advisor uses the Event Hubs timestamp as the event timestamp if the user data source does not contain a timestamp field.
The timestamp field must match one of these two formats:

    * "YYYY-MM-DDTHH:MM:SSZ" format;

    * Number of seconds or milliseconds from the epoch of 1970-01-01T00:00:00Z.

    No matter which timestamp field it left aligns to granularity.For example, if timestamp is "2019-01-01T00:03:00Z", granularity is 5 minutes, then Metrics Advisor aligns the timestamp to "2019-01-01T00:00:00Z". If the event timestamp is "2019-01-01T00:10:00Z",  Metrics Advisor uses the timestamp directly without any alignment.
-->
## <a name="span-idsqlazure-sql-database--sql-serverspan"></a><span id="sql">Azure SQL Database | SQL Server</span>

* **Chaîne de connexion** : Metrics Advisor accepte une [chaîne de connexion de style ADO.NET](/dotnet/framework/data/adonet/connection-string-syntax) pour la source de données SQL Server.

    Exemple de chaîne de connexion :

    ```
    Data Source=db-server.database.windows.net:[port];initial catalog=[database];User ID=[username];Password=[password];Connection Timeout=10ms;
    ```

* **Requête** : Une requête SQL permettant d’obtenir et de formuler des données dans des données de séries chronologiques multidimensionnelles. Vous pouvez utiliser une variable `@StartTime` dans votre requête pour obtenir de l’aide sur l’obtention de la valeur de mesures attendue.

  * `@StartTime` : une valeur DateTime au format `yyyy-MM-dd HH:mm:ss`

    Exemple de requête :
    
    ``` mssql
    select StartDate, JobStatusId, COUNT(*) AS JobNumber from IngestionJobs WHERE and StartDate = @StartTime
    ```
    
    Requête réelle exécutée pour la tranche de données du 12/12/2019 :
    
    ``` mssql
    select StartDate, JobStatusId, COUNT(*) AS JobNumber from IngestionJobs WHERE and StartDate = '2019-12-12 00:00:00'
    ```

## <a name="span-idtableazure-table-storagespan"></a><span id="table">Stockage de tables Azure</span>

* **Chaîne de connexion** : Consultez [Afficher et copier une chaîne de connexion](../../storage/common/storage-account-keys-manage.md?tabs=azure-portal&toc=%2fazure%2fstorage%2ftables%2ftoc.json#view-account-access-keys) pour plus d’informations sur la façon de récupérer la chaîne de connexion à partir du Stockage Table Azure.

* **Nom de la table** : Spécifiez une table à interroger. Il se trouve dans votre instance de compte de Stockage Azure. Cliquez sur **Tables** dans la section **Service de table**.

* **Requête** Vous pouvez utiliser le `@StartTime` dans votre requête. `@StartTime` est remplacé par une chaîne de format aaaa-MM-jjThh:mm:ss dans le script.

    ``` mssql
    let StartDateTime = datetime(@StartTime); let EndDateTime = StartDateTime + 1d; 
    SampleTable | where Timestamp >= StartDateTime and Timestamp < EndDateTime | project Timestamp, Market, RPM
    ```

## <a name="span-ideselasticsearchspan"></a><span id="es">Elasticsearch</span>

* **Hôte** : spécifiez l’hôte principal du cluster Elasticsearch.
* **Port** : spécifiez le port principal du cluster Elasticsearch.
* **En-tête d’autorisation** : spécifiez la valeur de l’en-tête Authorization du cluster Elasticsearch.
* **Requête** : spécifiez la requête pour obtenir des données. L’espace réservé @StartTime est pris en charge. (Par exemple, lorsque les données de 2020-06-21T00:00:00Z sont ingérées, @StartTime = 2020-06-21T00:00:00)

## <a name="span-idhttphttp-requestspan"></a><span id="http">Requête HTTP</span>

* **URL de requête** : Une URL HTTP qui peut retourner un JSON. Les espaces réservés %A,%m,%d,%h,%M sont pris en charge : %A = année au format aaaa, %m = mois au format MM, %j = jour au format jj, %h = heure au format HH, %M = minute au format mm. Par exemple : `http://microsoft.com/ProjectA/%Y/%m/X_%Y-%m-%d-%h-%M`.
* **Méthode de requête HTTP** : Utilisez GET ou POST.
* **En-tête de requête** : Ajout possible de l’authentification de base. 
* **Charge utile de la requête** : Seule la charge utile JSON est prise en charge. L’espace réservé @StartTime est pris en charge dans la charge utile. La réponse doit être au format JSON suivant : [{"timestamp": "2018-01-01T00:00:00Z", "market":"en-us", "count":11, "revenue":1.23}, {"timestamp": "2018-01-01T00:00:00Z", "market":"zh-cn", "count":22, "revenue":4.56}].(Par ex. lorsque les données de 2020-06-21T00:00:00Z sont ingérées, @StartTime = 2020-06-21T00:00:00.0000000+00:00)

## <a name="span-idinfluxdbinfluxdb-influxqlspan"></a><span id="influxdb">InfluxDB (InfluxQL)</span>

* **Chaîne de connexion** : La chaîne de connexion pour accéder à votre InfluxDB.
* **Base de données** : La base de données à interroger.
* **Requête** : Une requête SQL permettant d’obtenir et de formuler des données dans des données de séries chronologiques multidimensionnelles pour ingestion.
* **Nom d’utilisateur** : Facultatif pour l’authentification. 
* **Mot de passe** : Facultatif pour l’authentification. 

## <a name="span-idmongodbmongodbspan"></a><span id="mongodb">MongoDB</span>

* **Chaîne de connexion** : La chaîne de connexion pour accéder à votre MongoDB.
* **Base de données** : La base de données à interroger.
* **Commande** : Une commande permettant d’obtenir et de formuler des données dans des données de séries chronologiques multidimensionnelles pour ingestion.

## <a name="span-idmysqlmysqlspan"></a><span id="mysql">MySQL</span>

* **Chaîne de connexion** : La chaîne de connexion pour accéder à votre base de données MySQL.
* **Requête** : Une requête SQL permettant d’obtenir et de formuler des données dans des données de séries chronologiques multidimensionnelles pour ingestion.

## <a name="span-idpgsqlpostgresqlspan"></a><span id="pgsql">PostgreSQL</span>

* **Chaîne de connexion** : La chaîne de connexion pour accéder à votre base de données PostgreSQL.
* **Requête** : Une requête SQL permettant d’obtenir et de formuler des données dans des données de séries chronologiques multidimensionnelles pour ingestion.

## <a name="next-steps"></a>Étapes suivantes

* En attendant que vos données de métrique soient ingérées dans le système, découvrez [comment gérer des configurations de flux de données](how-tos/manage-data-feeds.md).
* Lorsque vos données de métrique sont ingérées, vous pouvez [Configurer les métriques et ajuster la configuration de détection](how-tos/configure-metrics.md).