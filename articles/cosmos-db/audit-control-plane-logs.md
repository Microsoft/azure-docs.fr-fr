---
title: Guide pratique pour auditer les opérations de plan de contrôle Azure Cosmos DB
description: Découvrez comment auditer les opérations de plan de contrôle telles que l’ajout d’une région, la mise à jour du débit, le basculement de région, l’ajout d’un réseau virtuel, etc., dans Azure Cosmos DB
author: SnehaGunda
ms.service: cosmos-db
ms.topic: how-to
ms.date: 10/05/2020
ms.author: sngun
ms.openlocfilehash: a0feaf4a984f40ddee7a30291fe0a8f671b6512a
ms.sourcegitcommit: 295db318df10f20ae4aa71b5b03f7fb6cba15fc3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/15/2020
ms.locfileid: "94636841"
---
# <a name="how-to-audit-azure-cosmos-db-control-plane-operations"></a>Guide pratique pour auditer les opérations de plan de contrôle Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Dans Azure Cosmos DB, le plan de contrôle est un service RESTful qui vous permet d’effectuer un ensemble diversifié d’opérations sur le compte Azure Cosmos. Il expose un modèle de ressource publique (par exemple, une base de données ou un compte) et différentes opérations aux utilisateurs finaux pour effectuer des actions sur le modèle de ressource. Les opérations de plan de contrôle incluent les modifications apportées au compte ou au conteneur Azure Cosmos. Par exemple, des opérations telles que la création d’un compte Azure Cosmos, l’ajout d’une région, la mise à jour du débit, le basculement de région, l’ajout d’un réseau virtuel, etc., sont des opérations de plan de contrôle. Cet article explique comment auditer les opérations de plan de contrôle dans Azure Cosmos DB. Vous pouvez exécuter les opérations du plan de contrôle sur les comptes Azure Cosmos à l’aide d’Azure CLI, de PowerShell ou du Portail Azure, tandis que pour les conteneurs, vous devez utiliser Azure CLI ou PowerShell.

Voici quelques exemples de scénarios dans lesquels l’audit des opérations du plan de contrôle est utile :

* Vous souhaitez recevoir une alerte lorsque les règles de pare-feu pour votre compte Azure Cosmos sont modifiées. L’alerte est nécessaire pour détecter les modifications non autorisées des règles qui régissent la sécurité réseau de votre compte Azure Cosmos et agir rapidement.

* Vous souhaitez recevoir une alerte si une région est ajoutée ou supprimée de votre compte Azure Cosmos. L’ajout ou la suppression de régions a des implications sur les exigences en matière de facturation et de souveraineté des données. Cette alerte vous permet de détecter l’ajout ou la suppression accidentels de la région sur votre compte.

* Vous souhaitez obtenir plus de détails dans les journaux de diagnostic sur ce qui a changé. Par exemple, un réseau virtuel a été modifié.

## <a name="disable-key-based-metadata-write-access"></a>Désactiver l’accès en écriture aux métadonnées basé sur les clés

