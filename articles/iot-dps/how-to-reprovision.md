---
title: Réapprovisionner des appareils dans le service Azure IoT Hub Device Provisioning
description: Découvrez comment reprovisionner des appareils avec votre instance DPS (Device Provisioning Service) et pourquoi vous devrez peut-être effectuer cette opération.
author: wesmc7777
ms.author: wesmc
ms.date: 04/04/2019
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
ms.openlocfilehash: e5cc5b557aa4dff793f7e87093eeb65028da4f8c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010943"
---
# <a name="how-to-reprovision-devices"></a>Comment reprovisionner des appareils

Durant le cycle de vie d’une solution IoT, il est fréquent d’avoir à déplacer des appareils d’un hub IoT à un autre. Voici quelques scénarios nécessitant le déplacement d’un appareil :

* **Géolocalisation** : quand un appareil change d’emplacement, il doit être migré vers un hub IoT plus proche de son emplacement pour améliorer la latence du réseau.

* **Multilocation** : un appareil peut être utilisé dans la même solution IoT, mais réaffecté ou donné en location à un nouveau client ou au site d’un client. Le service fourni à ce nouveau client peut utiliser un autre hub IoT.

* **Changement de solution** : un appareil est déplacé dans une nouvelle solution IoT ou une solution IoT mise à jour. Cette réaffectation peut nécessiter que l’appareil communique avec un nouveau hub IoT qui est connecté à d’autres composants backend. 

* **Mise en quarantaine** : ce scénario est similaire à un changement de solution. Un appareil défectueux, compromis ou obsolète est réassigné à un hub IoT où il doit obligatoirement être mis à jour afin de redevenir conforme. Une fois que l’appareil fonctionne correctement, il est migré vers son hub principal d’origine.

Pour plus d’informations sur le reprovisionnement, consultez [Concepts du reprovisionnement d’appareils IoT Hub](concepts-device-reprovision.md).


## <a name="configure-the-enrollment-allocation-policy"></a>Configurer la stratégie d’allocation de l’inscription

La stratégie d’allocation détermine comment les appareils associés à l’inscription sont alloués ou affectés à un hub IoT une fois reprovisionnés.

Les étapes suivantes configurent la stratégie d’allocation pour l’inscription d’un appareil :

