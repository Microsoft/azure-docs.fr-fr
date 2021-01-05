---
title: Fichier include
description: Fichier include
services: iot-hub
ms.service: iot-hub
author: robinsh
ms.topic: include
ms.date: 02/17/2019
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: 7f1f7d6f9ab6036fbcfcd1d19e175302bbd1a2a8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87298879"
---
## <a name="customize-and-extend-the-device-management-actions"></a>Personnaliser et étendre les actions de gestion d’appareils

Vos solutions IoT peuvent étendre l’ensemble défini de modèles de gestion d’appareils ou activer des modèles personnalisés en utilisant les primitives de la méthode cloud-à-appareil et du jumeau d’appareil. La réinitialisation des paramètres d’usine, la mise à jour du microprogramme, la mise à jour logicielle, la gestion de l’alimentation, la gestion du réseau et de la connectivité, et le chiffrement des données sont d’autres exemples d’actions de gestion des appareils.

## <a name="device-maintenance-windows"></a>Fenêtres de maintenance d’appareil

En règle générale, vous configurez des appareils pour effectuer des actions à un moment qui minimise les interruptions et les temps d’arrêt. Les fenêtres de maintenance d’appareil constituent un modèle couramment utilisé pour définir l’heure à laquelle un appareil doit mettre à jour sa configuration. Vos solutions principales peuvent utiliser les propriétés souhaitées de la représentation d’appareil pour définir et activer une stratégie sur votre appareil qui permet d’obtenir une fenêtre de maintenance. Lorsqu’un appareil reçoit la stratégie de fenêtre de maintenance, il peut utiliser la propriété signalée de la représentation d’appareil pour indiquer l’état de la stratégie. L’application principale peut ensuite utiliser des requêtes de représentation d’appareil pour certifier la conformité des appareils et de chaque stratégie.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez utilisé une méthode directe pour déclencher un redémarrage à distance sur un appareil. Vous avez utilisé les propriétés signalées pour indiquer le moment du dernier redémarrage de l’appareil et vous avez interrogé le jumeau d’appareil pour découvrir l’heure du dernier redémarrage de l’appareil à partir du cloud.

Pour poursuivre votre découverte d’IoT Hub et des modèles de gestion d’appareils, comme la mise à jour du microprogramme à distance, consultez [Mettre à jour un microprogramme](../articles/iot-hub/tutorial-firmware-update.md).

Pour savoir comment étendre votre solution IoT et planifier des appels de méthode sur plusieurs appareils, consultez [Planifier et diffuser des travaux](../articles/iot-hub/iot-hub-node-node-schedule-jobs.md).