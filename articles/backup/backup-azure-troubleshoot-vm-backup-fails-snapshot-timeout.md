---
title: Détecter un problème lié aux agents et aux extensions
description: Symptômes, causes et résolution des défaillances de la Sauvegarde Azure liées à l’agent, à l’extension et aux disques.
ms.topic: troubleshooting
ms.date: 07/05/2019
ms.service: backup
ms.openlocfilehash: dd6f4d7884b120d2f8b5ea3f3ccb8d5385dd0880
ms.sourcegitcommit: 0d171fe7fc0893dcc5f6202e73038a91be58da03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93377103"
---
# <a name="troubleshoot-azure-backup-failure-issues-with-the-agent-or-extension"></a>Résoudre les problèmes d’une Sauvegarde Azure : Problèmes d’agent ou d’extension

Cet article indique les étapes à suivre pour résoudre les erreurs de la Sauvegarde Azure liées à la communication avec l’agent et l’extension de machine virtuelle.

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="step-by-step-guide-to-troubleshoot-backup-failures"></a>Guide pas à pas pour résoudre les échecs de sauvegarde

Les échecs de sauvegarde les plus courants peuvent se résoudre de manière autonome en suivant les étapes de résolution des problèmes énumérées ci-dessous :

### <a name="step-1-check-azure-vm-health"></a>Étape 1 : Vérifier l’intégrité de la machine virtuelle Azure

