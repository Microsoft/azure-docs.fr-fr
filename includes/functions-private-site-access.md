---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 05/06/2020
ms.author: glenga
ms.openlocfilehash: 2d66e7f497f85141de172c59b67676e1bb93955e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96020984"
---
Le [point de terminaison privé Azure](../articles/private-link/private-endpoint-overview.md) est une interface réseau qui vous connecte de manière privée et sécurisée à un service s’appuyant sur Azure Private Link.  Le point de terminaison privé utilise une adresse IP privée de votre réseau virtuel pour apporter le service à votre réseau virtuel de façon effective.

Vous pouvez utiliser un point de terminaison privé pour vos fonctions hébergées dans les plans [Premium](../articles/azure-functions/functions-premium-plan.md) et [App Service](../articles/azure-functions/functions-scale.md#app-service-plan).

Lors de la création d’une connexion de point de terminaison privé entrant pour les fonctions, vous aurez également besoin d’un enregistrement DNS pour résoudre l’adresse privée.  Par défaut, un enregistrement DNS privé est créé pour vous lors de la création d’un point de terminaison privé à l’aide du portail Azure.

Pour plus d’informations, consultez [Utilisation de points de terminaison privés pour Web Apps](../articles/app-service/networking/private-endpoint.md).