Avant d’auditer les opérations de plan de contrôle dans Azure Cosmos DB, désactivez l’accès en écriture aux métadonnées basé sur les clés sur votre compte. Quand l’accès en écriture aux métadonnées basé sur les clés est désactivé, les clients qui se connectent au compte Azure Cosmos par le biais de clés de compte ne peuvent pas accéder au compte. Vous pouvez désactiver l’accès en écriture en affectant à la propriété `disableKeyBasedMetadataWriteAccess` la valeur true. Une fois cette propriété définie, les modifications apportées à une ressource peuvent être effectuées par un utilisateur qui a le rôle Azure et les informations d’identification appropriés. Pour en savoir plus sur la définition de cette propriété, consultez l’article [Prévention des modifications à partir des SDK](role-based-access-control.md#prevent-sdk-changes). 

Une fois le `disableKeyBasedMetadataWriteAccess` activé, si les clients basés sur le kit de développement logiciel (SDK) exécutent des opérations de création ou de mise à jour, une erreur *« l’opération « POST » sur la ressource « ContainerNameorDatabaseName » n’est pas autorisée via le point de terminaison Azure Cosmos DB* est retournée. Vous devez activer l’accès à ces opérations pour votre compte ou effectuer les opérations de création/mise à jour via Azure Resource Manager, Azure CLI ou Azure PowerShell. Pour revenir en arrière, définissez la valeur de disableKeyBasedMetadataWriteAccess sur **false** à l’aide de Azure CLI comme décrit dans l’article [Éviter les modifications dans le kit de développement logiciel (SDK) Cosmos](role-based-access-control.md#prevent-sdk-changes). Veillez à remplacer la valeur de `disableKeyBasedMetadataWriteAccess` par false au lieu de true.

Tenez compte des points suivants lors de la désactivation de l’accès en écriture aux métadonnées :

* Vérifiez que vos applications n’effectuent pas d’appels de métadonnées qui changent les ressources mentionnées plus haut (par exemple, création d’une collection, mise à jour du débit, etc.) à l’aide du SDK ou de clés de compte.

* Comme le portail Azure utilise des clés de compte pour les opérations de métadonnées, ces opérations sont bloquées. Vous pouvez également utiliser Azure CLI, les kits SDK ou les déploiements de modèle Resource Manager pour effectuer ces opérations.

## <a name="enable-diagnostic-logs-for-control-plane-operations"></a>Activer les journaux de diagnostic pour les opérations de plan de contrôle

Vous pouvez activer les journaux de diagnostic pour les opérations de plan de contrôle à l’aide du portail Azure. Après l’activation, les journaux de diagnostic enregistrent l’opération en tant que paire d’événements de démarrage et de fin avec des détails pertinents. Par exemple, les *RegionFailoverStart* et *RegionFailoverComplete* terminent l’événement de basculement de la région.

Effectuez les étapes suivantes pour activer la journalisation sur les opérations du plan de contrôle :

1. Connectez-vous au [portail Azure](https://portal.azure.com) et accédez à votre compte Azure Cosmos.

1. Ouvrez le volet **Paramètres de diagnostic**, puis fournissez un **Nom** pour les journaux à créer.

1. Sélectionnez **ControlPlaneRequests** comme type de journal et sélectionnez l’option **Envoyer à Log Analytics**.

Vous pouvez également stocker les journaux dans un compte de stockage ou les transmettre à un hub d’événements. Cet article explique comment envoyer des journaux à Log Analytics, puis les interroger. Une fois l’activation effectuée, la prise en compte des journaux de diagnostic prend quelques minutes. Toutes les opérations de plan de contrôle effectuées par la suite peuvent faire l’objet d’un suivi. La capture d’écran suivante montre comment activer les journaux du plan de contrôle :

:::image type="content" source="./media/audit-control-plane-logs/enable-control-plane-requests-logs.png" alt-text="Activer la journalisation des demandes du plan de contrôle":::

## <a name="view-the-control-plane-operations"></a>Afficher les opérations de plan de contrôle

Après avoir activé la journalisation, suivez les étapes ci-dessous pour effectuer le suivi des opérations d’un compte spécifique :

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Ouvrez l’onglet **Superviser** à partir de la barre de navigation de gauche, puis sélectionnez le volet **Journaux**. Une interface utilisateur s’ouvre, dans laquelle vous pouvez facilement exécuter des requêtes avec ce compte spécifique dans l’étendue. Exécutez la requête suivante pour voir les journaux du plan de contrôle :

   ```kusto
   AzureDiagnostics
   | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="ControlPlaneRequests"
   | where TimeGenerated >= ago(1h)
   ```

Les captures d’écran suivantes capturent des journaux lorsqu’un niveau de cohérence est modifié par un compte Azure Cosmos :

:::image type="content" source="./media/audit-control-plane-logs/add-ip-filter-logs.png" alt-text="Journaux du plan de contrôle lors de l’ajout d’un réseau virtuel":::

Les captures d’écran suivantes capturent les journaux lorsque l’espace de clés ou une table d’un compte Cassandra sont créés et lorsque le débit est mis à jour. Les journaux du plan de contrôle pour les opérations de création et de mise à jour sur la base de données et le conteneur sont consignés séparément, comme le montre la capture d’écran suivante :

:::image type="content" source="./media/audit-control-plane-logs/throughput-update-logs.png" alt-text="Journaux du plan de contrôle lors de la mise à jour du débit":::

## <a name="identify-the-identity-associated-to-a-specific-operation"></a>Identifier l’identité associée à une opération spécifique

Si vous souhaitez effectuer un débogage supplémentaire, vous pouvez identifier une opération spécifique dans le **journal d’activité** à l’aide de l’ID d’activité ou de l’horodatage de l’opération. L’horodatage est utilisé pour certains clients Resource Manager dans les cas où l’ID d’activité n’est pas explicitement passée. Le journal d’activité fournit des détails sur l’identité sous laquelle l’opération a été lancée. La capture d’écran suivante montre comment utiliser l’ID d’activité et rechercher les opérations qui lui sont associées dans le journal d’activité :

:::image type="content" source="./media/audit-control-plane-logs/find-operations-with-activity-id.png" alt-text="Utiliser l’ID d’activité et rechercher les opérations":::

## <a name="control-plane-operations-for-azure-cosmos-account"></a>Opérations du plan de contrôle pour le compte Azure Cosmos

Voici les opérations du plan de contrôle accessibles au niveau du compte. La plupart des opérations sont suivies au niveau du compte. Ces opérations sont accessibles en tant que métriques dans Azure Monitor :

* Région ajoutée
* Région supprimée
* Compte supprimé
* Région basculée
* Compte créé
* Réseau virtuel supprimé
* Paramètres réseau de compte mis à jour
* Paramètres de réplication de compte mis à jour
* Clés de compte mises à jour
* Paramètres de sauvegarde du compte mis à jour
* Paramètres de diagnostic du compte mis à jour

## <a name="control-plane-operations-for-database-or-containers"></a>Opérations du plan de contrôle pour la base de données ou les conteneurs

Voici les opérations du plan de contrôle accessibles au niveau de la base de données et di conteneur. Ces opérations sont accessibles en tant que métriques dans Azure Monitor :

* Base de données SQL créée
* Base de données SQL mise à jour
* Débit de base de données SQL mis à jour
* Base de données SQL supprimée
* Conteneur SQL créé
* Conteneur SQL mis à jour
* Débit de conteneur SQL mis à jour
* Conteneur SQL supprimé
* Espace de clés Cassandra créé
* Espace de clés Cassandra mis à jour
* Débit d’espace de clés Cassandra mis à jour
* Espace de clés Cassandra supprimé
* Table Cassandra créée
* Table Cassandra mise à jour
* Débit de table Cassandra mis à jour
* Table Cassandra supprimée
* Base de données Gremlin créée
* Base de données Gremlin mise à jour
* Débit de base de données Gremlin mis à jour
* Base de données Gremlin supprimée
* Graphique Gremlin créé
* Graphique Gremlin mis à jour
* Débit de graphique Gremlin mis à jour
* Graphique Gremlin supprimé
* Base de données Mongo créée
* Base de données Mongo mise à jour
* Débit de base de données Mongo mis à jour
* Base de données Mongo supprimée
* Collection Mongo créée
* Collection Mongo mise à jour
* Débit de collection Mongo mis à jour
* Collection Mongo supprimée
* Table AzureTable créée
* Table AzureTable mise à jour
* Débit de table AzureTable mis à jour
* Table AzureTable supprimée

## <a name="diagnostic-log-operations"></a>Opérations sur le journal de diagnostic

Voici les noms d’opération dans les journaux de diagnostic pour différentes opérations :

* RegionAddStart, RegionAddComplete
* RegionRemoveStart, RegionRemoveComplete
* AccountDeleteStart, AccountDeleteComplete
* RegionFailoverStart, RegionFailoverComplete
* AccountCreateStart, AccountCreateComplete
* AccountUpdateStart, AccountUpdateComplete
* VirtualNetworkDeleteStart, VirtualNetworkDeleteComplete
* DiagnosticLogUpdateStart, DiagnosticLogUpdateComplete

Pour les opérations spécifiques à l’API, l’opération est nommée selon le format suivant :

* GenreApi + TypeRessourceGenreApi + TypeOpération
* GenreApi + TypeRessourceGenreApi + « Débit » + TypeOpération

**Exemple** 

* CassandraKeyspacesCreate
* CassandraKeyspacesUpdate
* CassandraKeyspacesThroughputUpdate
* SqlContainersUpdate

La propriété *ResourceDetails* contient l’intégralité du corps de la ressource en tant que charge utile de la demande, ainsi que toutes les propriétés requises pour la mise à jour.

## <a name="diagnostic-log-queries-for-control-plane-operations"></a>Requêtes de journaux de diagnostic pour les opérations de plan de contrôle

Voici quelques exemples pour obtenir des journaux de diagnostic pour les opérations de plan de contrôle :

```kusto
AzureDiagnostics 
| where Category startswith "ControlPlane"
| where OperationName contains "Update"
| project httpstatusCode_s, statusCode_s, OperationName, resourceDetails_s, activityId_g
```

```kusto
AzureDiagnostics 
| where Category =="ControlPlaneRequests"
| where TimeGenerated >= todatetime('2020-05-14T17:37:09.563Z')
| project TimeGenerated, OperationName, apiKind_s, apiKindResourceType_s, operationType_s, resourceDetails_s
```

```kusto
AzureDiagnostics 
| where Category =="ControlPlaneRequests"
| where  OperationName startswith "SqlContainersUpdate"
```

```kusto
AzureDiagnostics 
| where Category =="ControlPlaneRequests"
| where  OperationName startswith "SqlContainersThroughputUpdate"
```

Lancez une requête pour obtenir l’activityId et l’appelant à l’origine de l’opération de suppression de conteneur :

```kusto
(AzureDiagnostics
| where Category == "ControlPlaneRequests"
| where OperationName == "SqlContainersDelete"
| where TimeGenerated >= todatetime('9/3/2020, 5:30:29.300 PM')
| summarize by activityId_g )
| join (
AzureActivity
| parse HTTPRequest with * "clientRequestId\": \"" activityId_g "\"" * 
| summarize by Caller, HTTPRequest, activityId_g)
on activityId_g
| project Caller, activityId_g
```

Lancez une requête pour obtenir les mises à jour de l’index ou de la durée de vie. Vous pouvez ensuite comparer la sortie de cette requête avec une mise à jour antérieure pour voir les modifications apportées à l’index ou à la durée de vie.

```Kusto
AzureDiagnostics
| where Category =="ControlPlaneRequests"
| where  OperationName == "SqlContainersUpdate"
| project resourceDetails_s
```

**sortie :**

```json
{id:skewed,indexingPolicy:{automatic:true,indexingMode:consistent,includedPaths:[{path:/*,indexes:[]}],excludedPaths:[{path:/_etag/?}],compositeIndexes:[],spatialIndexes:[]},partitionKey:{paths:[/pk],kind:Hash},defaultTtl:1000000,uniqueKeyPolicy:{uniqueKeys:[]},conflictResolutionPolicy:{mode:LastWriterWins,conflictResolutionPath:/_ts,conflictResolutionProcedure:}
```

## <a name="next-steps"></a>Étapes suivantes

* [Explorer Azure Monitor pour Azure Cosmos DB](../azure-monitor/insights/cosmosdb-insights-overview.md?toc=/azure/cosmos-db/toc.json&bc=/azure/cosmos-db/breadcrumb/toc.json)
* [Superviser et déboguer à l’aide de métriques dans Azure Cosmos DB](use-metrics.md)
