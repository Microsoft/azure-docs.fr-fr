---
title: Restauration d’un volume à partir d’une sauvegarde sur un appareil de la gamme StorSimple 8000 | Microsoft Docs
description: Explique comment utiliser le Catalogue de sauvegarde du service StorSimple Device Manager pour restaurer un volume StorSimple à partir d’un jeu de sauvegarde.
services: storsimple
documentationcenter: NA
author: alkohli
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.date: 07/15/2020
ms.author: alkohli
ms.openlocfilehash: ff70df225b5ed27960c96889d409d7005f0fbcc4
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94950726"
---
# <a name="restore-a-storsimple-volume-from-a-backup-set"></a>Restauration d’un volume StorSimple à partir d’un jeu de sauvegarde

## <a name="overview"></a>Vue d’ensemble

Ce didacticiel décrit la restauration d’un appareil de la gamme StorSimple 8000 à l’aide d’un jeu de sauvegarde existant. Utilisez le panneau **Catalogue de sauvegarde** pour restaurer un volume à partir d’une sauvegarde locale ou cloud. Le panneau **Catalogue de sauvegarde** affiche tous les jeux de sauvegarde créés lors de sauvegardes manuelles ou automatisées. L’opération de restauration à partir d’un jeu de sauvegarde met le volume en ligne immédiatement pendant le téléchargement des données en arrière-plan.

## <a name="before-you-restore"></a>Avant de procéder à la restauration

Avant de démarrer une restauration, prenez garde aux éléments suivants :

