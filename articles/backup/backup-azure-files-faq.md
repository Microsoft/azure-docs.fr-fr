---
title: FAQ sur la sauvegarde des fichiers Azure
description: Cet article fournit des réponses à des questions courantes sur la protection de vos partages de fichiers Azure avec le service Sauvegarde Microsoft Azure.
ms.date: 04/22/2020
ms.topic: conceptual
ms.openlocfilehash: e2b6afb25e189ee2848f25c0ba59d843baf37090
ms.sourcegitcommit: 541bb46e38ce21829a056da880c1619954678586
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/11/2020
ms.locfileid: "91940833"
---
# <a name="questions-about-backing-up-azure-files"></a>Questions sur la sauvegarde des fichiers Azure

Cet article répond aux questions courantes sur la sauvegarde des fichiers Azure. Certaines réponses comportent des liens vers les articles présentant des informations complètes. Vous pouvez également publier des questions sur le service Sauvegarde Azure dans la [page de questions Microsoft Q&A](/answers/topics/azure-backup.html).

Pour analyser rapidement les sections de cet article, utilisez les liens sur la droite, en dessous de **Dans cet article**.

## <a name="configuring-the-backup-job-for-azure-files"></a>Configuration de la tâche de sauvegarde pour les fichiers Azure

### <a name="why-cant-i-see-some-of-my-storage-accounts-that-i-want-to-protect-which-contain-valid-azure-file-shares"></a>Pourquoi ne puis-je pas voir certains des comptes de stockage que je souhaite protéger et contenant des partages de fichiers Azure valides ?

Consultez la [matrice de prise en charge de la sauvegarde des partages de fichiers Azure](azure-file-share-support-matrix.md) pour vérifier que le compte de stockage appartient à l'un des types de compte de stockage pris en charge. Il est également possible que le compte de stockage que vous recherchez soit déjà protégé ou inscrit auprès d’un autre coffre. [Annulez l’inscription du compte de stockage](manage-afs-backup.md#unregister-a-storage-account) auprès du coffre pour découvrir le compte de stockage dans d’autres coffres pour la protection.

### <a name="why-cant-i-see-some-of-my-azure-file-shares-in-the-storage-account-when-im-trying-to-configure-backup"></a>Pourquoi ne puis-je pas voir certains de mes partages de fichiers Azure dans le compte de stockage lorsque j’essaie de configurer la sauvegarde ?

Vérifiez si le partage de fichiers Azure est déjà protégé dans le même coffre Recovery Services ou s’il a été supprimé récemment.

### <a name="can-i-protect-file-shares-connected-to-a-sync-group-in-azure-files-sync"></a>Puis-je protéger les partages de fichiers connectés à un groupe de synchronisation dans Azure Files Sync ?

Oui. La protection des partages de fichiers Azure connectés à des groupes de synchronisation est activée.

### <a name="when-trying-to-back-up-file-shares-i-selected-a-storage-account-to-discover-the-file-shares-in-it-however-i-didnt-protect-them-how-do-i-protect-these-file-shares-with-any-other-vault"></a>Quand j’ai essayé de sauvegarder des partages de fichiers, j’ai sélectionné un compte de stockage pour découvrir les partages de fichiers qu’il contenait. Toutefois, je ne les ai pas protégés. Comment protéger ces partages de fichiers avec un autre coffre ?

