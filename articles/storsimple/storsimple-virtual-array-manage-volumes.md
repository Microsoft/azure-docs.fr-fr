---
title: Gérer des volumes sur StorSimple Virtual Array | Microsoft Docs
description: Décrit le service StorSimple Device Manager et vous explique comment l’utiliser pour gérer des volumes sur votre instance StorSimple Virtual Array.
services: storsimple
documentationcenter: ''
author: manuaery
manager: syadav
editor: ''
ms.assetid: caa6a26b-b7ba-4a05-b092-1a79450225cf
ms.service: storsimple
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/21/2016
ms.author: manuaery
ms.openlocfilehash: 79e3ce8c1605e5d68ff44901f53854d2f5f10abc
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96014378"
---
# <a name="use-storsimple-device-manager-service-to-manage-volumes-on-the-storsimple-virtual-array"></a>Utiliser le service StorSimple Device Manager pour gérer les volumes sur l’instance StorSimple Virtual Array

## <a name="overview"></a>Vue d’ensemble

Ce didacticiel explique comment utiliser le service StorSimple Device Manager pour créer et gérer des volumes sur votre instance StorSimple Virtual Array.

Le service StorSimple Device Manager est une extension dans le portail Azure qui vous permet de gérer votre solution StorSimple à partir d’une seule interface web. Outre la gestion des partages et des volumes, le service StorSimple Device Manager prend en charge l’affichage et la gestion des appareils, l’affichage des alertes,l’affichage et la gestion des stratégies de sauvegardes et du catalogue de sauvegarde.

## <a name="volume-types"></a>Types de volume

Les volumes StorSimple peuvent être les suivants :

* **Épinglé localement** : les données de ces volumes demeurent à tout moment sur le tableau, sans jamais déborder dans le cloud.
* **Hiérarchisé** : les données de ces volumes peuvent se disperser dans le cloud. Lorsque vous créez un volume à plusieurs niveaux, environ 10 % de l’espace est configuré au niveau local et 90 % dans le cloud. Par exemple, si vous avez configuré un volume de 1 To, 100 Go résident dans l'espace local et 900 Go sont utilisés dans le cloud lorsque les données sont stockées en niveaux. Cela implique que si vous n’avez plus d’espace local sur l’appareil, vous ne pouvez pas configurer un volume à plusieurs niveaux (car les 10 % requis sur l’espace local ne seront pas disponibles).

### <a name="provisioned-capacity"></a>Capacité allouée
Reportez-vous au tableau suivant pour connaître la capacité maximale allouée pour chaque type de volume.

| **Identificateur de la limite**                                       | **Limite**     |
|------------------------------------------------------------|---------------|
| Taille minimale d’un volume hiérarchisé                            | 500 Go        |
| Taille maximale d’un volume hiérarchisé                            | 5 To          |
| Taille minimale d'un volume épinglé localement                    | 50 Go         |
| Taille maximale d'un volume épinglé localement                    | 200 Go        |

## <a name="the-volumes-blade"></a>Le panneau Volumes
Sur le panneau de synthèse du service StorSimple, le menu **Volumes** affiche la liste des volumes de stockage sur un tableau StorSimple considéré et vous donne les moyens de les gérer.

![Panneau Volumes](./media/storsimple-virtual-array-manage-volumes/volumes-blade.png)

Un volume est constitué d’une série d’attributs :

* **Nom du volume** : nom descriptif qui doit être unique et vous aide à identifier le volume.
* **État** : peut être en ligne ou hors connexion. Si un volume est hors connexion, il n’est pas visible pour les initiateurs (serveurs) qui sont autorisés à l’utiliser.
* **Type** : indique si le volume est **Hiérarchisé** (par défaut) ou **Épinglé localement**.
* **Capacité** : spécifie la quantité de données utilisées par rapport au volume total des données pouvant être stockées par l’initiateur (serveur).
* **Sauvegarde** : dans le cas d’une instance StorSimple Virtual Array, l’ensemble des volumes sont automatiquement activés pour la sauvegarde.
* **Hôtes connectés** : indique les initiateurs (serveurs) autorisés à accéder à ce volume.

![Détails de volumes](./media/storsimple-virtual-array-manage-volumes/volume-details.png)

Suivez les instructions de ce didacticiel pour effectuer les tâches suivantes :

* Ajout d’un volume
* Modification d’un volume
* Mise hors connexion d’un volume
* Suppression d’un volume

## <a name="add-a-volume"></a>Ajout d’un volume

1. À partir du panneau de synthèse du service StorSimple, cliquez sur **+ Ajouter un volume** dans la barre de commandes. Le panneau **Ajouter un volume** s’ouvre.
   
    ![Capture d’écran montrant le bouton Ajouter un volume et le volet Ajouter un volume.](./media/storsimple-virtual-array-manage-volumes/add-volume.png)
2. Dans le panneau **Ajouter un volume**, procédez comme suit :
   
   * Dans le champ **Nom du volume**, saisissez un nom unique pour votre volume. Le nom doit être une chaîne contenant entre 3 et 127 caractères.
   * Dans la liste déroulante **Type**, spécifiez si vous souhaitez créer un volume **Hiérarchisé** ou **Attaché localement**. Pour les charges de travail qui nécessitent des garanties locales, une faible latence et les meilleures performances possibles, sélectionnez **Volume attaché localement**. Pour toutes les autres données, sélectionnez le volume **hiérarchisé**.
   * Dans le champ **Capacité**, spécifiez la taille du volume. Un volume hiérarchisé doit être compris entre 500 Go et 5 To, tandis qu’un volume attaché doit être compris entre 50 Go et 500 Go.
   * * Cliquez sur **Hôtes connectés**, sélectionnez un enregistrement de contrôle d’accès correspondant à l’initiateur iSCSI que vous souhaitez connecter à ce volume, puis cliquez sur **Sélectionner**.
