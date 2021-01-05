---
title: Résoudre des problèmes de sauvegarde de partages de fichiers Azure
description: Cet article contient des informations de dépannage concernant les problèmes qui se produisent lors de la protection de vos partages de fichiers Azure.
ms.date: 02/10/2020
ms.topic: troubleshooting
ms.openlocfilehash: 4908b8ed97bad43d9d24427660a8691ee43d7eaf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89376976"
---
# <a name="troubleshoot-problems-while-backing-up-azure-file-shares"></a>Résoudre des problèmes lors de la sauvegarde de partages de fichiers Azure

Cet article fournit des informations pour vous aider à résoudre des problèmes que vous rencontrez lors de la configuration de la sauvegarde ou de la restauration de partages de fichiers Azure à l’aide du service Sauvegarde Azure.

## <a name="common-configuration-issues"></a>Problèmes de configuration courants

### <a name="could-not-find-my-storage-account-to-configure-backup-for-the-azure-file-share"></a>Mon compte de stockage pour configurer la sauvegarde du partage de fichiers Azure est introuvable

- Attendez la fin de la découverte.
- Vérifiez si un partage de fichiers du compte de stockage est déjà protégé par un autre coffre Recovery Services.

  >[!NOTE]
  >Tous les partages de fichiers dans un compte de stockage peuvent être protégés sous un seul coffre Recovery Services. [Ce script](scripts/backup-powershell-script-find-recovery-services-vault.md) vous aide à trouver le coffre Recovery Services dans lequel votre compte de stockage est inscrit.

- Assurez-vous que le partage de fichiers ne figure pas dans un des compte de stockage non pris en charge. Pour trouver les comptes de stockage pris en charge, vous pouvez consulter la [Matrice de prise en charge pour la sauvegarde de partage de fichiers Azure](azure-file-share-support-matrix.md).
- Vérifiez que la longueur combinée du nom du compte de stockage et du nom du groupe de ressources ne dépasse pas 84 caractères en cas de nouveaux comptes de stockage, et 77 caractères dans le cas de comptes de stockage classiques.
- Vérifiez les paramètres de pare-feu du compte de stockage pour vous assurer que l’option autorisant les services Microsoft approuvés à accéder au compte de stockage est activée.

### <a name="error-in-portal-states-discovery-of-storage-accounts-failed"></a>Erreur dans le portail indiquant que la découverte des comptes de stockage a échoué

Si vous avez un abonnement partenaire (compatible fournisseur de solutions Cloud), ignorez l’erreur. Si votre abonnement n’est pas compatible fournisseur de solutions Cloud, et que vos comptes de stockage ne peuvent pas être découverts, contactez le support technique.

### <a name="selected-storage-account-validation-or-registration-failed"></a>Échec de la validation ou de l’inscription du compte de stockage sélectionné

Réessayez l’inscription. Si le problème persiste, contactez le support technique.

### <a name="could-not-list-or-find-file-shares-in-the-selected-storage-account"></a>Impossible de répertorier ou de trouver des partages de fichiers dans le compte de stockage sélectionné

- Assurez-vous que le compte de stockage existe dans le groupe de ressources et qu’il n’a pas été supprimé ou déplacé après la dernière validation ou le dernier enregistrement dans le coffre.
- Assurez-vous que le partage de fichiers que vous souhaitez protéger n’a pas été supprimé.
- Assurez-vous que le compte de stockage est bien pris en charge pour la sauvegarde de partage de fichiers. Pour trouver les comptes de stockage pris en charge, vous pouvez consulter la [Matrice de prise en charge pour la sauvegarde de partage de fichiers Azure](azure-file-share-support-matrix.md).
- Vérifiez si le partage de fichiers est déjà protégé dans le même coffre Recovery Services.

### <a name="backup-file-share-configuration-or-the-protection-policy-configuration-is-failing"></a>La configuration du partage de fichiers de sauvegarde (ou la configuration de la stratégie de protection) échoue

- Réessayez l’opération pour voir si le problème persiste.
- Assurez-vous que le partage de fichiers que vous souhaitez protéger n’a pas été supprimé.
- Si vous essayez de protéger plusieurs partages de fichiers en même temps, et si l’échec touche certains partages de fichiers, réessayez de configurer la sauvegarde pour les partages de fichiers ayant échoué.

