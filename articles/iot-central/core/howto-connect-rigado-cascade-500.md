---
title: Connecter un Rigado Cascade 500 dans Azure IoT Central | Microsoft Docs
description: Découvrez comment connecter un appareil de passerelle Rigado Cascade 500 à votre application IoT Central.
services: iot-central
ms.service: iot-central
ms.topic: how-to
ms.custom:
- iot-storeAnalytics-conditionMonitor
- iot-p0-scenario
ms.author: avneets
author: avneet723
ms.date: 11/27/2019
ms.openlocfilehash: 0000e7690ab92f469a7417e82cb375c524e0b343
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96014760"
---
# <a name="connect-a-rigado-cascade-500-gateway-device-to-your-azure-iot-central-application"></a>Connecter un appareil de passerelle Rigado Cascade 500 à votre application IoT Central

*Cet article s’applique aux créateurs de solutions.*

Cet article vous explique comment, en tant que concepteur de solutions, vous pouvez connecter un appareil de passerelle Rigado Cascade 500 à votre application Microsoft Azure IoT Central. 

## <a name="what-is-cascade-500"></a>Qu’est-ce qu’un Cascade 500 ?

Une passerelle IoT Cascade 500 est une offre matérielle de Rigado incluse dans le cadre de sa solution Cascade EaaS. Il fournit aux équipes commerciales et de projets IoT une solution flexible de calcul de périphérie, un environnement d’application en conteneur robuste et un large éventail d’options de connectivité de périphériques sans fil, notamment Bluetooth 5, LTE, & Wi-Fi.

Cascade 500 est précertifié pour Azure IoT Plug-and-Play (préversion), ce qui permet aux créateurs de solutions d’intégrer facilement l’appareil dans leurs solutions de bout en bout. La passerelle Cascade vous permet d’établir une connexion sans fil avec un large éventail de capteurs de surveillance de condition qui sont à proximité de l’appareil de passerelle. Ces capteurs peuvent être intégrés dans IoT Central via l’appareil de passerelle.

## <a name="prerequisites"></a>Prérequis
Pour parcourir ce guide pratique, vous devez disposer des ressources suivantes :

* Un appareil Rigado Cascade 500. Pour plus d’informations, veuillez visiter [Rigado](https://www.rigado.com/).
* Une application Azure IoT Central. Pour plus d'informations, consultez [Créer une application](./quick-deploy-iot-central.md).

## <a name="add-a-device-template"></a>Ajouter un modèle d’appareil

Pour intégrer un appareil de passerelle Cascade 500 à votre instance d’application Azure IoT Central, vous devez configurer un modèle d’appareil correspondant dans votre application.

Pour ajouter un modèle d’appareil Cascade 500 : 

1. Accédez à l’onglet ***Modèles d’appareil** _ dans le volet gauche, sélectionnez _*+ Nouveau** : ![Créer un nouveau modèle d’appareil](./media/howto-connect-rigado-cascade-500/device-template-new.png)
1. La page vous donne la possibilité de **_Créer un modèle personnalisé_* ou d’_*_Utiliser un modèle d’appareil préconfiguré_*_
1. Sélectionnez le modèle d’appareil C500 dans la liste des modèles d’appareil préconfigurés, comme indiqué ci-dessous : ![Sélectionner un modèle d’appareil C500](./media/howto-connect-rigado-cascade-500/device-template-preconfigured.png)
1. Sélectionnez _*_Suivant : Personnaliser_*_ pour passer à l'étape suivante. 
1. Dans l'écran suivant, sélectionnez _*_Créer_*_ pour intégrer le modèle d'appareil C500 à votre application IoT Central.

## <a name="retrieve-application-connection-details"></a>Récupérer les détails de connexion de l’application

Vous devez maintenant récupérer _ *l’ID d’étendue** et la **clé primaire** pour votre application Azure IoT Central afin de connecter l’appareil Cascade 500. 

1. Accédez à **Administration** dans le volet gauche, puis cliquez sur **Connexion de l’appareil**. 
2. Prenez note de l’**ID d’étendue** pour votre application IoT Central.
![ID d’étendue d’application](./media/howto-connect-rigado-cascade-500/app-scope-id.png)
3. Cliquez maintenant sur **Afficher les clés** et prenez note de la **Clé primaire**
![Clé primaire](./media/howto-connect-rigado-cascade-500/primary-key-sas.png)  

## <a name="contact-rigado-to-connect-the-gateway"></a>Contacter Rigado pour connecter la passerelle 

Pour connecter l’appareil Cascade 500 à votre application IoT Central, vous devez contacter Rigado et lui fournir les détails de connexion de l’application à partir des étapes ci-dessus. 

Une fois l’appareil connecté à Internet, Rigado est en mesure de transmettre une mise à jour de configuration à l’appareil de passerelle Cascade 500 via un canal sécurisé. 

Cette mise à jour appliquera les détails de connexion IoT Central sur l’appareil Cascade 500 et il apparaîtra dans la liste des appareils. 

![Liste d’appareils](./media/howto-connect-rigado-cascade-500/devices-list-c500.png)  

Vous êtes maintenant prêt à utiliser votre appareil C500 dans votre application IoT Central !

## <a name="next-steps"></a>Étapes suivantes

Si vous êtes un développeur d’appareils, nous vous suggérons les étapes suivantes :

- En savoir plus sur la [connectivité des appareils dans Azure IoT Central](./concepts-get-connected.md)
- Découvrir comment [superviser la connectivité des appareils à l’aide d’Azure CLI](./howto-monitor-devices-azure-cli.md)
