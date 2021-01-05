---
title: Déclencheur Azure IoT Hub pour Azure Functions
description: Découvrez comment répondre aux événements envoyés à un flux d’événements IoT Hub dans Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.date: 02/21/2020
ms.author: cshoe
ms.openlocfilehash: 25396da3fb9a4293633308bf2e9d3c6b3d07265d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87041645"
---
# <a name="azure-iot-hub-trigger-for-azure-functions"></a>Déclencheur Azure IoT Hub pour Azure Functions

Cet article explique comment utiliser des liaisons Azure Functions pour IoT Hub. La prise en charge d'IoT Hub repose sur la [liaison Azure Event Hubs](functions-bindings-event-hubs.md).

Pour plus d’informations sur les détails d’installation et de configuration, consultez la [vue d’ensemble](functions-bindings-event-iot.md).

> [!IMPORTANT]
> Si les exemples de code suivants utilisent l'API Event Hub, la syntaxe donnée s'applique aux fonctions IoT Hub.

[!INCLUDE [functions-bindings-event-hubs](../../includes/functions-bindings-event-hubs-trigger.md)]

## <a name="next-steps"></a>Étapes suivantes

- [Écrire des événements dans un flux d’événements (liaison de sortie)](./functions-bindings-event-iot-output.md)
