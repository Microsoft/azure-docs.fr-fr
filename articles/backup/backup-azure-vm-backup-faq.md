---
title: FAQ - Sauvegarder des machines virtuelles Azure
description: Cet article fournit des réponses à des questions courantes sur la sauvegarde des machines virtuelles Azure avec le service Sauvegarde Microsoft Azure.
ms.topic: conceptual
ms.date: 09/17/2019
ms.openlocfilehash: ba2779305302e91f68cb2664c90f53fdf9a9ca55
ms.sourcegitcommit: 273c04022b0145aeab68eb6695b99944ac923465
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97008348"
---
# <a name="frequently-asked-questions-back-up-azure-vms"></a>Forum aux questions - Sauvegarde de machines virtuelles Azure

Cet article fournit des réponses à des questions courantes sur la sauvegarde des machines virtuelles Azure avec le service [Sauvegarde Azure](./backup-overview.md).

## <a name="backup"></a>Backup

### <a name="which-vm-images-can-be-enabled-for-backup-when-i-create-them"></a>Quelles images de machine virtuelle peuvent être activées pour la sauvegarde lorsque je les crée ?

Lorsque vous créez une machine virtuelle, vous pouvez activer la sauvegarde pour les machines virtuelles exécutant des [systèmes d’exploitation pris en charge](backup-support-matrix-iaas.md#supported-backup-actions).

### <a name="why-initial-backup-is-taking-lot-of-time-to-complete"></a>Pourquoi la sauvegarde initiale prend-elle du temps ?

La sauvegarde initiale est toujours une sauvegarde complète et elle dépend du volume des données et du moment où la sauvegarde est traitée. <br>
Pour améliorer les performances de sauvegarde, consultez [Bonnes pratiques de sauvegarde](./backup-azure-vms-introduction.md#best-practices), [Considérations relatives à la sauvegarde](./backup-azure-vms-introduction.md#backup-and-restore-considerations) et [Performances de sauvegarde](./backup-azure-vms-introduction.md#backup-performance).<br>
Si le temps de sauvegarde total des sauvegardes incrémentielles est inférieur à 24 heures, cela peut ne pas être le cas pour la première sauvegarde.

### <a name="is-the-backup-cost-included-in-the-vm-cost"></a>Le coût de la sauvegarde est-il inclus dans le coût de la machine virtuelle ?

Non. Les coûts de sauvegarde sont distincts des coûts d’une machine virtuelle. Apprenez-en davantage sur la [tarification de la Sauvegarde Azure](https://azure.microsoft.com/pricing/details/backup/).

### <a name="which-permissions-are-required-to-enable-backup-for-a-vm"></a>Quelles autorisations sont requises pour activer la sauvegarde sur une machine virtuelle ?

Si vous êtes un contributeur de machines virtuelles, vous pouvez activer la sauvegarde sur la machine virtuelle. Si vous utilisez un rôle personnalisé, vous devez disposer des autorisations suivantes pour activer la sauvegarde sur la machine virtuelle :

- Microsoft.RecoveryServices/Vaults/write
- Microsoft.RecoveryServices/Vaults/read
- Microsoft.RecoveryServices/locations/*
- Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/*/read
- Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/read
- Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/write
- Microsoft.RecoveryServices/Vaults/backupFabrics/backupProtectionIntent/write
- Microsoft.RecoveryServices/Vaults/backupPolicies/read
- Microsoft.RecoveryServices/Vaults/backupPolicies/write

Si votre coffre Recovery Services et votre machine virtuelle ont des groupes de ressources différents, assurez-vous que vous disposez des autorisations d’écriture dans le groupe de ressources du coffre Recovery Services.  

### <a name="does-an-on-demand-backup-job-use-the-same-retention-schedule-as-scheduled-backups"></a>Est-ce qu’un travail de sauvegarde à la demande utilise le même planning de conservation que les autres sauvegardes planifiées ?

Non. Spécifiez la plage de rétention pour un travail de sauvegarde à la demande. Par défaut, il est conservé pendant 30 jours s’il a été déclenché à partir du portail.

### <a name="i-recently-enabled-azure-disk-encryption-on-some-vms-will-my-backups-continue-to-work"></a>J’ai récemment activé Azure Disk Encryption sur certaines machines virtuelles. Est-ce que mes sauvegardes continueront de fonctionner ?

Accordez au service Sauvegarde Azure l’autorisation d’accéder à Key Vault. Spécifiez les autorisations dans PowerShell, comme décrit dans la section **Activer la sauvegarde** de la documentation sur [PowerShell et Sauvegarde Azure](backup-azure-vms-automation.md).

### <a name="i-migrated-vm-disks-to-managed-disks-will-my-backups-continue-to-work"></a>J’ai migré des disques de machine virtuelle vers des disques managés. Est-ce que mes sauvegardes continueront de fonctionner ?

Oui, les sauvegardes fonctionnent de manière transparente. Vous n’avez rien à reconfigurer.

### <a name="why-cant-i-see-my-vm-in-the-configure-backup-wizard"></a>Pourquoi je ne vois pas ma machine virtuelle dans l’Assistant Configuration de la sauvegarde ?

L’Assistant liste uniquement les machines virtuelles situées dans la même région que le coffre et qui ne sont pas déjà en cours de sauvegarde.

### <a name="my-vm-is-shut-down-will-an-on-demand-or-a-scheduled-backup-work"></a>Ma machine virtuelle est arrêtée. Les sauvegardes à la demande et planifiées fonctionneront-elles ?

Oui. Les sauvegardes s’exécutent quand une machine est arrêtée. Le point de récupération est marqué comme étant cohérent en cas d’incident.

### <a name="can-i-cancel-an-in-progress-backup-job"></a>Puis-je annuler un travail de sauvegarde en cours ?

Oui. Vous pouvez annuler un travail de sauvegarde dont la **prise de l’instantané est en cours**. Vous ne pouvez pas annuler un travail si un transfert de données à partir de l’instantané est en cours.

### <a name="i-enabled-a-lock-on-the-resource-group-created-by-azure-backup-service-for-example-azurebackuprg_geo_number-will-my-backups-continue-to-work"></a>J’ai verrouillé le groupe de ressources créé par le service Sauvegarde Azure (p. ex. : `AzureBackupRG_<geo>_<number>`). Est-ce que mes sauvegardes continueront de fonctionner ?

Si vous verrouillez le groupe de ressources créé par le service Sauvegarde Azure, les sauvegardes échoueront, car il existe une limite maximale de 18 points de restauration.

Supprimez le verrou et effacez la collection de points de restauration de ce groupe de ressources pour que les futures sauvegardes réussissent. [Procédez comme suit](backup-azure-troubleshoot-vm-backup-fails-snapshot-timeout.md#clean-up-restore-point-collection-from-azure-portal) pour supprimer la collection de points de restauration.

### <a name="i-have-a-lock-at-the-resource-group-level-that-contains-all-the-resources-related-to-my-virtual-machine-will-my-backup-work"></a>J’ai un verrou au niveau du groupe de ressources qui contient toutes les ressources associées à ma machine virtuelle. Ma sauvegarde fonctionnera-t-elle ?

La Sauvegarde Azure crée un groupe de ressources distinct au format `AzureBackupRG_<geo>_<number>` pour stocker les objets ResourcePointCollections. Étant donné que ce groupe de ressources est détenu par un service, le verrouillage entraînera l’échec des sauvegardes. Les verrous ne peuvent être appliqués qu’aux groupes de ressources créés par le client.

### <a name="does-azure-backup-support-standard-ssd-managed-disks"></a>Le service Sauvegarde Microsoft Azure prend-il en charge les disques managés SSD standard ?

Oui, le service Sauvegarde Azure prend en charge les [disques managés SSD standard](../virtual-machines/disks-types.md#standard-ssd).

### <a name="can-we-back-up-a-vm-with-a-write-accelerator-wa-enabled-disk"></a>Pouvons-nous sauvegarder une machine virtuelle dotée d’un disque avec Accélérateur d’écriture ?

Les instantanés ne peuvent être effectués que sur les disques de données avec Accélérateur d’écriture et non sur les disques de système d’exploitation. Ainsi, seuls les disques de données avec Accélérateur d’écriture peuvent être protégés.

### <a name="i-have-a-vm-with-write-accelerator-wa-disks-and-sap-hana-installed-how-do-i-back-up"></a>J’ai une machine virtuelle dotée de disques avec Accélérateur d’écriture et sur laquelle SAP HANA est installé. Comment faire pour effectuer une sauvegarde ?

Sauvegarde Azure peut sauvegarder le disque de données avec Accélérateur d’écriture. Toutefois, la sauvegarde ne garantit pas la cohérence de la base de données.

Sauvegarde Azure fournit une solution de sauvegarde en continu pour les bases de données SAP HANA avec un RPO de 15 minutes. Elle est certifiée Backint par SAP pour la prise en charge de sauvegarde native à l’aide d’API natives de SAP HANA. En savoir plus [sur la sauvegarde des bases de données SAP HANA dans des machines virtuelles Azure](./sap-hana-db-about.md).

### <a name="what-is-the-maximum-delay-i-can-expect-in-backup-start-time-from-the-scheduled-backup-time-i-have-set-in-my-vm-backup-policy"></a>Quel est le délai maximal de lancement de la sauvegarde à partir de l’heure de sauvegarde planifiée définie dans ma stratégie de sauvegarde de machine virtuelle ?

La sauvegarde planifiée est déclenchée dans les 2 heures suivant l’heure de sauvegarde planifiée. Par exemple, si l’heure de début de la sauvegarde de 100 machines virtuelles est prévue à 2 h, alors la tâche de sauvegarde est en cours pour les 100 machines virtuelles au plus tard à 4 h. Si les sauvegardes planifiées ont été interrompues en raison d’une panne et ont repris ou essayé de reprendre, la sauvegarde peut démarrer en dehors de cette fenêtre de deux heures.

### <a name="what-is-the-minimum-allowed-retention-range-for-a-daily-backup-point"></a>Quelle est la durée de rétention minimale autorisée pour un point de sauvegarde quotidien ?

La stratégie de sauvegarde de la machine virtuelle Azure prend en charge une durée de rétention minimale allant de 7 à 9 999 jours. Toute modification apportée à une stratégie de sauvegarde de machine virtuelle existante avec une plage de rétention inférieure à sept jours nécessitera une mise à jour pour répondre à la durée de rétention minimale de sept jours.

### <a name="what-happens-if-i-change-the-case-of-the-name-of-my-vm-or-my-vm-resource-group"></a>Que se passe-t-il si je modifie la casse du nom de ma machine virtuelle ou de mon groupe de ressources de machine virtuelle ?

Si vous modifiez la casse (en majuscules ou en minuscules) du nom de votre machine virtuelle ou groupe de ressources de machine virtuelle, la casse du nom de l’élément de sauvegarde ne change pas. Toutefois, il s’agit du comportement qui est attendu de Sauvegarde Azure. Le changement de casse n’apparaît pas dans l’élément de sauvegarde, mais il est mis à jour au niveau du serveur principal.

### <a name="can-i-back-up-or-restore-selective-disks-attached-to-a-vm"></a>Puis-je sauvegarder ou restaurer certains disques attachés à une machine virtuelle ?

La sauvegarde Azure prend désormais en charge la sauvegarde et la restauration sélectives de disque à l’aide de la solution de sauvegarde de machine virtuelle Azure. Pour plus d’informations, consultez [Sauvegarde et restauration sélectives sur disque pour les machines virtuelles Azure](selective-disk-backup-restore.md).

### <a name="are-managed-identities-preserved-if-a-tenant-change-occurs-during-backup"></a>Les identités managées sont-elles préservées si une modification de locataire se produit pendant la sauvegarde ?

Si des [modifications du locataire](/azure/devops/organizations/accounts/change-azure-ad-connection) se produisent, vous devez désactiver, puis réactiver les [identités managées](../active-directory/managed-identities-azure-resources/overview.md) pour que les sauvegardes fonctionnent à nouveau.

## <a name="restore"></a>Restaurer

### <a name="how-do-i-decide-whether-to-restore-disks-only-or-a-full-vm"></a>Comment déterminer si je dois restaurer des disques uniquement ou une machine virtuelle complète ?

Considérez qu’une restauration de machine virtuelle est une option de création rapide d’une machine virtuelle Azure. Cette option modifie les noms des disques, les conteneurs utilisés par les disques, les adresses IP publiques et les noms d’interface réseau. Cette modification permet de conserver des ressources uniques lors de la création d’une machine virtuelle. La machine virtuelle n’est pas ajoutée à un groupe à haute disponibilité.

L’option de restauration de disque convient dans les cas suivants :

- Vous voulez personnaliser la machine virtuelle créée. Par exemple, vous voulez modifier sa taille.
- Vous voulez ajouter des paramètres de configuration qui n’existaient pas au moment de la sauvegarde.
- Vous voulez contrôler la convention d’affectation de noms pour les ressources créées.
- Vous voulez ajouter la machine virtuelle à un groupe à haute disponibilité.
- Vous voulez ajouter tout autre paramètre devant être configuré à l’aide de PowerShell ou d’un modèle.

### <a name="can-i-restore-backups-of-unmanaged-vm-disks-after-i-upgrade-to-managed-disks"></a>Puis-je restaurer des sauvegardes de disques de machine virtuelle non managés après avoir effectué une mise à niveau vers des disques managés ?

Oui, vous pouvez utiliser des sauvegardes effectuées avant la migration des disques non managés vers des disques managés.

### <a name="how-do-i-restore-a-vm-to-a-restore-point-before-the-vm-was-migrated-to-managed-disks"></a>Comment restaurer une machine virtuelle à un point de restauration antérieur à la migration de la machine virtuelle vers des disques managés ?

Le processus de restauration ne change pas. Si le point de récupération est un point dans le temps où la machine virtuelle avait des disques non managés, vous pouvez [restaurer les disques en tant que disques non managés](tutorial-restore-disk.md#unmanaged-disks-restore). Si la machine virtuelle avait des disques managés, vous pouvez [restaurer les disques en tant que disques managés](tutorial-restore-disk.md#managed-disk-restore). Vous pouvez ensuite [créer une machine virtuelle à partir de ces disques](tutorial-restore-disk.md#create-a-vm-from-the-restored-disk).

[Découvrez](backup-azure-vms-automation.md#restore-an-azure-vm) comment effectuer ces opérations dans PowerShell.

### <a name="if-the-restore-fails-to-create-the-vm-what-happens-to-the-disks-included-in-the-restore"></a>Si la restauration ne permet pas de créer la machine virtuelle, qu’advient-il des disques inclus dans la restauration ?

Dans le cas d’une restauration de machine virtuelle managée, même si la création de la machine échoue, les disques sont toujours restaurés.

### <a name="can-i-restore-a-vm-thats-been-deleted"></a>Puis-je restaurer une machine virtuelle qui a été supprimée ?

Oui. Même si vous supprimez la machine virtuelle, vous pouvez accéder à l’élément de sauvegarde correspondant dans le coffre et effectuer une restauration à partir d’un point de récupération.

### <a name="how-do-i-restore-a-vm-to-the-same-availability-sets"></a>Comment restaurer une machine virtuelle dans les mêmes groupes à haute disponibilité ?

Pour des machines virtuelles Azure à disque managé, la restauration dans des groupes à haute disponibilité est activée en proposant une option dans le modèle lors de la restauration en tant que disque managé. Ce modèle dispose du paramètre d’entrée appelé **Groupes à haute disponibilité**.

### <a name="how-do-we-get-faster-restore-performances"></a>Comment accélérer les performances de restauration ?

La capacité [Restauration instantanée](backup-instant-restore-capability.md) permet des sauvegardes plus rapides et des restaurations instantanées à partir des captures instantanées.

### <a name="what-happens-when-we-change-the-key-vault-settings-for-the-encrypted-vm"></a>Que se passe-t-il lorsque nous modifions les paramètres du coffre de clés pour la machine virtuelle chiffrée ?

Une fois que vous avez modifié les paramètres du coffre de clés pour la machine virtuelle chiffrée, les sauvegardes continuent de fonctionner avec le nouvel ensemble d’informations. Toutefois, après la restauration à partir d’un point de récupération antérieur à la modification, vous devrez restaurer les secrets du coffre de clés avant de pouvoir créer la machine virtuelle à partir de celui-ci. Pour plus d’informations, consultez cet [article](./backup-azure-restore-key-secret.md).

Des opérations telles que la restauration de secret/clé ne nécessitent pas cette étape et le même coffre de clés peut être utilisé après restauration.

### <a name="can-i-access-the-vm-once-restored-due-to-a-vm-having-broken-relationship-with-domain-controller"></a>Puis-je accéder à la machine virtuelle après restauration en raison d’une rupture de la relation entre la machine virtuelle et le contrôleur de domaine ?

Oui, vous accédez à la machine virtuelle après restauration en raison d’une rupture de la relation entre la machine virtuelle et le contrôleur de domaine. Pour plus d’informations, consultez cet [article](./backup-azure-arm-restore-vms.md#post-restore-steps).

### <a name="can-i-cancel-an-in-progress-restore-job"></a>Est-ce que je peux annuler un travail de restauration en cours ?
Non, vous ne pouvez pas annuler le travail de restauration en cours.

### <a name="why-restore-operation-is-taking-long-time-to-complete"></a>Pourquoi l’opération de restauration prend-elle du temps ?

La durée totale de la restauration varie selon les opérations d'entrée/sortie par seconde (IOPS) et le débit du compte de stockage. La durée totale de la restauration peut être affectée si le compte de stockage cible est chargé avec d'autres opérations de lecture et d'écriture de l'application. Pour améliorer l'opération de restauration, sélectionnez un compte de stockage qui n'est pas chargé avec d'autres données d'application.

### <a name="how-do-we-handle-create-new-virtual-machine-restore-type-conflicts-with-governance-policies"></a>Comment pouvons-nous traiter les conflits de type de restauration « Créer une machine virtuelle » avec des stratégies de gouvernance ?

Sauvegarde Azure utilise des disques « attach » à partir de points de récupération et ne prend pas en considération vos galeries ou références d’images. Par conséquent, dans la stratégie, vous pouvez vérifier que « storageProfile.osDisk.createOption est égal à Attach » ; la condition du script est alors :

`if (storageProfile.osDisk.createOption == "Attach") then { exclude <Policy> }`

## <a name="manage-vm-backups"></a>Gérer les sauvegardes de machine virtuelle

### <a name="what-happens-if-i-modify-a-backup-policy"></a>Que se passe-t-il si je modifie une stratégie de sauvegarde ?

La machine virtuelle est sauvegardée selon la planification et les paramètres de conservation indiqués dans la stratégie nouvelle ou modifiée.

- Si la conservation est prolongée, les points de récupération existants sont marqués et conservés conformément à la nouvelle stratégie.
- Si la conservation est réduite, ils sont marqués de sorte à être nettoyés lors de la prochaine tâche de nettoyage, puis supprimés.

### <a name="how-do-i-move-a-vm-backed-up-by-azure-backup-to-a-different-resource-group"></a>Comment déplacer une machine virtuelle sauvegardée par Sauvegarde Azure dans un autre groupe de ressources ?

1. Interrompez temporairement la sauvegarde et conservez les données de sauvegarde.
2. Pour déplacer des machines virtuelles configurées avec Sauvegarde Azure, effectuez les étapes suivantes :

   1. Recherchez l’emplacement de votre machine virtuelle.
   2. Recherchez un groupe de ressources dont le modèle de nommage est le suivant : `AzureBackupRG_<location of your VM>_1`. Par exemple, *AzureBackupRG_westus2_1*.
   3. Dans le Portail Azure, cochez la case **Afficher les types masqués**.
   4. Recherchez la ressource de type **Microsoft. Microsoft.Compute/restorePointCollections** dont le modèle de nommage est `AzureBackup_<name of your VM that you're trying to move>_###########`.
   5. Supprimez cette ressource. Cette opération supprime uniquement les points de récupération instantanée, et non les données sauvegardées dans le coffre.
   6. Une fois l’opération de suppression terminée, vous pouvez déplacer votre machine virtuelle.

3. Déplacez la machine virtuelle vers le groupe de ressources cible.
4. Reprenez la sauvegarde.

Vous pouvez restaurer la machine virtuelle à partir des points de restauration disponibles créés avant l’opération de déplacement.

### <a name="what-happens-after-i-move-a-vm-to-a-different-resource-group"></a>Que se passe-t-il lorsque je déplace une machine virtuelle vers un autre groupe de ressources ?

Une fois qu’une machine virtuelle est déplacée vers un autre groupe de ressources, il s’agit d’une nouvelle machine virtuelle en ce qui concerne la sauvegarde Azure.

Après avoir déplacé la machine virtuelle vers un nouveau groupe de ressources, vous pouvez reprotéger la machine virtuelle dans le même coffre ou dans un autre coffre. Étant donné qu’il s’agit d’une nouvelle machine virtuelle pour la sauvegarde Azure, vous êtes facturé séparément.

Les points de restauration de l’ancienne machine virtuelle seront disponibles pour la restauration, si nécessaire. Si vous n’avez pas besoin de ces données de sauvegarde, vous pouvez arrêter la protection de votre ancienne machine virtuelle avec des données supprimées.

### <a name="is-there-a-limit-on-number-of-vms-that-can-be-associated-with-the-same-backup-policy"></a>Existe-t-il un nombre limite de machines virtuelles pouvant être associées à la même stratégie de sauvegarde ?

Oui, il existe une limite de 100 machines virtuelles pouvant être associées à la même stratégie de sauvegarde à partir du portail. Pour plus de 100 machines virtuelles, nous recommandons de créer plusieurs stratégies de sauvegarde avec la même planification ou une planification différente.

### <a name="how-can-i-view-the-retention-settings-for-my-backups"></a>Comment puis-je consulter les paramètres de rétention de mes sauvegardes ?

Actuellement, vous pouvez consulter les paramètres de rétention au niveau d’un élément de sauvegarde (machine virtuelle) en fonction de la stratégie de sauvegarde qui est attribuée à la machine virtuelle.

Pour voir les paramètres de rétention de vos sauvegardes, vous pouvez accéder au [tableau de bord](./backup-azure-manage-vms.md#view-vms-on-the-dashboard) des éléments de sauvegarde de votre machine virtuelle dans le portail Azure. En sélectionnant le lien vers sa politique de sauvegarde, vous pouvez visualiser la durée de rétention de tous les points de rétention quotidiens, hebdomadaires, mensuels et annuels associés à la machine virtuelle.

Vous pouvez également utiliser [Explorateur de sauvegarde](./monitor-azure-backup-with-backup-explorer.md) pour afficher les paramètres de rétention de toutes vos machines virtuelles dans un seul volet transparent. Accédez à Explorateur de sauvegarde à partir de n’importe quel coffre Recovery Services, rendez-vous dans l’onglet **Éléments de sauvegarde** et sélectionnez l’affichage avancé pour voir les informations de rétention détaillées de chaque machine virtuelle.
