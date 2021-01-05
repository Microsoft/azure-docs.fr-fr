---
title: 'Kit de développement logiciel (SDK) Java Azure Cosmos DB v4 pour API SQL : notes de publication et ressources'
description: Découvrez le Kit de développement logiciel (SDK) Java Azure Cosmos DB v4 pour API SQL ainsi que le Kit de développement logiciel (SDK), notamment les dates de lancement, les dates de suppression et les modifications apportées entre chaque version du Kit de développement logiciel (SDK) Java Async SQL d’Azure Cosmos DB.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: reference
ms.date: 08/12/2020
ms.author: anfeldma
ms.custom: devx-track-java
ms.openlocfilehash: a70892a1900902cdf033434fe7edfed4c56f3b74
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93097105"
---
# <a name="azure-cosmos-db-java-sdk-v4-for-core-sql-api-release-notes-and-resources"></a>Kit de développement logiciel (SDK) Java Azure Cosmos DB v4 pour API Core (SQL) : notes de publication et ressources
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

Le Kit de développement logiciel (SDK) Java Azure Cosmos DB v4 pour API Core (SQL) combine une API asynchrone et une API synchrone en un seul artefact Maven. Le Kit de développement logiciel (SDK) v4 offre des performances améliorées, de nouvelles fonctionnalités d’API et une prise en charge asynchrone basée sur le réacteur de projet et la [bibliothèque Netty](https://netty.io/). Les utilisateurs peuvent s’attendre à des performances améliorées avec le Kit de développement logiciel (SDK) Java Azure Cosmos DB v4 par rapport au [Kit de développement logiciel (SDK) Java Azure Cosmos DB v2 asynchrone](sql-api-sdk-async-java.md) et au [Kit de développement logiciel (SDK) Java Azure Cosmos DB v2 synchrone](sql-api-sdk-java.md).

> [!IMPORTANT]  
> Ces notes de publication concernent uniquement le Kit de développement logiciel (SDK) Java Azure Cosmos DB v4. Si vous utilisez actuellement une version antérieure à v4, consultez le guide [Migrer votre application pour utiliser le SDK Java v4 Azure Cosmos DB](migrate-java-v4-sdk.md) afin d’obtenir de l’aide sur la mise à niveau vers v4.
>
> Voici trois étapes pour aller plus vite.
> 1. Installez le [Runtime Java minimal pris en charge, JDK 8,](/java/azure/jdk/?view=azure-java-stable&preserve-view=true) pour pouvoir utiliser le Kit de développement logiciel (SDK).
> 2. Utilisez le [guide de démarrage rapide pour le Kit de développement logiciel (SDK) Java Azure Cosmos DB v4](./create-sql-api-java.md), qui vous permet d’accéder à l’artefact Maven et de passer en revue les requêtes Azure Cosmos DB de base.
> 3. Pour optimiser le Kit de développement logiciel (SDK) pour votre application, lisez les [conseils sur les performances](performance-tips-java-sdk-v4-sql.md) et [sur le dépannage](troubleshoot-java-sdk-v4-sql.md) relatifs au Kit de développement logiciel (SDK) Java Azure Cosmos DB v4.
>
> Les [ateliers et laboratoires Azure Cosmos DB](https://aka.ms/cosmosworkshop) sont une autre formidable ressource pour apprendre à utiliser le Kit de développement logiciel (SDK) Java Azure Cosmos DB v4.
>

## <a name="helpful-content"></a>Contenu utile

| Contenu | Lien |
|---|---|
|**Téléchargement du Kit de développement logiciel (SDK)**| [Maven](https://mvnrepository.com/artifact/com.azure/azure-cosmos) |
|**Documentation de l’API** | [Documentation de référence sur l’API Java](/java/api/overview/azure/cosmosdb/client?preserve-view=true&view=azure-java-stable) |
|**Contribution au Kit de développement logiciel (SDK)** | [Référentiel centralisé relatif au Kit de développement logiciel (SDK) Azure pour Java sur GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cosmos/azure-cosmos) | 
|**Prise en main** | [Démarrage rapide : Créer une application Java pour gérer les données de l’API SQL d’Azure Cosmos DB](./create-sql-api-java.md) <br> [Référentiel GitHub avec code de démarrage rapide](https://github.com/Azure-Samples/azure-cosmos-java-getting-started) | 
|**Exemples de code de base** | [Azure Cosmos DB : exemples Java pour l’API SQL](sql-api-java-sdk-samples.md) <br> [Référentiel GitHub avec exemple de code](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples)|
|**Application console avec flux de modification**| [Flux de modification – Exemple de Kit de développement logiciel (SDK) Java v4](create-sql-api-java-changefeed.md) <br> [Référentiel GitHub avec exemple de code](https://github.com/Azure-Samples/azure-cosmos-java-sql-app-example)| 
|**Exemple d’application web**| [Créer une application web avec le Kit de développement logiciel (SDK) Java v4](sql-api-java-application.md) <br> [Référentiel GitHub avec exemple de code](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-todo-app)|
| **Conseils sur les performances**| [Conseils en performances pour le Kit de développement logiciel (SDK) Java v4](performance-tips-java-sdk-v4-sql.md)| 
| **Dépannage** | [Résoudre les problèmes du Kit de développement logiciel (SDK) Java v4](troubleshoot-java-sdk-v4-sql.md) |
| **Migration d’un ancien Kit de développement logiciel (SDK) vers la v4** | [Migrer vers le Kit de développement logiciel (SDK) Java V4](migrate-java-v4-sdk.md) |
| **Runtime minimal pris en charge**|[JDK 8](/java/azure/jdk/?view=azure-java-stable&preserve-view=true) | 
| **Ateliers et laboratoires Azure Cosmos DB** |[Page d’accueil relative aux ateliers Cosmos DB](https://aka.ms/cosmosworkshop)

[!INCLUDE[Release notes](~/azure-sdk-for-java-cosmos-db/sdk/cosmos/azure-cosmos/CHANGELOG.md)]

## <a name="faq"></a>Questions fréquentes (FAQ)
[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)] 

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur Cosmos DB, consultez la page du service [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/).