---
title: Résoudre des problèmes liés aux exceptions non disponible du service Azure Cosmos DB
description: Découvrez comment diagnostiquer et corriger des exceptions non disponibles du service Azure Cosmos DB.
author: j82w
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.date: 08/06/2020
ms.author: jawilley
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: d19d30c03412ba7212211b30646acb50c3f55ece
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93340020"
---
# <a name="diagnose-and-troubleshoot-azure-cosmos-db-service-unavailable-exceptions"></a>Diagnostiquer et résoudre des problèmes liés aux exceptions non disponibles du service Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Le kit de développement logiciel (SDK) n’a pas pu se connecter à Azure Cosmos DB.

## <a name="troubleshooting-steps"></a>Étapes de dépannage
La liste suivante répertorie les causes connues et les solutions pour les exceptions de service non disponible.

### <a name="the-required-ports-are-being-blocked"></a>Les ports requis sont bloqués
Vérifiez que tous les [ports requis](sql-sdk-connection-modes.md#service-port-ranges) sont activés.

### <a name="client-side-transient-connectivity-issues"></a>Problèmes de connectivité temporaires côté client
Des exceptions non disponibles du service peuvent apparaître en cas de problèmes de connectivité temporaires qui provoquent des expirations de délai. En règle générale, l’arborescence des appels de procédure associée à ce scénario contient une erreur `TransportException`. Par exemple :

```C#
TransportException: A client transport error occurred: The request timed out while waiting for a server response. 
(Time: xxx, activity ID: xxx, error code: ReceiveTimeout [0x0010], base error: HRESULT 0x80131500
```

Suivez les [étapes de résolution des problèmes liés à l’expiration des délais de requête](troubleshoot-dot-net-sdk-request-timeout.md#troubleshooting-steps) pour la résoudre.

### <a name="service-outage"></a>Interruption de service
Vérifiez [l’état d’Azure](https://status.azure.com/status) pour voir s’il existe un problème en cours.


## <a name="next-steps"></a>Étapes suivantes
* [Diagnostiquer et résoudre](troubleshoot-dot-net-sdk.md) des problèmes lors de l’utilisation du kit de développement logiciel (SDK) .NET Azure Cosmos DB.
* Découvrez les recommandations relatives aux performances pour [.NET V3](performance-tips-dotnet-sdk-v3-sql.md) et [.NET V2](performance-tips.md).