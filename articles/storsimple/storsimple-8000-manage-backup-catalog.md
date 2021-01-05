---
title: Gérer votre catalogue de sauvegarde StorSimple | Microsoft Docs
description: Explique comment utiliser la page Catalogue de sauvegarde du service StorSimple Device Manager pour répertorier, sélectionner et supprimer des jeux de sauvegarde.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 06/29/2017
ms.author: alkohli
ms.openlocfilehash: 6ab4694159c6269f2f72d3d591147d43aef34a0a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017370"
---
# <a name="use-the-storsimple-device-manager-service-to-manage-your-backup-catalog"></a>Utiliser le service StorSimple Device Manager pour gérer votre catalogue de sauvegarde
## <a name="overview"></a>Vue d’ensemble
Le panneau **Catalogue de sauvegarde** du service StorSimple Device Manager affiche tous les jeux de sauvegarde créés lors de sauvegardes manuelles ou planifiées. Vous pouvez utiliser cette page pour répertorier toutes les sauvegardes pour une stratégie de sauvegarde ou un volume, sélectionner ou supprimer des sauvegardes, ou utiliser une sauvegarde pour restaurer ou cloner un volume.

Ce didacticiel explique comment répertorier, sélectionner et supprimer un jeu de sauvegarde. Pour savoir comment restaurer votre appareil à partir d’une sauvegarde, accédez à [Restaurer l’appareil à partir d’un jeu de sauvegarde](storsimple-8000-restore-from-backup-set-u2.md). Pour découvrir comment cloner un volume, accédez à [Cloner un volume StorSimple](storsimple-8000-clone-volume-u2.md).

![Catalogue de sauvegarde](./media/storsimple-8000-manage-backup-catalog/bucatalog.png) 

Le panneau **Catalogue de sauvegarde** comprend une zone de requête pour affiner la sélection des jeux de sauvegarde. Vous pouvez filtrer les jeux de sauvegarde récupérés selon les paramètres suivants :

* **Appareil** : appareil sur lequel le jeu de sauvegarde a été créé.
* **Stratégie de sauvegarde ou volume** : stratégie de sauvegarde ou volume associé à ce jeu de sauvegarde.
* **De et À** : plage de dates et d’heures de création du jeu de sauvegarde.

Les jeux de sauvegarde filtrés sont ensuite affichés sous forme de tableau sur la base des attributs suivants :

* **Nom** : nom de la stratégie de sauvegarde ou du volume associé à ce jeu de sauvegarde.
* **Taille** : taille réelle du jeu de sauvegarde.
* **Créé le** : date et heure auxquelles les sauvegardes ont été créées. 
* **Type** : les jeux de sauvegarde peuvent être des instantanés locaux ou des instantanés cloud. Un instantané local est une sauvegarde de toutes les données de volume stockées localement sur l’appareil, tandis qu’un instantané cloud correspond à la sauvegarde des données de volume résidant dans le cloud. Les instantanés locaux offrent un accès plus rapide, alors que les instantanés cloud sont choisis pour la résilience des données.
* **Initié par** : les sauvegardes peuvent être lancées automatiquement par une planification ou manuellement par un utilisateur. Vous pouvez utiliser une stratégie de sauvegarde pour planifier des sauvegardes. Vous pouvez également utiliser l'option **Effectuer une sauvegarde** pour effectuer une sauvegarde manuelle.

## <a name="list-backup-sets-for-a-backup-policy"></a>Répertorier les jeux de sauvegarde pour une stratégie de sauvegarde
Procédez comme suit pour répertorier toutes les sauvegardes pour une stratégie de sauvegarde.

#### <a name="to-list-backup-sets"></a>Pour répertorier les jeux de sauvegarde
1. Accédez à votre service StorSimple Device Manager et cliquez sur **Catalogue de sauvegarde**.

2. Filtrez les sélections comme suit :
   
   1. Indiquez l’intervalle de temps.
   2. Sélectionnez l’appareil approprié.
   3. Filtrez par **Stratégie de sauvegarde** pour afficher les sauvegardes correspondantes.
   3. Dans la liste déroulante de la stratégie de sauvegarde, choisissez **Tout** pour afficher toutes les sauvegardes sur l’appareil sélectionné.
   4. Cliquez sur **Appliquer** pour exécuter cette requête.
      
      Les sauvegardes associées à la stratégie de sauvegarde sélectionnée doivent figurer dans la liste des jeux de sauvegarde.

      ![Accéder au catalogue de sauvegarde](./media/storsimple-8000-manage-backup-catalog/bucatalog1.png)

