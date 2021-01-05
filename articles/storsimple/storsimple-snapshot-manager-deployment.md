---
title: Déploiement du Gestionnaire d’instantanés StorSimple | Microsoft Docs
description: Découvrez comment télécharger et installer le Gestionnaire d'instantanés StorSimple, un composant logiciel enfichable MMC pour la gestion des fonctionnalités de sauvegarde et de protection de données StorSimple.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: f0128f57-519e-49ec-9187-23575809cdbe
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 06/05/2017
ms.author: alkohli
ms.openlocfilehash: 5f94564c5e769d3cf0e0abbe92a309a1ee2117a1
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96003869"
---
# <a name="deploy-the-storsimple-snapshot-manager-mmc-snap-in"></a>Déployer le composant logiciel enfichable MMC Gestionnaire d’instantanés StorSimple

## <a name="overview"></a>Vue d’ensemble
Le Gestionnaire d’instantanés StorSimple est un composant logiciel enfichable MMC (Microsoft Management Console) qui simplifie la protection des données et la gestion des sauvegardes dans un environnement Microsoft Azure StorSimple. Avec le Gestionnaire d’instantanés StorSimple, vous pouvez gérer le stockage Microsoft Azure StorSimple local et dans le cloud comme s’il s’agissait d’un système de stockage entièrement intégré, ce qui simplifie les processus de sauvegarde et de restauration et contribue à réduire les coûts. 

Ce didacticiel décrit la configuration requise, ainsi que les procédures d’installation, de suppression et de mise à niveau du Gestionnaire d’instantanés StorSimple.

> [!NOTE]
> * Vous ne pouvez pas utiliser le Gestionnaire d'instantanés StorSimple pour gérer les baies virtuelles Microsoft Azure StorSimple (également appelées appareils virtuels locaux StorSimple).
> * Si vous prévoyez d’installer StorSimple Update 2 sur votre appareil StorSimple, veillez à télécharger la dernière version de StorSimple Snapshot Manager et à l’installer **avant d’installer StorSimple Update 2**. La dernière version de StorSimple Snapshot Manager est à compatibilité descendante et fonctionne avec toutes les versions publiées de Microsoft Azure StorSimple. Si vous utilisez la version précédente de StorSimple Snapshot Manager, vous devez effectuer une mise à jour (il est inutile de désinstaller la version précédente avant d’installer la nouvelle).


## <a name="storsimple-snapshot-manager-installation"></a>Installation du Gestionnaire d’instantanés StorSimple
StorSimple Snapshot Manager peut être installé sur les ordinateurs exécutant le système d’exploitation Windows Server 2008 R2 SP1, Windows Server 2012 ou Windows Server 2012 R2. Sur les serveurs exécutant Windows 2008 R2, vous devez aussi installer Windows Server 2008 SP1 et Windows Management Framework 3.0.

Avant d’installer ou de mettre à niveau le composant logiciel enfichable Gestionnaire d’instantanés StorSimple pour la console MMC (Microsoft Management Console), assurez-vous que le serveur hôte et l’appareil Microsoft Azure StorSimple sont configurés correctement.

## <a name="configure-prerequisites"></a>Configuration préalable requise
Les étapes suivantes fournissent une vue d’ensemble des tâches de configuration requises avant d’installer le Gestionnaire d’instantanés StorSimple. Pour obtenir des informations complètes sur la configuration et l’installation de Microsoft Azure StorSimple, y compris la configuration système requise et des instructions pas à pas, consultez la page [Déploiement de votre appareil StorSimple local](storsimple-8000-deployment-walkthrough-u2.md).

