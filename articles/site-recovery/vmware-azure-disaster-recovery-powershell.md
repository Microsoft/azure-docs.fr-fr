---
title: Configurer la récupération d’urgence VMware à l’aide de PowerShell dans Azure Site Recovery
description: Découvrez comment configurer la réplication et le basculement vers Azure pour la reprise d’activité de machines virtuelles VMware à l’aide de PowerShell dans Azure Site Recovery.
author: sujayt
manager: rochakm
ms.service: site-recovery
ms.date: 01/10/2020
ms.topic: conceptual
ms.author: sutalasi
ms.openlocfilehash: de25a3f9df04b09a7337dc889a688a171d98db28
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86129914"
---
# <a name="set-up-disaster-recovery-of-vmware-vms-to-azure-with-powershell"></a>Configurer la reprise d’activité des machines virtuelles VMware sur Azure avec PowerShell

Dans cet article, vous découvrez comment répliquer et basculer des machines virtuelles VMware vers Azure à l’aide d’Azure PowerShell.

Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> - Créer un coffre Recovery Services et définir le contexte du coffre.
> - Valider l’inscription du serveur dans le coffre.
> - Configurer la réplication, y compris une stratégie de réplication. Ajouter un serveur vCenter et détecter des machines virtuelles.
> - Ajouter un serveur vCenter et effectuer des opérations de détection
> - Créer des comptes de stockage pour conserver les journaux ou les données de réplication, et répliquer les machines virtuelles.
> - Effectuer un basculement. Configurer les paramètres de basculement pour la réplication des machines virtuelles.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis

Avant de commencer :

- Assurez-vous que vous comprenez [l’architecture et les composants du scénario](vmware-azure-architecture.md).
- Vérifiez les [exigences de prise en charge](./vmware-physical-azure-support-matrix.md) pour tous les composants.
- Vous devez disposer du module Azure PowerShell `Az`. Si vous devez installer ou mettre à niveau Azure PowerShell, consultez le [guide sur l’installation et la configuration d’Azure PowerShell](/powershell/azure/install-az-ps).

## <a name="log-into-azure"></a>Se connecter à Azure

Connectez-vous à votre abonnement Azure à l’aide de la cmdlet Connect-Azccount :

```azurepowershell
Connect-AzAccount
```
Sélectionnez l’abonnement Azure vers lequel répliquer vos machines virtuelles VMware. Utilisez la cmdlet Get-AzSubscription pour obtenir la liste des abonnements Azure auxquels vous avez accès. Sélectionnez l’abonnement Azure à utiliser à l’aide de la cmdlet Select-AzSubscription.

```azurepowershell
Select-AzSubscription -SubscriptionName "ASR Test Subscription"
```
## <a name="set-up-a-recovery-services-vault"></a>Configurez un coffre Recovery Services.

1. Créez un groupe de ressources dans lequel créer le coffre Recovery Services. Dans l’exemple ci-dessous, le groupe de ressources est nommé VMwareDRtoAzurePS et créé dans la région Asie Est.

   ```azurepowershell
   New-AzResourceGroup -Name "VMwareDRtoAzurePS" -Location "East Asia"
   ```
   ```
   ResourceGroupName : VMwareDRtoAzurePS
   Location          : eastasia
   ProvisioningState : Succeeded
   Tags              :
   ResourceId        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/VMwareDRtoAzurePS
   ```

2. Créez un coffre Recovery Services. Dans l’exemple ci-dessous, le coffre Recovery Services est nommé VMwareDRToAzurePs et créé dans la région Asie Est, ainsi que dans le groupe de ressources créé à l’étape précédente.

   ```azurepowershell
   New-AzRecoveryServicesVault -Name "VMwareDRToAzurePs" -Location "East Asia" -ResourceGroupName "VMwareDRToAzurePs"
   ```
   ```
   Name              : VMwareDRToAzurePs
   ID                : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/VMwareDRToAzurePs/providers/Microsoft.RecoveryServices/vaults/VMwareDRToAzurePs
   Type              : Microsoft.RecoveryServices/vaults
   Location          : eastasia
   ResourceGroupName : VMwareDRToAzurePs
   SubscriptionId    : xxxxxxxx-xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
   Properties        : Microsoft.Azure.Commands.RecoveryServices.ARSVaultProperties
   ```

