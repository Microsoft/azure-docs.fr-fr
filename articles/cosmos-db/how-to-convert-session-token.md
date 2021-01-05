---
title: Comment convertir des formats de jeton de session dans le Kit de développement logiciel (SDK) .NET – Azure Cosmos DB
description: Découvrez comment convertir les formats de jeton de session pour garantir la compatibilité entre les différentes versions du Kit de développement logiciel (SDK) .NET
author: vinhms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 04/30/2020
ms.author: vitrinh
ms.custom: devx-track-csharp
ms.openlocfilehash: 7ed06eeb7330989b4a251dc77e516eb8ac578bff
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93342043"
---
# <a name="convert-session-token-formats-in-net-sdk"></a>Convertir les format de jeton de session dans le Kit de développement logiciel (SDK) .NET
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Cet article explique comment effectuer une conversion entre différents formats de jeton de session pour garantir la compatibilité entre les différentes versions du Kit de développement logiciel (SDK).

> [!NOTE]
> Par défaut, le Kit de développement logiciel (SDK) effectue automatiquement le suivi du jeton de session et utilise le jeton de session le plus récent.  Pour plus d’informations, consultez [Utiliser des jetons de session](how-to-manage-consistency.md#utilize-session-tokens). Les instructions de cet article s’appliquent uniquement aux conditions suivantes :
> * Votre compte Azure Cosmos DB utilise la Cohérence de session.
> * Vous gérez les jetons de session manuellement.
> * Vous utilisez plusieurs versions du Kit de développement logiciel (SDK) en même temps.

## <a name="session-token-formats"></a>Formats de jeton de session

Il existe deux formats de jeton de session : **simple** et **vector**.  Ces deux formats n’étant pas interchangeables, ils doivent être convertis lors de la transmission à l’application cliente avec des versions différentes.
- Le format de jeton de session **simple** est utilisé par le Kit de développement logiciel (SDK) .NET v1 (Microsoft.Azure.DocumentDB -version 1.x).
- Le format de jeton de session **vector** est utilisé par le Kit de développement logiciel (SDK) .NET v2 (Microsoft.Azure.DocumentDB -version 2.x).

### <a name="simple-session-token"></a>Jeton de session simple

Un jeton de session simple a le format suivant : `{pkrangeid}:{globalLSN}`

### <a name="vector-session-token"></a>Jeton de session vector

Un jeton de session vector présente le format suivant : `{pkrangeid}:{Version}#{GlobalLSN}#{RegionId1}={LocalLsn1}#{RegionId2}={LocalLsn2}....#{RegionIdN}={LocalLsnN}`

## <a name="convert-to-simple-session-token"></a>Convertir en jeton de session simple

Pour transmettre un jeton de session à un client à l’aide du Kit de développement logiciel (SDK) .NET v1, utilisez un format de jeton de session **simple**.  Par exemple, utilisez l’exemple de code suivant pour le convertir.

```csharp
private static readonly char[] SegmentSeparator = (new[] { '#' });
private static readonly char[] PkRangeSeparator = (new[] { ':' });

// sessionTokenToConvert = session token from previous response
string[] items = sessionTokenToConvert.Split(PkRangeSeparator, StringSplitOptions.RemoveEmptyEntries);
string[] sessionTokenSegments = items[1].Split(SessionTokenHelpers.SegmentSeparator, StringSplitOptions.RemoveEmptyEntries);

string sessionTokenInSimpleFormat;

if (sessionTokenSegments.Length == 1)
{
    // returning the same token since it already has the correct format
    sessionTokenInSimpleFormat = sessionTokenToConvert;
}
else
{
    long version = 0;
    long globalLSN = 0;

    if (!long.TryParse(sessionTokenSegments[0], out version)
        || !long.TryParse(sessionTokenSegments[1], out globalLSN))
    {
        throw new ArgumentException("Invalid session token format", sessionTokenToConvert);
    }

    sessionTokenInSimpleFormat = string.Format("{0}:{1}", items[0], globalLSN);
}
```

## <a name="convert-to-vector-session-token"></a>Convertir en jeton de session vector

Pour transmettre un jeton de session à un client à l’aide du Kit de développement logiciel (SDK) .NET v2, utilisez un format de jeton de session **vector**.  Par exemple, utilisez l’exemple de code suivant pour le convertir.

```csharp

private static readonly char[] SegmentSeparator = (new[] { '#' });
private static readonly char[] PkRangeSeparator = (new[] { ':' });

// sessionTokenToConvert = session token from previous response
string[] items = sessionTokenToConvert.Split(PkRangeSeparator, StringSplitOptions.RemoveEmptyEntries);
string[] sessionTokenSegments = items[1].Split(SegmentSeparator, StringSplitOptions.RemoveEmptyEntries);

string sessionTokenInVectorFormat;

if (sessionTokenSegments.Length == 1)
{
    long globalLSN = 0;
    if (long.TryParse(sessionTokenSegments[0], out globalLSN))
    {
        sessionTokenInVectorFormat = string.Format("{0}:-2#{1}", items[0], globalLSN);
    }
    else
    {
        throw new ArgumentException("Invalid session token format", sessionTokenToConvert);
    }
}
else
{
    // returning the same token since it already has the correct format
    sessionTokenInVectorFormat = sessionTokenToConvert;
}
```

## <a name="next-steps"></a>Étapes suivantes

Consultez les articles suivants :

* [Utiliser des jetons de sessions pour gérer la cohérence dans Azure Cosmos DB](how-to-manage-consistency.md#utilize-session-tokens)
* [Choisir le niveau de cohérence approprié dans Azure Cosmos DB](./consistency-levels.md)
* [Compromis entre cohérence, disponibilité et niveau de performance dans Azure Cosmos DB](./consistency-levels.md)
* [Compromis entre disponibilité et performance pour différents niveaux de cohérence](./consistency-levels.md)