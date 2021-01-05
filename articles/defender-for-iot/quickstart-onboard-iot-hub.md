---
title: 'Démarrage rapide : Activer le service'
description: Découvrez comment intégrer et activer le service de sécurité Defender pour IoT dans votre hub Azure IoT.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/06/2020
ms.author: mlottner
ms.openlocfilehash: 786fcd1a0c6d7df2c38a086a830a63f7179d7d40
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96352505"
---
# <a name="quickstart-onboard-azure-defender-for-iot-service-in-iot-hub"></a>Démarrage rapide : Intégrer le service Azure Defender pour IoT à IoT Hub

Cet article explique comment activer le service Defender pour IoT sur votre hub IoT. Si vous n’avez pas de hub IoT, pour commencer, voir [Créer un hub IoT à l’aide du portail Azure](../iot-hub/iot-hub-create-through-portal.md).

> [!NOTE]
> Actuellement, Defender pour IoT prend uniquement en charge les hubs IoT de niveau standard.

## <a name="prerequisites-for-enabling-the-service"></a>Prérequis pour l’activation du service

- Espace de travail Log Analytics
  - Par défaut, deux types d’informations sont stockés par Defender pour IoT dans votre espace de travail Log Analytics : les **alertes de sécurité** et les **recommandations**.
  - Vous pouvez choisir d’ajouter le stockage d’un type d’information supplémentaire : les **événements bruts**. Notez que le stockage d’**événements bruts** dans Log Analytics engendre des frais de stockage supplémentaires.
- IoT Hub (niveau standard)
- Satisfaire à tous les [prérequis du service](service-prerequisites.md)

## <a name="enable-defender-for-iot-on-your-iot-hub"></a>Activer Defender pour IoT sur votre hub IoT

Pour activer la sécurité sur votre hub IoT :

1. Ouvrez votre **hub IoT** dans le portail Azure.
1. Dans le menu **Sécurité**, cliquez sur **Sécuriser votre solution IoT**.

Félicitations ! Vous avez terminé l’activation de Defender pour IoT sur votre hub IoT.

### <a name="geolocation-and-ip-address-handling"></a>Gestion de la géolocalisation et des adresses IP

Pour sécuriser votre solution IoT, les adresses IP des connexions entrantes et sortantes vers et depuis vos appareils IoT, IoT Edge ainsi que les hubs IoT sont collectées et stockées par défaut. Ces informations sont essentielles pour détecter une connectivité anormale à partir de sources IP suspectes, par exemple lorsque des tentatives sont effectuées pour établir des connexions à partir d’une source IP d’un botnet connu ou d’une source IP en dehors de votre géolocalisation. Le service Defender pour IoT offre la flexibilité nécessaire pour activer et désactiver la collecte des données d’adresses IP à tout moment.

Pour activer ou désactiver la collecte des données d’adresses IP :

1. Ouvrez votre hub IoT, puis sélectionnez **Paramètres** dans le menu **Sécurité**.
1. Choisissez l’écran **Collecte de données** et modifiez les paramètres de géolocalisation et/ou de gestion IP comme vous le souhaitez.

### <a name="log-analytics-creation"></a>Création de Log Analytics

Quand Defender pour IoT est activé, un espace de travail Azure Log Analytics par défaut est créé pour stocker les événements de sécurité bruts, les alertes et les recommandations pour vos appareils IoT, IoT Edge et IoT Hub. Chaque mois, les cinq (5) premiers Go de données ingérées par client sur le service Azure Log Analytics sont gratuits. Chaque Go de données ingérées dans votre espace de travail Azure Log Analytics est conservé gratuitement pendant les 31 premiers jours. Découvrez plus en détail les tarifs de [Log Analytics](https://azure.microsoft.com/pricing/details/monitor/).

Pour modifier la configuration de l’espace de travail de Log Analytics :

1. Ouvrez votre hub IoT, puis sélectionnez **Paramètres** dans le menu **Sécurité**.
1. Choisissez l’écran **Collecte de données** et modifiez les paramètres de configuration de l’espace de travail de Log Analytics comme vous le souhaitez.

### <a name="customize-your-iot-security-solution"></a>Personnaliser votre solution de sécurité IoT

Par défaut, l’activation de la solution Defender pour IoT sécurise automatiquement tous les hubs IoT dans votre abonnement Azure.

Pour activer ou désactiver le service Defender pour IoT sur un hub IoT spécifique :

1. Ouvrez votre hub IoT, puis sélectionnez **Paramètres** dans le menu **Sécurité**.
1. Choisissez l’écran **Collecte de données** et modifiez les paramètres de sécurité de n’importe quel hub IoT dans votre abonnement Azure comme vous le souhaitez.

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant pour configurer votre solution...

> [!div class="nextstepaction"]
> [Configurer votre solution](quickstart-configure-your-solution.md)