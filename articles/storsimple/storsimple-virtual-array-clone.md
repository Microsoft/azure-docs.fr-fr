---
title: Cloner une sauvegarde StorSimple Virtual Array | Microsoft Docs
description: Découvrez comment cloner une sauvegarde et récupérer un fichier à partir de votre instance StorSimple Virtual Array.
services: storsimple
documentationcenter: NA
author: alkohli
manager: carmonm
editor: ''
ms.assetid: af6e979c-55e3-477c-b53e-a76a697f80c9
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/21/2016
ms.author: alkohli
ms.openlocfilehash: b6cc89cb082cd0ed32abd88e3a6683c60a27ba90
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96023038"
---
# <a name="clone-from-a-backup-of-your-storsimple-virtual-array"></a>Cloner à partir d’une sauvegarde de votre instance StorSimple Virtual Array

## <a name="overview"></a>Vue d’ensemble

Cet article décrit étape par étape comment cloner un jeu de sauvegarde de vos partages ou volumes sur votre instance Microsoft Azure StorSimple Virtual Array. La sauvegarde clonée est utilisée pour récupérer un fichier supprimé ou perdu. L'article inclut également une procédure détaillée pour effectuer une récupération au niveau de l’élément sur votre instance StorSimple Virtual Array configurée comme un serveur de fichiers.

## <a name="clone-shares-from-a-backup-set"></a>Cloner des partages à partir d’un jeu de sauvegarde

