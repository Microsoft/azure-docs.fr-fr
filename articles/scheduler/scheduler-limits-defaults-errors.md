---
title: Limites, quotas et seuils dans Azure Scheduler
description: Découvrez les limites, quotas, valeurs par défaut et seuils de limitation applicables dans Azure Scheduler
services: scheduler
ms.service: scheduler
author: derek1ee
ms.author: deli
ms.reviewer: klam, estfan
ms.topic: article
ms.date: 08/18/2016
ms.openlocfilehash: 610232dab776648bb3dcc7c301ec292e9acad9fc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "78898524"
---
# <a name="limits-quotas-and-throttle-thresholds-in-azure-scheduler"></a>Limites, quotas et seuils de limitation dans Azure Scheduler

> [!IMPORTANT]
> [Azure Logic Apps](../logic-apps/logic-apps-overview.md) remplace Azure Scheduler, qui est en phase de [mise hors service](../scheduler/migrate-from-scheduler-to-logic-apps.md#retire-date). Pour poursuivre les travaux que vous avez configurés dans Scheduler, veuillez [migrer vers Azure Logic Apps](../scheduler/migrate-from-scheduler-to-logic-apps.md) dès que possible. 
>
> Scheduler n’est plus disponible dans le portail Azure, mais l’[API REST](/rest/api/scheduler) et les [applets de commande Azure Scheduler PowerShell](scheduler-powershell-reference.md) restent disponibles pour vous permettre de gérer vos travaux et collections de travaux.

## <a name="limits-quotas-and-thresholds"></a>Limites, quotas et seuils

[!INCLUDE [scheduler-limits-table](../../includes/scheduler-limits-table.md)]

## <a name="x-ms-request-id-header"></a>L’en-tête x-ms-request-id

Chaque requête adressée au service Scheduler retourne un en-tête de réponse nommé **x-ms-request-id**. Cet en-tête contient une valeur opaque qui identifie de façon unique la requête. Par conséquent, si une requête échoue constamment et que vous avez vérifié qu’elle est formulée correctement, vous pouvez signaler l’erreur à Microsoft en indiquant la valeur de l’en-tête de réponse **x-ms-request-id** ainsi que les informations suivantes : 

* La valeur de l’en-tête **x-ms-request-id**
* L’heure approximative de la requête effectuée 
* Les identificateurs de l’abonnement Azure, de la collection de travaux et du travail 
* Le type d’opération tentée par la requête

## <a name="next-steps"></a>Étapes suivantes

* [Concepts, terminologie et hiérarchie d’entités d’Azure Scheduler](scheduler-concepts-terms.md)
* [Plans et facturation pour Azure Scheduler](scheduler-plans-billing.md)
* [Informations de référence sur l’API REST d’Azure Scheluler](/rest/api/scheduler)
* [Informations de référence sur les applets de commande PowerShell d’Azure Scheluler](scheduler-powershell-reference.md)