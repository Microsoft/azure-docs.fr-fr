---
title: Surveillez les données Azure Cosmos DB à l'aide des paramètres de diagnostic Azure
description: Découvrez comment utiliser les paramètres de diagnostic Azure pour surveiller les performances et la disponibilité des données stockées dans Azure Cosmos DB
author: SnehaGunda
services: cosmos-db
ms.service: cosmos-db
ms.topic: how-to
ms.date: 10/28/2020
ms.author: sngun
ms.openlocfilehash: 18850fafd1f6cb084c9e5fdb9a24e9c4fd8bb4cc
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93097564"
---
# <a name="monitor-azure-cosmos-db-data-by-using-diagnostic-settings-in-azure"></a>Surveillez les données Azure Cosmos DB à l’aide des paramètres de diagnostic dans Azure
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Les paramètres de diagnostic dans Azure sont utilisés pour collecter les journaux des ressources. Les journaux de ressources Azure sont émis par une ressource et fournissent des données riches et fréquentes sur le fonctionnement de cette ressource. Ces journaux sont capturés par requête et sont également appelés « journaux des plans de données ». Les opérations Delete, Insert et readFeed sont des exemples d’opérations de plan de données. Le contenu de ces journaux d’activité varie en fonction du type de ressource.

Les indicateurs de performance de la plateforme et le journal d’activité sont collectés automatiquement, mais vous devez créer un paramètre de diagnostic pour collecter les journaux des ressources ou les transférer en dehors d’Azure Monitor. Vous pouvez activer le paramètre de diagnostic pour les comptes Azure Cosmos en procédant comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Accédez à votre compte Azure Cosmos. Ouvrez le volet des **Paramètres de Diagnostic** , puis sélectionnez l’option **Ajouter le paramètre de diagnostic**.

1. Dans le volet **Paramètres de Diagnostic** , remplissez le formulaire avec les détails suivants : 

    * **Name**  : Entrez un nom pour les journaux d’activité à créer.

    * Vous pouvez stocker les journaux dans **Archiver dans un compte de stockage** , **Diffuser vers un hub d’événements** ou **Envoyer à Log Analytics**

1. Lorsque vous créez un paramètre de diagnostic, vous spécifiez la catégorie des journaux à collecter. Les catégories de journaux prises en charge par Azure Cosmos DB sont répertoriées ci-dessous, ainsi que l’exemple de journal collecté par ces derniers :

 * **DataPlaneRequests** : sélectionnez cette option pour enregistrer les requêtes de back-end pour toutes les API, dont les comptes SQL, Graph, MongoDB, Cassandra et Table API dans Azure Cosmos DB. Les propriétés clés à noter sont les suivantes : `Requestcharge`, `statusCode`, `clientIPaddress`, `partitionID`, `resourceTokenPermissionId` et `resourceTokenPermissionMode`.

    ```json
    { "time": "2019-04-23T23:12:52.3814846Z", "resourceId": "/SUBSCRIPTIONS/<your_subscription_ID>/RESOURCEGROUPS/<your_resource_group>/PROVIDERS/MICROSOFT.DOCUMENTDB/DATABASEACCOUNTS/<your_database_account>", "category": "DataPlaneRequests", "operationName": "ReadFeed", "properties": {"activityId": "66a0c647-af38-4b8d-a92a-c48a805d6460","requestResourceType": "Database","requestResourceId": "","collectionRid": "","statusCode": "200","duration": "0","userAgent": "Microsoft.Azure.Documents.Common/2.2.0.0","clientIpAddress": "10.0.0.24","requestCharge": "1.000000","requestLength": "0","responseLength": "372", "resourceTokenPermissionId": "perm-prescriber-app","resourceTokenPermissionMode": "all", "resourceTokenUserRid": "","region": "East US","partitionId": "062abe3e-de63-4aa5-b9de-4a77119c59f8","keyType": "PrimaryReadOnlyMasterKey","databaseName": "","collectionName": ""}}
    ```

