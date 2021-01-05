---
title: Sauvegarder des partages de fichiers Azure dans le portail Azure
description: Découvrir comment utiliser le portail Azure pour sauvegarder des partages de fichiers Azure sauvegardés dans le coffre Recovery Services
ms.topic: conceptual
ms.date: 01/20/2020
ms.openlocfilehash: ca49f1ad48ab0534b27b91ad6a5a50b393cda782
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88890346"
---
# <a name="back-up-azure-file-shares"></a>Sauvegarder des partages de fichiers Azure

Cet article explique comment sauvegarder les [partages de fichiers Azure](../storage/files/storage-files-introduction.md) à partir du portail Azure.

Dans cet article, vous allez apprendre à :

* Créez un coffre Recovery Services.
* Configurer la sauvegarde à partir du coffre Recovery Services
* Configurer la sauvegarde à partir du volet de partage de fichiers
* Exécuter un travail de sauvegarde à la demande pour créer un point de restauration

## <a name="prerequisites"></a>Prérequis

* [Découvrez](azure-file-share-backup-overview.md) la solution de sauvegarde de partage de fichiers Azure basée sur une capture instantanée de partage.
* Assurez-vous que le partage de fichiers est présent dans l’un des [types de comptes de stockage pris en charge](azure-file-share-support-matrix.md).
* Identifiez ou créez un [coffre Recovery Services](#create-a-recovery-services-vault) dans la même région que le compte de stockage hébergeant le partage de fichiers.

[!INCLUDE [How to create a Recovery Services vault](../../includes/backup-create-rs-vault.md)]

## <a name="configure-backup-from-the-recovery-services-vault"></a>Configurer la sauvegarde à partir du coffre Recovery Services

Les étapes suivantes expliquent comment configurer la sauvegarde de plusieurs partages de fichiers à partir du volet Coffre Recovery Services :

1. Dans le [Portail Azure](https://portal.azure.com/), ouvrez le coffre Recovery Services que vous souhaitez utiliser pour configurer la sauvegarde de votre partage de fichiers.

1. Dans le volet **Coffre Recovery Services**, sélectionnez **+Sauvegarde** dans le menu situé en haut.

   ![Coffre Recovery Services](./media/backup-afs/recovery-services-vault.png)

    1. Dans le volet **Objectif de sauvegarde**, définissez **Où s'exécute votre charge de travail ?** sur **Azure** en sélectionnant l’option **Azure** dans la liste déroulante.

          ![Choisir Azure en tant que charge de travail](./media/backup-afs/backup-goal.png)

    2. Dans **Que souhaitez-vous sauvegarder ?** , sélectionnez **Partage de fichiers Azure** à partir de la liste déroulante.

          ![Sélectionner Azure FileShare](./media/backup-afs/select-azure-file-share.png)

    3. Sélectionnez **Sauvegarde** pour inscrire l’extension de partage de fichiers Azure dans le coffre.

          ![Sélectionnez Sauvegarde pour associer le partage de fichiers Azure au coffre](./media/backup-afs/register-extension.png)

1. Une fois que vous avez sélectionné **Sauvegarde**, le volet **Sauvegarde** s’ouvre. Pour sélectionner le compte de stockage hébergeant le partage de fichiers que vous souhaitez protéger, sélectionnez le texte du lien **Sélectionner** en dessous de la zone de texte **Compte de stockage**.

   ![Choisir le lien Sélectionner](./media/backup-afs/choose-select-link.png)

1. Le volet **Sélectionner un compte de stockage** qui s’ouvre à droite répertorie l’ensemble de comptes de stockage pris en charge qui ont été détectés. Ceux-ci sont associés à ce coffre ou présents dans la même région que le coffre, mais pas encore associés à un coffre Recovery Services.

1. Dans la liste des comptes de stockage détectés, sélectionnez un compte, puis **OK**.

   ![Sélectionner parmi les comptes de stockage détectés](./media/backup-afs/select-discovered-storage-account.png)

1. L’étape suivante consiste à sélectionner les partages de fichiers que vous souhaitez sauvegarder. Sélectionnez le bouton **Ajouter** dans la section **Partages de fichiers à sauvegarder**.

   ![Sélectionner le partage de fichiers à sauvegarder](./media/backup-afs/select-file-shares-to-back-up.png)

1. Le volet contextuel **Sélectionner les partages de fichiers** s’ouvre à droite. Azure recherche dans le compte de stockage les partages de fichiers qui peuvent être sauvegardés. Si vous avez récemment ajouté vos partages de fichiers et ne les voyez pas dans la liste, patientez un peu jusqu’à ce qu’ils s’affichent.

1. Dans la liste **Sélectionner les partages de fichiers**, sélectionnez un ou plusieurs partages de fichiers à sauvegarder. Sélectionnez **OK**.

   ![Sélectionner les partages de fichiers](./media/backup-afs/select-file-shares.png)

1. Pour choisir une stratégie de sauvegarde pour votre partage de fichiers, vous avez trois options :

   * Choisissez la stratégie par défaut.<br>
   Cette option vous permet d’activer des sauvegardes quotidiennes qui seront conservées pendant 30 jours. Si vous n’avez pas de stratégie de sauvegarde existante dans le coffre, le volet de sauvegarde s’ouvre avec les paramètres de stratégie par défaut. Si vous voulez choisir les paramètres par défaut, vous pouvez sélectionner directement **Activer la sauvegarde**.

   * Créer une nouvelle stratégie <br>

      1. Pour créer une stratégie de sauvegarde pour votre partage de fichiers, sélectionnez le texte du lien en dessous de la liste déroulante dans la section **Stratégie de sauvegarde**.<br>

         ![Créez une stratégie.](./media/backup-afs/create-new-policy.png)

      1. Le volet contextuel **Stratégie de sauvegarde** s’ouvre à droite. Spécifiez un nom de stratégie dans la zone de texte et choisissez la période de rétention en fonction de vos besoins. Seule l’option de rétention quotidienne est activée par défaut. Si vous souhaitez une rétention hebdomadaire, mensuelle ou annuelle, activez la case à cocher correspondante et indiquez la valeur de rétention souhaitée.

      1. Après avoir spécifié les valeurs de conservation et un nom de stratégie valide, sélectionnez **OK**.<br>

         ![Fournir les valeurs de noms de stratégie et de rétention](./media/backup-afs/policy-name.png)

   * Choisir l’une des stratégies de sauvegarde existantes <br>

      Pour choisir l’une des stratégies de sauvegarde existantes pour la configuration de la protection, sélectionnez la stratégie de votre choix dans la liste déroulante **Stratégie de sauvegarde**.<br>

      ![Choisir une stratégie existante](./media/backup-afs/choose-existing-policy.png)

1. Sélectionnez **Activer la sauvegarde** pour commencer à protéger le partage de fichiers.

   ![Choisir Activer la sauvegarde](./media/backup-afs/enable-backup.png)

Une fois que vous avez défini une stratégie de sauvegarde, un instantané des partages de fichiers est pris à l’heure planifiée. Le point de récupération est également conservé pendant la période choisie.

>[!NOTE]
>Sauvegarde Azure prend désormais en charge les stratégies de conservation quotidienne, hebdomadaire, mensuelle et annuelle pour la sauvegarde des fichiers partagés Azure.

## <a name="configure-backup-from-the-file-share-pane"></a>Configurer la sauvegarde à partir du volet de partage de fichiers

Les étapes suivantes expliquent comment configurer la sauvegarde de partages de fichiers individuels à partir de leur volet de partage de fichiers respectif :

1. Sur le [portail Azure](https://portal.azure.com/), ouvrez le compte de stockage hébergeant le partage de fichiers à sauvegarder.

1. Une fois dans le compte de stockage, sélectionnez la vignette intitulée **Partages de fichiers**. Vous pouvez aussi accéder à **Partages de fichiers** via la table des matières du compte de stockage.

   ![Compte de stockage](./media/backup-afs/storage-account.png)

1. Dans la liste des partages de fichiers doivent figurer tous les partages de fichiers présents dans le compte de stockage. Sélectionnez le partage de fichiers que vous voulez sauvegarder.

   ![Liste des partages de fichiers](./media/backup-afs/file-shares-list.png)

1. Sélectionnez **Sauvegarde** sous la section **Opérations** du volet de partage de fichiers. Le volet **Configurer la sauvegarde** se charge à droite.

   ![Volet Configurer la sauvegarde](./media/backup-afs/configure-backup.png)

1. Pour la sélection du coffre Recovery Services, procédez de l’une des façons suivantes :

    * Si vous disposez déjà d’un coffre, sélectionnez la case d’option **Sélectionner** pour les coffres Recovery Services, puis choisissez l’un des coffres existants dans le menu déroulant **Nom du coffre**.

       ![Sélectionner un coffre existant](./media/backup-afs/select-existing-vault.png)

    * Si vous n’avez pas de coffre, sélectionnez la case d’option **Créer nouveau** pour les coffres Recovery Services. Nommez le coffre. Il est créé dans la même région que le partage de fichiers. Par défaut, le coffre est créé dans le même groupe de ressources que le partage de fichiers. Si vous voulez choisir un autre groupe de ressources, sélectionnez le lien **Créer nouveau** sous la liste déroulante **Type de ressource**, puis attribuez un nom au groupe de ressources. Sélectionnez **OK** pour continuer.

       ![Créer un coffre](./media/backup-afs/create-new-vault.png)

      >[!IMPORTANT]
      >Si le compte de stockage est inscrit auprès d’un coffre ou qu’il y a peu de partages protégés dans le compte de stockage hébergeant le partage de fichiers que vous tentez de protéger, le nom du coffre Recovery Services est prérempli et vous n’êtes pas autorisé à le modifier. [Apprenez-en davantage ici](backup-azure-files-faq.md#why-cant-i-change-the-vault-to-configure-backup-for-the-file-share).

1. Pour sélectionner une **Stratégie de sauvegarde**, procédez de l’une des façons suivantes :

    * Conservez la stratégie par défaut. Elle planifiera les sauvegardes quotidiennes avec une période de conservation de 30 jours.

    * Sélectionnez une stratégie de sauvegarde existante, si vous en avez une, dans le menu déroulant **Stratégie de sauvegarde**.

       ![Choisir une stratégie de sauvegarde](./media/backup-afs/choose-backup-policy.png)

    * Créez une stratégie avec une conservation quotidienne/hebdomadaire/mensuelle/annuelle, selon vos besoins.  

         1. Sélectionnez le texte du lien **Créer une stratégie**.

         2. Le volet contextuel **Stratégie de sauvegarde** s’ouvre à droite. Spécifiez un nom de stratégie dans la zone de texte et choisissez la période de rétention en fonction de vos besoins. Seule l’option de rétention quotidienne est activée par défaut. Si vous souhaitez une rétention hebdomadaire, mensuelle ou annuelle, activez la case à cocher correspondante et indiquez la valeur de rétention souhaitée.

         3. Après avoir spécifié les valeurs de conservation et un nom de stratégie valide, sélectionnez **OK**.

            ![Créer une stratégie de sauvegarde](./media/backup-afs/create-new-backup-policy.png)

1. Sélectionnez **Activer la sauvegarde** pour commencer à protéger le partage de fichiers.

   ![Sélectionner Activer la sauvegarde](./media/backup-afs/select-enable-backup.png)

1. Vous pouvez suivre la progression de la configuration dans les notifications du portail ou en supervisant les travaux de sauvegarde relevant du coffre dont vous vous servez pour protéger le partage de fichiers.

   ![Notifications du portail](./media/backup-afs/portal-notifications.png)

1. Une fois l’opération de configuration de la sauvegarde terminée, sélectionnez **Sauvegarde** sous la section **Opérations** du volet de partage de fichiers. Le volet contextuel listant les **éléments essentiels du coffre** se charge à droite. De là, vous pouvez déclencher des opérations de sauvegarde et de restauration à la demande.

   ![Éléments essentiels du coffre](./media/backup-afs/vault-essentials.png)

## <a name="run-an-on-demand-backup-job"></a>Exécuter un travail de sauvegarde à la demande

Vous pouvez occasionnellement générer un instantané de sauvegarde ou un point de récupération en dehors des heures planifiées dans la stratégie de sauvegarde. Souvent que la génération d'une sauvegarde à la demande intervient immédiatement après que vous avez configuré la stratégie de sauvegarde. Selon la planification définie dans la stratégie de sauvegarde, des heures ou des jours peuvent s’écouler avant la prise d’un instantané. Pour protéger vos données en attendant que la stratégie de sauvegarde génère un instantané, effectuez une sauvegarde à la demande. La création d’une sauvegarde à la demande est souvent nécessaire avant d’apporter des modifications prévues à vos partages de fichiers.

### <a name="from-the-recovery-services-vault"></a>À partir du coffre Recovery Services

1. Ouvrez le coffre de Recovery Services que vous avez utilisé pour sauvegarder votre partage de fichiers. Dans le volet **Vue d’ensemble**, dans la section **Éléments protégés**, sélectionnez **Éléments de sauvegarde**.

   ![Sélectionner les éléments de sauvegarde](./media/backup-afs/backup-items.png)

1. Une fois les **Éléments de sauvegarde** sélectionnés, un nouveau volet répertoriant tous les **Types de gestion des sauvegardes** apparaît en regard du volet **Vue d’ensemble**.

   ![Liste des types de gestion des sauvegardes](./media/backup-afs/backup-management-types.png)

1. Dans la liste **Type de gestion de sauvegarde**, sélectionnez **Stockage Azure (Azure Files)** . Vous voyez s’afficher une liste de tous les partages de fichiers et des comptes de stockage correspondants sauvegardés à l’aide de ce coffre.

   ![Éléments de sauvegarde Stockage Azure (Azure Files)](./media/backup-afs/azure-files-backup-items.png)

1. Dans la liste des partages de fichiers Azure, sélectionnez le partage de fichiers de votre choix. Les détails associés à **Élément de sauvegarde** s’affichent. Dans le menu **Élément de sauvegarde**, sélectionnez **Sauvegarder maintenant**. Comme il s’agit d’un travail de sauvegarde à la demande, aucune stratégie de rétention n’est associée au point de récupération.

   ![Sélectionner Sauvegarder maintenant](./media/backup-afs/backup-now.png)

1. Le volet **Sauvegarder maintenant** s’ouvre. Spécifiez le dernier jour que vous souhaitez conserver dans le point de récupération. La conservation maximale d'une sauvegarde à la demande est de 10 ans.

   ![Choisissez la date de conservation](./media/backup-afs/retention-date.png)

1. Cliquez sur **OK** pour confirmer l’exécution du travail de sauvegarde à la demande.

1. Surveillez les notifications du portail pour conserver une trace de la bonne exécution des tâches de sauvegarde. Vous pouvez surveiller la progression du travail dans le tableau de bord du coffre. Sélectionnez **Travaux de sauvegarde** > **En cours**.

### <a name="from-the-file-share-pane"></a>À partir du volet de partage de fichiers

1. Ouvrez le volet **Vue d’ensemble**  du partage de fichiers pour lequel vous voulez effectuer une sauvegarde à la demande.

1. Sélectionnez **Sauvegarde** sous la section **Opération**. Le volet contextuel listant les **éléments essentiels du coffre** se charge à droite. Sélectionnez **Sauvegarder maintenant** pour effectuer une sauvegarde à la demande.

   ![Sélectionner Sauvegarder maintenant](./media/backup-afs/select-backup-now.png)

1. Le volet **Sauvegarder maintenant** s’ouvre. Spécifiez la période de conservation du point de récupération. La conservation maximale d'une sauvegarde à la demande est de 10 ans.

   ![Conserver la date de sauvegarde](./media/backup-afs/retain-backup-date.png)

1. Sélectionnez **OK** pour confirmer.

>[!NOTE]
>Sauvegarde Azure verrouille le compte de stockage lorsque vous configurez la protection pour tout partage de fichiers dans le compte correspondant. Cette méthode offre une protection contre la suppression accidentelle d'un compte de stockage avec des partages de fichiers sauvegardés.

## <a name="best-practices"></a>Meilleures pratiques

* Ne supprimez pas d’instantanés créés par Sauvegarde Azure. La suppression d’instantanés peut provoquer une perte de points de récupération et/ou des échecs de restauration.

* Ne supprimez pas le verrou effectué par Sauvegarde Azure au niveau du compte de stockage. Si vous supprimez le verrou, votre compte de stockage risque d’être supprimé accidentellement et, dans ce cas, vous perdrez vos instantanés ou vos sauvegardes.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment :

* [Restaurer des partages de fichiers Azure](restore-afs.md)
* [Gérer les sauvegardes de partage de fichiers Azure](manage-afs-backup.md)
