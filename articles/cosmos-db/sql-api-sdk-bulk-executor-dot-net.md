---
title: 'Azure Cosmos DB : API .NET Bulk Executor, SDK et ressources'
description: Tout savoir sur l’API .NET et le SDK de l’exécuteur en bloc, notamment les dates de publication, les dates de déclassement et les modifications apportées entre chaque version du SDK .NET de l’exécuteur en bloc Azure Cosmos DB.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: dotnet
ms.topic: reference
ms.date: 08/12/2020
ms.author: anfeldma
ms.openlocfilehash: 21b3a5024b7b38f3f284b7213853c7089838930f
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93097207"
---
# <a name="net-bulk-executor-library-download-information"></a>Bibliothèque d’exécuteurs en bloc .NET : Télécharger des informations 
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [Kit de développement logiciel (SDK) .NET v3](sql-api-sdk-dotnet-standard.md)
> * [SDK .NET v2](sql-api-sdk-dotnet.md)
> * [SDK .NET Core v2](sql-api-sdk-dotnet-core.md)
> * [SDK .NET Change Feed v2](sql-api-sdk-dotnet-changefeed.md)
> * [Node.JS](sql-api-sdk-node.md)
> * [SDK Java v4](sql-api-sdk-java-v4.md)
> * [SDK Java Async v2](sql-api-sdk-async-java.md)
> * [SDK Java Sync v2](sql-api-sdk-java.md)
> * [Spring Data v2](sql-api-sdk-java-spring-v2.md)
> * [Spring Data v3](sql-api-sdk-java-spring-v3.md)
> * [Spark Connector](sql-api-sdk-java-spark.md)
> * [Python](sql-api-sdk-python.md)
> * [REST](/rest/api/cosmos-db/)
> * [API REST Resource Provider](/rest/api/cosmos-db-resource-provider/)
> * [SQL](./sql-query-getting-started.md)
> * [Exécuteur en bloc – .NET v2](sql-api-sdk-bulk-executor-dot-net.md)
> * [Exécuteur en bloc – Java](sql-api-sdk-bulk-executor-java.md)

