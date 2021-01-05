---
title: Prise en charge de Stockage Table Azure dans Azure Cosmos DB
description: Découvrez comment l’API Table Azure Cosmos DB et les tables de stockage Azure fonctionnent ensemble en partageant le même modèle de données de table et les mêmes opérations.
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.topic: how-to
ms.date: 05/21/2020
author: sakash279
ms.author: akshanka
ms.reviewer: sngun
ms.openlocfilehash: 5e586ae8a6b6a4010419254ce3e380f377d370f2
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93101084"
---
# <a name="developing-with-azure-cosmos-db-table-api-and-azure-table-storage"></a>Développement avec l’API Table Azure Cosmos DB et Stockage Table Azure
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

L’API Table Azure Cosmos DB et Stockage Table Azure partagent le même modèle de données de table et exposent les mêmes opérations de création, suppression, mise à jour et interrogation par le biais de leurs SDK.

[!INCLUDE [storage-table-cosmos-comparison](../../includes/storage-table-cosmos-comparison.md)]

## <a name="developing-with-the-azure-cosmos-db-table-api"></a>Développement avec l’API Table Azure Cosmos DB

À ce stade, [l’API Table de Azure Cosmos DB](table-introduction.md) a quatre kits disponibles pour le développement : 

* [Microsoft.Azure.Cosmos.Table](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table) : SDK .NET. Cette bibliothèque cible .NET Standard et présente les mêmes classes et signatures de méthode que le [SDK Stockage Microsoft Azure](https://www.nuget.org/packages/WindowsAzure.Storage) public, mais elle peut également se connecter à des comptes Azure Cosmos DB à l’aide de l’API Table. Il est recommandé aux utilisateurs de la bibliothèque .NET Framework [Microsoft.Azure.CosmosDB.Table](https://www.nuget.org/packages/Microsoft.Azure.CosmosDB.Table/) de mettre à niveau vers [Microsoft.Azure.Cosmos.Table](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table), car elle est en mode maintenance et sera bientôt dépréciée.

* [SDK Python](table-sdk-python.md) : Le nouveau SDK Azure Cosmos DB Python est le seul SDK qui prend en charge le stockage Table Azure dans Python. Ce kit de développement logiciel se connecte à la fois avec l’API Table Azure Cosmos DB et le Stockage Table Azure.

* [SDK Java](table-sdk-java.md) : Ce kit de développement pour le stockage Azure a la possibilité de se connecter à des comptes Azure Cosmos DB à l’aide de l’API Table.

* [SDK Node.js](table-sdk-nodejs.md) : Ce kit de développement pour le stockage Azure a la possibilité de se connecter à des comptes Azure Cosmos DB à l’aide de l’API Table.


Des informations supplémentaires sur l’utilisation de l’API Table sont disponibles dans l’article [FAQ : Développer avec l’API Table](table-api-faq.md).

## <a name="developing-with-azure-table-storage"></a>Développement avec Stockage de tables Azure

Ces Kit de développement logiciel (SDK) sont disponibles pour le développement avec le stockage de tables Azure :

- Les bibliothèques [Microsoft.Azure.Storage.Blob](https://www.nuget.org/packages/Microsoft.Azure.Storage.Blob/), [Microsoft.Azure.Storage.File](https://www.nuget.org/packages/Microsoft.Azure.Storage.File/), [Microsoft.Azure.Storage.Queue](https://www.nuget.org/packages/Microsoft.Azure.Storage.Queue/) et [Microsoft.Azure.Storage.Common](https://www.nuget.org/packages/Microsoft.Azure.Storage.Common/) permettent d’utiliser le service Stockage Table Azure. Si vous utilisez l’API Table dans Azure Cosmos DB, vous pouvez utiliser plutôt la bibliothèque [Microsoft.Azure.CosmosDB.Table](https://www.nuget.org/packages/Microsoft.Azure.CosmosDB.Table/).
- [Kit de développement logiciel (SDK) Python](https://github.com/Azure/azure-cosmos-table-python). Le SDK Table Azure Cosmos DB pour Python prend en charge le service Stockage Table (parce que le service Stockage Table Azure et l’API Table de Cosmos DB partagent les mêmes caractéristiques et fonctionnalités, et dans le but de factoriser nos efforts de développement de SDK, nous vous recommandons d’utiliser ce SDK).
- [Kit de développement logiciel (SDK) Stockage Azure pour Java](https://github.com/azure/azure-storage-java). Ce Kit de développement logiciel (SDK) du Stockage Azure fournit une bibliothèque cliente en Java pour consommer le stockage de tables Azure.
- [Kit de développement logiciel (SDK) Node.js](https://github.com/Azure/azure-storage-node). Ce Kit de développement logiciel (SDK) fournit un package Node.js et une bibliothèque de client compatible avec le navigateur JavaScript afin de consommer le service de Table de stockage.
- [Module PowerShell AzureRmStorageTable](https://www.powershellgallery.com/packages/AzureRmStorageTable). Ce module PowerShell comporte des cmdlet permettant de travailler avec des Tables de stockage.
- [Bibliothèque de client de stockage Azure pour C++](https://github.com/Azure/azure-storage-cpp/). Cette bibliothèque vous permet de créer des applications sur Stockage Azure.
- [Bibliothèque de client du service de Table de Stockage Azure pour Ruby](https://github.com/azure/azure-storage-ruby/tree/master/table). Ce projet fournit un package Ruby qui facilite l’accès aux services de Table de stockage Azure.
- [Bibliothèque de client PHP du service de Table de Stockage Azure pour Ruby](https://github.com/Azure/azure-storage-php/tree/master/azure-storage-table). Ce projet fournit une bibliothèque de client PHP qui facilite l’accès aux services de Table de stockage Azure.


   





