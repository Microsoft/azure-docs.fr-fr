---
title: Basculement et récupération d’urgence vers un autre appareil StorSimple 8000
description: Découvrez comment basculer votre appareil physique de la gamme StorSimple 8000 vers un autre appareil physique.
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
ms.date: 05/03/2017
ms.author: alkohli
ms.openlocfilehash: 67d6fb1f3cc359288ed942d915e186542a62b0fc
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017132"
---
# <a name="fail-over-to-a-storsimple-8000-series-physical-device"></a>Basculer vers appareil physique de la gamme StorSimple 8000

## <a name="overview"></a>Vue d’ensemble

Ce didacticiel décrit les étapes requises pour basculer un appareil physique de la gamme StorSimple 8000 vers un autre appareil physique StorSimple en cas d’incident. StorSimple utilise l’option de basculement d’appareil pour migrer les données d’un appareil physique source dans le centre de données vers un autre appareil physique. Les instructions de ce didacticiel s’appliquent aux appareils physiques de la gamme StorSimple 8000 exécutant le logiciel Update 3 et versions ultérieures.

Pour plus d’informations sur le basculement d’appareil et son utilisation à des fins de récupération après un incident, accédez à [Failover and disaster recovery for your StorSimple 8000 series device](storsimple-8000-device-failover-disaster-recovery.md) (Basculement et récupération d’urgence pour vos appareils StorSimple de la gamme 8000).

Pour basculer un appareil physique StorSimple vers une instance StorSimple Cloud Appliance, accédez à [Fail over to your StorSimple Cloud Appliance](storsimple-8000-device-failover-cloud-appliance.md) (Basculement vers votre StorSimple Cloud Appliance). Pour basculer un appareil physique vers lui-même, accédez à [Fail over your StorSimple physical device to same device](storsimple-8000-device-failover-same-device.md) (Basculer votre appareil physique StorSimple vers lui-même).


## <a name="prerequisites"></a>Conditions préalables requises

- Assurez-vous d’avoir passé en revue les considérations relatives au basculement d’appareil. Pour plus d’informations, accédez à [Considérations courantes relatives au basculement d’appareil](storsimple-8000-device-failover-disaster-recovery.md).

- Vous devez disposer d’un appareil physique de la gamme StorSimple 8000 déployé dans le centre de données. L’appareil doit exécuter la version logicielle Update 3 ou une version ultérieure. Pour plus d’informations, accédez à la page [Déployer votre appareil local StorSimple](storsimple-8000-deployment-walkthrough-u2.md).


## <a name="steps-to-fail-over-to-a-physical-device"></a>Procédure de basculement vers un appareil physique

Procédez comme suit pour restaurer votre appareil vers un appareil physique.

1. Vérifiez que le conteneur de volumes que vous souhaitez basculer est associé à des instantanés cloud. Pour plus d’informations, accédez à [Utiliser le service StorSimple Device Manager dans le portail Azure pour gérer les stratégies de sauvegarde](storsimple-8000-manage-backup-policies-u2.md).
2. Accédez à votre StorSimple Device Manager et cliquez sur **Appareils**. Dans le panneau **Appareils**, accédez à la liste des appareils connectés à votre service.
    ![Sélectionner l’appareil](./media/storsimple-8000-device-failover-disaster-recovery/failover-phy-dev1.png)
3. Sélectionnez votre appareil source et cliquez dessus. L’appareil source comprend les conteneurs de volumes que vous souhaitez basculer. Accédez à **Paramètres > Conteneurs de volumes**.
4. Sélectionnez un conteneur de volume que vous souhaitez basculer vers un autre appareil. Cliquez sur le conteneur de volume pour afficher la liste des volumes dans ce conteneur. Sélectionnez un volume, cliquez dessus avec le bouton droit, puis cliquez sur **Mettre hors connexion** afin de mettre le volume hors connexion. Répétez ce processus pour tous les volumes dans le conteneur de volume.
5. Répétez l’étape précédente pour tous les conteneurs de volume que vous souhaitez basculer vers un autre appareil.
6. Revenez au panneau **Appareils**. Dans la barre de commandes, cliquez sur **Effectuer un basculement**.
    ![Cliquer sur Effectuer un basculement](./media/storsimple-8000-device-failover-disaster-recovery/failover-phy-dev2.png)
    
7. Dans le panneau **Effectuer un basculement**, procédez comme suit :
   
   1. Cliquez sur **Source**. Les conteneurs de volumes avec des volumes associés à des captures instantanées cloud sont affichés. Seuls les conteneurs affichés sont éligibles pour le basculement. dans la liste des conteneurs de volume, sélectionnez les conteneurs de volume que vous souhaitez basculer. **Seuls les conteneurs de volumes associés à des instantanés cloud et des volumes hors connexion sont affichés.**

       ![Sélectionner une source](./media/storsimple-8000-device-failover-disaster-recovery/failover-phy-dev5.png)
   2. Cliquez sur **Cible**. Pour les conteneurs de volumes sélectionnés lors de l’étape précédente, sélectionnez un appareil cible dans la liste déroulante des appareils disponibles. Seuls les appareils disposant d’une capacité suffisante pour accueillir les conteneurs de volumes source sont affichés dans la liste.

        ![Sélectionner la cible](./media/storsimple-8000-device-failover-disaster-recovery/failover-phy-dev6.png)

   3. Enfin, passez en revue tous les paramètres de basculement sous **Résumé**. Après avoir passé en revue les paramètres, cochez la case indiquant que les volumes dans les conteneurs de volumes sélectionnés sont hors connexion. Cliquez sur **OK**.

       ![Passer en revue les paramètres de basculement](./media/storsimple-8000-device-failover-disaster-recovery/failover-phy-dev8.png)
  
8. StorSimple crée un travail de basculement. Pour surveiller le travail de basculement, cliquez sur la notification de travail via le panneau **Travaux**.

    Si le conteneur de volumes que vous avez basculé possède des volumes locaux, vous voyez les travaux de restauration individuels pour chaque volume local (et non pour les volumes à plusieurs niveaux) dans le conteneur. Ces tâches de restauration peuvent prendre un certain temps. Il est probable que le travail de basculement se termine plus tôt. Ces volumes auront des garanties locales uniquement une fois les travaux de restauration terminés.

    ![Surveiller le travail de basculement](./media/storsimple-8000-device-failover-disaster-recovery/failover-phy-dev13.png)

9. Une fois le basculement terminé, accédez au panneau **Appareils**.
   
   1. Sélectionnez l’appareil qui a été utilisé en tant qu’appareil cible pour le processus de basculement.

       ![Sélection de l’appareil 2](./media/storsimple-8000-device-failover-disaster-recovery/failover-phy-dev14.png)

   2. Accédez au panneau **Conteneurs de volumes**. Tous les conteneurs de volume, ainsi que les volumes de l’ancien appareil, doivent être répertoriés.

       ![Afficher les conteneurs de volumes cibles](./media/storsimple-8000-device-failover-disaster-recovery/failover-phy-dev16.png)


## <a name="next-steps"></a>Étapes suivantes

* Après avoir effectué un basculement, vous devrez peut-être [désactiver ou supprimer votre appareil StorSimple](storsimple-8000-deactivate-and-delete-device.md).
* Pour plus d’informations sur l’utilisation du service StorSimple Device Manager, accédez à [Use the StorSimple Device Manager service to administer your StorSimple device](storsimple-8000-manager-service-administration.md) (Utiliser le service StorSimple Device Manager pour gérer votre appareil StorSimple).