### <a name="unable-to-delete-the-recovery-services-vault-after-unprotecting-a-file-share"></a>Impossible de supprimer le coffre Recovery Services après avoir ôté la protection d’un partage de fichiers

Dans le portail Azure, ouvrez votre **coffre** > **Infrastructure de sauvegarde** > **Comptes de stockage**. Sélectionnez **Désinscrire** pour supprimer les comptes de stockage du coffre Recovery Services.

>[!NOTE]
>Vous ne pouvez supprimer un coffre Recovery Services qu’après avoir annulé l’inscription de tous les comptes de stockage inscrits auprès du coffre.

## <a name="common-backup-or-restore-errors"></a>Erreurs courantes de sauvegarde ou de restauration

>[!NOTE]
>Reportez-vous à [ce document](./backup-rbac-rs-vault.md#minimum-role-requirements-for-the-azure-file-share-backup) pour vous assurer que vous disposez des autorisations suffisantes pour effectuer des opérations de sauvegarde ou de restauration.

### <a name="filesharenotfound--operation-failed-as-the-file-share-is-not-found"></a>FileShareNotFound : échec de l’opération parce que le partage de fichiers est introuvable

Code d’erreur : FileShareNotFound

Message d’erreur : L’opération a échoué parce que le partage de fichiers est introuvable

Assurez-vous que le partage de fichiers que vous tentez de protéger n’a pas été supprimé.

### <a name="usererrorfileshareendpointunreachable--storage-account-not-found-or-not-supported"></a>UserErrorFileShareEndpointUnreachable : compte de stockage introuvable ou non pris en charge

Code d’erreur : UserErrorFileShareEndpointUnreachable

Message d’erreur : Compte de stockage introuvable ou non pris en charge

- Assurez-vous que le compte de stockage existe dans le groupe de ressources et n’a pas été supprimé ou retiré de celui-ci après la dernière validation.

- Assurez-vous que le compte de stockage est bien pris en charge pour la sauvegarde de partage de fichiers.

### <a name="afsmaxsnapshotreached--you-have-reached-the-max-limit-of-snapshots-for-this-file-share-you-will-be-able-to-take-more-once-the-older-ones-expire"></a>AFSMaxSnapshotReached : Vous avez atteint la limite maximale d’instantanés pour ce partage de fichiers, vous serez en mesure d’en effectuer davantage une fois que les anciens ont expiré

Code d’erreur : AFSMaxSnapshotReached

Message d’erreur : Vous avez atteint la limite maximale d’instantanés pour ce partage de fichiers, vous serez en mesure d’en effectuer davantage une fois que les anciens ont expiré.

- Cette erreur peut se produire lorsque vous créez plusieurs sauvegardes à la demande pour un partage de fichiers.
- Il existe une limite de 200 instantanés par partage de fichiers, y compris ceux pris par Sauvegarde Azure. Les anciennes sauvegardes (ou instantanés) planifiées sont nettoyées automatiquement. Les sauvegardes (ou instantanés) à la demande doivent être supprimées si la limite maximale est atteinte.

Supprimez les sauvegardes à la demande (instantanés du partage de fichiers Azure) à partir du portail de fichiers Azure.

>[!NOTE]
> Si vous supprimez les instantanés créés par Sauvegarde Azure, vous perdez les points de récupération.

### <a name="usererrorstorageaccountnotfound--operation-failed-as-the-specified-storage-account-does-not-exist-anymore"></a>UserErrorStorageAccountNotFound : L’opération a échoué, car le compte de stockage spécifié n’existe plus

Code d’erreur : UserErrorStorageAccountNotFound

Message d’erreur : L’opération a échoué, car le compte de stockage spécifié n’existe plus.

Assurez-vous que le compte de stockage existe toujours et qu’il n’a pas été supprimé.

### <a name="usererrordtsstorageaccountnotfound--the-storage-account-details-provided-are-incorrect"></a>UserErrorDTSStorageAccountNotFound : les détails du compte de stockage fournis sont incorrects

Code d’erreur : UserErrorDTSStorageAccountNotFound

Message d’erreur : Les détails du compte de stockage fournis sont incorrects.

Assurez-vous que le compte de stockage existe toujours et qu’il n’a pas été supprimé.

### <a name="usererrorresourcegroupnotfound--resource-group-doesnt-exist"></a>UserErrorResourceGroupNotFound : le groupe de ressources n’existe pas

Code d’erreur : UserErrorResourceGroupNotFound

Message d’erreur : Le groupe de ressources n’existe pas.

Sélectionnez ou créez un groupe de ressources.

### <a name="parallelsnapshotrequest--a-backup-job-is-already-in-progress-for-this-file-share"></a>ParallelSnapshotRequest : un travail de sauvegarde est déjà en cours pour ce partage de fichiers

Code d’erreur : ParallelSnapshotRequest

Message d’erreur : Un travail de sauvegarde est déjà en cours pour ce partage de fichiers.

- La sauvegarde du partage de fichiers ne prend pas en charge les demandes d’instantanés parallèles sur le même partage de fichiers.

- Attendez que la sauvegarde en cours se termine, puis réessayez. Si vous ne trouvez pas un travail de sauvegarde dans le coffre Recovery Services, vérifiez les autres coffres Recovery Services du même abonnement.

### <a name="filesharebackupfailedwithazurerprequestthrottling-filesharerestorefailedwithazurerprequestthrottling--file-share-backup-or-restore-failed-due-to-storage-service-throttling-this-may-be-because-the-storage-service-is-busy-processing-other-requests-for-the-given-storage-account"></a>FileshareBackupFailedWithAzureRpRequestThrottling/FileshareRestoreFailedWithAzureRpRequestThrottling : la sauvegarde ou la restauration du partage de fichiers ont échoué en raison d’une limitation de bande passante du service de stockage Cela peut être dû au fait que le service de stockage est occupé à traiter d’autres demandes pour le compte de stockage donné

Code d’erreur : FileshareBackupFailedWithAzureRpRequestThrottling/ FileshareRestoreFailedWithAzureRpRequestThrottling

Message d’erreur : La sauvegarde ou la restauration du partage de fichiers a échoué en raison d’une limitation de bande passante du service de stockage. Cela peut être dû au fait que le service de stockage est occupé à traiter d’autres demandes pour le compte de stockage donné.

Réessayez l’opération de sauvegarde/restauration ultérieurement.

### <a name="targetfilesharenotfound--target-file-share-not-found"></a>TargetFileShareNotFound : partage de fichiers cible introuvable

Code d’erreur : TargetFileShareNotFound

Message d’erreur : Partage de fichiers cible introuvable.

- Assurez-vous que le compte de stockage sélectionné existe et que le partage de fichiers cible n’a pas été supprimé.

- Assurez-vous que le compte de stockage est bien pris en charge pour la sauvegarde de partage de fichiers.

### <a name="usererrorstorageaccountislocked--backup-or-restore-jobs-failed-due-to-storage-account-being-in-locked-state"></a>UserErrorStorageAccountIsLocked : échec des travaux de sauvegarde ou de restauration en raison de l’état verrouillé du compte de stockage

Code d’erreur : UserErrorStorageAccountIsLocked

Message d’erreur : Échec des travaux de sauvegarde ou de restauration en raison de l’état verrouillé du compte de stockage.

Supprimez le verrou sur le compte de stockage ou utilisez **supprimer verrou** au lieu de **verrou de lecture**, puis réessayez l’opération de sauvegarde ou de restauration.

### <a name="datatransferservicecoflimitreached--recovery-failed-because-number-of-failed-files-are-more-than-the-threshold"></a>DataTransferServiceCoFLimitReached : échec de la récupération, car le nombre de fichiers ayant échoué dépasse le seuil

Code d’erreur : DataTransferServiceCoFLimitReached

Message d’erreur : Échec de la récupération, car le nombre de fichiers ayant échoué dépasse le seuil.

- Les raisons des échecs de la récupération sont répertoriées dans un fichier (chemin d’accès fourni dans les détails du travail). Traitez les échecs et recommencez l’opération de restauration uniquement pour les fichiers dont la récupération a échoué.

- Causes courantes des échecs de restauration de fichiers :

  - les fichiers dont la restauration a échoué sont en cours d’utilisation
  - un répertoire du même nom que le fichier dont la récupération a échoué existe dans le répertoire parent.

### <a name="datatransferserviceallfilesfailedtorecover--recovery-failed-as-no-file-could-be-recovered"></a>DataTransferServiceAllFilesFailedToRecover : échec de la récupération, car aucun fichier ne peut être récupéré

Code d’erreur : DataTransferServiceAllFilesFailedToRecover

Message d’erreur : Échec de la récupération, car aucun fichier ne peut être récupéré.

- Les raisons des échecs de la récupération sont répertoriées dans un fichier (chemin d’accès fourni dans les détails du travail). Traitez les échecs et recommencez l’opération de restauration uniquement pour les fichiers ayant échoué.

- Causes courantes des échecs de restauration de fichiers :

  - les fichiers dont la restauration a échoué sont en cours d’utilisation
  - un répertoire du même nom que le fichier dont la récupération a échoué existe dans le répertoire parent.

### <a name="usererrordtssourceurinotvalid---restore-fails-because-one-of-the-files-in-the-source-does-not-exist"></a>UserErrorDTSSourceUriNotValid : échec de la restauration, car l’un des fichiers de la source n’existe pas

Code d’erreur : DataTransferServiceSourceUriNotValid

Message d’erreur : Échec de la restauration, car l’un des fichiers de la source n’existe pas.

- Les éléments sélectionnés ne sont pas présents dans les données du point de récupération. Pour restaurer les fichiers, fournissez la liste de fichiers correcte.
- L’instantané du partage de fichiers qui correspond au point de récupération est supprimé manuellement. Sélectionnez un autre point de récupération et recommencez l’opération de restauration.

### <a name="usererrordtsdestlocked--a-recovery-job-is-in-process-to-the-same-destination"></a>UserErrorDTSDestLocked : un travail de récupération est en cours vers la même destination

Code d’erreur : UserErrorDTSDestLocked

Message d’erreur : Un travail de récupération est en cours vers la même destination.

- La sauvegarde de partage de fichiers ne prend pas en charge la récupération parallèle au même partage de fichiers cible.

- Attendez que la récupération en cours se termine, puis réessayez. Si vous ne trouvez pas de travail de récupération dans le coffre Recovery Services, vérifiez les autres coffres Recovery Services dans le même abonnement.

### <a name="usererrortargetfilesharefull--restore-operation-failed-as-target-file-share-is-full"></a>UserErrorTargetFileShareFull : échec de l’opération de restauration, car le partage de fichiers cible est plein

Code d’erreur : UserErrorTargetFileShareFull

Message d’erreur : Échec de l’opération de restauration, car le partage de fichiers cible est plein.

Augmentez le quota de taille du partage de fichiers cible pour prendre en charge les données de restauration, puis recommencez l’opération de restauration.

### <a name="usererrortargetfilesharequotanotsufficient--target-file-share-does-not-have-sufficient-storage-size-quota-for-restore"></a>UserErrorTargetFileShareQuotaNotSufficient : le partage de fichiers cible n’a pas un quota de taille de stockage suffisant pour la restauration

Code d’erreur : UserErrorTargetFileShareQuotaNotSufficient

Message d’erreur : Le partage de fichiers cible n’a pas un quota de taille de stockage suffisant pour la restauration.

Augmentez le quota de taille du partage de fichiers cible pour prendre en charge les données de restauration, puis recommencez l’opération.

### <a name="file-sync-prerestorefailed--restore-operation-failed-as-an-error-occurred-while-performing-pre-restore-operations-on-file-sync-service-resources-associated-with-the-target-file-share"></a>File Sync PreRestoreFailed : l’opération de restauration a échoué car une erreur s’est produite lors de l’exécution d’opérations de pré-restauration sur les ressources du service de synchronisation de fichiers associées au partage de fichiers cible

Code d’erreur : File Sync PreRestoreFailed

Message d’erreur : L’opération de restauration a échoué car une erreur s’est produite lors de l’exécution d’opérations de pré-restauration sur les ressources du service de synchronisation de fichiers associées au partage de fichiers cible.

Essayez de restaurer les données ultérieurement. Si le problème persiste, contactez le support technique Microsoft.

### <a name="azurefilesyncchangedetectioninprogress--azure-file-sync-service-change-detection-is-in-progress-for-the-target-file-share-the-change-detection-was-triggered-by-a-previous-restore-to-the-target-file-share"></a>AzureFileSyncChangeDetectionInProgress : la détection des modifications Azure File Sync est en cours pour le partage de fichiers cible La détection des modifications a été déclenchée par une restauration précédente sur le partage de fichiers cible

Code d’erreur : AzureFileSyncChangeDetectionInProgress

Message d’erreur : La détection des modifications Azure File Sync est en cours pour le partage de fichiers cible. La détection des modifications a été déclenchée par une restauration précédente sur le partage de fichiers cible.

Utilisez un autre partage de fichiers cible. Sinon, vous pouvez attendre la fin de la détection de changements du service Azure File Sync sur le partage de fichiers cible avant de réessayer la restauration.

### <a name="usererrorafsrecoverysomefilesnotrestored--one-or-more-files-could-not-be-recovered-successfully-for-more-information-check-the-failed-file-list-in-the-path-given-above"></a>UserErrorAFSRecoverySomeFilesNotRestored : un ou plusieurs fichiers n’ont pas pu être récupérés avec succès Pour plus d’informations, vérifiez la liste des fichiers ayant échoué dans le chemin d’accès indiqué ci-dessus.

Code d’erreur : UserErrorAFSRecoverySomeFilesNotRestored

Message d’erreur : Un ou plusieurs fichiers n’ont pas pu être récupérés avec succès. Pour plus d’informations, vérifiez la liste des fichiers ayant échoué dans le chemin d’accès indiqué ci-dessus.

- Les raisons des échecs de récupération sont répertoriées dans le fichier (chemin d’accès fourni dans les détails du travail). Traitez les raisons et recommencez l’opération de restauration uniquement pour les fichiers dont la récupération a échoué.
- Causes courantes des échecs de restauration de fichiers :

  - les fichiers dont la restauration a échoué sont en cours d’utilisation
  - un répertoire du même nom que le fichier dont la récupération a échoué existe dans le répertoire parent.

### <a name="usererrorafssourcesnapshotnotfound--azure-file-share-snapshot-corresponding-to-recovery-point-cannot-be-found"></a>UserErrorAFSSourceSnapshotNotFound : l’instantané de partage de fichiers Azure correspondant à ce point de récupération est introuvable

Code d’erreur : UserErrorAFSSourceSnapshotNotFound

Message d’erreur : L’instantané de partage de fichiers Azure correspondant à ce point de récupération est introuvable.

- Assurez-vous que l’instantané de partage de fichiers, correspondant au point de récupération que vous tentez d’utiliser pour la récupération existe toujours.

  >[!NOTE]
  >Si vous supprimez un instantané de partage de fichiers créé par Sauvegarde Azure, les points de récupération correspondants deviennent inutilisables. Nous vous recommandons de ne pas supprimer les instantanés pour garantir la récupération.

- Essayez de sélectionner un autre point de restauration pour récupérer vos données.

### <a name="usererroranotherrestoreinprogressonsametarget--another-restore-job-is-in-progress-on-the-same-target-file-share"></a>UserErrorAnotherRestoreInProgressOnSameTarget : un autre travail de restauration est en cours sur le même partage de fichiers cible

Code d’erreur : UserErrorAnotherRestoreInProgressOnSameTarget

Message d’erreur : Un autre travail de restauration est en cours sur le même partage de fichiers cible.

Utilisez un partage de fichiers cible différent. Vous pouvez également annuler l’autre restauration ou attendre qu’elle se termine.

## <a name="common-modify-policy-errors"></a>Erreurs courantes de stratégie de modification

### <a name="bmsusererrorconflictingprotectionoperation--another-configure-protection-operation-is-in-progress-for-this-item"></a>BMSUserErrorConflictingProtectionOperation : une autre opération de configuration de la protection est en cours pour cet élément

Code d’erreur : BMSUserErrorConflictingProtectionOperation

Message d’erreur : Une autre opération de configuration de la protection est en cours pour cet élément.

Attendez que l’opération précédente de stratégie de modification soit terminée, puis réessayer.

### <a name="bmsusererrorobjectlocked--another-operation-is-in-progress-on-the-selected-item"></a>BMSUserErrorObjectLocked : une autre opération est en cours sur l’élément sélectionné

Code d’erreur : BMSUserErrorObjectLocked

Message d’erreur : Une autre opération est en cours sur l’élément sélectionné.

Attendez que l’autre opération en cours soit terminée, puis réessayez.

## <a name="common-soft-delete-related-errors"></a>Erreurs courantes liées à la suppression réversible

### <a name="usererrorrestoreafsinsoftdeletestate--this-restore-point-is-not-available-as-the-snapshot-associated-with-this-point-is-in-a-file-share-that-is-in-soft-deleted-state"></a>UserErrorRestoreAFSInSoftDeleteState - Ce point de restauration n'est pas disponible, car l'instantané qui lui est associé se trouve dans un partage de fichiers qui est dans un état de suppression réversible.

Code d’erreur : UserErrorRestoreAFSInSoftDeleteState

Message d’erreur : Ce point de restauration n'est pas disponible, car l'instantané qui lui est associé se trouve dans un partage de fichiers qui est dans un état de suppression réversible.

Vous ne pouvez pas effectuer d'opération de restauration lorsque le partage de fichiers est dans un état de suppression réversible. Annulez la suppression du partage de fichiers à partir du portail Files ou à l'aide du [script Annuler la suppression](scripts/backup-powershell-script-undelete-file-share.md), puis essayez de le restaurer.

### <a name="usererrorrestoreafsindeletestate--listed-restore-points-are-not-available-as-the-associated-file-share-containing-the-restore-point-snapshots-has-been-deleted-permanently"></a>Les points de restauration listés ne sont pas disponibles, car le partage de fichiers associé contenant les instantanés de point de restauration a été supprimé définitivement.

Code d’erreur : UserErrorRestoreAFSInDeleteState

Message d’erreur : Les points de restauration listés ne sont pas disponibles, car le partage de fichiers associé contenant les instantanés de point de restauration a été supprimé définitivement.

Déterminez si le partage de fichiers sauvegardés a été supprimé. S'il était dans un état de suppression réversible, déterminez si la période de rétention de la suppression réversible est terminée et vérifiez que celle-ci n'a pas été récupérée. Dans les deux cas, vous perdrez définitivement tous vos instantanés et ne pourrez pas récupérer les données.

>[!NOTE]
> Nous vous recommandons de ne pas supprimer le partage de fichiers sauvegardé ou, s’il est dans un état de suppression réversible, d’annuler la suppression avant la fin de la période de conservation de la suppression réversible, pour éviter de perdre tous vos points de restauration.

### <a name="usererrorbackupafsinsoftdeletestate---backup-failed-as-the-azure-file-share-is-in-soft-deleted-state"></a>UserErrorBackupAFSInSoftDeleteState - La sauvegarde a échoué car le partage de fichiers Azure est dans un état de suppression réversible

Code d’erreur : UserErrorBackupAFSInSoftDeleteState

Message d’erreur : La sauvegarde a échoué car le partage de fichiers Azure est dans un état de suppression réversible

Annulez la suppression du partage de fichiers à partir du **portail Files** ou à l'aide du [script Annuler la suppression](scripts/backup-powershell-script-undelete-file-share.md) pour poursuivre la sauvegarde et empêcher la suppression définitive des données.

### <a name="usererrorbackupafsindeletestate--backup-failed-as-the-associated-azure-file-share-is-permanently-deleted"></a>UserErrorBackupAFSInDeleteState - La sauvegarde a échoué car le partage de fichiers Azure associé a été définitivement supprimé

Code d’erreur : UserErrorBackupAFSInDeleteState

Message d’erreur : La sauvegarde a échoué car le partage de fichiers Azure associé a été définitivement supprimé

Déterminez si le partage de fichiers sauvegardé a été définitivement supprimé. Si oui, arrêtez la sauvegarde du partage de fichiers pour éviter les échecs de sauvegarde répétés. Pour en savoir plus sur l'arrêt de la protection, consultez [Arrêter la protection du partage de fichiers Azure](./manage-afs-backup.md#stop-protection-on-a-file-share).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la sauvegarde des partages de fichiers Azure, consultez les références suivantes :

- [Sauvegarder des partages de fichiers Azure](backup-afs.md)
- [FAQ sur la sauvegarde des partages de fichiers Azure](backup-azure-files-faq.md)
