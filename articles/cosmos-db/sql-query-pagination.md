---
title: Pagination dans Azure Cosmos DB
description: Découvrez les concepts de pagination et les jetons de continuation
author: timsander1
ms.author: tisande
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 07/29/2020
ms.openlocfilehash: c49fee169b7bd01ee7cf8a6d539c2125cf6568b3
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96545313"
---
# <a name="pagination-in-azure-cosmos-db"></a>Pagination dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Dans Azure Cosmos DB, les requêtes peuvent comporter plusieurs pages de résultats. Ce document explique les critères que le moteur de requête d’Azure Cosmos DB utilise pour décider s’il faut répartir les résultats de la requête sur plusieurs pages. Vous pouvez éventuellement utiliser des jetons de continuation pour gérer les résultats de requête qui s’étendent sur plusieurs pages.

## <a name="understanding-query-executions"></a>Déroulement des exécutions d’une requête

Les résultats d’une requête sont parfois répartis sur plusieurs pages. Les résultats de chaque page sont générés par une exécution de requête distincte. Lorsque les résultats de la requête ne peuvent pas être retournés en une seule exécution, Azure Cosmos DB répartit automatiquement les résultats sur plusieurs pages.

Vous pouvez spécifier le nombre maximal d’éléments retournés par une requête en définissant `MaxItemCount`. `MaxItemCount` est spécifié par demande et indique que le moteur de requête renverra ce nombre d’éléments au maximum. Vous pouvez définir `MaxItemCount` sur `-1` si vous ne voulez pas limiter le nombre de résultats par exécution de requête.

De plus, il existe d’autres raisons pour lesquelles le moteur de requête peut être amené à répartir les résultats d’une requête sur plusieurs pages. notamment :

- Le conteneur a fait l’objet d’une limitation et il n’existait pas d’unités de requête disponibles pour retourner davantage de résultats de requête
- La réponse d’exécution de requête était trop volumineuse
- La durée d’exécution de requête était trop longue
- Il était plus efficace pour le moteur de requête de retourner les résultats dans des exécutions supplémentaires

Le nombre d’éléments retournés par exécution de requête est toujours inférieur ou égal à `MaxItemCount`. Toutefois, il est possible que d’autres critères aient limité le nombre de résultats que la requête a pu retourner. Si vous exécutez la même requête plusieurs fois, le nombre de pages risque de ne pas être constant. Par exemple, si une requête fait l’objet d’une limitation, il peut y avoir moins de résultats disponibles par page, ce qui signifie que la requête aura des pages supplémentaires. Dans certains cas, il est également possible que votre requête retourne une page de résultats vide.

## <a name="handling-multiple-pages-of-results"></a>Gestion de plusieurs pages de résultats

Pour garantir des résultats de requête précis, vous devez progresser tout au long des pages. Vous devez continuer à exécuter les requêtes jusqu’à ce qu’il n’y ait plus de pages supplémentaires.

Voici quelques exemples de traitement des résultats à partir de requêtes avec plusieurs pages :

- [Kit de développement logiciel (SDK) .NET](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/Queries/Program.cs#L280)
- [Kit SDK Java](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/documentcrud/sync/DocumentCRUDQuickstart.java#L162-L176)
- [Kit de développement logiciel (SDK) Node.js](https://github.com/Azure/azure-sdk-for-js/blob/83fcc44a23ad771128d6e0f49043656b3d1df990/sdk/cosmosdb/cosmos/samples/IndexManagement.ts#L128-L140)
- [Kit de développement logiciel (SDK) Python](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/examples.py#L89)

## <a name="continuation-tokens"></a>Jetons de continuation

Dans les kits SDK .NET et Java, vous pouvez éventuellement utiliser des jetons de continuation comme signet pour la progression de votre requête. Les exécutions de requête Azure Cosmos DB sont sans état côté serveur et peuvent reprendre à tout moment à l’aide du jeton de continuation. Les jetons de continuation ne sont pas pris en charge dans le kit SDK Node.js. Pour le kit de développement logiciel (SDK) Python, il est pris en charge pour les requêtes à partition unique, et la clé primaire doit être spécifiée dans l’objet d’options, car il n’est pas suffisant dans la requête elle-même.

Voici quelques exemples d’utilisation de jetons de continuation :

- [Kit de développement logiciel (SDK) .NET](https://github.com/Azure/azure-cosmos-dotnet-v2/blob/master/samples/code-samples/Queries/Program.cs#L699-L734)
- [Kit SDK Java](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L216)
- [Kit de développement logiciel (SDK) Python](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/test/test_query.py#L533)

Si la requête retourne un jeton de continuation, il existe des résultats de requête supplémentaires.

Dans l’API REST d’Azure Cosmos DB, vous pouvez gérer les jetons de continuation avec l’en-tête `x-ms-continuation`. Comme pour l’interrogation avec le kit SDK .NET ou Java, si l’en-tête de réponse `x-ms-continuation` n’est pas vide, cela signifie que la requête a des résultats supplémentaires.

Tant que vous utilisez la même version du kit SDK, les jetons de continuation n’expirent pas. Vous pouvez éventuellement [restreindre la taille d’un jeton de continuation](/dotnet/api/microsoft.azure.documents.client.feedoptions.responsecontinuationtokenlimitinkb?preserve-view=true&view=azure-dotnet#Microsoft_Azure_Documents_Client_FeedOptions_ResponseContinuationTokenLimitInKb). Quels que soient la quantité de données et le nombre de partitions physiques dans votre conteneur, les requêtes renvoient un jeton de continuation unique.

Vous ne pouvez pas utiliser de jetons de continuation pour les requêtes avec [GROUP BY](sql-query-group-by.md) ou [DISTINCT](sql-query-keywords.md#distinct), car ces requêtes nécessitent le stockage d’une grande quantité d’informations d’état. Pour les requêtes avec `DISTINCT`, vous pouvez utiliser des jetons de continuation si vous ajoutez `ORDER BY` à la requête.

Voici un exemple de requête avec `DISTINCT` qui pourrait utiliser un jeton de continuation :

```sql
SELECT DISTINCT VALUE c.name
FROM c
ORDER BY c.name
```

## <a name="next-steps"></a>Étapes suivantes

- [Présentation d’Azure Cosmos DB](introduction.md)
- [Exemples .NET Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v3)
- [Clause ORDER BY](sql-query-order-by.md)
