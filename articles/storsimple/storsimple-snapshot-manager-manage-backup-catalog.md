---
title: Catalogue de sauvegarde du Gestionnaire d’instantanés StorSimple | Microsoft Docs
description: Décrit comment utiliser le composant logiciel enfichable MMC du Gestionnaire d’instantanés StorSimple pour afficher et gérer le catalogue de sauvegarde.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: 6abdbfd2-22ce-45a5-aa15-38fae4c8f4ec
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 06/05/2017
ms.author: alkohli
ms.openlocfilehash: 2efc35e65ca1db2b5241e1d3b2798e068880c87e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90054991"
---
# <a name="use-storsimple-snapshot-manager-to-manage-the-backup-catalog"></a>Utiliser le Gestionnaire d’instantanés StorSimple pour gérer le catalogue de sauvegarde

## <a name="overview"></a>Vue d’ensemble
La principale fonction de StorSimple Snapshot Manager consiste à permettre la création de copies de sauvegarde des volumes StorSimple cohérentes au niveau applicatif sous la forme d’instantanés. Les instantanés sont ensuite répertoriés dans un fichier XML appelé *catalogue de sauvegarde*. Le catalogue de sauvegarde organise les instantanés par groupe de volumes, puis par instantané local ou instantané cloud.

Ce didacticiel explique comment utiliser le nœud **Catalogue de sauvegarde** pour effectuer les tâches suivantes :

* Restaurer un volume
* Cloner un volume ou un groupe de volumes
* Supprimer une sauvegarde
* Récupérer un fichier
* Restaurer la base de données du Gestionnaire d’instantanés StorSimple

Vous pouvez afficher le catalogue de sauvegarde en développant le nœud **Catalogue de sauvegarde** dans le volet **Étendue**, puis en développant le groupe de volumes.

* Si vous cliquez sur le nom du groupe de volumes, le volet **Résultats** affiche le nombre d’instantanés locaux et d’instantanés cloud disponibles pour le groupe de volumes. 
* Si vous cliquez sur **Instantané local** ou **Instantané cloud**, le volet **Résultats** affiche les informations suivantes sur chaque instantané de sauvegarde (selon les paramètres sélectionnés pour **Affichage**) :
  
  * **Nom** : heure à laquelle l’instantané a été capturé.
  * **Type** : indique s’il s’agit d’un instantané local ou d’un instantané cloud.
  * **Propriétaire** : propriétaire du contenu. 
  * **Disponible** : indique si l’instantané est actuellement disponible. La valeur **True** indique que l’instantané est disponible et peut être restauré ; la valeur **False** indique que l’instantané n’est plus disponible. 
  * **Importé** : indique si la sauvegarde a été importée. La valeur **True** indique que la sauvegarde a été importée à partir du service StorSimple Device Manager au moment où l’appareil a été configuré dans le Gestionnaire d’instantanés StorSimple ; la valeur **False** indique qu’elle n’a pas été importée, mais qu’elle a été créée par le Gestionnaire d’instantanés StorSimple. (Vous pouvez facilement identifier un groupe de volumes importé, car un suffixe identifiant l’appareil à partir duquel le groupe de volumes a été importé est ajouté.)
    
    ![Catalogue de sauvegarde](./media/storsimple-snapshot-manager-manage-backup-catalog/HCS_SSM_Backup_catalog.png)
* Si vous développez **Instantané local** ou **Instantané cloud**, puis cliquez sur un nom d’instantané individuel, le volet **Résultats** affiche les informations suivantes sur l’instantané sélectionné :
  
  * **Nom** : volume identifié par une lettre de lecteur. 
  * **Nom local** : nom local du lecteur (s’il est disponible). 
  * **Appareil** : nom de l’appareil sur lequel réside le volume. 
  * **Disponible** : indique si l’instantané est actuellement disponible. La valeur **True** indique que l’instantané est disponible et peut être restauré ; la valeur **False** indique que l’instantané n’est plus disponible. 

