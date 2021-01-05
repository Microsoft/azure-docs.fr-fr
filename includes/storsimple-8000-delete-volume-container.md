---
title: Fichier Include
description: Fichier Include
services: storsimple
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 06/08/2018
ms.author: alkohli
ms.custom: include file
ms.openlocfilehash: 30bbd06e36ed1e03caa391165a8abc275f1899a7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87102629"
---
Pour supprimer un conteneur de volumes, vous devez
 - supprimer les volumes du conteneur de volumes. Si le conteneur de volumes a des volumes associés, commencez par mettre ces volumes hors connexion. Suivez les étapes de la [Mise hors connexion d’un volume](../articles/storsimple/storsimple-8000-manage-volumes-u2.md#take-a-volume-offline). Une fois les volumes hors connexion, vous pouvez les supprimer. 
 - supprimer les stratégies de sauvegarde associées et les instantanés cloud. Vérifiez si le conteneur de volumes est associé à des stratégies de sauvegarde et à des instantanés cloud. Si c’est le cas, [supprimez les stratégies de sauvegarde](../articles/storsimple/storsimple-8000-manage-backup-policies-u2.md#delete-a-backup-policy). Les instantanés cloud sont également supprimés. 
 
Lorsque le conteneur de volumes n’est associé à aucun volume, aucune stratégie de sauvegarde et aucun instantané cloud, vous pouvez le supprimer. Pour supprimer un conteneur de volumes, procédez comme suit.

#### <a name="to-delete-a-volume-container"></a>Pour supprimer un conteneur de volumes
1. Accédez à votre service StorSimple Device Manager et cliquez sur **Appareils**. Sélectionnez l’appareil et cliquez dessus, puis accédez à **Paramètres > Gérer > Conteneurs de volumes**.

    ![Panneau Conteneurs de volumes](./media/storsimple-8000-create-volume-container/createvolumecontainer2.png)

2. Dans la liste tabulaire des conteneurs de volumes, sélectionnez le conteneur de volumes à supprimer, cliquez avec le bouton droit sur **...** , puis sélectionnez **Supprimer**.

    ![Supprimer un conteneur de volumes](./media/storsimple-8000-delete-volume-container/deletevolumecontainer1.png)

3. Si un conteneur de volumes n’est associé à aucun volume, aucune stratégie de sauvegarde et aucun instantané cloud, il peut être supprimé. Lorsque vous êtes invité à confirmer l’opération, vérifiez et cochez l’option indiquant l’impact de la suppression du conteneur de volumes. Cliquez sur **Supprimer**, puis supprimez le conteneur de volumes.

    ![Confirmer la suppression](./media/storsimple-8000-delete-volume-container/deletevolumecontainer2.png)

La liste des conteneurs de volumes est mise à jour pour refléter la suppression du conteneur de volumes.

![Capture d’écran de la page Conteneur de volumes. La liste tabulaire des conteneurs de volumes ne contient plus le conteneur supprimé.](./media/storsimple-8000-delete-volume-container/deletevolumecontainer5.png)


