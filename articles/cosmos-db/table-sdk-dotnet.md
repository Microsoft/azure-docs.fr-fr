---
title: Ressources et kit SDK de l’API .NET Table Azure Cosmos DB
description: Découvrez l’API Table Azure Cosmos DB pour .NET, notamment les dates de version, les dates de mise hors service et les modifications apportées à chaque version.
author: sakash279
ms.author: akshanka
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.devlang: dotnet
ms.topic: reference
ms.date: 08/17/2018
ms.custom: devx-track-dotnet
ms.openlocfilehash: ea5f3fca18e6a5781bf46112134ae4efc7159106
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93101202"
---
# <a name="azure-cosmos-db-table-net-api-download-and-release-notes"></a>API .NET Table Azure Cosmos DB : Téléchargement et notes de publication
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

> [!div class="op_single_selector"]
> * [.NET](table-sdk-dotnet.md)
> * [.NET Standard](table-sdk-dotnet-standard.md)
> * [Java](table-sdk-java.md)
> * [Node.JS](table-sdk-nodejs.md)
> * [Python](table-sdk-python.md)

|   |   |
|---|---|
|**Téléchargement du Kit de développement logiciel (SDK)**|[NuGet](https://www.nuget.org/packages/Microsoft.Azure.CosmosDB.Table)|
|**Démarrage rapide**|[Azure Cosmos DB : Créer une application avec .NET et l’API Table](create-table-dotnet.md)|
|**Didacticiel**|[Azure Cosmos DB : Développer avec l’API Table dans .NET](tutorial-develop-table-dotnet.md)|
|**Infrastructure actuellement prise en charge**|[Microsoft .NET Framework 4.5.1](https://www.microsoft.com/en-us/download/details.aspx?id=40779)|

> [!IMPORTANT]
> Le kit de développement logiciel (SDK) .NET Framework [Microsoft.Azure.CosmosDB.Table](https://www.nuget.org/packages/Microsoft.Azure.CosmosDB.Table) est en mode maintenance et sera bientôt déprécié. Mettez à niveau vers la nouvelle bibliothèque .NET Standard [Microsoft.Azure.Cosmos.Table](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table) pour continuer à bénéficier des dernières fonctionnalités prises en charge par l’API Table.

> Si vous avez créé un compte d’API Table dans la préversion, créez un [nouveau compte d’API Table](create-table-dotnet.md#create-a-database-account) pour utiliser les Kits de développement logiciels (SDK) mis à la disposition générale pour l’API Table.
>

## <a name="release-notes"></a>Notes de publication

### <a name="212"></a><a name="2.1.2"></a>2.1.2

* Résolution des bogues

### <a name="210"></a><a name="2.1.0"></a>2.1.0

* Résolution des bogues

### <a name="200"></a><a name="2.0.0"></a>2.0.0

* Ajout de la prise en charge des écritures multirégions
* Dépendances de package NuGet corrigées sur Microsoft.Azure.DocumentDB, Microsoft.OData.Core, Microsoft.OData.Edm et Microsoft.Spatial

### <a name="113"></a><a name="1.1.3"></a>1.1.3

* Dépendances de package NuGet corrigées sur Microsoft.Azure.Storage.Common et Microsoft.Azure.DocumentDB.
* Résolution des bogues dans la sérialisation de table lorsque JsonConvert.DefaultSettings est configuré.

### <a name="111"></a><a name="1.1.1"></a>1.1.1

* Validation supplémentaire pour les ETAGs incorrectement formés en mode direct.
* Bogue de requête LINQ corrigé en mode de passerelle.
* API synchrones désormais exécutées sur le pool de threads avec SynchronizationContext.

### <a name="110"></a><a name="1.1.0"></a>1.1.0

* Ajouter TableQueryMaxItemCount, TableQueryEnableScan, TableQueryMaxDegreeOfParallelism et TableQueryContinuationTokenLimitInKb à TableRequestOptions
* Correctifs de bogues

### <a name="100"></a><a name="1.0.0"></a>1.0.0

* Version en disponibilité générale

### <a name="090-preview"></a><a name="0.1.0-preview"></a>0.9.0 - préversion

* Version préliminaire initiale

## <a name="release-and-retirement-dates"></a>Dates de publication et de retrait

Microsoft envoie une notification au moins **12 mois** avant le retrait d’un Kit de développement logiciel (SDK) pour faciliter la transition vers une version plus récente/prise en charge.

La bibliothèque `Microsoft.Azure.CosmosDB.Table` n’est actuellement disponible que pour .NET Framework. Elle est en mode de maintenance et sera bientôt déconseillée. Les nouvelles fonctionnalités et optimisations sont uniquement ajoutées à la bibliothèque .NET Standard [Microsoft.Azure.Cosmos.Table](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table). Il est donc recommandé de mettre à niveau vers [Microsoft.Azure.Cosmos.Table](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table).

Le package de préversion [WindowsAzure.Storage-PremiumTable](https://www.nuget.org/packages/WindowsAzure.Storage-PremiumTable/0.1.0-preview) a été déconseillé. Le kit de développement logiciel (SDK) WindowsAzure.Storage-PremiumTable sera retiré le 15 novembre 15, 2018 ; à compter de cette date, les requêtes envoyées au kit de développement logiciel (SDK) ne seront plus autorisées.

| Version | Date de sortie | Date de suppression |
| --- | --- | --- |
| [2.1.2](#2.1.2) |16 septembre 2019| |
| [2.1.0](#2.1.0) |22 janvier 2019|01 avril 2020 |
| [2.0.0](#2.0.0) |26 septembre 2018|01 mars 2020 |
| [1.1.3](#1.1.3) |17 juillet 2018|01 décembre 2019 |
| [1.1.1](#1.1.1) |26 mars 2018|01 décembre 2019 |
| [1.1.0](#1.1.0) |21 février 2018|01 décembre 2019 |
| [1.0.0](#1.0.0) |15 novembre 2017|15 novembre 2019 |
| 0.9.0 - préversion |11 novembre 2017 |11 novembre 2019 |

## <a name="troubleshooting"></a>Dépannage

Si vous obtenez l’erreur 

```
Unable to resolve dependency 'Microsoft.Azure.Storage.Common'. Source(s) used: 'nuget.org', 
'CliFallbackFolder', 'Microsoft Visual Studio Offline Packages', 'Microsoft Azure Service Fabric SDK'`
```

lorsque vous tentez d’utiliser le package Microsoft.Azure.CosmosDB.Table, vous avez deux options pour résoudre le problème :

* Utilisez la console de gestion des packages pour installer le package Microsoft.Azure.CosmosDB.Table et ses dépendances. Pour ce faire, saisissez la commande suivante dans la console du gestionnaire de packages pour votre solution. 

    ```powershell
    Install-Package Microsoft.Azure.CosmosDB.Table -IncludePrerelease
    ```

    
* À l’aide de votre outil de gestion de packages NuGet préféré, installez le package NuGet de Microsoft.Azure.Storage.Common Nuget package avant d’installer Microsoft.Azure.CosmosDB.Table.

## <a name="faq"></a>Questions fréquentes (FAQ)

[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="see-also"></a>Voir aussi

Pour plus d’informations sur l’API Table Azure Cosmos DB, consultez [Présentation d’Azure Cosmos DB : API Table](table-introduction.md). 
