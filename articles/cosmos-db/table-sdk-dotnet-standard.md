---
title: Ressources et kit SDK .NET Standard de l’API Table Azure Cosmos DB
description: Découvrez plus d’informations sur l’API Table d’Azure Cosmos DB et le SDK .NET Standard , notamment les dates de publication, les dates de mise hors service et les modifications effectuées entre chaque version.
author: sakash279
ms.author: akshanka
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.devlang: dotnet
ms.topic: reference
ms.date: 03/18/2019
ms.custom: devx-track-dotnet
ms.openlocfilehash: eee59d906da46e315ede52796bd89c7827833ede
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93095728"
---
# <a name="azure-cosmos-db-table-net-standard-api-download-and-release-notes"></a>API .NET Standard Table Azure Cosmos DB : Téléchargement et notes de publication
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]
> [!div class="op_single_selector"]
> 
> * [.NET](table-sdk-dotnet.md)
> * [.NET Standard](table-sdk-dotnet-standard.md)
> * [Java](table-sdk-java.md)
> * [Node.JS](table-sdk-nodejs.md)
> * [Python](table-sdk-python.md)

|   |   |
|---|---|
|**Téléchargement du Kit de développement logiciel (SDK)**|[NuGet](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table)|
|**Exemple**|[Exemple .NET d'API Table Cosmos DB](https://github.com/Azure-Samples/azure-cosmos-table-dotnet-core-getting-started)|
|**Démarrage rapide**|[Démarrage rapide](create-table-dotnet.md)|
|**Didacticiel**|[Didacticiel](tutorial-develop-table-dotnet.md)|
|**Infrastructure actuellement prise en charge**|[Microsoft .NET Standard 2.0](https://www.nuget.org/packages/NETStandard.Library)|
|**Signaler un problème**|[Signaler un problème](https://github.com/Azure/azure-cosmos-table-dotnet/issues)|

## <a name="release-notes-for-200-series"></a>Notes de publication pour la série 2.0.0
La série 2.0.0 est dépendante de [Microsoft.Azure.Cosmos](https://www.nuget.org/packages/Microsoft.Azure.Cosmos/). Elle apporte des améliorations de performances et assure le regroupement des espaces de noms sur le point de terminaison Cosmos DB.

### <a name="200-preview"></a><a name="2.0.0-preview"></a>2.0.0-Preview
* Il s’agit de la préversion initiale du SDK Table série 2.0.0, dépendante de [Microsoft.Azure.Cosmos](https://www.nuget.org/packages/Microsoft.Azure.Cosmos/). Elle apporte des améliorations de performances et assure le regroupement des espaces de noms sur le point de terminaison Cosmos DB. L’API publique reste la même.

## <a name="release-notes-for-100-series"></a>Notes de publication pour la série 1.0.0
La série 1.0.0 est dépendante de [Microsoft.Azure.DocumentDB.Core](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB.Core/).

### <a name="108"></a><a name="1.0.8"></a>1.0.8
* Ajouter la prise en charge pour définir correctement la propriété TTL s’il s’agit du point de terminaison cosmosdb 
* Respecter la stratégie de nouvelle tentative après le délai d’attente et l’exception de tâche annulée
* Corriger l’exception intermittente de tâche annulée visible dans les applications ASP .net
* Corriger la récupération du stockage de la table Azure à partir du point de terminaison secondaire uniquement en mode d’emplacement
* Mettre à jour `Microsoft.Azure.DocumentDB.Core` la version de dépendance vers 2.11.2 qui corrige l’exception de référence nulle intermittente
* Mettre à jour `Odata.Core` la version de dépendance vers 7.6.4, qui résout les conflits de compatibilité avec Azure Shell

### <a name="107"></a><a name="1.0.7"></a>1.0.7
* Amélioration des performances grâce à la définition du niveau de trace par défaut du SDK Table sur SourceLevels.Off, qui peut être activé via app.config

### <a name="105"></a><a name="1.0.5"></a>1.0.5
* Introduction d’une nouvelle configuration sous TableClientConfiguration afin d’utiliser Rest Executor pour communiquer avec l’API Table de Cosmos DB

### <a name="105-preview"></a><a name="1.0.5-preview"></a>1.0.5-preview
* Résolution des bogues

### <a name="104"></a><a name="1.0.4"></a>1.0.4
* Résolution des bogues
* Indiquez l’option HttpClientTimeout pour RestExecutorConfiguration.

### <a name="104-preview"></a><a name="1.0.4-preview"></a>1.0.4-preview
* Résolution des bogues
* Indiquez l’option HttpClientTimeout pour RestExecutorConfiguration.

### <a name="101"></a><a name="1.0.1"></a>1.0.1
* Résolution des bogues

### <a name="100"></a><a name="1.0.0"></a>1.0.0
* Version en disponibilité générale

### <a name="0110-preview"></a><a name="0.11.0-preview"></a>0.11.0-preview
* Des modifications ont été apportées à la configuration de CloudTableClient. Elle accepte désormais un objet TableClientConfiguration pendant la construction. TableClientConfiguration propose différentes propriétés pour configurer le comportement du client selon que le point de terminaison cible est API Table Cosmos DB ou API Table Stockage Azure.
* Ajout de la prise en charge de TableQuery pour renvoyer des résultats triés dans une colonne personnalisée. Cette fonctionnalité est uniquement prise en charge sur les points de terminaison Table Cosmos DB.
* Ajout de la prise en charge d'exposition de RequestCharges sur différents types de résultats. Cette fonctionnalité est uniquement prise en charge sur les points de terminaison Table Cosmos DB.

### <a name="0101-preview"></a><a name="0.10.1-preview"></a>0.10.1-preview
* Ajout de la prise en charge du jeton SAS et des opérations de TablePermissions, ServiceProperties et ServiceStats par rapport à des points de terminaison Table de stockage Azure. 
   > [!NOTE]
   > Certaines fonctionnalités des précédents kits SDK Table du stockage Azure ne sont pas encore prises en charge, comme le chiffrement côté client.

### <a name="0100-preview"></a><a name="0.10.0-preview"></a>0.10.0-preview
* Ajout de la prise en charge des principales opérations CRUD, de traitement par lots et d'interrogation par rapport à des points de terminaison Table de stockage Azure. 
   > [!NOTE]
   > Certaines fonctionnalités des précédents kits SDK Table du stockage Azure ne sont pas encore prises en charge, comme le chiffrement côté client.

### <a name="091-preview"></a><a name="0.9.1-preview"></a>0.9.1-preview
* Le SDK .NET Standard Table d’Azure Cosmos DB est une bibliothèque .NET multiplateforme qui fournit un accès efficace au modèle de données Table sur Cosmos DB. Cette version initiale prend en charge l’ensemble complet des fonctionnalités CRUD + Requête sur les tables et les entités avec des API similaires à celles du [SDK Table de Cosmos DB pour .NET Framework](table-sdk-dotnet.md). 
   > [!NOTE]
   >  Les points de terminaison Table de Stockage Azure ne sont pas encore pris en charge dans la version 0.9.1-preview.

## <a name="release-and-retirement-dates"></a>Dates de publication et de retrait
Microsoft envoie une notification au moins **12 mois** avant le retrait d’un Kit de développement logiciel (SDK) pour faciliter la transition vers une version plus récente/prise en charge.

Cette bibliothèque .NET Standard multiplateforme [Microsoft.Azure.Cosmos.Table](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table) remplacera la bibliothèque .NET Framework [Microsoft.Azure.CosmosDB.Table](https://www.nuget.org/packages/Microsoft.Azure.CosmosDB.Table).

### <a name="200-series"></a>Série 2.0.0
| Version | Date de sortie | Date de suppression |
| --- | --- | --- |
| [2.0.0-preview](#2.0.0-preview) |22 août 2019 |--- |

### <a name="100-series"></a>Série 1.0.0
| Version | Date de sortie | Date de suppression |
| --- | --- | --- |
| [1.0.5](#1.0.5) |13 septembre 2019 |--- |
| [1.0.5-preview](#1.0.5-preview) |20 août 2019 |--- |
| [1.0.4](#1.0.4) |12 août 2019 |--- |
| [1.0.4-preview](#1.0.4-preview) |26 juillet 2019 |--- |
| 1.0.2-preview |2 mai 2019 |--- |
| [1.0.1](#1.0.1) |19 avril 2019 |--- |
| [1.0.0](#1.0.0) |13 mars 2019 |--- |
| [0.11.0-preview](#0.11.0-preview) |5 mars 2019 |--- |
| [0.10.1-preview](#0.10.1-preview) |22 janvier 2019 |--- |
| [0.10.0-preview](#0.10.0-preview) |18 décembre 2018 |--- |
| [0.9.1-preview](#0.9.1-preview) |18 octobre 2018 |--- |


## <a name="faq"></a>Questions fréquentes (FAQ)

[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="see-also"></a>Voir aussi
Pour plus d’informations sur l’API Table Azure Cosmos DB, consultez [Présentation d’Azure Cosmos DB : API Table](table-introduction.md).
