---
title: Activer la sauvegarde quand vous créez une machine virtuelle Azure
description: Explique comment activer la sauvegarde quand vous créez une machine virtuelle Azure avec Sauvegarde Azure.
ms.topic: conceptual
ms.date: 06/13/2019
ms.openlocfilehash: ad81300545686d61f42cdd8684e502c937b4fd43
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89377333"
---
# <a name="enable-backup-when-you-create-an-azure-vm"></a>Activer la sauvegarde quand vous créez une machine virtuelle Azure

Utilisez le service Sauvegarde Azure pour sauvegarder des machines virtuelles Azure. Les machines virtuelles sont sauvegardées selon une planification spécifiée dans une stratégie de sauvegarde, et les points de récupération sont créés à partir des sauvegardes. Les points de récupération sont stockés dans des coffres Recovery Services.

Cet article explique comment activer la sauvegarde quand vous créez une machine virtuelle dans le portail Azure.  

## <a name="before-you-start"></a>Avant de commencer

- [Vérifiez](backup-support-matrix-iaas.md#supported-backup-actions) quels systèmes d’exploitation sont pris en charge si vous activez la sauvegarde quand vous créez une machine virtuelle.

## <a name="sign-in-to-azure"></a>Connexion à Azure

Si vous n’êtes pas connecté à votre compte, connectez-vous sur le [portail Azure](https://portal.azure.com).

## <a name="create-a-vm-with-backup-configured"></a>Créer une machine virtuelle avec sauvegarde configurée

1. Dans le portail Azure, sélectionnez **Créer une ressource**.

2. Dans Place de marché Azure, sélectionnez **Calcul**, puis sélectionnez une image de machine virtuelle.

3. Configurez la machine virtuelle conformément aux instructions [Windows](../virtual-machines/windows/quick-create-portal.md) ou [Linux](../virtual-machines/linux/quick-create-portal.md).

4. Sous l’onglet **Gestion**, dans **Activer la sauvegarde**, sélectionnez **Activé**.
5. Sauvegarde Azure sauvegarde dans un coffre Recovery Services. Sélectionnez **Créer** si vous n’avez pas de coffre.
6. Acceptez le nom de coffre suggéré ou spécifiez le vôtre.
7. Spécifiez ou créez un groupe de ressources dans lequel se trouvera le coffre. Le groupe de ressources du coffre peut être différent du groupe de ressources de la machine virtuelle.

    ![Activer la sauvegarde pour une machine virtuelle](./media/backup-during-vm-creation/enable-backup.png)

8. Acceptez la stratégie de sauvegarde par défaut ou modifiez les paramètres.
    - Une stratégie de sauvegarde spécifie la fréquence de prise d’instantanés de sauvegarde de la machine virtuelle et la durée pendant laquelle ces copies de sauvegarde sont conservées.
    - La stratégie par défaut sauvegarde la machine virtuelle une fois par jour.
    - Vous pouvez personnaliser votre propre stratégie de sauvegarde pour une machine virtuelle Azure de manière à effectuer des sauvegardes quotidiennes ou hebdomadaires.
    - [Découvrez-en plus](backup-azure-vms-introduction.md#backup-and-restore-considerations) sur les considérations relatives à la sauvegarde des machines virtuelles Azure.
    - [Découvrez-en plus](backup-instant-restore-capability.md) sur la fonctionnalité de restauration instantanée.

      ![Stratégie de sauvegarde par défaut](./media/backup-during-vm-creation/daily-policy.png)

>[!NOTE]
>[SSE et PMK sont les méthodes de chiffrement par défaut](backup-encryption.md) pour les machines virtuelles Azure. Sauvegarde Azure prend en charge la sauvegarde et la restauration de ces machines virtuelles Azure.

## <a name="azure-backup-resource-group-for-virtual-machines"></a>Groupe de ressources Sauvegarde Azure pour les machines virtuelles

Le service de sauvegarde crée un groupe de ressources (RG) distinct du groupe de ressources de la machine virtuelle afin de stocker la collection de points de restauration (RPC). La RPC héberge les points de récupération instantanée des machines virtuelles managées. Le format de nom par défaut du groupe de ressources créé par le service de sauvegarde est le suivant : `AzureBackupRG_<Geo>_<number>`. Par exemple : *AzureBackupRG_northeurope_1*. Vous pouvez maintenant personnaliser le nom du groupe de ressources créé par Sauvegarde Azure.

Points à noter :

1. Vous pouvez utiliser le nom par défaut du groupe de ressources ou le modifier en fonction des besoins de votre entreprise.
2. Vous fournissez le modèle de nom de groupe de ressources comme entrée lors de la création de la stratégie de sauvegarde de machine virtuelle. Le nom du groupe de ressources doit être au format suivant : `<alpha-numeric string>* n <alpha-numeric string>`. « n » est remplacé par un entier (à partir de 1) et utilisé pour la montée en charge si le premier groupe de ressources est saturé. Un groupe de ressources peut avoir un maximum de 600 RPC aujourd’hui.
              ![Choisir un nom lors de la création de la stratégie](./media/backup-during-vm-creation/create-policy.png)
3. Le modèle doit suivre les règles de nommage des groupes de ressources ci-dessous et la longueur totale ne doit pas dépasser la longueur maximale autorisée pour le nom de groupe de ressources.
    1. Le nom d’un groupe de ressources accepte uniquement des caractères alphanumériques, des points, des traits de soulignement, des traits d'union et des parenthèses. Il ne peut pas se terminer sur une période.
    2. Les noms de groupes de ressources peuvent contenir jusqu’à 74 caractères, y compris le nom du groupe de routage et le suffixe.
4. La première `<alpha-numeric-string>` est obligatoire, tandis que la deuxième après « n » est facultative. Cela s’applique uniquement si vous donnez un nom personnalisé. Si vous n’entrez rien dans les deux zones de texte, le nom par défaut est utilisé.
5. Vous pouvez modifier le nom du groupe de ressources en modifiant la stratégie si nécessaire. Si le modèle de nom est modifié, les nouveaux RP seront créés dans le nouveau RG. Toutefois, les anciens RP résideront toujours dans l’ancien RG et ne seront pas déplacés, car la collection RP ne prend pas en charge le déplacement des ressources. Les RP finissent par récupérer le garbage collector à l’expiration des points.
![Changer le nom lors de la modification de la stratégie](./media/backup-during-vm-creation/modify-policy.png)
6. Il est conseillé de ne pas verrouiller le groupe de ressources créé pour une utilisation par le service de sauvegarde.

Afin de configurer le groupe de ressources Sauvegarde Azure pour Machines Virtuelles à l’aide de PowerShell, reportez-vous à [Créer un groupe de ressources Sauvegarde Azure lors de la conservation des instantanés](backup-azure-vms-automation.md#creating-azure-backup-resource-group-during-snapshot-retention).

## <a name="start-a-backup-after-creating-the-vm"></a>Démarrer une sauvegarde après la création de la machine virtuelle

Votre sauvegarde de machine virtuelle s’exécute conformément à votre stratégie de sauvegarde. Toutefois, nous vous recommandons d’exécuter une sauvegarde initiale.

Une fois la machine virtuelle créée, effectuez les étapes suivantes :

1. Dans les propriétés de la machine virtuelle, sélectionnez **Sauvegarde**. L’état de la machine virtuelle est « Sauvegarde initiale en attente » jusqu’à ce que la sauvegarde initiale s’exécute.
2. Sélectionnez **Sauvegarder maintenant** pour exécuter une sauvegarde à la demande.

    ![Exécuter une sauvegarde à la demande](./media/backup-during-vm-creation/run-backup.png)

## <a name="use-a-resource-manager-template-to-deploy-a-protected-vm"></a>Utiliser un modèle Resource Manager pour déployer une machine virtuelle protégée

La procédure précédente explique comment utiliser le portail Azure pour créer une machine virtuelle et la protéger dans un coffre Recovery Services. Pour déployer rapidement une ou plusieurs machines virtuelles et les protéger dans un coffre Recovery Services, consultez le modèle [Déployer une machine virtuelle Windows et activer la sauvegarde](https://azure.microsoft.com/resources/templates/101-recovery-services-create-vm-and-configure-backup/).

## <a name="next-steps"></a>Étapes suivantes

Votre machine virtuelle étant protégée, découvrez comment la gérer et la restaurer.

- [Gérer et superviser des machines virtuelles](backup-azure-manage-vms.md)
- [Restaurer une machine virtuelle](backup-azure-arm-restore-vms.md)

Si vous rencontrez des problèmes, [consultez](backup-azure-vms-troubleshoot.md) le guide de dépannage.
