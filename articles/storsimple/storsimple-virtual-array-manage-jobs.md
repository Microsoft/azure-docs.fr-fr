---
title: Afficher et gérer les tâches StorSimple Virtual Array | Microsoft Docs
description: Décrit la page Tâches du service StorSimple Device Manager et explique comment l’utiliser pour effectuer le suivi des tâches récentes et actuelles du StorSimple Virtual Array.
services: storsimple
documentationcenter: NA
author: alkohli
manager: carmonm
editor: ''
ms.assetid: 31879821-b599-4609-a7f4-d4b0f658a933
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: na
ms.date: 11/11/2016
ms.author: alkohli
ms.openlocfilehash: d806d8a04dc1bd8547808d20c77bfec310f7dd06
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95992992"
---
# <a name="use-the-storsimple-device-manager-service-to-view-jobs-for-the-storsimple-virtual-array"></a>Utiliser le service StorSimple Device Manager pour afficher les tâches du StorSimple Virtual Array
## <a name="overview"></a>Vue d’ensemble
Le panneau **Tâches** offre un portail centralisé unique pour afficher et gérer les tâches qui sont lancées sur les Virtual Arrays connectés à votre service StorSimple Manager. Vous pouvez consulter les tâches en cours d’exécution, terminées et en échec pour plusieurs appareils virtuels. Les résultats sont présentés sous forme de tableau.

![Panneau Tâches](./media/storsimple-virtual-array-manage-jobs/ova-jobs-blade.png)

Vous pouvez rechercher rapidement les tâches qui vous intéressent en filtrant sur les champs, à savoir :

* **Période** : les tâches peuvent être filtrées selon la date et l’heure.
* **Appareils** : les tâches sont initiées sur un appareil spécifique connecté à votre service. Les tâches filtrées sont ensuite affichées sous forme de tableau sur la base des attributs suivants :
  
  * **Nom** : le nom des tâches peut présenter la valeur **Toutes**, **Sauvegarder**, **Cloner**, **Effectuer un basculement**, **Télécharger les mises à jour** ou **Installer les mises à jour**.
  * **État** : l’état des tâches peut présenter la valeur **Toutes**, **En cours**, **Réussite**, **Échec** ou **Annulé**.
  * **Entité** : les tâches peuvent être associées à un volume, un partage ou un appareil.
  * **Appareil** : nom de l’appareil sur lequel la tâche a été lancée.
  * **Démarré le** : heure à laquelle la tâche a été lancée.
  * **Durée**: durée d’exécution de la tâche.
* **État** : vous pouvez rechercher la totalité des tâches ou celles en cours d’exécution, terminées ou en échec.
* **Type de tâche** : le type de tâche peut présenter la valeur Toutes, Sauvegarder, Restaurer, Effectuer un basculement, Télécharger les mises à jour ou Installer les mises à jour.

La liste des tâches est actualisée toutes les 30 secondes.

## <a name="view-job-details"></a>Affichage des détails d’une tâche
Pour afficher les détails d’une tâche, procédez comme suit.

#### <a name="to-view-job-details"></a>Pour afficher les détails d’une tâche
1. Dans le panneau **Tâches**, affichez la ou les tâches qui vous intéressent en exécutant une requête avec les filtres appropriés. Vous pouvez rechercher des tâches terminées ou en cours d’exécution.
2. Sélectionnez une tâche dans la liste des tâches sous forme de tableau.
   
    ![Panneau Tâches](./media/storsimple-virtual-array-manage-jobs/ova-jobs-blade.png)
3. En bas de la page, cliquez sur **Détails**.
4. La boîte de dialogue **Détails** indique l’état, les détails et les statistiques horaires. La figure suivante illustre la boîte de dialogue **Détails du travail Sauvegarde** .
   
    ![Détails du travail](./media/storsimple-virtual-array-manage-jobs/ova-jobs-details.png)

#### <a name="job-failures-when-the-virtual-machine-is-paused-in-the-hypervisor"></a>Échec des tâches lorsque la machine virtuelle est en pause dans l'hyperviseur
Lorsqu’une tâche est en cours sur votre StorSimple Virtual Array et que l’appareil (machine virtuelle approvisionnée dans l’hyperviseur) est en pause pendant plus de 15 minutes, la tâche échoue. Ceci en raison du fait que l’heure de votre StorSimple Virtual Array est désynchronisée avec l'heure de Microsoft Azure. 

L’erreur suivante s’affiche : « Your device time is out of sync with the Microsoft Azure time by more than 15 minutes. Ensure that the hypervisor and the device times are synchronized with an NTP server. Verify that there are no connectivity issues. To troubleshoot connectivity issues, run diagnostic tests from the local web UI of your virtual device. » (L’heure de votre appareil est désynchronisée de plus de 15 minutes par rapport à l’heure de Microsoft Azure. Assurez-vous que l’heure de l’hyperviseur et l’heure de l’appareil sont synchronisées à l’aide d’un serveur NTP. Vérifiez qu’il n’existe aucun problème de connectivité. Pour résoudre les problèmes de connectivité, exécutez des tests de diagnostic à partir de l’interface utilisateur web locale de votre appareil virtuel.)

Cet échec concerne les tâches de sauvegarde, de restauration, de mise à jour et de basculement. Si votre machine virtuelle est provisionnée dans Hyper-V, elle finit par synchroniser son heure avec celle de votre hyperviseur. Une fois que ce sera le cas, vous pourrez redémarrer votre tâche.

## <a name="next-steps"></a>Étapes suivantes
[Découvrez comment utiliser l’interface utilisateur web locale pour gérer votre StorSimple Virtual Array](storsimple-ova-web-ui-admin.md).

