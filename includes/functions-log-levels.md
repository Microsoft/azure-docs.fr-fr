---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 05/04/2020
ms.author: glenga
ms.openlocfilehash: 72588ccfd9ad00305cbdeaf8132999731d86b9b1
ms.sourcegitcommit: 419c8c8061c0ff6dc12c66ad6eda1b266d2f40bd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/18/2020
ms.locfileid: "92164906"
---
Un *niveau de journal* est affecté à chaque journal. La valeur est un entier qui indique l’importance relative :

|LogLevel    |Code| Description |
|------------|---|--------------|
|Trace       | 0 |Journaux contenant les messages les plus détaillés. Ces messages peuvent contenir des données d’application sensibles. Ils sont désactivés par défaut et ne doivent jamais être activés dans un environnement de production.|
|Débogage       | 1 | Journaux utilisés pour l’investigation interactive durant le développement. Ils doivent principalement contenir des informations utiles pour le débogage et n’ont pas d’intérêt à long terme. |
|Information | 2 | Journaux utilisés pour suivre le flux général de l’application. Ils ont généralement une utilité à long terme. |
|Avertissement     | 3 | Les journaux qui mettent en évidence un événement anormal ou inattendu dans le workflow de l’application, mais ne provoquent pas l’arrêt de l’exécution de l’application. |
|Error       | 4 | Les journaux qui surveillent le moment où le flux actuel de l’exécution est arrêté en raison d’un échec. Ces erreurs doivent indiquer un échec dans l’activité en cours, et non un échec sur l’ensemble de l’application. |
|Critique    | 5 | Journaux qui décrivent un plantage irrécupérable de l’application ou du système, ou une défaillance catastrophique nécessitant une attention immédiate. |
|None        | 6 | Désactive la journalisation pour la catégorie spécifiée. |

La configuration du [fichier *host.json*](../articles/azure-functions/functions-host-json.md) détermine la quantité de journalisation qu’une application de fonction envoie à Application Insights.  
