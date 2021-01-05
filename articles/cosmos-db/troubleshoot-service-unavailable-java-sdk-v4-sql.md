---
title: Résoudre des problèmes liés aux exceptions de service non disponible Azure Cosmos DB avec le SDK Java v4
description: Découvrez comment diagnostiquer et corriger des exceptions de service non disponible Azure Cosmos DB avec le SDK Java v4.
author: kushagrathapar
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.date: 10/28/2020
ms.author: kuthapar
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: 03c899307b00010ca87b279ed720b92946d3673c
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93340062"
---
# <a name="diagnose-and-troubleshoot-azure-cosmos-db-java-v4-sdk-service-unavailable-exceptions"></a>Diagnostiquer et résoudre des problèmes liés aux exceptions de service non disponible Azure Cosmos DB avec le SDK Java v4
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]
Le SDK Java v4 n’a pas pu se connecter à Azure Cosmos DB.

## <a name="troubleshooting-steps"></a>Étapes de dépannage
La liste suivante répertorie les causes connues et les solutions pour les exceptions de service non disponible.

### <a name="the-required-ports-are-being-blocked"></a>Les ports requis sont bloqués
Vérifiez que tous les [ports requis](sql-sdk-connection-modes.md#service-port-ranges) sont activés.

### <a name="client-side-transient-connectivity-issues"></a>Problèmes de connectivité temporaires côté client
Des exceptions non disponibles du service peuvent apparaître en cas de problèmes de connectivité temporaires qui provoquent des expirations de délai. En règle générale, l’arborescence des appels de procédure associée à ce scénario contient une erreur `ServiceUnavailableException` avec des détails du diagnostic. Par exemple :

```java
Exception in thread "main" ServiceUnavailableException{userAgent=azsdk-java-cosmos/4.6.0 Linux/4.15.0-1096-azure JRE/11.0.8, error=null, resourceAddress='null', requestUri='null', statusCode=503, message=Service is currently unavailable, please retry after a while. If this problem persists please contact support.: Message: "" {"diagnostics"}
```

Suivez les [étapes de résolution des problèmes liés à l’expiration des délais de requête](troubleshoot-request-timeout-java-sdk-v4-sql.md#troubleshooting-steps) pour la résoudre.

### <a name="service-outage"></a>Interruption de service
Vérifiez [l’état d’Azure](https://status.azure.com/status) pour voir s’il existe un problème en cours.


## <a name="next-steps"></a>Étapes suivantes
* [Diagnostiquer et résoudre](troubleshoot-java-sdk-v4-sql.md) des problèmes lors de l'utilisation du SDK Java v4 Azure Cosmos DB.
* Découvrez les recommandations relatives aux performances pour le [SDK Java v4](performance-tips-java-sdk-v4-sql.md).