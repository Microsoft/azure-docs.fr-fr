---
author: wesmc7777
ms.author: wesmc
ms.service: iot-hub
ms.topic: include
ms.date: 08/20/2019
ms.openlocfilehash: c412f7f3e20e4d04083e457bfb245b850b65e126
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "70050419"
---
> [!div class="op_single_selector"]
> * [Node.JS](../articles/iot-hub/iot-hub-node-node-twin-getstarted.md)
> * [.NET](../articles/iot-hub/iot-hub-csharp-csharp-twin-getstarted.md)
> * [Java](../articles/iot-hub/iot-hub-java-java-twin-getstarted.md)
> * [Python](../articles/iot-hub/iot-hub-python-twin-getstarted.md)

Les jumeaux d’appareil sont des documents JSON qui stockent des informations sur l’état des appareils (métadonnées, configurations et conditions). IoT Hub conserve une représentation d’appareil pour chaque appareil que vous y connectez.

[!INCLUDE [iot-hub-basic](iot-hub-basic-whole.md)]

Vous pouvez utiliser des jumeaux d’appareil pour répondre aux besoins suivants :

* Stockez les métadonnées d’appareil à partir de votre serveur principal de solution.

* Signaler les informations d’état actuel, telles que les capacités disponibles et les conditions, par exemple la méthode de connectivité utilisée, à partir de votre application d’appareil

* Synchronisez l’état des workflows de longue durée, par exemple les mises à jour de microprogramme et de configuration, entre une application d’appareil et une application back-end.

* Interroger les métadonnées, la configuration ou l’état de vos appareils

Les représentations d’appareil sont conçues pour les synchronisations et pour l’interrogation des configurations et des conditions d’appareil. Pour plus d’informations sur l’utilisation des jumeaux d’appareil, consultez [Comprendre les jumeaux d’appareil](../articles/iot-hub/iot-hub-devguide-device-twins.md).

Les jumeaux d’appareil sont stockés dans un hub IoT et contiennent les éléments suivants :

* **Tags** (balises). Métadonnées d’appareil uniquement accessibles par le back-end de solution.

* **Propriétés souhaitées (Desired)** . Objets JSON modifiables par le back-end de solution et observables par l’application d’appareil.

* **Propriétés signalées (Reported)** . Objets JSON modifiables par l’application d’appareil et consultables par le back-end de solution.

Les étiquettes et les propriétés ne peuvent pas contenir de tableaux, mais les objets peuvent être imbriqués.

L’illustration suivante montre l’organisation des jumeaux d’appareil :

![image de jumeau d’appareil affichant la fonctionnalité](./media/iot-hub-selector-twin-get-started/twin.png)

En outre, le serveur principal de solution peut interroger les représentations d’appareil concernant toutes les données ci-dessus.
Pour plus d’informations sur les jumeaux d’appareil, consultez [Présentation des jumeaux d’appareil](../articles/iot-hub/iot-hub-devguide-device-twins.md). Pour plus d’informations sur l’interrogation, consultez [Langage de requête IoT Hub](../articles/iot-hub/iot-hub-devguide-query-language.md).


Ce didacticiel vous explique les procédures suivantes :

* Création d’une application back-end ajoutant des étiquettes à un jumeau d’appareil, et création d’une application d’appareil simulé signalant son canal de connectivité sous la forme d’une propriété signalée sur le jumeau d’appareil.

* Interrogez les appareils à partir de votre application principale à l’aide de filtres sur les étiquettes et les propriétés précédemment créées.
