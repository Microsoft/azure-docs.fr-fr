---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 09/20/2020
ms.author: glenga
ms.openlocfilehash: 7d1bf8dd2d1c8feab8b051a8edad7d5e570ee11b
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96025611"
---
Le fuseau horaire par défaut utilisé avec les expressions CRON est le Temps universel coordonné (UTC). Pour baser votre expression CRON sur un autre fuseau horaire, créez un paramètre d’application nommé `WEBSITE_TIME_ZONE` pour votre application de fonction. 

La valeur de ce paramètre dépend du système d’exploitation et du plan sur lequel l’application de fonction s’exécute.

|Système d’exploitation |Plan |Valeur |
|-|-|-|
| **Windows** |Tous | Définissez la valeur sur le nom du fuseau horaire souhaité comme indiqué par la deuxième ligne de chaque paire donnée par la commande Windows `tzutil.exe /L` |
| **Linux** |Premium<br/>Dédié |Définissez la valeur sur le nom du fuseau horaire souhaité comme indiqué dans la [base de données tz](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones). |

> [!NOTE]
> `WEBSITE_TIME_ZONE` n’est pas pris en charge sur le plan consommation Linux.

Par exemple, l’heure de la côte est des États-Unis (représentée par `Eastern Standard Time` (Windows) ou `America/New_York` (Linux)) utilise actuellement l’heure UTC (temps universel coordonné) 05:00 pendant l’heure standard et l’heure UTC 04:00 pendant l’heure d’été. Pour qu’un déclencheur de minuteur s’active à 10:00 (heure de la côte est) chaque jour, créez un paramètre d’application nommé `WEBSITE_TIME_ZONE` pour votre application de fonction, définissez la valeur sur `Eastern Standard Time` (Windows) ou sur `America/New_York` (Linux), puis utilisez l’expression NCRONTAB suivante : 

```
"0 0 10 * * *"
``` 

Quand vous utilisez `WEBSITE_TIME_ZONE`, l’heure est ajustée en fonction des changements d’heure du fuseau horaire spécifique, notamment pour tenir compte de l’heure d’été et des changements de l’heure standard.
