---
title: Gérer les enregistrements de contrôle d’accès pour StorSimple Virtual Array | Microsoft Docs
description: Décrit comment gérer les enregistrements de contrôle d’accès pour déterminer les hôtes qui peuvent se connecter à un volume sur le StorSimple Virtual Array.
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: e154bb4f-faab-4d92-a593-900c3ddc9595
ms.service: storsimple
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/27/2017
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: bad0d7adfd77dff53b1582e63a91f2cd87a9233d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85507617"
---
# <a name="use-storsimple-device-manager-to-manage-access-control-records-for-storsimple-virtual-array"></a>Utiliser StorSimple Device Manager pour gérer les enregistrements de contrôle d’accès pour StorSimple Virtual Array

## <a name="overview"></a>Vue d’ensemble

Les enregistrements de contrôle d’accès vous permettent de spécifier les hôtes qui peuvent se connecter à un volume sur le StorSimple Virtual Array (également appelé appareil virtuel local StorSimple). Les enregistrements de contrôle d’accès sont définis pour un volume spécifique et contiennent les noms complets iSCSI (IQN) des ordinateurs hôtes. Quand un hôte essaie de se connecter à un volume, l’appareil vérifie l’enregistrement de contrôle d’accès associé à ce volume pour le nom IQN et, s’il existe une correspondance, la connexion est établie. Le panneau **Enregistrements de contrôle d’accès** de la section **Configuration** de votre service Device Manager affiche tous les enregistrements de contrôle d’accès avec les noms complets iSCSI (IQN) des hôtes correspondants.

![Gestion d’enregistrements de contrôle d’accès](./media/storsimple-virtual-array-manage-acrs/ova-manage-acrs.png)

Le didacticiel décrit les tâches courantes liées aux enregistrements de contrôle d’accès :

* Obtenir le nom IQN
* Ajouter un enregistrement de contrôle d’accès
* Modifier un enregistrement de contrôle d’accès
* Supprimer un enregistrement de contrôle d’accès

> [!IMPORTANT]
> 
> * Lorsque vous attribuez un enregistrement de contrôle d’accès à un volume, veillez à ce que plusieurs hôtes non cluster n’accèdent pas simultanément au volume, sans quoi celui-ci pourrait être endommagé.
> * Lorsque vous supprimez un enregistrement de contrôle d’accès d’un volume, assurez-vous que l’hôte correspondant n’accède pas au volume, car la suppression pourrait entraîner une perturbation des opérations de lecture-écriture.


## <a name="get-the-iqn"></a>Obtenir le nom IQN

Procédez comme suit pour obtenir le nom IQN d’un hôte Windows exécutant Windows Server 2012.

[!INCLUDE [storsimple-get-iqn](../../includes/storsimple-get-iqn.md)]

## <a name="add-an-acr"></a>Ajouter un enregistrement de contrôle d’accès

Pour ajouter des enregistrements de contrôle d’accès, utilisez le panneau **Enregistrements de contrôle d’accès** de la section **Configuration** de votre service StorSimple Device Manager. En général, vous associez un enregistrement de contrôle d’accès à un volume.

