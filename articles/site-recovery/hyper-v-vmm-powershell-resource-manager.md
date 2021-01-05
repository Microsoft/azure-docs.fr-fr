---
title: Configurer la reprise d’activité après sinistre pour Hyper-V (avec VMM) sur un site secondaire avec Azure Site Recovery/PowerShell
description: Décrit comment configurer la reprise d’activité de machines virtuelles Hyper-V dans des clouds VMM sur un site VMM secondaire avec Azure Site Recovery et PowerShell.
services: site-recovery
author: sujayt
manager: rochakm
ms.topic: article
ms.date: 1/10/2020
ms.author: sutalasi
ms.openlocfilehash: a4f7d330db9a4a0d9b435ebe7527e55e37c254e2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87086215"
---
# <a name="set-up-disaster-recovery-of-hyper-v-vms-to-a-secondary-site-by-using-powershell-resource-manager"></a>Configurer la reprise d’activité de machines virtuelles Hyper-V sur un site secondaire à l’aide de PowerShell (Resource Manager)

Cet article montre comment automatiser les étapes de la réplication des machines virtuelles Hyper-V dans les clouds System Center Virtual Machine Manager sur le cloud Virtual Machine Manager d’un site local secondaire en utilisant [Azure Site Recovery](site-recovery-overview.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis

- Examinez [l’architecture et les composants du scénario](hyper-v-vmm-architecture.md).
- Vérifiez les [exigences de prise en charge](./vmware-physical-secondary-support-matrix.md) pour tous les composants.
- Assurez-vous que les serveurs Virtual Machine Manager et les hôtes Hyper-V sont conformes aux [exigences de prise en charge](./vmware-physical-secondary-support-matrix.md).
- Vérifiez que les machines virtuelles à répliquer sont conformes à la [prise en charge de la machine répliquée](./vmware-physical-secondary-support-matrix.md).

## <a name="prepare-for-network-mapping"></a>Préparer le mappage réseau

Le [mappage réseau](hyper-v-vmm-network-mapping.md) effectue un mappage entre les réseaux de machines virtuelles Virtual Machine Manager locaux dans les clouds source et cible. Il effectue les opérations suivantes :

- Connecte les machines virtuelles aux réseaux de machines virtuelles cibles appropriés après le basculement.
- Place de manière optimale les machines virtuelles de réplication sur les serveurs hôtes Hyper-V cibles.
- Si vous ne configurez pas le mappage réseau, les machines virtuelles réplica ne seront connectées à aucun réseau de machines virtuelles après le basculement.

Préparez Virtual Machine Manager comme suit :

- Assurez-vous que vous avez des [réseaux logiques Virtual Machine Manager](/system-center/vmm/network-logical) sur les serveurs source et cible Virtual Machine Manager :
  - Le réseau logique sur le serveur source doit être associé au cloud source dans lequel se trouvent les hôtes Hyper-V.
  - Le réseau logique sur le serveur cible doit être associé au cloud cible.
- Assurez-vous que vous avez des [réseaux de machines virtuelles](/system-center/vmm/network-virtual) sur les serveurs Virtual Machine Manager source et cible. Les réseaux de machines virtuelles doivent être associés au réseau logique dans chaque emplacement.
- Connectez les machines virtuelles sur les hôtes Hyper-V sources au réseau de machines virtuelles source.

## <a name="prepare-for-powershell"></a>Préparer PowerShell

Assurez-vous qu’Azure PowerShell est prêt à l’emploi :

- Si vous utilisez déjà PowerShell, mettez à niveau vers la version 0.8.10 ou version ultérieure. [En savoir plus](/powershell/azure/) sur la configuration de PowerShell.
- Après avoir installé et configuré PowerShell, examinez les [cmdlets de service](/powershell/azure/).
- Pour en savoir plus sur l’utilisation des valeurs de paramètre, des entrées dans PowerShell, consultez le guide [Prise en main](/powershell/azure/get-started-azureps).

## <a name="set-up-a-subscription"></a>Configurer un abonnement

1. Dans PowerShell, connectez-vous à votre compte Azure.

   ```azurepowershell
   $UserName = "<user@live.com>"
   $Password = "<password>"
   $SecurePassword = ConvertTo-SecureString -AsPlainText $Password -Force
   $Cred = New-Object System.Management.Automation.PSCredential -ArgumentList $UserName, $SecurePassword
   Connect-AzAccount #-Credential $Cred
   ```

1. Récupérez la liste de vos abonnements, avec les ID d’abonnement. Notez l’ID de l’abonnement dans lequel vous voulez créer le coffre Recovery Services.

   ```azurepowershell
   Get-AzSubscription
   ```

1. Définissez l’abonnement du coffre.

   ```azurepowershell
   Set-AzContext –SubscriptionID <subscriptionId>
   ```

## <a name="create-a-recovery-services-vault"></a>Créer un coffre Recovery Services

1. Si vous ne disposez d’aucun groupe de ressources Azure Resource Manager, créez-en un.

   ```azurepowershell
   New-AzResourceGroup -Name #ResourceGroupName -Location #location
   ```

1. Créez un nouveau coffre Recovery Services. Enregistrez l’objet de coffre dans une variable pour une utilisation ultérieure.

   ```azurepowershell
   $vault = New-AzRecoveryServicesVault -Name #vaultname -ResourceGroupName #ResourceGroupName -Location #location
   ```

   Vous pouvez récupérer l’objet de coffre après sa création à l’aide de l’applet de commande `Get-AzRecoveryServicesVault`.

## <a name="set-the-vault-context"></a>Définir le contexte du coffre

1. Récupérez un coffre existant.

   ```azurepowershell
   $vault = Get-AzRecoveryServicesVault -Name #vaultname
   ```

1. Définir le contexte d’archivage.

   ```azurepowershell
   Set-AzRecoveryServicesAsrVaultContext -Vault $vault
   ```

## <a name="install-the-site-recovery-provider"></a>Installer le fournisseur Site Recovery

1. Sur la machine Virtual Machine Manager, créez un répertoire en exécutant la commande suivante :

   ```azurepowershell
   New-Item -Path C:\ASR -ItemType Directory
   ```

1. Extrayez les fichiers à l’aide du fichier de configuration du fournisseur téléchargé.

   ```console
   pushd C:\ASR\
   .\AzureSiteRecoveryProvider.exe /x:. /q
   ```

1. Installez le fournisseur et attendez que l’installation se termine.

   ```console
   .\SetupDr.exe /i
   $installationRegPath = "HKLM:\Software\Microsoft\Microsoft System Center Virtual Machine Manager Server\DRAdapter"
   do
   {
     $isNotInstalled = $true;
     if(Test-Path $installationRegPath)
     {
       $isNotInstalled = $false;
     }
   }While($isNotInstalled)
   ```

1. Inscrivez le serveur dans le coffre.

   ```console
   $BinPath = $env:SystemDrive+"\Program Files\Microsoft System Center 2012 R2\Virtual Machine Manager\bin"
   pushd $BinPath
   $encryptionFilePath = "C:\temp\".\DRConfigurator.exe /r /Credentials $VaultSettingFilePath /vmmfriendlyname $env:COMPUTERNAME /dataencryptionenabled $encryptionFilePath /startvmmservice
   ```

## <a name="create-and-associate-a-replication-policy"></a>Créer et associer une stratégie de réplication

1. Créez une stratégie de réplication, dans ce cas pour Hyper-V 2012 R2, de la manière suivante :

   ```azurepowershell
   $ReplicationFrequencyInSeconds = "300";        #options are 30,300,900
   $PolicyName = “replicapolicy”
   $RepProvider = HyperVReplica2012R2
   $Recoverypoints = 24                    #specify the number of hours to retain recovery points
   $AppConsistentSnapshotFrequency = 4 #specify the frequency (in hours) at which app consistent snapshots are taken
   $AuthMode = "Kerberos"  #options are "Kerberos" or "Certificate"
   $AuthPort = "8083"  #specify the port number that will be used for replication traffic on Hyper-V hosts
   $InitialRepMethod = "Online" #options are "Online" or "Offline"

   $policyresult = New-AzRecoveryServicesAsrPolicy -Name $policyname -ReplicationProvider $RepProvider -ReplicationFrequencyInSeconds $Replicationfrequencyinseconds -NumberOfRecoveryPointsToRetain $recoverypoints -ApplicationConsistentSnapshotFrequencyInHours $AppConsistentSnapshotFrequency -Authentication $AuthMode -ReplicationPort $AuthPort -ReplicationMethod $InitialRepMethod
   ```

   > [!NOTE]
   > Le cloud Virtual Machine Manager peut contenir des hôtes Hyper-V s’exécutant sur différentes versions de Windows Server, alors que la stratégie de réplication est destinée à une version spécifique d’un système d’exploitation. Si vos hôtes s’exécutent sur des systèmes d’exploitation différents, créez des stratégies de réplication distinctes pour chaque système. Par exemple, si vous avez cinq hôtes exécutés sur Windows Servers 2012 et trois hôtes exécutés sur Windows Server 2012 R2, créez deux stratégies de réplication. Vous en créez une pour chaque type de système d’exploitation.

1. Récupérez le conteneur de protection principal (cloud Virtual Machine Manager principal) et le conteneur de protection de récupération (cloud Virtual Machine Manager de récupération).

   ```azurepowershell
   $PrimaryCloud = "testprimarycloud"
   $primaryprotectionContainer = Get-AzRecoveryServicesAsrProtectionContainer -FriendlyName $PrimaryCloud;

   $RecoveryCloud = "testrecoverycloud"
   $recoveryprotectionContainer = Get-AzRecoveryServicesAsrProtectionContainer -FriendlyName $RecoveryCloud;
   ```

1. Récupérez la stratégie de réplication que vous avez créée en utilisant le nom convivial.

   ```azurepowershell
   $policy = Get-AzRecoveryServicesAsrPolicy -FriendlyName $policyname
   ```

1. Lancez l’association du conteneur de protection (cloud Virtual Machine Manager) avec la stratégie de réplication.

   ```azurepowershell
   $associationJob  = New-AzRecoveryServicesAsrProtectionContainerMapping -Policy $Policy -PrimaryProtectionContainer $primaryprotectionContainer -RecoveryProtectionContainer $recoveryprotectionContainer
   ```

1. Attendez que la tâche d’association de la stratégie se termine. Pour vérifier que la tâche est terminée, utilisez l’extrait de code PowerShell suivant :

   ```azurepowershell
   $job = Get-AzRecoveryServicesAsrJob -Job $associationJob

   if($job -eq $null -or $job.StateDescription -ne "Completed")
   {
     $isJobLeftForProcessing = $true;
   }
   ```

1. Une fois la tâche terminée, exécutez la commande suivante :

   ```azurepowershell
   if($isJobLeftForProcessing)
   {
     Start-Sleep -Seconds 60
   }
   While($isJobLeftForProcessing)
   ```

Pour vérifier que l’opération est terminée, suivez les étapes décrites à la section [Analyser l’activité](#monitor-activity).

##  <a name="configure-network-mapping"></a>Configurer le mappage réseau

1. Utilisez cette commande pour récupérer des serveurs pour le coffre actif. La commande stocke les serveurs Site Recovery dans la variable de tableau `$Servers`.

   ```azurepowershell
   $Servers = Get-AzRecoveryServicesAsrFabric
   ```

1. Exécutez cette commande pour récupérer les réseaux pour le serveur Virtual Machine Manager source et le serveur Virtual Machine Manager cible.

   ```azurepowershell
   $PrimaryNetworks = Get-AzRecoveryServicesAsrNetwork -Fabric $Servers[0]

   $RecoveryNetworks = Get-AzRecoveryServicesAsrNetwork -Fabric $Servers[1]
   ```

   > [!NOTE]
   > Le serveur Virtual Machine Manager source peut être le premier ou le deuxième serveur dans le tableau de serveurs. Vérifiez les noms de serveur Virtual Machine Manager et récupérez les réseaux appropriés.

1. L’applet de commande crée un mappage entre le réseau principal et le réseau de récupération. Il spécifie le réseau principal comme premier élément de `$PrimaryNetworks`. Il spécifie le réseau de récupération comme premier élément de `$RecoveryNetworks`.

   ```azurepowershell
   New-AzRecoveryServicesAsrNetworkMapping -PrimaryNetwork $PrimaryNetworks[0] -RecoveryNetwork $RecoveryNetworks[0]
   ```

## <a name="enable-protection-for-vms"></a>Activer la protection des machines virtuelles

Après avoir correctement configuré les serveurs, les clouds et les réseaux, activez la protection pour les machines virtuelles dans le cloud.

1. Afin d’activer la protection, exécutez la commande suivante pour récupérer le conteneur de protection :

   ```azurepowershell
   $PrimaryProtectionContainer = Get-AzRecoveryServicesAsrProtectionContainer -FriendlyName $PrimaryCloudName
   ```

1. Obtenez l’entité de protection (machine virtuelle) de la façon suivante :

   ```azurepowershell
   $protectionEntity = Get-AzRecoveryServicesAsrProtectableItem -FriendlyName $VMName -ProtectionContainer $PrimaryProtectionContainer
   ```

1. Activez la réplication de la machine virtuelle.

   ```azurepowershell
   $jobResult = New-AzRecoveryServicesAsrReplicationProtectedItem -ProtectableItem $protectionentity -ProtectionContainerMapping $policy -VmmToVmm
   ```

> [!NOTE]
> Si vous voulez répliquer sur des disques managés activés par une CMK dans Azure, effectuez les étapes suivantes avec la commande Az de PowerShell 3.3.0 et versions ultérieures :
>
> 1. Activez le basculement vers des disques managés en mettant à jour les propriétés des machines virtuelles.
> 1. Utilisez l’applet de commande `Get-AzRecoveryServicesAsrReplicationProtectedItem` pour extraire l’ID de chaque disque de l’élément protégé.
> 1. Créez un objet de dictionnaire à l’aide de l’applet de commande `New-Object "System.Collections.Generic.Dictionary``2[System.String,System.String]"` destiné à contenir le mappage de l’ID de disque au jeu de chiffrement de disque. Vous devez créer ces jeux de chiffrement de disque au préalable dans la région cible.
> 1. Mettez à jour les propriétés des machines virtuelles à l’aide de l’applet de commande `Set-AzRecoveryServicesAsrReplicationProtectedItem` en passant l’objet du dictionnaire dans le paramètre **DiskIdToDiskEncryptionSetMap**.

## <a name="run-a-test-failover"></a>Exécuter un test de basculement

Pour tester votre déploiement, exécutez un test de basculement pour une seule machine virtuelle. Vous pouvez également créer un plan de récupération qui contient plusieurs machines virtuelles et exécuter un test de basculement pour le plan. Il simule votre mécanisme de basculement et de récupération dans un réseau isolé.

1. Récupérez la machine virtuelle dans laquelle les machines virtuelles basculent.

   ```azurepowershell
   $Servers = Get-AzRecoveryServicesASRFabric
   $RecoveryNetworks = Get-AzRecoveryServicesAsrNetwork -Name $Servers[1]
   ```

1. Exécution d’un basculement de test.

   Pour une machine virtuelle :

   ```azurepowershell
   $protectionEntity = Get-AzRecoveryServicesAsrProtectableItem -FriendlyName $VMName -ProtectionContainer $PrimaryprotectionContainer

   $jobIDResult = Start-AzRecoveryServicesAsrTestFailoverJob -Direction PrimaryToRecovery -ReplicationProtectedItem $protectionEntity -VMNetwork $RecoveryNetworks[1]
   ```

   Pour un plan de récupération :

   ```azurepowershell
   $recoveryplanname = "test-recovery-plan"

   $recoveryplan = Get-AzRecoveryServicesAsrRecoveryPlan -FriendlyName $recoveryplanname

   $jobIDResult = Start-AzRecoveryServicesAsrTestFailoverJob -Direction PrimaryToRecovery -RecoveryPlan $recoveryplan -VMNetwork $RecoveryNetworks[1]
   ```

Pour vérifier que l’opération est terminée, suivez les étapes décrites à la section [Analyser l’activité](#monitor-activity).

## <a name="run-planned-and-unplanned-failovers"></a>Exécuter des basculements planifiés et non planifiés

1. Exécutez un basculement planifié.

   Pour une machine virtuelle :

   ```azurepowershell
   $protectionEntity = Get-AzRecoveryServicesAsrProtectableItem -Name $VMName -ProtectionContainer $PrimaryprotectionContainer

   $jobIDResult = Start-AzRecoveryServicesAsrPlannedFailoverJob -Direction PrimaryToRecovery -ReplicationProtectedItem $protectionEntity
   ```

   Pour un plan de récupération :

   ```azurepowershell
   $recoveryplanname = "test-recovery-plan"

   $recoveryplan = Get-AzRecoveryServicesAsrRecoveryPlan -FriendlyName $recoveryplanname

   $jobIDResult = Start-AzRecoveryServicesAsrPlannedFailoverJob -Direction PrimaryToRecovery -RecoveryPlan $recoveryplan
   ```

1. Exécutez un basculement non planifié.

   Pour une machine virtuelle :

   ```azurepowershell
   $protectionEntity = Get-AzRecoveryServicesAsrProtectableItem -Name $VMName -ProtectionContainer $PrimaryprotectionContainer

   $jobIDResult = Start-AzRecoveryServicesAsrUnplannedFailoverJob -Direction PrimaryToRecovery -ReplicationProtectedItem $protectionEntity
   ```

   Pour un plan de récupération :

   ```azurepowershell
   $recoveryplanname = "test-recovery-plan"

   $recoveryplan = Get-AzRecoveryServicesAsrRecoveryPlan -FriendlyName $recoveryplanname

   $jobIDResult = Start-AzRecoveryServicesAsrUnplannedFailoverJob -Direction PrimaryToRecovery -RecoveryPlan $recoveryplan
   ```

## <a name="monitor-activity"></a>Analyser l’activité

Utilisez les commandes suivantes pour analyser l’activité de basculement. Attendez que le traitement se termine entre les travaux.

```azurepowershell
Do
{
    $job = Get-AzRecoveryServicesAsrJob -TargetObjectId $associationJob.JobId;
    Write-Host "Job State:{0}, StateDescription:{1}" -f Job.State, $job.StateDescription;
    if($job -eq $null -or $job.StateDescription -ne "Completed")
    {
        $isJobLeftForProcessing = $true;
    }

if($isJobLeftForProcessing)
    {
        Start-Sleep -Seconds 60
    }
}While($isJobLeftForProcessing)
```

## <a name="next-steps"></a>Étapes suivantes

[En savoir plus](/powershell/module/az.recoveryservices) sur Site Recovery avec les cmdlets PowerShell Resource Manager.
