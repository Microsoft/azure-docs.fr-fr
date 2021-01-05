---
title: Détecter un problème d’erreurs de sauvegarde avec les machines virtuelles Azure
description: Dans cet article, découvrez comment résoudre les erreurs rencontrées lors de la sauvegarde et de la restauration de machines virtuelles Azure.
ms.reviewer: srinathv
ms.topic: troubleshooting
ms.date: 08/30/2019
ms.openlocfilehash: cb25d9263648fbd92bc075751c1a8e627d03bd44
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96325211"
---
# <a name="troubleshooting-backup-failures-on-azure-virtual-machines"></a>Résolution des échecs de sauvegarde sur les machines virtuelles Azure

Vous pouvez résoudre les erreurs rencontrées pendant l’utilisation de Sauvegarde Azure à l’aide des informations ci-dessous :

## <a name="backup"></a>Sauvegarde

Cette section traite de l’échec d’opération de sauvegarde d’une machine virtuelle Azure.

### <a name="basic-troubleshooting"></a>Dépannage de base

* Assurez-vous que l’agent de machine virtuelle (WA Agent) est la [version la plus récente](./backup-azure-arm-vms-prepare.md#install-the-vm-agent).
* Vérifiez que la version du système d’exploitation de la machine virtuelle Windows ou Linux est prise en charge, consultez la [matrice de prise en charge de sauvegarde de machine virtuelle IaaS](./backup-support-matrix-iaas.md).
* Vérifiez qu’aucun autre service de sauvegarde n’est en cours d’exécution.
  * Pour vous assurer qu’il n’existe aucun problème d’extension de capture instantanée, [désinstallez les extensions pour forcer le rechargement, puis réessayez la sauvegarde](./backup-azure-troubleshoot-vm-backup-fails-snapshot-timeout.md).
* Vérifiez la connectivité Internet de la machine virtuelle.
  * Vérifiez qu’aucun autre service de sauvegarde n’est en cours d’exécution.
* À partir de `Services.msc`, assurez-vous que le service d’**agent invité Windows Azure** est en **cours d’exécution**. Si le service d’**agent invité Windows Azure** est manquant, installez-le à partir de la [sauvegarde de machines virtuelles Azure dans un coffre Recovery Services](./backup-azure-arm-vms-prepare.md#install-the-vm-agent).
* Le **journal des événements** peut présenter des échecs de sauvegarde provenant d’autres produits de sauvegarde, par exemple la sauvegarde de Windows Server, qui ne sont pas dus à Sauvegarde Azure. Pour déterminer si le problème est lié à la sauvegarde Azure, procédez comme suit :
  * En cas d’erreur avec l’entrée **Sauvegarde** dans la source ou le message de l’événement, vérifiez si les sauvegardes de la machine virtuelle IaaS Azure ont réussi et si un point de restauration a été créé avec le type d’instantané souhaité.
  * Si la sauvegarde Azure fonctionne, le problème est probablement lié à une autre solution de sauvegarde.
  * Voici un exemple d’erreur 517 de l’observateur d’événements dans laquelle Sauvegarde Azure fonctionnait correctement, mais la « sauvegarde de Windows Server » a échoué : ![Échec de la Sauvegarde Windows Server](media/backup-azure-vms-troubleshoot/windows-server-backup-failing.png)
  * En cas d’échec de la Sauvegarde Azure, recherchez le code d’erreur correspondant dans la section Erreurs de sauvegarde de machine virtuelle courantes dans cet article.

## <a name="common-issues"></a>Problèmes courants

Voici les problèmes courants liés aux échecs de sauvegarde sur les machines virtuelles Azure.

### <a name="vmrestorepointinternalerror---antivirus-configured-in-the-vm-is-restricting-the-execution-of-backup-extension"></a>VMRestorePointInternalError, l’antivirus configuré dans la machine virtuelle, restreint l’exécution de l’extension de sauvegarde

Code d’erreur : VMRestorePointInternalError

Si, au moment de la sauvegarde, les **Journaux d’applications de l’observateur d’événements** affichent le message **Nom de l’application défaillante : IaaSBcdrExtension.exe**, il est confirmé que l’antivirus configuré dans la machine virtuelle restreint l’exécution de l’extension de sauvegarde.
Pour résoudre ce problème, excluez les répertoires ci-dessous dans la configuration de l’antivirus et réessayez l’opération de sauvegarde.

* `C:\Packages\Plugins\Microsoft.Azure.RecoveryServices.VMSnapshot`
* `C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.RecoveryServices.VMSnapshot`

### <a name="copyingvhdsfrombackupvaulttakinglongtime---copying-backed-up-data-from-vault-timed-out"></a>CopyingVHDsFromBackUpVaultTakingLongTime – La copie des données sauvegardées du coffre a expiré.

Code d’erreur : CopyingVHDsFromBackUpVaultTakingLongTime <br/>
Message d’erreur : La copie des données sauvegardées du coffre a expiré.

Cela peut se produire en raison d’erreurs de stockage temporaires ou d’un nombre insuffisant d’IOPS du compte de stockage pour que le service de sauvegarde transfère les données vers le coffre dans le délai imparti. Configurez la sauvegarde de machine virtuelle en appliquant ces [meilleures pratiques](backup-azure-vms-introduction.md#best-practices), puis relancez l’opération de sauvegarde.

### <a name="usererrorvmnotindesirablestate---vm-is-not-in-a-state-that-allows-backups"></a>UserErrorVmNotInDesirableState – La machine virtuelle n’est pas dans un état qui autorise les sauvegardes

Code d’erreur : UserErrorVmNotInDesirableState <br/>
Message d’erreur : La machine virtuelle n’est pas dans un état permettant les sauvegardes.<br/>

L’opération de sauvegarde a échoué parce que la machine virtuelle est en état d’échec. Pour une sauvegarde réussie, l’état de la machine virtuelle doit être En cours d’exécution, Arrêté ou Arrêté (désalloué).

* Si la machine virtuelle se trouve dans un état temporaire entre **En cours d’exécution** et **Arrêt**, attendez que l’état change. Déclenchez ensuite le travail de sauvegarde.
* Si la machine virtuelle est de type Linux et utilise le module de noyau Linux à sécurité avancée, excluez le chemin d’accès de l’agent Azure Linux **/var/lib/waagent** de la stratégie de sécurité, et assurez-vous que l’extension Sauvegarde Azure est installée.

### <a name="usererrorfsfreezefailed---failed-to-freeze-one-or-more-mount-points-of-the-vm-to-take-a-file-system-consistent-snapshot"></a>UserErrorFsFreezeFailed – Échec du gel d’un ou plusieurs points de montage de la machine virtuelle pour prendre une capture instantanée cohérente au niveau du système de fichiers.

Code d’erreur : UserErrorFsFreezeFailed <br/>
Message d’erreur : Impossible de figer un ou plusieurs points de montage de la machine virtuelle pour prendre une capture instantanée cohérente au niveau du système de fichiers.

* Démontez les appareils dont l’état du système de fichiers n’a pas été nettoyé à l’aide de la commande **unmount**.
* Effectuez une vérification de cohérence de système de fichiers sur ces appareils à l’aide de la commande **fsck**.
* Remontez les appareils, puis retentez l’opération de sauvegarde.</ol>

### <a name="extensionsnapshotfailedcom--extensioninstallationfailedcom--extensioninstallationfailedmdtc---extension-installationoperation-failed-due-to-a-com-error"></a>ExtensionSnapshotFailedCOM / ExtensionInstallationFailedCOM / ExtensionInstallationFailedMDTC – Échec de l’installation/opération d’extension en raison d’une erreur COM+

Code d’erreur : ExtensionSnapshotFailedCOM <br/>
Message d’erreur : Échec de l’opération de capture instantanée en raison d’une erreur COM+

Code d’erreur : ExtensionInstallationFailedCOM  <br/>
Message d’erreur : Échec de l’installation/opération d’extension en raison d’une erreur COM+

Code d’erreur : ExtensionInstallationFailedMDTC <br/>
Message d’erreur : L’installation de l’extension a échoué en renvoyant l’erreur « COM+ n’a pas pu communiquer avec le Microsoft Distributed Transaction Coordinator <br/>

L’opération de sauvegarde a échoué en raison d’un problème avec l’**application système COM+** du service Windows.  Pour résoudre ce problème, effectuez les étapes suivantes :

* Essayez de démarrer ou redémarrer l’**application système COM+** du service Windows  (dans une invite de commandes avec élévation de privilèges **- net start COMSysApp**).
* Vérifiez que le service **Distributed Transaction Coordinator** s’exécute en tant que compte de **Service réseau**. Sinon, modifiez l’**application système COM+** pour qu’elle s’exécute en tant que compte de **Service réseau**, puis redémarrez-la.
* Si vous ne parvenez pas à redémarrer le service, réinstallez le service **Distributed Transaction Coordinator** en effectuant les étapes suivantes :
  * Arrêtez le service MSDTC
  * Ouvrez une invite de commandes (cmd)
  * Exécutez la commande `msdtc -uninstall`
  * Exécutez la commande `msdtc -install`
  * Lancez le service MSDTC
* Démarrez le service Windows **Application système COM+** . Une fois que **Application système COM+** démarre, déclenchez un travail de sauvegarde à partir du Portail Azure.</ol>

### <a name="extensionfailedvsswriterinbadstate---snapshot-operation-failed-because-vss-writers-were-in-a-bad-state"></a>ExtensionFailedVssWriterInBadState – L’opération de capture instantanée a échoué, car les enregistreurs VSS étaient dans un état incorrect

Code d’erreur : ExtensionFailedVssWriterInBadState <br/>
Message d’erreur : L’opération de capture instantanée a échoué parce que les enregistreurs VSS étaient dans un état incorrect.

Cette erreur se produit parce que les enregistreurs VSS sont dans un état incorrect. Les extensions Sauvegarde Azure interagissent avec les enregistreurs VSS pour prendre des instantanés des disques. Pour résoudre ce problème, effectuez les étapes suivantes :

Étape 1 : Redémarrez les enregistreurs VSS qui se trouvent dans un état incorrect.

* À partir d’une invite de commandes avec élévation de privilèges, exécutez ```vssadmin list writers```.
* La sortie contient tous les enregistreurs VSS et leur état. Pour chaque enregistreur VSS dont l’état n’est pas **[1] Stable**, redémarrez le service de l’enregistreur VSS correspondant.
* Pour redémarrer le service, exécutez les commandes suivantes à partir d’une invite de commandes avec élévation de privilèges :

 ```net stop serviceName``` <br>
 ```net start serviceName```

> [!NOTE]
> Le redémarrage de certains services peut avoir un impact sur votre environnement de production. Assurez-vous que le processus d’approbation est respecté et que le service est redémarré à l’heure d’arrêt prévue.

Étape 2 : Si le redémarrage des enregistreurs VSS n’a pas résolu le problème, exécutez la commande suivante à partir d’une invite de commandes avec élévation de privilèges (en tant qu’administrateur) pour empêcher la création de threads pour les instantanés-blobs.

```console
REG ADD "HKLM\SOFTWARE\Microsoft\BcdrAgentPersistentKeys" /v SnapshotWithoutThreads /t REG_SZ /d True /f
```

Étape 3 : Si les étapes 1 et 2 n’ont pas résolu le problème, l’échec peut être dû à un dépassement du délai d’expiration des enregistreurs VSS en raison d’un IOPS limité.<br>

Pour le vérifier, accédez à ***Journaux du système et des applications de l'observateur d'événements** _ et recherchez le message d'erreur suivant :<br>
_Le fournisseur de clichés instantanés a dépassé le délai d'attente maximal lors de la mise en attente des écritures vers le volume cliché. Cela est probablement dû à une activité excessive sur le volume par une application ou un service système. Réessayez quand l'activité sur le volume sera moindre.*<br>

Solution :

* Vérifiez les possibilités de distribution de la charge sur les disques de machine virtuelle. Cela permet de réduire la charge sur les disques individuels. Vous pouvez [vérifier la limitation d’IOPS en activant les métriques de diagnostic au niveau du stockage](../virtual-machines/troubleshooting/performance-diagnostics.md#install-and-run-performance-diagnostics-on-your-vm).
* Modifiez la stratégie de sauvegarde pour effectuer des sauvegardes pendant les heures creuses, lorsque la charge sur la machine virtuelle est à son niveau le plus bas.
* Mettez à niveau les disques Azure pour prendre en charge des IOPS supérieures. [En savoir plus ici](../virtual-machines/disks-types.md)

### <a name="extensionfailedvssserviceinbadstate---snapshot-operation-failed-due-to-vss-volume-shadow-copy-service-in-bad-state"></a>ExtensionFailedVssServiceInBadState : échec de l’opération d’instantané en raison de l’état incorrect du service VSS (cliché instantané de volume)

Code d’erreur : ExtensionFailedVssServiceInBadState <br/>
Message d’erreur : Échec de l’opération d’instantané en raison de l’état incorrect du service VSS (cliché instantané de volume).

Cette erreur se produit parce que le service VSS est dans un état incorrect. Les extensions Sauvegarde Azure interagissent avec le service VSS pour prendre des instantanés des disques. Pour résoudre ce problème, effectuez les étapes suivantes :

Redémarrez le service VSS (cliché instantané de volume).

* Accédez à Services.msc et redémarrez « Service de cliché instantané du volume ».<br>
(ou)<br>
* Exécutez les commandes suivantes à partir d'une invite de commandes avec élévation de privilèges :

 ```net stop VSS``` <br>
 ```net start VSS```

Si le problème persiste, redémarrez la machine virtuelle lors des temps d’arrêt planifiés.

### <a name="usererrorskunotavailable---vm-creation-failed-as-vm-size-selected-is-not-available"></a>UserErrorSkuNotAvailable  : échec de création de la machine virtuelle, car la taille de machine virtuelle sélectionnée n’est pas disponible

Code d’erreur : UserErrorSkuNotAvailable Message d’erreur : Échec de création de la machine virtuelle car la taille de machine virtuelle sélectionnée n’est pas disponible.

Cette erreur se produit parce que la taille de machine virtuelle sélectionnée pendant l’opération de restauration n’est pas prise en charge. <br>

Pour résoudre ce problème, utilisez l’option [Restaurer les disques](./backup-azure-arm-restore-vms.md#restore-disks) au cours de l’opération de restauration. Utilisez ces disques pour créer une machine virtuelle à partir de la liste des [tailles de machines virtuelles prises en charge disponibles](./backup-support-matrix-iaas.md#vm-compute-support) à l'aide de [cmdlets PowerShell](./backup-azure-vms-automation.md#create-a-vm-from-restored-disks).

### <a name="usererrormarketplacevmnotsupported---vm-creation-failed-due-to-market-place-purchase-request-being-not-present"></a>UserErrorMarketPlaceVMNotSupported  : échec de création de la machine virtuelle en raison de l’absence d’une demande d’achat de Place de marché

Code d’erreur : UserErrorMarketPlaceVMNotSupported Message d’erreur : Échec de création de la machine virtuelle en raison de l’absence d’une demande d’achat de Place de marché.

Sauvegarde Azure prend en charge la sauvegarde et la restauration des machines virtuelles qui sont disponibles sur Place de marché Azure. Cette erreur se produit lorsque vous essayez de restaurer une machine virtuelle (avec un paramètre Plan/Éditeur spécifique) qui n’est plus disponible sur Place de marché Azure. [En savoir plus ici](/legal/marketplace/participation-policy#offering-suspension-and-removal).

* Pour résoudre ce problème, utilisez l’option [Restaurer les disques](./backup-azure-arm-restore-vms.md#restore-disks) au cours de l’opération de restauration, puis utilisez des cmdlets [PowerShell](./backup-azure-vms-automation.md#create-a-vm-from-restored-disks) ou [Azure CLI](./tutorial-restore-disk.md) pour créer la machine virtuelle avec les informations les plus récentes sur le marketplace qui correspondent à la machine virtuelle.
* Si l’éditeur ne dispose d’aucune information sur le marketplace, vous pouvez utiliser les disques de données pour récupérer vos données et les attacher à une machine virtuelle existante.

### <a name="extensionconfigparsingfailure--failure-in-parsing-the-config-for-the-backup-extension"></a>ExtensionConfigParsingFailure – Échec d’analyse de la configuration pour l’extension de sauvegarde

Code d’erreur : ExtensionConfigParsingFailure<br/>
Message d’erreur : Échec d’analyse de la configuration pour l’extension de sauvegarde.

Cette erreur se produit en raison de modifications des autorisations sur le répertoire **MachineKeys** : **%systemdrive%\programdata\microsoft\crypto\rsa\machinekeys**.
Exécutez la commande suivante et vérifiez que les autorisations sur le répertoire **MachineKeys** sont celles par défaut : `icacls %systemdrive%\programdata\microsoft\crypto\rsa\machinekeys`.

Les autorisations par défaut sont comme suit :

* Tout le monde : (R,W)
* BUILTIN\Administrateurs : (F)

Si les autorisations que vous voyez dans le répertoire **MachineKeys** sont différentes de celles par défaut, suivez les étapes ci-dessous pour les corriger, supprimer le certificat et déclencher la sauvegarde :

1. Corrigez les autorisations sur le répertoire **MachineKeys**. À l’aide des propriétés de sécurité de l’explorateur et des paramètres de sécurité avancés du répertoire, réinitialisez les autorisations aux valeurs par défaut. Supprimez tous les objets utilisateur (sauf ceux par défaut) du répertoire, et assurez-vous que l’autorisation **Tout le monde** dispose d’un accès spécial comme suit :

   * Lister le dossier/lire les données
   * Lire les attributs
   * Lire les attributs étendus
   * Créer les fichiers/écrire les données
   * Créer les dossiers/ajouter les données
   * Écrire les attributs
   * Écrire les attributs étendus
   * Autorisations de lecture
2. Supprimez tous les certificats dont le modèle de déploiement classique est de type **Délivré à** ou **générateur de certificats Windows Azure CRP** :

   * [Ouvrez les certificats sur une console d’ordinateur local](/dotnet/framework/wcf/feature-details/how-to-view-certificates-with-the-mmc-snap-in).
   * Sous **Certificats** > **personnels**, supprimez tous les certificats dont le modèle de déploiement classique est de type **Délivré à** ou **générateur de certificats Windows Azure CRP**.
3. Lancez une opération de sauvegarde de machine virtuelle.

### <a name="extensionstuckindeletionstate---extension-state-is-not-supportive-to-backup-operation"></a>ExtensionStuckInDeletionState – L’état de l’extension ne prend pas en charge l’opération de sauvegarde.

Code d’erreur : ExtensionStuckInDeletionState <br/>
Message d’erreur : L’état de l’extension ne prend pas en charge l’opération de sauvegarde.

L’opération de sauvegarde a échoué en raison d’un état incohérent de l’extension de sauvegarde. Pour résoudre ce problème, effectuez les étapes suivantes :

* Vérifiez que l’agent invité est installé et réactif
* Dans le portail Azure, accédez à **Machine virtuelle** > **Tous les paramètres** > **Extensions**
* Sélectionnez l’extension de sauvegarde VmSnapshot ou VmSnapshotLinux, puis sélectionnez **Désinstaller**.
* Après avoir supprimé l’extension de sauvegarde, recommencez l’opération de sauvegarde
* L’opération de sauvegarde suivante installera la nouvelle extension à l’état souhaité

### <a name="extensionfailedsnapshotlimitreachederror---snapshot-operation-failed-as-snapshot-limit-is-exceeded-for-some-of-the-disks-attached"></a>ExtensionFailedSnapshotLimitReachedError – L’opération de capture instantanée a échoué, car certains disques attachés ont dépassé la limite de captures instantanées

Code d’erreur : ExtensionFailedSnapshotLimitReachedError  <br/>
Message d’erreur : L’opération de capture instantanée a échoué, car certains disques attachés ont dépassé la limite de captures instantanées

L’opération de capture instantanée a échoué parce que la limite de captures instantanées a été dépassée pour certains des disques attachés. Suivez les étapes de dépannage ci-dessous, puis réessayez l’opération.

* Supprimez les instantanés d’objets blob de disque qui ne sont pas nécessaires. Veillez à ne pas supprimer les objets blob de disque. Seuls les instantanés d’objets blob doivent être supprimés.
* Si la suppression réversible est activée sur les comptes de stockage sur disque de machine virtuelle, configurez la conservation de suppression réversible afin que le nombre d’instantanés existants soit toujours inférieur au nombre maximal autorisé.
* Si Azure Site Recovery est activé sur la machine virtuelle sauvegardée, effectuez les étapes suivantes :

  * Assurez-vous que la valeur **isanysnapshotfailed** est définie sur false dans /etc/azure/vmbackup.conf.
  * Planifiez l’exécution d’Azure Site Recovery à un autre moment, afin qu’elle ne soit pas en conflit avec l’opération de sauvegarde.

### <a name="extensionfailedtimeoutvmnetworkunresponsive---snapshot-operation-failed-due-to-inadequate-vm-resources"></a>ExtensionFailedTimeoutVMNetworkUnresponsive – Échec de l’opération de capture instantanée en raison de ressources de machine virtuelle inadéquates

Code d’erreur : ExtensionFailedTimeoutVMNetworkUnresponsive<br/>
Message d’erreur : Échec de l’opération de capture instantanée en raison de ressources de machine virtuelle inadéquates.

L’opération de sauvegarde sur la machine virtuelle a échoué en raison d’un retard des appels réseau lors de l’exécution de l’opération de capture instantanée. Pour résoudre ce problème, effectuez l’étape 1. Si le problème persiste, essayez les étapes 2 et 3.

**Étape 1** : Créer une capture instantanée via l’hôte

À partir d’une invite de commandes avec élévation de privilèges (administrateur), exécutez la commande suivante :

```console
REG ADD "HKLM\SOFTWARE\Microsoft\BcdrAgentPersistentKeys" /v SnapshotMethod /t REG_SZ /d firstHostThenGuest /f
REG ADD "HKLM\SOFTWARE\Microsoft\BcdrAgentPersistentKeys" /v CalculateSnapshotTimeFromHost /t REG_SZ /d True /f
```

Cela garantira que les captures instantanées soient effectuées via l’hôte plutôt que via l’invité. Relancez l’opération de sauvegarde.

**Étape 2** : Essayer de modifier la planification de la sauvegarde en la définissant sur une heure à laquelle la machine virtuelle est moins chargée (moins de charge pour le processeur ou moins d'IOPS)

**Étape 3** : Essayer d’[augmenter la taille de machine virtuelle](../virtual-machines/windows/resize-vm.md), puis réessayez l’opération

### <a name="320001-resourcenotfound---could-not-perform-the-operation-as-vm-no-longer-exists--400094-bcmv2vmnotfound---the-virtual-machine-doesnt-exist--an-azure-virtual-machine-wasnt-found"></a>320001, ResourceNotFound – Impossible d’effectuer l’opération, car la machine virtuelle n’existe plus / 400094, BCMV2VMNotFound – La machine virtuelle n’existe pas / Machine virtuelle Azure introuvable

Code d’erreur : 320001, ResourceNotFound <br/> Message d’erreur : Impossible d’effectuer l’opération, car la machine virtuelle n’existe plus. <br/> <br/> Code d’erreur : 400094, BCMV2VMNotFound <br/> Message d’erreur : La machine virtuelle n’existe pas <br/>
Machine virtuelle Azure introuvable.

Cette erreur se produit lorsque la machine virtuelle principale est supprimée. Cependant, la stratégie de sauvegarde continue de rechercher une machine virtuelle à sauvegarder. Pour corriger cette erreur, suivez les étapes ci-dessous :

* Recréez la machine virtuelle avec le même nom et le même nom de groupe de ressources **nom du service cloud**,<br>or
* Arrêtez la protection de la machine virtuelle en supprimant ou non les données de sauvegarde. Pour plus d’informations, consultez [Arrêt de la protection des machines virtuelles](backup-azure-manage-vms.md#stop-protecting-a-vm).</li></ol>

### <a name="usererrorbcmpremiumstoragequotaerror---could-not-copy-the-snapshot-of-the-virtual-machine-due-to-insufficient-free-space-in-the-storage-account"></a>UserErrorBCMPremiumStorageQuotaError – Impossible de copier l’instantané de la machine virtuelle, car l’espace libre est insuffisant dans le compte de stockage

Code d’erreur : UserErrorBCMPremiumStorageQuotaError<br/> Message d’erreur : Impossible de copier l’instantané de la machine virtuelle, car l’espace libre est insuffisant dans le compte de stockage

 Pour les machines virtuelles Premium sur une pile de sauvegarde de machines virtuelles V1, nous copions la capture instantanée sur le compte de stockage. Cette étape permet de s’assurer que le trafic de gestion de sauvegarde, qui fonctionne sur la capture instantanée, ne limite pas le nombre d’IOPS accessibles à l’application à l’aide de disques Premium. <br><br>Nous vous conseillons d’allouer seulement 50 pour cent (soit 17,5 To) de l’espace du compte de stockage total. Ainsi, le service Sauvegarde Azure peut copier la capture instantanée sur le compte de stockage et transférer des données depuis cet emplacement copié vers le compte de stockage dans le coffre.

### <a name="380008-azurevmoffline---failed-to-install-microsoft-recovery-services-extension-as-virtual-machine--is-not-running"></a>380008, AzureVmOffline – L’installation de l’extension Microsoft Recovery Services a échoué, car la machine virtuelle n’est pas en cours d’exécution

Code d’erreur : 380008, AzureVmOffline <br/> Message d’erreur : L’installation de l’extension Microsoft Recovery Services a échoué, car la machine virtuelle n’est pas en cours d’exécution.

L’agent de machine virtuelle est une condition requise pour l’extension Recovery Services. Installez l’agent de machine virtuelle Azure, puis recommencez l’opération d’inscription. <br> <ol> <li>Vérifiez si l’agent de machine virtuelle a été installé correctement. <li>Vérifiez que l’indicateur de la configuration de la machine virtuelle est défini correctement.</ol> Apprenez-en plus sur l'installation de l'agent de machine virtuelle et sur la validation de cette opération.

### <a name="extensionsnapshotbitlockererror---the-snapshot-operation-failed-with-the-volume-shadow-copy-service-vss-operation-error"></a>ExtensionSnapshotBitlockerError – L’opération de capture instantanée a échoué en renvoyant l’erreur d’opération du service de cliché instantané de volume (VSS)

Code d’erreur : ExtensionSnapshotBitlockerError <br/> Message d’erreur : L’opération de capture instantanée a échoué en renvoyant l’erreur d’opération du service Cliché instantané de volume (VSS) **Ce lecteur est verrouillé par le chiffrement de lecteur BitLocker. Vous devez déverrouiller ce lecteur à partir du panneau de configuration.**

Désactivez BitLocker pour tous les lecteurs sur la machine virtuelle et vérifiez si le problème VSS est résolu.

### <a name="vmnotindesirablestate---the-vm-isnt-in-a-state-that-allows-backups"></a>VmNotInDesirableState – La machine virtuelle n’est pas dans un état qui autorise les sauvegardes

Code d’erreur : VmNotInDesirableState <br/> Message d’erreur :  La machine virtuelle n’est pas dans un état permettant les sauvegardes.

* Si la machine virtuelle se trouve dans un état temporaire entre **En cours d’exécution** et **Arrêt**, attendez que l’état change. Déclenchez ensuite le travail de sauvegarde.
* Si la machine virtuelle est de type Linux et utilise le module de noyau Linux à sécurité avancée, excluez le chemin d’accès de l’agent Azure Linux **/var/lib/waagent** de la stratégie de sécurité, et assurez-vous que l’extension Sauvegarde Azure est installée.

* L’agent de machine virtuelle n’est pas présent sur la machine virtuelle : <br>installez les composants requis et l’agent de machine virtuelle. Ensuite, relancez l’opération. |Apprenez-en plus sur [l’installation de l’agent de machine virtuelle et la validation de cette opération](#vm-agent).

### <a name="extensionsnapshotfailednosecurenetwork---the-snapshot-operation-failed-because-of-failure-to-create-a-secure-network-communication-channel"></a>ExtensionSnapshotFailedNoSecureNetwork – Échec de l’opération de capture instantanée en raison de l’échec de la création d’un canal de communication réseau sécurisé

Code d’erreur : ExtensionSnapshotFailedNoSecureNetwork <br/> Message d’erreur : Échec de l’opération de capture instantanée en raison de l’échec de la création du canal de communication réseau sécurisé.

* Ouvrez l’Éditeur du Registre en exécutant **regedit.exe** avec élévation de privilèges.
* Identifiez toutes les versions de. NET Framework présentes dans votre système. Elles se trouvent dans la hiérarchie de la clé de Registre **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft**.
* Pour chaque .NET Framework présent dans la clé de Registre, ajoutez la clé suivante : <br> **SchUseStrongCrypto"=dword:00000001**. </ol>

### <a name="extensionvcredistinstallationfailure---the-snapshot-operation-failed-because-of-failure-to-install-visual-c-redistributable-for-visual-studio-2012"></a>ExtensionVCRedistInstallationFailure – Échec de l’opération de capture instantanée en raison de l’échec de l’installation de Redistributable Visual C++ pour Visual Studio 2012

Code d’erreur : ExtensionVCRedistInstallationFailure <br/> Message d’erreur : Échec de l’opération de capture instantanée en raison de l’échec de l’installation de Redistribuable Visual C++ pour Visual Studio 2012.

* Accédez à `C:\Packages\Plugins\Microsoft.Azure.RecoveryServices.VMSnapshot\agentVersion` et installez vcredist2013_x64.<br/>Assurez-vous que la valeur de clé de Registre qui permet l’installation du service est correctement définie. Autrement dit, définissez la valeur **Démarrer** dans **HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Msiserver** sur **3** et non sur **4**. <br><br>Si vous rencontrez toujours des problèmes d’installation, redémarrez le service d’installation en exécutant **MSIEXEC /UNREGISTER** suivi de **MSIEXEC /REGISTER** dans une invite de commandes avec élévation de privilèges.
* Consultez le journal des événements pour détecter la présence de problèmes liés à l’accès. Par exemple : *Produit : Microsoft Visual C++ 2013 x64 Minimum Runtime - 12.0.21005 -- Erreur 1401. Impossible de créer la clé : Software\Classes.  Erreur système 5.  Vérifiez que vous disposez des droits suffisants pour cette clé ou contactez votre service de support technique.* <br><br> Vérifiez que le compte d’administrateur ou d’utilisateur dispose d’autorisations suffisantes pour mettre à jour la clé de Registre **HKEY_LOCAL_MACHINE\SOFTWARE\Classes**. Octroyez des autorisations suffisantes et redémarrez l’agent invité Windows Azure.<br><br> <li> Si des antivirus sont en place, vérifiez que leurs règles d’exclusion autorisent l’installation.

### <a name="usererrorrequestdisallowedbypolicy---an-invalid-policy-is-configured-on-the-vm-which-is-preventing-snapshot-operation"></a>UserErrorRequestDisallowedByPolicy – Une stratégie non valide est configurée sur la machine virtuelle qui empêche l’opération de capture instantanée

Code d’erreur :  UserErrorRequestDisallowedByPolicy <BR> Message d’erreur : Une stratégie non valide est configurée sur la machine virtuelle qui empêche l’opération de capture instantanée.

Si vous disposez d’une stratégie Azure Policy qui [régit les étiquettes au sein de votre environnement](../governance/policy/tutorials/govern-tags.md), vous pouvez soit envisager de passer d’un [effet Deny](../governance/policy/concepts/effects.md#deny) à un [effet Modify](../governance/policy/concepts/effects.md#modify) pour la stratégie, soit créer manuellement le groupe de ressources en fonction du [schéma de nommage exigé par Sauvegarde Azure](./backup-during-vm-creation.md#azure-backup-resource-group-for-virtual-machines).

## <a name="jobs"></a>travaux

| Détails de l’erreur | Solution de contournement |
| --- | --- |
| L’annulation n’est pas prise en charge pour ce type de travail : <br>Attendez que le travail se termine. |None |
| Le travail n’est pas dans un état annulable : <br>Attendez que le travail se termine. <br>**or**<br> Le travail sélectionné n’est pas dans un état annulable : <br>Attendez que le travail se termine. |Il est probable que le travail soit presque terminé. Attendez que le travail se termine.|
| Sauvegarde Azure ne peut pas annuler le travail, car il n’est pas en cours d’exécution : <br>L’annulation est uniquement prise en charge pour les travaux en cours. Essayez d’annuler un travail en cours d’exécution. |Cette erreur se produit en raison d’un état temporaire. Attendez une minute et relancez l’opération d’annulation. |
| Sauvegarde Azure n’a pas annulé le travail : <br>Attendez que le travail se termine. |None |

## <a name="restore"></a>Restaurer

### <a name="disks-appear-offline-after-file-restore"></a>Les disques apparaissent hors connexion après la restauration des fichiers

Si, après la restauration, vous remarquez que les disques sont hors connexion, alors :

* Vérifiez si l’ordinateur sur lequel le script est exécuté répond à la configuration requise du système d’exploitation. [En savoir plus](./backup-azure-restore-files-from-vm.md#step-3-os-requirements-to-successfully-run-the-script)  
* Assurez-vous que vous ne restaurez pas sur la même source. [En savoir plus](./backup-azure-restore-files-from-vm.md#step-2-ensure-the-machine-meets-the-requirements-before-executing-the-script).

### <a name="usererrorinstantrpnotfound---restore-failed-because-the-snapshot-of-the-vm-was-not-found"></a>UserErrorInstantRpNotFound : La restauration a échoué, car l’instantané de la machine virtuelle est introuvable

Code d’erreur : UserErrorInstantRpNotFound <br>
Message d’erreur : La restauration a échoué, car l’instantané de la machine virtuelle est introuvable. L’instantané a peut-être été supprimé, veuillez vérifier.<br>

Cette erreur se produit lorsque vous essayez de restaurer à partir d’un point de récupération qui n’a pas été transféré vers le coffre et qui a été supprimé lors de la phase de capture instantanée. 
<br>
Pour résoudre ce problème, essayez de restaurer la machine virtuelle à partir d’un autre point de restauration.<br>

#### <a name="common-errors"></a>Erreurs courantes 
| Détails de l’erreur | Solution de contournement |
| --- | --- |
| Échec de la restauration avec une erreur interne du cloud. |<ol><li>Le service cloud sur lequel vous essayez d’effectuer la restauration est configuré avec des paramètres DNS. Vous pouvez vérifier : <br>**$deployment = Get-AzureDeployment -ServiceName "ServiceName" -Slot "Production"     Get-AzureDns -DnsSettings $deployment.DnsSettings**.<br>Si **Adresse** est configuré, les paramètres DNS sont configurés.<br> <li>Le service cloud sur lequel vous tentez d’effectuer la restauration est configuré avec une **adresse IP réservée**, et les machines virtuelles existantes dans le service cloud sont à l’état arrêté. Vous pouvez vérifier qu’un service cloud a réservé une adresse IP à l’aide des cmdlets PowerShell suivantes : **$deployment = Get-AzureDeployment -ServiceName "servicename" -Slot "Production" $dep.ReservedIPName**. <br><li>Vous essayez de restaurer une machine virtuelle avec les configurations réseau spéciales suivantes dans le même service cloud : <ul><li>Machines virtuelles avec configuration d’un équilibreur de charge, internes et externes.<li>Machines virtuelles avec plusieurs adresses IP réservées. <li>Machines virtuelles avec plusieurs NIC. </ul><li>Sélectionnez un nouveau service cloud dans l’interface utilisateur ou consultez les [considérations relatives à la restauration](backup-azure-arm-restore-vms.md#restore-vms-with-special-configurations) des machines virtuelles avec des configurations réseau spéciales.</ol> |
| Le nom DNS sélectionné est déjà attribué : <br>Spécifiez un autre nom DNS, puis réessayez. |Ce nom DNS fait référence au nom du service cloud, qui se termine généralement par **.cloudapp.net**. Ce nom doit être unique. Si vous rencontrez cette erreur, vous devez choisir un autre nom de machine virtuelle pendant la restauration. <br><br> Cette erreur ne s’affiche que pour les utilisateurs du portail Azure. L’opération de restauration via PowerShell se déroule correctement, car elle ne fait que restaurer les disques et ne crée pas de machine virtuelle. L’erreur se rencontre lorsque la machine virtuelle est explicitement créée par vos soins après l’opération de restauration du disque. |
| La configuration de réseau virtuel spécifiée n’est pas correcte : <br>spécifiez une autre configuration de réseau virtuel et réessayez. |None |
| Le service cloud spécifié utilise une adresse IP réservée qui ne correspond pas à la configuration de la machine virtuelle en cours de restauration : <br>spécifiez un autre service cloud qui n’utilise pas d’adresse IP réservée. Ou choisissez un autre point de restauration à partir duquel effectuer la restauration. |None |
| Le service cloud a atteint sa limite sur le nombre de points de terminaison d’entrée : <br>relancez l’opération en spécifiant un autre service cloud ou en utilisant un point de terminaison existant. |None |
| Le compte de stockage cible et le coffre Recovery Services se trouvent dans deux régions distinctes : <br>vérifiez que le compte de stockage spécifié dans l’opération de restauration se trouve dans la même région Azure que votre coffre Recovery Services. |None |
| Le compte de stockage spécifié pour l’opération de restauration n’est pas pris en charge : <br>Seuls les comptes de stockage De base ou Standard avec des paramètres de réplication géoredondants ou redondants localement sont pris en charge. Sélectionnez un compte de stockage pris en charge. |None |
| Le type de compte de stockage spécifié pour l’opération de restauration n’est pas en ligne : <br>Vérifiez que le compte de stockage spécifié dans l’opération de restauration est en ligne. |Cette erreur peut se produire dans le cas d’une erreur temporaire dans Stockage Azure ou d’une panne. Choisissez un autre compte de stockage. |
| Le quota du groupe de ressources a été atteint : <br>Supprimez des groupes de ressources à partir du Portail Azure ou contactez le support Azure pour augmenter les limites. |None |
| Le sous-réseau sélectionné n’existe pas : <br>Sélectionnez un sous-réseau qui existe. |None |
| Le service Sauvegarde Azure n’a pas l’autorisation d’accéder aux ressources dans votre abonnement. |Pour résoudre cette erreur, commencez par restaurer les disques à l’aide de la procédure décrite dans [Restaurer des disques sauvegardés](backup-azure-arm-restore-vms.md#restore-disks). Utilisez ensuite les étapes PowerShell indiquées dans [Créer une machine virtuelle à partir de disques restaurés](backup-azure-vms-automation.md#restore-an-azure-vm). |

## <a name="backup-or-restore-takes-time"></a>Sauvegarde ou restauration qui prend du temps

Si votre sauvegarde prend plus de 12 heures, ou si la restauration prend plus de 6 heures, passez en revue les [meilleures pratiques](backup-azure-vms-introduction.md#best-practices) et les [considérations relatives aux performances](backup-azure-vms-introduction.md#backup-performance)

## <a name="vm-agent"></a>Agent VM

### <a name="set-up-the-vm-agent"></a>Configurer l’agent de machine virtuelle

En règle générale, l’agent de machine virtuelle est déjà présent dans les machines virtuelles qui sont créées à partir de la galerie Azure. Cependant, les machines virtuelles qui sont migrées à partir de centres de données locaux n’ont pas d’agent de machine virtuelle installé. Pour ces machines virtuelles, l’agent de machine virtuelle doit être installé de manière explicite.

#### <a name="windows-vms---set-up-the-agent"></a>Machines virtuelles Windows - Configurer l’agent

* Téléchargez et installez le fichier [MSI de l’agent](https://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409). Vous avez besoin de privilèges Administrateur pour terminer l’installation.
* Pour les machines virtuelles créées à l’aide du modèle de déploiement classique, [mettez à jour la propriété de la machine virtuelle](../virtual-machines/troubleshooting/install-vm-agent-offline.md#use-the-provisionguestagent-property-for-classic-vms) pour indiquer que l’agent est installé. Cette étape n’est pas requise pour les machines virtuelles Azure Resource Manager.

#### <a name="linux-vms---set-up-the-agent"></a>Machines virtuelles Linux - Configurer l’agent

* Installez la dernière version de l’agent à partir du référentiel de distribution. Pour plus de d’informations sur le nom du package, consultez le [référentiel de l’agent Linux](https://github.com/Azure/WALinuxAgent).
* Pour les machines virtuelles créées à l'aide du modèle de déploiement classique, [mettez à jour la propriété de la machine virtuelle](../virtual-machines/troubleshooting/install-vm-agent-offline.md#use-the-provisionguestagent-property-for-classic-vms) et vérifiez que l'agent est installé. Cette étape n’est pas requise pour les machines virtuelles du Gestionnaire des ressources.

### <a name="update-the-vm-agent"></a>Mettre à jour l’agent de machine virtuelle

#### <a name="windows-vms---update-the-agent"></a>Machines virtuelles Windows - Mettre à jour l’agent

* Pour mettre à jour l’agent de machine virtuelle, réinstallez les [fichiers binaires de l’agent de machine virtuelle](https://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409). Avant de mettre à jour l’agent, assurez-vous qu’aucune opération de sauvegarde ne se déroule pendant la mise à jour de l’agent de machine virtuelle.

#### <a name="linux-vms---update-the-agent"></a>Machines virtuelles Linux - Mettre à jour l’agent

* Pour mettre à jour l’agent de machine virtuelle Linux, suivez les instructions de l’article [Guide pratique pour mettre à jour l’agent Linux Azure sur une machine virtuelle](../virtual-machines/extensions/update-linux-agent.md?toc=/azure/virtual-machines/linux/toc.json).

    > [!NOTE]
    > Utilisez toujours le référentiel de distribution pour mettre à jour l’agent.

    Ne téléchargez pas le code de l’agent depuis GitHub. Si le dernier agent n’est pas disponible pour votre distribution, contactez le support de distribution pour savoir comment obtenir l’agent le plus récent. Vous pouvez également consulter les dernières informations sur [l’agent Linux de Windows Azure](https://github.com/Azure/WALinuxAgent/releases) dans le référentiel GitHub.

### <a name="validate-vm-agent-installation"></a>Valider l’installation de l’agent de machine virtuelle

Vérifiez la version de l’agent de machine virtuelle sur les machines virtuelles Windows :

1. Connectez-vous à la machine virtuelle Azure et accédez au dossier **C:\WindowsAzure\Packages**. Vous devez trouver le fichier **WaAppAgent.exe**.
2. Cliquez avec le bouton droit sur le fichier et accédez à **Propriétés**. Sélectionnez ensuite l’onglet **Détails**. Le champ **Version du produit** doit indiquer 2.6.1198.718 ou une version ultérieure.

## <a name="troubleshoot-vm-snapshot-issues"></a>Résoudre les problèmes de capture instantanée de machine virtuelle

La sauvegarde de machines virtuelles émet des commandes de capture instantanée à destination du stockage sous-jacent. Le fait de ne pas avoir accès au stockage ou tout retard dans l’exécution d’une tâche de capture instantanée peut faire échouer le travail de sauvegarde. Voici les causes possibles de l’échec d’une tâche de capture instantanée :

* **Les machines virtuelles pour lesquelles la sauvegarde SQL Server est configurée peuvent provoquer des retards de tâches de capture instantanée**. Par défaut, la sauvegarde de machines virtuelles crée une sauvegarde complète VSS sur les machines virtuelles Windows. Les machines virtuelles qui exécutent SQL Server, avec la sauvegarde SQL Server configurée, peuvent subir des retards dans les captures instantanées. Si des retards dans les captures instantanées font échouer la sauvegarde, définissez la clé de Registre suivante :

   ```console
   [HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\BCDRAGENT]
   "USEVSSCOPYBACKUP"="TRUE"
   ```

* **L’état de la machine virtuelle est rapporté de manière incorrecte en raison de l’arrêt de la machine virtuelle dans RDP**. Si vous avez arrêté la machine virtuelle à l’aide du bureau distant, vérifiez que l’état de la machine virtuelle dans le portail est correct. Si l’état est incorrect, utilisez l’option **Arrêt** dans le tableau de bord de la machine virtuelle du portail pour éteindre la machine virtuelle.
* **Si plus de quatre machines virtuelles partagent le même service cloud, répartissez les machines virtuelles sur plusieurs stratégies de sauvegarde**. Échelonnez les heures de sauvegarde de façon que quatre sauvegardes de machine virtuelle maximum ne démarrent pas en même temps. Essayez de décaler d’au moins une heure les démarrages d’une stratégie à une autre.
* **La machine virtuelle s’exécute avec un niveau de mémoire ou d’UC élevé**. Si la machine virtuelle sollicite fortement la mémoire ou le processeur (plus de 90 pour cent), votre tâche de capture instantanée est mise en file d’attente et retardée. Elle peut éventuellement arriver à expiration. Si ce problème se produit, essayez de procéder à une sauvegarde à la demande.

## <a name="networking"></a>Mise en réseau

Le protocole DHCP doit être activé dans l’invité pour que la sauvegarde de la machine virtuelle IaaS fonctionne. Si vous avez besoin d’une adresse IP privée statique, configurez-la via le Portail Azure ou PowerShell. Vérifiez que l’option DHCP à l’intérieur de la machine virtuelle est activée.
Pour obtenir plus d’informations sur la configuration d’une adresse IP statique via PowerShell :

* [Ajout d’une adresse IP interne statique à une machine virtuelle existante](/powershell/module/az.network/set-aznetworkinterfaceipconfig#description)
* [Modifier la méthode d’allocation pour une adresse IP privée affectée à une interface réseau](../virtual-network/virtual-networks-static-private-ip-arm-ps.md#change-the-allocation-method-for-a-private-ip-address-assigned-to-a-network-interface)