3. Pour ajouter un hôte connecté, cliquez sur **Ajouter nouveau**, saisissez un nom pour l’hôte et son nom complet iSCSI, puis cliquez sur **Ajouter**.
   
    ![Capture d’écran montrant le volet Hôtes connectés dans lequel vous pouvez ajouter de nouveaux hôtes.](./media/storsimple-virtual-array-manage-volumes/volume-add-acr.png)
4. Lorsque vous avez terminé de configurer votre volume, cliquez sur **Créer**. Un volume est créé avec les paramètres spécifiés ; une notification s’affiche en cas de création similaire. Par défaut, la sauvegarde est activée pour le volume.
5. Pour vérifier que le volume a été créé, accédez au panneau **Volumes**. Le volume devrait y apparaître.
   
    ![Le volume stimule la réussite](./media/storsimple-virtual-array-manage-volumes/volume-success.png)

## <a name="modify-a-volume"></a>Modification d’un volume

Modifiez un volume lorsque vous avez besoin de modifier les hôtes qui y accèdent. Une fois le volume créé, il est impossible de modifier les autres attributs d’un volume.

#### <a name="to-modify-a-volume"></a>Pour modifier un volume

1. À partir du paramètre **Volumes** du panneau de synthèse du service StorSimple, sélectionnez le tableau virtuel sur lequel est hébergé le volume à modifier.
2. **Sélectionnez** le volume, puis cliquez sur **Hôtes connectés** afin d’afficher l’hôte actuellement connecté et de l’affecter à un serveur différent.
   
    ![Modifier le volume](./media/storsimple-virtual-array-manage-volumes/volume-edit-acr.png)
3. Pour enregistrer les modifications, cliquez sur la barre de commandes **Enregistrer**. Vos paramètres spécifiés sont appliqués ; une notification s’affiche.

## <a name="take-a-volume-offline"></a>Mise hors connexion d’un volume

Vous devrez peut-être mettre un volume hors connexion si vous envisagez de le modifier ou de le supprimer. Lorsqu’un volume est hors connexion, il n’est pas disponible pour l’accès en lecture-écriture. Vous devrez mettre le volume hors connexion à la fois sur l’ordinateur hôte et sur l’appareil.

#### <a name="to-take-a-volume-offline"></a>Pour mettre un volume hors connexion

1. Assurez-vous que le volume en question n’est pas utilisé avant de le mettre hors connexion.
2. Mettez d’abord le volume hors connexion sur l’ordinateur hôte. Cela élimine tout risque d’endommagement des données sur le volume. Pour les instructions spécifiques, reportez-vous aux instructions du système d’exploitation de l’ordinateur hôte.
3. Une fois le volume de l’hôte est hors connexion, mettez le volume du tableau hors connexion en procédant comme suit :
   
   * À partir du paramètre **Volumes** du panneau de synthèse du service StorSimple, sélectionnez le tableau virtuel sur lequel est hébergé le volume que vous souhaitez déplacer hors connexion.
   * **Sélectionnez** le volume, puis cliquez sur **...** (sinon cliquez avec le bouton droit sur cette ligne), puis dans le menu contextuel, sélectionnez **Mettre hors connexion**.
     
        ![Volume hors connexion](./media/storsimple-virtual-array-manage-volumes/volume-offline.png)
   * Passez en revue les informations du panneau **Mettre hors connexion**, puis confirmez votre acceptation de l’opération. Cliquez sur **Mettre hors connexion** afin de mettre le volume hors connexion. Une notification de l’opération en cours s’affiche.
   * Pour vérifier que le volume a été mis hors connexion, accédez au panneau **Volumes**. Le volume doit apparaître comme élément hors connexion.
     
       ![Confirmation du volume hors connexion](./media/storsimple-virtual-array-manage-volumes/volume-offline-confirm.png)

## <a name="delete-a-volume"></a>Suppression d’un volume

> [!IMPORTANT]
> Vous pouvez supprimer un volume uniquement s’il est hors connexion.
> 
> 

Pour supprimer un volume, procédez comme indiqué ci-dessous.

#### <a name="to-delete-a-volume"></a>Pour supprimer un volume

1. À partir du paramètre **Volumes** du panneau de synthèse du service StorSimple, sélectionnez le tableau virtuel sur lequel est hébergé le volume à supprimer.
2. **Sélectionnez** le volume, puis cliquez sur **...** (sinon cliquez avec le bouton droit sur cette ligne), puis dans le menu contextuel, sélectionnez **Supprimer**.
   
    ![Supprimer un volume](./media/storsimple-virtual-array-manage-volumes/volume-delete.png)
3. Vérifiez l’état du volume à supprimer. Si le volume que vous souhaitez supprimer n’est pas hors connexion, mettez-le d’abord hors connexion, en suivant la procédure [Mise hors connexion d’un volume](#take-a-volume-offline).
4. Lorsque vous êtes invité à confirmer l’opération dans le panneau **Supprimer**, validez la confirmation, puis cliquez sur **Supprimer**. Le volume est supprimé, et le panneau **Volumes** représente la liste mise à jour des volumes au sein du tableau virtuel.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [cloner un volume StorSimple](storsimple-virtual-array-clone.md).

