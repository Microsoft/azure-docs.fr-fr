---
title: Afficher et gérer des travaux pour la gamme StorSimple 8000 | Microsoft Docs
description: Décrit le panneau Travaux (jobs) du service StorSimple Device Manager et explique comment l’utiliser pour effectuer le suivi des travaux de sauvegarde planifiée, actuelle et récente.
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
ms.openlocfilehash: dc630021cbf93074119d4feedcb3a09fae1453e3
ms.sourcegitcommit: a2d8acc1b0bf4fba90bfed9241b299dc35753ee6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/12/2020
ms.locfileid: "91949426"
---
# <a name="use-the-storsimple-device-manager-service-to-view-and-manage-jobs-update-3-and-later"></a>Utiliser le service StorSimple Device Manager pour afficher et gérer des travaux (Update 3 et versions ultérieures)

## <a name="overview"></a>Vue d’ensemble
Le panneau **Travaux (jobs)** est un portail centralisé unique qui permet de consulter et de gérer les travaux qui ont été lancés sur les appareils connectés à votre service StorSimple Device Manager. Vous pouvez consulter les tâches planifiées, en cours d'exécution, terminées, annulées et en échec pour plusieurs appareils. Les résultats sont présentés sous forme de tableau.

![Panneau Tâches](./media/storsimple-8000-manage-jobs-u2/jobs1.png)

Vous pouvez rechercher rapidement les tâches qui vous intéressent en filtrant sur les champs, à savoir :

* **État** : les travaux peuvent être en cours, s’être bien déroulés, avoir échoué, avoir été annulés ou s’être bien déroulés mais avec des erreurs.
* **Période** : les tâches peuvent être filtrées selon la date et l’heure. Les plages sont : dernière heure, dernières 24 heures, derniers 7 jours, 30 derniers jours, année dernière ou une date personnalisée.
* **Type** : le type de travail peut être une sauvegarde planifiée, une sauvegarde manuelle, une restauration de sauvegarde, un clonage de volume, un basculement de conteneurs de volumes, une création de volume épinglé localement, une modification de volume, une installation de mises à jour, une collecte de journaux d’activité de support et une création d’appliance cloud.
* **Appareils** : les tâches sont initiées sur un certain appareil connecté à votre service.
  
Les tâches filtrées sont ensuite affichées sous forme de tableau sur la base des attributs suivants :
  
* **Nom** : une sauvegarde planifiée, une sauvegarde manuelle, une restauration de sauvegarde, un clonage de volume, un basculement de conteneurs de volumes, une création de volume épinglé localement, une modification de volume, une installation de mises à jour, une collecte de journaux d’activité de support ou une création d’appliance cloud.
* **État** : en cours d'exécution, terminées, annulées, en échec, en cours d'annulation ou terminées avec des erreurs.
* **Entité** : les tâches peuvent être associées à un volume, une stratégie de sauvegarde ou un appareil. Par exemple, une tâche de clonage est associée à un volume, tandis qu'une tâche de sauvegarde planifiée est associée à une stratégie de sauvegarde. Une tâche d’appareil est créée à la suite d’une récupération d'urgence ou d’une opération de restauration.
* **Appareil** : nom de l’appareil sur lequel la tâche a été lancée.
* **Démarré le** : heure à laquelle la tâche a été lancée.
* **Durée** : le temps nécessaire pour terminer le travail.

La liste des tâches est actualisée toutes les 30 secondes.

Cette page vous permet d’effectuer diverses actions liées aux tâches, à savoir :

* Affichage des détails d’une tâche
* Annulation d’une tâche

## <a name="view-job-details"></a>Affichage des détails d’une tâche
Pour afficher les détails d’une tâche, procédez comme suit.

#### <a name="to-view-job-details"></a>Pour afficher les détails d’une tâche
1. Accédez à votre service StorSimple Device Manager et cliquez sur **Travaux (jobs)** .

2. Dans le panneau **Travaux (jobs)** , affichez les travaux qui vous intéressent en exécutant une requête avec les filtres appropriés. Vous pouvez rechercher des tâches terminées, en cours d'exécution ou annulées.

    ![Panneau Tâches 2](./media/storsimple-8000-manage-jobs-u2/jobs1.png)

2. Sélectionnez un travail et cliquez dessus.

    ![Panneau Tâches 3](./media/storsimple-8000-manage-jobs-u2/jobs3.png)

3. Dans le panneau des détails du travail, vous pouvez consulter l’état, les détails, les statistiques temporelles et les statistiques de données.
   
    ![Détails du travail](./media/storsimple-8000-manage-jobs-u2/jobs4.png)

## <a name="cancel-a-job"></a>Annulation d’une tâche
Pour annuler une tâche en cours d’exécution, procédez comme suit.

> [!NOTE]
> Certaines tâches, telles que la modification d'un volume pour modifier le type de volume ou l'extension d'un volume, ne peuvent pas être annulées.


### <a name="to-cancel-a-job"></a>Pour annuler une tâche
1. Dans la page **Tâches** , affichez les tâches en cours d’exécution que vous voulez annuler en exécutant une requête avec les filtres appropriés. Sélectionnez la tâche.

2. Cliquez avec le bouton droit sur le travail sélectionné pour appeler le menu contextuel, puis cliquez sur **Annuler**.

    ![Détails du travail 2](./media/storsimple-8000-manage-jobs-u2/jobs2.png)

3. Cliquez sur **Oui**lorsque vous êtes invité à confirmer l’opération. Cette tâche est à présent annulée.

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur la [gestion de vos stratégies de sauvegarde StorSimple](storsimple-8000-manage-backup-policies-u2.md).
* Découvrez comment [utiliser le service StorSimple Device Manager pour gérer votre appareil StorSimple](storsimple-8000-manager-service-administration.md).