* **MongoRequests** : sélectionnez cette option pour enregistrer les requêtes initiées par l’utilisateur depuis le serveur front-end pour servir des demandes à l’API Azure Cosmos DB pour MongoDB. Ce type de journal n'est pas disponible pour les autres comptes d'API. Les propriétés importantes à noter sont : `Requestcharge`, `opCode`. Lorsque vous activez MongoRequests dans les journaux de diagnostic, veillez à désactiver l'option DataPlaneRequests. Vous verrez un journal pour chaque requête adressée à l'API.

    ```json
    { "time": "2019-04-10T15:10:46.7820998Z", "resourceId": "/SUBSCRIPTIONS/<your_subscription_ID>/RESOURCEGROUPS/<your_resource_group>/PROVIDERS/MICROSOFT.DOCUMENTDB/DATABASEACCOUNTS/<your_database_account>", "category": "MongoRequests", "operationName": "ping", "properties": {"activityId": "823cae64-0000-0000-0000-000000000000","opCode": "MongoOpCode_OP_QUERY","errorCode": "0","duration": "0","requestCharge": "0.000000","databaseName": "admin","collectionName": "$cmd","retryCount": "0"}}
    ```

* **CassandraRequests**  : sélectionnez cette option pour enregistrer les requêtes initiées par l'utilisateur depuis le serveur front-end pour servir les requêtes adressées à l'API Azure Cosmos DB pour Cassandra. Ce type de journal n'est pas disponible pour les autres comptes d'API. Les principales propriétés sont les suivantes : `operationName`, `requestCharge`, `piiCommandText`. Lorsque vous activez CassandraRequests dans les journaux de diagnostic, veillez à désactiver l'option DataPlaneRequests. Vous verrez un journal pour chaque requête adressée à l'API.

   ```json
   { "time": "2020-03-30T23:55:10.9579593Z", "resourceId": "/SUBSCRIPTIONS/<your_subscription_ID>/RESOURCEGROUPS/<your_resource_group>/PROVIDERS/MICROSOFT.DOCUMENTDB/DATABASEACCOUNTS/<your_database_account>", "category": "CassandraRequests", "operationName": "QuerySelect", "properties": {"activityId": "6b33771c-baec-408a-b305-3127c17465b6","opCode": "<empty>","errorCode": "-1","duration": "0.311900","requestCharge": "1.589237","databaseName": "system","collectionName": "local","retryCount": "<empty>","authorizationTokenType": "PrimaryMasterKey","address": "104.42.195.92","piiCommandText": "{"request":"SELECT key from system.local"}","userAgent": """"}}
   ```

* **QueryRuntimeStatistics** : Sélectionnez cette option pour enregistrer le texte de requête qui a été exécuté. Ce type de journal est disponible uniquement pour les comptes d’API SQL.

    ```json
    { "time": "2019-04-14T19:08:11.6353239Z", "resourceId": "/SUBSCRIPTIONS/<your_subscription_ID>/RESOURCEGROUPS/<your_resource_group>/PROVIDERS/MICROSOFT.DOCUMENTDB/DATABASEACCOUNTS/<your_database_account>", "category": "QueryRuntimeStatistics", "properties": {"activityId": "278b0661-7452-4df3-b992-8aa0864142cf","databasename": "Tasks","collectionname": "Items","partitionkeyrangeid": "0","querytext": "{"query":"SELECT *\nFROM c\nWHERE (c.p1__10 != true)","parameters":[]}"}}
    ```

