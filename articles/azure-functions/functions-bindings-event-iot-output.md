---
title: Liaison de sortie Azure IoT Hub pour Azure Functions
description: Découvrez comment écrire des messages dans des flux Azure IoT Hubs avec Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.date: 02/21/2020
ms.author: cshoe
ms.openlocfilehash: d4dbf43fb5684d829e581be29832e94ad46b2936
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91871777"
---
# <a name="azure-iot-hub-output-binding-for-azure-functions"></a>Liaison de sortie Azure IoT Hub pour Azure Functions

Cet article explique comment utiliser des liaisons de sortie Azure Functions pour IoT Hub. La prise en charge d'IoT Hub repose sur la [liaison Azure Event Hubs](functions-bindings-event-hubs.md).

Pour plus d’informations sur les détails d’installation et de configuration, consultez la [vue d’ensemble](functions-bindings-event-iot.md).

> [!IMPORTANT]
> Si les exemples de code suivants utilisent l'API Event Hub, la syntaxe donnée s'applique aux fonctions IoT Hub.

[!INCLUDE [functions-bindings-event-hubs](../../includes/functions-bindings-event-hubs-output.md)]

## <a name="next-steps"></a>Étapes suivantes

- [Répondre aux événements envoyés à un flux d’événements Event Hub (déclencheur)](./functions-bindings-event-iot-trigger.md)
