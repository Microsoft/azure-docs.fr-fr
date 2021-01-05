---
title: Désactiver et supprimer un appareil de la gamme StorSimple 8000 | Microsoft Docs
description: Apprenez à désactiver et à supprimer un appareil StorSimple qui est connecté à un service StorSimple Device Manager.
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/23/2018
ms.author: alkohli
ms.openlocfilehash: 07d108306fdca9bfe8f793b61660550e43151d71
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017183"
---
# <a name="deactivate-and-delete-a-storsimple-device"></a>Désactiver et supprimer un appareil StorSimple

## <a name="overview"></a>Vue d’ensemble

Cet article explique comment désactiver et supprimer un appareil StorSimple qui est connecté à un service StorSimple Device Manager. Les instructions de cet article s’appliquent uniquement aux appareils de la gamme StorSimple 8000, y compris les instances StorSimple Cloud Appliance. Si vous utilisez un StorSimple Virtual Array, consultez [Désactiver et supprimer un StorSimple Virtual Array](storsimple-virtual-array-deactivate-and-delete-device.md).

La désactivation interrompt la connexion entre l’appareil et le service StorSimple Device Manager correspondant. Vous pouvez souhaiter mettre un appareil StorSimple hors service (par exemple, si vous voulez le remplacer ou le mettre à niveau, ou si vous ne voulez plus utiliser StorSimple). Dans ce cas, vous devez désactiver l’appareil pour pouvoir le supprimer.

Si vous désactivez un appareil, les données stockées localement sur ce dernier ne sont plus accessibles. Seules les données associées à l’appareil qui a été stocké dans le cloud peuvent être récupérées.

> [!WARNING]
> La désactivation est une opération DÉFINITIVE et ne peut pas être annulée. Un appareil désactivé ne peut pas être inscrit auprès du service StorSimple Device Manager à moins de rétablir ses paramètres d’usine.
>
> La réinitialisation aux paramètres d’usine supprime toutes les données stockées localement sur votre appareil. Par conséquent, vous devez impérativement prendre un instantané cloud de toutes vos données avant de désactiver un appareil. Cet instantané cloud vous permet de récupérer ultérieurement l’ensemble des données.

> [!NOTE]
>
> - Avant de désactiver un appareil physique StorSimple ou une appliance cloud, veillez à ce que les données du conteneur de volume supprimé soient bien supprimées de l’appareil. Surveillez les graphiques de consommation du cloud, et lorsque vous constatez une baisse d’utilisation du cloud due aux sauvegardes supprimées, désactivez l’appareil. Si vous désactivez l’appareil avant de constater la baisse, les données sont bloquées dans le compte de stockage et occasionnent des frais.
>
> - Avant de désactiver un appareil physique ou une appliance cloud StorSimple, arrêtez ou supprimez les clients et les hôtes qui en dépendent.
>
> - Si le ou les comptes de stockage ou les conteneurs du compte de stockage associé aux conteneurs de volumes sont déjà supprimés avant de supprimer les données de l’appareil, vous rencontrerez une erreur et ne pourrez peut-être pas supprimer les données. Nous vous recommandons de supprimer les données sur l’appareil avant de supprimer le compte de stockage ou les conteneurs de ces données. Toutefois, dans ce cas, vous devrez procéder à la désactivation et à la suppression des appareils en supposant que les données sont déjà supprimées du compte de stockage.

Après avoir lu ce didacticiel, vous pourrez :

- Désactiver un appareil et supprimer ses données.
- Désactiver un appareil et conserver ses données.

## <a name="deactivate-and-delete-data"></a>Désactiver et supprimer des données

Si vous êtes intéressé par la suppression complète de l’appareil et que vous ne voulez pas conserver ses données, procédez comme suit :

### <a name="to-deactivate-the-device-and-delete-the-data"></a>Pour désactiver l’appareil et supprimer ses données

1. Avant de désactiver un appareil, vous devez supprimer tous les conteneurs de volumes (et les volumes) qui lui sont associés. Vous ne pouvez supprimer les conteneurs de volumes qu’après avoir supprimé les sauvegardes associées. Reportez-vous à la vue d’ensemble ci-dessus avant de désactiver un appareil physique StorSimple ou une appliance Cloud.