* **Vous devez mettre le volume hors connexion** – Placez le volume hors connexion sur l’hôte et sur l’appareil avant de lancer une opération de restauration. Bien que l’opération de restauration mette automatiquement le volume en ligne sur l’appareil, vous devez le faire manuellement sur l’hôte. Vous pouvez mettre le volume en ligne sur l’hôte dès que le volume est en ligne sur l’appareil. (Il est inutile d’attendre la fin de l’opération de restauration.) Pour connaître les procédures, consultez [Mise hors connexion d’un volume](storsimple-8000-manage-volumes-u2.md#take-a-volume-offline).

* **Type de volume après restauration** – Les volumes supprimés sont restaurés en fonction du type dans l’instantané. Ainsi, les volumes qui étaient épinglés localement sont restaurés comme tels et les volumes à plusieurs niveaux de même.

    Pour les volumes existants, le type de volume selon l’utilisation actuelle remplace le type stocké dans l’instantané. Par exemple, si vous restaurez un volume à partir d’un instantané qui a été pris lorsque le type de volume était à plusieurs niveaux et que celui-ci est désormais épinglé localement (à la suite d’une opération de conversion), le volume sera restauré comme volume épinglé localement. De même, si un volume épinglé localement existant a été étendu puis restauré à partir d’un instantané précédent pris lorsque le volume était plus petit, le volume restauré conserve la taille étendue actuelle.

    Vous ne pouvez pas convertir un volume à plusieurs niveaux en un volume épinglé localement, ou un volume épinglé localement en un volume à plusieurs niveaux pendant sa restauration. Vous devez attendre la fin de l’opération de restauration avant de convertir le volume en un autre type. Pour plus d’informations sur la conversion d’un volume, consultez la page [Changer de type de volume](storsimple-8000-manage-volumes-u2.md#change-the-volume-type). 

* **La taille du volume est reflétée dans le volume restauré** : il s’agit d’un élément important si vous restaurez un volume épinglé localement qui a été supprimé (car les volumes épinglés localement sont complètement configurés). Vérifiez que vous disposez d’un espace suffisant avant d’essayer de restaurer un volume épinglé localement qui a été supprimé.

* **Vous ne pouvez pas étendre un volume pendant sa restauration** – Attendez la fin de l’opération de restauration avant d’essayer d’étendre le volume. Pour plus d’informations sur l’extension d’un volume, consultez la page [Modifier un volume](storsimple-8000-manage-volumes-u2.md#modify-a-volume).

* **Vous pouvez effectuer une sauvegarde lorsque vous restaurez un volume local** – Pour connaître les procédures, accédez à [Utiliser le service StorSimple Device Manager pour gérer les stratégies de sauvegarde](storsimple-8000-manage-backup-policies-u2.md).

* **Vous pouvez annuler une opération de restauration** – Si vous annulez le travail de restauration, le volume sera restauré à l’état où il se trouvait avant le lancement de l’opération de restauration. Pour connaître les procédures, consultez la page [Annuler un travail](storsimple-8000-manage-jobs-u2.md#cancel-a-job).

## <a name="how-does-restore-work"></a>Fonctionnement du processus de restauration

Pour les appareils exécutant Update 4 ou version ultérieure, une restauration basée sur une carte thermique est implémentée. Étant donné que l’hôte demande à accéder aux données de l’appareil, ces demandes font l’objet d’un suivi et une carte thermique est créée. Un taux de demande élevé se traduit par des segments de données avec plus de chaleur, tandis qu’un taux de demande plus faible se traduit par des segments avec moins de chaleur. Vous devez accéder aux données à deux reprises minimum pour qu’elles soient marquées comme _chaudes_. Un fichier modifié est également marqué comme _chaud_. Une fois que vous lancez la restauration, le rafraîchissement proactif des données se produit en fonction de la carte thermique. Pour les versions antérieures à Update 4, le téléchargement des données a lieu pendant la restauration basée sur l’accès uniquement.

Les mises en garde suivantes s’appliquent aux restaurations basées sur la carte thermique :

* Le suivi de carte thermique est activé uniquement pour les volumes hiérarchisés (les volumes épinglés localement ne sont pas pris en charge).

* La restauration basée sur la carte thermique n’est pas prise en charge lors du clonage d’un volume sur un autre appareil. 

* Si une restauration en place et si une capture instantanée locale existent sur l’appareil pour le volume à restaurer, aucun rafraîchissement n’est nécessaire (car les données sont déjà disponibles localement). 

* Par défaut, lors de la restauration, les travaux de rafraîchissement sont initiés et rafraîchissent les données de façon proactive en fonction de la carte thermique. 

Dans Update 4, les cmdlets Windows PowerShell peuvent être utilisées pour interroger des travaux de rafraîchissement en cours d’exécution, annuler un travail de rafraîchissement et obtenir l’état du travail de rafraîchissement.

* `Get-HcsRehydrationJob` : cette cmdlet obtient l’état du travail de rafraîchissement. Un seul travail de rafraîchissement est déclenché pour un volume.

* `Set-HcsRehydrationJob` : cette cmdlet vous permet d’interrompre, d’arrêter ou de reprendre le travail de rafraîchissement au cours du rafraîchissement.

Pour plus d’informations sur les cmdlets de rafraîchissement, consultez le [document de référence sur les cmdlets Windows PowerShell pour StorSimple](/powershell/module/hcs/?viewFallbackFrom=winserverr2-ps).

Avec le rafraîchissement automatique, les performances de lecture temporaires sont généralement plus élevées. L'ampleur des améliorations dépend de divers facteurs, tels que le modèle d'accès, le taux d'activité des données et le type de données. 

Pour annuler un travail de rafraîchissement vous pouvez utiliser la cmdlet PowerShell. Si vous souhaitez désactiver définitivement les travaux de rafraîchissement pour toutes les restaurations futures, [contactez le Support Microsoft](storsimple-8000-contact-microsoft-support.md).

## <a name="how-to-use-the-backup-catalog"></a>Utilisation du catalogue de sauvegarde

Le panneau **Catalogue de sauvegarde** comprend une zone de requête qui vous permet d’affiner la sélection des jeux de sauvegarde. Vous pouvez filtrer les jeux de sauvegarde récupérés selon les paramètres suivants :

* **Intervalle de temps** : plage de dates et d’heures de création du jeu de sauvegarde.
* **Appareil** : appareil sur lequel le jeu de sauvegarde a été créé.
* **Stratégie de sauvegarde** ou **Volume** : stratégie de sauvegarde ou volume associé à ce jeu de sauvegarde.

Les jeux de sauvegarde filtrés sont ensuite affichés sous forme de tableau sur la base des attributs suivants :

* **Nom** : nom de la stratégie de sauvegarde ou du volume associé à ce jeu de sauvegarde.
* **Type** : les jeux de sauvegarde peuvent être des instantanés locaux ou des instantanés cloud. Un instantané local est une sauvegarde de toutes les données de volume stockées localement sur l’appareil, tandis qu’un instantané cloud correspond à la sauvegarde des données de volume résidant dans le cloud. Les instantanés locaux offrent un accès plus rapide, alors que les instantanés cloud sont choisis pour la résilience des données.
* **Taille** : taille réelle du jeu de sauvegarde.
* **Créé le** : date et heure auxquelles les sauvegardes ont été créées. 
* **Volumes** : le nombre de volumes associés au jeu de sauvegarde.
* **Initié** : les sauvegardes peuvent être lancées automatiquement suivant une planification ou manuellement par un utilisateur. (Vous pouvez utiliser une stratégie de sauvegarde pour planifier des sauvegardes. Vous pouvez également utiliser l’option **Exécuter la sauvegarde** pour effectuer une sauvegarde interactive ou à la demande.)

## <a name="how-to-restore-your-storsimple-volume-from-a-backup"></a>Comment restaurer votre volume StorSimple à partir d’une sauvegarde

Vous pouvez utiliser le panneau **Catalogue de sauvegarde** pour restaurer votre volume StorSimple à partir d’une sauvegarde spécifique. N’oubliez pas cependant que la restauration d’un volume rétablit l’état dans lequel il se trouvait au moment de la sauvegarde. Toutes les données qui ont été ajoutées après l’opération de sauvegarde seront perdues.

> [!WARNING]
> La restauration à partir d’une sauvegarde remplace les volumes existants à partir de la sauvegarde. Cela peut entraîner la perte des données qui ont été écrites après la sauvegarde.


### <a name="to-restore-your-volume"></a>Pour restaurer votre volume
1. Accédez à votre service StorSimple Device Manager, puis cliquez sur **Catalogue de sauvegarde**.

2. Sélectionnez un jeu de sauvegarde comme suit :
   
   1. Indiquez l’intervalle de temps.
   2. Sélectionnez l’appareil approprié.
   3. Dans la liste déroulante, choisissez la stratégie de sauvegarde ou le volume pour la sauvegarde à sélectionner.
   4. Cliquez sur **Appliquer** pour exécuter cette requête.

      Les sauvegardes associées au volume ou à la stratégie de sauvegarde sélectionné doivent figurer dans la liste des jeux de sauvegarde.
   
      ![Liste des jeux de sauvegarde](./media/storsimple-8000-restore-from-backup-set-u2/bucatalog.png)     
     
3. Développez le jeu de sauvegarde pour afficher les volumes associés. Ces volumes doivent être mis hors connexion sur l’hôte et l’appareil avant leur restauration. Accédez aux volumes dans le panneau **Volumes** de votre appareil, puis suivez les étapes de la rubrique [Mise hors connexion d’un volume](storsimple-8000-manage-volumes-u2.md#take-a-volume-offline) pour les mettre hors connexion.
   
   > [!IMPORTANT]
   > Veillez à mettre les volumes hors connexion sur l’ordinateur hôte avant de les mettre hors connexion sur l’appareil. Sans quoi, vous vous exposez à un risque d’altération des données.
   
4. Revenez à l’onglet **Catalogue de sauvegarde** et sélectionnez un jeu de sauvegarde. Cliquez dessus avec le bouton droit puis sélectionnez **Restaurer** dans le menu contextuel.

    ![Liste des jeux de sauvegarde 2](./media/storsimple-8000-restore-from-backup-set-u2/restorebu1.png)

5. Vous êtes invité à confirmer l’opération. Passez en revue les informations de restauration, puis cochez la case de confirmation.
   
    ![Page Confirmation](./media/storsimple-8000-restore-from-backup-set-u2/restorebu2.png)

7. Cliquez sur **Restaurer**. Cette opération lance une tâche de restauration que vous pouvez afficher en accédant à la page **Tâches**.

   ![Page Confirmation 2](./media/storsimple-8000-restore-from-backup-set-u2/restorebu5.png)

8. Une fois la restauration terminée, vérifiez que le contenu de vos volumes a été remplacé par les volumes provenant de la sauvegarde.


## <a name="if-the-restore-fails"></a>Si la restauration échoue

Vous recevez une alerte si l’opération de restauration échoue pour une raison quelconque. Le cas échéant, actualisez la liste de sauvegarde pour vérifier que la sauvegarde est toujours valide. Si la sauvegarde est valide et que vous restaurez à partir du cloud, le problème peut être dû à des problèmes de connectivité.

Pour terminer l’opération de restauration, mettez le volume hors connexion sur l’hôte et recommencez l’opération de restauration. Notez que toutes les modifications apportées aux données des volumes pendant le processus de restauration seront perdues.

## <a name="next-steps"></a>Étapes suivantes
* Découvrez comment [gérer des volumes StorSimple](storsimple-8000-manage-volumes-u2.md).
* Découvrez comment [utiliser le service StorSimple Device Manager pour gérer votre appareil StorSimple](storsimple-8000-manager-service-administration.md).