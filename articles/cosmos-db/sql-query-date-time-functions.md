---
title: 'Langage de requête Azure Cosmos DB : fonctions de date et heure'
description: Découvrez les fonctions système SQL de date et d’heure dans Azure Cosmos DB pour effectuer des opérations Date/Heure et timestamp.
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 08/18/2020
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: cfa81b6ec5f10218a70de6b9b55e502d87898194
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96549172"
---
# <a name="date-and-time-functions-azure-cosmos-db"></a>Fonctions de date et heure (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Les fonctions de date et heure vous permettent d’effectuer des opérations d’horodatage et DateHeure dans Azure Cosmos DB.

## <a name="functions-to-obtain-the-date-and-time"></a>Fonctions permettant d'obtenir la date et l'heure

Les fonctions scalaires suivantes vous permettent d’obtenir la date et l’heure UTC actuelles sous trois formes : une chaîne conforme au format ISO 8601, un horodatage numérique dont la valeur est le nombre de millisecondes qui se sont écoulées depuis l’époque UNIX ou des cycles numériques dont la valeur est le nombre de cycles de 100 nanosecondes qui se sont écoulés depuis l’époque UNIX :

* [GetCurrentDateTime](sql-query-getcurrentdatetime.md)
* [GetCurrentTimestamp](sql-query-getcurrenttimestamp.md)
* [GetCurrentTicks](sql-query-getcurrentticks.md)

## <a name="functions-to-work-with-datetime-values"></a>Fonctions à utiliser avec les valeurs DateTime

Les fonctions suivantes vous permettent de manipuler facilement les valeurs de DateHeure, d’horodatage et de cycle :

* [DateTimeAdd](sql-query-datetimeadd.md)
* [DateTimeDiff](sql-query-datetimediff.md)
* [DateTimeFromParts](sql-query-datetimefromparts.md)
* [DateTimePart](sql-query-datetimepart.md)
* [DateTimeToTicks](sql-query-datetimetoticks.md)
* [DateTimeToTimestamp](sql-query-datetimetotimestamp.md)
* [TicksToDateTime](sql-query-tickstodatetime.md)
* [TimestampToDateTime](sql-query-timestamptodatetime.md)

## <a name="next-steps"></a>Étapes suivantes

- [Fonctions système Azure Cosmos DB](sql-query-system-functions.md)
- [Présentation d’Azure Cosmos DB](introduction.md)
- [Fonctions définies par l’utilisateur](sql-query-udfs.md)
- [Agrégats](sql-query-aggregate-functions.md)