* **PartitionKeyStatistics** : sélectionnez cette option pour journaliser les statistiques des clés de partition. Les statistiques sont actuellement représentées par la taille de stockage (Ko) des clés de partition. Consultez la section [Résolution des problèmes à l’aide des requêtes de diagnostic Azure](#diagnostic-queries) de cet article. Par exemple, les requêtes qui utilisent « PartitionKeyStatistics ». Le journal est émis sur les trois premières clés de partition qui occupent la plupart du stockage de données. Ce journal contient des données telles que l’ID d’abonnement, le nom de la région, le nom de la base de données, le nom de la collection, la clé de partition et la taille de stockage en Ko.

    ```json
    { "time": "2019-10-11T02:33:24.2018744Z", "resourceId": "/SUBSCRIPTIONS/<your_subscription_ID>/RESOURCEGROUPS/<your_resource_group>/PROVIDERS/MICROSOFT.DOCUMENTDB/DATABASEACCOUNTS/<your_database_account>", "category": "PartitionKeyStatistics", "properties": {"subscriptionId": "<your_subscription_ID>","regionName": "West US 2","databaseName": "KustoQueryResults","collectionname": "CapacityMetrics","partitionkey": "["CapacityMetricsPartition.136"]","sizeKb": "2048270"}}
    ```

* **PartitionKeyRUConsumption** : Ce journal indique la consommation agrégée par seconde RU/s des clés de partition. Actuellement, Azure Cosmos DB signale des clés de partition pour les comptes d’API SQL uniquement et pour les opérations de lecture/écriture de point et de procédure stockée. les autres API et types d’opération ne sont pas pris en charge. Pour les autres API, la colonne clé de partition du tableau du journal de diagnostic est vide. Ce journal contient des données telles que l’ID d’abonnement, le nom de la région, le nom de la base de données, le nom du regroupement, la clé de partition, le type d’opération et les frais de demande. Consultez la section [Résolution des problèmes à l’aide des requêtes de diagnostic Azure](#diagnostic-queries) de cet article. Par exemple, les requêtes qui utilisent « PartitionKeyRUConsumption ». 

* **ControlPlaneRequests** : Ce journal contient des détails sur les opérations de plan de contrôle telles que la création d’un compte, l’ajout ou la suppression d’une région, la mise à jour des paramètres de réplication de compte, etc. Ce type de journal est disponible pour tous les types d’API qui incluent SQL (Core), MongoDB, Gremlin, Cassandra, API Table.

* **Requêtes**  : Sélectionnez cette option pour collecter les données de métriques d’Azure Cosmos DB dans les destinations du paramètre de diagnostic. Il s’agit des mêmes données que celles collectées automatiquement dans les métriques Azure. Collectez les données de métriques avec les journaux de ressources pour analyser les deux types de données ensemble et envoyer des données de métrique en dehors d’Azure Monitor.

Pour plus d’informations sur la création d’un paramètre de diagnostic à l’aide du Portail Azure, de l’interface CLI ou de PowerShell, consultez [Créer un paramètre de diagnostic pour collecter les journaux et les indicateurs de performance de la plateforme dans Azure](../azure-monitor/platform/diagnostic-settings.md) article.


## <a name="troubleshoot-issues-with-diagnostics-queries"></a><a id="diagnostic-queries"></a> Résoudre les problèmes liés aux requêtes de diagnostic

1. Comment rechercher les opérations dont l’exécution prend plus de 3 millisecondes :

   ```Kusto
   AzureDiagnostics 
   | where toint(duration_s) > 3 and ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="DataPlaneRequests" 
   | summarize count() by clientIpAddress_s, TimeGenerated
   ```

1. Comment rechercher l’agent utilisateur qui exécute les opérations :

   ```Kusto
   AzureDiagnostics 
   | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="DataPlaneRequests" 
   | summarize count() by OperationName, userAgent_s
   ```

1. Comment rechercher les opérations durables :

   ```Kusto
   AzureDiagnostics 
   | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="DataPlaneRequests" 
   | project TimeGenerated , duration_s 
   | summarize count() by bin(TimeGenerated, 5s)
   | render timechart
   ```
    
1. Comment obtenir des statistiques de clé de partition afin d’évaluer le décalage entre les 3 premières partitions pour un compte de base de données :

   ```Kusto
   AzureDiagnostics 
   | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="PartitionKeyStatistics" 
   | project SubscriptionId, regionName_s, databaseName_s, collectionName_s, partitionKey_s, sizeKb_d, ResourceId 
   ```

1. Comment obtenir les frais de requête pour les requêtes coûteuses ?

   ```Kusto
   AzureDiagnostics
   | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="DataPlaneRequests" and todouble(requestCharge_s) > 10.0
   | project activityId_g, requestCharge_s
   | join kind= inner (
   AzureDiagnostics
   | where ResourceProvider =="MICROSOFT.DOCUMENTDB" and Category == "QueryRuntimeStatistics"
   | project activityId_g, querytext_s
   ) on $left.activityId_g == $right.activityId_g
   | order by requestCharge_s desc
   | limit 100
   ```

1. Comment trouver les opérations qui prennent en charge la plupart des RU/s ?

    ```Kusto
   AzureDiagnostics
   | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="DataPlaneRequests"
   | where TimeGenerated >= ago(2h) 
   | summarize max(responseLength_s), max(requestLength_s), max(requestCharge_s), count = count() by OperationName, requestResourceType_s, userAgent_s, collectionRid_s, bin(TimeGenerated, 1h)
   ```

1. Comment obtenir toutes les requêtes qui consomment plus de 100 RU/s jointes à des données de **DataPlaneRequests** et **QueryRunTimeStatistics**.

   ```Kusto
   AzureDiagnostics
   | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="DataPlaneRequests" and todouble(requestCharge_s) > 100.0
   | project activityId_g, requestCharge_s
   | join kind= inner (
           AzureDiagnostics
           | where ResourceProvider =="MICROSOFT.DOCUMENTDB" and Category == "QueryRuntimeStatistics"
           | project activityId_g, querytext_s
   ) on $left.activityId_g == $right.activityId_g
   | order by requestCharge_s desc
   | limit 100
   ```

1. Comment obtenir les frais de requête et la durée d’exécution d’une requête ?

   ```kusto
   AzureDiagnostics
   | where TimeGenerated >= ago(24hr)
   | where Category == "QueryRuntimeStatistics"
   | join (
   AzureDiagnostics
   | where TimeGenerated >= ago(24hr)
   | where Category == "DataPlaneRequests"
   ) on $left.activityId_g == $right.activityId_g
   | project databasename_s, collectionname_s, OperationName1 , querytext_s,requestCharge_s1, duration_s1, bin(TimeGenerated, 1min)
   ```


1. Comment obtenir la distribution pour différentes opérations ?

   ```Kusto
   AzureDiagnostics
   | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="DataPlaneRequests"
   | where TimeGenerated >= ago(2h) 
   | summarize count = count()  by OperationName, requestResourceType_s, bin(TimeGenerated, 1h) 
   ```

1. Quel est le débit maximal qui a été consommé par une partition ?

   ```Kusto
   AzureDiagnostics
   | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="DataPlaneRequests"
   | where TimeGenerated >= ago(2h) 
   | summarize max(requestCharge_s) by bin(TimeGenerated, 1h), partitionId_g
   ```

1. Comment obtenir les informations sur les clés de partition consommables par seconde ?

   ```Kusto
   AzureDiagnostics 
   | where ResourceProvider == "MICROSOFT.DOCUMENTDB" and Category == "PartitionKeyRUConsumption" 
   | summarize total = sum(todouble(requestCharge_s)) by databaseName_s, collectionName_s, partitionKey_s, TimeGenerated 
   | order by TimeGenerated asc 
   ```

1. Comment obtenir les frais de requête pour une clé de partition spécifique

   ```Kusto
   AzureDiagnostics 
   | where ResourceProvider == "MICROSOFT.DOCUMENTDB" and Category == "PartitionKeyRUConsumption" 
   | where parse_json(partitionKey_s)[0] == "2" 
   ```

1. Comment obtenir les principales clés de partition avec la plupart des RU/s consommées au cours d’une période donnée ? 

   ```Kusto
   AzureDiagnostics 
   | where ResourceProvider == "MICROSOFT.DOCUMENTDB" and Category == "PartitionKeyRUConsumption" 
   | where TimeGenerated >= datetime("11/26/2019, 11:20:00.000 PM") and TimeGenerated <= datetime("11/26/2019, 11:30:00.000 PM") 
   | summarize total = sum(todouble(requestCharge_s)) by databaseName_s, collectionName_s, partitionKey_s 
   | order by total desc
    ```

1. Comment obtenir les journaux des clés de partition dont la taille de stockage est supérieure à 8 Go ?

   ```Kusto
   AzureDiagnostics
   | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="PartitionKeyStatistics"
   | where todouble(sizeKb_d) > 800000
   ```

1. Comment obtenir des latences de réplication P99 ou P50 pour les opérations, la charge de requête ou la durée de la réponse ?

   ```Kusto
   AzureDiagnostics
   | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="DataPlaneRequests"
   | where TimeGenerated >= ago(2d)
   | summarize
   percentile(todouble(responseLength_s), 50), percentile(todouble(responseLength_s), 99), max(responseLength_s),
   percentile(todouble(requestCharge_s), 50), percentile(todouble(requestCharge_s), 99), max(requestCharge_s),
   percentile(todouble(duration_s), 50), percentile(todouble(duration_s), 99), max(duration_s),
   count()
   by OperationName, requestResourceType_s, userAgent_s, collectionRid_s, bin(TimeGenerated, 1h)
   ```
 
1. Comment se procurer les journaux ControlPlane ?
 
   N’oubliez pas d’activer l’indicateur comme décrit dans l’article [Désactiver l’accès en écriture aux métadonnées basé sur les clés](audit-control-plane-logs.md#disable-key-based-metadata-write-access) et exécutez les opérations en utilisant Azure PowerShell, l’interface CLI Azure ou Azure Resource Manager.
 
   ```Kusto  
   AzureDiagnostics 
   | where Category =="ControlPlaneRequests"
   | summarize by OperationName 
   ```

## <a name="next-steps"></a>Étapes suivantes

* [Azure Monitor pour Azure Cosmos DB](../azure-monitor/insights/cosmosdb-insights-overview.md?toc=/azure/cosmos-db/toc.json)
* [Superviser et déboguer à l’aide de métriques dans Azure Cosmos DB](use-metrics.md)
