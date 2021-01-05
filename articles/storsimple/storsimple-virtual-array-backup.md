---
title: Didacticiel de sauvegarde de Microsoft Azure StorSimple Virtual Array | Microsoft Azure
description: Créez des sauvegardes planifiées et manuelles pour Microsoft Azure StorSimple Virtual Array, et effectuez une récupération au niveau de l’élément pour restaurer un fichier supprimé sur votre tableau virtuel.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: e3cdcd9e-33b1-424d-82aa-b369d934067e
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 02/27/2017
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 99c7f4ab92e4cce3937a933fec97008c281a0eae
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96018288"
---
# <a name="back-up-shares-or-volumes-on-your-storsimple-virtual-array"></a>Sauvegarde de partages ou de volumes sur votre StorSimple Virtual Array

## <a name="overview"></a>Vue d’ensemble

StorSimple Virtual Array est un périphérique virtuel local de stockage cloud hybride qui peut être configuré comme un serveur de fichiers ou un serveur iSCSI. Le tableau virtuel vous permet de créer des sauvegardes planifiées et manuelles de l’ensemble des partages ou des volumes sur l’appareil. Configuré comme serveur de fichiers, il permet également la récupération au niveau de l’élément. Ce didacticiel vous explique comment créer des sauvegardes planifiées et manuelles et effectuer une récupération au niveau de l’élément pour restaurer un fichier supprimé sur votre tableau virtuel.

Ce didacticiel s’applique uniquement aux instances StorSimple Virtual Array. Pour plus d’informations sur la gamme 8000, accédez à [Create a backup for 8000 series device](./storsimple-8000-manage-backup-policies-u2.md) (Créer une sauvegarde pour un appareil de la gamme 8000).

## <a name="back-up-shares-and-volumes"></a>Sauvegarder des partages et des volumes

Les sauvegardes fournissent une protection jusqu’à une date et une heure, et optimisent la récupération tout en réduisant les délais de restauration pour les partages et les sauvegardes. Vous pouvez sauvegarder un partage ou un volume sur votre appareil StorSimple de deux manières : **planifiée** ou **manuelle**. Chacune des méthodes est abordée dans les sections suivantes.

## <a name="change-the-backup-start-time"></a>Modifier l’heure de début de la sauvegarde

> [!NOTE]
> Dans cette version, les sauvegardes planifiées sont créées à l’aide d’une stratégie par défaut qui s'exécute tous les jours à un moment précis et sauvegarde tous les partages ou volumes sur l'appareil. Il n'est pour l’instant pas possible de créer des stratégies personnalisées pour les sauvegardes planifiées.