- **Assurez-vous que l’état d’approvisionnement de la machine virtuelle Azure est « en cours d’exécution »**  : Si l’[état d’approvisionnement de la machine virtuelle](../virtual-machines/states-lifecycle.md#provisioning-states) est **Arrêté/Libéré/En cours de mise à jour**, cela interfère avec l’opération de sauvegarde. Ouvrez *Portail Azure > Machine virtuelle > Vue d’ensemble >* et vérifiez l’état de la machine virtuelle pour vous assurer qu’elle est **en cours d’exécution** et faites une nouvelle tentative de sauvegarde.
- **Passez en revue les mises à jour ou redémarrages du système d’exploitation en attente** : Assurez-vous qu’il n’existe aucune mise à jour du système d’exploitation ni redémarrages en attente sur la machine virtuelle.

### <a name="step-2-check-azure-vm-guest-agent-service-health"></a>Étape 2 : Vérifier l’intégrité du service de l’agent invité de la machine virtuelle Azure

- **Assurez-vous que le service de l’agent invité de la machine virtuelle Azure est démarré et à jour** :
  - Sur une machine virtuelle Windows :
    - Accédez à **services.msc** et vérifiez que le **service de l’agent invité de la machine virtuelle Windows Azure** est opérationnel. Assurez-vous également que la [version la plus récente](https://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409) est installée. Pour plus d’informations, consultez [Problèmes relatifs à l’agent invité de la machine virtuelle Windows](backup-azure-troubleshoot-vm-backup-fails-snapshot-timeout.md#the-agent-installed-in-the-vm-but-unresponsive-for-windows-vms).
    - L’agent de machine virtuelle Azure est installé par défaut sur toutes les machines virtuelles Windows déployées à partir d’une image Place de marché Azure à partir du portail, de PowerShell, de l’interface de ligne de commande ou d’un modèle Resource Manager. Une [installation manuelle de l’agent](../virtual-machines/extensions/agent-windows.md#manual-installation) peut s’avérer nécessaire lorsque vous créez une image de machine virtuelle personnalisée qui est déployée sur Azure.
    - Passez en revue la matrice de prise en charge pour vérifier si la machine virtuelle s’exécute sur le [système d’exploitation Windows pris en charge](backup-support-matrix-iaas.md#operating-system-support-windows).
  - Sur une machine virtuelle Linux :
    - Assurez-vous que le service de l’agent invité de machine virtuelle Azure fonctionne en exécutant la commande `ps-e`. Assurez-vous également que la [version la plus récente](../virtual-machines/extensions/update-linux-agent.md) est installée. Pour plus d’informations, consultez [Problèmes relatifs à l’agent invité de la machine virtuelle Linux](backup-azure-troubleshoot-vm-backup-fails-snapshot-timeout.md#the-agent-installed-in-the-vm-is-out-of-date-for-linux-vms).
    - Vérifiez que les [dépendances de l’agent de machine virtuelle Linux sur les packages système](../virtual-machines/extensions/agent-linux.md#requirements) ont la configuration prise en charge. Par exemple : La version de Python prise en charge est la version 2.6 et ultérieure.
    - Passez en revue la matrice de prise en charge pour vérifier si la machine virtuelle s’exécute sur le [système d’exploitation Linux pris en charge](backup-support-matrix-iaas.md#operating-system-support-linux).

### <a name="step-3-check-azure-vm-extension-health"></a>Étape 3 : Vérifier l’intégrité de l’extension de machine virtuelle Azure

- **Assurez-vous que toutes les extensions de machine virtuelle Azure sont à l’état « Approvisionnement réussi »**  : Si une extension est à l’état d’échec, cela peut interférer avec la sauvegarde.
- *Ouvrez Portail Azure > Machine virtuelle > Paramètres > Extensions > État de l’extension* et vérifiez que toutes les extensions sont à l’état **Approvisionnement réussi**.
- Assurez-vous que tous les [problèmes d’extension](../virtual-machines/extensions/overview.md#troubleshoot-extensions) sont résolus et relancez l’opération de sauvegarde.
- **Assurez-vous que l’application système COM+** est opérationnelle. En outre, le **service Distributed Transaction Coordinator** doit s’exécuter en tant que **compte de service réseau**. Suivez les étapes de cet article pour [résoudre les problèmes liés à COM+ et MSDTC](backup-azure-vms-troubleshoot.md#extensionsnapshotfailedcom--extensioninstallationfailedcom--extensioninstallationfailedmdtc---extension-installationoperation-failed-due-to-a-com-error).

### <a name="step-4-check-azure-backup-vm-extension-health"></a>Étape 4 : Vérifier l’intégrité de l’extension de machine virtuelle Sauvegarde Azure

Sauvegarde Azure se sert de l’extension de capture instantanée de machine virtuelle pour récupérer une sauvegarde cohérente des applications de la machine virtuelle Azure. Sauvegarde Azure installe l’extension lors de la première sauvegarde planifiée qui est déclenchée après l’activation du service de sauvegarde.

- **Assurez-vous que l’extension de capture instantanée de machine virtuelle n’est pas à l’état d’échec** : Suivez les étapes indiquées dans cette [section](backup-azure-troubleshoot-vm-backup-fails-snapshot-timeout.md#usererrorvmprovisioningstatefailed---the-vm-is-in-failed-provisioning-state) pour vérifier et vous assurer que l’extension de Sauvegarde Azure est saine.

- **Vérifiez si l’antivirus bloque l’extension** : Certains logiciels antivirus peuvent empêcher l’exécution d’extensions.
  
  Au moment de l’échec de la sauvegarde, vérifiez s’il existe des entrées de journal dans **_Journaux des applications de l’observateur d’événements_* _ avec le _*_nom de l’application défaillante : IaaSBcdrExtension.exe_*_. Si vous voyez des entrées, il se peut que l’antivirus configuré dans la machine virtuelle restreigne l’exécution de l’extension de sauvegarde. Effectuez un test en excluant les répertoires suivants dans la configuration de l’antivirus et relancez l’opération de sauvegarde.
  - `C:\Packages\Plugins\Microsoft.Azure.RecoveryServices.VMSnapshot`
  - `C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.RecoveryServices.VMSnapshot`

- _*Vérifiez si l’accès au réseau est requis** : Les paquets d’extensions sont téléchargés à partir du référentiel d’extensions de Stockage Azure, et les chargements d’état d’extension sont publiés sur Stockage Azure. [Plus d’informations](../virtual-machines/extensions/features-windows.md#network-access)
  - Si vous utilisez une version non prise en charge de l’agent, vous devez autoriser l’accès sortant vers le Stockage Azure dans cette région à partir de la machine virtuelle.
  - Si vous avez bloqué l’accès à l’adresse `168.63.129.16` à l’aide du pare-feu invité ou avec un proxy, les extensions échouent, que vous utilisiez ou non une version prise en charge. Les ports 80, 443 et 32526 sont nécessaires. [En savoir plus](../virtual-machines/extensions/features-windows.md#network-access).

- **Vérifiez que le protocole DHCP est activé à l’intérieur de la machine virtuelle invitée** : Il est nécessaire pour obtenir l’adresse de l’hôte ou de l’infrastructure à partir de DHCP pour que la sauvegarde de machine virtuelle IaaS fonctionne. Si vous avez besoin d’une adresse IP privée statique, vous devez la configurer via le portail Azure ou PowerShell et vérifier que l’option DHCP de la machine virtuelle est activée. [En savoir plus](backup-azure-troubleshoot-vm-backup-fails-snapshot-timeout.md#the-snapshot-status-cannot-be-retrieved-or-a-snapshot-cannot-be-taken).

- **Assurez-vous que le service d’enregistreur VSS est opérationnel** : Suivez ces étapes pour [résoudre les problèmes liés à l’enregistreur VSS](backup-azure-vms-troubleshoot.md#extensionfailedvsswriterinbadstate---snapshot-operation-failed-because-vss-writers-were-in-a-bad-state).
- **Suivez les recommandations en matière de meilleures pratiques de sauvegarde** : Passez en revue les [meilleures pratiques pour permettre la sauvegarde de machines virtuelles Azure](backup-azure-vms-introduction.md#best-practices).
- **Passez en revue les instructions pour les disques chiffrés** : Si vous activez la sauvegarde pour les machines virtuelles avec disque chiffré, assurez-vous d’avoir fourni toutes les autorisations requises. Pour en savoir plus, consultez [Sauvegarder et restaurer une machine virtuelle Azure chiffrée](backup-azure-vms-encryption.md).

## <a name="usererrorguestagentstatusunavailable---vm-agent-unable-to-communicate-with-azure-backup"></a><a name="UserErrorGuestAgentStatusUnavailable-vm-agent-unable-to-communicate-with-azure-backup"></a>UserErrorGuestAgentStatusUnavailable : L’agent de machine virtuelle ne peut pas communiquer avec le service Sauvegarde Azure

**Code d’erreur** : UserErrorGuestAgentStatusUnavailable <br>
**Message d’erreur** : L’agent de machine virtuelle ne peut pas communiquer avec la sauvegarde Microsoft Azure<br>

L’agent de machine virtuelle Azure peut être arrêté, obsolète, dans un état incohérent ou non installé. Ces états empêchent le service Sauvegarde Azure de déclencher des instantanés.

- **Ouvrez le volet Portail Azure > Machine virtuelle > Paramètres > Propriétés** et assurez-vous que l’**État** de la machine virtuelle est **En cours d’exécution** et que l’**État de l’agent** est **Prêt**. Si l’agent de machine virtuelle est arrêté ou est dans un état incohérent, redémarrez l’agent<br>
  - Pour les machines virtuelles Windows, suivez ces [étapes](#the-agent-installed-in-the-vm-but-unresponsive-for-windows-vms) pour redémarrer l’agent invité.<br>
  - Pour les machines virtuelles Linux, suivez ces [étapes](#the-agent-installed-in-the-vm-is-out-of-date-for-linux-vms) pour redémarrer l’agent invité.
- **Ouvrez Portail Azure > Machine virtuelle > Paramètres > Extensions** et vérifiez que toutes les extensions sont à l’état **Approvisionnement réussi**. Si ce n’est pas le cas, effectuez [ces étapes](#usererrorvmprovisioningstatefailed---the-vm-is-in-failed-provisioning-state) pour résoudre le problème.

## <a name="guestagentsnapshottaskstatuserror---could-not-communicate-with-the-vm-agent-for-snapshot-status"></a>GuestAgentSnapshotTaskStatusError : Impossible de communiquer avec l’agent de machine virtuelle pour obtenir l’état de l’instantané

**Code d’erreur** : GuestAgentSnapshotTaskStatusError<br>
**Message d’erreur** : Impossible de communiquer avec l’agent en charge de la machine virtuelle pour avoir un aperçu de l’état instantané <br>

Après avoir enregistré et planifié une machine virtuelle pour le service Sauvegarde Azure, ce dernier démarre le travail en communiquant avec l’extension de sauvegarde de la machine virtuelle pour prendre un instantané à un moment donné. Il est possible que l’une des conditions suivantes empêche le déclenchement de l’instantané. Si la capture instantanée n’est pas déclenchée, un échec de sauvegarde risque de se produire. Suivez les étapes de dépannage ci-dessous dans l’ordre indiqué, puis réessayez l’opération :  

**Cause 1 : [L’agent est installé dans la machine virtuelle, mais ne répond pas (machines virtuelles Windows)](#the-agent-installed-in-the-vm-but-unresponsive-for-windows-vms)**  

**Cause 2 : [L’agent installé dans la machine virtuelle est obsolète (machines virtuelles Linux)](#the-agent-installed-in-the-vm-is-out-of-date-for-linux-vms)**

**Cause 3 : [Impossible de récupérer l’état de l’instantané ou de capturer un instantané](#the-snapshot-status-cannot-be-retrieved-or-a-snapshot-cannot-be-taken)**

**Cause 4 : [Les options de configuration de l’agent de machine virtuelle ne sont pas définies (pour les machines virtuelles Linux)](#vm-agent-configuration-options-are-not-set-for-linux-vms)**

**Cause 5 : [La solution de contrôle d’application bloque IaaSBcdrExtension.exe](#application-control-solution-is-blocking-iaasbcdrextensionexe)**

## <a name="usererrorvmprovisioningstatefailed---the-vm-is-in-failed-provisioning-state"></a>UserErrorVmProvisioningStateFailed : Échec de l’approvisionnement de la machine virtuelle

**Code d’erreur** : UserErrorVmProvisioningStateFailed<br>
**Message d’erreur** : Échec de l’approvisionnement de la machine virtuelle<br>

Cette erreur se produit lorsqu’un des échecs d’extension met la machine virtuelle en état d’échec d’approvisionnement.<br>**Ouvrez Portail Azure > Machine virtuelle > Paramètres > Extensions > État de l’extension** et vérifiez que toutes les extensions sont à l’état **Approvisionnement réussi**. Pour plus d’informations, consultez [États de provisionnement](../virtual-machines/states-lifecycle.md#provisioning-states).

- Si une extension est à l’état d’échec, cela peut interférer avec la sauvegarde. Assurez-vous que ces problèmes d’extension sont résolus et recommencez l’opération de sauvegarde.
- Si l’état de provisionnement de machine virtuelle est En cours de mise à jour, il peut interférer avec la sauvegarde. Vérifiez qu’il est sain et réessayez l’opération de sauvegarde.

## <a name="usererrorrpcollectionlimitreached---the-restore-point-collection-max-limit-has-reached"></a>UserErrorRpCollectionLimitReached : Limite maximale de la collection de points de restauration atteinte

**Code d’erreur** : UserErrorRpCollectionLimitReached <br>
**Message d’erreur** : La limite maximale de la collection de points de restauration est atteinte. <br>

- Ce problème peut se produire si un verrou dans le groupe de ressources des points de récupération empêche le nettoyage automatique des points de récupération.
- Ce problème peut également se produire si plusieurs sauvegardes sont déclenchées tous les jours. Actuellement, nous recommandons une seule sauvegarde par jour, car les points de restauration instantanés sont conservés pendant 1 à 5 jours conformément à la rétention d’instantanés configurée, et seuls 18 points de restauration instantanés peuvent être associés à une machine virtuelle à un moment donné. <br>
- Le nombre de points de restauration entre les collections de points de restauration et les groupes de ressources d’une machine virtuelle ne peut pas dépasser 18. Pour créer un point de restauration, supprimez les points de restauration existants.

Action recommandée :<br>
Pour résoudre ce problème, supprimez le verrou du groupe de ressources de la machine virtuelle et recommencez l’opération pour déclencher le nettoyage.
> [!NOTE]
> Le service de sauvegarde crée un groupe de ressources distinct du groupe de ressources de la machine virtuelle afin de stocker la collection de points de restauration. Il est conseillé de ne pas verrouiller le groupe de ressources créé pour une utilisation par le service Sauvegarde. Le format d’affectation des noms du groupe de ressources créé par le service de sauvegarde est : AzureBackupRG_`<Geo>`_`<number>`. Par exemple : *AzureBackupRG_northeurope_1*

**Étape 1 : [Supprimer le verrou du groupe de ressources des points de restauration](#remove_lock_from_the_recovery_point_resource_group)** <br>
**Étape 2 : [ Nettoyer la collection de points de restauration](#clean_up_restore_point_collection)**<br>

## <a name="usererrorkeyvaultpermissionsnotconfigured---backup-doesnt-have-sufficient-permissions-to-the-key-vault-for-backup-of-encrypted-vms"></a>UserErrorKeyvaultPermissionsNotConfigured : Sauvegarde Azure ne dispose pas des autorisations d’accès suffisantes au coffre de clés pour la sauvegarde de machines virtuelles chiffrées

**Code d’erreur** : UserErrorKeyvaultPermissionsNotConfigured <br>
**Message d’erreur** : Sauvegarde Azure ne dispose pas des autorisations d’accès suffisantes au coffre de clés pour la sauvegarde de machines virtuelles chiffrées. <br>

Pour qu’une opération de sauvegarde puisse réussir sur des machines virtuelles chiffrées, elle doit avoir les autorisations d’accès au coffre de clés. Les autorisations peuvent être définies par l’intermédiaire du [Portail Azure](./backup-azure-vms-encryption.md) ou de [PowerShell](./backup-azure-vms-automation.md#enable-protection).

## <a name="extensionsnapshotfailednonetwork---snapshot-operation-failed-due-to-no-network-connectivity-on-the-virtual-machine"></a><a name="ExtensionSnapshotFailedNoNetwork-snapshot-operation-failed-due-to-no-network-connectivity-on-the-virtual-machine"></a>ExtensionSnapshotFailedNoNetwork : L’opération de capture instantanée a échoué, car la machine virtuelle ne présente aucune connectivité réseau

**Code d’erreur** : ExtensionSnapshotFailedNoNetwork<br>
**Message d’erreur** : L’opération de capture instantanée a échoué, car la machine virtuelle ne présente aucune connectivité réseau<br>

Après avoir enregistré et planifié une machine virtuelle pour le service Sauvegarde Azure, ce dernier démarre le travail en communiquant avec l’extension de sauvegarde de la machine virtuelle pour prendre un instantané à un moment donné. Il est possible que l’une des conditions suivantes empêche le déclenchement de l’instantané. Si la capture instantanée n’est pas déclenchée, un échec de sauvegarde risque de se produire. Suivez les étapes de résolution des problèmes ci-dessous, puis réessayez l’opération :

**[Impossible de récupérer l’état de l’instantané ou de capturer un instantané](#the-snapshot-status-cannot-be-retrieved-or-a-snapshot-cannot-be-taken)**  

## <a name="extensionoperationfailedformanageddisks---vmsnapshot-extension-operation-failed"></a><a name="ExtensionOperationFailed-vmsnapshot-extension-operation-failed"></a>ExtensionOperationFailedForManagedDisks : l’opération d’extension VMSnapshot a échoué

**Code d’erreur** : ExtensionOperationFailedForManagedDisks <br>
**Message d’erreur** : Échec de l’opération d’extension VMSnapshot<br>

Après avoir enregistré et planifié une machine virtuelle pour le service Sauvegarde Azure, ce dernier démarre le travail en communiquant avec l’extension de sauvegarde de la machine virtuelle pour prendre un instantané à un moment donné. Il est possible que l’une des conditions suivantes empêche le déclenchement de l’instantané. Si la capture instantanée n’est pas déclenchée, un échec de sauvegarde risque de se produire. Suivez les étapes de dépannage ci-dessous dans l’ordre indiqué, puis réessayez l’opération :  
**Cause 1 : [Impossible de récupérer l’état de l’instantané ou de capturer un instantané](#the-snapshot-status-cannot-be-retrieved-or-a-snapshot-cannot-be-taken)**  
**Cause 2 : [L’agent est installé dans la machine virtuelle, mais ne répond pas (machines virtuelles Windows)](#the-agent-installed-in-the-vm-but-unresponsive-for-windows-vms)**  
**Cause 3 : [L’agent installé dans la machine virtuelle est obsolète (machines virtuelles Linux)](#the-agent-installed-in-the-vm-is-out-of-date-for-linux-vms)**

## <a name="backupoperationfailed--backupoperationfailedv2---backup-fails-with-an-internal-error"></a>BackUpOperationFailed / BackUpOperationFailedV2 : La sauvegarde échoue, avec une erreur interne

**Code d’erreur** : BackUpOperationFailed / BackUpOperationFailedV2 <br>
**Message d’erreur** : Échec de l’opération de sauvegarde avec une erreur interne : veuillez retenter l’opération dans quelques minutes <br>

Après avoir enregistré et planifié une machine virtuelle pour le service Azure Backup , ce dernier lance le travail en communiquant avec l’extension de sauvegarde de la machine virtuelle pour prendre un instantané à un moment donné. Il est possible que l’une des conditions suivantes empêche le déclenchement de l’instantané. Si la capture instantanée n’est pas déclenchée, un échec de sauvegarde risque de se produire. Suivez les étapes de dépannage ci-dessous dans l’ordre indiqué, puis réessayez l’opération :  
**Cause 1 : [L’agent est installé dans la machine virtuelle, mais ne répond pas (machines virtuelles Windows)](#the-agent-installed-in-the-vm-but-unresponsive-for-windows-vms)**  
**Cause 2 : [L’agent installé dans la machine virtuelle est obsolète (machines virtuelles Linux)](#the-agent-installed-in-the-vm-is-out-of-date-for-linux-vms)**  
**Cause 3 : [Impossible de récupérer l’état de l’instantané ou de capturer un instantané](#the-snapshot-status-cannot-be-retrieved-or-a-snapshot-cannot-be-taken)**  
**Cause 4 : [Le service de sauvegarde n’est pas autorisé à supprimer les anciens points de restauration en raison du verrouillage d’un groupe de ressources](#remove_lock_from_the_recovery_point_resource_group)**<br>

## <a name="usererrorunsupporteddisksize---the-configured-disk-sizes-is-currently-not-supported-by-azure-backup"></a>UserErrorUnsupportedDiskSize : la taille de disque configurée n’est pas prise en charge par Sauvegarde Azure

**Code d’erreur** : UserErrorUnsupportedDiskSize <br>
**Message d’erreur** : La taille de disque configurée n’est pas prise en charge par Sauvegarde Azure. <br>

Votre opération de sauvegarde peut échouer lors de la sauvegarde d’une machine virtuelle dont la taille de disque est supérieure à 32 To. De plus, la sauvegarde de disques chiffrés d’une taille supérieure à 4 To n’est pas prise en charge actuellement. Vérifiez que la taille de disque est inférieure ou égale à la limite prise en charge en fractionnant les disques.

## <a name="usererrorbackupoperationinprogress---unable-to-initiate-backup-as-another-backup-operation-is-currently-in-progress"></a>UserErrorBackupOperationInProgress - Impossible de lancer la sauvegarde, car une autre opération de sauvegarde est en cours d’exécution

**Code d’erreur** : UserErrorBackupOperationInProgress <br>
**Message d’erreur** : Impossible de lancer la sauvegarde, car une autre opération de sauvegarde est en cours d’exécution<br>

L’un de vos derniers travaux de sauvegarde a échoué, car il y a déjà un travail de sauvegarde en cours. Vous ne pouvez pas en lancer de nouvelle tant que la tâche en cours n’est pas terminée. Vérifiez que l’opération de sauvegarde en cours est terminée avant d’en déclencher ou d’en programmer d’autres. Pour vérifier l’état des travaux de sauvegarde, procédez comme suit :

1. Connectez-vous au portail Azure, puis sélectionnez **Tous les services**. Saisissez Recovery Services, puis sélectionnez **Coffres Recovery Services**. La liste des archivages de Recovery Services s’affiche.
2. Dans la liste des coffres Recovery Services, sélectionnez un coffre dont la sauvegarde est configurée.
3. Dans le menu du tableau de bord du coffre, sélectionnez **Travaux de sauvegarde** pour afficher tous les travaux de sauvegarde.
   - Si une tâche de sauvegarde est en cours, attendez qu’elle se termine ou annulez-la.
     - Pour annuler le travail de sauvegarde, cliquez dessus avec le bouton droit, puis sélectionnez **Annuler** ou utilisez [PowerShell](/powershell/module/az.recoveryservices/stop-azrecoveryservicesbackupjob).
   - Si vous avez reconfiguré la sauvegarde dans un autre coffre, vérifiez qu’aucun travail de sauvegarde n’est en cours d’exécution dans l’ancien coffre. S’il y en a une, annulez-la.
     - Pour annuler le travail de sauvegarde, cliquez dessus avec le bouton droit, puis sélectionnez **Annuler** ou utilisez [PowerShell](/powershell/module/az.recoveryservices/stop-azrecoveryservicesbackupjob).
4. Réessayez l’opération de sauvegarde.

Si l’opération de sauvegarde planifiée empiète sur la configuration de la sauvegarde suivante, consultez [Bonnes pratiques](backup-azure-vms-introduction.md#best-practices), [Performances de sauvegarde](backup-azure-vms-introduction.md#backup-performance) et [Considérations sur la restauration](backup-azure-vms-introduction.md#backup-and-restore-considerations).

## <a name="usererrorcrpreportedusererror---backup-failed-due-to-an-error-for-details-see-job-error-message-details"></a>UserErrorCrpReportedUserError : échec de la sauvegarde en raison d’une erreur. Pour plus d’informations, consultez les détails du message d’erreur.

**Code d’erreur** : UserErrorCrpReportedUserError <br>
**Message d’erreur** : Échec de la sauvegarde en raison d’une erreur. Pour plus d’informations, consultez les détails du message d’erreur.

Cette erreur est signalée par la machine virtuelle IaaS. Pour identifier la cause racine du problème, accédez aux paramètres du coffre Recovery Services. Dans la section **Analyse**, sélectionnez **Travaux de sauvegarde** pour filtrer et afficher l’état. Sélectionnez **Échecs** pour consulter les détails du message d’erreur sous-jacent. Effectuez d’autres actions conformément aux recommandations figurant sur la page des détails de l’erreur.

## <a name="usererrorbcmdatasourcenotpresent---backup-failed-this-virtual-machine-is-not-actively-protected-by-azure-backup"></a>UserErrorBcmDatasourceNotPresent - Échec de la sauvegarde : cette machine virtuelle n'est pas (activement) protégée par Sauvegarde Azure

**Code d’erreur** : UserErrorBcmDatasourceNotPresent <br>
**Message d’erreur** : Échec de la sauvegarde : cette machine virtuelle n'est pas (activement) protégée par Sauvegarde Azure.

Vérifiez si la machine virtuelle est activement protégée par Sauvegarde Azure (autrement dit, qu’elle n’est pas en pause). Pour résoudre ce problème, assurez-vous que la machine virtuelle est active, puis retentez l'opération.

## <a name="causes-and-solutions"></a>Causes et solutions

### <a name="the-agent-is-installed-in-the-vm-but-its-unresponsive-for-windows-vms"></a><a name="the-agent-installed-in-the-vm-but-unresponsive-for-windows-vms"></a>L’agent est installé dans la machine virtuelle, mais ne répond pas (machines virtuelles Windows)

#### <a name="solution-for-this-error"></a>Solution pour cette erreur

Il se peut que l’agent de machine virtuelle soit endommagé ou que le service ait été arrêté. Réinstallez l’agent de machine virtuelle pour obtenir la dernière version. Cela permet également de redémarrer la communication avec le service.

1. Regardez si le service d’agent invité Microsoft Azure s’exécute dans les services de machine virtuelle (services.msc). Essayez de le redémarrer et de lancer la sauvegarde.
2. Si le service d’agent invité Windows Azure n’apparaît pas dans les services, accédez à **Programmes et fonctionnalités** dans le Panneau de configuration pour déterminer s’il est installé.
3. Si le service d’agent invité Wondows Azure figure sous **Programmes et fonctionnalités**, désinstallez-le.
4. Téléchargez et installez la [dernière version du MSI de l’agent](https://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409). Des droits d’administrateur sont nécessaires pour effectuer l’installation.
5. Vérifiez que le service d’agent invité Microsoft Azure apparaît dans les services.
6. Exécutez une sauvegarde à la demande :
   - Dans le portail, sélectionnez **Sauvegarder maintenant**.

Vérifiez également que [Microsoft .NET 4.5 est installé](/dotnet/framework/migration-guide/how-to-determine-which-versions-are-installed) dans la machine virtuelle. Si ce n’est pas le cas, l’agent de machine virtuelle ne pourra pas communiquer avec le service.

### <a name="the-agent-installed-in-the-vm-is-out-of-date-for-linux-vms"></a>L’agent installé dans la machine virtuelle est obsolète (pour les machines virtuelles Linux)

#### <a name="solution"></a>Solution

La plupart des échecs des machines virtuelles Linux liés aux agents ou aux extensions sont causés par des problèmes qui affectent un agent obsolète de machine virtuelle. Pour résoudre ce problème, suivez ces recommandations générales :

1. Suivez les instructions fournies pour la [mise à jour d’un agent de machine virtuelle Linux](../virtual-machines/extensions/update-linux-agent.md).

   > [!NOTE]
   > Nous *recommandons vivement* la mise à jour de l’agent uniquement par le biais de référentiel de distribution. Nous ne recommandons pas de télécharger le code de l’agent à partir de GitHub directement et d’effectuer la mise à jour. Si la dernière version de l’agent n’est pas disponible pour la distribution, contactez le support de distribution pour savoir comment l’installer. Pour rechercher l’agent le plus récent, accédez à la page relative à l’[agent Windows Azure Linux](https://github.com/Azure/WALinuxAgent/releases) du référentiel GitHub.

2. Vérifiez que l’agent Azure s’exécute sur la machine virtuelle à l’aide de la commande suivante : `ps -e`.

   Si le processus ne s’exécute pas, redémarrez-le à l’aide des commandes suivantes :

   - Pour Ubuntu : `service walinuxagent start`
   - Pour les autres distributions : `service waagent start`

3. [Configurez l’agent de redémarrage automatique](https://github.com/Azure/WALinuxAgent/wiki/Known-Issues#mitigate_agent_crash).
4. Exécutez une nouvelle sauvegarde de test. Si la défaillance persiste, collectez les journaux d’activité suivants à partir de la machine virtuelle :

   - /var/lib/waagent/*.xml
   - /var/log/waagent.log
   - /var/log/azure/*

Si vous exigez une journalisation détaillée pour waagent, procédez comme suit :

1. Dans le fichier /etc/waagent.conf, recherchez la ligne suivante : **Activer l’enregistrement des informations détaillées (y|n)**
2. Passez la valeur **Logs.Verbose** de *N* à *O*.
3. Enregistrez la modification, puis redémarrez waagent en suivant les étapes détaillées plus haut dans cette section.

### <a name="vm-agent-configuration-options-are-not-set-for-linux-vms"></a>Les options de configuration de l’agent de machine virtuelle ne sont pas définies (pour les machines virtuelles Linux)

Un fichier de configuration (/etc/waagent.conf) contrôle les actions de waagent. Les options du fichier de configuration **Extensions.Enable** doivent être définies sur **y** et **Provisioning.Agent** doivent être définies sur **auto** pour que Sauvegarde Microsoft Azure fonctionne.
Pour obtenir la liste complète des options du fichier config de l’agent de machine virtuelle, consultez <https://github.com/Azure/WALinuxAgent#configuration-file-options>.

### <a name="application-control-solution-is-blocking-iaasbcdrextensionexe"></a>La solution de contrôle d’application bloque IaaSBcdrExtension.exe

Si vous exécutez [AppLocker](/windows/security/threat-protection/windows-defender-application-control/applocker/what-is-applocker) (ou une autre solution de contrôle d’application) et que les règles sont basées sur un serveur de publication ou sur un chemin d’accès, elles peuvent bloquer l’exécution de l’exécutable **IaaSBcdrExtension.exe**.

#### <a name="solution-to-this-issue"></a>Solution à ce problème

Excluez le chemin d’accès `/var/lib` ou l’exécutable **IaaSBcdrExtension.exe** d’AppLocker (ou d’un autre logiciel de contrôle d’application).

### <a name="the-snapshot-status-cant-be-retrieved-or-a-snapshot-cant-be-taken"></a><a name="the-snapshot-status-cannot-be-retrieved-or-a-snapshot-cannot-be-taken"></a>Impossible de récupérer l’état de l’instantané ou de capturer un instantané

La sauvegarde de machine virtuelle émet une commande de capture instantanée à destination du stockage sous-jacent. Elle peut échouer pour deux raisons : soit elle n’a pas accès au compte de stockage, soit l’exécution de la tâche de capture instantanée est différée.

#### <a name="solution-for-this-issue"></a>Solution à ce problème

Voici les causes possibles de l’échec de la tâche de capture instantanée :

| Cause | Solution |
| --- | --- |
| L’état de la machine virtuelle est rapporté de manière incorrecte, car la machine virtuelle est arrêtée dans le protocole RDP (Remote Desktop Protocol). | Si vous avez arrêté la machine virtuelle dans le protocole RDP, retournez sur le portail pour vérifier que son état est correct. Si ce n’est pas le cas, arrêtez la machine virtuelle dans le portail à l’aide de l’option **Arrêter** dans le tableau de bord de la machine virtuelle. |
| La machine virtuelle ne parvient pas à récupérer l’adresse d’hôte/de structure à partir du protocole DHCP. | Le protocole DHCP doit être activé dans l’invité pour que la sauvegarde de la machine virtuelle IaaS fonctionne. Si la machine virtuelle ne parvient pas à récupérer l’adresse d’hôte/de structure à partir de la réponse 245 DHCP, elle ne peut ni télécharger, ni exécuter des extensions. Si vous avez besoin d’une adresse IP privée statique, vous devez la configurer via le **portail Azure** ou **PowerShell** et vérifier que l’option DHCP dans la machine virtuelle est activée. [Apprenez-en davantage](../virtual-network/virtual-networks-static-private-ip-arm-ps.md#change-the-allocation-method-for-a-private-ip-address-assigned-to-a-network-interface) sur la configuration d’une adresse IP statique avec PowerShell.

### <a name="remove-lock-from-the-recovery-point-resource-group"></a><a name="remove_lock_from_the_recovery_point_resource_group"></a>Supprimer le verrou du groupe de ressources des points de récupération

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Sélectionnez l’option **Toutes les ressources**, puis le groupe de ressources de la collection de points de restauration au format suivant : AzureBackupRG_`<Geo>`_`<number>`.
3. Dans la section **Paramètres**, sélectionnez **Verrous** pour afficher les verrous.
4. Pour supprimer le verrou, cliquez sur le bouton de sélection, puis sélectionnez **Supprimer**.

    ![Supprimer un verrou](./media/backup-azure-arm-vms-prepare/delete-lock.png)

### <a name="clean-up-restore-point-collection"></a><a name="clean_up_restore_point_collection"></a> Nettoyer la collection de points de restauration

Après avoir supprimé le verrou, vous devez nettoyer les points de restauration.

Si vous supprimez le groupe de ressources de la machine virtuelle, voire la machine virtuelle elle-même, les instantanés de restauration des disques managés restent actifs et expirent en fonction de la conservation définie. Pour supprimer les instantanés de restauration (dont vous n’avez plus besoin) stockés dans la collection de points de restauration, nettoyez cette dernière en suivant la procédure indiquée ci-dessous.

Pour nettoyer les points de restauration, suivez l’une des méthodes suivantes :<br>

- [Nettoyer la collection de points de restauration en exécutant une sauvegarde sur demande](#clean-up-restore-point-collection-by-running-on-demand-backup)<br>
- [Nettoyer la collection de points de restauration sur le portail Azure](#clean-up-restore-point-collection-from-azure-portal)<br>

#### <a name="clean-up-restore-point-collection-by-running-on-demand-backup"></a><a name="clean-up-restore-point-collection-by-running-on-demand-backup"></a>Nettoyer la collection de points de restauration en exécutant une sauvegarde sur demande

Après avoir supprimé le verrou, déclenchez une sauvegarde sur demande. Cette action garantit le nettoyage automatique des points de restauration. Attendez-vous à l’échec de cette opération à la demande la première fois. Toutefois, elle assure le nettoyage automatique à la place de la suppression manuelle des points de restauration. Après le nettoyage, votre sauvegarde planifiée suivante devrait réussir.

> [!NOTE]
> Le nettoyage automatique a lieu quelques heures après le déclenchement de la sauvegarde sur demande. Si votre sauvegarde planifiée échoue à nouveau, essayez de supprimer manuellement la collection de points de restauration à l’aide de la procédure indiquée [ici](#clean-up-restore-point-collection-from-azure-portal).

#### <a name="clean-up-restore-point-collection-from-azure-portal-br"></a><a name="clean-up-restore-point-collection-from-azure-portal"></a>Nettoyer la collection de points de restauration sur le portail Azure <br>

Pour effacer manuellement la collection des points de restauration, qui n’est pas effacée en raison du verrou du groupe de ressources, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Dans le menu **Hub**, sélectionnez **Toutes les ressources**, puis le groupe de ressources au format AzureBackupRG_`<Geo>`_`<number>` dans lequel se trouve votre machine virtuelle.

    ![Sélectionner le groupe de ressources](./media/backup-azure-arm-vms-prepare/resource-group.png)

3. Sélectionnez le groupe de ressources ; le volet **Vue d’ensemble** s’affiche.
4. Sélectionnez l’option **Afficher les types masqués** pour afficher toutes les ressources masquées. Sélectionnez les collections de points de restauration au format suivant : AzureBackupRG_`<VMName>`_`<number>`.

    ![Sélectionner la collection de points de restauration](./media/backup-azure-arm-vms-prepare/restore-point-collection.png)

5. Sélectionnez **Supprimer** pour nettoyer la collection de points de restauration.
6. Renouvelez l’opération de sauvegarde.

> [!NOTE]
 >Si la ressource (RP Collection) compte un grand nombre de points de restauration, leur suppression du portail peut entraîner un dépassement de temps et un échec. Il s’agit d’un problème CRP connu, où tous les points de restauration ne sont pas supprimés dans le temps imparti, ce qui entraîne l’expiration du délai d’attente de l’opération. Toutefois, l’opération de suppression est généralement effectuée après deux ou trois nouvelles tentatives.