## <a name="restore-a-volume"></a>Restaurer un volume
Pour restaurer un volume à partir d’une sauvegarde, procédez comme suit.

#### <a name="prerequisites"></a>Conditions préalables requises
Si ce n’est pas déjà fait, créez un volume et un groupe de volumes, puis supprimez le volume. Par défaut, le Gestionnaire d’instantanés StorSimple sauvegarde un volume avant d’autoriser sa suppression. Cette précaution permet d’éviter la perte de données si le volume est supprimé par inadvertance ou si les données doivent être récupérées pour une raison quelconque. 

Le Gestionnaire d’instantanés StorSimple affiche le message suivant pendant la création de la sauvegarde préventive.

![Message automatique d’instantané](./media/storsimple-snapshot-manager-manage-backup-catalog/HCS_SSM_Automatic_snap.png) 

> [!IMPORTANT]
> Vous ne pouvez pas supprimer un volume qui fait partie d’un groupe de volumes. L’option de suppression n’est pas disponible. <br>
> 
> 

#### <a name="to-restore-a-volume"></a>Pour restaurer un volume
1. Cliquez sur l’icône de bureau pour démarrer le Gestionnaire d’instantanés StorSimple. 
2. Dans le volet **Étendue**, développez le nœud **Catalogue de sauvegarde**, développez un groupe de volumes, puis cliquez sur **Instantanés locaux** ou **Instantanés cloud**. Une liste d’instantanés de sauvegarde s’affiche dans le volet **Résultats** .
3. Recherchez la sauvegarde à restaurer, cliquez dessus avec le bouton droit, puis cliquez sur **Restaurer**.
   
    ![Restaurer le catalogue de sauvegarde](./media/storsimple-snapshot-manager-manage-backup-catalog/HCS_SSM_Restore_BU_catalog.png) 
4. Sur la page de confirmation, passez en revue les détails, tapez **Confirmer**, puis cliquez sur **OK**. Le Gestionnaire d’instantanés StorSimple utilise la sauvegarde pour restaurer le volume.
   
    ![Message de confirmation de la restauration](./media/storsimple-snapshot-manager-manage-backup-catalog/HCS_SSM_Restore_volume_msg.png) 
5. Vous pouvez suivre la tâche de restauration en cours d’exécution. Dans le volet **Étendue**, développez le nœud **Tâches**, puis cliquez sur **En cours d’exécution**. Les détails de la tâche s’affichent dans le volet **Résultats** . À l’issue de la tâche de restauration, les détails de la tâche sont transférés vers la liste **Dernières 24 heures** .

## <a name="clone-a-volume-or-volume-group"></a>Cloner un volume ou un groupe de volumes
Pour dupliquer (cloner) un volume ou un groupe de volumes, procédez comme suit.

#### <a name="to-clone-a-volume-or-volume-group"></a>Pour cloner un volume ou un groupe de volumes
1. Cliquez sur l’icône de bureau pour démarrer le Gestionnaire d’instantanés StorSimple.
2. Dans le volet **Étendue**, développez le nœud **Catalogue de sauvegarde**, développez un groupe de volumes, puis cliquez sur **Instantanés cloud**. Une liste de sauvegardes s’affiche dans le volet **Résultats** .
3. Recherchez le volume ou le groupe de volumes à cloner, cliquez avec le bouton droit sur le volume ou le nom du groupe de volumes, puis cliquez sur **Cloner**. La boîte de dialogue **Cloner l’instantané cloud** s’affiche.
   
    ![Cloner un instantané cloud](./media/storsimple-snapshot-manager-manage-backup-catalog/HCS_SSM_Clone.png) 
4. Renseignez la boîte de dialogue **Cloner l’instantané cloud** comme suit : 
   
   1. Dans la zone de texte **Nom** , tapez un nom pour le volume cloné. Ce nom sera affiché dans le nœud **Volumes** . 
   2. (Facultatif) Sélectionnez **Lecteur**, puis sélectionnez une lettre de lecteur dans la liste déroulante.
   3. (Facultatif) Sélectionnez **Dossier (NTFS)** , puis tapez un chemin de dossier ou cliquez sur Parcourir et sélectionnez un emplacement pour le dossier. 
   4. Cliquez sur **Créer**.
