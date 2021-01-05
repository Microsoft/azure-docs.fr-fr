---
title: Tutoriel - Basculer un appareil physique StorSimple vers le même appareil
description: Découvrez les étapes requises pour basculer un appareil physique de la gamme StorSimple 8000 vers un autre appareil physique StorSimple en cas d’incident.
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
ms.date: 06/23/2017
ms.author: alkohli
ms.openlocfilehash: 17c116194aa52a82246bcee9114824e8a918ebbd
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017098"
---
# <a name="fail-over-your-storsimple-physical-device-to-same-device"></a>Basculer votre appareil physique StorSimple vers le même appareil

## <a name="overview"></a>Vue d’ensemble

Ce didacticiel décrit les étapes requises pour basculer un appareil physique de la gamme StorSimple 8000 vers un autre appareil physique StorSimple en cas d’incident.Ce didacticiel décrit les étapes requises pour basculer un appareil physique de la gamme StorSimple 8000 vers lui-même en cas d’incident. StorSimple utilise l’option de basculement d’appareil pour migrer les données d’un appareil physique source dans le centre de données vers un autre appareil physique. Les instructions de ce didacticiel s’appliquent aux appareils physiques de la gamme StorSimple 8000 exécutant le logiciel Update 3 et versions ultérieures.

Pour plus d’informations sur le basculement d’appareil et son utilisation à des fins de récupération après un incident, accédez à [Failover and disaster recovery for your StorSimple 8000 series device](storsimple-8000-device-failover-disaster-recovery.md) (Basculement et récupération d’urgence pour vos appareils StorSimple de la gamme 8000).

Pour basculer un appareil physique StorSimple vers un autre appareil physique, accédez à [Fail over to the same StorSimple physical device](storsimple-8000-device-failover-physical-device.md) (Basculer votre appareil physique StorSimple vers lui-même). Pour basculer un appareil physique StorSimple vers une instance StorSimple Cloud Appliance, accédez à [Fail over to your StorSimple Cloud Appliance](storsimple-8000-device-failover-cloud-appliance.md) (Basculement vers votre StorSimple Cloud Appliance).


## <a name="prerequisites"></a>Prérequis

- Assurez-vous d’avoir passé en revue les considérations relatives au basculement d’appareil. Pour plus d’informations, accédez à [Considérations courantes relatives au basculement d’appareil](storsimple-8000-device-failover-disaster-recovery.md).


## <a name="steps-to-fail-over-to-the-same-device"></a>Procédure de basculement vers le même appareil

Si vous avez besoin d’effectuer un basculement vers le même appareil, procédez comme suit.

1. Prenez des instantanés de cloud de tous les volumes de votre appareil. Pour plus d’informations, accédez à [Utiliser le service StorSimple Device Manager dans le portail Azure pour gérer les stratégies de sauvegarde](storsimple-8000-manage-backup-policies-u2.md).
2. Réinitialisez votre appareil aux valeurs par défaut. Suivez les instructions détaillées dans la section [Rétablissement des paramètres par défaut de l’appareil](storsimple-8000-manage-device-controller.md#reset-the-device-to-factory-default-settings).
3. Accédez au service StorSimple Device Manager, puis sélectionnez **Appareils**. Dans le panneau **Appareils**, l’ancien appareil doit présenter l’état **Hors connexion**.

    ![Appareil source hors connexion](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev2.png)

4. Configurez votre appareil et réinscrivez-le auprès de votre service StorSimple Device Manager. L’appareil nouvellement inscrit doit présenter l’état **Prêt pour la configuration**. Le nom du nouvel appareil est le même que celui de l’ancien, mais il est suivi d’un chiffre pour indiquer que ses paramètres d’usine ont été rétablis et qu’il a été inscrit à nouveau.

    ![Appareil nouvellement inscrit prêt pour la configuration](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev3.png)
5. Terminez l’installation du nouvel appareil. Pour plus d'informations, accédez à [Étape 4 : Mener à bien l'installation minimale de l'appareil](storsimple-8000-deployment-walkthrough-u2.md#step-4-complete-minimum-device-setup). Dans le panneau **Appareils**, l’appareil présente maintenant l’état **En ligne**.

   > [!IMPORTANT]
   > **Effectuez tout d’abord la configuration minimale. Sinon, la récupération d’urgence risque d’échouer.**

    ![Appareil nouvellement inscrit en ligne](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev7.png)

6. Sélectionnez l’ancien appareil (état Hors connexion) et dans la barre de commandes, cliquez sur **Basculement**. Dans le panneau **Basculement**, sélectionnez l’ancien appareil en tant que source et spécifiez l’appareil nouvellement inscrit en tant qu’appareil cible.

    ![Synthèse du basculement](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev11.png)

    Pour obtenir des instructions détaillées, reportez-vous à la section Basculer vers un autre appareil physique.

7. Un travail de restauration de l’appareil est créé. Vous pouvez suivre sa progression à partir du panneau **Travaux** .

8. Une fois ce travail terminé, accédez au nouvel appareil, puis au panneau **Conteneurs de volumes** . Vérifiez que tous les conteneurs de volumes de l’ancien appareil ont été migrés vers le nouvel appareil.

   ![Conteneurs de volumes migrés](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev13.png)

9. Une fois le basculement terminé, vous pouvez désactiver et supprimer l’ancien appareil à partir du portail. Sélectionnez l’ancien appareil, cliquez avec le bouton droit, puis sélectionnez **Désactiver**. Une fois l’appareil désactivé, son état est mis à jour.

     ![Appareil source désactivé](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev14.png)

10. Sélectionnez l’appareil désactivé, cliquez avec le bouton droit, puis sélectionnez **Supprimer**. Votre appareil est supprimé de la liste d’appareils.

    ![Appareil source supprimé](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev15.png)



## <a name="next-steps"></a>Étapes suivantes

* Après avoir effectué un basculement, vous devrez peut-être [désactiver ou supprimer votre appareil StorSimple](storsimple-8000-deactivate-and-delete-device.md).
* Pour plus d’informations sur l’utilisation du service StorSimple Device Manager, accédez à [Use the StorSimple Device Manager service to administer your StorSimple device](storsimple-8000-manager-service-administration.md) (Utiliser le service StorSimple Device Manager pour gérer votre appareil StorSimple).

