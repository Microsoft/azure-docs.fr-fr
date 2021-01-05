---
title: Gestionnaire d’instantanés StorSimple et volumes | Microsoft Docs
description: Décrit comment utiliser le composant logiciel enfichable MMC du Gestionnaire d’instantanés StorSimple pour afficher et gérer des volumes, ainsi que pour configurer des sauvegardes.
services: storsimple
documentationcenter: NA
author: alkohli
manager: carmonm
editor: ''
ms.assetid: 78896323-e57c-431e-bbe2-0cbde1cf43a2
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 04/18/2016
ms.author: alkohli
ms.openlocfilehash: 309fa85d0a4d877522a89dd8f1e6e71fb2074744
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96022936"
---
# <a name="use-storsimple-snapshot-manager-to-view-and-manage-volumes"></a>Utiliser le Gestionnaire d’instantanés StorSimple pour afficher et gérer les volumes
## <a name="overview"></a>Vue d’ensemble
Pour sélectionner des volumes et afficher des détails les concernant, vous pouvez accéder au nœud **Volumes** (sur le volet **Étendue**) du Gestionnaire d’instantanés StorSimple. Les volumes sont présentés en tant que lecteurs correspondant aux volumes montés par l’hôte. Le nœud **Volumes** répertorie les volumes et les types de volumes locaux qui sont pris en charge par StorSimple, notamment les volumes découverts via l’utilisation d’iSCSI et d’un appareil. 