Lorsque vous essayez de sauvegarder, le fait de sélectionner un compte de stockage pour détecter les partages de fichiers dans celui-ci permet d’enregistrer le compte de stockage avec le coffre à partir duquel cette action est effectuée. Si vous choisissez de protéger les partages de fichiers avec un autre coffre, [annulez l’enregistrement](manage-afs-backup.md#unregister-a-storage-account) du compte de stockage sélectionné à partir de ce coffre.

### <a name="why-cant-i-change-the-vault-to-configure-backup-for-the-file-share"></a>Pourquoi ne puis-je pas modifier le coffre pour configurer la sauvegarde pour le partage de fichiers ?

Si le compte de stockage est déjà inscrit auprès d’un coffre ou si d’autres partages de fichiers dans le compte de stockage sont protégés à l’aide d’un coffre, vous n’avez pas la possibilité de le modifier. Tous les partages de fichiers dans un compte de stockage peuvent uniquement être protégés par le même coffre. Si vous souhaitez modifier le coffre, vous devez [arrêter la protection de tous les partages de fichiers dans le compte de stockage](manage-afs-backup.md#stop-protection-on-a-file-share) à partir du coffre connecté, [annuler l’inscription](manage-afs-backup.md#unregister-a-storage-account) du compte de stockage, puis choisir un autre coffre pour la protection.

### <a name="can-i-change-the-vault-to-which-i-back-up-my-file-shares"></a>Puis-je modifier le coffre dans lequel je sauvegarde mes partages de fichiers ?

Oui. Toutefois, vous devrez [arrêter la protection sur un partage de fichiers](manage-afs-backup.md#stop-protection-on-a-file-share) à partir du coffre connecté, [désinscrire](manage-afs-backup.md#unregister-a-storage-account) ce compte de stockage, puis le protéger à partir d’un autre coffre.

### <a name="can-i-protect-two-different-file-shares-from-the-same-storage-account-to-different-vaults"></a>Puis-je protéger deux partages de fichiers différents du même compte de stockage dans différents coffres ?

Non. Tous les partages de fichiers dans un compte de stockage peuvent uniquement être protégés par le même coffre.

## <a name="backup"></a>Sauvegarde

### <a name="what-should-i-do-if-my-backups-start-failing-due-to-the-maximum-limit-reached-error"></a>Que dois-je faire si mes sauvegardes échouent en raison d’une erreur de limite maximale atteinte ?

À tout moment, vous pouvez disposer de 200 instantanés pour un partage de fichiers. La limite inclut des instantanés pris par Azure Backup, comme défini dans votre stratégie. Si vos sauvegardes échouent une fois la limite atteinte, supprimez des instantanés à la demande pour réussir les prochaines sauvegardes.

## <a name="restore"></a>Restaurer

### <a name="can-i-recover-from-a-deleted-azure-file-share"></a>Puis-je effectuer une restauration d’un partage de fichiers Azure supprimé ?

Si le partage de fichiers est dans l’état de suppression réversible, vous devez d’abord annuler la suppression du partage de fichiers pour effectuer l’opération de restauration. L’opération d’annulation de la suppression mettra le partage de fichiers dans l’état sauvegardé. Vous pourrez alors effectuer une restauration à n’importe quel point dans le temps. Pour découvrir comment annuler la suppression de votre partage de fichiers, cliquez sur [ce lien](../storage/files/storage-files-enable-soft-delete.md?tabs=azure-portal#restore-soft-deleted-file-share) ou consultez le [script d’annulation de la suppression de partage de fichiers](./scripts/backup-powershell-script-undelete-file-share.md). En cas de suppression définitive du partage de fichiers, vous ne pourrez pas restaurer le contenu et les captures instantanées.

### <a name="can-i-restore-from-backups-if-i-stopped-protection-on-an-azure-file-share"></a>Puis-je effectuer une restauration à partir de sauvegardes après avoir arrêté la protection sur un partage de fichiers Azure ?

Oui. Si vous avez choisi l’option **Conserver les données de sauvegarde** lors de l’arrêt de la protection, vous pouvez restaurer à partir de tous les points de restauration existants.

### <a name="what-happens-if-i-cancel-an-ongoing-restore-job"></a>Que se passe-t-il si j’annule un travail de restauration en cours ?

Si un travail de restauration en cours est annulé, le processus de restauration s’arrête et tous les fichiers restaurés avant l’annulation restent dans la destination configurée (emplacement d’origine ou autre) sans aucun restauration.

## <a name="manage-backup"></a>Gérer la sauvegarde

### <a name="can-i-use-powershell-to-configuremanagerestore-backups-of-azure-file-shares"></a>Puis-je utiliser PowerShell pour configurer/gérer/restaurer des sauvegardes de partages de fichiers Azure ?

Oui. Reportez-vous à la documentation détaillée [ici](backup-azure-afs-automation.md).

### <a name="can-i-access-the-snapshots-taken-by-azure-backups-and-mount-them"></a>Puis-je accéder aux instantanés effectués par les sauvegardes Azure et les monter ?

Tous les instantanés pris par Azure Backup sont accessibles par l’option Voir les instantanés dans le portail, PowerShell ou CLI. Pour en savoir plus sur les instantanés de partage de fichiers Azure, consultez [Vue d’ensemble des instantanés de partage pour Azure Files](../storage/files/storage-snapshots-files.md).

### <a name="what-happens-after-i-move-a-backed-up-file-share-to-a-different-subscription"></a>Que se passe-t-il après le déplacement d’un partage de fichiers sauvegardé vers un autre abonnement ?

Une fois qu’un partage de fichiers est déplacé vers un autre abonnement, il est considéré comme un nouveau partage de fichiers par Sauvegarde Azure. Les étapes recommandées sont les suivantes :
 
Scénario : Supposons que vous avez un partage de fichiers *FS1* dans l’abonnement *S1* et qu’il est protégé à l’aide du coffre *V1*. Vous souhaitez maintenant déplacer votre partage de fichiers vers l’abonnement *S2*.
 
1.  Déplacez le compte de stockage et le partage de fichiers (FS1) souhaités vers l’autre abonnement (S2).
2.  Dans le coffre V1, déclenchez l’arrêt de la protection avec l’opération de suppression de données pour FS1.
3.  Désinscrivez le compte de stockage qui héberge FS1 du coffre V1.
4.  Reconfigurez la sauvegarde pour FS1, désormais déplacée dans S2, avec un coffre (V2) dans l’abonnement S2. 
 
Notez qu’après la reconfiguration de la sauvegarde avec V2, les instantanés qui ont été pris avec V1 ne seront plus gérés par Sauvegarde Azure. Vous devrez donc supprimer ces instantanés manuellement en fonction de vos besoins.

### <a name="can-i-move-my-backed-up-file-share-to-a-different-resource-group"></a>Puis-je déplacer mon partage de fichiers sauvegardé vers un autre groupe de ressources ?
 
Oui, vous pouvez déplacer votre partage de fichiers sauvegardé vers un autre groupe de ressources. Toutefois, vous devrez reconfigurer la sauvegarde pour le partage de fichiers, car il sera traité comme une nouvelle ressource par Sauvegarde Azure. En outre, les instantanés qui ont été créés avant le déplacement du groupe de ressources ne seront plus gérés par Sauvegarde Azure. Vous devrez donc supprimer ces instantanés manuellement en fonction de vos besoins.

### <a name="what-is-the-maximum-retention-i-can-configure-for-backups"></a>Quelle est la durée de conservation maximale configurable pour les sauvegardes ?

Consultez la [matrice de prise en charge](azure-file-share-support-matrix.md) pour plus d’informations sur la rétention maximale. Sauvegarde Azure calcule en temps réel le nombre d’instantanée quand vous entrez les valeurs de rétention lors de la configuration de la stratégie de sauvegarde. Dès que le nombre d’instantanés correspondant à vos valeurs de rétention définies dépasse 200, le portail affiche un avertissement vous demandant d’ajuster vos valeurs de rétention. Cela vous évite de dépasser la limite du nombre maximal d’instantanés pris en charge à tout moment par Azure Files pour un partage de fichiers.

### <a name="what-is-the-impact-on-existing-recovery-points-and-snapshots-when-i-modify-the-backup-policy-for-an-azure-file-share-to-switch-from-daily-policy-to-gfs-policy"></a>Quel est l’impact sur les points de récupération et les instantanés existants quand je passe d’une stratégie de sauvegarde quotidienne à une stratégie GFS pour un partage de fichiers Azure ?

Quand vous passez d’une stratégie de sauvegarde quotidienne à une stratégie GFS (en ajoutant une conservation hebdomadaire/mensuelle/annuelle), le comportement est le suivant :

- **Conservation** : Si vous ajoutez une conservation hebdomadaire/mensuelle/annuelle dans le cadre de la modification de la stratégie, tous les points de récupération futurs créés dans le cadre de la sauvegarde planifiée seront marqués selon la nouvelle stratégie. Tous les points de récupération existants seront toujours considérés comme quotidiens et ne seront donc pas marqués comme étant hebdomadaires/mensuels/annuels.

- **Nettoyage des instantanés et des points de récupération** :

  - Si la conservation quotidienne est étendue, la date d’expiration des points de récupération existants est mise à jour en fonction de la valeur de conservation quotidienne configurée dans la nouvelle stratégie.
  - Si la conservation quotidienne est réduite, les points de récupération et les instantanés existants sont marqués pour suppression au cours du prochain travail d’exécution de nettoyage en fonction de la valeur de conservation quotidienne configurée dans la nouvelle stratégie, puis supprimés.

Voici un exemple qui montre comment cela fonctionne :

#### <a name="existing-policy-p1"></a>Stratégie existante [P1]

|Type de conservation |Planifier |Rétention  |
|---------|---------|---------|
|Quotidien    |    Tous les jours à 20:00    |  100 jours       |

#### <a name="new-policy-modified-p1"></a>Nouvelle stratégie [P1 modifiée]

| Type de conservation | Planifier                       | Rétention |
| -------------- | ------------------------------ | --------- |
| Quotidien          | Tous les jours à 21:00              | 50 jours   |
| Hebdomadaire         | Dimanche à 21:00              | 3 semaines   |
| Mensuelle        | Dernier lundi à 21:00         | 1 mois   |
| Annuelle         | Troisième dimanche de janvier à 21:00 | 4 ans   |

#### <a name="impact"></a>Impact

1. La date d’expiration des points de récupération existants est ajustée en fonction de la valeur de conservation quotidienne de la nouvelle stratégie, à savoir 50 jours. Tout point de récupération antérieur à 50 jours est donc marqué pour suppression.

2. Les points de récupération existants ne sont pas marqués comme étant hebdomadaires/mensuels/annuels selon la nouvelle stratégie.

3. Toutes les futures sauvegardes seront déclenchées en fonction de la nouvelle planification, c’est-à-dire à 21:00.

4. La date d’expiration de tous les futurs points de récupération sera alignée sur la nouvelle stratégie.

>[!NOTE]
>Les modifications apportées à la stratégie affecteront uniquement les points de récupération créés dans le cadre de l’exécution de la sauvegarde planifiée. Pour les sauvegardes à la demande, la conservation est déterminée par la valeur **Conserver la sauvegarde jusqu’au** spécifiée au moment de la sauvegarde.

### <a name="what-is-the-impact-on-existing-recovery-points-when-i-modify-an-existing-gfs-policy"></a>Quel est l’impact sur les points de récupération existants quand je modifie une stratégie GFS existante ?

Quand une nouvelle stratégie est appliquée à des partages de fichiers, toutes les futures sauvegardes planifiées seront exécutées conformément à la planification configurée dans la stratégie modifiée.  La conservation de tous les points de récupération existants est alignée en fonction des nouvelles valeurs de conservation configurées. Si la conservation est prolongée, les points de récupération existants sont marqués comme conservés conformément à la nouvelle stratégie. Si la conservation est réduite, ils sont marqués comme à nettoyer dans le prochain travail de nettoyage, puis supprimés.

Voici un exemple qui montre comment cela fonctionne :

#### <a name="existing-policy-p2"></a>Stratégie existante [P2]

| Type de conservation | Planifier           | Rétention |
| -------------- | ------------------ | --------- |
| Quotidien          | Tous les jours à 20:00 | 50 jours   |
| Hebdomadaire         | Dernier lundi à 20:00  | 3 semaines   |

#### <a name="new-policy-modified-p2"></a>Nouvelle stratégie [P2 modifiée]

| Type de conservation | Planifier               | Rétention |
| -------------- | ---------------------- | --------- |
| Quotidien          | Tous les jours à 21:00     | 10 jours   |
| Hebdomadaire         | Tous les lundis à 21:00      | 2 semaines   |
| Mensuelle        | Dernier lundi à 21:00 | 2 mois  |

#### <a name="impact-of-change"></a>Impact de la modification

1. La date d’expiration des points de récupération quotidiens existants sera ajustée en fonction de la nouvelle valeur de conservation quotidienne, à savoir 10 jours. Tous les points de récupération quotidiens de plus de 10 jours seront donc supprimés.

2. La date d’expiration des points de récupération hebdomadaires existants sera ajustée en fonction de la nouvelle valeur de conservation hebdomadaire, à savoir deux semaines. Tous les points de récupération hebdomadaires de plus de deux semaines seront donc supprimés.

3. Les points de récupération mensuels ne seront créés que dans le cadre des futures sauvegardes en fonction de la configuration de la nouvelle stratégie.

4. La date d’expiration de tous les futurs points de récupération sera alignée sur la nouvelle stratégie.

>[!NOTE]
>Les modifications apportées à la stratégie affecteront uniquement les points de récupération créés dans le cadre de la sauvegarde planifiée. Pour les sauvegardes à la demande, la conservation est déterminée par la valeur **Conserver la sauvegarde jusqu’au** spécifiée au moment de la sauvegarde.

## <a name="next-steps"></a>Étapes suivantes

- [Résoudre des problèmes lors de la sauvegarde de partages de fichiers Azure](troubleshoot-azure-files.md)
