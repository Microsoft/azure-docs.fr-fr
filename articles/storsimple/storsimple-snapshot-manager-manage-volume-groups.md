---
title: Groupes de volumes du Gestionnaire d’instantanés StorSimple | Microsoft Docs
description: Décrit comment utiliser le composant logiciel enfichable MMC du Gestionnaire d’instantanés StorSimple pour créer et gérer des groupes de volumes.
services: storsimple
documentationcenter: NA
author: alkohli
manager: carmonm
editor: ''
ms.assetid: 7a232414-6a28-4b81-bd7b-cf61e28b33d7
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 06/05/2017
ms.author: alkohli
ms.openlocfilehash: 805ab0e78f0679ce483af3a73203b41132be2a59
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96022953"
---
# <a name="use-storsimple-snapshot-manager-to-create-and-manage-volume-groups"></a>Utiliser le Gestionnaire d’instantanés StorSimple pour créer et gérer des groupes de volumes
## <a name="overview"></a>Vue d’ensemble
Vous pouvez utiliser le nœud **Groupes de volumes** du volet **Étendue** pour attribuer des volumes aux groupes de volumes, afficher des informations relatives à un groupe de volumes, planifier des sauvegardes et modifier des groupes de volumes.

Les groupes de volumes sont des pools de volumes associés utilisés pour vérifier que les sauvegardes sont cohérentes au niveau applicatif. Pour plus d’informations, consultez les sections [Volumes et groupes de volumes](storsimple-what-is-snapshot-manager.md#volumes-and-volume-groups) et [Intégration avec le service VSS de Windows](storsimple-what-is-snapshot-manager.md#integration-with-windows-volume-shadow-copy-service).

> [!IMPORTANT]
> * Tous les volumes d’un groupe de volumes doivent provenir d’un fournisseur de services cloud unique.
> * Lorsque vous configurez des groupes de volumes, ne mélangez pas des volumes partagés de cluster et des volumes non partagés de cluster dans un groupe de volumes. Le Gestionnaire d’instantanés StorSimple ne prend pas en charge les combinaisons de volumes partagés de cluster et non partagés de cluster dans un même instantané.

![Nœud de groupes de volumes](./media/storsimple-snapshot-manager-manage-volume-groups/HCS_SSM_Volume_groups.png)

**Figure 1 : Nœud Groupes de volumes du Gestionnaire d’instantanés StorSimple** 

Ce didacticiel vous explique comment utiliser le Gestionnaire d’instantanés StorSimple pour :

* Afficher des informations sur vos groupes de volumes
* Créer un groupe de volumes
* Sauvegarder un groupe de volumes
* Modifier un groupe de volumes
* Supprimer un groupe de volumes

L’ensemble de ces actions sont également disponibles dans le volet **Actions** .

## <a name="view-volume-groups"></a>Afficher les groupes de volumes
Si vous cliquez sur le nœud **Groupes de volumes**, le volet **Résultats** indique les informations suivantes sur chaque groupe de volumes, en fonction des colonnes sélectionnées. (Les colonnes du volet **Résultats** sont configurables. (Cliquez avec le bouton droit sur le nœud **Volumes**, sélectionnez **Affichage**, puis sélectionnez **Ajouter/Supprimer des colonnes**.)

| Colonne de résultats | Description |
|:--- |:--- |
| Nom |La colonne **Nom** comporte le nom du groupe de volumes. |
| Application |La colonne **Applications** indique le nombre d’enregistreurs VSS actuellement installés et exécutés sur l’hôte Windows. |
| Volumes sélectionnés |La colonne **Volumes sélectionnés** indique le nombre de volumes du groupe de volumes. Une valeur nulle (0) indique qu’aucune application n’est associée aux volumes du groupe de volumes. |
| Volumes importés |La colonne **Volumes importés** représente le nombre de volumes importés. Lorsqu’elle est définie sur **Vrai**, cette colonne indique qu’un groupe de volumes a été importé du portail Azure, et n’a pas été créé dans le Gestionnaire d’instantanés StorSimple. |

> [!NOTE]
> Les groupes de volumes du Gestionnaire d’instantanés StorSimple s’affichent également sur l’onglet **Stratégies de sauvegarde** du portail Azure.
> 
> 

## <a name="create-a-volume-group"></a>Créer un groupe de volumes
Exécutez la procédure suivante pour créer un groupe de volumes.

#### <a name="to-create-a-volume-group"></a>Pour créer un groupe de volumes
1. Cliquez sur l’icône de bureau pour démarrer le Gestionnaire d’instantanés StorSimple.
2. Dans le volet **Étendue**, cliquez avec le bouton droit sur **Groupes de volumes**, puis cliquez sur **Créer un groupe de volumes**.
   
    ![Créer un groupe de volumes](./media/storsimple-snapshot-manager-manage-volume-groups/HCS_SSM_Create_volume_group.png)
   
    La boîte de dialogue **Créer un groupe de volumes** apparaît.
   
    ![Boîte de dialogue Créer un groupe de volumes](./media/storsimple-snapshot-manager-manage-volume-groups/HCS_SSM_CreateVolumeGroup_dialog.png)
3. Entrez les informations suivantes :
   
   1. Dans la zone **Nom** , saisissez un nom unique pour le nouveau groupe de volumes.
   2. Dans la zone **Applications** , sélectionnez les applications associées aux volumes que vous ajouterez au groupe de volumes.
      
       La zone **Applications** répertorie uniquement les applications qui utilisent des volumes StorSimple et pour lesquelles des enregistreurs VSS sont activés. Un enregistreur VSS est activé uniquement si l’ensemble des volumes auxquels il a accès sont des volumes StorSimple. Si la zone Applications est vide, aucune application utilisant des volumes Azure StorSimple et présentant des enregistreurs VSS pris en charge n’est installée. (Actuellement, Azure StorSimple prend en charge Microsoft Exchange et SQL Server.) Pour plus d’informations sur les enregistreurs VSS, consultez la page [Intégration avec le service VSS de Windows](storsimple-what-is-snapshot-manager.md#integration-with-windows-volume-shadow-copy-service).
      
       Si vous sélectionnez une application, l’ensemble des volumes associés sont automatiquement sélectionnés. Inversement, si vous sélectionnez des volumes associés à une application spécifique, l’application est automatiquement sélectionnée dans la zone **Applications** . 
   3. Dans la zone **Volumes** , sélectionnez les volumes StorSimple à ajouter au groupe de volumes. 
      
      * Vous pouvez inclure des volumes à une ou plusieurs partitions. (Les volumes à plusieurs partitions peuvent être des disques dynamiques ou des disques de base avec plusieurs partitions.) Un volume contenant plusieurs partitions est traité comme une unité unique. Par conséquent, si vous ajoutez uniquement une des partitions au groupe de volumes, l’ensemble des autres partitions sont automatiquement ajoutées au même moment. Un volume à plusieurs partitions ajouté à un groupe de volumes continue à être considéré comme une unité unique.
      * Vous pouvez créer des groupes de volumes vides. Pour ce faire, ne leur attribuez aucun volume. 
      * Ne mélangez pas des volumes partagés de cluster et des volumes non partagés de cluster au sein d’un même groupe de volumes. Le Gestionnaire d’instantanés StorSimple ne prend pas en charge les combinaisons de volumes partagés de cluster et non partagés de cluster dans un même instantané.
4. Cliquez sur **OK** pour enregistrer le groupe de volumes.

## <a name="back-up-a-volume-group"></a>Sauvegarder un groupe de volumes
Exécutez la procédure suivante pour sauvegarder un groupe de volumes.

#### <a name="to-back-up-a-volume-group"></a>Pour sauvegarder un groupe de volumes
1. Cliquez sur l’icône de bureau pour démarrer le Gestionnaire d’instantanés StorSimple.
2. Dans le volet **Étendue**, développez le nœud **Groupes de volumes**, cliquez avec le bouton droit sur un nom de groupe de volumes, puis cliquez sur **Démarrer la sauvegarde**.
   
    ![Sauvegarde immédiate d’un groupe de volumes](./media/storsimple-snapshot-manager-manage-volume-groups/HCS_SSM_Take_backup.png)
3. Dans la boîte de dialogue **Démarrer la sauvegarde**, sélectionnez **Instantané local** ou **Instantané cloud**, puis cliquez sur **Créer**.
   
    ![Boîte de dialogue Démarrer la sauvegarde](./media/storsimple-snapshot-manager-manage-volume-groups/HCS_SSM_TakeBackup_dialog.png)
4. Pour vérifier que la sauvegarde est en cours, développez le nœud **Tâches**, puis cliquez sur **En cours d’exécution**. La sauvegarde doit être répertoriée.
5. Pour afficher l’instantané terminé, développez le nœud **Catalogue de sauvegarde**, développez le nom du groupe de volumes, puis cliquez sur **Instantané local** ou **Instantané cloud**. Si elle aboutit, la sauvegarde est répertoriée.

## <a name="edit-a-volume-group"></a>Modifier un groupe de volumes
Exécutez la procédure suivante pour modifier un groupe de volumes.

#### <a name="to-edit-a-volume-group"></a>Pour modifier un groupe de volumes
1. Cliquez sur l’icône de bureau pour démarrer le Gestionnaire d’instantanés StorSimple.
2. Dans le volet **Étendue**, développez le nœud **Groupes de volumes**, cliquez avec le bouton droit sur un nom de groupe de volumes, puis cliquez sur **Modifier**.
3. La boîte de dialogue Créer un groupe de volumes apparaît. Vous pouvez modifier les entrées **Nom**, **Applications** et **Volumes**.
4. Cliquez sur **OK** pour enregistrer vos modifications.

## <a name="delete-a-volume-group"></a>Supprimer un groupe de volumes
Exécutez la procédure suivante pour supprimer un groupe de volumes. 

> [!WARNING]
> Le cas échéant, vous supprimez également l’ensemble des sauvegardes associées au groupe de volumes.
> 
> 

#### <a name="to-delete-a-volume-group"></a>Pour supprimer un groupe de volumes
1. Cliquez sur l’icône de bureau pour démarrer le Gestionnaire d’instantanés StorSimple.
2. Dans le volet **Étendue**, développez le nœud **Groupes de volumes**, cliquez avec le bouton droit sur un nom de groupe de volumes, puis cliquez sur **Supprimer**.
3. La boîte de dialogue **Supprimer le groupe de volumes** apparaît. Saisissez **Confirmer** dans la zone de texte, puis cliquez sur **OK**.
   
    Le groupe de volumes supprimé disparaît de la liste du volet **Résultats** et toutes les sauvegardes associées à ce groupe de volumes sont supprimées du catalogue des sauvegardes.

## <a name="next-steps"></a>Étapes suivantes
* Découvrez comment [utiliser le Gestionnaire d’instantanés StorSimple pour gérer votre solution StorSimple](storsimple-snapshot-manager-admin.md).
* Découvrez comment [utiliser le Gestionnaire d’instantanés StorSimple pour créer et gérer des stratégies de sauvegarde](storsimple-snapshot-manager-manage-backup-policies.md).

