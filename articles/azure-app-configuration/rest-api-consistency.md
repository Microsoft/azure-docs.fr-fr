---
title: API REST Azure App Configuration – Cohérence
description: Pages de référence pour garantir la cohérence en temps réel à l’aide de l’API REST Azure App Configuration
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: reference
ms.date: 08/17/2020
ms.openlocfilehash: b324d23ce9abc1eb3893f316365aff828de2063d
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96932623"
---
# <a name="real-time-consistency"></a>Cohérence en temps réel

En raison de la nature de certains systèmes distribués, il est difficile d’appliquer la cohérence en temps réel entre les demandes de manière implicite. Une solution consiste à autoriser la prise en charge du protocole sous la forme de plusieurs jetons de synchronisation. Les jetons de synchronisation sont facultatifs.

## <a name="initial-request"></a>Demande initiale

Pour garantir la cohérence en temps réel entre les différentes instances et demandes de client, utilisez des en-têtes de demande et réponse `Sync-Token` facultatifs.

Syntaxe :

```http
Sync-Token: <id>=<value>;sn=<sn>
```

|Paramètre|Description|
|--|--|
| `<id>` | ID de jeton (opaque) |
| `<value>` | Valeur de jeton (opaque). Autorise une chaîne encodée en base64. |
| `<sn>` | Numéro de séquence du jeton (version). Une valeur plus élevée correspond à une version plus récente du même jeton. Permet une meilleure concurrence et une mise en cache client. Le client peut choisir d’utiliser uniquement la dernière version du jeton, car les versions de jeton sont inclusives. Ce paramètre n’est pas requis pour les demandes. |

## <a name="response"></a>response

Le service fournit un en-tête `Sync-Token` avec chaque réponse.

```http
Sync-Token: jtqGc1I4=MDoyOA==;sn=28
```

## <a name="subsequent-requests"></a>Demandes suivantes

Il est garanti que toute demande ultérieure recevra une réponse cohérente en temps réel par rapport au `Sync-Token` fourni.

```http
Sync-Token: <id>=<value>
```

Si vous omettez l’en-tête `Sync-Token` de la demande, il est possible que le service réponde avec des données mises en cache pendant une brève période de temps (jusqu’à quelques secondes) avant d’être réglé en interne. Ce comportement peut provoquer des lectures incohérentes si des modifications ont eu lieu immédiatement avant la lecture.

## <a name="multiple-sync-tokens"></a>Jetons de synchronisation multiples

Le serveur peut répondre avec plusieurs jetons de synchronisation à une demande individuelle. Pour conserver la cohérence en temps réel pour la demande suivante, le client doit répondre avec tous les jetons de synchronisation reçus. Plusieurs valeurs d’en-tête doivent être séparées par des virgules.

```http
Sync-Token: <token1-id>=<value>,<token2-id>=<value>
```
