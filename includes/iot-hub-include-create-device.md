---
title: Fichier include
description: Fichier include
services: iot-hub
author: robinsh
ms.service: iot-hub
ms.topic: include
ms.date: 11/06/2018
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: 7b022f71e197c5695876f2049ee376c3616afc6d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "70049013"
---
<!-- put the ## header in the file that includes this file -->

Dans cette section, vous allez créer une identité d’appareil dans le registre des identités de votre hub IoT. Un appareil ne peut pas se connecter à un hub, à moins d’avoir une entrée dans le registre des identités. Pour plus d’informations, consultez le [Guide du développeur IoT Hub](../articles/iot-hub/iot-hub-devguide-identity-registry.md#identity-registry-operations).

1. Dans le menu de navigation de votre hub IoT, ouvrez **Appareils IoT**, puis sélectionnez **Nouveau** pour ajouter un appareil à votre hub IoT.

    ![Créer une identité d’appareil sur le portail](./media/iot-hub-include-create-device/create-identity-portal-vs2019.png)

1. Dans **Créer un appareil**, indiquez le nom de votre nouvel appareil, par exemple **myDeviceId**, puis sélectionnez **Enregistrer**. Cette action permet de créer une identité d’appareil pour votre hub IoT.

   ![Ajouter un nouvel appareil](./media/iot-hub-include-create-device/create-a-device-vs2019.png)

   [!INCLUDE [iot-hub-pii-note-naming-device](iot-hub-pii-note-naming-device.md)]

1. Une fois l’appareil créé, ouvrez ce dernier à partir de la liste figurant dans le volet **Appareils IoT**. Copiez la **chaîne de connexion principale** à utiliser plus tard.

    ![Chaîne de connexion de l’appareil](./media/iot-hub-include-create-device/device-details-vs2019.png)

> [!NOTE]
> Le registre des identités IoT Hub stocke uniquement les identités des appareils pour permettre un accès sécurisé à IoT Hub. Il stocke les ID des appareils et les clés à utiliser comme informations d’identification de sécurité et un indicateur activé/désactivé que vous pouvez utiliser pour désactiver l’accès pour un appareil individuel. Si votre application a besoin de stocker d’autres métadonnées spécifiques aux appareils, elle doit utiliser un magasin spécifique aux applications. Pour plus d’informations, reportez-vous au [Guide du développeur IoT Hub](../articles/iot-hub/iot-hub-devguide-identity-registry.md).
