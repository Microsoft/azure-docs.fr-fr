---
title: Gérer les enregistrements de contrôle d’accès dans StorSimple | Microsoft Docs
description: Décrit comment utiliser les enregistrements de contrôle d’accès pour déterminer les hôtes qui peuvent se connecter à un volume sur l’appareil StorSimple.
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
ms.date: 05/31/2017
ms.author: alkohli
ms.openlocfilehash: 15c35fb314af27b1ced129a12f752d0a2794e0f8
ms.sourcegitcommit: a2d8acc1b0bf4fba90bfed9241b299dc35753ee6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/12/2020
ms.locfileid: "91949902"
---
# <a name="use-the-storsimple-manager-service-to-manage-access-control-records"></a>Utiliser le service StorSimple Manager pour gérer les enregistrements de contrôle d’accès

## <a name="overview"></a>Vue d’ensemble
Les enregistrements de contrôle d’accès vous permettent de spécifier les hôtes qui peuvent se connecter à un volume sur l’appareil StorSimple. Les enregistrements de contrôle d’accès sont définis pour un volume spécifique et contiennent les noms complets iSCSI (IQN) des ordinateurs hôtes. Lorsqu’un hôte essaie de se connecter à un volume, l’appareil vérifie l’enregistrement de contrôle d’accès associé à ce volume pour le nom complet iSCSI (IQN) et s’il existe une correspondance, la connexion est établie. Les enregistrements de contrôle d’accès dans la section **Configuration** du panneau de votre service StorSimple Device Manager affichent tous les enregistrements de contrôle d’accès avec les noms complets iSCSI (IQN) des hôtes correspondants.

Le didacticiel décrit les tâches courantes liées aux enregistrements de contrôle d’accès :

* Ajouter un enregistrement de contrôle d’accès
* Modifier un enregistrement de contrôle d’accès
* Supprimer un enregistrement de contrôle d’accès

> [!IMPORTANT]
> * Lorsque vous attribuez un enregistrement de contrôle d’accès à un volume, veillez à ce que plusieurs hôtes non cluster n’accèdent pas simultanément au volume, sans quoi celui-ci pourrait être endommagé.
> * Lorsque vous supprimez un enregistrement de contrôle d’accès d’un volume, assurez-vous que l’hôte correspondant n’accède pas au volume, car la suppression pourrait entraîner une perturbation des opérations de lecture-écriture.

## <a name="get-the-iqn"></a>Obtenir le nom IQN

Procédez comme suit pour obtenir le nom IQN d’un hôte Windows exécutant Windows Server 2012.

[!INCLUDE [storsimple-get-iqn](../../includes/storsimple-get-iqn.md)]


## <a name="add-an-access-control-record"></a>Ajouter un enregistrement de contrôle d’accès
Vous utilisez la section **Configuration** dans le panneau du service StorSimple Device Manager pour ajouter des enregistrements de contrôle d’accès (ACR). En général, vous associez un enregistrement de contrôle d’accès à un volume.

Effectuez les opérations suivantes pour ajouter un enregistrement de contrôle d’accès.

#### <a name="to-add-an-acr"></a>Pour ajouter un enregistrement de contrôle d’accès

1. Accédez à votre service StorSimple Device Manager, double-cliquez sur le nom du service puis, dans la section **Configuration**, cliquez sur **Enregistrements de contrôle d’accès**.
2. Dans le panneau **Enregistrements de contrôle d’accès**, cliquez sur **+ Ajouter un ACR**.

    ![Cliquer sur Ajouter un ACR](./media/storsimple-8000-manage-acrs/createacr1.png)

3. Dans le panneau **Ajouter un ACR**, effectuez les étapes suivantes :

    1. Saisissez un nom pour votre ACR.
    
    2. Sous **Nom d’initiateur iSCSI (IQN)** , indiquez le nom IQN de votre hôte Windows Server.

    3. Cliquez sur **Ajouter** pour créer l’ACR.

        ![Cliquer sur Ajouter un ACR 2](./media/storsimple-8000-manage-acrs/createacr2.png)

4.  L’ACR nouvellement ajouté s’affiche dans la liste tabulaire d’ACR.

    ![Cliquer sur Ajouter un ACR 3](./media/storsimple-8000-manage-acrs/createacr5.png)