1. Connectez-vous au [portail Azure](https://portal.azure.com) et accédez à votre instance du service Device Provisioning.

2. Cliquez sur **Gérer les inscriptions**, puis sur le groupe d’inscriptions ou sur l’inscription individuelle que vous voulez configurer pour le reprovisionnement. 

3. Sous **Sélectionner le mode d’affectation des appareils aux hubs**, sélectionnez une des stratégies d’allocation suivantes :

    * **Latence la plus faible** : cette stratégie affecte les appareils au hub IoT lié qui permet les communications avec la latence la plus faible entre l’appareil et IoT Hub. Cette option permet à l’appareil communiquer avec le hub IoT le plus proche de l’emplacement. 
    
    * **Distribution uniformément pondérée** : cette stratégie distribue les appareils entre les hubs IoT liés en fonction de la pondération d’allocation affectée à chaque hub IoT lié. Cette stratégie vous permet d’équilibrer la charge des appareils sur un groupe de hubs liés en fonction des pondérations d’allocation définies sur ces hubs. Si vous provisionnez des appareils sur un seul hub IoT, nous vous recommandons ce paramètre. Il s’agit du paramètre par défaut. 
    
    * **Configuration statique** : cette stratégie nécessite que le hub IoT souhaité figure dans la liste des entrées d’inscription pour un appareil à provisionner. Cette stratégie vous permet de désigner un hub IoT spécifique auquel vous voulez affecter les appareils.

4. Sous **Sélectionner les hubs IoT auxquels ce groupe peut être affecté**, sélectionnez les hubs IoT liés que vous voulez inclure avec votre stratégie d’allocation. Si vous le souhaitez, ajoutez un nouveau hub IoT lié en utilisant le bouton **Lier un nouveau hub IoT**.

    Avec la stratégie d’allocation **Latence la plus faible**, les hubs que vous sélectionnez sont inclus dans l’évaluation de la latence pour déterminer le hub le plus proche pour l’affectation des appareils.

    Avec la stratégie d’allocation **Distribution uniformément pondérée**, la charge des appareils est équilibrée entre les hubs que vous sélectionnez, en fonction de leurs pondérations d’allocation configurées et de leur charge actuelle d’appareils.

    Avec la stratégie d’allocation **Configuration statique**, sélectionnez le hub IoT auquel vous voulez affecter les appareils.

4. Cliquez sur **Enregistrer** ou passez à la section suivante pour définir la stratégie de reprovisionnement.

    ![Sélectionner la stratégie d’allocation de l’inscription](./media/how-to-reprovision/enrollment-allocation-policy.png)



## <a name="set-the-reprovisioning-policy"></a>Définir la stratégie de reprovisionnement

1. Connectez-vous au [portail Azure](https://portal.azure.com) et accédez à votre instance du service Device Provisioning.

2. Cliquez sur **Gérer les inscriptions**, puis sur le groupe d’inscriptions ou sur l’inscription individuelle que vous voulez configurer pour le reprovisionnement.

3. Sous **Sélectionner comment vous voulez que les données des appareils soient gérées lors du reprovisionnement sur un autre hub IoT**, choisissez une des stratégies de reprovisionnement suivantes :

    * **Reprovisionner et migrer les données** : cette stratégie est appliquée quand des appareils associés à l’entrée d’inscription envoient une demande de nouveau provisionnement. Selon la configuration de l’entrée d’inscription, l’appareil peut être réassigné à un autre hub IoT. Si l’appareil est déplacé vers un autre hub IoT, il est désinscrit du hub IoT initial. Toutes les données d’état de l’appareil provenant de ce hub IoT initial sont alors migrées vers le nouveau hub IoT. Durant la migration, l’état de l’appareil indique **Affectation**.

    * **Reprovisionner et rétablir la configuration initiale** : cette stratégie est appliquée quand des appareils associés à l’entrée d’inscription envoient une demande de nouveau provisionnement. Selon la configuration de l’entrée d’inscription, l’appareil peut être réassigné à un autre hub IoT. Si l’appareil est déplacé vers un autre hub IoT, il est désinscrit du hub IoT initial. Les données de configuration initiale que l’instance du service Device Provisioning a reçues au moment du provisionnement de l’appareil sont transmises au nouveau hub IoT. Durant la migration, l’état de l’appareil indique **Affectation**.

4. Cliquez sur **Enregistrer** pour activer le reprovisionnement de l’appareil en fonction de vos modifications.

    ![Capture d’écran qui met en évidence les modifications que vous avez apportées et le bouton Enregistrer.](./media/how-to-reprovision/reprovisioning-policy.png)



## <a name="send-a-provisioning-request-from-the-device"></a>Envoyer une demande de provisionnement depuis l’appareil

Pour que les appareils puissent être reprovisionnés en fonction des modifications de configuration apportées dans les sections précédentes, ces appareils doivent demander un reprovisionnement. 

La fréquence à laquelle un appareil envoie une demande de provisionnement varie selon le scénario. Il est cependant conseillé de programmer vos appareils pour qu’ils envoient une demande de provisionnement à une instance du service de provisionnement au redémarrage, et pour qu’ils prennent en charge une [méthode](../iot-hub/iot-hub-devguide-direct-methods.md) permettant de déclencher manuellement le provisionnement à la demande. Le provisionnement peut également être déclenché en définissant une [propriété souhaitée](../iot-hub/iot-hub-devguide-device-twins.md#desired-property-example). 

La stratégie de reprovisionnement sur une entrée d’inscription définit de quelle manière l’instance du service Device Provisioning gère ces demandes de provisionnement, et détermine si les données d’état de l’appareil doivent être migrées au moment du reprovisionnement. Les mêmes stratégies sont disponibles pour les inscriptions individuelles et les groupes d’inscriptions :

Pour obtenir un exemple de code qui envoie des demandes de provisionnement depuis un appareil lors d’une séquence de démarrage, consultez [Provisionnement automatique d’un appareil simulé](quick-create-simulated-device.md).


## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur le reprovisionnement, consultez [Concepts du reprovisionnement d’appareils IoT Hub](concepts-device-reprovision.md) 
- Pour en savoir plus sur le déprovisionnement, consultez [Guide pratique pour déprovisionner des appareils auparavant provisionnés automatiquement](how-to-unprovision-devices.md) 