## <a name="select-a-backup-set"></a>Sélectionner un jeu de sauvegarde
Procédez comme suit pour sélectionner un jeu de sauvegarde pour un volume ou une stratégie de sauvegarde.

#### <a name="to-select-a-backup-set"></a>Pour sélectionner un jeu de sauvegarde
1. Accédez à votre service StorSimple Device Manager et cliquez sur **Catalogue de sauvegarde**.
2. Filtrez les sélections comme suit :
   
   1. Indiquez l’intervalle de temps. 
   2. Sélectionnez l’appareil approprié. 
   3. Filtrez par volume ou stratégie de sauvegarde pour la sauvegarde à sélectionner.
   4. Cliquez sur **Appliquer** pour exécuter cette requête.
      
      Les sauvegardes associées au volume ou à la stratégie de sauvegarde sélectionné doivent figurer dans la liste des jeux de sauvegarde.

      ![Accéder au catalogue de sauvegarde 2](./media/storsimple-8000-manage-backup-catalog/bucatalog1.png)

3. Sélectionnez un jeu de sauvegarde et développez-le. Vous pouvez maintenant voir les jeux de sauvegarde répartis par les volumes qu’ils contiennent. Les options **Restaurer** et **Supprimer** sont disponibles via le menu contextuel (clic droit) du jeu de sauvegarde. Vous pouvez effectuer une de ces actions sur le jeu de sauvegarde que vous avez sélectionné.

    ![Accéder au catalogue de sauvegarde 3](./media/storsimple-8000-manage-backup-catalog/bucatalog2.png)

## <a name="delete-a-backup-set"></a>Supprimer un jeu de sauvegarde
Supprimez une sauvegarde quand vous ne souhaitez plus conserver les données qui lui sont associées. Procédez comme suit pour supprimer un jeu de sauvegarde.

#### <a name="to-delete-a-backup-set"></a>Pour supprimer un jeu de sauvegarde
 Accédez à votre service StorSimple Device Manager et cliquez sur **Catalogue de sauvegarde**.
1. Filtrez les sélections comme suit :
   
   1. Indiquez l’intervalle de temps. 
   2. Sélectionnez l’appareil approprié. 
   3. Filtrez par volume ou stratégie de sauvegarde pour la sauvegarde à sélectionner.
   4. Cliquez sur **Appliquer** pour exécuter cette requête.
      
      Les sauvegardes associées au volume ou à la stratégie de sauvegarde sélectionné doivent figurer dans la liste des jeux de sauvegarde.

      ![Accéder au catalogue de sauvegarde 4](./media/storsimple-8000-manage-backup-catalog/bucatalog1.png)

1. Sélectionnez un jeu de sauvegarde et développez-le. Vous pouvez maintenant voir les jeux de sauvegarde répartis par les volumes qu’ils contiennent. Les options **Restaurer** et **Supprimer** sont disponibles via le menu contextuel (clic droit) du jeu de sauvegarde. Cliquez avec le bouton droit sur le jeu de sauvegarde sélectionné et sélectionnez **Supprimer** dans le menu contextuel.

    ![Accéder au catalogue de sauvegarde 5](./media/storsimple-8000-manage-backup-catalog/bucatalog3.png)

1. Lorsque vous êtes invité à confirmer l’opération, passez en revue les informations affichées et cliquez sur **Supprimer**. La sauvegarde sélectionnée est supprimée définitivement.

    ![Accéder au catalogue de sauvegarde 6](./media/storsimple-8000-manage-backup-catalog/bucatalog4.png)  

1. Vous serez informé de la progression de la suppression et de son issue. Une fois la suppression terminée, actualisez la requête dans cette page. Le jeu de sauvegarde supprimé n’apparaîtra plus dans la liste des jeux de sauvegarde.

    ![Accéder au catalogue de sauvegarde 7](./media/storsimple-8000-manage-backup-catalog/bucatalog7.png)

## <a name="next-steps"></a>Étapes suivantes
* Découvrez comment [utiliser le catalogue de sauvegarde pour restaurer l’appareil à partir d’un jeu de sauvegarde](storsimple-8000-restore-from-backup-set-u2.md).
* Découvrez comment [utiliser le service StorSimple Device Manager pour gérer votre appareil StorSimple](storsimple-8000-manager-service-administration.md).

