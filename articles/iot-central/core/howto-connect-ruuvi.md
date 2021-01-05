---
title: Connecter un capteur RuuviTag dans Azure IoT Central | Microsoft Docs
description: Apprenez à connecter un capteur environnemental RuuviTag à votre application IoT Central.
services: iot-central
ms.service: iot-central
ms.topic: how-to
ms.custom:
- iot-storeAnalytics-conditionMonitor
- iot-p0-scenario
ms.author: avneets
author: avneet723
ms.date: 11/27/2019
ms.openlocfilehash: 7cf0d31895eaeaa41c23f9f07664b27bed1d3126
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96006385"
---
# <a name="connect-a-ruuvitag-sensor-to-your-azure-iot-central-application"></a>Connecter un capteur RuuviTag à votre application Azure IoT Central

*Cet article s’applique aux créateurs de solutions.*

Cet article vous explique comment, en tant que concepteur de solutions, vous pouvez connecter un capteur RuuviTag à votre application Microsoft Azure IoT Central.

Qu'est-ce qu'une balise Ruuvi ?

RuuviTag est une plateforme open source avancée pour balises dotées de capteurs conçue pour répondre aux besoins des clients professionnels, des développeurs, des fabricants, des étudiants et des amateurs. L’appareil est configuré pour fonctionner dès que vous le sortez de son emballage et que vous le déployez là où vous en avez besoin. Il s’agit d’une balise Bluetooth Low Energy intégrant un capteur environnemental et un accéléromètre.

RuuviTag communique via BLE (Bluetooth Low Energy) et nécessite un appareil de passerelle pour s’adresser à Azure IoT Central. Veillez à disposer d’un appareil de passerelle, comme le Rigado Cascade 500, configuré pour permettre à un capteur RuuviTag de se connecter à IoT Central.

Suivez les [instructions disponibles ici](./howto-connect-rigado-cascade-500.md) si vous souhaitez configurer un appareil de passerelle Rigado Cascade 500.

## <a name="prerequisites"></a>Prérequis

Pour connecter des capteurs RuuviTag, vous devez disposer des ressources suivantes :

* Un capteur RuuviTag. Pour plus d'informations, consultez le site de [RuuviTag](https://ruuvi.com/).
* Un appareil Rigado Cascade 500 ou une autre passerelle BLE. Pour plus d’informations, veuillez visiter [Rigado](https://www.rigado.com/).
* Une application Azure IoT Central. Pour plus d'informations, consultez [Créer une application](./quick-deploy-iot-central.md).

## <a name="add-a-ruuvitag-device-template"></a>Ajouter un modèle d'appareil RuuviTag

Pour intégrer un capteur RuuviTag dans votre instance d’application Azure IoT Central, vous devez configurer un modèle d’appareil correspondant dans votre application.

Pour ajouter un modèle d'appareil RuuviTag :

1. Accédez à l’onglet ***Modèles d’appareil** _ dans le volet gauche, sélectionnez _*+ Nouveau** :  ![Créer un modèle d'appareil](./media/howto-connect-ruuvi/devicetemplate-new.png). La page vous donne la possibilité de **_Créer un modèle personnalisé_*_ ou d'_*_Utiliser un modèle d'appareil préconfiguré_*_
1. Sélectionnez le modèle d'appareil RuuviTag dans la liste des modèles d'appareil préconfigurés, comme indiqué ci-dessous :  ![Sélectionner le modèle d'appareil RuuviTag](./media/howto-connect-ruuvi/devicetemplate-preconfigured.png)
1. Sélectionnez _*_Suivant : Personnaliser_*_ pour passer à l'étape suivante.
1. Dans l'écran suivant, sélectionnez _*_Créer_*_ pour intégrer le modèle d'appareil C500 à votre application IoT Central.

## <a name="connect-a-ruuvitag-sensor"></a>Connecter un capteur RuuviTag

Comme indiqué précédemment, pour connecter le capteur RuuviTag à votre application IoT Central, vous devez configurer un appareil de passerelle. Les étapes ci-dessous partent du principe que vous avez configuré un appareil de passerelle Rigado Cascade 500.  

1. Allumez votre appareil Rigado Cascade 500 et reliez-le à votre connexion réseau (via Ethernet ou sans fil).
1. Retirez le couvercle du RuuviTag et tirez sur la languette en plastique pour sécuriser la connexion avec la batterie.
1. Placez le capteur RuuviTag à proximité d’une passerelle Rigado Cascade 500 déjà configurée dans votre application IoT Central.
1. En quelques secondes, votre RuuviTag doit apparaître dans la liste des appareils d'IoT Central.  
    ![Liste des appareils RuuviTag](./media/howto-connect-ruuvi/ruuvi-devicelist.png)

Vous pouvez maintenant utiliser ce capteur RuuviTag dans votre application IoT Central.  

## <a name="create-a-simulated-ruuvitag"></a>Créer un RuuviTag simulé

Si vous n’avez pas de capteur RuuviTag physique, vous pouvez créer un capteur RuuviTag simulé à utiliser à des fins de test dans votre application Azure IoT Central.

Pour créer un RuuviTag simulé :

1. Sélectionnez _*Appareils > RuuviTag**.
1. Sélectionnez **+Nouveau**.
1. Spécifiez un **ID d'appareil** unique et un **nom d'appareil** convivial.  
1. Activez le paramètre **Simulé**.
1. Sélectionnez **Create** (Créer).  

## <a name="next-steps"></a>Étapes suivantes

Si vous êtes un développeur d’appareils, les étapes suivantes suggérées sont les suivantes :

- En savoir plus sur la [connectivité des appareils dans Azure IoT Central](./concepts-get-connected.md)
- Découvrir comment [superviser la connectivité des appareils à l’aide d’Azure CLI](./howto-monitor-devices-azure-cli.md)
