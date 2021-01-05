---
title: Gérer les appareils avec le Gestionnaire d’instantanés StorSimple | Microsoft Docs
description: Décrit comment utiliser le composant logiciel enfichable MMC du Gestionnaire d’instantanés StorSimple pour connecter et gérer des appareils StorSimple.
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: 966ecbe3-a7fa-4752-825f-6694dd949946
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 06/05/2017
ms.author: alkohli
ms.openlocfilehash: dbb7e835d110bcb6cb81fd5425a2aafbb578e692
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96022970"
---
# <a name="use-storsimple-snapshot-manager-to-connect-and-manage-storsimple-devices"></a>Utiliser le Gestionnaire d’instantanés StorSimple pour connecter et gérer des appareils StorSimple
## <a name="overview"></a>Vue d’ensemble
Vous pouvez utiliser les nœuds du volet **Étendue** du Gestionnaire d’instantanés StorSimple afin de vérifier les données importées de l’appareil StorSimple et d’actualiser les appareils de stockage connectés. Par ailleurs, lorsque vous cliquez sur le nœud **Appareils**, vous pouvez consulter une liste des appareils connectés et des informations de statut correspondantes dans le volet **Résultats**.

![Appareils connectés](./media/storsimple-snapshot-manager-manage-devices/HCS_SSM_connect_devices.png)

**Figure 1 : Appareil connecté du Gestionnaire d’instantanés StorSimple** 