Pour plus d’informations sur l’association d’un ACR à un volume, consultez la page [Ajouter un volume](storsimple-virtual-array-deploy3-iscsi-setup.md#step-3-add-a-volume).

> [!IMPORTANT]
> Lorsque vous attribuez un enregistrement de contrôle d’accès à un volume, veillez à ce que plusieurs hôtes non cluster n’accèdent pas simultanément au volume, sans quoi celui-ci pourrait être endommagé.


Effectuez les opérations suivantes pour ajouter un enregistrement de contrôle d’accès.

#### <a name="to-add-an-acr"></a>Pour ajouter un enregistrement de contrôle d’accès

1. Sur la page d’accueil du service, sélectionnez votre service, double-cliquez sur le nom du service, puis dans la section **Configuration**, cliquez sur **Enregistrements de contrôle d’accès**.
2. Dans le panneau **Enregistrements de contrôle d’accès**, cliquez sur **Ajouter**.
3. Dans le panneau **Ajouter un ACR**, procédez comme suit :
   
    1. Saisissez un **Nom** pour votre ACR.
    
    2. Sous **Nom d’initiateur iSCSI**, indiquez le nom IQN de votre hôte Windows. Pour obtenir le nom IQN de l’hôte Windows Server, procédez comme suit :
   
    3. Démarrez l’initiateur Microsoft iSCSI sur l’hôte Windows. Dans la fenêtre Propriétés de l’initiateur iSCSI, sous l’onglet **Configuration**, sélectionnez et copiez la chaîne affichée dans le champ **Nom de l’initiateur**.
    Collez cette chaîne dans le champ **IQN** du panneau **Ajouter un ACR**.
   
    6. Cliquez sur **Ajouter** pour ajouter l’enregistrement de contrôle d’accès.  
   
        ![Ajouter des enregistrements de contrôle d’accès](./media/storsimple-virtual-array-manage-acrs/ova-add-acrs.png)
4. La liste tabulaire est mise à jour pour refléter les modifications.

## <a name="edit-an-acr"></a>Modifier un enregistrement de contrôle d’accès

Pour modifier des enregistrements de contrôle d’accès, utilisez le panneau **Enregistrements de contrôle d’accès** de la section **Configuration** de votre service StorSimple Device Manager dans le portail Azure.

> [!NOTE]
> Vous ne devez pas modifier un enregistrement de contrôle d’accès en cours d’utilisation. Pour modifier un enregistrement de contrôle d’accès associé à un volume en cours d’utilisation, vous devez d’abord placer le volume hors connexion.


Effectuez les opérations suivantes pour modifier un enregistrement de contrôle d’accès.

#### <a name="to-edit-an-acr"></a>Pour modifier un enregistrement de contrôle d’accès

1. Sur la page d’accueil du service, sélectionnez votre service, double-cliquez sur le nom du service, puis dans la section **Configuration**, cliquez sur **Enregistrements de contrôle d’accès**.
2. Dans la liste tabulaire des enregistrements de contrôle d’accès du panneau **Enregistrements de contrôle d’accès**, double-cliquez sur l’enregistrement que vous souhaitez modifier.
3. Dans le panneau **Enregistrements de contrôle d’accès**, procédez comme suit :
   
    1. Indiquez le nom qualifié (IQN) de l’enregistrement de contrôle d’accès.
   
    2. Cliquez sur **Enregistrer** en haut du panneau pour enregistrer l’enregistrement de contrôle d’accès modifié. Le message de confirmation suivant s’affiche :
   
        ![Modifier des enregistrements de contrôle d’accès](./media/storsimple-virtual-array-manage-acrs/ova-edit-acrs.png)

## <a name="delete-an-access-control-record"></a>Supprimer un enregistrement de contrôle d’accès

Pour supprimer des enregistrements de contrôle d’accès, utilisez la page **Configuration** du portail Azure.

> [!NOTE]
> 
> * Vous ne devez pas supprimer un enregistrement de contrôle d’accès en cours d’utilisation. Pour supprimer un enregistrement de contrôle d’accès associé à un volume en cours d’utilisation, vous devez d’abord placer le volume hors connexion.
> * Lorsque vous supprimez un enregistrement de contrôle d’accès d’un volume, assurez-vous que l’hôte correspondant n’accède pas au volume, car la suppression pourrait entraîner une perturbation des opérations de lecture-écriture.


Pour supprimer un enregistrement de contrôle d’accès, procédez comme suit.

#### <a name="to-delete-an-access-control-record"></a>Pour supprimer un enregistrement de contrôle d’accès

1. Sur la page d’accueil du service, sélectionnez votre service, double-cliquez sur le nom du service, puis dans la section **Configuration**, cliquez sur **Enregistrements de contrôle d’accès**.

2. Dans la liste tabulaire des enregistrements de contrôle d’accès du panneau **Enregistrements de contrôle d’accès**, double-cliquez sur l’enregistrement que vous souhaitez supprimer.

3. Dans le panneau Enregistrements de contrôle d’accès, cliquez sur **Supprimer**.
   
    ![Supprimer des enregistrements de contrôle d’accès](./media/storsimple-virtual-array-manage-acrs/ova-del-acrs.png)

4. Lorsque vous êtes invité à confirmer la suppression, cliquez sur **Supprimer**. La liste tabulaire est mise à jour pour refléter la suppression.
   
   ![Message d'avertissement](./media/storsimple-virtual-array-manage-acrs/ova-del-acrs-warning.png)

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [ajouter des volumes et configurer les enregistrements de contrôle d’accès](storsimple-virtual-array-deploy3-iscsi-setup.md#step-3-add-a-volume).

