---
title: Conseils sur la gestion des erreurs et les nouvelles tentatives dans Azure Functions
description: Apprenez à gérer les erreurs et les événements de nouvelle tentative dans Azure Functions avec des liens vers des erreurs de liaison spécifiques.
author: craigshoemaker
ms.topic: conceptual
ms.date: 10/01/2020
ms.author: cshoe
ms.openlocfilehash: e52babd5b0612b4590f1422a941b25ba30c3f169
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93284448"
---
# <a name="azure-functions-error-handling-and-retries"></a>Gestion des erreurs et nouvelles tentatives dans Azure Functions

La gestion des erreurs dans Azure Functions est importante pour éviter des pertes de données, des événements manqués et pour superviser l’intégrité de votre application.

Cet article décrit les stratégies générales de gestion des erreurs, ainsi que des liens vers des erreurs propres à la liaison.

## <a name="handling-errors"></a>Gestion des erreurs

[!INCLUDE [bindings errors intro](../../includes/functions-bindings-errors-retries.md)]

## <a name="binding-error-codes"></a>Codes d’erreur de liaison

Lors de l’intégration aux services Azure, vous pouvez rencontrer des erreurs provenant des API des services sous-jacents. Les informations relatives aux erreurs propres à la liaison sont disponibles dans la section **Exceptions et codes de retour** des articles suivants :

+ [Azure Cosmos DB](functions-bindings-cosmosdb.md#exceptions-and-return-codes)

+ [Stockage Blob](functions-bindings-storage-blob-output.md#exceptions-and-return-codes)

+ [Hubs d'événements](functions-bindings-event-hubs-output.md#exceptions-and-return-codes)

+ [IoT Hubs](functions-bindings-event-iot-output.md#exceptions-and-return-codes)

+ [Notification Hubs](functions-bindings-notification-hubs.md#exceptions-and-return-codes)

+ [Stockage de files d’attente](functions-bindings-storage-queue-output.md#exceptions-and-return-codes)

+ [Service Bus](functions-bindings-service-bus-output.md#exceptions-and-return-codes)

+ [Stockage Table](functions-bindings-storage-table-output.md#exceptions-and-return-codes)