**Avant d'essayer de cloner des partages, assurez-vous de disposer de suffisamment d'espace sur l’appareil pour terminer cette opération.** Pour cloner à partir d’une sauvegarde, procédez comme suit dans le [portail Azure](https://portal.azure.com/).

#### <a name="to-clone-a-share"></a>Clonage d’un partage

1. Accédez au panneau **Appareils**. Sélectionnez et cliquez sur votre appareil, puis cliquez sur **Partages**. Sélectionnez le partage que vous souhaitez cloner, cliquez avec le bouton droit sur le partage pour appeler le menu contextuel. Sélectionnez **Cloner**.
   
   ![Cloner une sauvegarde](./media/storsimple-virtual-array-clone/cloneshare1.png)
2. Dans le panneau **Cloner**, cliquez sur **Sauvegarde > Sélectionner**, puis procédez comme suit : 
   
   a.    Filtrez une sauvegarde sur cet appareil en fonction de l’intervalle de temps. Vous pouvez choisir **7 derniers jours**, **30 derniers jours** ou **Dernière année**.
   
   b.    Dans la liste des sauvegardes filtrées affichées, sélectionnez une sauvegarde à partir de laquelle effectuer le clonage.
   
   c.    Cliquez sur **OK**.
   
   ![Cloner une sauvegarde 2](./media/storsimple-virtual-array-clone/cloneshare3.png)
3. Dans le panneau **Cloner**, cliquez sur **Paramètres de la cible**, puis procédez comme suit :
   
   a.    Nommez le partage. Le nom du partage doit contenir entre 3 et 127 caractères.
   
   b.    Vous pouvez fournir une description facultative du partage cloné.
   
   c.    Vous ne pouvez pas modifier le type du partage vers lequel vous effectuez la restauration. Un partage à plusieurs niveaux est cloné comme un partage à plusieurs niveaux, et un partage épinglé localement comme un partage épinglé localement.
   
   d.    La capacité est définie comme étant égale à la taille du partage à partir duquel vous effectuez le clonage.
   
   e.    Affectez les administrateurs pour ce partage. Vous pourrez modifier les propriétés du partage via l'Explorateur de fichiers une fois le clonage terminé.
   
   f.    Cliquez sur **OK**.
   
   ![Cloner une sauvegarde 3](./media/storsimple-virtual-array-clone/cloneshare6.png)

4. Cliquez sur **Cloner** pour démarrer le clonage. Une fois la tâche terminée, l’opération de clonage démarre et vous recevez une notification. Pour surveiller la progression du clone, accédez au panneau **Tâches** et cliquez sur la tâche pour afficher les détails de la tâche.
5. Une fois le clone créé, revenez au panneau **Partages** sur votre appareil.
6. Vous pouvez maintenant afficher le nouveau partage cloné dans la liste des partages sur votre appareil. Un partage à plusieurs niveaux est cloné comme un partage à plusieurs niveaux, et un partage épinglé localement comme un partage épinglé localement.
   
   ![Cloner une sauvegarde 4](./media/storsimple-virtual-array-clone/cloneshare10.png)

## <a name="clone-volumes-from-a-backup-set"></a>Cloner des volumes à partir d’un jeu de sauvegarde

Pour effectuer un clonage à partir d’une sauvegarde, dans le portail Azure, vous devez effectuer les étapes similaires à celles effectuées lors du clonage d’un partage. L'opération de clonage clone la sauvegarde vers un nouveau volume sur le même appareil virtuel ; vous ne pouvez pas cloner vers un autre appareil.

#### <a name="to-clone-a-volume"></a>Cloner un volume

1. Accédez au panneau **Appareils**. Sélectionnez et cliquez sur votre appareil, puis cliquez sur **Volumes**. Sélectionnez le volume que vous souhaitez cloner, cliquez avec le bouton droit sur le volume pour appeler le menu contextuel. Sélectionnez **Cloner**.
   
   ![Cloner un volume](./media/storsimple-virtual-array-clone/clonevolume1.png)
2. Dans le panneau **Cloner**, cliquez sur **Sauvegarde**, puis procédez comme suit : 
   
   a.    Filtrez une sauvegarde sur cet appareil en fonction de l’intervalle de temps. Vous pouvez choisir **7 derniers jours**, **30 derniers jours** ou **Dernière année**. 
   
   b.    Dans la liste des sauvegardes filtrées affichées, sélectionnez une sauvegarde à partir de laquelle effectuer le clonage.
   
   c.    Cliquez sur **OK**.
   
   ![Cloner un volume 2](./media/storsimple-virtual-array-clone/clonevolume3.png)
3. Dans le panneau **Cloner**, cliquez sur **Paramètres du volume cible**, puis procédez comme suit :
   
   a. Le nom de l’appareil est automatiquement renseigné.
   
   b. Fournissez un nom de volume pour le **volume cloné**. Le nom du volume doit contenir entre 3 et 127 caractères.
   
   c. Le type de volume est défini automatiquement sur le volume d’origine. Un volume à plusieurs niveaux est cloné comme un partage à plusieurs niveaux, et un volume épinglé localement comme un volume épinglé localement.
   
   d. En regard de **Hôtes connectés**, cliquez sur **Sélectionner**.
   
   ![Cloner un volume 3](./media/storsimple-virtual-array-clone/clonevolume4.png)
4. Dans le panneau **Hôtes connectés**, sélectionnez un ACR existant ou ajoutez un nouvel ACR. Pour ajouter un nouvel ACR, vous devez fournir un nom d’ACR et l’IQN hôte. Cliquez sur **Sélectionner**.
   
   ![Cloner un volume 4](./media/storsimple-virtual-array-clone/clonevolume5.png)
5. Cliquez sur **Cloner** pour lancer le clonage.
   
   ![Cloner un volume 5](./media/storsimple-virtual-array-clone/clonevolume6.png)  
6. Une fois la tâche de clonage créée, le clonage démarre. Une fois le clone créé, il est affiché dans le panneau Volumes sur votre appareil. Remarque : un volume à plusieurs niveaux est cloné comme un volume à plusieurs niveaux, et un volume épinglé localement comme un volume épinglé localement.
   
   ![Cloner un volume 6](./media/storsimple-virtual-array-clone/clonevolume8.png)
7. Une fois que le volume apparaît en ligne dans la liste des volumes, il est disponible pour utilisation. Sur l’hôte de l’initiateur iSCSI, actualisez la liste des cibles dans la fenêtre des propriétés de l’initiateur iSCSI. Une nouvelle cible qui contient le nom du volume cloné doit s’afficher comme « inactive » dans la colonne d’état.
8. Sélectionnez la cible et cliquez sur **Se connecter**. Une fois l’initiateur connecté à la cible, son état doit indiquer **Connecté**.
9. Dans la fenêtre **Gestion des disques** , les volumes montés s’affichent comme indiqué dans l’illustration suivante. Cliquez avec le bouton droit sur le volume détecté (cliquez sur le nom du disque), puis cliquez sur **En ligne**.

> [!IMPORTANT]
> Lorsque vous tentez de cloner un volume ou un partage à partir d’un jeu de sauvegarde, si la tâche de clonage échoue, un volume ou partage cible peut toujours être créé dans le portail. Il est important de supprimer ce volume ou partage cible dans le portail pour réduire les problèmes futurs découlant de cet élément.
> 
> 

## <a name="item-level-recovery-ilr"></a>Récupération au niveau de l'élément (ILR)

Cette version présente la récupération au niveau de l’élément (ILR) sur une instance StorSimple Virtual Array configurée comme un serveur de fichiers. Cette fonctionnalité vous permet d'effectuer une récupération granulaire de fichiers et de dossiers à partir d'une sauvegarde cloud contenant tous les partages sur l'appareil StorSimple. Grâce à un modèle en libre-service, vous pouvez récupérer des fichiers supprimés à partir de sauvegardes récentes.

Chaque partage comporte un dossier *.backups* contenant les sauvegardes les plus récentes. Vous pouvez naviguer vers la sauvegarde souhaitée, copier les fichiers et dossiers correspondants à partir de la sauvegarde, puis les restaurer. Cette fonctionnalité évite de contacter les administrateurs pour restaurer des fichiers à partir de sauvegardes.

1. Lorsque vous effectuez la récupération ILR, vous pouvez afficher les sauvegardes dans l'Explorateur de fichiers. Cliquez sur le partage spécifique pour lequel vous souhaitez afficher la sauvegarde. Sous le partage, vous verrez un dossier *.backups* contenant toutes les sauvegardes. Développez le dossier *.backups* pour afficher les sauvegardes. Le dossier affiche la vue éclatée de toute la hiérarchie de sauvegarde. La création de cette vue s’effectue à la demande et ne prend que quelques secondes.
   
   Les cinq dernières sauvegardes s’affichent de cette façon et peuvent être utilisées pour effectuer une récupération au niveau de l’élément. Les cinq sauvegardes récentes incluent les sauvegardes planifiées par défaut et les sauvegardes manuelles.
   
   * **Sauvegardes planifiées** nommées &lt;Nom de l’appareil&gt;DailySchedule-YYYYMMDD-HHMMSS-UTC.
   * **Sauvegardes manuelles** nommées Ad-hoc-YYYYMMDD-HHMMSS-UTC.
     
     ![Capture d’écran de l’Explorateur de fichiers qui présente le dossier .backups. À l’intérieur de ce dossier, le dossier Sauvegardes manuelles est sélectionné.](./media/storsimple-virtual-array-clone/image14.png)

2. Identifiez la sauvegarde contenant la version la plus récente du fichier supprimé. Bien que le nom du dossier contienne un horodatage UTC dans chacun des cas précédents, l'heure de création du dossier correspond à l’heure réelle de démarrage de la sauvegarde sur l’appareil. Utilisez l'horodatage du dossier pour localiser et identifier les sauvegardes.

3. Recherchez le dossier ou fichier que vous voulez restaurer dans la sauvegarde que vous avez identifiée à l'étape précédente. Notez que vous pouvez uniquement afficher les fichiers ou dossiers pour lesquels vous disposez d'autorisations. Si vous ne pouvez pas accéder à certains fichiers ou dossiers, contactez un administrateur de partage. L’administrateur peut utiliser l’Explorateur de fichiers pour modifier les autorisations de partage et vous permettre d’accéder au fichier ou au dossier en question. Il est recommandé que l'administrateur de partage soit un groupe d'utilisateurs plutôt qu'un utilisateur unique.

4. Copiez le fichier ou le dossier dans le partage approprié sur votre serveur de fichiers StorSimple.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur la [gestion de StorSimple Virtual Array à l’aide de l’interface utilisateur web locale](storsimple-ova-web-ui-admin.md).