5. Une fois le processus de clonage terminé, vous devez initialiser le volume cloné. Démarrez le Gestionnaire de serveur, puis l’outil Gestion des disques. Pour obtenir des instructions détaillées, consultez la section [Monter les volumes](storsimple-snapshot-manager-manage-volumes.md#mount-volumes). Une fois initialisé, le volume est répertorié sous le nœud **Volumes** dans le volet **Étendue**. Si le volume n’est pas répertorié, actualisez la liste des volumes (cliquez avec le bouton droit sur le nœud **Volumes**, puis cliquez sur **Actualiser**).

## <a name="delete-a-backup"></a>Supprimer une sauvegarde
Pour supprimer un instantané du catalogue de sauvegarde, procédez comme suit. 

> [!NOTE]
> La suppression d’un instantané entraîne la suppression des données sauvegardées associées à l’instantané. Cependant, le processus de nettoyage des données du cloud peut prendre un certain temps.<br>


#### <a name="to-delete-a-backup"></a>Pour supprimer une sauvegarde
1. Cliquez sur l’icône de bureau pour démarrer le Gestionnaire d’instantanés StorSimple.
2. Dans le volet **Étendue**, développez le nœud **Catalogue de sauvegarde**, développez un groupe de volumes, puis cliquez sur **Instantanés locaux** ou **Instantanés cloud**. Une liste d’instantanés s’affiche dans le volet **Résultats** .
3. Cliquez avec le bouton droit sur l’instantané à supprimer, puis cliquez sur **Supprimer**.
4. Lorsque le message de confirmation s’affiche, cliquez sur **OK**.

## <a name="recover-a-file"></a>Récupérer un fichier
Si un fichier est supprimé accidentellement d’un volume, vous pouvez le restaurer en récupérant un instantané antérieur à la suppression, en utilisant cet instantané pour créer un clone du volume, puis en copiant le fichier du volume cloné dans le volume d’origine.

#### <a name="prerequisites"></a>Conditions préalables requises
Avant de commencer, assurez-vous que vous disposez d’une sauvegarde actuelle du groupe de volumes. Ensuite, supprimez un fichier stocké sur un des volumes de ce groupe de volumes. Enfin, procédez comme suit pour restaurer le fichier supprimé de votre sauvegarde. 

#### <a name="to-recover-a-deleted-file"></a>Pour récupérer un fichier supprimé
1. Cliquez sur l’icône du Gestionnaire d’instantanés StorSimple de votre bureau. La fenêtre de console du Gestionnaire d’instantanés StorSimple s’affiche. 
2. Dans le volet **Étendue**, développez le **Catalogue de sauvegarde**, puis accédez à un instantané qui contient le fichier supprimé. En règle générale, vous devez sélectionner un instantané créé juste avant la suppression.
3. Recherchez le volume à cloner, cliquez dessus avec le bouton droit, puis cliquez sur **Cloner**. La boîte de dialogue **Cloner l’instantané cloud** s’affiche.
   
    ![Cloner un instantané cloud](./media/storsimple-snapshot-manager-manage-backup-catalog/HCS_SSM_Clone.png) 
4. Renseignez la boîte de dialogue **Cloner l’instantané cloud** comme suit : 
   
   1. Dans la zone de texte **Nom** , tapez un nom pour le volume cloné. Ce nom sera affiché dans le nœud **Volumes** . 
   2. (Facultatif) Sélectionnez **Lecteur**, puis sélectionnez une lettre de lecteur dans la liste déroulante. 
   3. (Facultatif) Sélectionnez **Dossier (NTFS)** , puis tapez un chemin de dossier ou cliquez sur **Parcourir** et sélectionnez un emplacement pour le dossier. 
   4. Cliquez sur **Créer**. 
5. Une fois le processus de clonage terminé, vous devez initialiser le volume cloné. Démarrez le Gestionnaire de serveur, puis l’outil Gestion des disques. Pour obtenir des instructions détaillées, consultez la section [Monter les volumes](storsimple-snapshot-manager-manage-volumes.md#mount-volumes). Une fois initialisé, le volume est répertorié sous le nœud **Volumes** dans le volet **Étendue**. 
   
    Si le volume n’est pas répertorié, actualisez la liste des volumes (cliquez avec le bouton droit sur le nœud **Volumes**, puis cliquez sur **Actualiser**).
6. Ouvrez le dossier NTFS contenant le volume cloné, développez le nœud **Volumes** , puis ouvrez le volume cloné. Recherchez le fichier à récupérer, puis copiez-le dans le volume principal.
7. Une fois le fichier restauré, vous pouvez supprimer le dossier NTFS contenant le volume cloné.

## <a name="restore-the-storsimple-snapshot-manager-database"></a>Restaurer la base de données du Gestionnaire d’instantanés StorSimple
Il est recommandé de sauvegarder régulièrement la base de données du Gestionnaire d’instantanés StorSimple sur l’ordinateur hôte. Si un incident se produit ou l’ordinateur hôte subit une défaillance quelconque, vous pourrez alors la restaurer à partir de la sauvegarde. La création de la sauvegarde de la base de données est un processus manuel.

#### <a name="to-back-up-and-restore-the-database"></a>Pour sauvegarder et restaurer la base de données
1. Arrêtez le service de gestion Microsoft StorSimple :
   
   1. Démarrez le Gestionnaire de serveur.
   2. Sur le tableau de bord du Gestionnaire de serveur, dans le menu **Outils**, sélectionnez **Services**.
   3. Dans la fenêtre **Services**, sélectionnez le **Service de gestion Microsoft StorSimple**.
   4. Dans le volet droit, sous **Service de gestion Microsoft StorSimple**, cliquez sur **Arrêter le service**.
2. Sur l’ordinateur hôte, accédez à C:\ProgramData\Microsoft\StorSimple\BACatalog. 
   
   > [!NOTE]
   > ProgramData est un dossier masqué.
   > 
   > 
3. Recherchez le fichier XML de catalogue, copiez le fichier et stockez la copie dans un emplacement sûr ou dans le cloud. Si l’hôte subit une défaillance, vous pouvez utiliser ce fichier de sauvegarde pour récupérer les stratégies de sauvegarde créées dans le Gestionnaire d’instantanés StorSimple.
   
    ![Fichier de catalogue de sauvegarde Azure StorSimple](./media/storsimple-snapshot-manager-manage-backup-catalog/HCS_SSM_bacatalog.png)
4. Redémarrez le service de gestion Microsoft StorSimple : 
   
   1. Sur le tableau de bord du Gestionnaire de serveur, dans le menu **Outils**, sélectionnez **Services**.
   2. Dans la fenêtre **Services**, sélectionnez le **Service de gestion Microsoft StorSimple**.
   3. Dans le volet droit, sous **Service de gestion Microsoft StorSimple**, cliquez sur **Redémarrer le service**.
5. Sur l’ordinateur hôte, accédez à C:\ProgramData\Microsoft\StorSimple\BACatalog. 
6. Supprimez le fichier XML de catalogue et remplacez-le par la version de sauvegarde que vous avez créée. 
7. Cliquez sur l’icône du Gestionnaire d’instantanés StorSimple sur le Bureau pour démarrer le Gestionnaire d’instantanés StorSimple. 

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur [l’utilisation du Gestionnaire d’instantanés StorSimple pour gérer votre solution StorSimple](storsimple-snapshot-manager-admin.md).
* En savoir plus sur [les tâches et les flux de travail du Gestionnaire d’instantanés StorSimple](storsimple-snapshot-manager-admin.md#storsimple-snapshot-manager-tasks-and-workflows).

