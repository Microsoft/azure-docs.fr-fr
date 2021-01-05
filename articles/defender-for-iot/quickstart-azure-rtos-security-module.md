---
title: 'Démarrage rapide : Configurer et activer le module de sécurité pour Azure RTOS'
description: Découvrez comment intégrer et activer le module de sécurité pour le service Azure RTOS dans votre hub Azure IoT.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: rkarlin
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/24/2020
ms.author: rkarlin
ms.openlocfilehash: 321c8d2b9e58aba943c5bf19adf54d6359c5be96
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96351774"
---
# <a name="quickstart-security-module-for-azure-rtos-preview"></a>Démarrage rapide : Module de sécurité pour Azure RTOS (préversion)

Cet article décrit les prérequis avant de commencer et explique comment activer le module de sécurité pour le service Azure RTOS sur un hub IoT. Si vous n’avez pas de hub IoT, pour commencer, voir [Créer un hub IoT à l’aide du portail Azure](../iot-hub/iot-hub-create-through-portal.md).

> [!NOTE]
> Le module de sécurité pour Azure RTOS est uniquement pris en charge dans les hubs IoT de niveau standard.

## <a name="prerequisites"></a>Prérequis 

### <a name="supported-devices"></a>Appareils pris en charge

- Kit de découverte ST STM32F746G
- NXP i.MX RT1060 EVK
- Puce SAM E54 Xplained Pro EVK

Téléchargez, compilez et exécutez l’un des fichiers .zip pour le tableau et l’outil spécifiques (IAR, IDE semi ou PC) de votre choix à partir du [module de sécurité pour la ressource GitHub Azure RTOS](https://github.com/azure-rtos/azure-iot-preview/releases).

### <a name="azure-resources"></a>Ressources Azure

La phase suivante de la mise en route consiste à préparer vos ressources Azure. Vous avez besoin d’un hub IoT et nous suggérons un espace de travail Log Analytics. Pour IoT Hub, vous aurez besoin d’une chaîne de connexion IoT Hub pour vous connecter à votre appareil. 
  
### <a name="iot-hub-connection"></a>Connexion au hub IoT

Une connexion au hub IoT est nécessaire pour commencer. 

1. Ouvrez votre **hub IoT** dans le portail Azure.
1. Copiez la chaîne de connexion IoT dans le [fichier de configuration](how-to-azure-rtos-security-module.md).


Les informations d’identification de connexion sont extraites de la configuration de l’application utilisateur **HOST_NAME**, **DEVICE_ID** et **DEVICE_SYMMETRIC_KEY**.

Le module de sécurité pour Azure RTOS utilise des connexions middleware Azure IoT basées sur le protocole **MQTT**.


### <a name="log-analytics-workspace"></a>Espace de travail Log Analytics

L’ingestion Log Analytics dans IoT Hub est désactivée par défaut dans la solution Defender pour IoT. Pour l’activer afin de l’utiliser avec le module de sécurité pour Azure RTOS, procédez comme suit : 
1. Sur le portail Azure, accédez à votre hub IoT.
1. Sélectionnez **Paramètres** sous le menu **Sécurité**.
   :::image type="content" source="media/quickstart/azure-rtos-hub-settings.png" alt-text="Accéder à l’option Collecte de données pour Azure RTOS"::: 
1. Sélectionnez **Collection de données**. 
1. À partir de l’option **Configuration de l’espace de travail**, passez le bouton bascule sur **Activé**. 
1. Créez un espace de travail Log Analytics où attachez-en un déjà existant. Assurez-vous que l’option **Accéder aux données de sécurité brutes** est sélectionnée. 
 :::image type="content" source="media/quickstart/azure-rtos-data-collection-on.png" alt-text="Configuration Azure RTOS avec options de collecte de données et d’accès aux données de sécurité brutes toutes deux sélectionnées":::
1. Sélectionnez **Enregistrer**.
1. Revenez à votre liste de ressources Azure et vérifiez que l’espace de travail Log Analytics que vous avez créé ou attaché est activé pour le hub IoT.
    :::image type="content" source="media/quickstart/verify-azure-resource-list.png" alt-text="Vérifier votre liste de ressources Azure pour confirmer l’ajout de l’espace de travail correct Log Analytics pour un hub IoT"::: 

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant pour terminer la configuration et la personnalisation de votre solution.

> [!div class="nextstepaction"]
> [Configurer le module de sécurité pour Azure RTOS](how-to-azure-rtos-security-module.md)