---
title: Tutoriel – Connecter une application cliente générique à Azure IoT Central | Microsoft Docs
description: Ce tutoriel vous explique comment, en tant que développeur d’appareils, connecter un appareil exécutant une application cliente C, C#, Java, JavaScript ou Python à votre application Azure IoT Central. Vous modifiez le modèle d’appareil généré automatiquement en ajoutant des vues qui permettent à un opérateur d’interagir avec un appareil connecté.
author: dominicbetts
ms.author: dobett
ms.date: 11/24/2020
ms.topic: tutorial
ms.service: iot-central
services: iot-central
ms.custom:
- mqtt
- device-developer
zone_pivot_groups: programming-languages-set-twenty-six
ms.openlocfilehash: 2757d696f5922263abf87399d6491e46b5e5513c
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033866"
---
# <a name="tutorial-create-and-connect-a-client-application-to-your-azure-iot-central-application"></a>Tutoriel : Créer et connecter une application cliente à votre application Azure IoT Central

*Cet article s’applique aux créateurs de solutions et aux développeurs d’appareils.*

Ce tutoriel vous explique comment, en tant que développeur d’appareils, connecter une application cliente à votre application Azure IoT Central. L’application simule le comportement d’un appareil à thermostat. Quand l’application se connecte à IoT Central, elle envoie l’ID du modèle de l’appareil à thermostat. IoT Central utilise cet ID du modèle pour récupérer le modèle d’appareil et créer un modèle pour votre appareil. Vous ajoutez des personnalisations et des vues au modèle d’appareil pour permettre à un opérateur d’interagir avec un appareil.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer et exécuter le code d’appareil et veiller à ce qu’il se connecte à votre application IoT Central.
> * Afficher la télémétrie simulée envoyée à partir de l’appareil.
> * Ajouter des vues personnalisées à un modèle d’appareil.
> * Publier le modèle d’appareil.
> * Utiliser une vue pour gérer les propriétés de l’appareil.
> * Appeler une commande pour contrôler l’appareil.

:::zone pivot="programming-language-ansi-c"

[!INCLUDE [iot-central-connect-device-c](../../../includes/iot-central-connect-device-c.md)]

:::zone-end

:::zone pivot="programming-language-csharp"

[!INCLUDE [iot-central-connect-device-csharp](../../../includes/iot-central-connect-device-csharp.md)]

:::zone-end

:::zone pivot="programming-language-java"

[!INCLUDE [iot-central-connect-device-java](../../../includes/iot-central-connect-device-java.md)]

:::zone-end

:::zone pivot="programming-language-javascript"

[!INCLUDE [iot-central-connect-device-nodejs](../../../includes/iot-central-connect-device-nodejs.md)]

:::zone-end

:::zone pivot="programming-language-python"

[!INCLUDE [iot-central-connect-device-python](../../../includes/iot-central-connect-device-python.md)]

:::zone-end

## <a name="view-raw-data"></a>Voir les données brutes

En tant que développeur d’appareils, vous pouvez utiliser la vue **Données brutes** pour examiner les données brutes que votre appareil envoie à IoT Central :

:::image type="content" source="media/tutorial-connect-device/raw-data.png" alt-text="Vue Données brutes":::

Dans cette vue, vous pouvez sélectionner les colonnes à afficher et définir une plage de temps à afficher. La colonne **Données démodélisées** affiche les données de l’appareil qui ne correspondent à aucune définition de propriété ou de télémétrie dans le modèle d’appareil.

## <a name="next-steps"></a>Étapes suivantes

Si vous préférez suivre l’ensemble des tutoriels IoT Central et en savoir plus sur la création d’une solution IoT Central, consultez :

> [!div class="nextstepaction"]
> [Créer un modèle d’appareil de passerelle](./tutorial-define-gateway-device-type.md)

En tant que développeur d’appareils, maintenant que vous avez appris les bases de la création d’un appareil en utilisant Java, les prochaines étapes suggérées sont les suivantes :

* Pour en savoir plus sur le rôle des modèles d’appareils quand vous implémentez votre code d’appareil, consultez [Présentation des modèles d’appareils](./concepts-device-templates.md).
* Lisez [Se connecter à Azure IoT Central](./concepts-get-connected.md) pour en savoir plus sur la façon d’inscrire des appareils auprès d’IoT Central et sur la manière dont IoT Central sécurise les connexions des appareils.
* Lisez [Charges utiles de télémétrie, de propriétés et de commandes](concepts-telemetry-properties-commands.md) pour en savoir plus sur les données échangées par l’appareil avec IoT Central.
* Lisez le [guide du développeur d’appareils IoT Plug-and-Play](../../iot-pnp/concepts-developer-guide-device.md).