| |  |
|---|---|
| **Description**| La bibliothèque de l’exécuteur en bloc .NET permet aux applications clientes d’effectuer des opérations en bloc sur des comptes Azure Cosmos DB. Cette bibliothèque fournit les espaces de noms BulkImport, BulkUpdate et BulkDelete. Le module BulkImport peut ingérer en bloc des documents de manière optimisée afin que le débit provisionné pour une collection soit consommé dans ses limites maximales. Le module BulkUpdate peut mettre à jour en bloc des données existantes dans des conteneurs Azure Cosmos sous forme de correctifs. Le module BulkDelete peut supprimer en bloc des documents de manière optimisée afin que le débit provisionné pour une collection soit consommé dans ses limites maximales.|
|**Téléchargement du Kit de développement logiciel (SDK)**| [NuGet](https://www.nuget.org/packages/Microsoft.Azure.CosmosDB.BulkExecutor/) |
| **Bibliothèque d’exécuteurs en bloc dans GitHub**| [GitHub](https://github.com/Azure/azure-cosmosdb-bulkexecutor-dotnet-getting-started)|
|**Documentation de l’API**|[Documentation de référence sur l’API .NET](/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor?preserve-view=true&view=azure-dotnet)|
|**Prise en main**|[Bien démarrer avec le kit SDK .NET de la bibliothèque de l’exécuteur en bloc](bulk-executor-dot-net.md)|
| **Infrastructure actuellement prise en charge**| Microsoft .NET Framework 4.5.2, 4.6.1 et .NET Standard 2.0 |

> [!NOTE]
> Si vous utilisez l’Exécuteur en bloc, consultez la version 3.x la plus récente du [Kit de développement logiciel (SDK) .NET](tutorial-sql-api-dotnet-bulk-import.md), qui a intégré l’Exécuteur en bloc dans le Kit de développement logiciel (SDK). 

## <a name="release-notes"></a>Notes de publication

### <a name="241-preview"></a><a name="2.4.1-preview"></a>2.4.1-preview

* Correction de TotalElapsedTime dans la réponse de BulkDelete pour mesurer correctement la durée totale, incluant les nouvelles tentatives.

### <a name="240-preview"></a><a name="2.4.0-preview"></a>2.4.0-preview

* Dépendance du kit SDK remplacée par >= 2.5.1

### <a name="230-preview2"></a><a name="2.3.0-preview2"></a>2.3.0-preview2

* Ajout de la prise en charge de l’exécuteur graphique en masse pour accepter ttl sur les sommets et les bords

### <a name="220-preview2"></a><a name="2.2.0-preview2"></a>2.2.0-preview2

* Correction d’un problème qui causait des exceptions lors de la mise à l’échelle élastique d’Azure Cosmos DB lors de l’exécution en mode Passerelle. Ce correctif le rend fonctionnellement équivalent à la version 1.4.1.

### <a name="210-preview2"></a><a name="2.1.0-preview2"></a>2.1.0-preview2

* Ajout de la prise en charge de BulkDelete pour les comptes d’API SQL, afin d’accepter les tuples d’ID de document, de clé de partition à supprimer. Cette modification le rend fonctionnellement équivalent à la version 1.4.0.

### <a name="200-preview2"></a><a name="2.0.0-preview2"></a>2.0.0-preview2

* Ajout de MongoBulkExecutor pour prendre en charge .NET Standard 2.0. Cette fonctionnalité le rend fonctionnellement équivalent à la version 1.3.0, avec l’ajout de la prise en charge de .NET Standard 2.0 comme framework cible.

### <a name="200-preview"></a><a name="2.0.0-preview"></a>2.0.0-preview

* Ajout de .NET Standard 2.0 comme l’un des frameworks cibles pris en charge pour que la bibliothèque d’exécuteurs en bloc fonctionne avec les applications .NET Core.

### <a name="189"></a><a name="1.8.9"></a>1.8.9

* Correction d’un problème avec BulkDeleteAsync lorsque des valeurs avec des guillemets d’échappement étaient passées en tant que paramètres d’entrée.

### <a name="188"></a><a name="1.8.8"></a>1.8.8

* Correction d’un problème sur MongoBulkExecutor qui augmentait de façon inattendue la taille du document en ajoutant un remplissage et, dans certains cas, en dépassant la limite de taille de document maximale.

### <a name="187"></a><a name="1.8.7"></a>1.8.7

* Correction d’un problème avec BulkDeleteAsync lorsque la collection a des chemins de clés de partition imbriqués.

### <a name="186"></a><a name="1.8.6"></a>1.8.6

* MongoBulkExecutor implémente désormais IDisposable et doit être supprimé après avoir été utilisé.

### <a name="185"></a><a name="1.8.5"></a>1.8.5

* Suppression du verrou sur la version du SDK. Le package est désormais dépendant du kit de développement logiciel de version 2.5.1 ou ultérieure.

### <a name="184"></a><a name="1.8.4"></a>1.8.4

* Correction du traitement des identificateurs lors de l’appel de BulkImport avec une liste d’objets POCO contenant des valeurs numériques.

### <a name="183"></a><a name="1.8.3"></a>1.8.3

* Correction de TotalElapsedTime dans la réponse de BulkDelete pour mesurer correctement la durée totale, incluant les nouvelles tentatives.

### <a name="182"></a><a name="1.8.2"></a>1.8.2

* Correction d'une consommation élevée du processeur dans certains scénarios.
* Le suivi utilise désormais TraceSource. Les utilisateurs peuvent définir des écouteurs pour la source `BulkExecutorTrace`.
* Correction d’un scénario rare susceptible d'entraîner un verrouillage lors de l’envoi de documents d'environ 2 Mo.

### <a name="160"></a><a name="1.6.0"></a>1.6.0

* Mise à jour d’exécuteur en bloc pour utiliser la dernière version du Kit de développement logiciel (SDK) Azure Cosmos DB.NET (2.4.0)

### <a name="150"></a><a name="1.5.0"></a>1.5.0

* Ajout de la prise en charge de l’exécuteur graphique en masse pour accepter ttl sur les sommets et les bords

### <a name="141"></a><a name="1.4.1"></a>1.4.1

* Correction d’un problème qui causait des exceptions lors de la mise à l’échelle élastique d’Azure Cosmos DB lors de l’exécution en mode Passerelle.

### <a name="140"></a><a name="1.4.0"></a>1.4.0

* Ajout de la prise en charge de BulkDelete pour les comptes d’API SQL, afin d’accepter les tuples d’ID de document, de clé de partition à supprimer.

### <a name="130"></a><a name="1.3.0"></a>1.3.0

* Résolution d’un problème qui provoquait un problème de mise en forme dans l’agent utilisateur utilisé par l’exécuteur en bloc.

### <a name="120"></a><a name="1.2.0"></a>1.2.0

* Améliorations des API d’importation et de mise à jour de l’exécuteur en bloc afin de s’adapter en toute transparence à la mise à l’échelle élastique de conteneur Cosmos quand le stockage dépasse la capacité actuelle sans lever d’exception.

### <a name="112"></a><a name="1.1.2"></a>1.1.2

* Amélioration de la dépendance du SDK .NET DocumentDB à la version 2.1.3.

### <a name="111"></a><a name="1.1.1"></a>1.1.1

* Résolution d’un problème qui faisait en sorte que l’exécuteur en bloc lève une erreur JSRT lors de l’importation vers des collections fixes.

### <a name="110"></a><a name="1.1.0"></a>1.1.0

* Ajout de la prise en charge de l’opération BulkDelete pour les comptes d’API SQL Azure Cosmos DB.
* Ajout de la prise en charge de l’opération BulkImport pour les comptes disposant de l’API Azure Cosmos DB pour MongoDB.
* Amélioration de la dépendance du Kit de développement logiciel (SDK) .NET DocumentDB à la version 2.0.0. 

### <a name="102"></a><a name="1.0.2"></a>1.0.2

* Ajout de la prise en charge de l’opération BulkImport pour les comptes d’API Gremlin Azure Cosmos DB.

### <a name="101"></a><a name="1.0.1"></a>1.0.1

* Correction d’un bogue mineur dans l’opération BulkImport pour les comptes d’API SQL Azure Cosmos DB.

### <a name="100"></a><a name="1.0.0"></a>1.0.0

* Ajout de la prise en charge des opérations BulkImport et BulkUpdate pour les comptes d’API SQL Azure Cosmos DB.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la bibliothèque de l’exécuteur en bloc Java, consultez l’article suivant :

[Informations sur la version et le SDK de bibliothèque de l’exécuteur en bloc Java](sql-api-sdk-bulk-executor-java.md)