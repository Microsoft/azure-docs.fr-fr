---
author: baanders
description: Fichier include pour Azure Digital Twins - Modes de gestion de l’instance
ms.service: digital-twins
ms.topic: include
ms.date: 11/11/2020
ms.author: baanders
ms.openlocfilehash: 887d185249f96b5d3be4aab6a96aa3c6c4a85690
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96231410"
---
Cet article montre comment effectuer différentes opérations de gestion à l’aide du [**kit SDK** .NET (C#) Azure Digital Twins](/dotnet/api/overview/azure/digitaltwins/management?view=azure-dotnet&preserve-view=true). Vous pouvez également créer les mêmes appels de gestion à l’aide des autres kits SDK de langage décrits dans [*Guide pratique : Utiliser les kits SDK et les API Azure Digital Twins*](../articles/digital-twins/how-to-use-apis-sdks.md).

> [!TIP] 
> N’oubliez pas que toutes les méthodes du SDK sont disponibles dans les versions synchrones et asynchrones. Pour les appels de pagination, les méthodes asynchrones retournent `AsyncPageable<T>`, tandis que les versions synchrones retournent `Pageable<T>`.

Une autre option de gestion consiste à appeler les [**API REST**](/rest/api/azure-digitaltwins/) Azure Digital Twins directement pour ce sujet via un client REST tel que Postman. Pour obtenir des instructions sur la façon de procéder, consultez [*Procédure : Faire des demandes avec Postman*](../articles/digital-twins/how-to-use-postman.md).

Enfin, vous pouvez effectuer les mêmes opérations de gestion à l’aide de l’**interface CLI** Azure Digital Twins. Pour en savoir plus sur l’utilisation de l’interface CLI, consultez [*Guide pratique : utiliser l’interface CLI Azure Digital Twins*](../articles/digital-twins/how-to-use-cli.md).