Pour en savoir plus sur les volumes pris en charge, accédez à la section [Prise en charge de plusieurs types de volumes](storsimple-what-is-snapshot-manager.md#support-for-multiple-volume-types).

![Liste des volumes dans le volet Résultats](./media/storsimple-snapshot-manager-manage-volumes/HCS_SSM_Volume_node.png)

Le nœud **Volumes** vous permet également de relancer l’analyse des volumes découverts par le Gestionnaire d’instantanés StorSimple, ou de les supprimer. 

Ce didacticiel vous explique comment monter, initialiser et formater les volumes, puis utiliser le Gestionnaire d’instantanés StorSimple pour :

* Afficher des informations sur les volumes 
* Supprimer des volumes
* Relancer l’analyse des volumes 
* Configurer un volume de base et le sauvegarder
* Configurer un volume dynamique mis en miroir et le sauvegarder

> [!NOTE]
> L’ensemble des actions du nœud **Volume** sont disponibles dans le volet **Actions**.
> 
> 

## <a name="mount-volumes"></a>Monter les volumes
Appliquez la procédure suivante pour monter, initialiser et formater des volumes StorSimple. Cette procédure utilise Gestion des disques, un utilitaire système dédié à la gestion des disques durs et des volumes et partitions correspondants. Pour plus d’informations sur le composant Gestion des disques, accédez à la section [Gestion des disques](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc770943(v=ws.11)) du site web Microsoft TechNet.

#### <a name="to-mount-volumes"></a>Pour monter des volumes
1. Sur votre ordinateur hôte, démarrez l’initiateur Microsoft iSCSI.
2. Renseignez l’une des adresses IP d’interface en tant qu’adresse de portail cible ou adresse IP de détection, puis connectez-vous à l’appareil. Une fois que l’appareil est connecté, les volumes sont accessibles par votre système Windows. Pour plus d’informations sur l’utilisation de l’initiateur Microsoft iSCSI, accédez à la section « Connexion à un appareil cible iSCSI » dans [Installation et configuration de l’initiateur Microsoft iSCSI][1].
3. Pour démarrer le composant Gestion des disques, utilisez l’une des options suivantes :
   
   * Dans la zone **Exécuter** , saisissez Diskmgmt.msc.
   * Lancez le Gestionnaire de serveur, développez le nœud **Stockage**, puis sélectionnez **Gestion des disques**.
   * Lancez **Outils d’administration**, développez le nœud **Gestion de l’ordinateur**, puis sélectionnez **Gestion des disques**. 
     
     > [!NOTE]
     > Vous devez utiliser des privilèges administrateur pour exécuter la Gestion des disques.
     > 
     > 
4. Prenez le(s) volume(s) en ligne :
   
   1. Dans Gestion des disques, cliquez avec le bouton droit sur un volume marqué **Hors connexion**.
   2. Cliquez sur **Réactiver le disque**. Après la réactivation, le disque doit être marqué **En ligne** .
5. Initialisez le(s) volume(s) :
   
   1. Cliquez avec le bouton droit sur les volumes découverts.
   2. Dans le menu, sélectionnez **Initialiser le disque**.
   3. Dans la boîte de dialogue **Initialiser le disque**, sélectionnez les disques que vous souhaitez initialiser, puis cliquez sur **OK**.
6. Formatez les volumes simples :
   
   1. Cliquez avec le bouton droit sur un volume que vous souhaitez mettre en forme.
   2. Dans le menu, sélectionnez **Nouveau volume simple**.
   3. Utilisez l’assistant Nouveau volume simple pour mettre en forme le volume :
      
      * Spécifiez la taille du volume.
      * Fournissez une lettre de lecteur.
      * Sélectionnez le système de fichiers NTFS.
      * Spécifiez une taille d’unité d’allocation 64 Ko.
      * Effectuez un formatage rapide.
7. Formatez des volumes à plusieurs partitions. Pour obtenir des instructions, accédez à la section « Partitions et volumes » de la page [Implémentation de la gestion des disques](/previous-versions/tn-archive/dd163556(v=technet.10)).

## <a name="view-information-about-your-volumes"></a>Afficher les informations sur les volumes
Appliquez la procédure suivante pour afficher des informations sur les volumes locaux et Azure StorSimple.

#### <a name="to-view-volume-information"></a>Pour afficher des informations sur les volumes
1. Cliquez sur l’icône de bureau pour démarrer le Gestionnaire d’instantanés StorSimple. 
2. Dans le volet **Étendue**, cliquez sur le nœud **Volumes**. Une liste des volumes locaux et montés, comportant notamment l’ensemble des volumes Azure StorSimple, apparaît dans le volet **Résultats** . Les colonnes du volet **Résultats** sont configurables. (Cliquez avec le bouton droit sur le nœud **Volumes**, sélectionnez **Affichage**, puis sélectionnez **Ajouter/Supprimer des colonnes**.)
   
    ![Configurer les colonnes](./media/storsimple-snapshot-manager-manage-volumes/HCS_SSM_View_volumes.png)
   
   | Colonne de résultats | Description |
   |:--- |:--- |
   |  Nom |La colonne **Nom** contient la lettre de lecteur attribuée à chaque volume découvert. |
   |  Appareil |La colonne **Appareil** comporte l’adresse IP de l’appareil connecté à l’ordinateur hôte. |
   |  Nom de volume d’appareil |La colonne **Nom de volume d’appareil** comporte le nom du volume d’appareil auquel appartient le volume sélectionné. Il s’agit du nom de volume défini dans le portail Azure pour ce volume spécifique. |
   |  Chemins d’accès |La colonne **Chemins d’accès** affiche le chemin d’accès au volume. Il s’agit de la lettre de lecteur ou du point de montage à partir de laquelle/duquel le volume est accessible sur l’ordinateur hôte. |

## <a name="delete-a-volume"></a>Suppression d’un volume
Appliquez la procédure suivante pour supprimer un volume du Gestionnaire d’instantanés StorSimple.

> [!NOTE]
> Vous ne pouvez pas supprimer un volume qui fait partie d’un groupe de volumes. (L’option de suppression n’est pas disponible pour les volumes membres d’un groupe de volumes.) Pour supprimer le volume, vous devez supprimer l’intégralité du groupe de volumes.

#### <a name="to-delete-a-volume"></a>Pour supprimer un volume
1. Cliquez sur l’icône de bureau pour démarrer le Gestionnaire d’instantanés StorSimple.
2. Dans le volet **Étendue**, cliquez sur le nœud **Volumes**. 
3. Dans le volet **Résultats** , cliquez avec le bouton droit sur le volume que vous souhaitez supprimer.
4. Dans le menu, cliquez sur **Supprimer**. 
   
    ![Suppression d’un volume](./media/storsimple-snapshot-manager-manage-volumes/HCS_SSM_Delete_volume.png) 
5. La boîte de dialogue **Supprimer le volume** s’affiche. Saisissez **Confirmer** dans la zone de texte, puis cliquez sur **OK**.
6. Par défaut, le Gestionnaire d’instantanés StorSimple sauvegarde un volume avant de le supprimer. Cette précaution peut vous protéger contre toute perte de données occasionnée par une suppression non intentionnelle. Le Gestionnaire d’instantanés StorSimple affiche un message de progression **Instantané automatique** pendant la sauvegarde du volume. 
   
    ![Message automatique d’instantané](./media/storsimple-snapshot-manager-manage-volumes/HCS_SSM_Automatic_snap.png) 

## <a name="rescan-volumes"></a>Relancer l’analyse des volumes
Appliquer la procédure suivante pour relancer l’analyse des volumes connectés au Gestionnaire d’instantanés StorSimple.

#### <a name="to-rescan-the-volumes"></a>Pour relancer l’analyse des volumes
1. Cliquez sur l’icône de bureau pour démarrer le Gestionnaire d’instantanés StorSimple.
2. Dans le volet **Étendue**, cliquez avec le bouton droit sur **Volumes**, puis cliquez sur **Relancer l’analyse des volumes**.
   
    ![Relancer l’analyse des volumes](./media/storsimple-snapshot-manager-manage-volumes/HCS_SSM_Rescan_volumes.png)
   
    Cette procédure synchronise la liste des volumes avec le Gestionnaire d’instantanés StorSimple. Toute modification, comme l’ajout ou la suppression de volumes, apparaîtra dans les résultats.

## <a name="configure-and-back-up-a-basic-volume"></a>Configurer et sauvegarder un volume de base
Appliquez la procédure suivante pour configurer la sauvegarde d’un volume de base, puis démarrer immédiatement la sauvegarde ou créer une stratégie de planification des sauvegardes.

### <a name="prerequisites"></a>Prérequis
Avant de commencer :

* Assurez-vous que l’appareil StorSimple et l’ordinateur hôte sont correctement configurés. Pour plus d’informations, accédez à la page [Déployer votre appareil local StorSimple](./storsimple-8000-deployment-walkthrough-u2.md).
* Installez et configurez le Gestionnaire d’instantanés StorSimple. Pour plus d’informations, accédez à la section [Déployer le Gestionnaire d’instantanés StorSimple](storsimple-snapshot-manager-deployment.md).

#### <a name="to-configure-backup-of-a-basic-volume"></a>Pour configurer la sauvegarde d’un volume de base
1. Créez un volume de base sur l’appareil StorSimple.
2. Montez, initialisez et formatez le volume, tel que décrit dans la section [Monter les volumes](#mount-volumes). 
3. Cliquez sur l’icône du Gestionnaire d’instantanés StorSimple de votre bureau. La fenêtre du Gestionnaire d’instantanés StorSimple apparaît. 
4. Dans le volet **Étendue** cliquez avec le bouton droit sur le nœud **Volumes**, puis sélectionnez **Relancer l’analyse des volumes**. À l’issue de l’analyse, une liste de volumes doit apparaître dans le volet **Résultats** . 
5. Dans le volet **Résultats**, cliquez avec le bouton droit sur le volume, puis sélectionnez **Créer un groupe de volumes**. 
   
    ![Créer un groupe de volumes](./media/storsimple-snapshot-manager-manage-volumes/HCS_SSM_Create_volume_group.png) 
6. Dans la boîte de dialogues **Créer un groupe de volumes**, entrez un nom pour le groupe de volumes, attribuez-lui des volumes, puis cliquez sur **OK**.
7. Dans le volet **Étendue**, développez le nœud **Groupes de volumes**. Le nouveau groupe de volumes doit apparaître sous le nœud **Groupes de volumes** . 
8. Cliquez avec le bouton droit sur le nom du groupe de volumes.
   
   * Pour démarrer une tâche de sauvegarde interactive (à la demande), cliquez sur **Démarrer la sauvegarde**. 
   * Pour planifier une sauvegarde automatique, cliquez sur **Créer une stratégie de sauvegarde**. Sur la page **Général** , sélectionnez un groupe de volumes dans la liste. Sur la page **Planifier** , entrez les détails de la planification. Quand vous avez terminé, cliquez sur **OK**. 
9. Pour vérifier que la tâche de sauvegarde a bien démarré, développez le nœud **Tâches** du volet **Étendue**, puis cliquez sur le nœud **En cours d’exécution**. La liste des tâches en cours apparaît dans le volet **Résultats** . 

## <a name="configure-and-back-up-a-dynamic-mirrored-volume"></a>Configurer et sauvegarder un volume dynamique mis en miroir
Procédez comme suit pour configurer la sauvegarde d’un volume dynamique mis en miroir :

* Étape 1 : Utiliser la gestion des disques pour créer un volume dynamique en miroir. 
* Étape 2 : Utiliser le Gestionnaire d’instantanés StorSimple pour configurer la sauvegarde.

### <a name="prerequisites"></a>Prérequis
Avant de commencer :

* Assurez-vous que l’appareil StorSimple et l’ordinateur hôte sont correctement configurés. Pour plus d’informations, accédez à la page [Déployer votre appareil local StorSimple](storsimple-8000-deployment-walkthrough-u2.md).
* Installez et configurez le Gestionnaire d’instantanés StorSimple. Pour plus d’informations, accédez à la section [Déployer le Gestionnaire d’instantanés StorSimple](storsimple-snapshot-manager-deployment.md).
* Configurez deux volumes sur l’appareil StorSimple. (Dans les exemples, les volumes disponibles sont **Disque 1** et **Disque 2**.) 

### <a name="step-1-use-disk-management-to-create-a-dynamic-mirrored-volume"></a>Étape 1 : Utiliser la gestion des disques pour créer un volume dynamique en miroir
Gestion des disques est un utilitaire système dédié à la gestion des disques durs et des volumes et partitions qu’ils contiennent. Pour plus d’informations sur le composant Gestion des disques, accédez à la section [Gestion des disques](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc770943(v=ws.11)) du site web Microsoft TechNet.

#### <a name="to-create-a-dynamic-mirrored-volume"></a>Pour créer un volume dynamique en miroir
1. Pour démarrer le composant Gestion des disques, utilisez l’une des options suivantes : 
   
   * Ouvrez la zone **Exécuter**, entrez **Diskmgmt.msc**, puis appuyez sur Entrée.
   * Lancez le Gestionnaire de serveur, développez le nœud **Stockage**, puis sélectionnez **Gestion des disques**. 
   * Lancez **Outils d’administration**, développez le nœud **Gestion de l’ordinateur**, puis sélectionnez **Gestion des disques**. 
2. Assurez-vous de disposer de deux volumes disponibles sur l’appareil StorSimple. (Dans l’exemple, les volumes disponibles sont **Disque 1** et **Disque 2**.) 
3. Dans la fenêtre Gestion des disques, dans la colonne de droite du volet inférieur, cliquez avec le bouton droit sur **Disque 1**, puis sélectionnez **Nouveau volume en miroir**. 
   
    ![Nouveau volume en miroir](./media/storsimple-snapshot-manager-manage-volumes/HCS_SSM_New_mirrored_volume.png) 
4. Sur la page de l’Assistant **Nouveau volume en miroir**, cliquez sur **Suivant**.
5. Sur la page **Sélectionner les disques**, choisissez **Disque 2** dans le volet **Sélectionné**, cliquez sur **Ajouter**, puis cliquez sur **Suivant**. 
6. Sur la page **Attribuer une lettre ou un chemin d’accès au lecteur**, acceptez les valeurs par défaut, puis cliquez sur **Suivant**. 
7. Sur la page **Formater le volume**, dans la zone **Taille d’unité d’allocation**, sélectionnez **64 Ko**. Sélectionnez la zone **Effectuer un formatage rapide**, puis cliquez sur **Suivant**. 
8. Sur la page **Exécution du nouveau volume en miroir**, passez en revue vos paramètres, puis cliquez sur **Terminer**. 
9. Un message s’affiche pour signaler que le disque de base sera converti en disque dynamique. Cliquez sur **Oui**.
   
    ![Message de conversion de disque dynamique](./media/storsimple-snapshot-manager-manage-volumes/HCS_SSM_Disk_management_msg.png) 
10. Dans Gestion des disques, vérifiez que Disque 1 et Disque 2 sont affichés comme des volumes en miroir dynamiques. (**Dynamique** doit apparaître dans la colonne d’état et la couleur de la barre de capacité doit passer au rouge, indiquant un volume en miroir.) 
    
    ![Disques dynamiques mis en miroir du composant Gestion des disques](./media/storsimple-snapshot-manager-manage-volumes/HCS_SSM_Verify_dynamic_disks_2.png) 

### <a name="step-2-use-storsimple-snapshot-manager-to-configure-backup"></a>Étape 2 : Utiliser le Gestionnaire d’instantanés StorSimple pour configurer la sauvegarde
Appliquez la procédure suivante pour configurer un volume dynamique mis en miroir, puis commencer immédiatement une sauvegarde ou créer une stratégie dédiée aux sauvegardes planifiées.

#### <a name="to-configure-backup-of-a-dynamic-mirrored-volume"></a>Pour configurer la sauvegarde d’un volume dynamique mis en miroir
1. Cliquez sur l’icône du Gestionnaire d’instantanés StorSimple de votre bureau. La fenêtre du Gestionnaire d’instantanés StorSimple apparaît. 
2. Dans le volet **Étendue** cliquez avec le bouton droit sur le nœud **Volumes**, puis sélectionnez **Relancer l’analyse des volumes**. À l’issue de l’analyse, une liste de volumes doit apparaître dans le volet **Résultats** . Le volume dynamique mis en miroir est répertorié en tant que volume unique. 
3. Dans le volet **Résultats**, cliquez avec le bouton droit sur le volume dynamique en miroir, puis cliquez sur **Créer un groupe de volumes**. 
4. Dans la boîte de dialogue **Créer un groupe de volumes**, saisissez un nom pour le groupe de volumes, attribuez le volume dynamique en miroir à ce groupe, puis cliquez sur **OK**. 
5. Dans le volet **Étendue**, développez le nœud **Groupes de volumes**. Le nouveau groupe de volumes doit apparaître sous le nœud **Groupes de volumes**. 
6. Cliquez avec le bouton droit sur le nom du groupe de volumes. 
   
   * Pour démarrer une tâche de sauvegarde interactive (à la demande), cliquez sur **Démarrer la sauvegarde**. 
   * Pour planifier une sauvegarde automatique, cliquez sur **Créer une stratégie de sauvegarde**. Sur la page **Général** , sélectionnez le groupe de volumes dans la liste. Sur la page **Planifier** , entrez les détails de la planification. Quand vous avez terminé, cliquez sur **OK**. 
7. Vous pouvez surveiller la tâche de sauvegarde en cours d’exécution. Dans le volet **Étendue**, développez le nœud **Tâches**, puis cliquez sur **En cours d’exécution** ; les détails de la tâche apparaissent dans le volet **Résultats**. À l’issue de la tâche de sauvegarde, les détails sont transférés vers la liste des tâches **Dernières 24 heures** . 

## <a name="next-steps"></a>Étapes suivantes
* Découvrez comment [utiliser le Gestionnaire d’instantanés StorSimple pour gérer votre solution StorSimple](storsimple-snapshot-manager-admin.md).
* Découvrez comment [utiliser le Gestionnaire d’instantanés StorSimple pour créer et gérer des groupes de volumes](storsimple-snapshot-manager-manage-volume-groups.md).

<!--Reference links-->
[1]: /previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/ee338480(v=ws.10)