Votre instance StorSimple Virtual Array comporte une stratégie de sauvegarde par défaut qui démarre à une heure spécifique de la journée (22h30) et sauvegarde une fois par jour tous les partages ou volumes sur l’appareil. Vous pouvez modifier l'heure à laquelle la sauvegarde démarre, mais la fréquence et la durée de rétention (qui spécifie le nombre de sauvegardes à conserver) ne peuvent pas être modifiées. Au cours de ces sauvegardes, l’ensemble de l’appareil virtuel est sauvegardé. Cela pourrait affecter les performances de l’appareil et les charges de travail déployées dessus. Par conséquent, nous vous recommandons de planifier ces sauvegardes pendant les heures creuses.

 Pour modifier l’heure de début par défaut de la sauvegarde, suivez la procédure suivante dans le [portail Azure](https://portal.azure.com/).

#### <a name="to-change-the-start-time-for-the-default-backup-policy"></a>Pour modifier l'heure de début de la stratégie de sauvegarde par défaut

1. Accédez à la page **Appareils**. La liste des appareils enregistrés avec votre service StorSimple Device Manager s’affiche. 
   
    ![accéder à la page appareils](./media/storsimple-virtual-array-backup/changebuschedule1.png)

2. Sélectionnez votre appareil, puis cliquez dessus. Le panneau **Paramètres** s’affiche. Accédez à **Gérer > Stratégies de sauvegarde**.
   
    ![sélectionner votre appareil](./media/storsimple-virtual-array-backup/changebuschedule2.png)

3. Dans le panneau **Stratégies de sauvegarde**, l’heure de début par défaut est 22:30. Vous pouvez spécifier la nouvelle heure de début de la sauvegarde quotidienne dans le fuseau horaire de l’appareil.
   
    ![accéder aux stratégies de sauvegarde](./media/storsimple-virtual-array-backup/changebuschedule5.png)

4. Cliquez sur **Enregistrer**.

### <a name="take-a-manual-backup"></a>Exécuter une sauvegarde manuelle

Outre les sauvegardes planifiées, vous pouvez à tout moment effectuer une sauvegarde manuelle (à la demande) sur les données de l’appareil.

#### <a name="to-create-a-manual-backup"></a>Création d’une sauvegarde manuelle

1. Accédez à la page **Appareils**. Sélectionnez votre appareil, puis cliquez sur **...** , tout à droite sur la ligne sélectionnée. Dans le menu contextuel, sélectionnez **Take backup** (Effectuer la sauvegarde).
   
    ![accéder à l’option d’exécution de la sauvegarde](./media/storsimple-virtual-array-backup/takebackup1m.png)

2. Dans le panneau **Take backup** (Effectuer la sauvegarde), cliquez sur **Take backup** (Effectuer la sauvegarde). Cette opération sauvegarde tous les partages sur le fichier de serveurs, ou l’ensemble des volumes sur votre serveur iSCSI. 
   
    ![démarrage de sauvegarde](./media/storsimple-virtual-array-backup/takebackup2m.png)
   
    Une sauvegarde à la demande démarre ; vous constatez qu’un travail de sauvegarde a démarré.
   
    ![démarrage de sauvegarde 2](./media/storsimple-virtual-array-backup/takebackup3m.png) 
   
    Une fois le travail terminé, vous être de nouveau averti. Le processus de sauvegarde peut démarrer.
   
    ![travail de sauvegarde créé](./media/storsimple-virtual-array-backup/takebackup4m.png)

3. Pour suivre l’avancement de vos sauvegardes et consulter les détails du travail, cliquez sur la notification. Vous accédez à **Détails du travail**.
   
     ![détails du travail de sauvegarde](./media/storsimple-virtual-array-backup/takebackup5m.png)

4. Une fois la sauvegarde terminée, accédez à **Gestion > Catalogue de sauvegarde**. Vous verrez un instantané cloud de tous les partages (ou volumes) sur votre appareil.
   
    ![Sauvegarde terminée](./media/storsimple-virtual-array-backup/takebackup19m.png) 

## <a name="view-existing-backups"></a>Afficher les sauvegardes existantes
Pour afficher les sauvegardes existantes, procédez comme suit dans le portail Azure.

#### <a name="to-view-existing-backups"></a>Pour afficher les sauvegardes existantes

1. Accédez au panneau **Appareils**. Sélectionnez votre appareil, puis cliquez dessus. Dans le panneau **Paramètres**, accédez à **Gestion > Catalogue de sauvegarde**.
   
    ![Accéder au catalogue de sauvegarde](./media/storsimple-virtual-array-backup/viewbackups1.png)
2. Spécifiez les critères suivants à utiliser pour le filtrage :
   
   - **Période** : peut être **Dernière heure**, **Dernières 24 heures**, **Derniers 7 jours**, **30 derniers jours**, **Année dernière** et **Date personnalisée**.
    
   - **Appareils** : effectuez votre sélection dans la liste des serveurs de fichiers ou des serveurs iSCSI enregistrés avec votre service de gStorSimple Device Manager.
   
   - **Initialisé** : peut être automatiquement **Planifié** (par une stratégie de sauvegarde) ou lancé **Manuellement** (par vous).
   
     ![Filtrer les sauvegardes](./media/storsimple-virtual-array-backup/viewbackups2.png)

3. Cliquez sur **Appliquer**. La liste filtrée des sauvegardes s’affiche dans le panneau **Catalogue de sauvegarde**. Uniquement 100 éléments de sauvegarde peuvent s’afficher simultanément.
   
    ![Catalogue de sauvegarde mis à jour](./media/storsimple-virtual-array-backup/viewbackups3.png)

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur la [gestion de votre StorSimple Virtual Array](storsimple-ova-web-ui-admin.md).