2. Désactivez l’appareil de la manière suivante :

   1. Accédez à votre service StorSimple Device Manager et cliquez sur **Appareils**. Dans le panneau **Appareils**, sélectionnez l’appareil à désactiver, cliquez avec le bouton droit, puis cliquez sur **Désactiver**.

        ![Désactiver l’appareil StorSimple](./media/storsimple-8000-deactivate-and-delete-device/deactivate1.png)
   2. Dans le panneau **Désactiver**, saisissez le nom de l’appareil pour confirmer l’opération, puis cliquez sur **Désactiver**. Le processus de désactivation, qui met plusieurs minutes à s’exécuter, démarre.

        ![Désactiver l'appareil StorSimple 2](./media/storsimple-8000-deactivate-and-delete-device/deactivate2.png)

3. À l’issue de la désactivation, vous pouvez supprimer complètement l’appareil. Si vous supprimez un appareil, il est retiré de la liste des appareils connectés au service. Le service ne peut alors plus gérer l’appareil supprimé. Procédez comme suit pour supprimer l’appareil :
   
   1. Accédez à votre service StorSimple Device Manager et cliquez sur **Appareils**. Dans le panneau **Appareils**, sélectionnez l’appareil désactivé à supprimer, cliquez avec le bouton droit, puis cliquez sur **Supprimer**.

        ![Désactiver l'appareil StorSimple 3](./media/storsimple-8000-deactivate-and-delete-device/deactivate5.png)
   2. Dans le panneau **Supprimer**, saisissez le nom de l’appareil pour confirmer l’opération, puis cliquez sur **Supprimer**. Le processus de suppression prend quelques minutes.

        ![Désactiver l'appareil StorSimple 4](./media/storsimple-8000-deactivate-and-delete-device/deactivate6.png)
   3. Vous êtes averti une fois la suppression terminée. La liste des appareils est également actualisée afin de refléter la suppression.

## <a name="deactivate-and-retain-data"></a>Désactiver et conserver des données

Si vous voulez supprimer un appareil tout en conservant ses données, procédez comme suit :

### <a name="to-deactivate-a-device-and-retain-the-data"></a>Pour désactiver un appareil et conserver ses données

1. Désactivez l’appareil. Tous les conteneurs de volumes et les instantanés de l’appareil sont conservés.
   
   1. Accédez à votre service StorSimple Device Manager et cliquez sur **Appareils**. Dans le panneau **Appareils**, sélectionnez l’appareil à désactiver, cliquez avec le bouton droit, puis cliquez sur **Désactiver**.

         ![Désactiver l'appareil StorSimple 5](./media/storsimple-8000-deactivate-and-delete-device/deactivate1.png)
   2. Dans le panneau **Désactiver**, saisissez le nom de l’appareil pour confirmer l’opération, puis cliquez sur **Désactiver**. Le processus de désactivation, qui met plusieurs minutes à s’exécuter, démarre.

         ![Désactiver l'appareil StorSimple 6](./media/storsimple-8000-deactivate-and-delete-device/deactivate2.png)
2. Vous pouvez maintenant basculer les conteneurs de volumes et les instantanés associés. Pour connaître les procédures, consultez [Basculement et récupération d'urgence pour votre appareil StorSimple](storsimple-8000-device-failover-disaster-recovery.md).
3. Après la désactivation et le basculement, vous pouvez supprimer complètement l’appareil. Si vous supprimez un appareil, il est retiré de la liste des appareils connectés au service. Le service ne peut alors plus gérer l’appareil supprimé. Pour supprimer l’appareil, procédez comme suit :
   
   1. Accédez à votre service StorSimple Device Manager et cliquez sur **Appareils**. Dans le panneau **Appareils**, sélectionnez l’appareil désactivé à supprimer, cliquez avec le bouton droit, puis cliquez sur **Supprimer**.

       ![Désactiver l'appareil StorSimple 7](./media/storsimple-8000-deactivate-and-delete-device/deactivate5.png)
   2. Dans le panneau **Supprimer**, saisissez le nom de l’appareil pour confirmer l’opération, puis cliquez sur **Supprimer**. Le processus de suppression prend quelques minutes.

       ![Désactiver l'appareil StorSimple 8](./media/storsimple-8000-deactivate-and-delete-device/deactivate6.png)
   3. Vous êtes averti une fois la suppression terminée. La liste des appareils est également actualisée afin de refléter la suppression.

## <a name="deactivate-and-delete-a-cloud-appliance"></a>Désactiver et supprimer une appliance cloud

Pour une instance Cloud StorSimple Appliance, la désactivation à partir du portail libère et supprime la machine virtuelle, ainsi que les ressources créées lors de son approvisionnement. Une fois l’appliance cloud désactivée, elle ne peut pas être restaurée vers son état précédent.

![Désactiver une instance StorSimple Cloud Appliance](./media/storsimple-8000-deactivate-and-delete-device/deactivate7.png)

La désactivation entraîne ce qui suit :

* L’instance StorSimple Cloud Appliance est supprimée du service.
* La machine virtuelle de l’instance StorSimple Cloud Appliance est supprimée.
* Les disques de données et de système d’exploitation créés pour StorSimple Cloud Appliance sont conservés. Si vous n’utilisez pas ces entités, vous devez les supprimer manuellement.
* Le service hébergé et le réseau virtuel créés lors de l’approvisionnement sont conservés. Si vous n’utilisez pas ces entités, vous devez les supprimer manuellement.
* Les instantanés cloud créés par l’instance StorSimple Cloud Appliance sont conservés.

Une fois l’appliance cloud désactivée, vous pouvez la supprimer de la liste des appareils. Sélectionnez l’appareil désactivé, cliquez avec le bouton droit, puis cliquez sur **Supprimer**. StorSimple vous avertit une fois l’appareil supprimé et la liste des appareils est actualisée.

## <a name="next-steps"></a>Étapes suivantes

* Pour restaurer les paramètres d’usine de l’appareil désactivé, consultez [Rétablir les paramètres d’usine de l’appareil](storsimple-8000-manage-device-controller.md#reset-the-device-to-factory-default-settings).
* Pour obtenir une assistance technique, [contactez le support technique de Microsoft](storsimple-8000-contact-microsoft-support.md).
* Pour plus d’informations sur l’utilisation du service StorSimple Device Manager, consultez [Use the StorSimple Device Manager service to administer your StorSimple device](storsimple-8000-manager-service-administration.md) (Utiliser le service StorSimple Device Manager pour gérer votre appareil StorSimple).