3. Téléchargez la clé d’inscription du coffre. La clé d’inscription du coffre est utilisée pour inscrire le serveur de configuration local dans ce coffre. L’inscription fait partie du traitement d’installation du logiciel du serveur de configuration.

   ```azurepowershell
   #Get the vault object by name and resource group and save it to the $vault PowerShell variable 
   $vault = Get-AzRecoveryServicesVault -Name "VMwareDRToAzurePS" -ResourceGroupName "VMwareDRToAzurePS"

   #Download vault registration key to the path C:\Work
   Get-AzRecoveryServicesVaultSettingsFile -SiteRecovery -Vault $Vault -Path "C:\Work\"
   ```
   ```
   FilePath
   --------
   C:\Work\VMwareDRToAzurePs_2017-11-23T19-52-34.VaultCredentials
   ```

4. Utiliser la clé d’inscription du coffre téléchargée, et suivez les étapes décrites dans les articles répertoriés ci-dessous pour terminer l’installation et l’inscription du serveur de configuration.
   - [Sélectionner vos objectifs en matière de protection](vmware-azure-set-up-source.md#choose-your-protection-goals)
   - [Configurer l’environnement source](vmware-azure-set-up-source.md#set-up-the-configuration-server)

### <a name="set-the-vault-context"></a>Définir le contexte du coffre

Définissez le contexte d’archivage à l’aide de la cmdlet Set-ASRVaultContext. Ensuite, les opérations suivantes d’Azure Site Recovery dans la session PowerShell sont effectuées dans le contexte du coffre sélectionné.

> [!TIP]
> Le module Azure Site Recovery PowerShell (module Az.RecoveryServices) est fourni avec des alias faciles à utiliser pour la plupart des cmdlets. Les cmdlets du module se présentent sous la forme *\<Operation>-**AzRecoveryServicesAsr**\<Object>* et possèdent des alias équivalents, au format *\<Operation>-**ASR**\<Object>* . Vous pouvez remplacer les alias de cmdlet pour plus de facilité d’utilisation.

Dans l’exemple ci-dessous, les détails du coffre obtenus via la variable $vault servent à spécifier le contexte d’archivage pour la session PowerShell.

   ```azurepowershell
   Set-ASRVaultContext -Vault $vault
   ```
   ```
   ResourceName      ResourceGroupName ResourceNamespace          ResourceType
   ------------      ----------------- -----------------          -----------
   VMwareDRToAzurePs VMwareDRToAzurePs Microsoft.RecoveryServices vaults
   ```

Au lieu de la cmdlet Set-ASRVaultContext, vous pouvez utiliser la cmdlet Import-AzRecoveryServicesAsrVaultSettingsFile pour définir le contexte d’archivage. Spécifiez le chemin d’accès du fichier de clé d’inscription du coffre en tant que paramètre -path de la cmdlet Import-AzRecoveryServicesAsrVaultSettingsFile. Par exemple :

   ```azurepowershell
   Get-AzRecoveryServicesVaultSettingsFile -SiteRecovery -Vault $Vault -Path "C:\Work\"
   Import-AzRecoveryServicesAsrVaultSettingsFile -Path "C:\Work\VMwareDRToAzurePs_2017-11-23T19-52-34.VaultCredentials"
   ```
Les sections suivantes de cet article supposent que le contexte d’archivage pour les opérations Azure Site Recovery a été défini.

## <a name="validate-vault-registration"></a>Valider l’inscription du coffre

Pour cet exemple, nous avons les éléments suivants :

- Un serveur de configuration (**ConfigurationServer**) a été inscrit dans ce coffre.
- Un serveur de traitement supplémentaire (**ScaleOut-ProcessServer**) a été inscrit dans *ConfigurationServer*.
- Des comptes (**vCenter_account**, **WindowsAccount** et **LinuxAccount**) ont été définis sur le serveur de configuration. Ces comptes sont utilisés pour ajouter le serveur vCenter afin de détecter les machines virtuelles et pour installer (push) le logiciel du service Mobilité sur les serveurs Windows et Linux qui doivent être répliqués.

1. Les serveurs de configuration inscrits sont représentés par un objet de structure dans Site Recovery. Obtenez la liste des objets de structure dans le coffre et identifiez le serveur de configuration.

   ```azurepowershell
   # Verify that the Configuration server is successfully registered to the vault
   $ASRFabrics = Get-AzRecoveryServicesAsrFabric
   $ASRFabrics.count
   ```
   ```
   1
   ```
   ```azurepowershell
   #Print details of the Configuration Server
   $ASRFabrics[0]
   ```
   ```
   Name                  : 2c33d710a5ee6af753413e97f01e314fc75938ea4e9ac7bafbf4a31f6804460d
   FriendlyName          : ConfigurationServer
   ID                    : /Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/VMwareDRToAzurePs/providers/Microsoft.RecoveryServices/vaults/VMwareDRToAzurePs/replicationFabrics
                           /2c33d710a5ee6af753413e97f01e314fc75938ea4e9ac7bafbf4a31f6804460d
   Type                  : Microsoft.RecoveryServices/vaults/replicationFabrics
   FabricType            : VMware
   SiteIdentifier        : ef7a1580-f356-4a00-aa30-7bf80f952510
   FabricSpecificDetails : Microsoft.Azure.Commands.RecoveryServices.SiteRecovery.ASRVMWareSpecificDetails
   ```

2. Identifiez les serveurs de traitement qui peuvent être utilisés pour répliquer des machines.

   ```azurepowershell
   $ProcessServers = $ASRFabrics[0].FabricSpecificDetails.ProcessServers
   for($i=0; $i -lt $ProcessServers.count; $i++) {
    "{0,-5} {1}" -f $i, $ProcessServers[$i].FriendlyName
   }
   ```
   ```
   0     ScaleOut-ProcessServer
   1     ConfigurationServer
   ```

   Dans la sortie ci-dessus, ***$ProcessServers[0]*** correspond à *ScaleOut-ProcessServer*, et ***$ProcessServers[1]*** correspond au rôle de serveur de traitement sur *ConfigurationServer*.

3. Identifiez les comptes qui ont été définis sur le serveur de configuration.

   ```azurepowershell
   $AccountHandles = $ASRFabrics[0].FabricSpecificDetails.RunAsAccounts
   #Print the account details
   $AccountHandles
   ```
   ```
   AccountId AccountName
   --------- -----------
   1         vCenter_account
   2         WindowsAccount
   3         LinuxAccount
   ```

   Dans la sortie ci-dessus, ***$AccountHandles[0]*** correspond au compte *vCenter_account*, ***$AccountHandles[1]*** au compte *WindowsAccount*, et ***$AccountHandles[2]*** au compte *LinuxAccount*.

## <a name="create-a-replication-policy"></a>Créer une stratégie de réplication

À cette étape, deux stratégies de réplication sont créées. La première sert à répliquer les machines virtuelles VMware vers Azure, et la seconde sert à répliquer sur les machines virtuelles basculées s’exécutant dans Azure vers le site VMware local.

> [!NOTE]
> La plupart des opérations Azure Site Recovery sont exécutées de façon asynchrone. Lorsque vous lancez une opération, un travail Azure Site Recovery est envoyé et un objet de suivi de travail est renvoyé. Cet objet de suivi de tâche permet de surveiller l’état de l’opération.

1. Créez une stratégie de réplication nommée *ReplicationPolicy* pour répliquer les machines virtuelles VMware vers Azure avec les propriétés spécifiées.

   ```azurepowershell
   $Job_PolicyCreate = New-AzRecoveryServicesAsrPolicy -VMwareToAzure -Name "ReplicationPolicy" -RecoveryPointRetentionInHours 24 -ApplicationConsistentSnapshotFrequencyInHours 4 -RPOWarningThresholdInMinutes 60

   # Track Job status to check for completion
   while (($Job_PolicyCreate.State -eq "InProgress") -or ($Job_PolicyCreate.State -eq "NotStarted")){
           sleep 10;
           $Job_PolicyCreate = Get-ASRJob -Job $Job_PolicyCreate
   }

   #Display job status
   $Job_PolicyCreate
   ```
   ```
   Name             : 8d18e2d9-479f-430d-b76b-6bc7eb2d0b3e
   ID               : /Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/VMwareDRToAzurePs/providers/Microsoft.RecoveryServices/vaults/VMwareDRToAzurePs/replicationJobs/8d18e2d
                      9-479f-430d-b76b-6bc7eb2d0b3e
   Type             :
   JobType          : AddProtectionProfile
   DisplayName      : Create replication policy
   ClientRequestId  : a162b233-55d7-4852-abac-3d595a1faac2 ActivityId: 9895234a-90ea-4c1a-83b5-1f2c6586252a
   State            : Succeeded
   StateDescription : Completed
   StartTime        : 11/24/2017 2:49:24 AM
   EndTime          : 11/24/2017 2:49:23 AM
   TargetObjectId   : ab31026e-4866-5440-969a-8ebcb13a372f
   TargetObjectType : ProtectionProfile
   TargetObjectName : ReplicationPolicy
   AllowedActions   :
   Tasks            : {Prerequisites check for creating the replication policy, Creating the replication policy}
   Errors           : {}
   ```

2. Créez une stratégie de réplication à utiliser pour la restauration à partir d’Azure vers le site de VMware local.

   ```azurepowershell
   $Job_FailbackPolicyCreate = New-AzRecoveryServicesAsrPolicy -AzureToVMware -Name "ReplicationPolicy-Failback" -RecoveryPointRetentionInHours 24 -ApplicationConsistentSnapshotFrequencyInHours 4 -RPOWarningThresholdInMinutes 60
   ```

   Utilisez les détails du travail dans *$Job_FailbackPolicyCreate* pour suivre l’opération jusqu’à la fin.

   * Créez un mappage de conteneur de protection pour mapper les stratégies de réplication avec le serveur de configuration.

   ```azurepowershell
   #Get the protection container corresponding to the Configuration Server
   $ProtectionContainer = Get-AzRecoveryServicesAsrProtectionContainer -Fabric $ASRFabrics[0]

   #Get the replication policies to map by name.
   $ReplicationPolicy = Get-AzRecoveryServicesAsrPolicy -Name "ReplicationPolicy"
   $FailbackReplicationPolicy = Get-AzRecoveryServicesAsrPolicy -Name "ReplicationPolicy-Failback"

   # Associate the replication policies to the protection container corresponding to the Configuration Server.

   $Job_AssociatePolicy = New-AzRecoveryServicesAsrProtectionContainerMapping -Name "PolicyAssociation1" -PrimaryProtectionContainer $ProtectionContainer -Policy $ReplicationPolicy

   # Check the job status
   while (($Job_AssociatePolicy.State -eq "InProgress") -or ($Job_AssociatePolicy.State -eq "NotStarted")){
           sleep 10;
           $Job_AssociatePolicy = Get-ASRJob -Job $Job_AssociatePolicy
   }
   $Job_AssociatePolicy.State

   <# In the protection container mapping used for failback (replicating failed over virtual machines
      running in Azure, to the primary VMware site.) the protection container corresponding to the
      Configuration server acts as both the Primary protection container and the recovery protection
      container
   #>
    $Job_AssociateFailbackPolicy = New-AzRecoveryServicesAsrProtectionContainerMapping -Name "FailbackPolicyAssociation" -PrimaryProtectionContainer $ProtectionContainer -RecoveryProtectionContainer $ProtectionContainer -Policy $FailbackReplicationPolicy

   # Check the job status
   while (($Job_AssociateFailbackPolicy.State -eq "InProgress") -or ($Job_AssociateFailbackPolicy.State -eq "NotStarted")){
           sleep 10;
           $Job_AssociateFailbackPolicy = Get-ASRJob -Job $Job_AssociateFailbackPolicy
   }
   $Job_AssociateFailbackPolicy.State

   ```

## <a name="add-a-vcenter-server-and-discover-vms"></a>Ajouter un serveur vCenter et découvrir des machines virtuelles

Ajoutez un serveur vCenter à partir d’une adresse IP ou d’un nom d’hôte. Le paramètre **-port** spécifie le port auquel se connecter sur le serveur vCenter, le paramètre **-nom** spécifie un nom convivial à utiliser pour le serveur vCenter, et le paramètre **-compte** spécifie le gestionnaire de compte sur le serveur de configuration à utiliser pour découvrir les machines virtuelles managées par le serveur vCenter.

```azurepowershell
# The $AccountHandles[0] variable holds details of vCenter_account

$Job_AddvCenterServer = New-AzRecoveryServicesAsrvCenter -Fabric $ASRFabrics[0] -Name "MyvCenterServer" -IpOrHostName "10.150.24.63" -Account $AccountHandles[0] -Port 443

#Wait for the job to complete and ensure it completed successfully

while (($Job_AddvCenterServer.State -eq "InProgress") -or ($Job_AddvCenterServer.State -eq "NotStarted")) {
        sleep 30;
        $Job_AddvCenterServer = Get-ASRJob -Job $Job_AddvCenterServer
}
$Job_AddvCenterServer
```
```
Name             : 0f76f937-f9cf-4e0e-bf27-10c9d1c252a4
ID               : /Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/VMwareDRToAzurePs/providers/Microsoft.RecoveryServices/vaults/VMwareDRToAzurePs/replicationJobs/0f76f93
                   7-f9cf-4e0e-bf27-10c9d1c252a4
Type             :
JobType          : DiscoverVCenter
DisplayName      : Add vCenter server
ClientRequestId  : a2af8892-5686-4d64-a528-10445bc2f698 ActivityId: 7ec05aad-002e-4da0-991f-95d0de7a9f3a
State            : Succeeded
StateDescription : Completed
StartTime        : 11/24/2017 2:41:47 AM
EndTime          : 11/24/2017 2:44:37 AM
TargetObjectId   : 10.150.24.63
TargetObjectType : VCenter
TargetObjectName : MyvCenterServer
AllowedActions   :
Tasks            : {Adding vCenter server}
Errors           : {}
```

## <a name="create-storage-accounts-for-replication"></a>Créer des comptes de stockage pour la réplication

**Pour écrire sur le disque managé, utilisez le [module PowerShell Az.RecoveryServices 2.0.0](https://www.powershellgallery.com/packages/Az.RecoveryServices/2.0.0-preview) ou les versions ultérieures.** Il requiert uniquement la création d’un compte de stockage de journal. Il est recommandé d’utiliser un type de compte d’utilisateur standard et la redondance LRS, car elle est utilisée pour stocker uniquement les journaux temporaires. Assurez-vous que le compte de stockage est créé dans la même région Azure que le coffre.

Si vous utilisez une version du module Az.RecoveryServices antérieure à la version 2.0.0, procédez comme suit pour créer des comptes de stockage. Ces comptes de stockage sont utilisés ultérieurement pour répliquer les machines virtuelles. Assurez-vous que les comptes de stockage sont créés dans la même région Azure que le coffre. Vous pouvez ignorer cette étape si vous envisagez d’utiliser un compte de stockage existant pour la réplication.

> [!NOTE]
> Lors de la réplication de machines virtuelles locales vers un compte de stockage Premium, vous devez spécifier un compte de stockage standard supplémentaire (compte de stockage de journal). Le compte de stockage de journal  conserve des journaux d’activité de réplication en tant que stockage intermédiaire jusqu’à ce que les journaux d’activité puissent être appliqués à la cible de stockage Premium.
>

```azurepowershell

$PremiumStorageAccount = New-AzStorageAccount -ResourceGroupName "VMwareDRToAzurePs" -Name "premiumstorageaccount1" -Location "East Asia" -SkuName Premium_LRS

$LogStorageAccount = New-AzStorageAccount -ResourceGroupName "VMwareDRToAzurePs" -Name "logstorageaccount1" -Location "East Asia" -SkuName Standard_LRS

$ReplicationStdStorageAccount= New-AzStorageAccount -ResourceGroupName "VMwareDRToAzurePs" -Name "replicationstdstorageaccount1" -Location "East Asia" -SkuName Standard_LRS
```

## <a name="replicate-vmware-vms"></a>Répliquer des machines virtuelles VMware

La découverte des machines virtuelles à partir du serveur vCenter prend environ 15 à 20 minutes. Ensuite, un objet d’élément protégeable est créé dans Azure Site Recovery pour chaque machine virtuelle découverte. À cette étape, trois machines virtuelles découvertes sont répliquées vers les comptes de stockage Azure créés à l’étape précédente.

Vous aurez besoin des détails suivants pour protéger une machine virtuelle découverte :

* Élément protégeable à répliquer.
* Compte de stockage vers lequel répliquer la machine virtuelle (uniquement si vous effectuez la réplication vers un compte de stockage). 
* Un stockage de journal est nécessaire pour protéger les machines virtuelles sur un compte de stockage Premium ou un disque managé.
* Serveur de traitement à utiliser pour la réplication. La liste des serveurs de traitement disponibles a été récupérée et enregistrée dans les variables ***$ProcessServers[0]*** *(ScaleOut-ProcessServer)* et ***$ProcessServers[1]*** *(ConfigurationServer)* .
* Compte à utiliser pour installer (push) le logiciel du service Mobilité sur les machines. La liste des comptes disponibles a été récupérée et stockée dans la variable ***$AccountHandles***.
* Mappage de conteneur de protection pour la stratégie de réplication à utiliser pour la réplication.
* Groupe de ressources dans lequel les machines virtuelles doivent être créées lors du basculement.
* (Facultatif) Réseau virtuel Azure et sous-réseau auquel la machine virtuelle basculée doit être connectée.

Maintenant, répliquez les machines virtuelles suivantes à l’aide des paramètres spécifiés dans ce tableau.


|Machine virtuelle  |Serveur de traitement        |Compte de stockage              |Compte de stockage de journal  |Policy           |Compte pour l’installation du service Mobilité|Groupe de ressources cible  | Réseau virtuel cible  |Sous-réseau cible  |
|-----------------|----------------------|-----------------------------|---------------------|-----------------|-----------------------------------------|-----------------------|-------------------------|---------------|
|CentOSVM1       |ConfigurationServer   |N/A| logstorageaccount1                 |ReplicationPolicy|LinuxAccount                             |VMwareDRToAzurePs      |ASR-vnet                 |Subnet-1       |
|Win2K12VM1       |ScaleOut-ProcessServer|premiumstorageaccount1       |logstorageaccount1   |ReplicationPolicy|WindowsAccount                           |VMwareDRToAzurePs      |ASR-vnet                 |Subnet-1       |   
|CentOSVM2       |ConfigurationServer   |replicationstdstorageaccount1| N/A                 |ReplicationPolicy|LinuxAccount                             |VMwareDRToAzurePs      |ASR-vnet                 |Subnet-1       |   


```azurepowershell

#Get the target resource group to be used
$ResourceGroup = Get-AzResourceGroup -Name "VMwareToAzureDrPs"

#Get the target virtual network to be used
$RecoveryVnet = Get-AzVirtualNetwork -Name "ASR-vnet" -ResourceGroupName "asrrg" 

#Get the protection container mapping for replication policy named ReplicationPolicy
$PolicyMap  = Get-AzRecoveryServicesAsrProtectionContainerMapping -ProtectionContainer $ProtectionContainer | where PolicyFriendlyName -eq "ReplicationPolicy"

#Get the protectable item corresponding to the virtual machine CentOSVM1
$VM1 = Get-AzRecoveryServicesAsrProtectableItem -ProtectionContainer $ProtectionContainer -FriendlyName "CentOSVM1"

# Enable replication for virtual machine CentOSVM1 using the Az.RecoveryServices module 2.0.0 onwards to replicate to managed disks
# The name specified for the replicated item needs to be unique within the protection container. Using a random GUID to ensure uniqueness
$Job_EnableReplication1 = New-AzRecoveryServicesAsrReplicationProtectedItem -VMwareToAzure -ProtectableItem $VM1 -Name (New-Guid).Guid -ProtectionContainerMapping $PolicyMap -ProcessServer $ProcessServers[1] -Account $AccountHandles[2] -RecoveryResourceGroupId $ResourceGroup.ResourceId -logStorageAccountId $LogStorageAccount.Id -RecoveryAzureNetworkId $RecoveryVnet.Id -RecoveryAzureSubnetName "Subnet-1"

# Alternatively, if the virtual machine CentOSVM1 has CMK enabled disks, enable replication using Az module 3.3.0 onwards as below
# $diskID is the Disk Encryption Set ID to be used for all replica managed disks and target managed disks in the target region
$Job_EnableReplication1 = New-AzRecoveryServicesAsrReplicationProtectedItem -VMwareToAzure -ProtectableItem $VM1 -Name (New-Guid).Guid -ProtectionContainerMapping $PolicyMap -ProcessServer $ProcessServers[1] -Account $AccountHandles[2] -RecoveryResourceGroupId $ResourceGroup.ResourceId -logStorageAccountId -DiskEncryptionSetId $diskId $LogStorageAccount.Id -RecoveryAzureNetworkId $RecoveryVnet.Id -RecoveryAzureSubnetName "Subnet-1"

#Get the protectable item corresponding to the virtual machine Win2K12VM1
$VM2 = Get-AzRecoveryServicesAsrProtectableItem -ProtectionContainer $ProtectionContainer -FriendlyName "Win2K12VM1"

# Enable replication for virtual machine Win2K12VM1
$Job_EnableReplication2 = New-AzRecoveryServicesAsrReplicationProtectedItem -VMwareToAzure -ProtectableItem $VM2 -Name (New-Guid).Guid -ProtectionContainerMapping $PolicyMap -RecoveryAzureStorageAccountId $PremiumStorageAccount.Id -LogStorageAccountId $LogStorageAccount.Id -ProcessServer $ProcessServers[0] -Account $AccountHandles[1] -RecoveryResourceGroupId $ResourceGroup.ResourceId -RecoveryAzureNetworkId $RecoveryVnet.Id -RecoveryAzureSubnetName "Subnet-1"

#Get the protectable item corresponding to the virtual machine CentOSVM2
$VM3 = Get-AzRecoveryServicesAsrProtectableItem -ProtectionContainer $ProtectionContainer -FriendlyName "CentOSVM2"

# Enable replication for virtual machine CentOSVM2
$Job_EnableReplication3 = New-AzRecoveryServicesAsrReplicationProtectedItem -VMwareToAzure -ProtectableItem $VM3 -Name (New-Guid).Guid -ProtectionContainerMapping $PolicyMap -RecoveryAzureStorageAccountId $ReplicationStdStorageAccount.Id  -ProcessServer $ProcessServers[1] -Account $AccountHandles[2] -RecoveryResourceGroupId $ResourceGroup.ResourceId -RecoveryAzureNetworkId $RecoveryVnet.Id -RecoveryAzureSubnetName "Subnet-1"

```

Une fois que la tâche d’activation de la réplication est effectuée, la réplication initiale est démarrée pour les machines virtuelles. La réplication initiale peut prendre un certain temps en fonction de la quantité de données à répliquer et de la bande passante disponible pour la réplication. Une fois la réplication initiale terminée, la machine virtuelle passe dans un état protégé. Une fois que la machine est dans cet état protégé, vous pouvez effectuer un test de basculement pour la machine virtuelle, l’ajouter aux plans de récupération, etc.

Vous pouvez vérifier l’état et l’intégrité de la réplication pour la machine virtuelle avec la cmdlet Get-ASRReplicationProtectedItem.

```azurepowershell
Get-AzRecoveryServicesAsrReplicationProtectedItem -ProtectionContainer $ProtectionContainer | Select FriendlyName, ProtectionState, ReplicationHealth
```
```
FriendlyName ProtectionState                 ReplicationHealth
------------ ---------------                 -----------------
CentOSVM1    Protected                       Normal
CentOSVM2    InitialReplicationInProgress    Normal
Win2K12VM1   Protected                       Normal
```

## <a name="configure-failover-settings"></a>Configurer les paramètres de basculement

Les paramètres de basculement pour les machines protégées peuvent être mis à jour à l’aide de la cmdlet Set-ASRReplicationProtectedItem. Voici certains des paramètres concernés :
* Nom de la machine virtuelle à créer lors du basculement
* Taille de la machine virtuelle à créer lors du basculement
* Réseau virtuel Azure et sous-réseau auquel les cartes réseau de la machine virtuelle doivent être connectées lors du basculement
* Basculement vers les disques managés
* Application d’Azure Hybrid Use Benefit
* Affectation d’une adresse IP statique à partir du réseau virtuel cible à la machine virtuelle lors du basculement

Dans cet exemple, nous mettons à jour la taille de la machine virtuelle à créer lors du basculement de la machine virtuelle *Win2K12VM1* et spécifions que celle-ci utilise des disques managés lors du basculement.

```azurepowershell
$ReplicatedVM1 = Get-AzRecoveryServicesAsrReplicationProtectedItem -FriendlyName "Win2K12VM1" -ProtectionContainer $ProtectionContainer

Set-AzRecoveryServicesAsrReplicationProtectedItem -InputObject $ReplicatedVM1 -Size "Standard_DS11" -UseManagedDisk True
```
```
Name             : cafa459c-44a7-45b0-9de9-3d925b0e7db9
ID               : /Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/VMwareDRToAzurePs/providers/Microsoft.RecoveryServices/vaults/VMwareDRToAzurePs/replicationJobs/cafa459
                   c-44a7-45b0-9de9-3d925b0e7db9
Type             :
JobType          : UpdateVmProperties
DisplayName      : Update the virtual machine
ClientRequestId  : b0b51b2a-f151-4e9a-a98e-064a5b5131f3 ActivityId: ac2ba316-be7b-4c94-a053-5363f683d38f
State            : InProgress
StateDescription : InProgress
StartTime        : 11/24/2017 2:04:26 PM
EndTime          :
TargetObjectId   : 88bc391e-d091-11e7-9484-000c2955bb50
TargetObjectType : ProtectionEntity
TargetObjectName : Win2K12VM1
AllowedActions   :
Tasks            : {Update the virtual machine properties}
Errors           : {}
```

## <a name="run-a-test-failover"></a>Exécuter un test de basculement

1. Exécutez une simulation de récupération d’urgence (test de basculement), en procédant comme suit :

   ```azurepowershell
   #Test failover of Win2K12VM1 to the test virtual network "V2TestNetwork"

   #Get details of the test failover virtual network to be used
   TestFailovervnet = Get-AzVirtualNetwork -Name "V2TestNetwork" -ResourceGroupName "asrrg" 

   #Start the test failover operation
   $TFOJob = Start-AzRecoveryServicesAsrTestFailoverJob -ReplicationProtectedItem $ReplicatedVM1 -AzureVMNetworkId $TestFailovervnet.Id -Direction PrimaryToRecovery
   ```
2. Une fois le travail de test de basculement terminé, vous remarquerez qu’une machine virtuelle avec le suffixe *« -Test »* (Win2K12VM1-Test, dans ce cas) accolé à son nom est créée dans Azure.
3. Vous pouvez maintenant vous connecter à la machine virtuelle basculée de test et valider le test de basculement.
4. Supprimez le test de basculement à l’aide de la cmdlet Start-ASRTestFailoverCleanupJob. Cette opération supprime la machine virtuelle créée dans le cadre de l’opération de test de basculement.

   ```azurepowershell
   $Job_TFOCleanup = Start-AzRecoveryServicesAsrTestFailoverCleanupJob -ReplicationProtectedItem $ReplicatedVM1
   ```

## <a name="fail-over-to-azure"></a>Basculer vers Azure

À cette étape, nous basculons la machine virtuelle Win2K12VM1 vers un point de récupération spécifique.

1. Obtenez la liste des points de récupération disponibles à utiliser pour le basculement :
   ```azurepowershell
   # Get the list of available recovery points for Win2K12VM1
   $RecoveryPoints = Get-AzRecoveryServicesAsrRecoveryPoint -ReplicationProtectedItem $ReplicatedVM1
   "{0} {1}" -f $RecoveryPoints[0].RecoveryPointType, $RecoveryPoints[0].RecoveryPointTime
   ```
   ```
   CrashConsistent 11/24/2017 5:28:25 PM
   ```
   ```azurepowershell

   #Start the failover job
   $Job_Failover = Start-AzRecoveryServicesAsrUnplannedFailoverJob -ReplicationProtectedItem $ReplicatedVM1 -Direction PrimaryToRecovery -RecoveryPoint $RecoveryPoints[0]
   do {
           $Job_Failover = Get-ASRJob -Job $Job_Failover;
           sleep 60;
   } while (($Job_Failover.State -eq "InProgress") -or ($JobFailover.State -eq "NotStarted"))
   $Job_Failover.State
   ```
   ```
   Succeeded
   ```

2. Après le basculement, vous pouvez valider cette opération et configurer la réplication inverse depuis Azure vers le site VMware local.

## <a name="next-steps"></a>Étapes suivantes
Découvrez comment automatiser plusieurs tâches à l’aide de la [référence Azure Site Recovery PowerShell](/powershell/module/Az.RecoveryServices).
