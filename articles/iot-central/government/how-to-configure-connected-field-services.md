---
title: Connecter votre application Azure IoT Central avec Dynamics 365 for Field Service | Microsoft Docs
description: Découvrez comment créer une solution de bout en bout avec Azure IoT Central et Dynamics 365 for Field Service
author: miriambrus
ms.author: miriamb
ms.date: 12/11/2020
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.openlocfilehash: 1d098f56bbadfe115620580c8d93fb6dd021550d
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97586672"
---
# <a name="build-end-to-end-solution-with-azure-iot-central-and-dynamics-365-field-service"></a>Créer une solution de bout en bout avec Azure IoT Central et Dynamics 365 for Field Service 
En tant que générateur, vous pouvez activer l’intégration de votre application Azure IoT Central à d’autres systèmes métier. 

Par exemple, dans une solution de gestion des déchets connectée, vous pouvez optimiser l’envoi de camions de ramassage d’ordures. L’optimisation peut être réalisée sur la base de données de capteurs IoT de conteneurs à déchets connectés. Dans votre [application de gestion des déchets connectée à IoT Central](./tutorial-connected-waste-management.md), vous pouvez configurer des règles et actions qui déclenchent des alertes dans Dynamics 365 for Field Service. Ce scénario est implémenté à l’aide de Power Automate, que vous configurez directement dans IoT Central pour automatiser les workflows entre les applications et les services. Par ailleurs, selon les activités de service dans Field Service, des informations peuvent être renvoyées à Azure IoT Central. 

## <a name="how-to-connect-your-azure-iot-central-application-with-dynamics-365-field-services"></a>Comment connecter votre application Azure IoT Central avec Dynamics 365 for Field Service 

Vous pouvez facilement implémenter les processus d’intégration ci-dessous sur la base d’une expérience de configuration pure :
* Azure IoT Central peut envoyer des informations relatives à des anomalies de périphériques (sous forme d’alertes IoT) à un service de terrain connecté à des fins de diagnostic.
* Ce service de terrain connecté peut créer des cas de support ou des ordres de travail déclenchés à partir d’anomalies de périphériques.
* Il peut également planifier des techniciens pour l’inspection afin d’éviter les temps d’arrêt.
* Le tableau de bord d’appareil Azure IoT Central peut être mis à jour avec les informations de planification et de service appropriées.


## <a name="next-steps"></a>Étapes suivantes
* Apprenez-en davantage sur les [modèles IoT Central Government](./overview-iot-central-government.md)
* Apprenez-en davantage sur [IoT Hub](../core/overview-iot-central.md)
* Apprenez-en davantage sur [Dynamics 365 for Field Service](/dynamics365/field-service/cfs-iot-overview)
