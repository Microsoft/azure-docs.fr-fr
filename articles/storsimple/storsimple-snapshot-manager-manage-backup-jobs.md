---
title: Tâches de sauvegarde du Gestionnaire d’instantanés StorSimple | Microsoft Docs
description: Explique comment utiliser le composant logiciel enfichable MMC Gestionnaire d’instantanés StorSimple pour afficher et gérer les tâches de sauvegarde planifiées, en cours d’exécution et terminées.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: bf4dcff6-c819-4766-b9d9-9922831cb200
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 06/05/2017
ms.author: alkohli
ms.openlocfilehash: 3c26a84e32a17cba83b5ca895f146e561072fa62
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95998190"
---
# <a name="use-storsimple-snapshot-manager-to-view-and-manage-backup-jobs"></a>Utiliser le Gestionnaire d’instantanés StorSimple pour afficher et gérer les tâches de sauvegarde

## <a name="overview"></a>Vue d’ensemble
Le nœud **Tâches** dans le volet **Étendue** présente les tâches de sauvegarde **planifiées**, des **dernières 24 heures** et **en cours** que vous avez lancées de façon interactive ou à l’aide d’une stratégie configurée. 

Ce didacticiel explique comment utiliser le nœud **Tâches** pour afficher des informations sur les tâches de sauvegarde planifiées, récentes et en cours d’exécution. (La liste des tâches et les informations correspondantes s’affichent dans le volet **Résultats**.) Vous pouvez également cliquer avec le bouton droit sur une tâche répertoriée et afficher un menu contextuel présentant les actions disponibles.

## <a name="view-scheduled-jobs"></a>Afficher les tâches planifiées
Pour afficher les tâches de sauvegarde planifiées, procédez comme suit.

#### <a name="to-view-scheduled-jobs"></a>Pour afficher les tâches planifiées
1. Cliquez sur l’icône de bureau pour démarrer le Gestionnaire d’instantanés StorSimple. 
2. Dans le volet **Étendue**, développez le nœud **Tâches**, puis cliquez sur **Planifiées**. Les informations suivantes sont affichées dans le volet **Résultats** :
   
   * **Nom** : nom de l’instantané planifié
   * **Exécution suivante** : date et heure du prochain instantané planifié
   * **Dernière exécution** : date et heure de l’instantané planifié le plus récent
     
     > [!NOTE]
     > Pour les instantanés ponctuels, les informations des colonnes **Exécution suivante** et **Dernière exécution** sont identiques.
     
     ![Tâches de sauvegarde planifiées](./media/storsimple-snapshot-manager-manage-backup-jobs/HCS_SSM_Jobs_scheduled.png) 
3. Pour effectuer des actions supplémentaires sur une tâche spécifique, cliquez avec le bouton droit sur le nom de la tâche dans le volet **Résultats** et sélectionnez les options de menu de votre choix.

## <a name="view-recent-jobs"></a>Afficher les tâches récentes
Pour afficher les tâches de sauvegarde et de restauration effectuées au cours des 24 dernières heures, procédez comme suit.

#### <a name="to-view-recent-jobs"></a>Pour afficher les tâches récentes
1. Cliquez sur l’icône de bureau pour démarrer le Gestionnaire d’instantanés StorSimple.
2. Dans le volet **Étendue**, développez le nœud **Tâches**, puis cliquez sur **Dernières 24 heures**. Le volet **Résultats** affiche les tâches de sauvegarde des dernières 24 heures (avec un maximum de 64 tâches). Les informations suivantes apparaissent dans le volet **Résultats**, selon les options spécifiées pour **Affichage** :
   
   * **Nom** : nom de l’instantané planifié.
   * **Démarré** : date et heure de début de l’instantané.
   * **Arrêté** : date et l’heure de fin ou d’arrêt de l’instantané.
   * **Écoulé** : intervalle de temps entre le **démarrage** et l’**arrêt**.
   * **État** : état de la tâche terminée récemment. **Succès** indique que la sauvegarde a bien été créée. **Échec** indique que la tâche ne s’est pas exécutée correctement.
   * **Informations** : raison de l’échec.
   * **Octets traités (Mo)** : volume de données du groupe de volumes qui ont été traitées (en Mo). 
     
     ![Tâches exécutées au cours des 24 dernières heures](./media/storsimple-snapshot-manager-manage-backup-jobs/HCS_SSM_Jobs_Last_24_hours.png) 
3. Pour effectuer des actions supplémentaires sur une tâche spécifique, cliquez avec le bouton droit sur le nom de la tâche dans le volet **Résultats** et sélectionnez les options de menu de votre choix.
   
    ![Supprimer un travail](./media/storsimple-snapshot-manager-manage-backup-catalog/HCS_SSM_Delete_backup.png)

## <a name="view-currently-running-jobs"></a>Afficher les tâches en cours d’exécution
Pour afficher les tâches en cours d’exécution, procédez comme suit.

#### <a name="to-view-currently-running-jobs"></a>Pour afficher les tâches en cours d’exécution
1. Cliquez sur l’icône de bureau pour démarrer le Gestionnaire d’instantanés StorSimple.
2. Dans le volet **Étendue**, développez le nœud **Tâches**, puis cliquez sur **En cours d’exécution**. Selon les options spécifiées pour **Affichage**, les informations suivantes apparaissent dans le volet **Résultats** :
   
   * **Nom** : nom de l’instantané planifié.
   * **Démarré** : date et heure de début de l’instantané.
   * **Point de contrôle** : action actuelle de la sauvegarde.
   * **État** : pourcentage d’achèvement.
   * **Écoulé** : intervalle de temps écoulé depuis le début de la sauvegarde. 
   * **Débit moyen (Mo/s)** : rapport entre le nombre total d’octets de données traités et la durée totale de traitement (Mo).
   * **Octets traités (Mo)** : nombre total d’octets de données traités (en Mo).
   * **Octets écrits (Mo)** : nombre total d’octets de données écrits (en Mo). Cela inclut les données et les métadonnées : la valeur de ce paramètre est donc généralement supérieure à la valeur du paramètre Octets traités.
     
     ![Tâches en cours d’exécution](./media/storsimple-snapshot-manager-manage-backup-jobs/HCS_SSM_Jobs_running.png)
3. Pour effectuer des actions supplémentaires sur une tâche spécifique, cliquez avec le bouton droit sur le nom de la tâche dans le volet **Résultats** et sélectionnez les options de menu de votre choix.

## <a name="next-steps"></a>Étapes suivantes
* Découvrez comment [utiliser le Gestionnaire d’instantanés StorSimple pour gérer votre solution StorSimple](storsimple-snapshot-manager-admin.md).
* Découvrez comment [utiliser le Gestionnaire d’instantanés StorSimple pour gérer le catalogue de sauvegarde](storsimple-snapshot-manager-manage-backup-catalog.md).