## <a name="edit-an-access-control-record"></a>Modifier un enregistrement de contrôle d’accès
Vous utilisez la section **Configuration** dans le panneau du service StorSimple Device Manager pour modifier des enregistrements de contrôle d’accès (ACR).

> [!NOTE]
> Il vous est recommandé de modifier uniquement les enregistrements de contrôle d’accès qui ne sont pas en cours d’utilisation. Pour modifier un enregistrement de contrôle d’accès associé à un volume en cours d’utilisation, vous devez d’abord placer le volume hors connexion.

Effectuez les opérations suivantes pour modifier un enregistrement de contrôle d’accès.

#### <a name="to-edit-an-access-control-record"></a>Pour modifier un enregistrement de contrôle d’accès
1.  Accédez à votre service StorSimple Device Manager, double-cliquez sur le nom du service puis, dans la section **Configuration**, cliquez sur **Enregistrements de contrôle d’accès**.

    ![Accéder aux enregistrements de contrôle d’accès](./media/storsimple-8000-manage-acrs/createacr1.png)

2. Dans la liste tabulaire des enregistrements de contrôle d’accès, cliquez et sélectionnez l’enregistrement que vous souhaitez modifier.

    ![Modifier des enregistrements de contrôle d’accès](./media/storsimple-8000-manage-acrs/editacr1.png)

3. Dans le panneau **Modifier l’enregistrement de contrôle d’accès**, indiquez un autre IQN correspondant à un autre hôte.

    ![Modifier des enregistrements de contrôle d’accès 2](./media/storsimple-8000-manage-acrs/editacr2.png)

4. Cliquez sur **Enregistrer**. Cliquez sur **Oui**lorsque vous êtes invité à confirmer l’opération. 

    ![Modifier des enregistrements de contrôle d’accès 3](./media/storsimple-8000-manage-acrs/editacr3.png)

5. Vous êtes informé lorsque l’ACR est mis à jour. La liste tabulaire est également mise à jour pour refléter la modification.

   
## <a name="delete-an-access-control-record"></a>Supprimer un enregistrement de contrôle d’accès
Vous utilisez la section **Configuration** dans le panneau du service StorSimple Device Manager pour supprimer des enregistrements de contrôle d’accès (ACR).

> [!NOTE]
> Vous pouvez uniquement supprimer les enregistrements de contrôle d’accès qui ne sont pas en cours d’utilisation. Pour supprimer un enregistrement de contrôle d’accès associé à un volume en cours d’utilisation, vous devez d’abord placer le volume hors connexion.

Pour supprimer un enregistrement de contrôle d’accès, procédez comme suit.

#### <a name="to-delete-an-access-control-record"></a>Pour supprimer un enregistrement de contrôle d’accès
1.  Accédez à votre service StorSimple Device Manager, double-cliquez sur le nom du service puis, dans la section **Configuration**, cliquez sur **Enregistrements de contrôle d’accès**.

    ![Accéder aux enregistrements de contrôle d’accès 1b](./media/storsimple-8000-manage-acrs/createacr1.png)

2. Dans la liste tabulaire des enregistrements de contrôle d’accès, cliquez et sélectionnez l’enregistrement que vous souhaitez supprimer.

    ![Accéder aux enregistrements de contrôle d’accès 2](./media/storsimple-8000-manage-acrs/deleteacr1.png)

3. Cliquez avec le bouton droit pour appeler le menu contextuel, puis sélectionnez **Supprimer**.

    ![Accéder aux enregistrements de contrôle d’accès 3](./media/storsimple-8000-manage-acrs/deleteacr2.png)

4. Lorsque vous êtes invité à confirmer l’opération, passez en revue les informations, puis cliquez sur **Supprimer**.

    ![Accéder aux enregistrements de contrôle d’accès 4](./media/storsimple-8000-manage-acrs/deleteacr3.png)

5. Vous êtes informé lorsque la suppression est terminée. La liste tabulaire est mise à jour pour refléter la suppression.

    ![Accéder aux enregistrements de contrôle d’accès 5](./media/storsimple-8000-manage-acrs/deleteacr5.png)

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur la [gestion des volumes StorSimple](storsimple-8000-manage-volumes-u2.md).
* En savoir plus sur [l’utilisation du service StorSimple Manager pour gérer votre appareil StorSimple](storsimple-8000-manager-service-administration.md).