> [!IMPORTANT]
> Avant de commencer, lisez la [liste de contrôle de déploiement](storsimple-8000-deployment-walkthrough-u2.md#deployment-configuration-checklist) et les [conditions préalables au déploiement](storsimple-8000-deployment-walkthrough-u2.md#deployment-prerequisites) dans l’article [Déploiement de votre appareil StorSimple local](storsimple-8000-deployment-walkthrough-u2.md).
> <br>
> 
> 

### <a name="before-you-install-storsimple-snapshot-manager"></a>Avant d’installer le Gestionnaire d’instantanés StorSimple
1. Déballez, montez et connectez l’appareil Microsoft Azure StorSimple en suivant les instructions de l’article [Installation de votre appareil StorSimple 8100](storsimple-8100-hardware-installation.md) ou [Installation de votre appareil StorSimple 8600](storsimple-8600-hardware-installation.md).
2. Assurez-vous que votre ordinateur hôte exécute l’un des systèmes d’exploitation suivants :
   
   * Windows Server 2008 R2 (sur les serveurs exécutant Windows 2008 R2, vous devez aussi installer Windows Server 2008 SP1 et Windows Management Framework 3.0)
   * Windows Server 2012
   * Windows Server 2012 R2
     
     Pour un appareil virtuel StorSimple, l’hôte doit être une machine virtuelle Microsoft Azure.
3. Assurez-vous que la configuration requise de Microsoft Azure StorSimple est entièrement respectée. Pour plus d’informations, consultez la page [Conditions préalables au déploiement](storsimple-8000-deployment-walkthrough-u2.md#deployment-prerequisites).
4. Connectez l’appareil à l’hôte et procédez à la configuration initiale. Pour plus d’informations, consultez la page [Étapes du déploiement](storsimple-8000-deployment-walkthrough-u2.md#deployment-steps).
5. Créez des volumes sur l’appareil, affectez-les à l’hôte et vérifiez que l’hôte peut les monter et les utiliser. Le Gestionnaire d’instantanés StorSimple prend en charge les types de volumes suivants :
   
   * Volumes de base
   * Volumes simples
   * Volumes dynamiques
   * Volumes dynamiques en miroir (RAID 1)
   * Volumes partagés de cluster
     
     Pour plus d’informations sur la création de volumes sur l’appareil StorSimple ou l’appareil virtuel StorSimple, consultez la section [« Étape 6 : création d’un volume »](storsimple-8000-deployment-walkthrough-u2.md#step-6-create-a-volume) de l’article [Déploiement de votre appareil StorSimple local](storsimple-8000-deployment-walkthrough-u2.md).

## <a name="install-a-new-storsimple-snapshot-manager"></a>Installation d’un nouveau Gestionnaire d’instantanés StorSimple
Avant d’installer le Gestionnaire d’instantanés StorSimple, assurez-vous que les volumes que vous avez créés sur l’appareil StorSimple ou l’appareil virtuel StorSimple sont montés, initialisés et formatés comme indiqué dans la section [Configuration préalable requise](#configure-prerequisites).

> [!IMPORTANT]
> * Pour un appareil virtuel StorSimple, l’hôte doit être une machine virtuelle Microsoft Azure. 
> * L’hôte doit exécuter Windows 2008 R2, Windows Server 2012 ou Windows Server 2012 R2. Si votre serveur exécute Windows Server 2008 R2, vous devez aussi installer Windows Server 2008 SP1 et Windows Management Framework 3.0.
> * Vous devez configurer une connexion iSCSI de l’hôte vers l’appareil StorSimple avant de pouvoir connecter l’appareil au Gestionnaire d’instantanés StorSimple.

Pour effectuer une nouvelle installation du Gestionnaire d’instantanés StorSimple, suivez les étapes ci-dessous. Si vous installez une mise à niveau, accédez à la section [Mise à niveau ou réinstallation du Gestionnaire d’instantanés StorSimple](#upgrade-or-reinstall-storsimple-snapshot-manager).

* Étape 1 : installation du Gestionnaire d’instantanés StorSimple 
* Étape 2 : connexion du Gestionnaire d’instantanés StorSimple à un appareil 
* Étape 3 : vérification de la connexion à l’appareil 

### <a name="step-1-install-storsimple-snapshot-manager"></a>Étape 1 : installation du Gestionnaire d’instantanés StorSimple
Pour installer le Gestionnaire d’instantanés StorSimple, procédez comme suit.

#### <a name="to-install-storsimple-snapshot-manager"></a>Pour installer le Gestionnaire d’instantanés StorSimple
1. Téléchargez le logiciel Gestionnaire d’instantanés StorSimple (accédez à [Gestionnaire d’instantanés StorSimple](https://www.microsoft.com/download/details.aspx?id=44220) dans le Centre de téléchargement Microsoft) et enregistrez-le localement sur l’hôte.
2. Dans l’Explorateur de fichiers, cliquez avec le bouton droit sur le dossier compressé, puis cliquez sur **Extraire tout**.
3. Dans la fenêtre **Extraire les dossiers compressés**, dans la zone **Sélectionner une destination et extraire les fichiers**, tapez ou recherchez le chemin d’accès où extraire le fichier.
   
    > [!IMPORTANT]
    > Vous devez installer le Gestionnaire d’instantanés StorSimple sur le lecteur C:.
    
4. Cochez la case **Afficher les dossiers extraits une fois l’opération terminée**, puis cliquez sur **Extraire**.
   
    ![Boîte de dialogue Extraire les fichiers](./media/storsimple-snapshot-manager-deployment/HCS_SSM_extract_files.png) 
5. Une fois l’extraction terminée, le dossier de destination s’ouvre. Double-cliquez sur l’icône d’installation de l’application présente dans le dossier de destination.
6. Quand le message **Installation/désinstallation réussie** s’affiche, cliquez sur **Fermer**. L’icône du Gestionnaire d’instantanés StorSimple doit être affichée sur votre bureau.
   
    ![Icône du Bureau](./media/storsimple-snapshot-manager-deployment/HCS_SSM_desktop_icon.png) 

### <a name="step-2-connect-storsimple-snapshot-manager-to-a-device"></a>Étape 2 : connexion du Gestionnaire d’instantanés StorSimple à un appareil
Pour connecter le Gestionnaire d’instantanés StorSimple à un appareil StorSimple, procédez comme suit.

#### <a name="to-connect-storsimple-snapshot-manager-to-a-device"></a>Pour connecter le Gestionnaire d’instantanés StorSimple à un appareil
1. Cliquez sur l’icône du Gestionnaire d’instantanés StorSimple de votre bureau. La fenêtre du Gestionnaire d’instantanés StorSimple apparaît. Cette fenêtre contient un volet **Étendue**, un volet **Résultats** et un volet **Actions**. 
   
    ![Interface utilisateur du gestionnaire d’instantanés StorSimple](./media/storsimple-snapshot-manager-deployment/HCS_SSM_gui_panes.png)
   
   * Le volet **Étendue** (volet gauche) contient une liste de nœuds organisée en arborescence. Vous pouvez développer certains nœuds pour sélectionner une vue ou des données spécifiques relatives à ce nœud. Cliquez sur l’icône en forme de flèche en regard d’un nœud pour le développer ou le réduire. Cliquez avec le bouton droit sur un élément du volet **Étendue** pour afficher la liste des actions disponibles pour cet élément.
   * Le volet **Résultats** (volet central) contient des informations d’état détaillées sur le nœud, la vue ou les données que vous avez sélectionnés dans le volet **Étendue**.
   * Le volet **Actions** répertorie les opérations que vous pouvez effectuer sur le nœud, la vue ou les données que vous avez sélectionnés dans le volet **Étendue**.
     
     Pour une description complète de l’interface utilisateur du Gestionnaire d’instantanés StorSimple, consultez l’article [Interface utilisateur du Gestionnaire d’instantanés StorSimple](storsimple-use-snapshot-manager.md).
2. Dans le volet **Étendue**, cliquez avec le bouton droit sur le nœud **Appareils**, puis cliquez sur **Configurer un appareil**. La boîte de dialogue **Configurer un appareil** apparaît.
   
    ![Configurer un appareil](./media/storsimple-snapshot-manager-deployment/HCS_SSM_config_device.png) 
3. Dans la zone de liste **Appareil** , sélectionnez l’adresse IP de l’appareil ou de l’appareil virtuel Microsoft Azure StorSimple. Dans la zone de texte **Mot de passe** , entrez le mot de passe du Gestionnaire d’instantanés StorSimple que vous avez créé pour l’appareil dans le portail Azure. Cliquez sur **OK**.
4. Le Gestionnaire d’instantanés StorSimple recherche l’appareil que vous avez identifié. Si l’appareil est disponible, le Gestionnaire d’instantanés StorSimple ajoute une connexion. Vous pouvez [vérifier la connexion à l’appareil](#to-verify-the-connection) pour confirmer que la connexion a bien été ajoutée.
   
    Si, pour une raison quelconque, l’appareil est indisponible, le Gestionnaire d’instantanés StorSimple renvoie un message d’erreur. Cliquez sur **OK** pour fermer le message d’erreur, puis cliquez sur **Annuler** pour fermer la boîte de dialogue **Configurer un appareil**.
5. Au moment de se connecter à un appareil, le Gestionnaire d’instantanés StorSimple importe chaque groupe de volumes configuré pour cet appareil, à condition que le groupe de volumes soit associé à des sauvegardes. Les groupes de volumes qui ne sont associés à aucune sauvegarde ne sont pas importés. De plus, les stratégies de sauvegarde créées pour un groupe de volumes ne sont pas importées. Pour afficher les groupes importés, cliquez avec le bouton droit sur le nœud **Groupes de volumes** situé le plus haut dans le volet **Étendue**, puis cliquez sur **Activer/Désactiver les groupes importés**.

### <a name="step-3-verify-the-connection-to-the-device"></a>Étape 3 : vérification de la connexion à l’appareil
Pour vérifier que le Gestionnaire d’instantanés StorSimple est connecté à l’appareil StorSimple, procédez comme suit.

#### <a name="to-verify-the-connection"></a>Pour vérifier la connexion
1. Dans le volet **Étendue**, cliquez sur le nœud **Appareils**.
   
    ![État de l’appareil dans le Gestionnaire d’instantanés StorSimple](./media/storsimple-snapshot-manager-deployment/HCS_SSM_Device_status.png) 
2. Vérifiez le volet **Résultats** : 
   
   * Si un indicateur vert est affiché sur l’icône de l’appareil et **Disponible** est indiqué dans la colonne **État**, cela signifie que l’appareil est connecté. 
   * Si un indicateur rouge est affiché sur l’icône de l’appareil et Non disponible est indiqué dans la colonne **État** , cela signifie que l’appareil n’est pas connecté. 
   * Si **Actualisation** s’affiche dans la colonne **État**, cela signifie que le Gestionnaire d’instantanés StorSimple récupère les groupes de volumes et les sauvegardes associées pour un appareil connecté.

## <a name="upgrade-or-reinstall-storsimple-snapshot-manager"></a>Mise à niveau ou réinstallation du Gestionnaire d’instantanés StorSimple
Vous devez désinstaller complètement le Gestionnaire d’instantanés StorSimple avant de mettre à niveau ou de réinstaller le logiciel. 

Avant de réinstaller le Gestionnaire d’instantanés StorSimple, sauvegardez la base de données existante du Gestionnaire d’instantanés StorSimple sur l’ordinateur hôte. Les informations de configuration et les stratégies de sauvegarde sont ainsi enregistrées pour faciliter la restauration de ces données à partir de la sauvegarde.

Pour mettre à niveau ou réinstaller le Gestionnaire d’instantanés StorSimple, suivez ces étapes :

* Étape 1 : désinstallation du Gestionnaire d’instantanés StorSimple 
* Étape 2 : sauvegarde de la base de données du Gestionnaire d’instantanés StorSimple 
* Étape 3 : réinstallation du Gestionnaire d’instantanés StorSimple et restauration de la base de données 

### <a name="step-1-uninstall-storsimple-snapshot-manager"></a>Étape 1 : désinstallation du Gestionnaire d’instantanés StorSimple
Pour désinstaller le Gestionnaire d’instantanés StorSimple, procédez comme suit.

#### <a name="to-uninstall-storsimple-snapshot-manager"></a>Pour désinstaller le Gestionnaire d’instantanés StorSimple
1. Sur l’ordinateur hôte, ouvrez le **Panneau de configuration**, cliquez sur **Programmes**, puis sur **Programmes et fonctionnalités**.
2. Dans le volet gauche, cliquez sur **Désinstaller ou modifier un programme**.
3. Cliquez avec le bouton droit sur **Gestionnaire d’instantanés StorSimple**, puis cliquez sur **Désinstaller**.
4. Le programme d’installation du Gestionnaire d’instantanés StorSimple démarre. Cliquez sur **Modifier l’installation**, puis sur **Désinstaller**.
   
   > [!NOTE]
   > Si des processus de la console MMC sont exécutés en arrière-plan, tels que le Gestionnaire d’instantanés StorSimple ou l’outil Gestion des disques, la désinstallation échoue et vous recevez un message vous invitant à fermer toutes les instances de la console MMC avant de désinstaller le programme. Sélectionnez **Fermer automatiquement les applications et tenter de les redémarrer une fois l’installation terminée**, puis cliquez sur **OK**.
   > 
   > 
5. Une fois le processus de désinstallation terminé, le message **Installation/désinstallation réussie** s’affiche. Cliquez sur **Fermer**.

### <a name="step-2-back-up-the-storsimple-snapshot-manager-database"></a>Étape 2 : sauvegarde de la base de données du Gestionnaire d’instantanés StorSimple
Pour créer et enregistrer une copie de la base de données du Gestionnaire d’instantanés StorSimple, procédez comme suit.

#### <a name="to-back-up-the-database"></a>Pour sauvegarder la base de données
1. Arrêtez le service de gestion Microsoft StorSimple :
   
   1. Démarrez le Gestionnaire de serveur.
   2. Sur le tableau de bord du Gestionnaire de serveur, dans le menu **Outils**, sélectionnez **Services**.
   3. Dans la page **Services**, sélectionnez **Service de gestion Microsoft StorSimple**.
   4. Dans le volet droit, sous **Service de gestion Microsoft StorSimple**, cliquez sur **Arrêter le service**.
      
        ![Arrêter le service StorSimple Device Manager](./media/storsimple-snapshot-manager-deployment/HCS_SSM_stop_service.png)
2. Accédez à C:\ProgramData\Microsoft\StorSimple\BACatalog. 
   
   > [!NOTE]
   > ProgramData est un dossier masqué.
  
3. Recherchez le fichier XML de catalogue, copiez le fichier et stockez la copie dans un emplacement sûr ou dans le cloud.
   
    ![Fichier de catalogue de sauvegarde StorSimple](./media/storsimple-snapshot-manager-deployment/HCS_SSM_bacatalog.png)
4. Redémarrez le service de gestion Microsoft StorSimple : 
   
   1. Sur le tableau de bord du Gestionnaire de serveur, dans le menu **Outils**, sélectionnez **Services**.
   2. Dans la page **Services**, sélectionnez **Service de gestion Microsoft StorSimple**.
   3. Dans le volet droit, sous **Service de gestion Microsoft StorSimple**, cliquez sur **Redémarrer le service**. 

### <a name="step-3-reinstall-storsimple-snapshot-manager-and-restore-the-database"></a>Étape 3 : réinstallation du Gestionnaire d’instantanés StorSimple et restauration de la base de données
Pour réinstaller le Gestionnaire d’instantanés StorSimple, suivez les étapes de la section [Installer un nouveau Gestionnaire d’instantanés StorSimple](#install-a-new-storsimple-snapshot-manager). Appliquez ensuite la procédure suivante pour restaurer la base de données du Gestionnaire d’instantanés StorSimple.

#### <a name="to-restore-the-database"></a>Pour restaurer la base de données
1. Arrêtez le service de gestion Microsoft StorSimple :
   
   1. Démarrez le Gestionnaire de serveur.
   2. Sur le tableau de bord du Gestionnaire de serveur, dans le menu **Outils**, sélectionnez **Services**.
   3. Dans la page **Services**, sélectionnez **Service de gestion Microsoft StorSimple**.
   4. Dans le volet droit, sous **Service de gestion Microsoft StorSimple**, cliquez sur **Arrêter le service**.
2. Accédez à C:\ProgramData\Microsoft\StorSimple\BACatalog.
   
   > [!NOTE]
   > ProgramData est un dossier masqué.
   > 
   > 
3. Supprimez le fichier XML de catalogue et remplacez-le par la version que vous avez enregistrée précédemment.
4. Redémarrez le service de gestion Microsoft StorSimple : 
   
   1. Sur le tableau de bord du Gestionnaire de serveur, dans le menu **Outils**, sélectionnez **Services**.
   2. Dans la page **Services**, sélectionnez **Service de gestion Microsoft StorSimple**.
   3. Dans le volet droit, sous **Service de gestion Microsoft StorSimple**, cliquez sur **Redémarrer le service**.

## <a name="next-steps"></a>Étapes suivantes
* Pour en savoir plus sur le Gestionnaire d’instantanés StorSimple, consultez [Qu’est-ce que le Gestionnaire d’instantanés StorSimple ?](storsimple-what-is-snapshot-manager.md)
* Pour en savoir plus sur l’interface utilisateur du Gestionnaire d’instantanés StorSimple, consultez [Interface utilisateur du Gestionnaire d’instantanés StorSimple](storsimple-use-snapshot-manager.md).
* Pour en savoir plus sur l’utilisation du Gestionnaire d’instantanés StorSimple, consultez [Utiliser le Gestionnaire d’instantanés StorSimple pour gérer votre solution StorSimple](storsimple-snapshot-manager-admin.md).

