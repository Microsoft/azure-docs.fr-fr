---
title: Utiliser un hub IoT existant avec la solution de simulation d’appareil – Azure | Microsoft Docs
description: Cet article décrit la configuration de l’accélérateur de solution de simulation d’appareil pour utiliser un IoT Hub existant.
author: dominicbetts
manager: timlt
ms.author: dobett
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 10/25/2018
ms.topic: conceptual
ms.openlocfilehash: 1f89e23d7bb279e7cce5c104060cc7898517f8b7
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96009362"
---
# <a name="use-an-existing-iot-hub-with-the-device-simulation-solution-accelerator"></a>Utiliser un IoT Hub existant avec l’accélérateur de solution de simulation d’appareil

Lorsque vous déployez la simulation d’appareil, vous pouvez également choisir de déployer un hub IoT à utiliser dans votre simulation. Cette option déploie un hub [S2 de niveau IoT avec une unité à échelle unique](../iot-hub/iot-hub-scaling.md). Si vous déployez ce hub IoT facultatif, vous pouvez toujours choisir de cibler un autre hub IoT pour une exécution de simulation.

Si vous choisissez de ne pas de déployer le IoT Hub facultatif, vous devez utiliser votre propre hub pour toutes les simulations exécutées.

Si vous n’avez pas d’IoT Hub, vous pouvez toujours en créer un à partir du [portail Azure](https://portal.azure.com).

Pour utiliser un IoT Hub déjà existant, vous avez besoin de la chaîne de connexion pour la stratégie d'accès partagé **iothubowner**. Vous pouvez obtenir cette chaîne de connexion auprès du [portail Azure](https://portal.azure.com) :

1. Dans la page de configuration du hub du portail, cliquez sur **Stratégies d’accès partagé**.

1. Cliquez sur **iothubowner**.

1. Copiez la chaîne de connexion primaire ou secondaire.

[![Obtenir la chaîne de connexion](./media/iot-accelerators-device-simulation-choose-hub/connectionstring-inline.png)](./media/iot-accelerators-device-simulation-choose-hub/connectionstring-expanded.png#lightbox)

Utilisez la chaîne de connexion que vous avez copiée lors de la configuration de la simulation :

![Configurer une simulation](./media/iot-accelerators-device-simulation-choose-hub/configuresimulation.png)

### <a name="next-steps"></a>Étapes suivantes

Grâce à ce guide de procédure, vous avez appris à utiliser un IoT Hub existant dans une simulation. Vous souhaiterez ensuite peut-être en savoir comment [Créer un modèle d’appareil avancé](iot-accelerators-device-simulation-advanced-device.md) pour une simulation.
