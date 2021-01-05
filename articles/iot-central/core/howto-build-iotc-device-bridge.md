---
title: Générer le pont d’appareil Azure IoT Central | Microsoft Docs
description: Générez le pont d’appareil IoT Central pour connecter d’autres clouds IoT (Sigfox, Particle, The Things Network etc.) à votre application IoT Central.
services: iot-central
ms.service: iot-central
author: viv-liu
ms.author: viviali
ms.date: 07/09/2019
ms.topic: how-to
manager: peterpr
ms.custom: device-developer
ms.openlocfilehash: fc8ea41e804344735cfa2400d5d763622d8811c8
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95026248"
---
# <a name="build-the-iot-central-device-bridge-to-connect-other-iot-clouds-to-iot-central"></a>Générer le pont d’appareil IoT Central pour connecter d’autres clouds IoT à IoT Central

*Cette rubrique s’applique aux administrateurs.*

Le pont d’appareil IoT Central est une solution open source qui connecte votre cloud Sigfox, Particle, The Things Network, ou autre, à votre application IoT Central. Que vous utilisiez des appareils de suivi des ressources connectés au réseau LPWAN de Sigfox, ou que vous utilisiez des appareils de surveillance de la qualité de l’air sur Particle Device Cloud ou des appareils de surveillance de l’humidité du sol sur TTN, vous pouvez directement tirer parti de la puissance de l’IoT Central à l’aide du pont d’appareil IoT Central. Le pont d’appareil connecte d’autres clouds IoT à IoT Central en transférant les données envoyées par vos appareils à d’autres clouds via votre application IoT Central. Dans votre application IoT Central, vous pouvez créer des règles et exécuter l’analytique de ces données, créer des flux de travail dans Power Automate et Azure Logic Apps, exporter les données et bien plus encore. Obtenir le [pont d’appareil IoT Central](https://aka.ms/iotcentralgithubdevicebridge) sur GitHub

## <a name="what-is-it-and-how-does-it-work"></a>Description et fonctionnement
Le pont d’appareil IoT Central est une solution open source dans GitHub. Elle est prête à l’emploi grâce à un bouton « Déployer sur Azure » qui déploie un modèle Azure Resource Manager personnalisé avec plusieurs ressources Azure dans votre abonnement Azure. Les ressources incluent :
-    Application Azure Function
-    Compte Stockage Azure
-    Plan de consommation
-    Azure Key Vault

L’application de fonction est l’élément critique du pont d’appareils. Elle reçoit les requêtes HTTP POST à partir d’autres plateformes IoT ou plateformes personnalisées via une intégration webhook simple. Nous vous proposons des exemples qui montrent comment vous connecter aux clouds Sigfox, Particle et TTN. Vous pouvez facilement étendre cette solution pour vous connecter à votre cloud IoT personnalisé si votre plateforme peut envoyer des requêtes HTTP POST à votre application de fonction.
L’application de fonction convertit les données dans un format accepté par IoT Central et les transfère via des API DPS.

![Capture d’écran d’Azure Functions](media/howto-build-iotc-device-bridge/azfunctions.png)

Si votre application IoT Central reconnaît l’appareil par ID d’appareil dans le message transféré, une nouvelle mesure s’affiche pour cet appareil. Si l’ID d’appareil n’a jamais été vu par votre application IoT Central, votre application de fonction tente d’inscrire un nouvel appareil avec cet ID d’appareil, et celui-ci s’affiche comme un « appareil non associé » dans votre application IoT Central. 

## <a name="how-do-i-set-it-up"></a>Configuration
Les instructions sont répertoriées en détail dans le fichier Lisez-moi dans le référentiel GitHub. 

## <a name="pricing"></a>Tarifs
Les ressources Azure seront hébergées dans votre abonnement Azure. Vous trouverez plus d’informations sur la tarification dans le [fichier Lisez-moi](https://aka.ms/iotcentralgithubdevicebridge).

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous savez comment générer le pont d’appareil IoT Central, voici l’étape suivante suggérée :

> [!div class="nextstepaction"]
> [Gestion de vos appareils](howto-manage-devices.md)
