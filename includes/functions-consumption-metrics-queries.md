---
ms.openlocfilehash: 0bae8946b7d81fbf45698cefe7c8b00bbc94d00c
ms.sourcegitcommit: 419c8c8061c0ff6dc12c66ad6eda1b266d2f40bd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/18/2020
ms.locfileid: "92168106"
---
#### <a name="determine-memory-usage"></a>Déterminer l’utilisation de la mémoire 

Sous **Supervision**, sélectionnez **Logs (Analytics)**, puis copiez la requête de télémétrie suivante et collez-la dans la fenêtre de requête, puis sélectionnez **Exécuter**. Cette requête retourne l’utilisation totale de la mémoire pour chaque durée échantillonnée.

```
performanceCounters
| where name == "Private Bytes"
| project timestamp, name, value
```

Les résultats sont semblables à l’exemple qui suit :

| Horodatage \[UTC\]          | name          | value       |
|----------------------------|---------------|-------------|
| 9/12/2019, 1:05:14\.947 AM | Octets privés | 209 932 288 |
| 9/12/2019, 1:06:14\.994 AM | Octets privés | 212 189 184 |
| 9/12/2019, 1:06:30\.010 AM | Octets privés | 231 714 816 |
| 9/12/2019, 1:07:15\.040 AM | Octets privés | 210 591 744 |
| 9/12/2019, 1:12:16\.285 AM | Octets privés | 216 285 184 |
| 9/12/2019, 1:12:31\.376 AM | Octets privés | 235 806 720 |

#### <a name="determine-duration"></a>Déterminer la durée 

Azure Monitor effectue le suivi des métriques au niveau de la ressource, qui, pour Functions, correspond à l’application de fonction. L’intégration à Application Insights émet des métriques par fonction. Voici un exemple de requête analytique pour obtenir la durée moyenne d’une fonction :

```
customMetrics
| where name contains "Duration"
| extend averageDuration = valueSum / valueCount
| summarize averageDurationMilliseconds=avg(averageDuration) by name
```

| name                       | averageDurationMilliseconds |
|----------------------------|-----------------------------|
| QueueTrigger AvgDurationMs | 16\.087                     |
| QueueTrigger MaxDurationMs | 90\.249                     |
| QueueTrigger MinDurationMs | 8\.522                      |
