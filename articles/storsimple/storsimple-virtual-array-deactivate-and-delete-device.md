---
title: Désactiver et supprimer une instance Microsoft Azure StorSimple Virtual Array | Microsoft Docs
description: Explique comment supprimer un appareil StorSimple du service en le désactivant dans un premier temps, puis en le supprimant.
services: storsimple
documentationcenter: ''
author: alkohli
manager: carmonm
editor: ''
ms.assetid: a929f5bc-03e2-4b01-b925-973db236f19f
ms.service: storsimple
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/21/2016
ms.author: alkohli
ms.openlocfilehash: 84fd1a2d3d0dc4ce6960469ea3212b8ca1d4d07c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96023021"
---
# <a name="deactivate-and-delete-a-storsimple-virtual-array"></a>Désactiver et supprimer un StorSimple Virtual Array

## <a name="overview"></a>Vue d’ensemble

Quand vous désactivez une instance StorSimple Virtual Array, vous interrompez la connexion entre l’appareil et le service StorSimple Device Manager correspondant. Ce didacticiel explique comment :

* Désactiver un appareil 
* Supprimer un appareil désactivé

Les informations publiées dans cet article s'appliquent uniquement aux instances StorSimple Virtual Array. Pour plus d’informations sur la série 8000, accédez la procédure relative à la [désactivation ou la suppression d’un appareil](./storsimple-8000-deactivate-and-delete-device.md).

## <a name="when-to-deactivate"></a>Quand désactiver ?

La désactivation est une opération DÉFINITIVE et ne peut pas être annulée. Un appareil désactivé avec le service  StorSimple Device Manager ne peut pas être de nouveau enregistré. Vous pouvez être amené à désactiver et à supprimer une instance StorSimple Virtual Array dans les scénarios suivants :

* **Basculement planifié** : Votre appareil est en ligne et vous envisagez de le basculer. Si vous envisagez de mettre à niveau vers un appareil plus grand, il vous faudra peut-être basculer votre appareil. Une fois la propriété des données transférée et le basculement exécuté, l’appareil source est automatiquement supprimé.
* **Basculement non planifié** : Votre appareil est hors ligne, vous devez le basculer. Ce scénario peut se produire durant un incident, lorsqu’une panne est identifiée dans le centre de données et que votre appareil principal est arrêté. Vous envisagez de basculer l’appareil vers un appareil secondaire. Une fois la propriété des données transférée et le basculement exécuté, l’appareil source est automatiquement supprimé.
* **Désactivation** : Vous souhaitez désactiver l’appareil. Vous devez dans un premier temps désactiver l’appareil, puis le supprimer. Si vous désactivez un appareil, il n’est plus possible d’accéder aux données stockées en local. Vous pouvez seulement accéder aux données stockées dans le cloud, et les récupérer. Si vous envisagez de conserver les données de l’appareil après la désactivation, vous devez préalablement prendre un instantané de cloud de toutes vos données. Cet instantané cloud vous permet de récupérer ultérieurement l’ensemble des données.

## <a name="deactivate-a-device"></a>Désactiver un appareil

Suivez la procédure suivante pour désactiver votre appareil.

#### <a name="to-deactivate-the-device"></a>Pour désactiver l’appareil

1. Dans votre service, accédez à **Gestion > Appareils**. Dans le panneau **Appareils**, cliquez sur (sélectionnez) l’appareil que vous souhaitez désactiver.
   
    ![Sélection de l’appareil à désactiver](./media/storsimple-virtual-array-deactivate-and-delete-device/deactivate-delete7.png)
2. Dans votre panneau **Device dashboard** (Tableau de bord de l’appareil), cliquez sur **… Plus**, puis sélectionnez **Désactiver** dans la liste.
   
    ![Clic sur Désactiver](./media/storsimple-virtual-array-deactivate-and-delete-device/deactivate-delete8.png)
3. Dans le panneau **Désactiver**, saisissez le nom de l’appareil, puis cliquez sur **Désactiver**. 
   
    ![Confirmation de la désactivation](./media/storsimple-virtual-array-deactivate-and-delete-device/deactivate-delete1.png)
   
    Le processus de désactivation, qui met plusieurs minutes à s’exécuter, démarre.
   
    ![Désactivation en cours](./media/storsimple-virtual-array-deactivate-and-delete-device/deactivate-delete2.png)
4. Après la désactivation, la liste des appareils est actualisée.
   
    ![Désactivation terminée](./media/storsimple-virtual-array-deactivate-and-delete-device/deactivate-delete3.png)
   
    Vous pouvez maintenant supprimer cet appareil.

## <a name="delete-the-device"></a>Supprimer l’appareil

Un appareil doit être désactivé avant d’être supprimé. Si vous supprimez un appareil, il est retiré de la liste des appareils connectés au service. Le service ne peut alors plus gérer l’appareil supprimé. Toutefois, les données associées à l’appareil demeurent dans le cloud. Ensuite, les frais associés aux données s’accumulent.

Pour supprimer l’appareil, procédez comme suit.

#### <a name="to-delete-the-device"></a>Pour supprimer l’appareil

1. Dans votre instance StorSimple Device Manager, accédez à **Gestion > Appareils**. Dans le panneau **Appareils** sélectionnez un appareil désactivé que vous souhaitez supprimer.
2. Dans le panneau **Device dashboard** (Tableau de bord de l’appareil), cliquez sur **… Plus**, puis sur **Supprimer**.
   
   ![Sélection de l’appareil à supprimer](./media/storsimple-virtual-array-deactivate-and-delete-device/deactivate-delete4.png)
3. Dans le panneau **Supprimer**, saisissez le nom de votre appareil pour confirmer la suppression, puis cliquez sur **Supprimer**. La suppression de l’appareil ne supprime pas les données de cloud associées à celui-ci. 
   
   ![Confirmation de suppression](./media/storsimple-virtual-array-deactivate-and-delete-device/deactivate-delete5.png) 
4. Le processus de suppression, qui met plusieurs minutes à s’exécuter, démarre.
   
   ![Suppression en cours](./media/storsimple-virtual-array-deactivate-and-delete-device/deactivate-delete6.png)
   
    Une fois que l’appareil est supprimé, la liste mise à jour des appareils s’affiche.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur la procédure de basculement, accédez à la page [Failover and disaster recovery of your StorSimple Virtual Array](storsimple-virtual-array-failover-dr.md) (Basculement et récupération d’urgence de votre instance StorSimple Virtual Array).

* Pour plus d’informations sur l’utilisation du service StorSimple Device Manager, accédez à [Utiliser le service StorSimple Device Manager pour gérer votre StorSimple Virtual Array](storsimple-virtual-array-manager-service-administration.md).