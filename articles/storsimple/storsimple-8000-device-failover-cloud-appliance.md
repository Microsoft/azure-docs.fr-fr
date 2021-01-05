---
title: Basculement et reprise d’activité vers une instance StorSimple Cloud Appliance
description: Découvrez comment basculer votre appareil physique de la gamme StorSimple 8000 vers une appliance cloud.
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/03/2017
ms.author: alkohli
ms.openlocfilehash: 18b34ff466f3935cb5cd18d46e6d26e36e756a9b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91774458"
---
# <a name="fail-over-to-your-storsimple-cloud-appliance"></a>Basculer vers votre StorSimple Cloud Appliance

## <a name="overview"></a>Vue d’ensemble

Ce didacticiel décrit les étapes requises pour basculer un appareil physique de la gamme StorSimple 8000 vers une StorSimple Cloud Appliance en cas de sinistre. StorSimple utilise l’option de basculement d’appareil pour migrer des données d’un appareil physique source dans le centre de données vers une appliance cloud exécutée dans Azure. Les instructions de ce didacticiel s’appliquent aux appareils physiques de gamme StorSimple 8000 et aux appliances cloud exécutant des versions logicielles Update 3 et versions ultérieures.

Pour plus d’informations sur le basculement d’appareil et son utilisation à des fins de récupération après un incident, accédez à [Failover and disaster recovery for your StorSimple 8000 series device](storsimple-8000-device-failover-disaster-recovery.md) (Basculement et récupération d’urgence pour vos appareils StorSimple de la gamme 8000).

Pour basculer un appareil physique StorSimple vers un autre appareil physique, accédez à [Basculer vers un appareil physique StorSimple](storsimple-8000-device-failover-physical-device.md). Pour basculer un appareil vers lui-même, accédez à [Basculer vers le même appareil physique StorSimple](storsimple-8000-device-failover-same-device.md).

## <a name="prerequisites"></a>Conditions préalables requises

- Assurez-vous d’avoir passé en revue les considérations relatives au basculement d’appareil. Pour plus d’informations, accédez à [Considérations courantes relatives au basculement d’appareil](storsimple-8000-device-failover-disaster-recovery.md).

- Vous devez avoir créé et configuré une StorSimple Cloud Appliance avant d’exécuter cette procédure. Si vous exécutez la version logicielle Update 3 ou une version ultérieure, envisagez d’utiliser une appliance cloud 8020 pour la récupération d’urgence. Le modèle 8020 a 64 To et utilise le stockage Premium. Pour plus d’informations, accédez à [Déployer et gérer une StorSimple Cloud Appliance](storsimple-8000-cloud-appliance-u2.md).

## <a name="steps-to-fail-over-to-a-cloud-appliance"></a>Étapes de basculement vers une appliance cloud

Procédez comme suit pour restaurer l’appareil vers une StorSimple Cloud Appliance cible.

1.  Vérifiez que le conteneur de volumes que vous souhaitez basculer est associé à des instantanés cloud. Pour plus d’informations, accédez à [Utiliser le service StorSimple Device Manager dans le portail Azure pour gérer les stratégies de sauvegarde](storsimple-8000-manage-backup-policies-u2.md).
2. Accédez à votre service StorSimple Device Manager et cliquez sur **Appareils**. Dans le panneau **Appareils**, accédez à la liste des appareils connectés à votre service.
    ![Sélectionner l’appareil](./media/storsimple-8000-device-failover-disaster-recovery/failover-cloud-dev1.png)
3. Sélectionnez votre appareil source et cliquez dessus. L’appareil source comprend les conteneurs de volumes que vous souhaitez basculer. Accédez à **Paramètres > Conteneurs de volumes**.

    ![Sélection de l’appareil 2](./media/storsimple-8000-device-failover-disaster-recovery/failover-cloud-dev2.png)
    
4. Sélectionnez un conteneur de volume que vous souhaitez basculer vers un autre appareil. Cliquez sur le conteneur de volume pour afficher la liste des volumes dans ce conteneur. Sélectionnez un volume, cliquez dessus avec le bouton droit, puis cliquez sur **Mettre hors connexion** afin de mettre le volume hors connexion.

    ![Sélection de l'appareil 3](./media/storsimple-8000-device-failover-disaster-recovery/failover-cloud-dev5.png)

5. Répétez ce processus pour tous les volumes dans le conteneur de volume.

     ![Sélection de l'appareil 4](./media/storsimple-8000-device-failover-disaster-recovery/failover-cloud-dev7.png)

6. Répétez l’étape précédente pour tous les conteneurs de volume que vous souhaitez basculer vers un autre appareil.

7. Revenez au panneau **Appareils**. Dans la barre de commandes, cliquez sur **Effectuer un basculement**.

    ![Cliquer sur Effectuer un basculement](./media/storsimple-8000-device-failover-disaster-recovery/failover-cloud-dev8.png)
8. Dans le panneau **Effectuer un basculement**, procédez comme suit :
   
    1. Cliquez sur **Source**. Sélectionnez les conteneurs de volumes à basculer. **Seuls les conteneurs de volumes associés à des instantanés cloud et des volumes hors connexion sont affichés.**
        ![Sélectionner une source](./media/storsimple-8000-device-failover-disaster-recovery/failover-cloud-dev11.png)
    2. Cliquez sur **Cible**. Sélectionnez une appliance cloud cible dans la liste déroulante des appareils disponibles. **Seuls les appareils disposant d’une capacité suffisante pour accueillir les conteneurs de volumes source sont affichés dans la liste.**

        ![Sélectionner la cible](./media/storsimple-8000-device-failover-disaster-recovery/failover-cloud-dev12.png)

    3. Passez en revue les paramètres de basculement sous **Résumé** et activez la case à cocher indiquant que les volumes dans les conteneurs de volumes sélectionnés sont hors connexion. 

        ![Passer en revue les paramètres de basculement](./media/storsimple-8000-device-failover-disaster-recovery/failover-cloud-dev13.png)

9. Un travail de basculement est créé. Pour surveiller le travail de basculement, cliquez sur la notification de travail.

    ![Surveiller le travail de basculement](./media/storsimple-8000-device-failover-disaster-recovery/failover-phy-dev13.png)

10. Une fois le basculement terminé, revenez au panneau **Appareils** .

    1. Sélectionnez l’appareil qui a été utilisé en tant que cible pour le basculement.

       ![Sélection de l'appareil 5](./media/storsimple-8000-device-failover-disaster-recovery/failover-phy-dev14.png)

    2. Cliquez sur **Conteneurs de volumes**. Tous les conteneurs de volume, ainsi que les volumes de l’ancien appareil, doivent être répertoriés.

       Si le conteneur de volumes que vous avez basculé contient des volumes épinglés localement, ces volumes sont basculés en tant que volumes à plusieurs niveaux. Les volumes épinglés localement ne sont pas pris en charge sur une StorSimple Cloud Appliance.

       ![Afficher les conteneurs de volumes cibles](./media/storsimple-8000-device-failover-disaster-recovery/failover-phy-dev17.png)


## <a name="next-steps"></a>Étapes suivantes

* Après avoir effectué un basculement, vous devrez peut-être [désactiver ou supprimer votre appareil StorSimple](storsimple-8000-deactivate-and-delete-device.md).

* Pour plus d’informations sur l’utilisation du service StorSimple Device Manager, accédez à [Use the StorSimple Device Manager service to administer your StorSimple device](storsimple-8000-manager-service-administration.md) (Utiliser le service StorSimple Device Manager pour gérer votre appareil StorSimple).