En fonction de vos sélections **Affichage**, le volet **Résultats** indique les informations suivantes à propos de chaque appareil. (Pour plus d’informations sur la configuration d’une vue, accédez au [Menu Affichage](storsimple-use-snapshot-manager.md#view-menu).)

| Colonne de résultats | Description |
|:--- |:--- |
| Name |Le nom de l’appareil, tel que configuré dans le portail Azure Classic |
| Modèle |Le numéro de modèle de l’appareil |
| Version |La version du logiciel installé sur l’appareil |
| Statut |La disponibilité de l’appareil |
| Dernière synchronisation |Date et heure de la dernière synchronisation de l’appareil |
| Numéro de série |Le numéro de série de l’appareil |

Si vous cliquez avec le bouton droit sur le nœud **Appareils** du volet **Étendue**, vous pouvez sélectionner les actions suivantes :

* Ajout ou remplacement d’un appareil
* Connexion d’un appareil et vérification des importations
* Actualisation des appareils connectés

Si vous cliquez sur le nœud **Appareils** puis cliquez avec le bouton droit sur le volet **Résultats**, vous pouvez sélectionner les actions suivantes :

* Authentification d’un appareil
* Afficher les détails sur l’appareil
* Actualisation d’un appareil
* Suppression de la configuration d’un appareil
* Modification du mot de passe d’un appareil

> [!NOTE]
> L’ensemble de ces actions sont également disponibles dans le volet **Actions** .


Ce didacticiel explique comment utiliser le Gestionnaire d’instantanés StorSimple pour connecter et gérer les appareils et effectuer les tâches suivantes :

* Ajout ou remplacement d’un appareil
* Connexion d’un appareil et vérification des importations
* Actualisation des appareils connectés
* Authentification d’un appareil
* Afficher les détails sur l’appareil
* Actualisation d’un appareil
* Suppression de la configuration d’un appareil
* Modification d’un mot de passe expiré d’appareil
* Remplacement d’un appareil défaillant

> [!NOTE]
> Pour consulter des informations générales sur l’utilisation de l’interface du Gestionnaire d’instantanés StorSimple, accédez à la page [Interface utilisateur du Gestionnaire d’instantanés StorSimple](storsimple-use-snapshot-manager.md).


## <a name="add-or-replace-a-device"></a>Ajout ou remplacement d’un appareil
Exécutez la procédure suivante pour ajouter ou remplacer un appareil StorSimple.

#### <a name="to-add-or-replace-a-device"></a>Pour ajouter ou remplacer un appareil
1. Cliquez sur l’icône de bureau pour démarrer le Gestionnaire d’instantanés StorSimple.
2. Dans le volet **Étendue**, cliquez avec le bouton droit sur le nœud **Appareils**, puis cliquez sur **Configurer un appareil**. La boîte de dialogue **Configurer un appareil** apparaît.
   
    ![Configurer un appareil StorSimple](./media/storsimple-snapshot-manager-manage-devices/HCS_SSM_config_device.png) 
3. Dans la liste déroulante **Appareil** , sélectionnez l’adresse IP de l’appareil ou de l’appareil virtuel. 
4. Dans la zone de texte **Mot de passe** , entrez le mot de passe du Gestionnaire d’instantanés StorSimple que vous avez créé pour l’appareil dans le portail Azure Classic. Cliquez sur **OK**. Le Gestionnaire d’instantanés StorSimple recherche l’appareil que vous avez identifié. 
   
   * Si l’appareil est disponible, le Gestionnaire d’instantanés StorSimple ajoute une connexion.
   * Si, pour une raison quelconque, l’appareil est indisponible, le Gestionnaire d’instantanés StorSimple renvoie un message d’erreur. Cliquez sur **OK** pour fermer le message d’erreur, puis cliquez sur **Annuler** pour fermer la boîte de dialogue **Configurer un appareil**.

## <a name="connect-a-device-and-verify-imports"></a>Connexion d’un appareil et vérification des importations
Appliquez la procédure suivante pour connecter un appareil StorSimple et vérifier que les groupes de volumes existants qui présentent des sauvegardes associées sont importés.

#### <a name="to-connect-a-device-and-verify-imports"></a>Pour connecter un appareil et vérifier les importations
1. Pour connecter un appareil au Gestionnaire d’instantanés StorSimple, suivez les instructions de la section Ajouter ou remplacer un appareil. Lorsqu’il se connecte à un appareil, le Gestionnaire d’instantanés StorSimple réagit de la manière suivante :
   
   * Si, pour une raison quelconque, l’appareil est indisponible, le Gestionnaire d’instantanés StorSimple renvoie un message d’erreur. 
   
   * Si l’appareil est disponible, le Gestionnaire d’instantanés StorSimple ajoute une connexion. Lorsque vous sélectionnez un appareil, il apparaît dans le volet **Résultats**, et le champ de statut indique que l’appareil est **Disponible**. Le Gestionnaire d’instantanés StorSimple importe les groupes de volumes configurés pour l’appareil, à condition que les groupes de volumes présentent des sauvegardes associées. Les stratégies de sauvegarde ne sont pas importées. Les groupes de volumes qui ne sont associés à aucune sauvegarde ne sont pas importés.
2. Cliquez sur l’icône de bureau pour démarrer le Gestionnaire d’instantanés StorSimple.
3. Cliquez sur le nœud supérieur du volet **Étendue**, puis cliquez sur **Basculer l’affichage des importations**.
   
    ![Sélectionner Basculer l’affichage des importations](./media/storsimple-snapshot-manager-manage-devices/HCS_SSM_Toggle_Imports_Display.png) 
4. La boîte de dialogue **Basculer l’affichage des importations** apparaît ; elle indique le statut des groupes de volumes et des sauvegardes importés. Cliquez sur **OK**.

Une fois que les groupes de volumes et les sauvegardes ont été importés, vous pouvez utiliser le Gestionnaire d’instantanés StorSimple pour les gérer de la manière dont vous géreriez des groupes de volumes et des sauvegardes créés et configurés avec le Gestionnaire d’instantanés StorSimple. 

## <a name="refresh-connected-devices"></a>Actualisation des appareils connectés
Exécutez la procédure suivante afin de synchroniser les appareils StorSimple connectés avec le Gestionnaire d’instantanés StorSimple.

#### <a name="to-refresh-connected-devices"></a>Pour actualiser les appareils connectés
1. Cliquez sur l’icône de bureau pour démarrer le Gestionnaire d’instantanés StorSimple.
2. Dans le volet **Étendue**, cliquez avec le bouton droit sur **Appareils**, puis cliquez sur **Actualiser la liste des appareils**. Cette action synchronise les appareils connectés avec le Gestionnaire d’instantanés StorSimple, ce qui vous permet de consulter les groupes de volumes et les sauvegardes, notamment les récents ajouts. 
   
    ![Actualiser les appareils StorSimple](./media/storsimple-snapshot-manager-manage-devices/HCS_SSM_Refresh_devices.png)

L’action **Actualiser la liste des appareils** permet de récupérer les nouveaux groupes de volumes et les sauvegardes associées des appareils connectés. Contrairement à l’action **Relancer l’analyse des volumes** disponible pour le nœud **Volumes**, l’action **Actualiser la liste des appareils** ne restaure pas le registre de sauvegarde.

## <a name="authenticate-a-device"></a>Authentification d’un appareil
Exécutez la procédure suivante afin d’authentifier un appareil StorSimple avec le Gestionnaire d’instantanés StorSimple.

#### <a name="to-authenticate-a-device"></a>Pour authentifier un appareil
1. Cliquez sur l’icône de bureau pour démarrer le Gestionnaire d’instantanés StorSimple.
2. Dans le volet **Étendue**, cliquez sur **Appareils**.
3. Dans le volet **Résultats**, cliquez avec le bouton droit sur le nom de l’appareil, puis cliquez sur **Authentifier**.
4. La boîte de dialogue **Authentifier** apparaît. Saisissez le mot de passe de l’appareil, puis cliquez sur **OK**.
   
    ![Boîte de dialogue Authentifier](./media/storsimple-snapshot-manager-manage-devices/HCS_SSM_Authenticate.png) 

## <a name="view-device-details"></a>Afficher les détails sur l’appareil
Exécutez la procédure suivante pour consulter les détails sur un appareil StorSimple et, si nécessaire, synchroniser de nouveau l’appareil avec le Gestionnaire d’instantanés StorSimple.

#### <a name="to-view-and-resynchronize-device-details"></a>Pour afficher et synchroniser de nouveau les détails sur l’appareil
1. Cliquez sur l’icône de bureau pour démarrer le Gestionnaire d’instantanés StorSimple.
2. Dans le volet **Étendue**, cliquez sur **Appareils**.
3. Dans le volet **Résultats**, cliquez sur le nom de l’appareil, puis cliquez sur **Détails**.

4. La boîte de dialogue **Informations sur l’appareil** s’affiche. Cette zone indique le nom, le modèle, la version, le numéro de série, le statut, le nom qualifié iSCSI cible, ainsi que la date et l’heure de la dernière synchronisation.

* Cliquez sur **Resynchroniser** pour synchroniser l’appareil.
* Cliquez sur **OK** ou sur **Annuler** pour fermer la boîte de dialogue.
  
  ![Informations sur l’appareil](./media/storsimple-snapshot-manager-manage-devices/HCS_SSM_Device_details.png) 

## <a name="refresh-an-individual-device"></a>Actualisation d’un appareil
Exécutez la procédure suivante pour synchroniser de nouveau un appareil StorSimple avec le Gestionnaire d’instantanés StorSimple.

#### <a name="to-refresh-a-device"></a>Pour actualiser un appareil
1. Cliquez sur l’icône de bureau pour démarrer le Gestionnaire d’instantanés StorSimple. 
2. Dans le volet **Étendue**, cliquez sur **Appareils**. 
3. Dans le volet **Résultats**, cliquez avec le bouton droit sur le nom de l’appareil, puis cliquez sur **Actualiser l’appareil**. L’appareil est synchronisé avec le Gestionnaire d’instantanés StorSimple.

## <a name="delete-a-device-configuration"></a>Suppression de la configuration d’un appareil
Utilisez la procédure suivante pour supprimer une configuration d’appareil StorSimple du Gestionnaire d’instantanés StorSimple.

#### <a name="to-delete-a-device-configuration"></a>Pour supprimer une configuration d’appareil
1. Cliquez sur l’icône de bureau pour démarrer le Gestionnaire d’instantanés StorSimple.
2. Dans le volet **Étendue**, cliquez sur **Appareils**. 
3. Dans le volet **Résultats**, cliquez avec le bouton droit sur le nom de l’appareil, puis cliquez sur **Supprimer**. 
4. Le message suivant s’affiche. Cliquez sur **Oui** pour supprimer la configuration, ou sur **Non** pour annuler la suppression.
   
    ![Supprimer la configuration de l’appareil](./media/storsimple-snapshot-manager-manage-devices/HCS_SSM_DeleteDevice.png)

## <a name="change-an-expired-device-password"></a>Modification d’un mot de passe expiré d’appareil
Vous devez entrer un mot de passe pour authentifier un appareil StorSimple avec le Gestionnaire d’instantanés StorSimple. Vous définissez ce mot de passe lorsque vous utilisez l’interface Windows PowerShell pour configurer l’appareil. Toutefois, le mot de passe peut expirer. Le cas échéant, vous avez la possibilité de le modifier à l’aide du portail Azure Classic. Ensuite, l’appareil ayant été configuré dans le Gestionnaire d’instantanés StorSimple avant l’expiration du mot de passe, vous devez de nouveau authentifier l’appareil dans le Gestionnaire d’instantanés StorSimple.

#### <a name="to-change-the-expired-password"></a>Pour modifier le mot de passe expiré
1. Dans le portail Azure Classic, démarrez le service StorSimple Manager.
2. Cliquez sur **Appareils** > **Configurer** pour l’appareil.
3. Faites défiler jusqu’à la section du Gestionnaire d’instantanés StorSimple. Entrez un mot de passe comportant entre 14 et 15 caractères. Assurez-vous que le mot de passe contient un mélange de majuscules, de minuscules, de chiffres et de caractères spéciaux.
4. Entrez de nouveau le mot de passe pour le confirmer.
5. Cliquez sur **Enregistrer** au bas de la page.

#### <a name="to-re-authenticate-the-device"></a>Pour authentifier de nouveau l’appareil
1. Démarrez le Gestionnaire d’instantanés StorSimple.
2. Dans le volet **Étendue**, cliquez sur **Appareils**. Une liste des appareils configurés apparaît dans le volet **Résultats** .
3. Sélectionnez l’appareil, cliquez avec le bouton droit, puis cliquez sur **Authentifier**.
4. Dans la fenêtre **Authentifier** saisissez le nouveau mot de passe.
5. Sélectionnez l’appareil, cliquez avec le bouton droit, puis sélectionnez **Actualiser l’appareil**. L’appareil est synchronisé avec le Gestionnaire d’instantanés StorSimple.

## <a name="replace-a-failed-device"></a>Remplacement d’un appareil défaillant
Si un appareil StorSimple défaillant est remplacé par un appareil de secours (basculement), procédez comme suit pour connecter le nouvel appareil et afficher les sauvegardes associées.

#### <a name="to-connect-to-a-new-device-after-failover"></a>Pour se connecter à un nouvel appareil après un basculement
1. Reconfigurez la connexion iSCSI au nouvel appareil. Pour obtenir les instructions, accédez à « Étape 7 : Montage, initialisation et formatage d’un volume » de la page [Déploiement de votre appareil StorSimple local](storsimple-8000-deployment-walkthrough-u2.md).

> [!NOTE]
> Si le nouvel appareil StorSimple possède la même adresse que l’ancien, il est possible que vous puissiez connecter l’ancienne configuration.


1. Arrêtez le service de gestion Microsoft StorSimple :
   
   1. Démarrez le Gestionnaire de serveur.
   2. Sur le tableau de bord du Gestionnaire de serveur, dans le menu **Outils**, sélectionnez **Services**.
   3. Dans la fenêtre **Services**, sélectionnez le **Service de gestion Microsoft StorSimple**.
   4. Dans le volet droit, sous **Service de gestion Microsoft StorSimple**, cliquez sur **Arrêter le service**.
2. Supprimez les informations de configuration relatives à l’ancien appareil :
   
   1. Dans l’Explorateur de fichiers, accédez à C:\ProgramData\Microsoft\StorSimple\BACatalog.
   2. Supprimez les fichiers du dossier BACatalog.
3. Redémarrez le service de gestion Microsoft StorSimple :
   
   1. Sur le tableau de bord du Gestionnaire de serveur, dans le menu **Outils**, sélectionnez **Services**.
   2. Dans la fenêtre **Services**, sélectionnez le **Service de gestion Microsoft StorSimple**.
   3. Dans le volet droit, sous **Service de gestion Microsoft StorSimple**, cliquez sur **Redémarrer le service**.
4. Démarrez le Gestionnaire d’instantanés StorSimple.
5. Pour configurer le nouvel appareil StorSimple, exécutez la procédure décrite dans Étape 2 : Connexion d’un appareil StorSimple de la page [Déployer le Gestionnaire d’instantanés StorSimple](storsimple-snapshot-manager-deployment.md).
6. Cliquez avec le bouton droit sur le nœud de niveau supérieur du volet **Étendue** (Gestionnaire d’instantanés StorSimple dans l’exemple), puis cliquez sur **Basculer l’affichage des importations**. 
7. Un message s’affiche lorsque les groupes de volumes et les sauvegardes importés sont visibles dans le Gestionnaire d’instantanés StorSimple. Cliquez sur **OK**.

## <a name="next-steps"></a>Étapes suivantes
* Découvrez comment [utiliser le Gestionnaire d’instantanés StorSimple pour gérer votre solution StorSimple](storsimple-snapshot-manager-admin.md).
* Découvrez comment [utiliser le Gestionnaire d’instantanés StorSimple pour afficher et gérer les volumes](storsimple-snapshot-manager-manage-volumes.md).

